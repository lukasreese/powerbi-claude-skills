# IBCS Color Palette

Standard colors used across all IBCS visual templates.

| Role | Hex | CSS Variable | Usage |
|------|-----|-------------|-------|
| Actual (AC) | `#0C3549` | `--ibcs-actual` | Current period bars/columns — dark blue |
| Comparison (PY/BU) | `#CCCCCC` | `--ibcs-comparison` | Prior year / budget / forecast — gray |
| Positive variance | `#44C088` | `--ibcs-positive` | Favorable delta — green |
| Negative variance | `#ED7373` | `--ibcs-negative` | Unfavorable delta — red |
| Negative variance (dark) | `#B34E4E` | `--ibcs-neg-dark` | Conditional font color for negative % |
| Axis / labels | `#404040` | `--ibcs-text` | Text, axis labels — dark gray |
| Grid / separator | `#c6c6c6` | `--ibcs-grid` | Grid lines, SVG center lines — light gray |
| White (separator) | `#FFFFFF` | `--ibcs-white` | White separator lines between sections |

## How to Customize

When generating visuals, replace the hex values in:
- `dataPoint[].properties.fill.solid.color` — bar/column fill colors
- `error[].properties.barColor.solid.color` — error bar (variance indicator) colors
- `y1AxisReferenceLine[].properties.lineColor.solid.color` — reference line colors
- SVG measures: `_ColorAC`, `_ColorPY`, `_ColorGreen`, `_ColorRed` variables

## PBIR JSON Color Format

Colors in PBIR visual.json use this format:
```json
{ "fill": { "solid": { "color": { "expr": { "Literal": { "Value": "'#0C3549'" } } } } } }
```
Note the single quotes around the hex value inside the JSON string.