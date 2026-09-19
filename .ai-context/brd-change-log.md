# BRD Change Log — EmployeeTransferPortal

Chronological record of every change to `.ai-context/BRD.md`: ingestions, requirement additions, revisions, removals, and scope changes. Each entry records the impact on existing specs so traceability from requirement to spec is never lost.

## Change Entries

| # | Date | BRD Version | Change Type | Req IDs Affected | Summary | Impacted Specs | Gate 0 Re-review |
|---|---|---|---|---|---|---|---|
| 1 | 2026-09-18 | 0.0.0 | Baseline Created | — | Empty BRD baseline established during project setup. No requirements supplied. | None | Not required |
| 2 | 2026-09-18 | 1.0 | Initial Ingestion | BRD-001 .. BRD-018 | Ingested `docs/Requirement for SDD.docx` (SDD Developer Assessment — Employee Internal Transfer Digital Journey). Extracted 18 functional requirements from document sections 2 and 3, 3 business rules, 5 assumptions, 6 out-of-scope items, 14 open questions and 6 document conflicts. No non-functional requirements exist in the source. Scope confirmed as Backend Only with the portal UI as an external consumer. Status set to Pending Review. | None — no specs exist yet | **Pending (Gate 0 in progress)** |
| 3 | 2026-09-18 | 1.1 | Requirement Added / Clarified | BRD-019 .. BRD-024 (new); BRD-001 .. BRD-018 (clarified) | Spec author proposed resolutions to all 14 open questions. Added 6 derived requirements: withdraw (BRD-019), manager decision (BRD-020), HR eligibility decision (BRD-021), audit trail (BRD-022), read authorization (BRD-023), retry and park (BRD-024). Recorded 7 consequences (C-01..C-07). None of it derives from the source document. Assumptions A-02 confirmed and A-05 superseded. `constitution.md` NFR and data-protection sections populated `[Provisional]`. Status remains Pending Review. | None — no specs exist yet | **Required — resolutions await ratification** |
| 4 | 2026-09-18 | 1.2 | Gate 0 Approved | BRD-001 .. BRD-024 | **Gate 0 APPROVED** by Vaibhaw Soni (vaibhaw.soni@intglobal.com) at 20:47:53. Gate 0 roster corrected from Soumyadeep Adhikary to Vaibhaw Soni — setup had inferred Gate 0 ownership without asking. Git repository initialised so `int-standards.md` §6 email verification could run; check PASSED. All 14 resolutions and 6 derived requirements ratified; A-03 confirmed; C-01..C-07 accepted as open work; DC-01..DC-06 acknowledged. `author != reviewer` WAIVED at Gate 0 only, logged in the review record. Spec generation UNBLOCKED. | None yet — specs may now be drafted | Complete |
| 5 | 2026-09-19 | 1.3 | Post-Approval Amendment Raised | BRD-014, ELIG-03 (OQ-01), OQ-03, OQ-04, OQ-05 | **AMD-01 raised** during author self-review of the `employee-directory` spec. Ratified resolutions OQ-03/OQ-04 (OrgDataPort as a stubbed external adapter) and OQ-05 (employee data owned locally) contradict each other. Consequences: BRD-014 has no write path and is unimplementable; ELIG-03 reads a column nothing can write, so it fails open permanently; the OQ-04 hard gate gives no protection. Three resolution options recorded (A: publish a narrow write operation — recommended; B: externalise org data; C: descope). No option selected. BRD status remains Approved with one open amendment. | `employee-directory` (open dependency #1) | **Required — Gate 0 ruling on AMD-01** |
| 6 | 2026-09-19 | 1.4 | Amendment Resolved | BRD-014, ELIG-03 (OQ-01), OQ-03, OQ-04, OQ-05 | **AMD-01 RESOLVED — Option A** by Vaibhaw Soni at 12:26:07. `employee-directory` publishes a narrow `applyTransfer` write operation. OQ-05 local ownership confirmed and unchanged; OQ-03 narrowed so org-data update is an in-process call rather than a stubbed external adapter (Payroll, IT, Facilities remain stubs); OQ-04 hard gate unchanged in shape but now protective; BRD-014 implementable; ELIG-03 functional. New consequences C-09 (coverage tier raised to Critical 80 percent), C-10 (opaque idempotency key, no FK), C-11 (AMD-02 raised). **AMD-02 raised and open** — effective-date timing, working assumption apply-immediately. | `employee-directory` rev 3 | Complete for AMD-01; **AMD-02 ruling still required** |
| 7 | 2026-09-19 | 1.5 | Amendment Resolved | BRD-014, ELIG-01 and ELIG-03 (OQ-01) | **AMD-02 RESOLVED — Option A** by Vaibhaw Soni at 12:37:43. The org-data change applies immediately on `applyTransfer`, ~30+ days before the OQ-06 effective date, keeping the OQ-04 fan-out supplied with the new department and location. Option B rejected (would leave the fan-out reading stale data); Option C rejected as disproportionate. Accepted consequence: the directory reports post-transfer values during that window, pinned by AC24. New consequence **C-12** — ELIG-03 must use an upper-bound comparison, never a between-range, or it fails open against the future `last_transfer_completed_at`. **All BRD amendments now closed.** | `employee-directory` rev 4; constraint carried to the future `approval` spec | Complete — no open amendments |

## Change Types

- **Baseline Created** — initial placeholder established at setup.
- **Initial Ingestion** — first BRD document ingested from `docs/`.
- **Requirement Added** — new requirement introduced.
- **Requirement Revised** — existing requirement changed; downstream specs must be reassessed.
- **Requirement Removed** — requirement withdrawn; dependent specs must be retired or revised.
- **Scope Change** — in-scope/out-of-scope boundary moved.

## Rules

1. This log is **append-only**. Never rewrite or delete an existing entry; correct it with a new entry.
2. Every BRD modification after initial ingestion requires a new row here.
3. A **Requirement Revised** or **Requirement Removed** entry against an approved requirement triggers Gate 0 re-review and reassessment of every impacted spec.
4. Changes reaching already-approved specs are processed through the Change Request workflow (`.ai-context/change_requests/CR-<YYYYMMDD>-<slug>.md`) and require Gate 1 re-approval.
