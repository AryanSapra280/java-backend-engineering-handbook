# 8.31 Concurrent Balance Update — The Interview Answer

Yes — **this is an excellent interview question for your payment/ledger project.** The core problem is a **race condition / lost update**.

Suppose:

```text
Current balance = ₹1000
```

Two requests arrive concurrently:

```text
Request A → debit ₹1000
Request B → debit ₹1000
```

Both must **not** succeed.

The correct result is:

```text
Request A → SUCCESS
Request B → INSUFFICIENT_FUNDS
Final balance → ₹0
```

---

## 1. What goes wrong with naive code?

Suppose you write:

```java
Account account = repository.findById(accountId);

if (account.getBalance().compareTo(amount) >= 0) {
    account.setBalance(account.getBalance().subtract(amount));
    repository.save(account);
}
```

Two threads can do:

```text
Time       Thread A              Thread B
------------------------------------------------
T1         Read balance=1000
T2                               Read balance=1000
T3         1000 >= 1000 ✓
T4                               1000 >= 1000 ✓
T5         Save balance=0
T6                               Save balance=0
```

Now both requests may report success.

The final balance happens to be `₹0`, but **₹2000 was authorized/debited**.

That's the dangerous part.

---

# 2. What should you say in the interview?

I'd give this answer:

> **"I would handle this using concurrency control at the database level rather than relying only on application-level checks. For a balance debit, I can use optimistic locking with a version field, or perform an atomic conditional update such as `balance >= debitAmount`. The database then guarantees that only one concurrent request can successfully update the balance. If the second request fails the update because the balance is no longer sufficient or the version has changed, I return an insufficient-funds or retry response."**

That's a strong answer.

There are actually **two good approaches** you should know.

---

# 3. Approach 1 — Atomic conditional update ⭐

For a payment/balance system, this is often a very clean approach.

Instead of:

```text
READ balance
     ↓
CHECK balance
     ↓
WRITE balance
```

do the operation atomically:

```sql
UPDATE account
SET balance = balance - 1000
WHERE account_id = ?
AND balance >= 1000;
```

Then check how many rows were updated.

### First request

```text
balance = 1000

UPDATE ... WHERE balance >= 1000
```

succeeds:

```text
rows updated = 1
balance = 0
```

### Second request

Now:

```text
balance = 0
```

So:

```sql
WHERE balance >= 1000
```

is false.

```text
rows updated = 0
```

Therefore:

```text
Request A → SUCCESS
Request B → INSUFFICIENT_FUNDS
```

This is beautiful because the **check and update happen atomically in the database**.

---

# 4. Approach 2 — Optimistic locking

You can also use a version field:

```text
Account

balance = 1000
version = 5
```

Both requests read:

```text
balance = 1000
version = 5
```

Request A updates:

```text
balance = 0
version = 6
```

Request B tries:

```text
UPDATE account
SET balance = 0,
    version = 6
WHERE account_id = ?
AND version = 5
```

But version is now:

```text
6
```

not:

```text
5
```

So:

```text
0 rows updated
```

Request B knows that someone modified the account after it read it.

That's **optimistic locking**.

---

# 5. In JPA, this is often done with `@Version`

For example:

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

Hibernate then uses the version to detect concurrent modifications.

Conceptually:

```text
Thread A                    Thread B

balance=1000                balance=1000
version=5                   version=5
     ↓                           ↓
update                       update
     ↓                           ↓
version 6                    version still 5?
     ↓                           ↓
SUCCESS                     OptimisticLockException
```

You catch the optimistic-lock failure and decide whether to retry or reject the transaction.

---

# 6. Which one should YOU mention first?

For this particular question:

> "Two concurrent payment requests want to debit ₹1000 from a ₹1000 balance."

I'd lead with:

### **Atomic conditional update**

because the business rule itself is:

```text
Debit only if balance >= amount
```

So we can express the business rule directly in the database operation:

```sql
UPDATE account
SET balance = balance - :amount
WHERE account_id = :accountId
AND balance >= :amount;
```

Then:

```text
rowsAffected == 1
    → debit successful

rowsAffected == 0
    → insufficient funds / concurrent update
```

Then tell the interviewer:

> "Alternatively, if I'm using entity-based updates, I can use optimistic locking with `@Version`."

That shows you know **both the practical solution and the concurrency mechanism**.

---

# 7. What about `synchronized`?

This is where interviewers may dig deeper.

You might think:

```java
synchronized
public void debit(...) {
    ...
}
```

But **don't use that as your primary answer**.

Why?

Because your application could have multiple instances:

```text
             Load Balancer
              /         \
             ↓           ↓
        Pod A           Pod B
          |               |
       Thread 1        Thread 2
```

A Java `synchronized` block in Pod A doesn't lock Pod B.

So:

```text
synchronized
```

only gives you JVM-level coordination.

For a distributed payment system, you need concurrency control that works across application instances.

---

# 8. What about a distributed lock?

You could also use something like a distributed lock:

```text
Redis distributed lock
        ↓
Account 123 locked
        ↓
Debit
        ↓
unlock
```

But I wouldn't lead with that.

It introduces:

* lock expiry
* ownership
* failure handling
* deadlocks
* availability concerns
* additional infrastructure

If the database can perform the operation atomically, that's often simpler.

---

# 9. Now connect this to your payment project

This is where your project becomes powerful.

You already have:

```text
Payment
   ↓
Idempotency
```

Now imagine:

```text
Payment Service
      |
      ↓
Wallet/Account Service
      |
      ↓
Balance = ₹1000
```

Two payment requests:

```text
Payment A → ₹1000
Payment B → ₹1000
```

Your wallet operation could be:

```text
Debit(accountId, ₹1000)
```

and internally:

```sql
UPDATE wallet
SET balance = balance - :amount
WHERE wallet_id = :walletId
AND balance >= :amount;
```

Then:

```text
                 Balance ₹1000
                      |
              ┌───────┴───────┐
              ↓               ↓
          Request A        Request B
          ₹1000             ₹1000
              ↓               ↓
              └───────┬───────┘
                      ↓
              Atomic DB update
                      |
              ┌───────┴───────┐
              ↓               ↓
           SUCCESS       0 rows updated
              ↓               ↓
          balance ₹0     INSUFFICIENT
```

That's a **very strong payment-system answer**.

---

# 10. One more thing: idempotency and concurrency solve different problems

This is important because you've just learned idempotency.

### Idempotency

Prevents:

> **The same request being processed twice.**

```text
Same idempotency key
        ↓
same operation
        ↓
don't create duplicate payment
```

### Concurrency control

Prevents:

> **Two different valid requests from incorrectly modifying the same balance simultaneously.**

```text
Request A → debit ₹1000
Request B → debit ₹1000
        ↓
balance ₹1000
        ↓
only one succeeds
```

So in a payment system you typically need **both**.

---

## Interview-ready answer

If they ask you this tomorrow, I'd answer in this order:

> "This is a concurrent balance update problem. I wouldn't solve it with just a read-check-write sequence because two requests could both read the same balance. For a debit, I'd prefer an atomic conditional database update such as `UPDATE wallet SET balance = balance - :amount WHERE wallet_id = :id AND balance >= :amount`. I'd check the affected row count: one means the debit succeeded, zero means the balance condition failed or another concurrent request consumed the balance. Alternatively, if I'm using entity updates, I can use optimistic locking with a version field such as JPA's `@Version`. I wouldn't rely on `synchronized` because that only protects threads within one JVM and doesn't solve the problem across multiple service instances."

**That's the answer I'd want you to be able to give confidently.**
