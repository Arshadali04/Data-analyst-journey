# Day 11: SQL — GROUP BY & HAVING, Stored Procedures, Triggers & More on MySQL

## 📋 Topics Covered

**SQL (MySQL):**
1. GROUP BY and HAVING
2. Triggers
3. Stored Procedures
4. More on MySQL

---

# PART 1: GROUP BY AND HAVING 📊

## What is GROUP BY?

> *"The GROUP BY clause is used to group rows that have the same values in specified columns. It is typically used with aggregate functions like COUNT, SUM, AVG, MIN, or MAX."*

Without GROUP BY, aggregate functions like `AVG()` or `COUNT()` return **one result for the whole table**. With GROUP BY, they return **one result per group**.

---

## Sample Table Used in Examples

| id | name   | gender | salary | referred_by_id |
|----|--------|--------|--------|----------------|
| 1  | Aarav  | Male   | 80000  | NULL           |
| 2  | Sneha  | Female | 75000  | 1              |
| 3  | Raj    | Male   | 72000  | 1              |
| 4  | Fatima | Female | 85000  | 2              |
| 5  | Priya  | Female | 70000  | NULL           |

---

## GROUP BY — Basic Examples

### Example 1: Average Salary by Gender

```sql
SELECT gender, AVG(salary) AS average_salary
FROM users
GROUP BY gender;
```

**How it works:**
- Splits the table into groups: one group for `Male`, one for `Female`
- Calculates `AVG(salary)` for each group separately

**Output:**

| gender | average_salary |
|--------|----------------|
| Male   | 76000          |
| Female | 76666.67       |

---

### Example 2: COUNT — How Many Users Were Referred by Each User

```sql
SELECT referred_by_id, COUNT(*) AS total_referred
FROM users
WHERE referred_by_id IS NOT NULL
GROUP BY referred_by_id;
```

**Output:**

| referred_by_id | total_referred |
|----------------|----------------|
| 1              | 2              |
| 2              | 1              |

- `WHERE referred_by_id IS NOT NULL` filters out non-referred users **before** grouping
- `GROUP BY referred_by_id` then groups the remaining rows by referrer
- `COUNT(*)` counts how many rows exist in each group

---

## The Full Query Execution Order

Understanding the order SQL processes clauses helps avoid confusion:

```
1. FROM       → Which table?
2. WHERE      → Filter individual rows (before grouping)
3. GROUP BY   → Form groups from the remaining rows
4. HAVING     → Filter groups (after aggregation)
5. SELECT     → Choose what to display
6. ORDER BY   → Sort the result
7. LIMIT      → Cap the output
```

---

## HAVING — Filtering Groups

> *"The HAVING clause is used to filter groups after aggregation — similar to how WHERE filters individual rows."*

### WHERE vs HAVING — The Key Difference

| Clause | When It Runs | Can Use Aggregates? | Example |
|--------|-------------|---------------------|---------|
| `WHERE` | **Before** grouping | ❌ No | `WHERE salary > 50000` |
| `HAVING` | **After** grouping | ✅ Yes | `HAVING AVG(salary) > 75000` |

> **Why can't WHERE use aggregates?** Because aggregates like `AVG()` are calculated during the GROUP BY step — they don't exist yet when WHERE runs.

---

### Example 1: Genders Where Average Salary > ₹75,000

```sql
SELECT gender, AVG(salary) AS avg_salary
FROM users
GROUP BY gender
HAVING AVG(salary) > 75000;
```

**Output:**

| gender | avg_salary |
|--------|------------|
| Female | 76666.67   |

Only groups (genders) whose average salary exceeds ₹75,000 appear.

---

### Example 2: Referrers Who Referred More Than 1 Person

```sql
SELECT referred_by_id, COUNT(*) AS total_referred
FROM users
WHERE referred_by_id IS NOT NULL
GROUP BY referred_by_id
HAVING COUNT(*) > 1;
```

- `WHERE` removes NULL referrers before grouping
- `GROUP BY` groups by referrer
- `HAVING` keeps only groups where count > 1

**Output:**

| referred_by_id | total_referred |
|----------------|----------------|
| 1              | 2              |

---

## Combining WHERE and HAVING

Both can be used in the same query — they filter at different stages:

```sql
SELECT gender, AVG(salary) AS avg_salary
FROM users
WHERE salary > 60000          -- filter rows BEFORE grouping
GROUP BY gender
HAVING AVG(salary) > 74000;  -- filter groups AFTER aggregation
```

---

## ROLLUP — Subtotals and Grand Totals

`WITH ROLLUP` adds **subtotal rows and a grand total row** to your GROUP BY result automatically.

```sql
SELECT gender, COUNT(*) AS total_users
FROM users
GROUP BY gender WITH ROLLUP;
```

**Output:**

| gender | total_users |
|--------|-------------|
| Female | 3           |
| Male   | 2           |
| NULL   | 5           |

- The `NULL` row at the bottom is the **grand total** — total users across all groups
- Great for management summary reports where subtotals are needed

---

## Key Takeaways — GROUP BY & HAVING

| Concept | Key Point |
|---------|-----------|
| `GROUP BY` | Groups rows with the same value; used with aggregate functions |
| `WHERE` | Filters individual rows **before** grouping; cannot use aggregates |
| `HAVING` | Filters groups **after** aggregation; can use aggregate functions |
| Execution order | WHERE → GROUP BY → HAVING |
| `WITH ROLLUP` | Adds subtotals and a grand total row to the result |
| Multi-column GROUP BY | `GROUP BY gender, city` — groups by both columns together |

---

# PART 2: STORED PROCEDURES ⚙️

## What is a Stored Procedure?

> *"A stored procedure is a saved SQL block that can be executed later. It's useful when you want to encapsulate logic that can be reused multiple times — like queries, updates, or conditional operations."*

Think of a Stored Procedure like a **function in programming** — you write it once, give it a name, and call it whenever you need it, with different inputs each time.

---

## Why Use Stored Procedures?

| Benefit | Description |
|---------|-------------|
| **Reusability** | Write once, call many times with different inputs |
| **Consistency** | Same logic runs every time — no risk of typos in repeated queries |
| **Simplicity** | Complex multi-step SQL becomes a single `CALL` statement |
| **Performance** | Procedures are compiled and cached by MySQL — faster than raw SQL each time |
| **Security** | Grant users access to run a procedure without giving them direct table access |

---

## The Delimiter Problem (Why DELIMITER $$?)

By default, MySQL uses `;` to mark the end of a SQL statement. But inside a stored procedure, you also use `;` to end each statement within the procedure body.

**The conflict:** MySQL sees the first `;` inside the procedure and thinks the entire CREATE PROCEDURE statement has ended — which breaks the procedure definition.

**The fix:** Temporarily change the delimiter to `$$` (or `//`) while writing the procedure. After the procedure is defined, reset it back to `;`.

```sql
DELIMITER $$     -- change delimiter to $$

CREATE PROCEDURE my_procedure()
BEGIN
    SELECT * FROM users;   -- ; here is fine now — MySQL ignores it as a statement ender
END$$            -- $$ marks the true end of the procedure

DELIMITER ;      -- reset delimiter back to ;
```

---

## Creating a Simple Stored Procedure (No Parameters)

```sql
DELIMITER $$

CREATE PROCEDURE GetAllUsers()
BEGIN
    SELECT * FROM users;
END$$

DELIMITER ;
```

### Calling It
```sql
CALL GetAllUsers();
```

Every time you run `CALL GetAllUsers()`, it executes `SELECT * FROM users` — no need to retype the query.

---

## Creating a Procedure with Input Parameters (IN)

Input parameters let you pass values into the procedure when calling it — making it dynamic.

```sql
DELIMITER $$

CREATE PROCEDURE AddUser(
    IN p_name   VARCHAR(100),
    IN p_email  VARCHAR(100),
    IN p_gender ENUM('Male', 'Female', 'Other'),
    IN p_dob    DATE,
    IN p_salary INT
)
BEGIN
    INSERT INTO users (name, email, gender, date_of_birth, salary)
    VALUES (p_name, p_email, p_gender, p_dob, p_salary);
END$$

DELIMITER ;
```

**Explanation:**
- `IN` keyword → declares an input parameter (value passed from caller into procedure)
- `p_name`, `p_email`, etc. → parameter names (convention: prefix with `p_` to distinguish from column names)
- The parameter types must match the column data types in the table

### Calling the Procedure with Values
```sql
CALL AddUser('Kiran Sharma', 'kiran@example.com', 'Female', '1994-06-15', 72000);
```

This inserts a new user with the provided details — same as writing the full INSERT, but cleaner and reusable.

---

## Viewing All Stored Procedures

```sql
SHOW PROCEDURE STATUS WHERE Db = 'startersql';
```

Lists all stored procedures in the `startersql` database — shows name, type, creation time, etc.

---

## Dropping a Stored Procedure

```sql
DROP PROCEDURE IF EXISTS AddUser;
```

- `IF EXISTS` prevents an error if the procedure doesn't exist
- After dropping, `CALL AddUser(...)` will no longer work

---

## Stored Procedure Commands — Summary

| Command | Purpose |
|---------|---------|
| `DELIMITER $$` | Temporarily change statement delimiter |
| `CREATE PROCEDURE name()` | Define a stored procedure without parameters |
| `CREATE PROCEDURE name(IN p_col TYPE)` | Define with input parameters |
| `BEGIN ... END$$` | Marks the body of the procedure |
| `CALL procedure_name(...)` | Execute the stored procedure |
| `SHOW PROCEDURE STATUS WHERE Db = 'dbname'` | List all procedures in a database |
| `DROP PROCEDURE IF EXISTS name` | Remove a stored procedure |
| `DELIMITER ;` | Reset delimiter back to default |

---

