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

