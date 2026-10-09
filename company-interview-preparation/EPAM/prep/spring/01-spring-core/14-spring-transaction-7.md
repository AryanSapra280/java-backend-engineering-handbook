Great — now we’ll do **Transaction + Exception Behavior** and connect all the pieces we’ve covered so far. This is where the interview questions become scenario-based.

# 7. Transaction + Exception Behavior ⭐⭐⭐⭐⭐

The key idea is:

> An exception and a transaction rollback are related, but they are **not the same thing**.

When an exception occurs, you need to reason through:

```text
Exception
   ↓
Does it escape the transactional method?
   ↓
What exception type is it?
   ↓
What rollback rules apply?
   ↓
Was the transaction marked rollback-only?
   ↓
Commit or rollback?
```

---

# 1. Basic scenario

```java
@Transactional
public void processPayment() {

    paymentRepository.save(payment);

    throw new RuntimeException("Payment failed");
}
```

Flow:

```text
BEGIN
  ↓
save payment
  ↓
RuntimeException
  ↓
TransactionInterceptor
  ↓
Rollback rules
  ↓
ROLLBACK
```

Because `RuntimeException` triggers rollback by default.

---

# 2. What if the exception is caught?

```java
@Transactional
public void processPayment() {

    paymentRepository.save(payment);

    try {
        paymentGateway.call();
    } catch (RuntimeException e) {
        log.error("Payment failed", e);
    }
}
```

Now the exception doesn't escape the method.

Therefore Spring's transaction interceptor may see:

```text
method completed normally
```

So the transaction can commit.

```text
save payment
    ↓
exception
    ↓
caught
    ↓
method returns normally
    ↓
COMMIT
```

This is one of the most important traps.

---

# 3. Interview Question

### "If a RuntimeException happens inside a transactional method, will the transaction always roll back?"

**No.**

A better answer:

> "By default, a RuntimeException that escapes the transactional invocation triggers rollback. If the exception is caught inside the method and the transaction isn't otherwise marked rollback-only, Spring may commit the transaction."

That's the senior-level answer.

---

# 4. Explicitly marking rollback-only

If you intentionally catch the exception but still want rollback:

```java
@Transactional
public void processPayment() {

    try {

        paymentRepository.save(payment);

        paymentGateway.call();

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
Exception
   ↓
caught
   ↓
setRollbackOnly()
   ↓
method finishes
   ↓
Spring attempts commit
   ↓
rollback-only detected
   ↓
ROLLBACK
```

This is possible, but don't use it mechanically.

Often a cleaner design is to let the exception propagate.

---

# 5. Better approach: propagate the exception

Often this is cleaner:

```java
@Transactional
public void processPayment() {

    paymentRepository.save(payment);

    paymentGateway.call();
}
```

If:

```java
paymentGateway.call()
```

throws an appropriate unchecked exception:

```text
RuntimeException
    ↓
escapes method
    ↓
Spring sees it
    ↓
ROLLBACK
```

This keeps transaction behavior straightforward.

---

# 6. Checked exception scenario

```java
@Transactional
public void processPayment() throws PaymentException {

    paymentRepository.save(payment);

    throw new PaymentException();
}
```

If:

```java
class PaymentException extends Exception
```

then it is checked.

By default:

```text
PaymentException
     ↓
checked exception
     ↓
NO rollback by default
```

If rollback is required:

```java
@Transactional(
    rollbackFor = PaymentException.class
)
```

---

# 7. Nested transactional calls

Now the more interesting scenario.

```java
@Transactional
public void outer() {

    saveOrder();

    try {
        inner();
    } catch (Exception e) {
        log.error("Inner failed", e);
    }
}
```

And:

```java
@Transactional
public void inner() {

    savePayment();

    throw new RuntimeException();
}
```

Both use:

```text
Propagation.REQUIRED
```

Therefore:

```text
outer()
 ↓
T1
 ↓
inner()
 ↓
joins T1
```

The inner failure can mark:

```text
T1 = rollback-only
```

Even though the outer method catches the exception.

Eventually:

```text
outer()
 ↓
method completes
 ↓
attempt COMMIT
 ↓
T1 is rollback-only
 ↓
ROLLBACK
```

Potentially resulting in:

```text
UnexpectedRollbackException
```

---

# 8. Why `UnexpectedRollbackException`?

The outer method thinks:

```text
"I caught the exception, so everything is okay."
```

But the transaction says:

```text
"I have already been marked rollback-only."
```

Therefore:

```text
Outer method
     ↓
tries to commit
     ↓
Transaction says NO
     ↓
UnexpectedRollbackException
```

This is a very common senior interview scenario.

---

# 9. How can we avoid this?

It depends on the business requirement.

Suppose the inner operation is an independent audit:

```java
@Transactional
public void outer() {

    saveOrder();

    try {
        auditService.saveAudit();
    } catch (Exception e) {
        log.error("Audit failed", e);
    }
}
```

If audit must be independent:

```java
@Transactional(propagation = Propagation.REQUIRES_NEW)
public void saveAudit() {
    ...
}
```

Now:

```text
T1
 ↓
order
 ↓
suspend T1
 ↓
T2
 ↓
audit
 ↓
T2 commits/rolls back
 ↓
resume T1
```

This separates the transaction semantics.

---

# 10. Important distinction: `REQUIRED` vs `REQUIRES_NEW`

### REQUIRED

```text
Outer T1
   ↓
Inner joins T1
   ↓
Inner failure can affect T1
```

### REQUIRES_NEW

```text
Outer T1
   ↓
Suspend T1
   ↓
Inner T2
   ↓
T2 completes independently
   ↓
Resume T1
```

This is why `REQUIRES_NEW` is useful for independent audit/error logging.

---

# 11. What if the inner method catches its own exception?

Example:

```java
@Transactional
public void inner() {

    try {

        repository.save(data);

        throw new RuntimeException();

    } catch (RuntimeException e) {

        log.error("Error", e);
    }
}
```

If nothing marks the transaction rollback-only:

```text
exception
 ↓
caught
 ↓
method completes normally
 ↓
transaction can commit
```

Again:

> The exception itself doesn't magically guarantee rollback after it has been caught.

---

# 12. `rollbackFor` + caught exception

Consider:

```java
@Transactional(rollbackFor = IOException.class)
public void process() {

    try {
        throw new IOException();
    } catch (IOException e) {
        log.error("Error", e);
    }
}
```

Will `rollbackFor` automatically cause rollback?

Not simply because the exception occurred.

The exception was caught and did not escape the transactional invocation.

If you want rollback despite catching it, you need to explicitly mark the transaction rollback-only or allow the exception to propagate.

This is an important distinction.

---

# 13. Exception translation

Now let's connect this with Spring's `@Repository`.

Suppose your database layer encounters a persistence-specific exception.

Spring can translate certain persistence exceptions into Spring's `DataAccessException` hierarchy.

Conceptually:

```text
Database/JPA exception
       ↓
@Repository exception translation
       ↓
DataAccessException
       ↓
RuntimeException
```

This is one reason Spring's exception abstraction works nicely with transaction rollback rules.

Because `DataAccessException` is unchecked, it normally participates in default rollback behavior.

---

# 14. Practical example

```java
@Transactional
public void createPayment(Payment payment) {

    paymentRepository.save(payment);

    ledgerRepository.save(
        createLedgerEntry(payment)
    );
}
```

Suppose the ledger insert violates a constraint.

The persistence layer may produce a data-access exception.

Conceptually:

```text
Ledger INSERT
    ↓
Database constraint violation
    ↓
Spring data-access exception
    ↓
RuntimeException
    ↓
ROLLBACK
```

Therefore the payment insert is also rolled back if both are part of the same transaction.

---

# 15. What if you catch `DataAccessException`?

Example:

```java
@Transactional
public void createPayment() {

    paymentRepository.save(payment);

    try {
        ledgerRepository.save(entry);
    } catch (DataAccessException e) {
        log.error("Ledger failed", e);
    }
}
```

Now you've caught the exception.

If the transaction hasn't been marked rollback-only:

```text
payment save
 ↓
ledger fails
 ↓
exception caught
 ↓
method returns normally
 ↓
transaction may commit
```

This can produce a very dangerous partial business result.

Therefore, don't catch database exceptions casually inside a transaction.

---

# 16. Interview scenario

### Interviewer:

> I have `@Transactional`, and inside the method I catch `Exception` and log it. Why did my database changes commit even though there was an exception?

Strong answer:

> "Because the exception was caught before it escaped the transactional boundary. Spring's transaction interceptor may therefore see normal method completion and commit the transaction. If the business requirement is rollback, I should either allow the exception to propagate, configure the appropriate rollback rule for a propagated checked exception, or explicitly mark the current transaction rollback-only."

---

# 17. What if we throw another exception?

Example:

```java
@Transactional
public void process() {

    try {
        repository.save(data);
        externalCall();
    } catch (Exception e) {
        throw new PaymentProcessingException(e);
    }
}
```

Suppose:

```java
class PaymentProcessingException
        extends RuntimeException {
}
```

Then:

```text
original exception
      ↓
caught
      ↓
wrapped in RuntimeException
      ↓
RuntimeException escapes
      ↓
ROLLBACK
```

This is a common production pattern.

---

# 18. Wrapping a checked exception

Suppose:

```java
try {
    externalCall();
} catch (IOException e) {
    throw new PaymentProcessingException(e);
}
```

where:

```java
class PaymentProcessingException
        extends RuntimeException {
}
```

Now the transaction sees:

```text
PaymentProcessingException
        ↓
RuntimeException
        ↓
rollback by default
```

This can be useful when the business operation should be atomic despite the original checked exception.

But don't wrap exceptions just to force rollback without considering whether the exception should actually cause the transaction to fail.

---

# 19. Transaction + external exception

Consider:

```java
@Transactional
public void processPayment() {

    paymentRepository.save(payment);

    paymentGateway.charge();
}
```

Suppose the gateway call fails.

If it throws:

```text
RuntimeException
```

then the local DB transaction may roll back.

But:

> That does NOT mean the external payment system was rolled back.

For example:

```text
DB
 ↓
payment saved

External payment
 ↓
charge succeeds

Application
 ↓
exception occurs later
```

The database can roll back while the external charge remains successful.

This is the classic distributed consistency problem.

---

# 20. Senior-level answer

### Interviewer:

> "If the payment API call succeeds but your database transaction later rolls back, what happens?"

Answer:

> "The local database transaction can roll back, but the remote payment operation won't automatically be undone because it belongs to a different system and transaction boundary. I'd need an idempotent payment API and potentially a compensation mechanism or a workflow such as Saga. Depending on the architecture, an outbox pattern can also help reliably coordinate local state changes with events."

That's a strong microservices answer.

---

# 21. Exception vs rollback decision tree

When you encounter an exception, mentally execute:

```text
                Exception
                    │
                    ↓
       Did it escape the method?
             /             \
           YES              NO
            ↓                ↓
      Check exception      Transaction
          type              may commit
            │
      ┌─────┴─────┐
      ↓           ↓
 Runtime/Error   Checked
      ↓           ↓
 Rollback       rollbackFor?
                  │
             ┌────┴────┐
            YES        NO
             ↓          ↓
          Rollback    Normally
                      commit
```

And separately:

```text
Was transaction already marked rollback-only?
             ↓
            YES
             ↓
Cannot successfully commit
             ↓
Potential UnexpectedRollbackException
```

---

# 22. Very important: transaction rollback isn't exception handling

These are different responsibilities.

### Exception handling

Answers:

> What should the application do when something fails?

Examples:

```text
log
retry
return error
fallback
transform exception
```

### Transaction management

Answers:

> Should the database changes be committed or rolled back?

You can have:

```text
Exception handled
+
Transaction committed
```

or:

```text
Exception propagated
+
Transaction rolled back
```

or:

```text
Exception caught
+
Transaction explicitly marked rollback-only
```

Don't mix these concepts.

---

# 23. Production recommendation

Avoid this pattern unless you have a specific reason:

```java
@Transactional
public void process() {

    try {
        // everything
    } catch (Exception e) {
        log.error("Something failed", e);
    }
}
```

Why?

Because it can hide failures and unintentionally allow the transaction to commit.

Prefer targeted handling:

```java
@Transactional
public void process() {

    try {
        processOptionalOperation();
    } catch (SpecificRecoverableException e) {
        log.warn("Optional operation failed", e);
    }

    saveRequiredState();
}
```

Or allow business-critical exceptions to propagate.

---

# 24. Senior interview traps

### Trap 1

> RuntimeException always means rollback.

**Incomplete.**

More accurate:

> By default, an unchecked exception that escapes the transactional boundary triggers rollback.

---

### Trap 2

> If an exception happens, transaction rolls back.

**Not necessarily.**

It depends on:

- exception type
- rollback rules
- whether it escaped
- transaction status

---

### Trap 3

> Catching an exception means rollback.

❌ No.

It may actually allow commit.

---

### Trap 4

> `rollbackFor` causes rollback even if the exception is caught.

❌ Not simply because the exception occurred.

The rollback rule matters when Spring evaluates the transactional outcome; a caught exception that never reaches the interceptor doesn't automatically trigger it.

---

### Trap 5

> Rollback undoes external API calls.

❌ No.

Local transaction ≠ distributed transaction.

---

### Trap 6

> `REQUIRES_NEW` solves all exception problems.

❌ No.

It changes transaction boundaries; it doesn't magically provide distributed consistency.

---

# 25. Interview-ready answer

### Q: How does Spring decide whether to roll back when an exception occurs?

> Spring evaluates the transaction's rollback rules when the transactional invocation fails. By default, RuntimeException and Error trigger rollback, while checked exceptions generally do not. The behavior can be customized with `rollbackFor` and `noRollbackFor`. If an exception is caught inside the transactional method and doesn't mark the transaction rollback-only, Spring may commit because the method completed normally. Also, an inner transactional scope participating in the same transaction can mark the transaction rollback-only, which can later result in `UnexpectedRollbackException` when an outer scope attempts to commit.

---

# 26. Final mental model

Remember this sequence:

```text
@Transactional
      ↓
Transaction starts/joins
      ↓
Business code executes
      ↓
Exception?
      ↓
Was it caught?
      ↓
If propagated:
    check rollback rules
      ↓
rollback / commit
```

And:

```text
Caught exception
      ↓
May commit
```

while:

```text
rollback-only
      ↓
Cannot commit
```

and:

```text
Local rollback
      ≠
Remote system rollback
```

---

# Next: Transaction + Async / CompletableFuture ⭐⭐⭐⭐⭐

This is particularly important because you've already worked with:

```java
CompletableFuture
ExecutorService
custom thread pools
```

We'll answer the practical questions:

- Does `@Transactional` propagate to another thread?
- Why does `CompletableFuture` break the normal transaction context?
- What happens with `@Async`?
- What happens if the parent transaction commits before the async task executes?
- How do you correctly structure DB + async processing?
- Can `@Transactional` and `@Async` be put on the same method?
- Why does their proxy interaction matter?
- Production payment/order examples
- Senior interview scenarios