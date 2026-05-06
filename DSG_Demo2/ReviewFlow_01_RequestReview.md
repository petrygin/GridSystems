# Task 01 — Request Review

**Scope:** Draft → In review transition. Author-side flow only.
**Related:** Reviewer picker details → Task 02. Reviewer notification → Task 03.

For state machine, status popover pattern, change counter format, reviewer statuses, and shared terminology, see `ReviewFlow_00_Overview.md`.

---

## Goal

Let the author submit the active project for review and lock it from further edits until the review concludes.

---

## Entry point

The action `Send for review` lives inside the **Draft status popover** in the header (see Overview → Status popover pattern). There is no separate primary button.

### Draft popover content

```
[Draft]                          ← outline pill, neutral border
Updated [relative time] by [author name]
12 changes                       ← large headline, neutral
─────────────────────────
[ Send for review ]              ← primary button, dark bg, full-width
[ Archive project ]              ← secondary button, outline, full-width
```

The headline uses the **headline-only** form of the change counter (see Overview → Change counter format). Example: `12 changes`. Breakdown is not shown here — it appears in the Send for Review modal.

When the project has zero changes, the headline reads `No changes vs master` and `Send for review` is disabled.

### Action availability

| Condition                                          | `Send for review` state                                       |
|----------------------------------------------------|---------------------------------------------------------------|
| Project has ≥1 change vs master                    | Enabled                                                       |
| Project has 0 changes vs master                    | Disabled, tooltip: "No changes to review"                     |
| Project has validation errors                      | Enabled (errors surface as inline warning inside the modal)   |
| User is not the author and has no write right      | Action hidden                                                 |

---

## Modal: Send for Review

Triggered by clicking `Send for review` in the Draft popover.

### Structure

```
┌─────────────────────────────────────────────────┐
│  Send for review                                │
│  [project name]                                 │
├─────────────────────────────────────────────────┤
│                                                 │
│  Changes summary                                │
│  ┌─────────────────────────────────────┐  │
│  │ 12 changes                                │  │   ← headline, neutral
│  │ 5 added · 4 modified · 3 removed          │  │   ← muted breakdown
│  └─────────────────────────────────────┘  │
│                                                 │
│  [⚠ inline validation warning, if applicable]   │
│                                                 │
│  Reviewers                                      │
│  [reviewer picker — see Task 02]                │
│                                                 │
│  Message (optional)                             │
│  [textarea, placeholder: "Add a note for        │
│   reviewers"]                                   │
│                                                 │
├─────────────────────────────────────────────────┤
│  N reviewer(s) will be notified  [Cancel] [Send for review] │
└─────────────────────────────────────────────────┘
```

The modal subtitle shows the project name (e.g. `GridSim West Upgrade`).

### Field details

| Field           | Required | Notes                                                                                       |
|-----------------|----------|---------------------------------------------------------------------------------------------|
| Changes summary | —        | Read-only. Headline `N changes` + muted breakdown `X added · Y modified · Z removed`. Computed from diff vs master. |
| Reviewers       | Yes      | Min 1, max not enforced in MVP. See Task 02 for picker behavior                            |
| Message         | No       | Plain textarea, no formatting. Persisted with the review request                           |

### Validation warning (inline, conditional)

If the project has unresolved hard errors, show an inline warning between the Changes summary card and the Reviewers field:

```
⚠ This project has [N] validation errors.
   You can still send it for review — reviewers will see the errors in review mode.
```

The warning is informational. It does not block submit.

### Footer

The footer has two parts:

- Left: meta text reflecting the current state. `Select at least 1 reviewer` when no reviewers are selected, otherwise `N reviewer(s) will be notified`
- Right: action buttons

| Button            | State                                                                |
|-------------------|----------------------------------------------------------------------|
| Cancel            | Always enabled. Closes modal, no state change.                       |
| Send for review   | Disabled until ≥1 reviewer is selected. Primary style.               |

---

## Post-action state

On successful submit:

1. Project state: `Draft` → `In review`
2. Status pill in header updates to amber outline + `In review`
3. Edit mode is locked. Visual cues (per Overview → Locked-state visual cues):
   - Banner above viewport: "Project is under review. Editing is disabled until the review is approved or cancelled."
   - Viewport toolbar: Add / Connect / Bind / Aggregate tools disabled
   - `E` shortcut and other edit affordances are no-ops
4. Status popover content updates to In review variant (see below)
5. All selected reviewers initialize with status `Pending`
6. Reviewers receive notification (Task 03)
7. Toast confirmation: `Review request sent to [N] reviewer(s)`

No modal dismissal screen. The pill state change in the header is the confirmation.

### In review popover content

```
[In review]                      ← outline pill, amber border
Sent [relative time] by [author name]
─────────────────────────
Reviewers
  [SC] Sarah Chen      Pending      (neutral)
  [LH] Liam Harper     Pending      (neutral)
  [MB] Maya Bennett    Pending      (neutral)
─────────────────────────
[ Cancel review ]                ← destructive button, red bg, full-width
```

For Task 01, all reviewer rows show `Pending`. Approved (green) and Declined (red) statuses are populated by reviewer actions in Task 04 / 05; Task 01 only renders them correctly when present.

---

## Cancel review (author)

While in `In review`, the author can revert the project to Draft via the In review popover action `Cancel review`.

### Confirmation

Lightweight confirmation in a small dialog:

```
Cancel review request?

Reviewers will be notified that the review was cancelled.
The project will return to Draft and become editable again.

[Keep review]  [Cancel review]
```

The `Cancel review` button is destructive-styled. The dialog is dismissible with ESC or backdrop click (treated as `Keep review`).

On confirm:

- Project state: `In review` → `Draft`
- Edit mode unlocks (banner removed, toolbar tools re-enabled)
- Reviewers receive notification (Task 03)
- Toast: `Review cancelled. Reviewers were notified.`
- The previously selected reviewer list is **not** retained for next submit — author picks again on resubmit

Rationale for not retaining: the act of cancelling implies meaningful change is coming, and reviewer relevance may shift.

---

## Edge cases

### No changes vs master

`Send for review` is disabled in the popover with tooltip `No changes to review`. The popover headline reads `No changes vs master` instead of a count. Author cannot reach the modal at all.

### No eligible reviewers

Handled in Task 02 (picker behavior). For Task 01: if the picker returns zero candidates, the modal still opens, the picker shows an empty state, and `Send for review` button stays disabled.

### User loses write permission mid-flow

If the author's role changes between opening the modal and clicking Send: submit fails with a toast `You no longer have permission to send this project for review`. Modal closes. No state change.

### Project is modified by another tab while modal is open

Out of scope for MVP. Last-write-wins. The summary counters reflect the diff at the moment the modal was opened.

### Author selects themselves as reviewer

Allowed. Author appears in the reviewer picker like any other eligible user. No special badge or treatment in MVP. Self-review is treated as a normal review for purposes of this task and downstream tasks.

---

## Open questions

None for Task 01. All major decisions resolved:

- Action lives in status popover, not a separate header button ✓
- Modal scope: changes summary + reviewers + optional message ✓
- Counter format: headline `N changes` + muted breakdown, neutral colors only ✓
- Validation errors do not block ✓
- Cancel review returns to Draft, no reviewer retention ✓
- Self-review permitted, no badge ✓
- No `Edit` button in header; lock is communicated via banner + toolbar state ✓
- Popover layout: small pill at top, meta line, headline/list, full-width buttons ✓
- Project (not branch) as the review unit ✓
- master, not main ✓

### Deferred to Task 04 / 05

- Per-reviewer status aggregation rule that drives In review → Approved transition
- UI for `Approve` / `Decline` reviewer actions inside review mode

---

## Acceptance criteria

- [ ] Draft popover shows `Send for review` action when project has ≥1 change vs master
- [ ] Action is disabled with tooltip when project has 0 changes
- [ ] Popover headline shows `N changes` (or `No changes vs master` for empty state)
- [ ] Popover layout: small outline pill at top, meta line, headline, divider, full-width buttons
- [ ] Modal summary card shows headline `N changes` + muted breakdown `X added · Y modified · Z removed`
- [ ] Counter colors are neutral (no green/blue/red on numbers)
- [ ] Validation warning appears inline when project has errors, does not block submit
- [ ] Submit button disabled until ≥1 reviewer is selected
- [ ] Footer meta text updates with reviewer count
- [ ] On submit: project state changes to In review, edit mode locks, toast appears, all reviewers initialize as Pending
- [ ] In review state shows banner above viewport and disables Add/Connect/Bind/Aggregate in toolbar
- [ ] In review popover renders reviewer list with per-reviewer status (Pending in MVP, Approved/Declined when present)
- [ ] Cancel review opens confirmation, then returns project to Draft
- [ ] Cancel triggers reviewer notification (Task 03 dependency)
