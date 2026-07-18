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

## 6. 