Absolutely — next is **Read-Only Transactions**. This looks simple, but there are several interview traps around what `readOnly = true` actually guarantees.

# 6. Read-Only Transactions ⭐⭐⭐⭐

## 1. What is `readOnly = true`?

You can configure:

```java
@Transactional(readOnly = true)
public Payment getPayment(Long id) {
    return paymentRepository.findById(id)
            .orElseThrow();
}
```

It tells Spring:

> "This transaction is intended for read operations rather than modifications."

A common use case is:

```text id="d3o5x8"
GET API
   ↓
Service
   ↓
@Transactional(readOnly = true)
   ↓
Repository
   ↓
Database
```

For example:

```java id="x8k2cd"
@Service
public class PaymentQueryService {

    @Transactional(readOnly = true)
    public Payment getPayment(String id) {
        return paymentRepository.findById(id)
                .orElseThrow();
    }
}
```

---

# 2. Does `readOnly = true` mean the method cannot perform a write?

**No.**

This is the first major interview trap.

Do NOT say:

> "`readOnly = true` prevents INSERT/UPDATE/DELETE."

That's too strong.

`readOnly` is primarily a **hint/configuration to the transaction infrastructure and underlying persistence technology**.

It does not universally enforce:

```text id="2m8q9c"
NO INSERT
NO UPDATE
NO DELETE
```

across every database and persistence technology.

---

# 3. Why is it called "read-only" then?

Because you're communicating your intention:

```java id="u2c8w4"
@Transactional(readOnly = true)
```

means:

> "This transaction is intended to be used for reading."

Spring can then pass that information to the underlying transaction/persistence infrastructure.

Different technologies can use that information differently.

---

# 4. What happens internally?

Conceptually:

```text id="q4y0x7"
@Transactional(readOnly = true)
          ↓
TransactionInterceptor
          ↓
Transaction attributes
          ↓
readOnly = true
          ↓
TransactionManager
          ↓
Underlying database / ORM
```

So `readOnly` is part of the transaction definition that Spring passes through the transaction infrastructure.

---

# 5. Why can read-only improve performance?

The exact optimization depends on the technology.

With JPA/Hibernate, read-only transaction semantics can influence how the persistence context handles entities and dirty checking.

Conceptually:

```text id="v0h2eq"
Normal transaction
      ↓
Entity loaded
      ↓
Persistence context tracks changes
      ↓
Dirty checking
      ↓
Potential UPDATE
```

For a read-only workload, the ORM can potentially reduce unnecessary work related to modification tracking.

This can be beneficial for read-heavy operations.

But don't claim:

> "readOnly=true makes the database faster."

The actual performance impact depends on:

- database
- JDBC driver
- transaction manager
- JPA provider
- Hibernate configuration
- query workload

---

# 6. Hibernate example

Suppose:

```java id="j2u4ph"
@Transactional
public Payment getPayment(Long id) {

    Payment payment =
        paymentRepository.findById(id)
                         .orElseThrow();

    return payment;
}
```

Hibernate may maintain the entity in the persistence context and perform dirty checking.

Now:

```java id="3f9x2q"
@Transactional(readOnly = true)
public Payment getPayment(Long id) {

    Payment payment =
        paymentRepository.findById(id)
                         .orElseThrow();

    return payment;
}
```

The read-only hint can allow Hibernate/Spring integration to optimize aspects of persistence-context behavior.

The exact optimization depends on the versions/configuration.

---

# 7. Interview Question: Does read-only mean no transaction is created?

No.

This is another common trap.

```java id="3evy5a"
@Transactional(readOnly = true)
```

is still a transactional method.

Conceptually:

```text id="l9eqv3"
BEGIN TRANSACTION
      ↓
READ operations
      ↓
COMMIT
```

The difference is that the transaction is marked as read-only.

It does **not** mean:

```text id="kmy7ag"
No transaction
```

---

# 8. `readOnly = true` vs no transaction

These are different:

### Read-only transaction

```java id="a0d6gd"
@Transactional(readOnly = true)
public Payment getPayment() {
    ...
}
```

There is still transaction infrastructure involved.

### No transaction

```java id="w1f8rv"
public Payment getPayment() {
    ...
}
```

There may be no surrounding application transaction.

Which one is appropriate depends on the persistence technology and consistency requirements.

---

# 9. Why use a read-only transaction for a SELECT?

Imagine:

```java id="6x9q0n"
@Transactional(readOnly = true)
public List<Payment> findPayments() {

    return paymentRepository
            .findByAccountId(accountId);
}
```

Benefits can include:

- expressing intent
- allowing persistence infrastructure to optimize
- avoiding unnecessary write-oriented behavior
- making the service contract clearer
- potentially reducing ORM overhead

It is particularly useful in read-heavy services.

---

# 10. Interview Question: Does `readOnly = true` guarantee that UPDATE will fail?

**No, not universally.**

For example:

```java id="nyb2r5"
@Transactional(readOnly = true)
public void process() {

    payment.setStatus("SUCCESS");

    paymentRepository.save(payment);
}
```

You should **not** assume Spring universally throws an exception simply because the transaction is read-only.

Depending on the transaction manager, database, driver, ORM, and configuration:

- the write may be rejected,
- ignored/handled differently,
- or potentially execute.

Therefore:

> `readOnly = true` should be treated primarily as a transaction optimization/semantic hint, not as a universal security mechanism for preventing writes.

If you need to enforce that a method cannot perform writes, design the application accordingly rather than relying only on this flag.

---

# 11. Read-only and Hibernate dirty checking

This is particularly relevant for Spring + JPA interviews.

Normally Hibernate uses dirty checking.

Suppose:

```java id="8u4lq3"
@Transactional
public void updatePayment(Long id) {

    Payment payment =
        paymentRepository.findById(id)
                         .orElseThrow();

    payment.setStatus("SUCCESS");
}
```

You don't necessarily need:

```java id="4rcb9v"
paymentRepository.save(payment);
```

because Hibernate can detect that the managed entity changed and generate an update during flush.

Conceptually:

```text id="xq2l1e"
Load entity
    ↓
Managed entity
    ↓
Change field
    ↓
Dirty checking
    ↓
UPDATE
```

With a read-only transaction, the persistence infrastructure can optimize the entity's read-only handling.

---

# 12. Interview Question: If I modify an entity inside `readOnly = true`, will Hibernate always update it?

Don't answer with an absolute yes/no without qualification.

A good senior answer is:

> "`readOnly = true` changes transaction/persistence semantics and can allow Hibernate/Spring to optimize dirty checking and flush behavior, but it isn't a universal write-prevention mechanism. The exact behavior depends on the JPA provider, transaction manager, and configuration. I use read-only transactions for operations that are semantically read-only rather than relying on them as an access-control mechanism."

That's a much safer and more accurate answer.

---

# 13. `readOnly` and database-level read-only

This distinction is important.

There are potentially multiple layers:

```text id="3g0vhl"
Spring transaction
      ↓
JPA/Hibernate
      ↓
JDBC
      ↓
Database
```

Spring's:

```java id="6v7w3e"
@Transactional(readOnly = true)
```

does not necessarily mean the database itself has been placed into a strict read-only mode.

Some transaction managers/drivers/databases can propagate read-only semantics further.

Others may treat it mainly as a hint.

Therefore:

> Spring's `readOnly` flag and a database's strict read-only transaction mode are related but not necessarily identical.

---

# 14. Production example

Suppose your payment service has:

```text id="j6uw6f"
POST /payments
GET /payments/{id}
GET /payments/history
```

Write operation:

```java id="zj31t7"
@Transactional
public Payment createPayment(...) {
    ...
}
```

Read operation:

```java id="x0u4ma"
@Transactional(readOnly = true)
public Payment getPayment(...) {
    ...
}
```

And:

```java id="p3cv8x"
@Transactional(readOnly = true)
public List<Payment> getPaymentHistory(...) {
    ...
}
```

This communicates intent clearly:

```text id="4v9q7w"
Command
→ normal transaction

Query
→ read-only transaction
```

---

# 15. CQRS connection

This concept becomes particularly useful in architectures separating:

```text id="9l0d4r"
Commands → modify state
Queries  → read state
```

For example:

```text id="g8a0s9"
Command Service
   ↓
@Transactional
   ↓
Database
```

versus:

```text id="9a2i1j"
Query Service
   ↓
@Transactional(readOnly = true)
   ↓
Database / read replica
```

This isn't required to use `readOnly`, but the semantic distinction fits naturally with read-heavy architectures.

---

# 16. Read-only and read replicas

This is an important production distinction.

Suppose you have:

```text id="f3e1w7"
Primary DB
   ↓
writes

Read Replica
   ↓
reads
```

You might think:

```java id="f8j3la"
@Transactional(readOnly = true)
```

automatically sends the query to the replica.

**It does not.**

`readOnly = true` does not automatically implement read/write database routing.

You need separate routing/configuration, such as:

```text id="k1p5k8"
RoutingDataSource
      ↓
readOnly → replica
write     → primary
```

or an architecture-specific data-source/routing solution.

This is an excellent senior interview trap.

---

# 17. Interview Question: Does `readOnly = true` automatically use a read replica?

**No.**

Strong answer:

> "`readOnly = true` communicates read-only transaction intent to Spring's transaction infrastructure. It doesn't by itself perform database routing. If we want read operations to go to a replica, we need explicit routing or a data-access architecture that supports primary/replica selection."

---

# 18. Read-only transactions and consistency

Suppose:

```text id="wz7k0m"
Primary DB
   ↓
write payment
```

and immediately:

```text id="7n9bqj"
read payment
```

If the read goes to a replica:

```text id="l3t4eq"
Primary
   ↓
replication lag
   ↓
Replica
```

the read might not immediately see the write.

So:

```text id="i7y7q5"
readOnly = true
```

doesn't solve replication consistency.

This is a separate architectural issue.

---

# 19. Read-only vs `Propagation.SUPPORTS`

Don't confuse:

```java id="4e5n1f"
@Transactional(readOnly = true)
```

with:

```java id="z7sp0n"
@Transactional(propagation = Propagation.SUPPORTS)
```

They solve different problems.

### `readOnly`

Answers:

> "Is this transaction intended to perform reads only?"

### `SUPPORTS`

Answers:

> "What should happen if a transaction already exists?"

You can even combine them:

```java id="4g3yq8"
@Transactional(
    propagation = Propagation.SUPPORTS,
    readOnly = true
)
public Payment getPayment(...) {
}
```

Meaning:

```text id="c9xv3g"
Existing TX?
    ↓
Join it

No TX?
    ↓
Execute without transaction

And:
    ↓
read-only intent
```

---

# 20. Interview Question: Should every GET endpoint use `readOnly = true`?

Not automatically.

A better answer:

> "For service methods that are genuinely read-only, `@Transactional(readOnly = true)` can make the intent explicit and may provide persistence-layer optimizations. But I wouldn't mechanically add it to every GET endpoint. The transaction requirement depends on the persistence behavior, consistency needs, and application architecture."

Also remember:

```text id="z1m6uj"
HTTP GET
```

doesn't necessarily mean:

```text id="c7e7y8"
database read only
```

The business operation determines that.

---

# 21. Read-only transaction and multiple reads

Suppose:

```java id="ps8bcm"
@Transactional(readOnly = true)
public AccountSummary getSummary(Long accountId) {

    Account account =
        accountRepository.findById(accountId);

    List<Payment> payments =
        paymentRepository.findRecentPayments(accountId);

    LedgerBalance balance =
        ledgerRepository.getBalance(accountId);

    return buildSummary(account, payments, balance);
}
```

The transaction can provide a consistent transactional context across these reads according to the database's isolation semantics.

This can be useful when you want several reads to belong to the same transaction rather than being completely independent operations.

---

# 22. Read-only doesn't mean immutable Java objects

Suppose:

```java id="9ph4m1"
@Transactional(readOnly = true)
public Payment getPayment() {

    Payment payment = repository.findById(id)
                                .orElseThrow();

    payment.setStatus("SUCCESS");

    return payment;
}
```

The Java object is still mutable.

`readOnly` doesn't magically turn:

```java id="a7q5w3"
Payment
```

into:

```text id="b6x5b9"
immutable Payment
```

It's a transaction/persistence concept, not a Java immutability feature.

---

# 23. Interview Question: Is read-only mainly for performance?

Not only.

There are two important aspects:

### 1. Semantic intent

You're telling the transaction infrastructure:

> "This operation is read-only."

### 2. Potential optimization

The underlying persistence stack may use that information to reduce unnecessary work.

So the answer is:

> It is both a semantic declaration and potentially an optimization, depending on the underlying technology.

---

# 24. Common mistakes

### Mistake 1

> "`readOnly=true` means writes are impossible."

❌ Too strong.

---

### Mistake 2

> "`readOnly=true` means there is no transaction."

❌ Wrong.

---

### Mistake 3

> "`readOnly=true` automatically routes to a replica."

❌ Wrong.

---

### Mistake 4

> "Every GET must have `readOnly=true`."

❌ Not necessarily.

---

### Mistake 5

> "readOnly guarantees better performance."

❌ Not guaranteed.

The actual benefit depends on the persistence stack and workload.

---

# 25. Interview-ready answer

### Q: What does `@Transactional(readOnly = true)` do?

> `readOnly = true` marks the transaction as read-only from the application's transaction-management perspective. Spring can propagate this hint to the underlying transaction and persistence infrastructure, and JPA/Hibernate can use it to optimize persistence behavior such as dirty checking and flushing depending on the configuration. It does not universally prevent writes, nor does it automatically route queries to a read replica. I typically use it for service methods whose business semantics are genuinely read-only.

---

# 26. Senior-level example

Suppose you have:

```java
@Service
public class PaymentQueryService {

    @Transactional(readOnly = true)
    public PaymentDetails getPayment(String paymentId) {

        Payment payment =
            paymentRepository.findById(paymentId)
                             .orElseThrow();

        List<PaymentAttempt> attempts =
            attemptRepository.findByPaymentId(paymentId);

        return buildDetails(payment, attempts);
    }
}
```

This is a good candidate because:

```text id="a9y6jg"
No state modification
        ↓
Multiple database reads
        ↓
Read-only transaction
        ↓
Clear transaction semantics
```

---

# 27. The mental model

When you see:

```java
@Transactional(readOnly = true)
```

think:

```text id="3x1b6n"
Transactional?
→ YES

Intended workload?
→ READ

Write prevention?
→ NOT universally guaranteed

Possible ORM optimization?
→ YES

Automatic replica routing?
→ NO

No transaction?
→ NO
```

And remember:

```text id="6p7p4y"
readOnly ≠ no transaction
readOnly ≠ immutable object
readOnly ≠ replica routing
readOnly ≠ universal write protection
```

---

# Next: Transaction + Exception Behavior

We've now covered rollback rules. The next item in the exact sequence is:

## 7. Transaction + Exception Behavior

We'll connect everything we've learned so far:

```text
@Transactional
      ↓
exception thrown
      ↓
checked / unchecked
      ↓
caught / uncaught
      ↓
rollback rules
      ↓
rollback-only
      ↓
commit / rollback
      ↓
UnexpectedRollbackException
```

Then we'll move into the especially important **Transaction + `CompletableFuture` / async** section.