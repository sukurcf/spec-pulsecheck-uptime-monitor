# 13 — Interview preparation

## Ten project questions

1. **What problem does CSV Scout solve?**
   - Key points: small files can have blanks, invalid values, and repeated keys; the CLI makes those issues visible before use.

2. **Why check the header before reading rows?**
   - Key points: columns are the input contract; an unexpected shape must fail clearly rather than count the wrong fields.

3. **How is a missing value defined?**
   - Key points: trim whitespace; an empty result is missing; report it by column.

4. **How do you decide whether a date is valid?**
   - Key points: exact `YYYY-MM-DD` shape and a real calendar date.

5. **Why use decimal parsing for amounts?**
   - Key points: avoid binary floating-point surprises; reject non-finite and negative values; allow zero.

6. **What does “first valid order ID” mean?**
   - Key points: process source order; invalid rows do not claim a key; keep the first valid row and quarantine later valid copies.

7. **How do you handle a row with more than one problem?**
   - Key points: count each applicable field problem, reject the row once, and list reason codes in fixed order.

8. **Why keep the source file unchanged?**
   - Key points: preserve original evidence; make review and repeat runs safe; write to separate paths.

9. **How do you prove output is deterministic?**
   - Key points: fixed key/column order, stable row order, fixed serialization, and a test that compares bytes across runs.

10. **What does coverage show, and what does it not show?**
    - Key points: executed lines and branches; it does not prove correct expected values, so exact assertions remain important.

## One-minute explanation outline

State the data-quality problem, describe the four-column input, explain validation and first-valid-key behavior, show the JSON and quarantine outputs, then name the tests that protect determinism and input immutability.

[Back to README](../README.md)
