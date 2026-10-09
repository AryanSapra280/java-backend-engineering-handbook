Next up: **Transaction Isolation**. This is a very common Senior Java/Spring interview area, and the best way to understand it is through concurrent transactions rather than memorizing definitions.

# 4. Transaction Isolation ⭐⭐⭐⭐⭐

## 1. What is transaction isolation?

Isolation defines:

> **How much one transaction is isolated from the changes made by other concurrent transactions.**

Imagine two transactions running at the same time:

```text
T1 → updating an account
T2 → reading that account
```

The question is:

> What should T2 be allowed to see while T1 is working?

That's what isolation controls.

---

# 2. Why do we need isolation?

Consider:

```text
Account balance = ₹10,000
```

Transaction T1:

```text
T1:
Read balance = 10,000
Deduct 2,000
Write 8,000
```

At the same time T2 reads the balance.

Depending on the isolation level, T2 might see:

```text
10,000
```

or:

```text
8,000
```

or potentially encounter blocking depending on the database and operation.

So isolation controls **concurrent transaction visibility and consistency**.

---

# 3. The three classic read anomalies

You absolutely need to know these:

```text
1. Dirty Read
2. Non-repeatable Read
3. Phantom Read
```

Let's understand each with T1 and T2.

---

# 4. Dirty Read

A **dirty read** occurs when one transaction reads data written by another transaction **before that transaction commits**.

Example:

Initial balance:

```text
₹10,000
```

T1:

```text
T1:
UPDATE balance = 8,000
```

But T1 has **not committed** yet.

T2:

```text
T2:
SELECT balance
```

If T2 sees:

```text
₹8,000
```

it has read **uncommitted data**.

Then T1 rolls back:

```text
T1:
ROLLBACK
```

Actual balance returns to:

```text
₹10,000
```

But T2 previously saw:

```text
₹8,000
```

That's a:

> **Dirty Read**

---

# 5. Visualizing dirty read

```text
T1                         T2
│                          │
│ UPDATE balance=8000      │
│                          │
│      (not committed)     │
│                          │
│                          │ SELECT
│                          │ ↓
│                          │ sees 8000
│                          │
│ ROLLBACK                 │
│                          │
│ balance = 10000          │
```

T2 saw something that ultimately never became committed state.

---

# 6. Which isolation level allows dirty reads?

Potentially:

```text
READ_UNCOMMITTED
```

It is the least restrictive standard isolation level.

At this level, a transaction may be allowed to read uncommitted changes from other transactions.

---

# 7. Does `READ_COMMITTED` allow dirty reads?

No.

`READ_COMMITTED` ensures that a transaction does not read uncommitted changes from another transaction.

So:

```text
READ_UNCOMMITTED
→ Dirty reads possible

READ_COMMITTED
→ Dirty reads prevented
```

---

# 8. Non-repeatable Read

Now suppose T1 reads the same row twice.

Initially:

```text
balance = ₹10,000
```

T1:

```text
SELECT balance
→ ₹10,000
```

Then T2 changes and commits:

```text
T2:
UPDATE balance = ₹8,000
COMMIT
```

T1 reads the same row again:

```text
SELECT balance
→ ₹8,000
```

The same query, within T1, returned different values.

That's:

> **Non-repeatable Read**

---

# 9. Visualizing non-repeatable read

```text
T1                         T2
│                          │
│ SELECT balance           │
│ → 10000                  │
│                          │
│                          │ UPDATE balance=8000
│                          │ COMMIT
│                          │
│ SELECT balance           │
│ → 8000                   │
```

T1 couldn't reproduce the same result for the same row.

---

# 10. Which isolation level prevents non-repeatable reads?

`READ_COMMITTED` prevents dirty reads but does **not necessarily prevent non-repeatable reads**.

Higher isolation such as:

```text
REPEATABLE_READ
```

is designed to prevent this anomaly.

Conceptually:

```text
READ_COMMITTED
→ Dirty read prevented
→ Non-repeatable read possible

REPEATABLE_READ
→ Dirty read prevented
→ Non-repeatable read prevented
```

The exact implementation depends on the database's concurrency model.

---

# 11. Phantom Read

This one is slightly different.

A phantom read occurs when the **set of rows matching a query changes** because another transaction inserts, deletes, or otherwise changes rows that satisfy the query.

Suppose:

```text
Orders:

id   status
1    PENDING
2    PENDING
3    COMPLETED
```

T1:

```sql
SELECT * FROM orders
WHERE status = 'PENDING';
```

Result:

```text
1
2
```

Then T2 inserts:

```text
id = 4
status = PENDING
```

and commits.

T1 executes the same query again:

```sql
SELECT * FROM orders
WHERE status = 'PENDING';
```

Now:

```text
1
2
4
```

The new row is a **phantom**.

---

# 12. Visualizing phantom reads

```text
T1                              T2
│                               │
│ SELECT PENDING               │
│ → rows 1,2                   │
│                               │
│                               │ INSERT row 4
│                               │ status=PENDING
│                               │ COMMIT
│                               │
│ SELECT PENDING               │
│ → rows 1,2,4                 │
```

The original rows weren't necessarily modified.

The **result set changed**.

That's the key distinction.

---

# 13. Dirty vs non-repeatable vs phantom

Memorize this table:

| Anomaly | What changes? |
|---|---|
| Dirty Read | You read uncommitted data |
| Non-repeatable Read | Same row gives different value |
| Phantom Read | Same query returns a different set of rows |

Simple mental model:

```text
Dirty
→ uncommitted value

Non-repeatable
→ same row changed

Phantom
→ new/disappearing matching rows
```

---

# 14. Spring isolation levels

Spring provides:

```java id="c1b42n"
Isolation.DEFAULT
Isolation.READ_UNCOMMITTED
Isolation.READ_COMMITTED
Isolation.REPEATABLE_READ
Isolation.SERIALIZABLE
```

You can specify it using:

```java id="h8t4xk"
@Transactional(
    isolation = Isolation.READ_COMMITTED
)
public void processPayment() {
}
```

---

# 15. `Isolation.DEFAULT`

```java id="g3ihg4"
@Transactional(isolation = Isolation.DEFAULT)
```

means:

> Use the database's default isolation level.

This is usually the safest practical starting point unless your application has a specific consistency requirement.

Important:

**`DEFAULT` does not mean a universal Spring isolation level.**

It delegates to the underlying database configuration.

---

# 16. `READ_UNCOMMITTED`

This is the least restrictive standard level.

Conceptually:

```text
Dirty reads        → possible
Non-repeatable     → possible
Phantom reads      → possible
```

It provides the weakest isolation.

Example:

```java id="9q1v0r"
@Transactional(
    isolation = Isolation.READ_UNCOMMITTED
)
public void readData() {
}
```

You might get better concurrency, but at the cost of consistency.

For financial systems such as payment or ledger processing, this would generally be inappropriate.

---

# 17. `READ_COMMITTED`

This ensures a transaction doesn't read another transaction's uncommitted changes.

Conceptually:

```text
Dirty reads        → prevented
Non-repeatable     → possible
Phantom reads      → possible
```

Example:

```java id="9a3o8y"
@Transactional(
    isolation = Isolation.READ_COMMITTED
)
public void processPayment() {
}
```

This is a commonly used isolation level.

---

# 18. `REPEATABLE_READ`

The goal is:

> If a transaction reads a row, another transaction shouldn't cause that row's committed value to appear differently during the transaction.

Conceptually:

```text
Dirty reads        → prevented
Non-repeatable     → prevented
Phantom reads      → database-dependent
```

That last point is important.

Don't blindly say:

> "`REPEATABLE_READ` always prevents phantom reads."

The SQL standard and actual database implementations can differ.

For example, database engines may use different locking or MVCC strategies.

---

# 19. `SERIALIZABLE`

This provides the strongest standard isolation semantics.

Conceptually, concurrent transactions behave as though they were executed serially.

```text
T1
 ↓
complete
 ↓
T2
 ↓
complete
```

rather than allowing conflicting operations to freely interleave.

It provides the strongest consistency guarantees among the standard isolation levels, but generally reduces concurrency and can increase:

```text
locking
contention
waiting
deadlocks/retries
latency
```

So:

> Strongest isolation does not automatically mean best performance.

---

# 20. Isolation level comparison

For interview purposes, remember the standard conceptual table:

| Isolation | Dirty Read | Non-repeatable Read | Phantom Read |
|---|---|---|---|
| READ_UNCOMMITTED | Possible | Possible | Possible |
| READ_COMMITTED | Prevented | Possible | Possible |
| REPEATABLE_READ | Prevented | Prevented | Depends on DB/implementation |
| SERIALIZABLE | Prevented | Prevented | Prevented |

This is the standard conceptual model.

---

# 21. The important trade-off

Think of isolation as:

```text
More isolation
      ↓
More consistency
      ↓
Potentially less concurrency
```

and:

```text
Less isolation
      ↓
More concurrency
      ↓
Potentially more anomalies
```

So don't say:

> "Always use SERIALIZABLE because it is safest."

That's not a good production answer.

---

# 22. Production example — payment system

Imagine two concurrent payments against the same account.

Initial balance:

```text
₹10,000
```

Two requests arrive simultaneously:

```text
T1 → withdraw ₹7,000
T2 → withdraw ₹7,000
```

If concurrency isn't controlled properly, both might read:

```text
₹10,000
```

and conclude:

```text
₹10,000 >= ₹7,000
```

Both proceed.

That can result in an invalid final state.

This is why financial systems often need carefully designed concurrency control.

And this is where simply knowing isolation levels isn't enough.

You may also need:

- row locking
- optimistic locking
- version columns
- atomic SQL updates
- appropriate transaction boundaries

For example:

```sql
UPDATE account
SET balance = balance - 7000
WHERE id = ?
  AND balance >= 7000;
```

Then check the affected row count.

That can be safer than blindly reading and then updating.

---

# 23. Isolation vs locking

These are related but not identical concepts.

Isolation is the **transaction-level consistency model**.

Locking is one mechanism databases can use to implement concurrency control.

Modern databases may also use:

```text
MVCC
```

(Multi-Version Concurrency Control).

So don't explain isolation as simply:

> "It puts locks on everything."

That's too simplistic.

---

# 24. Isolation vs propagation

Very important interview distinction:

### Propagation

Answers:

> "What happens when a transactional method calls another transactional method?"

```text
REQUIRED
REQUIRES_NEW
NESTED
...
```

### Isolation

Answers:

> "How does this transaction interact with concurrent transactions?"

```text
READ_COMMITTED
REPEATABLE_READ
SERIALIZABLE
...
```

So:

```java id="p1v3dx"
@Transactional(
    propagation = Propagation.REQUIRES_NEW,
    isolation = Isolation.READ_COMMITTED
)
```

means:

```text
Propagation
→ create independent transaction if necessary

Isolation
→ use READ_COMMITTED concurrency semantics
```

They solve different problems.

---

# 25. Interview Question: Which isolation level would you use for a payment system?

Don't immediately answer:

> `SERIALIZABLE`.

A better answer is:

> "I would first identify the consistency requirement and concurrency pattern. I would normally start with the database's appropriate default, often `READ_COMMITTED`, and use explicit locking, optimistic locking, or atomic updates where required. If the business operation genuinely requires serializable semantics, I would consider `SERIALIZABLE`, but I'd evaluate the throughput, contention, latency, and deadlock implications."

That's a much stronger senior-level answer.

---

# 26. Interview Question: Does higher isolation guarantee no deadlocks?

No.

In fact, stronger isolation can **increase contention** and potentially increase the likelihood of blocking and deadlock scenarios depending on the workload and database implementation.

For example:

```text
T1 locks A
T2 locks B

T1 wants B
T2 wants A
```

Now:

```text
T1 → waiting for B
T2 → waiting for A
```

This can create a deadlock.

Applications often need appropriate retry handling for transient database deadlocks.

---

# 27. Interview Question: Is `READ_COMMITTED` always the same across databases?

No.

The isolation level name is standardized conceptually, but implementation details vary between databases.

Different databases use different combinations of:

```text
locking
MVCC
snapshot mechanisms
```

Therefore:

> The same isolation-level name doesn't mean every database behaves identically in every edge case.

This is especially important when discussing `REPEATABLE_READ` and phantom behavior.

---

# 28. Setting isolation in Spring

Example:

```java id="xq9c0w"
@Service
public class PaymentService {

    @Transactional(
        isolation = Isolation.READ_COMMITTED
    )
    public void processPayment() {

        // database operations
    }
}
```

You can combine isolation with propagation:

```java id="6nd4kp"
@Transactional(
    propagation = Propagation.REQUIRES_NEW,
    isolation = Isolation.READ_COMMITTED
)
public void auditPayment() {
}
```

Again:

```text
propagation → transaction relationship
isolation   → concurrent visibility
```

---

# 29. Important production warning

Don't set isolation at random.

For example:

```java
@Transactional(isolation = Isolation.SERIALIZABLE)
```

may sound safer, but if your service processes:

```text
10,000 requests/sec
```

and every transaction becomes highly restrictive, you can create:

```text
high contention
↓
waiting
↓
latency
↓
timeouts
↓
reduced throughput
```

Transaction isolation is a **business consistency decision plus a database performance decision**.

---

# 30. Interview scenario: balance transfer

### Interviewer:

> Two users simultaneously withdraw money from the same account. How would you make the operation safe?

A strong answer shouldn't just say:

> "Use SERIALIZABLE."

Instead:

> "I would define the transfer as a transaction and then choose an appropriate concurrency-control mechanism based on the database and workload. For a balance update, I could use pessimistic row locking, optimistic locking with a version column, or an atomic conditional update such as `UPDATE ... SET balance = balance - ? WHERE balance >= ?`. The choice depends on contention and performance requirements. Isolation level is part of the solution, but I wouldn't rely on isolation alone without considering the actual update strategy."

That's the kind of answer expected from a senior engineer.

---

# 31. Interview scenario: Why not use SERIALIZABLE everywhere?

Answer:

> "Serializable provides the strongest standard isolation semantics, but it can significantly reduce concurrency because conflicting transactions may need to wait or retry. This can increase latency, contention, and deadlocks. Therefore I would use the weakest isolation level that still satisfies the business consistency requirements, combined with appropriate locking or optimistic concurrency control."

Excellent interview answer.

---

# 32. One thing to memorize

Think of the isolation levels as a ladder:

```text
READ_UNCOMMITTED
       ↓
READ_COMMITTED
       ↓
REPEATABLE_READ
       ↓
SERIALIZABLE
```

Moving downward in concurrency:

```text
Consistency ↑
Concurrency ↓
```

But remember:

> The exact behavior is database-dependent.

---

# 33. Interview-ready answer

### Q: Explain transaction isolation levels.

> Transaction isolation defines how concurrently executing transactions interact with each other's changes. The main anomalies are dirty reads, non-repeatable reads, and phantom reads. `READ_UNCOMMITTED` provides the weakest isolation and may allow dirty reads. `READ_COMMITTED` prevents dirty reads but can allow non-repeatable and phantom reads. `REPEATABLE_READ` prevents dirty and non-repeatable reads, while phantom behavior depends on the database implementation. `SERIALIZABLE` provides the strongest standard isolation semantics but can reduce concurrency and increase contention. In production, I would choose the weakest isolation that satisfies the business consistency requirements and combine it with appropriate locking or optimistic concurrency where necessary.

---

# 34. The mental model

When you hear:

> "Isolation?"

Immediately think:

```text
Concurrent Transactions
        ↓
What can T2 see from T1?
        ↓
Dirty Read?
        ↓
Non-repeatable Read?
        ↓
Phantom Read?
        ↓
Choose isolation level
```

Then:

```text
READ_UNCOMMITTED
→ dirty possible

READ_COMMITTED
→ dirty prevented

REPEATABLE_READ
→ same row remains stable

SERIALIZABLE
→ strongest standard isolation
```

And always remember:

```text
Propagation ≠ Isolation
```

---

# Next: Rollback Rules

The next transaction topic is:

### 5. Rollback Rules

We'll cover:

- Checked vs unchecked exceptions
- Why `RuntimeException` rolls back by default
- Why checked exceptions normally don't
- `rollbackFor`
- `noRollbackFor`
- `rollbackForClassName`
- What happens when exceptions are caught
- `rollback-only`
- `UnexpectedRollbackException`
- Practical payment/order examples
- Senior-level interview traps

This is another **very frequently asked `@Transactional` area**.