# PulseCheck DevOps, CI/CD, and quality

Purpose: This document defines the Git workflow, CI gates, Docker expectations, configuration rules, release process, and Definition of Done for PulseCheck.

## Git workflow

PulseCheck uses GitHub Flow.

1. Create a GitHub Issue for each feature, fix, or documentation task.
2. Create a short branch from `main`.
3. Commit small changes with Conventional Commits.
4. Open a pull request before merging to `main`.
5. Wait for CI to pass.
6. Complete the PR checklist.
7. Merge only after the branch protection rules allow it.

Direct pushes to `main` are not allowed. The trainer reviews at least 2 substantive pull requests each week.

## Branch naming

| Work type | Pattern | Example |
|---|---|---|
| Feature | `feature/<issue>-<short-name>` | `feature/12-check-command` |
| Bug fix | `fix/<issue>-<short-name>` | `fix/18-keyword-head-validation` |
| Test | `test/<issue>-<short-name>` | `test/21-ssl-expiry-thresholds` |
| Documentation | `docs/<issue>-<short-name>` | `docs/30-readme-setup` |
| Chore | `chore/<issue>-<short-name>` | `chore/34-ruff-upgrade` |

## Conventional Commits

Use one change type per commit. Keep the subject under 72 characters.

| Type | PulseCheck example |
|---|---|
| `feat` | feat(check): classify missing keyword as down |
| `fix` | fix(config): reject keyword on head targets |
| `test` | test(incident): cover recovery after degraded result |
| `docs` | docs(readme): add uv tool install steps |
| `refactor` | refactor(store): isolate sqlite repository methods |
| `chore` | chore(ci): add windows python 3.13 job |

## Pull request checklist

Each PR MUST answer these PulseCheck questions.

- Which FR, BR, NFR, or test case IDs does this PR change?
- Did `ruff format --check` and `ruff check` pass?
- Did `mypy --strict` pass for the core package?
- Did pytest pass with line coverage at least 90% and branch coverage at least 80%?
- Did pure logic modules keep 100% branch coverage?
- Are validation errors still in the documented three-column shape?
- Are webhook URLs, SMTP passwords, and secret headers absent from logs and commits?
- Does the CLI still return exit statuses 0, 1, 2, and 3 correctly?
- Did the README or CLI help change if user behaviour changed?
- Did TC-LOCAL-001 through TC-LOCAL-004 pass with prepared HTTP/TLS fixtures, no external egress, and saved persistence/failure evidence?
- Did the student describe significant AI help in the PR description?

## Branch protection

`main` MUST require these checks before merge.

| Rule | Required setting |
|---|---|
| Pull request required | Yes, for every change. |
| Required status checks | Lint, type check, tests with coverage, local acceptance, security scan, and build check. |
| Branch up to date | Required before merge. |
| Conversation resolution | Required. |
| Secret scanning | Required through Gitleaks and GitHub alerts when available. |
| Force push | Disabled. |
| Delete branch after merge | Enabled. |

## Code-quality tools and rules

| Tool | Required rule |
|---|---|
| Ruff | Run lint and format checks. Do not leave project-wide ignores for PulseCheck modules. |
| mypy | Run `mypy --strict` on the core package. Tests and scripts may use a separate relaxed config. |
| pytest | Run unit, integration, CLI, storage, TLS, and performance tests. |
| pytest-cov | Enforce at least 90% line and 80% branch coverage for the package. |
| pytest-cov per module | Enforce 100% branch coverage for classifier, incident engine, uptime calculator, p95 calculator, and SSL evaluator. |
| respx | Mock httpx only for unit tests that do not need a real socket. |
| pytest-httpserver | Exercise real HTTP status, delay, and keyword behaviour. |
| trustme | Create valid, expired, and wrong-host certificates for TLS tests. |
| time-machine | Freeze clocks for cooldown, MTTR, and SSL threshold tests. |
| pip-audit | Fail CI on known vulnerable Python dependencies unless a documented temporary exception exists. |
| Gitleaks | Fail CI if any token, webhook URL, or SMTP password is found. |
| Trivy | Should item for Docker image vulnerability scanning. |
| pre-commit | Run Ruff, mypy, whitespace, end-of-file, and Gitleaks hooks locally. |

## CI pipeline table

| Job | Trigger | Steps in words | Gate or failure condition |
|---|---|---|---|
| Validate project metadata | Pull request and push to `main` | Install uv, check lockfile, and verify package metadata. | Fails if `uv.lock` is missing or package metadata is invalid. |
| Lint and format | Pull request and push to `main` | Run Ruff format check and Ruff lint. | Fails on formatting drift or lint violations. |
| Type check | Pull request and push to `main` | Run `mypy --strict` on the core package. | Fails on any type error in application modules. |
| Unit and integration tests | Pull request and push to `main` | Run pytest with SQLite files, Typer CLI tests, httpx mocks, local HTTP server, and trustme TLS tests. | Fails on a test failure or coverage below gates. |
| Local acceptance | Pull request and push to `main` | Prepare dependencies, make/shell tooling, and trustme fixtures, then deny external egress while allowing loopback. Run `uv run --offline pytest -m local` for TC-LOCAL-001 to TC-LOCAL-004; save command logs, JSON reports, and SQLite ID assertions. | Fails on startup/demo mismatch, lost data/outbox IDs, unclear failures, or external network use. Ubuntu verifies actual make entrypoints; portable local cases run in every matrix cell. WSL2/macOS trainer evidence supplements CI. |
| Performance smoke | Pull request and push to `main` | Check 200 local targets with concurrency 50 and 100 ms delay. | Fails if the measured check phase is 10 seconds or more. |
| Security scan | Pull request and push to `main` | Run pip-audit and Gitleaks. | Fails on vulnerable dependencies or committed secrets. |
| Docker build | Pull request and push to `main`; Should once Docker exists | Build the pinned Docker image for `pulsecheck run`. | Fails if the image cannot start `pulsecheck --help`. |
| Docker scan | Pull request and push to `main`; Should once Docker exists | Run Trivy against the built image. | Fails on unaccepted high or critical findings. |
| TestPyPI publish | Version tag only; Should | Use trusted publishing to upload package to TestPyPI. | Fails if metadata, build, or trusted publisher config is invalid. |
| Markdown CLI guide verification | Pull request and push to `main`; Should | Compare local guide examples with CLI help/version and output snapshots. No site build/publication. | Fails on a documented command/output mismatch or broken local Markdown link. |

The required CI matrix is Python 3.12.x and 3.13.x on Ubuntu and Windows. WSL2/macOS start/stop evidence is required in trainer pre-check. Hosted runners, initial downloads, dependency scans, and publishing need internet; the local acceptance phase and local startup do not. No frontend coverage or cloud deployment job exists.

The SIGTERM subprocess test MUST run on Ubuntu only. Windows CI MUST cover graceful shutdown with Ctrl+C and KeyboardInterrupt. Portable unit, integration, and CLI tests MUST run in all four matrix cells.

## Docker and Compose requirements

Docker is a Should item for this CLI project. The Must scope MUST work without Docker.

| Item | Requirement |
|---|---|
| Image | Runs `pulsecheck run` by default with a mounted targets file and database directory. |
| Base image | Use an official Python 3.12 slim image pinned to a version tag, never `latest`. |
| User | Run as a non-root user where practical. |
| Config | Read targets from a mounted path such as `/config/targets.yml`. |
| Data | Store SQLite database under a mounted data directory. |
| Logs | Emit structured logs to stdout. |
| Compose services | PulseCheck, Mailpit, and one local HTTP test target. |
| Compose limits | Standard profile runs all services. Lite profile runs no mandatory Docker services. |
| Health check | The container can show `pulsecheck --help` or validate the mounted config. |
| Local ports and persistence | Preserve the document 06 fixture addresses; bind all published ports to loopback. Mailpit host SMTP/API are 1125/8125. Mount ignored `.local/` for SQLite. Optional Compose must not start alongside conflicting Python fixture listeners. |

## Configuration and secrets

Create a committed `.env.example` with names only. Do not commit `.env`.

| Name | Example value | Purpose | Secret? |
|---|---|---|---|
| `PULSECHECK_CONFIG` | `targets.yml` | Default targets file path. | No |
| `PULSECHECK_DB_PATH` | `.pulsecheck/pulsecheck.db` | SQLite database file path. | No |
| `PULSECHECK_LOG_LEVEL` | `INFO` | Default log level. | No |
| `PULSECHECK_LOG_FORMAT` | `text` | `text` or `json` logs. | No |
| `PULSECHECK_WEBHOOK_URL` | `https://example.com/webhook` | Generic Slack-compatible or Discord webhook URL. | Yes |
| `PULSECHECK_SMTP_HOST` | `localhost` | SMTP server host for Should notifier. | No |
| `PULSECHECK_SMTP_PORT` | `1125` | Loopback host SMTP port for optional Mailpit (container port 1025). | No |
| `PULSECHECK_SMTP_USERNAME` | `pulsecheck-user` | SMTP username when needed. | Yes |
| `PULSECHECK_SMTP_PASSWORD` | `replace-me` | SMTP password when needed. | Yes |
| `PULSECHECK_NOTIFICATION_FROM` | `pulsecheck@example.in` | Sender address for SMTP messages. | No |
| `PULSECHECK_TIMEZONE` | `Asia/Kolkata` | Display time zone for reports. | No |

Secret values MUST be read from environment variables or ignored local files. Secret values MUST be redacted in logs, reports, and exception messages.

Local entrypoints override configuration with `targets.local.yml`, database `.local/pulsecheck.db`, and the prepared CA bundle via `SSL_CERT_FILE`. Use `.local/demo.db` only for the recorded seed report. Runtime must not fetch dependencies or require GitHub/TestPyPI/external webhook credentials. Live mode uses its configured URLs and fails normally, without fixture fallback.

## Versioning and release

PulseCheck uses SemVer.

| Version change | Example | Meaning |
|---|---|---|
| Patch | `0.1.1` | Fixes a bug without changing CLI contracts. |
| Minor | `0.2.0` | Adds a command option or Should feature. |
| Major | `1.0.0` | Stable CLI, config, storage, and exit-status contracts. |

Release tags use `vMAJOR.MINOR.PATCH`. The student CHANGELOG MUST follow Keep a Changelog style. TestPyPI publishing is Should and must use trusted publishing, not a committed token.

## Dependency management

- Use uv and commit `uv.lock`.
- Pin direct dependencies to compatible minor lines from document 06.
- Review dependency updates in pull requests.
- Run tests after dependency updates.
- Use Dependabot if the student enables it.
- Record temporary security exceptions in an issue with an expiry date.
- Do not upgrade across a major line during week 6 unless a security fix requires it.

## Definition of Done

A PulseCheck story is done only when all matching items are true.

- The linked FR acceptance criteria pass.
- The linked BR values are unchanged.
- Unit or integration tests cover success, negative, and boundary cases.
- CLI output remains understandable without colour.
- `mypy --strict`, Ruff, pytest, coverage, pip-audit, and Gitleaks pass.
- The result is stored in SQLite only when the requirement says it is stored.
- Secrets are redacted in logs and reports.
- The README or user docs explain changed commands or options.
- The PR checklist is complete.
- The Friday demo can show the behaviour from a clean clone.
- `make local-start`, `make local-demo`, and `make local-stop` meet document 06; all four local cases pass, reset needs confirmation, and no frontend/site/cloud deliverable is introduced.

[Back to README](../README.md)
