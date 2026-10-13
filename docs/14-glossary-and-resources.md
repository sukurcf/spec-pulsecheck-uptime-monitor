# 14 — Glossary and resources

## Glossary

| Term | Simple meaning |
|---|---|
| CSV | A text file with values separated into columns |
| Header | The first CSV record that names the columns |
| Field | One value in a record |
| Record | One data row in a file |
| Schema | The agreed names, order, and rules for fields |
| Grain | What one record represents |
| Data profile | A small summary of values and quality counts |
| Validation | Checking data against stated rules |
| Missing value | A blank value after trimming spaces |
| Duplicate key | A later record with an already accepted key |
| Order ID | The stable key that identifies an order |
| Customer ID | The identifier used to count accepted customers |
| ISO date | A date written as `YYYY-MM-DD` |
| Decimal | A base-ten numeric value used for exact decimal parsing |
| Accepted row | A record that passes all required rules |
| Rejected row | A record written to quarantine |
| Quarantine | A separate output for records that need review |
| Reason code | A stable short label for a validation failure |
| Deterministic | Produces the same result from the same input |
| Idempotent output | Repeating a run gives identical output files |
| CLI | A program operated through text commands |
| Fixture | Small known input used by a test or demo |
| Line coverage | The share of executable lines exercised by tests |
| Branch coverage | The share of decision paths exercised by tests |

## Official learning links

These five official references support the project:

1. [Python 3.12 `csv` module](https://docs.python.org/3.12/library/csv.html) — reading and writing CSV safely.
2. [Python 3.12 `argparse` module](https://docs.python.org/3.12/library/argparse.html) — defining command-line arguments.
3. [Python 3.12 `json` module](https://docs.python.org/3.12/library/json.html) — encoding a stable JSON report.
4. [pytest documentation](https://docs.pytest.org/en/stable/) — writing and running tests.
5. [coverage.py documentation](https://coverage.readthedocs.io/en/latest/) — measuring line and branch coverage.

Recommended reading order: read the `csv` page, the `argparse` tutorial, the `json` page, the pytest getting-started page, and then the coverage.py branch coverage guide.

[Back to README](../README.md)
