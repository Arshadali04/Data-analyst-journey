# Day 21: Introduction to Tableau, Public vs Desktop, Data Connections & Interface

## 📋 Topics Covered

1. What is Tableau — overview, ownership, and market position
2. Features and advantages of Tableau
3. Disadvantages of Tableau
4. Tableau Public vs Tableau Desktop — full comparison
5. Installing Tableau Public
6. Connecting Tableau to data sources (Text/CSV, Excel, folders)
7. Tableau interface walkthrough — Workbook, Data Source, Worksheet, Dashboard, Story

---

# PART 1: Understanding Tableau

## Introduction

### What is Tableau
Tableau is a **visual analytics platform** where you build visualizations primarily through **drag-and-drop** — no prior coding or programming knowledge is required. Founded in 2003, it was acquired by **Salesforce in 2019**. According to industry reports, Tableau consistently ranks among the top data visualization tools used across companies of all sizes.

### Why It Matters
Whether you're a Data Analyst, Data Scientist, or transitioning into data roles, Tableau is one of the standard tools expected in the industry for building interactive dashboards and telling data stories.

### How Tableau Works Under the Hood
Every drag-and-drop action in Tableau is translated, behind the scenes, into **SQL queries** that fetch and optimize data from the connected source — you never have to write the SQL yourself, but it's useful to know this is happening.

---

# PART 2: Core Concepts

## 1. Features & Advantages of Tableau

### Definition
Tableau offers a full product ecosystem: Tableau Public, Tableau Desktop, Tableau Server, and an active community — compatible with both **Windows and macOS**.

### Key Advantages

| Advantage | Details |
|---|---|
| No coding required | Drag-and-drop interface; SQL is generated automatically in the background |
| Wide data connectivity | Connects to cloud sources, CSV, Excel, databases, and real-time website data |
| Quick Calculations | Pre-built formulas for common financial/statistical calculations — no complex manual formula-writing |
| Interactive dashboards | Build across multiple sheets, present as a Story, and share with your organization |
| Handles large datasets | Performs efficiently even with large-scale enterprise data |

### Interview Tip
If asked *"Why use Tableau over writing raw SQL/Excel reports?"* — highlight the combination of **no-code visualization + wide connector support + efficient large-data handling**, which collectively speed up the analysis-to-insight cycle.

---

## 2. Disadvantages of Tableau

### Definition
No tool is without trade-offs — Tableau has two commonly cited drawbacks.

| Disadvantage | Details |
|---|---|
| **High cost** | Comparatively more expensive than several other visualization tools on the market |
| **Limited data pre-processing** | Cleaning/transformation capabilities are far more limited than tools like Power BI — deep pre-processing generally requires a separate tool (e.g., Excel, or Tableau Prep on paid tiers) |

### Interview Tip
A fair, balanced answer to *"What are Tableau's limitations?"* signals maturity — mention cost and limited native data prep, and note that **Tableau Prep** exists specifically to address the second point on paid tiers.

---

## 3. Tableau Public vs Tableau Desktop

### Definition
Both share the same core visualization features — the differences lie in **data connectivity, data volume, storage/sharing, and cost**.

### Comparison Table

| Feature | Tableau Public | Tableau Desktop |
|---|---|---|
| **Data sources** | ~7 basic connectors (Excel, text/CSV, spatial, statistical files) | Extensive — cloud databases, data warehouses, data lakes, many database types |
| **Data volume** | Up to 1.5 million rows | Unlimited |
| **Local storage** | Can save locally (recent versions) but must ultimately publish to the public Tableau community for sharing | Can save locally and share within an organization's private workspace |
| **Data security** | None — anything saved to Tableau Public Community is visible to everyone | Secure — stays within the organization |
| **Cost** | Free | Paid (with a 14-day trial); notably more expensive than several competing tools |

### Decision Guide

| Who You Are | Recommended Tool |
|---|---|
| Student / learning practitioner | Tableau Public |
| Working professional needing to keep company data private | Tableau Desktop |
| A company that needs to analyze its own data securely | Tableau Desktop |

### Why It Matters
Nearly every feature available in Desktop is also available in Public — the differences are about **connectivity, scale, and where your finished work lives**, not about a stripped-down feature set. This means practicing on Tableau Public genuinely prepares you for professional Desktop use.

### Interview Tip
*"If Tableau Public and Desktop share the same features, why would a company still pay for Desktop?"* — Answer: data security (Public has none — everything published is visible to anyone) and unlimited data volume/connector access, both of which matter enormously at enterprise scale.

---

## 4. Installing Tableau Public

### Method
1. Go to the Tableau website and **Sign In** (create an account if needed)
2. Navigate to **Products → Tableau Public**
3. Click **Download Tableau Desktop Public Edition**
4. Fill in basic details requested → click **Download**
5. If the download doesn't start automatically, choose your OS (Windows/macOS) manually and click Save
6. Accept the license agreement → **Install** → grant any permission prompts

### Best Practice
Recent versions of Tableau Public allow saving workbook files **locally** on your machine — a feature that wasn't previously available, giving more flexibility before you're ready to publish.

---

# PART 3: Connecting to Data & Interface

## 5. Connecting Tableau to Data Sources

### Definition
Tableau Public's initial "Connect" pane shows a small number of file-based connectors — sufficient for most student/learning use cases: **Text file** (covers CSV as well), **Excel**, **Access**, **PDF**, **Spatial**, and **Statistical** files.

### Method — Text/CSV Files
1. From the Connect pane, choose **Text File**
2. Browse to and select the CSV file
3. Once opened, Tableau will also list **every other file in that same folder**, letting you quickly add related files without re-browsing

### Method — Excel Files
1. Choose **Microsoft Excel**
2. Select the workbook — Tableau shows all sheets contained within it (e.g., Orders, People, Returns)
3. Drag the sheet you want into the canvas area to load it — you can rename it there (e.g., "Order Table")

### Editing an Existing Connection
Click the dropdown next to a connected file for options: rename, view the data preview, or remove the connection entirely. To change the source file location or swap files, use **Edit Connection**.

### Real-World Example
A "Superstore Sales" dataset containing Orders, Managers, and Returns tables (each in separate Excel/CSV files) was connected by first loading the Orders sheet, then dragging in People and Returns sheets as needed — building up the full data source incrementally.

### Interview Tip
*"What data source types does Tableau Public support, and how does that compare to Desktop?"* — Answer: Public supports around 7 basic file-type connectors (Excel, text/CSV, spatial, statistical, etc.), sufficient for most learning/student use. Desktop supports many more — cloud platforms, data warehouses, data lakes, and numerous database types — making it the right choice whenever data needs to be pulled from varied enterprise sources.

---

## 6. Tableau Interface Walkthrough

### Definition
Tableau's interface is organized around **three core areas**, navigated via tabs at the bottom of the screen.

### The Three Core Areas

| Area | Purpose |
|---|---|
| **Data Source** | Where you connect to, preview, and model your data |
| **Worksheet(s)** | Where individual charts are built (one chart per sheet, typically) |
| **Dashboard / Story** | Where multiple sheets are combined into an interactive board or a narrative sequence |

### 6.1 The Workbook (Start Screen)
The very first page opened in Tableau — shows recent activity, file access options on the left (**Connect**), and recently used files on the right (**Open a Workbook**, **Discover**, and more).

### 6.2 The Data Source Page
Once a file is connected, you land here. Key regions:
- **Left side**: shows other tables/sheets available in the same connected source (all sheets are visible at once for CSV; for Excel, sheets must be dragged in individually)
- **Bottom name field**: rename your loaded table
- **Right side (preview)**: shows up to 100 rows of the actual data, along with a summary — total field (column) count and total row count

### Real-World Example
A CSV-based Orders table showed **21 fields** (columns) and **1,993 rows** clearly summarized on the Data Source page — this summary view is a quick sanity check before moving into visualization work.

> 🗒️ Note: Data modeling (relationships between tables) also happens directly on this Data Source page — covered in depth in Day 22.

### 6.3 The Worksheet (Visualization Area)

| Region | Purpose |
|---|---|
| **Left panel** | Lists all available columns (fields) from the connected data |
| **Center canvas** | Where charts are actually built — drag fields here, or into the "Drop field here" zones |
| **Top-right (Show Me)** | Suggests/lists chart types compatible with the fields currently in use |
| **Top ribbon** | Menu tabs for further formatting and functionality (explored in later sessions) |
| **Left sidebar (below field list)** | Pages, Filters, and Marks — used to enhance and filter chart detail |

### The Horizontal Line — Dimensions vs Measures Preview
In the field list, a **horizontal divider line** separates two categories of fields — everything above it is a **Dimension**, everything below it is a **Measure**. (Covered in full depth in Day 22.)

### 6.4 Dashboards
Combining multiple worksheets into a single interactive board — created via **New Dashboard** at the bottom tab bar. Sheets can be dragged directly onto the dashboard canvas, and the overall dashboard layout size can be configured.

### 6.5 Stories
A **Story** presents a sequence of dashboards/sheets as a narrative, page by page — created via **New Story**. Clicking **Present** displays it as a full-screen guided presentation, useful for walking an audience through insights step by step.

### Real-World Example
A simple story sequence: Page 1 — a bar chart showing sales by category (Technology highest, Furniture second, Office Supplies third); Page 2 — the same data broken down differently; each page annotated with a short explanation, together forming a guided narrative rather than a static dashboard.

### Interview Tip
*"Walk me through Tableau's interface at a high level."* — Answer: Three main areas — **Data Source** (connect and model data), **Worksheets** (build individual charts via drag-and-drop), and **Dashboards/Stories** (combine sheets into an interactive board or a guided narrative). Charts must exist before they can be added to a dashboard or story.

---

# 🎯 Key Takeaways

| Topic | Key Point |
|---|---|
| Tableau | Drag-and-drop visual analytics platform, owned by Salesforce since 2019, no coding required |
| Advantages | Wide connectivity, quick calculations, interactive dashboards, handles large datasets |
| Disadvantages | Higher cost than competitors; limited native data pre-processing |
| Public vs Desktop | Public = free, ~7 connectors, 1.5M row limit, no data security; Desktop = paid, extensive connectors, unlimited data, secure org-level sharing |
| Data connection | Text/CSV and Excel are the primary connectors in Public; connecting to a folder surfaces all related files |
| Interface | Three core areas — Data Source, Worksheet, Dashboard/Story |
| Field list divider | Horizontal line splits Dimensions (above) from Measures (below) |
| Story | A sequence of dashboard/chart pages presented as a guided narrative |

---

# ⚡ Pro Tips

- Practicing on Tableau Public genuinely builds Desktop-transferable skills — the feature set is nearly identical; only connectivity, scale, and security differ.
- When connecting to a folder-based CSV source, Tableau automatically surfaces every other file in that folder — useful for quickly building a multi-table data source without repeated browsing.
- Always check the Data Source page's row/field count summary before starting visualization — it's a fast sanity check that the right data loaded correctly.
- Use Stories (not just Dashboards) when the goal is to walk someone through a sequence of insights — Dashboards are best for an at-a-glance interactive single view.

---

# Practice Exercises

- [ ] Install Tableau Public and connect to a sample CSV file
- [ ] Connect to a multi-sheet Excel workbook and load at least two different sheets as separate tables
- [ ] Explore the Data Source page — note the row count and field count for your dataset
- [ ] Identify, without clicking, which fields in your dataset would fall above vs below the horizontal Dimension/Measure divider
- [ ] Build a one-page Story that walks through two different charts with a short caption on each page
- [ ] (Interview-level) Explain out loud the difference between Tableau Public and Tableau Desktop, covering all four comparison points (data sources, volume, sharing/security, cost)

---

# Interview Preparation

## Frequently Asked Questions

**Q: What is Tableau?**
A: A visual analytics platform for building data visualizations primarily through drag-and-drop, owned by Salesforce since 2019, requiring no prior coding knowledge.

**Q: What's the difference between Tableau Public and Tableau Desktop?**
A: Public is free with ~7 basic connectors, a 1.5 million row limit, and no data security (everything is publicly visible once published). Desktop is paid, supports extensive connectors including databases and cloud sources, allows unlimited data, and keeps data secure within an organization.

**Q: What are Tableau's main disadvantages?**
A: Higher cost compared to competing tools, and limited native data pre-processing/cleaning capability compared to tools like Power BI.

## Scenario-Based Questions

**Scenario:** A student wants to practice building dashboards for a portfolio project with public sample data. Which Tableau product should they use, and why?
**Answer:** Tableau Public — it's free, supports the data volume typical of practice datasets, and is specifically designed for sharing portfolio-style work publicly (though the lack of data security means it shouldn't be used for anything sensitive).

## Practical Questions

- You've connected to an Excel file with three sheets (Orders, Managers, Returns) but only one sheet appears on the canvas. How do you bring in the other two?
- Describe the difference between what you'd see on the Data Source page versus a Worksheet page for the same connected file.

---

# Revision Cheat Sheet

- **Tableau** = drag-and-drop visual analytics tool, owned by Salesforce (since 2019)
- **Advantages**: wide connectivity, quick calculations, interactive dashboards, large-data handling
- **Disadvantages**: high cost, limited data pre-processing
- **Public**: free, ~7 connectors, 1.5M row limit, no security | **Desktop**: paid, extensive connectors, unlimited data, secure
- **Three interface areas**: Data Source (connect/model) → Worksheet (build charts) → Dashboard/Story (combine/present)
- **Field list**: horizontal line splits Dimensions (above) from Measures (below)
- **Story** = guided narrative across multiple pages | **Dashboard** = single interactive combined view
