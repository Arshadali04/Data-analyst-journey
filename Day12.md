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

