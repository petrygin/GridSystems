# DSG Demo 2 — Component Interaction Flows

| Flow | Doc | Prototype | Status |
|------|-----|-----------|--------|
| Add | — | v3 HTML | Done |
| Connect | Summary + Discussion v0.2 | v3 HTML | Done |
| Bind | Summary + Discussion v0.1 | v3 HTML | Done |
| Aggregate | — | — | Next |

## Files

- `ConnectFlow_Summary.md` — Connect flow results
- `BindFlow_Summary.md` — Bind flow results (includes all iteration updates)
- `BindFlow_Discussion_RU.md` — Bind discussion doc (RU)
- `connect-flow-prototype-v3.html` — Interactive prototype (Add + Connect + Bind)

## How to use prototype

Download `connect-flow-prototype-v3.html` and open in browser.

### Key interactions

| Action | How |
|--------|-----|
| Add component (SLD) | Right-click → Add → Category → Variant (placed at click pos) |
| Add component (drag) | Open Add panel → drag variant from list onto SLD |
| Add structure (Map) | Right-click Map → Add structure → Pole/Vault/Pad |
| Connect (drag) | Hover → port appears → drag to target port |
| Connect (list) | Right-click → Connect to... → select from list |
| Connect (select two) | Click + Shift+click → Enter |
| Disconnect | Right-click → Disconnect → specific connection |
| Bind | Click component → Info Panel → + Add binding → pick on map |
| Bind (reverse) | Click structure on Map → Mount equipment → select component |
| Unbind | Info Panel → X button on specific binding row |
| Drag component | Click + drag (5px threshold) |
| Drag connection | Hover line → purple handles → drag |
| Undo | Cmd+Z / Ctrl+Z (50 steps) |

### Keyboard shortcuts

| Key | Action |
|-----|--------|
| V | Select mode |
| H | Pan mode |
| A | Open Add panel |
| Enter | Connect two selected |
| Cmd+Z | Undo |
| Esc | Cancel any mode |
