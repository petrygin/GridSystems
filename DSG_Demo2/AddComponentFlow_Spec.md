# Add Component Flow — Subtask Spec

**Figma:** Grid Model Work in Progress → Add component panel, Context menu

---

## Entry Points

### 1. Left Palette (drag-and-drop)

Sidebar anchored to the left edge. SLD has 8 electrical types, Map has 4 structure types.

| State | Layout |
|-------|--------|
| **Collapsed** | Single column, 40px wide, 32×32 icon buttons, tooltip on hover. Bottom: "..." (opens full panel) + expand toggle — stacked vertically |
| **Expanded** | SLD: 2×4 grid, 80×60 buttons with labels. Map: 2×2 grid. Bottom: "All components" text + collapse toggle — side by side |

Grab icon → drag onto canvas → generic component placed at drop point.

### 2. Context Menu (right-click empty space)

**SLD:** "Add component >" submenu → Connector, Transformer, Switch, Shunt, Voltage Regulator, Meter + separator + "All components". Plus "Add aggregate" as separate item.

**Map:** "Add structure >" submenu → Pole, Vault, Substation, Duct.

Click type → component placed instantly at cursor position. "All components" → opens full panel.

### 3. Add Component Panel

Full-height side panel (280px). Opened from: "All components" in palette, "All components" in context menu, or **A** key.

**Structure:**
- Header "Add component" + close — fixed
- Search — fixed
- Icon grid (SLD: 3 cols, Map: 2 cols) — draggable
- Separator
- Full type list — draggable items
- *(upper area scrolls together)*
- Recent section — fixed ~200px, own scroll

Two independent scroll areas. All items in grid and list are draggable onto canvas. Search filters both lists.

### 4. Toolbar "Add" button / A key

Opens the full Add Component Panel.

---

## Placement Behavior

| Source | How it places |
|--------|--------------|
| Palette drag / Panel drag | Drop at cursor position |
| Context menu click | Instant at right-click position |
| Panel click (not drag) | Placement mode: crosshair, click to place |
| Shift+click in placement mode | Place and stay in mode (serial) |

### Generic vs Specified

**Generic** (from palette, context menu, panel grid/list): type category known, variant not selected. Orange "?" badge. Validation warning V-U01.

**Specified** (from Recent or library drill-down): exact variant. No badge.

Specifying later: click component → Info Panel → "Specify type" section → pick variant → badge removed.

---

## Types

**Electrical (SLD):** Connector, Transformer, Switch, Shunt, Voltage Regulator, Protection, Meter, Sectionalizer

**Physical (Map):** Pole, Vault, Substation, Duct
