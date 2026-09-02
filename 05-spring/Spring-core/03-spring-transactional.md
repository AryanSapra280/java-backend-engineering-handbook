# Spring `@Transactional` — Interview Revision Notes

## 1. What is `@Transactional`?

`@Transactional` tells Spring that a method or class should execute within a managed database transaction.

```java
@Transactional
public void processContribution() {
    contributionRepository.save(contribution);
    transactionRepository.save(transaction);
    balanceRepository.updateBalance();
}
```

Conceptually:

```text
START TRANSACTION
       ↓
Operation 1
       ↓
Operation 2
       ↓
Operation 3
       ↓
COMMIT
```

If a failure causes rollback:

```text
START TRANSACTION
       ↓
Operation 1 ✓
       ↓
Operation 2 ✓
       ↓
Operation 3 ✗
       ↓
ROLLBACK
```

---

## 2. Why Do We Need Transactions?

A business operation may contain multiple database operations:

```text
Contribution
    ↓
Update Ledger
    ↓
Update Balance
    ↓
Create Transaction Record
```

Without an appropriate transaction, one operation could succeed while another fails.

The goal is **atomicity**:

> Operations participating in the transaction should commit together or be rolled back together according to the configured rules.

---

## 3. How Does `@Transactional` Work?

This connects directly to Spring AOP and proxies.

```text
Caller
   ↓
Spring Proxy
   ↓
Transaction Interceptor
   ↓
Begin / Join Transaction
   ↓
Target Method
   ↓
Commit / Rollback
```

### Interview answer

> `@Transactional` is implemented using Spring's transaction infrastructure, typically through proxy-based interception. The proxy intercepts the call, the transaction interceptor starts or joins a transaction, invokes the target method, and then commits or rolls back according to the outcome and configured rules.

---

## 4. Where Should `@Transactional` Usually Go?

Usually at the **service layer**:

```text
Controller
    ↓
Service       ← transaction boundary
    ↓
Repository
    ↓
Database
```

A service method often represents one business operation containing multiple repository calls.

---

# 5. Success vs Failure

Successful method:

```text
BEGIN
 ↓
Business logic
 ↓
COMMIT
```

Failure that triggers rollback:

```text
BEGIN
 ↓
Business logic
 ↓
Exception
 ↓
ROLLBACK
```

Do not simply memorize:

> "Any exception means rollback."

Rollback depends on Spring's rollback rules and configuration.

---

# 6. RuntimeException vs Checked Exception ⭐⭐⭐

By default, Spring's declarative transaction behavior generally rolls back for:

```text
RuntimeException
Error
```

Ordinary checked exceptions do not normally trigger rollback by default.

Example:

```java
@Transactional
public void process() {
    saveData();
    throw new RuntimeException();
}
```

Typically:

```text
RuntimeException
      ↓
Rollback
```

But:

```java
@Transactional
public void process() throws Exception {
    saveData();
    throw new Exception();
}
```

A checked exception does not normally trigger rollback by default.

### Interview answer

> By default, Spring rolls back for unchecked exceptions and `Error`, while checked exceptions do not normally trigger rollback. We can configure additional rollback rules using `rollbackFor`.

---

# 7. `rollbackFor`

To explicitly roll back for a checked exception:

```java
@Transactional(rollbackFor = Exception.class)
public void process() throws Exception {
    saveContribution();
    throw new Exception();
}
```

### Interview answer

> `rollbackFor` lets us explicitly specify exceptions that should cause the transaction to roll back.

---

# 8. Important Trap — Catching the Exception

Consider:

```java
@Transactional
public void process() {

    try {
        saveData();
        throw new RuntimeException();
    } catch (Exception e) {
        log.error("Failed", e);
    }
}
```

The exception was caught inside the method.

The method may therefore return normally to the transaction interceptor.

So do not say:

> "There was a RuntimeException, therefore Spring definitely rolled back."

If you catch an exception and still need rollback, you may need to explicitly mark the transaction rollback-only or configure the behavior appropriately.

### Interview takeaway

> Catching an exception can change the rollback behavior you expected.

---

# 9. Transaction Propagation ⭐⭐⭐

Propagation answers:

> **What should happen when one transactional method calls another transactional method?**

Example:

```text
Service A
   ↓
Service B
```

If both are transactional, should B:

- Join A's transaction?
- Create a new transaction?
- Suspend A's transaction?

That is transaction **propagation**.

---

# 10. `REQUIRED` — Default ⭐⭐⭐

```java
@Transactional(propagation = Propagation.REQUIRED)
```

Meaning:

> Join an existing transaction if one exists; otherwise create a new transaction.

### No existing transaction

```text
A called
 ↓
No transaction
 ↓
Create T1
 ↓
A executes
```

### Existing transaction

```text
A
 ↓
T1
 ↓
B
 ↓
B joins T1
```

Therefore:

```text
A ───────────────┐
                 │
              T1 │
                 │
B ───────────────┘
```

### Interview answer

> `REQUIRED` joins the current transaction if one exists; otherwise Spring creates a new transaction.

---

# 11. `REQUIRES_NEW` ⭐⭐⭐

Meaning:

> Always execute using a new transaction.

If a transaction already exists, it is suspended while the new transaction executes.

```text
T1
 ↓
Method A
 ↓
Method B
 ↓
Suspend T1
 ↓
Start T2
 ↓
B executes
 ↓
Commit/Rollback T2
 ↓
Resume T1
```

---

# 12. Your Important `REQUIRES_NEW` Doubt

Suppose:

```java
@Transactional
public void methodA() {

    saveA();

    methodB();

    saveMoreA(); // fails
}
```

and B uses:

```java
@Transactional(propagation = Propagation.REQUIRES_NEW)
public void methodB() {
    saveB();
}
```

Flow:

```text
Method A
   ↓
Transaction T1 starts
   ↓
saveA()
   ↓
T1 suspended
   ↓
Method B
   ↓
Transaction T2 starts
   ↓
saveB()
   ↓
B finishes
   ↓
COMMIT T2 ✅
   ↓
T1 resumes
   ↓
saveMoreA()
   ↓
FAIL ❌
   ↓
ROLLBACK T1
```

Final state:

```text
A changes → rolled back ❌
B changes → committed ✅
```

### Key point

> `REQUIRES_NEW` gives B an independent transaction. Once B commits, a later rollback of A does not roll B's transaction back.

---

# 13. `REQUIRED` Example

Suppose B uses the default:

```java
@Transactional
public void methodB() {
    saveB();
}
```

A:

```java
@Transactional
public void methodA() {

    saveA();

    methodB();

    saveMoreA(); // fails
}
```

B joins A's transaction:

```text
Method A
   ↓
Transaction T1
   ↓
saveA()
   ↓
Method B
   ↓
B joins T1
   ↓
saveB()
   ↓
B finishes
   ↓
NO COMMIT YET ❗
   ↓
saveMoreA()
   ↓
FAIL ❌
   ↓
ROLLBACK T1
```

Final state:

```text
A changes → rolled back ❌
B changes → rolled back ❌
```

### Critical distinction

Do not think:

> "B finished, therefore B's transaction committed."

Instead ask:

> **"Is B using the same transaction or a new transaction?"**

---

# 14. REQUIRED vs REQUIRES_NEW ⭐⭐⭐

| | REQUIRED | REQUIRES_NEW |
|---|---|---|
| Existing transaction? | Join it | Suspend it |
| New transaction? | Only if none exists | Always |
| A and B same transaction? | Yes | No |
| B commits independently? | No | Yes |
| A fails later | B rolls back with A | B can remain committed |

Mental shortcut:

```text
REQUIRED
"Join mine if one exists."

REQUIRES_NEW
"Give me my own."
```

### One-line interview answer

> With `REQUIRED`, B joins A's transaction, so if A eventually rolls back, B's changes roll back too. With `REQUIRES_NEW`, A's transaction is suspended and B gets a separate transaction; once B commits, a later rollback of A does not affect B.

---

# 15. `NESTED`

`NESTED` can execute using a nested transaction/savepoint model when supported.

Conceptually:

```text
Transaction T1
      ↓
Savepoint
      ↓
Nested operation
      ↓
Failure
      ↓
Rollback to savepoint
```

Unlike `REQUIRES_NEW`, it does not necessarily create a completely independent physical transaction.

For Monday:

> Know the difference conceptually. Do not spend too much time on implementation details unless asked.

---

# 16. Isolation ⭐⭐⭐

Propagation asks:

> **Which transaction am I using?**

Isolation asks:

> **How isolated is my transaction from concurrent transactions?**

Common levels:

```text
READ_UNCOMMITTED
READ_COMMITTED
REPEATABLE_READ
SERIALIZABLE
```

Generally:

```text
READ_UNCOMMITTED
       ↓
READ_COMMITTED
       ↓
REPEATABLE_READ
       ↓
SERIALIZABLE
```

More isolation generally means more consistency but can reduce concurrency.

Exact behavior depends on the database engine.

---

# 17. Dirty Read

T1 reads data written by T2 before T2 commits.

```text
T2:
balance = 5000
 ↓
updates balance = 3000
 ↓
NOT committed

T1:
reads 3000
```

If T2 rolls back:

```text
T2 rollback
 ↓
balance = 5000
```

T1 read data that never committed.

That is a **dirty read**.

---

# 18. Non-Repeatable Read

T1 reads:

```text
T1 → balance = 5000
```

T2 changes and commits:

```text
T2 → balance = 3000 → COMMIT
```

T1 reads again:

```text
T1 → balance = 3000
```

Same transaction, same row, different committed value.

That is a **non-repeatable read**.

---

# 19. Phantom Read

T1:

```sql
SELECT *
FROM account
WHERE balance > 5000;
```

It gets 10 rows.

T2 inserts another matching row and commits.

T1 runs the same query again and gets 11 rows.

The additional matching row is a **phantom**.

---

# 20. Isolation Cheat Sheet

| Isolation | Dirty Read | Non-repeatable Read | Phantom Read |
|---|---:|---:|---:|
| READ_UNCOMMITTED | Possible | Possible | Possible |
| READ_COMMITTED | Prevented | Possible | Possible |
| REPEATABLE_READ | Prevented | Prevented | Database-dependent |
| SERIALIZABLE | Prevented | Prevented | Prevented |

Important:

> Exact behavior can depend on the database engine and its implementation.

---

# 21. Self Invocation — Transaction Trap ⭐⭐⭐

Consider:

```java
@Service
public class PaymentService {

    public void methodA() {
        methodB();
    }

    @Transactional
    public void methodB() {
    }
}
```

The call:

```java
methodB();
```

is an internal call on the same object.

It does not go through the Spring proxy.

Therefore:

```text
External call
     ↓
Proxy
     ↓
@Transactional method
     ✓ proxy-based interception can apply
```

versus:

```text
methodA()
   ↓
this.methodB()
   ↓
Target directly
   ✗ proxy bypassed
```

### Interview answer

> Because Spring's declarative transaction management is proxy-based, self-invocation can bypass the proxy and prevent the expected transactional interception.

---

# 22. How to Fix Self Invocation

A common clean solution is to move the transactional operation into another Spring-managed bean.

```java
@Service
public class PaymentService {

    private final TransactionService transactionService;

    public PaymentService(TransactionService transactionService) {
        this.transactionService = transactionService;
    }

    public void methodA() {
        transactionService.methodB();
    }
}
```

```java
@Service
public class TransactionService {

    @Transactional
    public void methodB() {
    }
}
```

Now:

```text
PaymentService
      ↓
Spring Proxy
      ↓
TransactionService
      ↓
@Transactional method
```

The call crosses the proxy boundary.

---

# 23. Local Transaction ≠ Distributed Transaction ⭐⭐⭐

Suppose:

```text
PF Service
    ↓
Transaction Service
    ↓
Ledger Service
```

and:

```java
@Transactional
public void process() {
    callTransactionService();
    callLedgerService();
}
```

Do not assume that one local Spring transaction automatically covers all service databases.

For example:

```text
PF DB
  +
Transaction DB
  +
Ledger DB
```

does not automatically become:

```text
ONE ATOMIC TRANSACTION
```

Across microservices, you may need patterns such as:

```text
Saga
Outbox
Idempotency
Compensation
Event-driven workflows
```

We will cover these later in Microservices.

---

# 24. PF Interview Scenario

Suppose:

```text
ContributionController
        ↓
ContributionService
        ↓
Repository
        ↓
Postgres
```

and:

```java
@Transactional
public void processContribution() {

    saveContribution();

    updateLedger();

    updateBalance();
}
```

Desired flow:

```text
HTTP Request
     ↓
Controller
     ↓
Spring Proxy
     ↓
Transaction Interceptor
     ↓
BEGIN
     ↓
saveContribution()
     ↓
updateLedger()
     ↓
updateBalance()
     ↓
COMMIT
```

If a participating database operation fails and rollback rules apply:

```text
BEGIN
  ↓
Operation 1 ✓
  ↓
Operation 2 ✓
  ↓
Operation 3 ✗
  ↓
ROLLBACK
```

---

# 25. Cross-Microservice PF Scenario

Suppose:

```text
PF Service
     ↓
Transaction Service
     ↓
Ledger Service
```

Each service owns its own database.

A local:

```java
@Transactional
public void process() {
    callTransactionService();
    callLedgerService();
}
```

does not automatically mean:

```text
PF DB + Transaction DB + Ledger DB
        ↓
ONE TRANSACTION
```

Each service may have its own local transaction.

This is why distributed systems need:

```text
Consistency
Retries
Idempotency
Failure recovery
Compensation
Outbox / event publishing
```

---

# 26. High-Probability Interview Questions

### Q1. What does `@Transactional` do?

> It defines a transactional boundary around a method or class so Spring's transaction infrastructure can manage transaction creation, participation, commit, and rollback according to configured rules.

### Q2. How does `@Transactional` work internally?

> Spring typically uses proxy-based interception. The proxy intercepts the call, the transaction interceptor starts or joins a transaction, invokes the target method, and then commits or rolls back based on the result and rollback rules.

### Q3. What is the default propagation?

> `REQUIRED`.

### Q4. What does `REQUIRED` mean?

> Join the existing transaction if one exists; otherwise create a new one.

### Q5. What does `REQUIRES_NEW` mean?

> Suspend the existing transaction and execute in a new independent transaction.

### Q6. RuntimeException vs checked exception?

> By default, Spring rolls back for unchecked exceptions and `Error`, while checked exceptions do not normally trigger rollback. `rollbackFor` can be used to configure additional rollback rules.

### Q7. What is isolation?

> Isolation defines how a transaction interacts with concurrent transactions and what changes it can observe.

### Q8. What is a dirty read?

> Reading data written by another transaction before that transaction commits.

### Q9. What is a non-repeatable read?

> Reading the same row twice in one transaction and getting different committed values because another transaction changed it between reads.

### Q10. What is a phantom read?

> Re-running a query and seeing a different set of matching rows because another transaction inserted or removed matching rows.

### Q11. Does self-invocation trigger `@Transactional`?

> Not reliably with Spring's proxy-based model, because the internal call bypasses the proxy.

### Q12. Does `@Transactional` work across microservices automatically?

> No. A local Spring transaction does not automatically create one atomic transaction across independent service databases.

---

# 27. What to Memorize for Monday

If time is limited:

```text
1. @Transactional
   ↓
   Defines a transaction boundary.

2. Proxy
   ↓
   Spring intercepts the method call.

3. RuntimeException
   ↓
   Rollback by default.

4. Checked Exception
   ↓
   Usually no rollback by default.
   ↓
   rollbackFor can change this.

5. REQUIRED
   ↓
   Join existing transaction or create one.

6. REQUIRES_NEW
   ↓
   Suspend existing transaction and create a new one.

7. Self-invocation
   ↓
   Bypasses proxy.

8. Isolation
   ↓
   Controls concurrency/visibility behavior.

9. Microservices
   ↓
   Local transaction ≠ distributed transaction.
```

---

# 28. Final Mental Model

```text
                 @Transactional
                       ↓
                  Spring Proxy
                       ↓
              Transaction Interceptor
                       ↓
              BEGIN / JOIN TRANSACTION
                       ↓
              ┌───────────────────┐
              │   Business Logic  │
              │                   │
              │ DB operation #1   │
              │ DB operation #2   │
              │ DB operation #3   │
              └───────────────────┘
                    ↓       ↓
                 Success   Failure
                    ↓       ↓
                 COMMIT   ROLLBACK
```

Propagation:

```text
A @Transactional
      ↓
B @Transactional
      ↓

REQUIRED
→ B joins A's transaction

REQUIRES_NEW
→ suspend A
→ create B transaction
→ finish B
→ resume A
```

Most important distinction:

```text
REQUIRED:

A → T1
B → T1

A fails later
↓
T1 rolls back
↓
A + B roll back


REQUIRES_NEW:

A → T1
B → T2

B finishes
↓
T2 commits

A fails later
↓
T1 rolls back

B remains committed
```

---

# 29. Topics to Return to Later

Lower priority for Monday:

- Custom `PlatformTransactionManager`
- Transaction synchronization internals
- JDBC connection binding internals
- JTA
- XA transactions
- Two-phase commit
- Detailed savepoint implementation
- Transaction interceptor source code
- Database-specific MVCC internals

For the interview, master:

```text
@Transactional
↓
Proxy
↓
Commit / Rollback
↓
Rollback rules
↓
REQUIRED
↓
REQUIRES_NEW
↓
Isolation
↓
Self-invocation
↓
Local vs distributed transaction
```
