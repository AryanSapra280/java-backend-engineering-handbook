Perfect. Next in the sequence: **Transaction Boundaries**. This is where interviewers start testing whether you actually understand transactions rather than just knowing the annotation.

# 2. Transaction Boundaries ⭐⭐⭐⭐⭐

## 1. What is a transaction boundary?

A **transaction boundary** defines:

> Where a transaction starts and where it ends.

For a method annotated with:

```java
@Transactional
public void placeOrder() {
    saveOrder();
    reserveInventory();
    createPayment();
}
```

the typical boundary is:

```text
BEGIN TRANSACTION
       ↓
placeOrder()
       ↓
saveOrder()
       ↓
reserveInventory()
       ↓
createPayment()
       ↓
COMMIT
```

If rollback is required:

```text
BEGIN
  ↓
placeOrder()
  ↓
exception
  ↓
ROLLBACK
```

So the transaction boundary surrounds the **entire method invocation** intercepted by Spring.

---

# 2. Why does transaction boundary matter?

Suppose you have:

```java
public void placeOrder() {

    saveOrder();

    makePayment();

    updateInventory();
}
```

You need to decide:

> Should all three operations be part of the same transaction?

Usually, if they use the same transactional database and must succeed/fail together:

```text
placeOrder()
 ├── saveOrder()
 ├── makePayment()
 └── updateInventory()
```

should have one transaction.

```java
@Transactional
public void placeOrder() {
    saveOrder();
    makePayment();
    updateInventory();
}
```

Now:

```text
┌──────────── Transaction ────────────┐
│                                     │
│ saveOrder()                         │
│ makePayment()                       │
│ updateInventory()                   │
│                                     │
└─────────────────────────────────────┘
```

---

# 3. Where should we define the transaction boundary?

Usually at the **service layer**.

Example:

```java
@Service
public class OrderService {

    @Transactional
    public void placeOrder(OrderRequest request) {

        orderRepository.save(...);

        paymentRepository.save(...);

        inventoryRepository.update(...);
    }
}
```

Why service?

Because the service usually represents a **business operation**.

The controller shouldn't normally manage transactions:

```java
@RestController
public class OrderController {

    @Transactional   // generally not preferred
    @PostMapping("/orders")
    public void createOrder() {
    }
}
```

Instead:

```text
Controller
    ↓
Service
    ↓
Repository
```

and:

```text
Controller
    ↓
@Transactional Service Method
    ↓
Repositories
```

The service owns the business transaction.

---

# 4. Interview Question: Does every repository call start a new transaction?

No.

This is a very important distinction.

Suppose:

```java
@Transactional
public void createOrder() {

    orderRepository.save(order);

    paymentRepository.save(payment);

    inventoryRepository.save(inventory);
}
```

Normally, these repository operations participate in the **existing transaction** started for `createOrder()`.

Conceptually:

```text
Transaction T1
│
├── orderRepository.save()
│
├── paymentRepository.save()
│
└── inventoryRepository.save()
```

They don't necessarily create:

```text
T1 → order
T2 → payment
T3 → inventory
```

Instead, they participate in the transaction context established by the service method, subject to the propagation configuration.

This leads directly to the next major topic: **transaction propagation**.

---

# 5. Transaction boundary vs method boundary

These are not always the same concept.

Suppose:

```java
@Transactional
public void processOrder() {

    validateOrder();

    saveOrder();

    sendNotification();
}
```

The transaction boundary is approximately:

```text
BEGIN
 ↓
processOrder()
 ↓
validateOrder()
 ↓
saveOrder()
 ↓
sendNotification()
 ↓
COMMIT
```

The internal methods don't automatically create separate transactions just because they're separate Java methods.

The boundary is determined by the transactional interception and propagation configuration.

---

# 6. Interview Question: What if a non-transactional method calls a transactional method?

Example:

```java
public void process() {
    paymentService.savePayment();
}
```

And:

```java
@Transactional
public void savePayment() {
    repository.save(payment);
}
```

If `savePayment()` is invoked **through the Spring proxy**, Spring can start a transaction when entering `savePayment()`.

```text
process()
   ↓
paymentService proxy
   ↓
@Transactional savePayment()
   ↓
BEGIN
   ↓
DB operation
   ↓
COMMIT
```

So a transaction does not have to start at the controller.

It starts when execution crosses the relevant transactional proxy boundary.

---

# 7. But what if the call is inside the same class?

Example:

```java
@Service
public class PaymentService {

    public void process() {
        savePayment();
    }

    @Transactional
    public void savePayment() {
        repository.save(payment);
    }
}
```

The call is effectively:

```java
this.savePayment();
```

So it bypasses the proxy.

Therefore:

```text
process()
   ↓
this.savePayment()
   ↓
target method directly
```

instead of:

```text
process()
   ↓
proxy
   ↓
TransactionInterceptor
   ↓
savePayment()
```

Therefore the expected transactional boundary may not be created.

This is the **self-invocation problem** we discussed earlier.

---

# 8. Transaction boundary example

Consider:

```java
@Transactional
public void transferMoney() {

    debitAccount();

    creditAccount();
}
```

Transaction:

```text
                 TRANSACTION
┌─────────────────────────────────────┐
│                                     │
│ BEGIN                               │
│                                     │
│ debitAccount()                      │
│                                     │
│ creditAccount()                     │
│                                     │
│ COMMIT                              │
│                                     │
└─────────────────────────────────────┘
```

If:

```java
creditAccount();
```

throws a rollback-triggering exception:

```text
BEGIN
 ↓
debitAccount()
 ↓
creditAccount()
 ↓
Exception
 ↓
ROLLBACK
```

The debit is rolled back as part of the same transaction.

---

# 9. Transaction boundary and propagation

Now consider:

```java
@Service
public class OrderService {

    @Transactional
    public void placeOrder() {

        orderRepository.save(order);

        paymentService.processPayment();
    }
}
```

And:

```java
@Service
public class PaymentService {

    @Transactional
    public void processPayment() {

        paymentRepository.save(payment);
    }
}
```

Question:

> Do we have two transactions?

Not necessarily.

With the default propagation:

```java
Propagation.REQUIRED
```

the behavior is:

```text
OrderService.placeOrder()
        ↓
Transaction T1 starts
        ↓
PaymentService.processPayment()
        ↓
Existing T1 found
        ↓
Join T1
        ↓
paymentRepository.save()
        ↓
return
        ↓
T1 commits
```

So:

```text
        Transaction T1
┌─────────────────────────────┐
│ placeOrder()                │
│     ↓                       │
│ processPayment()            │
│     ↓                       │
│ paymentRepository.save()    │
└─────────────────────────────┘
```

This is the foundation of **propagation**.

---

# 10. What happens if the inner method uses `REQUIRES_NEW`?

Now:

```java
@Transactional
public void placeOrder() {

    orderRepository.save(order);

    paymentService.processPayment();
}
```

and:

```java
@Transactional(propagation = Propagation.REQUIRES_NEW)
public void processPayment() {

    paymentRepository.save(payment);
}
```

Now the behavior changes.

```text
Outer transaction T1
       ↓
orderRepository.save()
       ↓
processPayment()
       ↓
Suspend T1
       ↓
Start T2
       ↓
paymentRepository.save()
       ↓
Commit T2
       ↓
Resume T1
       ↓
Commit/Rollback T1
```

So:

```text
T1
┌──────────────────────────────┐
│ placeOrder                   │
│                              │
│ orderRepository.save()       │
│                              │
│      T1 suspended            │
└───────────┬──────────────────┘
            │
            ↓
          T2
┌──────────────────────────────┐
│ processPayment               │
│ paymentRepository.save()     │
│ COMMIT                       │
└───────────┬──────────────────┘
            │
            ↓
        T1 resumes
```

This is one of the most common propagation interview scenarios.

We'll go deep into it in the next section.

---

# 11. Interview Question: Can a transaction span multiple service methods?

Yes.

For example:

```java
@Transactional
public void processOrder() {

    createOrder();

    processPayment();

    updateInventory();
}
```

All three can participate in the same transaction if they use compatible propagation and the same transactional resource.

The transaction is not tied to one Java method.

It's tied to the **transaction context** established by the transactional boundary.

---

# 12. Can a transaction span multiple repositories?

Absolutely.

This is actually one of the main reasons service-level transactions are useful.

```java
@Transactional
public void processOrder() {

    orderRepository.save(order);

    paymentRepository.save(payment);

    inventoryRepository.save(inventory);

    ledgerRepository.save(entry);
}
```

Conceptually:

```text
             Transaction T1
                   │
       ┌───────────┼────────────┐
       ↓           ↓            ↓
   Order DB    Payment DB    Inventory DB
```

However, there is an important qualification:

If these are actually **different databases/resources**, things become more complicated.

A normal local database transaction doesn't automatically provide atomicity across arbitrary distributed resources.

That becomes a distributed transaction / saga / outbox type of problem.

---

# 13. Transaction boundary and microservices

This is extremely important for your EPAM interview.

Suppose:

```text
Order Service
      ↓
Payment Service
      ↓
Inventory Service
```

Each service has its own database.

You cannot simply do:

```java
@Transactional
public void createOrder() {

    orderRepository.save(order);

    paymentClient.pay();

    inventoryClient.reserve();
}
```

and claim:

> "All three services are inside one Spring transaction."

They are not.

Your local Spring transaction covers the local transactional resources.

The HTTP call:

```text
paymentClient.pay()
```

doesn't magically extend the local database transaction into the Payment Service.

So:

```text
Order DB
   ↓
Transaction T1

HTTP
   ↓

Payment Service
   ↓
Transaction T2
```

Potentially:

```text
T1 succeeds
T2 succeeds
T3 fails
```

Now you have a distributed consistency problem.

Typical approaches include:

- Saga pattern
- Outbox pattern
- compensating transactions
- event-driven workflows

This is a **senior-level distinction** worth remembering.

---

# 14. Transaction boundary and external APIs

Consider:

```java
@Transactional
public void processOrder() {

    orderRepository.save(order);

    paymentClient.chargeCard();

    inventoryRepository.reserve();
}
```

Should you keep an external HTTP call inside a database transaction?

Be careful.

The database transaction might remain open while waiting for:

```text
paymentClient.chargeCard()
```

If the external service takes 5 seconds:

```text
BEGIN DB TRANSACTION
       ↓
save order
       ↓
HTTP call
       ↓
wait 5 seconds
       ↓
inventory update
       ↓
COMMIT
```

The DB transaction remains open during the remote call.

That can cause:

- longer lock duration
- connection pool pressure
- reduced throughput
- contention
- timeout problems

So don't blindly make a huge service method transactional.

---

# 15. What makes a good transaction boundary?

A good transaction boundary generally:

### 1. Represents one business operation

Example:

```text
Place Order
Transfer Money
Create Payment
Post Ledger Entry
```

### 2. Is as small as reasonably possible

Avoid:

```text
BEGIN
 ↓
DB
 ↓
HTTP
 ↓
external API
 ↓
slow calculation
 ↓
file operation
 ↓
DB
 ↓
COMMIT
```

Prefer keeping the transaction around the database operations that actually need atomicity.

### 3. Contains operations that need to succeed/fail together

For example:

```text
Debit account
Credit account
```

should generally be atomic.

### 4. Avoids unnecessary external work

Don't hold database transactions open while waiting for unrelated remote systems unless there's a deliberate reason and appropriate architecture.

---

# 16. Interview scenario

### Interviewer:

> I have a method that saves an order, calls a payment API, sends an email, and updates inventory. Should I put `@Transactional` on the entire method?

### Strong answer:

> Not blindly. I would first identify which operations need atomicity. A database transaction can protect the local database operations, but it doesn't automatically make the external payment API or email atomic with the database. Keeping a transaction open during remote calls can also increase lock duration and consume database connections. I would normally keep the local database transaction focused on the required database state changes and use patterns such as an outbox or saga for reliable interaction with external systems.

That is much stronger than:

> "Yes, put `@Transactional` on the method."

---

# 17. Interview Question: What happens when a transactional method calls another transactional method?

The answer depends on **propagation**.

For the default:

```java
Propagation.REQUIRED
```

the inner method joins the existing transaction.

```text
Outer
  ↓
T1 starts
  ↓
Inner
  ↓
joins T1
```

For:

```java
Propagation.REQUIRES_NEW
```

the existing transaction is suspended and a new one starts.

```text
Outer
  ↓
T1
  ↓
suspend T1
  ↓
T2
  ↓
T2 completes
  ↓
resume T1
```

For:

```java
Propagation.NESTED
```

the behavior is based on a nested transaction/savepoint model where supported.

We'll cover each propagation mode individually next.

---

# 18. Transaction boundary and rollback

Suppose:

```java
@Transactional
public void process() {

    saveA();

    saveB();

    throw new RuntimeException();
}
```

The boundary is:

```text
BEGIN
 ↓
saveA
 ↓
saveB
 ↓
RuntimeException
 ↓
ROLLBACK
```

Both operations are normally rolled back if they participated in the same transaction and the exception triggers rollback.

But if `saveB()` used a separate transaction:

```text
T1
 ├── saveA
 │
 └── call saveB

T2
 └── saveB
```

then T2 might already have committed even if T1 subsequently rolls back.

That's exactly why propagation matters.

---

# 19. The most important mental model

Think of transactions as **context flowing through method calls**.

Example:

```text
Controller
    ↓
OrderService
    ↓
PaymentService
    ↓
Repository
```

If the service establishes:

```text
Transaction T1
```

then downstream operations can participate in T1 according to their propagation rules.

```text
Transaction T1
      ↓
OrderService
      ↓
PaymentService
      ↓
Repository
```

The transaction isn't simply:

> "attached to the first method."

It is a transactional context that Spring manages across the call chain.

---

# 20. Interview-ready answer

### Q: What is a transaction boundary and where should it be defined?

> A transaction boundary defines the scope within which database operations should execute atomically. In Spring applications, it is generally best defined at the service layer because a service method usually represents a business operation. When the transactional service method is invoked through Spring's proxy, the transaction interceptor starts or joins a transaction according to the propagation configuration. Repository operations executed within that scope can participate in the same transaction. The boundary ends when the method completes and Spring commits or rolls back the transaction according to the outcome and rollback rules.

---

# 21. Senior-level traps to remember

### Trap 1

> "Every repository call starts a transaction."

❌ Not necessarily.

Repository operations can participate in an existing transaction.

---

### Trap 2

> "Every `@Transactional` method creates a new transaction."

❌ Not necessarily.

With `REQUIRED`, an existing transaction is normally joined.

---

### Trap 3

> "A transaction automatically crosses microservices."

❌ No.

Each service normally has its own transaction boundary.

---

### Trap 4

> "HTTP calls are part of my database transaction because they're inside the method."

❌ No.

The remote service has its own execution and transaction context.

---

### Trap 5

> "Putting `@Transactional` on the controller is always wrong."

Not strictly. It can technically work, but service-layer transaction boundaries are generally preferred because they better represent business operations and keep transaction management out of the web layer.

---

### Trap 6

> "A bigger transaction is safer."

❌ Not necessarily.

Larger transactions can mean:

```text
longer locks
more contention
more connection usage
lower throughput
larger rollback scope
```

---

# 22. What comes next

Now that we understand **where the transaction boundary exists**, the next question is:

> **What happens when one transactional method calls another transactional method?**

That's **Propagation**.

We will cover every mode individually:

```text
REQUIRED
REQUIRES_NEW
NESTED
SUPPORTS
NOT_SUPPORTED
MANDATORY
NEVER
```

And for each one we'll cover:

```text
What it means
↓
What Spring does
↓
Existing transaction?
↓
New transaction?
↓
Commit/rollback behavior
↓
Real production example
↓
Interview trap
```

The most important ones for the interview are **REQUIRED, REQUIRES_NEW, and NESTED**.