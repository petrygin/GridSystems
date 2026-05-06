# Review Flow — Epic Overview

Shared context for the 7-task epic covering the project review and merge flow.
Individual task specs reference this document for state machine, status popover pattern, and shared terminology.

---

## Documentation alignment notes

This epic was reconciled with platform documentation (StatusModel v2.0, UC09 Versioning, VersionControl Unified Model). Key alignment decisions for MVP:

- **Review unit is Project** (not branch, not task). When the user clicks Send for review, the system performs a **group operation** on all Active tasks within the project. Tasks are not surfaced in the MVP UI — the project appears as a single unit being reviewed.
- **`In review`, `Approved`, `Merged`, `Archived`** are part of the **default workflow configuration** that ships with the platform, not hardcoded system statuses. System statuses for Project are only Draft / Active / Inactive (per StatusModel). The MVP UI is designed around this default workflow; admin customization of the workflow is out of scope.
- **Branch concept exists under the hood** (sandbox/workspace per VersionControl) but is not surfaced in the MVP UI. One project = one project branch is the simplifying assumption.
- **Trunk is named `master`** per documentation (not `main`).
- **Validation gates**: per StatusModel, L0/L1 block commit, L2/L3 block approval, L4/L5 are advisory. Our "validation errors don't block Send for review" policy is consistent: send-for-review is the entry into pre-approval, not approval itself.

---

## JTBD

When I finish working on a project, I want to submit it for review, have it validated and reviewed, and merged, so that approved changes become part of the master model.

---

## Project state machine (default workflow)

A project can be in one of 5 states under the default workflow:

```
Draft ──► In review ──► Approved ──► Merged
  ▲          │              │
  │          ▼              ▼
  └──── (cancel/reject) ────┘

Archived — manual archive from Draft or Merged
```

### Transitions

| From       | To         | Trigger                                       | Who               |
|------------|------------|-----------------------------------------------|-------------------|
| Draft      | In review  | `Send for review` action                      | Author            |
| In review  | Draft      | `Cancel review` action                        | Author            |
| In review  | Draft      | `Request changes` action                      | Reviewer          |
| In review  | Approved   | All-reviewers signal resolves to approval     | System (see Task 04) |
| Approved   | Draft      | `Reopen for review` action (post-approval edit)| Author           |
| Approved   | Merged     | `Merge to master` action                      | Author or reviewer with permission |
| Draft      | Archived   | `Archive project` action                      | Author or admin   |
| Merged     | Archived   | `Archive project` action                      | Author or admin   |

MVP keeps transitions linear. Complex flows (multi-approver thresholds, partial merges, branch-from-branch) are out of scope.

> **Note on In review → Approved:** the exact rule for when the project transitions from In review to Approved depends on per-reviewer responses (see "Reviewer statuses" below). The detailed decision logic is the scope of Task 04 / 05. For Task 01 we only need to know that the transition is **not immediate** — the project stays in In review while reviewers respond.

---

## Status popover pattern

The project status pill in the header is the primary control surface for the entire review flow. Clicking it opens a **status popover** that combines context + details + actions for the current state.

### Common structure (all 5 states)

```
[State pill]                    ← small outline pill at the top
[Timestamp + actor]             ← muted meta line
[State-specific content]        ← headline / reviewer list / message
─────────────────────────
[Action buttons]                ← full-width primary/secondary, role-filtered
```

Actions in the popover are **full-width buttons** (not menu items with icons on the left). Primary actions use dark/black background; destructive actions (Cancel review) use red; secondary actions use outlined style.

### State-by-state content (summary)

| State      | Meta line                 | Body                                          | Primary action(s) (author)            | Reviewer actions               |
|------------|---------------------------|-----------------------------------------------|---------------------------------------|--------------------------------|
| Draft      | Updated [time] by author  | Headline `N changes`                          | Send for review · Archive project     | —                              |
| In review  | Sent [time] by author     | Reviewer list with per-reviewer statuses      | Cancel review *(red, destructive)*    | Approve · Decline              |
| Approved   | Approved [time] by reviewer | Optional approval message                   | Merge to master · Reopen              | Merge to master                |
| Merged     | Merged [time] by actor    | Approved by + integration summary             | Archive project                       | —                              |
| Archived   | Archived [time] by actor  | —                                             | —                                     | —                              |

Detailed popover specs live in their corresponding task documents.

### Visual treatment of the pill

| State      | Pill style                             |
|------------|----------------------------------------|
| Draft      | Outline, neutral border                |
| In review  | Outline, amber border (`--warning`)    |
| Approved   | Outline, green border (`--success`)    |
| Merged     | Outline, blue border (`--info`)        |
| Archived   | Outline, muted/grey                    |

Pills are outline-style (transparent background, colored border + text). No filled-background variants.

---

## Reviewer statuses

Each reviewer assigned to a review request has an independent status that evolves over the review cycle:

| Status    | Color              | Meaning                                                |
|-----------|--------------------|--------------------------------------------------------|
| Pending   | Neutral / muted    | Reviewer has not yet responded                         |
| Approved  | `--success` green  | Reviewer accepted the changes                          |
| Declined  | `--destructive` red| Reviewer rejected the changes (requires changes)       |

All reviewers start as `Pending` immediately after Send for review. They transition independently to `Approved` or `Declined` when they act in review mode (Task 04 / 05 scope).

> **Open question for Task 04:** the exact rule that aggregates per-reviewer statuses into the Project state transition (In review → Approved or back to Draft). Task 01 does not need to resolve this — it only needs to render the per-reviewer status correctly when the popover is opened.

---

## Edit-mode coupling

Project state controls whether Edit mode is available:

| Project state | Edit mode                                                |
|---------------|----------------------------------------------------------|
| Draft         | Available — enter via viewport toolbar or `E` shortcut   |
| In review     | Locked                                                   |
| Approved      | Locked                                                   |
| Merged        | Locked — terminal state                                  |
| Archived      | Locked — terminal state                                  |

When Edit mode is locked, all entry points (viewport toolbar, `E` key, context-menu affordances) are no-ops.

### Locked-state visual cues

When the project is in any locked state, the user sees:

- **Status pill** in the header reflects the state via outline color
- **Banner** above the viewport explains the lock, e.g. "Project is under review. Editing is disabled until the review is approved or cancelled." Banner text varies by state.
- **Viewport toolbar** has Add / Connect / Bind / Aggregate tools disabled

There is no separate `Edit` button in the header. Mode entry happens through the viewport toolbar only.

Editing during review is not allowed. To edit, the author must `Cancel review` (returns project to Draft) or `Reopen for review` from Approved.

---

## Change counter format

When the project summary needs to communicate "how much changed vs master", use this format consistently across popovers, modals, panel headers, and notifications:

- **Headline**: total count, neutral, e.g. `12 changes`
- **Breakdown** (where space allows, e.g. inside the Send for Review modal): muted secondary line, `X added · Y modified · Z removed`

The unit is `changes` (not `components`, `structures`, or any specific entity type) to avoid implying a single entity scope. The detailed per-entity breakdown lives in the Changes panel (Task 04).

**Empty state**: when the project has zero changes vs master, the summary line reads `No changes vs master`. Counters are not shown.

**Color**: counters are always rendered in neutral text. Do not color-code added/modified/removed (no green/blue/red treatment) — it competes visually with semantic colors used elsewhere (warnings, errors, success states, reviewer statuses).

---

## Reviewer eligibility

A user is eligible to be a reviewer if at least one is true:

- Has role `Reviewer` (or higher) in any zone the project touches
- Has global role `Reviewer` or `Admin`

Self-review is permitted (small teams, testing). Detailed selection logic lives in Task 02.

---

## Merge permissions

From `Approved`, anyone with permission can merge: author, an approving reviewer, or an admin. First click wins. This avoids blocking flow when one party is unavailable.

---

## Validation policy

Validation errors **do not block** Send for review in the MVP. Errors surface as warnings inside the Send for Review modal but the user can always proceed to In review.

This is consistent with platform validation gates (StatusModel): L0/L1 block commit; L2/L3 block approval; L4/L5 are advisory. Send-for-review is entry into pre-approval, not approval itself, so validation does not gate it.

**Rationale:** real-world cases include junior engineers asking seniors to help resolve errors via the review process.

---

## Out of scope (MVP)

- Slash command `/review`
- Multiple required reviewers / weighted approvals
- Required vs optional reviewers
- Review priority / deadline / SLA fields
- Draft / published states for review requests
- Review comments threaded on individual components (general approve/decline only)
- Edit during active review (must cancel first)
- Branch-from-branch flows
- Partial merges (cherry-pick subsets of changes)
- Conflict resolution beyond keep-master / keep-project
- Color-coded change counters (kept neutral, see Change counter format)
- Task-level review granularity (review unit is Project, not Task)
- Workflow customization (admin app scope)

---

## File map

| File                                  | Scope                                              |
|---------------------------------------|----------------------------------------------------|
| `ReviewFlow_00_Overview.md`           | This file. Shared context.                         |
| `ReviewFlow_01_RequestReview.md`      | Draft → In review (author flow)                    |
| `ReviewFlow_02_AssignReviewers.md`    | Reviewer picker inside the request modal           |
| `ReviewFlow_03_Notifications.md`      | Reviewer notifications and entry into review       |
| `ReviewFlow_04_ReviewMode.md`         | Read-only review experience + Changes panel        |
| `ReviewFlow_05_Validation.md`         | Validation step inside review mode                 |
| `ReviewFlow_06_Merge.md`              | Approved → Merged + conflict resolution            |
| `ReviewFlow_07_PostMerge.md`          | Post-merge state, author notification, archive     |

---

## Terminology

| Term              | Meaning                                                                  |
|-------------------|--------------------------------------------------------------------------|
| Project           | The review/merge unit in the UI. Internally a sandbox branch holding tasks |
| Master            | The canonical model trunk. Updated only via merge from an approved project |
| Review request    | A formal submission of a project for reviewer evaluation                 |
| Reviewer          | A user with rights to approve or decline a review request                |
| Approval          | A reviewer's accept signal. Per-reviewer; aggregation rule per Task 04   |
| Decline           | A reviewer's reject signal. Returns the project to Draft (Task 04 / 05)  |
| Merge             | The act of integrating an approved project into master                   |
| Status pill       | The clickable badge in the header showing project state                  |
| Status popover    | The popover opened from the status pill, combining info and actions      |
| Change            | Any tracked delta vs master: added, modified, or removed entity          |
| Default workflow  | The baseline workflow configuration shipped with the platform            |
