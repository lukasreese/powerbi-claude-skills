---
description: Create IBCS-compliant variance charts and tables using native Power BI visuals with PBIR JSON
---

# IBCS Visuals Skill

Create IBCS (International Business Communication Standards) variance visualizations using native Power BI visuals.
All output is **PBIR file-based** — writes TMDL measures + visual.json files. No MCP server required.

## When to Use

Trigger on: "IBCS", "variance chart", "variance table", "IBCS column chart",
"IBCS bar chart", "IBCS table", "actual vs comparison chart", "AC vs PY chart".

## Workflow

### Step 1: Explore the Semantic Model

Read the user's PBIP/PBIR project to understand:
- What **measures** exist (read `.tmdl` files in `SemanticModel/definition/tables/`)
- What **tables and columns** are available (date tables, category dimensions)
- Whether **variance** and **variance %** measures already exist

### Step 2: Recommend Templates

Based on the user's analysis type, recommend the best template:

| Analysis Type | Recommended | Why |
|---|---|---|
| **Trend over time** (month, quarter, year) | Template 1: Column Variance | Columns show time progression naturally |
| **Ranked comparison** (country, product, region) | Template 2: Bar Variance | Horizontal bars for category ranking |
| **Detailed breakdown** (many rows, exact numbers needed) | Template 3: Simple Table | Table with SVG variance bars |
| **Full comparison view** (AC vs PY bars + variance bars) | Template 4: Full Table | All-SVG table showing everything |

**Default recommendation**: Charts first (1 & 2). Tables if the user wants detail or has many categories (>15 items).

### Step 3: Collect 3 Inputs

1. **Actual measure** — The current/actual period measure
   - Examples: `[Sales]`, `[Revenue]`, `[IST]`, `[Total Amount]`
   - Need: measure name AND the table it belongs to (Entity)

2. **Comparison measure** — The measure to compare against
   - Examples: `[Sales PY]`, `[Budget]`, `[IST VJ]`, `[Sales LY]`
   - Need: measure name AND the table it belongs to (Entity)

3. **Category column** — The axis/row dimension
   - Examples: `Date[MMM]`, `Calendar[Period]`, `Product[Category]`
   - Need: table name (Entity) AND column name (Property)
   - **Important**: For date columns, prefer columns with `sortByColumn` set (e.g., `MMM` sorted by `MonthNum`) to avoid alphabetical sorting

### Optional Inputs
- **Measure table**: Where to create helper measures (default: same table as actual measure)
- **Variance measure**: If user already has one (otherwise generate as `actual - comparison`)
- **Variance % measure**: If user already has one (otherwise generate as `DIVIDE(variance, comparison)`)
- **Visual title / subtitle**: Custom text for the chart header

### Step 4: Generate (PBIR File-Based)

All generation is file-based — write directly to the PBIP project files:

**For measures**: Append TMDL measure definitions to the appropriate `.tmdl` file
- Insert before the first `column` definition in the table's TMDL
- Use `displayFolder` to organize (e.g., `__IBCS Column Variance`)
- For SVG measures, set `dataCategory: ImageUrl`

**For visuals**: Write `visual.json` to the report's `pages/<pageId>/visuals/<visualName>/` folder
- Use `.mjs` helper scripts when paths contain spaces (EEXIST mkdir workaround)

### Step 5: User Reloads

User reloads the report in Power BI Desktop to see the changes.

## Available Templates

| # | Template | Visual Type | Measures Needed | Best For |
|---|----------|-------------|-----------------|----------|
| 1 | Column Variance Chart | `lineClusteredColumnComboChart` | 15 DAX measures (TMDL) | Time series comparison (columns) |
| 2 | Bar Variance Chart | `barChart` | 2-4 DAX + 13 NVC (embedded) | Ranked comparison (horizontal bars) |
| 3 | Variance Table (Simple) | `pivotTable` | 2-3 SVG measures (TMDL) | Table with numeric AC + SVG variance |
| 4 | Variance Table (Full) | `pivotTable` | 3-4 SVG measures (TMDL) | Full SVG table (AC,PY bars + variance) |

## Generation Steps per Template

### Template 1: Column Variance Chart

1. Read `references/ibcs-dax-measures.md` for all 15 measure templates
2. Replace placeholders (`{{ACTUAL}}`, `{{COMPARISON}}`, `{{DATE_TABLE}}`, etc.) with user's measure/table names
3. **Append all 15 measures to the user's TMDL file** (insert before first `column` definition)
4. Read `references/ibcs-column-variance.md` for the visual.json pattern
5. Generate complete `visual.json`:
   - Category → user's category column
   - Y projections → measures 02 (Actual) and 03 (Comparison)
   - Y2 projections → measures 07, 08, 13, 14
   - Error bars → referencing measures 11 (upperBound) and 12 (lowerBound)
   - valueAxis.end → measure 15
   - **Labels**: dynamicLabelValue on measures 13/14 showing values from measures 09/10
   - **Labels formatting**: measure 13 positioned `Above`, measure 14 positioned `Under`, both `fontSize: 10D`
   - **dataPoint**: fillTransparency `100D` on measures 13/14 (invisible positioning measures)
   - **seriesLabels**: hide for measures 07, 08, 13, 14
   - Colors from `references/ibcs-colors.md`
6. Write `visual.json` to target report folder

### Template 2: Bar Variance Chart

1. Read `references/ibcs-bar-variance.md` for the NativeVisualCalculation pattern
2. Check if these measures exist, **append to TMDL if missing**:
   - Variance measure (actual - comparison)
   - Variance % measure (DIVIDE(variance, comparison))
   - Positive abs variance (IF variance >= 0)
   - Negative abs variance (IF variance < 0)
3. Generate `visual.json` with all 13 NativeVisualCalculation expressions embedded:
   - NVC expressions reference the actual measure via `nativeQueryRef: "AC"`
   - NVC expressions reference the variance measure via `nativeQueryRef: "Variance"`
   - Dynamic labels on transparent bars reference external measures for actual, comparison, pos/neg variance, variance %
   - Conditional font color via `SelectRef` to `CF color var %` expression
4. Set dataPoint colors, reference lines, label configurations
5. **Label sizing**: Use `fontSize: 8D`, `labelPrecision: 0L`, `labelOverflow: true` for resize resilience
6. Write `visual.json` to target report folder

### Template 3: Variance Table (Simple)

1. Read `references/ibcs-svg-measures.md` for SVG measure templates
2. **Append to TMDL**:
   - `SVG Style` (text helper, no dataCategory)
   - `Delta SVG` (dataCategory: ImageUrl)
   - `Delta% SVG` (dataCategory: ImageUrl)
3. Read `references/ibcs-table-simple.md` for the visual.json pattern
4. Generate `visual.json`:
   - Rows: user's category column
   - Values: user's actual measure + Delta SVG + Delta% SVG
   - Grid image dimensions: 35x250, Style preset: Minimal
5. Write `visual.json` to target report folder

### Template 4: Variance Table (Full)

1. Same as Template 3, also create `AC,PY SVG` measure
2. Read `references/ibcs-table-full.md` for the visual.json pattern
3. Generate `visual.json` with AC,PY SVG as first value column instead of numeric actual
4. Write `visual.json` to target report folder

## IBCS Color Palette

See `references/ibcs-colors.md` for the full palette. Key colors:
- Actual: `#0C3549` (dark blue)
- Comparison: `#CCCCCC` (gray)
- Positive: `#44C088` (green)
- Negative: `#ED7373` (red)

## Visual JSON Schema

All visuals use schema version 2.6.0:
```
https://developer.microsoft.com/json-schemas/fabric/item/report/definition/visualContainer/2.6.0/schema.json
```

## Important Notes

- **PBIR-first**: All measures are written to TMDL files, all visuals as visual.json files. No MCP server dependency.
- **NativeVisualCalculation** (Template 2) embeds DAX directly in visual.json — fewer semantic model measures needed
- **SVG measures** (Templates 3 & 4) require `dataCategory: ImageUrl` to render as images in pivot tables
- **Locale handling**: SVG measures use `SUBSTITUTE(FORMAT(...), ",", ".")` to ensure dot decimals in SVG attributes
- **Error bars** (Template 1) are repurposed as IBCS variance bars — they're not showing data uncertainty
- **Reference lines** with `transparency: 100D` are invisible lines used purely for data label positioning
- **Variance cap**: Both templates cap variance bars (50% for Template 1, 40% for Template 2) to prevent visual overflow
- **sortByColumn**: Always verify date columns have `sortByColumn` set in TMDL to avoid alphabetical sorting
- **Write tool workaround**: Paths containing spaces need .mjs helper scripts (EEXIST mkdir error)
