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

## 10. Drill-Down Visuals (Multi-Dimension Charts)

### Definition
Adding **two or more fields** to the same axis (e.g., Category and Country both in the Y-axis of a bar chart) automatically enables drill-down — indicated by arrow icons appearing above the visual.

### Drill Controls

| Icon | Function |
|---|---|
| Down arrow (expand one level) | Shows the next level of detail for *all* items at once |
| Up arrow (drill up) | Returns to the parent level |
| Double-down arrow (next level) | Switches the entire chart from one dimension to the next, replacing rather than nesting |
| Single arrow (drill mode toggle) | Lets you click on *one specific item* to drill into just that item's detail, rather than all items at once |

### Why It Matters
Building one drill-down visual instead of two separate charts saves report real estate and lets viewers explore the contributing factors behind a total without navigating away.

### Best Practice
When drilling into a hierarchy, re-sort by the *category* level (not the measure) if the newly expanded groupings look scattered — sorting by measure value can visually separate items that belong to the same parent group.

---

## 11. Line Charts & Secondary Axis

### Definition
Line charts are the standard visual for **any data with a date/time sequence** — dates always belong in the X-axis (never the Y-axis), because unlike categorical dimensions, dates have an inherent, meaningful order.

### Why Line Charts (not bar charts) for Time Data
Bar/column charts *can* technically show data over time, but line charts are specifically designed to emphasize **continuity and trend** — the shape of the line over time is the point, not the comparison between individual bars.

### The Scaling Problem — and Secondary Axis
Adding two measures with very different magnitudes (e.g., Net Sales in the tens of thousands vs Total Orders in the low hundreds) to the same Y-axis makes the smaller-magnitude measure appear as a flat line — not because it isn't fluctuating, but because the shared axis scale is dominated by the larger measure.

### Solution: Secondary Y-Axis
Line charts uniquely support a **secondary Y-axis** — assign one measure to the primary axis and the other to the secondary axis, and each gets its own independent scale, making both trends visible simultaneously.

### Decision Guide

| Condition | Recommended Approach |
|---|---|
| Two measures with similar magnitude | Both on primary axis is fine |
| Two measures with very different magnitude/scale | Assign one to secondary axis |
| Comparing 3+ measures with different scales | Consider separate visuals instead — secondary axis only supports two scales |

### Interview Tip
*"Why does my orders trend line look flat compared to my sales trend line, even though orders are also fluctuating?"* — Answer: axis scaling. When two measures with very different magnitudes share one axis, the smaller one gets visually compressed — move it to a secondary axis.

---

## 12. Map Visuals

### Definition
Visuals for plotting data across geographical locations — Power BI offers three built-in map types (Map, Filled Map, Azure Map).

### Required Fields
- **Location**: any geographical field — country, state, city, zip code, or latitude/longitude
- **Bubble Size** (optional): a measure that scales the size of each location's marker, adding a second dimension of insight (proximity *and* magnitude)

### Why It Matters
Maps reveal **spatial patterns** that a table or bar chart cannot — e.g., identifying that most business clusters around a specific coastal region, not just which countries have the most sales.

### Real-World Example
Adding `Net Sales` to Bubble Size on a country-level map instantly reveals which markets are the biggest contributors — not just where customers are located, but how much business each location represents. Drilling further into City-level location revealed that most North American business specifically clustered along the **West Coast** — a pattern invisible at the country level alone.

### Best Practice
Use drill-down (Location field: Country → City) on map visuals the same way as on bar/column charts — it reveals finer-grained spatial patterns without needing a separate visual.

---

## 13. Filters Pane vs Slicers

### Definition
Both filter report data, but they work differently and serve different purposes.

### Comparison Table

| Feature | Slicer | Filters Pane |
|---|---|---|
| Location | Lives inside the canvas (a visible design element) | Lives outside the canvas — doesn't affect layout |
| Visibility to viewer | Always visible, interactive | Can be collapsed/hidden from the immediate view |
| Scope options | Page-specific by placement | Explicit scope: **this visual**, **this page**, or **all pages** |
| Filter type | Basic (select/deselect values) | Basic, Advanced (conditional), and **Top N** |

### The Three Filter Scopes
1. **Filters on this visual** — appears only when a specific visual is selected; affects only that one visual
2. **Filters on this page** — affects every visual on the current page
3. **Filters on all pages** — affects the entire report, every page, without needing to configure each page individually

### Advanced Filtering
For dimension fields, options include conditions like "starts with," "does not contain," "is blank"/"is not blank." For measure fields, options are numeric comparisons ("greater than," "less than," "is blank").

### Top N Filtering — A Key Feature
Restricts a visual to show only the top (or bottom) N values ranked by a chosen measure.

**Method:** Filter type → Top N → specify count (e.g., 10) → specify the ranking measure (drag it into "By value") → Apply.

### Why It Matters
Top N filters are **context-sensitive** — they automatically recalculate based on whatever other filters/slicers are currently active. A "Top 10 Countries by Sales" list will show a different set of countries when filtered to a specific product category than when viewing all categories, because the ranking itself changes with context.

### Real-World Example
Reports built for senior leadership often need to show only "the top 10 products" or "top 3 categories" rather than the full dataset — Top N filtering is the standard mechanism for surfacing only the most business-critical rows without manual curation.

### Interview Tip
Be ready to explain: *"How would you show only the top 10 customers by revenue, and have that list stay accurate even as the user changes other filters?"* — Answer: Top N filtering, which is inherently filter-context-aware and updates dynamically.

---

## 14. Report Design Elements

### Definition
Beyond data visuals, Power BI supports several non-data elements for structuring and polishing a report page.

| Element | Purpose |
|---|---|
| **Text Box** | Titles, disclaimers, footnotes, custom labels |
| **Shapes** | Visual dividers, backgrounds for grouping visuals, custom containers |
| **Buttons** | Built-in interactive actions (navigation, back, apply/clear slicers) |
| **Images** | Logos, icons, custom branding |

### Shapes as Visual Containers
A shape can be layered **behind** a visual (via Format → Send Backward) to create a "framed" look — combined with making the visual's own background transparent, this produces a cohesive card-like appearance without relying solely on the visual's built-in border/background options.

### Built-in Button Actions
- **Back button**: navigates to whichever page the viewer came from (not a fixed page) — requires Ctrl+Click in Power BI Desktop, but works with a single click once published to Power BI Service
- **Apply all slicers**: holds off filtering until the user explicitly clicks apply, useful when multiple slicer selections should take effect together rather than filtering after every individual click
- **Clear all slicers**: resets every slicer on the page in one click

### Best Practice
Use the "Apply all slicers" pattern when a report has many slicers and instant filtering on every click would feel sluggish or cause visual "flicker" as users make multiple selections.

---

# 🎯 Key Takeaways

| Topic | Key Point |
|---|---|
| CALENDAR | Builds a date table; use MIN/MAX of fact table's date field for a dynamic, self-maintaining range |
| Date Hierarchy | Auto-generated (Year/Quarter/Month/Day) for any date field; can be toggled off in favor of the plain date |
| TOTALMTD / TOTALYTD | Time intelligence functions that auto-filter to the latest month/year present in the data |
| CALCULATE | Evaluates an expression under a modified filter context — foundation of most comparative DAX |
| SAMEPERIODLASTYEAR vs DATEADD | SAMEPERIODLASTYEAR shifts by exact calendar period; DATEADD is more flexible (any interval, any direction) — critical for avoiding partial-period comparison errors |
| Canvas | The bounded design area; configure size/background before building visuals |
| Column vs Bar chart | Column = vertical, Bar = horizontal (Power BI-specific terminology) |
| Stacked vs Clustered | Stacked shows cumulative + breakdown; Clustered compares legend values side by side |
| Card visual | For displaying one standalone important number |
| Table vs Matrix | Table = flat list; Matrix = cross-tab with automatic subtotals and drill-down |
| Drill-down visuals | Add 2+ fields to one axis to enable expand/collapse exploration in a single chart |
| Line chart | Always use for date-based trends; dates go on X-axis |
| Secondary axis | Solves the scaling problem when combining measures of very different magnitude on a line chart |
| Map visuals | Reveal spatial patterns; Bubble Size adds a magnitude dimension |
| Filters pane vs Slicer | Filters pane lives outside canvas with 3 scopes (visual/page/all pages); Slicer is a visible canvas element |
| Top N filter | Context-aware ranking filter — recalculates dynamically with other active filters |
| Report elements | Text boxes, shapes, buttons, images add structure and interactivity beyond data visuals |

---

# ⚡ Pro Tips

- Build your date dimension table with `MIN`/`MAX` of the fact table's date column rather than hardcoded dates — it self-maintains as new data arrives.
- Before trusting any YoY or period-over-period metric, check whether the current period has **complete** data — a partial period compared against a full prior period will always look artificially worse.
- Sort every bar/column chart by its measure value, not alphabetically — sorted charts let viewers instantly read rankings without comparing bar heights.
- Reach for a Matrix visual, not a Table, the moment you need subtotals or a cross-tab view — retrofitting subtotals onto a Table means manual aggregation.
- If a trend line looks suspiciously flat, check whether it's sharing an axis with a much-larger-magnitude measure — move it to the secondary axis.
- Use Top N filtering for executive-facing reports — it's inherently filter-context-aware, so "top 10" stays accurate no matter what other filters the viewer applies.

---

# Practice Exercises

- [ ] Build a dynamic date table using `CALENDAR` with `MIN`/`MAX` of your fact table's date column
- [ ] Create `TOTALMTD` and `TOTALYTD` measures for any sales/revenue field, and display each in a Card visual
- [ ] Create a `CALCULATE` + `SAMEPERIODLASTYEAR` measure, then recreate the same comparison using `CALCULATE` + `DATEADD` with an exact day offset — compare the two results and explain any difference
- [ ] Build a clustered column chart with a category on the X-axis and a 3-value legend; then rebuild the same as stacked and compare
- [ ] Create a Matrix visual with two dimensions in Rows and one measure in Values — confirm subtotals and grand total appear correctly
- [ ] Build a two-level drill-down bar chart (e.g., Category → Country) and practice all four drill controls
- [ ] Build a line chart combining two measures of very different magnitude, observe the flattened line, then fix it using a secondary axis
- [ ] Apply a Top 10 filter to a table visual, then change an unrelated slicer and confirm the Top 10 list updates accordingly

---

# Interview Preparation

## Frequently Asked Questions

**Q: What does the CALCULATE function do?**
A: It evaluates an expression under a filter context modified by the filters supplied as its additional arguments — it's the foundation of most comparative and conditional DAX calculations.

**Q: What's the difference between SAMEPERIODLASTYEAR and DATEADD?**
A: SAMEPERIODLASTYEAR shifts dates back exactly one calendar year, preserving the same period type. DATEADD is more flexible — it can shift by any number of days, months, quarters, or years, in either direction, which matters when comparing partial periods where an exact day-count offset gives a more accurate like-for-like comparison.

**Q: When would you use a Matrix visual instead of a Table?**
A: Whenever automatic subtotals, a grand total, or a true cross-tabulation (two dimensions at once) are needed — a Table only supports a flat list with no built-in aggregation at intermediate levels.

**Q: Why would a line chart with two measures show one line as completely flat?**
A: If the two measures have very different magnitudes and share the same Y-axis, the smaller-magnitude measure gets visually compressed near zero. The fix is assigning it to the secondary axis.

**Q: What's the difference between the Filters pane and a Slicer?**
A: A Slicer is a visible canvas element that's part of the report's layout. The Filters pane lives outside the canvas, doesn't affect visual layout, and supports three distinct scopes — this visual, this page, or all pages — plus basic, advanced, and Top N filter types.

## Scenario-Based Questions

**Scenario:** Your month-to-date year-over-year comparison shows a dramatic decline, but you suspect something's wrong. What should you check first?
**Answer:** Whether the current period has complete data. If the latest month only has a few days of data, comparing it against a full prior-year month (via SAMEPERIODLASTYEAR) isn't a fair comparison — switch to DATEADD with an exact day-count offset instead.

**Scenario:** A stakeholder wants a report showing only the top 10 products by revenue, but wants that list to stay accurate as they filter by different regions. How do you build this?
**Answer:** Use a Top N filter (Filter type: Top N) on the product field, ranked by the revenue measure. Because Top N filters are filter-context-aware, the top-10 list will automatically recalculate for whatever region filter is applied.

## Practical Questions

- Write the DAX for a measure showing year-over-year growth percentage, using an existing YTD Sales measure and its prior-year equivalent.
- Describe the steps to build a report page where clicking a button toggles the entire page's view between "by Sales" and "by Orders" without creating two separate pages the user has to manually navigate between.

---

# Revision Cheat Sheet

- **CALENDAR(min, max)** → dynamic date table using fact table's actual date range
- **TOTALMTD / TOTALYTD** → auto-filter to latest month/year in the data
- **CALCULATE(expr, filters)** → evaluate under modified filter context — DAX's most powerful function
- **SAMEPERIODLASTYEAR** → exact calendar year shift | **DATEADD** → flexible interval shift (use for partial-period accuracy)
- **Column chart** = vertical bars | **Bar chart** = horizontal bars
- **Stacked** = cumulative + breakdown | **Clustered** = side-by-side comparison
- **Card visual** = one important standalone number
- **Table** = flat list, no subtotals | **Matrix** = cross-tab with automatic subtotals + drill-down
- **Drill-down** = 2+ fields on one axis → expand/collapse controls appear automatically
- **Line chart** = always for date trends; dates on X-axis; supports secondary Y-axis for mismatched scales
- **Map visual** = Location field + optional Bubble Size for magnitude
- **Slicer** = canvas element | **Filters pane** = outside canvas, 3 scopes (visual/page/all pages)
- **Top N filter** = dynamically ranked, filter-context-aware
- **Report elements**: text boxes, shapes (can frame visuals), buttons (built-in nav/apply/clear actions), images
