# Add Component Flow — Design Specification

**Date:** April 2026
**Status:** Design complete, prototyped in v5
**Figma:** Grid Model Work in Progress → Add component panel, Context menu

---

## 1. Concept

The application has two global modes that affect how the user interacts with the diagram and map.

| Mode | Purpose | Available actions |
|------|---------|-------------------|
| **View** | Browse model, inspect components | Select, pan, view Info Panel (read-only) |
| **Edit** | Modify model content | All View actions + add, connect, bind, delete, drag, aggregate |

Entry: toolbar **Edit** button or **E** key. Exit: **Done** button or **Esc**.

When in Edit mode, component creation is available through **four entry points** — each designed for a different workflow speed.

---

## 2. Entry Points

### 2.1 Left Palette (drag-and-drop)

Always-visible sidebar anchored to the left edge of each panel. Two variants:

**SLD palette (electrical)** — 8 component types:
Connector, Transformer, Switch, Shunt, Voltage Regulator, Protection, Meter, Sectionalizer

**Map palette (physical)** — 4 structure types:
Pole, Vault, Substation, Duct

Each palette has two states:

| State | Layout | Behavior |
|-------|--------|----------|
| **Collapsed** | Single column, 40px wide, icon buttons 32×32, no labels | Tooltip on hover. Bottom: "..." (opens All components panel) + expand toggle, stacked vertically |
| **Expanded** | SLD: 2×4 grid, buttons 80×60 with labels. Map: 2×2 grid | Bottom: "All components" text button + collapse toggle, side by side |

**Interaction:** Grab any icon → drag onto canvas → component placed at drop point as generic (unspecified variant). Ghost element follows cursor during drag.

### 2.2 Context Menu (right-click)

Right-click on empty space shows contextual add options.

**SLD empty space:**
```
┌──────────────────────────┐  ┌────────────────────┐
│ ⊞ Add component        › │→ │ ⊙ Connector        │
│ ⊞ Add aggregate          │  │ ⊙ Transformer      │
└──────────────────────────┘  │ ⊙ Switch            │
                              │ ⊙ Shunt             │
                              │ ⊙ Voltage Regulator │
                              │ ⊙ Meter             │
                              │───────────────────  │
                              │ ⋮ All components    │
                              └────────────────────┘
```

**Map empty space:**
```
┌──────────────────────────┐  ┌──────────────┐
│ ⚡ Add structure        › │→ │ ⛨ Pole       │
└──────────────────────────┘  │ ⊞ Vault      │
                              │ ⌂ Substation │
                              │ ⊙ Duct       │
                              └──────────────┘
```

**Interaction:** Click type in submenu → component placed instantly at cursor position. "All components" → opens full Add component panel.

### 2.3 Add Component Panel (full library)

Full-height side panel (280px), opened by: "All components" from palette, "All components" from context menu, or **A** key.

**SLD variant** — anchored left of SLD canvas, overlays diagram:

```
┌─────────────────────────────┐
│ Add component            ✕  │ ← header, fixed
├─────────────────────────────┤
│ 🔍 Search equipment        │ ← search, fixed
├─────────────────────────────┤
│ ┌─────┐ ┌─────┐ ┌─────┐   │
│ │ Con │ │ Pro │ │ Shu │   │ ← icon grid 3 cols
│ └─────┘ └─────┘ └─────┘   │   draggable
│ ┌─────┐ ┌─────┐ ┌─────┐   │
│ │ Tra │ │ Met │ │ Vol │   │
│ └─────┘ └─────┘ └─────┘   │
│ ┌─────┐ ┌─────┐            │
│ │ Swi │ │ Sec │            │
│ └─────┘ └─────┘            │
│─────────────────────────────│ ← separator
│ ⊙ Connector               │  ↑
│ ⊙ Transformer             │  │ upper scroll area
│ ⊙ Switch        ← hover   │  │ (icon grid + type list)
│ ⊙ Shunt                   │  │ draggable items
│ ⊙ Voltage Regulator       │  │
│ ⊙ Protection              │  │
│ ⊙ Meter                   │  ↓
├─────────────────────────────┤
│ 🕐 Recent                  │  ↑
│ ┊ Pad-mount  250kVA·12kV  │  │ lower scroll area
│ ┊ Pad-mount  167kVA·12kV  │  │ fixed ~200px height
│ ┊ Pad-mount  20MVA·69kV   │  ↓
└─────────────────────────────┘
```

**Map variant** — anchored left of map canvas, narrower (2 cols in grid):

```
┌────────────────────┐
│ Add compone... ✕   │
├────────────────────┤
│ 🔍 Search equip   │
├────────────────────┤
│ ┌─────┐ ┌─────┐   │
│ │ Con │ │ Pro │   │ ← 2 cols
│ └─────┘ └─────┘   │
│ ┌─────┐ ┌─────┐   │
│ │ Shu │ │ Tra │   │
│ └─────┘ └─────┘   │
├────────────────────┤
│ 🕐 Recent          │
│ ┊ Pad-mount 250kVA│
│ ┊ Pad-mount 167kVA│
│ ┊ Pad-mount 20MVA │
└────────────────────┘
```

**Two independent scroll areas:**
- Upper: icon grid + full type list (scrolls together)
- Lower: Recent section, fixed height ~200px, own scroll

**Interaction:** All items (icons in grid AND items in list) are draggable onto canvas. Search filters both type list and Recent section.

### 2.4 Toolbar Button

**A** key or **Add (+)** button in Edit mode toolbar → opens full Add component panel.

---

## 3. Component Placement Behavior

### 3.1 Generic placement (sketch-first)

All entry points except Recent items place a **generic component** — type category known (e.g. "Transformer") but specific variant not yet selected.

| Property | Value |
|----------|-------|
| Label | "{Type} {counter}" e.g. "Transformer 1" |
| Variant | `null` (unspecified) |
| Badge | Orange "?" on diagram |
| Validation | V-U01 warning: "N components without specified variant" |

### 3.2 Specified placement (from Recent / library drill-down)

Recent items and library drill-down (category → variant) place a **specified component** with exact model/specs.

| Property | Value |
|----------|-------|
| Label | Variant-based e.g. "PM-201" |
| Variant | Specific variant ID |
| Badge | None |
| Validation | No V-U01 warning |

### 3.3 Placement modes

| Source | Placement method |
|--------|------------------|
| Palette drag / Panel drag | Drop at cursor position |
| Context menu type click | Instant placement at right-click position |
| Panel type click | Enter placement mode (crosshair cursor), click to place |
| Shift+click in placement mode | Place and stay in mode (serial placement) |

### 3.4 Specifying variant later

Click generic component → Info Panel → orange "Specify type" section at top → select variant from list → badge removed, label updated.

---

## 4. Physical Structures (Map)

Same pattern, simplified:

| Entry point | Types |
|-------------|-------|
| Map palette (left) | Pole, Vault, Substation, Duct |
| Map context menu → Add structure | Pole, Vault, Substation, Duct |

Structures are placed on map with GPS coordinates. No "generic" concept — structures are always typed at creation (specific variant selected later in Info Panel).

---

## 5. View Mode Restrictions

In View mode the following are **hidden or disabled:**

| Element | State |
|---------|-------|
| SLD component palette | Hidden |
| Map structure palette | Hidden |
| Add component panel | Cannot open |
| Context menu → Add component | Not shown (replaced by "Edit mode (E)") |
| Context menu → Add aggregate | Not shown |
| Context menu → Add structure | Not shown |
| Toolbar Add button | Not shown |
| Drag components | Disabled |
| Connect ports | Disabled |

Available in View mode: Select, Pan, Info Panel (read-only), right-click → Show info / Locate.

---

## 6. Component Type Reference

### Electrical components (SLD)

| Type | Description | Terminals |
|------|-------------|-----------|
| Connector | Bus section | 5 positions (bottom) |
| Transformer | Power/distribution transformer | Primary (top), Secondary (bottom) |
| Switch | Load break, sectionalizer, disconnect | A (top), B (bottom) |
| Shunt | Capacitor bank, reactor | A (top), B (bottom) |
| Voltage Regulator | Step regulator | A (top), B (bottom) |
| Protection | Recloser, fuse, relay | A (top), B (bottom) |
| Meter | AMI meter, revenue meter | Terminal (top) |
| Sectionalizer | Automatic sectionalizer | A (top), B (bottom) |

### Physical structures (Map)

| Type | Description | Preposition |
|------|-------------|-------------|
| Pole | Wood/Steel/Concrete pole | ON |
| Vault | Underground splice vault, Handhole | IN |
| Substation | Substation site, Control building | AT |
| Duct | Duct bank, Conduit run | IN |

---

## 7. Keyboard Shortcuts

| Key | Action | Mode |
|-----|--------|------|
| E | Enter Edit mode | View |
| A | Open Add component panel | Edit |
| Shift+click | Serial placement (stay in mode) | Edit, placement mode |
| Esc | Cancel placement / exit mode | Edit |
| V | Select tool | Both |
| H | Pan tool | Both |

---

## 8. Figma References

| Screen | Description |
|--------|-------------|
| Add component panel | All panel states: collapsed palette, expanded palette, full panel |
| Context menu | SLD and Map context menus with submenus |
| Examples in interface | Panels in context with map and diagram |
| Left mini panel, Context menu panel | Full interface with both palettes and context menus |
