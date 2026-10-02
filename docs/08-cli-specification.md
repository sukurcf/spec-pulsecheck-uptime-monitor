# PulseCheck CLI specification

Purpose: This document defines the PulseCheck command-line interface and targets-file interface. It gives exact commands, options, defaults, exit codes, output shapes, webhook payloads, and log formats.

## CLI overview

The console entry point is `pulsecheck`. Commands use Typer. All commands MUST validate the targets file before they use it, except `init` and commands that only show help.

```text
$ pulsecheck --help
Usage: pulsecheck [OPTIONS] COMMAND [ARGS]...

Commands:
  init      Write a sample targets file.
  validate  Validate a targets file without running checks.
  check     Run one concurrent check for selected targets.
  run       Start the continuous scheduler.
  status    Show latest stored state for configured targets.
  history   Show stored check results.
  report    Produce uptime and incident reports.
  purge     Delete old retained evidence rows.
```

## Global options

| Option | Type | Default | Applies to | Description |
|---|---|---|---|---|
| `--config <path>` | path | `targets.yml` | all except `init` | Targets file path. |
| `--database <path>` | path | `.pulsecheck/pulsecheck.db` | all storage commands | SQLite database path. |
| `--verbose` | flag | false | all commands | Show debug logs and retry details. |
| `--quiet` | flag | false | all commands | Hide non-essential logs. |
| `--log-format <format>` | enum | `text` | all commands | `text` or `json`. |
| `--version` | flag | false | root command | Print SemVer package version. |

`--verbose` and `--quiet` MUST NOT be used together. The command MUST exit 2 with `verbosity | conflict | --verbose and --quiet cannot be used together`.

## Exit codes

| Code | Meaning | Commands |
|---:|---|---|
| 0 | Command succeeded and all selected targets are `UP` when target state applies. | all |
| 1 | At least one selected target is `DOWN`, `DEGRADED`, or `UNKNOWN`. | `check`, `status`, `run` at shutdown summary |
| 2 | Configuration, validation, or usage error. | all |
| 3 | Internal error such as database migration failure. | all |

## Targets file format

The targets file is YAML. It is a product interface that students and users write directly.

```yaml
global:
  concurrency: 50
  retries: 2
  timeout_seconds: 5.0
  interval_seconds: 60
  latency_threshold_ms: 1500
  incident_open_after_down: 3
  incident_close_after_recovered: 2
  notification_cooldown_minutes: 30
  notification_channels:
    - console
    - webhook
targets:
  - name: college-home
    url: https://www.example.in/
    method: GET
    expected_status_codes: [200]
    keyword: Welcome
    timeout_seconds: 3.0
    interval_seconds: 60
    latency_threshold_ms: 1200
    tags: [public, website]
    headers:
      User-Agent: PulseCheck/1.0
  - name: admissions-api
    url: https://api.example.in/health
    method: HEAD
    expected_status_codes: [200, 204]
    tags: [public, api]
  - name: local-status
    url: http://localhost:8080/status
    method: GET
    expected_status_codes: [200]
    interval_seconds: 120
    tags: [internal]
```

## Field reference

| Field | Type | Required | Default | Limits and rules |
|---|---|---|---|---|
| `global.concurrency` | integer | no | 10 | 1 to 200 active network attempts. |
| `global.retries` | integer | no | 2 | 0 to 5 extra attempts. |
| `global.timeout_seconds` | decimal seconds | no | 5.0 | 0.1 to 60.0 per attempt. |
| `global.interval_seconds` | integer seconds | no | 60 | 10 to 86400 scheduler interval. |
| `global.latency_threshold_ms` | integer ms | no | 1500 | 1 to 60000. Above this is `DEGRADED`. |
| `global.incident_open_after_down` | integer | no | 3 | 1 to 10 consecutive `DOWN` results. |
| `global.incident_close_after_recovered` | integer | no | 2 | 1 to 10 consecutive non-`DOWN` results. |
| `global.notification_cooldown_minutes` | integer minutes | no | 30 | 1 to 1440. |
| `global.notification_channels` | list enum | no | `[console]` | Allowed values: `console`, `webhook`, `smtp`. |
| `global.webhook_format` | enum | no | `generic` | Applies to webhook channel. Values: `generic`, `slack`, `discord`. |
| `targets[].name` | string | yes | none | Unique slug, 1 to 64 letters, numbers, dash, or underscore. |
| `targets[].url` | string | yes | none | Must start with `http://` or `https://`. |
| `targets[].method` | enum | no | `GET` | `GET` or `HEAD`. |
| `targets[].expected_status_codes` | list integer | no | `[200]` | Non-empty, each value 100 to 599. |
| `targets[].keyword` | string | no | none | 1 to 200 chars. Invalid with `method: HEAD`. |
| `targets[].timeout_seconds` | decimal seconds | no | global value | 0.1 to 60.0. |
| `targets[].interval_seconds` | integer seconds | no | global value | 10 to 86400. |
| `targets[].latency_threshold_ms` | integer ms | no | global value | 1 to 60000. |
| `targets[].tags` | list string | no | empty list | Each tag is a non-empty string. |
| `targets[].headers` | map string | no | empty map | Extra HTTP headers. Secret values MUST be redacted. |

Validation errors MUST use this line shape:

```text
path | code | reason
```

Example validation error output:

```text
targets[0].keyword | keyword_not_allowed_for_head | keyword is not allowed when method is HEAD
global.concurrency | invalid_range | must be between 1 and 200
```

## Command: `init`

Purpose: write a sample targets file.

Arguments: none.

| Option | Type | Default | Description |
|---|---|---|---|
| `--output <path>` | path | `targets.yml` | File to write. |
| `--force` | flag | false | Overwrite an existing file. |

Exit codes: 0 when the file is written; 2 when the file exists and `--force` is absent; 3 for internal write errors.

```console
$ pulsecheck init --output targets.yml
Wrote sample targets file: targets.yml
Next step: pulsecheck validate --config targets.yml
```

```console
$ pulsecheck init --output targets.yml
targets.yml already exists; use --force to overwrite
```

## Command: `validate`

Purpose: validate a targets file without network checks.

Arguments: none.

Options: global options only.

Exit codes: 0 for valid configuration; 2 for validation or YAML errors; 3 for internal errors.

```console
$ pulsecheck validate --config targets.yml
Configuration valid: 3 targets
```

```console
$ pulsecheck validate --config bad.yml
file | yaml_parse_error | could not parse targets file
targets[0].url | invalid_url_scheme | must start with http:// or https://
```

## Command: `check`

Purpose: run one concurrent check for each selected target and store final outcomes.

Arguments: none.

| Option | Type | Default | Description |
|---|---|---|---|
| `--tag <tag>` | string | none | Check only targets that have this tag. |
| `--format <format>` | enum | `table` | `table` or `json`. |
| `--no-store` | flag | false | Run checks without storing results. Used only for diagnosis. |

Exit codes: 0 when all selected targets are `UP`; 1 when any selected target is `DOWN` or `DEGRADED`; 2 when no target matches or config is invalid; 3 for internal errors.

```console
$ pulsecheck check --config targets.yml --tag public
Target          Status     Latency ms  Attempts  Reason
college-home    UP         118         1         -
admissions-api  DEGRADED   1640        1         latency above 1500 ms

Summary: 1 UP, 1 DEGRADED, 0 DOWN
Exit code: 1
```

JSON output shape:

```json
{
  "checked_at": "2026-10-02T07:00:00Z",
  "summary": {"up": 1, "degraded": 1, "down": 1},
  "results": [
    {
      "target_name": "college-home",
      "url": "https://www.example.in/",
      "status": "UP",
      "latency_ms": 118,
      "attempt_count": 1,
      "error_kind": null,
      "correlation_id": "01K6K8Q1A4N8J2G7S4Q0V6Y9BP"
    },
    {
      "target_name": "admissions-api",
      "url": "https://api.example.in/health",
      "status": "DEGRADED",
      "latency_ms": 1640,
      "attempt_count": 1,
      "error_kind": null,
      "correlation_id": "01K6K8R2V9P2J2G7S4Q0V6Y9BQ"
    },
    {
      "target_name": "fee-portal",
      "url": "https://fees.example.in/health",
      "status": "DOWN",
      "latency_ms": null,
      "attempt_count": 3,
      "error_kind": "timeout",
      "correlation_id": "01K6K8TSQJBR3C7BFSM8XQEX2M"
    }
  ]
}
```

## Command: `run`

Purpose: start the continuous scheduler.

Arguments: none.

| Option | Type | Default | Description |
|---|---|---|---|
| `--once` | flag | false | Run one scheduler tick and exit. This supports smoke tests. |
| `--tag <tag>` | string | none | Schedule only targets that have this tag. |

Exit codes: 0 after graceful shutdown when every selected target's latest state is `UP`; 1 when any selected target is `DOWN`, `DEGRADED`, or `UNKNOWN`; 2 for config errors; 3 for internal errors.

```console
$ pulsecheck run --config targets.yml
PulseCheck scheduler started: 3 targets, concurrency 50
2026-10-02T07:00:00Z college-home UP latency_ms=118 attempts=1
2026-10-02T07:00:00Z admissions-api DOWN reason=timeout attempts=3
2026-10-02T07:00:00Z local-status UP latency_ms=35 attempts=1
Shutdown requested; stored completed checks
Summary: 2 UP, 0 DEGRADED, 1 DOWN, 0 missed checks
```

Scheduler rules:

- Each target uses its effective `interval_seconds`.
- A target MUST NOT overlap with its own prior check.
- A due target that is still running creates one missed-check record.
- Ctrl+C and SIGTERM stop new work and wait up to 5 seconds.
- Missed checks are stored in `missed_checks`, not in `check_results`.

## Command: `status`

Purpose: show latest stored state for each configured target. It does not run network checks.

Arguments: none.

| Option | Type | Default | Description |
|---|---|---|---|
| `--format <format>` | enum | `table` | `table` or `json`. |

Exit codes: 0 when all configured targets have latest state `UP`; 1 when any target is `DOWN`, `DEGRADED`, or `UNKNOWN`; 2 for config errors; 3 for internal errors.

```console
$ pulsecheck status --config targets.yml
Target          Status   Checked at             Latency ms  Open incident  SSL days
college-home    UP       2026-10-02T07:00:00Z   118         -              42
new-api         UNKNOWN  -                      -           -              -
```

## Command: `history`

Purpose: list stored check results.

Arguments: none.

| Option | Type | Default | Description |
|---|---|---|---|
| `--target <name>` | string | none | Include one target. |
| `--since <datetime>` | ISO 8601 datetime | none | Inclusive lower bound. Converted to UTC. |
| `--until <datetime>` | ISO 8601 datetime | none | Exclusive upper bound. Converted to UTC. |
| `--limit <n>` | integer | 100 | 1 to 1000 newest rows. |
| `--format <format>` | enum | `table` | `table`, `json`, or Should `csv`. |

Exit codes: 0 for success, including empty results; 2 for invalid filters; 3 for internal errors.

```console
$ pulsecheck history --target fee-portal --limit 3
Checked at             Target      Status    Latency ms  Attempts  Reason
2026-10-02T07:04:00Z   fee-portal  DOWN      -           3         timeout
2026-10-02T07:03:00Z   fee-portal  DOWN      -           3         timeout
2026-10-02T07:02:00Z   fee-portal  DEGRADED  1800        1         -
```

JSON output shape:

```json
{
  "filters": {
    "target": "fee-portal",
    "since": "2026-10-02T00:00:00Z",
    "until": "2026-10-03T00:00:00Z",
    "limit": 3
  },
  "rows": [
    {
      "id": "33333333-3333-4333-8333-333333333333",
      "target_name": "fee-portal",
      "url": "https://fees.example.in/health",
      "method": "GET",
      "checked_at": "2026-10-02T07:04:00Z",
      "status": "DOWN",
      "latency_ms": null,
      "attempt_count": 3,
      "error_kind": "timeout",
      "actual_status_code": null,
      "correlation_id": "01K6K8TSQJBR3C7BFSM8XQEX2M"
    }
  ]
}
```

## Command: `report`

Purpose: produce uptime, latency, incident, missed-check, and MTTR metrics for a period.

Arguments: none.

| Option | Type | Default | Description |
|---|---|---|---|
| `--target <name>` | string | none | Report one target. |
| `--since <datetime>` | ISO 8601 datetime | required | Inclusive lower bound. |
| `--until <datetime>` | ISO 8601 datetime | current time | Exclusive upper bound. |
| `--format <format>` | enum | `markdown` | `markdown`, `json`, or Should `csv` and `html`. |
| `--output <path>` | path | stdout | Write report to a file. |

Exit codes: 0 for successful report; 2 for invalid period or output path; 3 for internal errors.

```console
$ pulsecheck report --since 2026-10-01T00:00:00Z --until 2026-10-08T00:00:00Z
# PulseCheck report

Period: 2026-10-01T00:00:00Z to 2026-10-08T00:00:00Z
Time zone: UTC
Completed checks: 100
Missed checks: 3
Uptime: 96.00%
Average latency: 200 ms
P95 latency: 1700 ms
Incidents: 3 closed, 0 open
MTTR: 900 seconds
```

JSON output shape:

```json
{
  "period": {
    "since": "2026-10-01T00:00:00Z",
    "until": "2026-10-08T00:00:00Z",
    "time_zone": "UTC"
  },
  "scope": {"target": null},
  "summary": {
    "completed_checks": 100,
    "missed_checks": 3,
    "up": 90,
    "degraded": 6,
    "down": 4,
    "uptime_percent": 96.0,
    "average_latency_ms": 200,
    "p95_latency_ms": 1700,
    "closed_incident_count": 3,
    "open_incident_count": 0,
    "mttr_seconds": 900
  },
  "targets": [
    {
      "target_name": "college-home",
      "completed_checks": 50,
      "missed_checks": 1,
      "up": 49,
      "degraded": 0,
      "down": 1,
      "uptime_percent": 98.0,
      "average_latency_ms": 120,
      "p95_latency_ms": 250,
      "incident_count": 1,
      "mttr_seconds": 600
    },
    {
      "target_name": "fee-portal",
      "completed_checks": 30,
      "missed_checks": 2,
      "up": 21,
      "degraded": 6,
      "down": 3,
      "uptime_percent": 90.0,
      "average_latency_ms": 420,
      "p95_latency_ms": 1700,
      "incident_count": 2,
      "mttr_seconds": 1050
    },
    {
      "target_name": "local-status",
      "completed_checks": 20,
      "missed_checks": 0,
      "up": 20,
      "degraded": 0,
      "down": 0,
      "uptime_percent": 100.0,
      "average_latency_ms": 100,
      "p95_latency_ms": 100,
      "incident_count": 0,
      "mttr_seconds": null
    }
  ]
}
```

## Command: `purge`

Purpose: delete old retained evidence rows.

Arguments: none.

| Option | Type | Default | Description |
|---|---|---|---|
| `--older-than <days>` | integer days | required | Delete eligible rows older than this number of days. Minimum 1. |
| `--dry-run` | flag | false | Show counts without deleting rows. |

Exit codes: 0 for successful purge; 2 for invalid days; 3 for internal errors.

```console
$ pulsecheck purge --older-than 90
Cutoff: 2026-07-04T07:00:00Z
check_results deleted: 1240
missed_checks deleted: 3
ssl_certificate_observations deleted: 44
incidents deleted: 3
notifications_sent deleted: 9
Open incidents preserved: 1
```

```console
$ pulsecheck purge --older-than 0
older_than | invalid_range | must be at least 1 day
```

## Result classification interface

All output uses these status values:

| Status | Meaning | Availability effect |
|---|---|---|
| `UP` | HTTP check succeeded and latency is within threshold. | Counts as available. |
| `DEGRADED` | HTTP check succeeded but latency exceeded `latency_threshold_ms`. | Counts as available. |
| `DOWN` | Connection, DNS, TLS, timeout, status, or keyword failure. | Counts as unavailable. |
| `UNKNOWN` | Display-only state for configured target with no result. | Makes `status` exit 1. |

Error kind values are `connection`, `dns`, `tls`, `timeout`, `status`, `keyword`, `hostname`, and `internal`.

## Webhook notification payload

The webhook channel sends one JSON POST. The URL comes from an environment variable, not from the sample file. The `global.webhook_format` value selects the body shape:

- `generic` sends the full PulseCheck JSON body below.
- `slack` sends `text` with the summary and stable `notification_id`.
- `discord` sends `content` with the summary and stable `notification_id`.

```json
{
  "notification_id": "notif_01K6K92P9T3GZCK9ADW6Q61CXR",
  "dedupe_key": "incident:44444444-4444-4444-8444-444444444444:opened",
  "event_type": "incident_opened",
  "severity": "critical",
  "target_name": "fee-portal",
  "target_url": "https://fees.example.in/health",
  "incident_id": "44444444-4444-4444-8444-444444444444",
  "status": "DOWN",
  "started_at": "2026-10-02T07:04:00Z",
  "checked_at": "2026-10-02T07:06:00Z",
  "summary": "PulseCheck incident opened for fee-portal after 3 DOWN checks",
  "correlation_id": "01K6K92P9TQ5R7M9KZ4YC5DV31"
}
```

For SSL expiry warnings, `event_type` is `ssl_expiry_warning`, `severity` is `warning`, and the payload includes `hostname`, `days_to_expiry`, `warning_threshold`, and `fingerprint_sha256`.

## JSON log line format

When `--log-format json` is set, each log line is one JSON object.

```json
{"timestamp":"2026-10-02T07:00:00Z","level":"INFO","event":"check_completed","target_name":"college-home","status":"UP","latency_ms":118,"attempt_count":1,"correlation_id":"01K6K8Q1A4N8J2G7S4Q0V6Y9BP","message":"target check completed"}
```

Required fields:

| Field | Type | Description |
|---|---|---|
| `timestamp` | string | UTC ISO 8601 timestamp. |
| `level` | string | `DEBUG`, `INFO`, `WARNING`, or `ERROR`. |
| `event` | string | Stable event name such as `check_completed`. |
| `target_name` | string or null | Target when the event belongs to one target. |
| `status` | string or null | `UP`, `DEGRADED`, `DOWN`, or null. |
| `latency_ms` | integer or null | Final attempt latency when known. |
| `attempt_count` | integer or null | Attempts used for the final outcome. |
| `correlation_id` | string | Stable ID for one check or command. |
| `message` | string | Human-readable summary without secrets. |

## Environment variables

| Name | Required | Used by | Description |
|---|---|---|---|
| `PULSECHECK_WEBHOOK_URL` | yes for webhook | webhook notifier | Slack-compatible or Discord incoming webhook URL. |
| `PULSECHECK_SMTP_HOST` | Should | SMTP notifier | SMTP host for Should e-mail support. |
| `PULSECHECK_SMTP_USERNAME` | Should | SMTP notifier | SMTP username. |
| `PULSECHECK_SMTP_PASSWORD` | Should | SMTP notifier | SMTP password. Must be redacted. |

## Date, time, and numeric formats

- All stored timestamps MUST use UTC ISO 8601 with `Z`.
- CLI inputs MAY include offsets such as `2026-10-02T00:00:00+05:30`.
- Inputs with offsets MUST be converted to UTC before filtering.
- Latency values are integer milliseconds.
- Uptime percentages show two decimals in human output.
- JSON percentages use numbers, not strings.

[Back to README](../README.md)
