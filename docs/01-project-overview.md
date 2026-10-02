# PulseCheck project overview

Purpose: This document explains the PulseCheck problem, product vision, scope, constraints, and success measures for version 1.1.

## Problem statement

Small companies and college IT teams run websites, APIs, and portals without a paid monitoring service. They often learn about downtime from users. They also miss SSL renewals until clients reject expired certificates. PulseCheck gives these teams a local command-line monitor that checks endpoints, stores evidence, detects incidents, sends alerts, and creates uptime reports.

## Business context

The example customer is a fictional college IT team in Pune. The team runs `www.example.in`, a fee portal, and internal APIs. They need a free tool that can run on a laptop, lab server, cron host, or CI job. The team lead needs weekly uptime evidence for management.

## Vision

PulseCheck MUST be a dependable local monitor for small teams. It MUST teach Python core skills with an SRE and production-support flavour. A fresher MUST be able to implement the mandatory scope in about 120 hours, including about 30 hours of guided learning and setup.

## Measurable goals

| Goal | Exact target |
|---|---:|
| Total project duration | 6 weeks, about 180 hours |
| Must scope effort budget | At most about 120 hours |
| Guided learning and setup budget | About 30 hours inside the Must budget |
| Remaining time | About 60 hours for Should items, hardening, viva, and contingency |
| Must completion date | End of week 4, or middle of week 5 at the latest |
| Mandatory performance test | 200 targets in under 10 seconds |
| Performance concurrency | 50 concurrent checks |
| Local test server delay | Fixed 100 ms response delay |
| Whole package coverage | Line coverage at least 90%; branch coverage at least 80% |
| Pure-logic coverage | 100% branch coverage for classifier, incident engine, uptime, percentile, and SSL evaluator |
| Test catalog size | At least 45 concrete test cases |

## Non-goals and out of scope

PulseCheck MUST NOT include multi-region probes, user accounts, a SaaS control plane, frontend work, terminal/web dashboards, report pages, documentation-site publication, SMS notifications, or cloud deployment exercises, even as optional work. It MUST NOT store targets in SQLite. Targets live only in the YAML targets file because that file is the CLI configuration interface. Reports are Markdown/text/JSON/CSV only.

## Scope by priority

### Must scope for the MVP

- YAML targets file with exact field validation.
- Commands: `init`, `validate`, `check`, `run`, `status`, `history`, `report`, and `purge`.
- Async HTTP checks with httpx, timeouts, retries, jitter, and a concurrency limit.
- Result classification as `UP`, `DEGRADED`, or `DOWN`.
- Scheduler that respects each target `interval_seconds` value.
- SQLite storage for final check results, incidents, notifications, SSL observations, and schema version.
- Incident state machine with default open threshold `N=3` and close threshold `M=2`.
- Uptime, average latency, p95 latency, incident count, missed check count, and MTTR reports.
- SSL expiry and hostname checks with warnings at 30, 14, and 7 days.
- Console and generic webhook notification plugins.
- Notification de-duplication and a default 30-minute cooldown.
- Logging controls, documented exit codes, and Python package installation.
- Local start/stop, migrations, fictional seed data, loopback HTTP/TLS/webhook fixtures, and offline acceptance after initial downloads (FR-LOCAL-01).

### Should scope after the MVP

- SMTP e-mail notifier tested with Mailpit.
- CSV history and report output.
- Python report statistics grouped by tag.
- SSL issuer and subject fields in output.
- Docker image and Docker Compose demo with Mailpit and a local HTTP test target.
- Maintenance windows that suppress alerts.
- TestPyPI publishing as a separate networked release exercise and a Markdown CLI usage guide.

### Could scope after all Must and Should work

- Python recorded-result replay validation with isolated SQLite evidence.
- Prometheus metrics endpoint.
- TCP port and DNS checks.
- Telegram notifier.
- Mutation testing with mutmut.

## Assumptions

- The student knows basic Python, SQL, Git, and terminal use.
- Windows users run commands inside WSL2 Ubuntu.
- The student repository is public and named `pulsecheck`.
- The trainer account is `@sukurcf`.
- All mandatory tools are free and run locally.
- The trainer validates the lite profile on an 8 GB laptop before week 1.
- Initial dependency downloads and hosted CI/submission need internet. Local training/tests use prepared localhost HTTP/TLS fixtures; normal live website checks still require access to the configured websites.

## Constraints

| Constraint | Requirement |
|---|---|
| Specification version | 1.1, dated 2026-10-02; v1.0 remains in the README release history |
| Python baseline | Python 3.12.x |
| CI Python versions | Python 3.12.x and 3.13.x |
| CLI framework | Typer |
| HTTP client | httpx with asyncio |
| Validation model | Pydantic v2 plus PyYAML parsing |
| Database | SQLite through the standard-library `sqlite3` module |
| Docker | SHOULD, not Must, for this CLI project |
| Secrets | Read from environment variables or ignored local files only |
| Delivery guarantee | At-least-once notifications with stable IDs and de-duplication |

## Success criteria

- `pulsecheck validate` reports all invalid target fields with field path and reason.
- `pulsecheck check` detects `UP`, `DEGRADED`, and `DOWN` targets in one run.
- `pulsecheck run` opens an incident after 3 consecutive `DOWN` results.
- `pulsecheck run` closes the incident after 2 consecutive non-`DOWN` results.
- SSL warnings are sent once per certificate at the 30, 14, and 7 day thresholds.
- `pulsecheck report` shows uptime %, average latency, p95 latency, incident count, missed checks, and MTTR.
- Exit codes are exactly 0, 1, 2, and 3 for the documented conditions.
- CI is green on `main`, coverage gates pass, and no secrets exist in Git history.
- The final demo shows a DOWN incident, recovery, SSL warning, purge, and weekly report.
- TC-LOCAL-001 through TC-LOCAL-004 prove startup, deterministic offline reporting/checks, persistence, and clear local dependency/input failures using the contract in document 06.

## Skills learned and job relevance

| Skill | Why it matters | Job-description keywords |
|---|---|---|
| Python packaging | Build an installable CLI tool. | `pyproject.toml`, uv, pipx, console script |
| Async programming | Check many endpoints without many threads. | asyncio, cancellation, timeout, concurrent I/O |
| HTTP operations | Diagnose common website failures. | status code, DNS, TLS, latency, retry |
| SQLite design | Store monitoring evidence locally. | schema version, transaction, index, repository pattern |
| CLI design | Support humans and automation. | Typer, Rich, exit code, JSON output |
| Testing | Prove behaviour with local services. | pytest, pytest-asyncio, respx, pytest-httpserver, trustme |
| SRE basics | Explain service health with metrics. | uptime, p95, MTTR, incident, alert cooldown |
| Engineering practice | Ship a maintainable portfolio project. | CI, pre-commit, mypy strict, Conventional Commits |

[Back to README](../README.md)
