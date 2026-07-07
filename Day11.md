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

# PART 3: TRIGGERS 🔔

## What is a Trigger?

> *"A trigger is a special kind of stored program that is automatically executed (triggered) when a specific event occurs in a table — such as INSERT, UPDATE, or DELETE."*

Unlike a stored procedure which you call manually, a **trigger fires automatically** in response to a data change event — you don't call it, MySQL calls it for you.

---

## Common Use Cases for Triggers

| Use Case | How Triggers Help |
|----------|------------------|
| **Audit logging** | Log every INSERT/UPDATE/DELETE to a separate history table |
| **Business rules** | Automatically enforce rules that can't be expressed as constraints |
| **Auto-update related data** | When one table changes, automatically update another |
| **Data validation** | Reject or transform data before it's written |

---

## Basic Trigger Structure

```sql
CREATE TRIGGER trigger_name
AFTER INSERT ON table_name
FOR EACH ROW
BEGIN
    -- SQL statements to execute automatically
END;
```

**Key parts explained:**

| Part | Options | Meaning |
|------|---------|---------|
| `BEFORE / AFTER` | `BEFORE`, `AFTER` | When the trigger fires relative to the event |
| `INSERT / UPDATE / DELETE` | Any one of the three | Which type of data change activates the trigger |
| `FOR EACH ROW` | (fixed) | The trigger body runs once for every affected row |
| `NEW.column` | Available in INSERT, UPDATE | Refers to the **new** value being written |
| `OLD.column` | Available in UPDATE, DELETE | Refers to the **old** value before the change |

---

## Trigger Timing — BEFORE vs AFTER

| Timing | When It Fires | Typical Use |
|--------|--------------|-------------|
| `BEFORE` | Before the data change is written | Validate or modify the data before it's saved |
| `AFTER` | After the data change is committed | Log the change, update related tables |

---

## Full Example: Auto-Log Every New User Insertion

### Step 1: Create the Log Table
```sql
CREATE TABLE user_log (
  id         INT AUTO_INCREMENT PRIMARY KEY,
  user_id    INT,
  name       VARCHAR(100),
  created_on TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

This table stores a record every time a new user is added to the `users` table.

### Step 2: Create the Trigger
```sql
DELIMITER $$

CREATE TRIGGER after_user_insert
AFTER INSERT ON users
FOR EACH ROW
BEGIN
    INSERT INTO user_log (user_id, name)
    VALUES (NEW.id, NEW.name);
END$$

DELIMITER ;
```

**Explanation:**
- `AFTER INSERT ON users` → fires after every INSERT into the `users` table
- `NEW.id` → the `id` of the newly inserted user
- `NEW.name` → the `name` of the newly inserted user
- The trigger automatically inserts a log entry without any manual action

### Step 3: Test the Trigger

Insert a new user (using the stored procedure from Part 2):
```sql
CALL AddUser('Ritika Jain', 'ritika@example.com', 'Female', '1996-03-12', 74000);
```

Now check the log table:
```sql
SELECT * FROM user_log;
```

**Output:** Ritika's information appears automatically in `user_log` — the trigger fired silently in the background.

---

## NEW and OLD — Accessing Row Data in Triggers

| Keyword | Available In | What It Refers To |
|---------|-------------|-------------------|
| `NEW.column` | `INSERT`, `UPDATE` | The new value being inserted/updated |
| `OLD.column` | `UPDATE`, `DELETE` | The original value before the change |

### Example — Trigger Using OLD (Log Deleted Users)
```sql
DELIMITER $$

CREATE TRIGGER before_user_delete
BEFORE DELETE ON users
FOR EACH ROW
BEGIN
    INSERT INTO user_log (user_id, name)
    VALUES (OLD.id, OLD.name);
END$$

DELIMITER ;
```

When a user is deleted, their `id` and `name` are logged **before** the deletion happens — preserving the record.

---

## Dropping a Trigger

```sql
DROP TRIGGER IF EXISTS after_user_insert;
```

---

## Trigger Summary Table

| Trigger Component | Description |
|-------------------|-------------|
| `BEFORE INSERT` | Fires before a new row is inserted |
| `AFTER INSERT` | Fires after a new row is inserted |
| `BEFORE UPDATE` | Fires before an existing row is updated |
| `AFTER UPDATE` | Fires after an existing row is updated |
| `BEFORE DELETE` | Fires before a row is deleted |
| `AFTER DELETE` | Fires after a row is deleted |
| `NEW.column` | The incoming/new value (INSERT, UPDATE) |
| `OLD.column` | The outgoing/old value (UPDATE, DELETE) |
| `FOR EACH ROW` | Runs once per affected row |

---

## Stored Procedure vs Trigger — Key Difference

| | Stored Procedure | Trigger |
|--|-----------------|---------|
| **Execution** | Called manually with `CALL` | Fires automatically on data events |
| **Control** | You decide when it runs | MySQL runs it automatically |
| **Parameters** | Can accept input parameters | Cannot — uses `NEW`/`OLD` instead |
| **Use for** | Reusable query logic | Auditing, auto-updates, business rules |

---

# PART 4: MORE ON MYSQL 🔧

## Essential MySQL Features and Operators

This section covers important MySQL concepts that make queries more powerful and flexible.

---

## 1. Logical Operators

Used to combine multiple conditions in a `WHERE` clause.

| Operator | Description | Example |
|----------|-------------|---------|
| `AND` | All conditions must be true | `WHERE salary > 50000 AND gender = 'Male'` |
| `OR` | At least one condition must be true | `WHERE gender = 'Male' OR gender = 'Other'` |
| `NOT` | Reverses a condition | `WHERE NOT gender = 'Female'` |

```sql
-- AND: both conditions required
SELECT * FROM users WHERE salary > 50000 AND gender = 'Male';

-- OR: either condition sufficient
SELECT * FROM users WHERE gender = 'Male' OR gender = 'Other';

-- NOT: exclude a condition
SELECT * FROM users WHERE NOT gender = 'Female';

-- Combining all three
SELECT * FROM users
WHERE (salary > 60000 OR referred_by_id IS NOT NULL)
AND NOT gender = 'Other';
```

> **Tip:** Use parentheses `( )` when combining AND and OR to make the logic explicit — AND has higher precedence than OR by default.

---

## 2. Adding a Column to an Existing Table

Use `ALTER TABLE` to add a new column anytime:

```sql
ALTER TABLE users
ADD COLUMN city VARCHAR(100);
```

- New column is added with `NULL` values for all existing rows (unless a DEFAULT is specified)
- To add with a default value:

```sql
ALTER TABLE users
ADD COLUMN is_active BOOLEAN DEFAULT TRUE;
```

---

## 3. Wildcard Operators (with LIKE)

Wildcards are used with `LIKE` for **pattern matching** in text columns.

| Wildcard | Matches | Example |
|----------|---------|---------|
| `%` | Any sequence of characters (including zero) | `'A%'` → starts with A |
| `_` | Exactly one character | `'_a%'` → second letter is 'a' |

```sql
-- Names starting with 'A'
SELECT * FROM users WHERE name LIKE 'A%';

-- Names ending with 'a'
SELECT * FROM users WHERE name LIKE '%a';

-- Names where second letter is 'a'
SELECT * FROM users WHERE name LIKE '_a%';

-- Names containing 'raj' anywhere (case-insensitive in MySQL by default)
SELECT * FROM users WHERE name LIKE '%raj%';

-- Names that are exactly 5 characters long
SELECT * FROM users WHERE name LIKE '_____';
```

---

## 4. LIMIT with OFFSET (Pagination)

`LIMIT` restricts how many rows are returned. `OFFSET` skips a number of rows before starting.

```sql
-- Get rows 11–15 (skip first 10, return next 5)
SELECT * FROM users
ORDER BY id
LIMIT 5 OFFSET 10;
```

**Alternative shorthand syntax:**
```sql
-- LIMIT offset, count
SELECT * FROM users
ORDER BY id
LIMIT 10, 5;
```

Both return the same result: skip 10 rows, return the next 5.

### Real-World Use Case — Pagination
```sql
-- Page 1: rows 1-10
SELECT * FROM users ORDER BY id LIMIT 10 OFFSET 0;

-- Page 2: rows 11-20
SELECT * FROM users ORDER BY id LIMIT 10 OFFSET 10;

-- Page 3: rows 21-30
SELECT * FROM users ORDER BY id LIMIT 10 OFFSET 20;
```

**Formula:** `OFFSET = (page_number - 1) × rows_per_page`

---

## 5. DISTINCT Keyword

Returns only **unique/distinct values** — removes duplicate rows from results.

```sql
-- List of unique genders (no duplicates)
SELECT DISTINCT gender FROM users;
```

**Output:**

| gender |
|--------|
| Male   |
| Female |
| Other  |

```sql
-- Count of unique cities users come from
SELECT COUNT(DISTINCT city) AS unique_cities FROM users;

-- Unique salary values in the table
SELECT DISTINCT salary FROM users ORDER BY salary DESC;
```

---

## 6. TRUNCATE vs DELETE

Both remove data from a table — but they work differently:

```sql
-- TRUNCATE: removes ALL rows instantly
TRUNCATE TABLE users;

-- DELETE without WHERE: also removes all rows (but slower)
DELETE FROM users;
```

| | `TRUNCATE TABLE` | `DELETE FROM` (no WHERE) |
|--|-----------------|--------------------------|
| **Speed** | Faster | Slower |
| **Rollback** | Cannot be rolled back | Can be rolled back (in a transaction) |
| **AUTO_INCREMENT reset** | ✅ Resets to 1 | ❌ Counter keeps going |
| **Triggers fired** | ❌ No | ✅ Yes |
| **Table structure** | Kept | Kept |

> **Use TRUNCATE when** you want to completely wipe a table fast and don't need rollback or trigger support.

---

## 7. CHANGE vs MODIFY Column

Both alter an existing column — but they serve slightly different purposes.

### MODIFY — Change Datatype Only

```sql
ALTER TABLE users
MODIFY COLUMN salary BIGINT;
```

Changes only the **data type** of the `salary` column. Column name stays the same.

### CHANGE — Rename and/or Change Datatype

```sql
ALTER TABLE users
CHANGE COLUMN city location VARCHAR(150);
```

**Renames** `city` → `location` AND changes its data type to `VARCHAR(150)` in one step.

| Command | Renames Column? | Changes Datatype? |
|---------|----------------|-------------------|
| `MODIFY` | ❌ No | ✅ Yes |
| `CHANGE` | ✅ Yes | ✅ Yes (required) |

> When using `CHANGE`, you must always specify the data type — even if you're not changing it.

---

## 🎯 OVERALL KEY TAKEAWAYS — Day 11

| Topic | Most Important Point |
|-------|---------------------|
| **GROUP BY** | Groups rows by a column; used with aggregate functions |
| **WHERE** | Filters rows before grouping; cannot use aggregates |
| **HAVING** | Filters groups after aggregation; can use aggregates |
| **WITH ROLLUP** | Adds subtotals and grand total to GROUP BY output |
| **Stored Procedure** | Saved reusable SQL block; called manually with `CALL` |
| **DELIMITER $$** | Must change delimiter when writing procedures to avoid `;` conflict |
| **IN parameter** | Input value passed into a procedure at call time |
| **Trigger** | Fires automatically on INSERT / UPDATE / DELETE events |
| **BEFORE vs AFTER** | BEFORE = validate/modify before save; AFTER = log/update after save |
| **NEW / OLD** | Access new/old row values inside a trigger |
| **Trigger vs Procedure** | Trigger = automatic; Procedure = manual `CALL` |
| **DISTINCT** | Returns unique values only; removes duplicates |
| **TRUNCATE** | Wipes all rows fast; resets AUTO_INCREMENT; no rollback |
| **MODIFY vs CHANGE** | MODIFY changes type only; CHANGE renames + changes type |
| **OFFSET** | Skips N rows before returning results; used for pagination |

---

## ⚡ Pro Tips — Day 11

1. **Always think WHERE vs HAVING** — if the condition involves an aggregate (`AVG`, `COUNT`, `SUM`), it must go in `HAVING`, not `WHERE`
2. **HAVING without GROUP BY** — technically works but applies the aggregate filter to the whole table as one group; rarely useful
3. **Name your procedures clearly** — `AddUser`, `GetHighEarners`, `UpdateSalary` — self-documenting names save time
4. **Always reset DELIMITER back to `;`** after creating a procedure — forgetting this breaks all subsequent queries
5. **Use `DROP PROCEDURE IF EXISTS`** before `CREATE PROCEDURE` when rewriting — avoids "already exists" errors
6. **Triggers are invisible** — they fire silently; document all triggers carefully or future developers won't know they exist
7. **`NEW` in BEFORE triggers** is writable — you can modify `NEW.column` to change the data before it's saved (useful for auto-formatting or validation)
8. **TRUNCATE resets AUTO_INCREMENT** — useful when wiping test data and restarting IDs from 1; DELETE does not reset it
9. **DISTINCT + COUNT** → `COUNT(DISTINCT col)` counts unique non-NULL values in a column — commonly used in analytics
10. **Use parentheses with AND/OR** — `WHERE (a OR b) AND c` vs `WHERE a OR (b AND c)` give different results; always be explicit

---

## Practice Exercises — Day 11

**GROUP BY & HAVING:**
- [ ] Write a query to find total salary paid per gender
- [ ] Find the gender with the highest average salary using HAVING
- [ ] Count how many users each person has referred — filter to show only those who referred more than 1 person
- [ ] Use `WITH ROLLUP` to add a grand total row to a gender-count report
- [ ] Combine `WHERE` and `HAVING` in a single query (e.g., only consider users with salary > 60000, then group by gender, then filter groups with count > 1)

**Stored Procedures:**
- [ ] Create a `GetAllUsers` procedure with no parameters
- [ ] Create an `AddUser` procedure with 5 IN parameters — call it to insert 3 new users
- [ ] View all procedures using `SHOW PROCEDURE STATUS`
- [ ] Drop the procedure and confirm it's gone with `SHOW PROCEDURE STATUS`
- [ ] Create a `GetUserByGender` procedure that accepts a gender as IN parameter and returns matching users

**Triggers:**
- [ ] Create a `user_log` table for audit logging
- [ ] Create an `AFTER INSERT` trigger on `users` that logs new insertions
- [ ] Test: insert a user and verify the log table was updated automatically
- [ ] Create a `BEFORE DELETE` trigger using `OLD` to log deleted user info
- [ ] Drop a trigger and confirm it no longer fires

**More on MySQL:**
- [ ] Use `AND`, `OR`, and `NOT` in a complex WHERE clause
- [ ] Test all LIKE patterns: `'A%'`, `'%a'`, `'_a%'`, `'%raj%'`
- [ ] Practice pagination: write 3 queries for page 1, 2, and 3 with 5 rows per page
- [ ] Use `SELECT DISTINCT` on gender and city columns
- [ ] Run `TRUNCATE TABLE` on a test table — verify AUTO_INCREMENT resets
- [ ] Use `MODIFY` to change a column's datatype
- [ ] Use `CHANGE` to rename a column and change its datatype in one step

---