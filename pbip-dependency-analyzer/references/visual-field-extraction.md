# Visual Field Extraction Patterns

This reference documents every JSON path where measure and column references appear in PBIR files (visual.json, page.json, report.json). Use these paths to systematically extract all field dependencies from the report layer.

## Entity/Property Extraction

All field references follow a common structure. Extract the **table name** (Entity) and **field name** (Property):

**Measure reference:**
```json
{
  "field": {
    "Measure": {
      "Expression": { "SourceRef": { "Entity": "TABLE_NAME" } },
      "Property": "MEASURE_NAME"
    }
  }
}
```

**Column reference:**
```json
{
  "field": {
    "Column": {
      "Expression": { "SourceRef": { "Entity": "TABLE_NAME" } },
      "Property": "COLUMN_NAME"
    }
  }
}
```

---

## 1. Query Projections (Primary Data Bindings)

**File:** `visual.json`
**Path:** `visual.query.queryState.<RoleName>.projections[].field`

This is the most common location. Each visual type uses specific role names.

### Role Names by Visual Type

| Visual Type | Roles |
|-------------|-------|
| `cardVisual` | `Data`, `ReferenceLabels`, `AdditionalMeasure` |
| `clusteredColumnChart`, `clusteredBarChart`, `columnChart`, `barChart` | `Category`, `Y`, `Series`, `Tooltips` |
| `lineChart`, `areaChart`, `stackedAreaChart` | `Category`, `Y`, `Y2`, `Series`, `Tooltips` |
| `lineClusteredColumnComboChart`, `lineStackedColumnComboChart` | `Category`, `ColumnY`, `LineY`, `Series`, `Tooltips` |
| `tableEx` | `Values` |
| `pivotTable` (matrix) | `Rows`, `Columns`, `Values`, `Tooltips` |
| `slicer` | `Values` |
| `donutChart`, `pieChart` | `Category`, `Y`, `Tooltips` |
| `gauge` | `Y`, `MinValue`, `MaxValue`, `TargetValue` |
| `waterfallChart` | `Category`, `Y`, `Breakdown`, `Tooltips` |
| `treemap` | `Group`, `Details`, `Values`, `Tooltips` |
| `scatterChart` | `Category`, `X`, `Y`, `Size`, `Series`, `Tooltips` |
| `funnel` | `Category`, `Y`, `Tooltips` |
| `map`, `filledMap` | `Category`, `Size`, `Tooltips`, `Color` (+ geo columns) |

### Field Types in Projections

**Direct Measure:**
```json
{ "field": { "Measure": { "Expression": { "SourceRef": { "Entity": "T" } }, "Property": "M" } } }
```

**Direct Column:**
```json
{ "field": { "Column": { "Expression": { "SourceRef": { "Entity": "T" } }, "Property": "C" } } }
```

**Aggregated Column:**
```json
{
  "field": {
    "Aggregation": {
      "Expression": {
        "Column": { "Expression": { "SourceRef": { "Entity": "T" } }, "Property": "C" }
      },
      "Function": 0
    }
  }
}
```
Aggregation functions: `0`=SUM, `1`=AVG, `2`=COUNT, `3`=MIN, `4`=MAX, `5`=COUNTA, `6`=DISTINCTCOUNT

**Hierarchy:**
```json
{ "field": { "Hierarchy": { "Expression": { "SourceRef": { "Entity": "T" } }, "Property": "H" } } }
```

**Hierarchy Level:**
```json
{
  "field": {
    "HierarchyLevel": {
      "Expression": {
        "Hierarchy": { "Expression": { "SourceRef": { "Entity": "T" } }, "Property": "H" }
      },
      "Level": "LevelName"
    }
  }
}
```

**NativeVisualCalculation (embedded DAX):**
```json
{
  "field": {
    "NativeVisualCalculation": {
      "Language": "dax",
      "Expression": "DAX_CODE_HERE",
      "Name": "CalcName"
    }
  }
}
```
Note: Parse the `Expression` string as DAX using `dax-parsing-patterns.md` to find embedded references.

---

## 2. Conditional Formatting (Dynamic Colors/Properties)

**File:** `visual.json`
**Path:** `visual.objects.<ObjectName>[].properties.<PropertyName>.solid.color.expr`

Common object names: `calloutValue`, `dataPoint`, `labels`, `categoryLabels`, `columnHeaders`, `rowHeaders`, `values`

```json
{
  "visual": {
    "objects": {
      "calloutValue": [{
        "properties": {
          "color": {
            "solid": {
              "color": {
                "expr": {
                  "Measure": {
                    "Expression": { "SourceRef": { "Entity": "T" } },
                    "Property": "ColorMeasure"
                  }
                }
              }
            }
          }
        }
      }]
    }
  }
}
```

**Also check for `fontColor`, `backColor`, `foreColor`** — same structure as `color`.

**CRITICAL:** These measures often do NOT appear in query projections. They are "invisible" dependencies that only show up in formatting config. Always scan the entire `visual.objects` tree for `.expr.Measure` patterns.

---

## 3. Error Bars (Variance Visualization)

**File:** `visual.json`
**Path:** `visual.objects.dataPoint[].errorBar.errorRange.explicit.upperBound` and `.lowerBound`

```json
{
  "errorBar": {
    "enabled": true,
    "errorRange": {
      "kind": "ErrorRange",
      "explicit": {
        "isRelative": true,
        "upperBound": {
          "Measure": {
            "Expression": { "SourceRef": { "Entity": "T" } },
            "Property": "UpperMeasure"
          }
        },
        "lowerBound": {
          "Measure": {
            "Expression": { "SourceRef": { "Entity": "T" } },
            "Property": "LowerMeasure"
          }
        }
      }
    }
  }
}
```

Used in IBCS variance charts to display variance bars as "error bars."

---

## 4. Dynamic Data Labels

**File:** `visual.json`
**Path:** `visual.objects.labels[].dynamicLabelValue`

```json
{
  "labels": [{
    "properties": { "show": { "expr": { "Literal": { "Value": "true" } } } },
    "dynamicLabelValue": {
      "Measure": {
        "Expression": { "SourceRef": { "Entity": "T" } },
        "Property": "LabelMeasure"
      }
    }
  }]
}
```

Binds a measure to display as the data label value instead of the default.

---

## 5. Reference Lines (Dynamic Position)

**File:** `visual.json`
**Path:** `visual.objects.y1AxisReferenceLine[].dynamicMeasure`

```json
{
  "y1AxisReferenceLine": [{
    "properties": {
      "show": { "expr": { "Literal": { "Value": "true" } } },
      "transparency": { "expr": { "Literal": { "Value": "100D" } } }
    },
    "dynamicMeasure": {
      "Measure": {
        "Expression": { "SourceRef": { "Entity": "T" } },
        "Property": "RefLineMeasure"
      }
    }
  }]
}
```

Often used with `transparency: 100D` (invisible) purely for label positioning.

---

## 6. Dynamic Axis Bounds

**File:** `visual.json`
**Path:** `visual.objects.valueAxis[].properties.start.expr` and `.end.expr`

```json
{
  "valueAxis": [{
    "properties": {
      "end": {
        "expr": {
          "Measure": {
            "Expression": { "SourceRef": { "Entity": "T" } },
            "Property": "MaxAxisMeasure"
          }
        }
      }
    }
  }]
}
```

Dynamically sets axis min/max based on a measure value.

---

## 7. SelectRef (Indirect References)

**File:** `visual.json`
**Path:** `visual.objects.<ObjectName>[].properties.<PropertyName>.solid.color.expr.SelectRef`

```json
{
  "color": {
    "solid": {
      "color": {
        "expr": {
          "SelectRef": { "ExpressionName": "select12" }
        }
      }
    }
  }
}
```

SelectRef does NOT directly contain Entity/Property. Instead:
1. Find the `ExpressionName` value (e.g., `"select12"`)
2. Look up the matching `queryRef` in `visual.query.queryState.<Role>.projections[]`
3. The projection with `queryRef: "select12"` contains the actual field reference

---

## 8. Filter Configuration

Filters can appear at three levels, all using the same structure.

### Report-Level Filters
**File:** `report.json`
**Path:** `filterConfig.filters[].field`

### Page-Level Filters
**File:** `page.json`
**Path:** `filterConfig.filters[].field`

### Visual-Level Filters
**File:** `visual.json`
**Path:** `filterConfig.filters[].field`

```json
{
  "filterConfig": {
    "filters": [{
      "name": "FilterName",
      "field": {
        "Column": {
          "Expression": { "SourceRef": { "Entity": "T" } },
          "Property": "C"
        }
      },
      "type": "Categorical",
      "filter": { "Version": 2, "From": [...], "Where": [...] }
    }]
  }
}
```

**Filter types:** Categorical, Range, Advanced, TopN, RelativeDate, RelativeTime, MeasureFilter, Tuple, Include, Exclude

Filters most commonly reference Columns, but MeasureFilter can reference Measures.

---

## 9. Drillthrough Page Bindings

**File:** `page.json`
**Path:** `pageBinding.bindingParameters[].field`

```json
{
  "pageBinding": {
    "type": "Drillthrough",
    "bindingParameters": [{
      "name": "param1",
      "field": {
        "Column": {
          "Expression": { "SourceRef": { "Entity": "T" } },
          "Property": "C"
        }
      }
    }]
  }
}
```

Drillthrough parameters define which columns are passed when drilling through to a page.

---

## 10. Sort Configuration

**File:** `visual.json`
**Path:** `visual.query.queryState.<Role>.projections[].sortOrder` (implicit — sorted by projection order)

Sort-by relationships are typically handled by the `sortByColumn` property in TMDL, not in visual.json. However, visuals can specify sort direction via `SortDirection` in query configurations.

---

## Extraction Algorithm

```
For each PBIR report:

1. READ report.json
   └─ Extract filterConfig.filters[].field → Column/Measure references

2. For each page folder:
   a. READ page.json
      ├─ Extract filterConfig.filters[].field → Column/Measure references
      └─ Extract pageBinding.bindingParameters[].field → Column references

   b. For each visual folder:
      READ visual.json
      ├─ Extract visual.query.queryState.<all roles>.projections[].field
      │   ├─ .Measure → measure ref
      │   ├─ .Column → column ref
      │   ├─ .Aggregation.Expression.Column → column ref
      │   ├─ .Hierarchy → hierarchy ref
      │   ├─ .HierarchyLevel → hierarchy + level ref
      │   └─ .NativeVisualCalculation.Expression → parse as DAX
      │
      ├─ Extract visual.objects (deep scan for .expr.Measure)
      │   ├─ .calloutValue[].properties.color.solid.color.expr.Measure
      │   ├─ .dataPoint[].properties.*.solid.color.expr.Measure
      │   ├─ .dataPoint[].errorBar.errorRange.explicit.upperBound.Measure
      │   ├─ .dataPoint[].errorBar.errorRange.explicit.lowerBound.Measure
      │   ├─ .labels[].dynamicLabelValue.Measure
      │   ├─ .y1AxisReferenceLine[].dynamicMeasure.Measure
      │   ├─ .valueAxis[].properties.start.expr.Measure
      │   ├─ .valueAxis[].properties.end.expr.Measure
      │   └─ Any .expr.SelectRef → resolve via queryRef lookup
      │
      └─ Extract filterConfig.filters[].field → Column/Measure references

3. DEDUPLICATE references (same Entity+Property appearing in multiple visuals)

4. MAP each reference to the semantic model inventory:
   - Entity → table name
   - Property → column or measure name in that table
   - Record: which visual, page, and report uses it
   - Record: context (query projection, filter, conditional formatting, etc.)
```

---

## Key Rules

1. **Case-sensitive matching** — Entity and Property must match the semantic model exactly
2. **CF measures are invisible** — always deep-scan `visual.objects` for `.expr.Measure`; they won't appear in query projections
3. **NativeVisualCalculation** embeds DAX as a string — parse it with DAX patterns to find referenced measures/columns
4. **SelectRef** is an indirect reference — resolve it via the `queryRef` in projections before extracting Entity/Property
5. **Slicer visuals** use the same projection structure as other visuals — a slicer's `Values` role column should be classified as `F` (filter), not `V`
6. **A table is implicitly referenced** whenever any of its columns or measures is referenced
