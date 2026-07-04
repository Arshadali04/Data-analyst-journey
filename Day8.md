# Day 8: SQL — Querying, Updating, Deleting, Constraints, Functions, Transactions & Primary Keys

## 📋 Topics Covered

**SQL (MySQL):**
1. Using Starter SQL
2. Querying Data
3. Updating the Data
4. Deleting Data
5. Constraints in Detail
6. Functions in MySQL
7. Auto Commit and Transactions
8. Primary Key & Auto Increment

---

# PART 1: USING STARTER SQL 🚀

## Setting Up the Database

Before writing any queries, you need a database to work in. This is the standard setup used throughout the course.

### Step 1: Create the Database
```sql
CREATE DATABASE startersql;
```

### Step 2: Switch to It
Either right-click the database in MySQL Workbench → **Set as Default Schema**, or run:
```sql
USE startersql;
```

### Step 3: Create the Users Table
```sql
CREATE TABLE users (
  id          INT AUTO_INCREMENT PRIMARY KEY,
  name        VARCHAR(100) NOT NULL,
  email       VARCHAR(100) UNIQUE NOT NULL,
  gender      ENUM('Male', 'Female', 'Other'),
  date_of_birth DATE,
  salary      DECIMAL(10, 2),
  created_at  TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

### Step 4: Insert Sample Data
```sql
INSERT INTO users (name, email, gender, date_of_birth, salary) VALUES
('Alice',   'alice@example.com',   'Female', '1995-05-14', 72000),
('Bob',     'bob@example.com',     'Male',   '1990-11-23', 55000),
('Charlie', 'charlie@example.com', 'Other',  '1988-02-17', 48000),
('David',   'david@example.com',   'Male',   '2000-08-09', 61000),
('Eva',     'eva@example.com',     'Female', '1993-12-30', 83000);
```

### Drop the Database (When Needed)
```sql
DROP DATABASE startersql;
```
> ⚠️ **Warning:** This deletes the entire database and all its tables permanently. Always double-check before running.

---

# PART 2: QUERYING DATA 🔍

## The SELECT Statement

The `SELECT` statement is how you read/retrieve data from a table.

### Select All Columns
```sql
SELECT * FROM users;
```
Fetches every column and every row from the `users` table. The `*` means "all columns".

### Select Specific Columns
```sql
SELECT name, email FROM users;
```
Fetches only the `name` and `email` columns — cleaner and faster than `SELECT *` on large tables.

---

## Filtering Rows with WHERE

`WHERE` narrows down results to only rows that match a condition.

### Comparison Operators

| Operator | Meaning | Example |
|----------|---------|---------|
| `=` | Equal to | `WHERE gender = 'Male'` |
| `!=` or `<>` | Not equal to | `WHERE gender != 'Female'` |
| `>` | Greater than | `WHERE id > 10` |
| `<` | Less than | `WHERE date_of_birth < '1995-01-01'` |
| `>=` | Greater than or equal | `WHERE id >= 5` |
| `<=` | Less than or equal | `WHERE id <= 20` |

```sql
-- Equal to
SELECT * FROM users WHERE gender = 'Male';

-- Not equal to (both work the same)
SELECT * FROM users WHERE gender != 'Female';
SELECT * FROM users WHERE gender <> 'Female';

-- Greater than / Less than
SELECT * FROM users WHERE id > 10;
SELECT * FROM users WHERE date_of_birth < '1995-01-01';

-- Greater than or equal / Less than or equal
SELECT * FROM users WHERE id >= 5;
SELECT * FROM users WHERE id <= 20;
```

---

## Working with NULL

`NULL` means a value is missing or unknown. You cannot use `=` or `!=` with NULL — you must use `IS NULL` or `IS NOT NULL`.

```sql
-- Find rows where date_of_birth is missing
SELECT * FROM users WHERE date_of_birth IS NULL;

-- Find rows where date_of_birth exists
SELECT * FROM users WHERE date_of_birth IS NOT NULL;
```

> **Why not `= NULL`?** Because `NULL` is not a value — it's the absence of one. `NULL = NULL` evaluates to unknown in SQL, not TRUE.

---

## BETWEEN

Selects rows where a value falls within an inclusive range.

```sql
SELECT * FROM users WHERE date_of_birth BETWEEN '1990-01-01' AND '2000-12-31';
```

- Both endpoints are **inclusive** (includes 1990-01-01 and 2000-12-31)
- Works on dates, numbers, and text

---

## IN

Checks if a value matches **any item in a list** — cleaner than writing multiple OR conditions.

```sql
SELECT * FROM users WHERE gender IN ('Male', 'Other');
```

This is equivalent to:
```sql
SELECT * FROM users WHERE gender = 'Male' OR gender = 'Other';
```

---

## LIKE (Pattern Matching)

Used to search for a pattern in text data. Uses two wildcards:
- `%` → any number of characters (including zero)
- `_` → exactly one character

```sql
-- Names starting with 'A'
SELECT * FROM users WHERE name LIKE 'A%';

-- Names ending with 'a'
SELECT * FROM users WHERE name LIKE '%a';

-- Names containing 'li' anywhere
SELECT * FROM users WHERE name LIKE '%li%';
```

---

## AND / OR

Combine multiple conditions in a single WHERE clause.

```sql
-- AND: BOTH conditions must be true
SELECT * FROM users WHERE gender = 'Female' AND date_of_birth > '1990-01-01';

-- OR: AT LEAST ONE condition must be true
SELECT * FROM users WHERE gender = 'Male' OR gender = 'Other';
```

---

## ORDER BY

Sorts the result set by one or more columns.

```sql
-- Ascending (A→Z, oldest→newest, smallest→largest) — default
SELECT * FROM users ORDER BY date_of_birth ASC;

-- Descending (Z→A, newest→oldest, largest→smallest)
SELECT * FROM users ORDER BY name DESC;
```

---

## LIMIT

Restricts how many rows are returned — useful for Top N queries and pagination.

```sql
-- Top 5 rows only
SELECT * FROM users LIMIT 5;

-- Skip first 5 rows, then get the next 10
SELECT * FROM users LIMIT 10 OFFSET 5;

-- Same as above (shorthand: LIMIT skip, count)
SELECT * FROM users LIMIT 5, 10;

-- Most recent 10 users (combine ORDER BY + LIMIT)
SELECT * FROM users ORDER BY created_at DESC LIMIT 10;
```

---

## Combining Clauses — Full Query Structure

The standard order of clauses in a SELECT query:

```sql
SELECT   columns
FROM     table
WHERE    condition(s)
ORDER BY column ASC/DESC
LIMIT    number;
```

### Example Queries

```sql
-- Top 5 highest-paid users created recently
SELECT * FROM users
WHERE salary > 60000
ORDER BY created_at DESC
LIMIT 5;

-- All users sorted by salary (highest first)
SELECT * FROM users ORDER BY salary DESC;

-- Users earning between ₹50,000 and ₹70,000
SELECT * FROM users WHERE salary BETWEEN 50000 AND 70000;
```

---

# PART 3: UPDATING DATA ✏️

## The UPDATE Statement

Used to **modify existing data** in one or more rows of a table.

### Basic Syntax
```sql
UPDATE table_name
SET column1 = value1, column2 = value2
WHERE condition;
```

> ⚠️ **Critical Rule:** Always include a `WHERE` clause unless you intentionally want to update every row.

---

### Example 1: Update One Column
```sql
UPDATE users
SET name = 'Alicia'
WHERE id = 1;
```
Changes the name of the user with `id = 1` to "Alicia". Only one row is affected.

### Example 2: Update Multiple Columns at Once
```sql
UPDATE users
SET name = 'Robert', email = 'robert@example.com'
WHERE id = 2;
```
Updates both `name` and `email` for the user with `id = 2` in a single statement.

### Example 3: Update Based on a Calculation
```sql
UPDATE users
SET salary = salary + 10000
WHERE salary < 60000;
```
Increases salary by ₹10,000 for all users earning less than ₹60,000.

### ⚠️ Without WHERE — Danger Zone
```sql
UPDATE users
SET gender = 'Other';
```
> This updates **every single row** in the table. Never omit `WHERE` unless you truly mean to update all rows.

---

## Quick Quiz — UPDATE

```sql
-- 1. Update salary of user with id = 5 to ₹70,000
UPDATE users SET salary = 70000 WHERE id = 5;

-- 2. Change name of user with specific email
UPDATE users SET name = 'Aisha Khan' WHERE email = 'aisha@example.com';

-- 3. Increase salary by ₹10,000 for lower earners
UPDATE users SET salary = salary + 10000 WHERE salary < 60000;

-- 4. Change gender for a specific user
UPDATE users SET gender = 'Other' WHERE name = 'Ishaan';

-- 5. Reset all salaries (⚠️ affects ALL rows!)
UPDATE users SET salary = 50000;
```

---

## Best Practice Before Updating

Run a `SELECT` with the same `WHERE` clause first to preview what will be changed:
```sql
-- Preview first
SELECT * FROM users WHERE id = 5;

-- Then update
UPDATE users SET salary = 70000 WHERE id = 5;
```

---

# PART 4: DELETING DATA 🗑️

## The DELETE Statement

Used to **remove rows** from a table.

### Basic Syntax
```sql
DELETE FROM table_name
WHERE condition;
```

---

### Example 1: Delete One Row
```sql
DELETE FROM users WHERE id = 3;
```
Removes only the row where `id = 3`.

### Example 2: Delete Multiple Rows
```sql
DELETE FROM users WHERE gender = 'Other';
```
Removes all rows where gender is 'Other'.

### Example 3: Delete All Rows (Keep Table Structure)
```sql
DELETE FROM users;
```
Removes **all data** from the table but the table itself (its structure/schema) remains. You can still insert new rows after this.

### Example 4: Drop the Entire Table
```sql
DROP TABLE users;
```
Removes **both the data and the table structure** permanently. The table no longer exists after this.

---

## DELETE vs DROP — Key Difference

| Command | What Gets Removed | Table Still Exists? |
|---------|------------------|---------------------|
| `DELETE FROM users` | All rows (data) | ✅ Yes — structure remains |
| `DELETE FROM users WHERE ...` | Specific rows | ✅ Yes |
| `DROP TABLE users` | Everything (data + structure) | ❌ No |

---

## Best Practices for DELETE

1. **Always use WHERE** unless you intentionally want to wipe everything
2. **Run SELECT first** with the same WHERE condition to preview what will be deleted:
```sql
-- Preview
SELECT * FROM users WHERE id = 3;

-- Then delete
DELETE FROM users WHERE id = 3;
```
3. **Back up important data** before running any destructive operations

---

## Quick Quiz — DELETE

```sql
-- What happens?
DELETE FROM users WHERE salary < 50000;
-- Deletes all users earning less than ₹50,000

DELETE FROM users WHERE salary IS NULL;
-- Deletes all users with no salary recorded
```

---

# PART 5: CONSTRAINTS IN DETAIL 🔒

Constraints are **rules enforced on table columns** to ensure data accuracy, validity, and integrity.

---

## 1. UNIQUE Constraint

Ensures all values in a column are **different** — no duplicates allowed.

```sql
-- During table creation
CREATE TABLE users (
  id    INT PRIMARY KEY,
  email VARCHAR(100) UNIQUE
);

-- Add UNIQUE to an existing column
ALTER TABLE users
ADD CONSTRAINT unique_email UNIQUE (email);
```

> **Key Point:** A column can be UNIQUE but still allow one NULL value (since NULL ≠ NULL in SQL logic).

---

## 2. NOT NULL Constraint

Ensures a column **cannot contain NULL** — a value must always be provided.

```sql
-- During creation
CREATE TABLE users (
  id   INT PRIMARY KEY,
  name VARCHAR(100) NOT NULL
);

-- Add NOT NULL to existing column
ALTER TABLE users
MODIFY COLUMN name VARCHAR(100) NOT NULL;

-- Remove NOT NULL (make nullable again)
ALTER TABLE users
MODIFY COLUMN name VARCHAR(100) NULL;
```

---

## 3. CHECK Constraint

Ensures values in a column satisfy a **specific condition** — like a validation rule.

```sql
-- Only allow dates of birth after Jan 1, 2000
ALTER TABLE users
ADD CONSTRAINT chk_dob CHECK (date_of_birth > '2000-01-01');
```

> **Naming the constraint** (`chk_dob`) is a best practice — it makes it easy to drop or modify the specific constraint later by name.

---

## 4. DEFAULT Constraint

Sets a **default value** for a column when no value is explicitly provided during insert.

```sql
-- During creation
CREATE TABLE users (
  id        INT PRIMARY KEY,
  is_active BOOLEAN DEFAULT TRUE
);

-- Add DEFAULT to an existing column
ALTER TABLE users
ALTER COLUMN is_active SET DEFAULT TRUE;
```

**How it works:** If you insert a row without specifying `is_active`, MySQL automatically sets it to `TRUE`.

---

## 5. PRIMARY KEY Constraint

**Uniquely identifies every row** in a table. Must be both UNIQUE and NOT NULL.

```sql
-- During creation
CREATE TABLE users (
  id   INT PRIMARY KEY,
  name VARCHAR(100)
);

-- Add PRIMARY KEY later
ALTER TABLE users
ADD PRIMARY KEY (id);
```

- Each table can have **only one** PRIMARY KEY
- The PRIMARY KEY column(s) cannot contain NULL values
- Often combined with `AUTO_INCREMENT` (see Part 8)

---

## 6. AUTO_INCREMENT

Automatically assigns the next unique integer to each new row — always used with PRIMARY KEY.

```sql
CREATE TABLE users (
  id   INT AUTO_INCREMENT PRIMARY KEY,
  name VARCHAR(100)
);
```

Each new row inserted gets the next available number for `id` — you never have to manually specify it.

---

## Constraints Summary Table

| Constraint | Purpose | Allows NULL? | Multiple Per Table? |
|------------|---------|--------------|---------------------|
| `UNIQUE` | No duplicate values | ✅ Yes (one NULL) | ✅ Yes |
| `NOT NULL` | Value must always exist | ❌ No | ✅ Yes |
| `CHECK` | Value must pass a condition | ✅ Yes | ✅ Yes |
| `DEFAULT` | Auto-fills when no value given | ✅ Yes | ✅ Yes |
| `PRIMARY KEY` | Uniquely identifies each row | ❌ No | ❌ Only one |
| `AUTO_INCREMENT` | Auto-generates sequential numbers | ❌ No | ❌ Only one |

---

# PART 6: FUNCTIONS IN MYSQL 🧮

SQL functions help you **analyze, transform, or summarize** data directly in your queries — without exporting to Excel.

---

## 1. Aggregate Functions

Return a **single summarized value** from a set of rows.

### COUNT() — Count Rows
```sql
-- Total number of users
SELECT COUNT(*) FROM users;

-- Count only female users
SELECT COUNT(*) FROM users WHERE gender = 'Female';
```

### MIN() and MAX() — Lowest and Highest Value
```sql
SELECT MIN(salary) AS min_salary,
       MAX(salary) AS max_salary
FROM users;
```

### SUM() — Total of a Column
```sql
SELECT SUM(salary) AS total_payroll FROM users;
```

### AVG() — Average Value
```sql
SELECT AVG(salary) AS avg_salary FROM users;
```

### GROUP BY — Aggregate Per Group
Splits the table into groups and applies the aggregate function to each group separately.

```sql
-- Average salary broken down by gender
SELECT gender, AVG(salary) AS avg_salary
FROM users
GROUP BY gender;
```

---

## 2. String Functions

Used to manipulate and inspect text data.

### LENGTH() — Length of a String
```sql
SELECT name, LENGTH(name) AS name_length FROM users;
```

### LOWER() and UPPER() — Change Case
```sql
SELECT name, LOWER(name) AS lowercase_name FROM users;
SELECT name, UPPER(name) AS uppercase_name FROM users;
```

### CONCAT() — Combine Strings
```sql
-- Format: Name <email>
SELECT CONCAT(name, ' <', email, '>') AS user_contact FROM users;
```

---

## 3. Date Functions

Used to work with date and timestamp columns.

### NOW() — Current Date and Time
```sql
SELECT NOW();
-- Returns: 2026-07-04 14:32:00  (current server timestamp)
```

### YEAR(), MONTH(), DAY() — Extract Date Parts
```sql
SELECT name, YEAR(date_of_birth)  AS birth_year  FROM users;
SELECT name, MONTH(date_of_birth) AS birth_month FROM users;
SELECT name, DAY(date_of_birth)   AS birth_day   FROM users;
```

### DATEDIFF() — Days Between Two Dates
```sql
-- How many days has each user been alive?
SELECT name, DATEDIFF(CURDATE(), date_of_birth) AS days_lived FROM users;
```

### TIMESTAMPDIFF() — Difference in a Specific Unit
```sql
-- Calculate age in years
SELECT name, TIMESTAMPDIFF(YEAR, date_of_birth, CURDATE()) AS age FROM users;
```

---

## 4. Mathematical Functions

### ROUND(), FLOOR(), CEIL()
```sql
SELECT salary,
       ROUND(salary) AS rounded,   -- Rounds to nearest integer
       FLOOR(salary) AS floored,   -- Rounds DOWN always
       CEIL(salary)  AS ceiled     -- Rounds UP always
FROM users;
```

### MOD() — Remainder After Division
```sql
-- Find even or odd IDs (remainder 0 = even, remainder 1 = odd)
SELECT id, MOD(id, 2) AS remainder FROM users;
```

---

## 5. Conditional Functions

### IF() — Inline Conditional Logic
```sql
SELECT name, gender,
       IF(gender = 'Female', 'Yes', 'No') AS is_female
FROM users;
```

Syntax: `IF(condition, value_if_true, value_if_false)`

---

## Functions Summary Table

| Function | Category | Purpose |
|----------|----------|---------|
| `COUNT()` | Aggregate | Count rows |
| `SUM()` | Aggregate | Total of a column |
| `AVG()` | Aggregate | Average of values |
| `MIN()` / `MAX()` | Aggregate | Lowest / highest value |
| `GROUP BY` | Aggregate | Aggregate per group |
| `LENGTH()` | String | Character count of a string |
| `LOWER()` / `UPPER()` | String | Convert case |
| `CONCAT()` | String | Merge multiple strings |
| `NOW()` / `CURDATE()` | Date | Current date and time |
| `YEAR()` / `MONTH()` / `DAY()` | Date | Extract date parts |
| `DATEDIFF()` | Date | Days between two dates |
| `TIMESTAMPDIFF()` | Date | Difference in years/months/days |
| `ROUND()` / `FLOOR()` / `CEIL()` | Math | Rounding numbers |
| `MOD()` | Math | Remainder after division |
| `IF()` | Conditional | Inline if-else logic |

---

# PART 7: AUTO COMMIT AND TRANSACTIONS 💾

## What is AutoCommit?

By default, MySQL runs in **AutoCommit mode** — every SQL statement is immediately and permanently saved to the database the moment you run it. There is no "undo" button.

```
AutoCommit ON (default):
  Run UPDATE → Change is saved permanently. Done.
```

---

## What is a Transaction?

A **Transaction** is a group of SQL statements treated as a single unit of work. Either **all of them succeed** (COMMIT) or **none of them are saved** (ROLLBACK).

**Real-world analogy:**
> Transferring ₹10,000 from Account A to Account B involves two steps: debit Account A, then credit Account B. If the debit succeeds but the credit fails halfway through, you've lost ₹10,000. A transaction ensures BOTH succeed or NEITHER happens.

---

## Transaction Commands

### SET autocommit = 0 — Turn Off AutoCommit
```sql
SET autocommit = 0;
```
Now changes you make are **not saved** until you explicitly run `COMMIT`. This gives you a safety net to review changes before making them permanent.

### COMMIT — Save Changes Permanently
```sql
COMMIT;
```
Makes all changes since the last `COMMIT` or `ROLLBACK` permanent and visible to others.

### ROLLBACK — Undo All Unsaved Changes
```sql
ROLLBACK;
```
Reverts all changes back to the state at the last `COMMIT` or `ROLLBACK`. Like a full undo.

### SET autocommit = 1 — Turn AutoCommit Back On
```sql
SET autocommit = 1;
```
Returns MySQL to its default behavior where every statement auto-saves.

---

## Full Transaction Workflow Example

```sql
-- Step 1: Turn off AutoCommit
SET autocommit = 0;

-- Step 2: Make a change
UPDATE users SET salary = 80000 WHERE id = 5;

-- Step 3a: Happy with the change? Make it permanent
COMMIT;

-- Step 3b: Made a mistake? Undo everything
ROLLBACK;
```

---

## SAVEPOINT — Partial Rollback

A **SAVEPOINT** marks a specific point within a transaction. You can rollback to a savepoint instead of rolling back the entire transaction.

```sql
SET autocommit = 0;

UPDATE users SET salary = 80000 WHERE id = 5;

SAVEPOINT before_delete;   -- Mark this point

DELETE FROM users WHERE id = 3;

-- Oops! Rollback only to the savepoint (undoes DELETE but keeps UPDATE)
ROLLBACK TO before_delete;

-- Commit the remaining change (the salary UPDATE)
COMMIT;
```

---

## COMMIT vs ROLLBACK — Quick Comparison

| | COMMIT | ROLLBACK |
|--|--------|----------|
| **What it does** | Saves changes permanently | Undoes all unsaved changes |
| **When to use** | When you're sure everything is correct | When something went wrong |
| **After running** | Changes are visible to everyone | Changes disappear as if they never happened |

---

## Best Practices — Transactions

1. **Use transactions for multi-step operations** — especially when steps depend on each other
2. **Always COMMIT or ROLLBACK** — never leave a transaction open indefinitely
3. **Use SAVEPOINT** when you want the option to partially undo without restarting everything
4. **Test with SELECT first** — preview results before committing destructive changes
5. **Turn AutoCommit back ON** after your session if you turned it off manually

---

# PART 8: PRIMARY KEY & AUTO INCREMENT 🔑

## What is a PRIMARY KEY?

A `PRIMARY KEY` is a column (or combination of columns) that **uniquely identifies every row** in a table. It is one of the most important concepts in database design.

### Rules for PRIMARY KEY
- Must be **UNIQUE** — no two rows can share the same primary key value
- Cannot be **NULL** — every row must have a value
- Each table can have **only one** PRIMARY KEY
- Can be a **single column** or a **combination of columns** (composite key)

```sql
CREATE TABLE users (
  id   INT AUTO_INCREMENT PRIMARY KEY,
  name VARCHAR(100)
);
```

---

## PRIMARY KEY vs UNIQUE — What's the Difference?

| Feature | PRIMARY KEY | UNIQUE |
|---------|-------------|--------|
| Must be unique | ✅ Yes | ✅ Yes |
| Allows NULL | ❌ No | ✅ Yes (one or more NULLs allowed) |
| How many per table | ❌ Only one | ✅ Multiple allowed |
| Required | Strongly recommended | Optional |
| Can be dropped easily | ❌ More restricted | ✅ Can be dropped anytime |

### Example Showing Both Together
```sql
CREATE TABLE users (
  id    INT AUTO_INCREMENT PRIMARY KEY,  -- Primary Key: unique identifier
  email VARCHAR(100) UNIQUE,             -- Unique: no duplicate emails
  name  VARCHAR(100)
);
```

- `id` → uniquely identifies each row (Primary Key)
- `email` → must be unique, but is not the primary identifier

---

## Dropping Constraints

### Drop a PRIMARY KEY
```sql
ALTER TABLE users DROP PRIMARY KEY;
```
> May fail if the primary key column uses `AUTO_INCREMENT` — remove `AUTO_INCREMENT` first.

### Drop a UNIQUE Constraint
```sql
ALTER TABLE users DROP INDEX email;
```

---

## AUTO_INCREMENT — Automatic ID Generation

`AUTO_INCREMENT` works alongside `PRIMARY KEY` to automatically assign the next unique integer to each new row — you never need to manually specify the ID.

```sql
CREATE TABLE users (
  id   INT AUTO_INCREMENT PRIMARY KEY,
  name VARCHAR(100)
);
```

### How It Works
```sql
INSERT INTO users (name) VALUES ('Alice');   -- id = 1 (auto)
INSERT INTO users (name) VALUES ('Bob');     -- id = 2 (auto)
INSERT INTO users (name) VALUES ('Charlie'); -- id = 3 (auto)
```
You only insert `name` — MySQL handles `id` automatically.

### Changing the Starting Value
By default, `AUTO_INCREMENT` starts at 1. You can change this:
```sql
-- Start IDs from 1000 instead of 1
ALTER TABLE users AUTO_INCREMENT = 1000;
```

**Use case:** When you want IDs to start from a specific number (e.g., customer IDs starting from 1001 for a new product line).

### Behavior After Deleting Rows
If you delete the row with `id = 5` and then insert a new row, the new row gets `id = 6` — MySQL does **not reuse** deleted IDs. The counter only goes up.

---

## Key Takeaways — PRIMARY KEY & AUTO_INCREMENT

| Concept | Key Point |
|---------|-----------|
| PRIMARY KEY | Uniquely identifies each row; NOT NULL + UNIQUE enforced automatically |
| One per table | Each table can have only one PRIMARY KEY |
| AUTO_INCREMENT | Automatically assigns the next integer; only works with integer columns |
| No NULL | Primary Key column can never contain NULL values |
| No reuse | Deleted AUTO_INCREMENT values are never reused |
| Starting value | Change starting number with `ALTER TABLE users AUTO_INCREMENT = N` |

---

# 🎯 OVERALL KEY TAKEAWAYS — Day 8

| Topic | Most Important Point |
|-------|---------------------|
| **SELECT** | Use `WHERE` to filter, `ORDER BY` to sort, `LIMIT` to cap results |
| **WHERE Operators** | `=`, `!=`, `>`, `<`, `BETWEEN`, `IN`, `LIKE`, `IS NULL`, `AND`, `OR` |
| **UPDATE** | Always use `WHERE` — omitting it updates every row |
| **DELETE** | `DELETE` removes rows; `DROP TABLE` removes the entire table |
| **UNIQUE** | Prevents duplicates; allows NULL unlike PRIMARY KEY |
| **NOT NULL** | Ensures a column always has a value |
| **CHECK** | Custom validation rule on a column value |
| **DEFAULT** | Auto-fills a value when none is provided |
| **PRIMARY KEY** | One per table; uniquely identifies each row; never NULL |
| **AUTO_INCREMENT** | Auto-generates sequential IDs; counter never decreases |
| **Aggregate Functions** | COUNT, SUM, AVG, MIN, MAX — summarize many rows into one result |
| **GROUP BY** | Splits data into groups before aggregating |
| **String Functions** | LENGTH, LOWER, UPPER, CONCAT — manipulate text |
| **Date Functions** | NOW, YEAR, DATEDIFF, TIMESTAMPDIFF — work with dates |
| **Transactions** | Group of statements; COMMIT to save, ROLLBACK to undo |
| **AutoCommit** | On by default; turn off for manual transaction control |
| **SAVEPOINT** | Mark a mid-transaction restore point for partial rollback |

---

## ⚡ Pro Tips — SQL

1. **Always run SELECT before UPDATE/DELETE** — preview what will be affected before changing it
2. **Never omit WHERE in UPDATE or DELETE** without being 100% intentional
3. **Use `IS NULL` / `IS NOT NULL`** — never `= NULL` or `!= NULL` (they don't work)
4. **`LIKE '%text%'`** is powerful but slow on large tables — use it only when needed
5. **Name your CHECK constraints** — it makes them much easier to drop or modify later
6. **Use transactions** whenever multiple SQL statements must succeed or fail together
7. **AUTO_INCREMENT** never reuses deleted values — don't rely on it for sequence counting
8. **GROUP BY** must include all non-aggregated columns from SELECT
9. **CONCAT()** is great for building formatted output (names + emails, full addresses, etc.)
10. **TIMESTAMPDIFF(YEAR, dob, CURDATE())** is the standard way to calculate age in MySQL

---

## Practice Exercises — Day 8 SQL

**Querying:**
- [ ] Select all users where salary is between ₹50,000 and ₹80,000
- [ ] Find all users whose name contains the letter 'a'
- [ ] Get the top 3 highest-paid users sorted by salary descending
- [ ] Find all users where `date_of_birth` is NULL

**Updating:**
- [ ] Give a ₹5,000 raise to all users earning less than ₹60,000
- [ ] Update the email of the user with id = 2
- [ ] Try an UPDATE without WHERE and then use ROLLBACK to undo it

**Deleting:**
- [ ] Delete all users where salary is NULL
- [ ] Run a SELECT first, confirm the rows, then DELETE
- [ ] Practice: DELETE all rows vs DROP TABLE — observe the difference

**Constraints:**
- [ ] Create a table with UNIQUE, NOT NULL, CHECK, and DEFAULT constraints
- [ ] Try inserting a duplicate email and observe the error
- [ ] Add a CHECK constraint that salary must be greater than 0

**Functions:**
- [ ] Count total users, male users, and female users separately
- [ ] Find the average salary grouped by gender
- [ ] Calculate each user's age using TIMESTAMPDIFF
- [ ] Use CONCAT to display "Name (email)" for all users

**Transactions:**
- [ ] Turn off AutoCommit, run an UPDATE, then ROLLBACK — confirm the change was undone
- [ ] Run the same UPDATE and this time COMMIT — confirm it was saved
- [ ] Practice using SAVEPOINT with two operations and rolling back to the savepoint

---