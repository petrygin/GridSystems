# Trace Flows — UX Spec

**Version:** 0.1 · **Status:** Draft · **Audience:** UI/UX Designer, Frontend
**Scope:** Select component → Upstream / Downstream / Neighbors tracing on Map and SLD views.

> Designs the user-facing flow for three graph traversal operations. Backend algorithms (S1/S2/S3) and API endpoints (`/analysis/path-to-source`, `/analysis/trace-downstream`, `/components/:id/chain`) are already specified. This document covers UX only — visual details (exact shades, stroke weights, animation curves) are resolved in Figma.

---

## 1. Goals

Users exploring the grid graph need to answer three questions fast, without leaving the Map/SLD view:

| Question | Flow | Backend |
|---|---|---|
| "Where does power to this come from?" | **Upstream** | `/analysis/path-to-source` |
| "What loses power if this opens/fails?" | **Downstream** | `/analysis/trace-downstream` |
| "What's directly connected to this?" | **Neighbors** (1-hop) | `/components/:id/chain` |

**Design principles:**
- Trace is a **filter/highlight**, not a mode. Click → result → Esc to clear.
- Same interaction pattern across Map and SLD. Results sync between both views.
- No confirmation steps. No wizard. Result appears inline.
- Advanced parameters (phase, switch state, max depth) stay hidden until context demands them.

---

## 2. Key Concepts (for designers)

### 2.1 Upstream vs Downstream vs Neighbors

- **Upstream** returns a **line** — the shortest path from selected component back to the nearest substation. Output is ordered: `[selected → hop1 → hop2 → ... → substation]`.
- **Downstream** returns a **tree** — everything reachable going away from source, fanning out through branches and junctions. Output is a set of components + connections, with terminal loads flagged separately.
- **Neighbors** returns a **ring** — components directly connected to the selected one, one hop away. Typically 1–4 items for most equipment.

Visual implication: upstream highlights a single chain. Downstream highlights a fan. Neighbors highlights a handful of adjacent components. The three results should feel visually distinct at a glance.

### 2.2 Phase — when and why it matters

Distribution grids carry power on three separate wires: **phase A, B, C**. Three-phase equipment (large transformers, main switches) uses all three. Single-phase equipment (residential transformers, individual meters) uses only one.

A trace walks through **one phase at a time**, because a switch can be closed on phase A but open on phase B — different results for different phases.

**UX rule:** we auto-detect phase from the selected component. If the component has only one phase, we silently use it. If it has multiple (ABC, AB, AC, etc.), we show a small `Phase: A ▾` selector in the Results Panel so the user can re-run for a different phase.

### 2.3 Switch state

By default the backend skips **open switches** — tracing follows only the currently-energized path. For MVP this is the only mode we expose. "Include open switches" toggle is deferred.

---

## 3. Entry Points

Priority order:

1. **Context menu** (right-click on component) — *primary*
2. **Info Panel** (when component is selected) — *equal secondary*
3. **Toolbar** (global, works on current selection) — *future, kept in scope for mental model*

### 3.1 Context menu

Right-click on a component on Map or SLD:

```
┌─────────────────────────────┐
│  Info                       │
│  ─────────────────────────  │
│  Trace upstream          ⇧U │
│  Trace downstream        ⇧D │
│  Show neighbors          ⇧N │
│  ─────────────────────────  │
│  Edit attributes            │
│  Delete                     │
└─────────────────────────────┘
```

Items appear only when a valid component is right-clicked. Items are disabled (greyed out) if the component cannot be traced (e.g., unspecified type, no terminals).

### 3.2 Info Panel

When a component is selected and Info Panel is open, a **Trace** section lives below primary attributes:

```
┌─────────────────────────────┐
│  Transformer T-42           │
│  Type: Pad-mounted 25kVA    │
│  ...                        │
│  ─────────────────────────  │
│  Trace                      │
│  [Upstream] [Downstream]    │
│  [Neighbors]                │
└─────────────────────────────┘
```

Three buttons, compact size. Clicking one runs the trace and opens Results Panel.

### 3.3 Toolbar (future)

A single **Trace ▾** button in the main toolbar with a dropdown (Upstream / Downstream / Neighbors). Acts on current selection. Disabled when nothing is selected. Called out here so the pattern is consistent — not shipping in v1.

### 3.4 Keyboard

Shortcuts match the context menu labels. Work when any component is selected on Map or SLD:

- `⇧U` → Trace upstream
- `⇧D` → Trace downstream
- `⇧N` → Show neighbors
- `Esc` → Clear trace (see §8)

---

## 4. Flow — Happy Path

1. User selects component (click on Map or SLD).
2. User invokes trace (right-click → "Trace upstream" / Info Panel button / shortcut).
3. **Loading state** — selected component pulses briefly; cursor shows wait indicator if > 200ms.
4. **Result state** — path/tree/ring highlights simultaneously on Map and SLD. Results Panel slides in.
5. User inspects. Can click any item in Results Panel → Map and SLD pan+zoom to it.
6. User presses `Esc` or clicks "Clear" → all highlights removed, Results Panel closes.

Running a new trace (of any type) from any entry point **replaces** the previous result without asking. Only one trace highlight is shown at a time.

---

## 5. States

### 5.1 Idle — no trace active
Normal canvas. No special highlighting beyond standard selection.

### 5.2 Running
- Selected component pulses.
- Entry point button shows loading spinner (Info Panel) or items remain visible with skeleton in Results Panel.
- Duration: typically <500ms for upstream on small models, can be up to 2–3s for downstream on large feeders.

### 5.3 Result — Found
- Highlighted components visible on both Map and SLD.
- Selected component retains its selection ring on top of trace highlight (selection wins visually).
- Results Panel populated.

### 5.4 Result — Not found / empty
Covers three distinct cases — each gets a clear empty-state message in the Results Panel:

| Case | Backend signal | Message |
|---|---|---|
| Disconnected from source | `found: false, reason: "No path to source"` | "No path to source found. Possible open switch or orphaned equipment." |
| Component has no matching phase | `found: false, reason: "No terminal with phase X"` | "This component has no phase X. Try phase [list of available]." |
| Downstream tree is empty | `reachedComponentIds: []` | "Nothing downstream — this is a leaf node." |
| Neighbors returns empty | `[]` | "No neighbors — this component is not connected." |

Empty states include a subtle illustration + the text + a "Clear" button. No red/orange — these are informational, not errors.

### 5.5 Error — network/backend failure
Toast: "Couldn't complete trace. Try again." with retry. Trace state clears.

---

## 6. Visual Language

### 6.1 Color roles (actual tokens resolved in Figma)

| Role | Token (approximate) | Where |
|---|---|---|
| Selection ring | `--selection` (blue-600 `#2563EB`) | Selected component |
| Trace highlight — components | `--trace` (cyan/teal, `~#0EA5E9` or similar) | All components in trace result |
| Trace highlight — connections | same as components | Lines between traced components |
| Source marker | `--trace-source` (stronger shade of same hue) | Substation at the end of upstream |
| Terminal load marker | `--trace-load` (same hue, lighter) | Loads at the ends of downstream |
| Hover (inside trace) | `--selection` on top of trace highlight | Component being pointed at |

Cyan/teal is chosen because it is visually distinct from orange-500 accents, blue-600 selection, green/red status badges, and the voltage-coded line colors on the map. Exact shade is confirmed in Figma against the full design system.

### 6.2 Stroke and fill treatment

- **Components in trace:** filled with trace color at lower opacity (~15–25%), outlined at full opacity. Keeps the equipment symbol readable.
- **Connections in trace:** recolored to trace color, stroke weight increased (+1px).
- **Non-trace components:** dimmed to 40–50% opacity so the highlighted set pops. This is the primary "focus the eye" mechanism.
- **Selected component on top of trace:** keeps blue-600 selection ring, rendered above trace layer.

Dashed lines remain reserved for UG conductors and "as planned" states (per existing design system). Trace does **not** use dashing.

### 6.3 Pulse / animation

Only the **selected component pulses** briefly (1–2 times, 800ms total) when the trace result arrives. This draws attention to the starting point without making the whole canvas flicker. No persistent animation on the trace itself.

### 6.4 Map vs SLD — differences

| | Map | SLD |
|---|---|---|
| Components | Circle/icon fill + outline in trace color | Symbol fill + outline in trace color |
| Connections | Geographic polylines (may be long, curved) | Orthogonal lines |
| Out-of-viewport traced items | Mini-markers at canvas edge pointing to off-screen results | Same pattern |
| Zoom behavior | User can "Fit trace to view" via button in Results Panel | Same |

Synchronization: triggering trace on Map highlights on both panels simultaneously. Clicking an item in Results Panel pans+zooms **both** panels to it.

---

## 7. Results Panel

Slides in from the right side (or docks below existing panels, TBD in Figma layout). Width ~320px.

### 7.1 Layout

```
┌────────────────────────────────┐
│ Trace upstream            × │  ← header: trace type + close
├────────────────────────────────┤
│ From: Transformer T-42         │  ← starting component
│ Phase: A ▾                     │  ← only if component has >1 phase
├────────────────────────────────┤
│ Path · 4 hops · 2.3 km         │  ← summary stats
├────────────────────────────────┤
│ ① T-42 — Transformer           │  ← list, clickable
│ ② SW-12 — Switch (closed)      │
│ ③ F-03 — Feeder                │
│ ④ Sub-North — Substation  ★    │  ← source marked
├────────────────────────────────┤
│ [Fit to view] [Clear trace]    │  ← footer actions
└────────────────────────────────┘
```

### 7.2 Content by trace type

| | Upstream | Downstream | Neighbors |
|---|---|---|---|
| Header | "Trace upstream" | "Trace downstream" | "Neighbors of..." |
| Summary | "N hops · X km · to [Source]" | "N components · M loads · depth D" | "N neighbors" |
| List | Ordered path, source at bottom marked ★ | Grouped by branch (or flat list sorted by depth). Loads marked 🔌 | Flat list. Each item shows connection label if present |
| Empty state | "No path to source..." | "Nothing downstream..." | "No neighbors" |

### 7.3 Interactions

- **Click item** → Map and SLD pan+zoom to it, item becomes selected (blue ring appears), trace remains.
- **Hover item** → that component pulses on both canvases (same hover sync as everywhere else).
- **Phase selector** (upstream/downstream only, only if multi-phase) → re-runs trace with new phase, list updates in place.
- **Fit to view** → both canvases zoom to bounding box of trace result.
- **Clear trace** → removes all highlights, closes panel, keeps selection.
- **Close (×)** → same as Clear trace.

### 7.4 Advanced settings (collapsed by default)

At the bottom of the panel, a small `⚙ Advanced` chevron expands to show:

```
Switch state: ○ Energized (respect open)  ○ Connectivity only
Max depth: [100]   ← downstream only
```

For MVP these can be read-only (show defaults) so users understand what's happening. Making them editable is a v1.1 item.

---

## 8. Clear & Reset Rules

Multiple ways to end a trace — all lead to the same clean state:

| Action | What happens |
|---|---|
| Press `Esc` | Clear trace highlight. Results Panel closes. Selection stays. |
| Click empty canvas (Map or SLD) | Clear selection + trace. |
| Click "Clear trace" button | Clear trace highlight. Panel closes. Selection stays. |
| Close Results Panel (×) | Same as "Clear trace". |
| Run new trace | Previous trace replaced. New result shown. |
| Select different component | Previous trace cleared. Selection updates. No new trace runs automatically — user must invoke. |
| Edit model in any way (v1.1+) | Trace invalidated with toast "Topology changed — trace cleared". |

---

## 9. Edge Cases

### 9.1 Loops (ring topologies, normally-open points)
BFS returns the shortest path. User sees one path, not multiple. If the loop is visible within Map/SLD extent, the *other* side of the ring will be dimmed (not-in-trace), making it obvious there is another route. No special UI for "multiple paths" in MVP.

### 9.2 Islands (disconnected subgraphs)
Upstream: returns empty with "No path to source" message (see §5.4). Downstream: works normally within the island. Neighbors: works normally.

### 9.3 Large downstream tree
If the tree is huge (1000+ components), we rely on the backend `maxDepth` default of 100. Results Panel shows `depth: 100 (truncated)` hint. Deferred: a "Load more" control to extend depth.

### 9.4 Unspecified / sketch-state components
Components without fully specified type/terminals may not trace. Context menu item is disabled with tooltip: "Specify type to trace". Info Panel buttons same behavior.

### 9.5 Trace starts on conductor or junction
Allowed. Start node is still treated as the seed. For conductors the "selected component" in Results Panel reads e.g. "Line L-42 (conductor)".

### 9.6 Mid-trace view switch (user toggles Map-only / SLD-only)
Trace persists. Highlighting continues on the visible canvas. When both reappear, both stay in sync.

---

## 10. Out of Scope (v1)

Deferred to v1.1 or later, captured here for alignment:

- **Multiple simultaneous traces** (compare upstream of A vs upstream of B side by side)
- **Expand neighbors iteratively** (2-hop, 3-hop from Neighbors result)
- **Include open switches / connectivity-only mode** as user-editable toggle
- **Editable max depth** for downstream
- **Save / export trace result** as CSV, GeoJSON, or shareable link
- **Trace history** (last N traces in a dropdown)
- **Animation of trace direction** (pulse traveling along the path)
- **Toolbar entry point** with dropdown
- **Slash commands integration** (`/trace upstream @T-42` in AI Assistant chat — already specified in AI Assistant spec, but doesn't block this flow)

---

## 11. Consistency Notes

- Selection color (blue-600) and trace color (cyan/teal) are **different hues**. Never swap. A traced component that is also selected shows both — selection ring on top.
- Context menu items ordering matches across Map and SLD.
- Shortcuts work identically on Map and SLD focus.
- Empty states use the same illustration style as elsewhere in the app (validation empty, filter empty).
- Results Panel layout mirrors the Validation Results Panel pattern (header + grouped list + footer actions) to leverage existing user familiarity.

---

## 12. Open Questions

Listed for follow-up, not blocking design:

1. **Location of Results Panel** — right side docked, or floating card near selection? Figma exploration needed.
2. **Highlighting priority during validation** — if a component has a validation error badge AND is in a trace result, which color wins? (Proposal: validation error stays visible on the badge; trace highlight applies to the body.)
3. **Trace from Tree Panel** — should right-click in Tree Panel also offer Trace? (Proposal: yes, for consistency, but lower priority than canvas entry points.)
4. **Phase "All phases"** — is there a real user need to show ABC combined? API doesn't support it natively; would require 3 backend calls merged client-side. Probably not for MVP.

---

## Appendix A — Backend Contract Summary

| Flow | Endpoint | Returns | Notes |
|---|---|---|---|
| Upstream | `POST /analysis/path-to-source` | `pathComponentIds`, `pathConnectionIds`, `visitedComponentIds`, `sourceComponentId`, `found`, `reason` | `phase` defaults to "A". Skips OPEN switches. |
| Downstream | `POST /analysis/trace-downstream` | `reachedComponentIds`, `reachedConnectionIds`, `loadComponentIds`, `totalReached`, `depth` | `maxDepth` defaults to 100. |
| Neighbors | `GET /components/:id/chain` | Array of `{component, connections[]}` | 1-hop only. Response includes full component props and connection labels. |

For full payloads see `04_Engineering/04.07_API/API_Spec_HierarchyTree/part3_tracing.md` in `gridmodel-docs`.

---

## Related

- S1 Get Neighbors · S2 Trace Upstream · S3 Trace Downstream (algorithms)
- UC04 Network Topology (business rules, especially Step 4 — Path to Power Source)
- Visualization Standards — Part 4 (dual-panel behavior, highlighting hierarchy)
- AI Assistant — UI States & Interaction Scenarios (`/trace` slash command, Trace Path response type)
