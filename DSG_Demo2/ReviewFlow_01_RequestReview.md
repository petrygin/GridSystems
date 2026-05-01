# Task 01 — Request Review

**Scope:** Draft → In review transition. Author-side flow only.
**Related:** Reviewer picker details → Task 02. Reviewer notification → Task 03.

For state machine, status popover pattern, change counter format, and shared terminology, see `ReviewFlow_00_Overview.md`.

---

## Goal

Let the author submit the active branch for review and lock it from further edits until the review concludes.

---

## Entry point

The action `Send for review` lives inside the **Draft status popover** in the header. There is no separate primary button.

### Draft popover content

```
Draft
Created [relative time] by [author name]
─────────────────────────
[N changes]                          →    ← clickable, opens Changes panel (Task 04)
─────────────────────────
▸ Send for review
▸ Archive branch
```

The summary line uses the **headline-only** form of the change counter (see Overview → Change counter format). Example: `12 changes`. Breakdown is not shown here — it appears in the Send for Review modal.

When the branch has zero changes, the line reads `No changes vs main` and `Send for review` is disabled.

### Action availability

| Condition                                          | `Send for review` state                                       |
|----------------------------------------------------|---------------------------------------------------------------|
| Branch has ≥1 change vs main                       | Enabled                                                       |
| Branch has 0 changes vs main                       | Disabled, tooltip: "No changes to review"                     |
| Branch has validation errors                       | Enabled (errors surface as inline warning inside the modal)   |
| User is not the branch author and has no write right| Action hidden                                                |

---

## Modal: Send for Review

Triggered by clicking `Send for review` in the Draft popover.

### Structure

```
┌─────────────────────────────────────────────────┐
│  Send for review                                │
│  [branch name]                                  │
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

### Field details

| Field           | Required | Notes                                                                                       |
|-----------------|----------|---------------------------------------------------------------------------------------------|
| Changes summary | —        | Read-only. Headline `N changes` + muted breakdown `X added · Y modified · Z removed`. Computed from diff vs main. |
| Reviewers       | Yes      | Min 1, max not enforced in MVP. See Task 02 for picker behavior                            |
| Message         | No       | Plain textarea, no formatting. Persisted with the review request                           |

### Validation warning (inline, conditional)

If the branch has unresolved hard errors, show an inline warning between the Changes summary card and the Reviewers field:

```
⚠ This branch has [N] validation errors.
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

1. Branch state: `Draft` → `In review`
2. Status pill in header updates color and label
3. Edit mode is locked. Visual cues (per Overview → Locked-state visual cues):
   - Banner above viewport: "Project is under review. Editing is disabled until the review is approved or cancelled."
   - Viewport toolbar: Add / Connect / Bind / Aggregate tools disabled
   - `E` shortcut and other edit affordances are no-ops
4. Status popover content updates to In review variant (see Overview, Task 02 for details)
5. Reviewers receive notification (Task 03)
6. Toast confirmation: `Review request sent to [N] reviewer(s)`

No modal dismissal screen. The state change in the header is the confirmation.

---

## Cancel review (author)

While in `In review`, the author can revert the branch to Draft via the In review popover action `Cancel review`.

### Confirmation

Lightweight confirmation in a small dialog:

```
Cancel review request?

Reviewers will be notified that the review was cancelled.
The branch will return to Draft and become editable again.

[Keep review]  [Cancel review]
```

The `Cancel review` button is destructive-styled. The dialog is dismissible with ESC or backdrop click (treated as `Keep review`).

On confirm:

- Branch state: `In review` → `Draft`
- Edit mode unlocks (banner removed, toolbar tools re-enabled)
- Reviewers receive notification (Task 03)
- Toast: `Review cancelled. Reviewers were notified.`
- The previously selected reviewer list is **not** retained for next submit — author picks again on resubmit

Rationale for not retaining: the act of cancelling implies meaningful change is coming, and reviewer relevance may shift.

---

## Edge cases

### No changes vs main

`Send for review` is disabled in the popover with tooltip `No changes to review`. The summary line reads `No changes vs main` instead of a count. Author cannot reach the modal at all.

### No eligible reviewers

Handled in Task 02 (picker behavior). For Task 01: if the picker returns zero candidates, the modal still opens, the picker shows an empty state, and `Send for review` button stays disabled.

### User loses write permission mid-flow

If the author's role changes between opening the modal and clicking Send: submit fails with a toast `You no longer have permission to send this branch for review`. Modal closes. No state change.

### Branch is modified by another tab while modal is open

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

---

## Acceptance criteria

- [ ] Draft popover shows `Send for review` action when branch has ≥1 change vs main
- [ ] Action is disabled with tooltip when branch has 0 changes
- [ ] Popover summary line shows `N changes` (or `No changes vs main` for empty state)
- [ ] Modal summary card shows headline `N changes` + muted breakdown `X added · Y modified · Z removed`
- [ ] Counter colors are neutral (no green/blue/red on numbers)
- [ ] Validation warning appears inline when branch has errors, does not block submit
- [ ] Submit button disabled until ≥1 reviewer is selected
- [ ] Footer meta text updates with reviewer count
- [ ] On submit: branch state changes to In review, edit mode locks, toast appears
- [ ] In review state shows banner above viewport and disables Add/Connect/Bind/Aggregate in toolbar
- [ ] Author can cancel review from In review popover, returning branch to Draft
- [ ] Cancel triggers reviewer notification (Task 03 dependency)
