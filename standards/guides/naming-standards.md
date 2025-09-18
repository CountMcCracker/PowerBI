# Analytics Naming & Titling Standards
ASCII-compliant measure names and IBCS-aligned chart titles/headers (with deltas & variance)

> **Scope:** This guide standardizes how we name measures (ASCII only) and how we title charts and label table columns following IBCS. It covers units, baselines, percent vs percentage-point change, and common patterns.

---

## 1) Measure names (ASCII only)

**Rules**
- Use plain ASCII characters. No symbols in names: no Δ, %, $, /, °.
- Use clear words. Prefer business-friendly names; reserve abbreviations for when space is tight.
- Use `per` in names; reserve `/` for UI labels.
- Put the metric first, then the change type, optional scope, then the baseline.

**Pattern**
```
<Metric> <DeltaType?> <Scope?> vs<Baseline?>
```
- **Metric:** Efficiency Pct, Shrink Pct, Saleable Pounds, Production Hours, Committed Cost per Lb, etc.
- **DeltaType (optional):**
  - `Variance`  → absolute change in same unit as metric (totals and unit rates)
  - `Delta Pct` → relative percent change = (Current − Base) / Base
  - `Delta PctPt` → percentage-point change = Current − Base (for percent metrics)
- **Scope (optional):** MTD, QTD, YTD, R13W, FYTD, etc.
- **Baseline:** PY, PL, Target, Budget.

**Examples**
- `Efficiency Pct`
- `Efficiency Pct PY`
- `Efficiency Pct Delta Pct vsPY`
- `Efficiency Pct Delta PctPt vsPY`
- `Saleable Lb per Labor Hour`
- `Saleable Lb Variance vsPL`
- `Committed Cost per Lb`
- `Committed Cost Variance vsBudget`

**Units inside names**
- Use singular tokens in names: `Lb`, `Hour`.
- Preferred names: `Saleable Lb`, `Production Hours`, `Committed Cost per Lb`.
- Avoid doubled units in names like `Lb/LH` or `$/lb` — move those to labels.

---

## 2) Units and formatting

**General**
- Units belong in **visual labels and headers**. Keep measure names clean.
- Weight: `lb` (lowercase, not pluralized).
- Time: `h` for hours.
- Percentage point: `pp`.
- Scaling markers: `K`, `M`, `B`.

**Labels and format strings**
- Percent measures: format as `%` (for example `0.0%`).
- Percentage-point change: format as `0.0 "pp"`.
- Currency: format via model string (for example `"$#,0"`). Only add currency code to the name if multi-currency is truly used (for example `Committed Cost USD`).

---

## 3) Baselines and scopes

**Standard baseline tokens**
- `vsPY`, `vsPL`, `vsTarget`, `vsBudget`.

**Time scopes**
- `MTD`, `QTD`, `YTD`, `FYTD`, `R13W` (Rolling 13 Weeks), `R12M` (Rolling 12 Months).

**Examples**
- `Saleable Lb Variance vsPY YTD`
- `Efficiency Pct Delta Pct vsTarget QTD`

---

## 4) DAX templates (copy-ready)

**Baseline example (PY, align to complete fiscal day)**
```DAX
Efficiency Pct :=
DIVIDE( [Good Units], [Total Units] )

Efficiency Pct PY :=
VAR currFY  = MAX( dimDate[Ops Fiscal Year Number] )
VAR currDay = MAX( dimDate[Ops Day of Fiscal Year Number] )
RETURN
CALCULATE(
    [Efficiency Pct],
    REMOVEFILTERS( dimDate ),
    dimDate[Ops Fiscal Year Number] = currFY - 1,
    dimDate[Ops Day of Fiscal Year Number] <= currDay
)
```

**Variance (absolute change, same unit as metric)**
```DAX
Committed Cost Variance vsPL :=
[Committed Cost] - [Committed Cost PL]
```

**Percent change (relative)**
```DAX
Efficiency Pct Delta Pct vsPY :=
VAR curr = [Efficiency Pct]
VAR base = [Efficiency Pct PY]
RETURN DIVIDE( curr - base, base )
```

**Percentage-point change (for percent metrics)**
```DAX
Efficiency Pct Delta PctPt vsPY :=
[Efficiency Pct] - [Efficiency Pct PY]
```

**Unit rate**
```DAX
Saleable Pounds per Labor Hour :=
DIVIDE( [Pounds Produced Saleable], [Labor Hours] )
```

**Tips**
- Use `DIVIDE` to avoid divide-by-zero.
- Keep baseline logic consistent across measures.

---

## 5) IBCS chart titling

IBCS favors a **message title** (the takeaway) with a **context line** (metric, unit, scenario, time, breakdown).

**Message + context pattern**
```
Title (message): <What changed or matters>
Context: <Metric and unit> | <Scenario/baseline> | <Time window> | <Breakdown>
```

**Examples**
- **Title:** `Efficiency up vs PY; Q3 strongest YTD`
  **Context:** `Production efficiency (%) | AC vs PY | FY2025 YTD | by fiscal quarter with monthly detail`

- **Title:** `Products ranked by saleable (lb) (↓)`
  **Context:** `Efficiency (%) • Saleable (lb) • Capacity (lb) • Production hours | FY2025 YTD | by product`

**Neutral single-line (when no message)**
- `Production efficiency (%), saleable (lb), capacity (lb), and production hours by product (sorted by saleable (lb) descending)`

**Do**
- Put units right after metric names in parentheses in the context line or title.
- Use sentence case or title case consistently — choose one org-wide.
- Indicate ranking either by wording in the title (`ranked by…`) or by an arrow on the sorted column header.

**Don’t**
- Don’t write “in PCT”. Use `(%).`
- Don’t embed currency or percent symbols in **measure names**.

---

## 6) Table and matrix headers (IBCS)

**When to put units in headers**
- If all numeric columns share the same unit and scale, you may place the unit in the subtitle.
- If columns use different units or scales, put units and scale on **each** column header.

**Header patterns**
- Percent: `Efficiency (%)`
- Percent change: `Δ% vs PL`
- Percentage-point change: `Δ vs PL (pp)`
- Weight with scaling: `Saleable (M lb)`, `Capacity (K lb)`
- Time with scaling: `Production (K h)`

**Sorting & scaling**
- Show sort direction on the sorted column: `Saleable (M lb) ↓`.
- Indicate K/M/B in the header, not in each cell.

**If a metric name already contains the unit word**
- Use either `Production (K h)` or keep the word and only show scale: `Production hours (K)`.

---

## 7) Naming quick-reference

**Good**
- `Committed Cost per Lb`
- `Committed Cost Variance vsPL`
- `Efficiency Pct Delta Pct vsPY`
- `Efficiency Pct Delta PctPt vsPY`
- `Saleable Pounds per Labor Hour`
- `Production Hours`

**Avoid**
- `ΔEfficiency%`
- `Saleable Lbs/LH`
- `Committed Lb USD`
- `Gain (Loss) on Shrinkage`
- `Cost/lb/lean %` (ambiguous)

---

## 8) Labels vs code: who shows what

**Measure names (code)**
- ASCII only. No symbols. Use words: `per`, `Pct`, `PctPt`, `Variance`.

**Visual labels and titles**
- May use symbols and slashes: `%`, `$`, `Δ`, `/`, `pp`.
- Examples: `USD/lb`, `Δ% vs PY`, `Δ vs PL (pp)`, `lb/h`.

---

## 9) Favorability and color rules

Document the sign convention once and apply it everywhere.
- Example: For Efficiency, higher is better — positive `Delta Pct` is favorable.
- For Shrink Pct, lower is better — negative `Delta Pct` is favorable.

**Optional helper measures:**
```DAX
IsImproving (Efficiency vsPY) :=
VAR c = [Efficiency Pct Delta Pct vsPY]
RETURN IF( NOT ISBLANK(c) && c >= 0, TRUE(), FALSE() )

IsImproving (Shrink Pct vsPY) :=
VAR c = [Shrink Pct Delta Pct vsPY]
RETURN IF( NOT ISBLANK(c) && c <= 0, TRUE(), FALSE() )
```
---

## 10) Pull-request checklist

- Name is ASCII, follows pattern `<Metric> <DeltaType?> <Scope?> vs<Baseline?>`.
- Display folder and format string are set.
- Units appear in labels/headers, not in the name.
- Uses `DIVIDE` with consistent baseline logic.
- Percent vs percentage-point changes are correctly distinguished.
- Chart titles use message + context, or neutral single-line if no message.
- Table headers show units and scaling where needed.
- Sorting and scaling are indicated in headers or title.
- Favorability and color map align with sign convention.

---

## 11) Copy snippets for quick reuse

**Percent change vs PY**
```DAX
<Metric> Delta Pct vsPY :=
VAR curr = [<Metric>]
VAR base = [<Metric> PY]
RETURN DIVIDE( curr - base, base )
```

**Percentage-point change vs PY**
```DAX
<Metric> Delta PctPt vsPY :=
[<Metric>] - [<Metric> PY]
```

**Variance vs PL**
```DAX
<Metric> Variance vsPL :=
[<Metric>] - [<Metric> PL]
```

**Title + context template**
```
Title: <Message about what changed>
Context: <Metric 1 (unit)> • <Metric 2 (unit)> | <Scenario> | <Time> | by <Breakdown>
```
