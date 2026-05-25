# Component Aggregation — UX Spec v2

> Task: [GRDM-637](https://sgsstms.atlassian.net/browse/GRDM-637) · Epic: [GRDM-636](https://sgsstms.atlassian.net/browse/GRDM-636)
> Figma: [Aggregating frame](https://www.figma.com/design/0FW49cYCouwjKw3YzqQwcI/Grid-Model-Work-in-Progress?node-id=3617-23252)
> Supersedes: v1 (GRDM-299, see `AggregateFlow_Summary.md`)
> Domain rules: `gridmodel-docs/UC-06 RegionalAggregation`

---

## 1. Overview

v2 replaces the templated, many-to-many aggregation model from v1 with a single-parent model that mirrors the existing binding flow.

- **An aggregate is a component.** It is a regular ModelItem of an aggregating type from the Equipment Library, acting as a parent for other components.
- **Children point to one parent.** Every child references its aggregate via a single `parentId`. There is no membership in multiple aggregates at once.
- **The interaction is the binding flow.** Picker side panel + modal "Aggregation mode" on canvas — same gesture and visual idiom as binding a component to a structure.
- **Nested aggregates are allowed.** An aggregating component may itself be the child of another aggregating component (e.g. a Circuit inside a Substation).

The aggregate has no library-defined composition rules, no min/max validation, and no allowed-child-type list. The library is responsible only for declaring that a type is *aggregating*, and for the type's display color.

---

## 2. Data model

| Field | Owner | Meaning |
|---|---|---|
| `parentId` | child component | UUID of the aggregating component the child belongs to. `null` for loose components. |
| `aggregatingType` flag | library type | Boolean — whether a library type may act as a parent. |
| `color` | library type | Display color of the aggregate wash/stroke and the type-marker square in pickers. |

Membership is read by querying `parentId`. There is no separate join table, no `aggsOfComp` array, no `composition` rules object.

---

## 3. Entry points

There are four entry points; all share the same Info Panel and Aggregation mode.

### 3.1 Create new aggregate — from selected components

User multi-selects components on MAP or SLD → right-click → context menu shows `Add to Aggregate` (with submenu chevron) → submenu opens with a list of existing aggregates and a `+ Create new` action at the bottom → user picks `+ Create new`.

Result: a new aggregating component is instantiated as draft, the selected components are pre-populated in its Components list, the Info Panel opens in Create mode, and the canvas enters Aggregation mode.

### 3.2 Create new aggregate — from empty area

User right-clicks on an empty area of MAP or SLD → context menu shows `Add component` and `Add aggregate` → user picks `Add aggregate`.

Result: same as 3.1, but with an empty Components list.

### 3.3 Add to existing aggregate — from selected components

User multi-selects components → right-click → `Add to Aggregate` submenu → user picks an existing aggregate from the list (searchable via the submenu header input).

Result: `parentId` of every selected component is set to the picked aggregate's id. If a component already had a different `parentId`, it is reparented (moved, not duplicated). A sonner confirms with the count and the target aggregate.

This is an instant operation — no Info Panel opens, no Aggregation mode.

### 3.4 Re-enter aggregating mode — on existing aggregate

User right-clicks on an existing aggregate on canvas → context menu shows `Aggregating mode` → user picks it.

Result: the aggregate's Info Panel opens in View mode and the canvas enters Aggregation mode against this aggregate. User keeps clicking components to add/remove from the child list.

---

## 4. Aggregation mode (canvas state)

Aggregation mode is a modal state of one canvas — MAP for physical-class components, SLD for electrical-class components. The class is determined by the aggregate's type.

**Active visual indicator.** While Aggregation mode is on, the active canvas shows a perimeter highlight ring around its full viewport.

**Click behavior.** Clicking a component on the canvas toggles its membership in the aggregate being built:
- If the component is loose or belongs to another aggregate — it is added (its `parentId` will be set to this aggregate on commit; on commit, reparenting is performed if needed).
- If the component is already a member of this aggregate — it is removed from the child list.

**Pick-mode component visual.** A component currently in the child list is rendered with two stacked rings — the standard selection ring (`--selection`, blue-600, 2px) on the inside, and a 1.5px ring in the aggregate type's color on the outside (~3px offset). This double-ring read communicates both "selected by you" and "assigned to this specific aggregate."

**Exit.** User clicks the `Done aggregating` button inside the Info Panel Components card to leave the mode. The Info Panel stays open, the canvas perimeter highlight is removed.

**Other interactions while in mode.** Right-click and the standard component context menu remain available. Selection by drag-rectangle is disabled — clicks during Aggregation mode are interpreted as add/remove toggles.

---

## 5. Aggregate Info Panel

The Info Panel is a 300px right-anchored sidebar with the standard layout: 48px header, scrollable content area, 44px footer.

### 5.1 Create mode

**Header:** `Create aggregate` + expand-icon + close ×.

**Defined card** — editable:
- `Type` — Select reusing the Add Component type picker. Selected option shows the type-color square + name. The dropdown lists aggregating types from the library.
- `ID` — text input, system-suggested but editable, must be unique.
- `Name` — text input, defaults to a system-suggested name based on Type + ID.

**Components card** — count badge + child list:
- Each child row: type icon + name + type label (right-aligned, muted) + × to remove. Nested aggregate child appears as `<name> · Aggregate · <count>`.
- If list is empty: empty-state copy + a single `Aggregating mode` button to enter the mode immediately.
- While Aggregation mode is ON: a sparkle-prefixed hint `Click on object to add to aggregate` appears at the top of the card body, and a primary-styled `Done aggregating` button replaces the secondary `Aggregating mode` button at the bottom of the card.

**Footer:** primary button `Done`.
- Enabled when Aggregation mode is OFF.
- Disabled while Aggregation mode is ON (user must exit the mode first via `Done aggregating`).

**Cancel.** Closing the panel via × discards the draft aggregate and any pending child assignments. If at least one component was added to the draft, a confirmation toast is shown ("Discard aggregate draft?" — `Discard` / `Keep`).

### 5.2 View mode — existing aggregate

**Header:** the aggregate's name (e.g. `Substation 1`) + expand-icon + close ×.

**Defined card** — read-only Horizontal Field group with `Type`, `ID`, `Name` rows.

**Components card** — identical to Create mode, including the `Aggregating mode` re-entry button.

**Footer:** primary button `Edit attributes` — switches Defined card to inputs. The Components card remains unchanged between View and Edit; child management is always available via `Aggregating mode` and inline ×.

---

## 6. Component Info Panel — Aggregate row

When a regular (non-aggregate) component is selected and its Info Panel opens, a single horizontal field row shows its current aggregate parent.

- One row in the shared field stack (sits with other Library / Calculated / Physical binding rows).
- Format: label `Aggregate` + type-color square + aggregate name + × at the end.
- × clears `parentId` (instant, with undo sonner).
- If the component has no parent (`parentId === null`), the row is omitted entirely — no empty placeholder.

This replaces the v1 collapsible "Aggregates" card. Since v2 allows at most one parent, a full card with header, badge, and accordion is overkill for a single value.

---

## 7. Aggregate visualization on canvas

Aggregates are drawn as bounded containers around their children, on both MAP and SLD.

**Container.** Light wash fill + colored stroke + name label anchored top-left. Bounding shape is auto-fit to children with a default padding.

**Color.** Wash, stroke, and label text all derive from the type's `color` attribute in the library. Starter set:
- Substation — blue
- Circuit — violet
- Feeder — red/pink

**Empty / just-created aggregate.** When an aggregate has zero children (e.g. just created via 3.2), it is drawn as a dashed outline of a default size with the name label only — no fill wash.

**Nested aggregates.** A child aggregate is rendered inside its parent's bounds as its own container, with its own wash/stroke. Overlapping washes are intentional — the type colors are distinguishable.

**Selection.** Clicking an aggregate stroke or label selects the aggregate (not its children). Selected aggregate gets the standard `--selection` outline.

**Hit testing.** The container fill is transparent to clicks on children — clicks "fall through" the wash to whatever component is underneath. Only the stroke and label are click targets for selecting the aggregate itself.

---

## 8. Aggregate context menu

Right-click on an aggregate (stroke or label hit) opens its context menu:

1. **Header label** — the aggregate's name, non-clickable.
2. `Show info` — opens the aggregate's Info Panel in View mode.
3. `Locate` — pans/zooms canvas to fit the aggregate.
4. ---
5. `Aggregating mode` — re-enters Aggregation mode against this aggregate (see 3.4).
6. ---
7. `Remove all members` — see 9.2.
8. `Dissolve aggregate` — see 9.1. Rendered with destructive (red) text.

---

## 9. Destructive operations

Both destructive operations are **instant**, with no confirmation dialog. They are reversible via a sonner-attached `Undo` action for ~8 seconds.

### 9.1 Dissolve aggregate

Removes the aggregate component itself. Children are re-parented to the aggregate's grandparent (or to `null` if the aggregate was at the top level).

Sonner: `Aggregate dissolved` + `Undo`.

### 9.2 Remove all members

Clears `parentId` for every direct child of the aggregate. The aggregate component itself remains, now empty. The canvas immediately shows the empty-aggregate dashed-outline state (see 7).

Sonner: `All members removed` / `<Aggregate name> is now empty` + `Undo`.

---

## 10. Sonners

All aggregation operations emit a sonner. Every sonner offers `Undo` for ~8s.

| Operation | Sonner title | Sonner detail |
|---|---|---|
| Create new aggregate, no children reparented | `Aggregate created` | — |
| Create new aggregate, some children reparented from other aggregates | `Aggregate created` | `<N> components moved from <Aggregate A>, <Aggregate B>` |
| Add components to existing aggregate (no reparenting) | `<N> components added to <Aggregate name>` | — |
| Add components to existing aggregate (with reparenting) | `<N> components moved to <Aggregate name>` | — |
| Dissolve aggregate | `Aggregate dissolved` | — |
| Remove all members | `All members removed` | `<Aggregate name> is now empty` |
| Clear single component's parent (× on the component's Aggregate row) | `<Component name> removed from <Aggregate name>` | — |

---

## 11. Library — aggregating types

Aggregating types are defined in the Equipment Library with two additional attributes beyond the standard component type:

- `aggregating: true` — the boolean flag that lets the type appear in the aggregate Type picker.
- `color` — the hex/token value used for the canvas wash, stroke, label, and the type-color square in pickers and rows.

Starter set for the MVP library: `Substation`, `Circuit`, `Feeder`. Adding a new aggregating type is a library-data change — no UX work required.

---

## 12. Out of scope

Carried over from v1 / GRDM-299 and explicitly dropped:

- Composition-rule templates (allowed child types, min/max counts).
- Many-to-many membership.
- Drag-and-drop a component onto an aggregate area (and the resulting Move / Add-to-both dialog).
- Confirmation dialog for `Dissolve aggregate` and `Remove all members` (replaced by instant + undo).
- Aggregate Info Panel "Composition rules" section.
- Map-view rendering rules for aggregates at hierarchy depth > 1 (visual rule deferred until the first real depth-2 case).

---

## 13. Related artefacts

- Figma: [Aggregating frame](https://www.figma.com/design/0FW49cYCouwjKw3YzqQwcI/Grid-Model-Work-in-Progress?node-id=3617-23252)
- Jira: [GRDM-637](https://sgsstms.atlassian.net/browse/GRDM-637), epic [GRDM-636](https://sgsstms.atlassian.net/browse/GRDM-636)
- Domain rules: `gridmodel-docs/05_Features/UC-06_RegionalAggregation.md`
- v1 design (deprecated): `DSG_Demo2/AggregateFlow_Summary.md` (GRDM-299)
- Binding flow reference (the pattern v2 mirrors): `DSG_Demo2/BindFlow_Summary.md`
