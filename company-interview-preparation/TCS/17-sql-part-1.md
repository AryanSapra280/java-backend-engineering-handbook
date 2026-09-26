Yes — understood. You **weren't asking me to just answer your friend's 8 questions**. You gave them as a **signal for the level and style of SQL questions TCS is asking**, so we should prepare the **whole SQL/Database section properly**, with these kinds of practical questions included.

We'll do it like this:

### SQL / Database — what we'll cover

1. **SQL fundamentals**

   * PK vs Unique
   * NULL behavior
   * constraints
   * NOT NULL, CHECK, DEFAULT
   * candidate/super keys

2. **Joins**

   * INNER
   * LEFT
   * RIGHT
   * FULL
   * CROSS
   * SELF
   * JOIN conditions
   * `ON` vs `WHERE`
   * practical queries + output prediction

3. **GROUP BY / HAVING**

   * `WHERE` vs `HAVING`
   * aggregate functions
   * common interview queries

4. **Subqueries**

   * correlated/non-correlated
   * `IN`, `EXISTS`
   * `ANY`, `ALL`

5. **Window functions**

   * `ROW_NUMBER`
   * `RANK`
   * `DENSE_RANK`
   * `LAG` / `LEAD`
   * practical interview problems

6. **Indexes**

   * why indexes
   * B-tree
   * composite indexes
   * index column order
   * covering/index-only scans
   * when indexes hurt
   * indexing strategy

7. **Execution plans**

   * `EXPLAIN`
   * `EXPLAIN ANALYZE`
   * Seq Scan
   * Index Scan
   * Bitmap Scan
   * Nested Loop
   * Hash Join
   * Merge Join
   * estimated vs actual rows
   * identifying bottlenecks

8. **Transactions**

   * ACID
   * commit / rollback
   * isolation levels
   * dirty/non-repeatable/phantom reads

9. **Locks & concurrency**

   * row locks
   * table locks
   * optimistic vs pessimistic
   * deadlocks
   * how to troubleshoot them

10. **Query optimization**

    * slow query troubleshooting
    * pagination
    * large result sets
    * N+1
    * avoiding unnecessary columns
    * indexing
    * statistics

11. **PostgreSQL-specific**

    * sequences / identity
    * `SERIAL`
    * `BIGSERIAL`
    * `ON CONFLICT`
    * `RETURNING`
    * JSON/JSONB basics
    * PostgreSQL indexes
    * `LIMIT/OFFSET`
    * `VACUUM` / `ANALYZE`

12. **Practical SQL problems**

    * second highest salary
    * duplicate records
    * employees without department
    * department-wise highest salary
    * nth highest salary
    * duplicate frequency
    * delete duplicates
    * consecutive records
    * top N per group
    * customers with no orders
    * joins + aggregation
    * date-based queries

And importantly, **I'll make you actually write/understand queries**, because your friend's interview shows they may put you in front of an editor and ask you to explain your SQL.

We won't deep-dive into database internals unnecessarily. We'll go **interview depth**: enough to answer follow-ups confidently.

### Let's restart SQL properly.

## 1. SQL Constraints — foundation

Before joins, you should be solid on these because PK/Unique questions commonly lead here.

### What is a constraint?

A constraint is a rule enforced by the database to maintain **data integrity**.

Main constraints:

```text
PRIMARY KEY
FOREIGN KEY
UNIQUE
NOT NULL
CHECK
DEFAULT
```

---

### PRIMARY KEY

Uniquely identifies a row.

```sql
CREATE TABLE employee (
    id BIGINT PRIMARY KEY,
    name VARCHAR(100)
);
```

Properties:

* unique
* cannot be NULL
* one primary-key constraint per table
* can contain multiple columns

Composite:

```sql
PRIMARY KEY (employee_id, department_id)
```

---

### UNIQUE

Ensures uniqueness.

```sql
email VARCHAR(255) UNIQUE
```

A table can have multiple unique constraints.

PostgreSQL allows multiple NULLs in a normal unique constraint.

---

### FOREIGN KEY

Maintains a relationship between tables.

```sql
CREATE TABLE employee (
    id BIGINT PRIMARY KEY,
    department_id BIGINT,

    FOREIGN KEY (department_id)
        REFERENCES department(id)
);
```

This means:

> `employee.department_id` must reference an existing `department.id` (unless the FK column is NULL).

---

### NOT NULL

Column must have a value.

```sql
name VARCHAR(100) NOT NULL
```

---

### CHECK

Restricts allowed values.

```sql
salary NUMERIC CHECK (salary > 0)
```

---

### DEFAULT

Provides a value when one isn't supplied.

```sql
status VARCHAR(20) DEFAULT 'ACTIVE'
```

---

## Very common interview question

### What is the difference between PRIMARY KEY, UNIQUE and FOREIGN KEY?

| Constraint | Purpose                               |
| ---------- | ------------------------------------- |
| PK         | Identifies a row                      |
| UNIQUE     | Prevents duplicate values             |
| FK         | Maintains relationship between tables |

Example:

```text
department
-----------
id PK
name UNIQUE

employee
--------
id PK
email UNIQUE
department_id FK
```

Think:

```text
PK     → "Who am I?"
UNIQUE → "Don't duplicate me."
FK     → "Who do I belong to?"
```

---

# 2. Joins — properly

Suppose:

### employee

| id | name | dept_id |
| -: | ---- | ------: |
|  1 | A    |      10 |
|  2 | B    |      20 |
|  3 | C    |      10 |
|  4 | D    |      30 |

### department

| id | name    |
| -: | ------- |
| 10 | IT      |
| 20 | HR      |
| 40 | Finance |

Now understand the joins through this data.

### INNER JOIN

```sql
SELECT e.name, d.name
FROM employee e
INNER JOIN department d
    ON e.dept_id = d.id;
```

Result:

```text
A  IT
B  HR
C  IT
```

Employee D doesn't appear because department 30 doesn't exist.

Finance doesn't appear because no employee belongs to 40.

**INNER = only matches.**

---

### LEFT JOIN

```sql
SELECT e.name, d.name
FROM employee e
LEFT JOIN department d
    ON e.dept_id = d.id;
```

Result:

```text
A  IT
B  HR
C  IT
D  NULL
```

All employees survive because employee is the **left table**.

**LEFT = keep everything from left.**

---

### RIGHT JOIN

```sql
SELECT e.name, d.name
FROM employee e
RIGHT JOIN department d
    ON e.dept_id = d.id;
```

Result:

```text
A     IT
B     HR
C     IT
NULL  Finance
```

All departments survive.

**RIGHT = keep everything from right.**

---

### FULL OUTER JOIN

```sql
SELECT e.name, d.name
FROM employee e
FULL OUTER JOIN department d
    ON e.dept_id = d.id;
```

Conceptually:

```text
A       IT
B       HR
C       IT
D       NULL
NULL    Finance
```

**FULL = keep everything from both sides.**

---

# The interview trap: RIGHT JOIN

You should be comfortable converting:

```sql
A RIGHT JOIN B
```

into:

```sql
B LEFT JOIN A
```

For example:

```sql
SELECT *
FROM employee e
RIGHT JOIN department d
    ON e.dept_id = d.id;
```

is logically equivalent to:

```sql
SELECT *
FROM department d
LEFT JOIN employee e
    ON e.dept_id = d.id;
```

That's why in real-world code you'll often see **LEFT JOIN more frequently than RIGHT JOIN** — it can make query structure easier to read.

---

# Most important JOIN trap: ON vs WHERE

This is exactly the kind of follow-up I would expect in an interview.

Suppose:

```sql
SELECT *
FROM employee e
LEFT JOIN department d
    ON e.dept_id = d.id
WHERE d.name = 'IT';
```

The `WHERE` condition removes rows where `d.name` is NULL.

So although it's syntactically a LEFT JOIN, the filtering can make the result behave like an inner join for that condition.

Compare:

```sql
SELECT *
FROM employee e
LEFT JOIN department d
    ON e.dept_id = d.id
   AND d.name = 'IT';
```

Here unmatched employees are still preserved.

### Interview line

> "`ON` determines how rows are matched during the join, while `WHERE` filters the result after the join."

That's a **must-remember statement**.

---

We'll continue from here with **GROUP BY + HAVING**, including practical SQL problems and the kind of follow-up questions that can come after each one.
