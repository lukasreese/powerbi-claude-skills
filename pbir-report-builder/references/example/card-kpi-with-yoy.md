# KPI Card with YoY Comparison Badge

## What This Template Achieves

A `cardVisual` that shows:
1. **Main value** — the primary measure (e.g. Total Gross Sales)
2. **Reference row** — prior year value with a custom label ("vs. PY:")
3. **Conditional badge** — YoY% with a green/red background driven by DAX measures

This pattern was reverse-engineered from a production Talking Rain report (Assortment Analytics v2, "Total Portfolio Revenue" card on Portfolio Health Brand page, visual folder `de11e25966317e0c37f1`).

---

## Required DAX Measures

| Role | Measure Name | Example DAX |
|------|-------------|-------------|
| Main value | `Sum Gross Sales` | `SUM(Fact_Sales[Gross_Sales])` |
| Prior year value | `Sum Gross Sales PY` | `CALCULATE([Sum Gross Sales], SAMEPERIODLASTYEAR(Dim_Date[Date]))` |
| YoY variance % | `Sum Gross Sales Var %` | `DIVIDE([Sum Gross Sales] - [Sum Gross Sales PY], [Sum Gross Sales PY])` |
| Badge background color | `KPI Color Gross Sales` | `IF([Sum Gross Sales Var %] >= 0, "#44C088", "#ED7373")` |
| Badge font color | `KPI Font Color Gross Sales` | `"#FFFFFF"` (white on both green and red) |

---

## Critical PBIR Rules

1. **Always include `$schema`** as the first property in visual.json
2. **Never use `vcObjects`** — use `visualContainerObjects` instead
3. **Never use `filterConfig`** — not in the visual schema
4. **Never add `"Schema": "extension"` in SourceRef** — use only `"Entity": "TABLE_NAME"`
5. **Always use `nativeQueryRef`** in projections, never `"active": true`
6. **Metadata selectors use `"TABLE.MEASURE"`** format — no `extension.` prefix
7. **Include `dataViewWildcard`** in selectors that set dynamic values

---

## Key Formatting Objects (What Makes It Look Clean)

These objects remove the default card clutter. Without them, you get visible outlines, labels, dividers.

| Object | Purpose | Value |
|--------|---------|-------|
| `outline` | Removes card border | `show: false` |
| `divider` | Removes horizontal divider | `show: false` |
| `fillCustom` | Removes custom fill background | `show: false` |
| `label` | **Hides the measure name label** (e.g. "Sum Gross Sales" above the number) | `show: false`, `matchValueAlignment: true` |
| `layout` | Inner padding | `paddingUniform: 12L` |
| `padding` | Outer padding | `paddingUniform: 0L` |
| `smallMultiplesHeader` | Hides SM header background | `backgroundShow: false` |
| `value` | Display units + precision | `labelDisplayUnits: "0D"`, `labelPrecision: "2L"` |
| `referenceLabelValue` | Reference number precision | `valuePrecision: "2L"` |
| `visualContainerObjects.background` | Transparent card background | `show: false` |

---

## Complete Visual JSON Template

Replace `<<TABLE>>`, `<<MAIN>>`, `<<PY>>`, `<<VAR_PCT>>`, `<<BG_COLOR>>`, `<<FONT_COLOR>>`, `<<TITLE>>`, `<<LABEL>>`, and `<<FIELD_ID>>` with your values.

```json
{
  "$schema": "https://developer.microsoft.com/json-schemas/fabric/item/report/definition/visualContainer/2.7.0/schema.json",
  "name": "v_kpi_card",
  "position": { "x": 30, "y": 80, "z": 1000, "height": 150, "width": 280, "tabOrder": 0 },
  "visual": {
    "visualType": "cardVisual",
    "query": {
      "queryState": {
        "Data": {
          "projections": [
            {
              "field": {
                "Measure": {
                  "Expression": { "SourceRef": { "Entity": "<<TABLE>>" } },
                  "Property": "<<MAIN>>"
                }
              },
              "queryRef": "<<TABLE>>.<<MAIN>>",
              "nativeQueryRef": "<<MAIN>>"
            }
          ]
        }
      },
      "sortDefinition": {
        "sort": [
          {
            "field": {
              "Measure": {
                "Expression": { "SourceRef": { "Entity": "<<TABLE>>" } },
                "Property": "<<MAIN>>"
              }
            },
            "direction": "Descending"
          }
        ],
        "isDefaultSort": true
      }
    },
    "objects": {
      "outline": [
        {
          "properties": {
            "show": { "expr": { "Literal": { "Value": "false" } } }
          },
          "selector": { "id": "default" }
        }
      ],
      "divider": [
        {
          "properties": {
            "show": { "expr": { "Literal": { "Value": "false" } } }
          },
          "selector": { "id": "default" }
        }
      ],
      "fillCustom": [
        {
          "properties": {
            "show": { "expr": { "Literal": { "Value": "false" } } }
          }
        }
      ],
      "label": [
        {
          "properties": {
            "matchValueAlignment": { "expr": { "Literal": { "Value": "true" } } },
            "show": { "expr": { "Literal": { "Value": "false" } } }
          },
          "selector": { "id": "default" }
        }
      ],
      "smallMultiplesHeader": [
        {
          "properties": {
            "backgroundShow": { "expr": { "Literal": { "Value": "false" } } }
          },
          "selector": { "id": "default" }
        }
      ],
      "layout": [
        {
          "properties": {
            "paddingUniform": { "expr": { "Literal": { "Value": "12L" } } }
          },
          "selector": { "id": "default" }
        }
      ],
      "padding": [
        {
          "properties": {
            "paddingUniform": { "expr": { "Literal": { "Value": "0L" } } }
          },
          "selector": { "id": "default" }
        }
      ],
      "referenceLabel": [
        {
          "properties": {
            "backgroundShow": { "expr": { "Literal": { "Value": "false" } } },
            "paddingUniform": { "expr": { "Literal": { "Value": "5L" } } },
            "paddingIndividual": { "expr": { "Literal": { "Value": "true" } } },
            "paddingTop": { "expr": { "Literal": { "Value": "10L" } } }
          },
          "selector": { "id": "default" }
        },
        {
          "properties": {
            "value": {
              "expr": {
                "Measure": {
                  "Expression": { "SourceRef": { "Entity": "<<TABLE>>" } },
                  "Property": "<<PY>>"
                }
              }
            }
          },
          "selector": {
            "data": [{ "dataViewWildcard": { "matchingOption": 0 } }],
            "metadata": "<<TABLE>>.<<MAIN>>",
            "id": "<<FIELD_ID>>",
            "order": 0
          }
        }
      ],
      "referenceLabelDetail": [
        {
          "properties": {
            "show": { "expr": { "Literal": { "Value": "true" } } }
          },
          "selector": {
            "metadata": "<<TABLE>>.<<MAIN>>",
            "id": "<<FIELD_ID>>"
          }
        },
        {
          "properties": {
            "detailValue": {
              "expr": {
                "Measure": {
                  "Expression": { "SourceRef": { "Entity": "<<TABLE>>" } },
                  "Property": "<<VAR_PCT>>"
                }
              }
            },
            "detailFontColor": {
              "solid": {
                "color": {
                  "expr": {
                    "Measure": {
                      "Expression": { "SourceRef": { "Entity": "<<TABLE>>" } },
                      "Property": "<<FONT_COLOR>>"
                    }
                  }
                }
              }
            },
            "detailBackgroundColor": {
              "solid": {
                "color": {
                  "expr": {
                    "Measure": {
                      "Expression": { "SourceRef": { "Entity": "<<TABLE>>" } },
                      "Property": "<<BG_COLOR>>"
                    }
                  }
                }
              }
            }
          },
          "selector": {
            "data": [{ "dataViewWildcard": { "matchingOption": 0 } }],
            "metadata": "<<TABLE>>.<<MAIN>>",
            "id": "<<FIELD_ID>>"
          }
        }
      ],
      "referenceLabelTitle": [
        {
          "properties": {
            "show": { "expr": { "Literal": { "Value": "true" } } },
            "titleContentType": { "expr": { "Literal": { "Value": "'custom'" } } },
            "titleText": { "expr": { "Literal": { "Value": "'<<LABEL>>'" } } }
          },
          "selector": {
            "metadata": "<<TABLE>>.<<MAIN>>",
            "id": "<<FIELD_ID>>"
          }
        }
      ],
      "value": [
        {
          "properties": {
            "labelDisplayUnits": { "expr": { "Literal": { "Value": "0D" } } },
            "labelPrecision": { "expr": { "Literal": { "Value": "2L" } } }
          },
          "selector": {
            "metadata": "<<TABLE>>.<<MAIN>>"
          }
        }
      ],
      "referenceLabelValue": [
        {
          "properties": {
            "valuePrecision": { "expr": { "Literal": { "Value": "2L" } } }
          },
          "selector": {
            "metadata": "<<TABLE>>.<<MAIN>>",
            "id": "<<FIELD_ID>>"
          }
        }
      ]
    },
    "visualContainerObjects": {
      "background": [
        {
          "properties": {
            "show": { "expr": { "Literal": { "Value": "false" } } }
          }
        }
      ],
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
  }
}
```

---

## Selector Metadata Format

The `metadata` value in selectors uses `"TABLE.MEASURE"` format (NO `extension.` prefix):

- `"!Measure.Sum Gross Sales"` (test project)
- `"_Measures.Dollar Sales"` (Talking Rain)

The `<<FIELD_ID>>` can be any unique string (e.g. `"field-gs-ref-001"`). It must be consistent across `referenceLabel`, `referenceLabelDetail`, `referenceLabelTitle`, and `referenceLabelValue`.

---

## Community Contribution

To add a new card variant:
1. Build in Power BI Desktop, export `visual.json` from the PBIP
2. Take a screenshot
3. Submit via GitHub PR to `lukasreese/powerbi-claude-skills`:
   - JSON → `references/json-templates/card-kpi-[variant].json`
   - Screenshot → `references/visual-gallery/images/cardVisual-[variant].png`
