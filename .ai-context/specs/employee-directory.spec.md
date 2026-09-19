# Spec: Employee Directory & Reference Data

## Spec ID
`employee-directory`

## Status
**In Peer Review** — revision 4

## Revision History

| Rev | Date | Change |
|---|---|---|
| 1 | 2026-09-18 21:02 | Initial authoring; submitted for Gate 1. |
| 2 | 2026-09-19 12:12 | Author self-review. Closed four code-generation blockers (principal mapping, published DTO shapes, security authorities, seed data). Corrected four internal inconsistencies. Added deterministic ordering, a 403 criterion and mandated log statements. Retargeted the N+1 criterion. Raised AMD-01. ACs renumbered. |
| 3 | 2026-09-19 12:26 | **AMD-01 ruled — Option A.** Added the published `applyTransfer` write operation with an idempotency store, four new acceptance criteria and five tests. Coverage tier raised to Critical 80 percent. Removed the blocking banner. Records the AMD-02 working assumption. |
| 4 | 2026-09-19 12:37 | **AMD-02 ruled — Option A, apply immediately.** Working assumption becomes ratified behaviour. Added AC24 pinning the accepted "directory shows post-transfer values early" behaviour so it cannot be filed as a defect at Gate 2. Records constraint C-12 forward to the `approval` spec. All amendments closed; no open dependencies on the BRD. |

## Roles & Assignments
- **Developer:** Vaibhaw Soni (vaibhaw.soni@intglobal.com)
- **Gate 1 Reviewer(s):** Soumyadeep Adhikary (soumyadeep@intglobal.com)
- **Gate 2 Reviewer(s):** TBD — unassigned

> ⚠ **`author ≠ reviewer` is IN FORCE at Gate 1.** The Gate 0 waiver applied to the BRD baseline only. This spec must be approved by Soumyadeep Adhikary under Git identity `soumyadeep@intglobal.com`. The author cannot self-approve.

> ✅ **AMD-01 resolved 2026-09-19 — Option A.** The module boundary is now final: `employee-directory` publishes one narrow write operation, `applyTransfer`. OQ-05 confirmed, OQ-03 narrowed. This spec is complete for Gate 1 review.
>
> ✅ **AMD-02 resolved 2026-09-19 — Option A.** The org-data change applies **immediately** on invocation, ~30+ days before the effective date. This is ratified behaviour, not an assumption. For that interval the directory reports post-transfer values **by design** — see AC24.

## Linked BRD

Primary requirements — `.ai-context/BRD.md`:

| BRD ID | Requirement | Relationship |
|---|---|---|
| BRD-002 | Employee selects proposed new department / business unit | Supplies and validates the selectable set |
| BRD-003 | Employee selects proposed new location | Supplies and validates the selectable set |
| BRD-004 | Employee selects proposed role / job position | Supplies and validates the selectable set |

Supporting requirements — this module is the data source, not the owner of the behaviour:

| BRD ID | Requirement | Relationship |
|---|---|---|
| BRD-013 | HR validation of employee eligibility | Supplies the attributes rules ELIG-01 to ELIG-05 evaluate |
| BRD-023 | Read access restricted by role and scope | Supplies the authority set and the `manager_id` hierarchy |
| BRD-014 | Update the employee's organisational information | **Owned here** via `applyTransfer` (AMD-01 Option A). `orchestration` invokes it as the OQ-04 hard gate. |

Governing resolutions: **OQ-05** (local ownership), **OQ-01** (eligibility rules), **OQ-14** (read authorization), **OQ-11** (data protection).

## Gate Approvals & History

| Gate | Approver Name | Approver Email/ID | Date/Time | Outcome | Approval Comment / Summary |
|---|---|---|---|---|---|
| Gate 1 (Spec Review) | _pending_ | _pending_ | _pending_ | **Pending Review** | Rev 2 submitted 2026-09-19 12:12:43. Record → `.ai-context/pr_reviews/GATE1-employee-directory-<timestamp>.md` |
| Gate 2 (Code Review) | _pending_ | _pending_ | _pending_ | Not Started | Blocked — Gate 2 reviewer unassigned |

## Intent

Establish the authoritative local store of organisational reference data — departments, locations, job positions — and employee records, so that a transfer request can be validated against real organisational structure, authorized against the manager hierarchy, and assessed for HR eligibility. The One-Point Portal reads the selectable reference sets over HTTP; other backend modules consume employee data through a published in-process interface. This module owns no transfer behaviour of its own; it is the foundation every other module in the journey depends on.

## Context

- **Builds on:** `.ai-context/architecture.md` §9.7 (`employee-directory`), §3 (module boundary rules), §9.3 (database boundaries)
- **Governed by:** `.ai-context/constitution.md` § Security Posture (DP-01 to DP-05), § Non-Functional Baselines, § Architectural Constraints
- **Depended on by:** `transfer-request` (validation, authorization scope), `approval` (eligibility attributes), `audit-trail` (actor resolution), the future authentication spec (authority resolution)
- **Depends on:** nothing upstream. The only module in the journey with no upstream dependency, which is why it is specified first.

### Terminology — two different things called "role"

The source document and OQ-14 both use the word *role* for different concepts. This spec separates them permanently, because conflating them is a live implementation hazard:

| Concept | This spec calls it | Example |
|---|---|---|
| The employee's **job position** (BRD-004) | **job position** — table `job_positions`, field `job_position_id` | "Senior Software Engineer" |
| The employee's **security authority** (OQ-14) | **authority** — table `employee_authorities` | `ROLE_HR` |

> **Deviation from OQ-05, flagged for Gate 1.** OQ-05 names the table `roles`. This spec renames it `job_positions`. The rename is terminological only — no semantic change — and BRD-004 itself says "role / job position". Ratify or reject.

### Consequence C-01 resolved here

BRD consequence **C-01** recorded that the OQ-01 eligibility rules require employee attributes the directory did not carry. This spec resolves it by **extending the `employees` schema with five attributes** — `role_started_at`, `performance_rating`, `disciplinary_status`, `notice_period_status`, `last_transfer_completed_at` — rather than narrowing the rules. The alternative was dropping ELIG-01, ELIG-02, ELIG-04 and ELIG-05.

`joined_at` from OQ-05 is **retained**. It is distinct from `role_started_at`: ELIG-01 measures tenure *in the current role*, not total service. Rev 1 dropped `joined_at` in error.

## API Contract

> **Spec-level assumption — API versioning.** Endpoints are mounted under `/api/v1`. `constitution.md` § Versioning Rules records this as `TO BE COMPLETED FROM BRD`. This spec **proposes** path versioning and requires Gate 1 ratification plus an ADR. If rejected, every path below changes.

> **Derivation note.** BRD-002 to BRD-004 say the employee *selects* a department, location and job position, but never that the system exposes the selectable lists. Because the portal UI is an external consumer (A-01, confirmed at Gate 0), selection is impossible unless the backend serves them. The three list endpoints are **derived**, not extracted. Gate 1 should confirm.

### Cross-cutting contract rules

These bind every endpoint below.

1. **Authenticated principal resolution.** The caller's identity resolves to exactly one `employees` row by matching the JWT subject claim against `employees.email`, case-insensitively. This is a **hard requirement placed on the future authentication spec**: the token's `sub` claim must carry the employee's corporate email. Flagged for Gate 1 — the alternative is a dedicated `employeeId` claim, which removes the AC8 "no directory record" case entirely.
2. **Deterministic ordering.** Every collection endpoint sorts by its display field ascending (`name`, or `title` for job positions), then by `id` ascending as a tiebreak. Without this, pagination is non-deterministic and untestable.
3. **Error envelope.** All errors return `{ "code": "<MACHINE_CODE>", "message": "<human readable>" }`. This requires a module-level `@RestControllerAdvice`, because `application.yml` sets `server.error.include-message: never`, which would otherwise yield Spring's default error shape.
4. **Pagination validation is explicit.** Spring Data silently clamps an oversized `size` and coerces a negative `page` to `0`. That default must be **overridden** — invalid pagination is rejected, not repaired.
5. **Response envelope is a custom DTO.** Spring's default `Page` serialization emits `number`, `pageable`, `sort`, `first`, `last`, `empty` and more. The contract below is a purpose-built DTO, not a serialized `Page`.

---

### `employee-directory.API01` — `GET /api/v1/departments`

**Query parameters:** `page` (integer, default `0`, must be ≥ 0), `size` (integer, default `20`, must be 1–100)

**Success response (`200 OK`):**
```json
{
  "content": [ { "id": "uuid", "name": "string" } ],
  "page": 0,
  "size": 20,
  "totalElements": 0,
  "totalPages": 0
}
```

**Exceptions:**

| Code | Condition | Response body |
|---|---|---|
| 400 | `size` > 100, `size` < 1, or `page` < 0 | `{ "code": "INVALID_PAGINATION", "message": "string" }` |
| 401 | Absent or invalid authentication token | `{ "code": "UNAUTHENTICATED", "message": "string" }` |
| 403 | Authenticated but lacking `ROLE_EMPLOYEE` | `{ "code": "FORBIDDEN", "message": "string" }` |

---

### `employee-directory.API02` — `GET /api/v1/locations`

Identical to API01, returning `{ "id": "uuid", "name": "string" }`. Exceptions as API01.

---

### `employee-directory.API03` — `GET /api/v1/job-positions`

Identical to API01, returning `{ "id": "uuid", "title": "string" }`, sorted by `title`. Exceptions as API01.

---

### `employee-directory.API04` — `GET /api/v1/employees/me`

Returns the authenticated employee's own profile, so the portal can show what they are transferring from.

**Success response (`200 OK`):**
```json
{
  "id": "uuid",
  "fullName": "string",
  "email": "string",
  "department":  { "id": "uuid", "name": "string" },
  "location":    { "id": "uuid", "name": "string" },
  "jobPosition": { "id": "uuid", "title": "string" },
  "managerId": "uuid|null"
}
```

**Exceptions:**

| Code | Condition | Response body |
|---|---|---|
| 401 | Absent or invalid authentication token | `{ "code": "UNAUTHENTICATED", "message": "string" }` |
| 403 | Authenticated but lacking `ROLE_EMPLOYEE` | `{ "code": "FORBIDDEN", "message": "string" }` |
| 404 | Subject claim matches no `employees.email` | `{ "code": "EMPLOYEE_NOT_FOUND", "message": "string" }` |

**Deliberately excluded:** `performanceRating`, `disciplinaryStatus`, `noticePeriodStatus`, `lastTransferCompletedAt`, `joinedAt`, `roleStartedAt` and the authority set. These are HR-sensitive and reach `approval` only through the in-process interface. Exposing an employee's own performance rating or disciplinary status serves no BRD requirement and widens the personal-data surface.

---

### `employee-directory.API05` — published in-process interface

The module's `api/` package, the only way other modules may reach this data (`architecture.md` §3 rule 1).

```java
public interface EmployeeDirectoryApi {
    Optional<EmployeeView> findEmployee(UUID employeeId);
    Optional<EmployeeView> findEmployeeByEmail(String email);
    Optional<EmployeeEligibilityAttributes> findEligibilityAttributes(UUID employeeId);
    Set<String> findAuthorities(UUID employeeId);
    boolean isManagerOf(UUID managerId, UUID employeeId);
    boolean isSelectableDepartment(UUID departmentId);
    boolean isSelectableLocation(UUID locationId);
    boolean isSelectableJobPosition(UUID jobPositionId);

    // Write — added by the AMD-01 ruling (Option A). The only write this module publishes.
    ApplyTransferOutcome applyTransfer(ApplyTransferCommand command);
}
```

**Published DTOs.** Immutable records in the `api/` package. No JPA entity ever crosses the module boundary.

```java
public record EmployeeView(
    UUID id,
    String fullName,
    String email,
    UUID departmentId,
    String departmentName,
    UUID locationId,
    String locationName,
    UUID jobPositionId,
    String jobPositionTitle,
    UUID managerId,        // nullable
    LocalDate joinedAt
) {}

public record EmployeeEligibilityAttributes(
    UUID employeeId,
    LocalDate roleStartedAt,
    PerformanceRating performanceRating,
    DisciplinaryStatus disciplinaryStatus,
    NoticePeriodStatus noticePeriodStatus,
    LocalDate lastTransferCompletedAt   // nullable
) {}
```

```java
public record ApplyTransferCommand(
    UUID transferRequestId,   // idempotency key — opaque, no FK to transfer-request
    UUID employeeId,
    UUID newDepartmentId,
    UUID newLocationId,
    UUID newJobPositionId,
    LocalDate effectiveDate
) {}

public enum ApplyTransferOutcome { APPLIED, ALREADY_APPLIED }
```

`EmployeeEligibilityAttributes` carries **raw attributes only** — no computed booleans and no date arithmetic. Evaluating "≥ 12 months" belongs to `approval`.

> **Constraints passed to the `approval` spec.**
>
> 1. **Injectable `Clock`.** ELIG-01 and ELIG-03 are relative-date calculations. `approval` must evaluate them against an injectable `java.time.Clock`, never `LocalDate.now()`, or its tests become time-dependent and will fail near month boundaries. This module performs no date arithmetic, so the constraint does not apply here.
> 2. **C-12 — ELIG-03 must not fail open.** Under the AMD-02 ruling, `role_started_at` and `last_transfer_completed_at` hold **future dates** for ~30 days after approval. ELIG-03 must therefore be written as `last_transfer_completed_at IS NULL OR last_transfer_completed_at <= today - 12 months` — **a simple upper bound, never a between-range**. A range of `today - 12 months .. today` would exclude a future date and permit a second transfer while one is already in flight. ELIG-01 needs no special handling: a future `role_started_at` yields a negative interval and correctly fails the rule.

| Operation | Consumer | Purpose |
|---|---|---|
| `findEmployee`, `findEmployeeByEmail` | `transfer-request`, `audit-trail`, auth spec | Current values for "must differ" validation; actor resolution; principal lookup |
| `findEligibilityAttributes` | `approval` | Evaluates ELIG-01 to ELIG-05 |
| `findAuthorities` | Authentication spec | Builds the Spring Security authority set |
| `isManagerOf` | `transfer-request` | Scopes `ROLE_MANAGER` access per OQ-14 |
| `isSelectable*` | `transfer-request` | Validates BRD-002 to BRD-004 selections |

### `applyTransfer` semantics (AMD-01 Option A)

The only state-changing operation this module exposes. Invoked by `orchestration` as the OQ-04 hard gate, before Payroll, IT and Facilities fan out.

| Aspect | Behaviour |
|---|---|
| Effect | Sets `department_id`, `location_id`, `job_position_id` to the new values; sets `role_started_at` and `last_transfer_completed_at` to `effectiveDate` |
| Timing | **Applied immediately on invocation** — ratified by the AMD-02 ruling. The change lands ~30+ days before the effective date so the OQ-04 fan-out reads the new org data it needs. |
| Idempotency | First call returns `APPLIED`. Any repeat with the same `transferRequestId` is a no-op returning `ALREADY_APPLIED`. Required because OQ-13 mandates up to 3 retry attempts. |
| Atomicity | The employee update and the idempotency record commit in one transaction. A partial apply is not permitted. |
| Validation | Every target id must reference an **active** row. An unknown or inactive target is rejected — the operation must not silently write a dangling reference. |
| Failure | Throws a typed exception. `orchestration` treats it as a failed gate step and applies its own retry policy; this module performs no retry of its own. |
| Boundary | `transferRequestId` is an **opaque idempotency key**. No foreign key to `transfer-request` tables — that would breach `architecture.md` §3 rule 2. |

> **Why `role_started_at` is reset.** ELIG-01 measures tenure in the *current* role. A transferred employee starts a new role, so the clock restarts. `joined_at` is never modified — total service is continuous.

> **Ownership note — "must differ from current".** OQ-05 requires that a proposed department, location or job position differ from the employee's current value. That check is owned by **`transfer-request`**, not here: `isSelectable*` answers only "is this a valid, active option". `transfer-request` obtains current values via `findEmployee` and performs the comparison. Stated explicitly because rev 1 left it ambiguous.

## Data Model

Owned exclusively by this module. No other module may read these tables directly (`architecture.md` §3 rules 2 and 3).

| Table | Columns |
|---|---|
| `departments` | `id` uuid pk · `name` varchar(120) not null · `active` boolean not null default true |
| `locations` | `id` uuid pk · `name` varchar(120) not null · `active` boolean not null default true |
| `job_positions` | `id` uuid pk · `title` varchar(120) not null · `active` boolean not null default true |
| `employees` | `id` uuid pk · `full_name` varchar(200) not null · `email` varchar(254) not null unique · `department_id` fk not null · `location_id` fk not null · `job_position_id` fk not null · `manager_id` fk → employees.id **nullable** · `joined_at` date not null · `role_started_at` date not null · `performance_rating` enum not null · `disciplinary_status` enum not null · `notice_period_status` enum not null · `last_transfer_completed_at` date **nullable** |
| `employee_authorities` | `employee_id` fk not null · `authority` varchar(40) not null · pk (`employee_id`, `authority`) |
| `applied_transfers` | `transfer_request_id` uuid **pk** · `employee_id` fk not null · `effective_date` date not null · `applied_at` timestamptz not null. Idempotency store for `applyTransfer`. `transfer_request_id` is an opaque key with **no foreign key** to `transfer-request`. |

**Indexes:** every foreign key, plus unique on `lower(employees.email)` and an index on `employees.manager_id`, per `constitution.md` ("Index every foreign key and every column used for filtering"). The functional lower-case index backs the case-insensitive principal lookup.

**Enumerations** (values derived from the ratified OQ-01 rules, not from the source document):

| Enum | Values |
|---|---|
| `performance_rating` | `BELOW_EXPECTATIONS`, `MEETS_EXPECTATIONS`, `EXCEEDS_EXPECTATIONS` |
| `disciplinary_status` | `NONE`, `ACTIVE` |
| `notice_period_status` | `NOT_SERVING`, `SERVING` |

**Authorities.** `employee_authorities.authority` holds `ROLE_EMPLOYEE`, `ROLE_MANAGER` or `ROLE_HR` (OQ-14). Authorities are **stored explicitly, never derived**: deriving `ROLE_MANAGER` from "is anyone's `manager_id`" would make a security decision depend on an incidental data shape. Every employee carries `ROLE_EMPLOYEE`.

> **Scope note for Gate 1.** Storing authorities arguably belongs to the authentication spec. It is placed here because this module owns employee data and OQ-14 is otherwise unimplementable — nothing anywhere records who is HR. The authentication spec consumes it via `findAuthorities`. Ratify or relocate.

**Migrations.** `ddl-auto: none`; schema is created by migration only. The migration tool is **undecided** (`architecture.md` §7 item 1, ADR required) — see Open Dependencies.

## Seed Data

OQ-05 ratified this data as "seeded" but never specified it. Without a concrete set, the acceptance criteria below are unexecutable. Delivered as a repeatable seed migration, applied in every environment including test.

**Departments** — Engineering, Finance, Human Resources, Operations, Sales (active); Legacy R&D (**inactive**)

**Locations** — Kolkata, Bengaluru, Pune, London (active); Chennai (**inactive**)

**Job positions** — Software Engineer, Senior Software Engineer, Technical Lead, HR Business Partner, Finance Analyst (active); Trainee Engineer (**inactive**)

**Employees** — six records giving every acceptance criterion a fixture:

| # | Email | Manager | Authorities | Eligibility shape |
|---|---|---|---|---|
| E1 | `asha.rao@example.com` | — (none) | `ROLE_EMPLOYEE`, `ROLE_MANAGER` | Fully eligible |
| E2 | `ben.oyelaran@example.com` | E1 | `ROLE_EMPLOYEE` | Fully eligible |
| E3 | `chen.wei@example.com` | E1 | `ROLE_EMPLOYEE` | Fails ELIG-01 — `role_started_at` 3 months ago |
| E4 | `dara.smith@example.com` | E1 | `ROLE_EMPLOYEE`, `ROLE_HR` | Fully eligible |
| E5 | `elif.demir@example.com` | E1 | `ROLE_EMPLOYEE` | Fails ELIG-02 and ELIG-04 — disciplinary `ACTIVE`, rating `BELOW_EXPECTATIONS` |
| E6 | `farid.haddad@example.com` | E1 | `ROLE_EMPLOYEE` | Fails ELIG-03 and ELIG-05 — transferred 4 months ago, `SERVING` notice |

Seeded emails use the reserved `example.com` domain so no real personal data enters the repository.

> Seed dates are stored as **fixed absolute dates**, not offsets from "today". A seed computed relative to insertion time makes eligibility fixtures drift and the `approval` tests flaky.

## Acceptance Criteria

1. **`employee-directory.AC01`** — Given the directory holds active and inactive departments, when `GET /api/v1/departments` is called, then only rows with `active = true` are returned.
2. **`employee-directory.AC02`** — Given the directory holds active and inactive locations, when `GET /api/v1/locations` is called, then only rows with `active = true` are returned.
3. **`employee-directory.AC03`** — Given the directory holds active and inactive job positions, when `GET /api/v1/job-positions` is called, then only rows with `active = true` are returned.
4. **`employee-directory.AC04`** — Given more active rows exist than one page holds, when a collection endpoint is called with `page` and `size`, then exactly that page is returned with correct `totalElements` and `totalPages`.
5. **`employee-directory.AC05`** — Given a collection endpoint is called twice with the same `page` and `size`, when the two responses are compared, then the ordering is identical and sorted by display field ascending, `id` ascending as tiebreak.
6. **`employee-directory.AC06`** — Given a collection endpoint is called with `size` > 100, `size` < 1 or `page` < 0, when the request is handled, then `400 INVALID_PAGINATION` is returned, no data is disclosed, and the value is **not** silently clamped.
7. **`employee-directory.AC07`** — Given an authenticated principal whose subject claim matches an `employees.email` (case-insensitively), when `GET /api/v1/employees/me` is called, then that employee's profile is returned with department, location, job position and `managerId`.
8. **`employee-directory.AC08`** — Given an authenticated principal whose subject claim matches no `employees.email`, when `GET /api/v1/employees/me` is called, then `404 EMPLOYEE_NOT_FOUND` is returned.
9. **`employee-directory.AC09`** — Given no valid authentication token, when any of API01 to API04 is called, then `401 UNAUTHENTICATED` is returned with the standard error envelope and no directory data.
10. **`employee-directory.AC10`** — Given an authenticated principal lacking `ROLE_EMPLOYEE`, when any of API01 to API04 is called, then `403 FORBIDDEN` is returned and no directory data is disclosed.
11. **`employee-directory.AC11`** — Given `GET /api/v1/employees/me` succeeds, when the response body is inspected, then it contains none of `performanceRating`, `disciplinaryStatus`, `noticePeriodStatus`, `lastTransferCompletedAt`, `joinedAt`, `roleStartedAt` or any authority field.
12. **`employee-directory.AC12`** — Given a department, location or job position id that is unknown or inactive, when the matching `isSelectable*` operation is called, then `false` is returned.
13. **`employee-directory.AC13`** — Given employee E whose `manager_id` is M, when `isManagerOf(M, E)` is called then `true` is returned; and given employee X not managed by M, when `isManagerOf(M, X)` is called then `false` is returned.
14. **`employee-directory.AC14`** — Given an employee with a null `manager_id`, when `isManagerOf` is called with any manager id, then `false` is returned and no exception is thrown.
15. **`employee-directory.AC15`** — Given an employee record exists, when `findEligibilityAttributes` is called, then `roleStartedAt`, `performanceRating`, `disciplinaryStatus`, `noticePeriodStatus` and `lastTransferCompletedAt` are all returned as raw values, so ELIG-01 to ELIG-05 are each evaluable by `approval`.
16. **`employee-directory.AC16`** — Given an employee holding `ROLE_EMPLOYEE` and `ROLE_HR`, when `findAuthorities` is called, then exactly those two authorities are returned; and given an unknown employee id, then an empty set is returned.
17. **`employee-directory.AC17`** — Given a successful `GET /api/v1/employees/me` and a 404 from the same endpoint, when the emitted log statements are inspected, then each request produced at least one log line, and no line contains any employee `full_name` or `email`; employees appear as `id` only.
18. **`employee-directory.AC18`** — Given `GET /api/v1/employees/me` is served, when executed statements are counted, then the employee and its department, location and job position are fetched without triggering a separate query per association (no N+1).
19. **`employee-directory.AC19`** — Given the seed migration has run against an empty schema, when it is applied a second time, then it completes without error and produces no duplicate rows.
20. **`employee-directory.AC20`** — Given a valid `ApplyTransferCommand` for an employee, when `applyTransfer` is called, then the employee's department, location and job position are updated to the new values, `role_started_at` and `last_transfer_completed_at` are both set to `effectiveDate`, `joined_at` is unchanged, and `APPLIED` is returned.
21. **`employee-directory.AC21`** — Given `applyTransfer` has already succeeded for a `transferRequestId`, when it is called again with that same id, then no further change is written and `ALREADY_APPLIED` is returned.
22. **`employee-directory.AC22`** — Given an `ApplyTransferCommand` naming an unknown or inactive department, location or job position, when `applyTransfer` is called, then the command is rejected, no field on the employee is modified, and no idempotency record is written.
23. **`employee-directory.AC23`** — Given `applyTransfer` fails partway, when the transaction resolves, then neither the employee update nor the idempotency record persists — the operation is all-or-nothing.
24. **`employee-directory.AC24`** — Given `applyTransfer` has succeeded with an `effectiveDate` in the future, when that employee calls `GET /api/v1/employees/me` **before** the effective date arrives, then the response shows the **new** department, location and job position. This is ratified behaviour under AMD-02 Option A, not a defect.

## Unit Test Cases (spec-derived)

| Test ID | Maps to AC | Scenario | Expected |
|---|---|---|---|
| `employee-directory.UT01` | AC01 | 5 active, 1 inactive department seeded | Exactly the 5 active returned |
| `employee-directory.UT02` | AC02 | 4 active, 1 inactive location seeded | Exactly the 4 active returned |
| `employee-directory.UT03` | AC03 | 5 active, 1 inactive job position seeded | Exactly the 5 active returned |
| `employee-directory.UT04` | AC04 | 5 active departments, `page=0&size=2` | 2 items, `totalElements=5`, `totalPages=3` |
| `employee-directory.UT05` | AC04 | Same data, `page=2&size=2` | 1 item, `totalElements=5`, `totalPages=3` |
| `employee-directory.UT06` | AC05 | Call `/departments` twice, same params | Identical order; ascending by `name` |
| `employee-directory.UT07` | AC06 | `size=101` | `400 INVALID_PAGINATION` |
| `employee-directory.UT08` | AC06 | `size=0` | `400 INVALID_PAGINATION` |
| `employee-directory.UT09` | AC06 | `page=-1` | `400 INVALID_PAGINATION`, not coerced to page 0 |
| `employee-directory.UT10` | AC07 | Principal `BEN.OYELARAN@EXAMPLE.COM` (upper case) | `200`, resolves to E2 — case-insensitive match |
| `employee-directory.UT11` | AC07 | Principal E1 (null `manager_id`) | `200` with `"managerId": null` |
| `employee-directory.UT12` | AC08 | Principal `nobody@example.com` | `404 EMPLOYEE_NOT_FOUND` |
| `employee-directory.UT13` | AC09 | No bearer token, each of API01 to API04 | `401 UNAUTHENTICATED` with `{code,message}` envelope |
| `employee-directory.UT14` | AC10 | Authenticated, authorities empty | `403 FORBIDDEN`, no data |
| `employee-directory.UT15` | AC11 | Inspect `/employees/me` payload keys | None of the seven excluded fields present |
| `employee-directory.UT16` | AC12 | Unknown uuid, then inactive-row uuid, all three selectors | `false` in every case |
| `employee-directory.UT17` | AC13 | `isManagerOf(E1, E2)`; `isManagerOf(E2, E3)` | `true`; `false` |
| `employee-directory.UT18` | AC14 | `isManagerOf(E2, E1)` — E1 has null manager | `false`, no exception |
| `employee-directory.UT19` | AC15 | E6 — fails ELIG-03 and ELIG-05 | All five attributes returned raw, unevaluated |
| `employee-directory.UT20` | AC16 | `findAuthorities(E4)`; `findAuthorities(random uuid)` | `{ROLE_EMPLOYEE, ROLE_HR}`; empty set |
| `employee-directory.UT21` | AC17 | Capture log appender across the 200 and 404 paths | ≥1 line per request; no name or email in any line |
| `employee-directory.UT22` | AC18 | Count statements for `/employees/me` | Constant count; no per-association query |
| `employee-directory.UT23` | AC19 | Apply the seed migration twice | Second run succeeds; row counts unchanged |
| `employee-directory.UT24` | AC20 | E2 transferred to Finance / London / Finance Analyst, effective 2027-01-01 | Fields updated; `role_started_at` and `last_transfer_completed_at` = 2027-01-01; `joined_at` unchanged; returns `APPLIED` |
| `employee-directory.UT25` | AC21 | Same command invoked three times (simulating the OQ-13 retry policy) | First `APPLIED`; second and third `ALREADY_APPLIED`; employee row written exactly once |
| `employee-directory.UT26` | AC22 | Command naming the **inactive** Legacy R&D department | Rejected; employee unchanged; no `applied_transfers` row |
| `employee-directory.UT27` | AC22 | Command naming an unknown location uuid | Rejected; employee unchanged; no `applied_transfers` row |
| `employee-directory.UT28` | AC23 | Force a failure after the employee update, before the idempotency insert | Both rolled back; a subsequent call returns `APPLIED`, not `ALREADY_APPLIED` |
| `employee-directory.UT29` | AC24 | Apply a transfer for E2 effective 2027-01-01, then call `/employees/me` as E2 on 2026-12-01 | Response shows the **new** department, location and job position, not the old ones |

## Explicitly Out of Scope

- **Write APIs for reference data.** No create, update or delete endpoints for departments, locations or job positions, and no HTTP write surface of any kind. The BRD describes selection, never administration. The single exception is `applyTransfer`, which is **in-process only** and reachable solely by `orchestration` through the published interface (AMD-01 Option A).
- **An HR admin surface.** Not mentioned anywhere in the source document.
- **Synchronisation with an external HRIS.** OQ-05 ratified local ownership; an integration adapter was the rejected alternative.
- **Authentication, token issuing and validation.** This spec states the contract it *requires* of the auth spec (subject claim = email, authorities from `findAuthorities`) but implements none of it.
- **Eligibility evaluation and all date arithmetic.** This module returns raw attributes; `approval` applies ELIG-01 to ELIG-05 under an injectable `Clock`.
- **Exposing HR-sensitive attributes or authorities over HTTP.** In-process interface only (AC11).
- **Skip-level or delegate manager hierarchy.** OQ-14 grants direct-report scope only.
- **Audit logging of directory reads.** DP-04 mandates read auditing; the mechanism is owned by `audit-trail`. This module emits nothing until that exists.
- **DP-05 retention and purge of employee records.** DP-05 sets 7-year retention for the transfer journey. Whether it reaches directory records is unstated in the BRD and is **not** decided here.

## Non-Functional Constraints (from constitution.md)

| Constraint | Source | Applies here as |
|---|---|---|
| p95 < 500 ms (Tier 2) | § Non-Functional Baselines `[Provisional]` | All four HTTP endpoints |
| Coverage floor **80 percent (Critical)** | § Non-Functional Baselines `[Provisional]`, raised by C-09 | All module code; enforced at Gate 2 once JaCoCo lands |
| DP-01 — no PII in logs | § Security Posture | AC17 |
| DP-03 — read access restricted | § Security Posture | AC09, AC10, AC11 |
| Paginate all collection endpoints | § Java / Spring standards | AC04, AC05, AC06 |
| No N+1; index every FK and filtered column | § Java / Spring standards | AC18; data model indexes |
| Constructor injection; thin controllers; no field injection | § Java / Spring standards | Verified at Gate 2 |
| Schema by migration only (`ddl-auto: none`) | § Architectural Constraints | Blocked on the migration ADR |

> **Coverage tier — raised to Critical 80 percent.** `constitution.md` placed `employee-directory` in the Utility tier at 60 percent. Two things moved it: it stores the authority set and manager hierarchy behind every authorization decision (OQ-14, BRD-023), and the AMD-01 ruling gave it a state-changing operation on HR data. Recorded as consequence **C-09**. Gate 1 should confirm, and `constitution.md` needs the corresponding edit.

## Open Dependencies

| # | Dependency | Blocks | Status |
|---|---|---|---|
| — | ~~AMD-01~~ and ~~AMD-02~~ | — | ✅ **Both resolved 2026-09-19.** No BRD dependency remains. |
| 2 | **Migration tooling ADR** (Flyway vs Liquibase) | TDD GREEN and the seed migration — no table exists without it | `architecture.md` §7 item 1, **open** |
| 3 | **API versioning ADR** | Endpoint paths in API01 to API04 | `status.md` open item 7, **open** |
| 4 | **Gate 1 architecture approval** of the six-module proposal | Creating `modules/employee-directory/` on disk | `architecture.md` §9.7, **not approved** |
| 5 | **JaCoCo in the Maven build** | Enforcing the coverage floor at Gate 2 | BRD consequence C-06, **open** |
| 6 | **Gate 2 reviewer assignment** | Gate 2 entirely | `project_context.md`, **unassigned** |
| 7 | **Authentication spec** | AC09 and AC10 are testable against a mocked `SecurityContext`, but not end-to-end until JWT validation exists. The auth spec must honour the subject-claim contract above. | **Not written** |

## Traceability Summary

| BRD / Constraint | Acceptance Criteria | Tests |
|---|---|---|
| BRD-002 | AC01, AC04, AC05, AC06, AC12 | UT01, UT04–UT09, UT16 |
| BRD-003 | AC02, AC04, AC05, AC06, AC12 | UT02, UT04–UT09, UT16 |
| BRD-004 | AC03, AC04, AC05, AC06, AC12 | UT03, UT04–UT09, UT16 |
| BRD-013 (support) | AC15 | UT19 |
| BRD-023 (support) | AC10, AC13, AC14, AC16 | UT14, UT17, UT18, UT20 |
| BRD-014 | AC20, AC21, AC22, AC23, AC24 | UT24–UT29 |
| DP-01 / DP-03 | AC09, AC10, AC11, AC17 | UT13, UT14, UT15, UT21 |
| Constitution NFR | AC04, AC05, AC06, AC18 | UT04–UT09, UT22 |
| Seed integrity | AC07, AC19 | UT10, UT11, UT23 |
