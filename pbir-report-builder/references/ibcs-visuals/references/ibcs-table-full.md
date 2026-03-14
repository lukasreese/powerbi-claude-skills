# Template 4: IBCS Variance Table (Full)

**Visual type**: `pivotTable`
**Required measures**: 3 SVG measures + SVG Style helper (see `ibcs-svg-measures.md`)
**Complexity**: Medium

## Overview

A native pivot table with 3 SVG value columns (no numeric column):
1. **AC,PY SVG** — overlapping bars showing actual (front) vs comparison (back)
2. **Delta SVG** — centered bar showing absolute variance
3. **Delta% SVG** — pin chart showing percentage variance

## Required Measures

Create these measures in Power BI Desktop:
1. `SVG Style` — shared SVG CSS helper
2. `AC,PY SVG` — from `ibcs-svg-measures.md` template
3. `Delta SVG` — from `ibcs-svg-measures.md` template
4. `Delta% SVG` — from `ibcs-svg-measures.md` template

All SVG measures need `dataCategory: ImageUrl` in their definition.

## Query Structure

Same as Template 3 but the first value column is `AC,PY SVG` instead of a numeric measure:

```json
{
  "Rows": { "projections": [{ "field": { "Column": { "Entity": "{{DATE_TABLE}}", "Property": "{{DATE_COLUMN}}" } } }] },
  "Values": { "projections": [
    { "field": { "Measure": { "Entity": "{{MEASURE_TABLE}}", "Property": "AC,PY SVG" } }, "nativeQueryRef": "AC,PY" },
    { "field": { "Measure": { "Entity": "{{MEASURE_TABLE}}", "Property": "Delta SVG" } }, "displayName": "Variance" },
    { "field": { "Measure": { "Entity": "{{MEASURE_TABLE}}", "Property": "Delta% SVG" } }, "displayName": "Variance %" }
  ] }
}
```

## Key Object Settings

Same as Template 3 with adjusted column widths:
- AC,PY SVG: ~272px (wider to show overlapping bars)
- Delta SVG: ~230px
- Delta% SVG: ~345px

All other settings identical to Template 3 (grid, urlIcon, stylePreset, etc.).