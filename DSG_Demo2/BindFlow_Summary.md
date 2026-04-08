# Bind Components Flow — Summary

**Task:** [DSG] Demo 2 — Bind Components flow
**Status:** Complete
**Date:** 2026-04-08

## Implemented in prototype v3

### Info Panel — Physical binding section (compact multi-bind)
- Each binding = one line: `ON P-1201 Pole [change] [unbind]`
- Counter in header: "Physical binding (3)"
- If empty: warning icon + "Not bound"
- "+ Add binding" button always visible at bottom
- Hover on binding row highlights structure on Map

### Bind picker panel (Add binding)
- Opens on "+ Add binding" click
- **Simultaneous pick-on-map**: pickMode=true, green dashed rings on structures, blue outline (#00BFFF) around Map panel
- Back button returns to Info Panel
- Search input (autofocus)
- Hint: "Click to bind · Pick on map · Hold Shift to add more"
- Recent structures section (last 3 used)
- All structures list with distance, type. Already-bound = gray + "bound" tag
- **Single click** = bind + exit to Info Panel
- **Shift+click** = bind + stay in picker (list updates, item turns gray)
- **Click on map** = same Shift behavior supported
- Done button to close

### Mount equipment picker
- Identical pattern to bind picker for consistency
- Back button, Search, Hint, compact rows, Done footer
- All components shown. Already-mounted = gray + "mounted" tag
- Single click = mount + exit, Shift+click = mount + stay
- Blue outline (#00BFFF) around SLD panel during mount mode

### Bind operations
- **Bind** — picker panel or pick-on-map (simultaneous)
- **Unbind** — X button per binding row
- **Rebind** — arrows button per binding row (removes old + opens picker)
- **One-to-many** — component can be bound to multiple structures

### Connection colors by validation status
- var(--success) = full phase match (green)
- var(--warning) = partial phase match (orange)
- var(--destructive) = incompatible phases (red, solid)
- #00BFFF = hover (overrides)

### Conductor picker redesigned
- Search at top (autofocus), Recent section, All conductors with scroll (22 types)
- Recent tracking: last-used conductor moves to top dynamically

### Add flow streamlined (3 clicks)
- Right-click → Category → Variant → placed immediately at click position
- Ghost preview (dashed outline + crosshair) at placement position
- Drag-and-drop from variant list to SLD canvas (5px threshold)
- Toast with "+ Add more" instead of success screen
- Info Panel opens automatically after placement

### Toolbar simplified
- Select (V) | Pan (H) | Add (+) | Fit
- Connect mode removed — ports appear on hover

## Decisions

| # | Topic | Decision |
|---|-------|----------|
| 1 | Binding cardinality | One-to-many (component → multiple structures) |
| 2 | Entry point | Info Panel → Physical binding section |
| 3 | Reverse direction | Structure Info Panel → Mount equipment |
| 4 | Prepositions | ON (transformer ON pole), AT (meter AT site) |
| 5 | Structure deletion | Blocked if equipment bound |
| 6 | Compact display | One line per binding: prep + name + type + icon buttons |
| 7 | Dashed lines | Reserved for UG conductors and "as planned" — not for errors |
| 8 | Panel highlight | Blue #00BFFF outline during bind/mount mode |
| 9 | Multi-bind | Click = bind+exit, Shift+click = bind+stay |
| 10 | Consistency | Bind and Mount pickers use identical patterns |
