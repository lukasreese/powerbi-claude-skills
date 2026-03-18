# Sample TMDL Table Files

Annotated examples of real TMDL table definitions showing all object types the dependency analyzer must parse.

---

## Example 1: Imported Table (Most Common)

**File:** `tables/Sales.tmdl`

```tmdl
/// Fact table containing all sales transactions
table Sales

	/// Total revenue across all transactions
	measure 'Total Sales' = SUM('Sales'[Amount])
		formatString: $ #,##0.00
		displayFolder: Revenue
		lineageTag: 8a2b3c4d-1234-5678-9abc-def012345678

	/// Prior year total sales using time intelligence
	measure 'Total Sales PY' = CALCULATE([Total Sales], SAMEPERIODLASTYEAR('Calendar'[Date]))
		formatString: $ #,##0.00
		displayFolder: Revenue
		lineageTag: 9b3c4d5e-2345-6789-abcd-ef0123456789

	/// Year-over-year variance
	measure 'Sales YoY' = [Total Sales] - [Total Sales PY]
		formatString: $ #,##0.00
		displayFolder: Variance
		lineageTag: 0c4d5e6f-3456-789a-bcde-f01234567890

	/// Year-over-year variance percentage
	measure 'Sales YoY %' = DIVIDE([Sales YoY], [Total Sales PY])
		formatString: 0.0%;-0.0%;0.0%
		displayFolder: Variance
		lineageTag: 1d5e6f7a-4567-89ab-cdef-012345678901

	/// Conditional color for KPI cards
	measure 'Sales KPI Color' = IF([Sales YoY] >= 0, "#00B050", "#FF0000")
		displayFolder: _Formatting
		isHidden
		lineageTag: 2e6f7a8b-5678-9abc-def0-123456789012

	column Amount
		dataType: double
		formatString: #,##0.00
		sourceColumn: Amount
		summarizeBy: sum
		lineageTag: 3f7a8b9c-6789-abcd-ef01-234567890123

	column 'Product Key'
		dataType: int64
		isHidden
		sourceColumn: ProductKey
		summarizeBy: none
		lineageTag: 4a8b9c0d-789a-bcde-f012-345678901234

	column 'Date Key'
		dataType: dateTime
		isHidden
		sourceColumn: OrderDate
		summarizeBy: none
		lineageTag: 5b9c0d1e-89ab-cdef-0123-456789012345

	column Quantity
		dataType: int64
		sourceColumn: Quantity
		summarizeBy: sum
		lineageTag: 6c0d1e2f-9abc-def0-1234-567890123456

	column Region
		dataType: string
		sourceColumn: Region
		summarizeBy: none
		lineageTag: 7d1e2f3a-abcd-ef01-2345-678901234567

	/// Calculated column: profit margin per row
	column 'Profit Margin' = DIVIDE('Sales'[Amount] - 'Sales'[Cost], 'Sales'[Amount])
		dataType: double
		formatString: 0.0%
		summarizeBy: none
		isDataTypeInferred
		lineageTag: 8e2f3a4b-bcde-f012-3456-789012345678

	column Cost
		dataType: double
		isHidden
		sourceColumn: Cost
		summarizeBy: sum
		lineageTag: 9f3a4b5c-cdef-0123-4567-890123456789

	partition Sales = m
		mode: import
		source =
			let
				Source = Sql.Database(Server, Database),
				dbo_Sales = Source{[Schema="dbo", Item="Sales"]}[Data],
				FilteredRows = Table.SelectRows(dbo_Sales, each [OrderDate] >= #date(2020, 1, 1)),
				RenamedColumns = Table.RenameColumns(FilteredRows, {{"OrderDate", "Date Key"}})
			in
				RenamedColumns
```

**What to extract:**
| Object | Type | Name | Dependencies |
|--------|------|------|-------------|
| Measure | measure | Total Sales | Column: Sales[Amount] |
| Measure | measure | Total Sales PY | Measure: [Total Sales], Column: Calendar[Date] |
| Measure | measure | Sales YoY | Measures: [Total Sales], [Total Sales PY] |
| Measure | measure | Sales YoY % | Measures: [Sales YoY], [Total Sales PY] |
| Measure | measure | Sales KPI Color | Measure: [Sales YoY] |
| Column | source | Amount | Partition source |
| Column | source | Product Key | Partition source (hidden) |
| Column | source | Date Key | Partition source (hidden) |
| Column | source | Quantity | Partition source |
| Column | source | Region | Partition source |
| Column | calculated | Profit Margin | Columns: Sales[Amount], Sales[Cost] |
| Column | source | Cost | Partition source (hidden) |
| Partition | M | Sales | References: Server, Database parameters |

---

## Example 2: Measures-Only Table

**File:** `tables/Measures.tmdl`

```tmdl
table Measures

	measure 'Gross Profit' = [Total Sales] - [Total Cost]
		formatString: $ #,##0.00
		lineageTag: aa1b2c3d-1111-2222-3333-444455556666

	measure 'Gross Margin %' = DIVIDE([Gross Profit], [Total Sales])
		formatString: 0.0%
		lineageTag: bb2c3d4e-2222-3333-4444-555566667777

	measure 'Active Customers' =
		CALCULATE(
			DISTINCTCOUNT('Sales'[CustomerID]),
			'Sales'[Amount] > 0
		)
		formatString: #,##0
		lineageTag: cc3d4e5f-3333-4444-5555-666677778888

	column 'Measures Column'
		dataType: string
		isHidden
		sourceColumn: [Value]
		summarizeBy: none

	partition Measures = m
		mode: import
		source = Table.FromValue(null)
```

**Key observations:**
- Measures-only tables often have a hidden dummy column and a `Table.FromValue(null)` partition
- Measures reference objects in OTHER tables (`Sales[Amount]`, `Sales[CustomerID]`)
- These cross-table references are critical for dependency tracking

---

## Example 3: Calculated Table (DAX Source)

**File:** `tables/DateBridge.tmdl`

```tmdl
table DateBridge =
	CALENDAR(DATE(2020, 1, 1), DATE(2026, 12, 31))

	column [Date]
		dataType: dateTime
		formatString: Short Date
		isKey
		summarizeBy: none
		isNameInferred
		isDataTypeInferred
		lineageTag: dd4e5f6a-4444-5555-6666-777788889999
```

**Key observations:**
- Table expression follows `=` on the table line
- Columns have `isNameInferred` — names come from the DAX expression output
- ALL columns are inseparable from the table — deleting one means modifying the DAX
- Dependencies: The `CALENDAR` function has no table/column references, but more complex calculated tables (e.g., `SUMMARIZE`, `ADDCOLUMNS`) will reference other tables/columns

---

## Example 4: Calendar/Date Table

**File:** `tables/Calendar.tmdl`

```tmdl
table Calendar
	dataCategory: Time

	measure 'Date Range' =
		FORMAT(MIN('Calendar'[Date]), "MMM YYYY") & " - " & FORMAT(MAX('Calendar'[Date]), "MMM YYYY")
		lineageTag: ee5f6a7b-5555-6666-7777-888899990000

	column Date
		dataType: dateTime
		isKey
		formatString: Short Date
		sourceColumn: Date
		summarizeBy: none
		lineageTag: ff6a7b8c-6666-7777-8888-999900001111

	column Year
		dataType: int64
		sourceColumn: Year
		summarizeBy: none
		lineageTag: 007b8c9d-7777-8888-9999-000011112222

	column 'Month Name'
		dataType: string
		sourceColumn: MonthName
		summarizeBy: none
		sortByColumn: 'Month Number'
		lineageTag: 118c9d0e-8888-9999-0000-111122223333

	column 'Month Number'
		dataType: int64
		isHidden
		sourceColumn: MonthNum
		summarizeBy: none
		lineageTag: 229d0e1f-9999-0000-1111-222233334444

	column Quarter
		dataType: string
		sourceColumn: Quarter
		summarizeBy: none
		lineageTag: 330e1f2a-0000-1111-2222-333344445555

	hierarchy 'Date Hierarchy'
		lineageTag: 441f2a3b-1111-2222-3333-444455556666

		level Year
			column: Year

		level Quarter
			column: Quarter

		level 'Month Name'
			column: 'Month Name'

	partition Calendar = m
		mode: import
		source =
			let
				Source = List.Dates(#date(2020,1,1), 2557, #duration(1,0,0,0)),
				ToTable = Table.FromList(Source, Splitter.SplitByNothing(), {"Date"}, null, ExtraValues.Error),
				AddYear = Table.AddColumn(ToTable, "Year", each Date.Year([Date]), Int64.Type),
				AddMonthNum = Table.AddColumn(AddYear, "MonthNum", each Date.Month([Date]), Int64.Type),
				AddMonthName = Table.AddColumn(AddMonthNum, "MonthName", each Date.ToText([Date], "MMM"), type text),
				AddQuarter = Table.AddColumn(AddMonthName, "Quarter", each "Q" & Text.From(Date.QuarterOfYear([Date])), type text)
			in
				AddQuarter
```

**Key observations:**
- `dataCategory: Time` marks this as the date table
- `sortByColumn: 'Month Number'` creates a hidden dependency — Month Number must exist for Month Name to sort correctly
- Hierarchy levels reference columns — those columns become dependencies if the hierarchy is used
- The M partition creates all columns programmatically — no external query dependencies

---

## Example 5: Calculation Group

**File:** `tables/Time Intelligence.tmdl`

```tmdl
table 'Time Intelligence'
	calculationGroup

	column Name
		dataType: string
		sourceColumn: Name
		sortByColumn: Ordinal
		lineageTag: 552a3b4c-2222-3333-4444-555566667777

	column Ordinal
		dataType: int64
		isHidden
		sourceColumn: Ordinal
		summarizeBy: none
		lineageTag: 663b4c5d-3333-4444-5555-666677778888

	calculationItem Current = SELECTEDMEASURE()
		ordinal: 0

	calculationItem YTD = CALCULATE(SELECTEDMEASURE(), DATESYTD('Calendar'[Date]))
		ordinal: 1

	calculationItem 'Prior Year' = CALCULATE(SELECTEDMEASURE(), SAMEPERIODLASTYEAR('Calendar'[Date]))
		ordinal: 2

	calculationItem 'YoY Change' =
		var current = SELECTEDMEASURE()
		var py = CALCULATE(SELECTEDMEASURE(), SAMEPERIODLASTYEAR('Calendar'[Date]))
		return current - py
		ordinal: 3
```

**Key observations:**
- `calculationGroup` keyword on the table line
- `calculationItem` instead of `measure`
- `SELECTEDMEASURE()` is a dynamic reference — cannot be statically resolved
- Still has regular columns (`Name`, `Ordinal`)
- Explicit `ordinal` property controls display order
- DAX in calculation items references `Calendar[Date]` — this is a real dependency
