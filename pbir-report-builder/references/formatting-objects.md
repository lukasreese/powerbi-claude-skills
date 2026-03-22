# Formatting Objects in PBIR

Visual formatting in PBIR is controlled through `objects` properties at the visual level and `visualContainerObjects` at the container level. This doc covers the most common patterns.

## Container-Level Formatting (visualContainerObjects)

These go in the `visual.json` at the top level alongside `position` and `visual`:

### Title
```json
"visualContainerObjects": {
  "title": [{
    "properties": {
      "show": { "expr": { "Literal": { "Value": "true" } } },
      "text": { "expr": { "Literal": { "Value": "'Sales Overview'" } } },
      "fontColor": { "solid": { "color": { "expr": { "Literal": { "Value": "'#333333'" } } } } },
      "fontSize": { "expr": { "Literal": { "Value": "12D" } } },
      "fontFamily": { "expr": { "Literal": { "Value": "'Segoe UI'" } } },
      "bold": { "expr": { "Literal": { "Value": "true" } } },
      "alignment": { "expr": { "Literal": { "Value": "'left'" } } }
    }
  }]
}
```

### Background
```json
"visualContainerObjects": {
  "background": [{
    "properties": {
      "show": { "expr": { "Literal": { "Value": "true" } } },
      "color": { "solid": { "color": { "expr": { "Literal": { "Value": "'#FFFFFF'" } } } } },
      "transparency": { "expr": { "Literal": { "Value": "0D" } } }
    }
  }]
}
```

### Border
```json
"visualContainerObjects": {
  "border": [{
    "properties": {
      "show": { "expr": { "Literal": { "Value": "true" } } },
      "color": { "solid": { "color": { "expr": { "Literal": { "Value": "'#E0E0E0'" } } } } },
      "radius": { "expr": { "Literal": { "Value": "8D" } } },
      "width": { "expr": { "Literal": { "Value": "1D" } } }
    }
  }]
}
```

### Shadow
```json
"visualContainerObjects": {
  "dropShadow": [{
    "properties": {
      "show": { "expr": { "Literal": { "Value": "true" } } },
      "color": { "solid": { "color": { "expr": { "Literal": { "Value": "'#000000'" } } } } },
      "transparency": { "expr": { "Literal": { "Value": "80D" } } },
      "position": { "expr": { "Literal": { "Value": "'Outer'" } } },
      "preset": { "expr": { "Literal": { "Value": "'BottomRight'" } } }
    }
  }]
}
```

### Padding
```json
"visualContainerObjects": {
  "padding": [{
    "properties": {
      "top": { "expr": { "Literal": { "Value": "8D" } } },
      "bottom": { "expr": { "Literal": { "Value": "8D" } } },
      "left": { "expr": { "Literal": { "Value": "12D" } } },
      "right": { "expr": { "Literal": { "Value": "12D" } } }
    }
  }]
}
```

## Visual-Level Formatting (objects inside visual)

These go inside the `visual` object, alongside `visualType` and `query`:

### Card Visual — Callout Value
```json
"objects": {
  "calloutValue": [{
    "properties": {
      "color": { "solid": { "color": { "expr": { "Literal": { "Value": "'#333333'" } } } } },
      "fontSize": { "expr": { "Literal": { "Value": "28D" } } },
      "fontFamily": { "expr": { "Literal": { "Value": "'DIN'" } } }
    }
  }],
  "cards": [{
    "properties": {
      "showLabel": { "expr": { "Literal": { "Value": "true" } } }
    }
  }],
  "referenceLabel": [{
    "properties": {
      "show": { "expr": { "Literal": { "Value": "true" } } },
      "color": { "solid": { "color": { "expr": { "Literal": { "Value": "'#999999'" } } } } },
      "fontSize": { "expr": { "Literal": { "Value": "10D" } } }
    }
  }]
}
```

### Card Visual — Conditional Color from Measure
```json
"objects": {
  "calloutValue": [{
    "properties": {
      "color": {
        "solid": {
          "color": {
            "expr": {
              "Measure": {
                "Expression": { "SourceRef": { "Entity": "_Measures" } },
                "Property": "KPI Color"
              }
            }
          }
        }
      }
    }
  }]
}
```

### Chart — Data Colors
```json
"objects": {
  "dataPoint": [
    {
      "properties": {
        "fill": { "solid": { "color": { "expr": { "Literal": { "Value": "'#4472C4'" } } } } }
      },
      "selector": { "data": [{ "dataViewWildcard": { "matchingOption": 0 } }] }
    }
  ]
}
```

### Chart — X/Y Axis
```json
"objects": {
  "categoryAxis": [{
    "properties": {
      "show": { "expr": { "Literal": { "Value": "true" } } },
      "fontSize": { "expr": { "Literal": { "Value": "9D" } } },
      "fontColor": { "solid": { "color": { "expr": { "Literal": { "Value": "'#666666'" } } } } },
      "showAxisTitle": { "expr": { "Literal": { "Value": "false" } } }
    }
  }],
  "valueAxis": [{
    "properties": {
      "show": { "expr": { "Literal": { "Value": "true" } } },
      "fontSize": { "expr": { "Literal": { "Value": "9D" } } },
      "showAxisTitle": { "expr": { "Literal": { "Value": "false" } } },
      "gridlineShow": { "expr": { "Literal": { "Value": "true" } } },
      "gridlineColor": { "solid": { "color": { "expr": { "Literal": { "Value": "'#F0F0F0'" } } } } }
    }
  }]
}
```

### Chart — Legend
```json
"objects": {
  "legend": [{
    "properties": {
      "show": { "expr": { "Literal": { "Value": "true" } } },
      "position": { "expr": { "Literal": { "Value": "'Top'" } } },
      "fontSize": { "expr": { "Literal": { "Value": "9D" } } }
    }
  }]
}
```

### Chart — Data Labels
```json
"objects": {
  "labels": [{
    "properties": {
      "show": { "expr": { "Literal": { "Value": "true" } } },
      "fontSize": { "expr": { "Literal": { "Value": "9D" } } },
      "color": { "solid": { "color": { "expr": { "Literal": { "Value": "'#333333'" } } } } },
      "labelDisplayUnits": { "expr": { "Literal": { "Value": "0L" } } }
    }
  }]
}
```

## Page-Level Formatting

In `page.json`, the `objects` property controls page background:

### Page Background Image
```json
"objects": {
  "background": [{
    "properties": {
      "image": {
        "image": {
          "name": { "expr": { "Literal": { "Value": "'background.png'" } } },
          "url": {
            "expr": {
              "ResourcePackageItem": {
                "PackageName": "RegisteredResources",
                "PackageType": 1,
                "ItemName": "background_17944747330361153.png"
              }
            }
          },
          "scaling": { "expr": { "Literal": { "Value": "'Normal'" } } }
        }
      },
      "transparency": { "expr": { "Literal": { "Value": "0D" } } }
    }
  }]
}
```

### Page Background Color (no image)
```json
"objects": {
  "background": [{
    "properties": {
      "color": { "solid": { "color": { "expr": { "Literal": { "Value": "'#F5F5F5'" } } } } },
      "transparency": { "expr": { "Literal": { "Value": "0D" } } }
    }
  }]
}
```

## Literal Value Types

| Type | Format | Example |
|---|---|---|
| Boolean | `"true"` or `"false"` | `{ "Value": "true" }` |
| String | Single-quoted | `{ "Value": "'Segoe UI'" }` |
| Decimal | Number + D suffix | `{ "Value": "12D" }` |
| Long/Integer | Number + L suffix | `{ "Value": "0L" }` |
| Color | Single-quoted hex | `{ "Value": "'#FF0000'" }` |
| Enum | Single-quoted name | `{ "Value": "'Top'" }` |
