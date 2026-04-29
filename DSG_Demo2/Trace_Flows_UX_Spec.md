# Trace Flows — UX Spec

**Version:** 1.0 MVP · **Status:** Final for MVP · **Audience:** Design, Frontend, Backend
**Scope:** Select component or pole → Upstream / Downstream / Neighbors on Map and SLD.

> Backend algorithms (S1/S2/S3) and API endpoints are already specified. This document covers UX, frontend rendering, and integration. Visual details are resolved in Figma; behaviors not shown in static Figma frames live in the reference prototype `trace_prototype.html`.

---

## 1. Goals

Three user questions, each answered without leaving Map/SLD:

| Question | Flow | Endpoint |
|---|---|---|
| "Where does power to this come from?" | **Upstream** | `POST /analysis/path-to-source` |
| "What loses power if this opens/fails?" | **Downstream** | `POST /analysis/trace-downstream` |
| "What's directly connected?" | **Neighbors** (1-hop) | `GET /components/:id/chain` |

**Design principles:**
- Trace is a **filter and highlight**, not a mode. Click → result → Esc to clear.
- Same pattern across Map and SLD. The result is highlighted synchronously on both.
- No confirmation steps, no wizard. Result appears inline.
- No advanced parameters in MVP (phase, switch state, max depth — all defaults).

---

## 2. Primary user

**Planner / Network Engineer** — uses trace for **impact analysis** before making decisions. Typical scenarios:
- "I need to take SW-10 out for maintenance. What loses power?" → Downstream
- "Customer complaint from L-200. Trace power to source." → Upstream
- "What's connected to this junction?" → Neighbors

Validation/Field Engineers and Reviewers use the same UI — trace is universal.

---

## 3. Two-paradigm model

**Map = physical world.** Visible objects:
- **Poles** — physical anchors
- **Substation building** — its own physical structure
- **Conductors** between structures
- Electrical components mounted on a pole are **not rendered individually** on Map — they're "inside" the pole

**SLD = electrical world.** Visible objects:
- All **electrical components** (switches, transformers, junctions, loads, feeders, substations)
- **Connections** between them (including intra-pole)
- Poles are **absent** from SLD — they're a physical concept only

**Trace highlighting:**
- **On Map** — a pole is highlighted **as a whole** when at least one component mounted on it is part of the trace. Conductors between traced poles are also highlighted.
- **On SLD** — individual **components** and connections between them are highlighted.
- One trace, two levels of detail.

---

## 4. Pole-level trace

A pole is a physical container for electrically co-located components. A trace **from a pole** is semantically equivalent to a trace from any of its mounted components — the path to source / loads is the same.

**Implementation:** UI initiates trace from pole; the algorithm uses the first mounted component as the technical seed. UI displays the seed as the pole (e.g. `PL-121 (Pole)`).

**Counts diverge between Map and SLD** (and that's correct):
- Map: `PL-121 · Trace upstream: 2 components` (physical structures along the path)
- SLD: `TR-312 · Trace upstream: 3 components · 2.3 km` (electrical components)

This is visible in the per-view status bar.

---

## 5. Entry points

**Primary**
1. **Right-click a pole on Map** → context menu with pole-level Upstream / Downstream / Neighbors
2. **Right-click a component on SLD** → context menu with `Trace ▸` submenu

**Secondary**
3. **Keyboard** `⇧U` / `⇧D` / `⇧N` when something is selected (component or pole)
4. **Esc** — clear trace

**Out of MVP:**
- Per-component options inside the pole context menu (planned as v1.1+ enhancement, see §12)
- Toolbar entry point with dropdown
- Trace section inside Info Panel
- AI Assistant slash commands

### 5.1 Context menu — component (SLD)

```
MT-101
─────────────────────────
Show info
Add to Aggregate         ▸
Remove from Circuit 02
Connect to…
Disconnect Bus 12kV
Bind to structure…
Trace                    ▸ ──┬─ Trace upstream      ⇧U
Add to scope                  ├─ Trace downstream    ⇧D
Remove from scope             └─ Show neighbors      ⇧N
Edit in project…
Edit attributes
Duplicate
Delete
```

Trace items are disabled (with tooltip "Specify type to trace") when the component cannot be traced — sketch state, no terminals, no type.

### 5.2 Context menu — pole (Map)

```
PL-121 · Pole · 3 mounted
─────────────────────────
↑ Trace upstream         ⇧U
↓ Trace downstream       ⇧D
✦ Show neighbors         ⇧N
```

Pole-level trace covers the Planner's typical question — "where does power to this pole come from / what does it feed". Per-component initiation from the pole menu is **out of scope for MVP** (see §12).

### 5.3 Keyboard

- A **component** is selected → shortcuts trace from that component
- A **pole** is selected → shortcuts run pole-level trace
- Nothing selected → shortcuts do nothing

---

## 6. Trace states

### 6.1 Idle
Standard canvas. Selection ring on the selected item if any.

### 6.2 Running
Loading indication (spinner / skeleton). Typical durations: <500ms upstream, up to 2–3s downstream on large feeders.

### 6.3 Result — Found
- Trace highlight on both views (see §7)
- Floating "**Clear trace**" button at the bottom-center of the canvas
- Per-view status bar at the bottom shows summary on each side (see §8)

### 6.4 Result — Empty / Failed

Two MVP cases:

| reason | Trigger | Sonner |
|---|---|---|
| `no-path` | Upstream couldn't reach a substation | `No path to source from <seed>. An open switch may block the route, or this is an orphan component.` |
| `leaf` | Downstream from a leaf node | `<seed> is a leaf node — nothing downstream.` |

`<seed>` resolves to the component label (`L-999`) or pole ID (`P-99`) depending on the entry point.

Fallback: `No trace result.` — for cases where backend returns `found:false` without a `reason`.

### 6.5 Result — Error
Out of MVP toast set; production will add `Couldn't complete trace. Try again.` for network errors.

---

## 7. Visual language

### 7.1 Color tokens

| Role | Light | Dark |
|---|---|---|
| Selection ring | `blue-600` (#2563EB) | `blue-400` |
| **Trace highlight** | **`sky-500`** | **`sky-400`** |
| Non-trace (dimmed) | 50% opacity | 50% opacity |
| Open switch on path | `orange` dashed conductor | `orange` dashed conductor |

### 7.2 Map — trace highlight

- **Pole in trace:** dot fill + cross-arm in `sky-500`, label sky bold, soft halo behind the pole
- **Pole not in trace:** dimmed 50%
- **Conductor in trace:** `sky-500`, +1px stroke
- **Conductor with open switch on the path:** orange dashed (regardless of trace state — this is a separate signal)

### 7.3 SLD — trace highlight

- **Component in trace:** `sky-500` outline, sky-wash fill, sky bold label
- **Component not in trace:** dimmed 50%
- **Connection in trace:** `sky-500`, +1px stroke
- **Selected component on top of trace:** `blue-600` ring drawn over the sky fill (selection wins visually)

### 7.4 Pulse

Seed pulses 1–2 times (~800ms total) when the trace result arrives. No persistent animation on the trace itself.

---

## 8. Status bar — per-view

The bottom status bar has two independent sections, one per view:

```
[ Selected: PL-121 (Pole)   |   Trace upstream: 2 components ]
                                                              [ Selected: TR-312 (Transformer) | Trace upstream: 3 components · 2.3 km ]
```

**Each section contains:**
- Left: `Selected: <label> (<type>)` or `No selection`
- Right: trace summary (only when trace is active)

**Trace summary by type:**

| Kind | Map | SLD |
|---|---|---|
| Upstream | `Trace upstream: N components` | `Trace upstream: N components · ~X km` |
| Downstream | `Trace downstream: N components` | `Trace downstream: N components · M loads · ~Z kW` |
| Neighbors | `Trace neighbors: N neighbors · 1-hop` | `Trace neighbors: N neighbors · 1-hop` |

On empty/failed trace, the section uses a warning style with text like `Trace upstream: no path to source` or `Trace downstream: leaf — nothing downstream`.

---

## 9. Floating "Clear trace"

Appears at the bottom-center between Map and SLD whenever a trace is active. Icon ✕ + label "Clear trace". Click → removes the highlight, status bar reverts to pre-trace state.

---

## 10. Reset / Clear

| Action | Result |
|---|---|
| `Esc` | Clears trace. Selection stays. |
| Floating "Clear trace" | Same as Esc. |
| Click on empty canvas | Clears both selection and trace. |
| Run new trace | Replaces previous trace. |
| Select a different component/pole | Clears previous trace. New trace does not auto-run. |

---

## 11. Edge cases

### 11.1 Loops (ring topologies)
BFS returns the shortest path. The other side of the ring will be dimmed — the user sees there's an alternate route. UI for "multiple paths" is out of MVP.

### 11.2 Islands (orphan clusters)
Upstream → empty `no-path`. Downstream works inside the island. Neighbors work normally.

### 11.3 Open switch on path
Visible on Map as an orange dashed conductor. Combined with the `no-path` sonner, it's the primary signal for the most common real-world failure mode.

### 11.4 Pole with electrically separated mounted components
The algorithm picks the first mounted component as seed. If the pole has electrically separated groups, the result reflects the first mounted component. This is an MVP approximation; v1.1+ may add union-of-traces from all mounted components.

### 11.5 Large downstream tree
Backend default `maxDepth = 100`. MVP doesn't surface truncation in UI — handled in v1.1.

### 11.6 Trace on substation
Substation is its own physical anchor (no pole). Trace from a substation behaves normally — upstream returns `alreadyAtSource`, downstream returns the entire reachable network.

### 11.7 Sketch-state components
Without type/terminals, trace is unavailable. Trace items in the context menu are disabled with the tooltip "Specify type to trace".

---

## 12. Out of MVP scope

Captured for v1.1+:
- **Per-component trace from pole context menu** — when a pole has >1 mounted components, the menu would expose a secondary section listing each mounted component with its own Upstream/Downstream/Neighbors icons. Useful for power users investigating a specific phase or a specific mounted device on a complex pole.
- Trace section inside Info Panel
- Toolbar entry point with dropdown
- AI Assistant slash commands (`/trace upstream @T-42`)
- Multiple simultaneous traces
- Iterative neighbor expansion (2-hop, 3-hop)
- Editable switch state / max depth in UI
- Export result (CSV, GeoJSON, deep link)
- Trace history
- Animated direction pulse along the path

---

## Appendix A — Backend contract

| Flow | Endpoint | Returns |
|---|---|---|
| Upstream | `POST /analysis/path-to-source` | `pathComponentIds`, `pathConnectionIds`, `sourceComponentId`, `found`, `reason` |
| Downstream | `POST /analysis/trace-downstream` | `reachedComponentIds`, `reachedConnectionIds`, `loadComponentIds`, `totalReached`, `depth` |
| Neighbors | `GET /components/:id/chain` | Array of `{component, connections[]}` |

All requests use backend defaults — no explicit `phase` or `switchState` parameters from the UI.

---

## Related

- S1 Get Neighbors · S2 Trace Upstream · S3 Trace Downstream — algorithm specs
- UC04 Network Topology — business context, especially Step 4 (Path-to-Power-Source check)
- Visualization Standards — Part 4 (dual-panel synchronization, highlighting hierarchy)
- Reference prototype: `DSG_Demo2/trace_prototype.html` (interactive companion to this spec)
