Absolutely. Let's keep this as our **SQL Theory Notes** and keep adding to it as we progress. You can paste this directly into a `.md` file.

````md
# SQL Interview Preparation — Theory Notes

## 1. SQL Logical Execution Order

The logical execution order of a SQL query is:

1. `FROM`
2. `JOIN / ON`
3. `WHERE`
4. `GROUP BY`
5. `HAVING`
6. `SELECT`
7. `ORDER BY`
8. `LIMIT / OFFSET`

### Example

```sql
SELECT department_id, COUNT(*)
FROM employee
WHERE salary > 50000
GROUP BY department_id
HAVING COUNT(*) > 5
ORDER BY COUNT(*) DESC
LIMIT 3;
````

Conceptually:

```text
FROM employee
      ↓
WHERE salary > 50000
      ↓
GROUP BY department_id
      ↓
HAVING COUNT(*) > 5
      ↓
SELECT department_id, COUNT(*)
      ↓
ORDER BY
      ↓
LIMIT
```

> Important: This is the **logical** execution order. The database optimizer may physically execute operations in a different order.

---

## 2. Why SELECT Alias Usually Cannot Be Used in WHERE

Example:

```sql
SELECT salary * 12 AS annual_salary
FROM employee
WHERE annual_salary > 1000000;
```

This generally doesn't work because:

```text
WHERE
 ↓
SELECT
```

`WHERE` is logically evaluated before `SELECT`.

The alias `annual_salary` is created during `SELECT`, so it doesn't exist yet when `WHERE` is evaluated.

However, the alias can generally be used in `ORDER BY`:

```sql
SELECT salary * 12 AS annual_salary
FROM employee
ORDER BY annual_salary;
```

### Interview Answer

> "`WHERE` is evaluated before `SELECT`, so a column alias defined in the `SELECT` clause isn't available to `WHERE`. It can generally be used in `ORDER BY` because `ORDER BY` is evaluated after `SELECT`."

---

# 3. WHERE vs HAVING

## WHERE

`WHERE` filters **individual rows before grouping**.

```sql
SELECT *
FROM employee
WHERE salary > 50000;
```

Here, individual employee records are filtered.

## HAVING

`HAVING` filters **groups after GROUP BY**.

It is commonly used with aggregate functions such as:

* `COUNT()`
* `SUM()`
* `AVG()`
* `MAX()`
* `MIN()`

Example:

```sql
SELECT department_id, COUNT(*) AS employee_count
FROM employee
GROUP BY department_id
HAVING COUNT(*) > 5;
```

Conceptually:

```text
Rows
 ↓
WHERE
 ↓
GROUP BY
 ↓
Groups
 ↓
HAVING
 ↓
Filtered groups
```

### Mental Model

```text
WHERE  → filters rows
HAVING → filters groups
```

### Interview Answer

> "`WHERE` filters individual rows before grouping, while `HAVING` filters groups after `GROUP BY`. `HAVING` is commonly used with aggregate functions."

---

## 4. Why Use WHERE Before GROUP BY?

Example:

```sql
SELECT department_id, COUNT(*)
FROM employee
WHERE salary > 50000
GROUP BY department_id
HAVING COUNT(*) > 5;
```

The requirement is:

> Find departments having more than 5 employees whose salary is greater than 50,000.

Therefore:

```text
WHERE salary > 50000
        ↓
Remove unwanted employee rows
        ↓
GROUP BY department_id
        ↓
Count remaining employees
        ↓
HAVING COUNT(*) > 5
```

We use `WHERE` because salary is an attribute of an **individual employee row**.

Then `HAVING` is used because `COUNT(*)` belongs to the **group**.

---

# 5. INNER JOIN vs LEFT JOIN

Suppose:

### employee

| id | name  |
| -- | ----- |
| 1  | Aryan |
| 2  | Rahul |
| 3  | Amit  |

### salary

| emp_id | salary |
| ------ | -----: |
| 1      | 100000 |
| 2      |  80000 |

Amit has no salary record.

---

## INNER JOIN

```sql
SELECT e.name, s.salary
FROM employee e
INNER JOIN salary s
    ON e.id = s.emp_id;
```

Result:

```text
Aryan  100000
Rahul   80000
```

Amit is excluded because there is no matching row in `salary`.

### Mental Model

```text
INNER JOIN
→ Only matching rows
```

---

## LEFT JOIN

```sql
SELECT e.name, s.salary
FROM employee e
LEFT JOIN salary s
    ON e.id = s.emp_id;
```

Result:

```text
Aryan  100000
Rahul   80000
Amit     NULL
```

The left table is preserved.

### Mental Model

```text
LEFT JOIN
→ Keep everything from LEFT table
→ Match from RIGHT table if possible
→ NULL when there is no match
```

### Interview Answer

> "`INNER JOIN` returns only records that have a matching row in both tables. `LEFT JOIN` returns all records from the left table and matching records from the right table. If there is no match, the right-side columns are NULL."

---

# 6. WHERE Condition vs JOIN ON Condition

This is especially important with `LEFT JOIN`.

Consider:

### employee

| id | name  |
| -- | ----- |
| 1  | Aryan |
| 2  | Rahul |
| 3  | Amit  |

### salary

| emp_id | salary |
| ------ | -----: |
| 1      | 100000 |
| 2      |  40000 |

---

## Query A — Condition in WHERE

```sql
SELECT *
FROM employee e
LEFT JOIN salary s
    ON e.id = s.emp_id
WHERE s.salary > 50000;
```

Conceptually:

```text
LEFT JOIN
    ↓
Aryan → 100000
Rahul → 40000
Amit  → NULL
    ↓
WHERE salary > 50000
    ↓
Aryan → 100000
```

The `WHERE` condition removes Rahul and Amit.

Amit has:

```text
s.salary = NULL
```

and:

```text
NULL > 50000
```

doesn't evaluate to TRUE, so the row is removed.

---

## Query B — Condition in ON

```sql
SELECT *
FROM employee e
LEFT JOIN salary s
    ON e.id = s.emp_id
   AND s.salary > 50000;
```

Here the condition determines which right-side rows can match.

```text
Aryan → 100000 → MATCH
Rahul → 40000  → NO MATCH
Amit  → no salary → NO MATCH
```

But because this is still a `LEFT JOIN`, Rahul and Amit are preserved.

Result:

```text
Aryan  100000
Rahul  NULL
Amit   NULL
```

---

## Key Difference

```text
ON
→ Determines which rows from the right table match

WHERE
→ Filters the resulting rows after the join
```

### Very Important

For a `LEFT JOIN`:

```sql
LEFT JOIN ... ON condition
```

can preserve unmatched left-side rows as `NULL`.

But:

```sql
LEFT JOIN ...
WHERE right_table.column = ...
```

can remove those unmatched rows.

Therefore, the two queries are **not necessarily equivalent**.

### Interview Answer

> "For a LEFT JOIN, a condition in the ON clause affects which rows from the right table match while still preserving unmatched rows from the left table. A condition in WHERE is applied after the join and can remove those unmatched rows."

---

## 7. Important Optimization Note

Do **not** automatically say:

> "Putting the filter in ON is faster because it filters during the join."

The database optimizer can rewrite queries and push predicates down when appropriate.

Therefore, the safer interview statement is:

> "The primary difference is semantic — ON controls matching, while WHERE filters the result. The optimizer may independently optimize the physical execution."

---

# Quick Revision

### SQL execution order

```text
FROM
 ↓
JOIN / ON
 ↓
WHERE
 ↓
GROUP BY
 ↓
HAVING
 ↓
SELECT
 ↓
ORDER BY
 ↓
LIMIT / OFFSET
```

### WHERE vs HAVING

```text
WHERE  → rows
HAVING → groups
```

### INNER vs LEFT JOIN

```text
INNER JOIN → matching rows only

LEFT JOIN
→ all left rows
→ matching right rows
→ NULL when no right match
```

### ON vs WHERE

```text
ON
→ controls matching

WHERE
→ filters resulting rows
```

### One-line mental model

```text
ON      = "Can these rows match?"
WHERE   = "Should I keep this resulting row?"
```

```
```
