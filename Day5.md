# Day 5: Excel's 31 Most Used Formulas & SQL Fundamentals

## 📋 Topics Covered

**Excel:**
1. 31 Most Used Formulas

**SQL:**
1. What is a Database?
2. Creating a Table
3. Dropping the Database
4. Writing and Saving our SQL Script
5. Datatypes and Constraints in MySQL
6. Selecting Data From Table
7. Altering a Table
8. Inserting Data

---

# PART 1: EXCEL — 31 Most Used Formulas 🧮

## 1. FREQUENCY 📊

### Purpose
Counts how many times values occur within specified ranges (bins). An alternative to COUNTIF when you want a single array-based count of repeating values.

### How It Works
- Get unique values first using **Remove Duplicates** (Data → Remove Duplicates)
- Apply the formula:
```
=FREQUENCY(data_array, bins_array)
```
- `data_array` → the full original range (with duplicates)
- `bins_array` → the unique values range

### Critical Step: Array Formula
- Select the entire result range first
- Press **F2** (edit mode) on the first cell
- Press **Ctrl + Shift + Enter** to convert it into an **array formula**
- Without this step, the formula gives **wrong totals** because it doesn't know to treat the range as an array

### Example
If checking how many employees belong to each age group:
```
=FREQUENCY(C2:C21, F2:F10)
```
Then **Ctrl + Shift + Enter**.

### Key Takeaway
Always verify the total count matches your original data count — mismatched totals mean the array conversion step was skipped.

---

## 2. INDEX 🎯

### Purpose
Returns the value at a given row and column position within a specified range/array.

### Syntax
```
=INDEX(array, row_number, column_number)
```

### Example
```
=INDEX(A2:E9, 2, 2)
```
- `array` → A2:E9 (full data table)
- `row_number` → 2 (2nd row in the range)
- `column_number` → 2 (2nd column in the range)

### Key Notes
- Row and column numbers are **relative to the selected range**, not the worksheet
- Works well combined with MATCH for dynamic lookups (see below)

---

## 3. MATCH 🔍

### Purpose
Finds the **position** (row or column number) of a value within a range — does NOT return the value itself, just its location.

### Syntax
```
=MATCH(lookup_value, lookup_array, match_type)
```
- `match_type = 0` → Exact match (most commonly used)

### Example
```
=MATCH("9 am", A2:A9, 0)
```
Returns the position number where "9 am" is found in that range.

### Combining INDEX + MATCH (Dynamic Lookup)
This is the **most powerful combination** — replaces VLOOKUP's left-to-right limitation.

```
=INDEX(data_range, MATCH(row_lookup_value, row_range, 0), MATCH(column_lookup_value, column_range, 0))
```

**Why use INDEX + MATCH over VLOOKUP?**
- Works in **both directions** (left lookup too, not just right)
- Fully **dynamic** — change the row/column lookup value and result updates automatically
- More flexible for inserting/deleting columns without breaking the formula

---

## 4. LOOKUP 🔎

### Purpose
Looks up a value in one vector (row/column) and returns a corresponding value from another vector — works in **both directions** (left or right of the lookup column).

### Syntax
```
=LOOKUP(lookup_value, lookup_vector, result_vector)
```

### Example
```
=LOOKUP(name_cell, Name_Column_Range, Paid_Days_Column_Range)
```

### Key Notes
- `lookup_vector` and `result_vector` must be **selected ranges**, not just headers
- Unlike VLOOKUP, **LOOKUP can pull values from columns to the LEFT** of the lookup column too — no need to rearrange data
- Great for quick reverse lookups (e.g., finding a Serial Number from a Name, even when Serial Number is in a column to the left)

---

## 5. VLOOKUP ⬇️

### Purpose
The most popular lookup formula — searches for a value in the **first column** of a range and returns a value from a specified column to its right.

### Syntax
```
=VLOOKUP(lookup_value, table_array, col_index_number, range_lookup)
```
- `range_lookup = 0` → Exact Match
- `range_lookup = 1` (or omitted) → Approximate Match

### Best Practice: Naming the Range
- Select full data columns (e.g., click column header A, drag to E) so new rows added later are automatically included
- In the **Name Box** (top-left corner), type a name (e.g., `List1`) — **no spaces allowed**
- Press Enter to assign the name

### Example
```
=VLOOKUP(H1, List1, 2, 0)
```

### Making VLOOKUP Draggable (Dynamic Column Index)
Instead of manually changing the column index number each time:
```
=VLOOKUP($H$1, List1, ROW(), 0)
```
- `ROW()` returns the current row number, which auto-increments as you drag
- Fix the `lookup_value` cell reference using **F4** (absolute reference: `$H$1`) so it doesn't shift while dragging

### Key Notes
- VLOOKUP only looks **right** from the lookup column — cannot search left
- Use **Ctrl + '** (apostrophe) to quickly copy the formula from the cell above

---

## 6. HLOOKUP ➡️

### Purpose
Same as VLOOKUP but works **horizontally** — searches the first **row** of a range and returns a value from a specified row below it.

### Syntax
```
=HLOOKUP(lookup_value, table_array, row_index_number, range_lookup)
```

### Converting Vertical Data to Horizontal
- Copy the vertical data (Ctrl + A, Ctrl + C)
- Paste using **Paste Special → Transpose**
- Use **Alt + H, O, I** to auto-fit column widths after transposing

### Making HLOOKUP Dynamic (Draggable)
```
=HLOOKUP(H1, table_array, COLUMN()-2, 0)
```
- `COLUMN()` returns the current column number
- Subtract an offset (e.g., `-2`) to align the column number with the correct row index in your range
- Fix the lookup column with `$` (absolute reference) so the lookup value column doesn't shift when dragging horizontally, but allow the row reference to shift vertically

---

## 7. SUMIF ➕

### Purpose
Adds up values in a range that meet **one** specified condition.

### Syntax
```
=SUMIF(range, criteria, sum_range)
```

### Example
```
=SUMIF(B2:B10, "MIS Executive", C2:C10)
```
- `range` → column containing the criteria to check
- `criteria` → the condition (text or cell reference)
- `sum_range` → column containing the values to total

### Key Notes
- Fix the `range` and `sum_range` using **F4** before dragging down, so they don't shift
- Can be replaced by a **Pivot Table** for the same summarized result without writing formulas

---

## 8. SUMIFS ➕➕

### Purpose
Adds up values based on **multiple** conditions (criteria) at once.

### Syntax
```
=SUMIFS(sum_range, criteria_range1, criteria1, criteria_range2, criteria2, ...)
```

### Important Difference from SUMIF
- In SUMIFS, `sum_range` comes **first** (opposite order from SUMIF, where sum_range is last)

### Example
```
=SUMIFS($C$2:$C$13, $B$2:$B$13, $F2, $G$2:$G$13, G$1)
```

### Fixing References for Dragging Both Directions
| Reference | Fix Type | Reason |
|-----------|----------|--------|
| `sum_range` | Fully fixed (`$C$2:$C$13`) | Range should never shift |
| First criteria_range | Column fixed (`$F2`) | Row changes vertically, column stays fixed (since dragging right means columns shouldn't move) |
| Second criteria_range | Row fixed (`G$1`) | Column changes horizontally, row stays fixed |

### Key Notes
- Can take multiple criteria pairs — keep them to 2–3 for clarity
- Mixed absolute/relative references are essential for dragging formulas both vertically and horizontally correctly

---

## 9. COUNTIF 🔢

### Purpose
Counts the number of cells that meet **one** condition.

### Syntax
```
=COUNTIF(range, criteria)
```

### Example
```
=COUNTIF($F$2:$F$13, "Online")
```

### Key Notes
- Fix the `range` (F4) before dragging, since it shouldn't move
- Criteria can change naturally as you drag (since each criteria is typed separately in adjacent cells)

---

## 10. COUNTIFS 🔢🔢

### Purpose
Counts cells that meet **multiple** conditions simultaneously.

### Syntax
```
=COUNTIFS(criteria_range1, criteria1, criteria_range2, criteria2, ...)
```

### Example
```
=COUNTIFS($F$2:$F$13, $A2, $F$2:$F$13, B$1)
```

### Key Notes
- Same logic as SUMIFS for fixing references: fix what shouldn't move, leave dynamic what should change while dragging
- Watch out for exact text matches — e.g., "Debit Card" with a space must be typed exactly as it appears in source data

---

## 11 & 12. COUNTA and COUNTBLANK 🟢⬜

### COUNTA — Counts Non-Empty Cells
```
=COUNTA(range)
```
Counts **all cells with any value** (text, number, etc.) — useful for "Total Booked" type counts.

### COUNTBLANK — Counts Empty Cells
```
=COUNTBLANK(range)
```
Counts **only empty/blank cells** — useful for "Pending" type counts.

### Example Use Case
Seat booking tracker: COUNTA gives total seats booked; COUNTBLANK gives seats still pending.

### Bonus: Conditional Formatting
Use **Home → Conditional Formatting → Highlight Cell Rules → Equal To** to visually color-code values (e.g., "M" in blue, "F" in green).

---

## 13 & 14. AVERAGEIF and AVERAGEIFS ➗

### AVERAGEIF — Average with One Condition
```
=AVERAGEIF(range, criteria, average_range)
```

### Example
```
=AVERAGEIF(F2:F13, "Online", C2:C13)
```

### AVERAGEIFS — Average with Multiple Conditions
```
=AVERAGEIFS(average_range, criteria_range1, criteria1, criteria_range2, criteria2, ...)
```

### Example
```
=AVERAGEIFS($C$2:$C$13, $F$2:$F$13, $A2, $G$2:$G$13, B$1)
```

### Handling #DIV/0! Errors
When a criteria combination has **zero matching records**, AVERAGEIFS returns an error (division by zero). Wrap it with **IFERROR**:
```
=IFERROR(AVERAGEIFS(...), 0)
```
- The second argument is what to display if there's an error (0, blank `""`, or custom text)

---

## 15. SUBTOTAL ∑

### Two Ways to Use It

#### A. Via the Subtotal Feature (Data Tab)
1. **Sort** the data first (A→Z) by the column you want to group on
2. Select all data → **Data → Outline → Subtotal**
3. In the dialog box:
   - **At Each Change In** → choose the grouping column (e.g., Transaction Type)
   - **Use Function** → choose Sum/Average/Max/etc.
   - **Add Subtotal To** → choose the value column
4. Click OK — Excel auto-inserts subtotal rows
5. Remove anytime via **Subtotal → Remove All**

#### B. Via the SUBTOTAL Formula
```
=SUBTOTAL(function_number, range)
```

| Function Number | Operation |
|---|---|
| 1 | AVERAGE |
| 2 | COUNT |
| 9 | SUM |

### Example
```
=SUBTOTAL(9, C2:C13)
```

### Key Notes
- SUBTOTAL formula ignores other subtotal results in the range (avoids double-counting) — useful with filtered data

---

## 16. SUMPRODUCT ✖️➕

### Purpose
Multiplies corresponding elements in two (or more) arrays, then sums the results — all in **one formula**, without needing helper columns.

### Syntax
```
=SUMPRODUCT(array1, array2)
```

### Example
```
=SUMPRODUCT(C2:C10, D2:D10)
```
(e.g., Quantity × Rate, summed across all rows)

### Key Notes
- **Both arrays must be the exact same size** — mismatched ranges return a `#VALUE!` error
- Eliminates the need to create a helper "Total" column per row, then SUM it separately

---

## 17. IF ⚖️

### Purpose
A logical formula that returns one value if a condition is **TRUE** and another if **FALSE**.

### Syntax
```
=IF(logical_test, value_if_true, value_if_false)
```

### Example
```
=IF(C2>=20000, "Yes", "No")
```
(Gives a Diwali bonus flag to employees earning ₹20,000 or more)

### Key Notes
- Text results must be wrapped in **double quotes** (`"Yes"`)
- Numeric results don't need quotes
- Use IF only when there are **exactly two possible outcomes**

---

## 18. IF + AND 🔗

### Purpose
Combines IF with AND when **all conditions must be true simultaneously** for the result to be TRUE.

### Syntax
```
=IF(AND(condition1, condition2), value_if_true, value_if_false)
```

### Example
```
=IF(AND(C2="Manager", D2>=20000), "Yes", "No")
```
Bonus given only if the person is BOTH a Manager AND earns ≥ ₹20,000.

### Real-World Analogy
Applying for a driving license requiring **both** Aadhar Card **and** Electricity Bill — missing either one means the application fails.

---

## 19. IF + OR 🔀

### Purpose
Combines IF with OR when **at least one condition** needs to be true for the result to be TRUE.

### Syntax
```
=IF(OR(condition1, condition2), value_if_true, value_if_false)
```

### Example
```
=IF(OR(C2="Manager", D2>=20000), "Yes", "No")
```
Bonus given if the person is **either** a Manager **or** earns ≥ ₹20,000 (only one condition needs to match).

### Real-World Analogy
Driving license requiring **either** Aadhar Card **or** Electricity Bill as address proof — only one of the two is needed.

---

## 20. Nested IF (IF inside IF) 🪆

### Purpose
Used when there are **more than two possible outcomes**, by nesting multiple IF statements inside each other.

### Syntax
```
=IF(condition1, result1, IF(condition2, result2, IF(condition3, result3, default_result)))
```

### Example
```
=IF(C2="Manager", 5000, IF(C2="Supervisor", 3000, IF(C2="Clerk", 2500, 2000)))
```
- Manager → ₹5,000 bonus
- Supervisor → ₹3,000 bonus
- Clerk → ₹2,500 bonus
- Anyone else (P1) → ₹2,000 bonus (default/fallback)

### Important Rule
For **numeric range-based conditions** (e.g., grading by percentage), always check conditions in **descending order** — start from the highest threshold first:
```
=IF(D2>=75, "A+", IF(D2>=60, "A", IF(D2>=40, "B", "Fail")))
```

### Key Notes
- Microsoft documentation allows nesting up to many levels (historically 7, now up to ~64 in modern Excel)
- For very long nested conditions, **IFS** (available from Office 2019 onward) is a cleaner alternative

---

## 21–24. LEFT, RIGHT, MID, FIND ✂️

### LEFT — Extract from the Start
```
=LEFT(text, num_characters)
```
Example: `=LEFT(A2, 6)` → pulls the first 6 characters from the left.

### RIGHT — Extract from the End
```
=RIGHT(text, num_characters)
```
Example: `=RIGHT(A2, 6)` → pulls the last 6 characters from the right.

### MID — Extract from the Middle
```
=MID(text, start_number, num_characters)
```
Example: `=MID(A2, 4, 3)` → starts at the 4th character, extracts 3 characters.

### FIND — Locate a Character's Position
```
=FIND(find_text, within_text, [start_number])
```
Example: `=FIND(" ", A2, 1)` → finds the position of the first space in the text.

### Combining LEFT + FIND (Dynamic Extraction)
Useful when text has **inconsistent lengths** (e.g., city names of varying length before a hyphen/space):
```
=LEFT(A2, FIND(" ", A2, 1) - 1)
```
- `FIND` locates the space position
- Subtracting `1` excludes the space itself, returning only the text before it

### Alternative: Text to Columns
- **Data → Text to Columns → Delimited**
- Choose delimiter (e.g., hyphen "-")
- Click **Next → Finish**

### Key Notes
- These can also be combined as RIGHT + FIND or MID + FIND for splitting names into first/middle/last name components

---

## 25. REPEAT 🔁

### Purpose
Repeats a given text a specified number of times — useful for visually representing ratings (like star ratings).

### Syntax
```
=REPEAT(text, number_of_times)
```

### Example
```
=REPEAT("★", B2)
```
If `B2 = 3`, this displays `★★★`.

### Key Notes
- Whatever text/symbol you input is repeated automatically as the number changes
- Great for building simple visual rating systems without conditional formatting icons

---

## 26. SUBSTITUTE 🔄

### Purpose
Replaces occurrences of specific text within a string with new text — useful for cleaning messy/raw data.

### Syntax
```
=SUBSTITUTE(text, old_text, new_text, [instance_number])
```

### Example 1 — Removing Unwanted Characters
```
=SUBSTITUTE(A2, "-", "")
```
Removes all hyphens, replacing them with nothing (empty string).

### Example 2 — Replacing One Character with Another
```
=SUBSTITUTE(A2, ",", "-")
```
Replaces commas with hyphens.

### Key Notes
- `instance_number` is optional — if omitted, **all occurrences** are replaced; if specified (e.g., 2), only the 2nd occurrence is replaced
- Useful for data cleaning before further analysis

---

## 27–31. Financial Functions 💰

### 27. PMT — Calculate EMI (Equal Monthly Installment)

**Syntax:**
```
=PMT(rate, nper, pv, [fv], [type])
```

**Argument Meanings:**
| Argument | Meaning |
|----------|---------|
| `rate` | Interest rate (must divide annual rate by 12 for monthly) |
| `nper` | Number of periods (loan duration in months) |
| `pv` | Present Value (loan amount) — must be negative |
| `fv` | Future Value (optional, usually omitted) |
| `type` | 0 = End of period (default), 1 = Beginning of period |

**Example**
```
=PMT(10%/12, 10, -100000)
```
Calculates monthly EMI for a ₹1,00,000 loan at 10% annual interest over 10 months.

**Key Notes**
- `pv` must be entered as **negative** (or the result will show as negative) — Excel's sign convention treats loan disbursement and repayment as opposite cash flows
- `type = 1` (Beginning of Period) reduces total interest slightly, since the first payment happens immediately (saving 30 days of interest)

---

### 28. IPMT and PPMT — Splitting EMI into Interest & Principal

**IPMT — Interest portion of a specific installment**
```
=IPMT(rate, per, nper, pv)
```

**PPMT — Principal portion of a specific installment**
```
=PPMT(rate, per, nper, pv)
```

**Example**
```
=IPMT($rate/12, B2, $nper, $pv)
=PPMT($rate/12, B2, $nper, $pv)
```
- `per` (the period number) changes as you drag down (1st month, 2nd month, etc.)
- `rate`, `nper`, and `pv` must be **fixed** (absolute references) since they don't change

**Key Insight**
- Interest amount **decreases** each month while Principal amount **increases** each month
- This is because the outstanding loan balance reduces with each payment, lowering the interest charged on the remaining amount
- Sum of all IPMT values across the loan term = Total Interest Paid
- Sum of all PPMT values = Original Loan Amount

---

### 29. PV — Present Value (How Much Loan You Can Get)

**Purpose**
Reverse-calculates the loan amount you can borrow, given a fixed EMI you can afford.

**Syntax**
```
=PV(rate, nper, pmt)
```

**Example**
```
=PV(10%/12, 60, -15000)
```
If you can pay ₹15,000/month for 5 years (60 months) at 10% interest, this tells you the maximum loan amount you qualify for.

**Key Notes**
- `pmt` must be entered as negative
- Useful for reverse-engineering "how much can I borrow" based on affordable monthly payments

---

### 31. FV — Future Value (Savings Growth)

**Purpose**
Calculates how much a series of regular deposits will grow to, given an interest rate, over time.

**Syntax**
```
=FV(rate, nper, pmt)
```

**Example**
```
=FV(6%/12, 12, -5500)
```
If you deposit ₹5,500/month for 12 months at 6% annual interest, this calculates the total amount accumulated (principal + interest).

**Key Notes**
- `pmt` (the regular deposit) must be entered as negative
- Without interest: Total = monthly deposit × number of months
- With interest (via FV): Total is slightly higher, reflecting compound growth

---

## 🎯 KEY TAKEAWAYS — Excel Formulas

| Formula | Best For |
|---------|----------|
| FREQUENCY | Counting distribution across ranges (array formula) |
| INDEX + MATCH | Dynamic 2-way lookups, replacing VLOOKUP |
| LOOKUP | Lookups in either direction (left or right) |
| VLOOKUP / HLOOKUP | Standard vertical/horizontal lookups |
| SUMIF(S) / COUNTIF(S) / AVERAGEIF(S) | Conditional totals, counts, and averages |
| SUBTOTAL | Grouped summaries with sort + outline |
| SUMPRODUCT | Multiply-then-sum without helper columns |
| IF / IF+AND / IF+OR / Nested IF | Conditional logic with 2+ outcomes |
| LEFT / RIGHT / MID / FIND | Text extraction and parsing |
| REPEAT | Visual rating displays |
| SUBSTITUTE | Cleaning/replacing text in raw data |
| PMT / IPMT / PPMT / PV / FV | Loan EMI, interest/principal split, and savings calculations |

---

## ⚡ Pro Tips — Excel

1. Always use **Ctrl + Shift + Enter** for array formulas like FREQUENCY
2. Use **F4** to toggle absolute/relative references quickly while building draggable formulas
3. **INDEX + MATCH** is more powerful and flexible than VLOOKUP — learn it well
4. Wrap formulas prone to errors (like AVERAGEIFS) with **IFERROR**
5. For SUMIFS/COUNTIFS/AVERAGEIFS, decide carefully **which references to fix** based on whether you're dragging vertically, horizontally, or both
6. Use **ROW()** and **COLUMN()** functions to make VLOOKUP/HLOOKUP draggable dynamically
7. For nested IFs with numeric ranges, always order conditions from **highest to lowest**

---

## Practice Exercises — Excel

- [ ] Apply FREQUENCY to find age-group distribution and verify with Ctrl+Shift+Enter
- [ ] Build a dynamic INDEX + MATCH lookup table with dropdown selectors
- [ ] Practice VLOOKUP with named ranges and make it draggable using ROW()
- [ ] Build SUMIFS and COUNTIFS with 2 criteria each
- [ ] Use AVERAGEIFS with IFERROR to handle blank/zero-match scenarios
- [ ] Create a Nested IF formula for grading (A+/A/B/Fail)
- [ ] Extract city names from a "City-Pincode" combined column using LEFT + FIND
- [ ] Calculate EMI using PMT for a sample loan amount
- [ ] Split an EMI into interest and principal using IPMT and PPMT

---

# PART 2: SQL FUNDAMENTALS 🗄️

## 1. What is a Database? 📦

A **database** is a container that stores related data in an organized way. In MySQL, a database holds one or more **tables**.

### Analogies to Understand It

**Folder Analogy:**
- A database is like a **folder**
- Each table is like a **file** inside that folder
- The rows in the table are like the **content** inside each file

**Excel Analogy:**
- A database is like an **Excel workbook**
- Each table is like a **separate sheet** inside that workbook
- Each row in the table is like a **row in Excel**

### What is a DBMS?
A **Database Management System (DBMS)** is software that interacts with end users, applications, and the database itself to capture and analyze data. It handles creation, retrieval, updating, and management of data. Once you understand one DBMS, transitioning to another is easy since they share similar core concepts.

### What is MySQL Workbench?
A visual tool for database architects, developers, and DBAs — provides data modeling, SQL development, and administration tools (server config, user admin, backups, etc.)

---

## 2. Creating a Database & Table 🛠️

### Step 1: Create a Database
```sql
CREATE DATABASE startersql;
```

After creating it, either:
- Right-click it in MySQL Workbench → **Set as Default Schema**, OR
- Use this command:
```sql
USE startersql;
```

### Step 2: Create a Table
```sql
CREATE TABLE users (
  id INT AUTO_INCREMENT PRIMARY KEY,
  name VARCHAR(100) NOT NULL,
  email VARCHAR(100) UNIQUE NOT NULL,
  gender ENUM('Male', 'Female', 'Other'),
  date_of_birth DATE,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

This table stores basic user info — every column definition includes a **datatype** and optional **constraints**.

---

## 3. Dropping the Database ⚠️

```sql
DROP DATABASE startersql;
```

> **Warning:** This deletes the *entire database* and all tables/data inside it permanently. Be very careful before running this — always confirm you have backups if the data matters.

---

## 4. Writing and Saving Our SQL Script 📝

When working in MySQL Workbench:
- Write your SQL statements in the query editor (the script area)
- Use **Ctrl + S** to save your `.sql` script file for future reuse
- Scripts can be re-run anytime to recreate your database structure or re-insert data
- Keeping a saved script is good practice — it acts as a backup blueprint of your schema

---

## 5. Datatypes and Constraints in MySQL 🔑

### Common Data Types

| Data Type | Description |
|-----------|-------------|
| `INT` | Integer type, used for whole numbers |
| `VARCHAR(100)` | Variable-length string, up to 100 characters |
| `ENUM` | A string object with a value chosen from a permitted list, e.g. `gender ENUM('Male', 'Female', 'Other')` |
| `DATE` | Stores date values, e.g. `date_of_birth DATE` |
| `TIMESTAMP` | Stores date and time, auto-set to current timestamp when row is created |
| `BOOLEAN` | Stores TRUE or FALSE, often used for flags like `is_active` |
| `DECIMAL(10,2)` | Stores exact numeric values, useful for financial data — first number = total digits, second = digits after decimal point |

### Common Constraints

| Constraint | Purpose |
|-----------|---------|
| `AUTO_INCREMENT` | Automatically generates a unique number for each row |
| `PRIMARY KEY` | Uniquely identifies each row in the table |
| `NOT NULL` | Ensures a column cannot have a NULL value |
| `UNIQUE` | Ensures all values in a column are different |
| `DEFAULT` | Sets a default value if none is provided, e.g. `created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP`, `is_active BOOLEAN DEFAULT TRUE` |

### Combined Example
```sql
CREATE TABLE users (
  id INT AUTO_INCREMENT PRIMARY KEY,
  name VARCHAR(100) NOT NULL,
  email VARCHAR(100) UNIQUE NOT NULL,
  gender ENUM('Male', 'Female', 'Other'),
  date_of_birth DATE,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

---

## 6. Selecting Data From a Table 🔍

### Select All Columns
```sql
SELECT * FROM users;
```
Fetches every column and every row from the `users` table.

### Select Specific Columns
```sql
SELECT name, email FROM users;
```
Fetches only the `name` and `email` columns from all rows.

### Renaming a Table
```sql
RENAME TABLE users TO customers;
```
To rename it back:
```sql
RENAME TABLE customers TO users;
```

---

## 7. Altering a Table ✏️

`ALTER TABLE` is used to modify an existing table's structure.

### Add a Column
```sql
ALTER TABLE users ADD COLUMN is_active BOOLEAN DEFAULT TRUE;
```

### Drop a Column
```sql
ALTER TABLE users DROP COLUMN is_active;
```

### Modify a Column's Datatype
```sql
ALTER TABLE users MODIFY COLUMN name VARCHAR(150);
```

### Move a Column to the First Position
```sql
ALTER TABLE users MODIFY COLUMN email VARCHAR(100) FIRST;
```

### Move a Column After Another Column
```sql
ALTER TABLE users MODIFY COLUMN gender ENUM('Male', 'Female', 'Other') AFTER name;
```

---

## 8. Inserting Data into MySQL Tables ➕

To add data into a table, we use the **INSERT INTO** statement.

### Method A: Insert Without Specifying Column Names (Full Row Insert)
```sql
INSERT INTO users VALUES
(1, 'Alice', 'alice@example.com', 'Female', '1995-05-14', DEFAULT);
```
- Requires values for **all columns in order**, except columns with defaults or AUTO_INCREMENT
- ⚠️ **Not recommended** if your table structure might change later (e.g., new columns added)

### Method B: Insert by Specifying Column Names (Best Practice) ✅
```sql
INSERT INTO users (name, email, gender, date_of_birth) VALUES
('Bob', 'bob@example.com', 'Male', '1990-11-23');
```
- Safer and more readable — you only insert into the columns you specify
- The remaining columns (`id` with AUTO_INCREMENT, `created_at` with DEFAULT) are automatically handled by MySQL

### Inserting Multiple Rows at Once (More Efficient)
```sql
INSERT INTO users (name, email, gender, date_of_birth) VALUES
('Charlie', 'charlie@example.com', 'Other', '1988-02-17'),
('David', 'david@example.com', 'Male', '2000-08-09'),
('Eva', 'eva@example.com', 'Female', '1993-12-30');
```
This is faster and cleaner than inserting rows one at a time.

---

## 🎯 KEY TAKEAWAYS — SQL

| Concept | Purpose |
|---------|---------|
| Database | Organized container holding related tables |
| CREATE TABLE | Defines table structure with columns, datatypes, constraints |
| DROP DATABASE | Permanently deletes a database and all its data |
| Datatypes | Define what kind of data a column can hold (INT, VARCHAR, DATE, etc.) |
| Constraints | Enforce rules on data (PRIMARY KEY, NOT NULL, UNIQUE, DEFAULT) |
| SELECT | Retrieves data from a table (all or specific columns) |
| ALTER TABLE | Modifies an existing table's structure (add/drop/modify/reorder columns) |
| INSERT INTO | Adds new rows of data into a table |

---

## ⚡ Pro Tips — SQL

1. Always specify **column names explicitly** in INSERT statements — it's safer and more readable
2. Insert **multiple rows in a single statement** for better performance
3. Use `AUTO_INCREMENT` with `PRIMARY KEY` so you never have to manually track unique IDs
4. Save your SQL scripts regularly (Ctrl + S) — they're your reusable schema blueprint
5. Be cautious with `DROP DATABASE` — there's no undo
6. `NOT NULL` + `UNIQUE` together (like on `email`) ensures both presence and distinctness of values

---

## Practice Exercises — SQL

- [ ] Create a new database and switch to it using `USE`
- [ ] Create a `users` table with at least 5 columns and 3 different datatypes
- [ ] Add a `NOT NULL` and `UNIQUE` constraint on at least one column
- [ ] Insert 5 rows into the table using the "specify column names" method
- [ ] Run a `SELECT *` and a `SELECT` with specific columns
- [ ] Use `ALTER TABLE` to add a new column, then drop it
- [ ] Rename your table, then rename it back
- [ ] Practice inserting multiple rows in a single INSERT statement

---