# PBIR Report Builder

A Claude Code skill that lets you create Power BI report pages and visuals by simply describing what you want. Claude writes the PBIR JSON files directly into your PBIP project — you just reopen it in Power BI Desktop and everything appears.

> **Work in Progress:** This skill is under active development and reflects my ongoing progress in reverse-engineering the PBIR format. You may encounter issues with certain visual types or edge cases. Feedback and contributions are welcome.

## The Problem

Building Power BI reports manually means dragging visuals one by one, clicking through formatting menus, and repeating the same layout work for every page. PBIP (Power BI Project) files store everything as JSON, which means they *can* be generated programmatically — but the JSON structure is complex and undocumented.

## The Solution

This skill teaches Claude the entire PBIR file format: the folder structure, the JSON schemas, the query bindings, and the formatting patterns. It includes ready-to-use JSON templates for every common visual type, plus full IBCS variance chart support.

### How we built it

1. **Reverse-engineered the PBIR format** — We studied how Power BI Desktop saves PBIP projects and mapped out the JSON structure for pages, visuals, queries, and formatting
2. **Created 12 JSON templates** — Pre-built patterns for KPI cards, bar/column charts, line charts, combo charts, tables, matrices, slicers, donut charts, and page types (standard, drillthrough, tooltip)
3. **Bundled 7 JSON schemas** — Microsoft's own validation schemas so Claude can verify the output before writing
4. **Added IBCS visuals** — Four variance chart/table templates following International Business Communication Standards, built entirely with native Power BI visuals (no paid add-ons needed)

### What's in the `references/` folder

The skill isn't just a single instruction file — it comes with a full reference library that Claude reads as needed:

```
references/
  visual-types.md          35+ visual type identifiers and their query roles
  field-references.md      How to bind columns and measures to visuals
  formatting-objects.md    JSON patterns for container/visual/page formatting
  folder-structure.md      Complete annotated PBIP/PBIR folder tree
  page-naming.md           Naming convention for pages and visuals
  navigation-templates.md  Navigation bar patterns

  json-templates/          12 ready-to-use visual templates
    card-kpi.json          KPI card with current year, prior year, YoY%, conditional color
    clustered-column.json  Vertical bar chart
    clustered-bar.json     Horizontal bar chart
    line-chart.json        Line trend over time
    combo-chart.json       Column + line combo
    table.json             Table with multiple columns/measures
    matrix.json            Pivot table with rows/columns/values
    slicer.json            Slicer dropdown
    donut.json             Donut/pie chart
    page-standard.json     Standard page definition
    page-drillthrough.json Drillthrough page
    report-settings.json   Report-level settings

  json-schemas/            7 Microsoft PBIR schemas for validation

  ibcs-visuals/            IBCS variance chart system
    IBCS-SKILL.md          Full IBCS workflow and template selection guide
    references/
      ibcs-colors.md       IBCS color palette (dark blue, gray, green, red)
      ibcs-dax-measures.md DAX measure templates for variance calculations
      ibcs-svg-measures.md SVG measure templates for inline chart bars
      ibcs-column-variance.md  Combo chart visual.json pattern
      ibcs-bar-variance.md     Bar chart with visual calculations
      ibcs-table-simple.md     Matrix with SVG variance bars
      ibcs-table-full.md       Full SVG table (AC, PY bars + variance)
```

## Getting Started (Step by Step)

### Step 1: Install the skill

**Easiest way** — download the `.skill` file and add it to Claude:

1. Download [`pbir-report-builder.skill`](./pbir-report-builder.skill) (click → download icon)
2. In Claude Desktop or Cowork: Settings → Skills → Add Skill
3. Select the downloaded file — done!

> **For developers:** You can also clone the repo and copy the `pbir-report-builder/` folder to `~/.claude/skills/`.

### Step 2: Prepare your Power BI project

1. Open **Power BI Desktop**
2. Connect to your data source (SQL, Excel, CSV, etc.)
3. Create the measures you need (e.g., Total Sales, Total Cost, YoY %)
4. **Save as PBIP format**: File > Save As > select "Power BI Project (.pbip)"

This creates the project folder structure that the skill writes into. The skill never creates a PBIP from scratch — it only adds pages and visuals to an existing project.

### Step 3: Close Power BI Desktop

The skill writes JSON files directly to your project folder. Power BI Desktop locks these files while open, so you need to close it first.

### Step 4: Tell Claude what you want

Open Claude Code and describe the report page you want. Claude will ask you for:

- **The PBIP project path** — where your .pbip file is saved
- **What visuals you want** — KPI cards, charts, tables, etc.
- **Which measures and columns to use** — the exact names from your semantic model (case-sensitive)

Example prompts:

> *"Create a sales overview page in my PBIP project at C:/Reports/SalesReport. I want 4 KPI cards at the top showing Total Revenue, Total Cost, Gross Profit, and Profit Margin. Below that, a clustered bar chart by Region and a line chart trending monthly revenue."*

> *"Add an IBCS column variance chart to my report comparing [Total Revenue] (actual) vs [Total Revenue PY] (prior year), broken down by month."*

### Step 5: Reopen in Power BI Desktop

Open your `.pbip` file in Power BI Desktop. The new page appears with all visuals placed and connected to your data.

## IBCS Variance Charts

One of the most powerful features. [IBCS](https://www.ibcs.com/) is a standard for consistent, clear business charts — typically requiring expensive add-on visuals. This skill builds them with native Power BI visuals only.

### How IBCS works in this skill

You provide just three inputs:

1. **Actual measure** — e.g., `Total Revenue`
2. **Comparison measure** — e.g., `Total Revenue PY`
3. **Category column** — e.g., `DimDate[Month]` or `DimProduct[Category]`

The skill then:
- Generates all helper DAX measures (absolute variance, percentage variance, conditional colors, etc.)
- Creates the visual JSON with the correct chart type and formatting
- Applies the IBCS color palette (dark blue for actual, gray for comparison, green/red for variance)

### Four IBCS templates

| Template | Visual | When to use |
|----------|--------|-------------|
| **Column Variance** | Combo chart (columns + line) | Comparing actual vs prior year over time (months, quarters) |
| **Bar Variance** | Stacked bar chart | Ranking categories (regions, products) with variance |
| **Simple Table** | Matrix with SVG bars | Tabular view with numbers + inline variance bars |
| **Full Table** | Matrix with full SVG | Complete visual table showing actual bars, comparison bars, and variance |

Example prompt:

> *"Create an IBCS bar variance chart comparing Total Sales (actual) vs Budget Sales (plan) by product category"*

Claude generates approximately 15 DAX measures and a fully formatted visual — all from that one sentence.

## Scope

This skill focuses entirely on the **report and visual layer** — pages, visuals, layout, formatting, and filters. The semantic model (tables, columns, measures, relationships) is managed in Power BI Desktop as usual. Just make sure your measures exist before using this skill.

## Uninstall

In Claude Desktop or Cowork: Settings → Skills → remove the skill. If you installed manually, delete the `~/.claude/skills/pbir-report-builder` folder.
