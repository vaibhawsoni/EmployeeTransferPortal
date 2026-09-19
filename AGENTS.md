# AGENTS.md — INT AI-First Engineering Policy

**Project:** EmployeeTransferPortal
**Scope:** This file governs every human and AI agent operating inside this repository.
**Vendor Neutrality:** This policy is tool-agnostic. It applies identically to any assistant (Claude, Gemini, Cursor, Windsurf, Copilot, or none at all). No repository behaviour may depend on a cloud-hosted or per-user AI configuration.

---

## 1. Authority Hierarchy

Rules are resolved top-down. A lower level may add detail but may never contradict a higher level.

| Priority | Source | Authority |
|---|---|---|
| 1 | `AGENTS.md` (this file) | Repository governance and policy |
| 2 | `.agent/rules/` | INT organizational standards — **immutable Control Plane** |
| 3 | `.agent/workflows/` | INT organizational workflows — **immutable Control Plane** |
| 4 | `.ai-context/constitution.md` | Project-specific non-negotiable constraints |
| 5 | `.ai-context/architecture.md` | Project architecture and module boundaries |
| 6 | `.agents/skills/<skill>/SKILL.md` | Local project skills |
| 7 | Global skills (e.g. `~/.gemini/config/skills/`, `~/.claude/skills/`) | Fallback only |

### 1.1 Skill & Governance Resolution — Local First, Global Second

1. **Priority 1 — Local Repository First.** Always check for `AGENTS.md` and `.agents/skills/<skill_name>/SKILL.md` in this repository root first. If present, load and execute the **local** version.
2. **Priority 2 — Global Fallback Second.** Fall back to a global skill **only** when the requested skill is absent from `.agents/skills/`.

### 1.2 Control Plane Immutability

`.agent/` is the INT organizational Control Plane. It MUST NOT be edited, renamed, summarized, rewritten, or extended from inside this project. Project-specific standards belong in `.ai-context/constitution.md`.

---

## 2. SDD Lifecycle

Every change flows through this sequence. Gates are blocking.

```
BRD  ->  Gate 0  ->  Spec  ->  Gate 1  ->  Plan  ->  Tasks  ->  Test Cases
                                            |
                                            v
                              TDD RED (tests/)  ->  TDD GREEN (src/)
                                            |
                                            v
                          Test Verification  ->  Gate 2  ->  Release
```

| Gate | Name | Evaluates | Record |
|---|---|---|---|
| Gate 0 | BRD Review | BRD completeness, scope boundaries, business rules | `.ai-context/pr_reviews/BRD-<timestamp>.md` |
| Gate 1 | Spec Peer Review | Spec alignment, API contracts, acceptance criteria | `.ai-context/pr_reviews/GATE1-<slug>-<timestamp>.md` |
| Gate 2 | Code Review | Implementation vs approved spec, quality, security, coverage | `.ai-context/pr_reviews/GATE2-<slug>-<timestamp>.md` |

### 2.1 Blocking Rules

- **Pre-Gate 0 spec block.** Drafting or generating any `.spec.md` is **STRICTLY PROHIBITED** until `.ai-context/BRD.md` holds `Approved` status via Gate 0. The agent MUST halt and end its turn upon presenting the BRD for Gate 0 review.
- **Gate 1 rejection block.** If a spec is `Rejected` or `Changes Requested` at Gate 1, all planning, task generation, test-case drafting, and implementation in `src/` are blocked until the spec is revised and re-approved.
- **Parallel spec execution.** Once Gate 0 is approved, specs progress independently and non-blockingly of one another.
- **Scope restriction.** Do not modify workflows before Spec Generation or after Gate 2 approval.

### 2.2 Reviewer Identity Enforcement

Approval rights are verified by **Git email only**. The authenticated `git config user.email` MUST match the reviewer roster in `.ai-context/project_context.md`. User *name* is not evaluated. On mismatch, reviewing and approving are blocked immediately. Pulling code does not grant approval rights; role separation is strictly enforced.

### 2.3 Change Request Trigger

The formal Change Request workflow (spec revision plus Gate 1 re-approval) is triggered **only** when a prompt explicitly contains "Change Request" or "CR". All other prompts are handled as development-related fixes under the active approved spec.

---

## 3. Repository Artifact Rules

### 3.1 Flat File Structure — No Feature Subdirectories

Artifacts sit **directly** at the root of their category folder. Creating a subdirectory named after a feature slug is prohibited.

| Artifact | Path |
|---|---|
| Spec | `.ai-context/specs/<feature-slug>.spec.md` |
| Plan | `.ai-context/plans/<feature-slug>.plan.md` |
| Tasks | `.ai-context/tasks/<feature-slug>.tasks.md` |
| Test cases | `.ai-context/test_cases/<feature-slug>.test_cases.md` |
| Decision | `.ai-context/decisions/ADR-NNN.md` |
| Incident | `.ai-context/incidents/INC-YYYY-NNN.md` |
| Hotfix | `.ai-context/hotfixes/HOTFIX-<slug>.md` |
| Release | `.ai-context/releases/RELEASE-vX.Y.Z.md` |
| Change request | `.ai-context/change_requests/CR-<YYYYMMDD>-<slug>.md` |

Every artifact is instantiated from its template in `.ai-context/templates/`.

### 3.2 Repository-Relative Paths Only

All path references written into repository artifacts MUST be relative to the repository root.

- Correct: `.ai-context/specs/transfer-request.spec.md`, `src/backend/src/main/java/com/intglobal/etp/...`
- Prohibited: any absolute local path (`C:\Users\...`, `/home/user/...`, `file:///...`).

Absolute local paths break for other collaborators and CI/CD.

### 3.3 Append-Only Audit Log

`.ai-context/prompt_history.md` is a **strictly append-only** chronological log. Full-file overwrite is prohibited. Read existing content first, then append the new entry beneath all previous entries, in the format defined by `.agent/rules/auto-log.md`:

```
### [YYYY-MM-DD HH:MM]
**User Request:** (one-sentence summary of the prompt)
**Agent Action:** (what was done, files modified, solution provided)
```

### 3.4 Test Artifact Separation

- `.ai-context/test_cases/` — test-case specifications and scenarios. **Not** executable code.
- `tests/backend/` — executable automated test suites and integration tests.
- `src/backend/src/test/java/` — Maven-resolved unit tests compiled by the build.

---

## 4. Engineering Guardrails

- Never hardcode secrets, credentials, connection strings, or API keys. Read them from environment variables.
- Validate and sanitize all incoming payload data.
- Never swallow errors; log through the project's standard logger.
- Never include comments stating that code was AI-generated.
- Never write commit messages that hint at AI generation.
- Do not add AI-specific configuration files outside `.agent/` and `.agents/`.
- Language-specific engineering standards for this project are defined in `.ai-context/constitution.md`.

---

## 5. Entry Points

| Intent | Skill |
|---|---|
| Initialize or re-sync the project baseline | `int-project-setup` |
| Ingest or update the BRD, generate module structure | `int-brd-ingestion` |
| Feature development, Gate 1 / Gate 2 lifecycle | `int-sdd-lifecycle` |
| Report and triage a production incident | `int-incident-management` |
| Emergency production fix | `int-hotfix-management` |
| Cut a release | `int-release-management` |
| Resume work after a session restart | `int-session-continuation` |
