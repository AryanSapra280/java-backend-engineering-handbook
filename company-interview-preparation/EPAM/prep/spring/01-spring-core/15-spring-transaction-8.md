Next: **Transaction + Async / `CompletableFuture`**. This is especially important for your EPAM prep because you’ve already worked with custom executors and `CompletableFuture`.

# 8. Transaction + Async / CompletableFuture ⭐⭐⭐⭐⭐

## 1. The fundamental problem

Suppose we have:

```java
@Transactional
public void processPayment() {

    paymentRepository.save(payment);

    CompletableFuture.runAsync(() -> {
        ledgerRepository.save(ledgerEntry);
    });
}
```

A common assumption is:

> "The async operation is inside the transactional method, so it should participate in the same transaction."

**That's generally wrong.**

The async task executes on a **different thread**.

---

# 2. Why doesn't the transaction automatically propagate?

Spring's normal transaction context is associated with the current execution thread through transaction synchronization/resource management.

Conceptually:

```text
Thread A
────────────────────────────

@Transactional
       ↓
Transaction T1
       ↓
paymentRepository.save()
       ↓
submit CompletableFuture
       ↓
method returns
       ↓
T1 commits
```

Meanwhile:

```text
Thread B
────────────────────────────

CompletableFuture
       ↓
ledgerRepository.save()
```

Thread B is a different execution context.

It does not automatically inherit Thread A's transaction.

So:

```text
Thread A
   ↓
Transaction T1

Thread B
   ↓
No automatic T1
```

---

# 3. The most important rule

Memorize this:

> **A Spring transaction does not automatically propagate across threads.**

So:

```text
@Transactional
+
CompletableFuture
```

does **not** mean:

```text
same transaction
```

---

# 4. Practical example

Consider:

```java
@Service
public class PaymentService {

    @Transactional
    public void processPayment(Payment payment) {

        paymentRepository.save(payment);

        CompletableFuture.runAsync(() -> {
            ledgerRepository.save(
                createLedgerEntry(payment)
            );
        });
    }
}
```

Execution:

```text
Thread A
    ↓
BEGIN T1
    ↓
save payment
    ↓
start async task
    ↓
processPayment() returns
    ↓
COMMIT T1
```

At the same time:

```text
Thread B
    ↓
ledgerRepository.save()
```

The async operation is not part of T1.

---

# 5. What if the parent transaction rolls back?

Suppose:

```java
@Transactional
public void processPayment() {

    paymentRepository.save(payment);

    CompletableFuture.runAsync(() -> {
        ledgerRepository.save(entry);
    });

    throw new RuntimeException();
}
```

Potential sequence:

```text
Thread A
    ↓
T1 starts
    ↓
save payment
    ↓
start async task
    ↓
RuntimeException
    ↓
ROLLBACK T1
```

Meanwhile Thread B may execute:

```text
Thread B
    ↓
ledgerRepository.save()
    ↓
its own transaction / transactional behavior
    ↓
COMMIT
```

You can end up with:

```text
Payment → rolled back
Ledger  → committed
```

That is a consistency problem.

---

# 6. Why can this happen?

Because the two operations aren't participating in the same transaction.

Think:

```text
T1
├── payment save
└── rollback

T2
└── ledger save
    └── commit
```

The transaction boundary is tied to the parent thread's execution.

---

# 7. Interview Question

### "Does `@Transactional` propagate to `CompletableFuture`?"

Strong answer:

> "No, not automatically. Spring's transaction context is normally associated with the current thread, while `CompletableFuture` may execute on another thread. The asynchronous operation therefore does not automatically participate in the caller's transaction. If the async operation needs transactional behavior, it should generally establish its own transaction in the async execution context."

That is the answer you want in an interview.

---

# 8. How do we give the async operation its own transaction?

Put the transaction on the method that executes in the async thread.

For example:

```java
@Service
public class LedgerService {

    @Transactional
    public void createLedgerEntry(LedgerEntry entry) {
        ledgerRepository.save(entry);
    }
}
```

Then:

```java
@Service
public class PaymentService {

    public void processPayment(Payment payment) {

        paymentRepository.save(payment);

        CompletableFuture.runAsync(() ->
            ledgerService.createLedgerEntry(entry)
        );
    }
}
```

Now the async method is independently transactional.

Conceptually:

```text
Thread A
   ↓
Payment operation
```

and:

```text
Thread B
   ↓
LedgerService proxy
   ↓
@Transactional
   ↓
Transaction T2
   ↓
Ledger save
   ↓
COMMIT
```

---

# 9. Important: Don't put the transaction only on the parent

This does **not** solve the async transaction problem:

```java
@Transactional
public void process() {

    CompletableFuture.runAsync(() -> {
        repository.save(data);
    });
}
```

The transaction belongs to the parent execution.

It doesn't magically move to the worker thread.

Instead, if the async operation requires a transaction:

```java
@Transactional
public void asyncOperation() {
    ...
}
```

should be on the **service method invoked by the worker thread**, and that call should go through the Spring proxy.

---

# 10. `@Async`

Spring provides:

```java
@Async
```

for asynchronous execution.

Example:

```java
@Async
public CompletableFuture<Void> processLedger() {
    ...
}
```

Conceptually:

```text
Caller Thread
      ↓
Spring Async Proxy
      ↓
Executor
      ↓
Worker Thread
```

Again:

```text
Worker Thread ≠ Caller Thread
```

Therefore the caller's transaction does not automatically propagate to the worker.

---

# 11. Can we use `@Transactional` and `@Async` together?

Technically, yes, but you need to understand what they mean.

For example:

```java
@Async
@Transactional
public void processLedger() {
    ledgerRepository.save(entry);
}
```

The important question is:

> Which thread executes the transactional method?

The async proxy causes the work to execute on a worker thread, and the transactional interceptor can establish a transaction for that worker-thread execution.

Conceptually:

```text
Caller
   ↓
Async Proxy
   ↓
Executor
   ↓
Worker Thread
   ↓
Transaction Proxy/Interceptor
   ↓
BEGIN T2
   ↓
DB operations
   ↓
COMMIT T2
```

The important point is:

> This is an **independent transaction**, not the caller's transaction.

---

# 12. Proxy ordering matters

Spring features such as:

```text
@Async
@Transactional
@Cacheable
@Retryable
```

are often implemented using proxies/interceptors.

So when multiple annotations are combined, proxy/interceptor ordering can matter.

For interview purposes, don't simply say:

> "`@Async` and `@Transactional` always work in exactly the same way."

Instead explain the execution model:

```text
Caller
 ↓
Spring proxy/interceptor chain
 ↓
Async execution
 ↓
worker thread
 ↓
transactional method execution
```

The exact proxy ordering depends on configuration and infrastructure.

---

# 13. The self-invocation problem appears again

Consider:

```java
@Service
public class PaymentService {

    public void process() {
        asyncProcess();
    }

    @Async
    public void asyncProcess() {
        ...
    }
}
```

Will `asyncProcess()` necessarily execute asynchronously?

Not through Spring's normal proxy mechanism.

Why?

Because:

```java
this.asyncProcess();
```

is a direct internal call.

It bypasses the Spring proxy.

So:

```text
Same bean
process()
   ↓
this.asyncProcess()
   ↓
NO @Async proxy interception
```

This is the same fundamental problem we saw with:

```text
@Transactional
```

and:

```text
@Async
```

Both rely on proxy-based interception in the standard setup.

---

# 14. Correct design

Move the async operation to another bean:

```java
@Service
public class PaymentService {

    private final LedgerAsyncService ledgerAsyncService;

    public void process() {

        ledgerAsyncService.processLedger();
    }
}
```

Then:

```java
@Service
public class LedgerAsyncService {

    @Async
    @Transactional
    public void processLedger() {

        ledgerRepository.save(entry);
    }
}
```

Now:

```text
PaymentService
      ↓
Spring Proxy
      ↓
LedgerAsyncService
      ↓
@Async
      ↓
Executor
      ↓
Worker Thread
      ↓
@Transactional
      ↓
T2
```

This is much cleaner.

---

# 15. Interview Question

### "If my outer transaction rolls back, will my `@Async` method roll back too?"

No.

Example:

```java
@Transactional
public void process() {

    saveOrder();

    asyncService.sendEvent();

    throw new RuntimeException();
}
```

The outer transaction can roll back:

```text
T1 → ROLLBACK
```

But the async operation may execute independently:

```text
T2 → COMMIT
```

Therefore:

```text
Outer DB change → rolled back
Async DB change  → may commit
```

---

# 16. A subtle timing problem

Consider:

```java
@Transactional
public void process() {

    saveOrder();

    asyncService.process();
}
```

The parent transaction hasn't committed yet.

But the async thread might execute immediately.

So:

```text
Thread A
T1
 ↓
save order
 ↓
async submit
 ↓
still inside T1
```

Thread B:

```text
T2
 ↓
tries to read order
```

Depending on isolation and database behavior, Thread B may **not see the uncommitted order**.

This is another important point:

> Starting an async task before the parent transaction commits does not mean the async task sees the parent's uncommitted database state.

---

# 17. Dangerous example

```java
@Transactional
public void createOrder() {

    orderRepository.save(order);

    asyncService.publishOrderCreated(order.getId());
}
```

Suppose async service does:

```java
@Async
public void publishOrderCreated(Long orderId) {

    Order order = orderRepository.findById(orderId)
                                 .orElseThrow();
}
```

Potentially:

```text
T1
 ↓
INSERT order
 ↓
not committed yet

T2
 ↓
SELECT order
 ↓
cannot see uncommitted row
 ↓
not found
```

Depending on timing and isolation.

This is why asynchronous work often needs to happen **after the transaction commits** when it depends on committed database state.

---

# 18. Better pattern: after transaction commit

A common Spring mechanism is to execute work after successful transaction completion.

For example, using a transactional event listener:

```java
@Transactional
public void createOrder() {

    orderRepository.save(order);

    publisher.publishEvent(
        new OrderCreatedEvent(order.getId())
    );
}
```

Then:

```java
@TransactionalEventListener(
    phase = TransactionPhase.AFTER_COMMIT
)
public void handleOrderCreated(OrderCreatedEvent event) {

    // async/event processing
}
```

Now the conceptual flow is:

```text
T1
 ↓
save order
 ↓
publish application event
 ↓
COMMIT
 ↓
AFTER_COMMIT listener
 ↓
async processing
```

This avoids starting dependent asynchronous work against uncommitted state.

There are still delivery/reliability considerations, which become important in event-driven architectures.

---

# 19. Transaction + CompletableFuture return value

Suppose:

```java
@Transactional
public CompletableFuture<Payment> process() {

    paymentRepository.save(payment);

    return CompletableFuture.supplyAsync(() -> {

        return paymentRepository.findById(payment.getId())
                                .orElseThrow();
    });
}
```

The transaction does **not** stay open until the `CompletableFuture` completes.

This is a very important point.

The transactional method returns:

```text
CompletableFuture
```

and Spring's transaction interceptor considers the method invocation complete.

Conceptually:

```text
T1
 ↓
method starts
 ↓
submit CompletableFuture
 ↓
method returns Future
 ↓
T1 commits
```

Then:

```text
Worker Thread
 ↓
Future computation
```

The future's lifetime does not automatically extend the transaction.

---

# 20. This is a common interview trap

### Wrong:

> "Because the method returns a CompletableFuture, the transaction remains open until the future completes."

❌ No.

The transaction boundary is based on the intercepted method invocation, not on the eventual completion of an arbitrary asynchronous computation.

---

# 21. How should you structure it?

Instead of trying to make one transaction span asynchronous work:

```text
T1
 ↓
async
 ↓
???
```

usually define explicit transaction boundaries:

```text
Request
 ↓
Transaction T1
 ↓
Commit
 ↓
Async task
 ↓
Transaction T2
```

For example:

```java
@Service
public class PaymentService {

    @Transactional
    public void createPayment(Payment payment) {

        paymentRepository.save(payment);

        eventPublisher.publishEvent(
            new PaymentCreatedEvent(payment.getId())
        );
    }
}
```

Then process the event asynchronously after commit.

---

# 22. Production architecture

For a payment system:

```text
HTTP Request
      ↓
Payment Service
      ↓
@Transactional
      ↓
Save Payment
      ↓
Save Ledger State
      ↓
COMMIT
      ↓
Async/Event Processing
      ↓
Notification / downstream processing
```

This is generally much safer than:

```text
Transaction
   ↓
Async task
   ↓
hope both remain consistent
```

---

# 23. Transaction + ExecutorService

You may have:

```java
ExecutorService executor =
        Executors.newFixedThreadPool(10);
```

and:

```java
@Transactional
public void process() {

    repository.save(data);

    executor.submit(() -> {
        repository.save(otherData);
    });
}
```

Same principle applies.

The executor creates worker threads.

The transaction doesn't automatically propagate.

```text
Caller Thread
→ T1

Worker Thread
→ no automatic T1
```

---

# 24. What about `ThreadLocal`?

Spring transaction infrastructure uses thread-associated resources/context.

That's why moving execution to another thread matters.

Conceptually:

```text
Thread A
ThreadLocal / transaction-associated resources
        ↓
T1
```

When work moves to:

```text
Thread B
```

the same transaction context is not automatically present.

Don't oversimplify this as:

> "Transactions are literally stored in one ThreadLocal."

Spring uses transaction synchronization/resource management mechanisms, with thread-bound resources being a key part of the standard model.

The interview-level takeaway is:

> **Spring's normal transaction context is thread-bound and doesn't automatically cross asynchronous thread boundaries.**

---

# 25. Should we manually copy the transaction context to another thread?

Generally:

**No.**

Trying to force a database transaction across arbitrary asynchronous threads is usually a sign that the transaction boundary needs redesign.

Instead:

```text
Parent transaction
→ complete business operation
→ commit
→ async operation
→ own transaction
```

is usually cleaner.

For distributed workflows:

```text
Outbox
Saga
Events
Idempotency
```

may be more appropriate.

---

# 26. Interview scenario

### Interviewer:

> I have `@Transactional` on a method. Inside it I call `CompletableFuture.runAsync()`. The async task updates the same database. Is the update part of the same transaction?

Answer:

> "No, not automatically. `CompletableFuture` executes on another thread, while Spring's normal transaction context is thread-bound. The asynchronous operation therefore doesn't automatically participate in the caller's transaction. If the async operation needs transactional semantics, I would give the worker operation its own transaction, typically through another Spring bean. If it depends on data committed by the parent transaction, I'd trigger it after commit rather than before."

Excellent senior-level answer.

---

# 27. Interview scenario

### Interviewer:

> Why did my async method not see the record inserted by the parent transaction?

Answer:

> "The parent transaction may not have committed yet when the async thread attempted the read. The async thread doesn't share the parent's transaction, so it generally cannot see the parent's uncommitted changes. I would execute the asynchronous work after successful transaction commit if it depends on committed state."

---

# 28. Interview scenario

### Interviewer:

> Can I put `@Async` and `@Transactional` on the same method?

Answer:

> "Yes, but they have different responsibilities. `@Async` moves execution to an executor thread, while `@Transactional` establishes transaction semantics for that execution. The transaction created there is independent of the caller's transaction; it doesn't propagate the caller's transaction. Because both rely on Spring interception, proxying and self-invocation also need to be considered."

---

# 29. Interview scenario

### Interviewer:

> I return a `CompletableFuture` from a `@Transactional` method. Does the transaction remain open until `future.get()` completes?

Answer:

> "No. The transaction normally ends when the intercepted transactional method invocation returns. Returning a `CompletableFuture` doesn't automatically extend the transaction until asynchronous completion. If the future executes on another thread, it also doesn't automatically inherit the original transaction."

---

# 30. Production recommendation

Avoid designs like:

```java
@Transactional
public void process() {

    saveA();

    CompletableFuture.runAsync(() -> {
        saveB();
    });

    saveC();
}
```

when you require:

```text
A + B + C
```

to be atomic.

You don't have one transaction covering them.

Instead, decide explicitly:

```text
A + C
→ Transaction T1

B
→ Async Transaction T2
```

or redesign the workflow so that the required state changes happen synchronously in T1 and asynchronous downstream processing happens after commit.

---

# 31. Senior-level mental model

Remember:

```text
@Transactional
      ↓
Thread A
      ↓
Transaction T1
```

Then:

```text
CompletableFuture
      ↓
Thread B
      ↓
NOT automatically T1
```

Therefore:

```text
Async work
      ↓
own transaction if needed
```

And if it depends on committed data:

```text
T1
 ↓
COMMIT
 ↓
AFTER_COMMIT
 ↓
Async
 ↓
T2
```

---

# 32. The biggest traps

### Trap 1

> `@Transactional` automatically propagates to `CompletableFuture`.

❌ No.

### Trap 2

> Returning `CompletableFuture` keeps the transaction open.

❌ No.

### Trap 3

> `@Async` shares the caller's transaction.

❌ No.

### Trap 4

> Async task can always immediately read data saved by the parent.

❌ Not before the parent commits.

### Trap 5

> `@Transactional` + `@Async` means one transaction.

❌ Usually it means the async execution can establish its **own** transaction.

### Trap 6

> `this.asyncMethod()` will always trigger `@Async`.

❌ Self-invocation bypasses the Spring proxy.

---

# 33. Interview-ready answer

### Q: Does a Spring transaction propagate to another thread?

> "No, not automatically. Spring's standard transaction context is associated with the current execution thread and its transaction resources. When execution moves to another thread through `@Async`, `CompletableFuture`, or an `ExecutorService`, the worker doesn't automatically participate in the caller's transaction. If the worker needs database transaction semantics, I would establish a separate transaction in the worker's service method. If the worker depends on data written by the parent transaction, I'd normally trigger it after the parent transaction successfully commits."

---

# 34. Final architecture to remember

For a senior-level microservice:

```text
HTTP Request
     ↓
Payment Service
     ↓
@Transactional
     ↓
DB changes
     ↓
COMMIT
     ↓
Event / AFTER_COMMIT
     ↓
Async processing
     ↓
Worker Thread
     ↓
@Transactional
     ↓
Independent DB transaction
```

This is the clean mental model.

---

# Next: Transaction + Kafka ⭐⭐⭐⭐⭐

The next topic is especially important for your background with **Kafka + Spring Boot microservices**.

We'll cover:

- `@Transactional` + Kafka
- DB transaction vs Kafka transaction
- Why DB commit + Kafka publish is a consistency problem
- What happens when DB succeeds but Kafka fails
- What happens when Kafka succeeds but DB rolls back
- `KafkaTransactionManager`
- Chained/distributed transaction misconceptions
- Transactional Kafka producer
- Consumer transaction
- Offset commit behavior
- Outbox pattern
- Why outbox is often preferred
- Idempotency and duplicate events
- Senior-level payment/ledger scenarios