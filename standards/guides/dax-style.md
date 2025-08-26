# DAX Style Guide (measures, naming, formatting)

This guide standardizes how we write DAX across BI projects. It covers:

- Consistent **measure naming patterns**
- **Comments and documentation** (including TMDL `///` descriptions)
- Readable **formatting** and variable usage
- **Display folders** and metadata
- Common **calculation templates**
- **Performance** and correctness tips

---

## Core principles

1. **Clarity first**: prefer explicit filters and readable names.
2. **One purpose per measure**: compose small measures rather than monoliths.
3. **Document** important business logic with a short description.
4. **Consistency** across models makes adoption easier.

---

## Measure naming pattern

**Pattern**
```
[Scenario] [Category] [Metric Type]
```

**Components**
- **Scenario**: AC, PY, PL, FC
- **Category**: Cost, Noncost, Labor, Production, Efficiency, Revenue, etc.
- **Metric Type**: Metric, Value, Pct, Count, Hours, LBS, etc.

**Examples**
- `AC Cost Metric`
- `AC Noncost Metric`
- `PY Production LBS`
- `FC Efficiency Pct`

**Suffixes (when needed)**: `YTD`, `MTD`, `Adj`, `v2`, `LBS`, `pct`.
- Example: `AC Noncost Metric YTD`

> Keep names short and literal. Use sentence case for long labels in visuals, but keep measure names concise.

---

## Comments and documentation

### Inline comments (preferred)
Use `//` to explain business intent, not syntax.
```DAX
// Percent change vs previous year for the selected context
AC Noncost ΔPY Pct = DIVIDE( [AC] - [PY], [PY] )
```

### TMDL descriptions
Add a concise, human-readable description line above each measure using `///` when managing metadata as TMDL.
```tmdl
/// Percent change vs previous year for noncost metric.
measure 'AC Noncost ΔPY Pct' =
    DIVIDE( [AC Noncost Metric] - [PY Noncost Metric], [PY Noncost Metric] )
```

**Good descriptions** are short, active, and explain *what* the measure returns.

---

## Formatting rules

- **One clause per line** for long measures.
- **Indent** function arguments on new lines for readability.
- **Spaces** around operators: `+ - * / &`.
- Prefer **named variables** for repeated expressions.

**Example**
```DAX
AC Noncost ΔPY Pct =
VAR ac = [AC Noncost Metric]
VAR py = [PY Noncost Metric]
RETURN
    DIVIDE( ac - py, py )
```

---

## Variables and reuse

- Use `VAR` for repeated logic or intermediate results.
- Keep variable names short and meaningful: `ac`, `py`, `startDate`.
- `RETURN` on its own line.

```DAX
AC Efficiency Pct =
VAR produced = [AC Production LBS]
VAR laborHrs = [AC Labor Hours]
RETURN
    DIVIDE( produced, laborHrs )
```

---

## Display folders and metadata

Organize measures to help users find them.

**Folder patterns**
- `KPIs/Production`
- `KPIs/Environmental`
- `KPIs/Efficiency`
- `Time Intelligence/YTD`
- `Time Intelligence/PY`

**TMDL example**
```tmdl
measure 'AC Noncost Metric' = [ ... ]
    displayFolder: KPIs/Noncost

annotation PBI_FormatHint = {"isPercentage": false, "isGeneralNumber": true}
```

---

## Filter semantics

Prefer explicit filter behavior.

- Use `REMOVEFILTERS()` to clear filters on specific columns or tables.
- Use `KEEPFILTERS()` to narrow existing filters without overwriting them.
- Use `ALLEXCEPT()` only when you truly need to preserve specific grouping columns.
- Use `TREATAS()` to pass filters between unrelated tables.

```DAX
AC Cost Metric All Dates =
CALCULATE( [AC Cost Metric], REMOVEFILTERS( dimDate ) )
```

---

## Fiscal time intelligence patterns (aligned to ops fiscal calendar)

> Assumptions: `dimDate` is a marked date table and includes fiscal columns such as:
> `dimDate[Date]`, `dimDate[Ops Fiscal Year Number]`, `dimDate[Ops Fiscal Month Number]` (1–12 or 1–13), and optionally `dimDate[Ops Fiscal Year Month Number]`, `dimDate[Ops Fiscal Week Number]`, `dimDate[Fiscal Day In Period]`.

Below are two robust strategies. Use **A** if your fiscal year simply starts on a different day but still uses **calendar months**. Use **B** for **5‑4‑4* calendars.

### A) Fiscal year with calendar months (e.g., FY starts Jul 1)
If your months are calendar months, you can use built‑in functions with a fiscal year‑end parameter.

**Base metric placeholder**
```DAX
AC = [AC]  // replace with your base measure
```

**Previous Year (PY)**
```DAX
PY Base = CALCULATE( [AC], DATEADD( dimDate[Date], -1, YEAR ) )
```

**Year to Date (YTD)**
> Set your fiscal year end (MM-DD). Example below assumes FY ends on June 30.
```DAX
AC Base YTD = CALCULATE( [AC], DATESYTD( dimDate[Date], "06-30" ) )
```

**Month to Date (MTD)**
```DAX
AC Base MTD = CALCULATE( [AC], DATESMTD( dimDate[Date] ) )
```

**Prior Year to Date (PYTD)**
```DAX
PYTD Base = 
    CALCULATE(
        [AC],
        DATESYTD( DATEADD( dimDate[Date], -1, YEAR ), "06-30" )
    )
```

> Tip: Prefer `DATEADD` over `SAMEPERIODLASTYEAR` when you also need custom `DATESYTD` logic.

---

### B) 5‑4‑4 fiscal calendars
Built‑ins like `DATESYTD/MTD` won’t align with fiscal periods. Use fiscal keys and the current context’s **max date**.

**Recommended date columns**
- `Ops Fiscal Year Number` (e.g., 2025)
- `Ops Fiscal Month Number` (1–13)
- `Ops Fiscal Year Month Number` (e.g., 202501 … 202513)
- `Ops Day of Fiscal Month Number` (1…28/35) ← optional but helpful for MTD/PYTD alignment
- `Ops Day of Fiscal Year Number` (1…365/371) ← optional but helpful for PY/PYTD alignment

**Base metric placeholder**
```DAX
AC = [AC]
```

**YTD (fiscal)**
```DAX
YTD := 
IF (
    [ShowValueForDates],                                                            -- only for dates available
    VAR LastDayAvailable =  MAX ( dimDate [Ops Day of Fiscal Year Number] )         -- max day of fiscal year
    VAR LastFiscalYearAvailable = MAX ( dimDate [Ops Fiscal Year Number] )          -- current fiscal year
    VAR Result =
        CALCULATE (
            [AC],                                                                   -- selected measure
            ALLEXCEPT ( dimDate, dimDate[Ops Day of Week] ),                        -- clear filters on date table, except day of week                  
            dimDate[Ops Day of Fiscal Year Number] <= LastDayAvailable,             -- same number of days from prior year, same time period
            dimDate[Ops Fiscal Year Number] = LastFiscalYearAvailable               -- current fiscal year
        )
    RETURN
        Result
)
```

**MTD (fiscal)**
MTD := 
IF (
    [ShowValueForDates],                                                                      -- only avilable dates
    VAR LastDayAvailable =  MAX ( dimDate[Day of Fiscal Month Number] )                       -- max day of fiscal month available
    VAR LastFiscalYearMonthAvailable = MAX ( dimDate[Fiscal Year Month Number] )              -- current fiscal month of fiscal year
    VAR Result =
        CALCULATE (
            [AC],                                                                             -- selected measure
            ALLEXCEPT ( dimDate, dimDate[Day of Week] ),
            dimDate [Day of Fiscal Month Number] <= LastDayAvailable,                         -- same days in month, same time period
            dimDate [Fiscal Year Month Number] = LastFiscalYearMonthAvailable                 -- current fiscal month of current fiscal year
        )
    RETURN
        Result
)
```

**PY (fiscal)** — align by day in fiscal year
```DAX
PY v2 :=
--calculation group
CALCULATE (
    SELECTEDMEASURE(),                                                                          -- measure, like [AC]
    FILTER (
        ALL ( dimDate ),                                                                        --clear filters on date table
        dimDate[Ops Fiscal Year Number] = VALUES ( dimDate[Ops Fiscal Year Number] ) - 1        -- prior fiscal year
            && CONTAINS(
                VALUES ( dimDate[Ops Day of Fiscal Year Number] ),                              -- table of day of fiscal year values
                dimDate[Ops Day of Fiscal Year Number] ,                                        -- source column
                dimDate[Ops Day of Fiscal Year Number]  )                                       -- target column, will match number of days between current and prior fiscal year
                && dimDate[DateWithPounds] = TRUE()                                             -- necessary, date with value
                && dimDate[DateWithLabor] = TRUE()                                              -- optional, date with value, as many as needed  
    )    
)
```

**PYTD (fiscal)** — align by day in fiscal year
```DAX
PYTD := 
IF (
    [ShowValueForDates],                                                                    -- only available dates
    VAR PreviousFiscalYear = MAX ( dimDate[Ops Fiscal Year Number] ) - 1                    -- prior year
    VAR LastDayOfFiscalYearAvailable =
        CALCULATE (
            MAX ( dimDate[Ops Day of Fiscal Year Number] ),                                 -- max day of current fiscal year available
            REMOVEFILTERS (                                                  
                dimDate[Ops Day of Week],                                                   -- removes outside filters on day of week and day of week number                 
                dimDate[Ops Day of Week Number]                                  
            ),
            dimDate[DateWithValue] = TRUE                                                   -- only dates with values
        )
    VAR Result =
        CALCULATE (
            [AC],                                                                           -- selected measure
            ALLEXCEPT ( dimDate, dimDate[Ops Day of Week] ),                                -- removes filters except for day of week
            dimDate[Ops Fiscal Year Number] = PreviousFiscalYear,                           -- prior fiscal year
            dimDate[Ops Day of Fiscal Year Number] <= LastDayOfFiscalYearAvailable,         -- same dates, same time period
            dimDate[DateWithValue] = TRUE                                                   -- only dates with values
        )
    RETURN
        Result
)
```

**Δ vs PY and % vs PY (fiscal)**
```DAX
ΔPY := [AC] - [PY]
ΔPY Pct := DIVIDE( [ΔPY], [PY] )
```

## Blank and divide-by-zero handling

Use `DIVIDE(x, y)` rather than `x / y` to avoid errors and control blanks.

```DAX
Efficiency Pct = DIVIDE( [AC Production LBS], [AC Labor Hours] )
```

Return blank for missing context unless business needs dictate a default.

---

## Iterators and row context

- Use iterators (`SUMX`, `AVERAGEX`, `MAXX`, `MINX`) only when necessary.
- Summarize at the **grain you need** before iterating.

```DAX
AC Production LBS (Selected Days Avg) =
AVERAGEX(
    SUMMARIZE( dimDate, dimDate[Date] ),
    [AC Production LBS]
)
```

---

## Common templates

**Base metric**
```DAX
AC Production LBS = SUM( factProduction[Produced LBS] )
```

**Filtered base**
```DAX
AC Production LBS (RTE only) =
CALCULATE( [AC Production LBS], dimProductionArea[Type] = "RTE" )
```

**Ratio / percentage**
```DAX
AC Labor $ per LBS = DIVIDE( [AC Labor $], [AC Production LBS] )
```

**Goal variance**
```DAX
AC Efficiency ΔGoal Pct =
DIVIDE( [AC Efficiency Pct] - [Efficiency Goal Pct], [Efficiency Goal Pct] )
```

**Top-N helper**
```DAX
Top N Flag =
VAR n = 10
RETURN
    IF( RANKX( ALL( dimProduct ), [AC Production LBS] ) <= n, 1, 0 )
```

---

## Performance tips

- Prefer **base measures** + **CALCULATE** over repeating expressions.
- Avoid iterators over large tables when a **simple aggregation** folds.
- Push filters to the storage engine: filter early in visuals or via model relationships.
- Keep row-level calculations in Power Query when possible.

---

## Testing and validation

- Create small **QA measures** (counts, min/max, sample values) and hide them later.
- Validate totals and subtotals behave as expected under different filter scenarios.
- For PY/YTD logic, test edge dates (year boundaries, partial months).

```DAX
QA Count Rows factProduction = COUNTROWS( factProduction )
```

---

## Do/Don't quick list

**Do**
- Use `DIVIDE` for safe ratios.
- Use `REMOVEFILTERS` intentionally.
- Keep measure names concise and consistent.
- Add a one-line description per measure in TMDL.

**Don't**
- Repeat long expressions inline; extract to base measures.
- Overuse `ALLEXCEPT` or `ALL` without clear intent.
- Return 0 when blank communicates "no data" better.

---

Adopt this guide for new measures and refactors. When in doubt, choose clarity and consistency.

