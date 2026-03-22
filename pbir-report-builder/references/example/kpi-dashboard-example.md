# KPI Dashboard — Complete Example

A ready-to-run example that generates a standard KPI dashboard page using the PBIR Report Builder skill. Use this as a starting point and adapt it to your own model.

---

## What Gets Built

- **Page:** `pg01Overview` — 1280×720 canvas
- **4 KPI Cards** (top row): Total Sales, Gross Margin %, Total Units, YoY Growth
- **1 Clustered Bar Chart** (bottom left): Sales by Product Category
- **1 Line Chart** (bottom right): Monthly Sales Trend
- **1 Slicer** (top right): Year filter

**Visual layout:**

```
┌─────────────────────────────────────────────────────────────┐
│  [Header Bar — full width]                                  │
├──────────────┬──────────────┬──────────────┬───────────────┤
│  Total Sales │ Gross Margin │ Total Units  │  YoY Growth   │
│  KPI Card    │  KPI Card    │  KPI Card    │   KPI Card    │
├──────────────┴──────────────┼──────────────┴───────────────┤
│                             │                               │
│  Sales by Category          │  Monthly Sales Trend          │
│  (Clustered Bar)            │  (Line Chart)                 │
│                             │                               │
└─────────────────────────────┴───────────────────────────────┘
```

Canvas: 1280×720 | Positions use the Standard layout grid

---

## Prerequisites

1. A PBIP project saved from Power BI Desktop
2. Semantic model with these tables/columns (rename in the script to match yours):

| Object | Type | Example Name |
|--------|------|-------------|
| `_Measures` | Measures table | your measures table |
| `Total Sales` | Measure | current year sales |
| `Total Sales PY` | Measure | prior year sales |
| `Sales YoY %` | Measure | year-over-year % change |
| `Gross Margin %` | Measure | margin as percentage |
| `Gross Margin % PY` | Measure | prior year margin % |
| `Margin YoY %` | Measure | margin YoY % change |
| `Total Units` | Measure | unit count |
| `Total Units PY` | Measure | prior year units |
| `Units YoY %` | Measure | units YoY % change |
| `Sales KPI Color` | Measure | returns `#44C088` or `#ED7373` |
| `Margin KPI Color` | Measure | returns `#44C088` or `#ED7373` |
| `Units KPI Color` | Measure | returns `#44C088` or `#ED7373` |
| `DimDate[Month]` | Column | month axis for line chart |
| `DimDate[Year]` | Column | slicer field |
| `DimProduct[Category]` | Column | bar chart axis |

---

## KPI Color Measures (DAX)

Add these to your measures table if they don't already exist. When Desktop is closed, write them directly to the TMDL file.

```dax
-- Sales KPI Color
Sales KPI Color =
    IF([Sales YoY %] >= 0, "#44C088", "#ED7373")

-- Margin KPI Color
Margin KPI Color =
    IF([Gross Margin % PY] = 0, "#9CA3AF",
       IF([Gross Margin %] >= [Gross Margin % PY], "#44C088", "#ED7373"))

-- Units KPI Color
Units KPI Color =
    IF([Units YoY %] >= 0, "#44C088", "#ED7373")
```

> **Color rule:** Always use `#44C088` (green) and `#ED7373` (red). Never use `#00B050` or `#FF0000` — those clash with the IBCS visual standards elsewhere in the report.

---

## Generation Script

Save as `kpi-dashboard.mjs` in your project folder and run with `node kpi-dashboard.mjs`.

**Before running:** Update `PROJECT_BASE`, `REPORT_FOLDER`, `MEASURES_TABLE`, and `DATE_TABLE` / `PRODUCT_TABLE` to match your project.

```javascript
import { writeFileSync, mkdirSync, readFileSync, readdirSync, existsSync } from 'fs';
import { join } from 'path';

// ─── CONFIGURE THESE ───────────────────────────────────────────────────────────
const PROJECT_BASE    = 'C:/path/to/YourProject.Report';
const REPORT_FOLDER   = join(PROJECT_BASE, 'definition');
const MEASURES_TABLE  = '_Measures';   // exact name of your measures table
const DATE_TABLE      = 'DimDate';     // exact name of your date table
const PRODUCT_TABLE   = 'DimProduct';  // exact name of your product/category table
// ───────────────────────────────────────────────────────────────────────────────

// ─── Schema detection ─────────────────────────────────────────────────────────
function detectSchema(pagesDir) {
  const fallback = 'https://developer.microsoft.com/json-schemas/fabric/item/report/definition/visualContainer/2.0.0/schema.json';
  try {
    for (const pg of readdirSync(pagesDir).filter(d => d.startsWith('pg'))) {
      const visDir = join(pagesDir, pg, 'visuals');
      if (!existsSync(visDir)) continue;
      for (const v of readdirSync(visDir)) {
        const f = join(visDir, v, 'visual.json');
        if (existsSync(f)) {
          const j = JSON.parse(readFileSync(f, 'utf8'));
          if (j['$schema']) return j['$schema'];
        }
      }
    }
  } catch {}
  return fallback;
}

function nextPageNum(pagesDir) {
  const nums = readdirSync(pagesDir)
    .filter(d => d.startsWith('pg'))
    .map(p => parseInt(p.replace(/^pg(\d+).*/, '$1')))
    .filter(n => !isNaN(n));
  return nums.length ? Math.max(...nums) + 1 : 1;
}

const pagesDir  = join(REPORT_FOLDER, 'pages');
const SCHEMA    = detectSchema(pagesDir);
const NUM       = String(nextPageNum(pagesDir)).padStart(2, '0');
const PAGE_NAME = `pg${NUM}Overview`;

// ─── Helper: measure field reference ──────────────────────────────────────────
function measure(table, name) {
  return {
    field: { Measure: { Expression: { SourceRef: { Entity: table } }, Property: name } },
    queryRef: `${table}.${name}`,
    nativeQueryRef: name
  };
}

// ─── Helper: column field reference ───────────────────────────────────────────
function column(table, name) {
  return {
    field: { Column: { Expression: { SourceRef: { Entity: table } }, Property: name } },
    queryRef: `${table}.${name}`,
    nativeQueryRef: name
  };
}

// ─── Helper: KPI card visual ───────────────────────────────────────────────────
function kpiCard(name, x, z, tabOrder, mCY, mPY, mYoY, mColor) {
  return {
    '$schema': SCHEMA,
    name,
    position: { x, y: 80, z, width: 280, height: 130, tabOrder },
    visual: {
      visualType: 'cardVisual',
      objects: {
        calloutValue: [{
          properties: {
            color: {
              solid: {
                color: {
                  expr: {
                    Measure: {
                      Expression: { SourceRef: { Entity: MEASURES_TABLE } },
                      Property: mColor
                    }
                  }
                }
              }
            }
          }
        }],
        cards: [{ properties: { showLabel: { expr: { Literal: { Value: 'true' } } } } }],
        referenceLabel: [{ properties: { show: { expr: { Literal: { Value: 'true' } } } } }]
      },
      query: {
        queryState: {
          Data: { projections: [measure(MEASURES_TABLE, mCY)] },
          ReferenceLabels: { projections: [measure(MEASURES_TABLE, mPY)] },
          AdditionalMeasure: { projections: [measure(MEASURES_TABLE, mYoY)] }
        }
      },
      drillFilterOtherVisuals: true
    }
  };
}

// ─── Visuals ───────────────────────────────────────────────────────────────────

const visuals = [

  // ── KPI 1: Total Sales ──
  kpiCard('v01KpiTotalSales',   30,  1000, 0,
    'Total Sales', 'Total Sales PY', 'Sales YoY %', 'Sales KPI Color'),

  // ── KPI 2: Gross Margin % ──
  kpiCard('v02KpiGrossMargin',  330, 1001, 1,
    'Gross Margin %', 'Gross Margin % PY', 'Margin YoY %', 'Margin KPI Color'),

  // ── KPI 3: Total Units ──
  kpiCard('v03KpiUnits',        630, 1002, 2,
    'Total Units', 'Total Units PY', 'Units YoY %', 'Units KPI Color'),

  // ── KPI 4: YoY Growth (text-only card — single measure) ──
  {
    '$schema': SCHEMA,
    name: 'v04KpiYoYGrowth',
    position: { x: 930, y: 80, z: 1003, width: 280, height: 130, tabOrder: 3 },
    visual: {
      visualType: 'cardVisual',
      objects: {
        calloutValue: [{
          properties: {
            color: {
              solid: {
                color: {
                  expr: {
                    Measure: {
                      Expression: { SourceRef: { Entity: MEASURES_TABLE } },
                      Property: 'Sales KPI Color'
                    }
                  }
                }
              }
            }
          }
        }],
        cards: [{ properties: { showLabel: { expr: { Literal: { Value: 'true' } } } } }]
      },
      query: {
        queryState: {
          Data: { projections: [measure(MEASURES_TABLE, 'Sales YoY %')] }
        }
      },
      drillFilterOtherVisuals: true
    }
  },

  // ── Bar Chart: Sales by Category ──
  {
    '$schema': SCHEMA,
    name: 'v05BarByCategory',
    position: { x: 30, y: 230, z: 2000, width: 600, height: 460, tabOrder: 4 },
    visual: {
      visualType: 'clusteredBarChart',
      query: {
        queryState: {
          Category: { projections: [column(PRODUCT_TABLE, 'Category')] },
          Y: { projections: [measure(MEASURES_TABLE, 'Total Sales')] }
        }
      },
      drillFilterOtherVisuals: true
    }
  },

  // ── Line Chart: Monthly Sales Trend ──
  {
    '$schema': SCHEMA,
    name: 'v06LineTrend',
    position: { x: 650, y: 230, z: 2001, width: 600, height: 460, tabOrder: 5 },
    visual: {
      visualType: 'lineChart',
      query: {
        queryState: {
          Category: { projections: [column(DATE_TABLE, 'Month')] },
          Y: { projections: [
            measure(MEASURES_TABLE, 'Total Sales'),
            measure(MEASURES_TABLE, 'Total Sales PY')
          ]}
        }
      },
      drillFilterOtherVisuals: true
    }
  }

];

// ─── Write files ───────────────────────────────────────────────────────────────

const pageDir    = join(pagesDir, PAGE_NAME);
const visualsDir = join(pageDir, 'visuals');
mkdirSync(visualsDir, { recursive: true });

// page.json
const pageJson = {
  '$schema': 'https://developer.microsoft.com/json-schemas/fabric/item/report/definition/page/2.0.0/schema.json',
  name: PAGE_NAME,
  displayName: 'Overview',
  displayOption: 'FitToPage',
  width: 1280,
  height: 720
};
writeFileSync(join(pageDir, 'page.json'), JSON.stringify(pageJson, null, 2), 'utf8');

// visual files
for (const v of visuals) {
  const vDir = join(visualsDir, v.name);
  mkdirSync(vDir, { recursive: true });
  writeFileSync(join(vDir, 'visual.json'), JSON.stringify(v, null, 2), 'utf8');
}

// update pages.json
const pagesJsonPath = join(pagesDir, 'pages.json');
const pagesJson = JSON.parse(readFileSync(pagesJsonPath, 'utf8'));
if (!pagesJson.pageOrder.includes(PAGE_NAME)) {
  pagesJson.pageOrder.push(PAGE_NAME);
}
writeFileSync(pagesJsonPath, JSON.stringify(pagesJson, null, 2), 'utf8');

// ─── Validate ─────────────────────────────────────────────────────────────────
let valid = true;
function validateDir(dir) {
  for (const e of readdirSync(dir, { withFileTypes: true })) {
    if (e.isDirectory()) validateDir(join(dir, e.name));
    else if (e.name.endsWith('.json')) {
      try {
        JSON.parse(readFileSync(join(dir, e.name), 'utf8'));
        console.log('✓', e.name);
      } catch (err) {
        console.error('✗ INVALID JSON:', join(dir, e.name), err.message);
        valid = false;
      }
    }
  }
}
validateDir(pageDir);
console.log(valid
  ? `\n✅ Done! Reload ${PROJECT_BASE.split('/').pop().replace('.Report','.pbip')} in Power BI Desktop`
  : '\n❌ Fix invalid JSON before opening in Desktop');
```

---

## After Running

1. **Close** Power BI Desktop if it's open
2. Run `node kpi-dashboard.mjs`
3. **Open** the `.pbip` file in Power BI Desktop
4. The `Overview` page appears with all 6 visuals

---

## Want to Improve This Example?

This is part of an open-source skill at **github.com/lukasreese/powerbi-claude-skills**.

If your real-world model produces different results, share them:
- **Submit a PR** with improved JSON templates
- **Open an issue** describing what broke
- **Share your PBIX/PBIP** so the skill can learn from real-world examples

Every example you contribute makes the skill better for everyone.
