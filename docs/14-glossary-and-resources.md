# PulseCheck glossary and resources

Purpose: This document defines PulseCheck terms and lists verified official documentation and free learning resources.

## Glossary

| Term | Simple definition |
|---|---|
| PulseCheck | The student-built Python CLI uptime and SSL monitor. |
| Target | One website or API endpoint listed in the targets file. |
| Targets file | The YAML file that defines global settings and monitored targets. |
| Check | One operation that tests a target and produces a final result. |
| Final outcome | The stored result after all retries for one check finish. |
| Attempt | One HTTP request inside a check. |
| Retry | Another attempt after a failed or timed-out attempt. |
| Jitter | A small random delay added to retry backoff. |
| Timeout | Maximum time allowed for one network attempt. |
| Concurrency | Number of network attempts allowed at the same time. |
| Latency | Time in milliseconds for a target response. |
| Latency threshold | Limit above which a successful response becomes `DEGRADED`. |
| UP | Status for a successful check within the latency threshold. |
| DEGRADED | Status for a successful check that is slower than the threshold. |
| DOWN | Status for connection, DNS, TLS, timeout, status, or keyword failure. |
| UNKNOWN | Display status for a configured target with no stored result yet. |
| Expected status code | HTTP status code that is accepted for a target. |
| Keyword check | A body-text check that must find a configured word or phrase. |
| Scheduler | The long-running process behind the `run` command. |
| Missed check | A due check that is skipped because the previous target check is still running. |
| Incident | A period where a target is treated as down by consecutive-result rules. |
| Open incident | Incident that has started and has not recovered yet. |
| Closed incident | Incident that recovered after enough non-DOWN results. |
| Recovery | The sequence of non-DOWN results that closes an incident. |
| MTTR | Mean time to recovery for closed incidents. |
| Uptime percentage | Completed checks not DOWN divided by completed checks, multiplied by 100. |
| p95 latency | Latency value at the nearest-rank 95th percentile. |
| Nearest-rank percentile | Percentile method that sorts values and uses rank `ceil(p * n)`. |
| SSL certificate | TLS document that proves a server name and enables HTTPS trust. |
| Hostname match | Check that the certificate names match the target URL host. |
| Certificate fingerprint | Stable hash that identifies one certificate. |
| Expiry warning | Alert sent when a certificate reaches 30, 14, or 7 days left. |
| Notification channel | Destination type such as console, webhook, or SMTP. |
| Webhook | HTTP endpoint that receives a JSON alert. |
| SMTP | E-mail protocol used by the optional e-mail notifier. |
| Dedupe key | Stable key used to prevent duplicate notifications. |
| Cooldown | Time window that suppresses repeat reminders for the same incident. |
| Repository layer | Code boundary that reads and writes SQLite data. |
| Schema version | Stored database version used for safe migrations. |
| Purge | Command that deletes old evidence rows. |
| Correlation ID | Identifier that links logs to one stored check result. |
| JSON log | One structured log object per line. |
| Exit code | Process result number used by cron and CI. |
| Lite profile | 8 GB laptop setup that runs all Must features without Docker. |
| TestPyPI | Test package registry used before real Python package publishing. |

## Official documentation links

These URLs were checked on 2026-10-02. Use official documentation or official project pages only.

| Tool | Official link |
|---|---|
| Python 3.12 | <https://docs.python.org/3.12/> |
| uv | <https://docs.astral.sh/uv/> |
| Typer | <https://typer.tiangolo.com/> |
| Rich | <https://rich.readthedocs.io/> |
| httpx | <https://www.python-httpx.org/> |
| asyncio | <https://docs.python.org/3.12/library/asyncio.html> |
| Pydantic | <https://docs.pydantic.dev/> |
| PyYAML | <https://pyyaml.org/wiki/PyYAMLDocumentation> |
| sqlite3 | <https://docs.python.org/3.12/library/sqlite3.html> |
| ssl | <https://docs.python.org/3.12/library/ssl.html> |
| Jinja2 | <https://jinja.palletsprojects.com/> |
| pytest | <https://docs.pytest.org/> |
| pytest-asyncio | <https://pytest-asyncio.readthedocs.io/> |
| AnyIO | <https://anyio.readthedocs.io/> |
| respx | <https://lundberg.github.io/respx/> |
| pytest-httpserver | <https://pytest-httpserver.readthedocs.io/> |
| trustme | <https://trustme.readthedocs.io/> |
| time-machine | <https://time-machine.readthedocs.io/> |
| Hypothesis | <https://hypothesis.readthedocs.io/> |
| Ruff | <https://docs.astral.sh/ruff/> |
| mypy | <https://mypy.readthedocs.io/> |
| pre-commit | <https://pre-commit.com/> |
| pip-audit | <https://github.com/pypa/pip-audit> |
| Gitleaks | <https://gitleaks.io/> |
| Docker | <https://docs.docker.com/> |
| Docker Compose | <https://docs.docker.com/compose/> |
| Mailpit | <https://mailpit.axllent.org/> |
| GitHub Actions | <https://docs.github.com/en/actions> |
| PyPI trusted publishing | <https://docs.pypi.org/trusted-publishers/> |
| MkDocs | <https://www.mkdocs.org/> |
| MkDocs Material | <https://squidfunk.github.io/mkdocs-material/> |
| pipx | <https://pipx.pypa.io/> |
| Trivy | <https://trivy.dev/> |
| Keep a Changelog | <https://keepachangelog.com/> |
| Semantic Versioning | <https://semver.org/> |

## Free learning resources

| Topic | Resource |
|---|---|
| Python basics and standard library | Python tutorial: <https://docs.python.org/3.12/tutorial/> |
| Packaging with uv | uv guides: <https://docs.astral.sh/uv/guides/> |
| Typer CLI design | Typer tutorial: <https://typer.tiangolo.com/tutorial/> |
| Rich tables and console output | Rich introduction: <https://rich.readthedocs.io/en/stable/introduction.html> |
| asyncio | Python asyncio docs: <https://docs.python.org/3.12/library/asyncio.html> |
| httpx async client | httpx async support: <https://www.python-httpx.org/async/> |
| Pydantic validation | Pydantic concepts: <https://docs.pydantic.dev/latest/concepts/models/> |
| SQLite | sqlite3 tutorial: <https://docs.python.org/3.12/library/sqlite3.html#tutorial> |
| TLS basics | Python ssl docs: <https://docs.python.org/3.12/library/ssl.html> |
| pytest | pytest getting started: <https://docs.pytest.org/en/stable/getting-started.html> |
| mypy | mypy getting started: <https://mypy.readthedocs.io/en/stable/getting_started.html> |
| GitHub Actions | GitHub Actions quickstart: <https://docs.github.com/en/actions/writing-workflows/quickstart> |
| Docker | Docker get started: <https://docs.docker.com/get-started/> |
| Trusted publishing | PyPI trusted publishers: <https://docs.pypi.org/trusted-publishers/> |

## Suggested reading order

1. Read the README and documents 01 to 04 to understand scope and rules.
2. Read document 07 before writing SQLite repository code.
3. Read document 08 before implementing any CLI command.
4. Read document 09 before naming tests or choosing fixtures.
5. Read Typer, Rich, and Pydantic docs before week 1 demo.
6. Read asyncio and httpx docs before implementing `check`.
7. Read sqlite3 and transaction docs before storing incidents.
8. Read ssl and trustme docs before SSL warnings.
9. Read pytest, mypy, Ruff, and coverage docs before making CI required.
10. Read Docker, Mailpit, TestPyPI, and MkDocs only after Must scope is green.

[Back to README](../README.md)
