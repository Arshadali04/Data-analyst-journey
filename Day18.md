# Day 18: DAX — Calculated Columns, Measures & Row/Filter Context

## 📋 Topics Covered

1. Creating calculated columns using DAX (`IF`, nested `IF`, `DATEDIFF`)
2. DAX syntax basics — text vs numeric literals, IntelliSense, field references
3. Column Tools — data type, formatting, default summarization
4. Implicit measures vs Explicit measures
5. Row Context vs Filter Context
6. Column vs Measure — the full comparison
7. Iterator (`X`) functions — why `SUMX` exists and how it differs from `SUM`
8. The `RELATED` function — pulling fields across related tables
9. Date Dimension tables — why and how to build them

---

# PART 1: Introduction to DAX

## What is DAX

**DAX (Data Analysis Expressions)** is Power BI's formula language for creating new columns, measures, and tables at the **data modeling** layer — distinct from M language, which operates at the Power Query/transformation layer.

### Where DAX Lives
Under the **Calculations** section of the ribbon, Power BI gives you three DAX-based creation options:

| Option | What it creates |
|---|---|
| **New Column** | A new field added to a specific table, calculated row by row |
| **New Measure** | An aggregated calculation stored in the data model, not tied to any visible column |
| **New Table** | An entirely new table generated via a DAX expression |

### Why It Matters
Understanding *when* to use each of these three is one of the most tested Power BI interview areas — get comfortable with the column-vs-measure distinction especially, since it comes up in nearly every Power BI interview.

---

# PART 2: Core Concepts

## 1. Writing Your First Calculated Column

### Definition
A calculated column is a new field, added to an existing table, whose value is computed **row by row** using a DAX formula.

### Syntax
```
ColumnName = <DAX Expression>
```

### When To Use
Whenever you need to derive a new value **per row** that should be visible as a field in the table — e.g., translating a numeric code into a readable label.

### Real-World Example
The `ShipVia` field in an Orders table stores shipping mode as a code (`1`, `2`, `3`) rather than a readable label. End users querying the report won't know what these codes mean — so a calculated column translates them into `Air`, `Water`, `Road`.

### DAX Example
```dax
Ship Mode =
IF (
    Orders[ShipVia] = 1, "Air",
    IF (
        Orders[ShipVia] = 2, "Water",
        "Road"
    )
)
```

### Explanation of Code
- `IF(<logical_test>, <result_if_true>, <result_if_false>)` — standard conditional logic, same concept as Excel's `IF`
- Nested `IF` handles more than two outcomes — first checks for `1` (Air), then within the "false" branch checks for `2` (Water), and anything left over falls to `Road`
- Text values (`"Air"`, `"Water"`, `"Road"`) **must** be wrapped in double quotes; numeric values (`1`, `2`) are written as-is, no quotes

### Expected Output
Every row in the Orders table now shows `Air`, `Water`, or `Road` instead of the raw numeric code.

### Common Mistakes
- Forgetting double quotes around text literals — DAX will throw a *"fail to resolve name"* error because it interprets an unquoted word as a table/column/function reference, not literal text
- Not verifying the field reference turned the correct color in the formula bar — a gray/unstyled reference usually signals a typo or invalid field name. **Best practice:** start typing the field name and select it from IntelliSense rather than typing it out fully by hand

### Best Practices
- Always let IntelliSense suggest and auto-complete field/function names — reduces syntax errors significantly
- Use nested `IF` sparingly; for many conditions, `SWITCH` (a cleaner alternative) is usually preferred, though `IF`/nested `IF` is the natural starting point

### Interview Tip
Be ready to explain **why quotes matter** in DAX — a common quick-check question to see if you actually understand DAX syntax rather than just clicking through examples.

---

## 2. Column Tools — Data Type, Format & Default Summarization

### Definition
Selecting any column (in Table view) reveals a **Column Tools** ribbon tab with options to set the column's name, data type, format, and **default summarization**.

### Why It Matters
Default summarization determines what aggregation is automatically applied the moment you drag a numeric field into a visual — this explains behavior you might otherwise think is "automatic magic."

### Real-World Example
Dragging a `Freight` column into a table visual automatically shows the **sum** of freight per row group — not because Power BI guessed correctly, but because the column's **Summarization** property (in Column Tools) is explicitly set to `Sum`.

### Best Practice
Set default summarization deliberately for every numeric column (Sum, Average, Count, or **Don't Summarize** for ID-like fields) — this avoids incorrect default aggregations later and improves performance on large datasets.

---

## 3. Date Calculations with `DATEDIFF`

### Definition
`DATEDIFF` returns the difference between two dates, measured in a specified interval (day, week, month, quarter, year).

### Syntax
```dax
DATEDIFF(<start_date>, <end_date>, <interval>)
```

### When To Use
Any time-based performance metric — e.g., measuring order-to-ship turnaround time, a classic supply-chain efficiency KPI.

### DAX Example
```dax
Days to Ship = DATEDIFF ( Orders[OrderDate], Orders[ShipDate], DAY )
```

### Explanation of Code
- First parameter = the earlier/starting date (`OrderDate`)
- Second parameter = the later/ending date (`ShipDate`)
- Third parameter = the unit to measure the difference in (`DAY`, `WEEK`, `MONTH`, `QUARTER`, `YEAR`, `HOUR`, `MINUTE`, `SECOND`)

### Real-World Example
"Days to Ship," when set to **Average** as its default summarization, tells a business how efficiently its team processes orders on average — a metric that should never be summed (a sum of days-to-ship across thousands of orders is meaningless) but is highly meaningful as an average.

### Interview Tip
A great follow-up question interviewers ask: *"Why would you set this to Average rather than Sum?"* — Answer: because the business question ("how efficient is our shipping process?") is inherently about a typical/central value, not a total.

---

## 4. Calculating Business Metrics — Net Sales Example

### Definition
Combining multiple existing columns with arithmetic operators to derive a new business metric at the row level.

### Real-World Example
`Order Details` has `Unit Price`, `Quantity`, and `Discount` (stored as a percentage, e.g., `0.25` = 25% off). The business wants **Net Sales** — revenue *after* discount is applied.

### DAX Example
```dax
Net Sales Column = Order Details[Unit Price] * Order Details[Quantity] * ( 1 - Order Details[Discount] )
```

### Explanation of Code
- `Unit Price × Quantity` = gross sales for that line item
- `× (1 − Discount)` = applies the discount percentage to bring it down to net sales
- This entire expression is evaluated **per row**, since it's a calculated column

### Common Mistakes
- Misreading the discount field as a flat dollar amount instead of a percentage — always check the column's actual meaning/format before building a formula on top of it
- Assuming this same row-wise formula would work unmodified as a measure (see Section 7 — it will not, without an iterator function)

---

## 5. Counting Unique Records — Why Not Always a Column

### Definition
Sometimes you need a count of **distinct values** in a column (e.g., total unique orders placed) — a classic aggregation scenario.

### Common Mistake (Important Conceptual Trap)
Creating this as a **calculated column** using `DISTINCTCOUNT` will technically run — but it repeats the *same total value* on every single row, because the calculation evaluates across the entire table, not per row. This is a strong signal that the calculation belongs in a **measure**, not a column.

```dax
-- ❌ Wrong approach: as a calculated column
Total Orders (column) = DISTINCTCOUNT ( 'Order Details'[Order ID] )
-- Every row shows the same repeated total — not useful as a column
```

### Correct Approach
```dax
-- ✅ Correct: as a measure
Total Orders = DISTINCTCOUNT ( 'Order Details'[Order ID] )
```

### Interview Tip
This exact scenario — *"I created a column with DISTINCTCOUNT and every row shows the same number, what's wrong?"* — is a common practical/scenario-based interview question. The answer is always: **aggregated results belong in measures, not columns.**

---

## 6. Implicit vs Explicit Measures

### Definition

| Type | Definition |
|---|---|
| **Implicit Measure** | Created "on the fly" by dragging a raw column into a visual and changing its aggregation (Sum, Count, Average, etc.) directly within that visual |
| **Explicit Measure** | A separately created, named DAX formula (`New Measure`) with the aggregation logic explicitly written into the formula itself |

### When To Use

| Condition | Recommended Approach |
|---|---|
| One-off, quick visual, aggregation logic is simple | Implicit measure (drag field, change aggregation) |
| Reused across multiple visuals/reports | Explicit measure |
| Calculation logic is more complex than a single aggregation | Explicit measure (required) |
| Team collaboration / maintainability matters | Explicit measure |

### DAX Example (Explicit Measure)
```dax
Total Orders = DISTINCTCOUNT ( 'Order Details'[Order ID] )
```

### Why It Matters
- **Reusability**: an explicit measure is defined once and can be dropped into any visual without re-configuring the aggregation each time
- **Consistency**: removes the risk of accidentally using the wrong aggregation in different visuals
- **Discoverability**: explicit measures show up clearly in the data model's Fields list, making them easy for other report builders to find and reuse

### Common Mistake
Assuming implicit measures are "lesser" or wrong — they're perfectly valid for quick, one-off analysis, but don't scale well across a team-maintained report.

### Best Practice
For anything going into a shared/production report, default to **explicit measures** — even for simple aggregations — for consistency and reusability.

---

## 7. Row Context vs Filter Context

### Definition

| Concept | Applies To | Meaning |
|---|---|---|
| **Row Context** | Calculated Columns | The formula is evaluated **one row at a time**, using values from that same row |
| **Filter Context** | Measures | The formula's result is filtered by whatever other fields (dimensions) are present in the same visual |

### Real-World Example
A `Total Orders` measure evaluates to a single number (e.g., `830`) when viewed alone. Add `Company Name` to the same visual, and the measure recalculates **per company** — each row now shows a different, correctly filtered count. That surrounding filtering — driven by whatever dimension fields are in the visual — is the **filter context**.

```
Filter Context Example:
Ship Country | Total Customers (measure)
Germany      | 45
France       | 38
USA          | 122
```
Here, `Ship Country` is the filter context under which the `Total Customers` measure is evaluated.

### Interview Tip
This is one of the **most frequently asked DAX interview questions**: *"What is filter context / row context in Power BI?"* Have a one-line answer ready for each, plus the key distinction: **columns work at row context, measures work at filter context.**

---

## 8. Column vs Measure — Full Comparison

### Comparison Table

| Feature | Column | Measure |
|---|---|---|
| Evaluation level | Row-by-row (row context) | Aggregated, filtered by visual context (filter context) |
| Requires aggregation function? | No | **Yes — mandatory** |
| Where it's added | A specific table (visible as a field) | The entire data model (not tied to a visible table field) |
| Storage impact | Increases data model size (materialized per row) | Minimal — calculation only runs when visualized |
| Typical output type | Any (text, date, numeric) | Typically numeric (though not exclusively) |
| Editable "home table"? | Fixed to the table it was created in | Can be moved to any table via Measure Tools without affecting results |

### Why It Matters
> ⚠️ **Performance best practice:** Whenever a calculation can be done as *either* a column or a measure, always prefer the **measure** — it does not inflate your data model's memory footprint, unlike a calculated column which allocates storage for every row.

### Interview Tip
Expect a direct question: *"When would you use a calculated column vs a measure?"* — Answer: use a **column** when you need a row-level value visible as a field (e.g., for slicing/filtering, or as an input to further row-level logic); use a **measure** whenever the output is an aggregated number responding to visual-level filtering.

### Demonstration: Why Measures Don't Care Which Table They're Created In
A measure created under `Order Details` can be moved (via **Measure Tools → Home Table**) to `Categories`, and it will still return identical, correctly-filtered results — because measures belong to the entire data model and respect all existing relationships, regardless of which table they're visually organized under.

---

## 9. Iterator Functions (`X` functions) — `SUMX`

### Definition
A family of DAX functions ending in **X** (`SUMX`, `AVERAGEX`, `COUNTX`, `MAXX`, `MINX`, etc.) that evaluate an expression **row by row first**, then apply the aggregation — as opposed to standard aggregation functions (`SUM`, `AVERAGE`), which can only operate on a single existing column.

### The Problem They Solve
Trying to replicate a multi-column row-wise formula (like Net Sales) as a measure using plain `SUM` fails badly:

```dax
-- ❌ WRONG — sums each column FIRST, then multiplies the totals together
Net Sales (Wrong) =
    SUM ( 'Order Details'[Unit Price] )
    * SUM ( 'Order Details'[Quantity] )
    * ( 1 - SUM ( 'Order Details'[Discount] ) )
-- Produces wildly incorrect results (even negative values in the trillions)
```

This is wrong because `SUM(Unit Price) × SUM(Quantity)` is **not** the same as summing `(Unit Price × Quantity)` per row — order of operations matters completely.

### Correct Approach — `SUMX`
```dax
Net Sales =
SUMX (
    'Order Details',
    'Order Details'[Unit Price] * 'Order Details'[Quantity] * ( 1 - 'Order Details'[Discount] )
)
```

### Syntax
```dax
SUMX ( <table>, <expression> )
```

### Explanation of Code
- First argument = the table to iterate over row by row
- Second argument = the row-level expression to evaluate (identical logic to what you'd write in a calculated column)
- `SUMX` evaluates the expression **for every row first**, then sums all those row-level results — giving the mathematically correct answer

### Expected Output
Matches exactly what the calculated-column version of Net Sales produces — but as a measure, with all the performance/reusability benefits that come with that.

### Common Mistakes
- Using plain `SUM` when a formula involves **multiple columns multiplied/combined together** — this is the #1 trigger for needing an iterator function
- Forgetting that `SUMX`'s second argument is a *row context expression* — you can reference columns directly here just like inside a calculated column

### Interview Tip
A near-guaranteed interview question: *"What's the difference between SUM and SUMX?"* — Answer: `SUM` aggregates a single existing column directly. `SUMX` evaluates a custom row-level expression across a table first, then sums the row-level results — necessary whenever your calculation involves multiple columns combined per row before aggregating.

### Best Practice
Any time a measure's logic mirrors a calculated column's multi-column formula, default to the `X` version of the aggregation function you need (`SUMX`, `AVERAGEX`, etc.) rather than trying to force plain `SUM`/`AVERAGE` to work.

---

## 10. The `RELATED` Function

### Definition
`RELATED` pulls a column's value from a **related table** into the current table, following an existing relationship in the data model.

### Syntax
```dax
RELATED ( <related_table_column> )
```

### When To Use
When you need a field from a **dimension table** (the "one" side of a relationship) available directly inside a **fact table** (the "many" side) — e.g., pulling a product's list price from the `Products` table into the `Order Details` table.

### DAX Example
```dax
Cost Price = RELATED ( Products[Unit Price] )
```

### Explanation of Code
- Works because `Order Details` and `Products` have an established **many-to-one** relationship (many order-detail rows can reference the same one product)
- `RELATED` looks up the single matching row in the "one" side of the relationship (via `Product ID`) and brings back the requested column's value for that row

### Common Mistakes
- Trying to use `RELATED` across a relationship where the target table's matching side isn't unique (i.e., trying to pull from the "many" side into the "many" side) — `RELATED` only works when there's exactly one matching row to look up, which requires a many-to-one direction
- Not verifying the relationship actually exists before assuming `RELATED` will find the field — if there's no relationship, the field won't even appear as an option

### Interview Tip
Expect: *"How would you bring a field from another table without physically merging tables in Power Query?"* — Answer: `RELATED`, provided a relationship already exists in the data model.

---

## 11. Date Dimension Tables

### Definition
A dedicated table containing one row per date (or per date+hour, depending on required granularity) between a defined start and end date — used instead of relying directly on a date field from a fact table.

### Why It Matters
1. **Enables richer date analysis** — a date dimension table can hold derived columns like `Year`, `Month`, `Quarter`, `Week Number`, `Weekday`/`Weekend` flag — attributes that shouldn't bloat the fact table itself
2. **Preserves data integrity** — fact tables only contain dates where a transaction actually happened. If there were zero orders on a given day, that date simply won't appear in the fact table at all. A date dimension table guarantees every date in the analysis period exists — including ones with no activity — so reports can correctly show "no sales" rather than silently omitting the date entirely

### When To Use
Any data model where date-based analysis is required (which is nearly all business reporting) — building a date dimension table is considered a **standard best practice**, not an edge-case technique.

### Real-World Example
An orders dataset spans several years, but on some individual days there simply were no orders. Without a date dimension table, a monthly trend chart would silently skip those zero-order days rather than showing a true dip to zero — misleading anyone reading the trend.

### Requirements for a Date Dimension Table
- One row per date (or finer granularity, e.g., per hour, if the fact data requires it) between a **user-defined start date** and **end date**
- Connected to the fact table(s) via a relationship on the corresponding date field
- Used in visuals/filters **instead of** the raw date field from the fact table directly

### Decision Guide

| Condition | Recommended Approach |
|---|---|
| Model has any date-based fact data | Build a dedicated Date dimension table |
| Need derived date attributes (Year, Quarter, Weekday) | Add them as columns in the Date table, not the fact table |
| Fact table granularity is hourly | Date table granularity should match (one row per hour) |

---

# 🎯 Key Takeaways

| Topic | Key Point |
|---|---|
| DAX | Power BI's modeling-layer formula language — creates columns, measures, and tables |
| Calculated Column | Row-level formula, becomes a visible field in a table, uses row context |
| Text vs numeric literals | Numbers written as-is; text must be wrapped in double quotes |
| Column Tools | Set data type, format, and **default summarization** per column |
| DATEDIFF | Returns difference between two dates in a chosen interval (day/week/month/etc.) |
| Implicit measure | Aggregation applied on-the-fly inside a visual, using a raw column |
| Explicit measure | Separately defined, reusable, named DAX measure with aggregation built in |
| Row Context | How columns are evaluated — one row at a time |
| Filter Context | How measures are evaluated — filtered by whatever dimensions are in the visual |
| Column vs Measure | Column = row-level, no aggregation required, tied to a table; Measure = aggregated, mandatory aggregation function, belongs to the whole data model |
| Performance rule | Prefer measures over columns whenever the calculation is feasible either way — measures don't inflate model size |
| Iterator (X) functions | `SUMX`, `AVERAGEX`, etc. — evaluate expressions row by row, then aggregate; required for multi-column row-wise formulas turned into measures |
| RELATED | Pulls a column from a related table (many-to-one direction) into the current table |
| Date Dimension Table | Dedicated table with one row per date; enables richer date analysis and preserves data integrity for dates with zero activity |

---

# ⚡ Pro Tips

- Always type field names manually and pick them from IntelliSense rather than guessing full syntax — it drastically reduces reference errors, and the formula bar's color-coding instantly confirms a valid reference.
- If a calculated column shows the *same value repeated on every row*, that's a strong signal it should be a measure instead — this is one of the most common beginner mistakes.
- Whenever a formula multiplies or combines **more than one column** and you need it as a measure, reach for the `X` (iterator) version of the aggregation function immediately — don't try to force plain `SUM`/`AVERAGE` first.
- Set default summarization deliberately on every numeric column — don't leave it to guesswork, especially for ID-like fields which should be set to "Don't Summarize."
- Build a date dimension table as a standing habit on any model with transactional/date data — treat it as a default step, not an optional add-on.
- In interviews, always tie your answer on Column vs Measure back to **row context vs filter context** — it shows you understand the underlying mechanism, not just the surface rule.

---

# Practice Exercises

- [ ] Create a calculated column using nested `IF` to translate a numeric code field into readable text labels
- [ ] Create a `DATEDIFF`-based calculated column measuring the gap (in days) between two date fields in your dataset
- [ ] Deliberately create a `DISTINCTCOUNT` calculation as a **column** first, observe the repeated-value problem, then fix it by recreating it as a **measure**
- [ ] Create both an implicit measure (drag + change aggregation) and an explicit measure (`New Measure`) that return the same result — compare their behavior across different visuals
- [ ] Write a calculated column that multiplies 2–3 columns together (e.g., a "line total" style calculation)
- [ ] Attempt to replicate that same calculation as a measure using plain `SUM` first (observe the incorrect result), then fix it using `SUMX`
- [ ] Use `RELATED` to pull one field from a dimension table into a fact table
- [ ] (Interview-level) Explain out loud, without looking at notes: the difference between row context and filter context, using your own example

---

# Mini Project

**Objective:** Build a small but complete DAX layer on top of an existing relational data model

**Dataset:** Continue with the Orders / Order Details / Products / Customers / Categories model from Day 17

**Tasks:**
1. Create a `Ship Mode` calculated column translating a coded field into readable labels
2. Create a `Days to Ship` calculated column using `DATEDIFF`, set its default summarization to Average
3. Create a `Net Sales` calculated column (multi-column row-wise formula)
4. Recreate `Net Sales` as an **explicit measure** using `SUMX`, and confirm both approaches produce matching totals in a visual
5. Create an explicit `Total Orders` measure using `DISTINCTCOUNT`, and an explicit `Total Customers` measure the same way
6. Use `RELATED` to bring one dimension-table field into a fact table
7. Build a simple Date dimension table and relate it to the Orders fact table by `Order Date`

**Skills Practiced:** DAX syntax, row vs filter context reasoning, iterator functions, cross-table lookups, date dimension modeling

---

# Interview Preparation

## Frequently Asked Questions

**Q: What is the difference between a calculated column and a measure?**
A: A calculated column computes a value row by row and is stored as a visible field in a specific table (row context). A measure computes an aggregated result that responds dynamically to whatever filters/dimensions are present in a visual (filter context), and must include an aggregation function in its formula.

**Q: What is the difference between an implicit and an explicit measure?**
A: An implicit measure is created on the fly by dragging a raw column into a visual and setting its aggregation there. An explicit measure is a separately defined, named DAX formula with the aggregation logic built in, reusable across any visual.

**Q: What is row context vs filter context?**
A: Row context is how calculated columns are evaluated — one row at a time, using values from that same row. Filter context is how measures are evaluated — the result is filtered by whatever other fields/dimensions are present in the same visual.

**Q: Why would you use SUMX instead of SUM?**
A: SUM can only aggregate a single existing column. SUMX evaluates a custom row-level expression (often combining multiple columns) for every row first, then sums those row-level results — necessary whenever a measure's logic mirrors a multi-column calculated-column formula.

**Q: What does the RELATED function do?**
A: It pulls a column's value from a related table into the current table, following an existing many-to-one relationship in the data model.

**Q: Why build a separate Date dimension table instead of using the date field directly from a fact table?**
A: It allows richer derived date attributes (year, month, quarter, etc.) without bloating the fact table, and it preserves data integrity by ensuring every date in the analysis period is represented — including dates with zero transactions — so trends aren't misleadingly gapped.

## Scenario-Based Questions

**Scenario:** You create a `DISTINCTCOUNT` calculation as a column, and every row shows the identical total value. What went wrong, and how do you fix it?
**Answer:** The calculation is aggregated in nature, but columns evaluate at row context, so it repeats the whole-table result on every row. Fix: recreate the same logic as a measure instead, so it correctly responds to filter context.

**Scenario:** You write a measure using `SUM(ColumnA) * SUM(ColumnB)` to calculate total revenue, but the result is wildly incorrect (even negative in some cases). What's the issue?
**Answer:** Summing each column independently before multiplying is not the same as summing the row-wise product of the two columns. The fix is to use `SUMX` with a row-level expression (`ColumnA * ColumnB`) evaluated per row, then summed.

## Practical Questions

- Write the DAX for a calculated column that flags each order as "On Time" or "Delayed" based on comparing `Ship Date` against `Required Date`.
- Given a `Products` table and an `Order Details` table related by `Product ID`, write the DAX to bring the product's `Category` field into `Order Details` without modifying the Power Query layer.

---

# Revision Cheat Sheet

- **DAX** = modeling-layer formula language (columns, measures, tables)
- **Text literals** → double quotes; **numeric literals** → no quotes
- **Calculated column** → row context, no aggregation required, visible field in a table
- **Measure** → filter context, aggregation function **mandatory**, belongs to entire data model
- **Implicit measure** = drag + change aggregation in a visual | **Explicit measure** = `New Measure`, reusable
- **Repeated value on every row in a column?** → should almost certainly be a measure instead
- **SUM** = aggregates one column directly | **SUMX** = row-level expression evaluated first, then summed (needed for multi-column formulas)
- **RELATED** = pull a field from a related table (many-to-one direction only)
- **Date dimension table** = one row per date; add derived date attributes here, not in the fact table; preserves data integrity for zero-activity dates
- **Performance rule**: prefer measures over columns whenever both are possible — measures don't inflate model size
