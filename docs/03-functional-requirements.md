# 03 — Functional requirements

All five requirements are Must requirements. They are bounded to one UTF-8 CSV with at most 500 data rows.

## Must requirements

| ID | Priority | Requirement | Given | When | Then |
|---|---|---|---|---|---|
| FR-CSV-01 | Must | Read the file contract | A file has the exact four headers in the required order and each data record has four fields | The learner runs `validate` | The CLI reads the records in source order, counts data records but not the header, and rejects files over 500 records before writing outputs |
| FR-CSV-02 | Must | Profile values | A structurally valid file contains blank, valid, or invalid field values | The CLI processes its records | The report gives row count, missing counts for every column, date and amount error counts, duplicate copies, accepted and rejected row counts, and unique accepted customers |
| FR-CSV-03 | Must | Accept or quarantine | Records are considered in source order | The CLI applies the data rules | It accepts only rows with valid required values and a first valid order ID; it writes every other row once to quarantine with stable reason codes |
| FR-CSV-04 | Must | Write stable outputs | The input and two distinct output paths are valid | The CLI finishes processing | It overwrites the JSON report and quarantine CSV with deterministic content, preserves source order, and never changes the input |
| FR-CSV-05 | Must | Handle local errors | The command receives a missing, empty, malformed, oversized, or unwritable file/path | The learner runs `validate` | It exits with code 2, prints the defined error to standard error, and does not create partial results for an input or format error |

## Data rules

| ID | Rule |
|---|---|
| BR-01 | The header must be exactly `order_id,customer_id,order_date,amount`, in that order. Missing, extra, repeated, or reordered header names are invalid. |
| BR-02 | Every data record must contain exactly four fields. An empty file, a malformed CSV record, or more than 500 data records is a fatal input error. A header-only file is valid and has zero records. |
| BR-03 | Trim leading and trailing whitespace when checking values and comparing identifiers. A blank or whitespace-only value is missing. Identifier comparisons are case-sensitive. |
| BR-04 | A nonmissing date must be a real calendar date written exactly as `YYYY-MM-DD`. Missing dates count as missing, not as invalid dates. |
| BR-05 | A nonmissing amount must be a finite decimal number and must not be less than zero. Zero is valid. Missing amounts count as missing, not as nonnumeric or negative. |
| BR-06 | Process rows in source order. Only a row that passes all non-duplicate checks can claim its trimmed `order_id`. Keep its first valid occurrence; quarantine later valid copies. A previously rejected row does not claim an ID. Count unique customers only among accepted rows, after trimming. |

For a row with more than one value error, count every applicable missing/invalid field but count the row only once as rejected. The order and exact spelling of reason codes are defined in [08 — CLI specification](08-cli-specification.md).

[Back to README](../README.md)
