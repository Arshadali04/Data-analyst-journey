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

