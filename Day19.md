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

# PART 2: Data Visualization Fundamentals

## 6. The Canvas & Report Page Settings

### Definition
The **canvas** is the bounded design area (marked by dotted lines in Report view) where all visuals live. Before adding visuals, you can configure canvas-level settings.

### Settings Available
- **Canvas size**: standard presets or custom height/width ratios (choose based on the target display — monitor vs projector vs mobile)
- **Canvas background**: solid color or custom image, with adjustable transparency

### Best Practice
Decide on canvas size and background before building visuals — retrofitting a size change after populating a page can disrupt layout alignment.

---

## 7. Column Charts & Bar Charts

### Definition

| Term (Power BI) | Orientation |
|---|---|
| **Column Chart** | Vertical bars |
| **Bar Chart** | Horizontal bars |

> Note: In some other BI tools, "bar chart" refers to both orientations generically — Power BI uses distinct terminology.

### Stacked vs Clustered

| Type | Legend Behavior | When To Use |
|---|---|---|
| **Stacked** | Legend values stack on top of each other within one bar; shows both individual contribution and cumulative total | When the *total* matters as much as the breakdown |
| **Clustered** | Legend values appear as separate bars side by side | When comparing legend categories directly *against each other* |

### Best Practices
- **Always sort your chart** — by the measure (ascending/descending), not by category name alphabetically. Sorted data lets viewers instantly identify rankings (e.g., "4th highest") without comparing bar heights one by one
- **Use tooltips** to check exact values, but don't rely on them as the primary way to convey data — turn on **Data Labels** (Format → Data Label) for values the audience needs to read at a glance
- **Be selective with Legend fields** — only use fields with a small number of distinct values. A legend with too many categories (e.g., month instead of year) makes the chart visually noisy and hard to interpret

### Common Mistake
Adding a high-cardinality field (like individual months across years, or product names) to the Legend — this fragments the chart into too many colors/segments to be readable.

### Real-World Example
Comparing yearly sales by category across a 3-year legend (1996, 1997, 1998) revealed that 1998 appeared consistently lower — but this was because the dataset only had **partial data through May 1998**, not because performance actually declined. Always account for partial-period data before drawing conclusions from a chart.

---

## 8. Card Visuals

### Definition
A visual purpose-built for displaying **one single, important number** — the "headline metric" of a report.

### When To Use
Any KPI that deserves standalone visibility without being broken down by other dimensions — e.g., Total Net Sales, Total Orders, YTD Sales.

### Formatting Options
- **Callout value**: the number itself — font size, color, decimal precision, and **display units** (None / Thousands / Millions / Auto) are all configurable
- **Category label**: the measure's name, shown below the value — can be hidden or customized
- **General/Effects**: background color, border (including rounded corners and custom border color), and a separate **Title** with its own font, background, and alignment settings

### Best Practice
Set a consistent display unit (e.g., always show in thousands) across all Card visuals in a report for visual consistency — mixing raw numbers, thousands, and millions across cards creates confusion.

---

## 9. Table Visuals vs Matrix Visuals

### Comparison Table

| Feature | Table Visual | Matrix Visual |
|---|---|---|
| Layout | Flat — one column per field added | Flexible — separate Rows, Columns, and Values sections |
| Subtotals | Not shown automatically; must be manually aggregated | Automatic subtotals at every row/column level, plus grand total |
| Cross-tabulation | Not possible | Supports true cross-tab (e.g., Country × Category grid) |
| Drill-down | Not supported | Supported, when multiple fields are added to Rows/Columns |
| Best for | Simple, flat detail lists | Multi-dimensional summaries needing subtotals |

### When To Use
- **Table**: quick detailed listing, e.g., country-level sales and order counts, sorted by any column with a single click on the header
- **Matrix**: whenever you need both granular detail *and* aggregated subtotals simultaneously — e.g., Country (rows) × Category (columns) with Net Sales in the values, showing category totals, country totals, and a grand total all at once

### Real-World Example
A table showing Country, Total Orders, and Net Sales revealed **different top-4 rankings** depending on which column it was sorted by — USA/Germany/Austria/Brazil by sales, but Germany/USA/Brazil/France by order count. Sorting alone can't show both rankings simultaneously — this is where **conditional formatting** (background color or data bars applied via Format → Cell Elements) helps visually highlight one metric's ranking even while sorted by another.

### Conditional Formatting Options

| Option | Recommendation |
|---|---|
| Background color | ✅ Recommended — clear visual gradient |
| Data bars | ✅ Recommended — intuitive proportional bars |
| Font color | ⚠️ Not recommended — hard to read text with color-only variation |

### Best Practice — Matrix Drill-Down
Adding a second field to Rows (e.g., Country then City) automatically enables drill-down in a Matrix — click the expand icon on any single row to drill into just that row's detail, or use the header-level drill controls to expand/collapse the entire level at once.

---

