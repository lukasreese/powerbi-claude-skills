# PBIR Navigation & Button Templates
# Use these JSON patterns as templates when generating navigation visuals

## Visual Types Overview

| Type | Visual Type | Use Case |
|------|------------|----------|
| Sidebar Nav Button | actionButton | Left sidebar menu items (icon + text) |
| Icon-Above Button | actionButton | Compact grid nav (icon on top, text below) |
| Active Nav Button | actionButton | Highlighted/current page button |
| CTA Button | actionButton | Action buttons (Filter, Add, etc.) |
| Page Navigator | pageNavigator | Horizontal tab strip for page switching |
| Button Slicer | advancedSlicerVisual | Filter chips/pills bound to a data column |

---

## 1. Sidebar Navigation Button (Icon + Text)
Usage: Vertical sidebar menu with icon on left, text centered. 155x35px.

Key JSON structure for actionButton with page navigation:
- visualType: "actionButton"
- icon.shapeType: "custom" (for custom PNG icon)
- icon.image: ResourcePackageItem reference
- icon.iconSize: 19D
- icon.horizontalAlignment: "left"
- icon.leftMargin: 10L
- fill default: ThemeDataColor ColorId=0 (white)
- fill hover: #F1F5F9 (light gray)
- outline.show: false
- shape.roundEdge: 10L
- text.horizontalAlignment: "center"
- text.fontColor: ThemeDataColor ColorId=1 (dark)
- text margins: left=20L, right=8L, bottom=6L
- visualContainerObjects.border: show=false, radius=10D
- visualContainerObjects.visualLink:
  - show: true
  - type: "PageNavigation"
  - navigationSection: "TARGET_PAGE_ID"
  - showDefaultTooltip: false

Position pattern for vertical sidebar stack:
- x: 17, width: 155, height: 35
- y values: 155, 190, 226, 261 (35px spacing)
- z order: increment by 1000

---

## 2. Active/Highlighted Navigation Button
Same as sidebar nav but replace default fill color with:
- fillColor: "#4285F4" (Google blue) instead of ThemeDataColor
- This indicates the current/active page

Active state colors:
- Blue active: #4285F4
- Dark blue active: #2563EB
- Hover stays: #F1F5F9

---

## 3. Icon-Above Button (Compact Square)
Usage: Grid of square buttons with icon above text. 70x45px.

Differences from sidebar nav:
- Size: 70x45 (taller, narrower)
- Icon placement: "above" (icon on top, text below)
- Text margins: 4L all around (tighter)
- Add to icon default selector: placement = "above"

Grid layout: x positions at 189, 259, 329 (70px spacing)

---

## 4. CTA/Action Button (Colored with Icon)
Usage: Primary action buttons like "Filter", "Add New". ~107x40px.

Key differences:
- fill.fillColor: "#2563EB" (Tailwind blue-600)
- text.fontColor: ThemeDataColor ColorId=0 (white text)
- text.horizontalAlignment: "right"
- Icon: left-aligned, white icon image
- No hover state (stays solid)
- Typical position: top-right corner of page

---

## 5. Page Navigator (Horizontal Tab Strip)
Usage: Built-in visual that auto-generates tabs from report pages. ~540x47px.

Key JSON structure:
- visualType: "pageNavigator"
- shape.tileShape: "rectangleRounded"
- shape.rectangleRoundedCurve: 10L
- fill selected: #F1F5F9
- fill default: ThemeDataColor 0 (white)
- fill hover: #EFF4F8
- outline.show: false
- pages: array of page selectors with showPage true/false per page ID
- pages.showHiddenPages: true, showByDefault: false
- layout.orientation: 2D (horizontal)
- accentBar.show: true
- accentBar.position: "Top" or "Bottom"
- accentBar.color: "#184EA6" (navy)
- accentBar selector: "selected" (only shows on active tab)
- title.show: false

Page selector format:
  { "properties": { "showPage": { "expr": { "Literal": { "Value": "true" } } } }, "selector": { "id": "PAGE_ID" } }

---

## 6. Button Slicer (advancedSlicerVisual)
Usage: Filter chips/pills bound to a data column. Auto-generates buttons from data values.

Key JSON structure:
- visualType: "advancedSlicerVisual"
- query.queryState.Values.projections:
  - field.Column.Expression.SourceRef.Entity: "TABLE_NAME"
  - field.Column.Property: "COLUMN_NAME"
  - queryRef: "TABLE_NAME.COLUMN_NAME"
  - nativeQueryRef: "COLUMN_NAME"
- layout.columnCount: NL (e.g., 4L or 5L)
- layout.rowCount: 1L
- outline.show: false (both default and selection:selected)
- value selected: fontColor ThemeDataColor 1, bold=true, fontSize=11D
- value default: fontSize=11D
- background selected: show=false
- fillCustom default: show=false
- fillCustom selection:selected: show=true, fillColor="#4285F4"
- fillCustom interaction:rest: show=true, fillColor="#F4F4F4"
- shapeCustomRectangle: tileShape="rectangleRoundedByPixel", curve=10L
  (both selection:selected and interaction:rest)
- label.show: false
- visualContainerObjects: title=false, background=false, dropShadow=false, border=false

### Adding a default filter:
Add general.filter object with filter expression:
- filter.Version: 2
- filter.From: [{ Name: "d", Entity: "TABLE_NAME", Type: 0 }]
- filter.Where: In condition with Expressions and Values

### Underline variant (accent bar instead of fill):
- accentBar: show=true, position="Bottom", width=2D, selector="selection:selected"
- padding: paddingSelection="Narrow"
- shapeCustomRectangle: tileShape="rectangle" (not rounded)
- fillCustom selected: show=false (no fill, just underline)

---

## Common Color Tokens

| Token | Hex | Usage |
|-------|-----|-------|
| ThemeDataColor 0 | white | Default button fill, white text on dark buttons |
| ThemeDataColor 1 | dark text | Button labels, selected tab text |
| #F1F5F9 | light gray | Hover state, selected page nav tab |
| #EFF4F8 | slightly lighter | Page navigator hover |
| #F4F4F4 | neutral gray | Button slicer rest state |
| #4285F4 | Google blue | Active nav button, selected slicer chip |
| #2563EB | Tailwind blue-600 | CTA button fill |
| #184EA6 | Navy | Page navigator accent bar |

## Icon Resources
Icons are stored in the report StaticResources/RegisteredResources/ folder.
Common icons used:
- home-2x.png — Home/dashboard
- boxes.png — Products/inventory
- people.png — Clients/users
- credit-card.png — Billing/payments
- funnel.png — Filter
- plus-2x white.png — Add/create action (white for dark buttons)

Note: When creating from scratch without registered icon resources,
use built-in shapeType values (e.g., "arrow", "blank") or add image files to
the .Report/StaticResources/RegisteredResources/ folder.