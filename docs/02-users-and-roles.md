# 02 — Users and roles

## Roles and personas

| Role | Persona | Main need | Permission |
|---|---|---|---|
| Junior data engineer | A beginner who receives small CSV extracts | Understand file quality before using the data | Reads the chosen input; writes only the chosen report and quarantine paths |
| Trainer | A reviewer checking the learner's reasoning | See clear, repeatable evidence for each rule | Reviews the specification, tests, and demo results |

The CLI has no sign-in or role switch. File permissions are controlled by the learner's operating system.

## User stories

| ID | Story |
|---|---|
| US-01 | As a junior data engineer, I want counts for rows, blanks, invalid values, duplicate IDs, and accepted customers so I can judge the source quickly. |
| US-02 | As a junior data engineer, I want rejected rows and reason codes in a separate CSV so I can review quality problems without losing source data. |
| US-03 | As a trainer, I want a fixed fixture, exact outputs, and distinct tests so I can verify the same requirements for every learner. |

## Ownership

The learner chooses the input and output paths for a run. The CLI must not write to the input path. The trainer reviews the result but does not operate a separate service.

[Back to README](../README.md)
