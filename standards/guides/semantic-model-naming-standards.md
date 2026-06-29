# Semantic Model Standardization Standard (ASCII Naming Convention)

> **Revision note (v3):** Builds on v2. Adds object name length limits, resolves the `Distinct Count` vs `DistinctCount` question, defines empty/null naming and reserved words, and specifies casing for approved abbreviations.

## 1. Scope & Principles

This standard governs naming and structure of all objects in Power BI semantic models. All object names must use ASCII characters only (code points 32-126). No accented characters, em-dashes, smart quotes, or non-Latin glyphs.

**Core principles:**

- Consistency over preference - pick one pattern, apply everywhere.
- Names are for humans (report authors), so user-facing objects are readable; technical objects are terse.
- ASCII-only ensures portability across XMLA, TMDL, source control, and CI/CD tooling.

## 2. Allowed Character Set

| Allowed | Notes |
|---|---|
| `A-Z`, `a-z` | Letters |
| `0-9` | Digits, never as first character |
| Space ` ` | User-facing names only (measures, columns, tables) |
| Underscore `_` | Technical objects, calculation groups, parameters |
| Bracket `[ ]` | DAX reference only, not part of the name |

**Forbidden:** tabs, leading/trailing spaces, double spaces, and all characters outside ASCII 32-126. Forbidden symbols inside names: `, ; ' " / \ | * ? : < > = + . % # @ & ( ) { }`. (The `%` character is the sole exception, permitted only as a trailing suffix on measures per Section 3.)

**Length limits:** the engine permits up to 128 characters, but cap practical names well below that. Tables and measures: 50 characters. Columns: 40. Calculation items: 30. Names longer than the cap must be shortened or use an approved abbreviation (Section 7), never truncated mid-word. A name must never be only a reserved DAX keyword (e.g. `Date`, `Year`, `Month`, `Time`, `Filter`, `Value`) used unqualified in a way that collides with a function; where such a name is unavoidable (e.g. the `Date` table), it must always be referenced quoted or qualified in DAX.

## 3. Object Naming Rules

### Tables

- **Fact tables:** `Fact ` prefix removed. Use business friendly names, plural. `Sales`, `Inventory Movements`.
- **Dimension tables:** `Dim ` prefix removed. Use business friendly names, entity, singular. `Customer`, `Date`.
- **Hidden/technical/bridge tables:** leading underscore, hidden from report view. `_Measures`, `_Account Customer`.
- Title Case. No abbreviations except approved ones (Section 7).

### Columns

- Title Case, spaces allowed, singular noun. `Customer Name`, `Order Date`.
- Key columns: suffix `Key` (surrogate) or `ID` (natural/business). `Customer Key`, `Order ID`.
- Boolean columns: phrased as a true statement. `Is Active`, `Has Discount`.
- Foreign keys carry the referenced dimension name: `Customer Key` (not `FK Customer`).
- Hidden technical columns: leading underscore. `_Sort Order`.

### Measures

- Title Case, spaces allowed, descriptive. `Total Sales`, `Sales YoY %`.
- Stored centrally in a dedicated hidden `_Measures` table, never in fact tables.
- Aggregation prefix where helpful: `Total `, `Avg `, `Count `, `Distinct Count `, `Min `, `Max `. Note `Distinct Count` is two spelled-out words with a space, not `DistinctCount` (Title Case with spaces applies to prefixes too); it is not an abbreviation and so is not listed in Section 7.
- Time-intelligence suffix: ` YTD`, ` QTD`, ` MTD`, ` PY`, ` YoY %`, ` MoM %`. Example: `Total Sales YTD`, `Total Sales YoY %`.
- Percentages end with ` %`; ratios end with ` Ratio`.
- Base measures unprefixed: `Sales Amount`. Variants build from base: `Sales Amount PY`.
- **Collision avoidance:** a base measure and any variant must remain distinguishable by name alone; never rely on display folder to disambiguate two measures that would otherwise share a name. `Sales Amount` and `Sales Amount PY` are distinct; do not create a second `Sales Amount` in another folder.

### Hierarchies

- Title Case singular describing the drill path. `Calendar`, `Product Category`.
- Levels named individually: `Year`, `Quarter`, `Month`.

### Relationships

- No user-facing name, but document direction and cardinality. Single-direction by default; bidirectional only where justified and recorded in the model glossary.

### Calculation Groups & Items

- Group name: terse, underscore-style technical naming. `Time_Intelligence`, `Currency_Conversion`.
- Calculation items: Title Case, spaces. `Current`, `YTD`, `Prior Year`, `YoY %`.
- **Ordinal/precedence:** set an explicit calculation-group precedence and an explicit ordinal on each item so evaluation order and display order are deterministic, not alphabetical.

### Roles (RLS)

- Pattern `Role_<Scope>`. `Role_Region_EMEA`, `Role_Manager`.

### Parameters (Field & What-If)

- What-if: Title Case user-facing. `Growth Rate %`.
- Field parameters: leading underscore. `_Period Select`.
- Hidden technical object.

### KPIs, Perspectives & Translations

- **KPI objects:** named after the base measure they target, suffix ` KPI`. `Total Sales KPI`. Target and status measures follow normal measure rules.
- **Perspectives:** Title Case, describe the audience or subject. `Finance`, `Sales Leadership`. No underscores.
- **Translations:** only the translated caption may use non-ASCII glyphs; the underlying object name remains ASCII-only per Section 1.

## 4. Visibility & Source Control

- Every non-user-facing object (keys, sort columns, bridge tables, field parameters, the `_Measures` table) is set to **Hidden**.
- Object names are the unit of diff in source control; renaming an object is a breaking change to downstream reports and must be done via a tracked rename, not delete-and-recreate.
- TMDL/XMLA serialization assumes ASCII names (Section 1); any non-ASCII glyph that survives into a name will break round-tripping through source control.

## 5. DAX Conventions

- Reference columns fully qualified: `Customer[Customer Name]`.
- Reference measures unqualified: `[Total Sales]`.
- Variables: `lower_camel` or `snake_case`, descriptive. `var current_sales = ...`, `var _current`.
- Single-quote a table name only when it contains a space or starts with a non-letter. Single-word names (`Sales`, `Customer`, `Date`) need no quotes. Because dimension and fact prefixes are stripped (Section 3), the table is `Date`, not `Dim Date`.

```dax
Total Sales YoY % =
VAR _current = [Total Sales]
VAR _prior = CALCULATE([Total Sales], SAMEPERIODLASTYEAR(Date[Date]))
RETURN
    DIVIDE(_current - _prior, _prior)
```

## 6. Folders, Visibility & Format

- **Display folders** for measures, grouped by subject: `Sales\Core`, `Sales\Time Intelligence`.
- All key columns, sort columns, and technical columns set to **Hidden**.
- Every measure has an explicit **format string** (`#,0`, `0.0%`, `$#,0;($#,0)`).
- Boolean columns carry an explicit format and a default; do not leave them implicit.
- Each measure carries a **description** (one sentence, plain English).

## 7. Approved Abbreviations

Only these may appear in names; everything else spelled out.

| Abbrev | Meaning |
|---|---|
| `ID` | Identifier (natural key) |
| `Key` | Surrogate key |
| `YTD / QTD / MTD` | To-date periods |
| `PY / PM / PQ` | Prior period |
| `YoY / MoM / QoQ` | Period-over-period |
| `Avg` | Average |
| `Qty` | Quantity |
| `Amt` | Amount (avoid where space allows; prefer `Amount`) |
| `%` | Percent (as suffix) |

**Casing:** abbreviations keep the casing shown above regardless of position (`ID`, `Key`, `YTD`, `Avg`, `Qty`). Do not re-case to fit Title Case — write `Order ID`, not `Order Id`, and `Avg Sale`, not `AVG Sale`.

**Empty / placeholder names:** no object may be named with a placeholder such as `Column`, `Column2`, `Measure`, `Measure 1`, `Table`, or `Untitled`. Every object carries a meaningful name before the model is published; auto-generated names are a cleanup item under Section 8.

## 8. Legacy Cleanup Checklist

Apply in order when remediating an existing model:

1. Replace all non-ASCII characters in every object name.
2. Strip leading/trailing and doubled spaces.
3. Move all measures into `_Measures`; delete implicit measures.
4. Replace auto-generated placeholder names (`Column2`, `Measure 1`, `Table`, etc.) with meaningful names (Section 7).
5. Rename tables to user friendly pattern (strip `Fact `/`Dim ` prefixes).
6. Standardize key suffixes (`Key`/`ID`).
7. Apply aggregation prefixes and time-intelligence suffixes to measures.
8. Add format strings and descriptions to every measure.
9. Hide keys, sort, and technical columns.
10. Enforce length caps (Section 2); shorten overlong names using approved abbreviations.
11. Validate each name against the correct pattern for its object type (Section 2) via the regexes below.

**Validation regexes**

- **User-facing name** (tables, columns, measures, hierarchies, what-if parameters):
  `^[A-Za-z][A-Za-z0-9 ]*( %)?$` with a no-double-space guard `(?!.*  )`.
  This permits ` %` only as a trailing suffix, not mid-name, fixing the prior `Sales % YoY` false-pass.

- **Technical name** (tables/columns/parameters with a leading underscore, calculation groups, roles):
  `^_?[A-Za-z][A-Za-z0-9_]*$`.

Apply the technical pattern to any object whose name begins with `_` or uses underscore-style naming; apply the user-facing pattern to everything else. The prior checklist applied only the user-facing pattern, which incorrectly rejected `_Measures` and `_Sort Order`.
