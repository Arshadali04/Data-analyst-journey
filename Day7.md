# Day 7: Power Pivot — Data Model, Relationships & DAX KPIs

## 📋 Topics Covered

**Excel — Power Pivot:**
1. What is Power Pivot & Why Use It?
2. Data Model — What It Is & How to Load Data Into It
3. Table Types — Fact Table vs Dimension Table
4. Relationships — One-to-Many, Diagram View, Edit/Delete
5. Star Schema vs Snowflake Schema
6. Building a Cross-Table Pivot Table from the Data Model
7. DAX Basics — What is DAX?
8. KPI using DAX — Previous Month Sales

---

# PART 1: POWER PIVOT 🔋

---

## 1. What is Power Pivot? 🤔

### The Problem Power Pivot Solves

In a standard Pivot Table, all your data must be in **one single sheet**. But real-world data is never like that — you have:

- Sales data in one sheet
- Customer data in another sheet
- Product data in another sheet
- Date/Calendar data in another sheet

To build a single Pivot Table from all of these, you'd normally have to VLOOKUP everything into one master sheet — slow, error-prone, and breaks with large data.

**Power Pivot solves this entirely.**

### What is Power Pivot?

> *"Power Pivot is an Excel add-in that allows you to load large amounts of data from multiple sources, create relationships between them, and build powerful Pivot Tables and reports — all without VLOOKUP."*

### Key Capabilities

| Feature | Description |
|---------|-------------|
| 🗄️ **Data Model** | Stores multiple tables in memory simultaneously |
| 🔗 **Relationships** | Links tables by common columns — no VLOOKUP needed |
| 📊 **Cross-Table Pivot Table** | Build reports pulling data from different tables at once |
| 🧮 **DAX Formulas** | Powerful formula language for custom KPIs and calculations |
| 📦 **Large Data** | Handles millions of rows that Excel sheets cannot |

### How to Access Power Pivot

**Excel 2016/2019/365 (Professional/Plus editions):**
```
Data Tab → Manage Data Model
```
Or:
```
Power Pivot Tab → Manage
```

> **Note:** If you don't see the Power Pivot tab, go to:
> `File → Options → Add-ins → COM Add-ins → Go → Check "Microsoft Power Pivot for Excel" → OK`

---

## 2. The Data Model 🗃️

### What is the Data Model?

The **Data Model** is an in-memory database inside Excel that stores multiple tables and their relationships. Think of it as a mini database engine running inside your Excel file.

**Analogy:**
- Regular Excel sheets = individual files on your desktop
- Data Model = a database folder that holds all files and knows how they connect to each other

### How to Add Data to the Data Model

**Method 1: From an Excel Table**
1. Select your data → `Ctrl + T` → Convert to Table → OK
2. Go to **Power Pivot tab → Add to Data Model**

**Method 2: While importing via Power Query**
- Data → Get & Transform → load and choose "Add to Data Model"

**Method 3: While inserting a Pivot Table**
- Insert → PivotTable → Check **"Add this data to the Data Model"** → OK

### Viewing the Data Model (Power Pivot Window)
```
Power Pivot Tab → Manage
```
Opens the **Power Pivot window** — a separate interface showing all tables loaded into the Data Model.

- Each table appears as a **tab** at the bottom (like sheets in Excel)
- You can switch between tables by clicking their tabs
- The data looks like a spreadsheet but is stored in compressed memory

### Two Views in Power Pivot Window

| View | Purpose |
|------|---------|
| **Data View** | See the actual table data row by row |
| **Diagram View** | See all tables as boxes and draw relationships between them visually |

Switch between them using the icons at the bottom-right of the Power Pivot window.

---

## 3. Fact Table vs Dimension Table 📦

Understanding these two types is fundamental to data modeling.

### Dimension Table (The "Who/What/When" Table)

> *"A Dimension Table is where you have UNIQUE records."*

- Contains descriptive, categorical data
- Each row is unique — no duplicates on the key column
- Examples: Customer Table, Product Table, Store Table, Date Table

| Customer ID | Customer Name | Gender | City |
|-------------|---------------|--------|------|
| C001 | Rahul Sharma | Male | Mumbai |
| C002 | Priya Verma | Female | Delhi |
| C003 | Amit Patel | Male | Surat |

Every Customer ID appears **only once** — this is a Dimension Table.

### Fact Table (The "Transaction/Event" Table)

> *"A Fact Table is where you have MULTIPLE records for the same key."*

- Contains transactional/measurable data
- The same Customer ID, Product ID, etc. appear **multiple times** (once per transaction)
- Examples: Sales Table, Orders Table, Payments Table

| Order ID | Customer ID | Product ID | Order Date | Net Revenue |
|----------|-------------|------------|------------|-------------|
| O001 | C001 | P12 | 2023-01-05 | 5000 |
| O002 | C001 | P15 | 2023-01-08 | 3200 |
| O003 | C002 | P12 | 2023-01-09 | 4100 |

Customer C001 appears **twice** — this is a Fact Table.

### Quick Rule to Identify

| Question | Answer → Table Type |
|----------|---------------------|
| Does each key value appear only once? | ✅ Dimension Table |
| Does the same key repeat multiple times? | ✅ Fact Table |

---

## 4. Relationships 🔗

### What is a Relationship?

A **Relationship** is a connection between two tables using a **common column** (a shared key). It tells the Data Model: *"These two tables are related — use this column to match their records."*

### Why Relationships Instead of VLOOKUP?

| VLOOKUP Approach | Relationship Approach |
|------------------|-----------------------|
| Adds new columns to the sheet | No changes to source tables |
| Slows down with large data | Fast — handled in memory |
| Breaks when columns are inserted | Stable and dynamic |
| Manual effort for each column | Set once, works everywhere |

### One-to-Many Relationship (The Only Type in Power Pivot)

In Power Pivot, all relationships are **One-to-Many**:
- **One** side → Dimension Table (unique values)
- **Many** side → Fact Table (repeated values)

**Example:**
- Customer Table: each Customer ID appears once → **"One" side**
- Sales Table: same Customer ID appears in many rows → **"Many" side**

The relationship goes: `Customer[Customer ID]` → `Sales[Customer ID]`

### The Filter Flow Arrow

In Diagram View, each relationship shows an **arrow**:
- The arrow points **from the Dimension Table → to the Fact Table**
- This arrow represents the **filter flow direction**
- Meaning: when you filter by a Customer, that filter flows into the Sales table

> *"This arrow denotes filter flow — filters flow from the Dimension Table into the Fact Table."*

### Creating a Relationship in Diagram View

**Step 1: Open Diagram View**
```
Power Pivot Window → Diagram View (icon at bottom-right)
```

**Step 2: Locate the common column**
- In the Dimension Table box, find the key column (e.g., `Customer ID`)
- In the Fact Table box, find the matching column (e.g., `Customer ID`)

**Step 3: Drag and Drop**
- Click and drag from the key column in one table
- Drop it onto the matching column in the other table
- A relationship line with an arrow appears automatically

**Step 4: Verify**
- Hover over the relationship line — both connected columns get highlighted
- The arrow direction confirms which side is "one" and which is "many"

### Creating Multiple Relationships (Example Setup)

For a typical retail dataset:

| Relationship | From (Dimension) | To (Fact) |
|--------------|-----------------|-----------|
| Date → Sales | `Date[Order Date]` | `Sales[Order Date]` |
| Customer → Sales | `Customer[Customer ID]` | `Sales[Customer ID]` |
| Product → Sales | `Product[Product ID]` | `Sales[Product ID]` |
| Store → Sales | `Store[Store ID]` | `Sales[Store ID]` |

Each one is created by drag-and-drop in Diagram View.

### Editing a Relationship

**Method 1:** Right-click the relationship line → **Edit Relationship**
- Opens a dialog showing both connected columns
- Matching columns are highlighted — confirms it's correct

**Method 2:** Power Pivot window → **Design tab → Manage Relationships**
- Lists all relationships in a table
- Edit or Delete from here

### Deleting a Relationship
Right-click the relationship line in Diagram View → **Delete**

### Making a Relationship Inactive
Right-click the relationship line → **Mark as Inactive**

**When to use:** When you have two possible relationships between the same tables (e.g., Sales has both Order Date and Ship Date, both linking to the Date Table), only one can be active at a time. The other is marked inactive and activated via DAX using `USERELATIONSHIP()` when needed.

> Instructor analogy: *"Mark as Inactive = temporary divorce — the relationship exists but isn't being used right now."*

---

## 5. Star Schema vs Snowflake Schema ⭐❄️

### Star Schema

> *"When you have ONE Fact Table surrounded by multiple Dimension Tables — this is called a Star Schema."*

```
         [Date Table]
              |
[Product] — [Sales Table] — [Customer]
              |
         [Store Table]
```

- One central Fact Table
- Multiple Dimension Tables connected directly to it
- Looks like a star when drawn in Diagram View
- **Most common schema in Power Pivot / Power BI**

### Snowflake Schema

> *"When you have MULTIPLE Fact Tables — you go into Snowflake Schema."*

- Multiple Fact Tables connected to each other and to Dimension Tables
- More complex structure
- Used in advanced data warehousing
- **Not typically covered at this level** — requires Power BI / advanced modeling

### Key Rule for This Course
- One Fact Table + Multiple Dimension Tables = **Star Schema** ✅
- Multiple Fact Tables = **Snowflake Schema** (advanced topic)

---

## 6. Cross-Table Pivot Table from Data Model 📊

### The Power: Pulling Data from Two Different Tables

**Scenario:**
- `Month Name` is in the **Date Table**
- `Net Revenue` is in the **Sales Table**
- Both are loaded into the Data Model with a relationship

Normally in a regular Pivot Table, you can't combine fields from two different tables. But because of the relationship in the Data Model, Power Pivot makes this possible.

### Step-by-Step: Building the Cross-Table Pivot Table

**Step 1: Insert Pivot Table from Data Model**
```
Insert Tab → PivotTable
```
In the dialog:
- Select **"Use this workbook's Data Model"** (not "From Table/Range")
- Choose New Worksheet or Existing Worksheet
- Click OK

> **Why "From Data Model"?** Because all your tables are loaded there — not in regular Excel sheets. If you select "From Table/Range", you only get one table's fields.

**Step 2: Expand tables in the Field List**
On the right side, you'll see all your Data Model tables listed.
- Click the **arrow (►)** next to a table name to expand it and see its columns

**Step 3: Drag fields from different tables**
- Expand **Date Table** → drag `Month Name` → **Rows**
- Expand **Sales Table** → drag `Net Revenue` → **Values**
- Result: Month-wise Net Revenue — even though Month and Revenue are in two separate tables!

**This only works because of the relationship you built between them.**

### Example 2: Customer-wise Revenue
- Duplicate the Pivot Table (Copy → Paste next to it)
- Remove `Month Name` from Rows
- Expand **Customer Table** → drag `Customer Name` → **Rows**
- Keep `Net Revenue` in Values
- Result: Customer-wise Revenue report — again pulling from two tables

### Duplicating a Pivot Table
```
Click on the Pivot Table → Ctrl + A → Ctrl + C → Click a blank cell → Ctrl + V
```
Then modify the fields in the copy independently.

---

## 7. What is DAX? 🧮

### Definition

> **DAX = Data Analysis Expressions**

DAX is the **formula language** used in Power Pivot (and Power BI) to create custom calculations, KPIs, and measures that go beyond what standard Pivot Table values can do.

### DAX vs Regular Excel Formulas

| Excel Formulas | DAX |
|----------------|-----|
| Works on cell ranges (`A2:A100`) | Works on entire table columns (`Sales[Revenue]`) |
| Row-by-row logic | Context-aware — adapts to filters automatically |
| Used in worksheet cells | Used inside Power Pivot / Power BI |
| Limited to one table | Can reference multiple related tables |

### Two Types of DAX Calculations

| Type | Where it lives | What it does |
|------|---------------|-------------|
| **Calculated Column** | Added as a new column in the Data Model table | Row-by-row calculation (like a regular formula) |
| **Measure** | Lives in the Values area of a Pivot Table | Aggregates data based on current filter context |

> For KPIs like "Previous Month Sales", we use **Measures** — because they must respond dynamically to whatever filter (month, year, slicer) is currently applied.

### DAX Syntax Basics

```
Measure Name = DAX_FUNCTION(Table[Column])
```

Example:
```
Total Revenue = SUM(Sales[Net Revenue])
```

- Table name comes first: `Sales`
- Column name in square brackets: `[Net Revenue]`
- This is different from Excel where you write cell ranges like `C2:C100`

---

## 8. KPI Using DAX — Previous Month Sales 📅

### What is a KPI?

> **KPI = Key Performance Indicator**

A metric that helps management quickly assess performance. "Previous Month Sales" is a classic KPI — it lets you compare current performance against last month.

### The Problem Without DAX

In a regular Pivot Table, you can show current month's sales. But showing **the previous month's sales right next to the current month** — in the same row — requires DAX. You can't do this with standard Pivot Table features alone.

### DAX Time Intelligence Functions

DAX has a special set of functions called **Time Intelligence** functions that work with dates automatically. The key one for this KPI:

```
PREVIOUSMONTH()
```

This function returns the previous month's date range — and when combined with `CALCULATE()`, it pulls the sales for that period.

### The DAX Measure: Previous Month Sales

```dax
Previous Month Sales = 
CALCULATE(
    SUM(Sales[Net Revenue]),
    PREVIOUSMONTH(Date[Order Date])
)
```

**Breaking it down:**

| Part | What it Does |
|------|-------------|
| `CALCULATE(...)` | Evaluates a formula in a **modified filter context** |
| `SUM(Sales[Net Revenue])` | The base calculation — total revenue |
| `PREVIOUSMONTH(Date[Order Date])` | Changes the filter context to the previous month's dates |

**In plain English:**
> *"Calculate the Sum of Net Revenue, but instead of using the current month's dates, use the previous month's dates."*

### How to Create the Measure

**Step 1: Open Power Pivot Window**
```
Power Pivot Tab → Manage
```

**Step 2: Go to the Fact Table (Sales Table)**
Click the Sales table tab at the bottom.

**Step 3: Click in the Calculation Area**
The blank area below the table data — this is where measures live.

**Step 4: Type the DAX formula**
```dax
Previous Month Sales = CALCULATE(SUM(Sales[Net Revenue]), PREVIOUSMONTH(Date[Order Date]))
```

**Step 5: Press Enter**
The measure is created and appears in the calculation area.

**Step 6: Add it to the Pivot Table**
Back in Excel, in the Pivot Table Field List:
- Find the measure under the Sales table
- Drag it to **Values**
- It now appears as a new column showing previous month's sales

### How it Behaves with Filters

| Current Month Filter | Previous Month Sales Shows |
|---------------------|---------------------------|
| June | May's sales |
| September | August's sales |
| November | October's sales |
| January (with Year filter) | December of the previous year |

> *"If January is selected and the year filter is set to 2017, Previous Month Sales shows December 2016's figure."*

### Real-World Use Case

When working with **current/live data**, PREVIOUSMONTH() auto-detects today's date, calculates the current month, and automatically shows the previous month's sales — no manual adjustment needed. This makes it extremely valuable for monthly dashboards and management reports.

### Why This KPI Matters for Interviews

- Demonstrates understanding of **Time Intelligence** in DAX
- Shows you can build **dynamic KPIs** that update automatically
- Commonly asked in Data Analyst interviews: *"How would you calculate month-over-month comparison?"*

---

## 🎯 KEY TAKEAWAYS — Power Pivot

| Concept | Key Point |
|---------|-----------|
| **Power Pivot** | Handles multiple tables in memory — no VLOOKUP needed |
| **Data Model** | In-memory database inside Excel storing all tables and relationships |
| **Fact Table** | Transaction data — same key repeats multiple times |
| **Dimension Table** | Reference data — each key appears only once (unique) |
| **Relationship** | Links two tables via a common column; always One-to-Many in Power Pivot |
| **Filter Flow** | Arrow goes from Dimension Table → Fact Table; filters flow in that direction |
| **Diagram View** | Visual interface to create/edit relationships by drag-and-drop |
| **Star Schema** | One Fact Table + multiple Dimension Tables — most common setup |
| **Snowflake Schema** | Multiple Fact Tables — more complex, used in advanced modeling |
| **Cross-Table Pivot** | Select "From Data Model" when inserting Pivot Table to use multiple tables |
| **DAX** | Formula language for Power Pivot; works on columns, not cell ranges |
| **CALCULATE()** | Modifies the filter context for a DAX calculation |
| **PREVIOUSMONTH()** | Time Intelligence function; returns previous month's date range |
| **Inactive Relationship** | Exists but unused — activated in DAX via `USERELATIONSHIP()` |

---

## ⚡ Pro Tips — Power Pivot

1. **Always convert source data to Excel Tables** (`Ctrl + T`) before adding to the Data Model — tables auto-expand when new rows are added
2. **Dimension Table = Unique values** — if your key column has duplicates, clean it first or the relationship will fail
3. **Filter flow direction matters** — if your Pivot Table shows wrong totals, check which direction the arrow points in Diagram View
4. **Never write VLOOKUP after learning Power Pivot** — relationships are faster, cleaner, and don't break
5. **Hover over relationship lines** in Diagram View to see which columns are connected — great for debugging
6. **Mark as Inactive** when you have two relationships between the same tables — don't delete the second one; you may need it via DAX
7. **DAX measures are dynamic** — they recalculate automatically based on whatever slicer/filter is active in the Pivot Table
8. **CALCULATE() is the most important DAX function** — almost every advanced KPI uses it to modify filter context
9. **Time Intelligence functions** (PREVIOUSMONTH, SAMEPERIODLASTYEAR, DATESYTD) require a properly set up **Date Table** with no gaps
10. **Use Diagram View** regularly to keep your data model organized — color-code or arrange tables clearly as your model grows

---

## Practice Exercises — Power Pivot

- [ ] Load 3 tables (Sales, Customer, Product) into the Data Model
- [ ] Open Diagram View and identify which tables are Fact and which are Dimension
- [ ] Create a One-to-Many relationship between Customer Table and Sales Table
- [ ] Create a relationship between Product Table and Sales Table
- [ ] Build a cross-table Pivot Table showing Month-wise Net Revenue
- [ ] Build a second Pivot Table showing Customer-wise Net Revenue from the same Data Model
- [ ] Write a basic DAX measure: `Total Sales = SUM(Sales[Net Revenue])`
- [ ] Write the Previous Month Sales DAX measure using CALCULATE + PREVIOUSMONTH
- [ ] Add the measure to a Pivot Table and verify it shows the correct previous month values
- [ ] Apply a Year slicer and confirm the Previous Month Sales for January shows December of the prior year

---

## 📌 Interview Questions — Power Pivot & Data Modeling

| Question | Key Answer Points |
|----------|-------------------|
| What is a Data Model in Excel? | In-memory database storing multiple tables and their relationships inside Excel |
| What is the difference between a Fact Table and a Dimension Table? | Fact = repeated transactional records; Dimension = unique reference records |
| What type of relationship does Power Pivot support? | One-to-Many only (natively, without DAX) |
| What is Star Schema? | One Fact Table + multiple Dimension Tables connected to it |
| What is DAX? | Data Analysis Expressions — formula language for Power Pivot and Power BI |
| What does CALCULATE() do in DAX? | Evaluates a formula inside a modified filter context |
| How do you show Previous Month Sales in a Pivot Table? | Use `CALCULATE(SUM(...), PREVIOUSMONTH(Date[Column]))` |
| Why use Power Pivot over VLOOKUP? | Faster, scalable to millions of rows, no helper columns, dynamic relationships |

---