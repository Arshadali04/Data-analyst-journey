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

