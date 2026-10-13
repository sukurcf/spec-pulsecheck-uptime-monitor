# 01 — Project overview

## Problem

A small CSV can contain blanks, repeated keys, invalid dates, or unusable amounts. A junior data engineer needs a repeatable way to find these problems before another step uses the file.

## Five-day outcome

Build a local Python CLI that checks one fictional `orders.csv` with no more than 500 data rows. It writes a profile as JSON and rejected rows as a separate CSV. The source file stays unchanged. The demo result is fixed in [08 — CLI specification](08-cli-specification.md).

## Scope

**In scope**

- Read a CSV with exactly four required headers.
- Count rows, missing values, invalid values, later duplicate order IDs, and unique accepted customers.
- Keep the first valid occurrence of each order ID; quarantine later valid copies.
- Write deterministic outputs and clear local errors.
- Test success, data-quality, boundary, repeat-run, and file-error behavior.

**Out of scope**

- Multiple input files, database storage, scheduling, or remote data.
- Data correction, automatic guessing, or changing the source.
- User-built presentation features or services.

## Skill-to-job map

| Project task | Entry-level data engineering skill |
|---|---|
| Check the header and row width | Enforce a file contract |
| Count blanks and invalid values | Profile data quality |
| Keep a stable key and quarantine bad rows | Explain accepted and rejected records |
| Write repeatable JSON and CSV | Produce usable pipeline outputs |
| Add focused tests and a local demo | Make a small data tool reproducible |

The specification uses fictional data. The project is complete when the Must requirements and pass gates in documents 03, 09, and 12 are met within 30 hours.

[Back to README](../README.md)
