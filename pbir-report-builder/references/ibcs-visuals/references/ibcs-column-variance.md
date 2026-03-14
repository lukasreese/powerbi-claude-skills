# Template 1: IBCS Column Variance Chart

**Visual type**: `lineClusteredColumnComboChart`
**Required measures**: 15 helper measures (see `ibcs-dax-measures.md`)
**Complexity**: High

## Overview

A combo chart showing actual vs comparison columns with variance indicators:
- **Y axis (columns)**: Comparison (gray, behind) + Actual (dark blue, front, overlapping)
- **Y2 axis (lines)**: Hidden lines used purely for positioning reference lines and data labels
- **Error bars**: Green (positive) and red (negative) variance bars growing from reference line
- **Reference lines**: AC header + variance % header line (with data labels as column titles)
- **Dynamic data labels**: Variance percentages positioned above/below error bars

## Query Structure

```json
{
  "Category": { "projections": [{ "field": { "Column": { "Entity": "{{DATE_TABLE}}", "Property": "{{DATE_COLUMN}}" } } }] },
  "Y": { "projections": [
    { "field": { "Measure": { "Entity": "{{MEASURE_TABLE}}", "Property": "03. Comparison Value" } } },
    { "field": { "Measure": { "Entity": "{{MEASURE_TABLE}}", "Property": "02. Actual Value" } } }
  ] },
  "Y2": { "projections": [
    { "field": { "Measure": { "Entity": "{{MEASURE_TABLE}}", "Property": "07. Ref. line Position pos. rel. var %" } } },
    { "field": { "Measure": { "Entity": "{{MEASURE_TABLE}}", "Property": "08. Ref. line Position neg. rel. var %" } } },
    { "field": { "Measure": { "Entity": "{{MEASURE_TABLE}}", "Property": "13. Data Label Position pos. rel var. %" } } },
    { "field": { "Measure": { "Entity": "{{MEASURE_TABLE}}", "Property": "14. Data Label Position neg. rel var. %" } } }
  ] }
}
```

## Key Object Settings

### valueAxis (Y axis)
- `show: false` — hide Y axis labels
- `secShow: false` — hide secondary Y axis
- `alignZeros: true`
- `end`: Bound to measure `15. Max Y-axis Value` via `Measure` expr (dynamic scaling)

### layout
- `clusteredGapOverlaps: true` — comparison bar behind actual bar
- `clusteredGapSize: 75D` — overlap percentage

### dataPoint colors
- Actual (`02.`): fill `#0C3549`
- Comparison (`03.`): fill `#CCCCCC`
- Position lines (`13.`, `14.`): `fillTransparency: 100D` (invisible)

### lineStyles
- `strokeShow: false` — hide all line strokes (Y2 lines are invisible)

### Error Bars (the variance indicators)

Two error bar configs on the Y2 positioning measures:

**Positive (green, on measure 07):**
```json
{
  "enabled": true,
  "barColor": "#44C088",
  "barWidth": 5,
  "barMatchSeriesColor": false,
  "markerShape": "circle", "markerSize": 7,
  "errorRange": { "kind": "ErrorRange", "explicit": {
    "isRelative": true,
    "upperBound": { "Measure": "11. Upper Bound error bar pos. rel. var. %" }
  } }
}
```

**Negative (red, on measure 08):**
```json
{
  "enabled": true,
  "barColor": "#ED7373",
  "barWidth": 5,
  "errorRange": { "kind": "ErrorRange", "explicit": {
    "isRelative": true,
    "lowerBound": { "Measure": "12. Lower Bound error bar neg. rel. var. %" }
  } }
}
```

### Reference Lines (y1AxisReferenceLine)

3 reference lines:

1. **AC** (id "1"): Solid dark line at 0, `width: 2D`, label "AC" (dataLabelText: Name)
2. **Variance %** (id "2"): Solid dark line positioned at measure `07.`, `width: 4D`, label shows variance header
3. **White separator** (id "3"): White line at measure `07.` position, `position: front`, separates sections

### Data Labels

Critical label configurations:

- **Comparison & positioning measures**: `showSeries: false`, `enableValueDataLabel: false` (hide their labels)
- **Actual**: `labelPosition: InsideCenter`, colored background
- **Positive variance %** (on `13.`): `labelPosition: Above`, `dynamicLabelValue` bound to measure `09.`
- **Negative variance %** (on `14.`): `labelPosition: Under`, `dynamicLabelValue` bound to measure `10.`

### Series Labels
- Show on primary Y series (left position, max width 4)
- Hide on all Y2 positioning series

### Visual Container Objects
- Title: configurable text, `fontSize: 16D`
- Subtitle: `show: true`, `fontSize: 12D`
- Divider: `show: true`
- Legend: `show: false`