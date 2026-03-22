# Field References in PBIR

Every visual that shows data needs field references that point to columns or measures in the semantic model. This document shows the exact JSON patterns.

## Column Reference

Use when binding a column from a table (for axes, categories, slicers):

```json
{
  "field": {
    "Column": {
      "Expression": {
        "SourceRef": { "Entity": "TableName" }
      },
      "Property": "ColumnName"
    }
  },
  "queryRef": "TableName.ColumnName",
  "nativeQueryRef": "ColumnName"
}
```

**Example** — Product category on a bar chart axis:
```json
{
  "field": {
    "Column": {
      "Expression": {
        "SourceRef": { "Entity": "DimProduct" }
      },
      "Property": "Category"
    }
  },
  "queryRef": "DimProduct.Category",
  "nativeQueryRef": "Category"
}
```

## Measure Reference

Use when binding a DAX measure (for values, KPIs, totals):

```json
{
  "field": {
    "Measure": {
      "Expression": {
        "SourceRef": { "Entity": "TableName" }
      },
      "Property": "MeasureName"
    }
  },
  "queryRef": "TableName.MeasureName",
  "nativeQueryRef": "MeasureName"
}
```

**Example** — Total Sales measure in a card:
```json
{
  "field": {
    "Measure": {
      "Expression": {
        "SourceRef": { "Entity": "_Measures" }
      },
      "Property": "Total Sales"
    }
  },
  "queryRef": "_Measures.Total Sales",
  "nativeQueryRef": "Total Sales"
}
```

## Aggregated Column Reference

Use when you want Power BI to aggregate a column (SUM, AVG, COUNT, etc.) instead of using a DAX measure:

```json
{
  "field": {
    "Aggregation": {
      "Expression": {
        "Column": {
          "Expression": {
            "SourceRef": { "Entity": "TableName" }
          },
          "Property": "ColumnName"
        }
      },
      "Function": 0
    }
  },
  "queryRef": "Sum(TableName.ColumnName)",
  "nativeQueryRef": "Sum of ColumnName"
}
```

### Aggregation Function Codes
| Code | Function |
|---|---|
| `0` | SUM |
| `1` | AVG |
| `2` | COUNT |
| `3` | MIN |
| `4` | MAX |
| `5` | COUNTNONBLANK (Count Distinct) |
| `6` | None (Don't Summarize) |

## Conditional Formatting with Measures

Use a measure to dynamically set a visual property (like color):

```json
"color": {
  "solid": {
    "color": {
      "expr": {
        "Measure": {
          "Expression": {
            "SourceRef": { "Entity": "_Measures" }
          },
          "Property": "KPI Color"
        }
      }
    }
  }
}
```

This is how KPI cards get green/red coloring based on performance.

## Literal Values

For static values in formatting objects:

```json
// Boolean
{ "expr": { "Literal": { "Value": "true" } } }

// String
{ "expr": { "Literal": { "Value": "'FitToPage'" } } }

// Number (append D for decimal)
{ "expr": { "Literal": { "Value": "0D" } } }

// Integer (append L for long)
{ "expr": { "Literal": { "Value": "12L" } } }

// Color
{ "expr": { "Literal": { "Value": "'#FF0000'" } } }
```

## Complete Query Example

A clustered column chart with Category axis and two measures (CY vs PY):

```json
"query": {
  "queryState": {
    "Category": {
      "projections": [
        {
          "field": {
            "Column": {
              "Expression": { "SourceRef": { "Entity": "DimProduct" } },
              "Property": "Category"
            }
          },
          "queryRef": "DimProduct.Category",
          "nativeQueryRef": "Category"
        }
      ]
    },
    "Y": {
      "projections": [
        {
          "field": {
            "Measure": {
              "Expression": { "SourceRef": { "Entity": "_Measures" } },
              "Property": "Total Sales"
            }
          },
          "queryRef": "_Measures.Total Sales",
          "nativeQueryRef": "Total Sales"
        },
        {
          "field": {
            "Measure": {
              "Expression": { "SourceRef": { "Entity": "_Measures" } },
              "Property": "Total Sales PY"
            }
          },
          "queryRef": "_Measures.Total Sales PY",
          "nativeQueryRef": "Total Sales PY"
        }
      ]
    }
  }
}
```

## Key Rules

1. **Entity** must match a table name in the semantic model exactly (case-sensitive)
2. **Property** must match a column or measure name exactly (case-sensitive)
3. **queryRef** format: `TableName.FieldName` (dot-separated)
4. **nativeQueryRef**: just the field name (used for display)
5. Multiple fields in one role = multiple projections in the array
6. The `active: true` flag on the first projection indicates it's the primary field
