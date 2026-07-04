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

