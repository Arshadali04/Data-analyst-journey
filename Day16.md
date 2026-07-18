# Day 16: Introduction to Power BI, BI Overview & Data Connections

## 📋 Topics Covered

1. What is Business Intelligence (BI) and why it matters
2. Self-service BI tools vs traditional reporting
3. What is Power BI and why Microsoft's tool leads the market
4. Comparison: Excel vs SQL vs Power BI
5. Power BI ecosystem — Desktop, Service, Mobile, Gateway
6. Power BI licensing types (Free, Pro, Premium Per User, Premium Per Capacity)
7. Power BI Desktop UI — Ribbon, Panes, Views, Canvas
8. Standard Power BI project development stages
9. Connecting to data sources (Get Data, connectors, Navigator)
10. Data warehouse & ETL — where Power BI fits in an org's data flow

---

# PART 1: Understanding Business Intelligence

## Introduction

Before touching Power BI, it's worth understanding *why* BI tools exist in the first place — this is a common interview opener ("What is BI? What is Power BI?") so it's worth having a crisp answer.

**Business Intelligence (BI)** = setting up a system or process that continuously converts raw data into meaningful insights.

It's not a one-time report. It's an always-on pipeline: raw data → cleaning → analysis → decision-ready output.

### Real-world example
Banks don't call every customer offering a loan. A BI system continuously scores customer transaction history and surfaces a prioritized call list to salespeople. Government tax departments don't manually scan every citizen's filings — a BI system flags defaulters automatically. This is BI in action: turning operational data into an actionable, prioritized output without a human digging through raw tables every time.

### Why BI evolved: Traditional vs Self-Service

| Aspect | Traditional BI (15–20 yrs ago) | Self-Service BI (Today) |
|---|---|---|
| Who gets the answer | Business user waits on a technical specialist to write queries | Business user connects directly to data |
| Speed | Slow — multiple systems, multiple people involved | Fast — one person, one tool |
| Dependency | High (on DB/query experts) | Low — user owns the process end-to-end |
| Example tools | Manual SQL reporting | Power BI, Tableau, Qlik, Domo, Looker, Spotfire |

Power BI belongs to this second category — a **self-service BI tool**.

---

# PART 2: Core Concepts

## 1. What is Power BI

### Definition
Power BI is a **self-service business intelligence tool** by Microsoft that connects to data sources, cleans/transforms data, models relationships between tables, and builds interactive visual reports/dashboards.

### Why It Matters
Organizations need to turn scattered raw data (files, databases, cloud apps) into fast, visual, shareable insight — without depending on a data engineering team for every question.

### Industry Relevance
Power BI is one of the two market-leading self-service BI tools globally (alongside Tableau), and it's the most commonly requested BI skill in Data Analyst job postings because of its low cost and tight Microsoft ecosystem integration.

### Why Power BI specifically leads the market
- **Microsoft product** → integrates naturally with Excel, Azure, and other Microsoft tools already used in most orgs
- **User-friendly** → UI feels familiar if you know Excel
- **Not costly** → cheaper licensing than Tableau, Qlik, Domo
- **Fully featured & free desktop tool** → no cost barrier to start learning/building

> 🗒️ Fun fact: Power BI's first release (2015) was actually an **add-in inside Excel**, not a standalone product. Microsoft later split it into its own dedicated tool.

---

## 2. Excel vs SQL vs Power BI — Where Each Fits

### Definition
These three tools represent three different **scales** of working with data — not competitors, but a progression.

| Feature | Excel | SQL (Databases) | Power BI |
|---|---|---|---|
| Data volume | Small | Large (millions of rows) | Large, connects to DB/warehouse |
| Storage | Local, single file | Centralized database | Connects to files/DB/cloud |
| Collaboration | Poor (file-based) | Depends on DB access | Strong (via Power BI Service) |
| Purpose | Store, calculate, basic visuals | Store + query structured data | Model + visualize + share interactively |
| Multi-user analysis | No | Query-based, not visual | Yes, fully interactive |

### When To Use
- **Excel** → quick, small-scale personal analysis
- **SQL** → when data volume grows beyond what a spreadsheet can hold, and you need a query language to retrieve it
- **Power BI** → when you need to connect, clean, model, and visualize that data interactively for others to consume

### Interview Tip
A common trick question: *"If Power BI can connect to Excel, why do we need SQL/databases at all?"* — Answer: Excel doesn't scale (row limits, no concurrent multi-user writes, no schema enforcement); a database is built for volume, integrity, and concurrent access. Power BI is the visualization/modeling layer sitting *on top of* whichever storage layer (Excel, SQL, warehouse) an org uses.

---

## 3. The Power BI Ecosystem (Tools)

### Definition
Power BI isn't one single application — it's a suite of tools that work together across the development-to-consumption lifecycle.

| Tool | Type | Purpose | Cost |
|---|---|---|---|
| **Power BI Desktop** | Local Windows app | Full development — connect, clean, model, visualize | Free |
| **Power BI Service** | Cloud/online app | Publish, collaborate, share reports with others | Paid (per license) |
| **Power BI Mobile** | Mobile app (iOS/Android) | View published reports on the go | Free (needs valid license to view) |
| **Power BI Gateway** | Connector tool | Keeps data refreshed between source and Power BI Service | Included with paid tiers |

### Why It Matters
Interviewers often ask *"What is the difference between Power BI Desktop and Power BI Service?"* — the clean answer:
- **Desktop** = where you *build* (local, free, full-featured, Windows-only)
- **Service** = where you *share/collaborate* (cloud, paid, browser-based)
- **Mobile** = where others *consume* (view-only)
- **Gateway** = what *keeps it all up to date* automatically

### Common Mistake
Assuming Power BI Desktop works on Mac. It doesn't — it's Windows-only. Mac users need a virtual Windows machine (e.g., Parallels) to run it.

### Real-World Example
A BI developer builds a sales dashboard in Desktop → publishes it to Service → the regional sales manager views it on their phone via Mobile → Gateway ensures the numbers refresh nightly from the company's SQL server without anyone touching the report again.

---

## 4. Power BI Licensing

### Definition
Licensing determines what you're allowed to do with Power BI Service — Desktop itself is always free.

| License | Shared Workspaces? | Data Model Limit | Storage | Refreshes/Day | Cost |
|---|---|---|---|---|---|
| **Free** | ❌ No | — | 1 GB total | — | Free |
| **Pro** | ✅ Yes | 1 GB per report | 10 GB per user | 8 | $14/user/month |
| **Premium Per User** | ✅ Yes | 10 GB per report | 100 TB (org-wide) | 48 | $24/user/month |
| **Premium Per Capacity** | ✅ Yes | 10 GB per report | 100 TB (org-wide) | 48 | Flat $5,000/month (unlimited users) |

### Interview Tip
Don't memorize exact dollar figures (they change) — know the **relative hierarchy** and *why* premium exists: capacity-based pricing makes sense once an org has enough users that a flat fee is cheaper than per-seat licensing.

---

## 5. Power BI Desktop — UI Walkthrough

### Key UI Components

| Component | Description |
|---|---|
| **Ribbon/Menus** | Top tabs (Home, Insert, Modeling, View) — similar to Excel's ribbon |
| **Panes** | Right-side collapsible panels: Visualizations pane, Data pane, Filters pane |
| **Views** | Left-side icons for switching between Report / Table / Model views |
| **Canvas** | The blank central workspace where visuals are built (only in Report view) |
| **Report Pages** | Bottom `+` button to add multiple pages to one report |

### The Three Core Views

```
Report View  → build and see visuals on the canvas (end-user's final view)
     ↓
Table View   → see raw data in rows/columns format
     ↓
Model View   → see all tables + relationships between them
```

### Best Practice
> ⚠️ Don't sign in to Power BI Desktop unless you already have a Power BI Service account you want to link. Signing in isn't required to use Desktop — it's a common beginner confusion point.

---

## 6. Standard Power BI Development Stages

### Why It Matters
Every Power BI project — regardless of domain — follows the same six stages. Knowing this sequence is a strong interview signal that you understand the *process*, not just the button clicks.

```
1. Data Source Connection
        ↓
2. Data Cleaning & Transformation   (Power Query)
        ↓
3. Data Modeling                    (Relationships + Calculations)
        ↓
4. Data Visualization
        ↓
5. Report Development               (storytelling across visuals)
        ↓
6. Sharing Reports                  (Power BI Service)
```

---

## 7. Connecting to Data Sources

### Definition
Power BI connects to a data source via a **connector** — a pre-built adapter specific to that data type (Excel, SQL Server, Web/API, etc.). There are 70+ connectors available.

### Categories of Connectors
- **File-based**: Excel, CSV, text files
- **Database**: SQL Server, MySQL, Oracle, Access
- **Online/Web**: Web (HTML tables), OData feeds, REST APIs
- **Platform-specific**: Azure, Power Platform, Microsoft Fabric

### Step-by-step: Connecting to Data

1. Go to **Home → Get Data**
2. Choose the relevant connector (e.g., *Excel Workbook*, *Web*, *SQL Server database*)
3. Provide connection details (file path / server name / URL) and authenticate if required
4. Power BI opens the **Navigator** window — lists all tables/sheets found in that source
5. Preview each table, select the ones needed
6. Choose **Load** (bring data in as-is) or **Transform Data** (clean first, in Power Query Editor)

### Real-World Example
Connecting to a public Wikipedia page via the **Web connector**: Power BI scans the HTML page, detects every `<table>` element, and lists them in the Navigator so you can pick which table(s) to import — useful for quick public-data pulls without manual copy-pasting.

### Common Mistakes
- Clicking **Load** directly without previewing the data — you might load messy headers or blank rows unintentionally
- Not checking whether a web page is secured/authenticated before assuming the Web connector will work

### Best Practices
- Always preview data in the Navigator before loading
- Prefer **Transform Data** over **Load** when you're unsure the data is clean — cheaper to fix now than after loading into the report

---

## 8. Data Warehouses & ETL — Where Power BI Sits

### Definition
In real organizations, data typically doesn't live in one place — sales data, marketing data, and accounting data usually sit in *separate systems*. A **data warehouse** is a centralized database where all of this gets consolidated for analysis.

### The Process: ETL / ELT
**ETL = Extract, Transform, Load** (or **ELT** = Extract, Load, Transform)

```
Source Systems (CRM, Marketing tool, Accounting tool)
        ↓  (continuous extraction)
   ETL Process (cleaning, transformation)
        ↓
   Data Warehouse (centralized, historical data)
        ↓  (Power BI connects here, not to individual source systems)
   Power BI (pulls data, builds reports)
```

### Why It Matters
This is a **data engineering team's responsibility**, not the BI developer's. As a Data/BI Analyst, you typically connect Power BI to the *warehouse*, not to each individual source system directly. You are still responsible for **scheduling data refreshes** in Power BI so your reports reflect the latest warehouse data — the warehouse being updated doesn't automatically update your report.

### Interview Tip
If asked *"Does Power BI store or manage the ETL process?"* — the answer is generally **no**. Power BI Desktop *can* do light cleaning via Power Query, but large-scale ETL into a warehouse is typically a separate data engineering pipeline (e.g., using tools like SSIS, Azure Data Factory, dbt).

---

# 🎯 Key Takeaways

| Topic | Key Point |
|---|---|
| Business Intelligence | A continuous system converting raw data into decision-ready insights |
| Self-service BI | Lets business users connect, model, and analyze data without depending on IT/query specialists |
| Power BI | Microsoft's self-service BI tool — free desktop, paid cloud sharing |
| Power BI Desktop | Free, Windows-only, full-featured — where all development happens |
| Power BI Service | Paid cloud app — for sharing/collaboration |
| Power BI Mobile | View-only app for consuming published reports |
| Power BI Gateway | Keeps data refreshed between the source and Power BI Service |
| Licensing | Free → Pro ($14/user/mo) → Premium Per User ($24/user/mo) → Premium Per Capacity ($5,000/mo flat) |
| Development stages | Connect → Clean/Transform → Model → Visualize → Report → Share |
| Data warehouse | Central store where multiple source systems' data is consolidated via ETL; Power BI connects here in enterprise setups |

---

# ⚡ Pro Tips

- Power BI Desktop doesn't require sign-in unless linking to Power BI Service — skip that prompt while learning.
- Know your system type (32-bit vs 64-bit) before downloading Desktop — most modern systems are 64-bit.
- In interviews, be ready to explain the **six development stages** in order — it signals process maturity, not just tool familiarity.
- Don't over-invest time memorizing exact license prices; know the *relative differences* (storage limits, refresh frequency, shared workspace access) instead — pricing changes over time.
- When asked "What is Power BI?" in an interview, always mention it's **self-service** — that single word differentiates it from legacy BI reporting.

---

# Practice Exercises

- [ ] Download and install Power BI Desktop (confirm Windows OS or set up a virtual machine if on Mac)
- [ ] Connect to any public Wikipedia page using the Web connector and explore the Navigator window
- [ ] Open Get Data → More, and browse the full list of 70+ connectors by category (File / Database / Online Services)
- [ ] Explain in your own words the difference between Power BI Desktop, Service, and Mobile (interview-level)
- [ ] Draw the 6-stage Power BI development pipeline from memory
- [ ] Research: What's the current Power BI Pro license price today? (Prices change — practice looking it up rather than memorizing)

---

# Interview Preparation

## Frequently Asked Questions

**Q: What is Power BI?**
A: A self-service business intelligence tool by Microsoft used to connect to data sources, clean and model data, and build interactive visual reports/dashboards.

**Q: What's the difference between Power BI Desktop and Power BI Service?**
A: Desktop is a free, Windows-only local application used for development (connecting, cleaning, modeling, visualizing). Service is a paid cloud application used to publish and share reports for collaboration.

**Q: Is Power BI Desktop free?**
A: Yes, fully free and fully featured — no license required for individual use. Licensing (Pro/Premium) is only needed to collaborate/share via Power BI Service.

**Q: What connectors does Power BI support?**
A: 70+ connectors across categories — file-based (Excel, CSV), database (SQL Server, MySQL, Oracle), online/web (Web, OData, APIs), and platform-specific (Azure, Power Platform).

## Scenario-Based Questions

**Scenario:** Your manager wants a report shared with 15 people across departments, updated automatically every morning. What Power BI components are involved?
**Answer:** Build in Power BI Desktop → publish to Power BI Service (Pro or higher license for sharing) → configure a Gateway connection to the data source → schedule a data refresh (e.g., daily) → team views via browser or Power BI Mobile.

## Practical Questions

- Given a CSV file and a SQL Server database as two possible data sources for the same analysis, which connector would you choose for each, and why?
- If your Power BI report's Navigator shows 10 tables and you only need 3, what steps do you take to avoid loading unnecessary data?

---

# Revision Cheat Sheet

- **BI** = system that continuously turns raw data into insight
- **Power BI** = Microsoft's self-service BI tool
- **Desktop** (free, build) → **Service** (paid, share) → **Mobile** (view) → **Gateway** (auto-refresh)
- **License tiers**: Free < Pro < Premium Per User < Premium Per Capacity
- **6 dev stages**: Connect → Clean → Model → Visualize → Report → Share
- **Data warehouse**: centralized store; Power BI connects here in enterprise setups, not to individual source systems
- **Get Data → Navigator → Load/Transform** = standard connection flow
