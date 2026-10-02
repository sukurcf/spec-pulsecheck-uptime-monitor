# PulseCheck interview preparation

Purpose: This document helps the student explain PulseCheck in interviews, prepare topic answers, improve profiles, and practise mock interviews.

## Explain your project in 2 minutes using STAR

| STAR part | PulseCheck answer outline |
|---|---|
| Situation | Small teams discover website downtime and SSL expiry too late. They need a free local monitor. |
| Task | I built `pulsecheck`, an installable Python CLI for endpoint checks, incidents, alerts, and reports. |
| Action | I used Typer, asyncio, httpx, Pydantic, sqlite3, TLS checks, notification plugins, and CI quality gates. |
| Result | The tool checks 200 local targets under 10 seconds, stores evidence, opens incidents after 3 `DOWN` results, warns at 30/14/7 SSL days, and reports uptime, p95, missed checks, and MTTR. |

A concise spoken version:

> PulseCheck is a Python CLI uptime and SSL monitor for small teams. I built it because teams often learn about outages from users. The tool reads a YAML targets file, validates it, checks endpoints concurrently with asyncio and httpx, stores final results in SQLite, opens incidents after 3 consecutive DOWN results, sends console or webhook alerts, and reports uptime, p95 latency, missed checks, and MTTR. I also added strict typing, tests, coverage gates, and GitHub Actions. This project helped me practise Python core, async I/O, HTTP, TLS, SQLite, packaging, and production-support thinking.

## Python core questions

| Question | Key points to cover |
|---|---|
| 1. Why did you use classes or protocols for notification plugins? | Common send contract; console and webhook interchangeable; easier unit tests; SMTP can be added later. |
| 2. Where did you use exceptions in PulseCheck? | Validation errors become exit code 2; storage failures become exit code 3; network exceptions become `DOWN` results. |
| 3. Which data structures are important in the incident engine? | Per-target state map; counters for consecutive DOWN and recovery; stable incident IDs. |
| 4. How would you make target iteration memory efficient? | Parse once; iterate target objects; avoid storing per-attempt rows; use generators for report rows when possible. |
| 5. What does type checking protect in this project? | Optional latency handling; enum-like status values; plugin method signatures; SQLite row mapping. |
| 6. How would a context manager help SQLite writes? | Opens transaction; commits on success; rolls back on exception; closes cursor safely. |
| 7. Why keep classifier logic pure? | Easy tests; no network or database side effects; supports 100% branch coverage; clear interview explanation. |

## Async, HTTP, and TLS questions

| Question | Key points to cover |
|---|---|
| 8. Why is asyncio useful for PulseCheck? | Many waiting network calls; fewer threads; concurrency limit controls pressure; cancellation helps shutdown. |
| 9. How do you enforce at most 50 checks at once? | Use a semaphore or bounded task scheduling; apply per-attempt timeouts; test active attempt count. |
| 10. What is the difference between timeout and retry? | Timeout limits one attempt; retry starts a later attempt; final outcome stores one attempt count. |
| 11. What is exponential backoff with jitter? | Delay grows 0.25s, 0.5s, 1s; cap 2s; jitter 0-250 ms prevents synchronized retries. |
| 12. Why is HTTP 503 classified as `DOWN` here? | It is an unexpected status unless configured; users see failure; exit code becomes 1. |
| 13. Why is a slow HTTP 200 `DEGRADED` and not `DOWN`? | Response is successful; latency exceeds threshold; degraded still counts as available. |
| 14. What does TLS hostname matching check? | Certificate names match URL host; wrong host is a TLS error; protects users from misleading certificates. |
| 15. Why must SSL checks not block the event loop? | Scheduler and other checks must continue; use asyncio stream methods or `asyncio.to_thread`; test shutdown timing. |

## SQLite and reporting questions

| Question | Key points to cover |
|---|---|
| 16. Why use SQLite instead of PostgreSQL? | Local CLI; no server setup; fits 200 targets; standard library requirement. |
| 17. Why keep targets in YAML instead of SQLite? | YAML is the user-facing interface; Git-reviewable; spec forbids SQLite as target source of truth. |
| 18. What transaction does incident opening need? | Store final check result and incident state together; rollback both on failure; avoid orphan evidence. |
| 19. How do indexes help `history` and `report`? | Filter by target and time; count status by period; find open incidents quickly. |
| 20. How do you calculate uptime %? | Completed checks not DOWN divided by completed checks; degraded counts; missed checks excluded. |
| 21. How do you calculate p95 nearest-rank? | Sort ascending; rank `ceil(0.95 * n)`; use 1-based index; five values choose the fifth. |
| 22. What does MTTR mean here? | Average duration of closed incidents; duration is ended minus started; open incidents excluded. |

## Git, Linux, packaging, and DevOps questions

| Question | Key points to cover |
|---|---|
| 23. Why does cron need stable exit codes? | Automation reacts to 0, 1, 2, and 3; distinguishes outage from config error; supports alerts. |
| 24. What is the benefit of a console entry point? | User runs `pulsecheck`; package install is clean; Typer app is discoverable. |
| 25. Why commit `uv.lock`? | Reproducible installs; stable CI; easier review of dependency changes. |
| 26. What does `mypy --strict` catch that tests may miss? | Optional values; wrong return types; plugin contract mismatch; untyped functions. |
| 27. Why run CI on Windows and Ubuntu? | Students use WSL and recruiters may use Windows; catches path and signal differences. |
| 28. What does Gitleaks protect? | Webhook URLs; SMTP passwords; bearer tokens; accidental `.env` commits. |
| 29. Why use Conventional Commits? | Clear history; changelog support; easy review; demonstrates professional practice. |
| 30. How would you debug a failing GitHub Actions job? | Read first failing step; reproduce command locally; compare OS and Python version; add a focused test or fix. |

## Project deep-dive questions

| Question | Key points to cover |
|---|---|
| 1. How did you prevent duplicate alerts during a long outage? | Dedupe by `channel` and `dedupe_key`; 30-minute cooldown; stable notification ID; retries are at-least-once. |
| 2. How did you model incident state? | Healthy, SuspectDown, Open, Recovering, Closed; opens after 3 DOWN; closes after 2 non-DOWN. |
| 3. Why does `DEGRADED` close an incident recovery sequence? | It is non-DOWN by rule; service is reachable; still visible in reports as degraded. |
| 4. How did you test SSL expiry warnings? | trustme certificates; frozen time; thresholds 30, 14, 7; fingerprint and threshold dedupe. |
| 5. What happens if validation finds three bad fields? | It reports all found errors; each line has path, code, and reason; exit code 2. |
| 6. Why store only final check outcomes? | Brief requires it; avoids noisy attempt rows; attempt count preserves retry evidence. |
| 7. How does `purge` avoid deleting important incident data? | Deletes old check and SSL evidence; never deletes open incidents; reports row counts. |
| 8. How would you add a Telegram notifier safely? | New plugin; token from env; no logs of token; tests for failure; ADR if architecture changes. |
| 9. How do missed checks affect reports? | Reported separately; excluded from uptime denominator; indicate scheduler overload. |
| 10. What would you improve after six weeks? | Better dashboard or Prometheus metrics; more notifier plugins; stronger documentation; measure real target load. |

## Fundamentals check

### Python core

Know functions, modules, packages, classes, dataclasses, abstract base classes, Protocols, exceptions, context managers, decorators, iterators, generators, type hints, `Enum`, and `pathlib`. Connect each answer to a PulseCheck module.

### SQL

Practise `SELECT`, `WHERE`, `GROUP BY`, joins, transactions, indexes, constraints, and window functions. Explain reports from `check_results`, incidents from `incidents`, and cooldown checks from `notifications_sent`.

### Git

Practise branch creation, staging, commits, rebasing a small branch, resolving one conflict, reading `git log`, and connecting PRs to Issues. Explain why direct pushes to `main` are risky.

### Linux

Know `pwd`, `ls`, `cd`, `cat`, `grep`, `chmod`, environment variables, process signals, exit codes, cron, and file permissions. Explain how cron reads PulseCheck exit codes.

### HTTP and networking

Know DNS, TCP, TLS, certificates, hostname validation, HTTP methods, status codes, headers, timeouts, retries, and latency. Explain why internet tests are avoided in CI.

### Local tools mapped to AWS services

| Local PulseCheck tool | Similar AWS idea | Interview note |
|---|---|---|
| SQLite file | Amazon RDS or DynamoDB | Local persistence versus managed storage. |
| Cron or scheduler process | EventBridge Scheduler | Periodic execution of checks. |
| Console logs | CloudWatch Logs | Structured logs with correlation IDs. |
| Webhook notifier | SNS or EventBridge target | Alert delivery with retry and dedupe. |
| Docker Compose | ECS task or local container stack | Reproducible runtime environment. |
| GitHub Actions | CodeBuild or CodePipeline | Automated quality gates. |
| MkDocs Pages | S3 static website or Amplify | Static documentation hosting. |

## Resume bullet templates

- Built `pulsecheck`, a Python CLI uptime monitor that checked `[N]` endpoints concurrently with asyncio and httpx in `[T]` seconds.
- Implemented SQLite-backed incident tracking with 3-sample DOWN detection, 2-sample recovery, and MTTR reporting for `[N]` incidents.
- Added TLS expiry and hostname validation with 30/14/7-day warning thresholds and duplicate alert suppression.
- Raised quality with mypy strict, Ruff, pytest, `[line]%` line coverage, `[branch]%` branch coverage, pip-audit, and Gitleaks in GitHub Actions.
- Packaged the tool with uv and a `pulsecheck` console entry point, then documented clean-clone setup in `[N]` steps.

## GitHub profile tips

- Pin the `pulsecheck` repository.
- Keep the README short, runnable, and honest.
- Add a screenshot or terminal output only if it contains no secrets.
- Keep Issues, PRs, and ADRs public and tidy.
- Show the CI badge only when `main` is green.

## LinkedIn tips

- Write one post about the incident engine or SSL warning design.
- Mention Python, asyncio, httpx, Typer, SQLite, pytest, mypy, and GitHub Actions.
- Link to the final demo video.
- Do not claim production use unless a real team used it.
- Use metrics with placeholders only after you measure them.

## Mock-interview checklist

- Explain the project in 2 minutes without reading notes.
- Draw the incident state machine from memory.
- Calculate uptime and p95 for a small sample.
- Explain one SQLite transaction and one index.
- Explain one Windows CI problem and one Linux signal problem.
- Show one failing validation example.
- Explain why `mypy --strict` matters.
- Admit one limitation and propose a safe improvement.

[Back to README](../README.md)
