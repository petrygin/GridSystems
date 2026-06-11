# Aggregate Component · UX Spec v2

GRDM-637 · sibling document to [Spec.md](https://github.com/SGSystems/gridmodel-docs/blob/main/05_Features/Map_SLD_Editor/Aggregate_Component/Spec.md) (data model) and [API.md](https://github.com/SGSystems/gridmodel-docs/blob/main/05_Features/Map_SLD_Editor/Aggregate_Component/API.md) (BE contract).

This document defines the UX of aggregate creation, membership management, and canvas representation in the Map / SLD editor. Membership is **many-to-many** — confirmed with PM, requires Spec.md / API.md update from BE (see §12).

---

## 1. Membership model

A component may belong to **multiple aggregates simultaneously**. Adding a component to an aggregate adds a new membership; it does not remove existing memberships. There is no "move" semantics — only Add. If the user wants the component out of an old aggregate, they explicitly Remove it from there.

This is a return to v1's many-to-many model. Spec.md v2 had assumed single-parent and explicitly listed many-to-many as *Not planned*; PM has authorised the revert.

---

## 2. Entry points

**Multi-select → Create aggregate.** Selection on a single surface; context menu offers Create aggregate; the Info Panel opens in Create mode with the selection as the initial child list.

**Single component → Add to Aggregate → existing.** Submenu lists every existing aggregate of the matching domain. Picking one adds the component as a member of that aggregate, without removing it from any other aggregate it is currently in. Instant action with a sonner.

**Single component → Add to Aggregate → Create new.** Opens the Create aggregate panel with the component as the sole initial child.

**Multi-select → Add to Aggregate** uses the same submenu and adds all selected components.

The Create aggregate entry is offered only when the selection is on one surface (Map or SLD). Mixed-domain selections do not expose this action.

---

## 3. Aggregating mode

Aggregating mode is the canvas pick-mode driven from inside the Info Panel sidebar — not a separate canvas banner.

While active:

- Clicking a component on the canvas toggles its membership in the child list. Shift / Ctrl click extends.
- The hint card in the Components section reads `Click on map object to add to aggregate. Hold Shift or Ctrl for multi-select.`
- The footer **Done** button is disabled. Only **Done aggregating** inside the Components section is active.

Mode terminates on **Done aggregating**, Esc, or clicking outside the canvas. The collected child list is preserved.

---

## 4. Pick-mode visual on components

- **Default:** stock styling. Eligible for selection.
- **Hover:** standard hover treatment. Cursor `pointer`.
- **In child list:** standard `blue-600` selection ring (2 px). No additional inner ring — the colored agg-type frame was dropped to reduce visual noise, since under many-to-many a component can already belong to several aggregates with their own canvas tints.

Disallowed components (cycle, domain mismatch) get cursor `not-allowed` with a tooltip explaining the cause.

---

## 5. Canvas representation of the aggregate

The aggregate appears on the diagram as a tinted container drawn around the bounding box of its current children, with **24 px padding outward**.

- Fill: agg-type-color wash (alpha ≈ 0.08).
- Stroke: agg-type-color, 1 px.
- Label: top-left, agg-type-color square + aggregate name. 11 px / 600.

**Overlapping zones.** When a component belongs to multiple aggregates, the tinted backgrounds overlap on the canvas. The overlap is rendered as the simple sum of the two alpha-blended fills — no special blending rule, no extra UI on the overlap region.

States:

- **Expanded (default, with members):** as above.
- **Empty:** dashed 1 px container with the agg label, no fill.
- **Hover:** stroke α bumped to ~0.6.

Map representation:

- 1 member: dashed circle around the GPS point + label above.
- N members: dashed rectangle around the GPS bounding box + label above.

Re-aggregation is rendered as nested containers — the inner aggregate keeps its own tint inside the outer aggregate's tint.

---

## 6. Aggregate types (MVP)

Three types ship in the MVP. Color is read from each type's library definition.

| Type | Color | Notes |
|---|---|---|
| Substation | blue | Most common on SLD. |
| Circuit | purple | |
| Feeder | red | |

Adding new aggregate types in future iterations is library extension; UX logic is type-agnostic.

---

## 7. Behavior rules

### Add semantics

Adding a component to an aggregate adds membership; existing memberships are preserved. There is no "move" path through the UI.

### Smart placement (new aggregate position in hierarchy)

When the user creates a new aggregate Z from a selection, Z's own membership is decided automatically:

- If all selected components share **exactly one** common existing aggregate X, then Z becomes a member of X (Z is placed inside X in the hierarchy).
- Otherwise (no common aggregate, or more than one common aggregate), Z is top-level.

The user is not prompted. The rule is deliberately strict — "exactly one" — to avoid ambiguity when multiple shared aggregates exist.

### Selection normalization

When the multi-selection includes both an aggregate component and a member of that aggregate, the member is silently removed from the selection. Only the aggregate participates. The member remains a member through the existing hierarchy.

### Domain isolation

Multi-select across Map / SLD layers (physical vs electrical) is disallowed in the editor. Create aggregate is offered only when the selection is on a single surface.

### Cycle prevention

In Aggregating mode, components that would create a cycle in the membership graph (ancestors of the current aggregate through any path) are not selectable. Cursor `not-allowed`; tooltip: `Cannot create cycle`. Spec.md error `AGGREGATION_CYCLE` is the BE backstop.

### Re-aggregation

Aggregates may themselves be members of higher-level aggregates. The same Add / Create flow applies. In a child list, an aggregate is shown as a regular row with the type badge `Aggregate · N` indicating its current member count. The panel does not expand the nested hierarchy.

### Empty aggregate

An aggregate without members is a valid state. It is shown on canvas as a dashed placeholder rect with the agg label. It may receive members through Aggregating mode at any later time.

---

## 8. Member management

### Add to existing aggregate

From a single component's context menu → **Add to Aggregate** → existing aggregate name. The component gains membership in that aggregate. Other memberships unchanged. Instant action with a sonner.

For multi-select, the same path adds all selected components.

### Remove from this aggregate (per-member)

In the Aggregate's Info Panel, every child row has an `X` button. Clicking it removes only this component's membership in this aggregate. The component remains a member of any other aggregates it was in. Instant action with a sonner.

### Remove from all aggregates (per-component)

In a Component's Info Panel, the **Aggregates** section's kebab menu has one item: `Remove from all aggregates`. Strips every membership for this component. The component becomes loose (not a member of anything). Instant action with a sonner.

### Remove all members

Action in the Aggregate's **Components section kebab**. All children lose their membership in this aggregate. Other memberships are preserved. The aggregate component remains, now empty. Confirm dialog required before action (§10).

### Delete aggregate

Action in the Aggregate's **Components section kebab**. Confirm dialog with checkbox **Delete all children (N)** (§10):

- Checkbox **unchecked** (default): the aggregate container is deleted. Children lose membership in this aggregate but keep all other memberships and continue to exist.
- Checkbox **checked**: the aggregate container is deleted, **and** every child component is also deleted entirely (regardless of other memberships).

There is no separate "Dissolve" action — Delete with checkbox covers both flows.

---

## 9. Sonners

Every mutating action shows a sonner. Sonners are informational only — Undo is not supported by the editor.

| # | Sonner title | Subtitle | Triggered by |
|---|---|---|---|
| 1 | `Aggregate created` | — | Create aggregate (any case) |
| 2 | `5 components added to Substation 1` | — | Add to Aggregate (any case) |
| 3 | `Aggregate deleted` | — | Delete aggregate, children kept |
| 4 | `Aggregate deleted` | `5 components deleted` | Delete aggregate, children deleted too |
| 5 | `All members removed` | `Substation 1 is now empty` | Remove all members |

There are no `with re-parenting` variants because under many-to-many there is no re-parenting — Add only ever extends membership, never removes it.

---

## 10. Confirm dialogs

Only two actions trigger a confirm dialog. Other operations are instant (Create, Add to existing, Remove from this aggregate, Remove from all aggregates) — they have no destructive effect on existing data.

**Delete aggregate.** Title: `Delete Substation 1?` Body: depends on checkbox.
- Checkbox `Delete all children (5)`: when unchecked, body reads `5 components will move out. Substation 1 will be removed.` When checked, body reads `5 components will be deleted. Substation 1 will be removed.`
- Buttons: `Cancel` · `Delete` (destructive red).
- Reason: the aggregate container is destroyed permanently; the checked variant also destroys children.

**Remove all members.** Title: `Remove all 5 members from Substation 1?` Body: `Components will move out. Substation 1 will remain empty.`
- Buttons: `Cancel` · `Remove all`.
- Reason: bulk detach; manually re-adding all members afterward would be tedious.

Both dialogs include the aggregate name in the title for clear anchoring on the affected object.

---

## 11. Info Panel structure

### Aggregate Info Panel (existing aggregate)

- **Header:** aggregate name, expand icon, close X.
- **Section Defined:** read-only Type / ID / Name rows.
- **Section Components:** counter badge + kebab menu, child list with `X` per row, **Aggregating mode** button at the bottom (full-width, primary).
- **Kebab in Components section:** `Remove all members` · `Delete aggregate` (red).
- **No footer** — the kebab carries destructive actions; Aggregating mode is the only primary, anchored at the section bottom.

`Locate` is not needed here — the user is already viewing this aggregate, navigation isn't a separate action.

### Component Info Panel (any component)

- Existing sections (Defined, Library info, Calculated, Physical binding, Files, etc.).
- **Section Aggregates:** counter badge + kebab menu, list of every aggregate this component is a member of. Each row: colored square (agg type color) · aggregate name · type · `X`.
- The per-row `X` removes only that one membership.
- The kebab on the section header carries one item: `Remove from all aggregates`.
- The `Aggregate member` field previously sitting in the Defined section is removed — the Aggregates section is the single source of truth for memberships.

### Create aggregate panel (during creation)

- **Header:** `Create aggregate`, expand icon, close X.
- **Section Defined:** Type (dropdown), ID (input, auto-suggested), Name (input).
- **Section Components:** counter, hint card (when Aggregating mode active) or empty-state message, editable child list with `X` per row, **Aggregating mode** / **Done aggregating** button.
- **Footer:** **Done** button, disabled while Aggregating mode is active.

---

## 12. Scope extensions required from data model

Spec.md v2 currently:

- Assumes single-parent membership (`A component has at most one parent.`)
- Lists many-to-many as *Not planned*.
- Lists Dissolve / Remove operations as *TODO — later version*.

The UX above requires all three to flip in the same release:

1. **Membership is many-to-many.** A child carries an array of parent references (or equivalent join structure).
2. **Remove operations.** Per-membership removal, bulk removal of all members from an aggregate, removal of one component from all aggregates.
3. **Delete operation, with optional cascade.** Removes the aggregate container; the user-controlled checkbox decides whether children are also deleted.

PM has confirmed this direction. Spec.md and API.md need to be updated to match; a sibling ticket in epic GRDM-636 should track that work, or GRDM-637 itself should be scope-extended.

All operations follow the same task / RBAC / idempotency / event-logging rules already specified in Spec.md.

---

## 13. Out of scope · future

| Item | Note |
|---|---|
| Aggregate types beyond MVP three (Region, Zone, etc.) | Library extension; UX logic is type-agnostic. |
| Map view rendering for nested aggregates beyond depth 1 | Visual rule deferred until first real depth-2 case. |
| Composition templates (allowed types, min / max) | Not planned. |
| Drag-in onto aggregate area | Not planned — drag & drop removed from editor. |
| Confirm dialog on `Remove from all aggregates` | Currently instant; revisit if support reports surprise. |

---

## 14. Related artefacts

- Figma: `Grid Model Work in Progress`, node `3617-23252`.
- Spec.md · data model (needs revert to many-to-many): see top of file.
- API.md · BE contract (needs updates): see top of file.
- Epic: GRDM-636 · Ticket: GRDM-637.
