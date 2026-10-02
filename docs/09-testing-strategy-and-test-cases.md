# PulseCheck testing strategy and test cases

Purpose: This document defines the PulseCheck test approach, tools, environments, data, coverage gates, concrete test cases, traceability, performance conditions, and defect report fields.

## Test strategy

PulseCheck tests MUST prove that the CLI is safe for automation and useful during incidents. The pyramid favours fast pure-logic tests, then integration tests with local services, then focused CLI and performance tests.

Maintain at least **45 meaningful cases**; the catalog remains above that minimum. Version 1.1 replaces page/dashboard evidence with Python statistics/replay validation and adds TC-LOCAL-001 through TC-LOCAL-004 as mandatory local acceptance.

| Level | Approximate count | Main purpose | Examples |
|---|---:|---|---|
| Unit | 24 | Exhaust pure decisions and boundary rules. | Classification, incident state, p95, uptime, SSL evaluator, validation. |
| Integration | 14 | Test local I/O with real libraries. | pytest-httpserver, trustme TLS, SQLite repository, webhook dispatch. |
| CLI | 12 | Verify Typer commands, output, and exit codes. | `init`, `validate`, `check`, `status`, `history`, `report`, `purge`. |
| Security | 4 | Verify safe defaults and redaction. | TLS verification, secret logs, pip-audit, Gitleaks. |
| Performance and recovery | 4 | Verify concurrency, shutdown, and scale. | 200 targets under 10 seconds, signal shutdown. |
| Local acceptance | 4 | Prove the implementation operation contract. | Clean startup, offline demo, persisted restart, dependency/input failures. |

## Test tools and reference versions

| Tool | Reference line | Used for |
|---|---|---|
| Python | 3.12.x baseline; CI also 3.13.x | Runtime and CI matrix. |
| uv | 0.12.x | Dependency locking and test commands. |
| pytest | 9.1.x | Main test runner. |
| pytest-cov / coverage | 7.1.x / 7.16.x | Line and branch coverage gates. |
| pytest-asyncio | 1.4.x | Async checker and scheduler tests. |
| AnyIO | 4.15.x | Optional async test backend. |
| httpx / respx | 0.28.x / 0.23.x | HTTP client and mocked HTTP tests. |
| pytest-httpserver | 1.1.x | Real local HTTP server tests. |
| trustme | 1.2.x | TLS certificates for valid, expired, and wrong-host tests. |
| time-machine | 3.5.x | Time-controlled incident and cooldown tests. |
| Hypothesis | 6.x | Should property tests for calculators. |
| Ruff | 0.16.x | Lint and format checks. |
| mypy | 2.4.x | Strict type checking for the application package. |
| pip-audit | 2.10.x | Dependency vulnerability scan. |
| Gitleaks | 8.30.x | Secret scanning. |

## Test environments

| Environment | Operating system | Python versions | Required services | Purpose |
|---|---|---|---|---|
| Developer local lite | Windows 11 WSL2 Ubuntu, macOS, or Linux | 3.12.x | SQLite and prepared localhost HTTP/TLS/webhook fixtures | Daily development and local acceptance. |
| CI Ubuntu | Ubuntu runner | 3.12.x and 3.13.x | Local SQLite, local HTTP, local TLS, and local webhook fixtures | Main gate for tests and coverage. |
| CI Windows | Windows runner | 3.12.x and 3.13.x | Local SQLite, local HTTP, local TLS, and local webhook fixtures | Same portable tests as Ubuntu; SIGTERM is excluded. |
| Standard optional | 16 GB laptop | 3.12.x | Optional Mailpit and local test target | Should SMTP and Docker checks. |

All portable unit, integration, CLI, local HTTP, TLS, and webhook fixture tests MUST run on Ubuntu and Windows with Python 3.12.x and 3.13.x. Only the SIGTERM subprocess test is Ubuntu-only. Windows MUST cover Ctrl+C shutdown through KeyboardInterrupt.

## Test data strategy

- Targets fixtures MUST use `example.in`, `example.com`, `localhost`, or pytest-httpserver URLs.
- SQLite tests MUST use isolated database files created under the test workspace.
- Incident tests MUST control time with time-machine.
- TLS tests MUST use trustme certificates for valid, expired, and wrong-host cases.
- HTTP tests MUST use pytest-httpserver for local success, delay, status, and body fixtures.
- Webhook tests MUST use a local HTTP receiver and fixed payload assertions.
- Performance tests MUST generate 200 local targets with deterministic names `target-001` to `target-200`.
- No test MAY use real personal data, real company endpoints, or internet calls.
- DNS failure tests MUST stub the resolver, not query an external DNS server. Example-domain strings are validation/payload data, not endpoints to contact.
- Store the recorded five-result seed fixture from document 06 under `tests/fixtures/`; prepare trustme certificate files and dependency caches before network isolation. Loopback remains available; external egress MUST be denied during local acceptance.

## Coverage thresholds

| Scope | Measured packages | Line threshold | Branch threshold | Enforced separately | Exclusions |
|---|---|---:|---:|---|---|
| Whole package | `pulsecheck` | 90% | 80% | Yes | tests, generated files, documented `__main__` blocks only; implemented optional Python modules remain measured. |
| Classifier module | `pulsecheck.checks.classifier` classifier decisions | 100% | 100% | Yes | none. |
| Incident engine | `pulsecheck.incidents.engine` incident transitions | 100% | 100% | Yes | none. |
| Uptime calculator | `pulsecheck.reports.uptime` availability formula | 100% | 100% | Yes | none. |
| Percentile calculator | `pulsecheck.reports.percentiles` nearest-rank math | 100% | 100% | Yes | none. |
| SSL expiry evaluator | `pulsecheck.checks.ssl_eval` certificate rules | 100% | 100% | Yes | none. |

The CI build MUST fail when a line threshold or a branch threshold is below its configured value.

## Test case catalog

| ID | Title | Type | Priority | Linked requirement IDs | Preconditions | Steps | Test data | Expected result |
|---|---|---|---|---|---|---|---|---|
| TC-UT-001 | Classify HTTP 200 as UP | Unit | Must | FR-CHECK-03 | Classify HTTP 200 as UP fixture is ready. | Call classifier with final response status 200, expected `[200]`, no keyword, latency 120 ms, threshold 1500 ms. | Target `college-home`. | Result status is `UP`, `error_kind` is null, and availability flag is true. |
| TC-UT-002 | Classify unexpected status as DOWN | Unit | Must | FR-CHECK-03 | Classify unexpected status as DOWN fixture is ready. | Call classifier with final response status 503, expected `[200]`, no keyword, latency 80 ms. | Target `fee-portal`. | Result status is `DOWN`, `error_kind` is `status`, and reason text contains `expected [200], got 503`. |
| TC-UT-003 | Classify missing keyword as DOWN | Unit | Must | FR-CHECK-03 | Classify missing keyword as DOWN fixture is ready. | Call classifier with status 200, body `Service ready`, keyword `Welcome`, and latency 90 ms. | Target `college-home`. | Result status is `DOWN`, `error_kind` is `keyword`, and `keyword_matched` is false. |
| TC-UT-004 | Classify slow success as DEGRADED | Unit | Must | FR-CHECK-03 | Classify slow success as DEGRADED fixture is ready. | Call classifier with status 200, expected `[200]`, no keyword, latency 1600 ms, threshold 1500 ms. | Target `admissions-api`. | Result status is `DEGRADED`, `error_kind` is null, and availability flag is true. |
| TC-UT-005 | Classify timeout exception as DOWN | Unit | Must | FR-CHECK-03 | Classifier accepts transport errors. | Call classifier with final error kind `timeout`, no HTTP status, and attempt count 3. | Timeout after 0.5 seconds. | Result status is `DOWN`, `error_kind` is `timeout`, and `latency_ms` is null. |
| TC-UT-006 | Open incident after three DOWN results | Unit | Must | FR-INC-01 | Time is frozen at `2026-10-02T07:00:00Z`. | Process results `DOWN`, `DOWN`, `DOWN` for `fee-portal` at one-minute intervals. | Open threshold 3. | One incident opens on third result with `started_at` `2026-10-02T07:00:00Z`. |
| TC-UT-007 | Do not open after interrupted DOWN sequence | Unit | Must | FR-INC-01 | Incident engine has no open incident. | Process `DOWN`, `UP`, `DOWN` for `api-health`. | Open threshold 3. | No incident exists and current down count is 1 after the final result. |
| TC-UT-008 | Close incident after two non-DOWN results | Unit | Must | FR-INC-01 | Open incident started at `2026-10-02T07:00:00Z`. | Process `DEGRADED` at `07:05:00Z`, then `UP` at `07:06:00Z`. | Close threshold 2. | Incident state becomes `closed`, `ended_at` is `2026-10-02T07:05:00Z`, and recovery count is 2. |
| TC-UT-009 | Reset recovery on DOWN during incident | Unit | Must | FR-INC-01 | Open incident is recovering with recovery count 1. | Process `DOWN` for the same target at `2026-10-02T07:08:00Z`. | Target `fee-portal`. | Incident remains `open`, recovery count becomes 0, and down count becomes 1. |
| TC-UT-010 | Calculate uptime with degraded available | Unit | Must | FR-CLI-07 | Calculate uptime with degraded available fixture is ready. | Calculate uptime for 90 UP, 6 DEGRADED, 4 DOWN, and 3 missed checks. | Completed checks 100. | Uptime is exactly `96.00`, denominator is 100, and missed count is 3. |
| TC-UT-011 | Calculate empty-period uptime | Unit | Must | FR-REPORT-03 | Calculate empty-period uptime fixture is ready. | Calculate report metrics for zero completed checks and 2 missed checks. | Empty period. | Uptime is null, completed checks is 0, and missed checks is 2. |
| TC-UT-012 | Calculate nearest-rank p95 | Unit | Must | FR-CLI-07 | Calculate nearest-rank p95 fixture is ready. | Calculate p95 for latencies `100, 120, 200, 800, 1000`. | Five values. | Rank is 5 and p95 latency is `1000` ms. |
| TC-UT-013 | Calculate nearest-rank p95 for one value | Unit | Must | FR-REPORT-02 | Calculate nearest-rank p95 for one value fixture is ready. | Calculate p95 for one latency value `240`. | One value. | Rank is 1 and p95 latency is `240` ms. |
| TC-UT-014 | Calculate MTTR average | Unit | Must | FR-INC-02 | Report calculator has closed incident durations. | Calculate MTTR for incidents lasting 600 seconds and 1200 seconds. | Two closed incidents. | MTTR is exactly 900 seconds and open incidents are excluded. |
| TC-UT-015 | Evaluate SSL 30-day warning | Unit | Must | FR-SSL-02 | SSL evaluator has no prior warnings. | Evaluate hostname `www.example.in`, certificate fingerprint `abc123`, and 30 days to expiry. | Target `college-home`. | One warning event has threshold 30 and dedupe key `ssl:www.example.in:abc123:30`. |
| TC-UT-016 | Suppress duplicate SSL threshold | Unit | Must | FR-SSL-02 | SSL evaluator has existing warning for hostname `www.example.in` and threshold 30. | Evaluate same hostname, same fingerprint, and 30 days to expiry again. | Existing dedupe key `ssl:www.example.in:abc123:30`. | No new warning event is returned and prior warning count stays 1. |
| TC-UT-017 | Validate HEAD keyword conflict | Unit | Must | FR-CFG-02 | Validate HEAD keyword conflict fixture is ready. | Validate one target with `method: HEAD` and `keyword: Welcome`. | Path `targets[0].keyword`. | Error line is `targets[0].keyword \| keyword_not_allowed_for_head \| keyword is not allowed when method is HEAD`. |
| TC-UT-018 | Validate multiple config errors in one pass | Unit | Must | FR-CFG-02 | Validate multiple config errors in one pass fixture is ready. | Validate `global.concurrency: 0` and target interval `5`. | Invalid YAML object. | Two errors are returned: `global.concurrency` range and `targets[0].interval_seconds` range. |
| TC-UT-019 | Validate duplicate target names | Unit | Must | FR-CFG-02 | Validate duplicate target names fixture is ready. | Validate two targets both named `college-home`. | Two-target config. | Error contains `targets[1].name \| duplicate_target_name \| must be unique`. |
| TC-UT-020 | Validate retry boundaries | Unit | Must | FR-CHECK-02 | Validate retry boundaries fixture is ready. | Validate `global.retries: -1`, then validate `global.retries: 6`. | Two config objects. | First error says `must be between 0 and 5`; second error uses the same path with value 6. |
| TC-IT-001 | Store final outcome only after retries | Integration | Must | FR-CHECK-02 | SQLite database is empty. | Configure retries 2; local server fails twice with connection error and returns 200 on third attempt; run one check. | Target `retry-api`. | `check_results` has exactly 1 row with status `UP` and `attempt_count` 3. |
| TC-IT-002 | Enforce concurrency limit of two | Integration | Must | FR-CHECK-01 | A local threaded pytest-httpserver or ASGI test server records active requests. | Run 5 local targets with `global.concurrency: 2`, `global.retries: 0`, and each response delayed 200 ms. | Targets `target-1` to `target-5`. | Maximum simultaneous active requests observed by a server that can serve at least 50 concurrent requests is 2. |
| TC-IT-003 | Continue after one target DNS failure | Integration | Must | FR-CHECK-01 | One local healthy server runs; resolver stub raises a DNS error for `missing.invalid`. | Run the bad-DNS target and local 200 endpoint with external egress denied. | Targets `bad-dns`, `college-home`. | `bad-dns` is `DOWN` with `dns`; `college-home` is `UP`; both are in the summary and no external DNS query occurs. |
| TC-IT-004 | Store incident and result in one transaction | Integration | Must | FR-STORE-01 | SQLite repository starts a writable transaction. | Process the third DOWN result that opens an incident. | Target `api-health`. | One check row and one incident row commit together; forced exception before commit leaves neither row. |
| TC-IT-005 | Purge old evidence and preserve open incident | Integration | Must | FR-STORE-02 | Database has old rows and one open incident. | Run purge repository operation with cutoff 90 days. | 2 old check rows, 3 expired missed checks by `scheduled_at`, 1 old SSL row, 1 open incident. | Deletes 2 `check_results` rows, 3 `missed_checks` rows, and 1 SSL row; open incident count remains 1. |
| TC-IT-006 | Query history with UTC conversion | Integration | Must | FR-CLI-06 | Database has rows around midnight UTC. | Query history since `2026-10-02T00:00:00+05:30` and until `2026-10-03T00:00:00+05:30`. | Target `fee-portal`. | Filter bounds convert to UTC and return only rows from `2026-10-01T18:30:00Z` inclusive. |
| TC-IT-007 | trustme valid certificate is accepted | Integration | Must | FR-SSL-01 | trustme HTTPS server has certificate for `localhost`. | Run HTTPS check against `https://localhost:<port>/health`. | Certificate expires in 20 days. | Target status is not `DOWN`, SSL observation has `hostname_match` true and `days_to_expiry` 20. |
| TC-IT-008 | trustme expired certificate fails | Integration | Must | FR-SSL-01 | trustme HTTPS server presents expired certificate. | Run HTTPS check with verification enabled. | Expired localhost certificate. | Target status is `DOWN`, `error_kind` is `tls`, and no verification bypass occurs. |
| TC-IT-009 | trustme wrong-host certificate fails | Integration | Must | FR-SSL-01 | HTTPS server certificate has SAN `wrong.example.in` and no SAN for `localhost`. | Run check against `https://localhost:<port>/health`. | Wrong-host certificate. | Target status is `DOWN`, `error_kind` is `tls`, and no SSL observation is stored. |
| TC-IT-010 | Webhook incident payload succeeds | Integration | Must | FR-NOTIF-01 | Local webhook receiver returns 204. | Open an incident and dispatch webhook with `PULSECHECK_WEBHOOK_URL` set. | Incident ID `44444444-4444-4444-8444-444444444444`. | Receiver gets one JSON POST with event `incident_opened`, stable `notification_id`, and matching `dedupe_key`. |
| TC-IT-011 | Webhook failure does not close incident | Integration | Must | FR-NOTIF-01 | Local webhook receiver returns 500. | Dispatch incident-opened notification. | Target `fee-portal`. | Notification row status is `failed`, attempt count is 1, and incident state remains `open`. |
| TC-IT-012 | Cooldown suppresses reminder at ten minutes | Integration | Must | FR-NOTIF-02 | Prior webhook sent at `2026-10-02T07:00:00Z`. | Freeze time at `07:10:00Z` and process another DOWN for same open incident. | Cooldown 30 minutes. | No webhook request is sent and notification status is `suppressed`. |
| TC-IT-013 | Cooldown allows reminder at thirty-one minutes | Integration | Must | FR-NOTIF-02 | Prior webhook sent at `2026-10-02T07:00:00Z`. | Freeze time at `07:31:00Z` and process another DOWN for same open incident. | Cooldown 30 minutes. | One `incident_reminder` webhook is sent with a new stable notification ID. |
| TC-IT-014 | SQLite migration failure exits internal error | Integration | Must | FR-CLI-08 | Repository migration is configured to raise a migration error. | Start any storage command through the application service. | Command `status`. | Command result uses exit code 3 and prints `Internal error: database migration failed`. |
| TC-CLI-001 | init writes sample file | CLI | Must | FR-CLI-01 | CliRunner isolated workspace has no `targets.yml`. | Execute the init writes sample file command: `pulsecheck init --output targets.yml`. | Output path `targets.yml`. | Exit code is 0, file exists, and stdout contains `Wrote sample targets file: targets.yml`. |
| TC-CLI-002 | init refuses overwrite without force | CLI | Must | FR-CLI-01 | CliRunner workspace already has `targets.yml`. | Execute the init refuses overwrite without force command: `pulsecheck init --output targets.yml`. | Existing file contains `sentinel`. | Exit code is 2, stdout contains `targets.yml already exists; use --force to overwrite`, and file content remains `sentinel`. |
| TC-CLI-003 | validate valid file | CLI | Must | FR-CLI-02 | Valid file has three targets. | Execute the validate valid file command: `pulsecheck validate --config targets.yml`. | Complete example config. | Exit code is 0 and stdout is `Configuration valid: 3 targets`. |
| TC-CLI-004 | validate malformed YAML | CLI | Must | FR-CLI-02 | Config file has invalid YAML syntax. | Execute the validate malformed yaml command: `pulsecheck validate --config bad.yml`. | Content `global: [`. | Exit code is 2 and stdout contains `file | yaml_parse_error | could not parse targets file`. |
| TC-CLI-005 | check filters by tag | CLI | Must | FR-CLI-03 | Config has `public` and `internal` targets. | Execute the check filters by tag command: `pulsecheck check --tag public`. | Two public targets and one internal target. | Output table includes both public target names and excludes `local-status`. |
| TC-CLI-006 | check no tag match exits 2 | CLI | Must | FR-CLI-03 | Config has no `payments` tag. | Execute the check no tag match exits 2 command: `pulsecheck check --tag payments`. | Tag `payments`. | Exit code is 2 and stdout contains `No targets matched tag payments`. |
| TC-CLI-007 | check returns exit 1 for degraded | CLI | Must | FR-CLI-08 | Local server responds 200 after 1600 ms. | Execute the check returns exit 1 for degraded command: `pulsecheck check` with threshold 1500 ms. | Target `slow-home`. | Exit code is 1 and output row shows `slow-home`, `DEGRADED`, and latency above 1500 ms. |
| TC-CLI-008 | status shows UNKNOWN for new target | CLI | Must | FR-CLI-05 | Config has one target with no database row. | Execute the status shows unknown for new target command: `pulsecheck status --config targets.yml`. | Target `new-api`. | Exit code is 1 and row shows `new-api` with status `UNKNOWN`. |
| TC-CLI-009 | history enforces limit lower bound | CLI | Must | FR-CLI-06 | Valid config and database exist. | Execute the history enforces limit lower bound command: `pulsecheck history --limit 0`. | Limit value 0. | Exit code is 2 and output contains `limit | invalid_range | must be between 1 and 1000`. |
| TC-CLI-010 | history JSON shape | CLI | Must | FR-CLI-06 | Database has 20 `fee-portal` rows. | Execute the history json shape command: `pulsecheck history --target fee-portal --limit 5 --format json`. | Target `fee-portal`. | JSON has `filters.limit` 5 and exactly 5 objects in `rows`. |
| TC-CLI-011 | report Markdown metrics | CLI | Must | FR-CLI-07 | Database has 100 completed rows and 3 missed checks. | Run report for `2026-10-01T00:00:00Z` to `2026-10-08T00:00:00Z`. | 90 UP at 100 ms, 6 DEGRADED at 1700 ms, and 4 timeouts with null latency. | Output contains `Uptime: 96.00%`, `Average latency: 200 ms`, `P95 latency: 1700 ms`, and `Missed checks: 3`. |
| TC-CLI-012 | report rejects reversed period | CLI | Must | FR-REPORT-03 | Valid config exists. | Execute the report rejects reversed period command: `pulsecheck report --since 2026-10-08T00:00:00Z --until 2026-10-01T00:00:00Z`. | Reversed dates. | Exit code is 2 and output contains `period | invalid_range | since must be before until`. |
| TC-CLI-013 | purge reports counts | CLI | Must | FR-STORE-02 | Database contains purge-eligible rows. | Execute the purge reports counts command: `pulsecheck purge --older-than 90`. | 1240 old checks, 3 expired missed checks by `scheduled_at`, and 44 old SSL observations. | Exit code is 0 and output includes `check_results deleted: 1240`, `missed_checks deleted: 3`, and `ssl_certificate_observations deleted: 44`. |
| TC-CLI-014 | JSON log line contains required fields | CLI | Must | FR-OBS-01 | Local server returns 200. | Execute the json log line contains required fields command: `pulsecheck check --log-format json`. | Target `college-home`. | Each log line parses as JSON and includes `timestamp`, `level`, `event`, `target_name`, `status`, `attempt_count`, and `correlation_id`. |
| TC-SEC-001 | Redact Authorization header in logs | Security | Must | FR-OBS-01 | Config has header `Authorization: REDACTION-MARKER-12345`. | Execute the redact authorization header in logs command: `pulsecheck check --verbose` against local 200 server. | Harmless marker `REDACTION-MARKER-12345`. | Logs contain `[REDACTED]` and do not contain `REDACTION-MARKER-12345`. |
| TC-SEC-002 | Verify certificates by default | Security | Must | FR-SSL-01 | Wrong-host trustme server is running. | Run HTTPS check without any insecure option. | Certificate for `wrong.example.in`. | Check fails with TLS error and there is no CLI option that disables verification in Must scope. |
| TC-SEC-003 | pip-audit runs on dependencies | Security | Must | NFR-SEC-03 | Lockfile exists. | Run CI dependency scan job. | uv-locked dependencies. | Job runs pip-audit 2.10.x and fails if a known vulnerability without waiver is found. |
| TC-SEC-004 | Gitleaks blocks committed secret | Security | Must | NFR-SEC-03 | A throw-away test branch contains a synthetic, never-issued value in the GitHub personal-access-token format. The value is never committed to `main`. | Run secret scan job. | Fake token in ignored test fixture path is not allowed. | Gitleaks job fails and reports the file path without printing the full token. |
| TC-PERF-001 | Check 200 targets under 10 seconds | Performance | Must | FR-CHECK-01 | Local threaded pytest-httpserver or ASGI test server can serve at least 50 concurrent requests. | Execute `pulsecheck check` with 200 targets, `global.concurrency: 50`, and `global.retries: 0`. | Targets `target-001` to `target-200`; each response delays 100 ms. | Measured check phase completes in under 10 seconds and all 200 results are stored as `UP`. |
| TC-PERF-002 | Shutdown exits from latest states | Performance | Must | FR-SCHED-02 | Scheduler has 4 in-flight delayed checks and seeded latest `UP` rows for all selected targets. | On Ubuntu, send SIGTERM to `pulsecheck run`; on Windows, send Ctrl+C and assert KeyboardInterrupt handling. | Delayed checks last 10 seconds. | Process exits 0 within 5 seconds and no partial result is stored for unfinished checks. |
| TC-PERF-003 | Scheduler records missed check without overlap | Recovery | Must | FR-SCHED-01 | One target interval is 10 seconds and check takes 15 seconds. | Run scheduler for 25 seconds with controlled clock. | Target `slow-api`. | Exactly one missed-check record exists and server never observes overlapping requests for `slow-api`. |
| TC-PERF-004 | One failing notification does not stop checks | Recovery | Must | NFR-REL-01 | Webhook receiver returns 500 and two targets are due. | Run scheduler tick for `fee-portal` and `college-home`. | One incident opens and one healthy check succeeds. | Failed notification row is stored and healthy target result is still stored as `UP`. |
| TC-PERF-005 | Shutdown exits one for degraded state | Recovery | Must | FR-SCHED-02, FR-CLI-08 | Latest selected states contain one `DEGRADED` target before shutdown. | Send Ctrl+C to `pulsecheck run` after the degraded state is stored. | Target `slow-home` latest state `DEGRADED`. | Process exits 1 after the shutdown message and stores no partial result. |
| TC-CLI-015 | verbose and quiet conflict | CLI | Must | FR-OBS-01 | Valid config exists. | Execute the verbose and quiet conflict command: `pulsecheck check --verbose --quiet`. | Both flags set. | Exit code is 2 and output is `verbosity \| conflict \| --verbose and --quiet cannot be used together`. |
| TC-CLI-016 | package exposes pulsecheck help | CLI | Must | FR-PKG-01 | Package is installed with uv tool or pipx in a clean environment. | Execute the package exposes pulsecheck help command: `pulsecheck --help`. | Installed package version `1.0.0`. | Exit code is 0 and help lists `init`, `validate`, `check`, `run`, `status`, `history`, `report`, and `purge`. |
| TC-IT-015 | non-HTTPS target skips SSL observation | Integration | Must | FR-SSL-01 | Local HTTP server returns 200. | Run check against `http://localhost:<port>/status`. | Target `local-status`. | Check result is `UP` and `ssl_certificate_observations` has zero rows for `local-status`. |
| TC-CLI-017 | report JSON shape includes target metrics | CLI | Must | FR-CLI-07 | Database has rows for `college-home`. | Execute the report json shape includes target metrics command: `pulsecheck report --format json --target college-home --since 2026-10-01T00:00:00Z`. | 50 completed checks. | JSON `scope.target` is `college-home` and `targets[0].completed_checks` is 50. |
| TC-IT-016 | schema version migration runs before read | Integration | Must | FR-STORE-01 | Database schema version is one version behind. | Start `status` repository read. | Migration note `add ssl observation index`. | Migration updates `schema_version` before status query returns rows. |
| TC-CLI-018 | colour disabled still shows text status | CLI | Must | FR-REPORT-01 | Terminal colour is disabled. | Execute the colour disabled still shows text status command: `pulsecheck status` on one DOWN target. | Environment disables colour. | Output contains plain text `DOWN` and does not require colour to identify failure. |
| TC-IT-017 | SMTP missing auth credentials warning | Integration | Should | FR-NOTIF-03 | SMTP authentication is requested with username `training` but no password. | Validate settings; separately verify local unauthenticated Mailpit settings need no password. | Missing `PULSECHECK_SMTP_PASSWORD` in auth case only. | Auth case warns/disables SMTP with console retained; unauthenticated local Mailpit remains enabled without an account/key. |
| TC-CLI-019 | CSV history header | CLI | Should | FR-REPORT-04 | Database has three rows. | Execute the csv history header command: `pulsecheck history --format csv`. | Three check rows. | First line is `target_name,checked_at,status,latency_ms,attempt_count,error_kind`. |
| TC-CLI-020 | Tag report statistics export | CLI | Should | FR-REPORT-05, BR-09, BR-10 | Two stored final responses belong to current YAML tag `training`. | Run report for their period with `--group-by tag --format json`; repeat with CSV. | UP 100 ms, DOWN 200 ms, plus 1 missed check. | Group has completed 2, down 1, uptime 50.00, p95 200; missed check is outside denominator; CSV parses and no page is produced. |
| TC-UT-021 | Incident thresholds one and one | Unit | Must | FR-INC-01 | Incident engine uses open threshold 1 and close threshold 1. | Process `DOWN`, then `UP` for `fee-portal`. | Thresholds `N=1`, `M=1`. | One incident opens on the first result and closes on the first recovery result. |
| TC-UT-022 | Incident thresholds four and three | Unit | Must | FR-INC-01 | Incident engine uses open threshold 4 and close threshold 3. | Process four `DOWN` results, then `UP`, `DEGRADED`, `UP`. | Thresholds `N=4`, `M=3`. | One incident opens on the fourth DOWN and closes with `ended_at` at the first `UP`. |
| TC-UT-023 | Retry backoff without jitter | Unit | Must | FR-CHECK-02 | Jitter is set to 0 for a deterministic clock. | Calculate retry delays for 5 retries. | Backoff policy `0.25, 0.5, 1, 2, 2`. | The scheduled delays are exactly `0.25`, `0.5`, `1`, `2`, and `2` seconds. |
| TC-UT-024 | Retry jitter stays in range | Unit | Must | FR-CHECK-02 | Random jitter source is controlled. | Generate 100 retry delays for attempts 1 to 5. | Jitter range 0 to 250 ms. | Every delay is base backoff plus a jitter value from 0 to 250 ms. |
| TC-UT-025 | Scheduler immediate first checks | Unit | Must | FR-SCHED-01 | Clock starts at `2026-10-02T07:00:00Z`. | Simulate due calculation for 600 seconds with end excluded. | Intervals 60 seconds and 300 seconds. | The 60-second target starts exactly 10 checks and the 300-second target starts exactly 2 checks. |
| TC-IT-018 | Pending notification resumes after restart | Integration | Must | FR-NOTIF-02 | Database has one `pending` webhook notification with attempt count 0. | Start `pulsecheck run` with a local receiver returning 204. | Notification ID `notif_01K6K92P9T3GZCK9ADW6Q61CXR`. | The same notification ID is sent and the row becomes `sent`. |
| TC-IT-019 | Webhook formats are local tested | Integration | Must | FR-NOTIF-01 | Local webhook receiver captures three requests. | Dispatch one incident notification for `generic`, `slack`, and `discord`. | Summary `PulseCheck incident opened for fee-portal`. | Generic body has `notification_id`; Slack body has `text`; Discord body has `content`; all include the stable ID. |
| TC-IT-020 | Failed notification retries after thirty seconds | Integration | Must | FR-NOTIF-02 | Webhook receiver returns 500, then 204 after the retry time. | Dispatch one notification, advance the clock by 30 seconds, and run the retry loop. | Notification ID `notif_01K6K92P9T3GZCK9ADW6Q61CXR`. | The second request uses the same notification ID and the row becomes `sent` with attempt count 2. |
| TC-IT-021 | SAN match passes with different subject CN | Integration | Must | FR-SSL-01 | trustme HTTPS server has SAN `localhost` and a different subject CN. | Run HTTPS check against `https://localhost:<port>/health`. | Valid SAN certificate. | Target status is `UP` and SSL observation has `hostname_match` true. |
| TC-UT-026 | SSL first observation at twenty days | Unit | Must | FR-SSL-02 | No prior warning exists for hostname `www.example.in` and fingerprint `abc123`. | Evaluate days remaining 20. | Thresholds 30, 14, and 7. | Exactly one warning is returned with dedupe key `ssl:www.example.in:abc123:30`. |
| TC-UT-027 | SSL first observation at six days | Unit | Must | FR-SSL-02 | No prior warning exists for hostname `www.example.in` and fingerprint `abc123`. | Evaluate days remaining 6. | Thresholds 30, 14, and 7. | Three warnings are returned for thresholds 30, 14, and 7. |
| TC-UT-028 | Shared host SSL warning de-duplicates | Unit | Must | FR-SSL-02 | Two targets use hostname `www.example.in` and the same certificate fingerprint. | Evaluate both targets for threshold 30. | Targets `college-home` and `admissions-home`. | Only one warning exists for dedupe key `ssl:www.example.in:abc123:30`. |
| TC-CLI-021 | Replay validates incident/outbox invariants | CLI | Could | FR-REPLAY-01 | Recorded seed exists and destination is fresh. | Replay the five-result seed; repeat with an out-of-order timestamp and with an existing runtime destination. | DOWN/DOWN/DOWN/UP/UP; isolated `.local/replay.db`. | Valid replay exits 0 with checks 5, closed incidents 1, notification keys 2 and no sends; invalid replay exits 2 `replay-invalid` without rows; existing destination exits 2 `replay-database-not-fresh`. |

## Local acceptance catalog

Implement a pytest `local` marker and run `uv run --offline pytest -m local` after preparing the environment. Tests use an isolated project-local workspace, not a student's live database. Use the exact [document 06 contract](06-tech-stack-and-setup.md#local-operation-contract); network denial covers external egress, not localhost.

| ID | Linked IDs | Preconditions and operation | Exact expected result |
|---|---|---|---|
| TC-LOCAL-001 | FR-LOCAL-01, FR-PKG-01, FR-STORE-01, NFR-PORT-01, NFR-DOC-01, BR-22 | Clean clone; cached dependencies and prepared TLS files; no `.local/` databases. Run `make local-start`, health checks, and local validate. | Start exits 0 with `local-ready targets=2`; HTTP/TLS/webhook health returns 200 `{"status":"ok"}`; validate prints `Configuration valid: 2 targets`; migrated demo has 5 checks, 1 closed incident, 2 console notifications. |
| TC-LOCAL-002 | FR-LOCAL-01, FR-CLI-07, FR-SSL-01, FR-REPORT-02, BR-09, BR-10, BR-11, BR-22 | Deny external egress after preparation. Run `make local-demo` twice. | Both runs exit 0 with `local checks: local-api=UP local-tls=UP` and the exact seed summary in document 06: completed 5, down 3, uptime 40.00%, p95 100, closed 1, open 0, missed 0, MTTR 180, console notifications 2. Seed row counts do not grow. |
| TC-LOCAL-003 | FR-LOCAL-01, FR-STORE-01, FR-NOTIF-02, NFR-REL-02, BR-12, BR-15, BR-22 | Save stored result/incident IDs; leave one pending webhook row with fixed ID and due retry. Stop/start with receiver available; attempt reset without confirmation. | Stop prints `local-stopped data-preserved`; saved IDs remain; retry uses the same notification ID, leaving 1 row for its channel/key; reset exits 2 `local-reset-confirmation-required` and changes no data. |
| TC-LOCAL-004 | FR-LOCAL-01, FR-CFG-02, FR-CLI-08, FR-CHECK-03, NFR-USE-01, BR-02, BR-17, BR-22 | In isolated runs: invalid concurrency 0; occupied port 8765; missing TLS fixtures/tool; stop only HTTP fixture; lock SQLite beyond retry budget. | Invalid config exits 2 with path/code/reason; start exits 2 `local-port-in-use: 8765`, `local-tls-fixture-missing`, or `local-dependency-missing: <tool>`; check stores HTTP DOWN/connection and TLS UP, exit 1; locked DB exits 3 `Internal error: database is locked`. No downloads/live fallback. |

## Functional traceability matrix

| Requirement ID | Test cases or demo step |
|---|---|
| FR-CFG-01 | func-a evidence: TC-CLI-003 |
| FR-CFG-02 | func-b evidence: TC-UT-017, TC-UT-018, TC-UT-019 |
| FR-CLI-01 | func-c evidence: TC-CLI-001, TC-CLI-002 |
| FR-CLI-02 | func-d evidence: TC-CLI-003, TC-CLI-004 |
| FR-CLI-03 | func-e evidence: TC-CLI-005, TC-CLI-006 |
| FR-CLI-04 | func-f evidence: TC-PERF-003 |
| FR-CLI-05 | func-g evidence: TC-CLI-008 |
| FR-CLI-06 | func-h evidence: TC-CLI-009, TC-CLI-010, TC-IT-006 |
| FR-CLI-07 | func-i evidence: TC-UT-010, TC-CLI-011, TC-CLI-017 |
| FR-CLI-08 | func-j evidence: TC-CLI-007, TC-CLI-008, TC-PERF-005, TC-IT-014 |
| FR-CHECK-01 | func-k evidence: TC-IT-002, TC-IT-003, TC-PERF-001 |
| FR-CHECK-02 | func-l evidence: TC-UT-020, TC-UT-023, TC-UT-024, TC-IT-001 |
| FR-CHECK-03 | func-m evidence: TC-UT-001, TC-UT-002, TC-UT-003, TC-UT-004, TC-UT-005 |
| FR-SCHED-01 | func-n evidence: TC-UT-025, TC-PERF-003 |
| FR-SCHED-02 | func-o evidence: TC-PERF-002, TC-PERF-005 |
| FR-STORE-01 | func-p evidence: TC-IT-004, TC-IT-014, TC-IT-016 |
| FR-STORE-02 | func-q evidence: TC-IT-005, TC-CLI-013 |
| FR-INC-01 | func-r evidence: TC-UT-006, TC-UT-007, TC-UT-008, TC-UT-009, TC-UT-021, TC-UT-022 |
| FR-INC-02 | func-s evidence: TC-UT-014 |
| FR-SSL-01 | func-t evidence: TC-IT-007, TC-IT-008, TC-IT-009, TC-IT-015, TC-IT-021 |
| FR-SSL-02 | func-u evidence: TC-UT-015, TC-UT-016, TC-UT-026, TC-UT-027, TC-UT-028 |
| FR-NOTIF-01 | func-v evidence: TC-IT-010, TC-IT-011, TC-IT-019, TC-PERF-004 |
| FR-NOTIF-02 | func-w evidence: TC-IT-012, TC-IT-013, TC-IT-018, TC-IT-020 |
| FR-OBS-01 | func-x evidence: TC-CLI-014, TC-CLI-015, TC-SEC-001 |
| FR-PKG-01 | func-y evidence: TC-CLI-016 |
| FR-LOCAL-01 | TC-LOCAL-001, TC-LOCAL-002, TC-LOCAL-003, TC-LOCAL-004 |
| FR-REPORT-01 | func-z evidence: TC-CLI-007, TC-CLI-018 |
| FR-REPORT-02 | func-aa evidence: TC-UT-010, TC-UT-012, TC-UT-013, TC-CLI-011 |
| FR-REPORT-03 | func-ab evidence: TC-UT-011, TC-CLI-011, TC-CLI-012 |
| FR-NOTIF-03 | func-ac evidence: TC-IT-017 |
| FR-REPORT-04 | func-ad evidence: TC-CLI-019 |
| FR-REPORT-05 | func-ae evidence: TC-CLI-020 |
| FR-SSL-03 | func-af evidence: Demo-D01: show issuer and subject in optional SSL output. |
| FR-OPS-01 | func-ag evidence: Demo-D02: start optional Docker image with mounted config. |
| FR-SCHED-03 | func-ah evidence: Demo-D03: show maintenance window suppressing notification. |
| FR-DOC-01 | func-ai evidence: Demo-D04: follow the local Markdown CLI usage guide and compare help/version snapshots. |
| FR-REPLAY-01 | func-aj evidence: TC-CLI-021 |
| FR-MET-01 | func-ak evidence: Demo-D06: optional Prometheus endpoint disabled by default. |
| FR-CHECK-04 | func-al evidence: Demo-D07: optional TCP and DNS target types. |
| FR-NOTIF-04 | func-am evidence: Demo-D08: optional Telegram notifier with env token. |
| FR-TEST-01 | func-an evidence: Demo-D09: optional mutmut run on classifier. |

## Business-rule traceability matrix

| Business rule | Test cases or demo step |
|---|---|
| BR-01 | rule-a evidence: TC-CLI-001, TC-CLI-003 |
| BR-02 | rule-b evidence: TC-UT-018 |
| BR-03 | rule-c evidence: TC-UT-017 |
| BR-04 | rule-d evidence: TC-UT-001, TC-UT-002, TC-UT-003, TC-UT-004 |
| BR-05 | rule-e evidence: TC-UT-020, TC-UT-023, TC-UT-024, TC-IT-001 |
| BR-06 | rule-f evidence: TC-IT-001 |
| BR-07 | rule-g evidence: TC-UT-006, TC-UT-007, TC-UT-021, TC-UT-022 |
| BR-08 | rule-h evidence: TC-UT-008, TC-UT-009, TC-UT-021, TC-UT-022 |
| BR-09 | rule-i evidence: TC-UT-010, TC-UT-011 |
| BR-10 | rule-j evidence: TC-UT-012, TC-UT-013 |
| BR-11 | rule-k evidence: TC-UT-014 |
| BR-12 | rule-l evidence: TC-IT-004 |
| BR-13 | rule-m evidence: TC-IT-007, TC-IT-015, TC-IT-021 |
| BR-14 | rule-n evidence: TC-UT-015, TC-UT-016, TC-UT-026, TC-UT-027, TC-UT-028 |
| BR-15 | rule-o evidence: TC-IT-010, TC-IT-012, TC-IT-013, TC-IT-018, TC-IT-019, TC-IT-020 |
| BR-16 | rule-p evidence: TC-IT-005, TC-CLI-013 |
| BR-17 | rule-q evidence: TC-CLI-007, TC-PERF-005, TC-IT-014 |
| BR-18 | rule-r evidence: TC-CLI-014, TC-SEC-001 |
| BR-19 | rule-s evidence: TC-CLI-016 |
| BR-20 | rule-t evidence: TC-CLI-010, TC-CLI-011, TC-CLI-017 |
| BR-21 | rule-u evidence: Demo-D02 |
| BR-22 | TC-LOCAL-001, TC-LOCAL-002, TC-LOCAL-003, TC-LOCAL-004 |

## NFR traceability matrix

| NFR ID | Test cases or demo step |
|---|---|
| NFR-PERF-01 | nfr-a evidence: TC-PERF-001 |
| NFR-PERF-02 | nfr-b evidence: TC-PERF-002 |
| NFR-SCALE-01 | nfr-c evidence: TC-PERF-001 |
| NFR-REL-01 | nfr-d evidence: TC-IT-003, TC-PERF-004 |
| NFR-REL-02 | nfr-e evidence: TC-IT-010, TC-IT-012, TC-IT-018, TC-IT-020 |
| NFR-SEC-01 | nfr-f evidence: TC-IT-007, TC-IT-008, TC-IT-009, TC-IT-021, TC-SEC-002 |
| NFR-SEC-02 | nfr-g evidence: TC-SEC-001, TC-SEC-004 |
| NFR-SEC-03 | nfr-h evidence: TC-SEC-003, TC-SEC-004 |
| NFR-PRIV-01 | nfr-i evidence: Demo-D10: review fixtures and examples for only fictional data. |
| NFR-MAINT-01 | nfr-j evidence: Demo-D11: CI mypy 2.4.x strict job passes. |
| NFR-MAINT-02 | nfr-k evidence: Demo-D12: CI Ruff 0.16.x lint and format jobs pass. |
| NFR-TEST-01 | nfr-l evidence: Coverage gate from the coverage thresholds table. |
| NFR-TEST-02 | nfr-m evidence: TC-UT-001 to TC-UT-016 plus per-module branch reports. |
| NFR-OBS-01 | nfr-n evidence: TC-CLI-014 |
| NFR-OBS-02 | nfr-o evidence: TC-CLI-014 |
| NFR-USE-01 | nfr-p evidence: TC-UT-017, TC-UT-019 |
| NFR-PORT-01 | nfr-q evidence: TC-LOCAL-001 to TC-LOCAL-004; Demo-D13: Ubuntu and Windows matrix plus WSL2/macOS trainer entrypoint evidence. |
| NFR-HW-01 | nfr-r evidence: Demo-D14: trainer lite-profile pre-check on 8 GB laptop. |
| NFR-LIC-01 | nfr-s evidence: Demo-D15: dependency licence review evidence before final demo. |
| NFR-CI-01 | nfr-t evidence: Demo-D16: CI timing report under 10 minutes after cache warm-up. |
| NFR-ACC-01 | nfr-u evidence: TC-CLI-018 |
| NFR-DOC-01 | nfr-v evidence: TC-LOCAL-001 to TC-LOCAL-004; Demo-D17: trainer completes README setup in 10 steps or fewer. |

## Performance test conditions

| Condition | Exact value |
|---|---|
| Reference hardware | 4-core laptop with 8 GB RAM. |
| Target count | 200 configured HTTP targets. |
| Concurrency | 50 active network attempts. |
| Server | Local threaded pytest-httpserver or ASGI server that serves at least 50 concurrent requests. |
| Retries | `global.retries: 0` for the measured performance path. |
| Response delay | Fixed 100 ms for every target response. |
| Network | Loopback only. No internet calls. |
| Warm-up | One unmeasured request to the local server before timing. |
| Measured phase | Starts before `check` dispatch and stops after all final outcomes are stored. |
| Pass condition | Under 10 seconds with 200 stored `UP` rows. |
| Failure evidence | Duration, target count, concurrency, CPU count, Python version, and slowest 10 latencies. |

## Defect report fields

| Field | Required content |
|---|---|
| Title | Short symptom, such as `check stores duplicate retry attempts`. |
| Environment | OS, Python version, PulseCheck version, database path type, and command. |
| Requirement IDs | Linked FR, BR, or NFR IDs. |
| Steps to reproduce | Exact command, config snippet name, and local server setup. |
| Expected result | Exact status, output line, exit code, or database state. |
| Actual result | Exact observed output, exit code, logs, or row counts. |
| Evidence | Test name, log excerpt with secrets redacted, and screenshot only if useful. |
| Severity | Critical, High, Medium, or Low. |
| Fix notes | Root cause, changed modules, and regression test ID. |

## Entry criteria

- The student repository has `pyproject.toml`, `uv.lock`, and console entry point `pulsecheck`.
- The sample targets file validates successfully.
- The CI pipeline can run pytest, Ruff, mypy, pip-audit, and Gitleaks.
- The local test server fixtures run without internet access.
- SQLite repository tests can create isolated database files under the test workspace.

## Exit criteria

- All Must test cases pass on the main branch.
- Whole-package coverage is at least 90% line and 80% branch.
- Pure-logic modules have 100% branch coverage.
- The 200-target performance test passes under 10 seconds.
- TLS tests cover valid, expired, and wrong-host certificates.
- CLI tests prove exit codes 0, 1, 2, and 3.
- Security scans complete with no unapproved findings.
- The final demo shows validation, check, incident, notification, report, purge, and logs.
- All four local acceptance cases pass with saved command/JSON/SQLite evidence. CI and the grading gates enforce them; no frontend coverage or deliverable exists.

[Back to README](../README.md)
