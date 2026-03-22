# IBCS DAX Measure Templates

These measures support the **IBCS Column Variance Chart** (Template 1).
All use placeholders — replace with the user's actual measure/table names.

## Placeholders

| Placeholder | Example | Description |
|-------------|---------|-------------|
| `{{ACTUAL}}` | `[Sales]`, `[IST]`, `[Revenue]` | The user's actual period measure |
| `{{COMPARISON}}` | `[Sales PY]`, `[IST VJ]`, `[Budget]` | The user's comparison period measure |
| `{{DATE_TABLE}}` | `Kalender`, `Date`, `Calendar` | Date/time dimension table |
| `{{DATE_COLUMN}}` | `Monat`, `Month`, `Period` | Category column on the date table |
| `{{DATE_DATE_COL}}` | `Datum`, `Date` | The actual date column (for ALLSELECTED) |
| `{{DATE_SORT_COL}}` | `Monat (Zahl)`, `MonthNumber` | Numeric sort column for the category |
| `{{MEASURE_TABLE}}` | `_IBCS`, `_Measures` | Table where helper measures are created |

## Measure Dependency Chain

Create in this order (each depends on prior measures):

```
01 → 02, 03 → 04, 05, 06 → 07, 08, 09, 10 → 11, 12 → 13, 14 → 15
```

## Measure Definitions

### 01. Month <= Selected Month
```dax
VAR __SelectedMonth = MAX({{DATE_TABLE}}[{{DATE_SORT_COL}}])
VAR __SupplMonth = MAX({{DATE_TABLE}}[{{DATE_SORT_COL}}])
VAR __Result = IF(__SupplMonth <= __SelectedMonth, 1, 0)
RETURN __Result
```
Format: `0` | Display folder: `__IBCS Column Variance`

### 02. Actual Value
```dax
{{ACTUAL}}
```

### 03. Comparison Value
```dax
{{COMPARISON}}
```

### 04. Data Label Position abs. var.
```dax
VAR __IsHistoricalMonth = [01. Month <= Selected Month]
VAR __Comparison = [03. Comparison Value]
VAR __Actual = [02. Actual Value]
VAR __MaxValue = MAX(__Comparison, __Actual)
VAR __Result = IF(__IsHistoricalMonth = 1, __MaxValue, BLANK())
RETURN __Result
```

### 05. Data Label Value abs. var.
```dax
VAR __IsHistoricalMonth = [01. Month <= Selected Month]
VAR __Actual = [02. Actual Value]
VAR __Comparison = [03. Comparison Value]
VAR __Variance = __Actual - __Comparison
VAR __Result = IF(__IsHistoricalMonth = 1, __Variance, BLANK())
RETURN __Result
```

### 06. Max Value of Columns
```dax
VAR __MAX_Value_Actual = MAXX(ALLSELECTED({{DATE_TABLE}}[{{DATE_DATE_COL}}]), {{ACTUAL}})
VAR __MAX_Value_Comparison = MAXX(ALLSELECTED({{DATE_TABLE}}[{{DATE_DATE_COL}}]), {{COMPARISON}})
VAR __Result = MAX(__MAX_Value_Actual, __MAX_Value_Comparison)
RETURN __Result
```

### 07. Ref. line Position pos. rel. var %
```dax
[06. Max Value of Columns] * 1.75
```

### 08. Ref. line Position neg. rel. var %
```dax
[06. Max Value of Columns] * 1.75
```

### 09. Data Label Value pos. rel. var. %
```dax
VAR __IsHistoricalMonth = [01. Month <= Selected Month]
VAR __Actual = [02. Actual Value]
VAR __Comparison = [03. Comparison Value]
VAR __Variance = __Actual - __Comparison
VAR __RelativeVariance = DIVIDE(__Variance, __Comparison)
VAR __Result = IF(__IsHistoricalMonth = 1 && __Variance >= 0, __RelativeVariance, BLANK())
RETURN __Result
```
Format: `+0.0%;-0.0%;0.0%`

### 10. Data Label Value neg. rel. var. %
```dax
VAR __IsHistoricalMonth = [01. Month <= Selected Month]
VAR __Actual = [02. Actual Value]
VAR __Comparison = [03. Comparison Value]
VAR __Variance = __Actual - __Comparison
VAR __RelativeVariance = DIVIDE(__Variance, __Comparison)
VAR __Result = IF(__IsHistoricalMonth = 1 && __Variance < 0, __RelativeVariance, BLANK())
RETURN __Result
```
Format: `+0.0%;-0.0%;0.0%`

### 11. Upper Bound error bar pos. rel. var. %
```dax
// Scale (x1.5): makes bars longer. Cap (50%): avoids overlaps.
VAR __IsHistoricalMonth = [01. Month <= Selected Month]
VAR __RelativeVariance = [09. Data Label Value pos. rel. var. %]
VAR __MaxBarLength = [06. Max Value of Columns] * 1.5
VAR __MaxCap = 0.5
VAR __Result = IF(__IsHistoricalMonth = 1, MIN(__RelativeVariance, __MaxCap) * __MaxBarLength, BLANK())
RETURN __Result
```

### 12. Lower Bound error bar neg. rel. var. %
```dax
VAR __IsHistoricalMonth = [01. Month <= Selected Month]
VAR __RelativeVariance = [10. Data Label Value neg. rel. var. %]
VAR __MaxBarLength = [06. Max Value of Columns] * 1.5
VAR __MaxCap = -0.5
VAR __Result = IF(__IsHistoricalMonth = 1, MAX(__RelativeVariance, __MaxCap) * __MaxBarLength, BLANK())
RETURN __Result
```

### 13. Data Label Position pos. rel var. %
```dax
IF(
    [11. Upper Bound error bar pos. rel. var. %] >= 0,
    [11. Upper Bound error bar pos. rel. var. %] + [07. Ref. line Position pos. rel. var %]
)
```

### 14. Data Label Position neg. rel var. %
```dax
IF(
    [12. Lower Bound error bar neg. rel. var. %] < 0,
    [12. Lower Bound error bar neg. rel. var. %] + [08. Ref. line Position neg. rel. var %]
)
```

### 15. Max Y-axis Value
```dax
MAXX(
    ALLSELECTED({{DATE_TABLE}}[{{DATE_SORT_COL}}], {{DATE_TABLE}}[{{DATE_COLUMN}}]),
    [13. Data Label Position pos. rel var. %]
) * 1.1
```

## Bar Chart Additional Measures (Template 2 — only needed if NOT using NativeVisualCalculation)

### Positive abs. variance
```dax
VAR __Variance = {{ACTUAL}} - {{COMPARISON}}
VAR __Result = IF(__Variance >= 0, __Variance, BLANK())
RETURN __Result
```

### Negative abs. variance
```dax
VAR __Variance = {{ACTUAL}} - {{COMPARISON}}
VAR __Result = IF(__Variance < 0, __Variance, BLANK())
RETURN __Result
```

### Variance
```dax
{{ACTUAL}} - {{COMPARISON}}
```

### Variance %
```dax
VAR __Comparison = {{COMPARISON}}
VAR __Actual = {{ACTUAL}}
RETURN IF(
    NOT(AND(ISBLANK(__Actual), ISBLANK(__Comparison))),
    DIVIDE(__Actual - __Comparison, ABS(__Comparison))
)
```