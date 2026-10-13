# 09 — Testing strategy and test cases

Use local, fictional CSV fixtures only. The 20 cases below cover each Must requirement. Each case has Must priority. Cases may be unit or CLI-level tests; CLI-level cases compare exit status, exact output, file bytes, or counts as stated.

## Acceptance cases

| ID | Priority | Input and action | Expected result | Requirement |
|---|---|---|---|---|---|
| TC-CSV-001 | Must | Run the exact 20-row demo fixture and command from document 08 | Exit `0`; exact stdout, JSON, quarantine rows, and counts match document 08: 20 read, 15 accepted, 5 rejected, 12 unique customers | FR-CSV-01–FR-CSV-04 |
| TC-CSV-002 | Must | Correct header plus two valid rows with distinct IDs | Exit `0`; 2 accepted, 0 rejected, all error counters 0 | FR-CSV-01–FR-CSV-03 |
| TC-CSV-003 | Must | Input path `fixtures/missing.csv` does not exist | Exit `2`; stderr is `csvscout: input file not found: fixtures/missing.csv`; no outputs created | FR-CSV-05 |
| TC-CSV-004 | Must | Separately try header `order_id,customer_id,order_date`, `order_id,customer_id,order_date,amount,extra`, `order_id,customer_id,amount,order_date`, and `order_id,customer_id,amount,amount` | Each exits `2` with `csvscout: invalid header: expected order_id,customer_id,order_date,amount`; no outputs | FR-CSV-01, FR-CSV-05 |
| TC-CSV-005 | Must | Zero-byte input | Exit `2`; exact empty-input message; no outputs | FR-CSV-05 |
| TC-CSV-006 | Must | Correct header and no data records | Exit `0`; row, accepted, rejected, and unique-customer counts are 0; quarantine contains its header only | FR-CSV-01–FR-CSV-04 |
| TC-CSV-007 | Must | A data record has three fields; repeat with five fields | Each exits `2`; exact malformed-record message names record 2; no outputs | FR-CSV-01, FR-CSV-05 |
| TC-CSV-008 | Must | Correct header and 500 valid rows with unique IDs | Exit `0`; `row_count=500`, `accepted_rows=500`, `rejected_rows=0` | FR-CSV-01–FR-CSV-03 |
| TC-CSV-009 | Must | Correct header and 501 valid rows | Exit `2`; exact row-limit message; no outputs | FR-CSV-01, FR-CSV-05 |
| TC-CSV-010 | Must | One row has four empty cells: `,,,` | Exit `0`; each missing-column count is 1; one rejected row with the four missing reason codes in defined order | FR-CSV-02–FR-CSV-03 |
| TC-CSV-011 | Must | Row has `order_id` and `amount` made of spaces, with other fields valid | Exit `0`; order ID and amount missing counts are 1; row rejected once; blank values are not counted as other invalid values | FR-CSV-02–FR-CSV-03 |
| TC-CSV-012 | Must | Valid row except date is `2026-02-30` | Exit `0`; `invalid_iso_dates=1`; row has `invalid_iso_date` | FR-CSV-02–FR-CSV-03 |
| TC-CSV-013 | Must | Valid row except date is `2026-1-02` | Exit `0`; `invalid_iso_dates=1`; non-padded date is rejected | FR-CSV-02–FR-CSV-03 |
| TC-CSV-014 | Must | Valid row except amount is `-0.01` | Exit `0`; `negative_amounts=1`; row has `negative_amount` | FR-CSV-02–FR-CSV-03 |
| TC-CSV-015 | Must | Valid row except amount is `abc`; repeat with `NaN` | Each run exits `0`; `nonnumeric_amounts=1`; row has `nonnumeric_amount` | FR-CSV-02–FR-CSV-03 |
| TC-CSV-016 | Must | One row has a blank customer, date `2026-02-30`, and amount `-1` | Exit `0`; rejected once; missing customer, invalid-date, and negative counters are each 1; reason codes are in the defined order | FR-CSV-02–FR-CSV-03 |
| TC-CSV-017 | Must | Two otherwise valid rows share order ID `A1`; second has a different customer | Exit `0`; first row accepted, second quarantined; `duplicate_order_ids=1`, accepted count 1 | FR-CSV-03 |
| TC-CSV-018 | Must | First row for order ID `A1` has an invalid date; second `A1` row is valid | Exit `0`; invalid row quarantined; valid row accepted; duplicate count 0 | FR-CSV-03 |
| TC-CSV-019 | Must | Two accepted rows have customer IDs `C1 ` and ` C1`; one rejected row has customer `C2` | Exit `0`; accepted count 2 and unique accepted customer count 1 | FR-CSV-02–FR-CSV-03 |
| TC-CSV-020 | Must | After setup, disable the network; save source bytes, run the demo twice to the same output paths, and compare source/report/quarantine bytes | Both runs exit `0` without network access; source is unchanged; output files persist after each run and are byte-for-byte identical across runs | FR-CSV-04 |

## Coverage and traceability

Measure coverage for the `csvscout` Python package. The target is at least **80% line coverage and 70% branch coverage**. Do not claim a measured result in this specification. Map each test to one or more FR IDs as shown in the table. Fatal input cases must also verify that neither output is created or overwritten.

[Back to README](../README.md)
