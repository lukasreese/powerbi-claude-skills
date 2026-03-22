# Template 2: IBCS Bar Variance Chart

**Visual type**: `barChart`
**Required measures**: NONE in semantic model — all DAX is embedded via NativeVisualCalculation
**Complexity**: High (but self-contained)

## Overview

A stacked bar chart with 13 series using NativeVisualCalculation:
- **Actual bar** (dark blue): The user's actual measure
- **Transparent spacer bars**: Position data labels and variance sections
- **Positive bar** (green): Favorable variance, capped at 40% of max
- **Negative bar** (red): Unfavorable variance, capped at 40% of max
- **Reference lines**: Column headers (AC, Variance, Variance %, Comparison)
- **Conditional font color**: Red for negative variance %, black for positive

## Key Advantage: NativeVisualCalculation

All 13 DAX expressions are embedded in the visual JSON under `query.queryState.Y.projections[]`.
This means **zero measures need to be created in the semantic model** — only the user's base
actual measure, comparison measure, and variance measure must exist.

## Query Structure — The 13 Projections

Each NativeVisualCalculation uses this JSON pattern:
```json
{
  "field": {
    "NativeVisualCalculation": {
      "Language": "dax",
      "Expression": "...DAX expression...",
      "Name": "Calculation Name"
    }
  },
  "queryRef": "select",       // select, select1, select2... (unique)
  "nativeQueryRef": "Name",   // display name
  "hidden": true              // optional, hides from tooltips
}
```

### Projection Order (must be exact):

| # | queryRef | Name | Purpose | Visible |
|---|----------|------|---------|---------|
| 0 | select | Max Actual Value | `EXPANDALL(MAXX(ROWS, [AC]), ROWS)` | hidden |
| 1 | select1 | Min X-axis | Min of actual, or 0 if positive | hidden |
| 2 | — | AC | User's `{{ACTUAL}}` measure (regular Measure, not NVC) | visible |
| 3 | select4 | Transp. Bar1 | Spacer after AC bar | transparent |
| 4 | select5 | Transp. Bar2 | Spacer for negative label | transparent |
| 5 | select2 | Negative Bar | `IF(variance < 0, MIN(ABS(variance), cap))` | red |
| 6 | select6 | Positive Bar | `IF(variance >= 0, MIN(variance, cap))` | green |
| 7 | select7 | Transp. Bar3 | Spacer after positive bar | transparent |
| 8 | select8 | Transp. Bar4 | Spacer for variance % | transparent |
| 9 | select3 | Ref. line abs. var. | Reference line position calc | hidden |
| 10 | select9 | Transp. Bar5 | Spacer for comparison label | transparent |
| 11 | select10 | Ref. line var. % | Reference line position | hidden |
| 12 | select11 | Ref. line comparison value | Reference line position | hidden |
| 13 | select12 | CF color var % | Returns `"#B34E4E"` or `"#000000"` | hidden |

### NativeVisualCalculation DAX (parameterized)

Replace `[AC]` with user's actual measure ref, `[Variance]` with variance measure ref.
The `[AC]` in NVC refers to queryRef "Kennzahlen und Werte.IST" (the actual measure projection).

**Variance cap**: `__Cap = 0.4` (40% of max actual value)
**Spacing multipliers**: Transp. Bar3 uses `0.35`, Transp. Bar4 uses `0.12`
**Reference line**: `(Max Actual + Max Negative) * 1.5`

## Key Object Settings

### valueAxis
- `show: false`
- `start`: Bound to `Min X-axis` via `SelectRef: { ExpressionName: "select1" }`

### layout
- `stackedGapSize: 0D` — no gaps between stacked segments
- `seriesOrderSorted: false`, `seriesOrderReversed: false`

### dataPoint colors
- AC: `#0C3549`
- Positive Bar: `#44C088`
- Negative Bar: `#ED7373`
- Transp. Bar1-5: `fillTransparency: 100D`

### Labels with dynamicLabelValue

Each transparent bar shows a different measure's value as its label:
- Transp. Bar1 (select4): Shows `{{ACTUAL}}` value → `labelPosition: InsideBase`
- Transp. Bar2 (select5): Shows `Negative abs. variance` → `labelPosition: InsideEnd`
- Transp. Bar3 (select7): Shows `Positive abs. variance` → `labelPosition: InsideBase`
- Transp. Bar4 (select8): Shows `{{VARIANCE_PCT}}` → `labelPosition: InsideEnd`
- Transp. Bar5 (select9): Shows `{{COMPARISON}}` → `labelPosition: InsideEnd`

### Conditional Font Color

The variance % label uses `SelectRef` for dynamic color:
```json
{
  "color": { "solid": { "color": { "expr": { "SelectRef": { "ExpressionName": "select12" } } } } },
  "selector": { "data": [{ "dataViewWildcard": { "matchingOption": 1 } }], "metadata": "select8" }
}
```

### Reference Lines (y1AxisReferenceLine)

4 reference lines (used as column headers):

1. **AC header** (id "1"): Solid dark, `width: 2D`, label "IST" → rename to "AC" or user label
2. **Comparison header** (id "2"): `transparency: 100D` (invisible), positioned at `select11`, label only
3. **Variance header** (id "3"): Solid dark, `width: 2D`, positioned at `select3`
4. **Variance % header** (id "4"): `transparency: 100D`, positioned at `select10`, label only

### Tooltips
- Custom tooltip showing variance measure in Tooltips well

### Visual Container Objects
- Title: `fontSize: 16D`, `bold: true`
- Subtitle: `show: true`, `fontSize: 12D`
- Divider: `show: true`
- Border: `show: false`, `radius: 5D`
- Padding: `10D` all sides