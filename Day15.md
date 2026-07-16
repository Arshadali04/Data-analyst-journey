# Day 15: Reading Data from Sources & Python Recap

## 📋 Topics Covered

**Python — Data Sources:**
1. Reading Data from CSV Files
2. Reading Data from Excel Files
3. Reading Data from SQL Databases
4. Reading Data from APIs (JSON)
5. Reading Data from Web Scraping
6. Reading Other Formats (JSON, Parquet, XML, TSV)
7. Exporting / Writing Data
8. Choosing the Right Format

**Python — Recap:**
9. Variables & Data Types
10. Operators
11. Strings
12. Lists, Tuples, Sets & Dictionaries
13. Conditionals (if / elif / else)
14. Loops (for / while)
15. Functions
16. List Comprehensions
17. Error Handling (try / except)
18. File Handling

---

# PART 1: READING DATA FROM SOURCES 📂

---

## 1. Reading CSV Files 📄

CSV (Comma-Separated Values) is the most common data format in data analysis.

### Basic Read

```python
import pandas as pd

df = pd.read_csv('data.csv')
```

### With Full Options

```python
df = pd.read_csv(
    'data.csv',
    sep=',',                        # delimiter (use '\t' for TSV)
    header=0,                       # row index of column names (0 = first row)
    index_col='CustomerID',         # column to use as row index
    usecols=['Name', 'Sales', 'City'],  # load only these columns
    dtype={'Age': int, 'Salary': float},  # force dtypes on load
    nrows=500,                      # only load first 500 rows
    skiprows=[0, 1],                # skip specific rows by index
    skipfooter=2,                   # skip last N rows
    na_values=['N/A', 'none', '-', ''],  # treat these as NaN
    parse_dates=['Date'],           # auto-parse date columns
    encoding='utf-8',               # file encoding
    thousands=',',                  # treat commas as thousands separator
    engine='python'                 # needed with skipfooter
)
```

### Reading Large Files in Chunks

```python
# Process large CSV in chunks of 10,000 rows
chunk_size = 10000
chunks = []

for chunk in pd.read_csv('large_data.csv', chunksize=chunk_size):
    # Process each chunk (e.g., filter rows)
    chunk = chunk[chunk['Sales'] > 1000]
    chunks.append(chunk)

df = pd.concat(chunks, ignore_index=True)
print(df.shape)
```

> **When to use chunks:** When your CSV is larger than available RAM — processes it piece by piece instead of loading everything at once.

---

## 2. Reading Excel Files 📊

```python
# Basic read (reads first sheet by default)
df = pd.read_excel('data.xlsx')

# Specify sheet name or index
df = pd.read_excel('data.xlsx', sheet_name='Sales')
df = pd.read_excel('data.xlsx', sheet_name=0)   # first sheet

# Read all sheets at once → returns a dict of DataFrames
all_sheets = pd.read_excel('data.xlsx', sheet_name=None)
df_jan = all_sheets['January']
df_feb = all_sheets['February']

# With options
df = pd.read_excel(
    'data.xlsx',
    sheet_name='Sheet1',
    header=1,             # if column names are in the second row
    skiprows=3,           # skip first 3 rows
    usecols='A:E',        # only columns A through E
    nrows=100,
    na_values=['N/A', '-']
)
```

### Installing openpyxl (Required for .xlsx)

```bash
pip install openpyxl
```

### Reading Multiple Sheets and Combining

```python
all_sheets = pd.read_excel('sales.xlsx', sheet_name=None)

# Combine all sheets into one DataFrame
combined = pd.concat(all_sheets.values(), ignore_index=True)
```

---

## 3. Reading from SQL Databases 🗄️

### Using SQLite (Built into Python — No Server Needed)

```python
import sqlite3
import pandas as pd

# Create a connection
conn = sqlite3.connect('my_database.db')

# Read entire table
df = pd.read_sql('SELECT * FROM users', conn)

# Read with conditions
df = pd.read_sql('''
    SELECT name, salary, city
    FROM users
    WHERE salary > 50000
    ORDER BY salary DESC
''', conn)

# Write a DataFrame to SQL
df.to_sql('clean_users', conn, if_exists='replace', index=False)
# if_exists: 'fail', 'replace', 'append'

conn.close()
```

### Using MySQL with mysql-connector

```bash
pip install mysql-connector-python
```

```python
import mysql.connector
import pandas as pd

conn = mysql.connector.connect(
    host='localhost',
    user='root',
    password='yourpassword',
    database='startersql'
)

df = pd.read_sql('SELECT * FROM users', conn)
print(df.head())

conn.close()
```

### Using SQLAlchemy (Universal SQL Connector)

```bash
pip install sqlalchemy
```

```python
from sqlalchemy import create_engine
import pandas as pd

# MySQL
engine = create_engine('mysql+mysqlconnector://user:password@localhost/dbname')

# SQLite
engine = create_engine('sqlite:///my_database.db')

# PostgreSQL
engine = create_engine('postgresql://user:password@localhost/dbname')

# Read
df = pd.read_sql('SELECT * FROM users', engine)

# Write
df.to_sql('users_backup', engine, if_exists='replace', index=False)
```

---

## 4. Reading from APIs (JSON Data) 🌐

APIs (Application Programming Interfaces) return data — usually as JSON — that you can fetch over the internet.

### Basic API Request

```bash
pip install requests
```

```python
import requests
import pandas as pd

# GET request to an API endpoint
url = 'https://jsonplaceholder.typicode.com/users'
response = requests.get(url)

# Check if request was successful
print(response.status_code)   # 200 = OK, 404 = not found, 500 = server error

# Parse JSON response
data = response.json()
print(type(data))   # list or dict

# Convert to DataFrame
df = pd.DataFrame(data)
print(df.head())
```

### API with Parameters and Headers

```python
url = 'https://api.example.com/sales'

# Query parameters
params = {
    'start_date': '2024-01-01',
    'end_date':   '2024-12-31',
    'limit':      1000
}

# Authentication headers (API key)
headers = {
    'Authorization': 'Bearer YOUR_API_KEY',
    'Content-Type': 'application/json'
}

response = requests.get(url, params=params, headers=headers)

if response.status_code == 200:
    data = response.json()
    df = pd.DataFrame(data['results'])  # often nested under a key
else:
    print(f"Error: {response.status_code}")
```

### Handling Nested JSON

```python
import json

# If JSON is deeply nested
data = response.json()

# Flatten nested JSON
df = pd.json_normalize(data, sep='_')

# Or manually extract nested values
records = data['data']['users']  # navigate to the list
df = pd.DataFrame(records)
```

### Reading JSON from a File

```python
# Read JSON file directly
df = pd.read_json('data.json')

# For nested/complex JSON
with open('data.json', 'r') as f:
    data = json.load(f)

df = pd.DataFrame(data)
```

---

## 5. Web Scraping (HTML Tables) 🕷️

When data is in an HTML table on a webpage, Pandas can read it directly.

```bash
pip install lxml beautifulsoup4
```

```python
import pandas as pd

url = 'https://en.wikipedia.org/wiki/List_of_countries_by_GDP'

# Read ALL tables on the page
tables = pd.read_html(url)

# tables is a list of DataFrames — one per HTML table
print(f"Found {len(tables)} tables")

# Select the relevant one
df = tables[0]
print(df.head())
```

---

## 6. Other Data Formats 📦

### TSV (Tab-Separated Values)

```python
df = pd.read_csv('data.tsv', sep='\t')
```

### Parquet (Fast Columnar Format for Big Data)

```bash
pip install pyarrow
```

```python
# Read
df = pd.read_parquet('data.parquet')

# Write
df.to_parquet('data.parquet', index=False)
```

> **Why Parquet?** Much faster to read/write than CSV for large datasets. Stores column data types natively. Industry standard for big data pipelines (Spark, Hive, Databricks).

### XML

```python
df = pd.read_xml('data.xml')
```

### Google Sheets (via gspread)

```bash
pip install gspread gspread-dataframe google-auth
```

```python
import gspread
from gspread_dataframe import get_as_dataframe
from google.oauth2.service_account import Credentials

scopes = ['https://spreadsheets.google.com/feeds']
creds  = Credentials.from_service_account_file('credentials.json', scopes=scopes)
client = gspread.authorize(creds)

sheet = client.open('My Google Sheet').sheet1
df    = get_as_dataframe(sheet)
```

---

## 7. Exporting / Writing Data 💾

```python
# CSV
df.to_csv('output.csv', index=False)
df.to_csv('output.csv', index=False, encoding='utf-8-sig')  # with BOM for Excel

# Excel
df.to_excel('output.xlsx', index=False, sheet_name='Results')

# Multiple sheets in one Excel file
with pd.ExcelWriter('multi_sheet.xlsx') as writer:
    df1.to_excel(writer, sheet_name='Sales',     index=False)
    df2.to_excel(writer, sheet_name='Customers', index=False)
    df3.to_excel(writer, sheet_name='Summary',   index=False)

# JSON
df.to_json('output.json', orient='records', indent=2)

# SQL
df.to_sql('table_name', engine, if_exists='replace', index=False)

# Parquet
df.to_parquet('output.parquet', index=False)
```

---

## 8. Choosing the Right Format 🧭

| Format | Best For | Pros | Cons |
|--------|----------|------|------|
| **CSV** | General exchange, small–medium data | Universal, human-readable | Slow on large data, no types |
| **Excel** | Business reports, sharing with non-coders | Familiar format | Slow, size limits |
| **SQL** | Persistent storage, multi-user access | Structured, queryable | Requires a DB server |
| **JSON** | APIs, web data, nested structures | Flexible, widely used | Verbose, slow for large flat data |
| **Parquet** | Big data, data engineering pipelines | Fast, compressed, typed | Not human-readable |
| **API** | Real-time / live data | Always up to date | Requires auth, rate limits |

---

# PART 2: PYTHON RECAP 🐍

---

## 9. Variables & Data Types 📦

### Variable Assignment

```python
name    = "Arshad"    # str
age     = 22          # int
salary  = 75000.50    # float
is_active = True      # bool
nothing   = None      # NoneType
```

### Checking Type

```python
print(type(name))    # <class 'str'>
print(type(age))     # <class 'int'>
```

### Type Conversion

```python
int("42")        # → 42
float("3.14")    # → 3.14
str(100)         # → "100"
bool(0)          # → False
bool(1)          # → True
list("abc")      # → ['a', 'b', 'c']
```

---

## 10. Operators ➕

### Arithmetic Operators

```python
10 + 3   # 13  (addition)
10 - 3   # 7   (subtraction)
10 * 3   # 30  (multiplication)
10 / 3   # 3.333... (true division — always float)
10 // 3  # 3   (floor division — integer result)
10 % 3   # 1   (modulus — remainder)
10 ** 3  # 1000 (exponentiation)
```

### Comparison Operators

```python
5 == 5    # True
5 != 3    # True
5 > 3     # True
5 < 3     # False
5 >= 5    # True
5 <= 4    # False
```

### Logical Operators

```python
True and False   # False
True or False    # True
not True         # False
```

### Assignment Operators

```python
x = 10
x += 5    # x = 15
x -= 3    # x = 12
x *= 2    # x = 24
x /= 4    # x = 6.0
x //= 2   # x = 3.0
x **= 3   # x = 27.0
```

---

## 11. Strings 📝

```python
s = "Hello, World!"

# Length
len(s)                  # 13

# Indexing & Slicing
s[0]                    # 'H'
s[-1]                   # '!'
s[0:5]                  # 'Hello'
s[::-1]                 # '!dlroW ,olleH' (reversed)

# Case methods
s.upper()               # 'HELLO, WORLD!'
s.lower()               # 'hello, world!'
s.title()               # 'Hello, World!'

# Strip whitespace
"  hello  ".strip()     # 'hello'

# Split & Join
"a,b,c".split(',')      # ['a', 'b', 'c']
','.join(['a', 'b', 'c'])   # 'a,b,c'

# Replace
s.replace('World', 'Python')   # 'Hello, Python!'

# Check contents
s.startswith('Hello')  # True
s.endswith('!')        # True
'World' in s           # True

# f-strings (modern string formatting)
name = "Arshad"
age  = 22
print(f"My name is {name} and I am {age} years old.")
print(f"Next year I'll be {age + 1}.")
print(f"Salary: ₹{75000:.2f}")
```

---

## 12. Collections 🗄️

### List — Ordered, Mutable, Allows Duplicates

```python
fruits = ['apple', 'banana', 'cherry', 'apple']

# Access
fruits[0]          # 'apple'
fruits[-1]         # 'apple'

# Slicing
fruits[1:3]        # ['banana', 'cherry']

# Modify
fruits.append('mango')       # add to end
fruits.insert(1, 'grape')    # insert at position
fruits.remove('banana')      # remove by value
fruits.pop()                 # remove last
fruits.pop(0)                # remove at index

# Other
len(fruits)
fruits.sort()
fruits.reverse()
fruits.count('apple')
fruits.index('cherry')
sorted_fruits = sorted(fruits)   # returns new sorted list
```

### Tuple — Ordered, Immutable, Allows Duplicates

```python
coords = (28.7041, 77.1025)   # latitude, longitude

# Same indexing as list
coords[0]         # 28.7041

# Unpacking
lat, lon = coords
print(lat, lon)

# When to use: data that should NOT change (coordinates, RGB values, DB records)
```

### Set — Unordered, No Duplicates

```python
skills = {'Python', 'SQL', 'Excel', 'Python'}  # Python appears once

# Common operations
skills.add('Tableau')
skills.remove('Excel')
'SQL' in skills        # True

# Set operations
a = {1, 2, 3, 4}
b = {3, 4, 5, 6}

a | b     # union: {1, 2, 3, 4, 5, 6}
a & b     # intersection: {3, 4}
a - b     # difference: {1, 2}
a ^ b     # symmetric difference: {1, 2, 5, 6}
```

### Dictionary — Key-Value Pairs, Ordered (Python 3.7+)

```python
person = {
    'name': 'Arshad',
    'age': 22,
    'city': 'Bengaluru',
    'skills': ['Python', 'SQL']
}

# Access
person['name']              # 'Arshad'
person.get('email', 'N/A')  # 'N/A' (safe — no KeyError)

# Modify
person['email'] = 'arshad@email.com'   # add new key
person['age'] = 23                      # update existing

# Delete
del person['city']
person.pop('age')

# Iteration
for key in person:
    print(key, person[key])

for key, val in person.items():
    print(f"{key}: {val}")

# Useful methods
person.keys()     # dict_keys([...])
person.values()   # dict_values([...])
person.items()    # dict_items([...])
```

---

## 13. Conditionals — if / elif / else ⚖️

```python
score = 78

if score >= 90:
    grade = 'A+'
elif score >= 75:
    grade = 'A'
elif score >= 60:
    grade = 'B'
elif score >= 40:
    grade = 'C'
else:
    grade = 'Fail'

print(f"Grade: {grade}")   # Grade: A
```

### Ternary (One-Line if-else)

```python
status = "Pass" if score >= 40 else "Fail"

# Equivalent to:
if score >= 40:
    status = "Pass"
else:
    status = "Fail"
```

### Combining Conditions

```python
age    = 25
salary = 60000

if age >= 21 and salary >= 50000:
    print("Eligible for loan")

if city == 'Mumbai' or city == 'Delhi':
    print("Metro city")

if not is_blacklisted:
    print("Clear background")
```

---

## 14. Loops 🔁

### for Loop — Iterate Over a Sequence

```python
# Over a list
fruits = ['apple', 'banana', 'cherry']
for fruit in fruits:
    print(fruit)

# Over a range
for i in range(5):         # 0, 1, 2, 3, 4
    print(i)

for i in range(1, 11):     # 1 to 10
    print(i)

for i in range(0, 20, 2):  # 0, 2, 4, ... 18 (step 2)
    print(i)

# enumerate — get index AND value
for i, fruit in enumerate(fruits):
    print(f"{i}: {fruit}")

# zip — iterate two lists together
names   = ['Alice', 'Bob', 'Carol']
salaries = [50000, 60000, 70000]
for name, salary in zip(names, salaries):
    print(f"{name} earns ₹{salary}")

# Over a dictionary
for key, value in person.items():
    print(f"{key} → {value}")
```

### while Loop — Repeat Until Condition is False

```python
count = 0
while count < 5:
    print(f"Count: {count}")
    count += 1   # ← always update the variable or infinite loop!
```

### Loop Control Statements

```python
# break — exit the loop immediately
for i in range(10):
    if i == 5:
        break
    print(i)    # prints 0 to 4

# continue — skip rest of current iteration
for i in range(10):
    if i % 2 == 0:
        continue
    print(i)    # prints 1, 3, 5, 7, 9

# else on a loop (runs if loop completed without break)
for i in range(5):
    print(i)
else:
    print("Loop finished cleanly")
```

### Nested Loops

```python
# Multiplication table
for i in range(1, 4):
    for j in range(1, 4):
        print(f"{i} × {j} = {i*j}")
```

---

## 15. Functions 🔧

### Defining and Calling

```python
def greet(name):
    print(f"Hello, {name}!")

greet("Arshad")   # Hello, Arshad!
```

### Parameters and Return Values

```python
def add(a, b):
    return a + b

result = add(10, 20)
print(result)   # 30
```

### Default Parameters

```python
def greet(name, greeting="Hello"):
    return f"{greeting}, {name}!"

greet("Arshad")            # "Hello, Arshad!"
greet("Arshad", "Hi")      # "Hi, Arshad!"
```

### Keyword Arguments

```python
def user_info(name, age, city):
    print(f"{name}, {age}, {city}")

user_info(age=22, city="Bengaluru", name="Arshad")  # order doesn't matter
```

### *args — Variable Positional Arguments

```python
def total(*numbers):
    return sum(numbers)

total(10, 20, 30)       # 60
total(5, 10)            # 15
```

### **kwargs — Variable Keyword Arguments

```python
def profile(**info):
    for key, val in info.items():
        print(f"{key}: {val}")

profile(name="Arshad", age=22, city="Bengaluru")
```

### Returning Multiple Values

```python
def stats(numbers):
    return min(numbers), max(numbers), sum(numbers) / len(numbers)

low, high, avg = stats([5, 10, 15, 20])
print(low, high, avg)   # 5 20 12.5
```

### Lambda — Anonymous One-Line Function

```python
# Regular function
def square(x):
    return x ** 2

# Equivalent lambda
square = lambda x: x ** 2
square(5)   # 25

# Most common use: inside map, filter, sorted
numbers = [3, 1, 4, 1, 5, 9, 2]
sorted(numbers, key=lambda x: -x)    # sort descending
list(map(lambda x: x ** 2, numbers)) # square all
list(filter(lambda x: x > 3, numbers))  # keep only > 3
```

---

## 16. List Comprehensions ⚡

A concise way to build lists — faster and more readable than loops.

```python
# Standard for loop
squares = []
for x in range(10):
    squares.append(x ** 2)

# List comprehension — same result, one line
squares = [x ** 2 for x in range(10)]

# With a condition (filter)
even_squares = [x ** 2 for x in range(10) if x % 2 == 0]

# Nested list comprehension (flatten a matrix)
matrix = [[1, 2, 3], [4, 5, 6], [7, 8, 9]]
flat   = [val for row in matrix for val in row]
# [1, 2, 3, 4, 5, 6, 7, 8, 9]

# Dictionary comprehension
word_lengths = {word: len(word) for word in ['apple', 'banana', 'cherry']}
# {'apple': 5, 'banana': 6, 'cherry': 6}

# Set comprehension (unique values)
unique_lengths = {len(word) for word in ['apple', 'banana', 'cherry']}
# {5, 6}
```

---

## 17. Error Handling — try / except 🛡️

Prevents your program from crashing when something goes wrong.

```python
# Basic try-except
try:
    result = 10 / 0
except ZeroDivisionError:
    print("Cannot divide by zero!")

# Catch specific error types
try:
    age = int(input("Enter age: "))
except ValueError:
    print("Invalid input — enter a number")

# Catch multiple errors
try:
    file = open('data.csv')
    data = int(file.read())
except FileNotFoundError:
    print("File not found")
except ValueError:
    print("File content is not a number")

# else — runs if no exception occurred
try:
    result = 10 / 2
except ZeroDivisionError:
    print("Error!")
else:
    print(f"Result: {result}")   # runs only if no error

# finally — ALWAYS runs (cleanup)
try:
    conn = open_database()
    query_data()
except Exception as e:
    print(f"Error: {e}")
finally:
    conn.close()   # always close the connection

# Raise your own exception
def validate_age(age):
    if age < 0 or age > 120:
        raise ValueError(f"Invalid age: {age}")
    return age
```

### Common Exception Types

| Exception | When It Occurs |
|-----------|---------------|
| `ValueError` | Wrong value type (e.g., `int("abc")`) |
| `TypeError` | Wrong type in operation (e.g., `"a" + 1`) |
| `ZeroDivisionError` | Dividing by zero |
| `FileNotFoundError` | File doesn't exist |
| `KeyError` | Dictionary key doesn't exist |
| `IndexError` | List index out of range |
| `AttributeError` | Object doesn't have that attribute |
| `ImportError` | Module can't be imported |

---

## 18. File Handling 📁

```python
# Write a file
with open('output.txt', 'w') as f:
    f.write("Hello, World!\n")
    f.write("Second line\n")

# Append to a file
with open('output.txt', 'a') as f:
    f.write("Third line\n")

# Read entire file
with open('output.txt', 'r') as f:
    content = f.read()
    print(content)

# Read line by line
with open('output.txt', 'r') as f:
    for line in f:
        print(line.strip())

# Read all lines into a list
with open('output.txt', 'r') as f:
    lines = f.readlines()

# Read CSV manually (prefer pandas for data work)
import csv
with open('data.csv', 'r') as f:
    reader = csv.reader(f)
    for row in reader:
        print(row)
```

> **Always use `with open()`** — it automatically closes the file even if an error occurs. Never use `f = open()` and manually close it.

---

## 🎯 KEY TAKEAWAYS — Day 15

| Topic | Key Point |
|-------|-----------|
| **CSV chunking** | Use `chunksize` for files larger than available RAM |
| **pd.read_sql** | Directly query a database into a DataFrame |
| **requests.get()** | Fetch data from any API; parse with `.json()` |
| **pd.json_normalize()** | Flatten nested JSON into a flat DataFrame |
| **Parquet** | Fastest format for large datasets; industry standard in data engineering |
| **f-strings** | Best modern way to format strings: `f"Hello {name}"` |
| **`dict.get(key, default)`** | Safe dictionary lookup — never raises KeyError |
| **List comprehension** | Faster and cleaner than for-loop + append pattern |
| **Lambda** | Anonymous function for short one-liners; best inside `map`/`filter`/`sorted` |
| **try/except/finally** | Always handle exceptions; use `finally` for cleanup (closing files/connections) |
| **`with open()`** | Always use context manager for file handling — auto-closes on exit |

---

## ⚡ Pro Tips — Day 15

1. **Always check `response.status_code`** before parsing API data — a 200 means success; anything else needs handling
2. **Use `pd.json_normalize()`** for nested API responses — it's far easier than manually navigating nested dicts
3. **Parquet over CSV for anything > 100MB** — smaller file size, faster read/write, column types preserved
4. **f-strings over `.format()`** — cleaner, faster, and easier to read
5. **`dict.get(key, default)`** always — avoids KeyError crashes in production code
6. **List comprehensions are not always better** — for very complex logic, a regular for loop is more readable
7. **`enumerate()` over manual counters** — `for i, val in enumerate(lst)` is cleaner than `i = 0; i += 1`
8. **`*args` and `**kwargs`** are powerful for building flexible, reusable utility functions
9. **Log exceptions with `except Exception as e`** — always print or log `e` so you know what went wrong
10. **Chunk your API calls** — most APIs have rate limits; add `time.sleep(1)` between requests to avoid getting blocked

---

## Practice Exercises — Day 15

**Reading Data:**
- [ ] Read a CSV with specific columns only using `usecols`
- [ ] Load a large CSV in chunks of 5,000 rows and filter inside the loop
- [ ] Read an Excel file with multiple sheets and combine them into one DataFrame
- [ ] Connect to a SQLite database, create a table, insert data, and read it back with `pd.read_sql`
- [ ] Fetch data from a public API (e.g., `https://jsonplaceholder.typicode.com/posts`) and load into a DataFrame
- [ ] Flatten a nested JSON response using `pd.json_normalize()`
- [ ] Write a DataFrame to CSV, Excel, and JSON — compare file sizes

**Python Recap:**
- [ ] Write a function that takes a list of numbers and returns mean, median, and std
- [ ] Use `*args` to write a function that accepts any number of scores and returns the highest
- [ ] Rewrite a for-loop that builds a list using list comprehension
- [ ] Write a dictionary comprehension that maps names to their lengths
- [ ] Write a try/except block that handles FileNotFoundError and ValueError separately
- [ ] Read a text file line by line and count how many lines contain the word "error"
- [ ] Write a lambda to sort a list of dictionaries by a specific key
- [ ] Create a function with default parameter values and call it both with and without the default

---
