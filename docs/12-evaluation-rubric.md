# PulseCheck evaluation rubric

Purpose: This document defines mandatory gates, scoring weights, bonus rules, deductions, grade bands, and viva questions for PulseCheck.

## Mandatory gates

If any gate fails, the result is **Rework required** for any score.

| Gate | Evidence required |
|---|---|
| CI is green on `main` | GitHub Actions run passes for Python 3.12 and 3.13 on Ubuntu and Windows. |
| Coverage thresholds are met | Whole package line coverage is at least 90%, and branch coverage is at least 80%. |
| Pure-logic coverage is met | Classifier, incident engine, uptime calculator, p95 calculator, and SSL evaluator have 100% branch coverage. |
| Clean-clone setup works | Trainer can follow README setup in 10 steps or fewer and run `pulsecheck --help`. |
| No secrets in Git history | Gitleaks and manual review find no webhook URLs, SMTP passwords, or tokens. |
| Must commands work | `init`, `validate`, `check`, `run`, `status`, `history`, `report`, and `purge` exist. |
| Student can explain code | Student explains and changes a selected module live during the viva. |

## Weighted rubric

| Category | Points |
|---|---:|
| Functional completeness (Must FRs work in the demo) | 30 |
| Code quality and design | 15 |
| Testing and coverage | 15 |
| DevOps: CI/CD, Docker, reproducibility | 10 |
| Documentation (README, ADRs, API/CLI/pipeline docs) | 10 |
| Git and engineering practices (PRs, commits, issues) | 5 |
| Final demo and viva | 15 |
| **Total** | **100** |

## Category criteria

### Functional completeness — 30 points

| Level | Criteria |
|---|---|
| Excellent | All Must FRs work from a clean clone. Validation reports multiple errors with `path | code | reason`. `check` handles concurrency, retry, status, keyword, timeout, DNS, and TLS cases. `run` opens after 3 `DOWN` results and closes after 2 non-`DOWN` results. Reports show uptime, p95 nearest-rank, missed checks, incident count, and MTTR. |
| Good | All Must commands exist, and the main happy path works. One or two edge cases need minor fixes, such as a weak error message or a missing report field. Incident and SSL flows are demonstrable. |
| Needs work | A Must command is missing, scheduler behaviour is incomplete, exit codes are wrong, SQLite evidence is unreliable, or incident and SSL rules cannot be shown. |

### Code quality and design — 15 points

| Level | Criteria |
|---|---|
| Excellent | CLI, validation, checker, storage, incident engine, SSL evaluator, and notification plugins are separated. Pure logic has small typed functions. Repository writes use transactions. `mypy --strict` passes without broad ignores. |
| Good | Modules are understandable and typed. Some functions are too large, or CLI code knows too much about storage. No design issue blocks use. |
| Needs work | Most logic lives in CLI handlers, async code is hard to cancel, storage is duplicated, type checks are bypassed, or errors are swallowed. |

### Testing and coverage — 15 points

| Level | Criteria |
|---|---|
| Excellent | At least 45 concrete tests cover validation, CLI exit codes, retries, timeouts, HTTP server integration, TLS certificates, SQLite transactions, incidents, p95, purge, and notification cooldown. Performance test proves 200 targets under 10 seconds. |
| Good | Coverage gates pass and most Must behaviours are tested. A few tests rely on broad mocks or miss boundary values. |
| Needs work | Coverage fails, tests use real internet services, pure logic lacks branch tests, or performance evidence is missing. |

### DevOps: CI/CD, Docker, reproducibility — 10 points

| Level | Criteria |
|---|---|
| Excellent | CI matrix covers Python 3.12 and 3.13 on Ubuntu and Windows. uv lockfile is committed. Ruff, mypy, tests, coverage, pip-audit, and Gitleaks are required checks. Docker and Compose Should items use pinned tags if claimed. |
| Good | Required CI gates pass, but Docker, TestPyPI, or MkDocs Should items are partial. Local setup is repeatable. |
| Needs work | CI is flaky, matrix is incomplete, lockfile is missing, security scans are absent, or Docker uses `latest`. |

### Documentation — 10 points

| Level | Criteria |
|---|---|
| Excellent | README setup works in 10 steps or fewer. CLI examples show exact commands, exit codes, and outputs. ADRs explain asyncio, sqlite3 repository, notification plugin, SSL method, and Docker choices. CHANGELOG matches tags. |
| Good | README and ADRs are useful but miss one operational detail, such as purge safety or environment variables. |
| Needs work | README cannot set up the tool, ADRs are missing, or documentation contradicts implemented CLI behaviour. |

### Git and engineering practices — 5 points

| Level | Criteria |
|---|---|
| Excellent | Branch names, Issues, PRs, commit messages, and reviews show steady weekly progress. PRs are small and linked to FR IDs. AI help is disclosed when significant. |
| Good | Git history is understandable, but some PRs are large or issue links are missing. |
| Needs work | Most work is one big commit, direct pushes to `main` are common, or PR checklist evidence is absent. |

### Final demo and viva — 15 points

| Level | Criteria |
|---|---|
| Excellent | The 10-minute demo follows the script, uses realistic local targets, explains failures clearly, and leaves time for live code changes. Viva answers connect Python, asyncio, HTTP, TLS, SQL, Git, Linux, and packaging to PulseCheck. |
| Good | Demo shows Must flows but runs slightly over time or needs prepared data for one flow. Viva answers are mostly correct. |
| Needs work | Demo cannot reproduce incidents or reports, student cannot explain their own modules, or answers rely on memorised text only. |

## Bonus rules

Bonus is at most +10 points. Bonus applies only when the base score is 60 or more and all mandatory gates pass. The final score is capped at 100. Each Could FR is worth 2 points.

| Bonus item | Maximum points | Conditions |
|---|---:|---|
| FR-UI-01 Textual dashboard | 2 | Dashboard shows text status labels and does not break normal CLI commands. |
| FR-MET-01 Prometheus metrics | 2 | Metrics endpoint is disabled by default and exposes uptime counts when enabled. |
| FR-CHECK-04 TCP and DNS checks | 2 | TCP and DNS target types work without changing HTTP target rules. |
| FR-NOTIF-04 Telegram notifier | 2 | Telegram sends one incident message and keeps tokens out of logs. |
| FR-TEST-01 Mutation testing | 2 | Mutmut kills a classifier mutant and does not block normal CI by default. |

## Deductions

| Issue | Deduction |
|---|---:|
| One undocumented environment variable affects the demo | 2 |
| CLI output relies on colour without text status | 2 |
| One report metric uses a different rule than the spec | 3 |
| README setup takes more than 10 steps | 3 |
| Docker or CI uses an unpinned `latest` tag | 3 |
| Validation stops after the first easy error | 4 |
| Webhook URL, SMTP password, or bearer token appears in logs | 5 |
| Git history hides weekly progress | 5 |
| Claimed bonus feature is not reproducible | Remove that bonus |

## Grade bands

| Score | Grade |
|---:|---|
| 85-100 | Distinction |
| 70-84 | Merit |
| 60-69 | Pass |
| Below 60 | Rework required |

## Ten project-specific viva questions

1. Why does PulseCheck count `DEGRADED` as available in the uptime formula?
2. How does nearest-rank p95 choose a value from `100, 120, 200, 800, 1000`?
3. What happens when a target is due while its previous check is still running?
4. Why are retries stored as one final result instead of one row per attempt?
5. How do you prevent duplicate webhook alerts for the same open incident?
6. Why is `keyword` invalid when the HTTP method is `HEAD`?
7. What makes a certificate hostname mismatch different from an expired certificate?
8. How does the repository layer protect incident updates and check results together?
9. Which exit code should cron see for one `DEGRADED` target, and why?
10. How would you debug a Windows-only CI failure in the Typer CLI tests?

[Back to README](../README.md)
