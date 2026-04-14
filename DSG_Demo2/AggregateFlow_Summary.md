# Aggregate Flow — Design Decisions

**Date:** April 2026
**Status:** Implemented in prototype v4, refined in v5

---

## Concept

Aggregate = grouping components into a named container with a type template. User creates a logical grouping (e.g. Substation) and the system validates composition rules.

---

## MVP Templates (from LIB-16)

Three hardcoded Vendor templates:

| Template | Description | Must have | Key rules |
|----------|-------------|-----------|----------|
| Substation | Equipment at a single location | Bus (1+) | Transformer 1–10, Switch 0–50, Bus 1–20, Meter 0–100 |
| Circuit | Network section between substations | Switch (1+) | Switch 1–20, Meter 0–50, Transformer 0–5 |
| Feeder | Distribution line from substation | Transformer (1+) | Transformer 1–20, Switch 0–30, Meter 0–100 |

---

## Interaction Design

### Two creation scenarios

**Bottom-up (group existing):**
1. Select 2+ components (Shift+click)
2. Right-click → Add to Aggregate → Create new
3. Info Panel: select type, enter name → Create
4. Background + label appears on diagram, polygon on map

**Top-down (empty container):**
1. Right-click empty → Add aggregate
2. Select type, name → Create
3. Dashed placeholder appears
4. Populate via drag-in or right-click → Add to Aggregate

### Drag-in with undo snackbar

Drag component onto aggregate area → instant add → snackbar "TR-312 added to Substation 1" [Undo]. Auto-dismiss 4s.

### Visual representation

| State | Diagram | Map |
|-------|---------|-----|
| Expanded, has children | Tinted background + label (auto-fits children with padding) | Polygon boundary around GPS points |
| Collapsed | Compact block "Name (N)" | Single marker |
| Empty | Dashed placeholder "Drag components here" | — |

Colors by type: Substation = purple (#8B5CF6), Circuit = blue (#0EA5E9), Feeder = amber (#F59E0B).

### Info Panel

Click on aggregate background → Info Panel with:
- Type icon + status badge (Valid / Warning / Invalid)
- Member list with per-member remove button
- Composition Rules with ✓/⚠/✕ indicators
- Footer: Collapse / Dissolve buttons

### Context menu

- On aggregate: Show info, Collapse/Expand, Add component inside ▸, Dissolve
- On component: Add to Aggregate ▸ (existing list + Create new), Remove from "Name"

---

## Decisions

| Decision | Choice | Rationale |
|----------|--------|----------|
| Visual style | Background + label (not frame) | Less heavy, doesn't interfere with component rendering |
| Collapse | Available but not primary | Users mostly work expanded |
| Nesting | Not in MVP | Single level; full hierarchy (Region → Substation) deferred |
| GPS validation | Not in aggregate inline | Handled by full Verify, not composition rules |
| Drag-in confirmation | Snackbar with undo (not Yes/No dialog) | Faster workflow, Gmail-style pattern |
| Entry point | Right-click submenu "Add to Aggregate" | Consistent with other actions, discoverable |
| Single click on aggregate | Opens Info Panel | Consistent with component click behavior |
