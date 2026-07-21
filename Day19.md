# Day 19: Time Intelligence DAX & Data Visualization in Power BI

## 📋 Topics Covered

1. Building a Date Dimension table with the `CALENDAR` function
2. Automatic date hierarchies (Year → Quarter → Month → Day)
3. Time Intelligence functions — `TOTALMTD`, `TOTALYTD`
4. The `CALCULATE` function — modifying filter context
5. `SAMEPERIODLASTYEAR` vs `DATEADD` — like-for-like comparisons
6. Canvas settings and report design fundamentals
7. Column charts & bar charts (stacked vs clustered)
8. Card visuals and visual formatting
9. Table visuals vs Matrix visuals
10. Drill-down visuals (multi-level hierarchies in one chart)
11. Line charts, trend analysis, and secondary axis
12. Map visuals for geographical data
13. Filters pane (page/report/visual-level) vs Slicers
14. Top N filtering and filter context interaction
15. Report design elements — text boxes, shapes, buttons, images

---

# PART 1: Finishing the Date Dimension — Time Intelligence DAX

## 1. Building a Date Table with `CALENDAR`

### Definition
Beyond importing an existing calendar table, Power BI lets you generate a date dimension table directly using DAX via `New Table`.

### Syntax
```dax
CALENDAR ( <start_date>, <end_date> )
```

### DAX Example — Dynamic Date Table
```dax
Date Table =
CALENDAR (
    MIN ( Orders[OrderDate] ),
    MAX ( Orders[OrderDate] )
)
```

### Explanation of Code
- Hardcoding a start/end date (e.g., `DATE(2025,1,1)` to `DATE(2025,1,31)`) works but is static and won't necessarily match your fact table's actual date range
- Using `MIN(Orders[OrderDate])` and `MAX(Orders[OrderDate])` instead makes the date table **dynamic** — it automatically covers the full span of your fact data and stays correct as new data arrives, with no manual maintenance required

### Best Practices
- Always relate the finished date table to the fact table on the corresponding date field (many-to-one, since many transactions can share one date)
- Set the date column's data type to `Date` (not `Date/Time`) if there's no time component
- Use this date table — not the raw date field on the fact table — for all date-based visuals and filters going forward

### Common Mistake
Hardcoding a start/end date range that doesn't match the actual data range results in a relationship that finds zero matching rows — the date table technically exists but is functionally useless for filtering.

---

## 2. Automatic Date Hierarchies

### Definition
Whenever a `Date` field exists in the model, Power BI automatically builds a hidden **date hierarchy** — Year → Quarter → Month → Day — behind the scenes, without any manual setup.

### Why It Matters
Dragging a date field into a visual by default pulls in this entire hierarchy, not a single flat date value. Understanding this prevents confusion when a visual unexpectedly shows Year/Quarter/Month/Day as four separate levels instead of one date column.

### How To Control It
- The field dropdown in the visual's field well lets you toggle between using the **hierarchy** or the **actual date value** directly
- Within the hierarchy, you can drop specific levels you don't need (e.g., remove Quarter and Day to show only Year and Month)
- Drill-down arrows automatically appear on any visual using a hierarchical date field, letting you expand/collapse between Year, Quarter, Month, and Day on the fly

### Interview Tip
A common practical question: *"Why does my line chart show Year, Quarter, Month, Day as separate axis fields when I only dragged in one Date column?"* — Answer: Power BI auto-generates a date hierarchy for any date field, and it defaults to using the full hierarchy unless you explicitly switch to the plain date value.

---

## 3. Time Intelligence Functions — `TOTALMTD` and `TOTALYTD`

### Definition
**Time Intelligence functions** automatically filter and evaluate an expression for standard business time periods — Month-to-Date (MTD), Quarter-to-Date (QTD), Year-to-Date (YTD) — without manually writing custom filter logic.

### Why It Matters
Nearly every business tracks performance against these three standard periods. Power BI has purpose-built DAX functions for this instead of requiring hand-rolled date filtering logic (which other BI tools often require).

### Syntax
```dax
TOTALMTD ( <expression>, <dates_column> [, <filter>] )
TOTALYTD ( <expression>, <dates_column> [, <filter>] )
```

### DAX Example
```dax
Month-to-Date Sales = TOTALMTD ( [Net Sales], 'Date Table'[Date] )

YTD Sales = TOTALYTD ( [Net Sales], 'Date Table'[Date] )
```

### Explanation of Code
- First argument: any existing expression or explicit measure (here, the `Net Sales` measure)
- Second argument: the date column from your **date dimension table** — this is why building a proper date table matters
- Third (optional) argument, shown in square brackets in DAX syntax: any additional filter — square-bracketed parameters in DAX are always optional
- The function automatically scans the date table, identifies the latest available month (for MTD) or year (for YTD) in the data, and filters the expression to only that period

### Real-World Example
When to display a single, standalone number like MTD or YTD sales — without breaking it down by any other dimension — use a **Card visual**. It's purpose-built for showing one important figure prominently.

### Common Mistakes
- Referencing the raw fact table's date column instead of the date dimension table's date column — time intelligence functions are designed to work against a proper date table
- Assuming `TOTALMTD`/`TOTALYTD` reflect "today's date" — they actually reflect the **latest date present in your data**, which may not be the actual current calendar date if the data is historical or not fully up to date

---

## 4. `CALCULATE` — Modifying Filter Context

### Definition
`CALCULATE` is one of the most powerful functions in DAX — it evaluates an expression within a **modified filter context**, letting you override or add filters on top of whatever context already exists.

### Syntax
```dax
CALCULATE ( <expression>, <filter1>, <filter2>, ... )
```

### Why It Matters
Nearly all comparative business calculations (year-over-year, previous period, same period last year) are built using `CALCULATE` combined with a date-shifting function.

### DAX Example — Last Year's MTD Sales
```dax
Last Year MTD Sales =
CALCULATE (
    [Month-to-Date Sales],
    SAMEPERIODLASTYEAR ( 'Date Table'[Date] )
)
```

### Explanation of Code
- First argument: the existing `Month-to-Date Sales` measure — reused rather than rewritten
- Second argument: a filter modifier — here, `SAMEPERIODLASTYEAR` — that shifts the evaluated dates back exactly one year before recalculating the same expression

### Interview Tip
`CALCULATE` is almost guaranteed to come up: *"What does CALCULATE do, and why is it considered so powerful?"* — Answer: it lets you evaluate any expression under a modified filter context, which is the foundation for nearly all comparative and conditional DAX calculations (YoY growth, same-period comparisons, exception handling).

---

## 5. `SAMEPERIODLASTYEAR` vs `DATEADD` — Getting Comparisons Right

### Definition

| Function | Behavior |
|---|---|
| `SAMEPERIODLASTYEAR` | Shifts dates back exactly one calendar year, preserving the same period type (same month, same quarter, same year) |
| `DATEADD` | Flexibly shifts dates forward or backward by any number of intervals (days, months, quarters, years) |

### The Critical Trap: Partial Period Comparisons
If your latest data only covers a **few days into the current month**, comparing it against the **entire previous month last year** using `SAMEPERIODLASTYEAR` is **not a like-for-like comparison** — you'd be comparing 5–6 days of current data against a full 31 days of prior-year data, producing a misleadingly negative growth number.

### Real-World Example
A month-to-date sales comparison showed a **-66% decline** using `SAMEPERIODLASTYEAR` — but this was comparing partial current-month data (a handful of days) against the last year's *complete* month. Switching to `DATEADD` with an exact **365-day** offset instead produced a **positive** growth figure — revealing the business was actually performing *better*, not worse. The first result was simply an apples-to-oranges comparison.

### DAX Example — Correct Apples-to-Apples Comparison
```dax
Last Year Same Period (365 Days Back) =
CALCULATE (
    [Month-to-Date Sales],
    DATEADD ( 'Date Table'[Date], -365, DAY )
)
```

### Syntax
```dax
DATEADD ( <dates_column>, <number_of_intervals>, <interval> )
```

### Explanation of Code
- Negative numbers move backward in time; positive numbers move forward
- The interval unit (`DAY`, `MONTH`, `QUARTER`, `YEAR`) gives fine-grained control — `DATEADD` can replicate `SAMEPERIODLASTYEAR`'s behavior with `-1, YEAR`, but can also do things `SAMEPERIODLASTYEAR` cannot, like shifting by an exact day count or comparing to 30/90 days prior instead of a full year

### Decision Guide

| Condition | Recommended Function |
|---|---|
| Comparing complete periods (full month vs full month, full year vs full year) | Either works |
| Comparing a **partial** current period against history | `DATEADD` with an exact day offset (e.g., -365 days), not `SAMEPERIODLASTYEAR` |
| Need to compare against 30/90 days ago rather than exactly one year | `DATEADD` (more flexible) |

### Interview Tip
This exact scenario — *"my YoY comparison shows a huge decline, but something feels off"* — is a strong scenario-based interview question testing whether a candidate checks for partial-period data before trusting a comparison metric.

### Best Practice
Always sanity-check date-based comparison metrics against the actual data completeness for the current period before presenting them — a technically correct DAX formula can still produce a misleading business conclusion if the underlying periods aren't truly comparable.

---

