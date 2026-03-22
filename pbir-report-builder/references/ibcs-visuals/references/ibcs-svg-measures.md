# IBCS SVG-Generating DAX Measures

These measures create inline SVG images for use in **native pivotTable** visuals (Templates 3 & 4).
Each returns a `data:image/svg+xml;utf8,...` string. Set `dataCategory: ImageUrl` on the measure.

## Placeholders

| Placeholder | Example | Description |
|-------------|---------|-------------|
| `{{ACTUAL}}` | `[Sales]` | User's actual measure |
| `{{COMPARISON}}` | `[Sales PY]` | User's comparison measure |
| `{{VARIANCE}}` | `[Variance]` | `{{ACTUAL}} - {{COMPARISON}}` |
| `{{VARIANCE_PCT}}` | `[Variance %]` | `DIVIDE({{ACTUAL}} - {{COMPARISON}}, {{COMPARISON}})` |
| `{{DATE_TABLE}}` | `Date` | Date dimension table |
| `{{DATE_COLUMN}}` | `Month` | Category column |

## 1. SVG Style (shared helper)

```dax
"<style><![CDATA[text{font-family:Segoe UI;font-size:14px;alignment-baseline:middle;}]]></style>"
```

No `dataCategory` needed — this is a text helper, not rendered as an image.

## 2. AC,PY SVG (overlapping bars)

Shows actual (dark) overlapping comparison (gray) with an AC value label.

```dax
VAR _ColorAC = "#0C3549"
VAR _ColorPY = "#CCCCCC"
VAR _ValueAC = IF(HASONEVALUE({{DATE_TABLE}}[{{DATE_COLUMN}}]), {{ACTUAL}}, {{ACTUAL}})
VAR _FontWeight = IF(HASONEVALUE({{DATE_TABLE}}[{{DATE_COLUMN}}]), "normal", "bold")
VAR _ValuePY = {{COMPARISON}}
VAR _maxValue = MAX(
    MAXX(ALLSELECTED({{DATE_TABLE}}), {{ACTUAL}}),
    MAXX(ALLSELECTED({{DATE_TABLE}}), {{COMPARISON}})
) / 0.9
VAR _WidthAC = FORMAT(DIVIDE(_ValueAC, _maxValue), "0%")
VAR _WidthPY = FORMAT(DIVIDE(_ValuePY, _maxValue), "0%")
VAR _Rank = 100000 + RANKX(ALLSELECTED({{DATE_TABLE}}), {{ACTUAL}}, , ASC)
VAR _AC_Label =
    VAR a = ABS(_ValueAC)
    RETURN SWITCH(TRUE(),
        ISBLANK(_ValueAC), "",
        a >= 1000000, FORMAT(DIVIDE(_ValueAC, 1000000), "0.00") & "M",
        a >= 1000, FORMAT(DIVIDE(_ValueAC, 1000), "0.0") & "K",
        FORMAT(_ValueAC, "#,0")
    )
VAR _SVG =
    "data:image/svg+xml;utf8," &
    "<svg xmlns=""http://www.w3.org/2000/svg"">
    <!-- " & _Rank & " -->
    <rect y=""2%"" x=""0"" width=""" & _WidthPY & """ height=""67%"" fill=""" & _ColorPY & """ />
    <rect y=""17%"" x=""0"" width=""" & _WidthAC & """ height=""67%"" fill=""" & _ColorAC & """ />
    <text x=""" & _WidthAC & """ dx=""5"" y=""55%"" font-weight=""" & _FontWeight & """ font-family=""Segoe UI"" font-size=""12"">" & _AC_Label & "</text>
    " & [SVG Style] & "
    </svg>"
RETURN _SVG
```

**dataCategory**: `ImageUrl`

## 3. Delta SVG (absolute variance bar)

Centered bar: green right (positive) or red left (negative) with value label.

```dax
VAR _ColorGrey = "#c6c6c6"
VAR _ColorRed = "#ED7373"
VAR _ColorGreen = "#44C088"
VAR _Value = IF(HASONEVALUE({{DATE_TABLE}}[{{DATE_COLUMN}}]), {{VARIANCE}}, [Average Variance])
VAR _FontWeight = IF(HASONEVALUE({{DATE_TABLE}}[{{DATE_COLUMN}}]), "normal", "bold")
VAR _maxValue_raw = MAX(
    ABS(MAXX(ALLSELECTED({{DATE_TABLE}}), {{VARIANCE}})),
    ABS(MINX(ALLSELECTED({{DATE_TABLE}}), {{VARIANCE}}))
)
VAR _maxValue = IF(_maxValue_raw > 0, _maxValue_raw, 1)
VAR _WidthValue = (DIVIDE(ABS(_Value), _maxValue) / 2) * 0.8
VAR _barColor = IF(_Value > 0, _ColorGreen, _ColorRed)
VAR _Width = SUBSTITUTE(FORMAT(_WidthValue, "0.####%"), ",", ".")
VAR _X = IF(_Value >= 0, "50%", SUBSTITUTE(FORMAT(0.5 - _WidthValue, "0.####%"), ",", "."))
VAR _AbsVal = ABS(_Value)
VAR _LabelCore = SWITCH(TRUE(),
    _AbsVal >= 1000000, FORMAT(DIVIDE(_AbsVal, 1000000), "0.00") & "M",
    _AbsVal >= 1000, FORMAT(DIVIDE(_AbsVal, 1000), "0.0") & "K",
    FORMAT(_AbsVal, "#,0")
)
VAR _LabelText = SWITCH(TRUE(), _Value > 0, "+" & _LabelCore, _Value < 0, "-" & _LabelCore, _LabelCore)
VAR _Gap = 0.02
VAR _XNumPos = 0.5 + _WidthValue
VAR _XNumNeg = 0.5 - _WidthValue
VAR _XTextNum = IF(_Value >= 0, _XNumPos + _Gap, _XNumNeg - _Gap)
VAR _ClampLeft = 0.04
VAR _ClampRight = 0.96
VAR _XTextNumClamped = MAX(_ClampLeft, MIN(_ClampRight, _XTextNum))
VAR _XText = SUBSTITUTE(FORMAT(_XTextNumClamped, "0.####%"), ",", ".")
VAR _Anchor = IF(_Value >= 0, "start", "end")
VAR _DX = IF(_Value >= 0, 5, -5)
VAR _Rank = 10000 + RANKX(ALLSELECTED({{DATE_TABLE}}), {{VARIANCE}}, , ASC)
VAR _SVG =
    "data:image/svg+xml;utf8," &
    "<svg xmlns=""http://www.w3.org/2000/svg"">
    /*" & _Rank & "*/
    <line x1=""50%"" x2=""50%"" y1=""0%"" y2=""100%"" stroke=""" & _ColorGrey & """ stroke-width=""1""></line>
    <rect y=""17%"" x=""" & _X & """ width=""" & _Width & """ height=""67%"" fill=""" & _barColor & """></rect>
    <text text-anchor=""" & _Anchor & """ x=""" & _XText & """ dx=""" & _DX & """ y=""55%"" font-weight=""" & _FontWeight & """ font-family=""Segoe UI"" font-size=""12"">" & _LabelText & "</text>
    " & [SVG Style] & "
    </svg>"
RETURN _SVG
```

**dataCategory**: `ImageUrl`

## 4. Delta% SVG (percentage variance with pinhead)

Pin-style bar: narrow colored bar with thick black pinhead marker, percentage label.

```dax
VAR _ColorGrey = "#c6c6c6"
VAR _ColorRed = "#ED7373"
VAR _ColorGreen = "#44C088"
VAR _ColorBlack = "#404040"
VAR _Value = IF(HASONEVALUE({{DATE_TABLE}}[{{DATE_COLUMN}}]), {{VARIANCE_PCT}}, [Average Variance %])
VAR _FontWeight = IF(HASONEVALUE({{DATE_TABLE}}[{{DATE_COLUMN}}]), "normal", "bold")
VAR _maxValue_raw = MAX(
    ABS(MAXX(ALLSELECTED({{DATE_TABLE}}), {{VARIANCE_PCT}})),
    ABS(MINX(ALLSELECTED({{DATE_TABLE}}), {{VARIANCE_PCT}}))
)
VAR _maxValue = IF(_maxValue_raw > 0, _maxValue_raw, 1)
VAR _WidthValue = (DIVIDE(ABS(_Value), _maxValue) / 2) * 0.65
VAR _barColor = IF(_Value > 0, _ColorGreen, _ColorRed)
VAR _Width = SUBSTITUTE(FORMAT(_WidthValue, "0.####%"), ",", ".")
VAR _X = IF(_Value >= 0, "50%", SUBSTITUTE(FORMAT(0.5 - _WidthValue, "0.####%"), ",", "."))
VAR _XPinPos = SUBSTITUTE(FORMAT(0.5 + _WidthValue - 3/200, "0.####%"), ",", ".")
VAR _XPinNeg = SUBSTITUTE(FORMAT(0.5 - _WidthValue - 3/200, "0.####%"), ",", ".")
VAR _XPinhead = IF(_Value >= 0, _XPinPos, _XPinNeg)
VAR _Gap = 0.02
VAR _ClampLeft = 0.04
VAR _ClampRight = 0.96
VAR _XTextNum = IF(_Value >= 0, 0.5 + _WidthValue + _Gap, 0.5 - _WidthValue - _Gap)
VAR _XText = SUBSTITUTE(FORMAT(MAX(_ClampLeft, MIN(_ClampRight, _XTextNum)), "0.####%"), ",", ".")
VAR _Anchor = IF(_Value >= 0, "start", "end")
VAR _DX = IF(_Value >= 0, 6, -6)
VAR _Rank = 10000 + RANKX(ALLSELECTED({{DATE_TABLE}}), {{VARIANCE_PCT}}, , ASC)
VAR _LabelText = FORMAT(_Value, "+0.0%;-0.0%;0.0%")
VAR _SVG =
    "data:image/svg+xml;utf8," &
    "<svg xmlns=""http://www.w3.org/2000/svg"">
    /*" & _Rank & "*/
    <line x1=""50%"" x2=""50%"" y1=""0%"" y2=""100%"" stroke=""" & _ColorGrey & """ stroke-width=""1"" />
    <rect y=""35%"" x=""" & _XPinhead & """ width=""10"" height=""30%"" fill=""" & _ColorBlack & """/>
    <rect y=""45%"" x=""" & _X & """ width=""" & _Width & """ height=""10%"" fill=""" & _barColor & """/>
    <text text-anchor=""" & _Anchor & """ x=""" & _XText & """ dx=""" & _DX & """ y=""55%"" font-weight=""" & _FontWeight & """ font-family=""Segoe UI"" font-size=""11"">" & _LabelText & "</text>
    " & [SVG Style] & "
    </svg>"
RETURN _SVG
```

**dataCategory**: `ImageUrl`

## Supporting Measures for SVG Tables

These aggregation measures are needed for the total/subtotal rows:

### Average Variance
```dax
ROUND(AVERAGEX(ALLSELECTED({{DATE_TABLE}}[{{DATE_COLUMN}}]), {{VARIANCE}}), 0)
```

### Average Variance %
```dax
VAR _DeltaTotal = SUMX(ALLSELECTED({{DATE_TABLE}}[{{DATE_COLUMN}}]), {{VARIANCE}})
VAR _CompTotal = SUMX(ALLSELECTED({{DATE_TABLE}}[{{DATE_COLUMN}}]), {{COMPARISON}})
RETURN ROUND(DIVIDE(_DeltaTotal, _CompTotal) * 100, 0)
```