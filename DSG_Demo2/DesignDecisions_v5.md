# DSG Demo 2 — Design Decisions (v5)

Consolidated decisions across all four flows. Current as of April 2026.
Prototype: `prototype-v5-sketch.html`

---

## 1. Add Component

| Decision | Choice | Rationale |
|----------|--------|----------|
| Sketch-first approach | User places generic components (type category only), specifies variant later | Mirrors wireframe→hi-fi workflow. Lets users lay out topology fast without getting stuck in details |
| Three entry points | Left palette (drag), context menu (instant place), Add Panel (full library) | Different speeds for different workflows: quick sketch, precise placement, detailed selection |
| Generic component badge | Orange "?" (HelpCircle) 12×12 | Soft warning — "fill in later", not an error |
| Variant specification | Info Panel → orange "Specify type" section at top → pick from list | No separate modal — stays in context, same panel used for all editing |
| Left palette collapsed | 40px wide, 32×32 icon buttons, tooltip on hover | Minimal footprint, doesn't obscure diagram |
| Left palette expanded | 2×4 grid, 80×60 buttons with labels, gap 4px | Matches Figma mockup. Large enough touch targets with readable labels |
| Palette footer (collapsed) | "..." + expand — stacked vertically | Fits 40px width constraint |
| Palette footer (expanded) | "All components" text + collapse — side by side | Horizontal layout when space allows |
| Add Panel layout | 280px, search (fixed) + icon grid 3 cols + type list (upper scroll) + Recent ~200px (lower scroll) | Two independent scroll areas prevent Recent from being buried. All items draggable |
| Context menu submenu | 8 type names + separator + "All components" | Quick access without opening panel. "All components" for full library |
| SLD types | 8: Connector, Transformer, Switch, Shunt, Voltage Regulator, Protection, Meter, Sectionalizer | Per documentation — covers all electrical component categories |
| Map structures | 4: Pole, Vault, Substation, Duct | Per documentation: "21 types + 5 structures" minus Conduit (merged into Duct as attribute) |
| Pad type | Removed | Pad is a variant of Vault or Substation, not a separate structure category |
| Placement from drag | Drop at cursor position, instant | No confirmation step |
| Placement from context menu | Instant at right-click position | No placement mode needed — position already known |
| Placement from panel click | Crosshair placement mode, click to place | Position unknown, user picks location |
| Serial placement | Shift+click stays in placement mode | Matches Bind/Mount picker pattern |
| Toast on placement | "{Label} placed — specify type in Info Panel" | Guides user to next step without forcing it |

## 2. Connect

| Decision | Choice | Rationale |
|----------|--------|----------|
| Primary connection method | Drag from port to port | Most intuitive, direct manipulation |
| Port visibility | Hidden by default, appear on hover | Keeps diagram clean, ports discoverable |
| Line routing | Orthogonal 4-segment paths with draggable midpoints | Industry standard for SLD diagrams |
| Conductor picker | 22 types, search + recent + scrollable list | Appears when connecting distant components that need a conductor |
| Phase compatibility feedback | Green (full) / Orange (partial) / Red (incompatible) | Real-time visual during drag — uses shadcn CSS vars (--success, --warning, --destructive) |
| Multi-select connect | Shift+click two components → Enter | Faster than drag for non-adjacent components |
| Connect to... list | Context menu → Connect to... → scrollable list | For large diagrams where target is off-screen |
| Midpoint handles | Purple handles on hover, drag to adjust route | Direct manipulation of line geometry |
| Map reflection | OH = solid line, UG = dashed between nearest structures | Automatic — no manual map wiring needed |
| Dashed lines | Reserved for UG conductors and "as planned" state | Per design system — solid = OH/active, dashed = UG/planned |
| Disconnect | Context menu → Disconnect → per-connection list | Explicit, prevents accidental disconnects |

## 3. Bind

| Decision | Choice | Rationale |
|----------|--------|----------|
| Binding model | `bindings: []` array, one-to-many | One electrical component can span multiple structures |
| Bind picker pattern | Search + recent + scrollable list, single click exits, Shift+click stays | Identical to Mount picker — symmetric patterns |
| Pick on map | Banner "Click a structure on the map to bind" + click | Spatial selection, active panel outlined with selection color |
| Mount Equipment | Reverse direction — from structure card | Same picker pattern, opposite entry point |
| Prepositions | ON (Pole), IN (Vault, Duct), AT (Substation) | Auto-assigned based on structure type |
| Unbound indicator | Grey Unlink icon 12×12 | Soft warning — "bind later", not blocking |
| Rebind | Button in binding list → reopens picker | No unbind-then-rebind — single action |
| Active panel highlight | CSS `outline` with `--selection` color | Not box-shadow — avoids layout shift |

## 4. Aggregate

| Decision | Choice | Rationale |
|----------|--------|----------|
| MVP templates | 3: Substation, Circuit, Feeder | From LIB-16 documentation. Hardcoded vendor templates |
| Bottom-up creation | Select components → right-click → Add to Aggregate → submenu | Contextual, works with existing selection |
| Top-down creation | Right-click empty → Add aggregate → empty container | For planning before placing components |
| Drag-in | Drag component onto aggregate area → instant add → snackbar with Undo | Gmail-style — no Yes/No dialog. Faster workflow |
| Drag-in confirmation | Snackbar with Undo (not dialog) | Per Peter's feedback — instant action + undo beats confirmation dialogs |
| Visual: expanded | Tinted background + label (purple/blue/amber by type) | Light, doesn't interfere with component rendering |
| Visual: collapsed | Compact block "{Name} ({N} components)" | EXISTS IN PROTOTYPE but NOT in requirements. Peter removed from Figma mockups |
| Visual: empty | Dashed placeholder "Drag components here" | Clear affordance |
| Visual: map | Polygon boundary around GPS points of children | Automatic geometry from child positions |
| Info Panel | Click aggregate background → type, status badge, member list, composition rules | Reuses Info Panel — no separate aggregate panel |
| Composition validation | Inline ✓/⚠/✕ per rule in Info Panel | Real-time feedback while building |
| Dissolve | Right-click → Dissolve aggregate. Container removed, children stay | Matches documentation "Dissolve" operation |
| Nesting | Not in MVP | Single level. Full hierarchy (Region → Substation) deferred |
| GPS validation | Not in aggregate rules | Belongs to Verify stage, not composition |

## 5. View / Edit Mode

| Decision | Choice | Rationale |
|----------|--------|----------|
| Default mode | View | Safe default — user can't accidentally modify |
| View toolbar | [Select] [Edit] — two buttons | Compact. Edit button is entry point |
| Edit toolbar | [Select] [Pan] | [Add] | [Fit] | [✓ Done] | Done is primary-styled (dark bg + check icon) |
| Enter edit | Edit button or E key | Quick keyboard access |
| Exit edit | Done button | Explicit exit. No accidental mode switch |
| View mode restrictions | Palettes hidden, drag/connect/add disabled, context menu shows only info/locate | Read-only browsing, Info Panel still works |
| Context menu (View, empty) | "Edit mode (E)" | Discoverable entry to editing |

## 6. Error Badge System

| Decision | Choice | Rationale |
|----------|--------|----------|
| Badge count | 3 icons total | Minimal set covers all cases |
| ? (HelpCircle) | Grey, 12×12 — unspecified type or attributes | Soft: "fill in later" |
| Unlink | Grey, 12×12 — not bound to physical structure | Soft: "bind later" |
| ! (AlertCircle) | Orange, 12×12 — any hard validation error | Hard: phase mismatch, ampacity, orphaned, broken binding, composition |
| Priority | ! > Unlink > ? | Only highest shown. One badge per component |
| Hover | Dark tooltip with all issues: "[V-U01] Type not specified" | Full detail without opening panel |
| Click | Opens Info Panel | Direct path to fix the issue |
| ZapOff (orphaned) | Rejected — merged into ! | Восклицательного знака достаточно для всех ошибок |

## 7. Selection Color

| Decision | Choice | Rationale |
|----------|--------|----------|
| Light theme | `--selection: #2563EB` (blue-600) | Reserved exclusively for selection, not semantic |
| Dark theme | `--selection: #60A5FA` (blue-400) | Lighter shade for dark backgrounds |
| Wash | `--selection-wash: rgba(37,99,235,0.08)` | For selected item backgrounds |
| Previous color | `#00BFFF` replaced everywhere | Unified via CSS variable, single point of change |
