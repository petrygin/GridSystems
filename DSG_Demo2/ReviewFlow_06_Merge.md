# Task 06 — Merge

**Scope:** Merging an Approved project into master. Conflict surfacing across Draft and Approved, the dedicated Conflicts Panel for per-component resolution, `Pull from master` in Draft, the resolve choices `Keep master` / `Keep project` in Approved, and the confirm step before master is modified.

**Related:** Approved-state status popover and Changes Panel slot → Task 04. Validation independence from Merge → Task 05. Post-merge state and notifications → Task 07.

**Design:** [Figma — Conflicts Panel and Merge dialogs](https://www.figma.com/design/0FW49cYCouwjKw3YzqQwcI/Grid-Model-Work-in-Progress?node-id=3397-141265)

For shared terminology and state model, see `ReviewFlow_00_Overview.md`. For the Changes Panel layout, header toggle pattern, and status pill popover, see `ReviewFlow_04_ReviewMode.md`.

---

## Goal

Allow the author or any approved reviewer to merge an Approved project into master with explicit, per-component handling of conflicts against the current master state. The reviewer or author carries the merge decision; the system enforces only one rule — conflicts must be resolved before the merge button activates.

In Draft, the author can pull master values into their project ahead of time to clear conflicts that have arisen while they worked.

---

## Architectural framing

Merge extends the project lifecycle past Approved. The status popover gains a new primary action (`Merge to master`), and a new left-sidebar surface — the **Conflicts Panel** — handles the resolution workspace.

The Conflicts Panel and the existing Changes Panel are **two independent panels** sharing the same left-sidebar slot:

- **Changes Panel** — project-vs-base diff (what this project introduced). Already defined in Task 04.
- **Conflicts Panel** — project-vs-current-master diff (where this project disagrees with what is in master right now). New in Task 06.

Each panel has its own toggle in the project header (`10 changes`, `● 5 conflicts`). The red dot on the conflicts indicator marks the non-zero state. Toggles are independent — opening one closes the other in the shared slot.

Conflicts are not the same as changes. A change is anything the project edits relative to its base point. A conflict is the **subset** where master ALSO edited the same component since that base point. Components touched only by the project are changes; components touched only by master are silent updates; components touched by both are conflicts.

Conflict detection runs in the background and surfaces in three places:

1. **Reviews list** — `Sync` column (`Current` / `N conflicts`) ahead of opening the project
2. **Project header** — `● N conflicts` indicator with count
3. **Conflicts Panel** — full resolution workspace

---

## Conflicts Panel

A 320px left-sidebar panel, same slot pattern as the Changes Panel. Title: `Conflicts vs master`. Red badge in the panel header shows the unresolved count (hidden when zero).

The panel is available in both **Draft** and **Approved** phases — same UI shell, different action set per phase.

### Open and close behavior

- **Auto-opens** on first view of the project in Approved state when at least one conflict exists
- **In Draft**, the panel does not auto-open; user toggles it manually via the `● N conflicts` header chip
- **User can close** the panel at any time via its close control or the header toggle
- **User can reopen** the panel anytime via the header toggle
- **Hidden completely** when zero conflicts exist; the header indicator is also hidden
- Opening the Conflicts Panel closes the Changes Panel and vice versa — shared sidebar slot

### Header

```
Conflicts vs master    [3]   ✕
1/3 resolved
```

The numeric badge tracks unresolved conflicts (3 in the example). The subtext line tracks resolution progress in `N/M resolved` form.

### Conflict card

Each conflict surfaces as a card with a light grey background, component identification, the side-by-side `PROJECT` / `MASTER` value comparison, and phase-specific action buttons.

```
─────────────────────────────────────
⎓ TR-131                  Transformer
─────────────────────────────────────
PROJECT          MASTER
Voltage          Voltage
230V             240V
Name             Name
North            North Sub
─────────────────────────────────────
        Show all differences
─────────────────────────────────────
[ Pull from Master ]   ← Draft
or
[ Keep Project ] [ Keep Master ]   ← Approved
─────────────────────────────────────
```

By default the card shows the first **2 attributes** in each column. If the component has more divergent attributes, `Show all differences` reveals the rest inline (the component-type label on the right stays in place).

The card itself is the affordance — clicking anywhere on the card body locates the component on Map and SLD (cross-highlight with viewport centering) and opens the Info Panel in read-only. The Info Panel surfaces full per-component history including who changed what and when. No attribution is shown in the Conflicts Panel itself — it stays compact.

### Phase-specific actions

**Draft — `Pull from Master`** (single primary action per card)

Clicking `Pull from Master` immediately replaces the project's value for that component with master's value. The card collapses to a resolved one-liner (see below). The change is reflected in the project right away — the project model is editable in Draft.

**Approved — `Keep Project` / `Keep Master`** (two equal-weight actions per card)

Both buttons render identically, no primary bias. Clicking records the resolution choice and collapses the card to a resolved one-liner. The project model itself does not change yet (Approved is locked); the choice is applied at merge confirm. Resolution can be reverted any time before merge via the revert icon on the collapsed card.

### Resolved card

Once a card is resolved, it collapses to a one-line summary that keeps its position in the list (no reordering).

```
⎓ TR-132              Transformer
Pulled from Master              ← Draft
or
Kept from Master           ↶    ← Approved (with revert icon)
```

Draft resolved cards have no revert affordance — the undo path is via the toast shown right after the Pull (see Pull from master section below).

Approved resolved cards have a revert icon on the right; clicking it re-expands the card to its unresolved state with `Keep Project` / `Keep Master` available again. Revert is unlimited before merge confirm.

### Footer

- **Draft**: `[ Pull all from Master ]` primary button at the bottom of the panel
- **Approved**: empty (Merge action lives in the status popover)

### Empty state

When the panel opens with zero conflicts — for example after the last conflict is resolved — the body shows a green check, the title `No conflicts`, and the subtitle `Project is in sync with master model`. No footer button.

---

## Pull from master (Draft)

Pull replaces the project's value for the chosen component with master's. It is a real edit to the project model, not a recorded decision.

### Per-card Pull

Click `Pull from Master` on a card → project model updates immediately → card collapses to `Pulled from Master` → toast appears for 8 seconds:

```
✓  Pulled TR-131 from master              [Undo]
```

Clicking `Undo` within 8 seconds restores the project's previous value for that component and re-expands the card to its unresolved state. After 8 seconds the toast disappears; later changes are made by manually re-editing the component in Edit mode.

### Pull all from master (bulk)

The `Pull all from Master` primary button in the Draft footer applies Pull to every unresolved conflict at once. Because this is destructive at scale, it opens a confirm dialog before any change:

```
Pull all from master

Replace your project changes with master values for 15 components.
This action cannot be undone.

                                       [Cancel]   [Pull all]
```

Confirming applies master values across all listed components and clears them from the unresolved set. The bulk action does not show a per-card toast — it shows a single summary toast: `Pulled 15 components from master` (no Undo on the bulk action).

### Why Pull only exists in Draft

In Approved the project model is locked (Task 04). Pull is a project-model edit; the lock makes it unavailable. The Approved equivalent is `Keep master`, which records the same intent (master wins for this component) as a deferred decision applied at merge confirm.

---

## Conflict resolution model (Approved)

A conflict exists when a single component is modified in both the project branch and master, relative to the project's base point.

Resolution scope is **per-component**, not per-attribute. Even if a component has multiple diverging attributes, the user makes a single choice for the component as a whole.

- `Keep Master` → on merge, the project's changes to this component are discarded; the component in master remains as-is
- `Keep Project` → on merge, master's changes to this component are overwritten; the project's version wins

Other cases not surfaced as conflicts:

- Project modified, master unchanged → project changes apply silently
- Project unchanged, master modified → master version remains
- Both unchanged → component skipped

Deletion cases (component removed in one branch while the other modified it) are surfaced as conflicts with the same `Keep Master` / `Keep Project` choice. Keeping the deletion-side removes the component; keeping the modification-side preserves it with the modified attributes.

---

## Entry point and Merge action

Available users:

- Project author
- Any reviewer who approved the project

The Merge action lives in the **status pill popover** in Approved state — `[Merge to master]` primary. The button is disabled while unresolved conflicts exist; disabled state shows a tooltip: `Resolve N conflicts before merging`.

When no conflicts exist, `[Merge to master]` is enabled in the popover immediately, and the Conflicts Panel does not surface.

---

## Confirm dialog

Triggered by `[Merge to master]` when conflicts are resolved or when none existed. The dialog appears every time.

### With conflicts and validation errors

```
Merge to master                                          ✕

⚠ This project has 3 validation errors

────────────────────────────────────────────────
12 changes
5 added · 4 modified · 3 removed
Conflicts resolved: 1 Keep master · 1 Keep project
────────────────────────────────────────────────
GridSim West Upgrade 2025

                                  [Cancel]  [Merge to Master]
```

### Without conflicts

```
Merge to master                                          ✕

────────────────────────────────────────────────
12 changes
5 added · 4 modified · 3 removed
────────────────────────────────────────────────
GridSim West Upgrade 2025

                                  [Cancel]  [Merge to Master]
```

The change count is large and prominent (the most-scanned number). The breakdown line uses the same separator pattern as the Changes Panel. The conflicts-resolved line is omitted when there were none. The validation warning banner appears only when the project has unresolved validation errors at the moment of merge — it is informational and does not gate the action.

Confirming triggers the merge; project state, notifications, and master propagation are handled in Task 07.

---

## Validation relationship

Validation severity does not gate Merge. Errors and warnings continue to surface via the Validation Panel and the `Validation` column on the Reviews list. As a final reminder, the merge confirm dialog shows a non-blocking warning banner when errors are present, so the reviewer is not surprised by what they are about to commit.

The author or reviewer carries the decision, consistent with the Approve gate from Task 05.

---

## Out of scope

- Per-attribute conflict resolution and per-attribute attribution in the panel — full component history is available via Info Panel deep-dive (click card)
- Three-way merge UI (visual diff of base / master / project values for the same attribute)
- Audit log of resolution choices — data persistence is implied for Task 07's post-merge history
- Locking when two users open the same Approved project simultaneously — last-writer-wins for MVP; second user's attempt shows an `Already merged` toast
- Mid-session recomputation of conflicts — state at panel open is state at merge confirm
- Re-running validation as part of the merge action
- Cancel/withdraw approval — the project moves linearly Approved → Merged in MVP
- Pull from master in Approved — the project model is locked; the equivalent intent is `Keep master`

---

## Forward references

- **Task 07**: Post-merge transitions — project status moves to Merged, author receives a notification, master model picks up the merged changes, resolution choices recorded in project history

---

## Open questions

None for Task 06 — all decisions resolved in discussion.
