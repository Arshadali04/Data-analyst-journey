# Day 6: Excel Pivot Tables — Complete Guide & MIS Dashboard Project

## 📋 Topics Covered

**Excel:**
1. What is a Pivot Table? (Definition & Why Use It)
2. Creating a Basic Pivot Table
3. Recommended Pivot Tables
4. Pivot Chart
5. Slicers — Inserting, Settings & Report Connection
6. Timeline
7. Grouping & Custom Grouping
8. Pivot Table Options (All Settings)
9. Show Values As
10. Calculated Fields & Calculated Items
11. Filters in Pivot Table
12. Dynamic Pivot Table (OFFSET Formula + Table Method)
13. Pivot Table on Multiple Sheets (Power Query)
14. GETPIVOTDATA Formula
15. MIS Report — End-to-End Project (Dashboard with Macros)

---

# PART 1: PIVOT TABLES 📊

---

## 1. What is a Pivot Table? 🤔

### Definition (Interview-Ready)
> *"Without using any formula, we can get a summarized record of our data."*

A Pivot Table converts raw rows-and-columns data into **interactive 2D summary reports** — without writing a single formula.

### Why Use It? (The "Sharma Ji" Problem)
When someone gives you a large dataset (thousands of rows), and asks questions like:
- Which 5 countries have the highest sales?
- Which region sold the most products?
- What is the category-wise top product?

Answering these manually — using SUMIFS, COUNTIFS, etc. — takes a long time. A Pivot Table answers all of them in seconds with **drag and drop**.

### What Pivot Tables Can Do
| Capability | Description |
|------------|-------------|
| **Summarize** | Total, count, average, max, min — without formulas |
| **2D Reports** | Row and column breakdown simultaneously |
| **Visualize** | Convert to Pivot Charts instantly |
| **Slice & Filter** | Use Slicers and Timelines for interactive filtering |

---

## 2. Creating a Basic Pivot Table 🛠️

### Prerequisites (Important Disclaimer)
Before inserting a Pivot Table, your data must be clean:
- ✅ **No merged cells** — every column must have its own individual heading
- ✅ Headers must be in a single row, not spread across merged cells
- ✅ No blank rows or columns inside the data range
- ✅ Do NOT select a merged title row at the top — select only the data

### Step-by-Step

**Step 1: Select the data**
```
Ctrl + A (select all data)
```

**Step 2: Insert Pivot Table**
```
Insert Tab → PivotTable
```
Or use keyboard shortcut:
```
Alt → N → V  (sequential, not simultaneous)
```

**Step 3: In the dialog box**
- **Select Table/Range** → auto-filled if you selected data first
- **Choose where to place the report:**
  - **New Worksheet** → creates a fresh sheet (recommended for large data)
  - **Existing Worksheet** → place in the current sheet (select a blank cell with space to the right)

**Step 4: Click OK**

### The Pivot Table Task Pane (Field List)
After inserting, a **Field List panel** appears on the right side — only when your cursor is inside the Pivot Table.

> **Common Mistake:** If the Field List disappears, click inside the Pivot Table first. If it's been closed:
> - Go to **PivotTable Analyze tab** (only visible when cursor is in Pivot Table)
> - Click **Field List** to toggle it back on

### The Four Areas (Drag & Drop)
| Area | Purpose |
|------|---------|
| **Filters** | Top-level filter dropdowns |
| **Columns** | Horizontal headers (categories across columns) |
| **Rows** | Vertical list (categories down rows) |
| **Values** | Numeric data to summarize (Sum, Count, Average, etc.) |

### Building a Report: Country-Wise Sales Example
1. Drag `Country` → **Rows**
2. Drag `Sales` → **Values**
3. Result: Instant country-wise total sales — no formula needed

### 2D Report (Two-Way Breakdown)
- Drag `Country` → **Rows**
- Drag `Region` → **Columns**
- Drag `Sales` → **Values**
- Result: A matrix showing sales by country AND region simultaneously

### Expand/Collapse Groups
When you have multiple fields in Rows, a **+ / − button** appears next to each group:
- Click **+** to expand and see sub-items
- Click **−** to collapse back

---

## 3. Recommended Pivot Tables ⚡

Available in **Excel 2013 and later** only.

**How to use:**
1. Select data (Ctrl + A)
2. Go to **Insert → Recommended PivotTables**
3. Excel suggests ready-made reports based on your data
4. Preview and select the one you want → Click **OK**

**What it does:** Auto-inserts a Pivot Table with fields already dragged and dropped — no manual configuration needed. Saves time for quick reporting.

> If you want a blank Pivot Table from this dialog, click **Blank PivotTable**.

---

## 4. Pivot Chart 📈

### Two Ways to Insert a Pivot Chart

**Method A — From scratch (simultaneous Table + Chart):**
1. Select data → **Insert → PivotChart**
2. Choose chart type → Click OK
3. Both a Pivot Table AND a Pivot Chart are created together

**Method B — From an existing Pivot Table:**
1. Click inside the existing Pivot Table
2. **PivotTable Analyze → PivotChart**
3. A chart linked to that table is created

### Customizing the Chart
- **Change chart type:** Design tab → **Change Chart Type**
- **Add data labels:** Right-click on chart → **Add Data Labels**
- **Format data labels:** Right-click labels → **Format Data Labels**
  - Choose: Category Name, Value, Percentage — toggle as needed
- **Move legend:** Design → **Add Chart Element → Legend** → Top/Bottom/Left/Right/None
- **Change background:** Right-click chart area → **Format Chart Area** → Fill options
- **Hide field buttons on chart:** Right-click any field button on chart → **Hide All Field Buttons on Chart**

### Pie Chart with Percentages
1. Change Chart Type to Pie
2. Select a Percentage style in Design gallery
3. Right-click labels → Format Data Labels → check **Category Name** + **Percentage**, uncheck Value

---

## 5. Slicers — Smart Visual Filters 🔲

### What is a Slicer?
A **visual filter button panel** that lets you filter Pivot Table data by clicking, instead of using dropdown menus. Much faster and more presentation-friendly.

### How to Insert a Slicer
1. Click inside the Pivot Table
2. **PivotTable Analyze → Insert Slicer**
3. Check the field(s) you want to filter by → Click OK
4. A slicer panel appears — click any value to filter instantly

### Multiple Slicers
You can insert multiple slicers in one go:
- In the Insert Slicer dialog, check multiple fields at once
- Each field gets its own slicer panel

### Selecting Multiple Items in a Slicer
- **Single select:** Click one item (replaces current selection)
- **Multi-select (via icon):** Click the **Multi-Select (≡)** icon in the slicer header, then click multiple items
- **Multi-select (keyboard):** Hold **Ctrl** and click multiple items
- **Range select:** Click first item, then hold **Shift** and click last item
- **Clear filter:** Click the **Clear Filter (✕)** button on the top-right of the slicer

### Slicer Settings (Right-click slicer → Slicer Settings)
| Setting | Purpose |
|---------|---------|
| **Display Header** | Show/hide the title bar of the slicer |
| **Ascending / Descending** | Sort the items inside the slicer |
| **Hide items with no data** | Remove items that have zero matching records |
| **Show items with no data last** | Keep empty items but move them to the bottom |
| **Caption / Name** | Rename the slicer title |

### Slicer Tab (Format/Style)
When slicer is selected, the **Slicer tab** appears at the top:
- **Columns:** Change how many columns the slicer items display in (e.g., 3 columns = horizontal layout)
- **Button Height/Width:** Resize individual slicer buttons
- **Slicer Styles:** Change color theme of the slicer
- **Bring Forward / Send Backward:** Layer slicers over each other
- **Report Connection:** Connect the slicer to multiple Pivot Tables

### Report Connection (Connecting One Slicer to Multiple Tables)
1. Right-click slicer → **Report Connections** (or **PivotTable Connections**)
2. A dialog shows all Pivot Tables in the workbook
3. **Check** the tables you want this slicer to control
4. **Uncheck** tables you want to exclude

**Use case:** One slicer controlling 3 different Pivot Tables on the same dashboard sheet.

### Custom Slicer Formatting
1. Go to **Slicer tab → Slicer Styles → New Slicer Style**
2. Name it (e.g., "My Slicer")
3. Set colors for:
   - Whole slicer background
   - Header
   - Selected item with data
   - Selected item with no data
   - Unselected items
4. Click OK → select your custom style from the gallery

### Key Fix: Stop Pivot Table from Resizing When Filtered
When using slicers, the Pivot Table column width may jump around.
**Fix:**
1. Right-click Pivot Table → **PivotTable Options**
2. Layout & Format tab → **Uncheck "Autofit column widths on update"**

---

## 6. Timeline 📅

### What is a Timeline?
A **date-based visual filter** — like a slicer but designed specifically for date fields. Lets you filter by Year, Quarter, Month, or Day using a sliding selector.

### Prerequisite
Your data **must have a Date column** (Order Date, Purchase Date, Joining Date, etc.). Without a date column, Timeline cannot be inserted.

### How to Insert a Timeline
1. Click inside the Pivot Table
2. **PivotTable Analyze → Insert Timeline**
3. Select the date column → Click OK
4. Timeline appears with a scrollable bar

### Filtering with Timeline
| Level | How to use |
|-------|-----------|
| **Years** | Click a year bar to filter that year |
| **Quarters** | Click Q1, Q2, Q3, Q4 |
| **Months** | Click individual month |
| **Days** | Click individual day |

- **Range selection:** Click and drag across multiple periods to select a range
- **Clear filter:** Click the **Clear Filter** icon on the timeline
- **Change level:** Use the dropdown on the top-right of the timeline (Years / Quarters / Months / Days)

### Timeline Tab (Format Options)
| Setting | Purpose |
|---------|---------|
| **Timeline Styles** | Change color theme |
| **Report Connection** | Connect to multiple Pivot Tables |
| **Show / Hide** | Toggle header, scrollbar, selection label, time level display |

---

## 7. Grouping & Custom Grouping 📁

### Grouping Date Fields
When you drag a Date column into Rows:
- Excel 2016+ **auto-groups** by Month (and sometimes Year)
- You can **undo** the auto-grouping with `Ctrl + Z`

**Manual grouping:**
1. Right-click any date value in the Pivot Table
2. Click **Group**
3. Choose grouping level(s): **Days / Months / Quarters / Years**
4. You can select multiple levels simultaneously (e.g., Year + Quarter + Month)
5. Click OK

**Ungroup:**
Right-click → **Ungroup**

### Grouping by Number of Days
In the Group dialog:
- Select **Days** only
- Set **Number of days** (e.g., 7 = weekly grouping)

### Custom Grouping (Non-Date Fields)
You can group text items (like countries, products) into custom named groups:

**Example: Group countries into Zones**
1. In the Pivot Table, hold **Ctrl** and click the countries you want in one group
2. Right-click → **Group**
3. A "Group 1" is created — click on it and rename it (e.g., "South Zone")
4. Repeat for other groups
5. Ungroup if needed: Right-click a group → **Ungroup**

### Classic Layout for Better Grouping View
The default (Compact) layout stacks grouped fields. To see each field in its own column:
1. Right-click Pivot Table → **PivotTable Options → Display → Classic PivotTable Layout**
2. Or: **Design → Report Layout → Show in Tabular Form**

In Classic Layout with multiple grouping levels:
- Use **Merge & Center cells with labels** option (in PivotTable Options → Layout & Format) to center group headers across their rows

---

## 8. Pivot Table Options — All Settings ⚙️

Access via: **Right-click Pivot Table → PivotTable Options**

### Layout & Format Tab

| Setting | What It Does |
|---------|-------------|
| **Merge and center cells with labels** | Centers group label across merged rows (useful in Classic Layout) |
| **Indent row labels in compact form** | Sets indentation level for sub-items |
| **Display fields in report filter area: Down, then Over / Over, then Down** | Controls layout direction when multiple filters are stacked |
| **Report filter fields per row/column** | Set how many filters appear per row before wrapping |
| **For error values show:** | Replace error cells (#DIV/0!, #N/A, etc.) with a custom value (e.g., 0 or blank) |
| **For empty cells show:** | Replace blank cells with a custom value |
| **Autofit column widths on update** | ⚠️ **Uncheck this** when building dashboards to prevent layout shifting |
| **Preserve cell formatting on update** | Keep your manual formatting after refresh |

### Totals & Filters Tab

| Setting | What It Does |
|---------|-------------|
| **Show grand totals for rows** | Toggle the grand total row at the bottom |
| **Show grand totals for columns** | Toggle the grand total column on the right |
| **Allow multiple filters per field** | Enable filtering by both Label and Value at the same time |
| **Use Custom Lists when sorting** | Apply custom sort orders |

### Display Tab

| Setting | What It Does |
|---------|-------------|
| **Show expand/collapse buttons** | Show or hide the +/− buttons on grouped rows |
| **Show contextual tooltips** | Show details popup when hovering over a value cell |
| **Display field captions and filter dropdowns** | Show or hide the field names and dropdown arrows |
| **Classic PivotTable layout** | Enable drag-and-drop directly inside the table |
| **Sort A to Z / Sort in data source order** | Default sort behavior for items |

### Printing Tab

| Setting | What It Does |
|---------|-------------|
| **Repeat row labels on each printed page** | Repeat group headers on every page when printing multi-page tables |
| **Set print titles** | Include print titles in output |
| **Print expand/collapse buttons** | Include +/− buttons in printed output |

### Data Tab

| Setting | What It Does |
|---------|-------------|
| **Save source data with file** | Include the source data when saving |
| **Enable show details** | Allow double-clicking a value to drill down to source records |
| **Refresh data when opening the file** | Auto-refresh Pivot Table every time the file opens |
| **Number of items to retain per field** | Set to **None** to remove old/deleted items from filter dropdowns |

> **Key use case for "Number of items to retain per field = None":**
> If you change a value in the source data (e.g., "Pune" → "Surat") and refresh, the old value "Pune" may still appear in the filter dropdown. Setting this to **None** and refreshing removes old ghost values.

### Alt Text Tab
Write a description/title for the Pivot Table — useful as a reference note when you've built many complex reports. Reminds you what filters/conditions were applied.

---

