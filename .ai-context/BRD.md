# Business Requirements Document — EmployeeTransferPortal

## Document Status

| Field | Value |
|---|---|
| Status | **PENDING** (Gate 0 Review Required) · ✅ AMD-01 and AMD-02 both resolved · **no open amendments** |
| Version | 1.5 |
| Ingested | 2026-09-18 · Open questions resolved 2026-09-18 |
| Source Document | `docs/Requirement for SDD.docx` |
| Source Title | "SDD Developer Assessment — Employee Internal Transfer Digital Journey" |
| Gate 0 Approver | TBD |
| Gate 0 Review Record | None |
| Author | Vaibhaw Soni (vaibhaw.soni@intglobal.com) |

> **Gate 0 PENDING — spec generation is BLOCKED.** Waiting for an independent reviewer (Technical Lead or Manager) to approve this baseline.
>
> ✅ **All amendments closed 2026-09-19.** **AMD-01** — Option A: `employee-directory` publishes a narrow `applyTransfer` write operation; OQ-05 confirmed, OQ-03 narrowed; BRD-014 implementable, ELIG-03 functional. **AMD-02** — Option A: the org-data change applies immediately, so the directory shows post-transfer values for ~30 days by design. Constraint **C-12** carries forward to the `approval` spec. See § Post-Approval Amendments.

### Nature of the source document

`docs/Requirement for SDD.docx` is an **assessment brief**, not a conventional client BRD. It contains a genuine business requirement (§2 Business Context, §3 Business Requirement) wrapped in assessment scaffolding (deliverables, evaluation weights, a 10-day timeline, milestones).

The functional baseline below is drawn **only** from §2 and §3. The assessment scaffolding is preserved separately in [§10 Engagement Context](#10-engagement-context) because it governs process expectations, not system behaviour. Nothing has been added that the source document does not support.

---

## 1. Objective

Provide a **single digital journey** for internal employee transfers through the organisation's One-Point Employee Portal.

Today an employee requesting an internal transfer must interact with multiple teams and systems separately — manager, HR, organisational data, payroll, IT and facilities. The organisation wants the portal to **orchestrate those downstream activities** and give the employee **one consolidated view of progress**.

*Source: document §1 Objective, §2 Business Context, §3 Business Requirement.*

---

## 2. Scope

### In Scope

- Capture and submission of an Internal Transfer Request by an employee.
- Orchestration of the downstream activities triggered by that request.
- Status and pending-action visibility for the requesting employee.
- Confirmation to the employee on completion.

### Scope boundary confirmed with the project owner

This repository is a **Backend Only** project. The requirement baseline is therefore scoped to **REST APIs and orchestration**. The One-Point Employee Portal UI is treated as an **external consumer** of those APIs and is listed under [§8 Out of Scope](#8-out-of-scope).

Every acceptance criterion derived from this baseline must be expressible and testable at the API level.

---

## 3. Actors

| Actor | Role in the journey | Source |
|---|---|---|
| **Employee** | Initiates the transfer request, supplies the proposed department, location, role, effective date and optional reason; tracks status; receives confirmation | §2, §3 |
| **Manager** | Discusses the transfer with the employee and confirms it | §2 |
| **HR** | Validates the employee's eligibility for transfer | §2 |
| **Payroll** | May need to be updated as a result of the transfer | §2 |
| **IT** | May need to provision or remove access | §2 |
| **Facilities** | May need to arrange the employee's new location | §2 |
| **One-Point Employee Portal** | The channel through which the employee accesses the journey; external consumer of this system's APIs | §2, §3 |

> The document does not state whether Manager, HR, Payroll, IT and Facilities are **human actors acting through an interface** or **system integrations**. See [OQ-03](#9-open-questions).

---

## 4. Functional Requirements

Requirement IDs are stable and must never be renumbered. Where the source document uses conditional language ("may need to"), that conditionality is preserved verbatim rather than resolved.

### 4.1 Request Initiation & Capture

| ID | Requirement | Source | Notes |
|---|---|---|---|
| BRD-001 | An employee shall be able to initiate an Internal Transfer Request from the One-Point Employee Portal. | §3 | — |
| BRD-002 | The employee shall be able to select the proposed new department / business unit. | §3 | Source of valid values not stated — see [OQ-05](#9-open-questions) |
| BRD-003 | The employee shall be able to select the proposed new location. | §3 | Source of valid values not stated — see [OQ-05](#9-open-questions) |
| BRD-004 | The employee shall be able to select the proposed role / job position. | §3 | Source of valid values not stated — see [OQ-05](#9-open-questions) |
| BRD-005 | The employee shall be able to provide an effective date for the transfer. | §3 | No validity constraints stated — see [OQ-06](#9-open-questions) |
| BRD-006 | The employee shall be able to provide an **optional** reason for the transfer. | §3 | Explicitly optional in the source |
| BRD-007 | The employee shall be able to submit the request. | §3 | — |

### 4.2 Visibility & Tracking

| ID | Requirement | Source | Notes |
|---|---|---|---|
| BRD-008 | The employee shall be able to view the current status of the request. | §3 | Status values not enumerated — see [OQ-02](#9-open-questions) |
| BRD-009 | The employee shall be able to view actions that are pending with other stakeholders. | §3 | — |
| BRD-010 | The system shall provide the employee with a single consolidated view of progress across all downstream activities. | §3 | — |

### 4.3 Orchestration of Downstream Activities

| ID | Requirement | Source | Notes |
|---|---|---|---|
| BRD-011 | The system shall orchestrate the downstream activities arising from a submitted transfer request. | §3 | Central requirement of the journey |
| BRD-012 | The journey shall include **manager confirmation** of the transfer. | §2 | — |
| BRD-013 | The journey shall include **HR validation of the employee's eligibility**. | §2 | Eligibility rules not supplied — see [OQ-01](#9-open-questions) |
| BRD-014 | The journey shall include **updating the employee's organisational information**. | §2 | ✅ Implementable via `EmployeeDirectoryApi.applyTransfer` (AMD-01 Option A) |
| BRD-015 | The journey shall include **updating Payroll**, where required. | §2 | Conditional in source: "Payroll *may* need to be updated" |
| BRD-016 | The journey shall include **provisioning or removing IT access**, where required. | §2 | Conditional in source: "IT *may* need to provision or remove access" |
| BRD-017 | The journey shall include **arranging the employee's new location via Facilities**, where required. | §2 | Conditional in source: "Facilities *may* need to arrange..." |
| BRD-018 | The employee shall receive **confirmation** on completion of the journey. | §2 | Delivery channel not stated — see [OQ-07](#9-open-questions) |

---

## 5. Non-Functional Requirements

**The source document specifies no non-functional requirements.** It states no latency, throughput, availability, concurrency, data-volume, retention, recovery or compliance targets.

None were invented during ingestion. **Targets have since been proposed by the spec author under OQ-10** — INT tiered defaults, recorded `[Provisional]`, pending Gate 0 ratification. They are not derived from this document.

The reviewer must ratify, amend or reject those proposed targets at Gate 0.

---

## 6. Business Rules

**The source document states very few business rules explicitly.** Only the following are directly supported:

| ID | Rule | Source |
|---|---|---|
| BR-001 | The transfer reason is **optional**; department, location, role and effective date are supplied by the employee. | §3 (explicit) |
| BR-002 | Payroll, IT and Facilities activities are **conditional** — they occur only where required, not on every transfer. | §2 ("may need to") |
| BR-003 | Manager confirmation and HR eligibility validation are both required parts of the journey. | §2 |

### Deliberately not asserted as rules

§2 presents the current-state journey as an illustrative sequence, introduced with **"For example:"**. Its ordering is therefore **not** treated as a normative process rule. Whether manager confirmation must precede HR validation, whether downstream activities run sequentially or in parallel, and what happens when a step is rejected are all **unresolved** — see [OQ-02](#9-open-questions), [OQ-04](#9-open-questions) and [OQ-08](#9-open-questions).

Inferring a state machine from an illustrative example would fabricate business logic the organisation has not agreed.

---

## 7. Assumptions

Assumptions are working positions adopted so that architecture and specification can proceed. Each must be confirmed or corrected at Gate 0.

| ID | Assumption | Basis | Status |
|---|---|---|---|
| A-01 | The deliverable is a backend system exposing REST APIs; the One-Point Portal UI is an external consumer. | Confirmed with the project owner during ingestion | Confirmed |
| A-02 | Downstream systems (HR, Payroll, IT, Facilities, organisational data) are reached through **stubbed adapter interfaces behind defined ports**. No live external system is available or integrated. | Working assumption; nothing in the document describes integration mechanics | **Confirmed by OQ-03** — Manager and HR are humans via REST; the four downstream activities are stubbed ports |
| A-03 | One transfer request concerns one employee, and the requesting employee is the subject of the transfer. | Consistent throughout §2–§3; never stated explicitly | **Confirmed at Gate 0** |
| A-04 | Employee identity is established by the calling portal; this system authenticates requests via Spring Security + JWT. | Project configuration (`.ai-context/project_context.md`), **not** the BRD | Project-derived |
| A-05 | Employee, department, location and role reference data originates outside this system. | Implied by "select" in §3; no master-data ownership stated | **Superseded by OQ-05** — reference data is owned locally by `employee-directory`, not external |

---

## 8. Out of Scope

| Item | Reason |
|---|---|
| The One-Point Employee Portal front-end (screens, components, client-side routing) | Project is Backend Only; the portal is an external consumer (A-01) |
| Building or replacing the HR, Payroll, IT, Facilities or organisational master systems | The document positions these as existing organisation functions |
| Other employee services offered by the portal (HR, payroll, IT, learning, facilities as standalone journeys) | §2 lists them as portal context; only the **internal transfer** journey is required |
| External transfers, promotions, secondments, resignations, or any non-transfer HR event | Not mentioned anywhere in the document |
| Migration of historical transfer records | Not mentioned |
| Reporting, analytics or dashboards | Not mentioned |

---

## 9. Open Questions — Ratified Resolutions

All 14 questions were answered by the spec author (Vaibhaw Soni) on 2026-09-18 and **ratified at Gate 0** on 2026-09-18 20:47:53.

> **RATIFIED at Gate 0** on 2026-09-18 by Vaibhaw Soni. These resolutions are now binding on all downstream specs.
>
> ⚠ **Approved by their own author.** `author ≠ reviewer` was waived at Gate 0, so nothing below received independent review before becoming binding. The waiver is recorded in `.ai-context/pr_reviews/BRD-20260918-200238.md`. Independent scrutiny resumes at Gate 1.
>
> **Provenance:** none of these resolutions comes from the source document. `docs/Requirement for SDD.docx` is silent on all 14 points. They are author-proposed working decisions adopted so that specification can proceed, and every invented threshold is marked as such below. The source document explicitly asks for exactly this (§1: "Identify ambiguity and missing decisions"; §5: "Ambiguity & discovery", 15%).

| ID | Type | Question | Resolution | Status |
|---|---|---|---|---|
| OQ-01 | B | HR eligibility rules | **Standard rule set, hardcoded** in the `approval` module | **Ratified** |
| OQ-02 | B | Authoritative status model | **Full 11-state lifecycle** with rejection and withdrawal paths | **Ratified** |
| OQ-03 | T | Human actors vs system integrations | **Mixed** — Manager and HR are humans via REST; the rest are stubbed outbound ports | **Ratified** |
| OQ-04 | B | Sequential vs parallel downstream | **Org update is a hard gate**, then Payroll / IT / Facilities fan out in parallel | **Ratified** |
| OQ-05 | B/T | Master-data ownership | **Owned locally** by `employee-directory` as seeded reference data | **Ratified** |
| OQ-06 | B | Effective-date constraints | **Minimum 30 days notice, maximum 12 months horizon** | **Ratified** |
| OQ-07 | B/T | Confirmation channel | **In-portal notification only**; no email | **Ratified** |
| OQ-08 | B | Rejection and withdrawal | **Withdraw until APPROVED**; rejection terminal with mandatory reason | **Ratified** |
| OQ-09 | B | Concurrent and editable requests | **One open request per employee**; immutable once submitted | **Ratified** |
| OQ-10 | B | Non-functional targets | **INT tiered defaults**, recorded `[Provisional]` | **Ratified** |
| OQ-11 | B/T | Data protection | **Confidential HR data** — masking, encryption at rest, full audit, 7-year retention | **Ratified** |
| OQ-12 | B | Audit trail | **Immutable append-only trail**, role-scoped reads | **Ratified** |
| OQ-13 | T | Failure and timeout behaviour | **Bounded retry with backoff, then park** for manual action. No compensation. | **Ratified** |
| OQ-14 | B | Read authorization | **Employee (own), Manager (direct reports), HR (all)** | **Ratified** |

---

### OQ-01 — HR eligibility rules *(Business)*

A transfer is eligible when **all** of the following hold:

| Rule | Condition |
|---|---|
| ELIG-01 | Employee has held their current role for **≥ 12 months** |
| ELIG-02 | Employee has **no active disciplinary action** |
| ELIG-03 | Employee has **no completed transfer in the last 12 months** |
| ELIG-04 | Employee's performance rating is **≥ "Meets Expectations"** |
| ELIG-05 | Employee is **not serving a notice period** |

Rules are implemented in the `approval` module. A rule change requires a code change and a release.

> **Invented thresholds:** 12 months tenure, 12 months transfer recency, and the "Meets Expectations" bar have no basis in the source document. HR must ratify or correct them at Gate 0.
>
> **Data dependency:** ELIG-02, ELIG-04 and ELIG-05 require disciplinary status, performance rating and notice-period fields that OQ-05 does not currently place in `employee-directory`. See [Consequences](#consequences-arising-from-these-resolutions) C-01.

---

### OQ-02 — Authoritative status model *(Business)*

```text
DRAFT
  └→ SUBMITTED
       ├→ WITHDRAWN                    (terminal, employee-initiated)
       └→ PENDING_MANAGER
            ├→ REJECTED_MANAGER        (terminal, reason required)
            └→ PENDING_HR
                 ├→ INELIGIBLE         (terminal, reason required)
                 └→ APPROVED
                      └→ IN_PROGRESS
                           ├→ COMPLETED             (terminal)
                           ├→ PARTIALLY_COMPLETED   (terminal — see OQ-13)
                           └→ FAILED                (terminal)
```

Transitions not shown are illegal and must be rejected. The transition set is the authority for BRD-008 and BRD-010.

> **`PARTIALLY_COMPLETED` was not in the model as originally chosen.** It is required by the OQ-13 resolution, where one parallel step exhausts its retries while the others succeed. Added for consistency and flagged for Gate 0 — see [Consequences](#consequences-arising-from-these-resolutions) C-02.

---

### OQ-03 — Human actors vs system integrations *(Technical)*

| Stakeholder | Mechanism |
|---|---|
| Manager | **Human** — inbound REST decision endpoint |
| HR | **Human** — inbound REST decision endpoint |
| Organisational data | **System** — outbound `OrgDataPort` (stubbed) |
| Payroll | **System** — outbound `PayrollPort` (stubbed) |
| IT access | **System** — outbound `ItAccessPort` (stubbed) |
| Facilities | **System** — outbound `FacilitiesPort` (stubbed) |

All four outbound ports are **stubbed adapters**. No live external system is integrated. This **confirms assumption A-02**.

> ✅ **Amended by the AMD-01 ruling (Option A).** `OrgDataPort` is **narrowed**: organisational-data update is an **in-process call** into `employee-directory` via its published interface, not a stubbed external adapter. Payroll, IT and Facilities remain stubbed outbound ports, unchanged.

---

### OQ-04 — Downstream execution order *(Business)*

```text
APPROVED
   │
   ▼
OrgDataPort.update()          ← hard gate; must succeed
   │
   ├──→ PayrollPort       ┐
   ├──→ ItAccessPort      │  parallel
   └──→ FacilitiesPort    ┘
   │
   ▼  (all settled)
COMPLETED / PARTIALLY_COMPLETED
```

The gate is justified: payroll, IT and facilities all act on the employee's **new** department and location, so the organisational record must be updated first.

Conditional steps (BR-002) that do not apply are marked `NOT_APPLICABLE` and skipped without blocking completion.

---

### OQ-05 — Master-data ownership *(Business / Technical)*

`employee-directory` owns, as seeded reference data:

| Table | Fields |
|---|---|
| `departments` | id, name, active |
| `locations` | id, name, active |
| `roles` | id, title, active |
| `employees` | id, department, location, role, manager_id, joined_at |

**Validation:** a selection must reference an `active` row, and the proposed department, location or role must differ from the employee's current value.

This **supersedes assumption A-05**, which had assumed the data originated outside the system. `employee-directory` is confirmed as a real module, not an anti-corruption layer.

> ✅ **Confirmed by the AMD-01 ruling (Option A).** Local ownership stands. `employee-directory` additionally publishes a narrow `applyTransfer` write operation so BRD-014 has a legitimate call target.

> The `employees` table must be extended to carry the fields ELIG-02, ELIG-04 and ELIG-05 depend on. See [Consequences](#consequences-arising-from-these-resolutions) C-01.

---

### OQ-06 — Effective-date constraints *(Business)*

| Rule | Condition |
|---|---|
| DATE-01 | Effective date **≥ today + 30 days** (notice period) |
| DATE-02 | Effective date **≤ today + 365 days** (planning horizon) |

Violations are rejected with a field-level validation error at submission.

> **Invented thresholds:** 30 days and 365 days have no basis in the source document.
>
> No working-day or payroll-cut-off alignment is required.

---

### OQ-07 — Confirmation channel *(Business / Technical)*

The `notification` module persists in-portal notifications, which the One-Point Portal reads over the API. **No email is sent** and no external notification service is integrated.

| Concern | Decision |
|---|---|
| Store | `notifications` (id, employee_id, type, payload, read_at, created_at) |
| API | `GET /notifications`, `POST /notifications/{id}/read` |
| Raised on | `COMPLETED`, `PARTIALLY_COMPLETED`, `FAILED`, `REJECTED_MANAGER`, `INELIGIBLE`, and each step becoming pending |

---

### OQ-08 — Rejection and withdrawal *(Business)*

| Rule | Condition |
|---|---|
| WD-01 | Withdrawal permitted while status is `SUBMITTED`, `PENDING_MANAGER` or `PENDING_HR` |
| WD-02 | Withdrawal **blocked** once status is `APPROVED` or later |
| WD-03 | `REJECTED_MANAGER` is terminal; a reason is **mandatory** |
| WD-04 | `INELIGIBLE` is terminal; a reason is **mandatory** |
| WD-05 | `WITHDRAWN` is terminal; a reason is optional |
| WD-06 | A new request may be raised after any terminal state |

There is no recourse once downstream activity has begun — by design, because no compensation logic exists (OQ-13).

---

### OQ-09 — Concurrent and editable requests *(Business)*

| Rule | Condition |
|---|---|
| REQ-01 | An employee may hold **at most one** request in a non-terminal state. A second submission returns **409 Conflict**. |
| REQ-02 | Once `SUBMITTED`, a request is **immutable**. To amend it, the employee withdraws and raises a new one. |

> **Consistency point for Gate 0.** The OQ-02 model includes a `DRAFT` state, but this resolution makes requests immutable after submission without stating whether `DRAFT` is persisted and editable. Author's reading: **`DRAFT` is a transient client-side state and is not persisted** — requests are created directly in `SUBMITTED`. `DRAFT` is retained in the model only as the logical pre-submission state. Flagged for ratification — see [Consequences](#consequences-arising-from-these-resolutions) C-03.

---

### OQ-10 — Non-functional targets *(Business)*

All values below are `[Provisional]`: they are INT SDD standard defaults, **not** derived from the source document, and require TL confirmation.

**p95 API latency**

| Tier | Target | Scope |
|---|---|---|
| Tier 1 — Critical | **< 300 ms** | Authentication, request submission, status read |
| Tier 2 — Standard CRUD | **< 500 ms** | Reference-data reads, decision endpoints |
| Tier 3 — Heavy | **< 2 s** | Bulk operations, audit-trail queries |

**Availability & recovery**

| Metric | Target |
|---|---|
| Availability | **99.5%** |
| RPO | **15 minutes** |
| RTO — critical | **2 hours** |
| RTO — standard | **4 hours** |

**Test coverage floors** (measured on changed files only)

| Tier | Floor | Applies to |
|---|---|---|
| Critical | **80%** | `approval`, authentication, audit |
| Business | **70%** | `transfer-request`, `orchestration` |
| Utility | **60%** | `employee-directory`, `notification`, shared helpers |

These supersede the `TO BE COMPLETED FROM BRD` placeholders in `.ai-context/constitution.md` § Non-Functional Baselines.

---

### OQ-11 — Data protection *(Business / Technical)*

**Classification: CONFIDENTIAL — HR personal data.**

| Rule | Requirement |
|---|---|
| DP-01 | **No PII in logs.** Log employee identifiers only; names, emails and addresses are masked. |
| DP-02 | The **transfer reason is encrypted at rest** (free text, may contain sensitive personal disclosure). |
| DP-03 | Read access is restricted to the roster defined in OQ-14. |
| DP-04 | **Every read and write is audited** (see OQ-12). |
| DP-05 | **Retention: 7 years** after journey completion, then purge. |

> **Invented obligations:** the 7-year retention period and the encryption requirement have no basis in the source document. They reflect a defensible default for HR personal data, not a stated legal obligation. The applicable jurisdiction and regulation are **still unknown** and must be confirmed at Gate 0.
>
> DP-02 and DP-05 constrain the **data model**, not merely the logging policy, so they must be settled before the first entity ships.

---

### OQ-12 — Audit trail *(Business)*

**Append-only, immutable.** `UPDATE` and `DELETE` are prohibited on the audit table.

| Column | Purpose |
|---|---|
| `id`, `request_id` | Identity and linkage |
| `actor_id`, `actor_role` | Who acted |
| `action` | What was done |
| `from_status`, `to_status` | Transition recorded |
| `reason` | Decision rationale where supplied |
| `occurred_at` | When |

**Read access**

| Role | Scope |
|---|---|
| Employee | Own request history only |
| Manager | Requests they acted on |
| HR | All requests |

The trail doubles as the data source for BRD-009 (pending actions) and BRD-010 (consolidated progress).

---

### OQ-13 — Failure and timeout behaviour *(Technical)*

**Retry policy:** 3 attempts per step, exponential backoff at 1 s / 4 s / 16 s.

| Scenario | Outcome |
|---|---|
| `OrgDataPort` (the gate) exhausts retries | Request → `FAILED`. **No parallel step is dispatched.** |
| One parallel step exhausts retries | That step → `FAILED`; the others continue. Request → `PARTIALLY_COMPLETED`, surfaced for manual resolution. |
| All parallel steps succeed | Request → `COMPLETED` |

**No compensation or rollback is performed.** A failed journey may leave real-world state inconsistent, resolved by human intervention. This is a deliberate trade — compensation would require a saga pattern, a compensator on every port, and an ADR.

> **Invented thresholds:** the 3-attempt limit and the backoff schedule have no basis in the source document.

---

### OQ-14 — Read authorization *(Business)*

| Role | Scope |
|---|---|
| `ROLE_EMPLOYEE` | Own requests only |
| `ROLE_MANAGER` | Requests of direct reports, where `employee.manager_id == caller` |
| `ROLE_HR` | All requests |
| Anything else | **403 Forbidden** |

Manager scope is derived from `employee-directory`, which makes that module a dependency of the authorization decision. No skip-level, delegate or downstream-stakeholder access is granted — consistent with OQ-03, where Payroll, IT and Facilities are systems rather than human viewers.

---

## Post-Approval Amendments

Defects found in the **ratified** resolutions after Gate 0 approval. Each requires a Gate 0 ruling. **Both are now closed.**

| ID | Summary | Status |
|---|---|---|
| AMD-01 | Org-data write path contradiction (OQ-03/OQ-04 vs OQ-05) | ✅ **Resolved** 2026-09-19 — Option A |
| AMD-02 | Effective-date timing of the org-data change | ✅ **Resolved** 2026-09-19 — Option A, apply immediately |

Amendment IDs are `AMD-nn`, distinct from `DC-nn` (defects in the source document) and `C-nn` (consequences of the resolutions).

---

### AMD-01 — The org-data write path does not exist ✅ RESOLVED *(Raised and ruled 2026-09-19 by Vaibhaw Soni)*

**Status:** ✅ **Resolved 2026-09-19 12:26:07 — Option A.** The analysis below is retained as the record of why; the ruling is at the end of this section.
**Severity:** Blocking. BRD-014 is unimplementable as ratified, and ELIG-03 cannot function at runtime.
**Found during:** author self-review of `.ai-context/specs/employee-directory.spec.md` rev 1.

#### The contradiction

Two ratified resolutions cannot both be true.

| Resolution | What it says |
|---|---|
| **OQ-03 / OQ-04** | `OrgDataPort → update employee record` is a **stubbed outbound port to an external system**, and it is the **hard gate** that must succeed before Payroll, IT and Facilities fan out. |
| **OQ-05** | Employee, department, location and job-position data is **owned locally** by `employee-directory` as seeded reference data. The external-adapter alternative was explicitly rejected. |

If the employee record lives locally, then "updating organisational information" is a **local database write**, not an outbound integration. A stub cannot update data the system itself owns.

#### Consequences if left unresolved

| # | Effect |
|---|---|
| 1 | **BRD-014 is unimplementable.** `orchestration` has nothing legitimate to call. `architecture.md` §3 rule 2 forbids it writing to another module's tables, and `EmployeeDirectoryApi` exposes only read operations. |
| 2 | **ELIG-03 silently always passes.** The rule requires "no completed transfer in the last 12 months", read from `employees.last_transfer_completed_at`. Nothing can write that column, so it stays null forever and the eligibility check is permanently inert — failing open, not closed. |
| 3 | **The OQ-04 hard gate is meaningless.** A stub that writes nothing always succeeds, so the gate protecting Payroll, IT and Facilities from acting on stale org data gives no protection. |
| 4 | **The `employee-directory` module boundary is unsettled**, which blocks finalising its spec and, transitively, the `orchestration` spec. |

#### Resolution options

**Option A — `employee-directory` publishes a narrow write operation (author's recommendation).**

Keeps OQ-05 intact. `OrgDataPort` becomes an in-process call into this module rather than an external stub.

```java
// added to EmployeeDirectoryApi
void applyTransfer(UUID employeeId,
                   UUID newDepartmentId,
                   UUID newLocationId,
                   UUID newJobPositionId,
                   LocalDate effectiveDate);
```

The operation updates the employee's org fields, sets `role_started_at` and `last_transfer_completed_at` to the effective date, and is idempotent per transfer request.

- ✅ Consistent with OQ-05; no re-litigation of master-data ownership
- ✅ Makes BRD-014 implementable and ELIG-03 functional
- ✅ Module boundary preserved — the write goes through the published interface, never direct table access
- ⚠️ `employee-directory` stops being read-only, so its coverage tier and test surface both grow
- ⚠️ Narrows OQ-03: org data becomes the one downstream activity that is **not** an external stub

**Option B — employee org data genuinely lives externally.**

OQ-05 is amended so `employee-directory` becomes an anti-corruption layer over an external HRIS, as originally proposed and rejected.

- ✅ Keeps OQ-03 uniform — all four downstream activities are external
- ❌ Reverses a ratified Gate 0 decision
- ❌ `employee-directory` loses its tables; its spec is rewritten, not amended
- ❌ Reference data becomes remote, so selection validation needs caching and staleness handling

**Option C — accept the gap, descope BRD-014 and ELIG-03.**

Record both as deferred; the stub remains inert and eligibility drops to four rules.

- ✅ No rework
- ❌ Abandons a requirement extracted verbatim from the source document (§2)
- ❌ Weakens eligibility with no HR input
- ❌ Leaves a known always-passing security-adjacent rule in the codebase

#### ✅ RULING — Option A, 2026-09-19 12:26:07

| Field | Value |
|---|---|
| Decision | **Option A — `employee-directory` publishes a narrow write operation** |
| Ruled by | Vaibhaw Soni (vaibhaw.soni@intglobal.com), Gate 0 approver |
| Date/Time | 2026-09-19 12:26:07 |
| Status | **Resolved** |

**Effect on the ratified resolutions.** Neither is reversed; OQ-03 is narrowed.

| Resolution | Amendment |
|---|---|
| **OQ-05** | **Unchanged and confirmed.** Employee, department, location and job-position data remains locally owned by `employee-directory`. |
| **OQ-03** | **Narrowed.** `OrgDataPort` is no longer a stubbed *external* adapter. Organisational-data update becomes an **in-process call** into `employee-directory` through its published `api/` interface. Payroll, IT and Facilities remain stubbed outbound ports, unchanged. |
| **OQ-04** | **Unchanged in shape.** The org-data update remains the hard gate before the parallel fan-out. It is now a real write that can genuinely succeed or fail, so the gate has actual protective value. |
| **OQ-01 / ELIG-03** | **Now functional.** `last_transfer_completed_at` acquires a writer, so the rule stops failing open. |
| **BRD-014** | **Implementable.** `orchestration` calls the published operation. |

**Published operation.** Added to `EmployeeDirectoryApi`, specified in `.ai-context/specs/employee-directory.spec.md`:

```java
void applyTransfer(UUID transferRequestId,
                   UUID employeeId,
                   UUID newDepartmentId,
                   UUID newLocationId,
                   UUID newJobPositionId,
                   LocalDate effectiveDate);
```

Sets the employee's department, location and job position, and sets both `role_started_at` and `last_transfer_completed_at` to the effective date. **Idempotent** — OQ-13 mandates up to three retry attempts, so the operation must tolerate repeated invocation without corrupting state. `transferRequestId` is the idempotency key.

**Module boundary preserved.** The write goes through the published `api/` interface, so `architecture.md` §3 rule 1 holds and rule 2 is not breached — `orchestration` never touches `employee-directory` tables. A new `orchestration → employee-directory` dependency edge is added; it introduces no cycle.

**Consequences accepted with this ruling.**

| ID | Consequence |
|---|---|
| C-09 | `employee-directory` is no longer read-only. Its coverage tier moves from **Utility 60% to Critical 80%**, since it now performs a state-changing operation on HR data. |
| C-10 | The module needs an idempotency store. `transferRequestId` is held as an **opaque key with no foreign key** to `transfer-request` tables, which would breach §3 rule 2. |
| C-11 | **AMD-02 raised** — effective-date timing is now undefined. See below. |

---

### AMD-02 — When does the org-data change take effect? ✅ RESOLVED *(Raised and ruled 2026-09-19 by Vaibhaw Soni)*

**Status:** ✅ **Resolved 2026-09-19 12:37:43 — Option A.** The analysis below is retained as the record of why; the ruling is at the end of this section.
**Severity:** Moderate. Does not block specification or TDD; does change observable behaviour.

**The gap.** OQ-06 requires an effective date **at least 30 days in the future**. OQ-04 runs the org-data update immediately after approval. So `applyTransfer` is called roughly 30+ days *before* the transfer actually takes effect. What should the directory show in the interim?

| Option | Behaviour |
|---|---|
| **A — Apply immediately** | The employee appears in the new department the moment approval completes, weeks early. Simple; visibly wrong to anyone reading the directory. |
| **B — Schedule for the effective date** | Requires a scheduler and a pending-change store. Correct, materially more work. |
| **C — Effective-dated records** | The directory holds history and resolves reads as-of a date. Most correct; largest change to this module. |

#### ✅ RULING — Option A, 2026-09-19 12:37:43

| Field | Value |
|---|---|
| Decision | **Option A — apply immediately** |
| Ruled by | Vaibhaw Soni (vaibhaw.soni@intglobal.com), Gate 0 approver |
| Date/Time | 2026-09-19 12:37:43 |
| Status | **Resolved** — no longer a working assumption; this is now ratified behaviour |

`applyTransfer` writes the employee's new department, location and job position at the moment it is invoked, roughly 30+ days before the OQ-06 effective date. The OQ-04 hard gate therefore hands Payroll, IT and Facilities the **new** organisational data they need to prepare the move, which is what the gate exists for.

**Accepted behaviour, not a defect.** For the interval between approval and the effective date, the directory reports the employee's *post-transfer* department, location and job position. `GET /api/v1/employees/me` reflects this immediately. Recorded explicitly so it is not filed as a bug at Gate 2.

**Rejected alternatives.** Option B (schedule for the effective date) was rejected because it would leave the fan-out reading stale org data, defeating the purpose of the OQ-04 gate. Option C (effective-dated assignment records) was rejected as disproportionate at this stage.

#### C-12 — constraint passed to the `approval` spec

Because `role_started_at` and `last_transfer_completed_at` are set to the **effective date**, both columns hold **future dates** for roughly 30 days after a transfer is approved. `approval` must evaluate the affected rules accordingly:

| Rule | Required formulation | Why |
|---|---|---|
| ELIG-01 | `today - role_started_at >= 12 months` | A future `role_started_at` yields a negative interval, so the employee is correctly ineligible. No special handling needed. |
| ELIG-03 | `last_transfer_completed_at IS NULL OR last_transfer_completed_at <= today - 12 months` | **Must be a simple upper-bound comparison, never a between-range.** A range of `today - 12 months .. today` would exclude a future date and let the rule **fail open**, permitting a second transfer during the very window a transfer is already in flight. |

This is a correctness requirement, not a style preference. It must appear in the `approval` spec and carry its own acceptance criterion.

---

---

## Consequences Arising From These Resolutions


New work and new questions created by the resolutions themselves. **Accepted at Gate 0** as open engineering work rather than blockers — each must still be resolved before the work it gates begins.

| ID | Consequence | Action required |
|---|---|---|
| C-01 | OQ-01's rules ELIG-02, ELIG-04 and ELIG-05 require **disciplinary status, performance rating and notice-period** data, which the OQ-05 `employees` table does not carry. | Extend the `employee-directory` schema, or narrow the eligibility rules. Reviewer decision. |
| C-02 | OQ-13 requires a **`PARTIALLY_COMPLETED`** state that was not in the OQ-02 model as selected. | Added to the model above. Ratify at Gate 0. |
| C-03 | OQ-09 makes requests immutable after submission, leaving **`DRAFT`'s persistence** undefined. | Author's reading: `DRAFT` is transient and not persisted. Ratify or correct. |
| C-04 | OQ-04 (parallel fan-out) plus OQ-13 (retry with backoff) require **asynchronous execution and scheduled retry**. | Decide the mechanism — Spring `@Async` with a bounded executor and scheduled retry, versus a message broker. A broker is a new datastore-class dependency and **requires an ADR** per `architecture.md` §3. |
| C-05 | OQ-11's DP-02 mandates **encryption at rest** for the transfer reason. | Choose the mechanism — JPA attribute converter versus database-level encryption. Affects the data model. |
| C-06 | OQ-10 sets **coverage floors** that must be enforceable at Gate 2. | Add a coverage tool (JaCoCo) to the Maven build, with per-tier thresholds. |
| C-07 | OQ-11 and OQ-12 name a **jurisdiction-free retention and audit policy**. The governing regulation remains unknown. | Confirm the applicable jurisdiction at Gate 0. |
| C-09 | The AMD-01 ruling makes `employee-directory` state-changing, so its **coverage tier rises from Utility 60 percent to Critical 80 percent**. | Applied 2026-09-19 in `constitution.md` and the spec. |
| C-10 | `applyTransfer` needs an **idempotency store**, because OQ-13 mandates up to 3 retry attempts. | `applied_transfers` table added, keyed on an opaque `transfer_request_id` with **no foreign key** to `transfer-request`. |
| C-11 | The AMD-01 ruling left **effective-date timing** undefined. | Raised as AMD-02; **resolved 2026-09-19** — apply immediately. |
| C-12 | The AMD-02 ruling leaves `role_started_at` and `last_transfer_completed_at` holding **future dates** for ~30 days. | **ELIG-03 must use `last_transfer_completed_at <= today - 12 months`, never a between-range**, or it fails open. Must appear in the `approval` spec with its own acceptance criterion. |

## Requirements Added by These Resolutions

The resolutions introduce behaviour beyond the 18 requirements extracted from the source document. These are **derived**, not extracted, and were **ratified at Gate 0** on 2026-09-18. They are binding on downstream specs.

| ID | Requirement | Derived from |
|---|---|---|
| BRD-019 | An employee shall be able to **withdraw** a transfer request before it is approved. | OQ-08 |
| BRD-020 | A manager shall be able to **record an approval or rejection decision**, with a mandatory reason on rejection. | OQ-03, OQ-08 |
| BRD-021 | HR shall be able to **record an eligibility decision**, with a mandatory reason on ineligibility. | OQ-03, OQ-08 |
| BRD-022 | The system shall maintain an **immutable audit trail** of every state transition and decision. | OQ-11, OQ-12 |
| BRD-023 | The system shall **restrict read access** to a transfer request by role and scope. | OQ-14 |
| BRD-024 | The system shall **retry** a failed downstream activity and park it for manual resolution on exhaustion. | OQ-13 |

## 10. Engagement Context

Recorded from §4–§7 of the source document. This is **process** scaffolding, not system behaviour, and produces no functional requirement.

### Required deliverables (§4)

| # | Deliverable |
|---|---|
| 1 | Requirement / Discovery Analysis — objective, users, journey stages, business rules, known decisions, open questions, assumptions, dependencies, out-of-scope; distinguishing business vs technical decisions |
| 2 | Feature Specification (`.md`) with individually identifiable Acceptance Criteria |
| 3 | Spec-Derived Test Cases |
| 4 | Technical Plan |
| 5 | Task Decomposition |
| 7 | AI Prompts |
| 8 | Security Assessment |
| 9 | Gate 1 Review |
| 10 | Gate 2 Evidence |

> **Document defect:** the deliverable list skips **number 6** — it runs 1, 2, 3, 4, 5, 7, 8, 9, 10. Either a deliverable is missing or the numbering is in error. Raised as [DC-01](#document-conflicts--defects).

### Evaluation criteria (§5)

| Area | Weight |
|---|---|
| Business journey understanding | 10% |
| Ambiguity & discovery | 15% |
| Specification quality | 20% |
| Acceptance criteria & testability | 15% |
| Task decomposition | 10% |
| Test-first approach | 5% |
| Security & failure handling | 5% |
| SDD traceability | 20% |
| **Total** | **100%** |

### Timeline & milestones (§6–§7)

A 10-day recommended timeline is given, with four milestones: Discovery & Specification, Gate 1, Plan → Tasks, and Implementation & Gate 2. Stated effort is approximately **30–35 hours across 8 working days**.

---

## Document Conflicts & Defects

**Acknowledged at Gate 0** as defects in the source brief. None affects the requirement baseline. Per governance rules, conflicts are flagged rather than silently resolved.

| ID | Conflict | Detail |
|---|---|---|
| DC-01 | Deliverable numbering skips 6 | §4 lists 1, 2, 3, 4, 5, 7, 8, 9, 10 |
| DC-02 | Milestone 1 dates contradict themselves | §7 titles Milestone 1 "Days 1–2" but then requires its output "by the end of Day 3" |
| DC-03 | Gate 1 date conflict | §7 places Gate 1 (Milestone 2) on **Day 3**; the §6 timeline places Gate 1 Peer Review on **Day 4** |
| DC-04 | Milestone 4 date conflict | §7 places Implementation & Gate 2 on **Days 6–8**; the §6 timeline runs implementation and Gate 2 through **Day 10** |
| DC-05 | Duration conflict | §6 presents a **10-day** timeline; §7 closes with "approximately 30–35 hours over **8 working days**" |
| DC-06 | SDD chain omits Gate 0 | §1 states the chain as `Business Requirement → Spec → Gate 1 → …`, with no BRD review gate. `.agent/rules/int-standards.md` §6 **mandates Gate 0 BRD review before any spec drafting**. The organisational rule governs; this document is therefore being processed through Gate 0. Flagged rather than resolved silently. |

---

## 11. Acceptance Criteria (business level)

Business-level acceptance conditions, each traceable to a stated requirement. They are **restatements of the source, not inventions**.

Individually identifiable, testable acceptance criteria are **Deliverable 2** and belong in the feature specs, which cannot be drafted until this document is approved at Gate 0.

| ID | Condition | Traces to |
|---|---|---|
| BRD-AC01 | An employee can initiate and submit an internal transfer request specifying department, location, role and effective date, with an optional reason. | BRD-001 to BRD-007 |
| BRD-AC02 | A submitted request causes the downstream activities of the transfer journey to be orchestrated by the system rather than by the employee contacting teams individually. | BRD-011 to BRD-017 |
| BRD-AC03 | At any point after submission, the employee can retrieve the current status of the request and see which actions are pending with which stakeholders, from a single consolidated view. | BRD-008 to BRD-010 |
| BRD-AC04 | The journey includes manager confirmation and HR eligibility validation. | BRD-012, BRD-013 |
| BRD-AC05 | On completion of the journey, the employee receives confirmation. | BRD-018 |

> **Now testable in principle.** The blocking questions OQ-02, OQ-03 and OQ-04 have proposed resolutions, so the status model, actor mechanics and execution order are defined. BRD-AC02 and BRD-AC04 can be decomposed into individually identifiable criteria at spec time — **once Gate 0 ratifies those resolutions.** Six further conditions arise from the resolutions (BRD-019 to BRD-024) and need their own acceptance criteria.

---

## 12. Proposed Business Domains

Candidate module boundaries derived from this baseline, recorded in full in `.ai-context/architecture.md` §9.

**These are a proposal only.** Per `int-brd-ingestion` Step 3, **no business module folder may be created until Gate 1 architecture approval.** `src/backend/src/main/java/com/intglobal/etp/modules/` remains empty.

| Candidate module | Covers |
|---|---|
| `transfer-request` | Request capture, validation, lifecycle and status — BRD-001 to BRD-010, BRD-019, BRD-023 |
| `approval` | Manager confirmation and HR eligibility validation — BRD-012, BRD-013, BRD-020, BRD-021 |
| `orchestration` | Downstream activity coordination, retry and progress aggregation — BRD-011, BRD-014 to BRD-017, BRD-024 |
| `notification` | Employee confirmation and stakeholder alerts — BRD-018 |
| `employee-directory` | Employee, department, location and role reference data; manager hierarchy — supports BRD-002 to BRD-004, BRD-023 |
| `audit` *(new)* | Immutable append-only journey audit trail — BRD-022. Arises from OQ-11 and OQ-12; may live in `shared/` rather than as a business module. |

---

## 13. Gate 0 Review Record

| Field | Value |
|---|---|
| Review Status | **PENDING** |
| Approver | TBD |
| Review Date/Time | None |
| Review Record | None |
| Identity verification | Pending |
| Governance waiver | None — `author ≠ reviewer` rule is strictly enforced |

### Decisions Proposed for Gate 0

| Item | Status |
|---|---|
| OQ-01 to OQ-14 | **Pending Ratification** |
| BRD-019 to BRD-024 | **Pending Ratification** as derived requirements |
| Assumption A-03 | **Pending Confirmation** |
| Consequences C-01 to C-07 | **Pending Acceptance** |
| Conflicts DC-01 to DC-06 | **Pending Acknowledgement** |
| Scope boundary (§2) and out-of-scope list (§8) | **Pending Confirmation** |

### Standing caveat carried forward

A substantial number of values in §9 are author-invented with no support from `docs/Requirement for SDD.docx` — the eligibility thresholds (OQ-01), the effective-date window (OQ-06), every non-functional figure (OQ-10), the retention period and encryption requirement (OQ-11), and the retry policy (OQ-13). 

They are treated as **proposals only**. They must be explicitly reviewed and ratified by an independent Technical Lead at Gate 0 before they become binding.

### Gates still ahead

| Gate | Owner | Status |
|---|---|---|
| Gate 1 — Spec Peer Review | Soumyadeep Adhikary | `author ≠ reviewer` **in force**; independent review resumes here |
| Gate 1 — Architecture approval | Soumyadeep Adhikary | Six-module proposal in `architecture.md` §9.7 **not approved**; no module folder may be created |
| Gate 2 — Code Review | **Unassigned** | Blocks release until a reviewer is assigned |
