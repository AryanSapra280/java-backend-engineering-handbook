Absolutely — continuing **exactly from where we left off: Transactions → `@Transactional`**.

# Spring Transactions ⭐⭐⭐⭐⭐

Transactions are one of the most important Spring topics for a Senior Software Engineer interview because interviewers usually don't stop at:

> "What does `@Transactional` do?"

They typically continue with:

- How does it work internally?
- Where is the transaction actually started?
- Who commits it?
- Who rolls it back?
- What happens when an exception occurs?
- Why doesn't `@Transactional` work on self-invocation?
- What happens if the method is called from another thread?
- What happens with `CompletableFuture`?
- What happens when Kafka is involved?

We will build this from the ground up.

---

# 1. What is a transaction?

A **transaction** is a logical unit of work consisting of one or more database operations that should be treated as one consistent operation.

For example, transferring ₹1,000:

```text
Account A: -₹1,000
Account B: +₹1,000
```

These two operations should not be partially completed.

We want:

```text
Both succeed
       OR
Both rollback
```

Not:

```text
A debited
B failed
```

That is the fundamental purpose of a transaction.

---

# 2. What does `@Transactional` do?

`@Transactional` tells Spring:

> "Execute this method inside a transaction."

Example:

```java
@Service
public class PaymentService {

    @Transactional
    public void transferMoney() {

        debitAccount();

        creditAccount();
    }
}
```

Conceptually Spring does:

```text
Start Transaction
       ↓
debitAccount()
       ↓
creditAccount()
       ↓
Commit
```

If an appropriate exception occurs:

```text
Start Transaction
       ↓
debitAccount()
       ↓
creditAccount()
       ↓
Exception
       ↓
Rollback
```

The important point:

**`@Transactional` itself does not perform the transaction.**

It is metadata that tells Spring how the method should be executed.

Spring's transaction infrastructure performs the actual work.

---

# 3. Where should `@Transactional` normally be placed?

Usually at the **service layer**.

```java
@Service
public class OrderService {

    @Transactional
    public void placeOrder(Order order) {

        orderRepository.save(order);

        paymentRepository.save(payment);

        inventoryRepository.updateInventory();
    }
}
```

Why service layer?

Because the service method normally represents a **business operation**.

The transaction boundary should cover the complete business operation.

For example:

```text
Controller
    ↓
Service
    ↓
Repository
```

Usually:

```text
Controller
    ↓
@Transactional Service Method
    ↓
Repository
    ↓
Database
```

The service decides:

> "These operations together form one business transaction."

---

# 4. Interview Question: Can we put `@Transactional` on a repository method?

Yes.

For example:

```java
@Repository
public interface OrderRepository
        extends JpaRepository<Order, Long> {

    @Transactional
    void deleteByStatus(String status);
}
```

Spring Data supports transactional repository operations.

But for business-level transaction boundaries, the **service layer is generally preferred**.

Why?

Suppose:

```java
orderRepository.save(order);

paymentRepository.save(payment);

inventoryRepository.update(inventory);
```

If each repository operation manages its own transaction, you don't necessarily have one transaction covering the entire business operation.

Instead, you want:

```text
Service transaction
       |
       +---- Order repository
       |
       +---- Payment repository
       |
       +---- Inventory repository
```

So the entire operation succeeds or fails together.

---

# 5. What happens internally when we call a `@Transactional` method?

This is one of the most important interview questions.

Suppose we have:

```java
@Service
public class PaymentService {

    @Transactional
    public void processPayment() {

        paymentRepository.save(payment);

    }
}
```

You might imagine:

```text
processPayment()
    ↓
Spring sees @Transactional
    ↓
starts transaction
```

But that's not exactly how it happens.

Spring normally creates a **proxy** around the bean.

Instead of the caller directly holding:

```text
PaymentService
```

the caller effectively interacts with:

```text
PaymentService Proxy
        ↓
Actual PaymentService
```

So:

```text
Caller
  ↓
Proxy
  ↓
TransactionInterceptor
  ↓
Actual PaymentService
```

---

# 6. The transaction proxy

Conceptually:

```java
PaymentService proxy =
        createProxy(new PaymentService());
```

The caller invokes:

```java
proxy.processPayment();
```

The proxy intercepts the call.

It roughly performs:

```text
1. Start transaction
2. Invoke actual method
3. If successful → commit
4. If rollback-triggering exception → rollback
5. Return result / throw exception
```

Conceptually:

```java
beginTransaction();

try {

    target.processPayment();

    commit();

} catch (Exception e) {

    rollback();

    throw e;
}
```

This is simplified pseudocode, but it captures the core idea.

---

# 7. What is `TransactionInterceptor`?

Spring uses transaction interception infrastructure.

A key class involved is:

```text
TransactionInterceptor
```

It is an AOP interceptor.

Its responsibility is essentially:

> Intercept a method invocation and apply transaction semantics around it.

Conceptually:

```text
Caller
   ↓
Proxy
   ↓
TransactionInterceptor
   ↓
Target method
```

The interceptor determines things such as:

```text
Should a transaction be started?
Which propagation?
Which isolation?
Read-only?
Rollback rules?
```

Then it delegates transaction management to the configured transaction manager.

---

# 8. What is `PlatformTransactionManager`?

Spring provides the abstraction:

```java
PlatformTransactionManager
```

It represents the mechanism responsible for transaction management.

Different technologies can have different transaction managers.

For relational databases, a common implementation is:

```text
DataSourceTransactionManager
```

For JPA:

```text
JpaTransactionManager
```

So the architecture is approximately:

```text
@Transactional
      ↓
TransactionInterceptor
      ↓
PlatformTransactionManager
      ↓
Database/JPA transaction infrastructure
```

This abstraction is important because your business code doesn't need to manually manage transactions.

---

# 9. Full internal flow

Suppose:

```java
@Transactional
public void createOrder() {

    orderRepository.save(order);

    paymentRepository.save(payment);
}
```

The flow is approximately:

```text
Client
  ↓
Spring Proxy
  ↓
TransactionInterceptor
  ↓
TransactionManager
  ↓
BEGIN TRANSACTION
  ↓
createOrder()
  ↓
orderRepository.save()
  ↓
paymentRepository.save()
  ↓
Method returns successfully
  ↓
TransactionManager
  ↓
COMMIT
```

If an exception occurs:

```text
Client
  ↓
Proxy
  ↓
TransactionInterceptor
  ↓
BEGIN TRANSACTION
  ↓
createOrder()
  ↓
orderRepository.save()
  ↓
paymentRepository.save()
  ↓
Exception
  ↓
TransactionInterceptor
  ↓
ROLLBACK
  ↓
Exception propagated to caller
```

This is the mental model you should give in an interview.

---

# 10. Interview Question: Does Spring create a new database connection for every `@Transactional` method?

Not necessarily.

The important concept is that Spring obtains transaction resources through the configured transaction infrastructure.

With JDBC/JPA, the transaction is associated with a database connection/entity manager as appropriate.

Connection pooling may mean the physical connection is reused.

So don't say:

> "Every transaction creates a brand-new physical database connection."

Instead say:

> "Spring obtains the required transactional resource from the configured transaction infrastructure, typically using a connection/entity manager associated with the current transaction. Connection pooling can reuse physical database connections."

That's a much more senior answer.

---

# 11. Interview Question: Who actually commits the transaction?

Not `@Transactional`.

Not the repository.

The transaction infrastructure does it through the configured:

```text
PlatformTransactionManager
```

Conceptually:

```text
@Transactional
      ↓
TransactionInterceptor
      ↓
PlatformTransactionManager
      ↓
commit()
```

---

# 12. What happens before the method executes?

The transaction interceptor determines the transaction attributes.

For example:

```java
@Transactional(
    propagation = Propagation.REQUIRED,
    isolation = Isolation.READ_COMMITTED,
    readOnly = false
)
```

Spring uses this metadata to determine how the transaction should behave.

Then it asks the transaction manager to obtain/start the appropriate transaction.

Conceptually:

```text
@Transactional metadata
        ↓
Transaction attributes
        ↓
TransactionInterceptor
        ↓
TransactionManager
        ↓
Transaction started
```

---

# 13. What happens after successful execution?

Suppose:

```java
@Transactional
public void createOrder() {

    orderRepository.save(order);

}
```

If the method completes normally:

```text
Method returns
     ↓
Interceptor gets control
     ↓
Transaction commit
     ↓
Result returned to caller
```

So:

```text
Success → COMMIT
```

---

# 14. What happens when an exception occurs?

Suppose:

```java
@Transactional
public void createOrder() {

    orderRepository.save(order);

    throw new RuntimeException("Payment failed");
}
```

Conceptually:

```text
BEGIN
 ↓
save order
 ↓
RuntimeException
 ↓
ROLLBACK
```

But there's an important detail:

**Not every exception automatically causes rollback.**

This is extremely important and will be covered properly in the **rollback rules** section.

The default Spring behavior is generally:

```text
RuntimeException
Error
    → rollback

Checked Exception
    → generally no rollback by default
```

We will go deep into this later.

---

# 15. Why does `@Transactional` require a proxy?

Because Spring needs a point where it can intercept the method call.

Without interception, Spring wouldn't have an opportunity to do:

```text
BEGIN
 ↓
method
 ↓
COMMIT/ROLLBACK
```

The proxy gives Spring that interception point.

So the important chain is:

```text
Caller
  ↓
Proxy
  ↓
TransactionInterceptor
  ↓
Target method
```

This is exactly why the self-invocation problem exists.

---

# 16. The famous self-invocation problem

Consider:

```java
@Service
public class PaymentService {

    public void process() {
        validate();
        savePayment();
    }

    @Transactional
    public void savePayment() {
        // database operation
    }
}
```

You might expect:

```text
process()
   ↓
savePayment()
   ↓
transaction starts
```

But that's not necessarily what happens.

Inside the same object:

```java
this.savePayment();
```

is effectively called.

The call does **not go through the Spring proxy**.

So:

```text
External caller
      ↓
    Proxy
      ↓
Target
```

works.

But:

```text
Target
  ↓
this.savePayment()
```

bypasses the proxy.

Therefore the transactional interceptor doesn't get a chance to run.

---

# 17. Interview Question: How do you solve self-invocation?

Best solution:

Move the transactional operation into another bean.

```java
@Service
public class PaymentService {

    private final PaymentTransactionService transactionService;

    public PaymentService(
            PaymentTransactionService transactionService) {
        this.transactionService = transactionService;
    }

    public void process() {

        validate();

        transactionService.savePayment();
    }
}
```

And:

```java
@Service
public class PaymentTransactionService {

    @Transactional
    public void savePayment() {
        // DB operations
    }
}
```

Now:

```text
PaymentService
      ↓
Spring Proxy
      ↓
PaymentTransactionService
      ↓
@Transactional
```

The call crosses the proxy.

This is generally the cleanest solution.

---

# 18. Interview trap: "Is `@Transactional` a database feature?"

No.

`@Transactional` is a **Spring abstraction** for declarative transaction management.

The actual transaction is ultimately handled by the underlying resource/database technology.

Spring provides the abstraction:

```text
@Transactional
     ↓
Spring transaction infrastructure
     ↓
Database/JPA/JDBC
```

---

# 19. Interview Question: Can `@Transactional` be applied to a class?

Yes.

```java
@Service
@Transactional
public class PaymentService {

    public void createPayment() {
    }

    public void cancelPayment() {
    }
}
```

This means transactional behavior can apply to methods in the class according to Spring's transaction attribute resolution rules.

You can also override at method level:

```java
@Service
@Transactional
public class PaymentService {

    public void createPayment() {
    }

    @Transactional(readOnly = true)
    public Payment getPayment() {
        return paymentRepository.findById(id);
    }
}
```

The method-level configuration can provide more specific transaction behavior.

---

# 20. Interview Question: Does `private @Transactional` work?

This is a common trap.

```java
@Transactional
private void savePayment() {
}
```

You should **not rely on this**.

Spring's proxy-based transaction interception works with methods that can be intercepted through the proxy; private methods cannot be overridden/intercepted in the normal proxy mechanism.

Similarly, proxy-based interception has limitations around:

```text
private methods
self-invocation
final methods/classes depending on proxy mechanism
```

So the practical rule is:

> Put transactional boundaries on externally invoked service methods that can actually be intercepted by Spring.

---

# 21. Interview Question: What about `final` methods?

This depends on the proxy mechanism.

For class-based proxying, a `final` method cannot be overridden, so it cannot be intercepted in the normal CGLIB proxy mechanism.

For example:

```java
@Transactional
public final void process() {
}
```

Don't assume Spring can apply the normal proxy-based transaction interception to it.

This is another reason to avoid unnecessarily making transactional service methods `final`.

---

# 22. The senior-level mental model

Don't memorize:

> "`@Transactional` means rollback."

That's incomplete.

Memorize this:

```text
@Transactional
      ↓
Spring creates/interacts with a proxy
      ↓
TransactionInterceptor intercepts call
      ↓
Transaction attributes are resolved
      ↓
PlatformTransactionManager manages transaction
      ↓
Transaction begins / joins existing transaction
      ↓
Target method executes
      ↓
Success → commit
Failure → rollback according to rollback rules
```

This mental model will make **propagation, isolation, rollback rules, async, and Kafka** much easier.

---

# 23. Interview-ready answer

### Q: How does `@Transactional` work internally?

**Answer:**

> `@Transactional` is implemented primarily using Spring's proxy-based AOP infrastructure. When a transactional bean is created, Spring can expose it through a proxy. When a caller invokes a transactional method through that proxy, `TransactionInterceptor` intercepts the invocation and obtains the transaction configuration such as propagation, isolation, read-only status, and rollback rules. It then delegates transaction management to the configured `PlatformTransactionManager`. The transaction manager starts or joins a transaction, the target method executes, and based on the outcome Spring commits or rolls back the transaction according to the configured rollback rules. This is also why self-invocation can bypass `@Transactional`, because the internal call doesn't pass through the proxy.

That is a **Senior Software Engineer-level answer**.

---

# 24. One production example

Imagine your payment service:

```java
@Service
public class PaymentService {

    @Transactional
    public void processPayment(PaymentRequest request) {

        paymentRepository.save(
            createPayment(request)
        );

        accountRepository.debit(
            request.accountId(),
            request.amount()
        );

        ledgerRepository.createEntry(
            request.accountId(),
            request.amount()
        );
    }
}
```

The business requirement is:

```text
Payment record
      +
Account debit
      +
Ledger entry
```

should represent one atomic database operation.

If:

```text
paymentRepository.save()       SUCCESS
accountRepository.debit()      SUCCESS
ledgerRepository.createEntry() FAILURE
```

the transaction can roll back the previous database changes.

Final state:

```text
Payment       → rolled back
Account       → rolled back
Ledger        → rolled back
```

**assuming these operations participate in the same transaction/resource and the failure triggers rollback.**

This last qualification is important in senior interviews.

---

# 25. Very important limitation: transaction ≠ distributed transaction

Suppose:

```text
Payment DB
    ↓
Payment transaction

Kafka
    ↓
Event

Inventory DB
    ↓
Another transaction
```

`@Transactional` does **not automatically make all of these one atomic transaction**.

For example:

```java
@Transactional
public void payment() {

    paymentRepository.save(payment);

    kafkaTemplate.send("payment-created", event);
}
```

A database transaction and Kafka publication have different consistency concerns.

You cannot simply say:

> "`@Transactional` guarantees database + Kafka atomicity."

It doesn't.

This becomes extremely important in microservices.

We'll cover **Transaction + Kafka** separately.

---

# 26. Another important limitation: async execution

Suppose:

```java
@Transactional
public void process() {

    CompletableFuture.runAsync(() -> {

        repository.save(data);

    });
}
```

The transaction does **not automatically propagate to the asynchronous thread**.

Why?

Because Spring's normal transaction context is associated with the current execution/thread-bound transactional resources.

The async task runs on another thread:

```text
Thread A
@Transactional
   ↓
Transaction A
   ↓
submit async task
   ↓
method returns
   ↓
Transaction A commits


Thread B
CompletableFuture
   ↓
repository.save()
```

Thread B does not automatically inherit Thread A's transaction.

This is a very common senior-level interview question.

We'll cover it deeply in:

**Transaction + Async / CompletableFuture.**

---

# 27. What you should remember for the interview

```text
@Transactional
      ↓
Declarative transaction management
      ↓
Spring AOP proxy
      ↓
TransactionInterceptor
      ↓
PlatformTransactionManager
      ↓
BEGIN / JOIN
      ↓
Business method
      ↓
SUCCESS → COMMIT
FAILURE → ROLLBACK according to rules
```

And the three biggest traps:

```text
1. Self-invocation
   → bypasses proxy

2. Async / CompletableFuture
   → transaction doesn't automatically propagate

3. DB + Kafka
   → @Transactional doesn't automatically make them one atomic transaction
```

**Next topic in the exact sequence is: Transaction Boundaries — where a transaction actually starts/ends and how Spring determines what operations belong to it.**