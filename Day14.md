# Day 14: Data Cleaning — Missing Values, Duplicates, Invalid Entries & Outliers

## 📋 Topics Covered

**Python — Data Cleaning:**
1. What is Data Cleaning & Why It Matters
2. The Data Cleaning Pipeline
3. Handling Missing Values
4. Handling Duplicates
5. Handling Invalid Entries & Data Type Issues
6. Standardizing Text Data
7. Handling Outliers
8. Fixing Structural Errors
9. Data Validation After Cleaning
10. Full End-to-End Cleaning Workflow

---

# PART 1: WHAT IS DATA CLEANING & WHY IT MATTERS 🧹

## Definition

> *"Data cleaning (also called data wrangling or data munging) is the process of detecting and correcting (or removing) corrupt, inaccurate, incomplete, or irrelevant records from a dataset — and transforming raw data into a format suitable for analysis."*

## The Golden Rule of Data Analysis

> **"Garbage In, Garbage Out."**

No matter how sophisticated your analysis or model is, if the input data is dirty, the output will be wrong. Data cleaning is the most important step in any data analysis workflow.

## Real-World Data is Always Dirty

| Problem | Example |
|---------|---------|
| Missing values | Customer age is blank |
| Duplicates | Same transaction recorded twice |
| Invalid entries | Age = -5, Phone = "abcdefg" |
| Wrong data types | Date stored as text: "15-Jan-2024" |
| Inconsistent text | "mumbai", "Mumbai", "MUMBAI" for the same city |
| Outliers | Salary = ₹9,999,999 in a dataset of junior employees |
| Structural errors | Extra spaces, special characters in column names |

## How Much Time Does Data Cleaning Take?

Industry surveys consistently show that **data scientists spend 60–80% of their time** on data cleaning — not on building models. Mastering it is not optional.

---

# PART 2: THE DATA CLEANING PIPELINE 🔄

A structured order for cleaning any dataset:

```
1. Load & Inspect
      ↓
2. Fix Column Names & Data Types
      ↓
3. Handle Missing Values
      ↓
4. Remove Duplicates
      ↓
5. Fix Invalid Entries
      ↓
6. Standardize Text
      ↓
7. Handle Outliers
      ↓
8. Validate & Export
```

---

# PART 3: HANDLING MISSING VALUES 🕳️

## Step 1: Detect Missing Values

```python
import pandas as pd
import numpy as np

df = pd.read_csv('data.csv')

# Count NaN per column
df.isnull().sum()

# Percentage of missing values per column
(df.isnull().sum() / len(df) * 100).round(2)

# Heatmap of missing values (visual)
import seaborn as sns
import matplotlib.pyplot as plt
sns.heatmap(df.isnull(), cbar=False, cmap='viridis')
plt.title('Missing Value Heatmap')
plt.show()

# Rows with ANY missing value
df[df.isnull().any(axis=1)]

# Total missing values in entire dataset
df.isnull().sum().sum()
```

---

## Step 2: Understand WHY Values Are Missing

Before deciding what to do, understand the pattern:

| Type | Description | Example | Action |
|------|-------------|---------|--------|
| **MCAR** (Missing Completely At Random) | No pattern — random gaps | Survey response randomly skipped | Safe to drop or impute |
| **MAR** (Missing At Random) | Missing depends on other columns | Income missing more for younger people | Impute carefully |
| **MNAR** (Missing Not At Random) | Missing because of the value itself | High earners skip salary field | Investigate before removing |

---

## Step 3: Strategies for Handling Missing Values

### Strategy 1: Drop Missing Values

```python
# Drop rows where ANY column is NaN
df.dropna(inplace=True)

# Drop rows where a SPECIFIC column is NaN
df.dropna(subset=['Age'], inplace=True)

# Drop columns with > 40% missing values
threshold = 0.4
df.dropna(thresh=int(len(df) * (1 - threshold)), axis=1, inplace=True)

# Drop rows where ALL values are NaN
df.dropna(how='all', inplace=True)
```

> **When to drop:** Use when missing data is < 5% of total rows, or when the column has > 50% missing and isn't critical.

### Strategy 2: Fill with a Fixed Value

```python
df['City'].fillna('Unknown', inplace=True)
df['Sales'].fillna(0, inplace=True)
```

### Strategy 3: Fill with Statistical Measures (Best for Numerics)

```python
# Mean — use when distribution is roughly symmetric
df['Age'].fillna(df['Age'].mean(), inplace=True)

# Median — use when data has outliers (skewed distribution)
df['Salary'].fillna(df['Salary'].median(), inplace=True)

# Mode — use for categorical columns
df['City'].fillna(df['City'].mode()[0], inplace=True)
```

> **Mean vs Median:**
> - Data is symmetric (bell-shaped) → fill with **mean**
> - Data has outliers (skewed) → fill with **median** (more robust)

### Strategy 4: Forward Fill / Backward Fill (Time Series)

```python
# Forward fill — propagate last valid value forward
df['Price'].fillna(method='ffill', inplace=True)

# Backward fill — propagate next valid value backward
df['Price'].fillna(method='bfill', inplace=True)

# Fill only N consecutive NaN values
df['Price'].fillna(method='ffill', limit=2, inplace=True)
```

> **When to use:** For time-series or ordered data where nearby values are good approximations.

### Strategy 5: Fill Per Group (Group-wise Imputation)

```python
# Fill Age with the mean age of the same Gender group
df['Age'] = df.groupby('Gender')['Age'].transform(
    lambda x: x.fillna(x.mean())
)
```

This is more accurate than filling with the overall mean — a 25-year-old and a 60-year-old shouldn't share the same average.

### Strategy 6: Interpolation (For Ordered Numeric Data)

```python
# Linear interpolation between values
df['Temperature'].interpolate(method='linear', inplace=True)

# Polynomial interpolation
df['Temperature'].interpolate(method='polynomial', order=2, inplace=True)
```

---

## Missing Value Decision Tree

```
Is the column critical for analysis?
    ├── No → Drop the column if > 50% missing
    └── Yes → How much is missing?
              ├── < 5%  → Drop those rows
              ├── 5–30% → Impute (mean/median/mode)
              └── > 30% → Investigate; consider dropping column
                          or use advanced imputation
```

---

# PART 4: HANDLING DUPLICATES 👥

## Step 1: Detect Duplicates

```python
# Count total duplicate rows
df.duplicated().sum()

# View the duplicate rows
df[df.duplicated()]

# Show ALL copies (including first occurrence)
df[df.duplicated(keep=False)]

# Check duplicates on specific columns only
df.duplicated(subset=['Name', 'Email']).sum()
```

## Step 2: Understand Types of Duplicates

| Type | Description | Example |
|------|-------------|---------|
| **Exact duplicate** | Every column is identical | Same transaction recorded twice |
| **Partial duplicate** | Key columns match, others differ | Same customer, different timestamps |
| **Logical duplicate** | Same entity, different formats | "Alice Smith" and "alice smith" |

## Step 3: Remove Duplicates

```python
# Remove exact duplicates (keep first occurrence)
df.drop_duplicates(inplace=True)

# Keep last occurrence instead
df.drop_duplicates(keep='last', inplace=True)

# Remove based on specific columns
df.drop_duplicates(subset=['Email'], inplace=True)

# Remove based on multiple key columns
df.drop_duplicates(subset=['Name', 'Phone', 'Date'], inplace=True)
```

## Step 4: Verify

```python
print("Duplicates remaining:", df.duplicated().sum())
# Should print 0
```

---

# PART 5: HANDLING INVALID ENTRIES & DATA TYPE ISSUES ❌

## Step 1: Inspect Data Types

```python
df.dtypes
df.info()
```

Common issues:
- Date columns stored as `object` (string)
- Numeric columns stored as `object` because of non-numeric characters ("₹5,000", "N/A")
- Boolean columns stored as `int` or `str`

## Step 2: Fix Data Types

### Converting to Numeric

```python
# Safe conversion — coerce errors to NaN instead of crashing
df['Age'] = pd.to_numeric(df['Age'], errors='coerce')
df['Salary'] = pd.to_numeric(df['Salary'], errors='coerce')

# Standard conversion (crashes if non-numeric values exist)
df['Sales'] = df['Sales'].astype(float)
df['ID'] = df['ID'].astype(int)
```

### Converting to DateTime

```python
# Auto-detect format
df['Date'] = pd.to_datetime(df['Date'], errors='coerce')

# Specify format explicitly (faster and more reliable)
df['Date'] = pd.to_datetime(df['Date'], format='%d-%m-%Y')
df['Date'] = pd.to_datetime(df['Date'], format='%Y/%m/%d')

# Extract date components after conversion
df['Year']  = df['Date'].dt.year
df['Month'] = df['Date'].dt.month
df['Day']   = df['Date'].dt.day
df['Weekday'] = df['Date'].dt.day_name()
```

### Removing Non-Numeric Characters Before Conversion

```python
# Remove currency symbols, commas, spaces
df['Salary'] = df['Salary'].str.replace('₹', '', regex=False)
df['Salary'] = df['Salary'].str.replace(',', '', regex=False)
df['Salary'] = df['Salary'].str.strip()
df['Salary'] = pd.to_numeric(df['Salary'], errors='coerce')

# One-liner using regex
df['Salary'] = df['Salary'].str.replace(r'[^\d.]', '', regex=True)
df['Salary'] = pd.to_numeric(df['Salary'], errors='coerce')
```

## Step 3: Detect & Fix Invalid Values

### Invalid Numeric Ranges

```python
# Ages must be between 0 and 120
invalid_age = df[(df['Age'] < 0) | (df['Age'] > 120)]
print(f"Invalid ages: {len(invalid_age)}")

# Fix: replace invalid with NaN
df.loc[(df['Age'] < 0) | (df['Age'] > 120), 'Age'] = np.nan

# Or clip to valid range
df['Age'] = df['Age'].clip(0, 120)
```

### Invalid Categorical Values

```python
# Check what values exist in a column
df['Gender'].value_counts()
# Output might show: Male, Female, Other, 'male', 'M', 'Unknown', 'N/A'

# Valid values defined
valid_genders = ['Male', 'Female', 'Other']

# Find invalid entries
df[~df['Gender'].isin(valid_genders)]

# Replace invalid with NaN
df.loc[~df['Gender'].isin(valid_genders), 'Gender'] = np.nan
```

### Invalid Email Format

```python
import re

# Check email format with regex
email_pattern = r'^[\w\.-]+@[\w\.-]+\.\w{2,}$'
df['valid_email'] = df['Email'].str.match(email_pattern)

# View invalid emails
df[~df['valid_email']]

# Replace invalid emails with NaN
df.loc[~df['valid_email'], 'Email'] = np.nan
df.drop(columns=['valid_email'], inplace=True)
```

### Invalid Phone Numbers

```python
# Remove non-digit characters first
df['Phone'] = df['Phone'].str.replace(r'\D', '', regex=True)

# Flag valid 10-digit numbers
df['valid_phone'] = df['Phone'].str.len() == 10
```

---

