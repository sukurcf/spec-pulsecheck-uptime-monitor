# 06 — Tech stack and setup

## Reference versions

Use these existing cohort version lines; do not add a runtime package dependency.

| Tool | Reference line | Use |
|---|---:|---|
| Python | 3.12.x | Required runtime and only CI Python version |
| Python standard library | Python 3.12.x | `csv`, `json`, `argparse`, plus standard-library date, decimal, and path handling |
| uv | 0.12.x | Prepare and run the learner's implementation environment |
| pytest | 9.1.x | Test runner |
| pytest-cov / coverage | 7.1.x / 7.16.x | Measure the specified test coverage |
| Ruff | 0.16.x | Lint and format Python files |
| Git | Existing cohort installation | Track the learner's implementation |
| GitHub Actions | Current hosted runners | Run the local quality checks in CI on Python 3.12.x only |

`argparse` is the selected CLI interface. Typer is not required. Python's standard library is the only runtime library. Pytest and the existing coverage and lint tools are development tools.

## Local setup

1. Install Python 3.12.x, Git, and uv 0.12.x using the trainer's approved setup.
2. In the learner's implementation repository, install its locked development tools once.
3. Keep the fictional fixture at `fixtures/orders.csv`.
4. Run the demo command below from the implementation repository root.

Initial tool preparation may download packages. After it is prepared, the demo and tests must work without network access. The specification repository itself contains no setup scripts or implementation code.

Windows learners may use WSL2 with Ubuntu 24.04 or the trainer-approved local Python 3.12 setup. No port is opened and no listener is bound; the CLI does not use `127.0.0.1` or any other network address.

## Exact demo command

```text
uv run csvscout validate fixtures/orders.csv --report .local/report.json --quarantine .local/rejected.csv
```

The CLI creates `.local/` if it is missing. It reads `fixtures/orders.csv` and overwrites only the two output paths. The expected line, JSON, and quarantine CSV are exact in [08 — CLI specification](08-cli-specification.md).

## Fixture and cleanup contract

Use the exact fictional fixture in document 08. Keep the source bytes unchanged. `.local/report.json` and `.local/rejected.csv` are generated local outputs and should not be committed. Repeating the command with the same arguments must produce the same output bytes. No start or stop command is needed.

[Back to README](../README.md)
