# DAX Parsing Patterns for Dependency Analysis

This reference provides regex patterns and strategies for extracting all object references from DAX expressions found in measures, calculated columns, calculated tables, RLS filters, and KPI definitions.

## Pre-Processing: Strip Non-Code Content

Before applying reference patterns, remove content that should NOT be parsed:

### 1. Remove Comments
```
// Single-line comment (everything to end of line)
Pattern: //[^\n]*

/* Multi-line comment */
Pattern: /\*[\s\S]*?\*/
```

### 2. Skip String Literals
DAX strings are enclosed in double quotes. Bracket characters inside strings must NOT be parsed as references.

```
"This is a string with [brackets] that are NOT references"
Pattern: "(?:[^"]|"")*"
```

**Strategy:** Replace all string literals with a placeholder (e.g., `"__STR__"`) before scanning for references. DAX escapes quotes by doubling: `"He said ""hello"""`.

---

## Reference Patterns

### Pattern 1: Measure Reference (Bare Brackets)

```
[MeasureName]
```

**Regex:** `\[([^\]]+)\]`

**Context disambiguation:** A bare `[Name]` is a measure reference ONLY when it is NOT preceded by a table name or single-quoted table name. If preceded by `TableName` or `'Table Name'`, it's a column reference (see Pattern 2).

**Resolution strategy:**
1. Extract all `[Name]` tokens
2. Check if preceded by an identifier or closing quote — if yes, it's a column ref
3. If standalone, look up `Name` across all measures in the model
4. If multiple measures share the same name (different tables), the reference is ambiguous — flag for investigation

### Pattern 2: Column Reference (Table[Column])

**Unquoted table:**
```
TableName[ColumnName]
```
**Regex:** `([A-Za-z_]\w*)\s*\[([^\]]+)\]`

**Quoted table:**
```
'Table Name'[ColumnName]
```
**Regex:** `'((?:[^']|'')*?)'\s*\[([^\]]+)\]`

**Combined regex (both forms):**
```
(?:'((?:[^']|'')*?)'|([A-Za-z_]\w*))\s*\[([^\]]+)\]
```
- Group 1: Quoted table name (unescape `''` → `'`)
- Group 2: Unquoted table name
- Group 3: Column or measure name

**Note:** This pattern also matches `Table[Measure]` — the fully qualified measure reference. After extraction, check whether the name resolves to a column or measure in the given table.

### Pattern 3: Table Reference (Function Arguments)

Many DAX functions take a table name as an argument:

| Function | Pattern | Extracts |
|----------|---------|----------|
| `FILTER(Table, ...)` | `FILTER\s*\(\s*(?:'((?:[^']|'')*?)'|([A-Za-z_]\w*))` | Table name |
| `ALL(Table)` | `ALL\s*\(\s*(?:'((?:[^']|'')*?)'|([A-Za-z_]\w*))` | Table name |
| `ALLEXCEPT(Table, ...)` | `ALLEXCEPT\s*\(\s*(?:'((?:[^']|'')*?)'|([A-Za-z_]\w*))` | Table name |
| `VALUES(Table[Col])` | `VALUES\s*\(\s*` then Pattern 2 | Table + Column |
| `DISTINCT(Table[Col])` | `DISTINCT\s*\(\s*` then Pattern 2 | Table + Column |
| `RELATEDTABLE(Table)` | `RELATEDTABLE\s*\(\s*(?:'((?:[^']|'')*?)'|([A-Za-z_]\w*))` | Table (+ implies relationship) |
| `RELATED(Table[Col])` | `RELATED\s*\(\s*` then Pattern 2 | Column (+ implies relationship) |
| `SELECTEDVALUE(Table[Col])` | `SELECTEDVALUE\s*\(\s*` then Pattern 2 | Table + Column |
| `HASONEVALUE(Table[Col])` | `HASONEVALUE\s*\(\s*` then Pattern 2 | Table + Column |
| `COUNTROWS(Table)` | `COUNTROWS\s*\(\s*(?:'((?:[^']|'')*?)'|([A-Za-z_]\w*))` | Table name |
| `SUMMARIZE(Table, ...)` | `SUMMARIZE\s*\(\s*(?:'((?:[^']|'')*?)'|([A-Za-z_]\w*))` | Table + subsequent columns |
| `SUMMARIZECOLUMNS(...)` | All column args | Multiple columns |
| `ADDCOLUMNS(Table, ...)` | `ADDCOLUMNS\s*\(\s*` | Table + expression columns |
| `CALCULATETABLE(Table, ...)` | First arg is table expression | Table |
| `LOOKUPVALUE(...)` | Multiple column args | Columns from multiple tables |
| `TREATAS(expr, Col1, Col2)` | Column args after first | Columns (virtual relationship) |

### Pattern 4: CALCULATE Filter Arguments

```dax
CALCULATE([Measure], Table[Column] = "value")
CALCULATE([Measure], KEEPFILTERS(Table[Column] = "value"))
CALCULATE([Measure], REMOVEFILTERS(Table[Column]))
```

After `CALCULATE(`, the first argument is an expression (contains measure refs). Subsequent arguments are filter expressions containing column references.

**Strategy:** Parse the entire CALCULATE expression, extract:
- First argument → measure/expression references
- Remaining arguments → column references from filter conditions

### Pattern 5: USERELATIONSHIP

```dax
USERELATIONSHIP(Table1[Col1], Table2[Col2])
```

**Regex:** `USERELATIONSHIP\s*\(`  then extract two column references using Pattern 2.

**Dependency impact:** This activates an inactive relationship. Both columns AND the relationship itself become active dependencies within this measure's context.

### Pattern 6: TREATAS (Virtual Relationship)

```dax
TREATAS(VALUES(Table1[Col1]), Table2[Col2])
```

**Dependency impact:** Creates a virtual relationship between the columns. Both columns are dependencies, and any table connected via TREATAS should be treated as having an implicit relationship.

### Pattern 7: VAR Declarations

```dax
VAR myVariable = [SomeMeasure]
VAR tableVar = FILTER('Sales', [Amount] > 0)
RETURN
    myVariable + [AnotherMeasure]
```

**Strategy:** VAR names become local identifiers. Do NOT confuse `myVariable` with a table or measure name. Parse the right side of `=` for actual references.

**Regex for VAR:** `VAR\s+([A-Za-z_]\w*)\s*=`
- Group 1 is the variable name — add to a "skip list" so it's not treated as a table reference

### Pattern 8: SWITCH / IF with Nested References

```dax
SWITCH(
    TRUE(),
    [Measure1] > 100, "High",
    [Measure1] > 50, "Medium",
    "Low"
)
```

**Strategy:** Recursively parse the entire expression. All `[BracketedNames]` and `Table[Column]` patterns within the SWITCH/IF body are valid references.

### Pattern 9: SELECTEDMEASURE() (Calculation Groups)

```dax
CALCULATE(SELECTEDMEASURE(), DATESYTD('Calendar'[Date]))
```

`SELECTEDMEASURE()` is a dynamic reference — it refers to whatever measure is being evaluated at runtime. This creates an implicit dependency on ALL measures that could be affected by the calculation group. Flag this as a **dynamic dependency** that cannot be fully resolved statically.

---

## Edge Cases

### 1. Same Measure Name in Different Tables

If `[Sales]` exists in both `Measures` table and `Revenue` table, a bare `[Sales]` reference is ambiguous. DAX resolves this based on context, but for static analysis:
- Flag as ambiguous
- Check if the measure's parent table has a default — usually the first-defined wins
- Prefer the table with the same name as the containing measure's table

### 2. Column vs Measure Ambiguity in Table[Name]

`Sales[Amount]` could be a column or a measure in the Sales table. After extraction:
- Look up `Amount` in the `Sales` table's columns list
- Look up `Amount` in the `Sales` table's measures list
- If both exist (unusual), column takes precedence in row context, measure in aggregation context

### 3. Aggregation Functions Wrapping Columns

```dax
SUM(Sales[Amount])
AVERAGE('Product'[Price])
COUNTA(Customer[Name])
```

The column inside the aggregation function is a column dependency, not a measure.

### 4. Table Constructor Syntax

```dax
{ "Red", "Blue", "Green" }
```

Curly braces create an anonymous table. Content is literal values, not references.

### 5. BLANK() and Other Constants

`BLANK()`, `TRUE()`, `FALSE()`, `NOW()`, `TODAY()` — these are functions with no object dependencies.

### 6. Row Context in Calculated Columns

In a calculated column, bare `[ColumnName]` refers to a column in the SAME table (row context), not a measure. This is a critical distinction:
- **Measure context:** `[Name]` → measure reference
- **Calculated column context:** `[Name]` → column in the same table (row context), OR a measure if the name matches a measure

**Strategy:** When parsing a calculated column expression, first check same-table columns before looking up measures.

### 7. EARLIER / EARLIEST

```dax
EARLIER([Column])
EARLIEST([Column])
```

References to the current row's column value in nested row context. The `[Column]` is a column in the same table.

---

## Parsing Algorithm

```
For each DAX expression:

1. STRIP comments and string literals (replace with placeholders)

2. EXTRACT all bracket references: [Name]
   - If preceded by table identifier → column ref (Pattern 2)
   - If standalone → measure ref (Pattern 1)
   - If in calculated column context → check same-table columns first

3. EXTRACT all function-argument table refs (Pattern 3)
   - FILTER, ALL, VALUES, RELATED, RELATEDTABLE, etc.

4. DETECT special patterns:
   - USERELATIONSHIP → activate relationship (Pattern 5)
   - TREATAS → virtual relationship (Pattern 6)
   - SELECTEDMEASURE → dynamic dependency (Pattern 9)

5. TRACK VAR declarations to avoid false positives (Pattern 7)

6. BUILD dependency list:
   - Tables referenced (directly or via columns)
   - Columns referenced
   - Measures referenced
   - Relationships activated (USERELATIONSHIP)
   - Virtual relationships (TREATAS)
   - Dynamic/unresolvable (SELECTEDMEASURE)
```

## Common DAX Functions That Create Dependencies

| Function | Dependencies Created |
|----------|---------------------|
| `SUM`, `AVERAGE`, `MIN`, `MAX`, `COUNT`, `COUNTA` | Column |
| `SUMX`, `AVERAGEX`, `MINX`, `MAXX`, `COUNTX` | Table + expression |
| `CALCULATE`, `CALCULATETABLE` | Measure/expression + filter columns |
| `FILTER`, `ALL`, `ALLEXCEPT`, `ALLSELECTED` | Table + filter columns |
| `VALUES`, `DISTINCT`, `SELECTEDVALUE`, `HASONEVALUE` | Table + column |
| `RELATED`, `RELATEDTABLE` | Column/table + relationship |
| `USERELATIONSHIP` | Two columns + relationship activation |
| `TREATAS` | Columns + virtual relationship |
| `LOOKUPVALUE` | Multiple columns from multiple tables |
| `SUMMARIZE`, `SUMMARIZECOLUMNS`, `ADDCOLUMNS` | Table + columns |
| `SWITCH`, `IF` | All references within branches |
| `DIVIDE` | Two expression arguments |
| `FORMAT` | Expression argument |
| `SELECTEDMEASURE` | Dynamic (all measures potentially) |
| `RANKX` | Table + expression |
| `TOPN` | Table + expression |
| `EARLIER`, `EARLIEST` | Same-table column |
| `ISBLANK`, `ISEMPTY` | Expression/table argument |
| `SAMEPERIODLASTYEAR`, `DATEADD`, `DATESYTD` | Date column |
| `CROSSJOIN`, `UNION`, `INTERSECT`, `EXCEPT` | Table arguments |
