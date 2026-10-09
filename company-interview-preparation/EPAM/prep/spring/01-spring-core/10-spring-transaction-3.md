Absolutely — now we move into **Transaction Propagation**, one of the highest-value Spring interview topics.

# 3. Transaction Propagation ⭐⭐⭐⭐⭐

## 1. What is transaction propagation?

Transaction propagation defines:

> **What should happen to the transaction when one transactional method calls another transactional method.**

Consider:

```java
@Transactional
public void placeOrder() {

    saveOrder();

    paymentService.processPayment();
}
```

Both methods may be transactional.

The question is:

> If `placeOrder()` already has a transaction, what should `processPayment()` do?

Should it:

- join the existing transaction?
- create a completely new transaction?
- suspend the existing transaction?
- fail if there is no transaction?
- execute without a transaction?

That behavior is controlled by **propagation**.

Spring provides:

```java
Propagation.REQUIRED
Propagation.REQUIRES_NEW
Propagation.NESTED
Propagation.SUPPORTS
Propagation.NOT_SUPPORTED
Propagation.MANDATORY
Propagation.NEVER
```

---

# 2. The default: `REQUIRED`

```java
@Transactional(
    propagation = Propagation.REQUIRED
)
```

`REQUIRED` means:

> **Use the existing transaction if one exists; otherwise create a new transaction.**

This is the default propagation for `@Transactional`.

So:

```java
@Transactional
```

is effectively:

```java
@Transactional(propagation = Propagation.REQUIRED)
```

---

# 3. REQUIRED — no existing transaction

Suppose:

```java
@Transactional
public void processPayment() {
    paymentRepository.save(payment);
}
```

And there is currently no transaction.

Spring does:

```text
Caller
  ↓
processPayment()
  ↓
No existing transaction
  ↓
Create T1
  ↓
Database operation
  ↓
COMMIT
```

So:

```text
No transaction
      ↓
REQUIRED
      ↓
Create T1
```

---

# 4. REQUIRED — existing transaction

Now:

```java
@Transactional
public void placeOrder() {

    orderRepository.save(order);

    paymentService.processPayment();
}
```

And:

```java
@Transactional
public void processPayment() {

    paymentRepository.save(payment);
}
```

Flow:

```text
placeOrder()
      ↓
No transaction
      ↓
Create T1
      ↓
orderRepository.save()
      ↓
processPayment()
      ↓
Transaction T1 already exists
      ↓
Join T1
      ↓
paymentRepository.save()
      ↓
return
      ↓
T1 COMMIT
```

There is **one transaction**, not two.

```text
             T1
┌─────────────────────────────┐
│ placeOrder()                │
│                             │
│ orderRepository.save()      │
│                             │
│ processPayment()            │
│     ↓                       │
│ paymentRepository.save()    │
│                             │
└─────────────────────────────┘
```

---

# 5. Interview Question: Does `REQUIRED` create a new transaction every time?

❌ No.

This is one of the most common interview traps.

Correct answer:

> `REQUIRED` creates a transaction only when no transaction currently exists. If a transaction already exists, the method participates in that transaction.

Remember:

```text
Existing transaction?
       │
   ┌───┴───┐
  YES      NO
   ↓        ↓
 JOIN      CREATE
```

---

# 6. What happens if the inner method fails?

Suppose:

```java
@Transactional
public void placeOrder() {

    orderRepository.save(order);

    paymentService.processPayment();
}
```

and:

```java
@Transactional
public void processPayment() {

    paymentRepository.save(payment);

    throw new RuntimeException("Payment failed");
}
```

Because both use `REQUIRED`:

```text
T1
│
├── save order
│
├── processPayment()
│      │
│      ├── save payment
│      └── RuntimeException
│
└── rollback T1
```

The entire transaction can roll back.

So:

```text
Order → rollback
Payment → rollback
```

assuming the exception reaches the transaction boundary and triggers rollback.

---

# 7. The important concept: logical vs physical transaction

This is a senior-level concept.

When multiple methods use:

```text
REQUIRED
```

Spring can create separate **logical transaction scopes**, but they can participate in the same underlying physical transaction.

For example:

```text
placeOrder()
    REQUIRED
       ↓
processPayment()
    REQUIRED
```

Conceptually:

```text
Logical scope 1
placeOrder()
       ↓
Logical scope 2
processPayment()
       ↓
Same physical transaction T1
```

This matters when discussing rollback behavior.

An inner method can mark the transaction as rollback-only.

Then the outer method may eventually encounter:

```text
UnexpectedRollbackException
```

We'll see this in the example below.

---

# 8. Important scenario: inner method catches the exception

Consider:

```java
@Transactional
public void placeOrder() {

    orderRepository.save(order);

    try {
        paymentService.processPayment();
    } catch (Exception e) {
        log.error("Payment failed", e);
    }

    orderRepository.updateStatus("PENDING");
}
```

Inner method:

```java
@Transactional
public void processPayment() {

    paymentRepository.save(payment);

    throw new RuntimeException("Payment failed");
}
```

You might think:

> "The outer method caught the exception, so the transaction should commit."

Not necessarily.

With `REQUIRED`, the inner operation participates in the same transaction.

Spring may mark that transaction as:

```text
rollback-only
```

Even though the exception was caught.

Then the outer method finishes:

```text
placeOrder()
      ↓
attempt COMMIT
      ↓
Transaction marked rollback-only
      ↓
ROLLBACK
```

The caller may see:

```text
UnexpectedRollbackException
```

This is a **very good senior-level interview scenario**.

---

# 9. Why does this happen?

Because:

```text
REQUIRED
```

doesn't mean:

> "Create a completely independent transaction for this method."

It means:

> "Participate in the current transaction."

If the inner scope determines that the transaction cannot safely commit, it can mark the shared transaction rollback-only.

Therefore catching the original exception doesn't necessarily make the transaction committable again.

---

# 10. `REQUIRES_NEW`

Now the interesting one.

```java
@Transactional(
    propagation = Propagation.REQUIRES_NEW
)
```

Meaning:

> **Always execute using a new transaction.**

If an existing transaction exists:

```text
Suspend existing transaction
        ↓
Create new transaction
        ↓
Execute method
        ↓
Commit/Rollback new transaction
        ↓
Resume old transaction
```

---

# 11. REQUIRES_NEW — no existing transaction

Suppose:

```java
@Transactional(propagation = Propagation.REQUIRES_NEW)
public void savePayment() {
    paymentRepository.save(payment);
}
```

No existing transaction:

```text
No transaction
      ↓
REQUIRES_NEW
      ↓
Create T1
      ↓
savePayment()
      ↓
COMMIT T1
```

Simple.

---

# 12. REQUIRES_NEW — existing transaction

Suppose:

```java
@Transactional
public void placeOrder() {

    orderRepository.save(order);

    auditService.audit();
}
```

And:

```java
@Transactional(propagation = Propagation.REQUIRES_NEW)
public void audit() {

    auditRepository.save(entry);
}
```

Flow:

```text
T1 starts
  ↓
save order
  ↓
audit()
  ↓
Suspend T1
  ↓
Start T2
  ↓
save audit
  ↓
COMMIT T2
  ↓
Resume T1
  ↓
COMMIT / ROLLBACK T1
```

Visually:

```text
T1
┌──────────────────────────────┐
│ save order                   │
│                              │
│      SUSPENDED               │
└──────────────┬───────────────┘
               ↓
              T2
       ┌───────────────────┐
       │ save audit        │
       │                   │
       │ COMMIT            │
       └─────────┬─────────┘
                 ↓
             T1 RESUMES
                 ↓
              COMMIT
```

---

# 13. Why is `REQUIRES_NEW` useful?

A classic production use case is **audit logging**.

Suppose:

```java
@Transactional
public void processPayment() {

    paymentRepository.save(payment);

    auditService.recordAudit("PAYMENT_STARTED");

    throw new RuntimeException("Payment failed");
}
```

If audit uses normal `REQUIRED`:

```text
Payment transaction T1
       ↓
Audit joins T1
       ↓
Payment fails
       ↓
T1 rollback
       ↓
Audit rollback too
```

Maybe that's not what you want.

You might want:

> "Even if the payment transaction fails, I want the audit record to survive."

Then:

```java
@Transactional(propagation = Propagation.REQUIRES_NEW)
public void recordAudit(String event) {
    auditRepository.save(...);
}
```

Now:

```text
T1
Payment
 ↓
Suspend T1
 ↓
T2
Audit
 ↓
COMMIT T2
 ↓
Resume T1
 ↓
ROLLBACK T1
```

Final state:

```text
Payment → rolled back
Audit   → committed
```

This is a classic interview example.

---

# 14. Another use case: independent failure logging

Suppose your main transaction fails:

```text
Order processing
     ↓
Exception
```

You still want:

```text
Failure audit
Error record
```

to persist.

A separate transaction can be appropriate.

However, don't blindly use `REQUIRES_NEW` everywhere.

---

# 15. `REQUIRES_NEW` production danger

Every `REQUIRES_NEW` transaction may require another transactional resource/connection while the outer transaction is suspended.

Imagine:

```text
Thread 1
  ↓
T1 holds connection C1
  ↓
REQUIRES_NEW
  ↓
T2 needs another connection C2
```

If your connection pool is too small and many threads do this:

```text
10 threads
 ↓
10 outer transactions holding connections
 ↓
all request REQUIRES_NEW
 ↓
need additional connections
```

You can run into:

```text
connection pool exhaustion
```

Potentially even deadlock-like resource starvation.

Senior answer:

> `REQUIRES_NEW` provides transaction independence, but it has resource and performance implications because the outer transaction may remain suspended while another transaction obtains its own database resources.

---

# 16. `NESTED`

Now:

```java
Propagation.NESTED
```

means, conceptually:

> Execute within a nested transaction scope, typically using a database savepoint when supported.

This is different from `REQUIRES_NEW`.

---

# 17. NESTED vs REQUIRES_NEW

This is extremely important.

### `REQUIRES_NEW`

```text
T1
 ↓
Suspend T1
 ↓
T2
 ↓
T2 commits independently
 ↓
Resume T1
```

### `NESTED`

```text
T1
 ↓
Create savepoint
 ↓
Nested operation
 ↓
Failure
 ↓
Rollback to savepoint
 ↓
Continue T1
```

So:

```text
REQUIRES_NEW → separate transaction

NESTED       → same transaction with nested savepoint
```

---

# 18. NESTED example

Suppose:

```java
@Transactional
public void processOrders() {

    saveOrder1();

    try {
        saveOrder2();
    } catch (Exception e) {
        log.error("Order 2 failed", e);
    }

    saveOrder3();
}
```

If `saveOrder2()` uses nested propagation and savepoints are supported:

```text
T1 starts
 ↓
saveOrder1()
 ↓
SAVEPOINT S1
 ↓
saveOrder2()
 ↓
failure
 ↓
ROLLBACK TO S1
 ↓
saveOrder3()
 ↓
COMMIT T1
```

Result:

```text
Order 1 → committed
Order 2 → rolled back
Order 3 → committed
```

This is fundamentally different from rolling back the entire transaction.

---

# 19. NESTED vs REQUIRES_NEW — interview table

| | `REQUIRED` | `REQUIRES_NEW` | `NESTED` |
|---|---|---|---|
| Existing transaction | Join | Suspend | Participate |
| New physical transaction | No | Yes | No |
| Uses savepoint | No | No | Typically yes |
| Outer transaction affected by inner rollback | Yes | No | Can continue after rollback to savepoint |
| Independent commit | No | Yes | No |
| Common use | Normal business operations | Independent audit | Partial rollback |

The exact availability/behavior of `NESTED` depends on the transaction manager and underlying resource capabilities, so don't claim it is universally supported.

---

# 20. `SUPPORTS`

```java
@Transactional(propagation = Propagation.SUPPORTS)
```

Meaning:

> If a transaction exists, participate in it. If no transaction exists, execute without a transaction.

Think:

```text
Existing transaction?
      │
  ┌───┴───┐
 YES      NO
  ↓        ↓
JOIN    NO TX
```

Example:

```java
@Transactional(propagation = Propagation.SUPPORTS)
public Payment getPayment(Long id) {
    return paymentRepository.findById(id).orElseThrow();
}
```

If called inside a transaction:

```text
T1
 ↓
getPayment()
 ↓
join T1
```

If called without one:

```text
getPayment()
 ↓
No transaction
```

---

# 21. When might SUPPORTS be useful?

It can be useful for operations that:

> Can work either inside an existing transaction or without one.

For example, certain read-oriented operations.

But don't use `SUPPORTS` simply because it sounds flexible.

Choose propagation based on the actual business and consistency requirement.

---

# 22. `NOT_SUPPORTED`

```java
@Transactional(propagation = Propagation.NOT_SUPPORTED)
```

Meaning:

> Execute without a transaction.

If a transaction currently exists:

```text
Suspend transaction
      ↓
Execute method without transaction
      ↓
Resume transaction
```

Example:

```java
@Transactional
public void process() {

    saveOrder();

    reportService.generateReport();
}
```

If report generation uses:

```java
@Transactional(propagation = Propagation.NOT_SUPPORTED)
public void generateReport() {
    ...
}
```

then:

```text
T1
 ↓
saveOrder()
 ↓
Suspend T1
 ↓
generateReport() [NO TRANSACTION]
 ↓
Resume T1
 ↓
continue
```

---

# 23. Why would we ever want NOT_SUPPORTED?

Suppose you have a long-running operation that doesn't need transactional consistency:

```text
Database transaction
      ↓
save business state
      ↓
Suspend
      ↓
expensive reporting operation
      ↓
Resume
```

However, you should carefully evaluate whether this is really appropriate.

It's not a performance magic switch.

---

# 24. `MANDATORY`

```java
@Transactional(propagation = Propagation.MANDATORY)
```

Meaning:

> A transaction MUST already exist.

If one exists:

```text
T1
 ↓
MANDATORY method
 ↓
Join T1
```

If none exists:

```text
MANDATORY method
 ↓
No transaction
 ↓
Exception
```

So:

```text
Existing TX?
  YES → JOIN
  NO  → FAIL
```

Useful when a method is only valid as part of an existing transactional operation.

---

# 25. `NEVER`

```java
@Transactional(propagation = Propagation.NEVER)
```

Meaning:

> The method must NOT execute inside a transaction.

If no transaction exists:

```text
NEVER
 ↓
Execute normally
```

If a transaction exists:

```text
T1
 ↓
NEVER method
 ↓
Exception
```

So:

```text
Existing TX?
  YES → FAIL
  NO  → execute
```

It's essentially the opposite constraint of `MANDATORY`.

---

# 26. All propagation modes at a glance

| Propagation | Existing TX? | No TX? |
|---|---|---|
| `REQUIRED` | Join | Create |
| `REQUIRES_NEW` | Suspend + create new | Create |
| `NESTED` | Nested/savepoint if supported | Create |
| `SUPPORTS` | Join | No transaction |
| `NOT_SUPPORTED` | Suspend | No transaction |
| `MANDATORY` | Join | Exception |
| `NEVER` | Exception | No transaction |

This table is worth memorizing.

---

# 27. The three most important ones

For most interviews, focus heavily on:

```text
REQUIRED
REQUIRES_NEW
NESTED
```

Understand them through this diagram:

```text
REQUIRED

T1
 ├── Outer
 └── Inner joins T1
```

```text
REQUIRES_NEW

T1
 ├── Outer
 │
 └── suspend
       ↓
      T2
       ↓
     commit
       ↓
      T1 resumes
```

```text
NESTED

T1
 ├── Outer
 │
 ├── SAVEPOINT
 │
 ├── Nested operation
 │
 ├── rollback to SAVEPOINT
 │
 └── continue T1
```

---

# 28. Senior interview scenario

### Interviewer:

> Order creation is transactional. During order creation, you create an audit record. Even if order creation fails, the audit record must remain. Which propagation would you use?

Answer:

```java
@Transactional(propagation = Propagation.REQUIRES_NEW)
public void createAudit(...) {
    ...
}
```

Reason:

> The audit operation needs an independent transaction. `REQUIRES_NEW` suspends the outer transaction, starts a new transaction for the audit operation, and commits it independently. If the outer transaction later rolls back, the audit transaction can remain committed.

---

# 29. Senior interview scenario

### Interviewer:

> You have an outer transaction and want one inner operation to roll back without necessarily rolling back the entire outer transaction. Which propagation might you consider?

Answer:

> `NESTED`, when supported by the transaction manager and underlying resource, because it can use a savepoint and roll back to that savepoint while allowing the outer transaction to continue.

Don't answer `REQUIRES_NEW` automatically.

The distinction matters:

```text
REQUIRES_NEW
→ independent transaction

NESTED
→ same transaction + savepoint
```

---

# 30. Senior interview scenario

### Interviewer:

> Why not use `REQUIRES_NEW` for every operation that needs independent behavior?

Strong answer:

> Because `REQUIRES_NEW` suspends the existing transaction and starts another transaction. That can require additional database resources, increase connection pool usage, and introduce resource contention. It should be used when independent commit/rollback semantics are actually required, not as a generic way of isolating every method.

---

# 31. One subtle but important point

Propagation is **not the same thing as isolation**.

Propagation answers:

> **What should this method do with an existing transaction?**

Isolation answers:

> **How isolated should concurrent transactions be from one another?**

For example:

```java
@Transactional(
    propagation = Propagation.REQUIRES_NEW,
    isolation = Isolation.SERIALIZABLE
)
```

These are two completely different concerns.

```text
Propagation
→ transaction relationship

Isolation
→ concurrent transaction visibility
```

Do not mix them in an interview.

---

# 32. Interview-ready answer

### Q: What is transaction propagation in Spring?

> Transaction propagation defines how a transactional method behaves when it is called in the context of an existing transaction. `REQUIRED`, which is the default, joins an existing transaction or creates one if none exists. `REQUIRES_NEW` suspends the current transaction and creates an independent transaction. `NESTED` can create a nested transaction scope using savepoints when supported. `SUPPORTS` joins if a transaction exists but otherwise executes without one, while `NOT_SUPPORTED` suspends an existing transaction. `MANDATORY` requires an existing transaction, and `NEVER` requires that no transaction exists.

---

# 33. The mental model to remember

When you see:

```java
@Transactional(...)
```

ask two questions:

### Question 1

```text
Is there already a transaction?
```

### Question 2

```text
What does this propagation mode tell Spring to do?
```

Then derive the behavior.

For example:

```text
REQUIRED
Existing? → JOIN
Missing?  → CREATE

REQUIRES_NEW
Existing? → SUSPEND + CREATE
Missing?  → CREATE

SUPPORTS
Existing? → JOIN
Missing?  → NO TX

MANDATORY
Existing? → JOIN
Missing?  → FAIL

NEVER
Existing? → FAIL
Missing?  → EXECUTE
```

That is much better than trying to memorize seven isolated definitions.

---

# 34. Next: Transaction Isolation ⭐⭐⭐⭐⭐

Now we move to the next item in the transaction sequence:

**Isolation**

We will cover:

```text
Dirty Read
Non-repeatable Read
Phantom Read
        ↓
READ_UNCOMMITTED
READ_COMMITTED
REPEATABLE_READ
SERIALIZABLE
```

And, importantly, we will use **two concurrent transactions (T1/T2)** to show exactly what each isolation level allows and prevents.