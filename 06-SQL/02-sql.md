Absolutely. I'll make this **add-on notes only**, continuing from the previous `.md` so you can append it directly. I’ll include the doubts you raised around `IN`, `EXISTS`, how the database conceptually works, and `JOIN` vs `EXISTS`.

````md
# SQL Interview Preparation — Theory Notes (Part 2)

---

# 8. DISTINCT vs GROUP BY

Both `DISTINCT` and `GROUP BY` can sometimes produce the same result, but their purpose is different.

## DISTINCT

`DISTINCT` removes duplicate rows from the result.

Example:

```sql
SELECT DISTINCT department
FROM employee;
````

If the table contains:

```text
IT
IT
HR
HR
HR
FINANCE
```

Result:

```text
IT
HR
FINANCE
```

### Mental Model

```text
DISTINCT
   ↓
Remove duplicate result rows
```

---

## GROUP BY

`GROUP BY` creates groups of rows based on one or more columns.

It is primarily used when we want to perform aggregation.

Example:

```sql
SELECT department, COUNT(*)
FROM employee
GROUP BY department;
```

Result:

```text
IT        2
HR        3
FINANCE   1
```

Here we are not merely removing duplicates.

We are creating groups:

```text
IT
 ↓
All employees belonging to IT
 ↓
COUNT(*)

HR
 ↓
All employees belonging to HR
 ↓
COUNT(*)
```

### Mental Model

```text
DISTINCT
→ Remove duplicates

GROUP BY
→ Create groups
→ Usually followed by aggregation
```

---

## Can DISTINCT and GROUP BY give the same result?

Yes.

These can produce the same result:

```sql
SELECT DISTINCT department
FROM employee;
```

and:

```sql
SELECT department
FROM employee
GROUP BY department;
```

But they communicate different intentions.

### Interview Answer

> "`DISTINCT` is used to remove duplicate rows from the result, whereas `GROUP BY` creates groups, usually so that we can perform aggregate operations such as COUNT, SUM, or AVG."

---

# 9. UNION vs UNION ALL

`UNION` combines the results of two queries and removes duplicates.

```sql
SELECT department
FROM employee_2025

UNION

SELECT department
FROM employee_2026;
```

If the results are:

```text
Query 1:
IT
HR
FINANCE

Query 2:
IT
SALES
HR
```

`UNION` returns:

```text
IT
HR
FINANCE
SALES
```

---

## UNION ALL

```sql
SELECT department
FROM employee_2025

UNION ALL

SELECT department
FROM employee_2026;
```

Result:

```text
IT
HR
FINANCE
IT
SALES
HR
```

Duplicates are retained.

---

## UNION vs UNION ALL

```text
UNION
→ combines results
→ removes duplicates
→ may require additional work for duplicate elimination

UNION ALL
→ combines results
→ keeps duplicates
→ generally faster when duplicate removal is unnecessary
```

### Interview Answer

> "`UNION` combines two result sets and removes duplicates. `UNION ALL` combines them without removing duplicates, so it generally has less overhead."

---

# 10. Requirements for UNION

The two queries do NOT need to have the same column names.

They need:

### 1. Same number of columns

This is valid:

```sql
SELECT employee_id, name
FROM employee

UNION

SELECT department_id, department_name
FROM department;
```

Both return two columns.

This is invalid:

```sql
SELECT employee_id, name
FROM employee

UNION

SELECT department_id
FROM department;
```

First query:

```text
2 columns
```

Second query:

```text
1 column
```

Therefore the UNION is invalid.

---

### 2. Compatible data types

For example:

```text
employee_id → INTEGER
department_id → INTEGER
```

is compatible.

But something like:

```text
employee_id → INTEGER
name        → VARCHAR
```

for corresponding positions would not be appropriate.

### Important

The column names themselves don't need to match.

```text
Query 1              Query 2

employee_id          department_id
name                 department_name
```

This is fine if the corresponding data types are compatible.

---

# 11. IN

`IN` is used when we want to check whether a value belongs to a set of values.

Example:

```sql
SELECT *
FROM employee e
WHERE e.department_id IN (
    SELECT d.department_id
    FROM department d
);
```

Conceptually:

```text
Department table
       ↓
Get department IDs
       ↓
[10, 20, 30]
       ↓
Check each employee
       ↓
Is employee.department_id
inside this set?
```

For example:

```text
Employee        Department ID

Aryan                10
Rahul                40
Amit                 20
```

If the department IDs are:

```text
10, 20, 30
```

Then:

```text
Aryan → 10 IN [10,20,30] → TRUE
Rahul → 40 IN [10,20,30] → FALSE
Amit  → 20 IN [10,20,30] → TRUE
```

---

## Mental Model for IN

```text
IN
→ "Is this value present in this set?"
```

---

# 12. EXISTS

`EXISTS` is used when we only care whether at least one matching row exists.

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

This may initially look confusing because of:

```sql
SELECT 1
```

---

## What does SELECT 1 mean?

We are NOT interested in retrieving department data.

We only want to answer:

```text
Does a matching department row exist?
```

Therefore:

```sql
SELECT 1
```

means approximately:

> "If you find a matching row, I don't care about its actual values. Just give me a constant."

For example:

```sql
SELECT 1
FROM department
WHERE department_id = 10;
```

If department 10 exists, the query finds a row.

The actual value `1` is not important.

The important thing is:

```text
A row exists → TRUE
No row exists → FALSE
```

---

# 13. How EXISTS Works

Consider:

### Employee

```text
emp_id | name  | department_id
--------------------------------
1      | Aryan | 10
2      | Rahul | 40
3      | Amit  | 20
```

### Department

```text
department_id | department_name
--------------------------------
10             | IT
20             | HR
```

Query:

```sql
SELECT *
FROM employee e
WHERE EXISTS (
    SELECT 1
    FROM department d
    WHERE d.department_id = e.department_id
);
```

Conceptually, the database evaluates the condition for an employee.

### Aryan

Outer row:

```text
Aryan → department_id = 10
```

Inner query effectively checks:

```sql
SELECT 1
FROM department d
WHERE d.department_id = 10;
```

Department 10 exists.

Therefore:

```text
EXISTS → TRUE
```

Aryan is kept.

---

### Rahul

Outer row:

```text
Rahul → department_id = 40
```

Inner query checks:

```sql
SELECT 1
FROM department d
WHERE d.department_id = 40;
```

No matching row.

Therefore:

```text
EXISTS → FALSE
```

Rahul is discarded.

---

### Amit

Outer row:

```text
Amit → department_id = 20
```

Inner query checks:

```sql
SELECT 1
FROM department d
WHERE d.department_id = 20;
```

Match exists.

Therefore:

```text
EXISTS → TRUE
```

Amit is kept.

---

# 14. Correlated Subquery

The previous `EXISTS` query is a **correlated subquery**.

Why?

Because the inner query refers to a column from the outer query:

```sql
WHERE d.department_id = e.department_id
                         ↑
                    outer query
```

`e` belongs to:

```sql
FROM employee e
```

which is the outer query.

Therefore the inner query depends on the current outer row.

### Mental Model

```text
Take outer employee
       ↓
Get its department_id
       ↓
Run/check inner condition
       ↓
Does matching department exist?
       ↓
YES → keep employee
NO  → discard employee
```

---

# 15. IN vs EXISTS

This is an important interview distinction.

## IN

Think:

> "Is this value present in a set?"

```sql
SELECT *
FROM employee e
WHERE e.department_id IN (
    SELECT d.department_id
    FROM department d
);
```

Conceptually:

```text
Get department IDs
       ↓
[10, 20, 30]
       ↓
Check employee.department_id
against the set
```

---

## EXISTS

Think:

> "Does a matching row exist?"

```sql
SELECT *
FROM employee e
WHERE EXISTS (
    SELECT 1
    FROM department d
    WHERE d.department_id = e.department_id
);
```

Conceptually:

```text
Take one employee
       ↓
Check whether matching department exists
       ↓
TRUE / FALSE
```

---

## Simple Mental Model

```text
IN
→ Compare against a set

EXISTS
→ Check whether a matching row exists
```

---

# 16. EXISTS vs JOIN

The same business requirement can sometimes be written using a JOIN.

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

### JOIN

```sql
SELECT e.*
FROM employee e
JOIN department d
    ON d.department_id = e.department_id;
```

For normal unique department IDs, these can produce the same employee set.

But they express different intentions.

---

## JOIN means:

> "I want to combine data from these tables."

For example, if I need the department name:

```sql
SELECT e.name, d.department_name
FROM employee e
JOIN department d
    ON d.department_id = e.department_id;
```

Now I actually need data from `department`.

---

## EXISTS means:

> "I don't need any data from the other table. I only need to know whether a matching row exists."

For example:

```sql
SELECT e.*
FROM employee e
WHERE EXISTS (
    SELECT 1
    FROM department d
    WHERE d.department_id = e.department_id
);
```

---

# 17. Important JOIN vs EXISTS Difference

Suppose the department table unexpectedly contains duplicate department IDs:

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
----------------------
1      | 10
```

A JOIN:

```sql
SELECT e.*
FROM employee e
JOIN department d
    ON d.department_id = e.department_id;
```

can produce:

```text
emp_id
------
1
1
```

because there are two matching department rows.

But:

```sql
SELECT e.*
FROM employee e
WHERE EXISTS (
    SELECT 1
    FROM department d
    WHERE d.department_id = e.department_id
);
```

returns:

```text
emp_id
------
1
```

because EXISTS only asks:

```text
Does at least one matching row exist?
```

It does not return multiple copies of the employee because multiple matches exist.

---

# 18. Does EXISTS Always Mean Faster?

Do NOT say:

> "EXISTS is always faster than IN."

That is too absolute.

Modern databases have query optimizers that can transform queries and choose efficient execution strategies.

Performance depends on:

* indexes
* table size
* data distribution
* database optimizer
* statistics
* query structure

A safer interview answer:

> "`EXISTS` can be useful when we only care about whether a related row exists. The database may be able to stop looking once a match is found, but we shouldn't claim that EXISTS is always faster than IN or JOIN. The actual execution plan should be checked."

---

# 19. NULL Basics

`NULL` represents the absence of a value.

Important:

```sql
NULL = NULL
```

does NOT evaluate to TRUE.

To check NULL:

```sql
WHERE column_name IS NULL
```

To check non-NULL:

```sql
WHERE column_name IS NOT NULL
```

Do NOT normally write:

```sql
WHERE column_name = NULL
```

or:

```sql
WHERE column_name != NULL
```

Use:

```sql
IS NULL
IS NOT NULL
```

instead.

---

# 20. LEFT JOIN + NULL

This connects directly to our previous JOIN discussion.

Suppose:

### employee

```text
id | name
---------
1  | Aryan
2  | Rahul
3  | Amit
```

### salary

```text
emp_id | salary
---------------
1      | 100000
2      | 40000
```

Query:

```sql
SELECT *
FROM employee e
LEFT JOIN salary s
    ON e.id = s.emp_id;
```

Result:

```text
Aryan  100000
Rahul   40000
Amit     NULL
```

Amit is preserved because it is a LEFT JOIN.

There is simply no matching salary row, so the right-side columns become NULL.

---

# 21. LEFT JOIN: ON vs WHERE

This is an important concept.

## Condition in ON

```sql
SELECT *
FROM employee e
LEFT JOIN salary s
    ON e.id = s.emp_id
   AND s.salary > 50000;
```

The condition:

```sql
s.salary > 50000
```

controls which salary rows can match.

But the LEFT JOIN still preserves all employees.

Result:

```text
Aryan  100000
Rahul  NULL
Amit   NULL
```

Rahul's salary is 40000, so it doesn't satisfy the ON condition.

But Rahul is still preserved because employee is the left table.

---

## Condition in WHERE

```sql
SELECT *
FROM employee e
LEFT JOIN salary s
    ON e.id = s.emp_id
WHERE s.salary > 50000;
```

The join happens conceptually first.

Then WHERE filters the resulting rows.

Result:

```text
Aryan  100000
```

Rahul is removed because:

```text
40000 > 50000 → FALSE
```

Amit is also removed because:

```text
NULL > 50000
```

does not evaluate to TRUE.

---

# 22. The Most Important Mental Model for ON vs WHERE

```text
ON
→ "Can this right-side row match the left-side row?"

WHERE
→ "After the result is produced, should I keep this row?"
```

For LEFT JOIN:

```text
LEFT JOIN + ON condition
→ left rows are preserved
→ unmatched right side becomes NULL
```

But:

```text
LEFT JOIN + WHERE condition on right table
→ unmatched rows can be filtered out
```

---

# 23. Important Optimization Correction

Do not blindly say:

> "Putting the filter in ON is always faster because it filters before the join."

That is not a reliable interview statement.

The database optimizer can rewrite queries and push predicates down when appropriate.

The primary difference between `ON` and `WHERE` is **semantics**, not guaranteed performance.

Better interview answer:

> "The key difference is semantic. ON controls which rows participate in the join, while WHERE filters the resulting rows. The optimizer may independently push predicates down or change the physical execution plan."

---

# 24. SQL Topics Completed So Far

```text
SQL Logical Execution Order
        ↓
FROM
JOIN / ON
WHERE
GROUP BY
HAVING
SELECT
ORDER BY
LIMIT / OFFSET

        ↓

WHERE vs HAVING

        ↓

INNER JOIN

        ↓

LEFT JOIN

        ↓

ON vs WHERE

        ↓

DISTINCT vs GROUP BY

        ↓

UNION vs UNION ALL

        ↓

UNION column requirements

        ↓

IN

        ↓

EXISTS

        ↓

Correlated Subquery

        ↓

IN vs EXISTS

        ↓

EXISTS vs JOIN

        ↓

Basic NULL handling
```

---

# 25. Quick Interview Cheat Sheet

```text
WHERE
→ Filter rows

GROUP BY
→ Create groups

HAVING
→ Filter groups

DISTINCT
→ Remove duplicate results

INNER JOIN
→ Matching rows only

LEFT JOIN
→ Keep all left rows

ON
→ Controls matching

WHERE
→ Filters final result

UNION
→ Combine + remove duplicates

UNION ALL
→ Combine + keep duplicates

IN
→ Is this value in the set?

EXISTS
→ Does a matching row exist?

JOIN
→ Combine data from tables

EXISTS
→ Check existence without needing right-side data

NULL
→ Absence of value

IS NULL
→ Check for NULL

IS NOT NULL
→ Check for non-NULL
```

---

# 26. What We Should Learn Next

Before starting SQL coding problems, the remaining high-value theory is:

### 1. Subqueries

* Scalar subquery
* Subquery in WHERE
* Subquery in FROM
* Correlated subquery

### 2. Aggregations

* COUNT
* SUM
* AVG
* MIN
* MAX
* COUNT(*) vs COUNT(column)

### 3. Window Functions ⭐

Especially:

```sql
ROW_NUMBER()
RANK()
DENSE_RANK()
```

Important for:

* Nth highest salary
* Top N per department
* Ranking employees
* Finding duplicates
* Latest record per employee

### 4. Indexes ⭐

* What is an index?
* B-tree
* Composite index
* Index on WHERE/JOIN columns
* Why too many indexes are bad
* Index scan vs sequential/full table scan

### 5. EXPLAIN / Query Plan ⭐

* Sequential scan
* Index scan
* Cost
* Rows
* Join strategies
* How to identify a slow query

### 6. Transactions

* ACID
* COMMIT
* ROLLBACK
* READ COMMITTED
* REPEATABLE READ
* SERIALIZABLE

After these, move directly to SQL coding problems.

---

# Final Mental Models

```text
WHERE
"Which rows do I want?"

GROUP BY
"How do I group these rows?"

HAVING
"Which groups do I want?"

JOIN
"What data do I want to combine?"

ON
"How should the tables match?"

LEFT JOIN
"Keep everything from my left table."

IN
"Is this value in that set?"

EXISTS
"Does at least one matching row exist?"

DISTINCT
"Remove duplicate results."

UNION
"Combine and remove duplicates."

UNION ALL
"Combine and keep duplicates."

INDEX
"Can the database find the required rows more efficiently?"

EXPLAIN
"How does the database plan to execute my query?"
```
