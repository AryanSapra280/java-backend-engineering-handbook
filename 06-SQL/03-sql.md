Exactly — you're right. The previous `.md` should have **continued from #26 and included the Subqueries section before Aggregations**, rather than starting at `#27` with aggregations.

Here is the **refined complete continuation**, with **Subqueries → Aggregations → Window Functions**, preserving the details and doubts we covered. You can replace the previous Part 3 with this.

````md
# SQL Interview Preparation — Theory Notes (Part 3)

---

# 27. Subqueries

A **subquery** is a query written inside another SQL query.

Example:

```sql
SELECT *
FROM employee
WHERE salary > (
    SELECT AVG(salary)
    FROM employee
);
````

The inner query:

```sql
SELECT AVG(salary)
FROM employee;
```

returns one value, for example:

```text
70000
```

The outer query then effectively becomes:

```sql
SELECT *
FROM employee
WHERE salary > 70000;
```

### Mental Model

```text
Inner query
    ↓
Produces a result
    ↓
Outer query uses that result
```

---

# 28. Scalar Subquery

A scalar subquery returns **one value**.

Example:

> Find employees earning more than the average salary.

```sql
SELECT *
FROM employee
WHERE salary > (
    SELECT AVG(salary)
    FROM employee
);
```

The inner query returns one value:

```text
70000
```

Therefore it can be used with:

```text
>
<
>=
<=
=
```

### Mental Model

```text
Scalar Subquery
→ Returns ONE value
```

---

# 29. Subquery in WHERE

A subquery can return multiple values and be used with `IN`.

Example:

> Find employees belonging to departments located in Bangalore.

```sql
SELECT *
FROM employee
WHERE department_id IN (
    SELECT department_id
    FROM department
    WHERE location = 'Bangalore'
);
```

The inner query might return:

```text
10
20
30
```

So conceptually the outer query becomes:

```sql
WHERE department_id IN (10, 20, 30)
```

### Mental Model

```text
Inner query
    ↓
Returns a set of values
    ↓
Outer query checks against that set
```

---

# 30. Subquery in FROM

A subquery can also be used inside `FROM`.

The result of the subquery behaves like a temporary/derived table for the outer query.

Example:

```sql
SELECT department_id, avg_salary
FROM (
    SELECT
        department_id,
        AVG(salary) AS avg_salary
    FROM employee
    GROUP BY department_id
) temp
WHERE avg_salary > 80000;
```

The inner query produces:

```text
department_id | avg_salary
--------------|-----------
10            | 90000
20            | 70000
30            | 85000
```

The outer query then filters this result:

```text
department_id | avg_salary
--------------|-----------
10            | 90000
30            | 85000
```

### Mental Model

```text
Employee table
      ↓
Inner query
      ↓
Derived result / temporary result
      ↓
Outer query
      ↓
Final result
```

---

# 31. Correlated Subquery

A correlated subquery is a subquery that refers to a column from the outer query.

Example:

> Find employees whose salary is greater than the average salary of their own department.

```sql
SELECT *
FROM employee e
WHERE e.salary > (
    SELECT AVG(e2.salary)
    FROM employee e2
    WHERE e2.department_id = e.department_id
);
```

Notice:

```sql
e2.department_id = e.department_id
                         ↑
                    outer query
```

`e` belongs to the outer query:

```sql
FROM employee e
```

The inner query depends on the current row of the outer query.

---

## Conceptual Working

For Aryan:

```text
Aryan
  ↓
Get Aryan's department
  ↓
Calculate average salary of Aryan's department
  ↓
Compare Aryan's salary with that average
  ↓
Keep / discard
```

Then the next employee is considered.

### Mental Model

```text
Normal subquery:

Outer query
     ↓
Uses one independent result


Correlated subquery:

Outer row
     ↓
Inner query depends on that row
     ↓
Result
     ↓
Next outer row
```

---

# 32. Subquery vs JOIN

Many requirements can be solved using either a subquery or a JOIN.

Example:

### Using Subquery

```sql
SELECT *
FROM employee
WHERE department_id IN (
    SELECT department_id
    FROM department
    WHERE location = 'Bangalore'
);
```

### Using JOIN

```sql
SELECT e.*
FROM employee e
JOIN department d
    ON e.department_id = d.department_id
WHERE d.location = 'Bangalore';
```

Both can solve the requirement.

Do NOT say:

> "JOIN is always faster."

or:

> "Subquery is always faster."

The actual performance depends on:

* indexes
* table size
* data distribution
* database optimizer
* statistics
* query structure

The important thing is to understand what the query is trying to express.

---

# 33. EXISTS Revisited

`EXISTS` checks whether at least one matching row exists.

Example:

```sql
SELECT *
FROM employee e
WHERE EXISTS (
    SELECT 1
    FROM department d
    WHERE d.department_id = e.department_id
);
```

The important part is:

```sql
SELECT 1
```

We don't actually care about the value `1`.

We only care whether a matching row exists.

Conceptually:

```text
Matching row exists → TRUE
No matching row      → FALSE
```

---

# 34. Why SELECT 1 in EXISTS?

Consider:

```sql
SELECT 1
FROM department
WHERE department_id = 10;
```

If department 10 exists, the query produces a row containing `1`.

The actual value doesn't matter.

Therefore:

```sql
SELECT 1
```

is commonly used to communicate:

> "I only care whether a row exists."

---

# 35. EXISTS vs IN

### IN

```sql
SELECT *
FROM employee e
WHERE e.department_id IN (
    SELECT d.department_id
    FROM department d
);
```

Mental model:

```text
Get a set of department IDs
        ↓
Check whether employee.department_id
belongs to that set
```

### EXISTS

```sql
SELECT *
FROM employee e
WHERE EXISTS (
    SELECT 1
    FROM department d
    WHERE d.department_id = e.department_id
);
```

Mental model:

```text
Take an employee
      ↓
Check whether a matching department exists
      ↓
TRUE / FALSE
```

### Remember

```text
IN
→ Is this value in this set?

EXISTS
→ Does a matching row exist?
```

---

# 36. EXISTS vs JOIN

These can sometimes produce the same result.

### JOIN

```sql
SELECT e.*
FROM employee e
JOIN department d
    ON d.department_id = e.department_id;
```

Meaning:

> Combine employee and department data.

### EXISTS

```sql
SELECT e.*
FROM employee e
WHERE EXISTS (
    SELECT 1
    FROM department d
    WHERE d.department_id = e.department_id
);
```

Meaning:

> I only want employees for whom a matching department exists.

---

## Important Difference

Suppose the department table contains duplicate IDs:

```text
department_id
-------------
10
10
20
```

Employee:

```text
emp_id | department_id
-------|--------------
1      | 10
```

JOIN can produce:

```text
emp_id
------
1
1
```

because there are two matching department rows.

EXISTS produces:

```text
emp_id
------
1
```

because it only asks:

```text
Does at least one matching row exist?
```

---

# 37. EXISTS and Performance

Do NOT say:

> "EXISTS is always faster than JOIN."

That is too absolute.

Modern database optimizers can transform queries and choose different execution strategies.

Performance depends on:

```text
Indexes
Table size
Data distribution
Statistics
Optimizer
Query structure
```

A safer interview answer:

> "`EXISTS` is useful when we only care whether a related row exists. The database may be able to stop looking once a match is found, but we should check the execution plan rather than assuming EXISTS is always faster."

---

# 38. Nth Highest Salary — Subquery Approach

A common interview problem:

> Find the second-highest salary.

One approach:

```sql
SELECT MAX(salary)
FROM employee
WHERE salary < (
    SELECT MAX(salary)
    FROM employee
);
```

Inner query:

```sql
SELECT MAX(salary)
FROM employee;
```

Suppose it returns:

```text
100000
```

Outer query then finds:

```text
Maximum salary < 100000
```

Result:

```text
90000
```

---

## Duplicate Salaries

Suppose:

```text
100000
100000
90000
80000
```

If the interviewer asks:

> Find the second-highest DISTINCT salary.

The answer is:

```text
90000
```

This is where `DISTINCT`, subqueries, and later window functions become useful.

---

# 39. Aggregate Functions

Aggregate functions perform calculations over multiple rows.

Common aggregate functions:

```text
COUNT()
SUM()
AVG()
MIN()
MAX()
```

---

# 40. COUNT(*)

`COUNT(*)` counts rows.

```sql
SELECT COUNT(*)
FROM employee;
```

Suppose:

```text
emp_id | name  | salary
-------|-------|-------
1      | Aryan | 100000
2      | Rahul | 80000
3      | Amit  | NULL
4      | Neha  | 60000
```

Then:

```sql
COUNT(*)
```

returns:

```text
4
```

because there are four rows.

### Mental Model

```text
COUNT(*)
→ Count rows
→ NULL values do not matter
```

---

# 41. COUNT(column)

`COUNT(column)` counts only non-NULL values in that column.

Using:

```text
emp_id | name  | salary
-------|-------|-------
1      | Aryan | 100000
2      | Rahul | 80000
3      | Amit  | NULL
4      | Neha  | 60000
```

Query:

```sql
SELECT COUNT(salary)
FROM employee;
```

Result:

```text
3
```

because one salary is NULL.

### Important Difference

```text
COUNT(*)
→ Counts rows

COUNT(column)
→ Counts non-NULL values
```

### Interview Answer

> "`COUNT(*)` counts all rows, whereas `COUNT(column)` counts only rows where that column is non-NULL."

---

# 42. SUM()

`SUM()` calculates the total of numeric values.

```sql
SELECT SUM(salary)
FROM employee;
```

For:

```text
100000
80000
60000
NULL
```

the result is:

```text
240000
```

NULL is ignored.

---

# 43. AVG()

`AVG()` calculates the average of non-NULL values.

For:

```text
100000
80000
NULL
60000
```

the calculation is:

```text
(100000 + 80000 + 60000) / 3
```

Result:

```text
80000
```

NULL is not treated as zero.

---

# 44. MIN() and MAX()

```sql
SELECT
    MIN(salary),
    MAX(salary)
FROM employee;
```

Returns the minimum and maximum non-NULL values.

---

# 45. Aggregate Functions + GROUP BY

Example:

> Find the number of employees in each department.

```sql
SELECT
    department_id,
    COUNT(*)
FROM employee
GROUP BY department_id;
```

Conceptually:

```text
Employee table
      ↓
GROUP BY department_id
      ↓
Create one group per department
      ↓
COUNT(*) within each group
```

Example:

```text
department_id | count
--------------|------
10            | 5
20            | 3
30            | 7
```

---

# 46. Multiple Aggregates

We can calculate several aggregates together.

```sql
SELECT
    department_id,
    COUNT(*) AS employee_count,
    SUM(salary) AS total_salary,
    AVG(salary) AS average_salary,
    MIN(salary) AS minimum_salary,
    MAX(salary) AS maximum_salary
FROM employee
GROUP BY department_id;
```

This gives department-level statistics.

---

# 47. HAVING with Aggregate Functions

Requirement:

> Find departments having more than 5 employees.

```sql
SELECT
    department_id,
    COUNT(*)
FROM employee
GROUP BY department_id
HAVING COUNT(*) > 5;
```

We cannot normally use:

```sql
WHERE COUNT(*) > 5
```

because `WHERE` filters rows before the grouping/aggregation result exists.

### Mental Model

```text
WHERE
→ Filter rows

GROUP BY
→ Create groups

HAVING
→ Filter groups
```

---

# 48. WHERE + GROUP BY + HAVING

These can be used together.

Example:

> Find departments where the average salary of employees earning more than 50,000 is greater than 80,000.

```sql
SELECT
    department_id,
    AVG(salary) AS avg_salary
FROM employee
WHERE salary > 50000
GROUP BY department_id
HAVING AVG(salary) > 80000;
```

Conceptually:

```text
Employee rows
      ↓
WHERE salary > 50000
      ↓
Remaining rows
      ↓
GROUP BY department
      ↓
Calculate AVG
      ↓
HAVING AVG > 80000
```

---

# 49. GROUP BY Selection Rule

When using GROUP BY, a selected column generally needs to be:

1. Present in GROUP BY
2. OR wrapped in an aggregate function

Valid:

```sql
SELECT
    department_id,
    COUNT(*)
FROM employee
GROUP BY department_id;
```

Usually invalid:

```sql
SELECT
    department_id,
    name,
    COUNT(*)
FROM employee
GROUP BY department_id;
```

Why?

Suppose department 10 contains:

```text
Aryan
Rahul
Amit
```

Which `name` should SQL return for department 10?

There is no single deterministic answer.

Therefore `name` must be grouped or aggregated if the SQL dialect requires it.

---

# 50. Window Functions ⭐

Window functions allow calculations across related rows **without collapsing those rows into one row**.

Important functions for interviews:

```text
ROW_NUMBER()
RANK()
DENSE_RANK()
```

Important concepts:

```text
PARTITION BY
ORDER BY
```

---

# 51. GROUP BY vs Window Function

Suppose:

```text
department | employee | salary
-----------|----------|-------
IT         | Aryan    | 100000
IT         | Rahul    | 80000
IT         | Amit     | 60000
HR         | Neha     | 90000
HR         | Ravi     | 70000
```

Using GROUP BY:

```sql
SELECT
    department,
    AVG(salary)
FROM employee
GROUP BY department;
```

Result:

```text
IT | 80000
HR | 80000
```

The individual employee rows are collapsed.

---

Using a window function:

```sql
SELECT
    name,
    department_id,
    salary,
    AVG(salary) OVER (
        PARTITION BY department_id
    ) AS department_avg
FROM employee;
```

Result conceptually:

```text
Aryan | IT | 100000 | 80000
Rahul | IT | 80000  | 80000
Amit  | IT | 60000  | 80000
Neha  | HR | 90000  | 80000
Ravi  | HR | 70000  | 80000
```

Every employee remains.

### Mental Model

```text
GROUP BY
→ Collapse rows into groups

Window Function
→ Keep the rows
→ Calculate using related rows
```

---

# 52. PARTITION BY

`PARTITION BY` divides rows into logical groups for the window function.

Example:

```sql
AVG(salary) OVER (
    PARTITION BY department_id
)
```

means:

> Calculate the average separately for each department.

Conceptually:

```text
All employees
      ↓
PARTITION BY department
      ↓
Department 10 → employees
Department 20 → employees
Department 30 → employees
      ↓
Calculate separately within each partition
```

### Important

```text
PARTITION BY
≠
GROUP BY
```

`GROUP BY` collapses rows.

`PARTITION BY` does not.

---

# 53. ROW_NUMBER()

`ROW_NUMBER()` gives every row a unique sequential number.

```sql
SELECT
    name,
    salary,
    ROW_NUMBER() OVER (
        ORDER BY salary DESC
    ) AS row_num
FROM employee;
```

Example:

```text
name  | salary | row_num
------|--------|--------
Aryan | 100000 | 1
Rahul | 90000  | 2
Amit  | 90000  | 3
Neha  | 80000  | 4
```

Even if two salaries are equal, they receive different row numbers.

---

# 54. RANK()

`RANK()` gives the same rank to tied values.

```sql
SELECT
    name,
    salary,
    RANK() OVER (
        ORDER BY salary DESC
    ) AS rank
FROM employee;
```

Result:

```text
name  | salary | rank
------|--------|-----
Aryan | 100000 | 1
Rahul | 90000  | 2
Amit  | 90000  | 2
Neha  | 80000  | 4
```

Notice:

```text
1
2
2
4
```

Rank 3 is skipped.

---

# 55. DENSE_RANK()

`DENSE_RANK()` also gives the same rank to ties, but does not skip ranks.

```sql
SELECT
    name,
    salary,
    DENSE_RANK() OVER (
        ORDER BY salary DESC
    ) AS rank
FROM employee;
```

Result:

```text
name  | salary | rank
------|--------|-----
Aryan | 100000 | 1
Rahul | 90000  | 2
Amit  | 90000  | 2
Neha  | 80000  | 3
```

---

# 56. ROW_NUMBER vs RANK vs DENSE_RANK

Remember:

```text
Function       Ties?          Skip rank?
-----------------------------------------
ROW_NUMBER     No             Not applicable
RANK           Same rank      YES
DENSE_RANK     Same rank      NO
```

### Interview Shortcut

If asked:

> Give every row a unique position.

Use:

```sql
ROW_NUMBER()
```

If asked:

> Rank employees and allow ties, with gaps.

Use:

```sql
RANK()
```

If asked:

> Rank employees and allow ties, without gaps.

Use:

```sql
DENSE_RANK()
```

---

# 57. PARTITION BY + ROW_NUMBER()

Very important interview pattern.

Requirement:

> Find the highest-paid employee in each department.

```sql
SELECT *
FROM (
    SELECT
        e.*,
        ROW_NUMBER() OVER (
            PARTITION BY department_id
            ORDER BY salary DESC
        ) AS rn
    FROM employee e
) temp
WHERE rn = 1;
```

Conceptually:

```text
Department 10
      ↓
Sort salary DESC
      ↓
Highest → row_number 1

Department 20
      ↓
Sort salary DESC
      ↓
Highest → row_number 1
```

Then:

```sql
WHERE rn = 1
```

keeps the highest-paid employee from every department.

---

# 58. Nth Highest Salary with DENSE_RANK()

Requirement:

> Find the second-highest distinct salary.

```sql
SELECT DISTINCT salary
FROM (
    SELECT
        salary,
        DENSE_RANK() OVER (
            ORDER BY salary DESC
        ) AS rnk
    FROM employee
) temp
WHERE rnk = 2;
```

Suppose:

```text
100000
90000
90000
80000
```

Ranks:

```text
100000 → 1
90000  → 2
90000  → 2
80000  → 3
```

Therefore:

```text
rnk = 2
```

returns:

```text
90000
```

---

# 59. Window Function Interview Patterns

Window functions are especially useful for:

```text
Nth highest salary
Top N employees per department
Ranking employees
Finding duplicates
Latest record per entity
Previous/next row comparisons
Running totals
```

For the current interview preparation, focus mainly on:

```text
ROW_NUMBER()
RANK()
DENSE_RANK()
PARTITION BY
ORDER BY
```

---

# 60. SQL Theory Remaining

After completing Subqueries, Aggregations and Window Functions, the remaining high-value theory is:

## 1. Indexes ⭐

Need to know:

```text
What is an index?
Why does an index improve queries?
B-tree
Composite indexes
Indexes on WHERE/JOIN columns
Why too many indexes are bad
Index scan
Sequential/full table scan
```

---

## 2. EXPLAIN / Query Plan ⭐

Need to know:

```text
EXPLAIN
EXPLAIN ANALYZE
Sequential Scan
Index Scan
Cost
Rows
Join strategies
How to identify a slow query
```

---

## 3. Transactions ⭐

Need to know:

```text
ACID
COMMIT
ROLLBACK
READ COMMITTED
REPEATABLE READ
SERIALIZABLE
```

These are particularly relevant because of backend transaction handling experience.

---

# 61. Final SQL Theory Roadmap

```text
SQL BASICS
    ↓
SELECT
FROM
WHERE
JOIN
ON
GROUP BY
HAVING
ORDER BY
LIMIT / OFFSET

    ↓

JOIN CONCEPTS
    ↓
INNER JOIN
LEFT JOIN
ON vs WHERE

    ↓

SET OPERATIONS
    ↓
UNION
UNION ALL

    ↓

SUBQUERIES
    ↓
Scalar Subquery
Subquery in WHERE
Subquery in FROM
Correlated Subquery
IN
EXISTS
EXISTS vs JOIN

    ↓

AGGREGATIONS
    ↓
COUNT
SUM
AVG
MIN
MAX
COUNT(*) vs COUNT(column)
GROUP BY
HAVING

    ↓

WINDOW FUNCTIONS
    ↓
ROW_NUMBER
RANK
DENSE_RANK
PARTITION BY
ORDER BY

    ↓

INDEXES
    ↓

EXPLAIN / QUERY PLAN
    ↓

TRANSACTIONS
    ↓
ACID
Isolation Levels

    ↓

SQL CODING PROBLEMS
```

---

# 62. SQL Coding Problems We Should Practice

Once the remaining theory is completed, practice in this order:

```text
LEVEL 1
Basic SELECT / WHERE

LEVEL 2
JOIN problems

LEVEL 3
Multiple JOINs

LEVEL 4
GROUP BY + HAVING

LEVEL 5
Subqueries

LEVEL 6
Nth highest salary

LEVEL 7
Duplicate records

LEVEL 8
Top N per department

LEVEL 9
Latest record per entity

LEVEL 10
EXISTS / NOT EXISTS

LEVEL 11
Window function problems

LEVEL 12
Complex interview-style SQL

LEVEL 13
Query optimization / EXPLAIN
```

---

# 63. Quick SQL Mental Models

```text
WHERE
→ Which rows do I want?

GROUP BY
→ How do I group these rows?

HAVING
→ Which groups do I want?

JOIN
→ What data do I want to combine?

ON
→ How should the tables match?

LEFT JOIN
→ Keep everything from my left table.

IN
→ Is this value in that set?

EXISTS
→ Does a matching row exist?

DISTINCT
→ Remove duplicate result rows.

UNION
→ Combine and remove duplicates.

UNION ALL
→ Combine and keep duplicates.

COUNT(*)
→ Count rows.

COUNT(column)
→ Count non-NULL values.

GROUP BY
→ Collapse rows into groups.

Window Function
→ Calculate across related rows while keeping the original rows.

ROW_NUMBER
→ Unique sequential number.

RANK
→ Ties share rank, gaps occur.

DENSE_RANK
→ Ties share rank, no gaps.

PARTITION BY
→ Define groups for a window calculation.

INDEX
→ Help the database locate required rows efficiently.

EXPLAIN
→ Show the database's planned execution strategy.
```
