# Project Status Board — EmployeeTransferPortal

**Last Updated:** 2026-09-19 12:37
**Current Phase:** Gate 1 pending — `employee-directory` rev 4, all BRD amendments closed

---

## Lifecycle State

| Stage | State | Notes |
|---|---|---|
| Project Setup | **Complete** | Control Plane, knowledge base, governance and execution layer created |
| BRD Ingestion | **Complete** | `docs/Requirement for SDD.docx` ingested into `.ai-context/BRD.md`, now v1.1 with all 14 open questions resolved |
| Gate 0 — BRD Review | **APPROVED** | AMD-01 and AMD-02 both resolved 2026-09-19. **No open amendments.** |
| Spec Generation | **In progress** | 1 of 6 specs authored (`employee-directory`) |
| Gate 1 — Spec Peer Review | **Pending Review** | `employee-directory` submitted 2026-09-18 21:02:21. Soumyadeep Adhikary. `author != reviewer` **in force**. |
| Plan / Tasks / Test Cases | Not started | — |
| TDD (RED then GREEN) | Not started | — |
| Gate 2 — Code Review | Not started | **Reviewer unassigned** |
| Release | Not started | — |

---

## Active Blocks

### 1. ~~Spec generation blocked~~ — CLEARED 2026-09-18

Gate 0 approved by Vaibhaw Soni at 20:47:53. `.ai-context/BRD.md` v1.2 is **Approved**. Specs may now be drafted via `int-sdd-lifecycle`.

> ⚠ **Approved under a waiver.** The approver authored the §9 resolutions, so `author != reviewer` was waived at Gate 0. The 14 resolutions and 6 derived requirements became binding **without independent review**. Many carry author-invented thresholds unsupported by the source document. Table them at the first Gate 1 review.

### 2. Gate 2 reviewer unassigned
`.ai-context/project_context.md` records Gate 2 as `TBD`. Approval rights are verified by Git email against that roster, so **no Gate 2 approval can be granted** and no feature can reach release.
**To clear:** assign a Technical Lead / Senior Developer and update the roster in `project_context.md`.

### 3. Database migration tooling undecided
`ddl-auto: none`, so the schema is never generated from entities. A migration tool (Flyway or Liquibase) must be selected via ADR **before the first entity ships**.
**To clear:** record `.ai-context/decisions/ADR-001.md`.

---
### 4. ~~Open questions~~ and ~~derived requirements~~ — RATIFIED 2026-09-18

All 14 resolutions (OQ-01 to OQ-14) and all 6 derived requirements (BRD-019 to BRD-024) ratified at Gate 0. Assumption A-03 confirmed. Consequences C-01 to C-07 accepted as open engineering work. Conflicts DC-01 to DC-06 acknowledged.

### 5. Seven consequences remain as engineering work

Accepted at Gate 0, not resolved by it. Sharpest: **C-01** eligibility rules need employee data `employee-directory` does not hold · **C-04** async fan-out plus retry may require a message broker and therefore an ADR · **C-06** JaCoCo needed before coverage floors are enforceable · **C-07** the jurisdiction governing 7-year retention is unknown.
**To clear:** address each before the work it gates begins.

### 6. Four ADRs now required

`architecture.md` §9.8: async execution and retry mechanism, database migration tooling, encryption at rest for the transfer reason, and `audit` module placement. Migration tooling was already open; the resolutions made it urgent by defining concrete tables.
**To clear:** record each in `.ai-context/decisions/`.


### 7. ~~AMD-01~~ — RESOLVED 2026-09-19 (Option A)

`employee-directory` publishes a narrow `applyTransfer` write operation. OQ-05 confirmed, OQ-03 narrowed. **BRD-014 is implementable and ELIG-03 is functional.** `orchestration` is unblocked for specification.

### 8. ~~AMD-02~~ — RESOLVED 2026-09-19 (Option A)

Org-data change applies **immediately**, keeping the OQ-04 fan-out supplied with the new department and location. The directory reports post-transfer values for ~30 days **by design**, pinned by `employee-directory.AC24`.

**Carried forward — C-12.** ELIG-03 must be written as `last_transfer_completed_at <= today - 12 months`, never a between-range, or it fails open against the future date. Must land in the `approval` spec with its own acceptance criterion.

**All BRD amendments are now closed.**

## Active Specs

| Spec ID | Title | Status | Owner | Last Updated | Notes |
|---|---|---|---|---|---|
| `employee-directory` | Employee Directory & Reference Data | **In Peer Review** (rev 4) | Vaibhaw Soni | 2026-09-19 | Complete. 24 ACs, 29 tests. Owns BRD-002–004, 014. No BRD dependencies remain. |

### Planned specs — not yet drafted

| Spec ID | Title | BRD coverage | Depends on |
|---|---|---|---|
| `transfer-request` | Transfer Request Lifecycle | BRD-001–010, 019, 023 | `employee-directory` |
| `approval` | Manager & HR Approval | BRD-012, 013, 020, 021 | `employee-directory`, `transfer-request` |
| `orchestration` | Downstream Orchestration | BRD-011, 014–017, 024 | `transfer-request`, `approval` |
| `notification` | Employee Notification | BRD-018 | `transfer-request` |
| `audit-trail` | Immutable Audit Trail | BRD-022 | `employee-directory` |

Slicing ratified 2026-09-18: six specs, one per module in `architecture.md` §9.7.

## Daily Execution Log

### 2026-09-19
- **`employee-directory`**: author self-review before reviewer pickup. Revised to rev 2 — closed four code-generation blockers (JWT principal mapping, published DTO shapes, security authorities, seed data), corrected four internal inconsistencies, added deterministic ordering, a 403 criterion and mandated log statements, retargeted the N+1 criterion. Now 19 ACs and 23 spec-derived tests.
- **BRD**: **AMD-01 raised and RESOLVED** the same day — Option A ruled at 12:26. OQ-05 confirmed, OQ-03 narrowed, BRD-014 implementable, ELIG-03 functional. BRD v1.4. **AMD-02 raised** (effective-date timing) with a working assumption.
- **`employee-directory`**: rev 3 — added `applyTransfer`, the `applied_transfers` idempotency store, 4 ACs and 5 tests. Coverage tier raised to Critical 80 percent.
- **BRD**: **AMD-02 raised and RESOLVED** — Option A at 12:37, apply immediately. BRD v1.5, **no open amendments**. Consequence C-12 carried to the `approval` spec.
- **`employee-directory`**: rev 4 — AC24 pins the accepted early-visibility behaviour; C-12 recorded as a forward constraint. 24 ACs, 29 tests. Spec complete for Gate 1.

### 2026-09-18
- **Project setup**: INT Control Plane, knowledge base and Spring Boot baseline created. Build compiles on Java 21.
- **BRD**: `docs/Requirement for SDD.docx` ingested to `.ai-context/BRD.md`; 14 open questions resolved; **Gate 0 APPROVED** under a recorded `author ≠ reviewer` waiver.
- **`employee-directory`**: spec authored and submitted for Gate 1 review. 14 acceptance criteria, 16 spec-derived test cases, 5 API contracts. Resolves consequence C-01 by extending the `employees` schema with four HR attributes. Raises three questions for the reviewer: derived list endpoints, `/api/v1` path versioning, and coverage tier. **HALTED** pending Gate 1.

## Reviewer Roster

| Gate | Reviewer | Email | Status |
|---|---|---|---|
| Gate 0 | Soumyadeep Adhikary | soumyadeep@intglobal.com | Assigned |
| Gate 1 | Soumyadeep Adhikary | soumyadeep@intglobal.com | Assigned |
| Gate 2 | TBD | TBD | **Unassigned** |

---

## Open Architectural Items

Tracked in full in `.ai-context/architecture.md` §7.

| # | Item | Status |
|---|---|---|
| 1 | Migration tooling (Flyway vs Liquibase) | Open — ADR required |
| 2 | JWT implementation | Open — spec required |
| 3 | Dockerfile / compose stack | Open |
| 4 | Java 21 to 25 upgrade | Open — build targets 21; JDK 25 not installed |
| 5 | Module decomposition | Proposed — 6 modules, awaiting Gate 0 then Gate 1 |
| 6 | Observability stack | Open |
| 7 | API versioning strategy | Open |
| 8 | Async execution & retry mechanism | **Open — ADR required** | 
| 9 | Encryption at rest for transfer reason | **Open — ADR required** |
| 10 | `audit` module placement (business module vs shared) | **Open — ADR required** |

---

## Repository Facts

| Fact | Value |
|---|---|
| Git repository | **Initialized** 2026-09-18 20:47 |
| Gate email verification | **Operational** — `user.email` = vaibhaw.soni@intglobal.com |
| Business modules | None — `modules/` is intentionally empty; 5 candidates proposed in `architecture.md` §9, awaiting Gate 1 |
| Build target | Java 21 (intended target: 25) |

---

## Build & Test Status (verified 2026-09-18)

| Check | Result |
|---|---|
| `./mvnw clean compile` | **BUILD SUCCESS** — Java 21, Spring Boot 4.1.1 |
| `./mvnw test` | **BUILD FAILURE** — 1 test, 1 error (expected at baseline, see below) |

### Why the baseline test is red

`EmployeeTransferPortalApplicationTests.contextLoads` starts the full Spring context, which initializes the JPA `EntityManagerFactory` and therefore requires a working database connection. The required environment variables are not set, so the datasource attempts to connect as the literal user `${ETP_DB_USERNAME}` and PostgreSQL rejects it.

A PostgreSQL server **is** reachable on `localhost:5432` — it returned an authentication failure rather than a connection refusal. Only the credentials and the database are missing.

This was **not** worked around by weakening the configuration: adding an embedded database would introduce a datastore without an ADR, and hardcoding credentials would violate the security posture.

### To turn the baseline green

1. Create the database, e.g. `CREATE DATABASE employee_transfer_portal;`
2. Export the required environment variables before building:
   - `ETP_DB_URL` (defaults to `jdbc:postgresql://localhost:5432/employee_transfer_portal`)
   - `ETP_DB_USERNAME`
   - `ETP_DB_PASSWORD`
3. Re-run `./mvnw test` from `src/backend/`.

### Build tooling note

The Maven installation on this machine is the **source** distribution, not the binary one — it has no `boot/` or `lib/` directory, so the `mvn` on `PATH` cannot run (`ClassNotFoundException: org.codehaus.plexus.classworlds.launcher.Launcher`). The project therefore ships the official **Maven Wrapper 3.3.4** in script-only mode: `./mvnw` downloads Apache Maven 3.9.16 on first use, and no `maven-wrapper.jar` is committed. Use `./mvnw` rather than `mvn` in this repository.


## Next Actions

**Blocking — Gate 1 review of `employee-directory` by Soumyadeep Adhikary.** Nothing downstream of this spec may proceed: no plan, no tasks, no test cases, no code.

1. **Gate 1 review** — Soumyadeep Adhikary reviews `.ai-context/specs/employee-directory.spec.md` under Git identity `soumyadeep@intglobal.com`. Four decisions needed: C-01 schema extension, derived list endpoints, `/api/v1` versioning, coverage tier.
2. **Record the migration ADR** (Flyway vs Liquibase) — hard-blocks TDD GREEN for this spec; no table can be created without it.
3. **Record the API versioning ADR** — changes every endpoint path in this spec if rejected.
4. **Gate 1 architecture approval** of the six-module proposal in `architecture.md` §9.7 — blocks creating `modules/employee-directory/` on disk.
5. **Assign the Gate 2 reviewer.** Still unassigned; blocks all release.
6. **Assign a Technical Lead.** Gate 1 technical concurrence cannot be recorded without one, and this spec touches Security Posture (DP-01, DP-03) and Architectural Constraints.
7. **Add JaCoCo** to `src/backend/pom.xml` so coverage floors are enforceable (consequence C-06).
