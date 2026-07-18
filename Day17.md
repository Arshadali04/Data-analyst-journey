# Day 17: Data Transformation (Power Query) & Data Modeling

## 📋 Topics Covered

1. Power Query Editor — overview and the "query vs table" concept
2. M Language — the engine behind every Power Query step
3. Data types in Power BI and why they matter
4. Data cleaning: removing columns, rows, duplicates, blanks
5. Find & Replace (Replace Values)
6. Splitting and merging columns
7. Text formatting & Extract options
8. Reshaping data — Transpose, Pivot, Unpivot
9. Combining tables — Append Queries vs Merge Queries
10. Custom columns and Conditional columns (calculated fields in Power Query)
11. Data source settings & refreshing data
12. Data Modeling — table relationships, cardinality, cross-filter direction
13. Fact tables vs Dimension tables
14. Introduction to DAX (Data Analysis Expressions)

---

# PART 1: Introduction to Power Query

## What is Power Query

Power Query is a **built-in data cleaning and transformation add-in** inside Power BI (also available in Excel). Whenever you click **Transform Data** after connecting to a source, Power BI opens a separate window — the **Power Query Editor** — where you shape your data *before* it's loaded into the report.

### Why It Matters
Raw data is almost never analysis-ready — blank rows, wrong data types, redundant columns, inconsistent headers are the norm, not the exception. Power Query lets you fix all of this **without writing code**, using a repeatable, auditable sequence of steps.

### Query vs Table — Key Distinction
> In Power Query, your data is called a **"query"**, not a "table." This is because you're actively modifying/filtering/reshaping it in multiple steps before it becomes a final table loaded into Power BI Desktop.

### The Applied Steps Panel
Every single change you make in Power Query — promoting headers, removing a column, filtering rows — gets recorded as a **step** under **Applied Steps** (Query Settings pane, right side). This means:
- You can click any step to see exactly what the data looked like at that point
- You can delete/undo any step at any time
- Steps run **sequentially**, top to bottom

```
Source → Navigation → Promoted Headers → Changed Type → [Your custom steps...]
```

---

# PART 2: Core Concepts

## 1. M Language

### Definition
**M (Mashup Language)** is the formula language that powers every action in Power Query. Every step you perform via clicks generates M code behind the scenes, visible in the **Formula Bar**.

### Why It Matters
You rarely need to *write* M manually as a beginner — but being able to *read* it helps you understand exactly what a step did, and troubleshoot when Power Query auto-applies an incorrect step.

### Interview Tip
Common question: *"What language does Power Query use?"* → **M language**, distinct from DAX (used in the data modeling layer).

---

## 2. Data Types

### Definition
Every column in Power BI must have an assigned data type — this tells Power BI how to store, aggregate, and display that column's values.

### Why It Matters
- Correct data types optimize storage
- Power BI only allows aggregation (sum, average, etc.) on numeric types — a text-typed number column can silently break your visuals
- Wrong date/time types cause chart and slicer issues later

### The Three Major Categories

| Category | Sub-types |
|---|---|
| **Numeric** | Whole Number, Decimal, Fixed Decimal, Percentage |
| **Text** | Text |
| **Date/Time** | Date, Date/Time, Time, Duration |
| Other | Boolean (True/False) |

### How to Identify Data Type in the Editor
Look at the small icon next to the column header:
- `ABC` = Text
- `1 2 3` = Whole Number
- 📅 calendar icon = Date
- `ABC 1 2 3` (undefined icon) = Data type not yet assigned

### Common Mistake
Trusting Power Query's **auto-detected** data types blindly. It scans values and guesses — it can get it wrong, especially with date/time columns that have no actual time component (e.g., all values showing `12:00 AM`). Always manually verify, especially for date and numeric columns.

### Best Practice
Convert unnecessary `Date/Time` columns to plain `Date` if there's no meaningful time component — cleaner for slicers and smaller in storage.

---

## 3. Header Promotion — Automatic vs Manual

### Definition
"Promoting headers" means telling Power Query that row 1 of your raw data is actually the **column header**, not a data value.

### When To Use
Power Query **auto-promotes headers** when it detects that most columns have a different data type in row 1 vs the remaining rows (e.g., row 1 = text labels, rest = numbers).

It does **not** auto-promote when all rows — including the header row — are the same data type (e.g., everything is text), because it can't tell the difference. In that case, use the **"Use First Row as Headers"** button manually (sometimes needed twice, if the real header is in row 2).

### Common Mistake
Assuming every connected table will have correctly promoted headers by default — always check the **Applied Steps** to confirm what happened automatically.

---

## 4. Data Cleaning Operations

### 4.1 Removing Columns

**When to use:** You've connected to a wide table (e.g., 100 columns) but only need a subset.

**Method:** `Home → Manage Columns → Choose Columns` → deselect unneeded columns in one go (faster than right-click → Remove on each column individually).

### 4.2 Removing Rows (Redundancy Handling)

| Redundancy Type | How to Handle |
|---|---|
| Duplicate rows | `Home → Reduce Rows → Remove Rows → Remove Duplicates` |
| Blank/null rows | `Home → Reduce Rows → Remove Rows → Remove Blank Rows` |
| Error values | `Home → Reduce Rows → Remove Rows → Remove Errors` |
| Business-specific (e.g., only "Consumer" segment) | Use column filter dropdown |

### ⚠️ Critical Best Practice: Remove Duplicates Correctly
Don't apply **Remove Duplicates** on a single selected column unless that's genuinely what you want — it removes duplicates *within that column only*, which is almost never correct for row-level deduplication.

**Correct approach:** Click the small table icon at the top-left corner of the data grid (selects the whole table, not a specific column) → *then* apply Remove Duplicates. This only removes rows where **every column** matches exactly.

```python
# Conceptually, this mirrors pandas' full-row dedup:
df.drop_duplicates()          # ✅ correct — full row match
df.drop_duplicates(subset=['segment'])   # ⚠️ column-level — usually wrong intent
```

### 4.3 Find & Replace (Replace Values)

**When to use:** Bulk-replacing a value across an entire column — e.g., replacing every "United States" with "US".

**Method:** Select column → `Home → Replace Values` → enter "Value to Find" and "Replace With" → OK.

**Real-world example:** A product name changed in the source system, but historical reports need the new name applied retroactively for consistency — replace values handles this in one step.

---

## 5. Splitting & Merging Columns

### 5.1 Split Column

**When to use:** One column contains multiple pieces of information that need to be separate (e.g., `Order ID` = `US-2024-1057`, combining country code, year, and order number).

**Method:** Select column → `Home/Transform → Split Column → By Delimiter` → choose delimiter (hyphen, comma, colon, custom) → choose split rule:
- **At leftmost delimiter** — split only at the first occurrence
- **At rightmost delimiter** — split only at the last occurrence
- **At each occurrence** — split at every delimiter instance

### ⚠️ Important Behavior
Splitting a column **overwrites the original column** — it disappears, replaced by the split parts. If you need to preserve the original AND get the split values, **duplicate the column first** (`Right-click → Duplicate Column`), then split the duplicate.

### 5.2 Merge Columns

**When to use:** Opposite of split — combining multiple columns into one (e.g., First Name + Last Name → Full Name).

**Method:** `Transform tab → Merge Columns` (requires selecting 2+ columns via Ctrl+Click first) → choose a separator (space, comma, colon, custom) → name the new column.

### Comparison Table

| Feature | Split Column | Merge Columns |
|---|---|---|
| Input | 1 column | 2+ columns |
| Output | Multiple columns | 1 column |
| Original column | Replaced | Original columns remain unless removed manually |
| Location in ribbon | Transform tab | Transform tab → Text Column section |

---

## 6. Extract & Format Options

### Extract
Used to pull out a **portion** of text from a column — similar to `SUBSTRING`/`LEFT`/`RIGHT` functions in SQL/Excel.

| Option | Use Case |
|---|---|
| Length | Get character count of each value |
| First Characters | Keep only the first N characters |
| Last Characters | Keep only the last N characters |
| Text Before Delimiter | Get everything before a specific character |
| Text After Delimiter | Get everything after a specific character |

> ⚠️ Extract **overwrites** the existing column (like Split), rather than creating a new one.

### Format
Used for text case standardization: lowercase, UPPERCASE, Capitalize Each Word, Trim (remove extra spaces), Clean (remove non-printable characters).

---

## 7. Reshaping Data: Transpose, Pivot, Unpivot

This is one of the most interview-relevant Power Query topics — understanding **measures vs dimensions** is the foundation.

### Measures vs Dimensions

| Term | Definition | Example |
|---|---|---|
| **Measure** | Quantitative data that can be aggregated (summed, averaged) | Sales, Profit, Quantity |
| **Dimension** | An attribute used to *view/slice* measures | Zone, Month, Product Category |

### 🎯 Golden Rule
**Measures must always be in columns, never in rows.** Aggregations (SUM, AVERAGE, COUNT) in BI tools operate at the *column* level — if your quantitative data is spread across row labels, you can't aggregate it correctly.

### 7.1 Transpose
**When to use:** Your entire table structure is flipped — what should be columns are currently rows, and vice versa.

**Method:** `Transform tab → Transpose` → converts all rows to columns and columns to rows in one click.

```
Before:                        After Transpose:
Zone    | South | East | North  Zone   | Revenue
Revenue | 500   | 300  | 700    South  | 500
                                East   | 300
                                North  | 700
```
*(Followed by: Use First Row as Header → Remove Blank Rows)*

### 7.2 Pivot Columns
**When to use:** A single column contains a "metric name," and another column has mixed values for multiple metrics (e.g., one `Metric` column contains both "Revenue" and "Profit" labels, with amounts in a shared `Amount` column) — you need each metric as its own column.

**Method:** Select the column with the metric *names* (e.g., `Metric`) → `Transform → Pivot Column` → choose the **Values Column** (e.g., `Amount`) → OK.

```
Before:                  After Pivot:
Zone | Metric  | Amount  Zone | Revenue | Profit
South| Revenue | 500     South| 500     | 120
South| Profit  | 120     East | 300     | 80
East | Revenue | 300
East | Profit  | 80
```

### 7.3 Unpivot Columns
**When to use:** The opposite problem — multiple columns represent the *same type* of measure across different categories (e.g., `2021 Revenue`, `2022 Revenue`, `2023 Revenue` as separate columns) and you need them combined into one `Year` + one `Revenue` column.

**Method:** Select all the columns to unpivot (Ctrl+Click) → `Transform → Unpivot Columns` → rename the resulting `Attribute`/`Value` columns appropriately (e.g., to `Year` and `Revenue`) → use **Extract → First Characters** to clean up any remaining text (e.g., stripping "Revenue" out of "2021 Revenue").

```
Before:                              After Unpivot:
Zone | 2021 Rev | 2022 Rev | 2023 Rev  Zone  | Year | Revenue
South| 500      | 550      | 600       South | 2021 | 500
                                       South | 2022 | 550
                                       South | 2023 | 600
```

### Decision Guide: Which Reshape Operation?

| Condition | Recommended Approach |
|---|---|
| Entire table's rows/columns are fully swapped | **Transpose** |
| One column has mixed metric names that need to become separate columns | **Pivot** |
| Multiple columns represent the same metric across categories, need to become rows | **Unpivot** |

---

## 8. Combining Tables: Append vs Merge

### Definition
Power Query offers two fundamentally different ways to combine tables — this distinction is a very common interview question.

| Feature | Append Queries | Merge Queries |
|---|---|---|
| Direction | Adds **rows** (stacks tables vertically) | Adds **columns** (joins tables horizontally) |
| SQL Equivalent | `UNION` | `JOIN` |
| Use Case | Two tables with the *same structure*, different records (e.g., Jan orders + Feb orders) | Two *related* tables sharing a common key column (e.g., Employees + Managers via Employee ID) |
| Requires common key? | No — just matching column structure | Yes — a common column to join on |

### 8.1 Append Queries
**When to use:** Same columns, different rows — e.g., Employee Table 1 (first batch) + Employee Table 2 (second batch) = one combined Employee table.

**Method:** `Home → Append Queries` (overwrites current query) or `Append Queries as New` (creates a separate combined query) → select 2+ tables → OK.

### 8.2 Merge Queries
**When to use:** Bringing a column from a *related* table into your current table — e.g., adding a `Manager` column into the Employees table by matching on `Employee ID`.

**Method:** `Home → Merge Queries` → select the second table → select the matching column in both tables → choose Join Kind (default: **Left Outer** — keeps all rows from the first table) → OK → expand the new merged column (the "⤢" icon) → select which specific column(s) to bring in.

### Interview Tip
If asked *"How would you combine two tables in Power Query?"* — first clarify: **"Are we adding new rows of the same structure, or pulling in related columns from another table?"** That single question demonstrates you understand Append vs Merge.

---

## 9. Custom Columns & Conditional Columns

### 9.1 Custom Column (calculated field)
**When to use:** Simple row-level calculations using existing columns — e.g., `Profit Margin = Profit / Sales`.

**Method:** `Add Column → Custom Column` → name it → build the formula using available fields and arithmetic operators (`+`, `-`, `*`, `/`).

```
Profit Margin = [Profit] / [Sales]
```

### 9.2 Conditional Column
**When to use:** IF/ELSE-style logic to categorize rows — e.g., flagging each transaction as "High Margin", "Low Margin", or "Loss".

**Method:** `Add Column → Conditional Column` → build IF / Else If / Else clauses visually.

```
IF [Profit Margin] > 0.5        → "High"
ELSE IF [Profit Margin] > 0     → "Low"
ELSE                            → "Loss"
```

### Best Practices
- Keep custom column formulas simple — Power Query's Custom Column editor isn't meant for complex logic (that's DAX's job, at the modeling layer)
- Always set the correct data type on the new column after creating it (e.g., Percentage for a margin column)

---

## 10. Data Source Settings & Refreshing

### Refresh
Clicking the **Refresh** button re-runs the entire connect → transform → load sequence, pulling any new data added at the source since the last load.

### Handling Broken Connections
If a source file/location moves, refresh will throw a **file not found / data source connection error**.

**Fix options:**
1. Move the file back to its original location, **or**
2. Update the connection via `Transform Data (dropdown) → Data Source Settings → Change Source` → point to the new file path/server

### Best Practice
Whenever a data source's physical location is likely to change (shared drives, reorganized folders), document the connection path so troubleshooting refresh errors is faster.

---

# PART 3: Data Modeling

## 11. What is Data Modeling

### Definition
Data Modeling is the second core stage after cleaning — it involves two things:
1. **Building table relationships**
2. **Creating new calculations** (via DAX)

### Why It Matters
Relationships are what let you drag fields from *different* tables into the *same* visual and get correctly filtered, related results — without relationships, Power BI has no idea how tables connect, and visuals will show incorrect, unfiltered totals.

---

## 12. Table Relationships

### Auto-Detection
Power BI automatically creates relationships when it detects **two columns with the same name and same data type** across tables. This is convenient but **not always correct** — always verify auto-created relationships in **Model View**.

### Creating a Relationship Manually
Drag the key column from one table and drop it onto the matching column in another table (in Model View) → confirm the relationship in the prompt.

### Deleting a Relationship
Right-click the relationship line in Model View → Delete.

### Real-World Impact (Demonstration)
Without a relationship between `Customers` and `Orders`, a table visual combining `Company Name` and `Total Freight` will show the **same overall total repeated for every row** — no filtering happens. With the relationship restored, each company correctly shows only its own freight total.

---

## 13. Cardinality

### Definition
**Cardinality** = the uniqueness of values in the relationship column, and how rows in one table relate to rows in another.

### The Four Cardinality Types

| Type | Meaning | Example |
|---|---|---|
| **One-to-Many (1:*)** | One row in Table A relates to many rows in Table B | One Customer → Many Orders |
| **Many-to-One (*:1)** | Same relationship, viewed from the other direction | Many Orders → One Customer |
| **One-to-One (1:1)** | Exactly one matching row on each side | Employee → Employee Login Record |
| **Many-to-Many (*:\*)** | Multiple rows on both sides can match multiple rows on the other | Students ↔ Courses |

### Interview Tip
1:Many and Many:1 are **the same relationship**, just described from opposite directions — a favorite "gotcha" question.

---

## 14. Cross-Filter Direction

### Definition
Cross-filter direction determines **which table's selections can filter the other table's visuals**.

### Default Behavior
By design, filtering flows from the **"one" side to the "many" side** — this is called **Single** direction. Selecting a Customer filters related Orders, but selecting an Order does NOT filter the Customers table back.

### Changing It: Single → Both
`Right-click relationship → Properties → Cross Filter Direction → Both` allows filtering to flow **in both directions** — useful, but should be used carefully (can cause ambiguous filter paths in complex models with many relationships).

### Decision Guide

| Condition | Recommended Setting |
|---|---|
| Standard dimension → fact relationship | Single (default) |
| Need filtering to flow both ways for a specific visual interaction | Both (change manually) |
| Complex model with multiple many-to-many paths | Be cautious — Both can create ambiguous circular filtering |

---

## 15. Fact Tables vs Dimension Tables

### Definition

| Type | Definition | Contains |
|---|---|---|
| **Fact Table** | Stores transactional information; updates frequently | Order ID, Customer ID, Product ID, Order Date — the "what, who, when" of each transaction |
| **Dimension Table** | Stores descriptive/additional attributes | Product Name, Description, Category, Customer Name, Address, Region |

### Why It Matters
Keeping additional attributes (like full product descriptions) OUT of the fact table and in a separate dimension table keeps the fact table small and fast — a core principle of efficient data modeling (this mirrors star-schema design in traditional data warehousing).

### Real-World Example
- **Fact table**: `Order Details` (Order ID, Product ID, Quantity, Discount) — grows every time a new order happens
- **Dimension table**: `Products` (Product ID, Name, Category, Unit Price) — grows only when new products are introduced, much less frequently

---

## 16. Introduction to DAX

### Definition
**DAX (Data Analysis Expressions)** is Power BI's formula language — used to create calculated columns, measures, and custom aggregations. It sits at the *modeling* layer, distinct from M language (which operates at the *Power Query/transformation* layer).

### Why It Matters
Comparable to Excel formulas — DAX has ~300 functions total, but a working knowledge of **15–20 core functions** covers the vast majority of real-world Data Analyst tasks (similar to how ~30 Excel functions cover most spreadsheet work).

### M vs DAX — Don't Confuse These

| | M Language | DAX |
|---|---|---|
| Used in | Power Query Editor | Data Modeling layer (Report/Table/Model view) |
| Purpose | Data cleaning & transformation | Calculations, measures, aggregations |
| Applied at | Row-by-row transformation stage | After data is loaded into the model |

### Interview Tip
A frequently asked question: *"What's the difference between M and DAX?"* — **M shapes the data before it's loaded; DAX calculates on top of data that's already loaded into the model.**

---

# 🎯 Key Takeaways

| Topic | Key Point |
|---|---|
| Power Query | Built-in tool for cleaning/transforming data before it loads into Power BI |
| M Language | The formula language behind every Power Query step |
| Data Types | Must be correctly assigned for accurate aggregation and optimized storage |
| Remove Duplicates | Always apply at full-row level, not on a single column |
| Split vs Merge Columns | Split = 1 column → many; Merge = many columns → 1 |
| Measures vs Dimensions | Measures = quantitative, must be in columns; Dimensions = attributes to slice by |
| Transpose / Pivot / Unpivot | Transpose flips full table; Pivot creates columns from row values; Unpivot creates rows from column values |
| Append vs Merge Queries | Append = stack rows (UNION); Merge = join columns (JOIN) |
| Custom vs Conditional Column | Custom = arithmetic formula; Conditional = IF/ELSE logic |
| Relationships | Enable cross-table filtering in visuals; auto-detected but must be verified |
| Cardinality | 1:Many, Many:1, 1:1, Many:Many — describes how rows match across tables |
| Cross-filter direction | Default is Single (one → many); can be changed to Both |
| Fact vs Dimension tables | Fact = transactional, frequently updated; Dimension = descriptive attributes |
| DAX | Power BI's formula/calculation language, distinct from M |

---

# ⚡ Pro Tips

- Always check the **Applied Steps** panel after connecting to a new source — Power BI often auto-applies header promotion and type detection, and it can get these wrong.
- When removing duplicates, click the small table-selector icon (not a column header) to ensure you're deduplicating the whole row, not a single column.
- If a Split Column or Extract step accidentally destroys a column you needed to keep, undo it and **duplicate the column first** next time before transforming.
- Remember the **measures-in-columns rule** — it's the single most common reason a Pivot/Unpivot is needed, and it's a strong interview signal when you can explain *why* the reshape is necessary, not just *how*.
- For combining tables, always ask "same structure, different rows?" (Append) vs "related tables, need a column from another?" (Merge) before touching the ribbon.
- Keep DAX and M separate in your mental model: **M = before load (Power Query)**, **DAX = after load (modeling/measures)**.

---

# Practice Exercises

- [ ] Connect to any multi-sheet Excel file and load all sheets into Power Query — identify which sheets need header promotion
- [ ] Take a table with a combined ID column (e.g., `US-2024-1057`) and split it into 3 separate columns using delimiter-based splitting
- [ ] Reverse the above: merge those 3 columns back into one, using a different separator
- [ ] Create a Conditional Column that classifies rows into 3+ categories based on a numeric threshold
- [ ] Given two tables with identical columns but different rows, use **Append Queries as New** to combine them
- [ ] Given two related tables sharing a common ID column, use **Merge Queries** to bring one additional column across
- [ ] In Model View, manually delete a relationship, observe the broken filtering behavior in a table visual, then recreate it
- [ ] (Interview-level) Explain out loud: What's the difference between a Fact table and a Dimension table, using your own example

---

# Mini Project

**Objective:** Practice the full clean → model pipeline on a multi-table dataset

**Dataset:** Any public "Orders + Customers + Products" style dataset (e.g., Northwind sample data via OData, or any Kaggle e-commerce dataset with multiple related tables)

**Tasks:**
1. Connect to all related tables via Power Query
2. Clean each table (fix headers, remove nulls, correct data types)
3. Create a calculated `Profit Margin` custom column and a `Margin Type` conditional column
4. Verify/create relationships between tables in Model View
5. Correctly identify which tables are Fact tables and which are Dimension tables
6. Build one table visual that pulls fields from at least 3 different related tables to confirm relationships are working

**Skills Practiced:** Power Query cleaning, column splitting/merging, pivot/unpivot, relationship building, cardinality reasoning, fact/dimension classification

---

# Interview Preparation

## Frequently Asked Questions

**Q: What is Power Query and what language does it use?**
A: Power Query is Power BI's built-in data cleaning/transformation tool, using the M (Mashup) language behind every applied step.

**Q: What's the difference between Append and Merge queries?**
A: Append stacks tables with the same structure vertically (adds rows, like SQL UNION). Merge joins related tables horizontally on a common key (adds columns, like SQL JOIN).

**Q: What's the difference between Pivot and Unpivot?**
A: Pivot turns unique row values into new columns (used when a metric-name column mixes multiple measures). Unpivot turns multiple columns into rows (used when the same measure is spread across separate columns, e.g., by year).

**Q: What is cardinality in Power BI relationships?**
A: It describes how rows in one table relate to rows in another — One-to-Many, Many-to-One, One-to-One, or Many-to-Many — based on the uniqueness of the relationship column's values in each table.

**Q: What's the default cross-filter direction and why?**
A: Single, flowing from the "one" side to the "many" side — because dimension tables (the "one" side) are meant to filter fact tables (the "many" side), not the reverse, by design.

**Q: Difference between Fact and Dimension tables?**
A: Fact tables store frequently-updating transactional data (what/who/when of each event). Dimension tables store descriptive attributes (names, categories, addresses) that don't change as often.

## Scenario-Based Questions

**Scenario:** You connect to a table where the header row is showing as "Column1, Column2, Column3" instead of proper field names — Power Query didn't auto-detect it. What do you do?
**Answer:** Manually apply "Use First Row as Headers" from the Home tab — this usually happens when all row values (including the actual header row) share the same data type (e.g., all text), so Power Query's auto-detection can't distinguish header from data.

**Scenario:** A table visual shows the same total value repeated across every row, regardless of which dimension you add. What's likely wrong?
**Answer:** Missing or broken relationship between the tables involved — without it, Power BI can't filter the measure by the dimension, so it just repeats the grand total.

## Practical Questions

- Given an `Order_ID` column formatted as `CA-2023-00456`, write out the steps to extract just the numeric order number into its own column, while preserving the original column.
- You have monthly revenue in 12 separate columns (`Jan_Revenue` … `Dec_Revenue`). Describe the exact steps to reshape this into a `Month` + `Revenue` two-column format.

---

# Revision Cheat Sheet

- **Power Query** = data cleaning layer, uses **M language**
- **Applied Steps** = sequential, editable log of every transformation
- **Remove Duplicates** → always full-row level (use the table-selector icon, not a column)
- **Split Column** (1→many) vs **Merge Columns** (many→1) — both live under Transform tab
- **Extract** = pull a text substring; **Format** = case/whitespace cleanup
- **Measures → columns, always.** Reshape data if they're not.
- **Transpose** = flips entire table | **Pivot** = row values → columns | **Unpivot** = columns → rows
- **Append** = stack rows (same structure) | **Merge** = join columns (common key)
- **Custom Column** = formula | **Conditional Column** = IF/ELSE logic
- **Relationships**: auto-detected but verify manually; drag-drop to create, right-click to delete
- **Cardinality**: 1:Many / Many:1 / 1:1 / Many:Many
- **Cross-filter direction**: default = Single (one → many); can set to Both
- **Fact table** = transactional, frequent updates | **Dimension table** = descriptive attributes
- **DAX** = modeling-layer formula language (post-load) — distinct from M (pre-load)
