# PulseCheck tech stack and setup

Purpose: This document defines the approved tools, versions, local setup, hardware profiles, accounts, and learning order for PulseCheck.

## Mandatory stack table

Use these reference lines. Pin compatible minor lines in `pyproject.toml`, Docker tags, and CI.

| Category | Tool | Reference version | Purpose | Why PulseCheck uses it |
|---|---|---:|---|---|
| Language | Python | 3.12.x | Main implementation language. | It matches the cohort baseline and teaches entry-level Python. |
| CI runtime | Python | 3.13.x | Extra CI test runtime. | It proves the package is ready for the next Python line. |
| Package manager | uv | 0.12.x | Manage environments, dependencies, and lockfile. | It gives fast, repeatable installs for a CLI portfolio project. |
| CLI framework | Typer | 0.27.x | Build `init`, `validate`, `check`, `run`, `status`, `history`, `report`, and `purge`. | It maps typed Python functions to a clean CLI. |
| Terminal output | Rich | 15.x | Tables, readable errors, and optional colour. | It keeps `UP`, `DEGRADED`, `DOWN`, and `UNKNOWN` readable. |
| HTTP client | httpx | 0.28.x | Async HTTP and HTTPS endpoint checks. | It supports asyncio, timeouts, and TLS verification. |
| Concurrency | asyncio | Python 3.12 stdlib | Concurrent checks and scheduler cancellation. | It teaches async I/O without a web framework. |
| Validation | Pydantic | 2.13.x | Validate targets file settings. | It gives field paths and strong types. |
| YAML parser | PyYAML | 6.0.x | Read the targets file product interface. | YAML is friendly for site owners. |
| Storage | sqlite3 | Python 3.12 stdlib | Store results, incidents, SSL observations, and notifications. | It is free, local, and enough for 200 targets. |
| TLS checks | ssl | Python 3.12 stdlib | Inspect certificate expiry and hostname match. | It teaches TLS basics without paid services. |
| Tests | pytest | 9.1.x | Main test runner. | It is standard in Python interviews and CI. |
| Coverage | pytest-cov / coverage | 7.1.x / 7.16.x | Enforce line and branch coverage gates. | It proves Must logic has automated evidence. |
| Async tests | pytest-asyncio | 1.4.x | Test coroutines and scheduler code. | It covers async checks directly. |
| Async support alternative | AnyIO | 4.15.x | Optional pytest async plugin. | It is allowed if the student records an ADR. |
| HTTP mock tests | respx | 0.23.x | Mock httpx calls in unit tests. | It avoids internet-dependent tests. |
| Local HTTP tests | pytest-httpserver | 1.1.x | Real local server for integration tests. | It proves status, latency, and keyword checks. |
| TLS tests | trustme | 1.2.x | Generate test certificates. | It proves valid, expired, and wrong-host cases. |
| Time tests | time-machine | 3.5.x | Control clocks in incident and SSL tests. | It makes cooldown and MTTR tests deterministic. |
| Property tests | Hypothesis | 6.x | Should item for pure logic properties. | It finds edge cases in percentiles and classifiers. |
| Lint and format | Ruff | 0.16.x | Lint and format checks. | It keeps small CLI modules consistent. |
| Type check | mypy | 2.4.x | Strict type checking. | This project MUST run `mypy --strict` on the core package. |
| Git hooks | pre-commit | 4.6.x | Run local checks before commits. | It catches mistakes before pull requests. |
| Security scan | pip-audit | 2.10.x | Python dependency vulnerability scan. | It supports the security gate. |
| Secret scan | Gitleaks | 8.30.x | Detect committed secrets. | Webhook URLs and SMTP passwords must not leak. |
| Container | Docker Engine | 29.x | Should item for running `pulsecheck run` in a container. | It teaches reproducible local operations. |
| Compose | Docker Compose | v5.x | Should demo stack with Mailpit and a local target. | It runs support services with one command. |
| Container scan | Trivy | 0.75.x | Should scan for Docker image vulnerabilities. | It catches high and critical CVEs in optional images. |
| SMTP test | Mailpit | 1.31.x | Should item for local e-mail tests. | MailHog is unmaintained, so PulseCheck uses Mailpit. |
| CI | GitHub Actions | Current hosted runners | Run the Python 3.12 and 3.13 matrix. | It is free for public student repositories. |
| TestPyPI publish | Trusted publishing | Current | Should item for tag releases. | It avoids long-lived package tokens. |
| CLI install | pipx | Current | Alternative tool installation path. | Recruiters can run `pulsecheck` without editing the project. |

## Allowed alternatives that need an ADR

| Decision | Allowed alternative | ADR question |
|---|---|---|
| Async test plugin | AnyIO pytest plugin instead of pytest-asyncio | Why does this make async tests clearer for PulseCheck? |
| Task runner | `just` internally, preserving the documented make entrypoints | Why does this help maintenance while `make local-start`/`make local-stop` still work on all supported systems? |
| HTTP mocks | Only pytest-httpserver, without respx | Which tests become slower, and why is that acceptable? |
| Docker base image | Official `python:3.12-slim` pinned tag | Which tag is used, and how are CVEs tracked? |
| Metrics Could item | prometheus-client 0.26.x | Why is a metrics endpoint useful after the CLI MVP is complete? |

## Not allowed

| Tool or approach | Reason |
|---|---|
| Requests plus threads for Must checks | The brief requires asyncio and httpx. |
| Storing targets in SQLite | The targets YAML file is the product interface and source of truth. |
| Disabling TLS verification | It would make FR-SSL-01 and NFR-SEC-01 false. |
| Docker image tags named `latest` | Builds must be reproducible and traceable. |
| MailHog | It is unmaintained; use Mailpit 1.31.x. |
| Bitnami images | The free image catalog changed in 2025; use official images. |
| Real webhook URLs in examples | They are secrets and can leak alerts. |
| Internet services in automated tests | Tests must be repeatable on a laptop and CI. |
| SQLAlchemy for Must storage | The brief requires the standard-library `sqlite3` module. |
| Frontend, report pages, terminal/web dashboards, documentation-site publication, or cloud deployment | No such implementation, alternative, or bonus work is in v1.1. |

## Local setup checklist for Windows 11 with WSL2 Ubuntu 24.04

1. Install Windows 11 updates and enable WSL2.
2. Install Ubuntu 24.04 from the Microsoft Store.
3. Put the project under the Linux filesystem, not under `C:\Users`.
4. Install Git and make inside Ubuntu; set the Git user name and e-mail.
5. Install Python 3.12.x in Ubuntu.
6. Install uv 0.12.x and confirm `uv --version`.
7. Install VS Code and the WSL extension.
8. Install Python, Ruff, mypy, and GitHub Actions extensions in VS Code.
9. Install Docker Desktop only if you attempt FR-OPS-01 or Mailpit tests.
10. Run `uv sync`, then run lint, type check, and tests before the first PR.

### Required `.wslconfig` values

Place the file at `C:\Users\<you>\.wslconfig`. Restart WSL after changes.

| Laptop RAM | `.wslconfig` memory | `.wslconfig` swap | Reason |
|---:|---:|---:|---|
| 8 GB | `4GB` | `4GB` | Leaves memory for Windows while allowing Python tests and optional Docker. |
| 16 GB | `8GB` | `4GB` | Gives Docker and Mailpit more room during Should work. |

## Local setup checklist for macOS

1. Install Xcode Command Line Tools (including make).
2. Install Homebrew only if the trainer permits it.
3. Install Python 3.12.x and uv 0.12.x.
4. Install Git and configure the student GitHub account.
5. Install VS Code with Python, Ruff, mypy, and GitHub Actions extensions.
6. Install Docker Desktop only for FR-OPS-01 and Mailpit Should work.
7. Run the same `uv`, `ruff`, `mypy --strict`, and `pytest` commands used in CI.

## Local setup checklist for Linux

1. Use Ubuntu 24.04 or another current desktop Linux.
2. Install Git, make, Python 3.12.x, and uv 0.12.x.
3. Install VS Code or another editor with Python language support.
4. Install Docker Engine 29.x only for Should work.
5. Confirm the shell can run `pulsecheck --help` after installation.
6. Run pre-commit before each pull request.

## Local operation contract

These are commands and behaviors students MUST implement in their own repository (FR-LOCAL-01), not files supplied by this specification.

### Preparation and modes

- Windows 11 WSL2 Ubuntu, macOS, and Linux MUST use the same entrypoints. Finish the OS checklist and initial `uv sync --frozen`/package installation with internet available. Cache Python dependencies, build artifacts, and test fixtures before isolation.
- Prepare local CA/server certificate fixtures with trustme before the offline lab. Store private test keys and the CA bundle in ignored `.local/tls/`. Certificates MUST have SANs `localhost` and `127.0.0.1`, be valid for the runtime check, and be trusted explicitly via `SSL_CERT_FILE` by both httpx and the stdlib SSL context. Never disable verification.
- Normal live mode uses the supplied website configuration and needs access to those URLs. Fictional `example.in` examples are not promised to resolve or be UP. Local mode explicitly uses `targets.local.yml`; it is never a fallback for a failed live check.
- Local runtime/acceptance requires no paid account, public hostname, API key, external DNS, or external webhook. GitHub submission, hosted CI, dependency scans, and optional TestPyPI publishing are separate networked activities.

### Entry points and persistence

| Command to implement | Required behavior |
|---|---|
| `make local-start` | Check installed tools, writable `.local/`, prepared certificates, and free ports. Apply SQLite migrations, seed fictional demo evidence once, start local fixture servers and `pulsecheck run` with the local config, then wait for health and print `local-ready targets=2`. Do not download anything. |
| `make local-stop` | Signal only project-owned processes; wait up to 5 seconds for the scheduler, close SQLite safely, stop fixtures, and print `local-stopped data-preserved`. A degraded scheduler state is not a failed stop. |
| `make local-demo` | Check fixture health and local HTTP/TLS outcomes, then report the fixed seed period from the separate demo database. Print the exact summary below; do not append or regenerate seed rows. |
| `make local-reset CONFIRM=DELETE_LOCAL_DATA` | Only while stopped, delete project-local SQLite/seed state after this explicit confirmation. Without it, exit 2 with `local-reset-confirmation-required`; retain certificates and dependency caches. |

Use ignored bind-mounted/local directory `.local/`: runtime database `.local/pulsecheck.db`, demo database `.local/demo.db`, fixture state, PID records, and redacted logs. Startup MUST run repository migrations before reads, seed only an empty demo database, and never overwrite existing evidence. Stop/start MUST preserve check/incident IDs, pending notification IDs, and cooldown/SSL warning keys. An optional Docker image MUST mount this directory rather than keep data in its writable layer.

### Ports, targets, and health

| Local listener | Loopback address | Exact check |
|---|---|---|
| HTTP fixture | `127.0.0.1:8765` | `GET /healthz` returns 200 `{"status":"ok"}`; target `local-api` at `/healthy` returns 200 body `training-ok`. |
| HTTPS fixture | `localhost:8766`, bound to `127.0.0.1` | `GET /healthz` returns 200 `{"status":"ok"}` with trusted TLS; target `local-tls` at `/healthy` returns 200 body `training-ok`. |
| Webhook receiver | `127.0.0.1:8767` | `GET /healthz` returns 200 `{"status":"ok"}`; `POST /events` returns 202 and records stable notification IDs locally. |
| Optional metrics | `127.0.0.1:8768` | Disabled by default; metric scrape only, no frontend. |
| Optional Mailpit SMTP/API | `127.0.0.1:1125` / `127.0.0.1:8125` | Should-only operations tool; e-mail can be inspected through its HTTP API without implementing a console. |

The local config has exactly two targets, `local-api` and `local-tls`, with keyword `training-ok`, interval 60 seconds, timeout 5 seconds, and a healthy TLS certificate prepared for 90 days. Tests additionally select recorded 503, timeout, expiry, and wrong-host fixtures explicitly. All listeners MUST bind to loopback; detect occupied ports before starting rather than switching ports or contacting a website.

### Fictional seed and deterministic demonstration

Store a recorded JSON fixture for `local-api`: final statuses `DOWN, DOWN, DOWN, UP, UP` at `2026-10-02T00:00:00Z` through `00:04:00Z` at one-minute intervals. HTTP statuses are 503/503/503/200/200; final latencies are all 100 ms, attempt count 1, no missed checks. Seed through the incident engine at recorded timestamps with N=3/M=2 and console-only delivery: one incident starts at `00:00:00Z`, closes at the first recovery `00:03:00Z`, and has two sent console notification keys (opened/closed). These are recorded measurements, not claimed live timings.

`make local-demo` MUST assert a fresh loopback check of both targets is `UP` with verified TLS, then execute the equivalent of `pulsecheck --config targets.local.yml --database .local/demo.db report --target local-api --since 2026-10-02T00:00:00Z --until 2026-10-02T00:05:00Z --format json`. It MUST print:

```text
local checks: local-api=UP local-tls=UP
seed report: completed=5 down=3 uptime=40.00% p95_ms=100 closed_incidents=1 open_incidents=0 missed=0 mttr_seconds=180 console_notifications=2
```

Block external egress but allow loopback for TC-LOCAL-001 through TC-LOCAL-004 after preparation. Exact latency assertions concern the recorded seed; live request latency is measured normally. Invalid config still exits 2; an unreachable target becomes DOWN/exit 1; unwritable/locked SQLite or migration failure exits 3 with the documented internal error. Startup prerequisite failures exit 2 with `local-dependency-missing: <tool>`, `local-port-in-use: <port>`, or `local-tls-fixture-missing`. No failure silently repairs itself by downloading or falling back to live mode.

## Hardware profiles

| Profile | Exact limits | Required behaviour |
|---|---|---|
| Lite | 8 GB RAM, 4 CPU cores, 2 GB free disk; WSL2 memory `4GB`, swap `4GB`; no mandatory Docker | All Must features use SQLite and Python loopback fixtures. The 200-target test remains the performance target, not a measured result. |
| Standard | 16 GB RAM, 4 or more CPU cores, 5 GB free disk; WSL2 memory `8GB`, swap `4GB` | Should items may add one local Compose/Mailpit support stack. |

Lite switches off Mailpit, Docker/Compose, and mutation testing. Proposed steady budgets are 512 MiB for the scheduler and 256 MiB combined for fixtures, with a 1 GiB total target including Python/SQLite overhead. These are planning limits, not measurements; the trainer MUST record observed startup, performance, and steady peaks before the cohort. Frontend work is absent in every profile.

## Trainer pre-check

Before week 1, the trainer MUST validate this list on an 8 GB laptop.

- Fresh clone of `pulsecheck` installs with uv.
- Cache dependencies and prepare certificate fixtures, then verify `make local-start`, fixture health, and `make local-demo` with external egress blocked and loopback allowed.
- Verify `make local-stop`/restart preserves data and outbox IDs; reset refuses to delete without confirmation.
- Run TC-LOCAL-001 through TC-LOCAL-004 and record OS, hardware, proposed versus observed memory, and disk usage.
- Unit tests and pure-logic branch coverage run without Docker.
- The 200-target performance test completes in under 10 seconds.
- `mypy --strict` completes on the core package.
- Gitleaks and pip-audit complete without secrets or high-risk findings.
- The README setup needs 10 steps or fewer.

## Free accounts needed

| Account | Required | Used for |
|---|---|---|
| GitHub | Must for submission, not local startup | Repository, Issues, Projects board, pull requests, and hosted CI. |
| TestPyPI | Should | Trusted publishing release practice. |
| Docker Hub | Optional | Only if the student pushes an image. Local Docker does not need it. |
| Slack or Discord | Optional | Manual webhook demo with a test channel. Never commit the URL. |

## Suggested learning order

| Order | Topic | Hours | Outcome |
|---:|---|---:|---|
| 1 | CLI basics with Typer and Rich | 4 | Build help text, options, tables, and exit codes. |
| 2 | Targets YAML and Pydantic validation | 5 | Report `path | code | reason` errors. |
| 3 | asyncio and httpx | 6 | Run concurrent checks with timeouts and retries. |
| 4 | Classification, p95, and incident state | 5 | Implement pure logic with 100% branch tests. |
| 5 | SQLite repository layer | 4 | Store final outcomes and incidents safely. |
| 6 | TLS certificate basics | 3 | Explain expiry, hostname match, and verification. |
| 7 | Packaging, uv, and console scripts | 2 | Install `pulsecheck` as a tool. |
| 8 | CI, coverage, and security checks | 4 | Keep main green with quality gates. |
| 9 | Python tag statistics, local fixtures/replay, Markdown CLI guide, and optional Docker/Mailpit/TestPyPI | 7 | Add backend-only Should work after the MVP; publishing is separate from startup. |

[Back to README](../README.md)
