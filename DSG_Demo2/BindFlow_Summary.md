# Bind Components Flow — Summary

**Task:** [DSG] Demo 2 — Bind Components flow
**Status:** Complete
**Date:** 2026-04-08

## Scope
- Link electrical to physical components (one-to-many)
- Clear representation of bindings and interaction

## Implemented in prototype v3

### Info Panel — Physical binding section
- Compact multi-bind list: each binding = one line `ON P-1201 Pole [change] [unbind]`
- Counter in header: "Physical binding (3)"
- If empty: warning icon + "Not bound"
- "+ Add binding" button (secondary) always visible at bottom
- Hover on binding row highlights structure on Map

### Bind operations
- **Bind** — click "+ Add binding" → pick-on-map (green rings on structures) or from Structure Info Panel
- **Unbind** — X button per binding row, removes single binding from list
- **Rebind** — arrows button per binding row, removes old + starts pick-on-map for new
- **One-to-many** — component can be bound to multiple structures (cable through 20+ poles)

### Structure Info Panel (Map click)
- Details: type, ID, GPS, characteristics
- Mounted equipment list with prepositions
- "+ Mount equipment" button (secondary) — reverse bind direction

### Add Structure on Map
- Right-click empty Map area → "Add structure..." → Pole / Vault / Pad picker
- Placement mode: crosshair + banner → click to place
- Map context menu on structures: Show info, Mount equipment, Delete

### Visual feedback
- Orphan badge (orange !) on SLD for unbound components (except Bus)
- Map hover on structures — blue highlight + status bar info
- Pick-on-map mode — green dashed rings on compatible structures
- Dropdown/list hover → structure highlights on Map

## Decisions

| # | Topic | Decision |
|---|-------|----------|
| 1 | Binding cardinality | One-to-many (component → multiple structures) |
| 2 | Entry point | Info Panel → Physical binding section |
| 3 | Reverse direction | Structure Info Panel → Mount equipment |
| 4 | Auto-bind on Add | No for MVP — always manual |
| 5 | Conductor binding | Automatic (straight line). Manual route post-MVP |
| 6 | Prepositions | ON (transformer ON pole), AT (meter AT site), IN (cable IN conduit) |
| 7 | Structure deletion | Blocked if equipment bound |
| 8 | Compact display | One line per binding: prep + name + type + icon buttons |

## Also updated in this round

### Toolbar simplified
Select (V) | Pan (H) | Add (+) | Fit. Connect mode removed — ports appear on hover.

### Connection colors by validation status
- var(--success) = full phase match
- var(--warning) = partial phase match
- var(--destructive) = incompatible phases
- #00BFFF = hover

### Conductor picker redesigned
- Search at top, Recent section, All conductors with scroll (22 types)
- Last-used conductor tracked dynamically

### Add flow streamlined (3 clicks)
- Right-click → Category → Variant → placed immediately at click position
- Ghost preview (dashed outline) at placement position
- Drag-and-drop from variant list to SLD canvas
- Toast with "+ Add more" instead of success screen
- Info Panel opens automatically after placement

### Connect to... (list-based connect)
- Right-click component → "Connect to..." → searchable panel with all compatible components
- Shows phase compatibility, distance, free terminals
- Hover highlights target on SLD
