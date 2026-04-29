# Status Bar — UX Spec

**Version:** 0.1 · **Status:** Draft · **Scope:** DSG Demo 2

The status bar is the always-visible context line at the bottom of each canvas. It lives **per canvas** — Map and SLD each have their own — and reflects what's on that canvas, not global app state.

Global system state (errors, save, validating, branch context) lives in the **header**, not here.

---

## 1. Architecture

Each canvas has one status bar with a single content zone:

```
┌──────────────────────────────────────────────────────────────┐
│ [ Mode dot + content ]                                       │
└──────────────────────────────────────────────────────────────┘
```

The bar has one job: show what's happening on this canvas right now. Content is one of:

- **Empty state** — per-view counters fill the zone (only place counters appear)
- **Hover** — `<ID> · <Type> · <facts>` for the hovered element
- **Selected** — same content, prefixed with `Selected:`
- **Active mode** — `●` + instruction
- **Trace result** — trace summary with `Clear` action
- **Overlay (Filter / Compare)** — overlay chip with `Clear` action

Counters do not get their own zone — the viewport is too narrow to fit both selection content and counters. They appear only when there's nothing else to show.

Height: 28px. Background: `var(--background)`. Top border: `1px solid var(--border)`. Padding: `0 16px`. Font: 12px Geist regular, neutrals for labels, foreground for values.

---

## 2. Mode dot — when it appears

**Rule: dot appears only when there's a pending action.** When the system is waiting for the user's next click, show `●` + instruction. Otherwise no dot.

This automatically dedupes across canvases — for cross-canvas operations (Bind), only the canvas waiting for the click gets the dot.

| State | Dot | Color | Example |
|---|---|---|---|
| Ready (passive) | — | — | (hover/select content or empty) |
| Adding | ● | `--warning` orange | `● Adding: Pad-mount 500 kVA · click on map to place · ESC to cancel` |
| Connecting (no source) | ● | `--info` blue | `● Connecting: pick source terminal` |
| Connecting (source picked) | ● | `--info` blue | `● Connecting: TR-312 T1 → pick target` |
| Binding | ● | `--info` blue | `● Binding: pick a structure` |
| Aggregate edit | ● | `--warning` orange | `● Editing: Sub Main · drag in to add · Done to finish` |
| Sketch refine | ● | `--warning` orange | `● Refining: ?-007 → pick component type` |
| Trace done | — | — | `Trace from TR-312 upstream: 3 components · 2.3 km` (result, not pending) |

Save / Validating / Locked are global — they go in the header, not here.

---

## 3. Center content — passive states

### 3.1 Empty

Center is empty. Don't pad with placeholder text.

### 3.2 Hover

Pattern: `<ID/Name> · <Type> · <2–3 key facts>`.

Hover **always wins** over selection and active-mode instructions. mouseleave → returns to whatever was there before.

Hover delay: TBD in prototype testing — likely 0ms (instant) since 12px text in periphery is low-noise. If mouseover causes flicker, add 150ms.

When hover target equals current selection: no change to the bar (no flicker).

### 3.3 Hover content by object type

| Object | Pattern |
|---|---|
| Component | `TR-312 · Transformer · 500 kVA · bound to P-045` |
| Component (in aggregate) | `TR-312 · Transformer · 500 kVA · in Substation 1` |
| Component (in N aggregates) | `TR-312 · Transformer · 500 kVA · in 2 aggregates` |
| Component with errors | `TR-312 · Transformer · 500 kVA · ⚠ 2 issues` (chip in `--warning` orange, clickable → Info Panel) |
| Unspecified | `?-007 · Unspecified · pick a type` |
| Conductor (SLD) | `WIRE-018 · ACSR 795 · T1 ↔ T2 · 3-phase` |
| Conductor (Map) | `WIRE-018 · ACSR 795 OH · 320 m` |
| Pole / structure | `P-045 · Wood 45ft Class 2 · 3 mounted` |
| Vault / duct | `V-002 · 4-way 8'×10'×7' · 5 conductors` |
| Aggregate | `Substation 1 · Substation · 14 components · 2 transformers · 5 switches` |
| Region / zone | `California · Region · 12 substations · 4 zones` |
| Terminal (port) | `TR-312.T1 · Phase A · 12 kV` |

Per-view content differs intentionally — same conductor on SLD shows electrical facts (terminals, phases), on Map shows physical facts (length, OH/UG).

### 3.4 Selected (single)

Same content as hover, prefixed with `Selected:`. Persists until deselect or new selection.

```
Selected: TR-312 · Transformer · 500 kVA · bound to P-045
```

When user hovers another element while something is selected, hover **temporarily replaces** the center. mouseleave returns to the selected content.

### 3.5 Multi-select

**Homogeneous** (all same type) — count + sum of relevant total + breakdown:

```
Selected: 5 transformers · 1.4 MVA total · 3 bound, 2 unbound
Selected: 8 conductors · 2.1 km total · 6 OH, 2 UG
Selected: 3 poles · 12 components mounted total
```

**Heterogeneous** (mixed types) — count + breakdown, no sum:

```
Selected: 12 items · 8 components, 3 conductors, 1 pole
```

Hover during multi-select with Shift held — show what would be added:

```
+ Add: TR-312 · Transformer · 500 kVA
```

---

## 4. Center content — active modes

### 4.1 Add mode

Two sub-states:

| Sub-state | Center |
|---|---|
| Add panel open, no type chosen | `● Adding: pick component type` |
| Type chosen, placing on canvas | `● Adding: Pad-mount 500 kVA · click on map to place · ESC to cancel` |

Hover during Add mode shows hover content (overrides the mode instruction). mouseleave returns to instruction.

### 4.2 Connect mode

Two sub-states:

| Sub-state | Center |
|---|---|
| No source picked | `● Connecting: pick source terminal` |
| Source picked | `● Connecting: TR-312 T1 → pick target` |

Hover on a candidate target shows the candidate + compatibility hint:

```
TR-313 · Transformer · 500 kVA · ABC-N · ✓ compatible
TR-313 · Transformer · 500 kVA · A-N · ⚠ phase mismatch
```

### 4.3 Bind mode (cross-canvas)

User initiates Bind on SLD by selecting TR-312 → `Bind` action. Map then waits for a structure click.

```
SLD bar:   Selected: TR-312 · Transformer · 500 kVA          (passive)
Map bar:   ● Binding: pick a structure                        (active)
```

Hover on Map element while binding:

```
Map bar:   P-045 · Wood 45ft · 3 mounted                      (hover wins)
```

mouseleave on Map → returns to `● Binding: pick a structure`.

After successful bind:

```
SLD bar:   Selected: TR-312 · Transformer · 500 kVA · bound to P-045
Map bar:   Selected: P-045 · Wood 45ft · 4 mounted
```

### 4.4 Trace mode (result, no dot)

Trace highlighting spans both canvases. Both bars show the trace summary as if it were an extended selection:

```
SLD bar:   Trace from TR-312 upstream: 3 components · 2.3 km · ends at Sub North
Map bar:   Trace from TR-312 upstream: 3 components · 2.3 km · ends at Sub North
```

Variants:

```
Trace from TR-312 downstream: 5 components · 2 loads · 40 kW
Trace from TR-312 downstream: leaf (no further path)
Trace from TR-312 upstream: no path found
```

Hover on a non-trace element during trace mode → shows hover content normally. mouseleave → returns to trace summary.

### 4.5 Aggregate edit

User enters edit mode on a selected aggregate.

```
● Editing: Sub Main · 14 components · drag in to add · Done to finish
```

Hover on member → standard hover content. Hover on candidate from outside → `+ Add to Sub Main: TR-312 · Transformer · 500 kVA`.

### 4.6 Sketch refine

User clicks an unspecified component to assign a type.

```
● Refining: ?-007 → pick component type · current bindings will keep
```

---

## 5. Counters and overlays — single-zone behavior

### 5.1 Counters appear only in empty state

When nothing is selected, no mode is active, no overlay running — the bar shows per-view counters. This is the only state where they appear.

| View | Counters |
|---|---|
| Map | `Structures: 47 · Lines: 12 · Length: 2.3 km` |
| SLD | `Components: 156 · Connections: 89 · Aggregates: 2` |

Counters are rendered in muted color (neutral-500) to read as ambient info, not active content.

The moment user hovers, selects, or enters a mode — counters disappear, replaced by whatever the active content is. mouseleave / deselect / mode-exit returns counters.

### 5.2 Overlays live in the same zone

Filter, Trace, and Compare are not separate zones — they're just a different kind of content in the same single zone, with a `Clear` action at the tail.

```
Filter: de-energized · 12 of 156 visible · Clear
Trace from TR-312 upstream: 3 components · 2.3 km · ends at Sub North · Clear
Compare: +12 added · 4 modified · 1 removed · Clear
```

Per-view filter counts (12 of 47 on Map vs 12 of 156 on SLD) reflect what each canvas is showing — same overlay, different scope.

### 5.3 Stacking — none for v1

Multiple overlays at once (e.g. Filter + Trace) is rare and would overflow a 12px-text bar on a narrow viewport. For v1, only one overlay can be active at a time. New overlay deactivates the previous one. Revisit if real usage demands it.

---

## 6. Hover behavior — fine rules

1. **Hover always wins.** Active mode, selection, trace info — all are temporarily replaced by hover content.
2. **mouseleave returns to previous.** Always restores the state that was there before the hover started.
3. **No flicker on equal targets.** If hover target is the same as current selection, the bar doesn't change.
4. **Delay: 0ms by default.** Add 150ms only if testing reveals flicker on fast mouseover sequences.
5. **Hover content respects view.** SLD hover shows electrical facts; Map hover shows physical facts. Same object, different content.
6. **Shift+hover during multi-select** prefixes content with `+ Add:`.

---

## 7. What lives in header (not status bar)

- Project / branch / scope
- Errors counter (clickable → Validation Panel)
- Save state (Saving… / Saved / Unsaved)
- Validating progress
- Locked / view-only badge

These are global system state. Status bar stays focused on canvas content.

---

## 8. Open questions

- Hover delay (0ms vs 150ms) — decide after prototype testing.
- Cursor coordinates on Map — bar stays single-zone for v1; consider adding coords in a separate left zone in v2 if engineers ask.
- Multiple simultaneous overlays — disallowed for v1; revisit if real usage demands it.
