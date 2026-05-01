# Task 01 — Request Review

**Scope:** Draft → In review transition. Author-side flow only.
**Related:** Reviewer picker details → Task 02. Reviewer notification → Task 03.

For state machine, status popover pattern, and shared terminology, see `ReviewFlow_00_Overview.md`.

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
[X added · Y modified · Z removed]   ← clickable, opens Changes panel (out of scope here)
─────────────────────────
▸ Send for review
▸ Archive branch
```

The summary line uses three counters separated by `·`. Zero-counters are still shown for transparency (e.g. `0 added · 4 modified · 0 removed`).

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
│  Send "[branch name]" for review                │
├─────────────────────────────────────────────────┤
│                                                 │
│  Summary                                        │
│  5 added · 4 modified · 3 removed               │
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
│                          [Cancel] [Send for review] │
└─────────────────────────────────────────────────┘
```

### Field details

| Field           | Required | Notes                                                                |
|-----------------|----------|----------------------------------------------------------------------|
| Summary         | —        | Read-only counters from diff vs main                                 |
| Reviewers       | Yes      | Min 1, max not enforced in MVP. See Task 02 for picker behavior      |
| Message         | No       | Plain textarea, no formatting. Persisted with the review request     |

### Validation warning (inline, conditional)

If the branch has unresolved hard errors, show an inline warning between Summary and Reviewers:

```
⚠ This branch has [N] validation errors.
   You can still send it for review.
```

The warning is informational. It does not block submit.

### Footer buttons

| Button            | State                                                                |
|-------------------|----------------------------------------------------------------------|
| Cancel            | Always enabled. Closes modal, no state change.                       |
| Send for review   | Disabled until ≥1 reviewer is selected. Primary style.               |

---

## Post-action state

On successful submit:

1. Branch state: `Draft` → `In review`
2. Status pill in header updates color and label
3. Edit mode is locked:
   - Toolbar switches to View mode
   - Edit button disabled, tooltip: "Project is under review"
   - `E` keyboard shortcut is a no-op
   - Add / Connect / Bind / Aggregate flows unavailable
4. Status popover content updates to In review variant (see Overview, Task 02 for details)
5. Reviewers receive notification (Task 03)
6. Toast confirmation: `Review request sent to [N] reviewer(s)`

No modal dismissal screen. The state change in the header is the confirmation.

---

## Cancel review (author)

While in `In review`, the author can revert the branch to Draft via the In review popover action `Cancel review`.

### Confirmation

Lightweight confirmation in a sub-popover or small dialog:

```
Cancel review request?
Reviewers will be notified that the review was cancelled.

[Keep review]  [Cancel review]
```

On confirm:

- Branch state: `In review` → `Draft`
- Edit mode unlocks
- Reviewers receive notification (Task 03)
- The previously selected reviewer list is **not** retained for next submit — author picks again on resubmit

Rationale for not retaining: the act of cancelling implies meaningful change is coming, and reviewer relevance may shift.

---

## Edge cases

### No changes vs main

`Send for review` is disabled in the popover with tooltip `No changes to review`. Author cannot reach the modal at all.

### No eligible reviewers

Handled in Task 02 (picker behavior). For Task 01: if the picker returns zero candidates, the modal still opens, the picker shows an empty state, and `Send for review` button stays disabled.

### User loses write permission mid-flow

If the author's role changes between opening the modal and clicking Send: submit fails with a toast `You no longer have permission to send this branch for review`. Modal closes. No state change.

### Branch is modified by another tab while modal is open

Out of scope for MVP. Last-write-wins. The summary counters reflect the diff at the moment the modal was opened.

### Author is a valid reviewer

Author appears in the reviewer picker and may select themselves. Self-review is allowed. No special bedge or treatment in MVP.

---

## Open questions

None for Task 01. All major decisions resolved:

- Action lives in status popover, not a separate header button ✓
- Modal scope: counters + reviewers + optional message ✓
- Validation errors do not block ✓
- Cancel review returns to Draft, no reviewer retention ✓
- Self-review permitted, no badge ✓

---

## Acceptance criteria

- [ ] Draft popover shows `Send for review` action when branch has ≥1 change vs main
- [ ] Action is disabled with tooltip when branch has 0 changes
- [ ] Modal shows accurate `X added · Y modified · Z removed` counters
- [ ] Validation warning appears inline when branch has errors, does not block submit
- [ ] Submit button disabled until ≥1 reviewer is selected
- [ ] On submit: branch state changes to In review, edit mode locks, toast appears
- [ ] Author can cancel review from In review popover, returning branch to Draft
- [ ] Cancel triggers reviewer notification (Task 03 dependency)
