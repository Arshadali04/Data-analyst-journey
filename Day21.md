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

