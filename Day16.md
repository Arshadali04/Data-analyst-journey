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

