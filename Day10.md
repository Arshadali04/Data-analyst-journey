# Day 10: SQL — Views, Indexes & Subqueries

## 📋 Topics Covered

**SQL (MySQL):**
1. Views
2. Indexes
3. Subqueries

---

# PART 1: VIEWS 👁️

## What is a View?

> *"A view in MySQL is a virtual table based on the result of a SELECT query. It does not store data itself — it always reflects the current data in the base tables."*

Think of a View as a **saved SELECT query with a name**. Every time you query the view, MySQL runs that underlying SELECT behind the scenes and returns fresh, live results.

### Analogy
A View is like a **window into your data** — the window itself doesn't hold anything, but what you see through it changes as the room behind it changes.

---

## When to Use Views

| Use Case | Why Views Help |
|----------|---------------|
| **Simplify complex queries** | Save a long JOIN + WHERE + GROUP BY as a named view — query it like a simple table |
| **Reuse logic** | Write the filtering/joining logic once, reuse it everywhere |
| **Hide sensitive columns** | Expose only `name` and `salary`, hide `password` or `email` from certain users |
| **Live filtered snapshot** | Always shows the latest data matching the filter — no manual refresh needed |

---

## Creating a View

Syntax:
```sql
CREATE VIEW view_name AS
SELECT ...
FROM ...
WHERE ...;
```

### Example — High Salary Users View
```sql
CREATE VIEW high_salary_users AS
SELECT id, name, salary
FROM users
WHERE salary > 70000;
```

This creates a view named `high_salary_users` that always shows users earning more than ₹70,000.

---

## Querying a View

Once created, query it exactly like a regular table:

```sql
SELECT * FROM high_salary_users;
```

**Output:**

| id | name   | salary |
|----|--------|--------|
| 2  | Sneha  | 75000  |
| 5  | Fatima | 80000  |

---

## Views Are Always Live (Not Cached)

This is the most important property of Views — **they always reflect the current state of the base table**.

### Demonstration

**Step 1: Query the view (before any change)**
```sql
SELECT * FROM high_salary_users;
```
| id | name   | salary |
|----|--------|--------|
| 2  | Sneha  | 75000  |
| 5  | Fatima | 80000  |

**Step 2: Update the base table**
```sql
UPDATE users
SET salary = 72000
WHERE name = 'Raj';
```

**Step 3: Query the view again (no changes to the view itself)**
```sql
SELECT * FROM high_salary_users;
```
| id | name   | salary |
|----|--------|--------|
| 2  | Sneha  | 75000  |
| 5  | Fatima | 80000  |
| 3  | Raj    | 72000  |

> Raj now appears in the view automatically — because his salary crossed the ₹70,000 threshold. The view was never modified — it just runs its SELECT query fresh every time.

---

## Dropping a View

```sql
DROP VIEW high_salary_users;
```

> This removes the view definition only — the underlying `users` table and its data are completely unaffected.

---

## More View Examples

### View Hiding Sensitive Columns
```sql
CREATE VIEW public_users AS
SELECT id, name, gender
FROM users;
```
This view exposes only non-sensitive columns — email and salary are hidden from anyone querying `public_users`.

### View with a JOIN
```sql
CREATE VIEW user_addresses AS
SELECT u.name, a.city, a.state
FROM users u
LEFT JOIN addresses a ON u.id = a.user_id;
```
A complex JOIN becomes a simple `SELECT * FROM user_addresses` — much cleaner.

### View with Aggregation
```sql
CREATE VIEW salary_summary AS
SELECT gender, AVG(salary) AS avg_salary, COUNT(*) AS total
FROM users
GROUP BY gender;
```

---

## Key Takeaways — Views

| Concept | Key Point |
|---------|-----------|
| View | A saved SELECT query with a name — not a copy of data |
| Always live | Reflects the current state of the base table automatically |
| No data stored | Views don't duplicate data — only the query definition is saved |
| Drop view | Removes the view only — base table data is untouched |
| Column hiding | Expose only selected columns for restricted access |
| Simplification | Complex JOIN + WHERE logic saved as a simple named view |

---

# PART 2: INDEXES ⚡

## What is an Index?

> *"Indexes in MySQL are used to speed up data retrieval. They work like the index of a book — helping the database engine find rows faster, especially for searches, filters, and joins."*

**Without an index:** MySQL scans every single row from top to bottom to find matching records — called a **Full Table Scan**. Slow on large tables.

**With an index:** MySQL jumps directly to the relevant rows using a pre-built lookup structure — much faster.

### Book Index Analogy
If a book has 500 pages and you're looking for "Mumbai", you could read every page (full scan) or check the index at the back which tells you it's on pages 42, 87, and 203 (indexed lookup). Same concept in databases.

---

## Viewing Existing Indexes

```sql
SHOW INDEXES FROM users;
```

Shows all indexes currently on the `users` table — including the **automatically created PRIMARY KEY index** (MySQL always indexes the primary key automatically).

---

## Creating a Single-Column Index

Use when you frequently filter or search by a specific column.

```sql
CREATE INDEX idx_email ON users(email);
```

**What this does:**
- Creates an index named `idx_email` on the `email` column
- Speeds up queries that filter by email:

```sql
-- This query now uses the index — much faster on large tables
SELECT * FROM users WHERE email = 'alice@example.com';
```

**Naming convention:** `idx_` prefix + column name (e.g., `idx_email`, `idx_salary`) — makes it easy to identify.

---

## Creating a Multi-Column Index (Composite Index)

Use when you frequently filter by **two or more columns together**.

```sql
CREATE INDEX idx_gender_salary ON users(gender, salary);
```

This index is optimized for queries that filter on **both** columns:

```sql
-- This query uses the composite index efficiently
SELECT * FROM users
WHERE gender = 'Female' AND salary > 70000;
```

---

## ⚠️ Index Column Order Matters

For a composite index on `(gender, salary)`:

```sql
-- ✅ Uses the index efficiently — first column (gender) is present
WHERE gender = 'Female' AND salary > 70000;

-- ⚠️ May NOT use the index — first column (gender) is missing
WHERE salary > 70000;
```

**Rule:** MySQL can use a composite index only if the query includes the **leftmost column(s)** of the index. Skipping the first column defeats the index.

---

## Dropping an Index

```sql
DROP INDEX idx_email ON users;
```

---

## The Trade-Off — When to Use Indexes

Indexes are not free — they come with a cost:

| | Without Index | With Index |
|--|--------------|------------|
| **SELECT / WHERE** | Slow (full scan) | Fast (indexed lookup) |
| **INSERT / UPDATE / DELETE** | Fast | Slightly slower (index must be updated too) |
| **Disk space** | Less | More (index data is stored separately) |

### Use Indexes On
- Columns used frequently in `WHERE` clauses
- Columns used in `JOIN` conditions
- Columns used in `ORDER BY`
- Columns with high selectivity (many distinct values — e.g., `email`)

### Avoid Indexes On
- Columns that are rarely queried
- Columns with very few distinct values (e.g., a `gender` column with only 3 values — index barely helps)
- Small tables where a full scan is already fast

---

## Index Summary Table

| Feature | Detail |
|---------|--------|
| `SHOW INDEXES FROM table` | View all current indexes on a table |
| `CREATE INDEX name ON table(col)` | Create a single-column index |
| `CREATE INDEX name ON table(col1, col2)` | Create a composite index |
| `DROP INDEX name ON table` | Remove an index |
| Auto-created index | PRIMARY KEY always gets indexed automatically |
| Column order in composite | First column must be in WHERE for index to be used |
| Trade-off | Speeds up reads; slightly slows writes; uses disk space |

---

# PART 3: SUBQUERIES 🪆

## What is a Subquery?

> *"A subquery is a query nested inside another query. Subqueries are useful for breaking down complex problems into smaller parts."*

A subquery is written inside parentheses `( )` and is executed **first** — its result is then used by the outer query.

```sql
SELECT ...
FROM ...
WHERE column > (SELECT ... FROM ...);
                 ↑
           inner query runs first
```

---

## Where Can Subqueries Appear?

| Location | Purpose |
|----------|---------|
| `WHERE` clause | Filter rows based on a computed value |
| `SELECT` columns | Show a calculated value alongside each row |
| `FROM` clause | Use the subquery result as a virtual/derived table |
| `IN` clause | Filter rows where a column matches a list of values from a subquery |

---

## Type 1: Scalar Subquery (Returns One Value)

A **scalar subquery** returns exactly one value — typically used with comparison operators (`>`, `<`, `=`).

### Example — Find Users Earning Above Average

```sql
SELECT id, name, salary
FROM users
WHERE salary > (
  SELECT AVG(salary) FROM users
);
```

**How it works:**
1. Inner query runs first: `SELECT AVG(salary) FROM users` → returns e.g. `62000`
2. Outer query then becomes: `WHERE salary > 62000`
3. Returns all users earning above the average

> **Why not just type the number?** Because the average changes as data changes. The subquery always calculates it dynamically.

---

## Type 2: Subquery with IN (Returns Multiple Values)

A subquery returning **multiple values** is used with `IN` — it acts like a dynamic list.

### Example — Find Users Referred by High Earners

```sql
SELECT id, name, referred_by_id
FROM users
WHERE referred_by_id IN (
  SELECT id FROM users WHERE salary > 75000
);
```

**How it works:**
1. Inner query: `SELECT id FROM users WHERE salary > 75000` → returns e.g. `(2, 5)`
2. Outer query becomes: `WHERE referred_by_id IN (2, 5)`
3. Returns all users who were referred by someone earning more than ₹75,000

---

## Type 3: Subquery in SELECT Column (Inline Scalar)

Show a calculated value **alongside every row** — without changing which rows are returned.

### Example — Show Each User's Salary vs. Company Average

```sql
SELECT name, salary,
  (SELECT AVG(salary) FROM users) AS average_salary
FROM users;
```

**Output:**

| name    | salary | average_salary |
|---------|--------|----------------|
| Alice   | 72000  | 62000          |
| Bob     | 55000  | 62000          |
| Charlie | 48000  | 62000          |

The average salary column shows the same value for every row — it's computed once by the subquery and displayed alongside each record. Great for comparison reports.

---

## Type 4: Subquery in FROM Clause (Derived Table)

Use a subquery as a **temporary virtual table** inside the FROM clause. Must be given an alias.

### Example — Average of the Top Earners

```sql
SELECT AVG(top_salary) AS avg_of_top
FROM (
  SELECT salary AS top_salary
  FROM users
  WHERE salary > 70000
) AS high_earners;
```

**How it works:**
1. Inner query creates a virtual table `high_earners` with salaries > ₹70,000
2. Outer query calculates the average of those filtered salaries

> The alias (`AS high_earners`) is **mandatory** — MySQL requires a name for derived tables in the FROM clause.

---

## Subquery vs JOIN — When to Use Which

| Situation | Prefer |
|-----------|--------|
| Filter based on a single computed value (avg, max, etc.) | **Subquery** |
| Need to display columns from both tables | **JOIN** |
| Check if a value exists in another table | **Subquery with IN** or **EXISTS** |
| Combining large datasets with performance concern | **JOIN** (generally faster) |

---

## Subquery Types — Full Summary

| Subquery Type | Where Used | Returns | Example Use Case |
|---------------|-----------|---------|-----------------|
| **Scalar Subquery** | `WHERE`, `SELECT` | One value | Users earning above average salary |
| **Subquery with IN** | `WHERE ... IN` | Multiple values | Users referred by high earners |
| **Subquery in SELECT** | Column list | One value per row | Show individual vs average salary |
| **Derived Table** | `FROM` | A virtual table | Aggregate on a filtered subset |

---

## 🎯 OVERALL KEY TAKEAWAYS — Day 10

| Topic | Most Important Point |
|-------|---------------------|
| **View** | A saved SELECT query — not stored data; always reflects live base table |
| **View is live** | Change the base table → view auto-updates without modifying the view |
| **Drop View** | Removes only the view definition — base table is untouched |
| **Index** | Pre-built lookup structure; speeds up SELECT, slows down writes slightly |
| **PRIMARY KEY index** | Auto-created by MySQL — always indexed |
| **Composite index** | Order matters — first column in index must appear in WHERE |
| **Index trade-off** | Faster reads, slower writes, uses disk space — use only where needed |
| **Scalar subquery** | Inner query returns one value; used with `>`, `<`, `=` |
| **Subquery with IN** | Inner query returns multiple values; acts as a dynamic list |
| **Subquery in SELECT** | Shows a calculated value alongside every row |
| **Derived table** | Subquery in FROM clause; must have an alias |

---

## ⚡ Pro Tips — Day 10

1. **Views don't cache data** — they re-run the query every time; for very complex views on huge tables, performance can be a concern
2. **Use views to enforce column-level security** — expose only the columns a role should see
3. **Name your indexes with `idx_` prefix** — keeps them identifiable in `SHOW INDEXES` output
4. **Don't over-index** — every index slows down INSERT/UPDATE/DELETE; only index columns you actually filter on
5. **Composite index column order = query column order** — put the most selective (highest variety) column first
6. **Run EXPLAIN before and after adding an index** — use `EXPLAIN SELECT ...` to see if MySQL is actually using your index
7. **Scalar subqueries run once** — MySQL computes the inner query value once and reuses it for all rows in the outer query
8. **Subquery with IN vs JOIN** — subqueries are easier to read for simple lookups; JOINs are faster for large datasets
9. **Always alias derived tables** — `FROM (SELECT ...) AS alias` — MySQL throws an error without the alias
10. **Test subqueries in isolation first** — run the inner query alone to verify it returns what you expect before wrapping it

---

## Practice Exercises — Day 10

**Views:**
- [ ] Create a view `active_users` showing only users with salary > ₹50,000
- [ ] Query the view — verify the output
- [ ] Update a user's salary in the base table to cross ₹50,000 — query the view again and confirm they appear
- [ ] Create a view that hides the `email` column from the users table
- [ ] Create a view using a JOIN (user names + their city from addresses)
- [ ] Drop the view and confirm it no longer exists

**Indexes:**
- [ ] Run `SHOW INDEXES FROM users` — note the auto-created PRIMARY KEY index
- [ ] Create an index on the `name` column
- [ ] Create a composite index on `(gender, salary)`
- [ ] Test: write a WHERE query using only `salary` — check if the composite index is used (`EXPLAIN`)
- [ ] Test: write a WHERE query using both `gender` and `salary` — confirm the composite index is used
- [ ] Drop the index and run `SHOW INDEXES` again to confirm it's gone

**Subqueries:**
- [ ] Write a scalar subquery to find all users earning above the average salary
- [ ] Write a subquery with IN to find users referred by someone earning > ₹60,000
- [ ] Write a SELECT subquery to display each user's salary alongside the company average
- [ ] Write a derived table subquery to find the average salary of users born after 1995
- [ ] Try running just the inner query alone first — confirm it returns the expected result before wrapping it

---