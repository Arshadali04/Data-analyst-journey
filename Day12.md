# Day 12: NumPy Basics — Arrays, Memory Efficiency & Performance

## 📋 Topics Covered

**Python — NumPy:**
1. What is NumPy & Why Use It?
2. Installing & Importing NumPy
3. NumPy Arrays vs Python Lists
4. Why NumPy is Faster (Memory & Performance)
5. Creating Arrays
6. Array Attributes
7. Array Indexing & Slicing
8. Array Operations (Vectorization)
9. 2D Arrays (Matrices)
10. Useful NumPy Functions
11. Broadcasting

---

# PART 1: WHAT IS NUMPY & WHY USE IT? 🤔

## Definition

> *"NumPy (Numerical Python) is a Python library used for numerical computations. It provides a powerful N-dimensional array object and tools for working with these arrays efficiently."*

NumPy is the **foundation of the entire Python data science ecosystem** — Pandas, Matplotlib, Seaborn, Scikit-learn, and TensorFlow are all built on top of NumPy.

---

## The Problem with Raw Python for Data

Suppose you have a list of 1 million sales figures and want to multiply every one by 1.18 (add GST):

**Pure Python approach:**
```python
sales = [100, 200, 300, ...]   # 1 million items

# Method 1: for loop
result = []
for s in sales:
    result.append(s * 1.18)

# Method 2: list comprehension
result = [s * 1.18 for s in sales]
```

Both work — but they are **slow and memory-hungry** on large data. Python lists:
- Store items as **individual Python objects** (each with type info, reference count, etc.)
- Process elements **one by one** (no parallelism)
- Use **more memory** than needed for pure numbers

**NumPy approach:**
```python
import numpy as np
sales = np.array([100, 200, 300, ...])
result = sales * 1.18   # Done in one line — blazing fast
```

---

## Why NumPy is Faster — The Core Reason

### 1. Fixed Data Type (dtype)

A Python list can hold **mixed types**:
```python
my_list = [1, "hello", 3.14, True]  # totally valid
```

Because of this flexibility, Python must store **type information** for every single element — wasting memory and slowing operations.

A NumPy array holds **only one data type**:
```python
arr = np.array([1, 2, 3, 4])   # all integers — dtype: int64
```

Since every element is the same type, NumPy:
- Doesn't need to store type info per element
- Stores elements as **raw bytes in contiguous memory** (like C arrays)
- Can apply operations to **all elements at once** using optimized C/Fortran code

### 2. Contiguous Memory Layout

**Python list in memory:**
```
List → [pointer1] → object1 (value + type + refcount)
     → [pointer2] → object2 (value + type + refcount)
     → [pointer3] → object3 (value + type + refcount)
```
Each element lives at a **random memory location** — the list just stores pointers to them. Accessing elements requires following pointers, which is slow.

**NumPy array in memory:**
```
Array → [1][2][3][4][5][6][7][8]  ← all values packed together
         ↑ stored side by side in a continuous block
```
All values are stored **next to each other** in a single block. The CPU can load them in bulk — much faster.

### 3. Vectorized Operations (No Python Loop)

NumPy operations run in **compiled C code under the hood** — not Python. This means:
- No Python interpreter overhead per element
- Operations run on the entire array at once (vectorized)
- Can utilize CPU-level optimizations (SIMD instructions)

---

## Speed Comparison (Conceptual)

| Operation | Python List | NumPy Array |
|-----------|------------|-------------|
| Multiply 1M numbers | ~0.5 seconds | ~0.005 seconds |
| Sum of 1M numbers | ~50 ms | ~1 ms |
| Memory for 1M integers | ~28 MB | ~8 MB |

> **Real benchmark:** NumPy is typically **10x–100x faster** than equivalent Python list operations.

---

# PART 2: INSTALLING & IMPORTING NUMPY 📦

## Installation

```bash
pip install numpy
```

## Importing (Standard Convention)

```python
import numpy as np
```

`np` is the universally accepted alias for NumPy — every data scientist uses this convention. You'll see it in every tutorial, documentation, and Stack Overflow answer.

---

# PART 3: NUMPY ARRAYS 🧱

## Creating a 1D Array

```python
import numpy as np

# From a Python list
arr = np.array([10, 20, 30, 40, 50])
print(arr)         # [10 20 30 40 50]
print(type(arr))   # <class 'numpy.ndarray'>
```

Notice: no commas between elements in the output — NumPy prints arrays differently from Python lists.

---

## NumPy Array vs Python List — Side by Side

| Feature | Python List | NumPy Array |
|---------|------------|-------------|
| Data types | Mixed (int, str, float together) | Single type only (homogeneous) |
| Memory | More (stores Python objects) | Less (stores raw bytes) |
| Speed | Slower (interpreted) | Faster (compiled C code) |
| Operations | Need loops | Vectorized (element-wise directly) |
| Dimensions | Nested lists for 2D | Native N-dimensional support |
| Math functions | Not built-in | Rich built-in math (`np.sin`, `np.log`, etc.) |

---

## Data Types (dtype)

NumPy auto-detects the dtype from the input:

```python
arr_int   = np.array([1, 2, 3])
arr_float = np.array([1.0, 2.0, 3.0])
arr_str   = np.array(['a', 'b', 'c'])

print(arr_int.dtype)    # int64
print(arr_float.dtype)  # float64
print(arr_str.dtype)    # <U1
```

### Specifying dtype Explicitly

```python
arr = np.array([1, 2, 3], dtype=np.float32)
print(arr.dtype)   # float32
print(arr)         # [1. 2. 3.]
```

### Common dtypes

| dtype | Description | Memory per element |
|-------|-------------|-------------------|
| `int8` | Integer (-128 to 127) | 1 byte |
| `int32` | Integer (±2 billion) | 4 bytes |
| `int64` | Large integer | 8 bytes |
| `float32` | Single precision float | 4 bytes |
| `float64` | Double precision float | 8 bytes |
| `bool` | True / False | 1 byte |
| `str` / `<U` | Unicode string | Variable |

> **Memory tip:** Use `float32` instead of `float64` when precision allows — halves memory usage on large arrays.

---

# PART 4: ARRAY ATTRIBUTES 📐

Every NumPy array has built-in properties that describe its structure:

```python
arr = np.array([[1, 2, 3],
                [4, 5, 6]])

print(arr.shape)    # (2, 3) → 2 rows, 3 columns
print(arr.ndim)     # 2 → number of dimensions
print(arr.size)     # 6 → total number of elements
print(arr.dtype)    # int64 → data type of elements
print(arr.itemsize) # 8 → bytes per element
print(arr.nbytes)   # 48 → total bytes (size × itemsize)
```

| Attribute | What It Returns | Example |
|-----------|----------------|---------|
| `.shape` | Tuple of dimensions | `(2, 3)` |
| `.ndim` | Number of dimensions | `2` |
| `.size` | Total element count | `6` |
| `.dtype` | Data type | `int64` |
| `.itemsize` | Bytes per element | `8` |
| `.nbytes` | Total bytes in array | `48` |

---

# PART 5: CREATING ARRAYS — ALL METHODS 🏗️

## From a Python List or Nested List

```python
# 1D array
arr1d = np.array([1, 2, 3, 4, 5])

# 2D array
arr2d = np.array([[1, 2, 3],
                  [4, 5, 6]])
```

## np.zeros() — Array of All Zeros

```python
np.zeros(5)          # [0. 0. 0. 0. 0.]  (1D)
np.zeros((3, 4))     # 3×4 matrix of zeros
```

## np.ones() — Array of All Ones

```python
np.ones(5)           # [1. 1. 1. 1. 1.]
np.ones((2, 3))      # 2×3 matrix of ones
```

## np.full() — Array Filled with a Specific Value

```python
np.full(5, 7)        # [7 7 7 7 7]
np.full((3, 3), 9)   # 3×3 matrix of nines
```

## np.arange() — Evenly Spaced Values (Like Python range)

```python
np.arange(10)           # [0 1 2 3 4 5 6 7 8 9]
np.arange(2, 10)        # [2 3 4 5 6 7 8 9]
np.arange(0, 10, 2)     # [0 2 4 6 8]  (step=2)
np.arange(0, 1, 0.1)    # [0.  0.1 0.2 ... 0.9] (float step)
```

## np.linspace() — Evenly Spaced Values (Fixed Count)

```python
np.linspace(0, 1, 5)     # [0.   0.25  0.5   0.75  1. ]
np.linspace(0, 100, 11)  # [0. 10. 20. 30. ... 100.]
```

> `arange` specifies the **step size** → exact values depend on step.
> `linspace` specifies the **number of points** → always exactly that many values.

## np.eye() — Identity Matrix

```python
np.eye(3)
# [[1. 0. 0.]
#  [0. 1. 0.]
#  [0. 0. 1.]]
```

## np.random — Random Arrays

```python
np.random.rand(3)          # 3 random floats [0, 1)
np.random.rand(2, 3)       # 2×3 random floats [0, 1)
np.random.randint(1, 10, 5)        # 5 random integers between 1-9
np.random.randint(1, 100, (3, 3))  # 3×3 random integers
np.random.randn(5)         # 5 values from standard normal distribution
np.random.seed(42)         # Set seed for reproducibility
```

---

# PART 6: INDEXING & SLICING ✂️

## 1D Array Indexing

```python
arr = np.array([10, 20, 30, 40, 50])

print(arr[0])    # 10  (first element)
print(arr[-1])   # 50  (last element)
print(arr[-2])   # 40  (second from last)
```

## 1D Array Slicing

```python
arr = np.array([10, 20, 30, 40, 50])

print(arr[1:4])   # [20 30 40]  (index 1 to 3)
print(arr[:3])    # [10 20 30]  (first 3)
print(arr[2:])    # [30 40 50]  (from index 2 to end)
print(arr[::2])   # [10 30 50]  (every 2nd element)
print(arr[::-1])  # [50 40 30 20 10]  (reversed)
```

## 2D Array Indexing

```python
arr = np.array([[1, 2, 3],
                [4, 5, 6],
                [7, 8, 9]])

print(arr[0, 0])   # 1  (row 0, col 0)
print(arr[1, 2])   # 6  (row 1, col 2)
print(arr[-1, -1]) # 9  (last row, last col)
```

## 2D Array Slicing

```python
arr = np.array([[1, 2, 3],
                [4, 5, 6],
                [7, 8, 9]])

print(arr[0, :])      # [1 2 3]  (entire first row)
print(arr[:, 1])      # [2 5 8]  (entire second column)
print(arr[0:2, 1:3])  # [[2 3] [5 6]]  (submatrix)
```

## Boolean Indexing (Filtering)

```python
arr = np.array([10, 25, 30, 45, 50, 65])

# Create a boolean mask
mask = arr > 30
print(mask)    # [False False False  True  True  True]

# Apply mask to filter
print(arr[mask])        # [45 50 65]
print(arr[arr > 30])    # Same — shorthand
print(arr[arr % 2 == 0])  # [10 30 50] — even numbers only
```

---

# PART 7: ARRAY OPERATIONS (VECTORIZATION) ⚡

## Element-wise Arithmetic

NumPy performs operations on **every element simultaneously** — no loop needed:

```python
a = np.array([1, 2, 3, 4, 5])
b = np.array([10, 20, 30, 40, 50])

print(a + b)    # [11 22 33 44 55]
print(a - b)    # [-9 -18 -27 -36 -45]
print(a * b)    # [10 40 90 160 250]
print(a / b)    # [0.1 0.1 0.1 0.1 0.1]
print(a ** 2)   # [1 4 9 16 25]
print(a % 2)    # [1 0 1 0 1]
```

## Scalar Operations (Broadcasting Scalar)

```python
arr = np.array([10, 20, 30, 40])

print(arr + 5)    # [15 25 35 45]
print(arr * 2)    # [20 40 60 80]
print(arr / 10)   # [1. 2. 3. 4.]
print(arr ** 2)   # [100 400 900 1600]
```

## Aggregate Functions

```python
arr = np.array([5, 2, 8, 1, 9, 3])

print(np.sum(arr))     # 28
print(np.min(arr))     # 1
print(np.max(arr))     # 9
print(np.mean(arr))    # 4.666...
print(np.median(arr))  # 4.0
print(np.std(arr))     # Standard deviation
print(np.var(arr))     # Variance
print(np.argmin(arr))  # 3 → index of minimum
print(np.argmax(arr))  # 4 → index of maximum
```

## Axis-wise Aggregation (2D Arrays)

```python
arr = np.array([[1, 2, 3],
                [4, 5, 6]])

print(np.sum(arr))          # 21  (total)
print(np.sum(arr, axis=0))  # [5 7 9]  (column sums)
print(np.sum(arr, axis=1))  # [6 15]   (row sums)
print(np.mean(arr, axis=0)) # [2.5 3.5 4.5] (column averages)
```

> **Axis rule:** `axis=0` → operate **down columns** (across rows). `axis=1` → operate **across columns** (across a row).

---

# PART 8: 2D ARRAYS (MATRICES) 🔢

## Reshaping Arrays

Change the shape of an array without changing its data:

```python
arr = np.arange(12)      # [0 1 2 3 4 5 6 7 8 9 10 11]

arr_2d = arr.reshape(3, 4)
# [[ 0  1  2  3]
#  [ 4  5  6  7]
#  [ 8  9 10 11]]

arr_3d = arr.reshape(2, 2, 3)  # 2 blocks, 2 rows, 3 cols

# -1 lets NumPy auto-calculate one dimension
arr.reshape(-1, 4)   # (3, 4) — auto-calculates 3 rows
arr.reshape(3, -1)   # (3, 4) — auto-calculates 4 cols
```

## Flattening (2D → 1D)

```python
arr_2d = np.array([[1, 2, 3], [4, 5, 6]])

print(arr_2d.flatten())  # [1 2 3 4 5 6] — returns a COPY
print(arr_2d.ravel())    # [1 2 3 4 5 6] — returns a VIEW (faster)
```

## Transpose

```python
arr = np.array([[1, 2, 3],
                [4, 5, 6]])

print(arr.T)
# [[1 4]
#  [2 5]
#  [3 6]]
```

## Matrix Operations

```python
A = np.array([[1, 2], [3, 4]])
B = np.array([[5, 6], [7, 8]])

# Element-wise multiplication
print(A * B)

# Matrix multiplication (dot product)
print(np.dot(A, B))
print(A @ B)           # @ operator — same as np.dot
```

---

# PART 9: USEFUL NUMPY FUNCTIONS 🔧

## Sorting

```python
arr = np.array([3, 1, 4, 1, 5, 9, 2, 6])

print(np.sort(arr))       # [1 1 2 3 4 5 6 9] — returns sorted COPY
arr.sort()                # sorts IN PLACE (modifies arr)
print(np.argsort(arr))    # indices that would sort the array
```

## Stacking Arrays

```python
a = np.array([1, 2, 3])
b = np.array([4, 5, 6])

np.concatenate([a, b])          # [1 2 3 4 5 6]
np.vstack([a, b])               # [[1 2 3] [4 5 6]] → vertical stack
np.hstack([a, b])               # [1 2 3 4 5 6] → horizontal stack
```

## Splitting Arrays

```python
arr = np.array([1, 2, 3, 4, 5, 6])

np.split(arr, 3)        # [array([1,2]), array([3,4]), array([5,6])]
np.split(arr, [2, 4])   # split at index 2 and 4
```

## Unique Values

```python
arr = np.array([1, 2, 2, 3, 3, 3, 4])

print(np.unique(arr))               # [1 2 3 4]
vals, counts = np.unique(arr, return_counts=True)
print(vals)    # [1 2 3 4]
print(counts)  # [1 2 3 1]
```

## Mathematical Functions

```python
arr = np.array([1, 4, 9, 16, 25])

print(np.sqrt(arr))     # [1. 2. 3. 4. 5.]
print(np.log(arr))      # Natural log
print(np.log10(arr))    # Base-10 log
print(np.exp(arr))      # e^x for each element
print(np.abs(arr))      # Absolute values
print(np.round(arr, 2)) # Round to 2 decimal places
```

---

# PART 10: BROADCASTING 📡

## What is Broadcasting?

> *"Broadcasting allows NumPy to perform operations on arrays of different shapes by automatically expanding the smaller array to match the larger one — without copying data."*

### Simple Example — Scalar Broadcasting

```python
arr = np.array([1, 2, 3, 4, 5])
result = arr + 10    # 10 is "broadcast" to match arr's shape

# Internally: [1+10, 2+10, 3+10, 4+10, 5+10]
print(result)  # [11 12 13 14 15]
```

### 2D Broadcasting Example

```python
matrix = np.array([[1, 2, 3],
                   [4, 5, 6],
                   [7, 8, 9]])

row = np.array([10, 20, 30])   # shape: (3,)

result = matrix + row
# row is broadcast to match each row of matrix:
# [[1+10, 2+20, 3+30],
#  [4+10, 5+20, 6+30],
#  [7+10, 8+20, 9+30]]

print(result)
# [[11 22 33]
#  [14 25 36]
#  [17 28 39]]
```

### Broadcasting Rules (Simple Version)

1. If arrays have different number of dimensions, pad the smaller shape with 1s on the left
2. Dimensions of size 1 are stretched to match the other array's size
3. If neither dimension is 1 and they don't match → error

---

## 🎯 KEY TAKEAWAYS — NumPy

| Concept | Key Point |
|---------|-----------|
| **Why NumPy** | Contiguous memory + fixed dtype + C-compiled operations = 10–100x faster |
| **ndarray** | Core NumPy object; N-dimensional, single data type |
| **dtype** | Fixed type per array; choose smaller types (float32) to save memory |
| **Vectorization** | Operations apply to entire array at once — no Python loop needed |
| **Indexing** | `arr[row, col]` for 2D; negative indices count from end |
| **Boolean indexing** | `arr[arr > 5]` — filter elements using conditions directly |
| **reshape(-1, n)** | -1 tells NumPy to auto-calculate that dimension |
| **axis=0** | Operate down columns; `axis=1` → operate across rows |
| **Broadcasting** | Smaller arrays auto-expand to match larger array shape |
| **flatten vs ravel** | `flatten()` returns copy; `ravel()` returns view (faster) |

---

## ⚡ Pro Tips — NumPy

1. **Avoid Python loops on NumPy arrays** — if you're writing a for loop over an ndarray, there's almost certainly a vectorized NumPy function that does it faster
2. **Use `dtype=float32`** on large datasets when float64 precision isn't needed — cuts memory in half
3. **`arr.shape`** is your best friend — check it after every reshape, transpose, or stack operation
4. **Boolean indexing** is how Pandas filtering works internally — mastering it here makes Pandas feel natural
5. **`np.random.seed(42)`** before any random operation ensures reproducibility — use it in every project
6. **`@` operator** for matrix multiplication is cleaner than `np.dot()` — use it in Python 3.5+
7. **Views vs Copies** — slicing returns a VIEW (modifying it modifies original); `.copy()` if you need independence
8. **`np.where(condition, x, y)`** — element-wise if-else on arrays; very useful for data transformations

---

## Practice Exercises — NumPy

- [ ] Create a 1D array of numbers 1–20 using `arange`
- [ ] Create a 4×4 matrix of random integers between 1 and 100
- [ ] Find the shape, size, dtype, and nbytes of both arrays
- [ ] Slice the last 2 rows and first 2 columns from your 4×4 matrix
- [ ] Use boolean indexing to find all elements greater than 50 in the matrix
- [ ] Multiply every element by 2 without a loop
- [ ] Reshape a 1D array of 24 elements into a 4×6 matrix, then transpose it
- [ ] Calculate column-wise and row-wise sums of a 2D array
- [ ] Stack two arrays vertically and horizontally
- [ ] Find unique values and their counts in an array with duplicates

---
