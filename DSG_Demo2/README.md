# DSG Demo 2 — Component Interaction Flows

| Flow | Status |
|------|--------|
| Add | Done |
| Connect | Done |
| Bind | Done |
| **Aggregate** | Next |

## Files

- `BindFlow_Summary.md` — Bind flow results (includes all updates)
- `ConnectFlow_Summary.md` — Connect flow results
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
| Bind | Click component → Info Panel → + Add binding → pick on map or list |
| Bind multi | Hold Shift while clicking structures (list or map) |
| Mount (reverse) | Click structure on Map → Mount equipment → select component |
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
