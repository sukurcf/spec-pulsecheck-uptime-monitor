# 08 — CLI specification

## Command

```text
uv run csvscout validate INPUT --report REPORT --quarantine CSV
```

All three paths are required. The report and quarantine paths must be different from each other and from the input after path resolution. The CLI creates missing output directories. Successful runs overwrite the selected outputs. Exit code `0` means the file was processed, even when some rows were rejected. Exit code `2` means the file or paths could not be processed.

The exact local demo invocation is:

```text
uv run csvscout validate fixtures/orders.csv --report .local/report.json --quarantine .local/rejected.csv
```

## Input and row decisions

The UTF-8 CSV header must be exactly `order_id,customer_id,order_date,amount`. Each data record must have exactly four fields. A zero-byte file, malformed record, bad header, or file with more than 500 data records is a fatal error. A file containing only the correct header is valid with zero data rows.

For each data record, trim values for validation and comparison. Count whitespace-only cells as missing. Dates must round-trip as real `YYYY-MM-DD` calendar dates. Amounts must be finite decimal values and not less than zero; zero is valid. Check all fields and record each applicable value reason. A row with several problems is rejected once.

After value checks, an otherwise valid row is accepted if its trimmed, case-sensitive `order_id` has not already been accepted. A later valid copy is rejected with `duplicate_order_id`. Rejected rows do not reserve an ID. The unique customer count includes only accepted, trimmed, case-sensitive customer IDs.

## Stable counters and reason codes

- `row_count`: all data records, not the header.
- `accepted_rows`: rows accepted after every rule.
- `rejected_rows`: each quarantined record counted once.
- `missing_by_column`: empty/whitespace-only values per source column, in header order.
- `duplicate_order_ids`: later valid copies, not the number of distinct duplicated IDs.
- `invalid_iso_dates`: nonempty date values that are not exact real `YYYY-MM-DD` dates.
- `negative_amounts`: nonempty finite numeric amounts below zero.
- `nonnumeric_amounts`: nonempty amounts that are not finite decimal numbers.
- `unique_accepted_customers`: distinct accepted customer IDs after trimming.

Reason codes use this order when a row has multiple problems: `missing_order_id`, `missing_customer_id`, `missing_order_date`, `invalid_iso_date`, `missing_amount`, `nonnumeric_amount`, `negative_amount`, `duplicate_order_id`. Join codes with `;` in `reason_codes`. Missing fields are not also counted as invalid dates or amounts.

## Exact fixture

The fixture has exactly 20 data records. Lines 16 and 17 are later duplicate copies; line 18 has an invalid date; line 19 has a negative amount; line 20 has a missing customer. The first 15 records are accepted.

```text
order_id,customer_id,order_date,amount
1001,C001,2026-10-01,10.00
1002,C002,2026-10-02,20.50
1003,C003,2026-10-03,0.00
1004,C001,2026-10-04,15.00
1005,C004,2026-10-05,9.99
1006,C005,2026-10-06,7.25
1007,C006,2026-10-07,30.00
1008,C002,2026-10-08,4.50
1009,C007,2026-10-09,17.00
1010,C008,2026-10-10,8.00
1011,C009,2026-10-11,11.75
1012,C010,2026-10-12,19.95
1013,C011,2026-10-13,5.00
1014,C012,2026-10-14,22.00
1015,C012,2026-10-15,14.00
1002,C999,2026-10-16,20.50
1005,C004,2026-10-17,9.99
1016,C013,2026-02-30,12.00
1017,C014,2026-10-18,-2.50
1018,,2026-10-19,6.00
```

## Exact demo output

On success, standard output is exactly one line:

```text
csvscout: row_count=20 accepted=15 rejected=5 report=.local/report.json quarantine=.local/rejected.csv
```

The JSON is UTF-8, two-space indented, has the shown key order, and ends with one newline:

```json
{
  "row_count": 20,
  "accepted_rows": 15,
  "rejected_rows": 5,
  "missing_by_column": {
    "order_id": 0,
    "customer_id": 1,
    "order_date": 0,
    "amount": 0
  },
  "duplicate_order_ids": 2,
  "invalid_iso_dates": 1,
  "negative_amounts": 1,
  "nonnumeric_amounts": 0,
  "unique_accepted_customers": 12
}
```

The quarantine CSV uses UTF-8, LF line endings, minimal CSV quoting, the shown header, and source order. It ends with one newline:

```text
order_id,customer_id,order_date,amount,reason_codes
1002,C999,2026-10-16,20.50,duplicate_order_id
1005,C004,2026-10-17,9.99,duplicate_order_id
1016,C013,2026-02-30,12.00,invalid_iso_date
1017,C014,2026-10-18,-2.50,negative_amount
1018,,2026-10-19,6.00,missing_customer_id
```

For the quarantine file, preserve the parsed field values as read; trim only for validation and comparison. Repeating the same successful command must write identical bytes to both output files and leave the input bytes unchanged.

## Errors

All errors below go to standard error, end with a newline, and exit `2`. `<path>` is the path supplied by the learner.

| Condition | Exact message |
|---|---|
| Missing input file | csvscout: input file not found: `<path>` |
| Zero-byte input | csvscout: empty input: `<path>` |
| Wrong, missing, extra, repeated, or reordered header | csvscout: invalid header: expected order_id,customer_id,order_date,amount |
| Record with other than four fields | csvscout: malformed CSV record `<n>`: expected 4 fields |
| CSV parser error | csvscout: malformed CSV input |
| More than 500 data records | csvscout: row limit exceeded: maximum 500 |
| Output paths alias each other or the input | csvscout: output paths must be different from each other and the input |
| An output path cannot be written | csvscout: cannot write output: `<path>` |

For a fatal input or format error, do not create or overwrite either output. The input is always read-only.

[Back to README](../README.md)
