# Day 1

Today I revised Python basics.

## 1. List

Definition: A list is a collection of items.

Syntax:

```python
my_list = [1, 2, 3]
```

Example:

```python
fruits = ["apple", "banana", "mango"]
```

## 2. Tuples

Definition: A tuple is a collection of items that cannot be changed.

Syntax:

```python
my_tuple = (1, 2, 3)
```

Example:

```python
days = ("Mon", "Tue", "Wed")
```

## 3. Loops

Definition: Loops repeat a block of code.

Syntax:

```python
for item in items:
    print(item)
```

Example:

```python
for fruit in fruits:
    print(fruit)
```

## 4. Dictionaries

Definition: A dictionary stores data in key and value form.

Syntax:

```python
my_dict = {"name": "Arshad", "age": 25}
```

Example:

```python
student = {"name": "Ali", "marks": 90}
```

## 5. String methods

Definition: String methods help us work with text.

Syntax:

```python
text.upper()
```

Example:

```python
name = "arshad"
name.upper()
```

## 6. Operators

Definition: Operators are used to perform calculations or comparisons.

Syntax:

```python
5 + 2
```

Example:

```python
10 > 5
```

## 7. SQL SELECT query

Definition: SELECT is used to get data from a table.

Syntax:

```sql
SELECT column_name FROM table_name;
```

Example:

```sql
SELECT name, marks FROM students;
```

Important SELECT queries:

```sql
SELECT * FROM world;
SELECT name, population FROM world;
SELECT population FROM world WHERE name = 'Germany';
SELECT name, population FROM world WHERE name IN ('Sweden', 'Norway', 'Denmark');
SELECT name, area FROM world WHERE area BETWEEN 200000 AND 250000;
```

The WHERE clause is used to filter rows.
Single quotes are used for text values.
