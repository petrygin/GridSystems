# Review Flow — Epic Overview

Shared context for the 7-task epic covering the project review and merge flow.
Individual task specs reference this document for state machine, status popover pattern, and shared terminology.

---

## JTBD

When I finish working on a project branch, I want to submit it for review, have it validated and reviewed, and merged, so that approved changes become part of the main model.

---

## Branch state machine

A branch can be in one of 5 states:

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
| In review  | Approved   | `Approve` action                              | Reviewer          |
| Approved   | Draft      | `Reopen for review` action (post-approval edit)| Author           |
| Approved   | Merged     | `Merge to main` action                        | Author or reviewer with permission |
| Draft      | Archived   | `Archive branch` action                       | Author or admin   |
| Merged     | Archived   | `Archive branch` action                       | Author or admin   |

MVP keeps transitions linear. Complex flows (multi-approver, partial merges, branch-from-branch) are out of scope.

---

## Status popover pattern

The branch status pill in the header context (project / branch block) is the primary control surface for the entire review flow. Clicking it opens a **status popover** that combines context + details + actions for the current state.

### Common structure (all 5 states)

```
[State name]                    ← header
[Timestamp + actor]             ← who/when last changed state
─────────────────────────
[State-specific content]        ← reviewers, summary, message
─────────────────────────
[Actions]                       ← state-appropriate, role-filtered
```

### State-by-state content (summary)

| State      | Header section            | Body                                          | Actions (author)               | Actions (reviewer)             |
|------------|---------------------------|-----------------------------------------------|--------------------------------|--------------------------------|
| Draft      | Created + author          | Change summary (counters)                     | Send for review · Archive      | —                              |
| In review  | Sent + author             | Reviewer list with pending/approved status    | Cancel review                  | Approve · Request changes      |
| Approved   | Approved + reviewer       | Optional approval message                     | Merge to main · Reopen         | Merge to main                  |
| Merged     | Merged + actor            | Approved by + integration summary             | Archive                        | —                              |
| Archived   | Archived + actor          | —                                             | —                              | —                              |

Detailed popover specs live in their corresponding task documents.

### Visual treatment of the pill

| State      | Pill color cue                    |
|------------|-----------------------------------|
| Draft      | Neutral (default outline)         |
| In review  | `--warning` accent (amber)        |
| Approved   | `--success` accent (green)        |
| Merged     | `--info` accent (blue)            |
| Archived   | Muted / disabled                  |

Final tokens decided in Figma. Do not specify hex values in specs.

---

## Edit-mode coupling

Branch state controls availability of Edit mode:

| Branch state | Edit button                                   |
|--------------|-----------------------------------------------|
| Draft        | Enabled — `E` shortcut works                  |
| In review    | Disabled — tooltip: "Project is under review" |
| Approved     | Disabled — tooltip: "Project is approved"     |
| Merged       | Disabled — tooltip: "Branch is merged"        |
| Archived     | Disabled — tooltip: "Branch is archived"      |

Editing during review is not allowed. To edit, the author must `Cancel review` (returns branch to Draft) or `Reopen for review` from Approved.

---

## Reviewer eligibility

A user is eligible to be a reviewer if at least one is true:

- Has role `Reviewer` (or higher) in any zone the branch touches
- Has global role `Reviewer` or `Admin`

Self-review is permitted (small teams, testing). One approval is sufficient to move from `In review` to `Approved`. Detailed selection logic lives in Task 02.

---

## Merge permissions

From `Approved`, anyone with permission can merge: author, the approving reviewer, or an admin. First click wins. This avoids blocking flow when one party is unavailable.

---

## Validation policy

Validation errors **do not block** any transition in the MVP. Errors surface as warnings in the relevant modals (Send for review, Merge) but the user can always proceed. Rationale: real-world cases include junior engineers asking seniors to help resolve errors via the review process.

---

## Out of scope (MVP)

- Slash command `/review`
- Multiple required reviewers / weighted approvals
- Required vs optional reviewers
- Review priority / deadline / SLA fields
- Draft / published states for review requests
- Review comments threaded on individual components (general approve/reject only)
- Edit during active review (must cancel first)
- Branch-from-branch flows
- Partial merges (cherry-pick subsets of changes)
- Conflict resolution beyond keep-main / keep-project

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
| Branch            | A working copy of the project model, isolated from main until merged     |
| Main              | The canonical model. Updated only via merge from an approved branch      |
| Review request    | A formal submission of a branch for reviewer evaluation                  |
| Reviewer          | A user with rights to approve or request changes on a branch             |
| Approval          | A reviewer's accept signal. One approval is sufficient in MVP            |
| Merge             | The act of integrating an approved branch into main                      |
| Status pill       | The clickable badge in the header showing branch state                   |
| Status popover    | The popover opened from the status pill, combining info and actions      |
