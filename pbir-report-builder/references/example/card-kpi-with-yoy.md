# KPI Card with YoY Comparison Badge

## What This Template Achieves

A `cardVisual` that shows:
1. **Main value** — the primary measure (e.g. Total Gross Sales)
2. **Reference row** — prior year value with a custom label ("vs. PY:")
3. **Conditional badge** — YoY% with a green/red background driven by DAX measures

This pattern was sourced from a real production report (Talking Rain Assortment Analytics v2).

---

## Required DAX Measures

You need 4 measures (substitute your own names):

| Role | Measure Name | Example DAX |
|------|-------------|-------------|
| Main value | `Sum Gross Sales` | `SUM(Fact_Sales[Gross_Sales])` |
| Prior year value | `Sum Gross Sales PY` | `CALCULATE([Sum Gross Sales], SAMEPERIODLASTYEAR(Dim_Date[Date]))` |
| YoY variance % | `Sum Gross Sales Var %` | `DIVIDE([Sum Gross Sales] - [Sum Gross Sales PY], [Sum Gross Sales PY])` |
| Badge background color | `KPI Color Gross Sales` | `IF([Sum Gross Sales Var %] >= 0, "#44C088", "#ED7373")` |
| Badge font color | `KPI Font Color Gross Sales` | `"#FFFFFF"` (white on both green and red) |

---

## Key JSON Objects

### 1. Query State — bind main + reference measures

```json
"query": {
  "queryState": {
    "Data": {
      "projections": [{
        "field": { "Measure": { "Expression": { "SourceRef": { "Entity": "!Measure" } }, "Property": "Sum Gross Sales" } },
        "queryRef": "!Measure.Sum Gross Sales",
        "active": true
      }]
    },
    "ReferenceLabels": {
      "projections": [{
        "field": { "Measure": { "Expression": { "SourceRef": { "Entity": "!Measure" } }, "Property": "Sum Gross Sales PY" } },
        "queryRef": "!Measure.Sum Gross Sales PY",
        "active": true
      }]
    }
  }
}
```

> **Buckets**: `Data` = main value. `ReferenceLabels` = the comparison row.
> `AdditionalMeasure` = optional extra badge (not used here).

---

### 2. `referenceLabel` — controls the reference row display

Two selectors are required:

```json
"referenceLabel": [
  {
    "properties": {
      "backgroundShow": { "expr": { "Literal": { "Value": "false" } } },
      "paddingTop": { "expr": { "Literal": { "Value": "10L" } } }
    },
    "selector": { "id": "default" }
  },
  {
    "properties": {
      "value": {
        "expr": { "Measure": { "Expression": { "SourceRef": { "Entity": "!Measure" } }, "Property": "Sum Gross Sales PY" } }
      }
    },
    "selector": {
      "metadata": "extension.!Measure.Sum Gross Sales",
      "id": "field-gs-ref-001",
      "order": 0
    }
  }
]
```

> **Note**: The `selector.metadata` value mirrors the main measure's queryRef but with the schema prefix (e.g. `extension.!Measure.Sum Gross Sales`). The `id` field (e.g. `field-gs-ref-001`) is a stable identifier you assign — it links `referenceLabel`, `referenceLabelTitle`, and `referenceLabelDetail` together. The `order: 0` marks it as the first (and usually only) reference label.

---

### 3. `referenceLabelTitle` — custom text above the reference value

```json
"referenceLabelTitle": [
  {
    "properties": {
      "titleContentType": { "expr": { "Literal": { "Value": "'custom'" } } },
      "titleText": { "expr": { "Literal": { "Value": "'vs. PY:'" } } }
    },
    "selector": {
      "metadata": "extension.!Measure.Sum Gross Sales",
      "id": "field-gs-ref-001"
    }
  }
]
```

> Change `titleText` to `'vs. YA:'`, `'vs. Budget:'`, etc.

---

### 4. `referenceLabelDetail` — the conditional YoY% badge

This is the heart of the pattern. Two selectors required:

```json
"referenceLabelDetail": [
  {
    "properties": {
      "show": { "expr": { "Literal": { "Value": "true" } } }
    },
    "selector": {
      "metadata": "extension.!Measure.Sum Gross Sales"
    }
  },
  {
    "properties": {
      "detailValue": {
        "expr": { "Measure": { "Expression": { "SourceRef": { "Entity": "!Measure" } }, "Property": "Sum Gross Sales Var %" } }
      },
      "detailBackgroundColor": {
        "solid": {
          "color": {
            "expr": { "Measure": { "Expression": { "SourceRef": { "Entity": "!Measure" } }, "Property": "KPI Color Gross Sales" } }
          }
        }
      },
      "detailFontColor": {
        "solid": {
          "color": {
            "expr": { "Measure": { "Expression": { "SourceRef": { "Entity": "!Measure" } }, "Property": "KPI Font Color Gross Sales" } }
          }
        }
      }
    },
    "selector": {
      "metadata": "extension.!Measure.Sum Gross Sales",
      "id": "field-gs-ref-001"
    }
  }
]
```

> `detailValue` = the percentage shown in the badge.
> `detailBackgroundColor` + `detailFontColor` = driven by DAX measures — this is what creates the green/red conditional coloring.

---

### 5. `visualContainerObjects.title` — visual title

```json
"visualContainerObjects": {
  "title": [
    {
      "properties": {
        "show": { "expr": { "Literal": { "Value": "true" } } },
        "text": { "expr": { "Literal": { "Value": "'Total Gross Sales'" } } }
      }
    }
  ]
}
```

---

## Complete Visual JSON Template

Replace `<<MAIN_MEASURE>>`, `<<PY_MEASURE>>`, `<<VAR_PCT_MEASURE>>`, `<<BG_COLOR_MEASURE>>`, `<<FONT_COLOR_MEASURE>>`, `<<TABLE>>`, `<<TITLE>>`, `<<LABEL_TEXT>>`, and `<<FIELD_ID>>` with your values.

```json
{
  "$schema": "https://developer.microsoft.com/json-schemas/fabric/item/report/definition/visualContainer/2.7.0/schema.json",
  "name": "v_kpi_card",
  "position": { "x": 30, "y": 100, "z": 1000, "width": 300, "height": 160, "tabOrder": 1 },
  "visual": {
    "visualType": "cardVisual",
    "query": {
      "queryState": {
        "Data": {
          "projections": [{
            "field": { "Measure": { "Expression": { "SourceRef": { "Entity": "<<TABLE>>" } }, "Property": "<<MAIN_MEASURE>>" } },
            "queryRef": "<<TABLE>>.<<MAIN_MEASURE>>",
            "active": true
          }]
        },
        "ReferenceLabels": {
          "projections": [{
            "field": { "Measure": { "Expression": { "SourceRef": { "Entity": "<<TABLE>>" } }, "Property": "<<PY_MEASURE>>" } },
            "queryRef": "<<TABLE>>.<<PY_MEASURE>>",
            "active": true
          }]
        }
      }
    },
    "objects": {
      "referenceLabel": [
        {
          "properties": {
            "backgroundShow": { "expr": { "Literal": { "Value": "false" } } },
            "paddingTop": { "expr": { "Literal": { "Value": "10L" } } }
          },
          "selector": { "id": "default" }
        },
        {
          "properties": {
            "value": {
              "expr": { "Measure": { "Expression": { "SourceRef": { "Entity": "<<TABLE>>" } }, "Property": "<<PY_MEASURE>>" } }
            }
          },
          "selector": {
            "metadata": "extension.<<TABLE>>.<<MAIN_MEASURE>>",
            "id": "<<FIELD_ID>>",
            "order": 0
          }
        }
      ],
      "referenceLabelTitle": [
        {
          "properties": {
            "titleContentType": { "expr": { "Literal": { "Value": "'custom'" } } },
            "titleText": { "expr": { "Literal": { "Value": "'<<LABEL_TEXT>>'" } } }
          },
          "selector": {
            "metadata": "extension.<<TABLE>>.<<MAIN_MEASURE>>",
            "id": "<<FIELD_ID>>"
          }
        }
      ],
      "referenceLabelDetail": [
        {
          "properties": {
            "show": { "expr": { "Literal": { "Value": "true" } } }
          },
          "selector": {
            "metadata": "extension.<<TABLE>>.<<MAIN_MEASURE>>"
          }
        },
        {
          "properties": {
            "detailValue": {
              "expr": { "Measure": { "Expression": { "SourceRef": { "Entity": "<<TABLE>>" } }, "Property": "<<VAR_PCT_MEASURE>>" } }
            },
            "detailBackgroundColor": {
              "solid": { "color": { "expr": { "Measure": { "Expression": { "SourceRef": { "Entity": "<<TABLE>>" } }, "Property": "<<BG_COLOR_MEASURE>>" } } } }
            },
            "detailFontColor": {
              "solid": { "color": { "expr": { "Measure": { "Expression": { "SourceRef": { "Entity": "<<TABLE>>" } }, "Property": "<<FONT_COLOR_MEASURE>>" } } } }
            }
          },
          "selector": {
            "metadata": "extension.<<TABLE>>.<<MAIN_MEASURE>>",
            "id": "<<FIELD_ID>>"
          }
        }
      ],
      "value": [
        { "properties": { "labelPrecision": { "expr": { "Literal": { "Value": "1L" } } } } }
      ]
    },
    "visualContainerObjects": {
      "title": [
        {
          "properties": {
            "show": { "expr": { "Literal": { "Value": "true" } } },
            "text": { "expr": { "Literal": { "Value": "'<<TITLE>>'" } } }
          }
        }
      ]
    },
    "drillFilterOtherVisuals": true
  },
  "filterConfig": { "filters": [] }
}
```

---

## Selector Metadata Format

The `metadata` value in selectors must follow this exact format:

```
"extension.<<TABLE>>.<<MEASURE_NAME>>"
```

Examples:
- `"extension.!Measure.Sum Gross Sales"`
- `"extension._Measures.Product Dollar Sales"` (Talking Rain pattern)

If Power BI Desktop modifies your JSON, it may rewrite the `id` values. The field ID (`<<FIELD_ID>>`) can be any unique string — e.g. `"field-gs-ref-001"` — as long as it's consistent across `referenceLabel`, `referenceLabelTitle`, and `referenceLabelDetail`.

---

## Community Contribution

To add a new card variant to the skill:
1. Build your visual in Power BI Desktop or via JSON
2. Export the `visual.json` from the PBIP project (`definition/pages/.../visuals/.../visual.json`)
3. Take a screenshot of the rendered visual
4. Submit via GitHub PR to `lukasreese/powerbi-claude-skills`:
   - JSON → `references/json-templates/card-kpi-[variant].json`
   - Screenshot → `references/visual-gallery/images/cardVisual-[variant].png`
