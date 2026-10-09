Exactly. Let's go straight into **JPA Concurrency & Locking** — this is a very important Senior Engineer area because it tests whether you understand what happens when **two requests modify the same database row simultaneously**.

# 4. Spring Data JPA / Hibernate ⭐⭐⭐⭐⭐
## Part 7 — Concurrency, Optimistic & Pessimistic Locking

---

# 1. The Core Problem: Concurrent Updates

Imagine a payment/account system:

```text
Account balance = ₹10,000
```

Two requests arrive almost simultaneously.

```text
Request A → withdraw ₹7,000
Request B → withdraw ₹6,000
```

Both read:

```text
₹10,000
```

Then both calculate:

```text
A: 10,000 - 7,000 = 3,000
B: 10,000 - 6,000 = 4,000
```

If both update the row without proper concurrency control, the final database state may be incorrect.

This is a **lost update / concurrency problem**.

---

# 2. What is a Lost Update?

### Interview Question

**What is a lost update?**

A lost update occurs when two transactions read the same data, both modify it, and one update overwrites the other's change.

Example:

```text
Initial balance = 10,000

Transaction A reads 10,000
Transaction B reads 10,000

A calculates 3,000
B calculates 4,000

A writes 3,000
B writes 4,000
```

Final:

```text
4,000
```

A's update has effectively been lost.

---

# 3. Why Doesn't `@Transactional` Automatically Solve This?

### Important interview question

Suppose:

```java
@Transactional
public void withdraw(...) {
    Account account = repository.findById(id).orElseThrow();

    account.setBalance(
        account.getBalance().subtract(amount)
    );
}
```

Two transactions can still execute concurrently.

`@Transactional` gives you a transaction boundary.

It does **not automatically mean**:

> "Nobody else can modify this row while I'm working on it."

Concurrency control requires appropriate:

- isolation
- locking
- optimistic concurrency
- database constraints
- application design

depending on the problem.

---

# 4. Two Major Locking Strategies

JPA commonly gives us:

```text
Optimistic Locking
Pessimistic Locking
```

Mental model:

```text
Optimistic
=
"Conflicts are relatively rare.
Let transactions proceed and detect conflict."

Pessimistic
=
"Conflict is possible.
Lock the database row while working."
```

---

# 5. Optimistic Locking ⭐⭐⭐⭐⭐

### Interview Question

**What is optimistic locking?**

Optimistic locking assumes concurrent modification conflicts are relatively uncommon.

Instead of locking the row when it is read, we detect whether somebody else modified it before committing our change.

JPA commonly implements this using:

```java
@Version
```

---

# 6. `@Version`

Example:

```java
@Entity
public class Account {

    @Id
    private Long id;

    private BigDecimal balance;

    @Version
    private Long version;
}
```

Suppose the database contains:

```text
id = 10
balance = 10000
version = 5
```

Transaction A reads:

```text
balance = 10000
version = 5
```

Transaction B also reads:

```text
balance = 10000
version = 5
```

A updates first.

Hibernate conceptually generates something similar to:

```sql
UPDATE account
SET balance = ?,
    version = 6
WHERE id = 10
  AND version = 5;
```

The update succeeds.

Database:

```text
version = 6
```

---

# 7. What Happens to Transaction B?

B still has:

```text
version = 5
```

It attempts:

```sql
UPDATE account
SET balance = ?,
    version = 6
WHERE id = 10
  AND version = 5;
```

But the database row is now:

```text
version = 6
```

Therefore:

```text
WHERE version = 5
```

matches zero rows.

Hibernate detects that the expected update didn't happen and reports an optimistic locking conflict, commonly surfaced as:

```text
OptimisticLockException
```

or a Spring data-access exception wrapping the underlying JPA/Hibernate exception, depending on the stack.

---

# 8. The Core Optimistic Locking Algorithm

Remember this:

```text
Read:
id = 10
version = 5

        ↓

Modify entity

        ↓

UPDATE ... WHERE id = 10
             AND version = 5

        ↓

If 1 row updated:
    SUCCESS

If 0 rows updated:
    CONFLICT
```

Then the successful transaction increments the version:

```text
5 → 6
```

---

# 9. Why is `@Version` So Powerful?

It converts:

```text
"Did somebody change this record after I read it?"
```

into a database-enforced check.

Without versioning:

```text
UPDATE account
SET balance = ?
WHERE id = 10;
```

With optimistic locking:

```text
UPDATE account
SET balance = ?,
    version = version + 1
WHERE id = 10
AND version = ?;
```

The second form detects stale updates.

---

# 10. What Types Can `@Version` Use?

JPA supports version properties using appropriate version types such as:

```text
int
Integer
long
Long
short
Short
Timestamp
```

The exact choice depends on the application.

A common choice is:

```java
@Version
private Long version;
```

---

# 11. Does `@Version` Lock the Row?

### Important trap

**No — not in the same sense as a pessimistic database row lock.**

Optimistic locking doesn't normally hold a database row lock for the entire business operation.

Instead:

```text
Optimistic:
Read → work → verify version during update
```

Pessimistic:

```text
Read + acquire database lock → work → release lock
```

---

# 12. When Should You Use Optimistic Locking?

Good candidate:

```text
Read-heavy
Write conflicts relatively uncommon
Transactions relatively short
```

Examples:

```text
Product details
User profile
Configuration
Order updates
Document editing
```

Suppose two users edit the same order.

Instead of silently allowing the second update to overwrite the first:

```text
User A saves → version 6
User B saves stale version 5 → conflict
```

You can tell User B:

> "This record was modified by someone else. Please refresh and retry."

---

# 13. Optimistic Locking in a REST API

A useful production design is:

```text
GET /orders/100
```

Response:

```json
{
  "id": 100,
  "status": "PENDING",
  "version": 5
}
```

Client modifies the order and sends:

```text
version = 5
```

Meanwhile another request updates the order:

```text
version 5 → 6
```

The stale client attempts to update version 5.

The database rejects the stale update.

The API can return a conflict such as:

```text
409 Conflict
```

This is a very good practical example to mention in an interview.

---

# 14. Optimistic Locking Does Not Automatically Retry

### Interview Question

**If optimistic locking fails, should we automatically retry?**

Not blindly.

Suppose:

```text
A updates record
B gets OptimisticLockException
```

B cannot simply repeat the same stale operation.

B needs to decide:

```text
Reload latest state
     |
     v
Recalculate business operation
     |
     v
Retry if appropriate
```

For example, a financial operation may need a completely different reconciliation path rather than a blind retry.

---

# 15. Pessimistic Locking ⭐⭐⭐⭐⭐

### Interview Question

**What is pessimistic locking?**

Pessimistic locking assumes that concurrent conflicts are possible and obtains a database lock while the transaction is working with the row.

Typical JPA lock modes include:

```text
PESSIMISTIC_READ
PESSIMISTIC_WRITE
PESSIMISTIC_FORCE_INCREMENT
```

The exact SQL lock semantics depend on the database.

---

# 16. `PESSIMISTIC_WRITE`

Example:

```java
@Lock(LockModeType.PESSIMISTIC_WRITE)
Optional<Account> findById(Long id);
```

Or using `EntityManager`:

```java
Account account =
    entityManager.find(
        Account.class,
        id,
        LockModeType.PESSIMISTIC_WRITE
    );
```

Conceptually, the database performs a locking read.

Depending on the database, the SQL may resemble:

```sql
SELECT ...
FROM account
WHERE id = ?
FOR UPDATE;
```

**Do not memorize `FOR UPDATE` as universal SQL.** The exact SQL is database-specific.

---

# 17. What Does `PESSIMISTIC_WRITE` Achieve?

Suppose:

```text
Account 10
Balance = 10,000
```

Transaction A:

```text
SELECT ... FOR UPDATE
```

Transaction B attempts to acquire the same write lock.

B may have to wait until A releases the lock, depending on database/transaction behavior.

Conceptually:

```text
Transaction A
     |
     +---- lock Account 10
     |
     +---- update
     |
     +---- commit
              |
              v
        lock released

Transaction B
     |
     +---- waits
     |
     +---- acquires lock
     |
     +---- reads latest state
```

---

# 18. Optimistic vs Pessimistic Locking

### ⭐⭐⭐⭐⭐ Must Know

| Optimistic | Pessimistic |
|---|---|
| Detect conflict later | Prevent/serialize conflicting access |
| Usually no long-held row lock during read/work | Database lock acquired |
| Uses version/check | Uses DB locking |
| Good when conflicts are relatively rare | Good when conflicts are frequent/critical |
| Better concurrency in many cases | Can reduce concurrency |
| Conflict may result in exception | Other transactions may wait/block |
| Often simpler to scale | Lock contention can become a bottleneck |

Mental model:

```text
Optimistic:
"Let's work and check whether someone changed it."

Pessimistic:
"Let's lock it before working."
```

---

# 19. How Do You Define Pessimistic Locking in Spring Data JPA?

Example:

```java
public interface AccountRepository
        extends JpaRepository<Account, Long> {

    @Lock(LockModeType.PESSIMISTIC_WRITE)
    @Query("select a from Account a where a.id = :id")
    Optional<Account> findAccountForUpdate(
        @Param("id") Long id
    );
}
```

Service:

```java
@Transactional
public void withdraw(Long id, BigDecimal amount) {

    Account account =
        repository.findAccountForUpdate(id)
                  .orElseThrow();

    account.withdraw(amount);
}
```

The transaction holds the database lock according to the database's locking/transaction semantics.

---

# 20. Why Must the Pessimistic Lock Usually Be Inside a Transaction?

Because the database lock is tied to transaction boundaries.

You generally want:

```text
BEGIN
  |
  v
Acquire lock
  |
  v
Read/modify
  |
  v
Commit / Rollback
  |
  v
Release lock
```

If the transaction is badly designed and remains open for a long time:

```text
lock held
   |
   |---- slow external API
   |---- expensive computation
   |---- network call
   |
   v
commit
```

you can create serious lock contention.

---

# 21. Production Rule: Don't Hold Database Locks During External Calls

Bad:

```java
@Transactional
public void processPayment(Long id) {

    Account account =
        repository.findForUpdate(id);

    paymentProvider.call();  // external network call

    account.setStatus(...);
}
```

Potential flow:

```text
DB lock acquired
       |
       v
External API takes 3 seconds
       |
       v
DB lock remains held
       |
       v
Other requests wait
```

This can destroy throughput.

Better architecture often separates:

```text
short DB transaction
+
external operation
+
reconciliation/idempotency
```

The exact design depends on the business requirement.

---

# 22. Optimistic Locking Example — Order

Entity:

```java
@Entity
public class Order {

    @Id
    private Long id;

    private String status;

    @Version
    private Long version;
}
```

Two requests:

```text
Request A → version 5
Request B → version 5
```

A:

```text
PENDING → CONFIRMED
5 → 6
```

B:

```text
PENDING → CANCELLED
```

B attempts to update version 5.

Result:

```text
0 rows updated
     |
     v
Optimistic locking conflict
```

Now the application can decide what should happen.

---

# 23. What is `PESSIMISTIC_READ`?

`PESSIMISTIC_READ` requests a pessimistic read lock according to the JPA/database locking semantics.

The exact behavior depends on the database's supported lock modes and isolation/locking implementation.

For interview purposes:

> "`PESSIMISTIC_READ` requests a database-level pessimistic read lock, while `PESSIMISTIC_WRITE` requests a lock suitable for preventing conflicting updates."

In practice, `PESSIMISTIC_WRITE` is more commonly encountered in application code where a row must be safely modified.

---

# 24. What is `PESSIMISTIC_FORCE_INCREMENT`?

This is more advanced.

It combines pessimistic locking semantics with a version increment.

You generally don't need to lead with this in an interview unless asked.

The important modes to know first are:

```text
OPTIMISTIC
OPTIMISTIC_FORCE_INCREMENT
PESSIMISTIC_READ
PESSIMISTIC_WRITE
PESSIMISTIC_FORCE_INCREMENT
```

For EPAM, prioritize:

```text
@Version
OPTIMISTIC
PESSIMISTIC_WRITE
```

---

# 25. Optimistic Locking and `@Version` — Important Trap

### Question

If I have:

```java
@Version
private Long version;
```

does Hibernate prevent two transactions from **reading** the same row?

### No.

Both can read:

```text
version = 5
```

The conflict is detected when they attempt to update using the stale version.

That's why it's called **optimistic**.

---

# 26. Pessimistic Locking — Important Trap

### Question

Does pessimistic locking mean the entire table is locked?

### No.

The lock scope depends on:

- query
- database
- indexes
- transaction
- isolation
- database lock implementation

Typically you intend to lock the relevant rows, not the entire table.

Don't say:

> "PESSIMISTIC_WRITE locks the whole table."

That's incorrect.

---

# 27. Deadlocks ⭐⭐⭐⭐⭐

### Interview Question

**Can pessimistic locking cause deadlocks?**

Yes.

Example:

```text
Transaction A:
lock Account 1
then tries Account 2

Transaction B:
lock Account 2
then tries Account 1
```

Now:

```text
A waits for B
B waits for A
```

```text
       Account 1
       ↑       |
       |       ↓
      A        B
       ↑       |
       |       ↓
       Account 2
```

This is a classic deadlock.

The database may detect it and abort one transaction.

---

# 28. How Do You Reduce Deadlocks?

A strong senior answer:

> "I try to acquire locks in a consistent order, keep transactions short, avoid unnecessary locks, ensure appropriate indexes, and avoid external/network calls while holding database locks. I also monitor deadlock errors and design retry/recovery behavior where appropriate."

Example:

Always lock:

```text
Account 1
then Account 2
```

instead of allowing one code path to use:

```text
1 → 2
```

and another:

```text
2 → 1
```

---

# 29. Optimistic Locking vs Database Isolation

### Important distinction

These are related but different concepts.

### Isolation

Controls how concurrent transactions interact from the database transaction perspective.

Examples:

```text
READ COMMITTED
REPEATABLE READ
SERIALIZABLE
```

### Optimistic locking

Adds explicit version-based conflict detection.

You can have:

```text
READ COMMITTED
+
@Version
```

These aren't alternatives.

Think:

```text
Transaction isolation
        +
Application/entity concurrency control
```

Both can be used together.

---

# 30. `@Version` vs `synchronized`

### Interview Question

**Can I just use Java `synchronized` instead of `@Version`?**

No, not for distributed application concurrency.

Suppose you have:

```text
Kubernetes Pod A
Kubernetes Pod B
Kubernetes Pod C
```

A Java lock such as:

```java
synchronized
```

only coordinates threads within the relevant JVM/object instance.

It does not coordinate independent application instances.

Database locking/versioning can coordinate concurrent transactions across application instances.

This is a **very important microservices interview point**.

---

# 31. Why Java Locks Don't Solve Database Concurrency

Imagine:

```text
             Database
                |
       +--------+--------+
       |        |        |
     Pod A    Pod B    Pod C
       |        |        |
     JVM A    JVM B    JVM C
```

If Pod A uses:

```java
synchronized
```

Pod B doesn't know about that Java monitor.

Therefore:

```text
Java lock
=
process/JVM-level coordination

Database lock / version
=
database-level coordination
```

---

# 32. Senior Scenario — Payment

Suppose two requests process the same payment:

```text
POST /payments
```

Both try to update:

```text
payment.status
```

You might use:

```text
idempotency key
+
database unique constraint
+
transaction
+
optimistic locking where appropriate
```

Don't automatically solve everything with pessimistic locks.

Concurrency control should match the business operation.

---

# 33. Senior Scenario — Ledger

For a financial ledger, you may have:

```text
Account balance
Ledger entries
```

A simplistic:

```java
balance = balance - amount;
```

approach is dangerous under concurrent updates.

Possible approaches include:

```text
atomic database updates
optimistic locking
pessimistic locking
append-only ledger
serializable/appropriate isolation
database constraints
```

The correct design depends on the actual consistency model.

The important interview point:

> **Don't blindly load a balance, calculate in Java, and write it back without considering concurrent transactions.**

---

# 34. Optimistic Lock Retry

Suppose:

```text
Transaction A succeeds
Transaction B gets optimistic conflict
```

A retry might look conceptually like:

```text
read latest state
     |
     v
re-evaluate business operation
     |
     v
attempt update
```

But don't do:

```text
catch OptimisticLockException
retry same stale object
```

because the object may still represent the old state.

---

# 35. Pessimistic Lock Timeout

Database/JPA configurations can impose lock timeout behavior.

The exact configuration and behavior are provider/database dependent.

The important production concept is:

```text
Request waits for lock
       |
       +---- lock acquired
       |
       +---- timeout
       |
       +---- exception/failure
```

Don't allow an application to wait indefinitely for a heavily contended resource.

---

# 36. What Happens When a Lock Transaction Rolls Back?

Suppose:

```text
Transaction A
    |
    +-- obtains lock
    |
    +-- modifies row
    |
    +-- rollback
```

The database releases the transaction's locks according to its transaction semantics.

Other waiting transactions can then proceed.

---

# 37. How Would You Answer This EPAM Question?

### Interviewer:

> "Two instances of your Spring Boot service update the same database record at the same time. How would you prevent lost updates?"

### Strong answer:

> "I would first identify the business consistency requirement. For relatively rare conflicts, I'd normally consider optimistic locking using a JPA `@Version` field. Hibernate includes the version in the update condition, so if another transaction has already modified the record, the stale update affects zero rows and an optimistic locking exception is raised. The application can then reload the latest state and decide whether the operation can be retried. If the operation requires serialization while the transaction is executing, I could use pessimistic locking such as `PESSIMISTIC_WRITE`. I'd keep the transaction short and avoid external calls while holding the database lock."

That's a strong senior answer.

---

# 38. Rapid-Fire EPAM Questions

### Q: What is optimistic locking?

Detect concurrent modification using version/state checks rather than holding a database lock during the whole operation.

### Q: How is optimistic locking commonly implemented in JPA?

Using:

```java
@Version
```

### Q: What happens if the version doesn't match?

The update affects zero rows and an optimistic locking conflict is raised.

### Q: Does `@Version` prevent concurrent reads?

No.

### Q: What is pessimistic locking?

Acquire a database lock to control concurrent access while the transaction is operating on the row.

### Q: Most commonly used pessimistic mode?

`PESSIMISTIC_WRITE` is a common choice when a row must be safely modified.

### Q: Does pessimistic locking lock the entire table?

Not necessarily. Lock scope depends on the query/database/transaction semantics.

### Q: Can pessimistic locking cause deadlocks?

Yes.

### Q: How do you reduce deadlocks?

Consistent lock ordering, short transactions, fewer locks, appropriate indexes, and avoiding external calls while holding locks.

### Q: Can `synchronized` solve concurrency between Kubernetes pods?

No. It only coordinates within the relevant JVM/object scope.

### Q: Is optimistic locking the same as transaction isolation?

No. They are different concurrency mechanisms and can be used together.

---

# 39. The Mental Model

When two requests modify the same database row:

```text
             Same Row
                |
       +--------+--------+
       |                 |
 Transaction A     Transaction B
       |                 |
       v                 v
   What happens?
       |
       +--------------------------+
       |                          |
       v                          v
 Optimistic                  Pessimistic
       |                          |
   @Version                  DB Lock
       |                          |
 detect conflict             serialize access
       |                          |
 exception if stale          possible waiting
```

Remember:

```text
Optimistic
=
detect conflict

Pessimistic
=
prevent/serialize conflicting access
```

---

## Next: JPA Performance & Production ⭐⭐⭐⭐⭐

The final major JPA section will bring everything together:

```text
Pagination
Offset vs Keyset
JDBC Batching
Batch Inserts / Updates
Indexes
Query Optimization
Execution Plans
HikariCP
Connection Pool Exhaustion
Slow Queries
N+1
LazyInitializationException
OptimisticLockException
```

We'll make this particularly practical because **pagination, connection pools, indexes, and large-data processing** are exactly the kinds of things that distinguish a senior-level answer from simply knowing JPA annotations.