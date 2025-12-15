# Analytics Naming & Titling Standards
ASCII-compliant measure names and IBCS-aligned chart titles/headers (with deltas & variance)

> **Scope:** This guide standardizes how we name measures (ASCII only) and how we title charts and label table columns following IBCS. It covers units, baselines, percent vs percentage-point change, temporal patterns, and common naming conventions.

---

## 1) Measure names (ASCII only)

**Rules**
- Use plain ASCII characters. No symbols in names: no Δ, %, $, /, °.
- Use clear words. Prefer business-friendly names; reserve abbreviations for when space is tight.
- Use `per` in names; reserve `/` for UI labels.
- Put the metric first, then the change type, optional scope, then the baseline.
- **Target length:** 50 characters or fewer; **Maximum:** 80 characters (BI tool practical limit)

**Pattern**
```
<Metric> <DeltaType?> <Scope?> <Baseline?>
```

### Metric Components

- **Metric:** Efficiency Pct, Shrink Pct, Saleable Lb, Production Hours, Committed Cost per Lb, etc.

- **DeltaType (optional):**
  - `Variance` → absolute change (Current − Baseline) in same unit as metric
    - Always computed as (Actual − Baseline); can be positive or negative
    - For favorable/unfavorable analysis, use conditional formatting or create separate calculated measures
  - `Delta Pct` → relative percent change = (Current − Baseline) / Baseline × 100
  - `Delta PctPt` → percentage-point change = Current − Baseline (for percent metrics only)

- **Scope (optional):** MTD, QTD, YTD, FYTD, R13W, R12M, Cumulative, Rolling (see Section 3)

- **Baseline:** vsPY, vsPL, vsTarget, vsBudget, vsFC

**Examples**
- `Efficiency Pct`
- `Efficiency Pct PY`
- `Efficiency Pct Delta Pct vsPY`
- `Efficiency Pct Delta PctPt vsPY`
- `Saleable Lb per Labor Hour`
- `Saleable Lb Variance vsPL`
- `Committed Cost per Lb`
- `Committed Cost Variance vsBudget`
- `Sales Cumulative YTD`
- `Efficiency Pct Rolling 13W Avg`

**Units inside names**
- Use singular tokens in names: `Lb`, `Hour`, `Unit`.
- Preferred names: `Saleable Lb`, `Production Hours`, `Committed Cost per Lb`.
- Avoid doubled units in names like `Lb/LH` or `$/lb` — move those to labels.

---

## 2) Rates, ratios, and percentages

### Definitions

- **Rate:** Metric per time or per unit of input
  - Use `per`: `Saleable Lb per Labor Hour`, `Committed Cost per Lb`, `Units per Shift`
  
- **Ratio:** Dimensionless comparison of two similar metrics
  - Use `to`: `Actual to Plan Ratio`, `Current to Target Ratio`
  
- **Percentage:** Part of whole, always use `Pct` in measure names
  - `Efficiency Pct`, `Shrink Pct`, `Yield Pct`
  - Format as percentage in visuals (e.g., `34.5%`)

### Percent Change vs Percentage-Point Change

- **Percent Change (`Delta Pct`):** Relative change
  - Example: Efficiency goes from 80% to 84% = +5% change
  - Formula: (84% − 80%) / 80% = 0.05 = 5%
  - Measure name: `Efficiency Pct Delta Pct vsPY`

- **Percentage-Point Change (`Delta PctPt`):** Absolute change for percentage metrics
  - Example: Efficiency goes from 80% to 84% = +4 percentage points
  - Formula: 84% − 80% = 4 pp
  - Measure name: `Efficiency Pct Delta PctPt vsPY`

---

## 3) Baselines, time scopes, and temporal naming

### Standard Baseline Tokens
All baseline tokens use camelCase after `vs`:
- `vsPY` — versus Prior Year
- `vsPL` — versus Plan
- `vsTarget` — versus Target
- `vsBudget` — versus Budget
- `vsFC` — versus Forecast

### Time Scopes
- `MTD` — Month-to-Date
- `QTD` — Quarter-to-Date
- `YTD` — Year-to-Date (calendar year)
- `FYTD` — Fiscal Year-to-Date
- `R13W` — Rolling 13 Weeks
- `R12M` — Rolling 12 Months
- `R4Q` — Rolling 4 Quarters

### Cumulative and Rolling Measures
Pattern: `<Metric> <CumulativeType> <Scope>`

- `Cumulative` — Running total within a defined period
  - `Sales Cumulative YTD`
  - `Saleable Lb Cumulative FYTD`
  
- `Rolling` — Moving window average or sum
  - `Efficiency Pct Rolling 13W Avg`
  - `Sales Rolling 12M Sum`

### Time Period Naming in Measures
When measures include specific time periods:

- **Fiscal year:** `FY2025`, `FY24`
- **Calendar year:** `CY2025` or `2025`
- **Quarters:** `Q1`, `Q2`, `Q3`, `Q4`
- **Months:** Use three-letter codes: `Jan`, `Feb`, `Mar`, etc.

**Examples:**
- `Sales MTD FY2025`
- `Efficiency Pct Q3 FY24`
- `Committed Cost YTD vsPY`

### Date Range Formatting (for visual labels only)
- Short ranges: `Jan-Mar 2025`, `Sep-Dec 2024`
- Abbreviated: `Jan-2.. Mar-7 2025` (if showing partial months)
- Full fiscal: `FY2024 Q1-Q4`, `FYTD through Nov 2024`

**Examples of Complete Measure Names:**
- `Saleable Lb Variance vsPY YTD`
- `Efficiency Pct Delta Pct vsTarget QTD`
- `Sales Cumulative FYTD vsPL`
- `Committed Cost Rolling 13W Avg vsBudget`

---

## 4) Dimensional scope in names

When measures are pre-aggregated or filtered to specific dimensions:

**Pattern:** `<Metric> by <Dimension>` or `<Metric> <Dimension> Level`

**Examples:**
- `Sales by Plant`
- `Efficiency Pct Plant Level`
- `Committed Cost by Product Line`
- `Saleable Lb by Region`

**Important:** Avoid embedding specific dimension members in measure names (e.g., NOT `Sales Chicago Plant`). Keep dimension filtering dynamic or create separate filtered measures with clear scope indicators.

---

## 5) Forecast, scenario, and flag measures

### Forecast Measures
- Use `FC` suffix for forecast values: `Sales FC`, `Efficiency Pct FC`
- For multiple forecast scenarios:
  - `Sales FC Optimistic`
  - `Sales FC Conservative`
  - `Sales FC Base`
- Forecast variance: `Sales Variance vsFC`

### Boolean and Flag Measures
- Prefix with `Is` or `Has`: 
  - `Is Saleable`
  - `Has Shortage`
  - `Is On Time`
- Suffix with `Flag` if clearer:
  - `Overtime Flag`
  - `Quality Alert Flag`
  - `Backorder Flag`

### Index and Score Measures
- Suffix with `Index` or `Score`
- Examples:
  - `Quality Index`
  - `Performance Score`
  - `Efficiency Index`
  - `Customer Satisfaction Score`

---

## 6) Measure type prefixes (optional)

For complex data models with many calculated measures, consider optional prefixes:

- `calc_` — calculated measures (e.g., `calc_Saleable Lb per Labor Hour`)
- `agg_` — pre-aggregated measures (e.g., `agg_Sales Plant Level`)
- No prefix — base measures from fact tables (e.g., `Saleable Lb`, `Production Hours`)

**Note:** Use prefixes sparingly. Only implement if your model complexity requires clear distinction between measure types.

---

## 7) Units and formatting

### General Rules
- Units belong in **visual labels and headers**. Keep measure names clean.
- Weight: `lb` (lowercase, not pluralized)
- Time: `h` for hours, `min` for minutes
- Percentage point: `pp`
- Scaling markers: `K`, `M`, `B` for thousands, millions, billions

### Format Strings and Labels

| Measure Type | Measure Name | Format String | Visual Label |
|--------------|--------------|---------------|--------------|
| Percentage | `Efficiency Pct` | `0.0%` | `Efficiency (%)` |
| Percentage-point change | `Efficiency Pct Delta PctPt vsPY` | `0.0 "pp"` | `Δ vs PY (pp)` |
| Currency | `Committed Cost` | `"$"#,0` | `Cost ($)` |
| Weight | `Saleable Lb` | `#,0` | `Saleable (lb)` |
| Weight scaled | `Saleable Lb` | `#,0,"K"` | `Saleable (K lb)` |
| Time | `Production Hours` | `#,0` | `Production (h)` |
| Rate | `Saleable Lb per Labor Hour` | `0.0` | `Productivity (lb/h)` |

**Currency Notes:**
- Only add currency code to measure name if multi-currency is truly used (e.g., `Committed Cost USD`, `Sales EUR`)
- For single-currency models, keep currency in format strings and labels only

---

## 8) IBCS scenario codes and abbreviations

Use these standard abbreviations in visual headers, context lines, and table columns (NOT in measure names):

| Code | Meaning |
|------|---------|
| **AC** | Actual |
| **PY** | Prior Year |
| **PL** | Plan / Forecast |
| **BU** | Budget |
| **FC** | Forecast |
| **Δ** | Delta / Variance |
| **ΔAC** | Actual Variance (from Plan) |
| **ΔPY** | Prior Year Variance |

**Example context line:**
```
Efficiency (%) | AC, PY, PL | Q1-Q4 FY2025 | by product line
```

---

## 9) IBCS chart titling

IBCS favors a **message title** (the takeaway) with a **context line** (metric, unit, scenario, time, breakdown).

### Message + Context Pattern

```
Title (message): <What changed or matters — the insight>
Context: <Metric and unit> | <Scenario codes> | <Time window> | <Breakdown/sort>
```

### Examples

**Message-driven (preferred for insights):**
```
Title: Efficiency up vs PY; Q3 strongest performance YTD
Context: Production efficiency (%) | AC vs PY | Jan-Mar 2025 | by fiscal quarter with monthly detail
```

**Structural (neutral — when no clear message):**
```
Title: Production metrics by product
Context: Efficiency (%), saleable (lb), capacity (lb), hours (h) | AC | Sep-Dec 2024 | sorted by saleable ↓
```

**Multi-metric overview:**
```
Title: Production output and efficiency by product line
Context: Saleable (lb), efficiency (%), capacity utilization (%) | AC, PL | FY2024 | sorted by saleable ↓
```

### Title Guidelines

**Do:**
- Put units right after metric names in parentheses: `Efficiency (%)`, `Saleable (lb)`, `Cost ($)`
- Use standard scenario codes: AC, PY, PL, BU, FC
- Include sort direction if relevant: `sorted by saleable ↓`
- Keep the message concise and insight-focused

**Don't:**
- Don't write "in PCT" or "in LBS". Use `(%)` or `(lb)` instead
- Don't mix message and structure in one line
- Don't include measure technical names in titles (use business-friendly terms)

---

## 10) Table and matrix headers (IBCS)

### When to Put Units in Headers

- **Same unit across all columns:** Place unit in the table title
  - Table title: `Production efficiency (%) by plant`
  - Columns: `AC | PY | Δ vs PY | PL | Δ vs PL`

- **Different units per column:** Put units in each column header
  - Columns: `Saleable (M lb) | Efficiency (%) | Hours (K h) | Cost ($)`

### Header Patterns

| Measure Type | Column Header |
|--------------|---------------|
| Percentage | `Efficiency (%)` |
| Percent change | `Δ% vs PL` or `ΔAC (%)` |
| Percentage-point change | `Δ vs PL (pp)` |
| Weight with scaling | `Saleable (M lb)`, `Capacity (K lb)` |
| Time with scaling | `Production (K h)` |
| Currency | `Cost ($)` or `Cost (K $)` |
| Rate | `Productivity (lb/h)` |

### Multi-Comparison Table Structure

When showing multiple baseline comparisons in one table:

**Option 1: Separate delta columns**
| Plant | AC | PY | Δ vs PY | PL | Δ vs PL |
|-------|----|----|---------|----|---------| 

**Option 2: With units in table title**
```
Table title: Production efficiency (%) by plant
Columns: AC | PY | Δ vs PY (pp) | PL | Δ vs PL (pp) | Sort ↓
```

### Sorting and Scaling

- Show sort direction on the sorted column: `Saleable (M lb) ↓` or `Efficiency (%) ↑`
- Indicate K/M/B in the header, not in each cell
- If a metric name already contains the unit word, use either:
  - `Production (K h)`, or
  - `Production hours (K)`

---

## 11) DAX templates (copy-ready)

### Baseline Calculation (PY, aligned to fiscal day)

```DAX
Efficiency Pct :=
DIVIDE( [Good Units], [Total Units] )

Efficiency Pct PY :=
CALCULATE (
    [Efficiency Pct],
    FILTER (
        ALL ( dimDate ),
        dimDate[Ops Fiscal Year Number] = VALUES ( dimDate[Ops Fiscal Year Number] ) - 1        
            && CONTAINS(
                VALUES ( dimDate[Ops Day of Fiscal Year Number] ),
                dimDate[Ops Day of Fiscal Year Number],                                        
                dimDate[Ops Day of Fiscal Year Number] )
            && dimDate[DateWithPounds] = TRUE()
    )    
)
```

### Variance (Absolute Change)

```DAX
Committed Cost Variance vsPL :=
[Committed Cost] - [Committed Cost PL]
```

### Percent Change (Relative)

```DAX
Efficiency Pct Delta Pct vsPY :=
VAR curr = [Efficiency Pct]
VAR base = [Efficiency Pct PY]
VAR result = DIVIDE( curr - base, base )
RETURN 
    IF(
        ISBLANK(base) || base = 0,
        BLANK(),  -- Return blank if no valid baseline
        result
    )
```

### Percentage-Point Change

```DAX
Efficiency Pct Delta PctPt vsPY :=
VAR curr = [Efficiency Pct]
VAR base = [Efficiency Pct PY]
RETURN 
    IF(
        ISBLANK(base),
        BLANK(),
        curr - base
    )
```

### Unit Rate

```DAX
Saleable Lb per Labor Hour :=
DIVIDE( [Pounds Produced Saleable], [Labor Hours] )
```

### Cumulative Measure (YTD)

```DAX
Sales Cumulative YTD :=
CALCULATE(
    [Sales],
    DATESYTD( dimDate[Date] )
)
```

### Rolling Average (13 weeks)

```DAX
Efficiency Pct Rolling 13W Avg :=
CALCULATE(
    AVERAGE( [Efficiency Pct] ),
    DATESINPERIOD(
        dimDate[Date],
        MAX( dimDate[Date] ),
        -13,
        WEEK
    )
)
```

### DAX Best Practices

- Use `DIVIDE` to avoid divide-by-zero errors
- Always handle BLANK values in delta calculations
- Keep baseline logic consistent across related measures
- Test edge cases: missing data, zero denominators, partial periods
- Use `VAR` for readability and performance

---

## 12) Naming quick-reference

### ✅ Good Examples

- `Committed Cost per Lb`
- `Committed Cost Variance vsPL`
- `Efficiency Pct Delta Pct vsPY`
- `Efficiency Pct Delta PctPt vsPY`
- `Saleable Lb per Labor Hour`
- `Production Hours`
- `Sales Cumulative FYTD`
- `Efficiency Pct Rolling 13W Avg`
- `Quality Index`
- `Is Saleable`
- `Sales FC Base`

### ❌ Avoid These

| Bad Example | Why to Avoid | Better Alternative |
|-------------|--------------|-------------------|
| `ΔEfficiency%` | Non-ASCII symbols | `Efficiency Pct Delta Pct vsPY` |
| `Saleable Lbs/LH` | Mixed units, plural, slash | `Saleable Lb per Labor Hour` |
| `Committed Lb USD` | Confusing unit order | `Committed Cost per Lb` |
| `Gain (Loss) on Shrinkage` | Parenthetical alternatives; create separate measures if needed | `Shrink Variance vsPL` |
| `Cost/lb/lean %` | Ambiguous, multiple slashes | `Lean Cost per Lb` |
| `Efficiency (%)` | Symbols in measure name | `Efficiency Pct` (use symbols in labels only) |
| `Sales vs. PY` | Period in name | `Sales vsPY` |
| `2024 Sales` | Year embedded | `Sales FY2024` or use time filters |

---

## 13) Labels vs code: who shows what

### Measure Names (Code/DAX)
- ASCII only
- No symbols: no Δ, %, $, /, °
- Use words: `per`, `Pct`, `PctPt`, `Variance`, `Delta`
- Example: `Efficiency Pct Delta Pct vsPY`

### Visual Labels and Titles
- May use symbols and slashes: `%`, `$`, `Δ`, `/`, `pp`
- Use IBCS scenario codes: AC, PY, PL, BU, FC
- Example: `Δ% vs PY`, `Efficiency (%)`, `USD/lb`, `Δ vs PL (pp)`, `lb/h`

### Transformation Examples

| Measure Name (Code) | Visual Label/Header |
|---------------------|---------------------|
| `Efficiency Pct` | `Efficiency (%)` |
| `Efficiency Pct Delta Pct vsPY` | `Δ% vs PY` or `ΔAC (%)` |
| `Efficiency Pct Delta PctPt vsPY` | `Δ vs PY (pp)` |
| `Saleable Lb per Labor Hour` | `Productivity (lb/h)` |
| `Committed Cost per Lb` | `Cost ($/lb)` |
| `Sales Variance vsPL` | `Δ vs PL ($)` |
| `Production Hours` | `Production (h)` |

---

## 14) Governance checklist

Before creating or approving a new measure, verify:

- [ ] Measure name uses ASCII-only characters
- [ ] Length is under 50 characters (or 80 maximum)
- [ ] Pattern follows: `<Metric> <DeltaType?> <Scope?> <Baseline?>`
- [ ] Units are singular in name (`Lb`, not `Lbs`)
- [ ] Baseline token is camelCase after `vs` (e.g., `vsPY`)
- [ ] Symbols are only in visual labels, not measure names
- [ ] `Pct` is used for percentages, not `%` or `Percent`
- [ ] `per` is used for rates, not `/`
- [ ] DAX includes error handling (BLANK for invalid baselines)
- [ ] IBCS scenario codes (AC, PY, PL) are used in visuals only
- [ ] Chart titles separate message from context
- [ ] Table headers include units and sort indicators where needed

---

## 15) Additional resources

- **IBCS Standards:** International Business Communication Standards (https://www.ibcs.com)
- **DAX Patterns:** https://www.daxpatterns.com
- **Power BI Best Practices:** Microsoft documentation on data modeling
- **Internal contacts:** Paul Tredrea

---

**Document Version:** 2.0  
**Last Updated:** December 2024  
**Maintained By:** Data Governance Team
