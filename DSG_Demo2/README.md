# DSG Demo 2 — Component Interaction Flows

Interactive HTML prototype for GridModel SLD + Map editor.

| Flow | Status | Version |
|------|--------|----------|
| Add | Done | v1 → v5 |
| Connect | Done | v2 → v5 |
| Bind | Done | v3 → v5 |
| Aggregate | Done | v4 → v5 |
| Sketch-first (palettes, generic components) | Done | v5 |
| Edit / View mode | Done | v5 |

## Files

| File | Description |
|------|-------------|
| `prototype-v5-sketch.html` | **Current** — full prototype with all flows |
| `connect-flow-prototype-v3.html` | Previous version (Add + Connect + Bind only) |
| `PrototypeSummary_v5.md` | Complete feature summary |
| `AggregateFlow_Summary.md` | Aggregate flow design decisions |
| `BindFlow_Summary.md` | Bind flow design decisions |
| `ConnectFlow_Summary.md` | Connect flow design decisions |

## Live preview

Enable GitHub Pages (Settings → Pages → main branch), then open:

```
https://petrygin.github.io/GridSystems/DSG_Demo2/prototype-v5-sketch.html
```

Or use htmlpreview:

```
https://htmlpreview.github.io/?https://github.com/petrygin/GridSystems/blob/main/DSG_Demo2/prototype-v5-sketch.html
```

## How to use

Download `prototype-v5-sketch.html` and open in browser, or use live preview link above.

### Modes

The prototype starts in **View mode** (read-only). Click **Edit** button or press **E** to enter Edit mode.

| Mode | Toolbar | Capabilities |
|------|---------|-------------|
| View | [Select] [Edit] | Browse, Info Panel, right-click → info only |
| Edit | [Select] [Pan] [Add] [Fit] [✓ Done] | Full editing: add, connect, bind, aggregate |

### Adding components (Edit mode)

| Method | How |
|--------|-----|
| Palette drag | Drag type icon from left palette onto diagram |
| Right-click | Right-click → Add component → select type |
| Add panel (A) | Press A → Quick add grid or From library |
| Shift+click | Serial placement (stay in placement mode) |

Components are placed as **generic** (orange ? badge). Specify exact variant later via Info Panel.

### Connecting

| Method | How |
|--------|-----|
| Port drag | Hover component → port appears → drag to target port |
| Connect to... | Right-click → Connect to... → select from list |
| Multi-select | Click + Shift+click two components → Enter |

### Binding (electrical → physical)

| Method | How |
|--------|-----|
| From component | Click component → Info Panel → + Add binding |
| Pick on map | Info Panel → Add binding → click structure on map |
| From structure | Click structure on map → Mount equipment |
| Multi-bind | Hold Shift while clicking (stay in picker mode) |

### Aggregating

| Method | How |
|--------|-----|
| Bottom-up | Select components → right-click → Add to Aggregate |
| Top-down | Right-click empty → Add aggregate → drag components in |
| Drag-in | Drag component onto aggregate area (snackbar + undo) |
| Dissolve | Right-click aggregate → Dissolve |

### Keyboard shortcuts

| Key | Action |
|-----|--------|
| E | Enter Edit mode |
| V | Select tool |
| H | Pan tool |
| A | Open Add panel |
| Enter | Connect two selected |
| Cmd/Ctrl+Z | Undo (50 steps) |
| Esc | Cancel any mode |

## Tech

Standalone HTML, ~180KB, ~3350 lines. Vanilla JS, no dependencies. SVG rendering.
