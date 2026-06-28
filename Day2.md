# Day 2

Excel basics.

## 1. Data entry in Excel

Definition: Data entry means typing data into cells.

Syntax:

```text
type data in a cell and press Enter
```

Example:

```text
A1 = Name
A2 = Arshad
B1 = Marks
B2 = 85
```

## 2. Essential formulas

Definition: These are basic formulas used for quick calculations.

Syntax:

```excel
=SUM(range)
=AVERAGE(range)
=MAX(range)
=MIN(range)
```

Examples:

```excel
=SUM(A1:A5)
=AVERAGE(A1:A5)
=MAX(A1:A5)
=MIN(A1:A5)
```

`SUM` adds values.
`AVERAGE` gives the average.
`MAX` gives the highest value.
`MIN` gives the lowest value.

## 3. IF formula

Definition: IF checks a condition and gives one value for true and another for false.

Syntax:

```excel
=IF(logical_test, value_if_true, value_if_false)
```

Example:

```excel
=IF(A2>=50, "Pass", "Fail")
```

If A2 is 50 or more, it shows Pass. Otherwise, it shows Fail.

## 4. Nested IF and IFS formula

Definition: Nested IF and IFS are used for more than two conditions.

Nested IF syntax:

```excel
=IF(test1, result1, IF(test2, result2, result3))
```

Nested IF example:

```excel
=IF(A2>=80, "A", IF(A2>=60, "B", "C"))
```

IFS syntax:

```excel
=IFS(test1, result1, test2, result2, TRUE, default_value)
```

IFS example:

```excel
=IFS(A2>=80, "A", A2>=60, "B", TRUE, "C")
```

This gives a grade based on the value in A2.

## 5. Sets

Definition: A set is a collection of unique items.

Syntax:

```python
my_set = {1, 2, 3}
```

Example:

```python
colors = {"red", "blue", "green"}
```

## 6. Casting

Definition: Casting means changing one data type into another.

Syntax:

```python
int(value)
float(value)
str(value)
```

Example:

```python
num = int("10")
```

## 7. Data types

Definition: Data types tell us what kind of value a variable stores.

Syntax:

```python
int
float
str
bool
list
tuple
set
dict
```

Example:

```python
age = 25
name = "Arshad"
marks = 89.5
```

## 8. Important string methods

Definition: String methods help us work with text.

Syntax:

```python
text.lower()
text.upper()
text.strip()
text.replace()
```

Example:

```python
name = "  Arshad  "
name.strip()
```

## 9. List important operators

Definition: Operators are used to work with lists.

Syntax:

```python
+
*
in
not in
```

Example:

```python
[1, 2] + [3, 4]
```

This gives [1, 2, 3, 4].
