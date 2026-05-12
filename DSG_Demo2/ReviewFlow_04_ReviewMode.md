# Task 04 — Review Mode

**Scope:** The project view when a project is in `In review` state. Map+SLD split viewport with diff highlights, universal Changes panel, status pill popover for reviewer actions.

**Related:** Entry from Reviews page → Task 03. Status pill base behavior → Task 01. Reviewer eligibility → Task 02. Comments thread → forward reference.

For shared terminology and state model, see `ReviewFlow_00_Overview.md`.

---

## Goal

Give reviewers a clear surface to:
- See what changed (Map + SLD + Changes panel)
- Inspect changes in detail (component popover with diff)
- Submit a verdict (Approve / Request changes / Decline)

Authors and watchers see the same surface — same data, role-appropriate actions.

---

## Architectural framing

Review Mode is the natural state of the project view when the project is `In review`. Anyone with access sees the same view; available actions vary by role.

Three pillars:

1. **Viewport** — Map + SLD shown side by side, both with diff highlights
2. **Changes panel** — universal sidebar, persistent across project states
3. **Status pill popover** — lifecycle control point, adapts to role and current reviewer status

---

## Viewport — Map + SLD split

Map (left) and SLD (right) render simultaneously. Both show the same diff highlighting from the underlying change data. Maximize of a single view is planned for v1.1.

### Diff visualization

| Change type | Visual treatment | Color |
|---|---|---|
| Added | Filled outline with pulse animation, `+` marker | Success green |
| Modified | Outline with `~` marker, light fill | Neutral grey |
| Removed | Ghost on previous position: dashed outline, crossed marker | Destructive red |
| Not changed | Default appearance | Neutral |

The `Not changed` state is for components that exist in both master and project unchanged. They render normally; they become visually attenuated when `Highlight by: Changes vs Master` is enabled in Layers.

### Layers panel integration

The existing Layers panel gets a `Highlight by: Changes vs Master` toggle. When enabled, diff highlights become more prominent and unchanged components are dimmed. This toggle is a Layers feature, not Review-Mode-specific — it works in Draft as well (author self-review).

---

## Changes panel (universal sidebar)

The Changes panel is persistent and universal — it exists for any project state:

- **Draft**: author sees their own changes vs master (self-review before send)
- **In review**: reviewers see changes for approval
- **Approved / Merged / Archived**: read-only audit view

### Toggle location

The Changes panel toggle lives in the right header group next to project status indicators:

```
[⚠ N warnings] [Status pill] [● N conflicts] [📋 N changes] [☾] [✨] [🔔] [PP]
```

Icon with counter (`N changes` or `99+` if >100). Counter hidden when count is 0. Click toggles the panel.

### Panel structure

```
Changes [N] [×]
[Physical & Electrical ▾] [All component types ▾]

▾ Added (N)
  cards…
▾ Modified (N)
  cards…
▾ Removed (N)
  cards…
```

### Grouping

Single level by change kind: Added / Modified / Removed.

Each group mixes entity types (components / connections / aggregates), distinguished by:

- Badge icon (shield for component, link for connection, layers for aggregate)
- Entity prefix in specs line (`Connection · `, `Aggregate · `)

### Card content

- Badge icon — entity type
- Name — component name + ID, connection endpoints, or aggregate name
- Specs — key attributes (e.g., `500 kVA · 12 kV`) + entity prefix for non-components
- Meta — author · timestamp; for Modified: `N changes · author · timestamp`

Sort within group: timestamp descending (newest first).

### Filters

Two filters in the panel header:

- `Physical & Electrical` — filter by view type (Physical for Map, Electrical for SLD). Multi-select.
- `All component types` — filter by component category (Transformer, Pole, Conductor, etc.). Multi-select.

Filters apply to panel content; they do not affect viewport highlighting.

### Click behavior

Click on a card:

- Highlights the corresponding component on Map and SLD (auto-routes: physical → Map, electrical → SLD, hybrid → both with focus on the active view)
- Centers the viewport on the component
- Opens the component popover with diff details

For connections and aggregates without map points, the click highlights related components without opening a popover.

---

## Component popover

Triggered by clicking a component on Map / SLD or in the Changes panel.

Content:

- Header: component name + type, change tag (Added / Modified / Removed)
- Author + timestamp
- Diff body:
  - **Added**: list of all attributes (snapshot at time of add)
  - **Modified**: `attr: old → new` per changed attribute
  - **Removed**: snapshot of all attributes as they were before removal
- Actions: `Add comment` and `Show info`

`Add comment` is a forward reference to the comments thread (separate scope, TBD).
`Show info` opens the existing Info Panel for the component.

---

## Status pill popover (review actions)

The status pill in the header is clickable and opens a popover. The popover adapts to the viewer's role and the current reviewer state.

### Popover content

- Header: status pill (e.g., `In review`) + `Sent N ago by <author>`
- Divider
- Reviewers section title
- Reviewer list: `Avatar · Full Name · Status` per reviewer
  - Status colors: Pending (muted), Approved (success green), Declined (destructive red), Requested changes (warning orange)
- Divider
- Action area — role and state dependent

### Action area — by role and state

| Viewer | State | Actions shown |
|---|---|---|
| Reviewer | Pending | `Approve` (primary) + `Request changes` (secondary) + `Decline` (secondary, destructive tint) |
| Reviewer | Approved | Status line `You approved · awaiting <next>` + `Withdraw approval` |
| Reviewer | Requested changes | Status line `You requested changes · Project returned to Draft` |
| Reviewer | Declined | Status line `You declined · Author notified` |
| Author | (always) | `Cancel review` (destructive) — defined in Task 01 |
| Watcher (no role) | — | Status display only |

### Action semantics

**Approve**
- Reviewer's status becomes `Approved`
- Author receives notification
- Toast appears with Undo action for ~8 seconds
- Reviewer can withdraw approval at any time until the project transitions to `Approved` overall (all reviewers approved) or `Merged`

**Request changes**
- Opens a modal with required textarea (`Comment to author`)
- On submit: project returns to `Draft`, all reviewers notified, author receives the comment
- Reviewer selection is cleared for the next submit

**Decline**
- Opens a modal with required textarea (`Reason for declining`)
- On submit: the reviewer drops out of the review
- Reviewer's status becomes `Declined`
- Project stays `In review` if other reviewers are still Pending
- The reviewer is removed from the active reviewer set for this review cycle

---

## Entry points to the popover

The popover has two entry points:

1. Click `In review` chip in the header
2. Click `Finish reviewing` button in the bottom toolbar

Both open the identical popover. Trade-offs:

- **Plus for chip**: Keeps a single header layout across all project states.
- **Plus for Finish reviewing button**: Prominent CTA in the bottom toolbar — hard to miss for new reviewers.
- **Minus**: Two surfaces for the same action.

Both entry points are kept. If implementing only one is cheaper, choose the chip.

---

## Bottom area — status bar + toolbar

Two stacked strips under the viewport, with distinct purposes:

**Status bar** — informational, contains no controls:

- `Review mode` indicator (red dot + label) — shown only in Review Mode
- Selected info (`Selected: <id> (<type>)`)

**Toolbar** — contains controls:

- `Finish reviewing` button — shown only in Review Mode
- Zoom controls (Map zoom · SLD zoom)

Outside Review Mode the status bar shows Selected info; the toolbar contains zoom controls.

---

## Dark mode

Fully designed (see Figma). Same component palette with dark-mode tokens. Selection ring uses `blue-400` for dark, `blue-600` for light. Diff colors adapt for contrast.

---

## Out of scope (Task 04)

- Comments thread implementation (separate scope, referenced via `Add comment` button)
- Re-running validation from Review Mode (Task 05)
- Real-time updates if author edits during review (separate scope)
- Side-by-side comparison view (master vs project rendered side by side) — only diff highlighting in proposed state
- Per-component reviewer status (this change OK, that needs changes) — review verdict is at project level
- Mark-as-reviewed checkboxes per component
- Bulk actions in Changes panel
- Maximize single view (Map only or SLD only) — planned v1.1

---

## Forward references

- Comments thread (`Add comment` CTA in component popover, `N Comments` counter referenced in Task 03 details)
- Validation re-run in Review Mode (Task 05)
- Conflict resolution between project and master (Task 06)
- Maximize single view toggle (v1.1)

---

## Open questions

None for Task 04 — all major decisions resolved.
