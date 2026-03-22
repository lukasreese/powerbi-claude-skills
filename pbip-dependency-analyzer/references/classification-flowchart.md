# Object Classification Decision Tree

This reference defines how to assign exactly one status code to each object (measure, column, table) in the semantic model. Follow the steps in order — the first match wins.

## Priority Order

```
V > F > CF > D > R > M > BROKEN > CIRC > O-CHAIN > O
```

If an object matches multiple statuses, use the highest-priority one.

---

## Decision Tree

### Step 1: Check for Visual Usage → `V` (Used in Visual)

**Question:** Is this object directly bound to at least one visual's query projections?

**Where to check:**
- `visual.query.queryState.<Role>.projections[].field` — any Measure or Column reference
- Roles: `Data`, `Category`, `Y`, `Y2`, `ColumnY`, `LineY`, `Series`, `Rows`, `Columns`, `Values`, `Tooltips`, `Group`, `Details`, `X`, `Size`, `MinValue`, `MaxValue`, `TargetValue`
- Also check: `NativeVisualCalculation` expressions for embedded DAX references

**Evidence required:** The object's `Entity` + `Property` appears in at least one projection's field reference.

**Result:** Status = `V` — this object is actively used. Stop.

---

### Step 2: Check for Filter Usage → `F` (Used in Filter)

**Question:** Is this object used only in filters or slicers (not in visual data bindings)?

**Where to check:**
- Report-level: `report.json` → `filterConfig.filters[].field`
- Page-level: `page.json` → `filterConfig.filters[].field`
- Visual-level: `visual.json` → `filterConfig.filters[].field`
- Slicer visuals: Check if a `slicer` visual's `Values` role uses this field
- Drillthrough fields: `page.json` → `pageBinding.bindingParameters[].field`

**Evidence required:** The object appears in filter config or slicer bindings but NOT in any non-slicer visual's query projections.

**Result:** Status = `F` — used for filtering. Stop.

---

### Step 3: Check for Conditional Formatting Usage → `CF` (Used in CF)

**Question:** Is this object used only for conditional formatting (not in visual data or filters)?

**Where to check:**
- `visual.objects.<any>[].properties.color.solid.color.expr.Measure` — dynamic color
- `visual.objects.<any>[].properties.*.expr.Measure` — any dynamic formatting property
- `visual.objects.dataPoint[].errorBar.errorRange.explicit.*.Measure` — error bar bounds
- `visual.objects.labels[].dynamicLabelValue.Measure` — dynamic label values
- `visual.objects.y1AxisReferenceLine[].dynamicMeasure.Measure` — reference lines
- `visual.objects.valueAxis[].properties.start.expr.Measure` — dynamic axis bounds
- `visual.objects.valueAxis[].properties.end.expr.Measure` — dynamic axis bounds

**Evidence required:** The object appears in visual formatting/objects but NOT in query projections (Step 1) or filters (Step 2).

**Result:** Status = `CF` — conditional formatting dependency. Stop.

---

### Step 4: Check for DAX Reference → `D` (Referenced by DAX)

**Question:** Is this object referenced by another object that IS active (status V, F, CF, or D)?

**How to check:**
1. Gather all objects already classified as V, F, or CF
2. Parse their DAX expressions (see `dax-parsing-patterns.md`)
3. For each V/F/CF object, find all measures, columns, and tables it references
4. Mark those referenced objects as `D`
5. **Repeat transitively:** If object A (status D) references object B, then B is also D
6. Continue until no new objects are marked

**Evidence required:** There exists a chain of DAX references from a V/F/CF object to this object.

**Important:** The chain must originate from an active object (V, F, CF). If the chain only originates from orphaned objects, see Step 9 (O-CHAIN).

**Result:** Status = `D` — indirectly active via DAX. Stop.

---

### Step 5: Check for Relationship Usage → `R` (Relationship Column)

**Question:** Is this column part of an active relationship?

**Where to check:**
- `relationships.tmdl` — all `fromColumn` and `toColumn` values
- Include both active AND inactive relationships (inactive ones may be activated by USERELATIONSHIP in an active measure)

**Evidence required:** The column appears as `fromColumn` or `toColumn` in any relationship definition.

**Scope:** This applies to COLUMNS only, not measures.

**Result:** Status = `R` — relationship infrastructure. Stop.

**Note:** If a column is both in a relationship AND in a visual, it gets `V` (higher priority). The `R` status only applies when the relationship is the ONLY reason the column exists.

---

### Step 6: Check for M/Power Query Dependency → `M`

**Question:** Is this object required by M/Power Query logic?

**Where to check:**
- Partition `source` expressions (M language)
- Named expressions in `expressions.tmdl`
- Column referenced in `Table.NestedJoin`, `Table.SelectColumns`, or similar M operations
- Query used as a source by another query

**Evidence required:** The object appears in an M expression that feeds an active table.

**Scope:** Primarily columns used as join keys in M, or intermediate queries referenced by final queries.

**Result:** Status = `M` — M/Power Query infrastructure. Stop.

---

### Step 7: Check for Broken References → `BROKEN`

**Question:** Does this object's DAX expression reference an object that does not exist in the model?

**How to check:**
1. Parse the DAX expression for all references
2. Look up each referenced table, column, or measure in the model inventory
3. If any reference cannot be resolved → BROKEN

**Evidence required:** At least one unresolvable reference in the expression.

**Result:** Status = `BROKEN` — fix or remove. Stop.

**Note:** An object can be both BROKEN and used in a visual. In that case, mark as `V` (higher priority) and add a quality warning about the broken reference.

---

### Step 8: Check for Circular Dependencies → `CIRC`

**Question:** Is this object part of a circular reference chain?

**How to check:**
1. Build a directed graph of DAX references
2. Detect cycles using DFS or topological sort
3. Any object in a cycle → CIRC

**Evidence required:** Object A references B, B references C, C references A (or A references A).

**Result:** Status = `CIRC` — circular dependency. Stop.

**Note:** Circular references in DAX are errors — Power BI won't evaluate them. These should always be flagged.

---

### Step 9: Check for Orphan Chain → `O-CHAIN`

**Question:** Is this object referenced ONLY by objects that are themselves orphaned?

**How to check:**
1. This object IS referenced by at least one other object's DAX
2. BUT none of those referencing objects are active (V, F, CF, D, R, M)
3. All referencing objects are themselves O or O-CHAIN

**Evidence required:** Has inbound references, but all from orphaned objects.

**Result:** Status = `O-CHAIN` — part of an orphaned dependency chain. The entire chain can be deleted together.

---

### Step 10: Default → `O` (Orphaned)

**Question:** None of the above conditions apply.

**Evidence:** No references found anywhere:
- Not in any visual query
- Not in any filter
- Not in any conditional formatting
- Not referenced by any DAX expression
- Not in any relationship
- Not required by M/Power Query
- No broken refs or circular deps
- Not referenced by any other object at all

**Result:** Status = `O` — orphaned, safe to investigate for deletion.

---

## Risk Assessment (for O and O-CHAIN objects)

After classification, assign a risk level:

| Risk | Criteria | Action |
|------|----------|--------|
| **LOW** | Status O, `isHidden: false`, no special flags | Safe to delete |
| **MEDIUM** | Status O-CHAIN, entire chain is orphaned | Delete the whole chain together |
| **HIGH** | Status O but `isHidden: true` | Investigate — may be hidden for external use (API, Live Connection reports, embedded scenarios) |
| **HIGH** | Status O but has `dataCategory: ImageUrl` | May be used in table/matrix as image — hard to detect statically |
| **HIGH** | Status O but in a calculation group | May affect other measures dynamically |

---

## Special Cases

### Hidden Objects
`isHidden: true` objects deserve extra scrutiny. They might be:
- **Intentionally hidden helper measures** — used by other DAX (should be classified D, not O)
- **Hidden for external consumers** — Live Connection reports, APIs, Excel connections
- **Leftover from development** — genuinely orphaned

If hidden AND orphaned → risk = HIGH, recommend investigation before deletion.

### Calculated Table Columns
If a calculated table is used (status V or D), ALL its columns inherit that status. A calculated table's columns are inseparable from the table — you can't delete individual columns without modifying the DAX expression.

### Sort-By Column Dependencies
If `Column A` has `sortByColumn: Column B`, then Column B is a dependency of Column A. If Column A is active (V/F/CF/D), then Column B should be at least `D`.

### Hierarchy Level Columns
If a hierarchy is used in a visual, all columns referenced by its levels are dependencies (status `D` at minimum).

---

## Flowchart Summary

```
START
  │
  ├─ In visual query projections? ──── YES → V
  │
  ├─ In filter/slicer/drillthrough? ── YES → F
  │
  ├─ In conditional formatting? ─────── YES → CF
  │
  ├─ Referenced by active DAX chain? ── YES → D
  │
  ├─ Column in a relationship? ──────── YES → R
  │
  ├─ Required by M/Power Query? ─────── YES → M
  │
  ├─ Has broken DAX reference? ──────── YES → BROKEN
  │
  ├─ In circular DAX chain? ─────────── YES → CIRC
  │
  ├─ Referenced only by orphans? ─────── YES → O-CHAIN
  │
  └─ None of the above ──────────────── O (Orphaned)
```
