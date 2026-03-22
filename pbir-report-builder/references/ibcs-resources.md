# IBCS — What It Is & How This Skill Implements It

---

## What is IBCS?

IBCS (International Business Communication Standards) is a set of rules for designing business reports and dashboards that communicate data clearly and consistently. The standards focus on removing visual noise, establishing uniform notation, and ensuring every chart tells its story without ambiguity.

The 7 IBCS principles follow the acronym **SUCCESS**:

| Principle | Meaning |
|-----------|---------|
| **S**ay | Convey a clear message — every chart answers a question |
| **U**nify | Use consistent notation across all reports |
| **C**ondense | Show more in less space — maximize data density |
| **C**heck | Verify content for accuracy and completeness |
| **E**xpress | Choose the right chart type for the data story |
| **S**implify | Remove all decoration that doesn't carry information |
| **S**tructure | Organize content logically for the reader |

---

## IBCS Notation Used in This Skill

### Color Encoding

IBCS specifies strict color meanings. This skill uses these exact values:

| Role | Color | Hex |
|------|-------|-----|
| **Actual (AC)** — current period values | Dark blue | `#0C3549` |
| **Previous Year (PY)** — comparison period | Light gray | `#CCCCCC` |
| **Positive variance** — actual better than comparison | Green | `#44C088` |
| **Negative variance** — actual worse than comparison | Red | `#ED7373` |

> **Rule:** Never substitute other greens or reds (`#00B050`, `#FF0000`, etc.). The IBCS palette is fixed throughout the skill for visual consistency.

### Chart Titles

IBCS requires structured titles:
- **Measure** | **Entity** | **Time** — e.g., "Net Sales | All Regions | Jan–Dec 2025 vs 2024"
- AC = Actual, PY = Previous Year, BU = Budget, FC = Forecast
- Always label comparison period on the chart

---

## The 4 IBCS Templates in This Skill

This skill builds IBCS-compliant visuals entirely from **native Power BI visuals** — no paid add-ons, no custom visuals, no third-party dependencies.

Each template is self-contained: you provide 3 inputs (actual measure, comparison measure, category/time column) and the skill generates all helper DAX measures + the PBIR JSON.

| # | Template | Visual Type | Description |
|---|----------|-------------|-------------|
| **1** | Column Variance Chart | `lineClusteredColumnComboChart` | Time-series comparison (AC vs PY by month/quarter). Columns = actual, line = comparison, bars = variance delta. Best for showing performance over time. |
| **2** | Bar Variance Chart | `barChart` with NativeVisualCalculation | Ranked category comparison (AC vs PY by product, region, etc.). Horizontal bars = actual, comparison markers = PY. Best for ranking many categories. |
| **3** | Variance Table (Simple) | `pivotTable` | Table with numeric actual values + SVG variance bars in a column. Clean, information-dense. Best for detailed breakdowns where precision matters. |
| **4** | Variance Table (Full) | `pivotTable` | Full SVG table — both AC and PY shown as proportional bars + variance column. Highest information density. Best for executive summaries. |

> For full generation instructions and DAX measure templates, see:
> - `ibcs-visuals/IBCS-REFERENCE.md` — workflow, template selection guide, generation steps
> - `ibcs-visuals/references/ibcs-dax-measures.md` — all 15 DAX measure templates
> - `ibcs-visuals/references/ibcs-svg-measures.md` — SVG measures for table visuals

---

## When to Use Which Template

```
Is the primary dimension TIME (months, quarters)?
  → YES → Use Template 1: Column Variance Chart
  → NO  → Are you comparing CATEGORIES (products, regions, customers)?
             → YES → Template 2: Bar Variance Chart (5+ categories)
                      Template 3: Variance Table Simple (detailed breakdown)
                      Template 4: Variance Table Full (executive summary)
```

**Rule of thumb:**
- Time series → column chart
- Rankings → bar chart
- Detailed numbers → simple table
- Executive 1-pager → full table

---

## Visual Gallery

See `visual-gallery/ibcs-visuals.md` for screenshots of all 4 templates as they appear in Power BI Desktop.
