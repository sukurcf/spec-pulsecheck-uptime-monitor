# 12 — Evaluation rubric

The rubric totals 100 points. It is sized for five training days and has no bonus points.

| Area | Points | Evidence |
|---|---:|---|
| Functional and data correctness | 40 | All five Must requirements; exact fixture counts and reason handling |
| Data engineering practice | 25 | Clear CSV contract, row validation, stable-key handling, useful profiling and quarantine |
| Tests and CI | 15 | 15–20 distinct cases, FR traceability, CI on Python 3.12.x, coverage target |
| Local reproducibility | 10 | Exact command works offline after setup; repeated outputs are identical |
| Documentation and Git | 5 | Concise setup, meaningful commits, clear change description |
| Demo and viva | 5 | Five-minute demo and clear explanation of key decisions |
| **Total** | **100** | |

## Pass gates

Before points are awarded, the learner must show all five Must requirements working, match the exact demo outputs, leave the source unchanged, pass the acceptance tests, and reach at least 80% line and 70% branch coverage for `csvscout`. These gates fit within the 30-hour plan.

The trainer records points using the evidence above. A strong explanation cannot replace a failing data rule, and a passing test suite cannot replace the exact demo contract.

[Back to README](../README.md)
