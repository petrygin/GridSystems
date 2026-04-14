# DSG Demo 2 — Prototype v5 Feature Summary

**Date:** April 2026
**File:** `prototype-v5-sketch.html`
**Size:** ~180KB, ~3350 lines, ~80 functions
**Tech:** Standalone HTML, Vanilla JS, SVG rendering, no dependencies

---

## 1. General Layout & Navigation

Screen split into two panels: Physical Map (left, dark) and SLD Diagram (right, light), with draggable divider. Header bar with project name, branch, date, Draft status. Status bar with state, hover info, counters (components, connections, aggregates).

**Two operating modes:**
- **View mode** — compact toolbar [Select] [Edit], viewing and Info Panel only, palettes hidden, editing blocked
- **Edit mode** — full toolbar [Select] [Pan] | [Add] | [Fit] | [✓ Done], palettes visible, all tools available. Entered via Edit button or E key

---

## 2. Add Component

**Sketch-first approach:** user lays out topology first without specifying exact models, then refines later.

**Three entry points:**
- **Left palette** (8 types: Connector, Transformer, Switch, Shunt, Voltage Regulator, Protection, Meter, Sectionalizer) — drag-and-drop onto diagram. Collapsed (40px, 32×32 buttons) and expanded (2×4 grid, 80×60 buttons) views
- **Right-click → Add component** — submenu with 8 types + From library
- **Add button (A)** — panel with Quick add grid + From library section

**Generic components:** placed with orange "?" badge — type unspecified. Validation V-U01 warning. Specification via Info Panel orange section with library variants.

**Detailed flow (from library):** category → variant (kVA, kV, phases) → form (ID/Name/Tags) → Library info → placement.

**Placement:** click on diagram (no GPS) or map (with GPS). Shift+click for serial placement.

**Structure palette on map:** Pole, Vault, Pad, Duct — drag-and-drop.

---

## 3. Connect

- **Drag-to-connect:** port-to-port, orthogonal 4-segment paths
- **Conductor picker:** 22 types, search + recent + scrollable list
- **Phase compatibility:** green (full) / orange (partial) / red (incompatible)
- **Connect to... (list-based):** for large diagrams, via context menu
- **Multi-select connect:** Shift+click two → Enter
- **Midpoint handles:** draggable route adjustment (purple handles)
- **Map reflection:** OH solid, UG dashed lines between nearest structures

---

## 4. Bind (Electrical → Physical)

- **Info Panel → Physical binding:** one-to-many bindings list
- **Bind picker:** search + compatible structures, click = bind, Shift+click = stay in mode
- **Mount Equipment picker:** reverse direction from structure card
- **Pick on map:** banner + click structure, active panel outlined #00BFFF
- **Prepositions:** ON (pole), IN (vault/pad) — auto-assigned
- **Rebind / Unbind:** per-binding buttons
- **Orphan badges:** orange "!" on diagram, orange marker on map

---

## 5. Aggregate

**Three MVP templates** (from documentation): Substation, Circuit, Feeder with composition rules (allowed types, min/max).

- **Bottom-up:** select components → right-click → Add to Aggregate (submenu) → existing or Create new
- **Top-down:** right-click empty → Add aggregate → empty container → populate
- **Drag-in:** drag onto aggregate area → snackbar "Added to X" with Undo
- **Dissolve:** right-click → Dissolve aggregate

**Visual on diagram:** expanded (tinted background + label), collapsed (compact block), empty (dashed placeholder).
**Visual on map:** polygon boundary around GPS points.

**Info Panel:** type icon, status badge (Valid/Warning/Invalid), member list with removal, composition rules (✓/⚠/✕).

**Validation:** min/max checks, mustHave rules per template.

---

## 6. Info Panel

- **Components:** Defined (ID, Name, Type, Phases), Library info (Terminals, Connections), Calculated (GPS), Physical binding (with rebind/unbind), Files (stub)
- **Structures:** Label, Type, coordinates, Mounted Equipment (with unbind)
- **Aggregates:** Type + status badge, Members with removal, Composition Rules
- **Specify type:** orange section for generic components with library variants

---

## 7. Context Menu

**Component (Edit):** Show info, Locate, Add to Aggregate ▸, Remove from aggregate, Add/Remove scope, Edit in project, Edit attributes, Replace type, Connect to..., Disconnect ▸, Duplicate, Delete, Add child, Select children, History, Show/Hide, Lock/Unlock

**Component (View):** Show info, Locate

**Connection:** Show info, Locate both ends, Disconnect

**Aggregate:** Show info, Collapse/Expand, Add component inside ▸, Dissolve

**Empty (Edit):** Add component ▸ (8 types + From library), Add aggregate, Paste

**Empty (View):** Edit mode (E)

**Map structure:** Show info, Mount equipment, Delete

**Map empty:** Add structure ▸ (Pole/Vault/Pad)

---

## 8. Validation

- **Inline:** phase compatibility colors, orphan badges, unspecified variant badges
- **Validation mini panel:** live error/warning list (V-E01 Ampacity, V-E02 Phase Continuity, V-E03 Orphaned Node, V-U01 Unspecified Type)
- **Aggregate composition:** min/max checks, mustHave rules

---

## 9. UX Patterns

- **Undo** (Cmd/Ctrl+Z) — all actions, 50 steps
- **Multi-select** (Shift+click) — hint at 2 selected
- **Drag threshold** (5px) — prevents accidental moves
- **Toast notifications** — action feedback
- **Snackbar with Undo** — aggregate drag-in
- **Keyboard:** V (select), H (pan), A (add), E (edit mode), Enter (connect), Escape (cancel)
- **Hover sync** — diagram ↔ map highlighting, status bar info
- **Design system:** shadcn/ui tokens, Geist font, 14px body, blue #00BFFF for selection only
