# SQL — Window Functions

This is a **high-value interview topic**. Don't just memorize `ROW_NUMBER()`, `RANK()`, etc. Understand the pattern:

> **GROUP BY reduces rows. Window functions keep the rows and calculate something across related rows.**

---

# 1. What is a Window Function?

A window function performs a calculation across a set of rows related to the current row **without collapsing those rows**.

Example:

```sql
SELECT
    name,
    department_id,
    salary,
    AVG(salary) OVER (PARTITION BY department_id) AS dept_avg
FROM employee;
```

Suppose:

| name | dept | salary |
| ---- | ---: | -----: |
| A    |   10 |    50K |
| B    |   10 |    70K |
| C    |   20 |    40K |

Result:

| name | dept | salary | dept_avg |
| ---- | ---: | -----: | -------: |
| A    |   10 |    50K |      60K |
| B    |   10 |    70K |      60K |
| C    |   20 |    40K |      40K |

Notice:

**GROUP BY** would give one row per department.

Window function keeps **every employee row**.

---

# 2. `PARTITION BY`

`PARTITION BY` divides the result into logical groups.

```sql
AVG(salary) OVER (
    PARTITION BY department_id
)
```

means:

> Calculate the average separately for each department.

Think:

```text
All employees
      ↓
PARTITION BY department
      ↓
IT group     HR group     Finance group
```

But unlike `GROUP BY`, the individual rows remain.

---

# 3. `ROW_NUMBER()`

Assigns a unique sequential number to each row.

```sql
SELECT
    name,
    salary,
    ROW_NUMBER() OVER (
        ORDER BY salary DESC
    ) AS rn
FROM employee;
```

Result:

| name | salary | rn |
| ---- | -----: | -: |
| A    |   100K |  1 |
| B    |    90K |  2 |
| C    |    90K |  3 |
| D    |    80K |  4 |

Even if salaries tie, `ROW_NUMBER()` gives different numbers.

---

# 4. `RANK()`

```sql
SELECT
    name,
    salary,
    RANK() OVER (
        ORDER BY salary DESC
    ) AS ranking
FROM employee;
```

Result:

| name | salary | rank |
| ---- | -----: | ---: |
| A    |   100K |    1 |
| B    |    90K |    2 |
| C    |    90K |    2 |
| D    |    80K |    4 |

Notice the gap:

```text
1
2
2
4
```

Because two employees occupy rank 2.

---

# 5. `DENSE_RANK()`

```sql
SELECT
    name,
    salary,
    DENSE_RANK() OVER (
        ORDER BY salary DESC
    ) AS ranking
FROM employee;
```

Result:

| name | salary | rank |
| ---- | -----: | ---: |
| A    |   100K |    1 |
| B    |    90K |    2 |
| C    |    90K |    2 |
| D    |    80K |    3 |

No gap.

### Memorize this:

```text
ROW_NUMBER
1
2
3
4

RANK
1
2
2
4

DENSE_RANK
1
2
2
3
```

🔥 This is one of the most common SQL interview questions.

---

# 6. `PARTITION BY` + Ranking

Now the more realistic problem:

> Rank employees within each department.

```sql
SELECT
    name,
    department_id,
    salary,
    RANK() OVER (
        PARTITION BY department_id
        ORDER BY salary DESC
    ) AS ranking
FROM employee;
```

Example:

| name | dept | salary | rank |
| ---- | ---: | -----: | ---: |
| A    |   10 |   100K |    1 |
| B    |   10 |    80K |    2 |
| C    |   10 |    80K |    2 |
| D    |   20 |    90K |    1 |
| E    |   20 |    70K |    2 |

The ranking **restarts for every department**.

---

# 7. Classic Interview Question: Second Highest Salary

There are multiple ways.

### Using `DENSE_RANK()`

```sql
SELECT *
FROM (
    SELECT
        e.*,
        DENSE_RANK() OVER (
            ORDER BY salary DESC
        ) AS rnk
    FROM employee e
) x
WHERE rnk = 2;
```

Why `DENSE_RANK()`?

Suppose:

```text
100K
90K
90K
80K
```

Ranks:

```text
100K → 1
90K  → 2
90K  → 2
80K  → 3
```

Therefore both employees earning 90K are returned.

---

# 8. Top 3 Salaries in Each Department

This is **extremely interview-worthy**.

Question:

> Find the top 3 salary levels in every department.

```sql
SELECT *
FROM (
    SELECT
        e.*,
        DENSE_RANK() OVER (
            PARTITION BY department_id
            ORDER BY salary DESC
        ) AS rnk
    FROM employee e
) x
WHERE rnk <= 3;
```

The important pattern is:

```text
PARTITION BY department
ORDER BY salary DESC
```

Then filter the ranking.

---

# 9. Why can't we simply use WHERE with the Window Function?

This is invalid:

```sql
SELECT
    name,
    salary,
    RANK() OVER (ORDER BY salary DESC) AS rnk
FROM employee
WHERE rnk <= 3;
```

Why?

Because the window function is calculated after the `WHERE` phase in the logical query processing order.

So we put it in a subquery/CTE first:

```sql
SELECT *
FROM (
    SELECT
        name,
        salary,
        RANK() OVER (ORDER BY salary DESC) AS rnk
    FROM employee
) x
WHERE rnk <= 3;
```

### PostgreSQL alternative

PostgreSQL supports:

```sql
QUALIFY
```

in some modern SQL systems, but **PostgreSQL does not traditionally support `QUALIFY`**, so for PostgreSQL interview questions, use a subquery or CTE.

---

# 10. `LAG()`

`LAG()` gets a value from a previous row.

Example:

```sql
SELECT
    transaction_date,
    amount,
    LAG(amount) OVER (
        ORDER BY transaction_date
    ) AS previous_amount
FROM transactions;
```

Result:

| date  | amount | previous |
| ----- | -----: | -------: |
| Jan 1 |    100 |     NULL |
| Jan 2 |    150 |      100 |
| Jan 3 |    120 |      150 |

Very useful for:

* comparing current vs previous transaction
* detecting changes
* calculating differences
* time-series analysis

---

# 11. `LEAD()`

Opposite of `LAG()`.

```sql
SELECT
    transaction_date,
    amount,
    LEAD(amount) OVER (
        ORDER BY transaction_date
    ) AS next_amount
FROM transactions;
```

```text
LAG  → previous row
LEAD → next row
```

---

# 12. Running Total

Another common practical question.

Suppose:

| date  | amount |
| ----- | -----: |
| Jan 1 |    100 |
| Jan 2 |    200 |
| Jan 3 |    150 |

Query:

```sql
SELECT
    transaction_date,
    amount,
    SUM(amount) OVER (
        ORDER BY transaction_date
    ) AS running_total
FROM transactions;
```

Result:

| date  | amount | running total |
| ----- | -----: | ------------: |
| Jan 1 |    100 |           100 |
| Jan 2 |    200 |           300 |
| Jan 3 |    150 |           450 |

---

# 13. Difference from Previous Row

Using `LAG()`:

```sql
SELECT
    transaction_date,
    amount,
    amount - LAG(amount) OVER (
        ORDER BY transaction_date
    ) AS difference
FROM transactions;
```

Result:

| date  | amount | difference |
| ----- | -----: | ---------: |
| Jan 1 |    100 |       NULL |
| Jan 2 |    200 |        100 |
| Jan 3 |    150 |        -50 |

---

# 14. Window Function vs GROUP BY

This is a **must-know conceptual question**.

### GROUP BY

```sql
SELECT department_id, AVG(salary)
FROM employee
GROUP BY department_id;
```

Result:

```text
department  avg_salary
10          60000
20          50000
```

One row per group.

### Window function

```sql
SELECT
    name,
    department_id,
    salary,
    AVG(salary) OVER (
        PARTITION BY department_id
    ) AS avg_salary
FROM employee;
```

Result:

```text
A  10  50000  60000
B  10  70000  60000
C  20  50000  50000
```

Every row remains.

### Interview answer

> "`GROUP BY` collapses rows into groups, while a window function performs calculations across related rows while retaining the individual rows."

🔥 Memorize that.

---

# 15. Window Function Syntax

General structure:

```sql
FUNCTION(...) OVER (
    PARTITION BY ...
    ORDER BY ...
)
```

For example:

```sql
RANK() OVER (
    PARTITION BY department_id
    ORDER BY salary DESC
)
```

Think:

```text
PARTITION BY → which group?
ORDER BY     → in what order?
FUNCTION     → what calculation?
```

---

# 16. Very Common Practical Questions

### Find highest-paid employee in each department

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
) x
WHERE rn = 1;
```

### Why `ROW_NUMBER()` here?

If there are two employees tied for highest salary and you want **exactly one employee**, `ROW_NUMBER()` gives one row.

If you want **all employees tied for highest salary**, use:

```sql
DENSE_RANK()
```

and:

```sql
WHERE rnk = 1
```

That's an excellent follow-up distinction.

---

# 17. Find Duplicate Records Using Window Function

Suppose duplicate emails exist:

```text
id | email
1    a@gmail.com
2    b@gmail.com
3    a@gmail.com
```

You can identify duplicates:

```sql
SELECT *
FROM (
    SELECT
        e.*,
        ROW_NUMBER() OVER (
            PARTITION BY email
            ORDER BY id
        ) AS rn
    FROM employee e
) x
WHERE rn > 1;
```

This returns the **extra copies**, while keeping the first occurrence.

---

# 18. Window Function Interview Rapid Fire

### `ROW_NUMBER()` vs `RANK()`?

`ROW_NUMBER()` always gives unique sequential numbers.

`RANK()` gives the same rank for ties and leaves gaps.

### `RANK()` vs `DENSE_RANK()`?

`RANK()` leaves gaps after ties.

`DENSE_RANK()` doesn't.

### What does `PARTITION BY` do?

Divides rows into independent groups for the window calculation.

### Does window function reduce rows?

**No.**

### Does GROUP BY reduce rows?

**Yes, generally.**

### `LAG()`?

Previous row.

### `LEAD()`?

Next row.

### How do you get top N per group?

```text
PARTITION BY group
ORDER BY metric DESC
→ ROW_NUMBER/RANK/DENSE_RANK
→ filter rank
```

---

# 🔥 One pattern you should remember for the interview

If they give you:

> "Find top 2 employees by salary from each department."

Immediately think:

```sql
SELECT *
FROM (
    SELECT
        e.*,
        DENSE_RANK() OVER (
            PARTITION BY department_id
            ORDER BY salary DESC
        ) AS rnk
    FROM employee e
) x
WHERE rnk <= 2;
```

That pattern solves a **huge number of interview questions**.

---

## ✅ Window Functions DONE

Next: **Indexes** — this is especially important for you because interviewers can connect it directly to your PostgreSQL + Spring/JPA experience:

**Why indexes → B-tree → composite indexes → column order → index scan vs sequential scan → covering indexes → when indexes hurt → practical indexing questions.**


Need to cover
Why indexes → B-tree → composite indexes → column order → index scan vs sequential scan → covering indexes → when indexes hurt → practical indexing questions.