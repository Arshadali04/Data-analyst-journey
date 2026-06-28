# Day 2

Excel basics.

1. Data entry in Excel
Syntax: type data in a cell and press Enter.
Example:
A1 = Name
A2 = Arshad
B1 = Marks
B2 = 85

2. Essential formulas
Syntax:
=SUM(range)
=AVERAGE(range)
=MAX(range)
=MIN(range)
Examples:
=SUM(A1:A5) adds the values in A1 to A5.
=AVERAGE(A1:A5) gives the average.
=MAX(A1:A5) gives the highest value.
=MIN(A1:A5) gives the lowest value.

3. IF formula
Syntax:
=IF(logical_test, value_if_true, value_if_false)
Example:
=IF(A2>=50, "Pass", "Fail")
If A2 is 50 or more, it shows Pass. Otherwise, it shows Fail.

4. Nested IF and IFS formula
Nested IF syntax:
=IF(test1, result1, IF(test2, result2, result3))
Nested IF example:
=IF(A2>=80, "A", IF(A2>=60, "B", "C"))
IFS syntax:
=IFS(test1, result1, test2, result2, TRUE, default_value)
IFS example:
=IFS(A2>=80, "A", A2>=60, "B", TRUE, "C")
This gives a grade based on the value in A2.
