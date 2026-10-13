# 07 — Data model

CSV Scout has no database. Its source, report, and quarantine files are the complete data contract.

## Source record

**Grain:** one CSV data record, excluding the header.

| Position | Column | Input type | Rule |
|---:|---|---|---|
| 1 | `order_id` | Text | Required after trimming; first valid value is the key |
| 2 | `customer_id` | Text | Required after trimming |
| 3 | `order_date` | Text | Required; exact calendar date `YYYY-MM-DD` |
| 4 | `amount` | Text | Required; finite decimal value greater than or equal to zero |

The names and positions are case-sensitive and fixed. Each record has exactly four fields. The full 20-record sample is in document 08.

| Example order_id | customer_id | order_date | amount | Result |
|---|---|---|---:|---|
| `1001` | `C001` | `2026-10-01` | `10.00` | Accepted |
| `1002` | `C002` | `2026-10-02` | `20.50` | Accepted |
| `1016` | `C013` | `2026-02-30` | `12.00` | Quarantined: invalid date |
| `1018` | empty | `2026-10-19` | `6.00` | Quarantined: missing customer |

## JSON report

**Grain:** one report per successful command. Count fields are integers. `missing_by_column` always uses source-column order. The exact field names, values, and serialization are in document 08.

## Quarantine CSV

**Grain:** one rejected source record. Its first four columns preserve the parsed input values; `reason_codes` contains one or more stable codes. Rows remain in source order. Accepted rows are not written to another file.

[Back to README](../README.md)
