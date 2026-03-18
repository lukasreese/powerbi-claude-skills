# Sample TMDL Relationship Examples

All relationships are defined in a single file: `relationships.tmdl`. This reference shows annotated examples of every relationship type.

---

## File Structure

```tmdl
// relationships.tmdl contains ALL relationships for the model

relationship <guid-or-id>
    fromColumn: <Table>.<Column>
    toColumn: <Table>.<Column>
    // additional properties...

relationship <guid-or-id>
    fromColumn: <Table>.<Column>
    toColumn: <Table>.<Column>
    // additional properties...
```

---

## Example 1: Active One-to-Many (Most Common)

```tmdl
relationship 5f8a1b2c-1234-5678-9abc-def012345678
	fromColumn: Sales.'Product Key'
	toColumn: Product.'Product Key'
	lineageTag: 3a4b5c6d-1111-2222-3333-444455556666
```

**Properties (defaults when omitted):**
- `fromCardinality: many` (default, omitted)
- `toCardinality: one` (default, omitted)
- `crossFilteringBehavior: oneDirection` (default, omitted)
- `isActive: true` (default, omitted)

**Dependency impact:**
- `Sales[Product Key]` → status `R` (relationship column)
- `Product[Product Key]` → status `R` (relationship column)
- Both columns MUST exist — deleting either breaks the relationship

---

## Example 2: Active One-to-Many with Explicit Properties

```tmdl
relationship 7d8e9f0a-2345-6789-abcd-ef0123456789
	fromColumn: Sales.'Date Key'
	toColumn: Calendar.Date
	fromCardinality: many
	toCardinality: one
	crossFilteringBehavior: oneDirection
	isActive: true
	lineageTag: 8b9c0d1e-2222-3333-4444-555566667777
```

Same as Example 1 but with all properties explicitly stated. Functionally identical.

---

## Example 3: Inactive Relationship

```tmdl
relationship 9a0b1c2d-3456-789a-bcde-f01234567890
	fromColumn: Sales.'Ship Date'
	toColumn: Calendar.Date
	isActive: false
	lineageTag: 0c1d2e3f-3333-4444-5555-666677778888
```

**Key observations:**
- `isActive: false` — this relationship is NOT used by default
- Activated via `USERELATIONSHIP(Sales[Ship Date], Calendar[Date])` in specific measures
- **Dependency impact:** The columns are still `R` status because the relationship exists
- If ANY active measure uses `USERELATIONSHIP` referencing these columns, the relationship and its columns are confirmed active

**How to detect usage:**
1. Scan all DAX expressions for `USERELATIONSHIP`
2. Extract the two column arguments
3. Match against inactive relationships
4. If matched → the relationship is used; columns stay `R`
5. If no USERELATIONSHIP references it → flag as potentially removable (but keep `R` status as a precaution)

---

## Example 4: Many-to-Many Relationship

```tmdl
relationship 1b2c3d4e-4567-89ab-cdef-012345678901
	fromColumn: Sales.'Customer Key'
	toColumn: CustomerSegment.'Customer Key'
	fromCardinality: many
	toCardinality: many
	crossFilteringBehavior: oneDirection
	lineageTag: 2d3e4f5a-4444-5555-6666-777788889999
```

**Key observations:**
- `fromCardinality: many` + `toCardinality: many` = Many-to-Many
- **Quality warning:** M2M relationships have performance implications and can produce unexpected aggregation results
- Flag as `MEDIUM` severity in Model Quality Warnings (Report D)
- Both columns → status `R`

---

## Example 5: Bidirectional Cross-Filter

```tmdl
relationship 3c4d5e6f-5678-9abc-def0-123456789012
	fromColumn: Sales.'Product Key'
	toColumn: Product.'Product Key'
	crossFilteringBehavior: bothDirections
	lineageTag: 4e5f6a7b-5555-6666-7777-888899990000
```

**Key observations:**
- `crossFilteringBehavior: bothDirections` — filter propagates BOTH ways
- **Quality warning:** Bidirectional filtering can cause ambiguous filter paths and performance issues
- Flag as `MEDIUM` severity in Model Quality Warnings
- Both columns → status `R`

---

## Example 6: Multiple Relationships Between Same Tables

```tmdl
// Active: Order Date → Calendar
relationship 5d6e7f8a-6789-abcd-ef01-234567890123
	fromColumn: Sales.'Order Date'
	toColumn: Calendar.Date
	lineageTag: 6f7a8b9c-6666-7777-8888-999900001111

// Inactive: Ship Date → Calendar
relationship 7e8f9a0b-789a-bcde-f012-345678901234
	fromColumn: Sales.'Ship Date'
	toColumn: Calendar.Date
	isActive: false
	lineageTag: 8a9b0c1d-7777-8888-9999-000011112222

// Inactive: Due Date → Calendar
relationship 9f0a1b2c-89ab-cdef-0123-456789012345
	fromColumn: Sales.'Due Date'
	toColumn: Calendar.Date
	isActive: false
	lineageTag: 0b1c2d3e-8888-9999-0000-111122223333
```

**Key observations:**
- Only ONE active relationship between the same two tables is allowed
- Inactive relationships are used via `USERELATIONSHIP` in specific measures
- All 4 columns (`Order Date`, `Ship Date`, `Due Date` in Sales; `Date` in Calendar) → status `R`
- The pattern is common for date role-playing dimensions

---

## Example 7: Relationship with Quoted Table Names

```tmdl
relationship ab1c2d3e-9abc-def0-1234-567890123456
	fromColumn: 'Fact Sales'.'Product ID'
	toColumn: 'Dim Product'.'Product ID'
	lineageTag: bc2d3e4f-9999-0000-1111-222233334444
```

**Parsing note:** Table names with spaces are single-quoted. The column reference format is `'Table Name'.'Column Name'`. Extract:
- From table: `Fact Sales`, from column: `Product ID`
- To table: `Dim Product`, to column: `Product ID`

---

## Parsing Summary

For each relationship in `relationships.tmdl`, extract:

| Property | How to Parse | Default if Omitted |
|----------|-------------|-------------------|
| ID/Name | After `relationship` keyword | (always present) |
| fromColumn | `fromColumn:` value — split on `.` for table.column | required |
| toColumn | `toColumn:` value — split on `.` for table.column | required |
| fromCardinality | `fromCardinality:` value | `many` |
| toCardinality | `toCardinality:` value | `one` |
| crossFilteringBehavior | `crossFilteringBehavior:` value | `oneDirection` |
| isActive | `isActive:` value | `true` |

**Column reference parsing:** The `fromColumn`/`toColumn` values use dot notation:
- Unquoted: `Sales.ProductKey` → table=`Sales`, column=`ProductKey`
- Quoted: `'Fact Sales'.'Product Key'` → table=`Fact Sales`, column=`Product Key`
- Mixed: `Sales.'Product Key'` → table=`Sales`, column=`Product Key`

**Regex for fromColumn/toColumn values:**
```
(?:'((?:[^']|'')*?)'|([A-Za-z_]\w*))\.(?:'((?:[^']|'')*?)'|([A-Za-z_]\w*))
```
- Groups 1/2: Table name (quoted or unquoted)
- Groups 3/4: Column name (quoted or unquoted)

---

## Dependency Analysis Checklist for Relationships

1. **Inventory all relationships** from `relationships.tmdl`
2. **Mark all fromColumn/toColumn** as status `R` (unless they have higher priority status)
3. **Flag quality warnings:**
   - Many-to-Many → `MEDIUM` severity
   - Bidirectional cross-filter → `MEDIUM` severity
   - Inactive relationship → `LOW` severity (informational)
4. **Check for USERELATIONSHIP** in all DAX expressions to confirm inactive relationship usage
5. **Island tables** — tables with NO relationships (neither from nor to) → quality warning `MEDIUM`
