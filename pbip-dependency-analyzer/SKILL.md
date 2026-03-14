---
name: pbip-dependency-analyzer
description: "Power BI PBIP Dependency Analyzer. Analyzes all Power BI Project (.pbip) files, TMDL exports, and semantic model definitions to map dependencies between measures, columns, tables, M/Power Query, relationships, and report visuals. Use this skill EVERY TIME the user asks to: find unused measures or columns, analyze dependencies, clean up a data model, check what would break if something is deleted, perform impact analysis, identify orphaned objects, audit model quality, or asks 'what can I delete'. Also trigger when the user uploads .pbip files, model.bim files, TMDL exports, or report.json files and wants to understand their structure or dependencies. Trigger phrases include: 'analyze dependencies', 'find unused', 'what can I delete', 'clean up model', 'impact analysis', 'dependency check', 'unused measures', 'unused columns', 'orphaned objects', 'model audit', 'PBIP analysis'."
---

# Power BI PBIP Dependency Analyzer

## Overview

This skill analyzes Power BI Project (PBIP) files to create a complete dependency map across all semantic models and reports. It identifies unused objects (measures, columns, calculated tables) that can be safely deleted, and warns about objects that would cause breaking changes if removed.

## When to Use

- User wants to find unused measures, columns, or tables
- User wants to know "what happens if I delete X"
- User uploads PBIP files or TMDL exports for analysis
- User wants to clean up or optimize a semantic model
- User asks about dependencies between objects
- User wants a model quality audit

## Analysis Workflow

When triggered, follow these steps in order:

### Step 1: Identify Available Files

Check what the user has provided. Look in project knowledge and uploads for:

```
Accepted file types:
- .pbip files (Power BI Project metadata)
- model.bim (full semantic model JSON)
- TMDL exports (.tmdl text files)
- report.json / pages/ folders (visual definitions)
- .pq / .m files (Power Query / M expressions)
```

PBIP folder structure reference:
```
MyReport.pbip
MyReport/
  definition/
    report.json              <- Visual field references
    pages/
      page1/
        visuals/             <- Individual visual configs
    model.bim                <- Full semantic model

MyModel.pbip  
MyModel/
  definition/
    model.bim                <- Semantic model definition
    tables/
      TableName/
        columns/             <- Column definitions
        measures/            <- Measure DAX definitions  
        partitions/          <- M/Power Query or DAX source
    relationships/           <- Relationship definitions
    expressions/             <- Shared M expressions
```

TMDL export structure (alternative to PBIP):
```
model.tmdl                   <- Model metadata
tables/
  TableName.tmdl             <- Table with columns, measures, partitions
relationships.tmdl           <- All relationships
expressions.tmdl             <- Shared expressions
```

### Step 2: Inventory All Objects

Create a complete inventory. Parse each file type as follows:

#### From Semantic Model (model.bim / TMDL / tables/):

**Tables** — For each table, record:
- Name
- Type: `imported` (M/Power Query source), `calculated` (DAX), or `calculationGroup`
- Source: The M expression or DAX expression that creates it

**Columns** — For each column, record:
- Name and parent table
- Type: `source` (from M/Power Query), `calculated` (DAX expression), or `rowNumber`
- If calculated: the full DAX expression
- isHidden flag

**Measures** — For each measure, record:
- Name and parent table
- Full DAX expression
- Display folder
- Description (if any)
- Format string
- isHidden flag

**Relationships** — For each relationship, record:
- From table + column
- To table + column
- Cardinality (One-to-Many, Many-to-Many, etc.)
- Cross-filter direction (Single, Both)
- isActive flag

**Calculated Tables** — Record:
- Name
- Full DAX expression
- All output columns

**Hierarchies** — Record:
- Name, parent table
- Level columns

#### From M/Power Query (partitions / expressions):

For each query/partition:
- Query name
- Full M expression
- Referenced queries (other queries used as source)
- Output columns
- Parameters used
- Merge/join operations and which columns they use

#### From Reports (report.json / pages/):

For each page and visual:
- Page name
- Visual type (table, matrix, card, chart, slicer, etc.)
- Fields in Rows/Columns/Values/Tooltips/Legend
- Filter fields (visual-level, page-level, report-level)
- Conditional formatting field references
- Drillthrough fields
- Sort-by fields
- Small multiples fields

**CRITICAL:** Conditional formatting measures often do NOT appear in the visual's value fields. They are defined in the visual's formatting config under `objects` → `values` → `backColor` / `foreColor` / `fontColor`. Always check these separately.

### Step 3: Build Dependency Graph

For each object, trace ALL inbound and outbound references:

#### DAX Reference Parsing

Scan every DAX expression (measures, calculated columns, calculated tables) for:

```
Reference patterns to detect:
- [MeasureName]                    -> measure reference
- TableName[ColumnName]            -> column reference  
- 'Table Name'[ColumnName]         -> column reference (quoted table)
- SELECTEDVALUE(Table[Column])     -> column reference
- CALCULATE(..., Table[Column])    -> column reference
- FILTER(TableName, ...)           -> table reference
- ALL(TableName)                   -> table reference
- VALUES(Table[Column])            -> table + column reference
- RELATEDTABLE(TableName)          -> table reference + relationship dependency
- RELATED(Table[Column])           -> column reference + relationship dependency
- USERELATIONSHIP(col1, col2)      -> specific relationship activation
- TREATAS(...)                     -> virtual relationship
```

Build two lists per object:
- **References** (what this object depends on)
- **Referenced By** (what depends on this object)

#### M/Power Query Reference Parsing

Scan M expressions for:
```
- Source references: = OtherQueryName
- Table.Join / Table.NestedJoin    -> which columns used in joins
- Table.SelectColumns              -> which columns are kept
- Table.RemoveColumns              -> which columns are dropped
- Table.RenameColumns              -> column name mappings
- #"Query Name"                    -> query references (quoted)
```

#### Relationship Dependencies

Columns used in relationships are ALWAYS considered active, even if they appear in no visual or DAX expression. Mark them as `ACTIVE - Relationship`.

#### Visual Dependencies

Map each visual field reference back to the semantic model:
- Direct measure references
- Direct column references
- Implicit table references (any column from a table makes the table active)

### Step 4: Classify Each Object

Assign each object exactly one status:

| Status | Code | Definition |
|--------|------|-----------|
| Used in Visual | `V` | Directly in at least one visual's values, rows, columns, legend, tooltip |
| Used in Filter | `F` | Used only in filters or slicers |
| Used in CF | `CF` | Used only for conditional formatting |
| Referenced by DAX | `D` | Not in any visual, but referenced by another measure/calc that IS used |
| Relationship Column | `R` | Column required for an active relationship |
| M Dependency | `M` | Column/table required by M/Power Query (join key, intermediate query) |
| Referenced by Orphan | `O-CHAIN` | Referenced only by objects that are themselves orphaned |
| Orphaned | `O` | Not referenced by anything, not in any visual |
| Broken Reference | `BROKEN` | References an object that does not exist |
| Circular | `CIRC` | Part of a circular dependency chain |

Classification priority (if multiple apply): V > F > CF > D > R > M > O-CHAIN > O

### Step 5: Generate Reports

Present findings in this order:

#### Report A: Object Inventory Summary

Quick stats:
```
Total tables: X (Y imported, Z calculated)
Total columns: X (Y source, Z calculated)
Total measures: X
Total relationships: X
Reports analyzed: X
Pages analyzed: X
Visuals analyzed: X
```

#### Report B: Unused Objects (Safe to Delete)

**Sort by impact (highest savings first):**

1. **Unused Calculated Tables** — Remove DAX calculated tables first (biggest memory/refresh savings)
2. **Unused Measures** — No data impact, just remove logic
3. **Unused Calculated Columns** — Saves recalculation time
4. **Unused Source Columns** — Requires M/Power Query edit (highest effort)

For each object:

| Object | Type | Table | Status | Risk | Why Unused |
|--------|------|-------|--------|------|------------|
| Name | Measure/Column/Table | Parent | O / O-CHAIN | LOW/MED/HIGH | No references found / Only referenced by [other orphan] |

Risk levels:
- **LOW** — Zero references in DAX, visuals, filters, CF, M. Safe to delete.
- **MEDIUM** — Referenced only by other orphaned objects. Delete the chain together.
- **HIGH** — No references found but isHidden=true (might be intentionally hidden for API/external use). Investigate.

#### Report C: Impact Analysis

For every ACTIVE object, show the dependency chain:

| Object | Direct Dependents | Indirect Dependents | Visuals | Pages | Reports |
|--------|-------------------|---------------------|---------|-------|---------|
| Name | [list] | [list] | [list] | [list] | [list] |

When user asks "what if I delete X", trace the full chain and list everything that would break.

#### Report D: Model Quality Warnings

Check for these issues:

| Category | Check | Severity |
|----------|-------|----------|
| Measures | No description | LOW |
| Measures | No display folder | LOW |
| Measures | Duplicate logic (similar DAX in multiple measures) | MEDIUM |
| Columns | Unused calculated column (expensive for nothing) | MEDIUM |
| Columns | Column with no references and isHidden=false | LOW |
| Relationships | Many-to-Many | HIGH |
| Relationships | Bidirectional cross-filter | MEDIUM |
| Relationships | Inactive relationship | LOW |
| Tables | Table with no relationships (island) | MEDIUM |
| Tables | Calculated table that could be M/Power Query | MEDIUM |
| M/Power Query | Query that disables folding unnecessarily | MEDIUM |
| M/Power Query | Intermediate query not marked as non-reportable | LOW |
| Model | Circular dependency | HIGH |
| Model | Broken references | CRITICAL |

#### Report E: Cross-Model Analysis (if multiple models)

If multiple semantic models are provided:
- Which reports connect to which models
- Shared table/column/measure names
- Potential consolidation opportunities
- Live connection dependencies (flag external reports risk)

### Step 6: Recommend Actions

Provide a prioritized action list:

```
PRIORITY 1 - Delete Now (zero risk):
- [objects with status O, risk LOW, no hidden flag]

PRIORITY 2 - Delete Together (low risk):  
- [orphan chains where all objects in chain are unused]

PRIORITY 3 - Investigate First (medium risk):
- [hidden objects, objects with unclear references]

PRIORITY 4 - Model Optimization:
- [quality warnings, performance improvements]

PRIORITY 5 - Cannot determine (external risk):
- [objects that might be used by Live Connection reports not in this project]
```

## Important Rules

1. **NEVER assume unused without checking ALL reference types**: DAX, M/Power Query, Visuals, Filters, CF, Drillthrough, Slicers, Sort-by, Tooltips, Bookmarks
2. **M/Power Query is a hidden dependency layer**: A column might look unused in DAX but be essential for a merge/join in M
3. **Relationship columns are ALWAYS active**: Even if not in any visual or DAX
4. **CF measures are invisible in visual fields**: Always check formatting config separately
5. **Hidden objects need investigation**: They might be hidden for external tools, API access, or Live Connection reports
6. **When in doubt, classify as WARNING not ORPHAN**: False positives (keeping something unused) are cheaper than false negatives (deleting something needed)
7. **Always warn about Live Connection risk**: Reports outside this project might depend on these objects
8. **Check isHidden on columns**: Hidden columns feeding calculated columns are common and must not be flagged as unused
9. **Calculated table columns inherit dependency**: If a calculated table is used, ALL its columns are considered used
10. **USERELATIONSHIP activates inactive relationships**: Those relationships and their columns become active in that measure's context

## Output Formatting

Always use tables for structured data. Keep analysis concise. Lead with actionable findings (what to delete) before detailed analysis.

When the user asks a specific question like "can I delete measure X", skip the full analysis and just trace that one object's dependencies. Show the chain and give a clear YES/NO answer with reasoning.
