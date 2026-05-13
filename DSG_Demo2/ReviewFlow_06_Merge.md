# Task 06 — Merge

**Scope:** Merging an Approved project into master. Conflict detection signals, the dedicated Conflicts Panel for per-component resolution, and the confirm step before master is modified.

**Related:** Approved-state status popover and Changes Panel slot → Task 04. Validation independence from Merge → Task 05. Post-merge state and notifications → Task 07.

**Design:** TBD (Figma frames to be linked)

For shared terminology and state model, see `ReviewFlow_00_Overview.md`. For the Changes Panel layout, header toggle pattern, and status pill popover, see `ReviewFlow_04_ReviewMode.md`.

---

## Goal

Allow the author or any approved reviewer to merge an Approved project into master with explicit, per-component handling of conflicts against the current master state. The reviewer or author carries the merge decision; the system enforces only one rule — conflicts must be resolved before the merge button activates.

---

## Architectural framing

Merge extends the project lifecycle past Approved. The status popover gains a new primary action (`Merge to master`), and a new left-sidebar surface — the **Conflicts Panel** — handles the resolution workspace.

The Conflicts Panel and the existing Changes Panel are **two independent panels** sharing the same left-sidebar slot:

- **Changes Panel** — project-vs-base diff (what this project introduced). Already defined in Task 04.
- **Conflicts Panel** — project-vs-current-master diff (where this project disagrees with what is in master right now). New in Task 06.

Each panel has its own toggle in the project header (`10 changes`, `● 5 conflicts`). The red dot on the conflicts indicator marks the non-zero state. Toggles are independent — opening one closes the other in the shared slot.

Conflict detection runs in the background and surfaces in three places:

1. **Reviews list** — `Sync` column (`Current` / `N conflicts`) ahead of opening the project
2. **Project header** — `● N conflicts` indicator with count
3. **Conflicts Panel** — full resolution workspace

---

## Entry point and Merge action

Available users:

- Project author
- Any reviewer who approved the project

The Merge action lives in two surfaces, both with identical behavior:

1. **Status pill popover** in Approved state — `[Merge to master]` primary
2. **Conflicts Panel footer** (when panel is open) — `[Merge to master]` primary

Both buttons are disabled while unresolved conflicts exist. Disabled state shows a tooltip: `Resolve N conflicts before merging`. Both open the same confirm dialog.

When no conflicts exist, `[Merge to master]` is enabled in the popover immediately, and the Conflicts Panel does not surface.

---

## Conflicts Panel

A 320px left-sidebar panel, same slot pattern as the Changes Panel.

### When it opens

- **Default-open** when the project enters Approved state with at least one conflict, on the user's next view of the project
- **Manually toggled** via the `● N conflicts` button in the project header
- **Hidden** when zero conflicts exist; the header indicator is also hidden

### Header

```
Conflicts (5)
2 resolved · 3 to go
```

Live counter that updates as the user resolves conflicts.

### Conflict row — unresolved

Expanded by default. Each row shows component identification, both diverging values, and the two resolution options.

```
⚠ Pole #45                  ⌖
Main:     voltage 230V
Project:  voltage 240V
[Keep main]  [Keep project]
```

The `⌖` icon focuses the component on Map and SLD (cross-highlight with viewport centering, plus Info Panel in read-only — identical to the cross-highlight pattern from Validation and Changes panels).

**Hover** on the row reveals an attribution line for both sides — who modified each side and when. Hidden by default to keep the row compact; appears on hover only.

### Conflict row — resolved

Collapses to a one-liner once the user picks a side:

```
✓ Pole #45 — Keep main      [Change]
```

The `[Change]` link re-expands the row to allow switching the choice. Resolved rows keep their position in the list; no reordering.

### Footer

```
[Merge to master]
```

Primary button. Disabled until all conflicts resolved. Mirrors the popover action.

### Empty state

When the panel opens with zero conflicts — edge case where conflicts were resolved server-side after the user opened the panel — shows: `No conflicts. Ready to merge.` with `[Merge to master]` enabled.

---

## Conflict resolution model

A conflict exists when a single component is modified in both the project branch and master, relative to the project's base point.

Resolution scope is **per-component**, not per-attribute. Even if a component has multiple diverging attributes, the user makes a single choice for the component as a whole.

- `Keep main` → on merge, the project's changes to this component are discarded; the component in master remains as-is
- `Keep project` → on merge, master's changes to this component are overwritten; the project's version wins

Other cases not surfaced as conflicts:

- Project modified, master unchanged → project changes apply silently
- Project unchanged, master modified → master version remains
- Both unchanged → component skipped

Deletion cases (component removed in one branch while the other modified it) are surfaced as conflicts with the same `Keep main` / `Keep project` choice. Keeping the deletion-side removes the component; keeping the modification-side preserves it with the modified attributes.

---

## Confirm dialog

Triggered by `[Merge to master]` from either surface when conflicts are resolved or when none existed. The dialog appears every time.

Content:

```
Merge to master

12 changes will be applied.
2 conflicts resolved (1 Keep main, 1 Keep project).

[Cancel]  [Merge]
```

The second line is omitted when there were no conflicts.

Confirming triggers the merge; project state, notifications, and master propagation are handled in Task 07.

---

## Validation relationship

Validation severity does not gate Merge. Errors and warnings continue to surface via the Validation Panel and the `Validation` column on the Reviews list, but they do not disable the Merge button. The author or reviewer carries the decision, consistent with the Approve gate from Task 05.

---

## Out of scope

- Per-attribute conflict resolution
- Three-way merge UI (visual diff of base / main / project values for the same attribute)
- Audit log of resolution choices — data persistence is implied for Task 07's post-merge history
- Locking when two users open the same Approved project simultaneously — last-writer-wins for MVP; second user's attempt shows an `Already merged` toast
- Mid-session recomputation of conflicts — state at panel open is state at merge confirm
- Re-running validation as part of the merge action
- Cancel/withdraw approval — the project moves linearly Approved → Merged in MVP

---

## Forward references

- **Task 07**: Post-merge transitions — project status moves to Merged, author receives a notification, master model picks up the merged changes, resolution choices recorded in project history

---

## Open questions

None for Task 06 — all decisions resolved in discussion.
