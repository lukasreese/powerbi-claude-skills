# Page & Visual Naming Convention

By default, Power BI generates random 20-character hex IDs for page and visual folder names (e.g., `8dca1fe4a154d6b4196d`). These are unreadable in file explorers and version control diffs.

**PBIR allows readable names** — the only constraints are:
- Max 50 characters
- Must be unique within the report (pages) or within the page (visuals)
- Must match the `name` field inside `page.json` / `visual.json`
- No special characters that break file system paths

## Naming Convention

### Pages: `pg##ShortDescription`

| Pattern | Example | displayName |
|---|---|---|
| `pg##Name` | `pg01Overview` | Overview |
| | `pg02SalesAnalysis` | Sales Analysis |
| | `pg03ProductDetail` | Product Detail |
| | `pg04Drillthrough` | Drillthrough |
| | `pg05Tooltip` | Tooltip |
| | `pg06MonthlyTrend` | Monthly Trend |

- **`pg`** prefix identifies it as a page
- **`##`** is a 2-digit sequence number (01-99) for ordering
- **`ShortDescription`** in PascalCase — what the page shows

### Visuals: `v##TypeDescription`

| Pattern | Example | Visual Type |
|---|---|---|
| `v##Kpi...` | `v01KpiTotalSales` | cardVisual (KPI) |
| `v##Bar...` | `v02BarByRegion` | clusteredBarChart |
| `v##Col...` | `v03ColByMonth` | clusteredColumnChart |
| `v##Line...` | `v04LineTrend` | lineChart |
| `v##Combo...` | `v05ComboSalesProfit` | lineClusteredColumnComboChart |
| `v##Table...` | `v06TableDetail` | tableEx |
| `v##Matrix...` | `v07MatrixPnL` | pivotTable |
| `v##Slicer...` | `v08SlicerDate` | slicer |
| `v##Donut...` | `v09DonutSplit` | donutChart |
| `v##Gauge...` | `v10GaugeTarget` | gauge |
| `v##Btn...` | `v11BtnDrillthrough` | actionButton |
| `v##Shape...` | `v12ShapeDivider` | basicShape |
| `v##Img...` | `v13ImgLogo` | image |

- **`v`** prefix identifies it as a visual
- **`##`** is a 2-digit sequence (01-99) for z-order / tab-order
- **`TypeDescription`** — visual type abbreviation + what it shows

### Bookmarks: `bm##Description`

| Example | Description |
|---|---|
| `bm01Default` | Default view |
| `bm02Filtered` | Filtered state |
| `bm03DrillActive` | Drillthrough active |

## How to Apply

### For new pages (recommended):
```json
// pages.json
{
  "pageOrder": ["pg01Overview", "pg02SalesAnalysis", "pg03Drillthrough"],
  "activePageName": "pg01Overview"
}

// pg01Overview/page.json
{
  "name": "pg01Overview",
  "displayName": "Overview",
  ...
}
```

### For new visuals:
```json
// pg01Overview/visuals/v01KpiTotalSales/visual.json
{
  "name": "v01KpiTotalSales",
  "position": { "x": 30, "y": 80, ... },
  "visual": { "visualType": "cardVisual", ... }
}
```

### Three things must match:
1. **Folder name** = `pg01Overview`
2. **`name` field** in `page.json` = `"pg01Overview"`
3. **`pages.json` pageOrder** entry = `"pg01Overview"`

Same for visuals:
1. **Folder name** = `v01KpiTotalSales`
2. **`name` field** in `visual.json` = `"v01KpiTotalSales"`

## Renaming Existing Pages

If you need to rename pages from hex IDs to readable names:

1. Rename the folder (e.g., `8dca1fe4a154d6b4196d` → `pg01Overview`)
2. Update `name` in `page.json` to `"pg01Overview"`
3. Update the entry in `pages.json` pageOrder array
4. Update any `activePageName` references
5. **Restart Power BI Desktop** (it doesn't detect external renames while running)

> **Caution**: Renaming may break bookmarks that reference old page names. Check `bookmarks/` folder if present.
