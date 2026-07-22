# Day 20: Interactive Dashboarding in Power BI

## 📋 Topics Covered

1. Report design planning — audience, KPIs, and story structure
2. Bookmarks — capturing report states for dynamic switching
3. Page navigation via shapes/buttons and hidden pages
4. Selection pane — controlling visual visibility
5. Sync Slicers — synchronizing filters across report pages
6. Edit Interactions — controlling which visuals filter which
7. Power BI Service — workspaces, publishing, and semantic models
8. Editing and versioning reports directly in Power BI Service
9. Sharing, exporting, and downloading reports
10. Power BI Gateway — on-premises vs cloud data sources
11. Scheduled refresh and refresh frequency by license
12. Row-Level Security (RLS) — roles and user assignment
13. Auto-create reports in Power BI Service
14. Dashboards vs Reports — key differences
15. Connectivity modes — Import vs DirectQuery

---

# PART 1: Planning & Building an Interactive Report

## 1. Report Design Planning

### Definition
Before opening Power BI, effective report development starts with three planning questions.

### The Planning Framework

| Question | Why It Matters |
|---|---|
| **Who is the end user?** | Determines the level of detail (executive vs analyst vs regional manager) |
| **What metrics should be shown?** | KPIs relevant to that audience's decisions |
| **How will you tell the story?** | The flow/structure across pages — not a random collection of visuals |

### Real-World Example
For a **global sales leadership team** audience, the relevant KPIs are high-level: total sales, total orders, total customers, sales per order, sales per customer. The story structure might be: an overview page (high-level KPIs and trend), a product/category-specific page, and a geography-level analysis page.

### Interview Tip
When asked to describe your report-building process, always lead with **audience and objective**, not tool mechanics — this signals you think about reports as communication tools, not just technical exercises.

---

## 2. Bookmarks

### Definition
A **bookmark** captures a specific *state* of a report page — which visuals are visible/hidden, which slicer selections are active, which visual is in focus — so you can return to that exact state later, or let a viewer toggle between states.

### Why It Matters
Bookmarks let you pack multiple "views" into a single page without creating separate report pages for each — useful when different views share the same page layout but show different subsets of visuals.

### How to Create a Bookmark
1. Open the **Selection pane** (View tab) to control which visuals are shown/hidden — click the eye icon next to any visual to toggle visibility
2. Arrange the page into the desired state (e.g., hide all customer-related visuals, show only product-related visuals)
3. Go to the **Bookmarks pane** (View tab) → **Add** → rename the bookmark descriptively (e.g., "Product View")
4. Repeat for each alternate state (e.g., hide product visuals, show customer visuals → "Customer View")

### Real-World Example
Two sets of visuals — "Top/Bottom 10 Products" and "Top/Bottom 10 Customers" — were built directly on top of each other (same screen position), each assigned to its own bookmark. Clicking a "Product View" button shows only the product bookmark's state; clicking "Customer View" swaps to the customer bookmark's state — creating the effect of two different pages within a single page, without navigating away.

### Linking Bookmarks to Clickable Elements
Bookmarks themselves don't do anything until they're wired to something clickable:
1. Insert a **Shape** with descriptive text (e.g., "Product View")
2. Select the shape → Format → **Action** → turn on → Action type: **Bookmark** → select the target bookmark name

### Common Mistake
Forgetting to also hide/show any accompanying title text boxes as part of the bookmarked state — if a title box isn't included in the visibility selection captured by the bookmark, it won't toggle along with the rest of the visuals.

### Interview Tip
*"How would you show two completely different views of data on the same report page without creating a new page?"* — Answer: Bookmarks, combined with the Selection pane to control visibility, linked to clickable shapes via the Bookmark action type.

---

## 3. Page Navigation

### Definition
Beyond bookmarks (which work *within* a page), Power BI supports navigating *between* pages using the same shape-and-action mechanism, with **Page Navigation** as the action type instead of Bookmark.

### Method
1. Insert a shape with descriptive text (e.g., "Show in Orders")
2. Format → Action → turn on → Action type: **Page navigation** → Destination: select the target page

### Hiding Pages from the Page Tab List
A page can be **hidden** (right-click the page tab → Hide) so it doesn't appear in the visible page-tab list — but it's still reachable via a page-navigation action from a shape/button. This is useful for creating an alternate view of a report (e.g., "same report, but by Orders instead of Sales") that's accessed only through a toggle button, keeping the visible page list clean.

### Real-World Example
A "Sales Overview" page (visible) and a duplicate "Sales Overview - Orders" page (hidden) each had a "Show in Orders" / "Show in Net Sales" shape linking to the other page — creating a seamless toggle experience while keeping only one page tab visible to viewers.

### Best Practice
In Power BI Desktop, page-navigation and bookmark actions on shapes require **Ctrl+Click** to trigger (to distinguish from simply selecting the shape while editing). Once published to Power BI Service, viewers can trigger the action with a normal single click.

---

## 4. Sync Slicers

### Definition
By default, a slicer's selection only affects the page it's placed on. **Sync Slicers** lets a selection made on one page automatically apply to the same slicer field wherever it appears across other pages.

### Why It Matters
Without syncing, a viewer might select "Brazil" on one page, navigate to another page, and see unfiltered (or differently filtered) data — creating confusion about whether a filter is active. Syncing keeps filter state consistent and predictable across the entire report.

### Method
`View tab → Sync Slicers` → for each slicer, check which pages it should apply to (or select all) → repeat for every slicer that should stay synchronized report-wide.

### Best Practice
Sync any slicer that represents a "global" filter concept for the report (e.g., Country, Date Range, Category) — especially in multi-page reports where the same dimension is used for filtering on more than one page.

---

## 5. Edit Interactions

### Definition
By default, every visual on a page can filter every other visual when clicked or selected. **Edit Interactions** lets you selectively disable this cross-filtering behavior for specific visual pairs.

### Why It Matters
Some visuals are meant to always show a fixed/global view regardless of other selections — e.g., an overall multi-year trend line that should *not* shrink to a single data point just because a date slicer was narrowed for other visuals on the page.

### Method
1. Select the visual whose filtering behavior you want to control *from* (e.g., a date slicer)
2. A new **Format** tab appears at the top → click **Edit Interactions**
3. Small icons appear above every other visual on the page — choose whether that visual should respond to the selected visual's filter (filter icon) or ignore it (**None**)
4. Click Edit Interactions again to exit and hide the icons

### Real-World Example
A "Sales Trend Over Time" line chart and a "YTD Sales" card were both set to **ignore** the page's date-range slicer, so they always display the complete historical trend and true year-to-date figure — even while a viewer narrows the date slicer to inspect other visuals for a specific sub-period.

### Common Mistake
Accidentally clicking the wrong interaction icon when multiple icons overlap closely above a small visual — always double-check the setting took effect by testing the slicer/filter interaction after configuring it.

### Interview Tip
*"How do you prevent one slicer from filtering a specific visual while still filtering everything else on the page?"* — Answer: Edit Interactions, selecting the slicer, then setting that one visual's interaction to None.

---

# PART 2: Power BI Service

## 6. Workspaces

### Definition
A **workspace** in Power BI Service functions like a shared folder in the cloud — a container for related reports, dashboards, and datasets that can be collaboratively accessed.

### Types of Workspaces

| Type | Sharing | Availability |
|---|---|---|
| **My Workspace** | Personal only — cannot be shared with others | Default, always available, even without a paid license |
| **Custom/Shared Workspace** | Can be shared with team members | Requires a Pro license (or trial) to create |

### Why It Matters
Organizing reports by department or project (e.g., "Sales and Operations") into dedicated workspaces keeps related content grouped and controls who has access to what.

### Best Practice
Name workspaces after the team, department, or subject area they serve, rather than by individual report names — a workspace typically holds multiple related reports over time.

---

## 7. Publishing & the Semantic Model

### Definition
Publishing a report from Power BI Desktop to Power BI Service (`Home → Publish` → select workspace) creates **two separate artifacts**, not one.

### The Two Artifacts

| Artifact | Contains |
|---|---|
| **Semantic Model** (a.k.a. Data Model) | All data transformations, relationships, and DAX calculations packaged together |
| **Report** | The visual layer — pages, visuals, formatting — which runs against the published semantic model |

### Why It Matters
Once published, the report is **no longer directly linked** to the original data source used in Power BI Desktop — it runs entirely against the packaged semantic model on the Service. A report cannot exist on Power BI Service without an associated semantic model.

### Interview Tip
*"When you publish a Power BI report, what actually gets created on the Service?"* — Answer: two separate items — a semantic model (containing all the modeling and DAX) and a report (the visual layer built on top of that model) — and the report depends entirely on the semantic model rather than the original data source.

---

## 8. Editing Reports on Power BI Service

### What You Can Do as an Editor
Anyone with edit permissions can modify a published report directly in the browser:
- Add, delete, or reposition visuals
- Change layout and formatting
- Add new pages

### What You Cannot Do
- Modify the underlying data model (relationships)
- Create or edit DAX calculations (measures, calculated columns)

These modeling-level changes must be made back in Power BI Desktop and re-published.

### Versioning: Save vs Save As
- **Save**: overwrites the existing published report
- **Save a Copy**: creates a new, separate version — preserving the original in case a collaborator's changes need to be reverted or compared

### Real-World Example
A junior team member continuing report development in a senior developer's absence can make visual-layer changes directly on Power BI Service and use "Save a Copy" to preserve a distinct version, protecting the original from being overwritten by potentially unreviewed changes.

### Other Editor Options
- **Reading view**: switch back from edit mode to how end users will see the report
- **Share**: send a direct link to specific users
- **Export**: to PDF, PowerPoint, or Excel (which exports the underlying data/model)
- **Download**: retrieve the full `.pbix` file — useful for making data-model-level changes later even without the original local file

---

## 9. Power BI Gateway — On-Premises vs Cloud Data Sources

### Definition
Every data source falls into one of two broad categories, and the category determines whether a **Gateway** is required for scheduled refresh to work on Power BI Service.

### The Two Data Source Categories

| Category | Definition | Examples |
|---|---|---|
| **On-premises** | Hosted within your own network/infrastructure or on a local machine | Local SQL Server, Oracle DB, a local Excel/CSV file |
| **Cloud** | Accessible via a web browser/internet | Google Drive, OneDrive, SharePoint, cloud-hosted APIs |

### Why It Matters
Power BI Service (in the cloud) **cannot directly connect** to an on-premises data source — there's no direct network path. A **Gateway** acts as an intermediary bridge, installed on a machine that *can* reach both the on-premises source and the internet.

### What the Gateway Does
```
Power BI Service ←→ Gateway (installed locally) ←→ On-premises data source
```
The Gateway is configured with one or more **connections** — each connection maps to a specific data source. A published semantic model's data source settings must then be **mapped** to the matching gateway connection for scheduled refresh to succeed.

### Gateway Modes

| Mode | Use Case | Cost |
|---|---|---|
| **Personal Mode** | Individual, single-user use | Free |
| **Standard Mode** | Multi-user, organizational use | Free |

### Best Practice
Organizations should default to **Standard mode**, even though both are free — it supports multiple users and semantic models sharing the same gateway installation, which Personal mode does not.

### Multiple Data Sources
If a single report combines multiple on-premises sources (e.g., an Excel file and a SQL Server database), a **separate connection must be created inside the gateway for each source** — think of the gateway as one door with multiple correctly-configured locks, one per data source.

### Interview Tip
*"Why would a scheduled refresh fail even though the report works fine when opened?"* — A likely cause: no Gateway connection has been configured and mapped for an on-premises data source, or an existing connection's credentials/path have changed.

---

## 10. Scheduled Refresh

### Definition
Configuring the semantic model to automatically pull updated data from its source at defined intervals, without manual intervention.

### Method
Semantic model settings → **Refresh** → toggle on **Configure a refresh schedule** → choose frequency (Daily/Weekly) → add one or more specific times.

### Refresh Limits by License

| License | Max Scheduled Refreshes/Day |
|---|---|
| Pro | 8 |
| Premium | 48 |

### Why It Matters
Refresh limits apply **per report/semantic model**, not account-wide — a Pro-licensed workspace with 10 reports can have all 10 refreshing at up to 8 different times each, independently.

---

## 11. Row-Level Security (RLS)

### Definition
**Row-Level Security** restricts which rows of data different users can see within the *same* published report, based on assigned roles — rather than maintaining separate reports per user group.

### Why It Matters
Without RLS, giving country-specific sales teams access to only their own country's data would require manually filtering, publishing, and maintaining a **separate report per country** — extremely time-consuming and hard to keep updated. RLS solves this with a single report and a single semantic model.

### Implementation — Two Steps

**Step 1: Create roles in the data model (Power BI Desktop)**
`Modeling tab → Security → Manage Roles → Create` → name the role (e.g., "Argentina Data") → select the relevant table (e.g., Customers) → add a filter (e.g., `Customer Country = "Argentina"`).

```dax
-- Conceptually, the role filter is equivalent to:
Customers[Country] = "Argentina"
```

**Step 2: Assign users to roles (Power BI Service)**
Publish the report → in the workspace, select the semantic model → **Security** → for each role, add the specific users or group emails who should see that role's filtered data → Save.

### Testing Roles Before Publishing
`Modeling tab → View As` → select a role → the report immediately re-renders as if filtered by that role, letting you verify the filter logic works correctly **before** publishing — critical since a typo in the filter value (e.g., misspelling the country name) silently produces a completely blank report.

### Common Mistake
Typing a filter value that doesn't exactly match the actual data value (e.g., filtering for "Australia" when the actual country value in the data is "Argentina") — the role filter finds zero matching rows, and the report appears entirely blank for anyone assigned to that role. **Always test with "View As" before publishing.**

### Interview Tip
*"How would you give 20 different regional sales teams access to only their own region's data, without maintaining 20 separate reports?"* — Answer: Row-Level Security — one role per region defined in the data model, with users assigned to their corresponding role in Power BI Service.

---

## 12. Auto-Create Reports (Power BI Service)

### Definition
When creating a new report directly on Power BI Service from an existing published semantic model, Power BI offers an **Auto-create report** option that generates a starter report automatically based on the model's fields.

### Why It Matters
It's a quick starting point, not a finished product — auto-generated visuals often include redundant charts (e.g., two visuals conveying the same information in different chart types) that need review and editing before the report is genuinely useful.

### Best Practice
Treat auto-created reports as a draft/scaffold to edit, not a final deliverable — always review each generated visual for relevance and remove or replace redundant ones.

---

## 13. Dashboards vs Reports

### Definition
Power BI Service draws a **strict, specific distinction** between these two terms — unlike some other BI tools where "dashboard" is used loosely.

### Comparison Table

| Feature | Report | Dashboard |
|---|---|---|
| Created in | Power BI Desktop (or Service edit mode) | Power BI Service only |
| Pages | Can have multiple pages | Always exactly **one page** |
| Data source | Tied to **one** semantic model | Can pull **pinned visuals from multiple different reports/semantic models** |
| Interactivity | Fully interactive (filters, slicers, cross-filtering, drill-down) | **Non-interactive** — clicking a tile navigates back to its source report, it doesn't filter in place |
| Purpose | Detailed analysis | High-level, at-a-glance summary |
| Typical audience | Analysts, department teams | Executives, leadership |

### Why It Matters
Reports and dashboards serve genuinely different purposes — a dashboard is not just "a smaller report," it's an aggregation layer that pulls the most important visuals from across an organization's various detailed reports into one single-page executive view.

### How to Build a Dashboard
1. Create a new Dashboard in a workspace (`New item → Dashboard`, name it)
2. Open any existing report → hover over the specific visual to pin → click **Pin visual** → choose the destination dashboard
3. Repeat across as many different reports as needed — all pinned visuals ("tiles") land on the same one-page dashboard canvas
4. Rearrange tiles by dragging them into the desired layout

### Real-World Example
An "Executive Summary" dashboard combined a Net Sales card, YTD Sales card, Total Customers card, Sales per Customer card, a Net Sales by Category chart, and a Top 10 Customers chart — all pinned from one underlying report — into a single consolidated view suited for a five-minute leadership review, rather than a deep-dive analysis session.

### Interview Tip
*"What's the difference between a Power BI report and a Power BI dashboard?"* — Answer: A report can have multiple pages, is built from one semantic model, and is fully interactive. A dashboard is always a single page, can combine pinned visuals from multiple different reports/models, but is non-interactive — clicking a dashboard tile navigates you to the source report rather than filtering in place.

---

## 14. Connectivity Modes — Import vs DirectQuery

### Definition
Every data connection uses one of two connectivity modes, determining how (and when) data actually moves from the source into Power BI.

### Comparison Table

| Feature | Import Mode | DirectQuery Mode |
|---|---|---|
| Data storage | Fully loaded/stored inside the Power BI file itself | Not stored — only a live connection + metadata about available fields is kept |
| When data is fetched | At load time (and on each scheduled/manual refresh) | Queried live, at the moment each visual is created or interacted with |
| Table view availability | Available — full data visible | **Not available** — only Report view and Model view |
| Data currency | Only as fresh as the last refresh | Refreshes automatically every ~5–10 minutes, no scheduling needed |
| Refresh scheduling | Required (daily/weekly + specific times) | Not required |
| Supported connections | All connection types | Limited — mainly database-type connections |
| DAX feature support | Full support for all DAX functions | Some DAX functions are **not supported** |
| Source system performance impact | None after initial load | Can **degrade source database performance**, since every interaction triggers a live query |
| Report responsiveness | Fast — works against local/native data | Can be slower, since every visual interaction round-trips to the source |
| Default mode | ✅ Yes | No — must be explicitly selected where available |

### How to Identify the Mode
If a data connector's dialog **does not show** an explicit "Import vs DirectQuery" choice (e.g., connecting to an Excel file), assume it defaults to **Import mode**. Database connectors (like SQL Server) explicitly present this choice at connection time.

### Real-World Demonstration
A SQL Server table connected via DirectQuery reflected a source-data update (a value changed directly in the database) **almost immediately** upon refreshing the browser page. The same table connected via Import mode showed the **old value** until its next scheduled/manual refresh ran — demonstrating the fundamental difference in data currency between the two modes.

### Microsoft's Recommendation
> **Default to Import mode** unless there's a specific, necessary reason to use DirectQuery (e.g., a data volume too large to import, or a genuine business need for near-real-time data). Import mode supports all features, performs faster for report development and viewing, and avoids straining the source system.

### When DirectQuery Might Be Necessary
- Data volume is too large to feasibly import and store
- Business requirement for near-real-time data (refreshing every few minutes rather than hours)
- Source system is a live operational database where importing periodically isn't sufficient

### Common Mistake
Defaulting to DirectQuery because "it refreshes automatically" without considering its downsides — the automatic refresh convenience often isn't worth the performance cost to both the report and the source database, unless real-time data is a genuine requirement.

### Interview Tip
*"When would you choose DirectQuery over Import mode, given Import is generally recommended?"* — Answer: When data volume is too large to import practically, or when the business genuinely requires near-real-time data that can't wait for a scheduled refresh — otherwise, Import is preferred for its performance, full feature support, and lack of impact on the source system.

---

# 🎯 Key Takeaways

| Topic | Key Point |
|---|---|
| Report planning | Always start with audience, KPIs, and story structure before building |
| Bookmarks | Capture a page's visual-visibility/selection state; link to shapes via Bookmark action for in-page toggling |
| Page navigation | Shape/button + Page Navigation action; pages can be hidden from the tab list but still reachable via action |
| Sync Slicers | Keeps the same slicer's selection consistent across multiple report pages |
| Edit Interactions | Selectively disables cross-filtering between specific visual pairs |
| Workspace | Cloud-based shared folder for organizing related reports/dashboards; "My Workspace" is personal-only |
| Publishing | Creates two artifacts — a semantic model (data + DAX) and a report (visual layer), no longer linked to the original source |
| Editing on Service | Visual-layer changes only — no data model or DAX edits; use "Save a Copy" to preserve versions |
| Gateway | Required bridge for Power BI Service to reach on-premises data sources; Standard mode recommended over Personal |
| Scheduled refresh | Pro = 8/day, Premium = 48/day, configured per semantic model |
| Row-Level Security | Role-based row filtering in one shared report/model; always test with "View As" before publishing |
| Auto-create report | Quick draft starting point from a semantic model — needs review/editing, not a final deliverable |
| Dashboard vs Report | Dashboard = one page, pinned from multiple reports, non-interactive; Report = multi-page, one model, fully interactive |
| Import vs DirectQuery | Import = stored data, scheduled refresh, full feature support, faster (recommended default); DirectQuery = live queries, auto-refresh every 5-10 min, limited connections/DAX support, can strain source performance |

---

# ⚡ Pro Tips

- Use the Selection pane together with Bookmarks to build multiple "views" on a single page — it's a more elegant solution than duplicating pages for closely related view toggles.
- Always sync slicers that represent report-wide filter concepts (country, date range, category) across multi-page reports — inconsistent filter state across pages is a common source of viewer confusion.
- Before publishing any Row-Level Security role, use **View As** in Power BI Desktop to confirm the filter actually returns data — a blank report from a typo'd filter value is one of the most common RLS mistakes.
- Default to Import mode unless you have a specific, justified reason for DirectQuery — Microsoft's own guidance, and it avoids the performance and feature-support trade-offs.
- When collaborating on a report in Power BI Service, use "Save a Copy" rather than "Save" if you want to preserve the original version while experimenting with changes.
- Treat a Dashboard as a curation layer, not a design layer — build and refine visuals in their source Reports first, then pin the finished, most important ones to the Dashboard.

---

# Practice Exercises

- [ ] Create two bookmarks on the same page that toggle between two different sets of visuals using the Selection pane, then link each to a clickable shape
- [ ] Build a hidden page reachable only via a shape's Page Navigation action
- [ ] Set up Sync Slicers for at least one slicer across a 2+ page report, and confirm a selection on one page reflects on the other
- [ ] Use Edit Interactions to make one visual on a page ignore a specific slicer while all other visuals still respond to it
- [ ] Publish a report to a workspace and identify the two separate artifacts created (semantic model and report)
- [ ] Set up Row-Level Security with at least two roles, test each with "View As" before publishing, then assign a test user to a role in Power BI Service
- [ ] Pin at least 4 visuals from one or more reports into a new single-page Dashboard
- [ ] Connect to the same data source twice — once in Import mode, once in DirectQuery — change a value at the source, and compare how quickly each reflects the change

---

# Mini Project

**Objective:** Build a complete interactive, multi-page report with navigation, security, and a summary dashboard

**Dataset:** Continue with the Orders/Customers/Products model from previous days

**Tasks:**
1. Plan the report: define the audience (e.g., "regional sales managers"), the KPIs to show, and a 2–3 page story structure
2. Build a "Sales Overview" page with KPI cards, a trend line chart, and category/country breakdowns
3. Duplicate the page for an "Orders" view and use a shape + Page Navigation action to toggle between them (hide one page from the tab list)
4. Build a "Top/Bottom Performers" page using overlapping visuals + Bookmarks to toggle between Product view and Customer view
5. Sync any shared slicers (e.g., Country, Date) across all pages
6. Set up Row-Level Security with at least two country-based roles; test with View As
7. Publish to a new workspace, assign RLS roles to test users, and configure a scheduled refresh
8. Pin the most important 4–6 visuals to a new single-page Dashboard

**Skills Practiced:** Bookmarks, page navigation, sync slicers, edit interactions, publishing, RLS, gateway/refresh concepts, dashboard curation

---

# Interview Preparation

## Frequently Asked Questions

**Q: What is a bookmark in Power BI, and how is it different from a page?**
A: A bookmark captures a specific state of visibility/selection on the *same* page — it doesn't create a new page. It's typically linked to a shape or button so viewers can toggle between different curated views without navigating away.

**Q: What's the difference between Sync Slicers and Edit Interactions?**
A: Sync Slicers keeps the *same* slicer's selection consistent across multiple pages. Edit Interactions controls whether *one visual's* selection filters *another specific visual* on the same page — they solve different problems (cross-page consistency vs. within-page filtering control).

**Q: What gets created when you publish a report to Power BI Service?**
A: Two separate items: a semantic model (containing the data, relationships, and DAX) and a report (the visual layer), with the report depending entirely on the semantic model rather than the original data source.

**Q: When is a Power BI Gateway required?**
A: When the data source is on-premises (hosted on local infrastructure) — Power BI Service cannot connect to it directly, so the Gateway acts as a bridge. It's optional for cloud data sources.

**Q: How does Row-Level Security work?**
A: It's implemented in two steps — first, roles with row-filtering DAX conditions are created in the data model (Power BI Desktop); second, specific users or groups are assigned to those roles in Power BI Service, so each user only sees the rows their assigned role permits.

**Q: What's the difference between a Dashboard and a Report in Power BI Service?**
A: A Report can have multiple pages, is built on one semantic model, and is fully interactive. A Dashboard is always exactly one page, can pin visuals from multiple different reports/models, but is non-interactive — clicking a tile navigates to the source report instead of filtering.

**Q: What's the difference between Import and DirectQuery mode?**
A: Import mode loads and stores data physically inside the Power BI file, refreshing on a schedule; it supports all features and performs faster, and is Microsoft's recommended default. DirectQuery keeps a live connection instead of storing data, auto-refreshing every 5–10 minutes, but supports fewer connection types and DAX functions, and can degrade source database performance under heavy interaction.

## Scenario-Based Questions

**Scenario:** You publish a Row-Level Security role for "Argentina Data," but a user assigned to that role sees a completely blank report. What's the likely cause?
**Answer:** The filter value in the role definition likely doesn't exactly match the actual data value (e.g., a typo or wrong country name) — the filter finds zero matching rows. Always verify with "View As" in Power BI Desktop before publishing.

**Scenario:** A stakeholder wants a single-page, high-level view combining the most important visuals from five different detailed reports, for a quick five-minute leadership review. What Power BI Service feature fits this need?
**Answer:** A Dashboard — pin the key visuals from each of the five reports onto one Dashboard canvas. This is exactly the use case dashboards are designed for, as opposed to building a sixth, separate multi-page report.

## Practical Questions

- Describe the exact steps to let a user toggle a report page between "filtered by Category" and "filtered by Region" views using a single button, without duplicating the page.
- A colleague says DirectQuery is strictly better than Import mode because "it's always up to date." Give a counterargument covering at least two trade-offs.

---

# Revision Cheat Sheet

- **Bookmark** = saved visibility/selection state on one page; link via shape's Bookmark action
- **Page Navigation action** = shape/button linked to jump to another page; pages can be hidden but still reachable
- **Sync Slicers** = same slicer, consistent selection across pages
- **Edit Interactions** = control which visuals respond to which visual's filter, on the same page
- **Workspace** = cloud shared folder; My Workspace = personal only, Custom = shareable (needs Pro+)
- **Publishing** creates: **Semantic Model** (data + DAX) + **Report** (visual layer) — two separate artifacts
- **Editing on Service**: visual layer only, no DAX/relationship changes — use Save a Copy to version
- **Gateway** = required bridge for on-premises sources; Standard mode preferred over Personal
- **Refresh limits**: Pro = 8/day | Premium = 48/day
- **RLS** = roles (Desktop) + user assignment (Service); always test with View As first
- **Dashboard**: one page, multi-report pinned tiles, non-interactive | **Report**: multi-page, one model, fully interactive
- **Import** (default, recommended): stored data, scheduled refresh, full features, faster
- **DirectQuery**: live queries, auto-refresh ~5-10 min, limited connections/DAX, can strain source performance
