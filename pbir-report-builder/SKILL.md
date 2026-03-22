---
name: pbi-report-builder
description: "[power-bi] Power BI PBIR Report Builder with IBCS Visuals. Generates Power BI report pages, visuals, and IBCS-compliant variance charts by writing PBIR JSON files directly into PBIP project folders. Use this skill EVERY TIME the user asks to: create a Power BI report page, add visuals to a report, generate KPI cards, create charts or tables in Power BI, build a dashboard layout with visuals, create IBCS variance charts, create actual vs plan visuals, or programmatically create Power BI visuals. Also trigger when the user mentions 'PBIR', 'IBCS', 'variance chart', 'variance table', 'actual vs plan', 'actual vs comparison', 'create visuals', 'add a page', 'build a report', 'KPI cards', 'place visuals', or wants to generate Power BI report content through code. If the user mentions any combination of Power BI + visuals/page/report/KPI/chart/table/IBCS/variance + create/build/generate/add, use this skill."
---

# PBIR Report Builder

Add pages and visuals to existing Power BI PBIP projects by writing PBIR (Enhanced Report Format) JSON files directly into the project folder structure.

## Critical: Hybrid Approach (Do NOT Create PBIP From Scratch)

**NEVER create an entire PBIP project from scratch.** Power BI Desktop generates boilerplate files (`report.json`, `.platform`, `version.json`, theme packages, settings) with version-specific schema references, real GUIDs, and internal settings that are impractical to replicate manually.

**The correct workflow:**
1. **User creates a blank PBIP** — open Power BI Desktop, connect to the data source, save as PBIP format. This generates all boilerplate correctly.
2. **Semantic model measures** — use `powerbi-modeling-mcp` if Desktop is running, OR write directly to TMDL files if Desktop is closed (see "Adding Measures When Desktop Is Closed" below).
3. **This skill adds pages and visuals** — write PBIR JSON files into the existing `.Report/definition/` folder to create report pages with visuals.

**Why not from scratch?** The `report.json` file contains Desktop-version-specific theme references (e.g., `CY26SU02` with nested version objects), resource packages, 6+ report settings, and the `.platform` files need real UUIDs. These change between Desktop versions and cannot be reliably generated.

### PBIP File Structure (for reference — user creates this by saving)
```
ProjectName.pbip                           ← entry point (Desktop creates)
ProjectName.Report/
  .platform                                ← Desktop creates (Report type, UUID)
  definition.pbir                          ← Desktop creates (links to semantic model)
  definition/
    report.json                            ← Desktop creates (theme, settings)
    version.json                           ← Desktop creates
    pages/
      pages.json                           ← WE MODIFY (add page to pageOrder)
      pg01Overview/                        ← WE CREATE
        page.json                          ← WE CREATE
        visuals/
          v01KpiSales/visual.json          ← WE CREATE
ProjectName.SemanticModel/
  .platform                                ← Desktop creates (SemanticModel type, UUID)
  definition.pbism                         ← Desktop creates (NOT definition.tmdl!)
  definition/
    database.tmdl                          ← Desktop creates (compatibilityLevel: 1600)
    model.tmdl                             ← Desktop creates or MCP server manages
    tables/*.tmdl                          ← MCP server manages
```

**Key learning:** The semantic model entry point is `definition.pbism` (a JSON file), NOT `definition.tmdl`. TMDL files go inside the `definition/` subfolder.

## References (Self-Contained)

All reference material is bundled inside this skill at `references/`:

**Reference docs:**
- `references/visual-types.md` — all 35+ visual type identifiers and query roles
- `references/field-references.md` — column/measure binding patterns, aggregation codes, conditional formatting
- `references/formatting-objects.md` — container/visual/page formatting JSON patterns
- `references/folder-structure.md` — annotated PBIP/PBIR/TMDL folder tree
- `references/page-naming.md` — readable naming convention (pg##/v## rules)

**JSON templates (ready-to-use):**
- `references/json-templates/card-kpi.json` — KPI card with CY, PY, YoY%, conditional color
- `references/json-templates/clustered-column.json` — vertical bar chart
- `references/json-templates/clustered-bar.json` — horizontal bar chart
- `references/json-templates/line-chart.json` — line trend over time
- `references/json-templates/combo-chart.json` — column + line combo
- `references/json-templates/table.json` — table with multiple columns/measures
- `references/json-templates/matrix.json` — pivot table with rows/columns/values
- `references/json-templates/slicer.json` — slicer dropdown
- `references/json-templates/donut.json` — donut/pie chart
- `references/json-templates/page-standard.json` — standard page definition
- `references/json-templates/page-drillthrough.json` — drillthrough page
- `references/json-templates/report-settings.json` — report-level settings

**Complete examples (start here for new users):**
- `references/examples/kpi-dashboard-example.md` — full working example: 4 KPI cards + bar chart + line chart, complete Node.js script, DAX measures, layout grid
- `references/examples/card-kpi-with-yoy.md` — **KPI card with conditional YoY badge** — `cardVisual` with prior-year reference row, custom label ("vs. PY:"), and green/red % badge driven by DAX color measures. Includes full `referenceLabelDetail` conditional formatting pattern, required DAX measures, complete JSON template with placeholders, and community contribution guide.

**JSON schemas (Microsoft originals):**
- `references/json-schemas/` — local copies of all PBIR schemas for offline validation

**IBCS Visuals (integrated):**
- `references/ibcs-visuals/IBCS-REFERENCE.md` — full IBCS workflow, template selection guide, generation steps
- `references/ibcs-visuals/references/ibcs-colors.md` — IBCS color palette (actual, comparison, positive, negative)
- `references/ibcs-visuals/references/ibcs-dax-measures.md` — 15 DAX measure templates for column variance
- `references/ibcs-visuals/references/ibcs-svg-measures.md` — SVG measure templates for table visuals
- `references/ibcs-visuals/references/ibcs-column-variance.md` — combo chart visual.json pattern
- `references/ibcs-visuals/references/ibcs-bar-variance.md` — bar chart with NativeVisualCalculation pattern
- `references/ibcs-visuals/references/ibcs-table-simple.md` — pivot table with SVG variance bars
- `references/ibcs-visuals/references/ibcs-table-full.md` — full SVG table (AC, PY bars + variance)

For IBCS visuals, read `references/ibcs-visuals/IBCS-REFERENCE.md` for the full workflow and template selection guide.

Read the relevant template file when building a visual type you haven't used recently.

## How It Works

### Step 0: Prerequisites

Before using this skill, verify:
1. **A PBIP project already exists** — user must have saved a `.pbip` from Power BI Desktop
2. **The semantic model is connected** — the `.SemanticModel/` folder has data tables with columns
3. **Measures exist or can be created** — either already in the model, creatable via MCP, or writable to TMDL
4. **Power BI Desktop is CLOSED** — files cannot be written while Desktop has the project open

If the user doesn't have a PBIP yet, instruct them to:
1. Open Power BI Desktop
2. Connect to their data source
3. File → Save As → select "Power BI Project (.pbip)" format
4. This creates the full boilerplate structure that this skill needs

### Step 1: Gather Requirements

Ask the user:
1. **Target PBIP project path** — where to write the files (must already exist as a saved PBIP)
2. **Page name and purpose** — e.g., "Sales Overview", "Product Drillthrough"
3. **Canvas size** — read from existing `page.json` files or use default 1280x720
4. **Visuals needed** — what charts/cards/tables, and what data they show
5. **Measures and columns** — table.field references for each visual (case-sensitive, must match model exactly)
6. **Layout** — natural language ("4 KPIs at top, bar chart left, line chart right") or pixel positions

If the user has a background SVG from the Background Designer skill, use those exact grid positions.

### Step 2: Generate the PBIR Files

Write files directly into the project's `.Report/definition/` folder.

#### File Generation Order:
1. Create page folder: `definition/pages/pg##Name/`
2. Create `page.json` in that folder
3. Create `visuals/` subfolder
4. For each visual, create `visuals/v##Name/visual.json`
5. Update `definition/pages/pages.json` — add the new page to `pageOrder`

### Step 3: Naming Convention

**ALWAYS use readable names:**
- Pages: `pg01Overview`, `pg02SalesDetail`, `pg03Drillthrough`
- Visuals: `v01KpiTotalSales`, `v02BarByRegion`, `v03LineTrend`
- The `name` field inside the JSON MUST match the folder name exactly

See `page-naming.md` for full convention.

### Step 4: Deliver

Tell the user:
1. Files have been written to `[path]`
2. Close Power BI Desktop if open, then reopen the `.pbip` file
3. The new page should appear with all visuals

---

## JSON Structure Reference

### page.json Template
```json
{
  "$schema": "https://developer.microsoft.com/json-schemas/fabric/item/report/definition/page/2.0.0/schema.json",
  "name": "pg01Overview",
  "displayName": "Overview",
  "displayOption": "FitToPage",
  "width": 1280,
  "height": 720
}
```

**Schema version detection:** Before writing any files, read an existing `visual.json` from the project to discover the schema version Desktop used (e.g., `2.6.0`, `2.7.0`). Use that same version for all new files. The template above shows `2.0.0` as a baseline — always override with the detected version.

Canvas sizes:
- Standard: `1280` x `720`
- Report: `1300` x `900`
- Wide: `1600` x `900`
- Full HD: `1920` x `1080`

Page types:
- Standard: omit `type` field
- Drillthrough: add `"type": "Drillthrough"`
- Tooltip: add `"type": "Tooltip"`, use smaller dimensions (320x240)
- Hidden: add `"visibility": "HiddenInViewMode"`

### pages.json — Adding a New Page
Read existing `pages.json`, add the new page name to `pageOrder`:
```json
{
  "$schema": "https://developer.microsoft.com/json-schemas/fabric/item/report/definition/pagesMetadata/1.0.0/schema.json",
  "pageOrder": ["pg01Overview", "pg02NewPage"],
  "activePageName": "pg01Overview"
}
```

### visual.json — Visual Container Structure
```json
{
  "$schema": "https://developer.microsoft.com/json-schemas/fabric/item/report/definition/visualContainer/2.0.0/schema.json",
  "name": "v01KpiTotalSales",
  "position": {
    "x": 30,
    "y": 80,
    "z": 1000,
    "width": 280,
    "height": 150,
    "tabOrder": 0
  },
  "visual": {
    "visualType": "cardVisual",
    "query": {
      "queryState": {
        "ROLE_NAME": {
          "projections": [
            {
              "field": { "..." : "..." },
              "queryRef": "Table.Field",
              "nativeQueryRef": "Field"
            }
          ]
        }
      }
    },
    "objects": {},
    "drillFilterOtherVisuals": true
  }
}
```

---

## Visual Type Patterns

### KPI Card (cardVisual) — The Core Pattern

This is the most common visual. Shows a main value, a comparison reference, and a change percentage with conditional coloring.

**Query roles:**
- `Data` — main KPI value (e.g., Total Sales CY)
- `ReferenceLabels` — comparison value (e.g., Total Sales PY)
- `AdditionalMeasure` — change metric (e.g., YoY Sales %)

**Conditional color via measure:** The `KPI Color` measure returns a hex color based on YoY performance. Use the IBCS color palette consistently: `"#44C088"` (green/positive) or `"#ED7373"` (red/negative). Do NOT use `"#00B050"` / `"#FF0000"` — those clash with the IBCS visual standards used elsewhere in the report.

```json
{
  "name": "v01KpiTotalSales",
  "position": { "x": 30, "y": 80, "z": 1000, "width": 280, "height": 150, "tabOrder": 0 },
  "visual": {
    "visualType": "cardVisual",
    "objects": {
      "calloutValue": [{
        "properties": {
          "color": {
            "solid": {
              "color": {
                "expr": {
                  "Measure": {
                    "Expression": { "SourceRef": { "Entity": "MEASURES_TABLE" } },
                    "Property": "KPI Color"
                  }
                }
              }
            }
          }
        }
      }],
      "cards": [{ "properties": { "showLabel": { "expr": { "Literal": { "Value": "true" } } } } }],
      "referenceLabel": [{ "properties": { "show": { "expr": { "Literal": { "Value": "true" } } } } }]
    },
    "query": {
      "queryState": {
        "Data": {
          "projections": [{
            "field": { "Measure": { "Expression": { "SourceRef": { "Entity": "MEASURES_TABLE" } }, "Property": "MEASURE_CY" } },
            "queryRef": "MEASURES_TABLE.MEASURE_CY",
            "nativeQueryRef": "MEASURE_CY"
          }]
        },
        "ReferenceLabels": {
          "projections": [{
            "field": { "Measure": { "Expression": { "SourceRef": { "Entity": "MEASURES_TABLE" } }, "Property": "MEASURE_PY" } },
            "queryRef": "MEASURES_TABLE.MEASURE_PY",
            "nativeQueryRef": "MEASURE_PY"
          }]
        },
        "AdditionalMeasure": {
          "projections": [{
            "field": { "Measure": { "Expression": { "SourceRef": { "Entity": "MEASURES_TABLE" } }, "Property": "MEASURE_YOY" } },
            "queryRef": "MEASURES_TABLE.MEASURE_YOY",
            "nativeQueryRef": "MEASURE_YOY"
          }]
        }
      }
    },
    "drillFilterOtherVisuals": true
  }
}
```

**To create multiple KPI cards**, repeat this pattern with different measures and increment:
- `name`: v01Kpi..., v02Kpi..., v03Kpi..., v04Kpi...
- `position.x`: space them horizontally (e.g., 30, 320, 610, 900)
- `position.z`: 1000, 1001, 1002, 1003
- `position.tabOrder`: 0, 1, 2, 3

### Bar/Column Chart

**Query roles:** `Category` (axis), `Y` (values), `Series` (legend, optional), `Tooltips` (optional)

```json
"visual": {
  "visualType": "clusteredColumnChart",
  "query": {
    "queryState": {
      "Category": { "projections": [{
        "field": { "Column": { "Expression": { "SourceRef": { "Entity": "TABLE" } }, "Property": "COLUMN" } },
        "queryRef": "TABLE.COLUMN", "nativeQueryRef": "COLUMN"
      }]},
      "Y": { "projections": [{
        "field": { "Measure": { "Expression": { "SourceRef": { "Entity": "MEASURES_TABLE" } }, "Property": "MEASURE" } },
        "queryRef": "MEASURES_TABLE.MEASURE", "nativeQueryRef": "MEASURE"
      }]}
    }
  },
  "drillFilterOtherVisuals": true
}
```

Visual types: `clusteredColumnChart`, `clusteredBarChart`, `columnChart` (stacked), `barChart` (stacked)

### Line Chart

**Query roles:** `Category` (axis), `Y` (values), `Y2` (secondary axis, optional), `Series` (legend, optional)

```json
"visual": {
  "visualType": "lineChart",
  "query": {
    "queryState": {
      "Category": { "projections": [{
        "field": { "Column": { "Expression": { "SourceRef": { "Entity": "DimDate" } }, "Property": "Month" } },
        "queryRef": "DimDate.Month", "nativeQueryRef": "Month"
      }]},
      "Y": { "projections": [{
        "field": { "Measure": { "Expression": { "SourceRef": { "Entity": "MEASURES_TABLE" } }, "Property": "MEASURE" } },
        "queryRef": "MEASURES_TABLE.MEASURE", "nativeQueryRef": "MEASURE"
      }]}
    }
  }
}
```

### Combo Chart

**Query roles:** `Category`, `ColumnY`, `LineY`, `Series`

Use `ColumnY` for bar values and `LineY` for line values (NOT `Y`).
Visual type: `lineClusteredColumnComboChart` or `lineStackedColumnComboChart`

### Table

**Query roles:** `Values` — each column/measure is a separate projection in the same array

```json
"visual": {
  "visualType": "tableEx",
  "query": {
    "queryState": {
      "Values": { "projections": [
        { "field": { "Column": { "Expression": { "SourceRef": { "Entity": "TABLE" } }, "Property": "COL1" } }, "queryRef": "TABLE.COL1", "nativeQueryRef": "COL1" },
        { "field": { "Measure": { "Expression": { "SourceRef": { "Entity": "MTABLE" } }, "Property": "M1" } }, "queryRef": "MTABLE.M1", "nativeQueryRef": "M1" },
        { "field": { "Measure": { "Expression": { "SourceRef": { "Entity": "MTABLE" } }, "Property": "M2" } }, "queryRef": "MTABLE.M2", "nativeQueryRef": "M2" }
      ]}
    }
  }
}
```

### Matrix

**Query roles:** `Rows`, `Columns`, `Values`

Visual type: `pivotTable`

### Slicer

**Query roles:** `Values` — the field to slice by

Visual type: `slicer`

### Donut / Pie

**Query roles:** `Category` (slices), `Y` (value)

Visual types: `donutChart`, `pieChart`

---

## Field Reference Patterns

### Column (for axes, categories, slicers):
```json
{ "Column": { "Expression": { "SourceRef": { "Entity": "TABLE_NAME" } }, "Property": "COLUMN_NAME" } }
```

### Measure (for values, KPIs):
```json
{ "Measure": { "Expression": { "SourceRef": { "Entity": "TABLE_NAME" } }, "Property": "MEASURE_NAME" } }
```

### Rules:
- `Entity` = exact table name from semantic model (case-sensitive)
- `Property` = exact column or measure name (case-sensitive)
- `queryRef` = `"Table.Field"` (dot-separated)
- `nativeQueryRef` = just the field name

---

## Layout Grid Positions

### Standard Canvas (1280x720) — No Sidebar

**4 KPI Cards at Top:**
| KPI | x | y | width | height |
|-----|---|---|-------|--------|
| KPI 1 | 30 | 80 | 280 | 130 |
| KPI 2 | 330 | 80 | 280 | 130 |
| KPI 3 | 630 | 80 | 280 | 130 |
| KPI 4 | 930 | 80 | 280 | 130 |

**Header Bar:**
| Element | x | y | width | height |
|---------|---|---|-------|--------|
| Header | 0 | 0 | 1280 | 60 |

**2x2 Visual Grid (below KPIs):**
| Visual | x | y | width | height |
|--------|---|---|-------|--------|
| Top-Left | 30 | 230 | 600 | 230 |
| Top-Right | 650 | 230 | 600 | 230 |
| Bottom-Left | 30 | 480 | 600 | 230 |
| Bottom-Right | 650 | 480 | 600 | 230 |

**Single Large Visual (below KPIs):**
| Visual | x | y | width | height |
|--------|---|---|-------|--------|
| Full Width | 30 | 230 | 1220 | 480 |

### Wide Canvas (1600x900) — With Sidebar

**Header + Sidebar Layout:**
| Element | x | y | width | height |
|---------|---|---|-------|--------|
| Header | 0 | 0 | 1600 | 60 |
| Sidebar | 0 | 60 | 220 | 840 |

**4 KPI Cards (to the right of sidebar):**
| KPI | x | y | width | height |
|-----|---|---|-------|--------|
| KPI 1 | 240 | 80 | 310 | 130 |
| KPI 2 | 570 | 80 | 310 | 130 |
| KPI 3 | 900 | 80 | 310 | 130 |
| KPI 4 | 1230 | 80 | 310 | 130 |

**2x2 Visual Grid (to the right of sidebar, below KPIs):**
| Visual | x | y | width | height |
|--------|---|---|-------|--------|
| Top-Left | 240 | 230 | 650 | 310 |
| Top-Right | 910 | 230 | 650 | 310 |
| Bottom-Left | 240 | 560 | 650 | 310 |
| Bottom-Right | 910 | 560 | 650 | 310 |

### Z-Order Convention:
- Slicers/header: z = 500-999
- KPI cards: z = 1000-1099
- Main visuals: z = 2000-2099
- Decorative elements: z = 100-499

---

## Implementation Method

### Writing Files

Use Node.js (.mjs) scripts to write files. This avoids the EEXIST error that the Write tool encounters on paths with spaces. The script handles schema detection, page numbering, file writing, and validation in one pass.

```javascript
import { writeFileSync, mkdirSync, readFileSync, readdirSync, existsSync } from 'fs';
import { join } from 'path';

const projectBase = 'PATH_TO_PBIP_PROJECT';
const reportDef = join(projectBase, 'ProjectName.Report', 'definition');

// --- STEP 0: Detect schema version from existing visuals ---
// Read any existing visual.json to get the schema version Desktop uses.
// Fall back to 2.0.0 only if no visuals exist at all.
function detectSchemaVersion(pagesDir) {
  const defaultSchema = 'https://developer.microsoft.com/json-schemas/fabric/item/report/definition/visualContainer/2.0.0/schema.json';
  try {
    const pages = readdirSync(pagesDir).filter(d => d.startsWith('pg'));
    for (const pg of pages) {
      const visDir = join(pagesDir, pg, 'visuals');
      if (!existsSync(visDir)) continue;
      const visuals = readdirSync(visDir);
      for (const v of visuals) {
        const f = join(visDir, v, 'visual.json');
        if (existsSync(f)) {
          const j = JSON.parse(readFileSync(f, 'utf8'));
          if (j['$schema']) return j['$schema'];
        }
      }
    }
  } catch (e) { /* ignore */ }
  return defaultSchema;
}

// --- STEP 0b: Auto-increment page number ---
// Scan existing pages to find the next available pg## number.
function nextPageNumber(pagesDir) {
  const pages = readdirSync(pagesDir).filter(d => d.startsWith('pg'));
  const nums = pages.map(p => parseInt(p.replace(/^pg(\d+).*/, '$1'))).filter(n => !isNaN(n));
  return nums.length > 0 ? Math.max(...nums) + 1 : 1;
}

const SCHEMA = detectSchemaVersion(join(reportDef, 'pages'));
const NEXT_NUM = nextPageNumber(join(reportDef, 'pages'));
const PAGE_NAME = `pg${String(NEXT_NUM).padStart(2, '0')}Overview`;

// --- STEP 1: Create page folder ---
const pageDir = join(reportDef, 'pages', PAGE_NAME);
const visualsDir = join(pageDir, 'visuals');
mkdirSync(visualsDir, { recursive: true });

// --- STEP 2: Write page.json ---
writeFileSync(join(pageDir, 'page.json'), JSON.stringify(pageJson, null, 2), 'utf8');

// --- STEP 3: Create visual folders and write visual.json files ---
// Every visual.json gets the detected $schema version.
for (const visual of visuals) {
  visual['$schema'] = SCHEMA;
  const vDir = join(visualsDir, visual.name);
  mkdirSync(vDir, { recursive: true });
  writeFileSync(join(vDir, 'visual.json'), JSON.stringify(visual, null, 2), 'utf8');
}

// --- STEP 4: Update pages.json ---
const pagesJsonPath = join(reportDef, 'pages', 'pages.json');
const pagesJson = JSON.parse(readFileSync(pagesJsonPath, 'utf8'));
if (!pagesJson.pageOrder.includes(PAGE_NAME)) {
  pagesJson.pageOrder.push(PAGE_NAME);
}
writeFileSync(pagesJsonPath, JSON.stringify(pagesJson, null, 2), 'utf8');

// --- STEP 5: Validate all generated JSON files ---
// Parse every file we just wrote to catch malformed JSON before Desktop sees it.
let allValid = true;
function validateDir(dir) {
  for (const entry of readdirSync(dir, { withFileTypes: true })) {
    if (entry.isDirectory()) validateDir(join(dir, entry.name));
    else if (entry.name.endsWith('.json')) {
      try {
        JSON.parse(readFileSync(join(dir, entry.name), 'utf8'));
        console.log('OK:', entry.name);
      } catch (e) {
        console.error('INVALID JSON:', join(dir, entry.name), e.message);
        allValid = false;
      }
    }
  }
}
validateDir(pageDir);
console.log(allValid ? '\n✓ All JSON valid!' : '\n✗ Fix invalid files before opening in Desktop!');
```

Save the script as `.mjs` (not `.js`) and run with `node script.mjs`. The script auto-detects the schema version, picks the next page number, and validates all output.

### Important Caveats

1. **Power BI Desktop must be closed** before writing files, then reopen the `.pbip` file
2. **Invalid JSON causes blocking errors** — always run the validation step (Step 5 in the template) after writing files. A single missing comma or bracket will prevent Desktop from loading the page.
3. **Entity and Property names are case-sensitive** — must match semantic model exactly. Read the TMDL files to confirm exact names before referencing them in visuals.
4. **The name field must match the folder name** — for both pages and visuals
5. **Max 50 characters** for page/visual names
6. **Always include $schema** — Power BI Desktop validates against these schemas
7. **NEVER create a PBIP from scratch** — always work with an existing Desktop-saved project
8. **Semantic model entry point is `definition.pbism`** (JSON), NOT `definition.tmdl` — TMDL files go in `definition/` subfolder
9. **`.platform` files need real UUIDs** — only Desktop generates these correctly
10. **Detect schema version from existing visuals** — never hardcode a version. Read an existing `visual.json` to discover what version Desktop uses (e.g., `2.6.0`, `2.7.0`). The version changes with Desktop updates and mismatches cause errors.
11. **Read existing `report.json` first** — match its schema version and theme name when adding pages
12. **Auto-increment page numbers** — scan existing `pages/` folder for the highest `pg##` number and use the next one. Never assume a fixed number.
13. **Use IBCS colors consistently** — green `#44C088`, red `#ED7373`, actual `#0C3549`, comparison `#CCCCCC`. Do not mix in other green/red shades (like `#00B050`/`#FF0000`) as it creates visual inconsistency.
14. **MCP fallback** — if `powerbi-modeling-mcp` is not connected (Desktop closed), write measures directly to TMDL files. Check for existence first to avoid duplicates.

---

## Adding Measures When Desktop Is Closed

When `powerbi-modeling-mcp` is unavailable (Desktop not running or MCP not connected), write measures directly to the TMDL files. This is the preferred approach when building a complete report from scratch with Desktop closed.

**How to append measures to an existing TMDL file:**

1. Read the existing `.tmdl` file (e.g., `definition/tables/!Measure.tmdl`)
2. Check if each measure already exists (`tmdl.includes("measure 'MeasureName'")`)
3. Append new measure blocks before the first `column` definition
4. Each measure block uses tab indentation and follows TMDL syntax:

```
\tmeasure 'Measure Name' =
\t\t\t
\t\t\tDAX_EXPRESSION_HERE
\t\tformatString: #,##0.00 €
\t\tdisplayFolder: FolderName
\t\tlineageTag: <uuid>
```

**Key rules for TMDL measure writing:**
- Use `crypto.randomUUID()` (Node.js) for lineageTag values
- Expression goes on the line after `=`, indented with 3 tabs
- Multi-line DAX: use triple backticks (` ``` `) after `=` and close with ` ``` ` on a new line
- `formatString`, `displayFolder`, `lineageTag` are indented with 2 tabs
- Insert new measures BEFORE the first `\tcolumn ` line (measures must come before columns in TMDL)
- Always check if a measure already exists before adding to avoid duplicates

**KPI Color measures** should use the IBCS color palette:
```dax
// Positive = green (#44C088), Negative = red (#ED7373)
IF([Variance Measure] >= 0, "#44C088", "#ED7373")
// For cost measures where lower is better, invert the logic:
IF([Cost Variance] <= 0, "#44C088", "#ED7373")
```

---

## Two-Layer Architecture

This skill works as part of a two-layer approach for Power BI development:

| Layer | Tool | What it handles |
|-------|------|----------------|
| **Semantic Model** | `powerbi-modeling-mcp` (MCP server) | Tables, columns, measures, relationships, partitions, roles |
| **Report / Visuals** | This skill (PBIR JSON) | Pages, visuals, layout, formatting, filters |

**The MCP server connects to a running Power BI Desktop instance** and can create/modify the data model live. This skill writes JSON files to disk that Desktop reads on next open.

**Workflow A — MCP available (Desktop running):**
1. MCP server creates measures/columns as needed
2. User closes Desktop
3. This skill writes page/visual JSON files
4. User reopens Desktop to see new pages

**Workflow B — MCP unavailable (Desktop closed, which is common):**
1. Read existing TMDL files to discover table/measure names
2. Append any missing measures directly to the TMDL file
3. This skill writes page/visual JSON files
4. User opens Desktop — everything loads together

Workflow B is actually the most common scenario because Power BI Desktop must be closed for us to write files. Try MCP first; if it fails with "no connection", fall back to direct TMDL writing without asking the user.

---

## IBCS Visuals (Integrated)

This skill includes full support for **IBCS (International Business Communication Standards)** variance charts and tables using only native Power BI visuals — no paid add-ons required.

**Trigger on:** "IBCS", "variance chart", "variance table", "actual vs plan", "actual vs comparison", "AC vs PY"

### IBCS Templates

| # | Template | Visual Type | Measures Generated | Best For |
|---|----------|-------------|-------------------|----------|
| 1 | Column Variance Chart | `lineClusteredColumnComboChart` | 15 DAX measures (TMDL) | Time series comparison |
| 2 | Bar Variance Chart | `barChart` | 2-4 DAX + 13 NativeVisualCalculation | Ranked category comparison |
| 3 | Variance Table (Simple) | `pivotTable` | 2-3 SVG measures (TMDL) | Table with numeric AC + SVG variance |
| 4 | Variance Table (Full) | `pivotTable` | 3-4 SVG measures (TMDL) | Full SVG table (AC, PY bars + variance) |

### IBCS Workflow

1. **Collect 3 inputs:** actual measure, comparison measure, category column
2. **Recommend template** based on analysis type (time trend → column, ranking → bar, detail → table)
3. **Generate all helper DAX measures** — the skill is 100% measure-agnostic
4. **Write visual.json** to the PBIP project's report folder
5. User reloads in Power BI Desktop

### IBCS Color Palette
- Actual: `#0C3549` (dark blue) | Comparison: `#CCCCCC` (gray)
- Positive variance: `#44C088` (green) | Negative variance: `#ED7373` (red)

For full IBCS workflow details, template selection, and generation steps, read: `references/ibcs-visuals/IBCS-REFERENCE.md`

---

## Context Sources

Before creating any deliverable, check these references:
- **Consultant Profile:** `00 - Business Profile/brand-identity/CONSULTANT-PROFILE.md` — Who Lukas is, his expertise, clients, services, voice & tone
- **Brand Identity:** Load the `brd-brand-identity` skill for colors, fonts, and design guidelines

## Related Skills

- **pbi-dependency-analyzer** — Analyze model dependencies before building visuals. Understand what measures exist and which tables connect
- **pro-background-designer-svg** — Create matching background SVGs for report pages
- **brd-brand-identity** — Follow brand colors and fonts for visual consistency
- **ops-learning-log** — Check `power-bi.md` learnings for PBIR patterns and conventions
