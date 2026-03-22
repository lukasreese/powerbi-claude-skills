# TMDL Syntax Reference

TMDL (Tabular Model Definition Language) is an indentation-based format for Power BI semantic models. This reference covers everything needed to parse `.tmdl` files for dependency analysis.

## File Structure

```
ProjectName.SemanticModel/
  definition/
    database.tmdl          <- Database metadata (compatibilityLevel)
    model.tmdl             <- Model metadata + ref declarations (object ordering)
    relationships.tmdl     <- All relationships in one file
    expressions.tmdl       <- Named expressions / parameters (M queries)
    tables/
      Sales.tmdl           <- One file per table (columns, measures, partitions, hierarchies)
      Product.tmdl
      Calendar.tmdl
    roles/
      RoleName.tmdl        <- One file per RLS role
    cultures/
      en-US.tmdl           <- One file per translation culture
    perspectives/
      ViewName.tmdl        <- One file per perspective
```

## Indentation Rules

- **Tab characters** (not spaces) define hierarchy
- Level 0: Object declaration (`table`, `relationship`, etc.)
- Level 1: Properties and child objects (one tab)
- Level 2: Nested properties (two tabs)
- Blank lines between objects are optional separators

## Object Declaration

```
objectType ObjectName
    property: value
```

**Naming rules:**
- Names with dots, equals, colons, quotes, or whitespace must be single-quoted: `'My Table Name'`
- Escape single quotes by doubling: `'User''s Data'`
- Simple names need no quotes: `Sales`, `ProductKey`

## Property Delimiters

Two delimiters:

| Delimiter | Usage | Examples |
|-----------|-------|---------|
| `=` (equals) | Default properties, expressions (DAX/M) | `measure Sales = SUM(...)`, `partition p = m` |
| `:` (colon) | All other properties | `dataType: int64`, `isHidden: true` |

## Expressions (DAX and M)

**Single-line:**
```
measure TotalSales = SUM('Sales'[Amount])
    formatString: $ #,##0
```

**Multi-line (indented continuation):**
```
measure GrossProfit =
    var revenue = [TotalSales]
    var cost = [TotalCost]
    return revenue - cost
    formatString: $ #,##0
```

**Multi-line with triple backticks (preserves formatting exactly):**
```
measure ComplexCalc = ```
    var result = SUMX(
        'Sales',
        [Quantity] * [Net Price]
    )
    return result
    ```
    formatString: $ #,##0
```

**Key rule:** Everything after `=` until the next property at the same indent level is the expression body.

## Descriptions (Triple-Slash)

```
/// Total sales revenue in USD
/// Updated monthly from source system
measure 'Sales Amount' = SUM('Sales'[Amount])
```

Descriptions appear immediately before the object declaration, no blank lines between.

---

## Table Definition

```
table Sales

    measure 'Total Sales' = SUM('Sales'[Amount])
        formatString: $ #,##0.00
        displayFolder: Revenue
        lineageTag: 8a2b3c4d-...

    measure 'Sales PY' = CALCULATE([Total Sales], SAMEPERIODLASTYEAR('Calendar'[Date]))
        formatString: $ #,##0.00
        displayFolder: Revenue

    column Amount
        dataType: double
        formatString: #,##0.00
        sourceColumn: Amount
        summarizeBy: sum
        lineageTag: 1a2b3c4d-...

    column 'Product Key'
        dataType: int64
        isHidden
        sourceColumn: ProductKey
        summarizeBy: none

    column 'Profit Margin' = DIVIDE([Amount] - [Cost], [Amount])
        dataType: double
        formatString: 0.00%
        summarizeBy: none
        isDataTypeInferred

    partition Sales = m
        mode: import
        source =
            let
                Source = Sql.Database(Server, Database),
                Sales = Source{[Schema="dbo", Item="Sales"]}[Data]
            in
                Sales

    hierarchy 'Product Hierarchy'
        level Category
            column: Category
        level Subcategory
            column: Subcategory
```

## Column Properties

| Property | Type | Values/Description |
|----------|------|-------------------|
| `dataType` | enum | `string`, `int64`, `double`, `decimal`, `dateTime`, `boolean`, `binary` |
| `sourceColumn` | string | Source column name from partition (imported columns) |
| `expression` (via `=`) | DAX | Calculated column expression (after column name) |
| `isHidden` | boolean | `true`/`false` (or just `isHidden` for true) |
| `formatString` | string | Display format |
| `summarizeBy` | enum | `none`, `sum`, `average`, `count`, `countNonNull`, `min`, `max` |
| `sortByColumn` | ref | Column name to sort by |
| `dataCategory` | string | `Address`, `City`, `Country`, `Latitude`, `Longitude`, `ImageUrl`, etc. |
| `isKey` | boolean | Primary key column |
| `isNameInferred` | boolean | Name derived from source |
| `isDataTypeInferred` | boolean | Type derived from source |
| `lineageTag` | guid | Unique identifier |

**Distinguishing column types:**
- **Source column** (imported): Has `sourceColumn` property, no `=` expression
- **Calculated column**: Has `=` expression after column name, no `sourceColumn`
- **Row number**: `column RowNumber` with `isHidden` and `isKey` (auto-generated)

## Measure Properties

| Property | Type | Description |
|----------|------|-------------|
| `expression` (via `=`) | DAX | The DAX formula (after measure name) |
| `formatString` | string | Display format (e.g., `$ #,##0`, `0.00%`) |
| `displayFolder` | string | Folder in field list |
| `description` | string | Measure description (or use `///` syntax) |
| `isHidden` | boolean | Hidden from report view |
| `dataCategory` | string | e.g., `ImageUrl` for SVG measures |
| `lineageTag` | guid | Unique identifier |

## Partition Types

**M/Power Query partition (most common):**
```
partition PartitionName = m
    mode: import
    source =
        let
            Source = Sql.Database(ServerParam, DatabaseParam),
            Table = Source{[Schema="dbo", Item="TableName"]}[Data]
        in
            Table
```

**Calculated table partition (DAX):**
```
partition PartitionName = calculated
    source =
        UNION(
            ROW("Key", 1, "Value", "A"),
            ROW("Key", 2, "Value", "B")
        )
```

**Key for dependency analysis:** The `source` expression in M partitions references other queries and parameters. In calculated partitions, the DAX expression references tables/columns/measures.

## Relationship Definition

All relationships in `relationships.tmdl`:

```
relationship 5f8a1b2c-...
    fromColumn: Sales.'Product Key'
    toColumn: Product.'Product Key'
    fromCardinality: many
    toCardinality: one
    crossFilteringBehavior: oneDirection
    isActive: true
    lineageTag: 3a4b5c6d-...

relationship 7d8e9f0a-...
    fromColumn: Sales.'Date Key'
    toColumn: Calendar.DateKey
    isActive: false
    crossFilteringBehavior: oneDirection
```

| Property | Values | Default |
|----------|--------|---------|
| `fromColumn` | `Table.Column` or `'Table Name'.'Column Name'` | required |
| `toColumn` | same format | required |
| `fromCardinality` | `one`, `many`, `none` | `many` |
| `toCardinality` | `one`, `many`, `none` | `one` |
| `crossFilteringBehavior` | `oneDirection`, `bothDirections` | `oneDirection` |
| `isActive` | `true`, `false` | `true` (omitted when true) |

**For dependency analysis:** Both `fromColumn` and `toColumn` columns are ALWAYS considered active dependencies, even if no visual or DAX references them.

## model.tmdl

```
model Model
    culture: en-US
    defaultPowerBIDataSourceVersion: powerBI_V3
    lineageTag: 1a2b3c4d-...

    ref table Sales
    ref table Product
    ref table Calendar

    annotation PBI_QueryOrder = ["Sales","Product","Calendar"]
```

The `ref table` declarations define table ordering. Each referenced table has a corresponding file in `tables/`.

## database.tmdl

```
database Sales
    compatibilityLevel: 1567
```

Minimal file — just the database name and compatibility level.

## Named Expressions (expressions.tmdl)

```
expression Server = "myserver.database.windows.net" meta [IsParameterQuery=true, Type="Text"]

expression Database = "ContosoDB" meta [IsParameterQuery=true, Type="Text"]

expression SharedQuery =
    let
        Source = Sql.Database(Server, Database),
        Result = Table.SelectRows(Source, each [Status] = "Active")
    in
        Result
```

**For dependency analysis:** Parameters and shared queries are referenced by table partitions. A partition's M expression may use `Server`, `Database`, or `SharedQuery` as source references.

## Role Definition (roles/)

```
role 'Regional Manager'
    modelPermission: read

    tablePermission Sales = [Region] = USERPRINCIPALNAME()
    tablePermission Product = [IsActive] = TRUE()
```

**For dependency analysis:** The `tablePermission` DAX filter expressions reference columns that must be considered active dependencies (RLS columns).

## Calculation Groups

```
table 'Time Intelligence'
    calculationGroup

    column Name
        dataType: string
        sourceColumn: Name
        sortByColumn: Ordinal

    column Ordinal
        dataType: int64
        sourceColumn: Ordinal
        isHidden

    calculationItem YTD = CALCULATE(SELECTEDMEASURE(), DATESYTD('Calendar'[Date]))
        ordinal: 0

    calculationItem QTD = CALCULATE(SELECTEDMEASURE(), DATESQTD('Calendar'[Date]))
        ordinal: 1
```

**For dependency analysis:** `SELECTEDMEASURE()` means the calculation group implicitly depends on whatever measure is being evaluated — this is a dynamic dependency that cannot be statically resolved.

## Case Sensitivity

- **Keywords** are case-insensitive on read (`TABLE`, `Table`, `table` all work)
- **Serialization** uses camelCase (`table`, `column`, `measure`, `dataType`)
- **Object names** (Entity/Property) are case-sensitive in DAX references — `[Sales Amount]` differs from `[sales amount]`

## Parsing Checklist for Dependency Analysis

1. **Read `model.tmdl`** — get list of all tables from `ref table` declarations
2. **Read each `tables/*.tmdl`** — extract:
   - Table name (from `table` declaration)
   - All measures (name + DAX expression)
   - All columns (name + type: source vs calculated)
   - All partitions (M or DAX source expression)
   - All hierarchies (level columns)
3. **Read `relationships.tmdl`** — extract all from/to column pairs
4. **Read `expressions.tmdl`** — extract parameters and shared M queries
5. **Read `roles/*.tmdl`** — extract DAX filter expressions referencing columns
6. **Parse all DAX expressions** — see `dax-parsing-patterns.md`
7. **Parse all M expressions** — look for query references and column operations
