# 10 — Git and quality workflow

## Git workflow

1. Create a short-lived branch in the learner's own implementation repository.
2. Make a small change with a clear commit message.
3. Run lint, tests, and coverage locally.
4. Open a pull request, explain the change, and link the FR and TC IDs.
5. Merge only when review and required checks pass.

## CI job

CI runs on **Python 3.12.x only**. It installs locked development tools, runs Ruff, then runs the test suite with line and branch coverage. The workflow configuration belongs in the learner's implementation repository; this specification contains no workflow file.

| Stage | Check | Pass condition |
|---|---|---|
| Setup | Python 3.12.x and locked project tools | Environment is created without changing the lock |
| Lint | Ruff check and format check | No reported issue |
| Tests | pytest with coverage for package `csvscout` | All cases pass; line coverage ≥80%, branch coverage ≥70% |

Local tests use fictional files and do not require network access after setup.

## Environment and generated files

| Setting | Required value |
|---|---|
| Runtime environment variables | None |
| Input path | Supplied with the CLI command |
| Report and quarantine paths | Supplied with the CLI command |
| Generated local files | `.local/report.json` and `.local/rejected.csv`; keep out of commits |

## Definition of Done

- All five Must requirements in document 03 have clear evidence.
- The exact demo command has the output specified in document 08.
- The input remains unchanged and repeated output is byte-for-byte stable.
- All 20 acceptance cases pass and coverage reaches the stated target.
- A clean local setup and the five-minute demo are documented.

[Back to README](../README.md)
