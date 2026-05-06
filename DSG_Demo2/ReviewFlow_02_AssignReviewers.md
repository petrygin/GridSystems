# Task 02 — Assign Reviewers

**Scope:** Reviewer picker inside the Send for Review modal.
**Related:** Picker is part of the modal defined in Task 01. Eligibility rules and notifications are referenced here but defined in Overview / Task 03.

For shared terminology, eligibility rules, and reviewer statuses, see `ReviewFlow_00_Overview.md`.

---

## Goal

Let the author select reviewers when submitting a project for review. Picker is inline inside the Send for Review modal — no separate dialog, no dropdown.

---

## Layout

The picker is a self-contained block inside the Send for Review modal, placed between the Changes summary card and the Description field.

```
Reviewers                                       [N selected]   ← section header
┌──────────────────────────────────────────────────────────┐
│ 🔍  Search by name                                        │   ← search field
├──────────────────────────────────────────────────────────┤
│ ☐  [SC]  Sarah Chen                              Reviewer │
│         Zone A-12334, Zone B-1244                         │
│ ☑  [LF]  Liam Foster                             Reviewer │
│         Zone B-12344                                      │
│ ☑  [MB]  Maya Bennett                            Reviewer │
│         Zone C-1234                                       │
│ ☐  [EC]  Ethan Cole                                 Owner │
│ ☐  [AL]  Anna Lee                                Reviewer │
│         Global                                            │
└──────────────────────────────────────────────────────────┘
```

### Section header

| Element       | Behavior                                                                  |
|---------------|---------------------------------------------------------------------------|
| `Reviewers`   | Static label, h2-style (18px / weight 600)                                |
| `N selected`  | Pill on the right. Neutral background when 0, dark filled when ≥1         |

The `N selected` pill replaces a footer meta line — it's the single source of truth for selection count and is closer to the action.

### Search field

| Property      | Value                                                                     |
|---------------|---------------------------------------------------------------------------|
| Placeholder   | `Search by name`                                                          |
| Search target | Filters by `name` and `zones` text (case-insensitive substring match)     |
| Clear         | Empty string by manual deletion. No explicit clear (×) button in MVP      |

### Reviewer rows

Each row contains:

- **Checkbox** (left): unchecked = outline, checked = filled black square with white check
- **Avatar**: 32×32 circle with initials
- **Name** (primary, bold): user's display name
- **Zones** (secondary, muted): comma-separated zone IDs the user has reviewer rights in. Hidden line when zones value is empty (e.g., for Owner).
- **Role label** (right-aligned, muted): the user's eligibility role (`Reviewer`, `Owner`, etc.)

Click anywhere on a row toggles selection. The whole row is the click target, not just the checkbox.

---

## Sorting

**Alphabetical by name** — single, deterministic order. No special treatment for recent collaborators or self.

Rationale: predictable, no backend metric required, and Peter's design explicitly does not surface "recent collaborator" labels.

---

## Eligibility (recap from Overview)

A user appears in the picker if at least one is true:

- Has `Reviewer` role (or higher) in any zone the project touches
- Has global `Reviewer` or `Admin` role
- Is the project author / owner (self-review allowed)

Author appears in the list with role label `Owner` (not `Reviewer`). Selectable like anyone else.

---

## Zones format

| Case                              | Display                                  |
|-----------------------------------|------------------------------------------|
| One specific zone                 | `Zone B-12344`                           |
| Multiple zones                    | `Zone A-12334, Zone B-1244` (comma sep)  |
| Many zones (3+)                   | `Zone A-12334, Zone B-1244, +N more`     |
| Global reviewer                   | `Global`                                 |
| Owner (no zone scope)             | (zones line hidden entirely)             |

When the user is eligible across multiple zones, **only zones relevant to the current project** are shown — not all zones the user can review. This signals *why* they're eligible.

---

## Selection rules

- **Min**: 1 reviewer required. Send button disabled until ≥1 selected.
- **Max**: not enforced in MVP. No warning at any threshold.
- **Self**: allowed. No badge or special label.
- **Persistence**: selections live only within the modal session. Cancel = discarded. Resubmit after Cancel review = empty selection (Task 01 spec).

---

## Empty states

| Situation                                  | Message in the picker list area                          |
|--------------------------------------------|---------------------------------------------------------|
| No matches for current search              | `No reviewers match "<query>"`                          |
| No eligible reviewers exist in the system  | `No eligible reviewers. Contact admin to assign reviewer roles.` |

In both cases, the search field stays usable, the list area shows the message centered, and the Send button stays disabled (per min-1 rule).

---

## Out of scope (MVP)

- Recent collaborators sorting / labeling
- Workload indicators (review queue size per reviewer)
- Online / last active status
- Self-review badge or special treatment
- Selection count warnings (e.g., "5+ reviewers selected")
- Dropdown / combobox alternative to inline list
- Group sections (e.g., "Recent" vs "All eligible")
- Search beyond name + zones (no role filter, no email match)
- Inviting users not in the system
- Reviewer presets / saved groups

---

## Open questions

None for Task 02. All decisions resolved:

- Inline picker (not dropdown) ✓
- Alphabetical sort, no recent grouping ✓
- `N selected` pill in section header replaces footer meta ✓
- Layout: avatar + name + zones (left), role label (right) ✓
- Min 1 reviewer, no max enforcement ✓
- Self-review allowed without badge ✓
- Owner role appears with zone line hidden ✓
- Multi-zone format: relevant zones only, comma-separated ✓
- Empty states: two distinct messages for no-match vs no-eligibles ✓

---

## Acceptance criteria

- [ ] Picker is inline inside the Send for Review modal, between summary card and Description
- [ ] Section header shows `Reviewers` title + `N selected` pill on the right
- [ ] Pill has dark filled state when N ≥ 1
- [ ] Search field filters by name and zones (case-insensitive substring)
- [ ] List sorted alphabetically by name
- [ ] Each row: checkbox (left) + avatar + name + zones (multi-line) + role label (right)
- [ ] Zones line hidden when user has no zone scope (e.g., Owner)
- [ ] Multi-zone format: comma-separated; truncated with `+N more` for 3+ zones (when implemented)
- [ ] Whole row is a click target for selection toggle
- [ ] Send button disabled while 0 reviewers selected
- [ ] Self-selection allowed without badge or warning
- [ ] No-match empty state: `No reviewers match "<query>"`
- [ ] No-eligibles empty state: `No eligible reviewers. Contact admin…`
