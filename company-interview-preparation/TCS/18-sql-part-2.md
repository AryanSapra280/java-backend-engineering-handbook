# SQL — GROUP BY, HAVING & Aggregations

This is a **very common interview area**, especially after joins. Let's cover it properly but quickly.

---

## 3. Aggregate Functions

Aggregate functions operate on multiple rows and produce a result.

Common ones:

```sql
COUNT()
SUM()
AVG()
MIN()
MAX()
```

Example:

```sql
SELECT COUNT(*)
FROM employee;
```

Returns total employees.

```sql
SELECT AVG(salary)
FROM employee;
```

Returns average salary.

---

# 4. GROUP BY

### Interview answer

> `GROUP BY` groups rows having the same values so that aggregate functions can be applied separately to each group.

Suppose:

| employee | department | salary |
| -------- | ---------- | -----: |
| A        | IT         |  50000 |
| B        | IT         |  60000 |
| C        | HR         |  40000 |
| D        | HR         |  50000 |

Query:

```sql
SELECT department, COUNT(*)
FROM employee
GROUP BY department;
```

Result:

```text
IT    2
HR    2
```

Another:

```sql
SELECT department, AVG(salary)
FROM employee
GROUP BY department;
```

Result:

```text
IT    55000
HR    45000
```

### Mental model

Without GROUP BY:

```text
ALL ROWS
   ↓
one aggregate result
```

With GROUP BY:

```text
ALL ROWS
   ↓
IT group → aggregate
HR group → aggregate
Finance group → aggregate
```

---

# 5. WHERE vs GROUP BY

Very important.

```sql
SELECT department, AVG(salary)
FROM employee
WHERE salary > 50000
GROUP BY department;
```

The processing conceptually happens as:

```text
FROM
 ↓
WHERE
 ↓
GROUP BY
 ↓
aggregate
 ↓
SELECT
```

So:

> `WHERE` filters individual rows **before grouping**.

Example:

```text
IT: 40k, 50k, 70k
```

With:

```sql
WHERE salary > 50000
```

only:

```text
70k
```

enters the grouping stage.

---

# 6. HAVING

### Interview answer

> `HAVING` filters groups after `GROUP BY`.

Example:

```sql
SELECT department, COUNT(*)
FROM employee
GROUP BY department
HAVING COUNT(*) > 5;
```

Meaning:

> Give me departments having more than 5 employees.

### WHERE vs HAVING

| WHERE                                 | HAVING                            |
| ------------------------------------- | --------------------------------- |
| Filters rows                          | Filters groups                    |
| Before GROUP BY                       | After GROUP BY                    |
| Normally doesn't use aggregate result | Commonly uses aggregate functions |

---

## Classic interview question

### Find departments having average salary greater than 50,000.

```sql
SELECT department_id, AVG(salary)
FROM employee
GROUP BY department_id
HAVING AVG(salary) > 50000;
```

**Don't write:**

```sql
WHERE AVG(salary) > 50000
```

That's invalid because the aggregate is calculated after row filtering/grouping.

---

# 7. WHERE + GROUP BY + HAVING

This is a very good interview query.

### Question

> Find departments where employees earning more than 40,000 have an average salary greater than 60,000.

```sql
SELECT
    department_id,
    AVG(salary) AS avg_salary
FROM employee
WHERE salary > 40000
GROUP BY department_id
HAVING AVG(salary) > 60000;
```

Think:

```text
employee rows
     ↓
salary > 40000
     ↓
GROUP BY department
     ↓
calculate AVG
     ↓
AVG > 60000
```

---

# 8. COUNT(*) vs COUNT(column)

This is an interview trap.

Suppose:

| id | name | manager_id |
| -: | ---- | ---------: |
|  1 | A    |         10 |
|  2 | B    |       NULL |
|  3 | C    |         10 |

### `COUNT(*)`

```sql
SELECT COUNT(*)
FROM employee;
```

Result:

```text
3
```

Counts rows.

### `COUNT(manager_id)`

```sql
SELECT COUNT(manager_id)
FROM employee;
```

Result:

```text
2
```

Because `COUNT(column)` ignores NULL values.

### Remember

```text
COUNT(*)       → rows
COUNT(column)  → non-NULL values
```

---

# 9. COUNT(DISTINCT)

Question:

> How many different departments have employees?

```sql
SELECT COUNT(DISTINCT department_id)
FROM employee;
```

If:

```text
10
10
20
20
30
```

result:

```text
3
```

---

# 10. GROUP BY Multiple Columns

Suppose:

| department | gender | salary |
| ---------- | ------ | -----: |
| IT         | M      |    50k |
| IT         | F      |    60k |
| IT         | M      |    70k |
| HR         | M      |    40k |

Query:

```sql
SELECT department, gender, COUNT(*)
FROM employee
GROUP BY department, gender;
```

Groups by the **combination**:

```text
IT + M
IT + F
HR + M
```

Not separately.

---

# 11. Practical Interview Questions

These are worth practicing.

### Q1. Find number of employees in each department.

```sql
SELECT department_id, COUNT(*) AS employee_count
FROM employee
GROUP BY department_id;
```

---

### Q2. Find departments having more than 10 employees.

```sql
SELECT department_id, COUNT(*) AS employee_count
FROM employee
GROUP BY department_id
HAVING COUNT(*) > 10;
```

---

### Q3. Find maximum salary in each department.

```sql
SELECT department_id, MAX(salary) AS max_salary
FROM employee
GROUP BY department_id;
```

---

### Q4. Find average salary by department.

```sql
SELECT department_id, AVG(salary) AS avg_salary
FROM employee
GROUP BY department_id;
```

---

### Q5. Find duplicate employee names.

```sql
SELECT name, COUNT(*) AS frequency
FROM employee
GROUP BY name
HAVING COUNT(*) > 1;
```

🔥 This is particularly relevant because your friend's Java question was duplicate frequency.

---

# 12. GROUP BY + JOIN

This is where interviews often become slightly harder.

Suppose:

```text
employee
---------
id
name
department_id
salary

department
----------
id
name
```

Question:

> Find the number of employees in each department, including departments with zero employees.

Use `LEFT JOIN`:

```sql
SELECT
    d.id,
    d.name,
    COUNT(e.id) AS employee_count
FROM department d
LEFT JOIN employee e
    ON e.department_id = d.id
GROUP BY d.id, d.name;
```

### Why `COUNT(e.id)` and not `COUNT(*)`?

Because for a department with no employee, the LEFT JOIN still produces a row:

```text
department    employee
HR            NULL
```

`COUNT(*)` would count that joined row.

But:

```sql
COUNT(e.id)
```

doesn't count it because `e.id` is NULL.

This is a **very good interview trap**.

---

# 13. Another Important Query

### Find departments with no employees.

```sql
SELECT d.id, d.name
FROM department d
LEFT JOIN employee e
    ON e.department_id = d.id
WHERE e.id IS NULL;
```

Mental model:

```text
department
    ↓
LEFT JOIN
    ↓
unmatched employee = NULL
    ↓
WHERE employee.id IS NULL
```

This pattern is extremely common.

---

# 14. GROUP BY Interview Trap

Consider:

```sql
SELECT department_id, name, COUNT(*)
FROM employee
GROUP BY department_id;
```

Is this valid?

**Generally no.**

Why?

Because `name` is neither:

* grouped
* nor aggregated

Correct:

```sql
SELECT department_id, name, COUNT(*)
FROM employee
GROUP BY department_id, name;
```

Or if you don't need name:

```sql
SELECT department_id, COUNT(*)
FROM employee
GROUP BY department_id;
```

---

# 15. SQL Logical Execution Order

🔥 **Memorize this for interviews.**

A simplified logical order is:

```text
FROM
JOIN
WHERE
GROUP BY
HAVING
SELECT
DISTINCT
ORDER BY
LIMIT/OFFSET
```

For example:

```sql
SELECT department_id, COUNT(*)
FROM employee
WHERE salary > 50000
GROUP BY department_id
HAVING COUNT(*) > 5
ORDER BY COUNT(*) DESC
LIMIT 10;
```

Conceptually:

```text
FROM employee
      ↓
WHERE salary > 50000
      ↓
GROUP BY department_id
      ↓
COUNT(*)
      ↓
HAVING COUNT(*) > 5
      ↓
SELECT
      ↓
ORDER BY
      ↓
LIMIT
```

This helps tremendously when explaining SQL.

---

## 🔥 Interview Rapid Fire

### `WHERE` or `HAVING` for salary > 50k?

`WHERE`.

### `WHERE` or `HAVING` for departments with COUNT(*) > 5?

`HAVING`.

### Can GROUP BY exist without aggregate?

**Yes.**

```sql
SELECT department_id
FROM employee
GROUP BY department_id;
```

It effectively gives distinct groups.

### Can aggregate exist without GROUP BY?

**Yes.**

```sql
SELECT MAX(salary)
FROM employee;
```

Produces one result for the entire table.

### Does `COUNT(column)` count NULL?

**No.**

### Does `COUNT(*)` count NULL rows?

**Yes**, because it counts rows.

### What is the difference between DISTINCT and GROUP BY?

`DISTINCT` removes duplicate result rows.

`GROUP BY` forms groups, generally for aggregation.

---

## ✅ GROUP BY / HAVING done

Next we'll move to **Subqueries + EXISTS/IN + correlated subqueries**, and then into **Window Functions**. Those are very common practical SQL interview questions.
