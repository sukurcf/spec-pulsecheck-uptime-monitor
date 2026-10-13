# CSV Scout

CSV Scout is a five-day Python project for entry-level data engineers. It reads one fictional orders CSV, profiles its data, validates each row, and writes a JSON report plus a quarantine CSV.

> This repository contains specifications only. It does not contain an application. Students implement the CLI in their own repository.

## Quick facts

| Item | Value |
|---|---|
| Target role | Junior Python Data Engineer |
| Duration | 5 days × 6 hours = 30 hours |
| Required runtime | Python 3.12.x standard library |
| CLI | `argparse` |
| Tests | `pytest` |
| Input | One fictional `orders.csv`, at most 500 data rows |
| Outputs | Stable JSON report and quarantine CSV |
| Local services | None |

## Goal

Build the habit of checking a small source file before using it. The CLI reports the source row count, missing values by column, later duplicate order IDs, invalid ISO dates, negative and nonnumeric amounts, accepted rows, rejected rows, and unique accepted customers. It keeps the input unchanged and puts rejected rows in source order into a separate file.

## Local demo

After the implementation environment is prepared, the project demo must work offline with this one command:

```text
uv run csvscout validate fixtures/orders.csv --report .local/report.json --quarantine .local/rejected.csv
```

The exact 20-row fixture and expected output are in [08 — CLI specification](docs/08-cli-specification.md). Its expected counts are 20 source rows, 15 accepted rows, 5 rejected rows, and 12 unique accepted customers.

## Scope

- Use only Python 3.12 standard-library runtime modules and the specified test and quality tools.
- Require no database, container, network access, frontend, or external service.
- Keep all required sample data fictional and deterministic.
- Do not modify the input file.
- Finish all required behavior and tests within five days.

## Read this specification

Start with documents 01–04, then read the data flow, data model, CLI contract, and tests. Use the remaining documents to guide setup, delivery, and review.

| Document | Purpose |
|---|---|
| [01 Project overview](docs/01-project-overview.md) | Problem, outcome, scope, and job skills |
| [02 Users and roles](docs/02-users-and-roles.md) | Two roles and three user stories |
| [03 Functional requirements](docs/03-functional-requirements.md) | Five Must requirements and data rules |
| [04 Non-functional requirements](docs/04-non-functional-requirements.md) | Local performance, reliability, and privacy |
| [05 System architecture](docs/05-system-architecture.md) | Data flow and design decisions |
| [06 Tech stack and setup](docs/06-tech-stack-and-setup.md) | Versions, setup, and offline demo |
| [07 Data model](docs/07-data-model.md) | Source, report, and quarantine contracts |
| [08 CLI specification](docs/08-cli-specification.md) | Exact command, fixture, and outputs |
| [09 Testing strategy](docs/09-testing-strategy-and-test-cases.md) | 20 acceptance cases and coverage |
| [10 Quality workflow](docs/10-devops-ci-cd-and-quality.md) | Git, CI, and Definition of Done |
| [11 Five-day plan](docs/11-milestones-and-deliverables.md) | Tasks and five-minute demo |
| [12 Evaluation rubric](docs/12-evaluation-rubric.md) | 100-point scoring and pass gates |
| [13 Interview preparation](docs/13-interview-preparation.md) | Ten questions and a short project pitch |
| [14 Glossary and resources](docs/14-glossary-and-resources.md) | Plain-language terms and official learning links |

## Student work and AI use

Work in your own implementation repository and be ready to explain every change. You may use an AI assistant to learn concepts, understand errors, or review your work. Disclose significant assistance as your trainer requests. Do not submit generated work that you cannot explain.

## Change log

| Version | Date | Notes |
|---|---|---|
| 1.0 | 2026-10-02 | First published version; retained as project history. |
| 1.1 | 2026-10-02 | Revised specification and grading; retained as project history. |
| 1.2 | 2026-10-13 | Assigns this brief to CSV Scout and sets the duration to 5 days × 6 hours = 30 hours. Grading now uses the five Must requirements, 15–20 distinct tests, 80% line and 70% branch coverage, and the exact local demo. Earlier assessment criteria are superseded; there is no bonus work. |
