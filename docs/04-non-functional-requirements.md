# PulseCheck non-functional requirements

Purpose: This document defines measurable quality requirements and verification methods for PulseCheck.

## NFR catalog

| ID | Category | Priority | Target | Verification method |
|---|---|---|---|---|
| NFR-PERF-01 | Performance | Must | Check 200 local HTTP targets in under 10 seconds with concurrency 50 and 100 ms server delay. | Automated performance test on a 4-core, 8 GB laptop. |
| NFR-PERF-02 | Performance | Must | Scheduler shutdown completes in 5 seconds after Ctrl+C or SIGTERM. | Signal test during active checks. |
| NFR-SCALE-01 | Scalability | Must | Must scope supports 200 configured targets in one YAML file. | Test fixture with 200 targets. |
| NFR-REL-01 | Reliability | Must | One target failure never stops checks for other due targets. | Integration test with one failing and one healthy target. |
| NFR-REL-02 | Reliability | Must | Notifications use at-least-once delivery with stable `notification_id` and `dedupe_key`. | Unit test for retry and de-duplication. |
| NFR-SEC-01 | Security | Must | HTTPS certificate verification is enabled by default. | trustme TLS tests for valid, expired, and wrong-host certificates. |
| NFR-SEC-02 | Security | Must | Secret headers and webhook tokens never appear in logs or reports. | Log snapshot test and Gitleaks scan. |
| NFR-SEC-03 | Security | Must | Dependency scans run on every pull request. | CI jobs for pip-audit and secret scanning. |
| NFR-PRIV-01 | Privacy | Must | Sample data uses fictional targets and no real personal data. | Documentation review before release. |
| NFR-MAINT-01 | Maintainability | Must | Application package passes mypy `--strict`. | CI type-check job. |
| NFR-MAINT-02 | Maintainability | Must | Ruff lint and format checks pass with no ignored project-wide failures. | CI lint job and pre-commit. |
| NFR-TEST-01 | Test coverage | Must | Whole package line coverage is at least 90% and branch coverage at least 80%. | pytest-cov gates in CI. |
| NFR-TEST-02 | Test coverage | Must | Classifier, incident engine, uptime calculator, p95 calculator, and SSL evaluator have 100% branch coverage. | Per-module coverage report. |
| NFR-OBS-01 | Observability | Must | Each stored check has a correlation ID and logs include target, status, latency, and attempts. | Log fixture assertions. |
| NFR-OBS-02 | Observability | Must | JSON log mode emits one JSON object per line. | CLI snapshot test. |
| NFR-USE-01 | Usability | Must | Validation errors include path, code, and reason. | CLI tests for invalid targets files. |
| NFR-PORT-01 | Portability | Must | Must scope runs on Windows 11 WSL2 Ubuntu, macOS, and Linux using the local contract, without external egress after preparation. | TC-LOCAL-001 to TC-LOCAL-004; Ubuntu/Windows CI and recorded macOS trainer smoke evidence. |
| NFR-HW-01 | Hardware | Must | Target 8 GB RAM, 4 cores, 2 GB free disk; no Docker required; proposed 1 GiB runtime budget. | Trainer pre-check records actual resource use; limits are not measured claims. |
| NFR-LIC-01 | Licence | Must | Mandatory dependencies use approved open-source licences. | Dependency licence review before final demo. |
| NFR-CI-01 | CI duration | Should | Required CI pipeline finishes in under 10 minutes after dependency cache warm-up. | GitHub Actions timing report. |
| NFR-ACC-01 | Accessibility | Must | Terminal output does not rely on colour alone. | Manual review and snapshot test with colour disabled. |
| NFR-DOC-01 | Documentation | Must | README setup from a clean clone uses 10 steps or fewer and documents start/stop, seed/demo, persistence, reset confirmation, and local failures. | Trainer follows document 06; TC-LOCAL-001 to TC-LOCAL-004. |

## Performance requirements

PulseCheck MUST complete the reference performance test in under 10 seconds. The test has 200 HTTP targets, concurrency 50, a fixed 100 ms local server delay, and no external internet calls. The measured phase MUST exclude dependency installation and server startup.

The scheduler MUST avoid overlapping checks for the same target. It MUST record a missed check when a due time arrives while the previous check for that target is still in progress. Missed checks MUST appear in reports but MUST NOT change the uptime formula.

## Scalability limits

PulseCheck is a local CLI tool, not a SaaS monitor. The mandatory scale target is 200 configured targets and one local SQLite database. Larger target counts are best-effort and need measurement notes in the student README.

## Reliability requirements

A failed DNS lookup, timeout, TLS error, HTTP status mismatch, or missing keyword MUST affect only that target. The scheduler MUST continue checking other due targets. A failed notification MUST NOT remove or close an incident.

Notifications MUST be at-least-once. The store MUST enforce de-duplication for `channel` plus `dedupe_key`. The default notification cooldown is 30 minutes for reminders about the same open incident.

## Security requirements

| Risk area | PulseCheck relevance | Required control |
|---|---|---|
| Cryptographic failures | HTTPS checks can be misleading if certificates are ignored. | Verify certificates by default and test expired and wrong-host certificates. |
| Injection | YAML, headers, URLs, and environment variables are untrusted input. | Validate all fields and never execute values as commands. |
| Security misconfiguration | A bad targets file can hide outages. | Fail closed with exit code 2 on validation errors. |
| Vulnerable components | CLI dependencies change over time. | Run pip-audit in CI and document upgrades. |
| Logging failures | Logs are used during incidents. | Include correlation IDs and redact secrets. |

Webhook URLs, SMTP passwords, and bearer tokens MUST come from environment variables or ignored local files. They MUST NOT be stored in sample files, reports, or logs.

## Privacy and DPDP Act 2023 awareness

PulseCheck does not require personal data. Live configuration examples MUST use fictional service names and `example.in` or `example.com` domains. Offline training MUST use localhost/loopback fixtures with fictional seed records instead; example domains are not offline services. If a real team puts personal data into target names, URLs, or tags, the student documentation MUST tell them to minimise that data and restrict report sharing.

## Maintainability requirements

Core logic MUST be separated from CLI and I/O code. The classifier, incident engine, uptime calculator, p95 calculator, and SSL evaluator MUST be pure or nearly pure modules with 100% branch coverage. Application packages MUST pass mypy `--strict`.

The repository SHOULD use small pull requests. Each feature area SHOULD have tests before it is marked complete. ADRs MUST explain major design choices listed in `05-system-architecture.md`.

## Observability requirements

Normal logs MUST be useful during an outage. Each check log line MUST include target name, final status, latency in milliseconds, attempt count, and correlation ID. JSON logs MUST be one object per line so a user can pipe them into local tools.

## Usability and accessibility requirements

The CLI MUST print clear text labels such as `UP`, `DEGRADED`, `DOWN`, and `UNKNOWN`. Colour MAY help, but text MUST remain meaningful when colour is disabled. Validation output MUST be easy to paste into a GitHub Issue.

## Portability and hardware profiles

| Profile | Minimum hardware | Required behaviour |
|---|---|---|
| Lite | 8 GB RAM, 4 CPU cores, 2 GB free disk | SQLite and Python HTTP/TLS/webhook fixtures; no mandatory Docker. Proposed budgets and the 200-target performance gate are verified, not assumed. |
| Standard | 16 GB RAM, 4 or more CPU cores, 5 GB free disk | Should items can run one optional Docker Compose/Mailpit support stack. |

Windows users SHOULD configure WSL2 with `.wslconfig` memory `4GB` and swap `4GB` on 8 GB laptops. On 16 GB laptops, use memory `8GB` and swap `4GB`. The trainer MUST validate the lite profile on an 8 GB laptop before week 1.

The authoritative [local contract](06-tech-stack-and-setup.md#local-operation-contract) binds fixture listeners to loopback, preserves SQLite across stop/start, and requires explicit reset confirmation. Local fixtures are selected deliberately; live website checks still need network access and never silently fall back to recorded results.

## Licence compliance

Mandatory dependencies MUST use permissive or approved open-source licences. The student MUST record dependency licence review evidence before the final demo. Docker images are Should items and MUST use pinned tags when added.

[Back to README](../README.md)
