# Project Context

## Identity
| Field | Value |
|---|---|
| Project Name | EmployeeTransferPortal |
| Project Type | Backend Only |
| Architecture Style | Modular Monolith (Microservice Ready) |
| Repository Root | `.` (this repository) |
| Setup Date | 2026-09-18 |
| Setup Standard | INT AI-First Development Architecture |

## Technology Stack
| Layer | Technology | Notes |
|---|---|---|
| Language | Java | Build targets **21** today; **25** is the agreed intended target |
| Framework | Spring Boot 4.x | Spring Framework 7 baseline |
| Build Tool | Maven | Wrapper committed under `src/backend/` |
| Database | PostgreSQL | Connection supplied via environment variables only |
| ORM / Data Access | Hibernate via Spring Data JPA | Bundled with `spring-boot-starter-data-jpa` |
| Authentication & Security | Spring Security + JWT | Framework present at baseline; JWT issuing/validation deferred to its own spec |
| Deployment Target | Docker | Dockerfile deferred; not generated during setup |
| Frontend | Not applicable | Backend Only project |

## Source Layout
The INT execution-layer contract (`src/backend/`, `tests/backend/`, `docs/`) is the outer structure.
A standard Maven project lives inside `src/backend/`, so conventional Spring and IDE tooling works unchanged.

| Path | Contents |
|---|---|
| `src/backend/pom.xml` | Maven build definition |
| `src/backend/src/main/java/com/intglobal/etp/` | Application source, base package |
| `src/backend/src/main/resources/application.yml` | Runtime configuration |
| `src/backend/src/test/java/com/intglobal/etp/` | Maven-resolved unit tests |
| `tests/backend/modules/` | Module-level integration suites |
| `tests/backend/config/` | Configuration and bootstrap tests |
| `tests/backend/shared/` | Shared-infrastructure tests |
| `docs/` | Source documents, including the BRD |

**Base Java package:** `com.intglobal.etp`
`com.int.*` cannot be used because `int` is a reserved Java keyword and is illegal in a package name.

## Reviewer Roster
Gate approval rights are verified by **Git email only** (`git config user.email`).
Reviewer *name* is not evaluated. A mismatch blocks review and approval immediately.

| Gate | Reviewer | Email / User ID | Status |
|---|---|---|---|
| Gate 0 (BRD Review) | Vaibhaw Soni | vaibhaw.soni@intglobal.com | Assigned — corrected 2026-09-18 |
| Gate 1 (Spec Peer Review) | Soumyadeep Adhikary | soumyadeep@intglobal.com | Assigned |
| Gate 2 (Code Review) | TBD | TBD | **Unassigned — must be filled before any Gate 2 approval** |

| Role | Person | Email |
|---|---|---|
| Developer / Project Owner | Vaibhaw Soni | vaibhaw.soni@intglobal.com |

> **Gate 0 ownership corrected 2026-09-18.** Project setup recorded Gate 0 as Soumyadeep Adhikary by inference — setup only asked who owned Gate 1 and Gate 2, and Gate 0 was never confirmed. Gate 0 is owned by Vaibhaw Soni. Gate 1 remains with Soumyadeep Adhikary.

> **Author = Gate 0 approver.** Vaibhaw Soni is both spec author and Gate 0 approver, so the `author ≠ reviewer` rule is **waived at Gate 0 only**, recorded explicitly in `.ai-context/pr_reviews/BRD-20260918-200238.md`. It remains in force at Gate 1 and Gate 2, where Soumyadeep Adhikary reviews independently.

## BRD Status
| Field | Value |
|---|---|
| BRD Supplied | Yes — `docs/Requirement for SDD.docx` |
| Baseline | `.ai-context/BRD.md` v1.2 |
| Requirements | BRD-001 to BRD-024 (18 extracted, 6 derived) |
| Ingestion Workflow | `int-brd-ingestion` (complete) |
| Gate 0 Status | **APPROVED** — Vaibhaw Soni, 2026-09-18 20:47:53 |
| Gate 0 Record | `.ai-context/pr_reviews/BRD-20260918-200238.md` |

**Spec generation is UNBLOCKED.** Gate 0 approved under a recorded `author ≠ reviewer` waiver — see the review record.

## Governance Entry Points
| Path | Role |
|---|---|
| `AGENTS.md` | Repository governance policy, authority hierarchy |
| `.agent/rules/` | INT organizational standards — immutable |
| `.agent/workflows/` | INT organizational workflows — immutable |
| `.agents/skills/` | Local project skills, loaded before any global skill |
| `.ai-context/constitution.md` | Project-specific non-negotiable constraints |
| `.ai-context/architecture.md` | Architecture and module boundaries |
| `.ai-context/status.md` | Current lifecycle state board |
