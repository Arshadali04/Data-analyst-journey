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

