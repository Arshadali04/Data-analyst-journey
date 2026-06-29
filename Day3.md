# Day 3

## Excel

### 1. Conditional formatting

Definition: Conditional formatting changes the cell style based on a rule.

Where it is:

1. Go to the Home tab.
2. Open the Styles group.
3. Click Conditional Formatting.

Main options:

1. Highlight Cells Rules
2. Top/Bottom Rules
3. Data Bars
4. Color Scales
5. Icon Sets
6. New Rule
7. Clear Rules
8. Manage Rules

Highlight Cells Rules:

- Greater Than
- Less Than
- Between
- Equal To
- Text That Contains
- Date Occurring
- Duplicate Values

Examples:

```text
Greater Than 60 -> highlight marks above 60
Less Than 35 -> highlight marks below 35
Equal To 35 -> highlight exact value 35
Between 25 and 50 -> highlight values in that range
Text That Contains Patil -> highlight names with Patil
Date Occurring -> highlight today, yesterday, this month, last month, next week, and similar date rules
Duplicate Values -> highlight repeated values
```

Top/Bottom Rules:

- Top 10 Items
- Top 10%
- Bottom 10 Items
- Bottom 10%
- Above Average
- Below Average

Examples:

```text
Top 5 marks -> highlight the highest 5 values
Above Average -> highlight values above the average
Below Average -> highlight values below the average
```

Data Bars:

- Shows a bar inside the cell
- Bigger values get a bigger bar
- Useful for quick visual comparison

Color Scales:

- Uses color shading to show high and low values
- High values can appear green
- Low values can appear red

Icon Sets:

- Uses icons like signals, arrows, or ratings
- Useful for reports and quick status checks

New Rule with formula:

1. Select the data range.
2. Go to Conditional Formatting.
3. Click New Rule.
4. Choose Use a formula to determine which cells to format.
5. Write the formula.
6. Choose the format.

Example:

```excel
=D2="Delhi"
```

This can highlight the whole row when the city is Delhi.

Important formula note:

- Use $ to fix a reference.
- Example: $D2 keeps column D fixed.
- D$2 keeps row 2 fixed.

Use Manage Rules to edit or remove rules later.

Use Clear Rules to remove formatting from selected cells or the whole sheet.

### 2. Sort data

Definition: Sort arranges data in ascending or descending order.

Where it is:

1. Data tab -> Sort & Filter group
2. Home tab -> Sort & Filter group

Shortcuts:

- Alt + A + S = open Sort
- A to Z = ascending sort
- Z to A = descending sort

Steps:

1. Select the column or table.
2. Go to Data or Home tab.
3. Click Sort A to Z, Z to A, or Custom Sort.
4. If only one column is selected, choose Expand the selection.
5. Keep My data has headers checked.

Important notes:

- If you select only one column, Excel asks whether to expand the selection.
- Expand the selection keeps the full row together.
- Continue with current selection sorts only that one column, which can break the data.
- Custom Sort is used for multiple levels.
- Sort Left to Right is used when you want to sort columns across a row.
- You can also sort by cell color or font color.

Examples:

```text
Sort Employee ID from 100 to 1 using Z to A.
Sort gross salary from highest to lowest.
Sort department first, then designation inside each department.
```

Custom sort example:

1. Sort by Department A to Z.
2. Add a second level.
3. Sort by Designation A to Z or Z to A.

Selection shortcuts used for sorting and filtering:

- Ctrl + Space = select column
- Shift + Space = select row
- Ctrl + Shift + Right Arrow = select data to the right
- Ctrl + Shift + Down Arrow = select data downward
- Ctrl + A = select all data
- Ctrl + Minus = delete selected row or column
- Ctrl + Plus = insert row or column
- Ctrl + 1 = open Format Cells
- F2 = edit the active cell

### 3. Filter data

Definition: Filter shows only the rows that match a condition.

Where it is:

1. Data tab -> Sort & Filter group
2. Home tab -> Sort & Filter group

Shortcut:

- Ctrl + Shift + L = toggle filter on and off
- Alt + A + T = apply filter

Steps:

1. Select the table.
2. Go to Data or Home tab.
3. Click Filter.
4. Use the drop-down arrow in each column.
5. Select the values you want.

Filter features:

- Search inside the filter drop-down
- Show only unique values in the list
- Clear filter from a column
- Remove all filters from the sheet
- Filter by color

Text Filters:

- Equals
- Does Not Equal
- Begins With
- Ends With
- Contains
- Does Not Contain

Number Filters:

- Greater Than
- Greater Than Or Equal To
- Less Than
- Less Than Or Equal To
- Between
- Top 10
- Above Average
- Below Average

Examples:

```text
Filter Department = Finance
Filter Designation Contains Senior
Filter Name Begins With A
Filter Salary Greater Than Or Equal To 500
Filter Top 5 salaries
```

Important note:

- When filter is applied, the filter icon changes.
- You can clear a single column filter or remove all filters.

Example:

Show only students with marks above 70.

Advanced Filter:

Definition: Advanced Filter can copy filtered data to another location.

Steps:

1. Keep the cursor inside the data.
2. Go to Data tab.
3. Click Advanced.
4. Choose Copy to another location.
5. Set List range.
6. Set Criteria range.
7. Set Copy to location.
8. Click OK.

Important notes:

- Use a blank sheet or a different output area for copying.
- You can filter by multiple conditions.
- Example: Gender = Male and Department = Finance.

Formula-based filter:

- SORT and FILTER formulas are available in newer Excel versions.
- If your version does not support them, you may need Excel 365 or the newer online version.

### 4. Sort and filter with formulas

Definition: Formula-based sorting and filtering help create results from conditions.

Useful formulas:

```excel
=SORT(range)
=SORTBY(array, by_array)
=FILTER(array, include)
```

Examples:

```excel
=SORT(A2:C10)
```

```excel
=FILTER(A2:C10, C2:C10>70)
```

Notes:

- SORT is used to sort a range.
- SORTBY is used when you want to sort by another array or column.
- FILTER is used to return only matching rows.

## Python

### 1. Arrays

Definition: In data analysis, arrays are used to store multiple values in one object.

Syntax:

```python
import numpy as np
arr = np.array([1, 2, 3])
```

Different ways to declare arrays:

```python
numbers = [1, 2, 3]
```

```python
numbers = (1, 2, 3)
```

```python
from array import array
numbers = array('i', [1, 2, 3])
```

```python
import numpy as np
numbers = np.array([1, 2, 3])
```

```python
import pandas as pd
numbers = pd.Series([1, 2, 3])
```

Example:

```python
import numpy as np
marks = np.array([80, 85, 90])
```

### 2. Conditions

Definition: Conditions help the code make decisions.

Syntax:

```python
if condition:
	statement
elif condition:
	statement
else:
	statement
```

Examples:

```python
if age >= 18:
	print("Adult")
```

```python
if marks >= 50:
	print("Pass")
else:
	print("Fail")
```

```python
if score >= 90:
	print("A")
elif score >= 75:
	print("B")
else:
	print("C")
```

### 3. While loop

Definition: A while loop repeats code while a condition is true.

Syntax:

```python
while condition:
	statement
```

Example:

```python
i = 1
while i <= 5:
	print(i)
	i += 1
```

### 4. For loop

Definition: A for loop repeats code for each item in a sequence.

Syntax:

```python
for item in sequence:
	statement
```

Example:

```python
fruits = ["apple", "banana", "mango"]
for fruit in fruits:
	print(fruit)
```

### 5. Python functions

Definition: A function is a block of code that runs when called. Functions help organize code, make it reusable, and easier to maintain.

---

#### **Type 1: Basic Function (No Arguments, No Return)**

**Explanation:** Simplest function that does something without needing input or sending output. Used for repetitive tasks like printing messages.

**Syntax:**

```python
def function_name():
	statement
```

**Example:**

```python
def greet():
	print("Hello")

greet()  # Output: Hello
```

**When to use:** For simple tasks like printing, displaying messages, or performing actions that don't depend on different values.

---

#### **Type 2: Function with Arguments**

**Explanation:** Function that accepts parameters (values) to work with. These parameters are used inside the function to perform calculations or operations.

**Syntax:**

```python
def add(a, b):
	return a + b
```

**Example:**

```python
result = add(2, 3)  # Output: 5
result = add(10, 20)  # Output: 30
```

**When to use:** When you want to pass different values each time the function is called. Essential for data processing.

---

#### **Type 3: Function with Return Value**

**Explanation:** Function that computes something and sends the result back to the calling code using `return`. Allows you to store results in variables.

**Syntax:**

```python
def square(x):
	return x * x
```

**Example:**

```python
area = square(5)  # area = 25
print(area)  # Output: 25
```

**When to use:** When you need to use the result in further calculations or store it for later use.

---

#### **Type 4: Default Arguments**

**Explanation:** Function parameters with default values. If caller doesn't provide an argument, the default value is used.

**Syntax:**

```python
def greet(name="Arshad"):
	print("Hello", name)
```

**Example:**

```python
greet()  # Output: Hello Arshad (uses default)
greet("Ali")  # Output: Hello Ali (overrides default)
```

**When to use:** When you have parameters that usually have the same value, but might change sometimes.

---

#### **Type 5: Keyword Arguments**

**Explanation:** Passing arguments by specifying their names instead of just positions. Makes code more readable and flexible.

**Syntax:**

```python
def greet(name, age):
	print(f"{name} is {age} years old")

greet(name="Ali", age=25)  # Order doesn't matter
greet(age=25, name="Ali")  # Same result
```

**Example:**

```python
def show_student(name, marks, grade):
	print(f"{name}: {marks} ({grade})")

show_student(grade="A", marks=90, name="Priya")
# Output: Priya: 90 (A)
```

**When to use:** When functions have many parameters or you want to make function calls clear and understandable.

---

#### **Type 6: Arbitrary Positional Arguments (*args)**

**Explanation:** Accept any number of positional arguments. Stored as a tuple. Useful when you don't know how many values will be passed.

**Syntax:**

```python
def total(*numbers):
	return sum(numbers)
```

**Example:**

```python
total(1, 2, 3)  # Output: 6
total(10, 20, 30, 40, 50)  # Output: 150
```

**When to use:** When you want a function to accept flexible number of values (like calculating sum, average, maximum).

---

#### **Type 7: Arbitrary Keyword Arguments (**kwargs)**

**Explanation:** Accept any number of keyword arguments. Stored as a dictionary. Useful for flexible, named parameters.

**Syntax:**

```python
def show(**data):
	print(data)
```

**Example:**

```python
show(name="Ali", age=25, city="Mumbai")
# Output: {'name': 'Ali', 'age': 25, 'city': 'Mumbai'}

def display_info(**info):
	for key, value in info.items():
		print(f"{key}: {value}")

display_info(name="Priya", salary=50000, department="Sales")
# Output: name: Priya
#         salary: 50000
#         department: Sales
```

**When to use:** When you need to handle flexible key-value pairs (like configuration settings, user data).

---

#### **Type 8: Lambda Function (Anonymous Function)**

**Explanation:** Small anonymous function written in one line. Used for simple operations without defining a full function.

**Syntax:**

```python
square = lambda x: x * x
```

**Example:**

```python
# Simple calculation
square = lambda x: x * x
print(square(5))  # Output: 25

# With multiple parameters
add = lambda x, y: x + y
print(add(3, 7))  # Output: 10

# Used with map()
numbers = [1, 2, 3, 4, 5]
squared = list(map(lambda x: x * x, numbers))
print(squared)  # Output: [1, 4, 9, 16, 25]

# Used with filter()
marks = [45, 78, 92, 55, 88]
passed = list(filter(lambda x: x >= 50, marks))
print(passed)  # Output: [78, 92, 55, 88]
```

**When to use:** For simple, one-time operations. Very common with `map()`, `filter()`, and `sorted()`.

---

#### **Type 9: Nested Function**

**Explanation:** Function defined inside another function. Inner function has access to outer function's variables.

**Syntax:**

```python
def outer():
	def inner():
		print("Inside inner function")
	inner()

outer()  # Output: Inside inner function
```

**Example:**

```python
def calculate(base):
	# Inner functions can use outer's variables
	def add_tax(amount):
		return amount + (amount * 0.18)  # 18% tax
	
	def apply_discount(amount):
		return amount * 0.9  # 10% discount
	
	taxed = add_tax(base)
	discounted = apply_discount(taxed)
	return discounted

print(calculate(1000))  # Output: 882.0
```

**When to use:** When you need helper functions that are only used by one parent function.

---

#### **Type 10: Recursion**

**Explanation:** Function that calls itself repeatedly until a base condition is met. Used for problems that have repetitive structure.

**Syntax:**

```python
def factorial(n):
	if n == 1:  # Base case: stop condition
		return 1
	return n * factorial(n - 1)  # Recursive call
```

**Example:**

```python
# Factorial example
def factorial(n):
	if n == 1:
		return 1
	return n * factorial(n - 1)

print(factorial(5))  # Output: 120 (5 × 4 × 3 × 2 × 1)

# Sum of numbers
def sum_numbers(n):
	if n == 0:
		return 0
	return n + sum_numbers(n - 1)

print(sum_numbers(5))  # Output: 15 (5+4+3+2+1)

# Fibonacci sequence
def fibonacci(n):
	if n <= 1:
		return n
	return fibonacci(n - 1) + fibonacci(n - 2)

print(fibonacci(6))  # Output: 8
```

**When to use:** For problems like factorial, tree traversal, and sequences. Be careful of infinite loops!

---

#### **Type 11: Variable Scope**

**Explanation:** Where a variable can be used. Global (entire code) or Local (only within function).

**Syntax:**

```python
name = "Arshad"  # Global variable

def show_name():
	print(name)  # Can access global variable

show_name()  # Output: Arshad
```

**Example:**

```python
# Global variable
city = "Mumbai"

def show_info():
	age = 25  # Local variable - only in function
	print(f"Age: {age}, City: {city}")

show_info()  # Output: Age: 25, City: Mumbai
print(age)  # Error: age is not defined globally

# Modifying global variable
counter = 0

def increment():
	global counter  # Tell Python to use global counter
	counter += 1
	print(counter)

increment()  # Output: 1
increment()  # Output: 2
```

**When to use:** Understand variable scope to avoid conflicts and unintended side effects.

---

#### **Best Practices:**

1. **Function names** should be simple, descriptive, and lowercase with underscores
   - Good: `calculate_total()`, `get_student_marks()`
   - Bad: `calc()`, `f()`, `function1()`

2. **Keep functions small** - do one thing well

3. **Use docstrings** to explain what function does:
   ```python
   def add(a, b):
       """Add two numbers and return the result"""
       return a + b
   ```

4. **Use meaningful parameter names** - helps understand what to pass

5. **Return early** - exit function as soon as you have result

### 6. Common data analysis syntax

Importing libraries:

```python
import pandas as pd
import numpy as np
```

Creating a DataFrame:

```python
df = pd.DataFrame({"name": ["A", "B"], "marks": [80, 90]})
```

Selecting columns:

```python
df["marks"]
```

Filtering rows:

```python
df[df["marks"] > 80]
```

Sorting values:

```python
df.sort_values(by="marks")
```

Basic aggregation:

```python
df["marks"].mean()
```