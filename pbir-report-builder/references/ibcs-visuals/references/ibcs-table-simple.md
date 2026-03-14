# Template 3: IBCS Variance Table (Simple)

**Visual type**: `pivotTable`
**Required measures**: 2 SVG measures + SVG Style helper (see `ibcs-svg-measures.md`)
**Complexity**: Medium

## Overview

A native pivot table with 3 value columns:
1. **Actual value** — numeric column showing the user's actual measure
2. **Delta SVG** — inline SVG bar showing absolute variance
3. **Delta% SVG** — inline SVG pin showing percentage variance

## Required Measures

Create these measures in Power BI Desktop:
1. `SVG Style` — shared SVG CSS helper
2. `Delta SVG` — from `ibcs-svg-measures.md` template
3. `Delta% SVG` — from `ibcs-svg-measures.md` template

All SVG measures need `dataCategory: ImageUrl` in their definition.

## Query Structure

```json
{
  "Rows": { "projections": [{ "field": { "Column": { "Entity": "{{DATE_TABLE}}", "Property": "{{DATE_COLUMN}}" } } }] },
  "Values": { "projections": [
    { "field": { "Measure": { "Entity": "{{ACTUAL_TABLE}}", "Property": "{{ACTUAL_NAME}}" } }, "nativeQueryRef": "AC" },
    { "field": { "Measure": { "Entity": "{{MEASURE_TABLE}}", "Property": "Delta SVG" } }, "displayName": "Variance" },
    { "field": { "Measure": { "Entity": "{{MEASURE_TABLE}}", "Property": "Delta% SVG" } }, "displayName": "Variance %" }
  ] }
}
```

## Key Object Settings

### grid
- `gridHorizontal: false` — no horizontal grid lines
- `imageHeight: 35D` — height for SVG images
- `imageWidth: 250D` — width for SVG images

### columnWidth
Set initial widths for each value column:
- Actual: ~107px
- Delta SVG: ~230px
- Delta% SVG: ~345px

### values, columnHeaders, rowHeaders
- `urlIcon: false` — hide the URL link icon (SVGs render as images, not links)

### expansionStates
```json
[{ "roles": ["Rows"], "levels": [{ "queryRefs": ["{{DATE_TABLE}}.{{DATE_COLUMN}}"], "isCollapsed": true, "isPinned": true }], "root": {} }]
```

### stylePreset
- `name: "Minimal"` — clean, minimal table style

### sortDefinition
- Sort by category column ascending