# Day 9: SQL — Foreign Keys, Joins, UNION & UNION ALL, Self Join

## 📋 Topics Covered

**SQL (MySQL):**
1. Foreign Key
2. MySQL Joins (INNER JOIN, LEFT JOIN, RIGHT JOIN)
3. UNION & UNION ALL
4. Self Join

---

# PART 1: FOREIGN KEY 🔗

## What is a Foreign Key?

> *"A foreign key is a column that creates a link between two tables. It ensures that the value in one table must match a value in another table."*

A `FOREIGN KEY` is used to **maintain data integrity between related tables** — it guarantees that a value stored in one table actually exists in another.

---

## Why Use Foreign Keys?

**The Problem:**
You have a `users` table. Now you want to store each user's address. You could add address columns directly to the `users` table — but that creates a bloated, messy table.

**The Better Approach:**
Create a separate `addresses` table and **link it** to `users` using a foreign key. This keeps your data organized, avoids repetition, and ensures every address always belongs to a valid user.

**Parent Table (Referenced):** `users` — holds the original `id`
**Child Table (Referencing):** `addresses` — holds `user_id` which must match a value in `users.id`

---

## Creating a Table with a Foreign Key

```sql
CREATE TABLE addresses (
  id      INT AUTO_INCREMENT PRIMARY KEY,
  user_id INT,
  street  VARCHAR(255),
  city    VARCHAR(100),
  state   VARCHAR(100),
  pincode VARCHAR(10),
  FOREIGN KEY (user_id) REFERENCES users(id)
);
```

**Explanation:**
- `user_id` is the **foreign key column** in the child table (`addresses`)
- `REFERENCES users(id)` means `user_id` must match an existing `id` in the `users` table
- If you try to insert an address with a `user_id` that doesn't exist in `users`, MySQL will **throw an error** — this is the integrity protection at work

---

## Naming the Foreign Key Constraint

MySQL auto-generates a constraint name if you don't provide one. But naming it yourself makes dropping it much easier later:

```sql
CREATE TABLE addresses (
  id      INT AUTO_INCREMENT PRIMARY KEY,
  user_id INT,
  CONSTRAINT fk_user FOREIGN KEY (user_id) REFERENCES users(id)
);
```

Now the constraint is named `fk_user` — easy to reference when you need to drop or modify it.

---

## Adding a Foreign Key Later (ALTER TABLE)

If the table was created without a foreign key, you can add it anytime:

```sql
ALTER TABLE addresses
ADD CONSTRAINT fk_user FOREIGN KEY (user_id) REFERENCES users(id);
```

---

## Dropping a Foreign Key

```sql
ALTER TABLE addresses
DROP FOREIGN KEY fk_user;
```

> You must use the **constraint name** to drop it — not the column name. If you didn't name it yourself, use `SHOW CREATE TABLE addresses;` to find the auto-generated name.

---

## ON DELETE — Controlling Cascading Behavior

By default, if you try to delete a user who has related addresses, MySQL will **throw an error** (it protects the child rows). You can change this behavior with `ON DELETE`:

### ON DELETE CASCADE — Auto-Delete Child Rows

If the parent user is deleted, all their addresses are automatically deleted too:

```sql
CREATE TABLE addresses (
  id      INT AUTO_INCREMENT PRIMARY KEY,
  user_id INT,
  street  VARCHAR(255),
  city    VARCHAR(100),
  state   VARCHAR(100),
  pincode VARCHAR(10),
  CONSTRAINT fk_user FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE
);
```

Or add it later:
```sql
ALTER TABLE addresses
ADD CONSTRAINT fk_user FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE;
```

---

## ON DELETE Options — Full Reference

| ON DELETE Option | Behavior |
|-----------------|---------|
| `RESTRICT` | Prevents deletion of parent if child rows exist **(default)** |
| `CASCADE` | Automatically deletes all related child rows |
| `SET NULL` | Sets the foreign key column to NULL in the child table |

### Example — ON DELETE SET NULL

```sql
ALTER TABLE addresses
ADD CONSTRAINT fk_user FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE SET NULL;
```

When the parent user is deleted, `user_id` in all their addresses becomes `NULL` instead of the rows being deleted.

---

## Key Takeaways — Foreign Key

| Concept | Key Point |
|---------|-----------|
| Foreign Key | Links a column in one table to the PRIMARY KEY of another |
| Parent Table | The table being referenced (e.g., `users`) |
| Child Table | The table that holds the foreign key (e.g., `addresses`) |
| Default behavior | `RESTRICT` — blocks deletion of parent if children exist |
| `ON DELETE CASCADE` | Deletes child rows automatically when parent is deleted |
| `ON DELETE SET NULL` | Sets FK column to NULL when parent is deleted |
| Naming constraint | Always name your constraint — makes dropping it much easier |

---

# PART 2: SQL JOINS 🔀

## What are JOINs?

> *"JOINs are used to combine rows from two or more tables based on related columns — usually a foreign key in one table referencing a primary key in another."*

Instead of querying one table at a time and manually matching records, JOINs let you pull data from multiple tables **in a single query**.

---

## Sample Tables Used in Examples

**`users` table:**

| id | name  |
|----|-------|
| 1  | Aarav |
| 2  | Sneha |
| 3  | Raj   |

**`addresses` table:**

| id | user_id | city    |
|----|---------|---------|
| 1  | 1       | Mumbai  |
| 2  | 2       | Kolkata |
| 3  | 4       | Delhi   |

> **Key observation:**
> - User `Raj` (id = 3) has **no address** in the addresses table
> - Address `Delhi` has `user_id = 4` which **doesn't exist** in the users table
>
> These "mismatches" show clearly how each JOIN type handles missing matches.

---

## 1. INNER JOIN 🔵

### What it Returns
Only rows where there is a **matching record in BOTH tables**. Non-matching rows from either side are excluded.

```sql
SELECT users.name, addresses.city
FROM users
INNER JOIN addresses ON users.id = addresses.user_id;
```

### Output

| name  | city    |
|-------|---------|
| Aarav | Mumbai  |
| Sneha | Kolkata |

- **Raj** is excluded — no matching address
- **Delhi** is excluded — its `user_id = 4` doesn't exist in users

### Visual Representation
```
users          addresses
  1  ──────────  1 (Mumbai)   ✅ Match → included
  2  ──────────  2 (Kolkata)  ✅ Match → included
  3  ✗           -            ❌ No match → excluded
  -              4 (Delhi)    ❌ No match → excluded
```

> **Use INNER JOIN when:** You only want complete, verified records — rows that exist in both tables.

---

## 2. LEFT JOIN 🟡

### What it Returns
**All rows from the left table** (`users`), plus matching rows from the right table (`addresses`). If no match is found for a left row, the right side columns return `NULL`.

```sql
SELECT users.name, addresses.city
FROM users
LEFT JOIN addresses ON users.id = addresses.user_id;
```

### Output

| name  | city    |
|-------|---------|
| Aarav | Mumbai  |
| Sneha | Kolkata |
| Raj   | NULL    |

- **Raj** is included even though he has no address — city shows as `NULL`
- **Delhi** is still excluded — only left table drives the result

### Visual Representation
```
users          addresses
  1  ──────────  1 (Mumbai)   ✅ Match
  2  ──────────  2 (Kolkata)  ✅ Match
  3  ──  NULL    -            ✅ Included with NULL
  -              4 (Delhi)    ❌ Excluded (right side extra)
```

> **Use LEFT JOIN when:** You want all records from the main table regardless of whether they have a related record. Great for finding "users without addresses", "orders without payments", etc.

**Finding unmatched rows (LEFT JOIN trick):**
```sql
-- Find users who have NO address
SELECT users.name
FROM users
LEFT JOIN addresses ON users.id = addresses.user_id
WHERE addresses.user_id IS NULL;
```

---

## 3. RIGHT JOIN 🟠

### What it Returns
**All rows from the right table** (`addresses`), plus matching rows from the left table (`users`). If no match is found for a right row, the left side columns return `NULL`.

```sql
SELECT users.name, addresses.city
FROM users
RIGHT JOIN addresses ON users.id = addresses.user_id;
```

### Output

| name  | city    |
|-------|---------|
| Aarav | Mumbai  |
| Sneha | Kolkata |
| NULL  | Delhi   |

- **Delhi** is included even though `user_id = 4` doesn't exist in users — name shows as `NULL`
- **Raj** is excluded — he has no address, and the right table drives the result

### Visual Representation
```
users          addresses
  1  ──────────  1 (Mumbai)   ✅ Match
  2  ──────────  2 (Kolkata)  ✅ Match
  -   NULL ──    4 (Delhi)    ✅ Included with NULL
  3              -            ❌ Excluded (left side extra)
```

> **Use RIGHT JOIN when:** You want all records from the secondary/child table — even if the parent record is missing. Less commonly used than LEFT JOIN.

> **Tip:** A `RIGHT JOIN` of table A and B is equivalent to a `LEFT JOIN` of table B and A — you can always rewrite a RIGHT JOIN as a LEFT JOIN by swapping the table order.

---

## JOIN Types — Full Comparison

| JOIN Type | What It Returns |
|-----------|----------------|
| `INNER JOIN` | Only matching rows from **both** tables |
| `LEFT JOIN` | **All** rows from left table + matching from right (NULL if no match) |
| `RIGHT JOIN` | **All** rows from right table + matching from left (NULL if no match) |

---

## Using Aliases with JOINs (Cleaner Syntax)

Instead of typing the full table name each time, use aliases:

```sql
SELECT u.name, a.city
FROM users u
INNER JOIN addresses a ON u.id = a.user_id;
```

- `u` is an alias for `users`
- `a` is an alias for `addresses`
- Makes queries shorter and more readable, especially with multiple joins

---

## Joining Multiple Tables

You can chain multiple JOINs in one query:

```sql
SELECT u.name, a.city, o.order_date
FROM users u
INNER JOIN addresses a ON u.id = a.user_id
INNER JOIN orders o ON u.id = o.user_id;
```

---

# PART 3: UNION & UNION ALL 🔄

## What is UNION?

> *"The UNION operator is used to combine the result sets of two or more SELECT statements. It removes duplicates by default."*

Where JOINs combine **columns** (side by side), UNION combines **rows** (stacked on top of each other).

---

## Example Scenario

You have a `users` table for regular users and an `admin_users` table for administrators. You want a single combined list of everyone.

### Step 1: Create the admin_users Table
```sql
CREATE TABLE admin_users (
  id            INT PRIMARY KEY,
  name          VARCHAR(100),
  email         VARCHAR(100),
  gender        ENUM('Male', 'Female', 'Other'),
  date_of_birth DATE,
  salary        INT
);
```

### Step 2: Insert Sample Data
```sql
INSERT INTO admin_users (id, name, email, gender, date_of_birth, salary) VALUES
(101, 'Anil Kumar',   'anil@example.com',   'Male',   '1985-04-12', 60000),
(102, 'Pooja Sharma', 'pooja@example.com',  'Female', '1992-09-20', 58000),
(103, 'Rakesh Yadav', 'rakesh@example.com', 'Male',   '1989-11-05', 54000),
(104, 'Fatima Begum', 'fatima@example.com', 'Female', '1990-06-30', 62000);
```

---

## Basic UNION — Combine & Remove Duplicates

```sql
SELECT name FROM users
UNION
SELECT name FROM admin_users;
```

Returns a **single list of unique names** from both tables. If any name appears in both tables, it shows **only once**.

---

## UNION ALL — Combine & Keep Duplicates

```sql
SELECT name FROM users
UNION ALL
SELECT name FROM admin_users;
```

Returns **all names** including duplicates. If "Alice" exists in both tables, she appears **twice**.

---

## Selecting Multiple Columns

Both SELECT statements must return the **same number of columns** with compatible data types:

```sql
SELECT name, salary FROM users
UNION
SELECT name, salary FROM admin_users;
```

The column names in the final output are taken from the **first SELECT statement**.

---

## Adding a Role Label Column

You can add a literal value as a column to distinguish which table each row came from:

```sql
SELECT name, 'User' AS role FROM users
UNION
SELECT name, 'Admin' AS role FROM admin_users;
```

**Output:**

| name         | role  |
|--------------|-------|
| Alice        | User  |
| Bob          | User  |
| Anil Kumar   | Admin |
| Pooja Sharma | Admin |

This is very useful for reporting — you know at a glance which records are users and which are admins.

---

## ORDER BY with UNION

`ORDER BY` applies to the **entire combined result**, not individual SELECT statements. Place it at the very end:

```sql
SELECT name FROM users
UNION
SELECT name FROM admin_users
ORDER BY name;
```

> ⚠️ Do NOT put `ORDER BY` inside each individual SELECT — only at the end of the full UNION query.

---

## Rules of UNION

| Rule | Detail |
|------|--------|
| **Same column count** | Both SELECT statements must return the same number of columns |
| **Compatible data types** | Corresponding columns must have compatible types (e.g., both VARCHAR) |
| **UNION removes duplicates** | Duplicate rows are removed automatically |
| **UNION ALL keeps duplicates** | All rows are returned including exact duplicates |
| **Column names from first SELECT** | The output column headers come from the first SELECT statement |
| **ORDER BY at the end** | Only one ORDER BY at the very end — applies to the full result |

---

## When to Use UNION

- Combining **current and archived** data (e.g., active users + deleted users)
- Merging **similar tables** with the same structure (e.g., users + admin_users)
- **Cross-category reporting** (e.g., high-salary employees from two different departments)
- Replacing multiple OR conditions when data spans different tables

---

## UNION vs UNION ALL — Summary

| | UNION | UNION ALL |
|--|-------|-----------|
| **Duplicates** | Removed automatically | Kept |
| **Performance** | Slower (must check for duplicates) | Faster (no deduplication step) |
| **Use when** | You need a clean unique list | You need all records, or know there are no duplicates |

---

# PART 4: SELF JOIN 🔁

## What is a Self Join?

> *"A Self JOIN is a regular join, but the table is joined with itself."*

This is useful when rows **within the same table are related to each other**. The classic example: a referral system where users refer other users, and the referrer's ID is stored in the same `users` table.

---

## The Scenario — Referral System

We have a `users` table. We want to track who referred whom. Instead of creating a separate table, we add a `referred_by_id` column to the same `users` table — storing the `id` of the person who made the referral.

---

## Step 1: Add the referred_by_id Column

```sql
ALTER TABLE users
ADD COLUMN referred_by_id INT;
```

- Will be `NULL` for users who were not referred by anyone
- Will contain the `id` of the user who referred them

---

## Step 2: Insert Referral Data

```sql
-- User 1 referred Users 2 and 3
UPDATE users SET referred_by_id = 1 WHERE id IN (2, 3);

-- User 2 referred User 4
UPDATE users SET referred_by_id = 2 WHERE id = 4;
```

The `users` table now looks like this:

| id | name  | referred_by_id |
|----|-------|----------------|
| 1  | Aarav | NULL           |
| 2  | Sneha | 1              |
| 3  | Raj   | 1              |
| 4  | Fatima| 2              |

---

## Step 3: Self JOIN Query

We want to display each user's name alongside the name of the person who referred them.

```sql
SELECT
  a.id,
  a.name        AS user_name,
  b.name        AS referred_by
FROM users a
INNER JOIN users b ON a.referred_by_id = b.id;
```

**How it works:**
- `a` → alias for the **user being queried** (the one who was referred)
- `b` → alias for the **referrer** (the one who referred them)
- `ON a.referred_by_id = b.id` → matches the referrer's id in table `b` with the stored `referred_by_id` in table `a`
- Both `a` and `b` point to the **same physical table** — just viewed from two different perspectives

### Output (INNER JOIN — Excludes Non-Referred Users)

| id | user_name | referred_by |
|----|-----------|-------------|
| 2  | Sneha     | Aarav       |
| 3  | Raj       | Aarav       |
| 4  | Fatima    | Sneha       |

> Aarav (id = 1) is **not shown** because his `referred_by_id` is NULL — no match in the INNER JOIN.

---

## Using LEFT JOIN to Include All Users

To include users who weren't referred by anyone (showing NULL for their referrer):

```sql
SELECT
  a.id,
  a.name        AS user_name,
  b.name        AS referred_by
FROM users a
LEFT JOIN users b ON a.referred_by_id = b.id;
```

### Output (LEFT JOIN — Includes Everyone)

| id | user_name | referred_by |
|----|-----------|-------------|
| 1  | Aarav     | NULL        |
| 2  | Sneha     | Aarav       |
| 3  | Raj       | Aarav       |
| 4  | Fatima    | Sneha       |

> Aarav now appears with `NULL` in the `referred_by` column — meaning he wasn't referred by anyone.

---

## Key Rules for Self JOIN

1. **Always use aliases** — since you're joining a table with itself, aliases (`a` and `b`) are mandatory to distinguish the two "copies"
2. **Choose the right JOIN type:**
   - `INNER JOIN` → only shows rows where a referrer exists (excludes `NULL` referred_by_id)
   - `LEFT JOIN` → shows all rows, including those with no referrer
3. **Same table, different perspectives** — `a` and `b` are just two different views into the same data

---

## Other Real-World Self JOIN Use Cases

| Scenario | Self JOIN Logic |
|----------|----------------|
| Employee & Manager | Employee table has a `manager_id` column referencing another employee's `id` |
| Category & Parent Category | Category table has a `parent_category_id` referencing another category |
| Friend connections | Users table with a `friend_id` referencing another user |
| Reply threads | Comments table with a `parent_comment_id` referencing another comment |

---

## 🎯 OVERALL KEY TAKEAWAYS — Day 9

| Topic | Most Important Point |
|-------|---------------------|
| **Foreign Key** | Links a child table column to the parent table's PRIMARY KEY |
| **RESTRICT** | Default ON DELETE — blocks parent deletion if children exist |
| **CASCADE** | Deletes child rows automatically when parent is deleted |
| **SET NULL** | Sets FK column to NULL in child when parent is deleted |
| **Name your FK** | Always name constraints (`CONSTRAINT fk_name`) for easy management |
| **INNER JOIN** | Only matching rows from both tables — strictest join |
| **LEFT JOIN** | All rows from left + matches from right (NULL if no match) |
| **RIGHT JOIN** | All rows from right + matches from left (NULL if no match) |
| **UNION** | Stacks rows from two queries; removes duplicates |
| **UNION ALL** | Stacks rows from two queries; keeps all duplicates |
| **UNION rules** | Same number of columns, compatible types, ORDER BY at the end only |
| **Self JOIN** | Table joined with itself using two aliases; needs LEFT JOIN for NULLs |
| **Self JOIN aliases** | `a` and `b` are mandatory — both reference the same table |

---

## ⚡ Pro Tips — Day 9

1. **Always name your FK constraints** — auto-generated names are hard to remember when you need to drop them
2. **Check behavior before using CASCADE** — deleting a parent can silently wipe many child rows
3. **Use LEFT JOIN by default** over INNER JOIN — you rarely want to silently exclude unmatched records without knowing
4. **LEFT JOIN + WHERE IS NULL** — a powerful pattern to find "orphan" records (rows with no match)
5. **RIGHT JOIN can always be rewritten as LEFT JOIN** — just swap the table order; LEFT JOIN is more readable
6. **UNION is slower than UNION ALL** — if you know there are no duplicates, always use UNION ALL
7. **Add a role/label column in UNION** (`'Admin' AS role`) — makes combined results easy to identify in reports
8. **Self JOIN always needs aliases** — without `a` and `b`, MySQL can't tell which "copy" of the table you mean
9. **Use LEFT JOIN in Self JOINs** — INNER JOIN silently drops rows where the FK is NULL (e.g., the top-level user with no referrer)
10. **Self JOIN is a design signal** — if you find yourself needing it, your table has a hierarchical/recursive relationship built into it

---

## Practice Exercises — Day 9

**Foreign Keys:**
- [ ] Create a `users` table and an `addresses` table with a named foreign key constraint
- [ ] Try inserting an address with a `user_id` that doesn't exist — observe the error
- [ ] Add `ON DELETE CASCADE` and test: delete a user and confirm their addresses are also deleted
- [ ] Add `ON DELETE SET NULL` instead — delete a user and confirm `user_id` becomes NULL
- [ ] Drop the foreign key using the constraint name

**JOINs:**
- [ ] Write an INNER JOIN query to get user names with their cities
- [ ] Write a LEFT JOIN and identify which users have no address (city = NULL)
- [ ] Write a RIGHT JOIN and identify which addresses have no matching user
- [ ] Add a third table (e.g., `orders`) and write a query joining all three tables
- [ ] Rewrite a RIGHT JOIN as a LEFT JOIN (just swap the table order)

**UNION & UNION ALL:**
- [ ] Combine `users` and `admin_users` names using UNION — note duplicates are removed
- [ ] Do the same with UNION ALL — confirm duplicates now appear
- [ ] Add a `'User'` / `'Admin'` label column to the combined result
- [ ] Use ORDER BY at the end of a UNION query to sort the combined result

**Self Join:**
- [ ] Add `referred_by_id` column to users table
- [ ] Insert referral data for at least 4 users
- [ ] Write a Self JOIN with INNER JOIN — observe that the top-level user is excluded
- [ ] Rewrite it with LEFT JOIN — confirm the top-level user now shows with NULL referrer
- [ ] Try a Self JOIN for an employee-manager scenario

---