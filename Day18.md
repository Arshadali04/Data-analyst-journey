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

