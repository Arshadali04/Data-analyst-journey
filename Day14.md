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

# PART 6: STANDARDIZING TEXT DATA 📝

Real-world text data is inconsistent. The same city might appear as "Mumbai", "mumbai", "MUMBAI", " Mumbai ", "mumbai ". These are all treated as different values — breaking GroupBy, filters, and merges.

## Case Standardization

```python
df['City'] = df['City'].str.lower()    # all lowercase → 'mumbai'
df['City'] = df['City'].str.upper()    # all uppercase → 'MUMBAI'
df['City'] = df['City'].str.title()    # title case → 'Mumbai'
df['City'] = df['City'].str.capitalize()  # only first letter
```

## Stripping Whitespace

```python
df['Name'] = df['Name'].str.strip()    # remove leading & trailing spaces
df['Name'] = df['Name'].str.lstrip()   # remove leading only
df['Name'] = df['Name'].str.rstrip()   # remove trailing only

# Remove extra spaces between words
df['Name'] = df['Name'].str.replace(r'\s+', ' ', regex=True)
```

## Replacing & Removing Characters

```python
# Replace specific text
df['City'] = df['City'].str.replace('Bengaluru', 'Bangalore')

# Remove special characters
df['Name'] = df['Name'].str.replace(r'[^a-zA-Z\s]', '', regex=True)

# Remove numbers from text
df['Description'] = df['Description'].str.replace(r'\d+', '', regex=True)
```

## Fixing Categorical Inconsistencies with Map

```python
# Create a mapping dictionary
gender_map = {
    'male': 'Male', 'm': 'Male', 'M': 'Male',
    'female': 'Female', 'f': 'Female', 'F': 'Female',
    'other': 'Other', 'o': 'Other'
}

df['Gender'] = df['Gender'].str.lower().map(gender_map)
# Values not in map become NaN — handle separately
```

## String Contains / Starts With / Ends With

```python
# Filter rows where name contains 'kumar' (case-insensitive)
df[df['Name'].str.contains('kumar', case=False, na=False)]

# Starts with specific text
df[df['City'].str.startswith('New')]

# Ends with
df[df['Email'].str.endswith('.com')]
```

---

# PART 7: HANDLING OUTLIERS 📈

## What is an Outlier?

> *"An outlier is a data point that differs significantly from other observations. It can be a genuine extreme value or the result of an error."*

| Type | Description | Example |
|------|-------------|---------|
| **Error outlier** | Data entry mistake | Salary = ₹99,999,999 |
| **Genuine outlier** | Real but extreme value | Elon Musk's net worth in a wealth dataset |

> **Key rule:** Always investigate before removing. A genuine outlier contains real information — removing it would distort your analysis.

---

## Method 1: Z-Score (Standard Deviation Method)

**Concept:** Values more than 3 standard deviations from the mean are considered outliers.

```python
from scipy import stats

# Calculate Z-scores
z_scores = np.abs(stats.zscore(df['Salary']))

# Identify outliers (Z-score > 3)
outlier_mask = z_scores > 3
print(f"Outliers found: {outlier_mask.sum()}")

# View outliers
df[outlier_mask]

# Remove outliers
df_clean = df[~outlier_mask]

# Or replace with NaN
df.loc[outlier_mask, 'Salary'] = np.nan
```

> **When to use Z-score:** Data is approximately normally distributed (bell curve). Z-score is sensitive to extreme outliers — one massive outlier can distort the mean and std.

---

## Method 2: IQR Method (Interquartile Range) — Recommended

**Concept:** Values below Q1 − 1.5×IQR or above Q3 + 1.5×IQR are outliers.

```python
Q1 = df['Salary'].quantile(0.25)
Q3 = df['Salary'].quantile(0.75)
IQR = Q3 - Q1

lower_bound = Q1 - 1.5 * IQR
upper_bound = Q3 + 1.5 * IQR

print(f"Lower bound: {lower_bound}")
print(f"Upper bound: {upper_bound}")

# Identify outliers
outliers = df[(df['Salary'] < lower_bound) | (df['Salary'] > upper_bound)]
print(f"Outliers: {len(outliers)}")

# Option 1: Remove outliers
df_clean = df[(df['Salary'] >= lower_bound) & (df['Salary'] <= upper_bound)]

# Option 2: Cap outliers (Winsorization) — less data loss
df['Salary'] = df['Salary'].clip(lower=lower_bound, upper=upper_bound)

# Option 3: Replace with NaN then impute
df.loc[(df['Salary'] < lower_bound) | (df['Salary'] > upper_bound), 'Salary'] = np.nan
df['Salary'].fillna(df['Salary'].median(), inplace=True)
```

> **IQR is preferred over Z-score** because it uses median-based statistics — it's not affected by the outliers themselves.

---

## Method 3: Visualizing Outliers

Always visualize before deciding:

```python
import matplotlib.pyplot as plt
import seaborn as sns

fig, axes = plt.subplots(1, 3, figsize=(15, 4))

# Boxplot — clearest view of outliers
axes[0].boxplot(df['Salary'].dropna())
axes[0].set_title('Boxplot — Salary')

# Histogram — see distribution shape
axes[1].hist(df['Salary'].dropna(), bins=30, edgecolor='black')
axes[1].set_title('Histogram — Salary')

# Scatter plot (against index)
axes[2].scatter(range(len(df)), df['Salary'], alpha=0.5)
axes[2].set_title('Scatter — Salary')

plt.tight_layout()
plt.show()
```

---

## Outlier Handling — Decision Guide

```
Investigate the outlier first:
    ├── Is it a data entry error?
    │       └── YES → Fix or remove it
    │
    ├── Is it a genuine extreme value?
    │       ├── Affects model/analysis badly? → Cap it (Winsorization)
    │       └── Provides real insight?        → Keep it
    │
    └── Are there many outliers?
            └── YES → Re-examine your data source / collection method
```

---

# PART 8: FIXING STRUCTURAL ERRORS 🔧

## Fixing Column Names

```python
# View current column names
print(df.columns.tolist())

# Common issues: spaces, special characters, mixed case
# Fix all at once
df.columns = (
    df.columns
    .str.strip()           # remove leading/trailing spaces
    .str.lower()           # lowercase
    .str.replace(' ', '_', regex=False)   # spaces → underscores
    .str.replace(r'[^\w]', '', regex=True)  # remove special characters
)

# Rename specific columns
df.rename(columns={
    'customer name': 'customer_name',
    'sale amount': 'sale_amount'
}, inplace=True)
```

## Fixing Index Issues

```python
# Reset index after filtering/dropping rows
df.reset_index(drop=True, inplace=True)

# Set a meaningful index
df.set_index('CustomerID', inplace=True)
```

## Handling Mixed Date Formats

```python
# When dates are in inconsistent formats
df['Date'] = pd.to_datetime(df['Date'], infer_datetime_format=True, errors='coerce')
```

## Removing Unwanted Characters from Entire DataFrame

```python
# Strip whitespace from all string columns at once
str_cols = df.select_dtypes(include='object').columns
df[str_cols] = df[str_cols].apply(lambda x: x.str.strip())
```

---

# PART 9: DATA VALIDATION AFTER CLEANING ✅

Before exporting, always validate that cleaning worked:

```python
print("=== POST-CLEANING VALIDATION ===")

# 1. Shape
print(f"Shape: {df.shape}")

# 2. Missing values
print(f"\nMissing values:\n{df.isnull().sum()}")

# 3. Duplicates
print(f"\nDuplicates: {df.duplicated().sum()}")

# 4. Data types
print(f"\nData types:\n{df.dtypes}")

# 5. Value ranges for numeric columns
print(f"\nNumeric summary:\n{df.describe()}")

# 6. Unique values for categorical columns
for col in df.select_dtypes(include='object').columns:
    print(f"\n{col}: {df[col].nunique()} unique values")
    print(df[col].value_counts().head())
```

---

# PART 10: FULL END-TO-END CLEANING WORKFLOW 🔄

```python
import pandas as pd
import numpy as np
from scipy import stats

# ── 1. LOAD ──────────────────────────────────────────────
df = pd.read_csv('raw_data.csv')
print("Original shape:", df.shape)

# ── 2. FIX COLUMN NAMES ──────────────────────────────────
df.columns = (df.columns.str.strip().str.lower()
              .str.replace(' ', '_').str.replace(r'[^\w]', '', regex=True))

# ── 3. FIX DATA TYPES ────────────────────────────────────
df['salary'] = df['salary'].str.replace(r'[^\d.]', '', regex=True)
df['salary'] = pd.to_numeric(df['salary'], errors='coerce')
df['date']   = pd.to_datetime(df['date'], errors='coerce')
df['age']    = pd.to_numeric(df['age'], errors='coerce')

# ── 4. HANDLE MISSING VALUES ──────────────────────────────
# Drop columns with > 50% missing
df.dropna(thresh=int(len(df) * 0.5), axis=1, inplace=True)
# Fill numeric columns with median
df['age'].fillna(df['age'].median(), inplace=True)
df['salary'].fillna(df['salary'].median(), inplace=True)
# Fill categorical columns with mode
df['city'].fillna(df['city'].mode()[0], inplace=True)

# ── 5. REMOVE DUPLICATES ─────────────────────────────────
df.drop_duplicates(inplace=True)
print("After dedup:", df.shape)

# ── 6. FIX INVALID ENTRIES ───────────────────────────────
df.loc[(df['age'] < 0) | (df['age'] > 120), 'age'] = np.nan
df['age'].fillna(df['age'].median(), inplace=True)

# ── 7. STANDARDIZE TEXT ──────────────────────────────────
df['city']   = df['city'].str.strip().str.title()
df['gender'] = df['gender'].str.strip().str.title()

# ── 8. HANDLE OUTLIERS (IQR) ─────────────────────────────
Q1  = df['salary'].quantile(0.25)
Q3  = df['salary'].quantile(0.75)
IQR = Q3 - Q1
df['salary'] = df['salary'].clip(Q1 - 1.5 * IQR, Q3 + 1.5 * IQR)

# ── 9. RESET INDEX ───────────────────────────────────────
df.reset_index(drop=True, inplace=True)

# ── 10. VALIDATE ─────────────────────────────────────────
print("\n=== CLEAN DATA SUMMARY ===")
print(f"Shape:       {df.shape}")
print(f"Missing:     {df.isnull().sum().sum()}")
print(f"Duplicates:  {df.duplicated().sum()}")
print(df.dtypes)

# ── 11. EXPORT ───────────────────────────────────────────
df.to_csv('clean_data.csv', index=False)
print("\nClean data exported.")
```

---

## 🎯 KEY TAKEAWAYS — Data Cleaning

| Topic | Key Point |
|-------|-----------|
| **Why clean** | Garbage In, Garbage Out — dirty data = wrong analysis |
| **Missing values** | Detect → understand why → drop or impute strategically |
| **Mean vs Median fill** | Mean for symmetric data; Median for skewed/outlier-heavy data |
| **Duplicates** | Always specify `subset` to catch logical duplicates by key columns |
| **`errors='coerce'`** | Converts unconvertible values to NaN instead of crashing |
| **Text standardization** | `.str.strip().str.title()` — always strip before comparing |
| **IQR method** | Most robust outlier detection — use this by default |
| **Winsorization** | `clip()` — cap outliers instead of removing them, preserves row count |
| **Validate after** | Always check shape, nulls, duplicates, and dtypes post-cleaning |
| **Pipeline order** | Column names → dtypes → missing → duplicates → invalid → text → outliers |

---

## ⚡ Pro Tips — Data Cleaning

1. **Never clean the original file** — always work on a copy: `df = raw_df.copy()`
2. **Log your cleaning steps** — print shape before and after every major operation so you know what was removed
3. **`errors='coerce'` is your safety net** — always use it with `pd.to_numeric()` and `pd.to_datetime()`
4. **`value_counts()` before cleaning text** — look at actual frequencies before standardizing, so you know what patterns exist
5. **IQR over Z-score** by default — Z-score is distorted by the outliers you're trying to detect
6. **`clip()` over `drop()`** for outliers in small datasets — dropping rows loses data; capping preserves it
7. **Group-wise imputation** is more accurate than overall mean/median — a 20-year-old's missing salary should be filled differently from a 50-year-old's
8. **Regex `errors='coerce'`** on date columns — inconsistent date formats (`15-Jan-2024` vs `2024/01/15`) are a nightmare; `infer_datetime_format=True` handles many cases
9. **Clean column names immediately** — snake_case with no spaces is the safest format for column names
10. **Document your decisions** — record WHY you imputed vs dropped so it's reproducible and explainable

---

## Practice Exercises — Data Cleaning

- [ ] Load a messy CSV and run `df.info()` and `isnull().sum()` to audit it
- [ ] Drop columns where more than 40% of values are missing
- [ ] Fill numeric columns with median and categorical columns with mode
- [ ] Detect and remove exact duplicate rows; then detect partial duplicates by key columns
- [ ] Find and replace invalid age values (< 0 or > 120) with NaN, then impute
- [ ] Convert a messy salary column with "₹" and commas to clean floats using `str.replace` + `pd.to_numeric`
- [ ] Parse inconsistent date formats using `pd.to_datetime(errors='coerce')`
- [ ] Standardize a 'city' column that has mixed case and whitespace issues
- [ ] Build a gender mapping dictionary and use `.map()` to fix inconsistent values
- [ ] Detect outliers in a salary column using IQR and cap them using `clip()`
- [ ] Run the full post-cleaning validation block and verify 0 nulls and 0 duplicates
- [ ] Write a reusable cleaning function that takes a DataFrame and returns a clean one

---
