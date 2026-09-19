# Architecture — EmployeeTransferPortal

## 1. Architecture Style

**Modular Monolith (Microservice Ready)** — the INT default for server-side systems.

- The application builds and deploys as a **single Spring Boot JAR**.
- Business capabilities live as **bounded modules** under `modules/`, each owning its controllers, services, repositories, domain model and DTOs.
- Shared infrastructure is isolated under `shared/` and contains **no business logic**.
- Each module is structured so it can be lifted into an independently deployable service later without rewriting its internals.
- Microservice deployment complexity (service discovery, distributed tracing, inter-service contracts) is **not** introduced at this stage.

## 2. Package Structure

Base package: `com.intglobal.etp`
(`com.int.*` is illegal — `int` is a reserved Java keyword.)

```text
src/backend/src/main/java/com/intglobal/etp/
├── EmployeeTransferPortalApplication.java   # single @SpringBootApplication entry point
├── app/                                     # application-wide wiring, no business logic
│   ├── config/                              # @Configuration beans, property binding
│   ├── security/                            # Spring Security filter chain, JWT wiring
│   └── web/                                 # global exception handlers, response envelopes, filters
├── modules/                                 # business modules — EMPTY until BRD Gate 0 approval
└── shared/                                  # cross-cutting infrastructure, no business logic
    ├── persistence/                         # base entities, auditing, converters
    ├── logging/                             # structured logging support
    ├── errors/                              # shared exception types and error codes
    └── utils/                               # stateless helpers
```

### Per-module internal shape (created only after Gate 0)

```text
modules/<module-name>/
├── api/            # published interface other modules may depend on
├── web/            # REST controllers, request/response DTOs
├── domain/         # entities, value objects, domain services
├── repository/     # Spring Data JPA repositories
├── service/        # application services, orchestration, transactions
└── config/         # module-local configuration
```

## 3. Module Boundary Rules

These rules are what make future extraction possible. They are enforced at Gate 2 code review.

1. **Published interfaces only.** A module may call another module exclusively through types under that module's `api/` package.
2. **No cross-module persistence access.** A module must never reference another module's repositories, entities, or tables directly. Cross-module data is exchanged as DTOs.
3. **No cross-module database joins.** Joining across module-owned tables couples their schemas and blocks extraction. Compose in the service layer instead.
4. **Shared is infrastructure only.** `shared/` holds technical concerns. Placing business rules there creates a hidden coupling point and is prohibited.
5. **Dependency direction.** `modules/ -> shared/` is allowed. `shared/ -> modules/` is forbidden. `app/` may depend on both; neither may depend on `app/`.
6. **Own your transaction.** A transaction must not span two modules' write paths; coordinate through the calling service instead.

### Extraction path to a service

A module is extractable when rules 1-6 hold. Extraction then becomes: replace the in-process `api/` implementation with a remote client, split the module's tables into their own schema, and deploy the module as its own Spring Boot application. No change to the module's internal code is required.

## 4. Data Architecture

| Aspect | Decision |
|---|---|
| Database | PostgreSQL |
| ORM | Hibernate via Spring Data JPA |
| Schema ownership | Each module owns its own tables |
| `ddl-auto` | `none` — the schema is never generated from entities |
| Credentials | Environment variables only; never committed |
| `open-in-view` | Disabled, to keep persistence sessions inside the service layer |

## 5. Security Architecture

| Aspect | Decision |
|---|---|
| Framework | Spring Security |
| Token model | JWT bearer tokens, stateless |
| Session policy | Stateless — no server-side session |
| Secret management | Environment variables only |
| Status | Spring Security is on the classpath at baseline. **Token issuing, validation, and the authorization model are deferred to their own spec.** |

## 6. Deployment Architecture

| Aspect | Decision |
|---|---|
| Target | Docker |
| Artifact | Executable Spring Boot JAR |
| Status | **Dockerfile and compose definitions are deferred** and were not generated during setup. |

## 7. Open Architectural Items

Each item below needs a decision before the relevant work begins. Items marked *ADR required* must be recorded in `.ai-context/decisions/`.

| # | Item | Status | Notes |
|---|---|---|---|
| 1 | Database migration tooling (Flyway vs Liquibase) | **Open — ADR required** | `ddl-auto: none` means a migration tool is needed before the first entity ships. Not chosen during setup because nothing in the BRD backs a choice yet. |
| 2 | JWT implementation (library, claims, expiry, refresh, key rotation) | **Open — spec required** | Strategy is fixed; the implementation is a feature. |
| 3 | Dockerfile and local compose stack | **Open** | Deployment target is Docker; artifacts deferred. |
| 4 | Java 21 to Java 25 upgrade | **Open** | Build targets 21 to match the installed JDK 21.0.10. Bump `<java.version>` once JDK 25 is installed. |
| 5 | Module decomposition | Proposed, 6 modules | `architecture.md` §9.7. Boundary settled by the AMD-01 ruling. Awaiting Gate 1. |
| 6 | Observability (logging format, metrics, tracing) | **Open** | `shared/logging/` is a placeholder until a decision is made. |
| 7 | API versioning strategy | **Open** | Decide before the first public endpoint. |

## 8. Relationship to the INT Control Plane

`.agent/rules/int-standards.md` is an organizational standard written for Node.js. It is copied verbatim and must not be edited. Its **language-agnostic** clauses (error handling discipline, secret management, input validation, PR gate governance, guardrails) apply in full. Its **Node-specific** clauses (ES6 syntax, `async/await`, `process.env`, event-loop guidance) have Java equivalents defined in `.ai-context/constitution.md`.

---

## 9. Proposed Business Domains (BRD-derived — PROPOSAL, NOT APPROVED)

Derived from `.ai-context/BRD.md` v1.1 on 2026-09-18, per `int-brd-ingestion` Step 2. Revised after the 14 open questions received author-proposed resolutions.

> **NOT APPROVED.** Per `int-brd-ingestion` Step 3, no business module folder may be created until **Gate 1 architecture approval**. `src/backend/src/main/java/com/intglobal/etp/modules/` remains empty. This proposal is additionally contingent on the BRD passing **Gate 0** first.

### 9.1 Functional boundaries

Related requirements are grouped into coherent business modules. Requirements are deliberately **not** mapped one-to-one onto modules.

| Candidate module | Responsibility | Requirements |
|---|---|---|
| `transfer-request` | Owns the transfer request aggregate: capture, validation, lifecycle state, and status/progress queries. The system of record for the journey. | BRD-001 to BRD-010 |
| `approval` | Manager confirmation and HR eligibility validation. Owns approval decisions and their outcomes. | BRD-012, BRD-013 |
| `orchestration` | Coordinates downstream activities once a request is approved, tracks each activity's state, and aggregates progress back to `transfer-request`. | BRD-011, BRD-014 to BRD-017 |
| `notification` | Employee confirmation on completion and stakeholder pending-action alerts. | BRD-018 |
| `employee-directory` | Read access to employee, department, location and role reference data used for selection and validation. | Supports BRD-002 to BRD-004 |

### 9.2 Dependency direction

```text
            transfer-request  (aggregate root of the journey)
                   |
        +----------+----------+
        |                     |
    approval            orchestration
        |                     |
        +----------+----------+
                   |
        employee-directory   notification
```

- `transfer-request` depends on `employee-directory` for validation and on `approval`/`orchestration` through their published `api/` interfaces.
- `orchestration` depends on `notification` to inform the employee.
- **No module depends back on `transfer-request`.** Downstream modules report outcomes through events or return values, never by reaching into the request aggregate.
- No circular dependency is permitted between business modules.

### 9.3 Database boundaries

Each module owns its own tables. No cross-module joins; composition happens in the service layer (see §3 rules 2 and 3).

| Module | Owns |
|---|---|
| `transfer-request` | The transfer request and its status history |
| `approval` | Approval decisions and eligibility check outcomes |
| `orchestration` | Downstream activity records and their individual states |
| `notification` | Notification dispatch records |
| `employee-directory` | Reference data, or a projection of it — ownership depends on OQ-05 |

### 9.4 API boundaries

Employee-facing operations (BRD-001 to BRD-010) are exposed by `transfer-request`. The One-Point Portal is the external consumer. Whether `approval` and `orchestration` expose their own external APIs depends on **OQ-03** — if those stakeholders act through interfaces, they need endpoints; if they are system integrations, they need adapters instead.

### 9.5 Candidate future service boundaries

`orchestration` and `notification` are the strongest extraction candidates: both are asynchronous, both are I/O-bound against external systems, and neither owns the journey's system of record. `transfer-request` should remain in-process as the aggregate root.

### 9.6 Open questions — resolved (pending Gate 0)

All five questions that blocked this proposal now carry author-proposed resolutions in `.ai-context/BRD.md` §9. The structure below is therefore **proposed-complete**, contingent on Gate 0 ratification.

| Question | Resolution | Effect on this proposal |
|---|---|---|
| **OQ-03** — humans vs systems | Manager and HR are humans via inbound REST; org data, payroll, IT and facilities are stubbed outbound ports | `approval` gains two decision endpoints. `orchestration` owns four outbound adapter ports. Both shapes are now fixed. |
| **OQ-04** — execution order | Org-data update is a hard gate; payroll, IT and facilities then run in parallel | `orchestration` is a **gated fan-out coordinator**, not a plain sequencer. Requires asynchronous execution. |
| **OQ-02** — status model | 11-state lifecycle plus `PARTIALLY_COMPLETED` | `transfer-request` owns the state machine; all other modules react to it. |
| **OQ-05** — master data | Owned locally as seeded reference data | `employee-directory` is confirmed a **real module** with its own tables, not an anti-corruption layer. It also supplies the manager hierarchy used by authorization (OQ-14). |
| **OQ-13** — failure behaviour | Bounded retry with backoff, then park for manual action. **No compensation.** | `orchestration` needs a retry scheduler and per-step state, but **no saga or compensator**. Materially smaller than a compensating design. |

### 9.7 Revised module set

The resolutions add one component to the original five.

| Module | Change from the original proposal |
|---|---|
| `transfer-request` | Unchanged in scope; now also owns withdrawal (BRD-019) and read-authorization scoping (BRD-023) |
| `approval` | Now exposes two inbound decision endpoints and implements the five hardcoded eligibility rules ELIG-01 to ELIG-05 |
| `orchestration` | Now a gated fan-out coordinator with per-step state and bounded retry (BRD-024) |
| `notification` | Confirmed in-portal only; owns a `notifications` table and two endpoints. No email adapter. |
| `employee-directory` | Confirmed as a real module owning departments, locations, roles and employees, including `manager_id` |
| **`audit`** *(new)* | Immutable append-only trail (BRD-022). Arises from OQ-11 and OQ-12. **Placement undecided** — a business module, or cross-cutting infrastructure under `shared/`. Resolve at Gate 1. |

### 9.8 Architectural decisions now required

| Candidate ADR | Status | Trigger |
|---|---|---|
| **Async execution & retry mechanism** | **Required** | OQ-04 parallel fan-out plus OQ-13 bounded retry. Spring `@Async` with a bounded executor and a scheduled retry table, versus a message broker. A broker is a new datastore-class dependency and needs an ADR per §3. (Consequence C-04) |
| **Database migration tooling** | **Required** | Pre-existing open item §7 #1. Now urgent — the resolutions define concrete tables. |
| **Encryption at rest for the transfer reason** | **Required** | OQ-11 DP-02. JPA attribute converter versus database-level encryption. Affects the data model. (Consequence C-05) |
| **Audit module placement** | **Required** | Whether `audit` is a business module or shared infrastructure. |
| Saga / compensation pattern | **Not required** | OQ-13 explicitly rules out compensation. |

> **Boundary risk to watch at Gate 2.** `audit` is written to by every module, and `employee-directory` is read by `transfer-request`, `approval` and the authorization layer. Both are natural places for the §3 boundary rules to erode. Access must go through published `api/` interfaces, never direct repository access.

### 9.9 AMD-01 — module boundary RESOLVED (Option A, 2026-09-19)

Ruled by Vaibhaw Soni at Gate 0. `.ai-context/BRD.md` § Post-Approval Amendments.

`employee-directory` publishes **one** narrow write operation. OQ-05 local ownership is confirmed; OQ-03 is narrowed so organisational-data update is an in-process call rather than a stubbed external adapter. Payroll, IT and Facilities remain stubbed outbound ports.

| Affected | Outcome |
|---|---|
| `employee-directory` | Gains `applyTransfer(ApplyTransferCommand)`, an `applied_transfers` idempotency table, and 4 new acceptance criteria. Coverage tier raised to **Critical 80%**. |
| `orchestration` | Unblocked. Its OQ-04 hard-gate step calls `EmployeeDirectoryApi.applyTransfer` instead of a stub, so the gate now genuinely succeeds or fails. |
| `approval` | ELIG-03 becomes functional — `last_transfer_completed_at` acquires a writer. |
| §3 boundary rules | **Intact.** The write goes through the published `api/` interface (rule 1). `orchestration` never touches these tables (rule 2). `transferRequestId` is an opaque key with no foreign key back to `transfer-request` (rule 3). |

### 9.10 Revised dependency direction

```text
            transfer-request
                   |
        +----------+----------+
        |                     |
    approval            orchestration
        |                     |
        |          +----------+----------+
        |          |                     |
        +----> employee-directory   notification
                   |
              audit-trail
```

The AMD-01 ruling adds **`orchestration → employee-directory`**. It introduces no cycle: `employee-directory` depends on nothing, so it remains a sink in the graph. Three modules now depend on it (`transfer-request`, `approval`, `orchestration`), which makes its published interface the most load-bearing contract in the system and justifies the raised coverage tier.

### 9.11 AMD-02 — effective-date timing RESOLVED (Option A, 2026-09-19)

Ruled by Vaibhaw Soni at Gate 0. The org-data change is **applied immediately** when `applyTransfer` is invoked, roughly 30+ days before the OQ-06 effective date.

This keeps the OQ-04 hard gate meaningful: Payroll, IT and Facilities fan out reading the **new** department and location, which is the data they need to prepare the move. Scheduling the change for the effective date (Option B) was rejected precisely because it would have left the fan-out acting on stale data.

**Accepted consequence.** Between approval and the effective date the directory reports post-transfer values. Pinned by acceptance criterion `employee-directory.AC24` so it cannot be filed as a defect at Gate 2.

**Consequence C-12 — carried to `approval`.** `role_started_at` and `last_transfer_completed_at` hold future dates during that window. ELIG-03 must be expressed as an upper-bound comparison (`<= today - 12 months`), never a between-range, or it fails open and permits a second transfer while one is in flight. This is a correctness requirement with its own acceptance criterion in the `approval` spec.

### 9.12 Amendment status

| ID | Subject | Outcome |
|---|---|---|
| AMD-01 | Org-data write path | ✅ Resolved 2026-09-19 — Option A, published `applyTransfer` |
| AMD-02 | Effective-date timing | ✅ Resolved 2026-09-19 — Option A, apply immediately |

No open BRD amendments. The `employee-directory` module boundary and behaviour are final for Gate 1.
