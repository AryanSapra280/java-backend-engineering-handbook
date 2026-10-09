Absolutely. Now we’re at **Rollback Rules** — this is one of the most common places interviewers expose shallow understanding of `@Transactional`.

# 5. Transaction Rollback Rules ⭐⭐⭐⭐⭐

## 1. What does rollback mean?

Rollback means:

> Undo the database changes made within the current transaction.

Example:

```java
@Transactional
public void transferMoney() {

    debitAccount(1000);

    creditAccount(1000);
}
```

Suppose:

```text
debitAccount()   → SUCCESS
creditAccount()  → FAILURE
```

If the transaction is rolled back:

```text
Before:
Account A = ₹10,000
Account B = ₹5,000

After rollback:
Account A = ₹10,000
Account B = ₹5,000
```

The successful operation is also undone because both operations belong to the same transaction.

---

# 2. Does every exception cause rollback?

**No.**

This is one of the most important `@Transactional` interview questions.

By default, Spring generally rolls back for:

```text
RuntimeException
Error
```

but **not checked exceptions**.

So:

```text
RuntimeException → rollback by default
Error           → rollback by default
Checked Exception → no rollback by default
```

---

# 3. What is an unchecked exception?

Unchecked exceptions are exceptions derived from:

```java
RuntimeException
```

For example:

```java
IllegalArgumentException
NullPointerException
IllegalStateException
ArithmeticException
```

Example:

```java
@Transactional
public void processPayment() {

    paymentRepository.save(payment);

    throw new RuntimeException("Payment failed");
}
```

By default:

```text
save payment
    ↓
RuntimeException
    ↓
ROLLBACK
```

---

# 4. What is a checked exception?

A checked exception is an exception that extends `Exception` but is not a `RuntimeException`.

For example:

```java
IOException
SQLException
```

Example:

```java
@Transactional
public void processPayment() throws IOException {

    paymentRepository.save(payment);

    throw new IOException("File processing failed");
}
```

By default, Spring does **not** automatically roll back just because a checked exception was thrown.

Conceptually:

```text
save payment
    ↓
IOException
    ↓
method exits with checked exception
    ↓
NO ROLLBACK by default
```

That surprises many candidates.

---

# 5. Interview Question: Why doesn't Spring roll back checked exceptions by default?

The important answer is:

> Spring's default rollback policy treats unchecked exceptions and `Error` as indicators of unexpected failure, while checked exceptions are generally considered conditions that application code may be expected to handle.

Therefore Spring doesn't automatically assume that every checked exception means the transaction should be discarded.

The key interview fact is:

```text
Default:
RuntimeException / Error → rollback
Checked Exception        → commit
```

assuming no other rollback-only condition exists.

---

# 6. Example of the dangerous case

Consider:

```java
@Transactional
public void createOrder() throws Exception {

    orderRepository.save(order);

    throw new Exception("Something went wrong");
}
```

A common incorrect answer is:

> "The transaction will roll back because an exception occurred."

Not necessarily.

Because `Exception` here is a checked exception.

By default:

```text
save order
    ↓
checked Exception
    ↓
transaction may COMMIT
```

That can be dangerous if your business requirement expects rollback.

---

# 7. `rollbackFor`

Spring lets you explicitly specify which exceptions should trigger rollback.

Example:

```java
@Transactional(rollbackFor = Exception.class)
public void createOrder() throws Exception {

    orderRepository.save(order);

    throw new Exception("Something went wrong");
}
```

Now:

```text
Exception
   ↓
matches rollbackFor
   ↓
ROLLBACK
```

So:

```text
Default:
Checked Exception → no rollback

With rollbackFor:
Checked Exception → rollback
```

---

# 8. Real production example

Suppose:

```java
@Transactional
public void processFile() throws IOException {

    saveBatchRecord();

    processFileContent();

    updateStatus();
}
```

If `processFileContent()` throws:

```java
IOException
```

and your business requirement is:

> "If file processing fails, none of the database changes should remain."

Then:

```java
@Transactional(rollbackFor = IOException.class)
```

is appropriate.

```java
@Transactional(rollbackFor = IOException.class)
public void processFile() throws IOException {
    ...
}
```

Now `IOException` triggers rollback.

---

# 9. `rollbackFor` can accept multiple exceptions

Example:

```java
@Transactional(
    rollbackFor = {
        IOException.class,
        SQLException.class
    }
)
public void process() throws IOException, SQLException {
}
```

Now both exception types are rollback triggers.

---

# 10. `rollbackFor = Exception.class`

You may see:

```java
@Transactional(rollbackFor = Exception.class)
```

This means:

> Roll back for `Exception` and its applicable subclasses.

So it can include checked exceptions as well as runtime exceptions.

However, don't blindly add:

```java
rollbackFor = Exception.class
```

to every method.

Transaction behavior should reflect the actual business semantics.

---

# 11. `noRollbackFor`

There is also the opposite:

```java
@Transactional(noRollbackFor = SomeException.class)
```

Meaning:

> Even though this exception would normally cause rollback, don't treat this exception as a rollback trigger.

Example:

```java
@Transactional(
    noRollbackFor = PaymentAlreadyProcessedException.class
)
public void processPayment() {
    ...
}
```

Suppose:

```java
paymentRepository.save(payment);

throw new PaymentAlreadyProcessedException();
```

If configured this way, that exception doesn't automatically mark the transaction for rollback.

---

# 12. `rollbackFor` vs `noRollbackFor`

Think:

```text
rollbackFor
→ "This exception should cause rollback."

noRollbackFor
→ "This exception should NOT cause rollback."
```

Example:

```java
@Transactional(
    rollbackFor = IOException.class,
    noRollbackFor = PaymentAlreadyProcessedException.class
)
```

The exact behavior depends on the exception types and Spring's rollback rule resolution, so avoid creating contradictory configurations unnecessarily.

---

# 13. Interview Question: What happens if you catch the exception?

This is extremely important.

Consider:

```java
@Transactional
public void processPayment() {

    try {

        paymentRepository.save(payment);

        throw new RuntimeException("Payment failed");

    } catch (RuntimeException e) {

        log.error("Payment failed", e);
    }
}
```

The exception never escapes the transactional method.

So you might think:

```text
Exception caught
    ↓
No exception reaches Spring
    ↓
COMMIT
```

And **yes, if nothing else marks the transaction rollback-only**, Spring may commit.

This is a common mistake.

---

# 14. Catching an exception does not automatically mean rollback

Example:

```java
@Transactional
public void process() {

    saveA();

    try {
        saveB();
    } catch (RuntimeException e) {
        log.error("Failed", e);
    }

    saveC();
}
```

If you catch the exception and don't mark the transaction rollback-only:

```text
saveA
 ↓
saveB fails
 ↓
exception caught
 ↓
saveC
 ↓
COMMIT
```

Potentially.

Therefore:

> **Exception occurrence and transaction rollback are related, but not identical.**

The transaction interceptor needs to know whether the transaction should be rolled back.

---

# 15. How can we explicitly mark rollback-only?

Spring provides:

```java
TransactionAspectSupport.currentTransactionStatus()
    .setRollbackOnly();
```

Example:

```java
@Transactional
public void process() {

    try {

        savePayment();

    } catch (RuntimeException e) {

        log.error("Payment failed", e);

        TransactionAspectSupport
            .currentTransactionStatus()
            .setRollbackOnly();
    }
}
```

Now:

```text
Exception caught
      ↓
setRollbackOnly()
      ↓
method continues
      ↓
transaction attempts commit
      ↓
rollback instead
```

This is an advanced mechanism.

In production, however, first consider whether propagating the exception or redesigning the flow is cleaner.

---

# 16. `rollback-only` is an important concept

Think of it as:

> "This transaction is no longer allowed to commit successfully."

Suppose:

```text
T1
 ↓
operation A
 ↓
operation B fails
 ↓
T1 marked rollback-only
 ↓
exception caught
 ↓
method continues
 ↓
attempt COMMIT
```

The transaction manager sees:

```text
rollbackOnly = true
```

and rolls it back.

---

# 17. The famous `UnexpectedRollbackException`

Now consider:

```java
@Transactional
public void outer() {

    try {
        inner();
    } catch (Exception e) {
        log.error("Inner failed", e);
    }
}
```

Inner:

```java
@Transactional
public void inner() {

    saveSomething();

    throw new RuntimeException();
}
```

Both use:

```text
REQUIRED
```

Therefore they participate in the same transaction.

The inner failure can cause:

```text
T1 → rollback-only
```

Outer catches the exception:

```text
outer()
 ↓
exception caught
 ↓
continues
```

Then outer tries to complete successfully.

But:

```text
T1 = rollback-only
```

Therefore Spring cannot commit it.

You may get:

```text
UnexpectedRollbackException
```

The important lesson:

> Catching the original exception does not necessarily reset the transaction's rollback-only status.

---

# 18. Why is this a senior-level interview question?

Because it tests whether you understand:

```text
Exception
   ↓
TransactionInterceptor
   ↓
rollback rules
   ↓
transaction status
   ↓
commit / rollback
```

rather than simply memorizing:

> "RuntimeException causes rollback."

---

# 19. Exception hierarchy matters

Suppose:

```java
class PaymentException extends RuntimeException {
}
```

Then:

```java
@Transactional
public void process() {
    throw new PaymentException();
}
```

By default:

```text
PaymentException
     ↓
RuntimeException
     ↓
rollback
```

Similarly, if:

```java
class PaymentException extends Exception {
}
```

then it's checked:

```text
PaymentException
     ↓
Exception
     ↓
no rollback by default
```

unless configured using:

```java
rollbackFor = PaymentException.class
```

---

# 20. Interview Question: Does `rollbackFor` mean the exception must be thrown?

The rollback rule is evaluated based on the exception that escapes the transactional invocation, subject to the transaction's status and other rules.

For example:

```java
@Transactional(rollbackFor = IOException.class)
public void process() throws IOException {

    throw new IOException();
}
```

The exception reaches the transaction interceptor.

Spring sees that it matches the rollback rule.

Therefore:

```text
IOException
    ↓
rollbackFor match
    ↓
ROLLBACK
```

---

# 21. What if `rollbackFor` is configured on a class?

Example:

```java
@Transactional(rollbackFor = Exception.class)
@Service
public class PaymentService {
}
```

The configuration can apply to methods according to Spring's transaction attribute resolution rules.

A method can provide more specific transaction configuration.

For example:

```java
@Transactional(rollbackFor = IOException.class)
public void processFile() {
}
```

Method-level configuration can override or refine class-level transactional metadata as applicable.

---

# 22. Rollback rules and inheritance

Suppose:

```java
class PaymentException extends IOException {
}
```

and:

```java
@Transactional(rollbackFor = IOException.class)
```

Then a thrown `PaymentException` is also covered because it is a subtype of `IOException`.

So rollback rules aren't just exact string comparisons.

The exception type hierarchy matters.

---

# 23. Interview trap: `Error`

Spring's default rollback rules also include:

```text
Error
```

For example:

```java
OutOfMemoryError
```

But you should **not** build application-level business logic around catching or recovering from JVM-level `Error`s.

The interview point is simply:

```text
RuntimeException → default rollback
Error            → default rollback
Checked Exception → default no rollback
```

---

# 24. Practical payment example

Suppose:

```java
@Transactional
public void transferMoney() {

    debitAccount();

    creditAccount();

    ledgerRepository.save(entry);
}
```

Now:

```text
debit       → success
credit      → success
ledger      → RuntimeException
```

Because `RuntimeException` normally triggers rollback:

```text
debit       → rollback
credit      → rollback
ledger      → rollback
```

The database returns to the transaction's previous consistent state.

---

# 25. Checked exception example

Now:

```java
@Transactional
public void transferMoney() throws IOException {

    debitAccount();

    creditAccount();

    writeAuditFile();

    throw new IOException();
}
```

Without explicit configuration:

```text
debit       → success
credit      → success
IOException → checked exception
```

The transaction may commit.

If business requirements demand rollback:

```java
@Transactional(rollbackFor = IOException.class)
```

Then:

```text
IOException
     ↓
rollback rule matches
     ↓
ROLLBACK
```

---

# 26. `noRollbackFor` practical example

Suppose an operation can raise a business notification:

```java
class NotificationException extends RuntimeException {
}
```

But the business requirement says:

> "Even if notification fails, the order database changes should still commit."

Then:

```java
@Transactional(
    noRollbackFor = NotificationException.class
)
public void createOrder() {

    orderRepository.save(order);

    notificationService.notifyCustomer();
}
```

Now the notification exception doesn't automatically cause the transaction to roll back.

But this design should be evaluated carefully. If the notification is important, an outbox/event-driven approach is often more reliable than deliberately committing while ignoring the failure.

---

# 27. Interview Question: Should we use `noRollbackFor` to solve every exception problem?

No.

If an external notification or Kafka event is important, simply saying:

```java
noRollbackFor = ...
```

doesn't guarantee reliable delivery.

For cross-system reliability, you may need:

```text
Outbox pattern
Retry
Dead-letter handling
Idempotency
Eventual consistency
```

This becomes particularly important in microservice architecture.

---

# 28. Rollback and database state

Suppose:

```java
@Transactional
public void process() {

    repository.save(A);

    repository.save(B);

    throw new RuntimeException();
}
```

Assuming both writes participate in the same transaction:

```text
Before:
A = old
B = old

Transaction:
A = new
B = new

Exception
 ↓
ROLLBACK

After:
A = old
B = old
```

The transaction provides atomicity.

---

# 29. Important limitation

Rollback only affects resources participating in the transaction.

For example:

```java
@Transactional
public void process() {

    repository.save(order);

    Files.write(...);

    throw new RuntimeException();
}
```

The database change can roll back.

But a filesystem write is **not automatically undone**.

Similarly:

```text
Database
→ transaction

HTTP API
→ separate system

Kafka
→ separate messaging system

File system
→ separate resource
```

A local Spring transaction does not magically make all these operations atomic.

---

# 30. Senior interview scenario

### Interviewer:

> Your method is `@Transactional`. It calls a payment API, writes to the database, then throws an exception. Will everything roll back?

Strong answer:

> "The database operations participating in the transaction can roll back according to the transaction's rollback rules. But the external payment API call is not automatically rolled back by the local database transaction. If the payment API has already processed the payment, the exception in our service doesn't undo it. For such workflows I would consider an idempotent API, compensation, saga, or an appropriate event/outbox architecture."

This is exactly the kind of distinction expected at senior level.

---

# 31. Common rollback mistakes

### Mistake 1

```java
@Transactional
public void process() throws Exception {
    ...
}
```

Assuming every `Exception` causes rollback.

❌ Not by default.

---

### Mistake 2

```java
try {
    ...
} catch (Exception e) {
    log.error(...);
}
```

Assuming Spring will still automatically roll back.

❌ Not necessarily.

---

### Mistake 3

Using:

```java
rollbackFor = Exception.class
```

everywhere without understanding business semantics.

❌ Overly broad transaction policy can hide design problems.

---

### Mistake 4

Using:

```java
noRollbackFor
```

to pretend external systems are transactional.

❌ It only affects Spring's rollback decision for that transaction.

---

### Mistake 5

Assuming rollback undoes everything the method did.

❌ Rollback applies to resources participating in that transaction, not arbitrary side effects.

---

# 32. Interview-ready answer

### Q: What are Spring's default rollback rules?

> By default, Spring rolls back a transaction when a `RuntimeException` or `Error` escapes the transactional method. Checked exceptions do not normally trigger rollback by default. We can customize this using `rollbackFor` or `rollbackForClassName`, and we can explicitly exclude exception types using `noRollbackFor` or `noRollbackForClassName`. If an exception is caught inside the transactional method and doesn't propagate or mark the transaction rollback-only, the transaction may still commit.

---

# 33. Interview-ready answer: Why did `UnexpectedRollbackException` occur?

> `UnexpectedRollbackException` can occur when an inner operation participating in the same transaction marks the transaction as rollback-only, but the outer method catches the original exception and continues as though the transaction were still committable. When the outer transaction eventually tries to commit, Spring detects that it has been marked rollback-only and cannot commit it, so it reports an unexpected rollback.

---

# 34. Final mental model

When an exception occurs inside `@Transactional`, think:

```text
Exception
    ↓
Is it caught?
    ↓
Does it escape the transactional boundary?
    ↓
What type is it?
    ↓
RuntimeException/Error?
    ↓
Default rollback
```

or:

```text
Checked Exception
    ↓
rollbackFor configured?
    ↓
YES → rollback
NO  → normally no rollback
```

And remember:

```text
Caught exception
      ↓
Transaction may still commit

rollback-only
      ↓
Transaction cannot successfully commit
```

---

# Next: Read-Only Transactions

The next item is:

## 6. Read-Only Transactions

We'll cover:

- `@Transactional(readOnly = true)`
- What it actually means
- What it does with JPA/Hibernate
- Does it prevent writes?
- Does it improve performance?
- JDBC/database behavior
- `readOnly` vs database-level read-only
- Common misconceptions
- Production use cases
- Interview traps