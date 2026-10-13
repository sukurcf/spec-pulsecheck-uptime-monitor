# 11 — Five-day plan and deliverables

The total effort is exactly **5 days × 6 hours = 30 hours**, including setup, learning, tests, local run, and demo preparation.

| Day | Hours | Work and evidence |
|---|---:|---|
| **Day 1** | 6 | Set up Python 3.12 and tools; inspect the fixture; agree the four-column contract; write the first test and a thin CLI slice. |
| **Day 2** | 6 | Implement CSV input, header and row-width checks, the 500-row limit, and field validation. Cover FR-CSV-01 and the input tests. |
| **Day 3** | 6 | Implement profile counters, accepted-key/customer tracking, JSON report, and quarantine output. Cover FR-CSV-02–FR-CSV-04. |
| **Day 4** | 6 | Finish duplicate rules, clear errors, stable output, repeat-run and input-immutability checks. Complete 15–20 distinct cases and meet coverage targets. |
| **Day 5** | 6 | Test from a clean local setup, fix defects, complete the README and demo, and prepare a short explanation of design and tests. |

## Deliverables

- A locally runnable CLI in the learner's own repository.
- The exact fictional 20-row fixture and the report/quarantine outputs.
- 15–20 distinct tests with FR traceability and the required coverage target.
- A short README showing setup and the one exact demo command.

## Five-minute demo

1. **Minute 1:** Show the input header and explain the four fields.
2. **Minute 2:** Enter the exact command from document 08 and show its expected summary line.
3. **Minute 3:** Show the JSON counts and how they match the fixture.
4. **Minute 4:** Show the five quarantined rows, reason codes, and unchanged source.
5. **Minute 5:** Repeat the command, show identical output, and explain one test and one validation decision.

[Back to README](../README.md)
