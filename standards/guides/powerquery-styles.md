[powerquery_style.md](https://github.com/user-attachments/files/21976505/powerquery_style.md)
# Power Query Style Guide (camelCase + commented steps)

This guide standardizes how we write Power Query (M) across BI projects. It focuses on:

- **camelCase** step names
- **clear comments** for each step or grouped sections
- predictable **sectioning** for filtering, transformations, and adding columns

Use this to make queries easy to read, maintain, and optimize.

---

## Core principles

1. **Be consistent**. Prefer the same patterns across all queries.
2. **Name for intent**. Step names should start with a verb and describe what changes.
3. **Comment for context**. Put a short comment above each step or a block comment above a group of related steps.
4. **Keep steps atomic**. One clear action per step when possible.
5. **Preserve query folding**. Prefer folding-friendly operations early (filters, column selection, types) before non-folding steps.
6. **Readable over clever**. Favor clarity even if it adds a line or two.

---

## Step naming convention

- **Format**: `camelCase`
- **Structure**: `<verb><Noun><Qualifier>`
- **Examples**:
  - `filteredRowsByFolderPath`
  - `invokedCustomFunction`
  - `renamedDataColumns`
  - `changedColumnTypes`
  - `removedOtherColumns`
  - `expandedTableColumn`

**Do**
- Use action-first verbs: `filtered`, `sorted`, `grouped`, `merged`, `added`, `renamed`, `changed`, `expanded`, `removed`, `reordered`, `buffered`.
- Keep names short but specific: `filteredRowsByDateRange` not `filteredRows2`.

**Avoid**
- Spaces or `#"..."` step names.
- Numeric suffixes like `step1`, `step2` unless there is a real sequence requirement.

> Columns with spaces or special characters should be wrapped as `#"Column Name"` inside brackets, for example: `[#"Qty-Stk"]`.

---

## Commenting standard

- Put a **single-line comment** `//` **above** each step to describe what and why.
- For a series of related steps, add a **section header** comment.

**Section header pattern**
```m
// === Filtering ===
```

**Good step comment**
```m
// Keep only files in the Production Journal folder
filteredRowsByFolderPath = Table.SelectRows(source, each [Folder Path] = "...")
```

**Avoid** comments that restate the code without intent. Explain the purpose, not the syntax.

---

## Recommended section order

Use sections to group related steps. Not every query needs every section.

1. **Source**
2. **Filtering** (folders, file names, date ranges, null checks)
3. **Shaping** (select, remove, reorder columns)
4. **Transformation** (rename columns, change types, expand tables)
5. **Adding columns** (derived columns, conditional logic)
6. **Joins and aggregations** (merge, group, summarize)
7. **Validation and cleanup** (remove errors, deduplicate)
8. **Performance** (buffer only if needed)
9. **Output** (final step name matches the last meaningful action)

---

## Filtering guidelines

- Filter early to reduce data size and improve folding.
- Prefer `Table.SelectRows` with simple predicates that fold.
- Chain filters with descriptive step names rather than one giant predicate.

Examples
```m
// Keep only current fiscal year
filteredRowsByFiscalYear = Table.SelectRows(source, each [Fiscal Year] = 2025)

// Remove null or blank dates
filteredRowsByDateNotBlank = Table.SelectRows(previousStep, each [Production Date] <> null and [Production Date] <> "")
```

---

## Transformation guidelines

- **Rename columns** early so later steps are easier to read.
- **Change types** as soon as columns stabilize. Types can affect folding and function behavior.
- **Expand table columns** using schema from a sample file when invoking file transforms.

Examples
```m
// Rename short or cryptic columns
renamedDataColumns = Table.RenameColumns(prev, {{"Btch Dt", "Batch Date"}, {"Shift", "Shift Cd"}})

// Set numeric and date types
changedColumnTypes = Table.TransformColumnTypes(renamedDataColumns, {{"Batch Date", type date}, {"Committed Weight", type number}})
```

---

## Adding columns

- Name derived columns for the business concept, not the formula.
- Prefer `each` with a single expression. If logic is complex, split into helper steps.

Examples
```m
// Add previous day flag for 12am–4am starts
addedStartDateAdjusted = Table.AddColumn(prev, "Start Date Adjusted", each if Time.From([Start Tmstp]) < #time(4,0,0) then Date.AddDays(Date.From([Start Tmstp]), -1) else Date.From([Start Tmstp]), type date)
```

---

## Sorting, buffering, and performance

- **Sort** only when needed and as late as possible.
- **Buffer** sparingly. `Table.Buffer` can help stabilize order or avoid re-evaluation, but it can break folding and increase memory usage. Comment why you used it.

Example
```m
// Buffer to stabilize order before distinct (breaks folding; required for deterministic output)
bufferedTable = Table.Buffer(sortedRows)
```

---

## Joins and aggregations

- Use descriptive names for join keys and results.
- After `Table.Join` or `Table.NestedJoin`, immediately expand only the needed columns.
- For aggregations, keep the grouping intent obvious.

Example
```m
// Join to bring in product attributes
joinedProducts = Table.NestedJoin(prev, {"Product Cd"}, dimProducts, {"Product Cd"}, "products", JoinKind.LeftOuter)
expandedProducts = Table.ExpandTableColumn(joinedProducts, "products", {"Category", "UOM"})
```

---

## Error handling and validation

- Remove or handle error rows near the end, unless they block folding.
- Add explicit null checks for critical columns.
- Consider a final row count or checksum for QA.

Example
```m
// Remove rows with errors in Quantity
removedErrorRows = Table.RemoveRowsWithErrors(prev, {"Quantity"})
```

---

## Formatting rules

- Indent with 4 spaces.
- Place a comma at the end of each step except the last before `in`.
- Align list items in functions like `Table.TransformColumnTypes` one per line.
- Keep lines under ~120 characters when possible.

---

## Naming patterns for common steps

- `filteredRowsBy...`
- `removedColumns` / `removedOtherColumns`
- `renamedDataColumns`
- `changedColumnTypes`
- `expandedTableColumn`
- `invokedCustomFunction`
- `bufferedTable`
- `removedDuplicates`

---

## Checklist before committing

- Step names use camelCase and clear verbs.
- Each step or section has a helpful comment.
- Filters applied early and foldable.
- Types set for all important columns.
- No unused columns or debug steps.
- Buffering only where justified and commented.
- Final step name matches the last meaningful action.

---

## Versioning and diffs

- Keep changes small and focused.
- If you rename a step, update all references in later steps.
- In PRs, call out any non-folding operations and why they are needed.

---

## Example: minimal before and after

**Before**
```m
let
    Source = SharePoint.Files(url, [ApiVersion = 15]),
    #"Filtered rows 1" = Table.SelectRows(Source, each [Folder Path] = path),
    #"Filtered rows 2" = Table.SelectRows(#"Filtered rows 1", each [Attributes]?[Hidden]? <> true)
in
    #"Filtered rows 2"
```

**After**
```m
let
    // Get files and limit to the target folder; exclude hidden files
    source = SharePoint.Files(url, [ApiVersion = 15]),
    filteredRowsByFolderPath = Table.SelectRows(source, each [Folder Path] = path),
    filteredOutHiddenFiles = Table.SelectRows(filteredRowsByFolderPath, each [Attributes]?[Hidden]? <> true)
in
    filteredOutHiddenFiles
```

---

Adopt this guide for new queries and refactors. When in doubt, favor clarity and comments.

