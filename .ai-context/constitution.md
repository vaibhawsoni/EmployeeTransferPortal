# Project Constitution — EmployeeTransferPortal

**Owner:** TBD (Technical Lead unassigned) · **Adopted:** 2026-09-18 · **Version:** v1.2

Governs every feature this repository will ever build. Written once, amended rarely, and amended only through the same review rigour as a spec. Each spec operates inside this document and never restates it.

Standard in force: **INT Engineering Guidelines — Specification-Driven Delivery (SDD) v1.0**

> **Provisional & Open Values Note:** lines marked `[Provisional]` are derived from initial baseline configuration and require TL confirmation. Lines marked `[Open]` require a decision before specs depending on them can proceed.

> **The ingested BRD contains no Constitution section.** `docs/Requirement for SDD.docx` supplies no testing thresholds, no security requirements, no non-functional targets, no compliance obligations and no versioning policy. Per `int-brd-ingestion` governance rule 9, **none have been invented.** Constraints below derive only from the setup discovery gate and from the ingested BRD's actual content. Everything still unknown is marked `TO BE COMPLETED FROM BRD` or `[Open]` and must be settled at Gate 0.

---

## Governance & Roles

| Role | Person | Email | Responsibility |
|---|---|---|---|
| Technical Lead / Architect | **TBD** | **TBD** | Owns this constitution; default Gate 2 code reviewer; technical concurrence at Gate 1 |
| Senior Software Engineer / Spec Author | Vaibhaw Soni | vaibhaw.soni@intglobal.com | Default Spec Author for feature and retro-specs |
| Gate 0 Approver & Project Owner | Vaibhaw Soni | vaibhaw.soni@intglobal.com | Owns the BRD baseline and Gate 0 approval |
| Project Manager / Tech Lead | Soumyadeep Adhikary | soumyadeep@intglobal.com | Gate 1 spec peer reviewer |

The ingested BRD names **no stakeholders and no email addresses**, so no role conflict arose during ingestion and no roster entry was overwritten.

### Core governance rules

- **Author ≠ Reviewer.** The Gate 1 reviewer is never the spec author. Satisfied at Gate 1 and Gate 2: Vaibhaw Soni authors, Soumyadeep Adhikary reviews. **Waived at Gate 0 on 2026-09-18** — Vaibhaw Soni owns Gate 0 and approved a baseline they authored. Logged in `.ai-context/pr_reviews/BRD-20260918-200238.md`.
- **Technical concurrence.** Gate 1 requires recorded TL technical concurrence whenever a spec touches Security Posture or Architectural Constraints. **[Open]** — no TL is assigned, so this cannot currently be recorded.
- **Reviewer split.** Gate 0 = BRD baseline review. Gate 1 = intent, scope and BRD traceability. Gate 2 = technical evidence and code review.
- **Identity enforcement.** Approval requires `git config user.email` to match the roster in `.ai-context/project_context.md`. Reviewer name is not evaluated. **[Open]** — Git is not yet initialised, so verification cannot run.
- **Gate 1 SLA.** Same working day for specs with 5 or fewer acceptance criteria; 48 hours maximum. **[Provisional]** — INT default; not stated in the BRD.

### INT amendments to SDD v1.0

1. **Granular chain:** BRD/SRS → Spec → Plan → Tasks.
2. **First quality gate:** Spec Review (Gate 1) is mandatory before coding begins.
3. **Slugs & identifiers:** mandatory sub-identifiers for specs, plans, tasks, test cases and branches.
4. **Status board:** maintained for every item in `.ai-context/status.md`.
5. **Traceable artifacts:** releases, hotfixes and change requests under `.ai-context/`.

> **Conflict flagged (DC-06).** The BRD states the chain as `Business Requirement → Spec → Gate 1 → …`, omitting Gate 0. `.agent/rules/int-standards.md` §6 mandates a Gate 0 BRD review before any spec drafting. The organisational rule governs; the conflict is flagged rather than silently resolved.

---

## Testing Discipline

- Development is **test-first**. For every task: write the failing test (TDD RED) before writing implementation code (TDD GREEN).
- Test-case *specifications* live in `.ai-context/test_cases/<feature-slug>.test_cases.md`. They are documents, never executable code.
- Executable tests live in `src/backend/src/test/java/` (unit, Maven-resolved) and `tests/backend/` (integration and module suites).
- Every acceptance criterion in a spec must map to at least one executable test, and every test case must cite the AC it verifies.
- The full suite must be green before a spec may enter Gate 2 review. A red suite blocks approval.
- Testing stack: JUnit 5 with `spring-boot-starter-test`; `spring-security-test` for secured endpoints.
- Coverage floors are now set `[Provisional]` under BRD open question OQ-10 — see § Non-Functional Baselines. Mutation-testing and performance-test gates remain **TO BE COMPLETED FROM BRD**.

## Security Posture

- Authentication is **Spring Security with stateless JWT bearer tokens**. No server-side sessions.
- **No secret may be hardcoded** — no credentials, connection strings, keys, or tokens in source, configuration, or test fixtures. All are read from environment variables.
- All inbound payloads are validated and sanitized before processing, using Jakarta Bean Validation on request DTOs.
- Errors are never swallowed. Every caught exception is logged through the project logger with context; `System.out`/`printStackTrace` are prohibited.
- API errors return standard HTTP status codes and a structured JSON error body. Internal exception details, stack traces, and SQL are never returned to a caller.
- Persistence goes through Spring Data JPA or parameterized queries. String-concatenated JPQL/SQL built from user input is prohibited.
- **Data classification: CONFIDENTIAL — HR personal data.** `[Provisional]`, proposed under BRD open question OQ-11, pending Gate 0 ratification:
  - **DP-01** No PII in logs. Log employee identifiers only; names, emails and addresses are masked.
  - **DP-02** The transfer reason is **encrypted at rest**.
  - **DP-03** Read access restricted by role and scope: employee (own), manager (direct reports), HR (all). Anything else returns 403.
  - **DP-04** Every read and write is audited to an **immutable append-only** trail; `UPDATE` and `DELETE` are prohibited on the audit table.
  - **DP-05** Retention: **7 years** after journey completion, then purge.
- **[Open]** The governing jurisdiction and regulation for DP-05 remain unknown and must be confirmed at Gate 0 (BRD consequence C-07).

## Architectural Constraints

- Architecture style is **Modular Monolith (Microservice Ready)** and may not be changed without an ADR.
- Module boundary rules in `.ai-context/architecture.md` §3 are binding and are checked at Gate 2:
  cross-module access only through `api/`; no cross-module repository, entity, or table access; no cross-module joins; `shared/` carries no business logic; `shared/` never depends on `modules/`.
- **No new datastore, message broker, cache, or third-party service may be introduced without an approved ADR** in `.ai-context/decisions/`.
- Technology constraints fixed at setup: Java (target 21 now, 25 intended), Spring Boot 4.x, Maven, PostgreSQL, Hibernate via Spring Data JPA. Changing any of these requires an ADR.
- Base Java package is `com.intglobal.etp`.
- Schema is never generated from entities (`ddl-auto: none`). A migration tool must be chosen by ADR before the first entity ships.
- `.agent/` is immutable. Project-specific standards belong in this file.

### Java / Spring engineering standards

`.agent/rules/int-standards.md` is an INT organizational standard authored for Node.js. It is copied verbatim and must not be edited, so the Java equivalents of its language-specific clauses are defined here:

| `int-standards.md` clause | Java / Spring equivalent for this project |
|---|---|
| ES6+, prefer `const`/`let` | Prefer immutability: `final` fields, constructor injection, records for DTOs. No field injection. |
| `async/await` over raw promises | Keep request handling synchronous unless a spec requires otherwise; if async is needed use `CompletableFuture` or `@Async` with an explicitly configured executor, never an unbounded pool. |
| Wrap async work in `try/catch` | Handle exceptions at a defined boundary — `@ControllerAdvice` for web, explicit handling in service orchestration. Never catch-and-ignore. |
| Log via Winston/Pino, not `console.log` | Log via SLF4J. `System.out.println` and `printStackTrace()` are prohibited. |
| Read secrets from `process.env` | Read secrets from environment variables bound through `@ConfigurationProperties` or `${ENV_VAR}` placeholders. |
| Avoid `eval()` | No runtime code evaluation, no reflective invocation driven by user input, no dynamic class loading from request data. |
| Prevent event-loop blocking | Do not block request threads on long operations; keep transactions short; do not hold a transaction open across a remote call. |
| Optimize queries, use indexes | No N+1 queries — use fetch joins or entity graphs. Index every foreign key and every column used for filtering. Paginate all collection endpoints. |
| Small, single-purpose functions | Controllers stay thin; business logic lives in services; repositories carry no business rules. |
| Guardrails | No comments stating code was AI-generated. No commit messages hinting at AI generation. No AI configuration files outside `.agent/` and `.agents/`. |

### Flagged for human review

1. **Standards-language mismatch.** `.agent/rules/int-standards.md` prescribes Node.js standards while this project is Java / Spring Boot. Resolved here by mapping rather than editing the Control Plane, because the Control Plane is immutable. **Flagged for Gate 1 review** — INT may wish to publish a Java variant of the organizational standard.
2. **Gate 2 reviewer unassigned.** The Gate 2 roster in `.ai-context/project_context.md` is `TBD`. Because approval is verified by Git email against that roster, **no Gate 2 approval can be granted until a reviewer is assigned.** This will block the first feature from reaching release.
3. **BRD supplies no Constitution.** `docs/Requirement for SDD.docx` contains no Constitution, engineering-standards or governance section. Per `int-brd-ingestion` rule 9, no coverage floors, latency targets, availability figures, token lifetimes or compliance rules have been invented. **The reviewer must supply them or explicitly waive them at Gate 0** (BRD open question OQ-10).
4. **Data-protection obligations unstated.** An internal transfer request carries employee personal data — identity, department, location, role, and a free-text reason. The BRD states no classification, masking, retention or audit requirement (BRD open question OQ-11). **This must be ruled on before the first entity ships**, because it constrains the data model, not just the logging policy.
5. **No Technical Lead assigned.** The Governance & Roles table has no TL. Gate 1 technical concurrence cannot be recorded for any spec touching Security Posture or Architectural Constraints, and Gate 2 has no reviewer.

## Non-Functional Baselines

> **`[Provisional]` — every figure below is an INT SDD standard default, not a requirement of the source document.** `docs/Requirement for SDD.docx` states no non-functional target of any kind. These were proposed by the spec author under BRD open question **OQ-10** and require Gate 0 ratification. See `.ai-context/BRD.md` §9.

### p95 API latency `[Provisional]`

| Tier | Target | Scope |
|---|---|---|
| Tier 1 — Critical | **< 300 ms** | Authentication, transfer-request submission, status read |
| Tier 2 — Standard CRUD | **< 500 ms** | Reference-data reads, manager and HR decision endpoints |
| Tier 3 — Heavy | **< 2 s** | Bulk operations, audit-trail queries |

### Availability & recovery `[Provisional]`

| Metric | Target |
|---|---|
| Availability | **99.5%** |
| RPO | **15 minutes** |
| RTO — critical | **2 hours** |
| RTO — standard | **4 hours** |

### Test coverage floors `[Provisional]`

Measured on **changed files only**. Enforced at Gate 2.

| Tier | Floor | Applies to |
|---|---|---|
| Critical | **80%** | `approval`, authentication, `audit`, **`employee-directory`** |
| Business | **70%** | `transfer-request`, `orchestration` |
| Utility | **60%** | `notification`, shared helpers |

> **`employee-directory` raised from Utility to Critical on 2026-09-19** (BRD consequence C-09). It stores the authority set behind every authorization decision, and the AMD-01 ruling gave it a state-changing write on HR data.

> **[Open]** — no coverage tool is configured. JaCoCo must be added to `src/backend/pom.xml` with per-tier thresholds before these floors are enforceable (BRD consequence C-06).

### Operational baselines

- Configuration is environment-driven; the same artifact runs in every environment with no code change.
- Runtime startup requires a configured PostgreSQL instance; credentials come from environment variables only.

## Versioning Rules

- Releases follow semantic versioning `vMAJOR.MINOR.PATCH` and are recorded in `.ai-context/releases/RELEASE-vX.Y.Z.md`.
- A spec reaches `Released (vX.Y.Z)` only after Gate 2 approval and a green suite.
- Hotfixes increment PATCH and require a post-hoc Gate 2 review.
- API versioning strategy, branching model, and release cadence: **TO BE COMPLETED FROM BRD**.

---

## Amendment Log

| Version | Date | Change |
|---|---|---|
| v1.0 | 2026-09-18 | Established at project setup from the discovery gate. No BRD available. |
| v1.1 | 2026-09-18 | BRD ingested. Added Governance & Roles. Recorded that the source supplies no Constitution and no NFRs; invented none. |
| v1.2 | 2026-09-18 | Author-proposed resolutions to BRD open questions OQ-10 and OQ-11 filled the Non-Functional Baselines and data-protection rules, all marked `[Provisional]`. **Pending Gate 0 ratification** — see `.ai-context/BRD.md` §9. |

> **Standing caveat.** Every `[Provisional]` value in this document originates from INT defaults or author judgement, not from `docs/Requirement for SDD.docx`, which states no non-functional, security or compliance requirement whatsoever. Ratification at Gate 0 converts them to binding constraints; rejection replaces them.
