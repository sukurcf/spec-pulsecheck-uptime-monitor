# 04 — Non-functional requirements

Targets below are proposed acceptance targets, not measured results.

| ID | Area | Requirement |
|---|---|---|
| **NFR-PORT-01** | Portability | Use Python 3.12.x and the standard library at runtime. The specified local command must work after the learner's environment is prepared. |
| **NFR-PERF-01** | Performance | On a local computer with 8 GB RAM and a modern two-core CPU, process the maximum 500-row fixture and write both outputs in under 2 seconds, excluding interpreter startup. The trainer should record observed results before grading. |
| **NFR-REL-01** | Reliability | The same input and arguments produce byte-for-byte identical report and quarantine files. A fatal input error must not leave partial output files. |
| **NFR-DATA-01** | Data integrity | Open the source read-only. Reject output paths that resolve to the input or to each other. A successful run never edits or renames the source. |
| **NFR-PRIV-01** | Privacy | Use fictional sample values only. The tool has no secrets, accounts, external services, or runtime network requirement. |
| **NFR-MAINT-01** | Maintainability | Keep parsing, rule decisions, counting, and output formatting easy to test. Use stable IDs from documents 03 and 09. |

## Hardware and local operation

A laptop with 8 GB RAM, two CPU cores, and 1 GB free disk is sufficient as the proposed training profile. No server process, special port, database, container, or internet access is needed to run the demo after initial development tools are available.

[Back to README](../README.md)
