
# Common `@Transactional` Mistakes + Senior Production Scenarios ⭐⭐⭐⭐⭐

This section is where we move from **knowing transactions** to being able to explain what can actually go wrong in production.

---

# PART 1 — Common `@Transactional` Mistakes

## 1. Calling a `@Transactional` method from the same class

This is one of the most important Spring transaction traps.

```java
@Service
public class PaymentService {

    public void processPayment() {
        savePayment();
    }

    @Transactional
    public void savePayment() {
        paymentRepository.save(...);
    }
}
```

You might think:

```text
processPayment()
      ↓
savePayment()
      ↓
@Transactional starts transaction
```

But Spring's `@Transactional` is normally implemented using a **proxy**.

The call:

```java
savePayment();
```

is a direct call on the same object.

It does **not** go through the Spring proxy.

Therefore the transactional interceptor is bypassed.

### Interview answer

> `@Transactional` is proxy-based by default. Self-invocation bypasses the Spring proxy, so the transactional advice is not applied to the internal method call.

### Better solution

Move the transactional operation into another bean:

```java
@Service
public class PaymentService {

    private final PaymentRepository paymentRepository;

    public void processPayment() {
        paymentRepositoryService.savePayment();
    }
}
```

```java
@Service
public class PaymentRepositoryService {

    @Transactional
    public void savePayment() {
        paymentRepository.save(...);
    }
}
```

Now:

```text
PaymentService
      ↓
Spring Proxy
      ↓
PaymentRepositoryService
      ↓
@Transactional
```

---

# 2. Making a `@Transactional` method `private`

```java
@Transactional
private void savePayment() {
}
```

This is problematic because Spring's proxy-based interception cannot normally intercept private methods.

Use:

```java
@Transactional
public void savePayment() {
}
```

or an appropriately visible method that can be intercepted through the proxy.

### Interview trap

> Does putting `@Transactional` on any method automatically create a transaction?

**No.**

The method has to be invoked through the Spring-managed proxy in the normal proxy-based model.

---

# 3. Putting `@Transactional` on the wrong layer

Suppose:

```java
@Controller
public class PaymentController {

    @Transactional
    public void createPayment() {
        ...
    }
}
```

Technically it can be proxied, but this is usually a poor transaction boundary.

Better:

```text
Controller
    ↓
Service
    ↓
Repository
```

with:

```java
@Service
public class PaymentService {

    @Transactional
    public void createPayment() {
        ...
    }
}
```

### Why?

The service layer represents a **business operation**.

For example:

```text
createPayment()
    ↓
validate
    ↓
save payment
    ↓
update ledger
    ↓
record audit
```

These operations should normally belong to one business transaction.

### Interview answer

> Transaction boundaries should generally be placed around service-layer business operations rather than individual repository calls because the service defines the unit of work.

---

# 4. Catching an exception and expecting rollback

Consider:

```java
@Transactional
public void createPayment() {

    try {
        paymentRepository.save(payment);

        processSomething();

    } catch (Exception e) {
        log.error("Payment failed", e);
    }
}
```

The exception is caught.

The method returns normally.

Therefore Spring may see:

```text
Method completed successfully
        ↓
COMMIT
```

even though something failed.

### Better

If you catch the exception but still want rollback:

```java
@Transactional
public void createPayment() {

    try {
        paymentRepository.save(payment);
        processSomething();

    } catch (Exception e) {

        log.error("Payment failed", e);

        throw e;
    }
}
```

Or explicitly mark rollback-only where appropriate.

### Important

Logging an exception does not cause rollback.

```java
catch (Exception e) {
    log.error(...);
}
```

is not equivalent to:

```text
ROLLBACK
```

---

# 5. Assuming checked exceptions automatically roll back

By default, Spring rolls back for:

```text
RuntimeException
Error
```

but not every checked exception.

Example:

```java
@Transactional
public void process() throws IOException {

    paymentRepository.save(payment);

    throw new IOException();
}
```

The checked exception may not trigger rollback by default.

Use:

```java
@Transactional(rollbackFor = IOException.class)
```

when that behavior is required.

---

# 6. Using `REQUIRES_NEW` without understanding its consequences

Consider:

```java
@Transactional
public void processPayment() {

    paymentRepository.save(payment);

    auditService.saveAudit();
}
```

and:

```java
@Transactional(propagation = Propagation.REQUIRES_NEW)
public void saveAudit() {
    auditRepository.save(...);
}
```

Now there are two transactions:

```text
Transaction T1
    |
    | payment save
    |
    ↓
Suspend T1
    |
    ↓
Transaction T2
    |
    | audit save
    |
    ↓
Commit T2
    |
    ↓
Resume T1
```

If T1 later rolls back:

```text
T1 → ROLLBACK
T2 → already COMMITTED
```

The audit remains.

That might be intentional.

But if you didn't understand `REQUIRES_NEW`, you could accidentally create inconsistent data.

---

# 7. Using `REQUIRES_NEW` everywhere

This is dangerous.

Imagine:

```text
Main transaction
    ↓
REQUIRES_NEW
    ↓
REQUIRES_NEW
    ↓
REQUIRES_NEW
```

Each transaction can require another database connection.

With a limited connection pool:

```text
Connection Pool = 10

T1 → connection 1
T2 → connection 2
T3 → connection 3
...
```

Under load, you can get connection starvation.

### Senior answer

> `REQUIRES_NEW` should be used intentionally because it suspends the existing transaction and usually requires another physical transaction/connection. Excessive use can increase connection usage and create pool exhaustion.

---

# 8. Assuming `readOnly = true` means "writes are impossible"

```java
@Transactional(readOnly = true)
public Payment getPayment(String id) {
    return repository.findById(id).orElseThrow();
}
```

`readOnly=true` is primarily a **hint/optimization**, not a universal security mechanism preventing writes.

Depending on the database and ORM:

```text
readOnly
    ↓
optimization
    ↓
less unnecessary work
```

But don't think:

```text
readOnly = true
        ↓
database physically rejects every write
```

Not universally.

---

# 9. Assuming singleton beans are thread-safe because transactions exist

Spring services are usually singleton beans.

For example:

```java
@Service
public class PaymentService {

    private int count;

    @Transactional
    public void process() {
        count++;
    }
}
```

`@Transactional` does **not** make:

```java
count++;
```

thread-safe.

Multiple requests can execute concurrently.

```text
Thread 1 → count++
Thread 2 → count++
Thread 3 → count++
```

Use proper concurrency control when shared mutable state exists.

---

# 10. Assuming transaction context follows `CompletableFuture`

This is a mistake we already discussed.

```java
@Transactional
public void process() {

    paymentRepository.save(payment);

    CompletableFuture.runAsync(() -> {
        paymentRepository.updateSomething();
    });
}
```

The worker thread does not automatically inherit the caller's transaction.

```text
Thread A
---------
Transaction T1
    ↓
save payment
    ↓
submit async task
    ↓
COMMIT


Thread B
---------
async task
    ↓
NO T1
```

If the async work needs its own transaction:

```java
@Async
@Transactional
public void processAsync() {
    ...
}
```

it can establish a separate transaction on the worker thread.

---

# 11. Returning `CompletableFuture` does not keep the transaction open

Consider:

```java
@Transactional
public CompletableFuture<Void> process() {

    paymentRepository.save(payment);

    return CompletableFuture.runAsync(() -> {
        doSomething();
    });
}
```

The transaction belongs to the synchronous method invocation.

It does not automatically remain open until the future completes.

Conceptually:

```text
T1
 |
 | save
 |
 | return CompletableFuture
 |
COMMIT
 |
Worker continues separately
```

This is an extremely useful interview distinction.

---

# 12. Calling external APIs inside a database transaction

Example:

```java
@Transactional
public void processPayment() {

    paymentRepository.save(payment);

    paymentGateway.chargeCard();

    ledgerRepository.save(ledger);
}
```

Suppose the payment gateway takes:

```text
8 seconds
```

The database transaction remains open while waiting.

That can cause:

```text
long-running transaction
        ↓
connection held
        ↓
locks held
        ↓
connection pool pressure
        ↓
lower throughput
```

### Better architecture

Keep database transactions short.

For example:

```text
Payment request
     ↓
Persist payment
     ↓
Commit
     ↓
Publish event
     ↓
Payment Gateway
     ↓
Update status
```

depending on the business consistency requirements.

---

# 13. Calling Kafka inside a DB transaction and assuming atomicity

Example:

```java
@Transactional
public void createPayment() {

    paymentRepository.save(payment);

    kafkaTemplate.send("payment-created", event);
}
```

This does **not automatically mean**:

```text
DB COMMIT
+
Kafka COMMIT
```

as one atomic operation.

This is the classic dual-write problem.

For reliable DB → Kafka publication:

```text
DB transaction
   ↓
Payment + Outbox
   ↓
COMMIT
   ↓
Outbox Publisher
   ↓
Kafka
```

---

# 14. `@Transactional` on methods that perform no meaningful transaction work

For example:

```java
@Transactional
public Payment getPayment(String id) {
    return repository.findById(id).orElseThrow();
}
```

This can be perfectly valid.

But don't blindly put:

```java
@Transactional
```

on every method.

Transactions have costs:

```text
transaction
connection
resource management
possible locks
transaction synchronization
```

Use them around meaningful units of work.

---

# 15. Long-running transactions

Bad:

```java
@Transactional
public void processMillionRecords() {

    for (...) {
        process();
    }
}
```

Potential problems:

```text
Huge transaction
      ↓
Long-held connection
      ↓
Long locks
      ↓
Large persistence context
      ↓
Memory pressure
      ↓
Rollback becomes expensive
```

For batch processing, consider appropriate chunking:

```text
Chunk 1 → transaction → commit
Chunk 2 → transaction → commit
Chunk 3 → transaction → commit
```

This is particularly important in large ETL/batch systems.

---

# PART 2 — Senior-Level Transaction Scenarios

Now let's turn those concepts into the kind of questions an EPAM interviewer can ask.

---

# Scenario 1 — Payment DB update + external payment gateway

### Interviewer

> You have a payment API. You need to update the payment database and call an external payment gateway. Would you put `@Transactional` on the whole method?

### Strong answer

Not blindly.

If I do:

```java
@Transactional
public void pay() {

    paymentRepository.save(payment);

    paymentGateway.charge();

    paymentRepository.updateStatus();
}
```

the database transaction can remain open while waiting for the external service.

That can cause:

- long-running transactions
- connection pool exhaustion
- locks being held longer
- poor throughput

I would usually separate the external interaction from the DB transaction and design the workflow around explicit payment states and asynchronous processing where appropriate.

Example:

```text
PENDING
   ↓
Payment Gateway
   ↓
SUCCESS / FAILED
```

---

# Scenario 2 — Payment saved but event not published

### Interviewer

> Payment is committed in PostgreSQL but Kafka publishing fails. How do you solve it?

### Answer

Use the Outbox Pattern.

```text
BEGIN
    INSERT payment
    INSERT outbox_event
COMMIT
```

Then:

```text
Outbox
   ↓
Publisher
   ↓
Kafka
```

If Kafka is temporarily unavailable:

```text
outbox_event = PENDING
```

and the publisher retries.

Consumer idempotency is still required because duplicate delivery is possible.

---

# Scenario 3 — Two users update the same account

Suppose:

```text
Account balance = 1000
```

Two requests arrive simultaneously:

```text
Thread A → withdraw 700
Thread B → withdraw 500
```

Both read:

```text
balance = 1000
```

Without proper concurrency control:

```text
A → 1000 - 700 = 300
B → 1000 - 500 = 500
```

The final result may incorrectly become:

```text
500
```

even though both operations were accepted.

### Possible solutions

Optimistic locking:

```java
@Version
private Long version;
```

or pessimistic locking:

```java
@Lock(LockModeType.PESSIMISTIC_WRITE)
```

or atomic database updates:

```sql
UPDATE account
SET balance = balance - :amount
WHERE id = :id
AND balance >= :amount;
```

The correct choice depends on contention and business requirements.

---

# Scenario 4 — Inventory overselling

### Interviewer

> Inventory is 1. Two users purchase simultaneously. How do you prevent inventory from becoming -1?

One solution is an atomic update:

```sql
UPDATE inventory
SET quantity = quantity - 1
WHERE product_id = ?
AND quantity > 0;
```

Then check affected rows:

```text
1 row updated → success
0 rows updated → insufficient inventory
```

This is often better than:

```text
SELECT quantity
UPDATE quantity
```

because the condition and modification happen atomically at the database level.

---

# Scenario 5 — `UnexpectedRollbackException`

Suppose:

```java
@Transactional
public void process() {

    try {
        serviceA();
    } catch (Exception e) {
        log.error("serviceA failed");
    }

    serviceB();
}
```

Inside `serviceA()`:

```java
@Transactional
public void serviceA() {

    repository.save(...);

    throw new RuntimeException();
}
```

Suppose the participating transaction becomes:

```text
rollback-only
```

The outer method catches the exception and continues.

At the end:

```text
Outer method appears successful
        ↓
Spring tries COMMIT
        ↓
Transaction is rollback-only
        ↓
UnexpectedRollbackException
```

### Interview answer

> Catching an exception does not necessarily clear the transaction's rollback-only state. If an inner transactional operation marks the transaction rollback-only, the outer transaction can fail during commit and Spring may throw `UnexpectedRollbackException`.

---

# Scenario 6 — Audit should survive business rollback

Suppose:

```text
Payment transaction fails
```

but you still want:

```text
Audit record = persisted
```

Use:

```java
@Transactional(propagation = Propagation.REQUIRES_NEW)
public void saveAudit(...) {
    auditRepository.save(...);
}
```

Flow:

```text
T1
Payment
  ↓
failure
  ↓
suspend T1

T2
Audit
  ↓
COMMIT

resume T1
  ↓
ROLLBACK
```

Result:

```text
Payment → ROLLBACK
Audit   → COMMIT
```

Use this intentionally because it introduces a separate transaction and additional resource usage.

---

# Scenario 7 — Async notification after transaction

### Interviewer

> Payment is saved in a transaction. You want to send a notification asynchronously only after the payment commits. How would you approach it?

A naive solution:

```java
@Transactional
public void payment() {

    paymentRepository.save(payment);

    notificationService.sendAsync();
}
```

is risky because the async task could execute before the DB transaction commits.

Better:

```text
DB Transaction
      ↓
COMMIT
      ↓
AFTER_COMMIT
      ↓
Async notification
```

Spring provides:

```java
@TransactionalEventListener(
    phase = TransactionPhase.AFTER_COMMIT
)
```

combined with appropriate asynchronous processing.

For reliable cross-service messaging, however, an Outbox/Kafka approach is often stronger than relying only on an in-process event.

---

# Scenario 8 — Kafka consumer crashes after DB update

Flow:

```text
Kafka event
   ↓
Consumer
   ↓
DB update SUCCESS
   ↓
Application crashes
   ↓
offset not committed
```

Kafka may redeliver.

Therefore:

```text
Consumer MUST be idempotent
```

For example:

```text
eventId = E123

processed_events
----------------
E123
```

On duplicate:

```text
E123 already exists
      ↓
ignore/reconcile
```

---

# Scenario 9 — Large batch transaction

### Interviewer

> You need to process 10 million records. Would you put the entire method under one `@Transactional`?

Usually no.

Bad:

```text
BEGIN
10 million records
COMMIT
```

Problems:

- huge transaction
- long-running locks
- large persistence context
- rollback of enormous work
- database pressure
- poor recovery

Better:

```text
1000 records
    ↓
transaction
    ↓
commit

1000 records
    ↓
transaction
    ↓
commit

...
```

This is one reason Spring Batch uses transaction boundaries around chunks.

---

# Scenario 10 — Retry + transaction

### Interviewer

> What happens if a transactional method is also retried?

Suppose:

```java
@Retryable
@Transactional
public void process() {
    ...
}
```

You must understand the proxy/interceptor ordering and transaction boundaries.

Conceptually, you may have:

```text
Retry
  ↓
Transaction
  ↓
Business logic
```

A failed attempt may roll back its transaction, and the retry can execute the operation again in a new transactional attempt.

This is exactly why retrying a non-idempotent operation can be dangerous.

Example:

```text
Attempt 1
    charge card
    ↓
    timeout

Retry

Attempt 2
    charge card again
```

The customer may be charged twice.

Therefore:

> **Retries must be combined with idempotency for operations that have side effects.**

This is a major production principle.

---

# Scenario 11 — `@Transactional` + `@Async`

### Interviewer

> If the parent method is transactional and calls an `@Async` method, does the async method participate in the same transaction?

**No.**

Different thread:

```text
Thread A
---------
Transaction T1
    ↓
save()
    ↓
@Async
    ↓
COMMIT


Thread B
---------
Async method
    ↓
separate thread
    ↓
doesn't inherit T1
```

If the async method has:

```java
@Async
@Transactional
```

it can establish its own transaction.

But that transaction is independent.

---

# Scenario 12 — Transaction isolation choice

### Interviewer

> When would you use `SERIALIZABLE`?

Answer:

> When the business operation requires the strongest isolation and I need to prevent concurrency anomalies such as dirty reads, non-repeatable reads and phantoms, and I'm willing to accept the lower concurrency and potentially higher locking/contention cost.

Don't say:

> "Always use SERIALIZABLE because it's safest."

That is not a production answer.

Usually we choose the **lowest isolation level that satisfies the business consistency requirement**.

---

# Scenario 13 — Optimistic vs pessimistic locking

### Interviewer

> Which would you choose for a highly contested account balance?

### Optimistic locking

```text
Read version 5
      ↓
Update WHERE version = 5
      ↓
version becomes 6
```

If someone changed it first:

```text
version != 5
      ↓
update fails
```

Good when conflicts are relatively uncommon.

### Pessimistic locking

```text
SELECT ... FOR UPDATE
```

The row is locked while the transaction operates.

Good when contention is high and serialization is required, but it can increase lock contention and deadlock risk.

### Senior answer

> The choice depends on contention, transaction duration, retry tolerance, and business semantics. I would generally prefer optimistic locking when conflicts are relatively rare and use pessimistic locking when strong serialization is required for a highly contended resource.

---

# Scenario 14 — Deadlock

Two transactions:

```text
T1:
lock A
 ↓
lock B

T2:
lock B
 ↓
lock A
```

Result:

```text
T1 waits for B
T2 waits for A
```

Database detects the deadlock and aborts one transaction.

### How would you reduce deadlocks?

- Keep transactions short.
- Acquire locks in a consistent order.
- Avoid unnecessary locking.
- Keep queries/indexes efficient.
- Avoid external calls inside transactions.
- Retry safely where appropriate.

Again:

> Retry requires idempotency if the transaction performs externally visible side effects.

---

# Scenario 15 — What should a senior engineer consider when designing a transaction?

When I see a transaction boundary, I ask:

```text
1. What business operation must be atomic?
2. Which database records must change together?
3. How long will the transaction remain open?
4. What locks can be acquired?
5. What isolation level is actually required?
6. Could concurrent requests modify the same data?
7. Can this operation be retried safely?
8. Are there external API calls?
9. Are there Kafka events?
10. What happens if the process crashes?
11. What happens if the consumer receives the event twice?
12. Do we need idempotency?
13. Could this transaction cause connection-pool pressure?
14. Is REQUIRES_NEW really necessary?
15. Is asynchronous processing involved?
```

This is the level of thinking expected from a senior engineer.

---

# FINAL INTERVIEW CHEAT SHEET

### `@Transactional` doesn't work when:

```text
❌ self-invocation
❌ private method
❌ object isn't Spring-managed
❌ misunderstanding proxy boundaries
```

### Be careful with:

```text
⚠ checked exceptions
⚠ caught exceptions
⚠ rollback-only
⚠ REQUIRES_NEW
⚠ long-running transactions
⚠ external API calls
⚠ @Async
⚠ CompletableFuture
⚠ Kafka dual writes
⚠ retries
⚠ shared mutable state
```

### Production architecture principles

```text
Short transactions
        +
Correct isolation
        +
Correct concurrency control
        +
Idempotency
        +
Outbox for DB → Kafka
        +
Retries with backoff
        +
Observability
```

### Most important senior-level statement

> **A transaction is not just an annotation. It is a boundary around a business unit of work, and a senior engineer must reason about concurrency, failure, retries, external systems, transaction duration, isolation, idempotency, and recovery.**