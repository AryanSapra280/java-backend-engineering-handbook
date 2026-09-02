# Java Concurrency — 06: Atomic Classes & CAS

> `volatile` gives visibility and ordering, but does **not** make compound operations such as `count++` atomic.
>
> `AtomicInteger` gives atomic operations on an integer.
>
> CAS (Compare-And-Set) is one of the fundamental mechanisms behind these atomic operations.

---

# 1. The Problem with `volatile`

```java
private volatile int count = 0;

void increment() {
    count++;
}
```

This is **not thread-safe**.

`count++` is conceptually:

```text
READ count
    ↓
ADD 1
    ↓
WRITE count
```

Two threads can read the same value and overwrite each other's updates.

Therefore:

```text
volatile
→ visibility + ordering

volatile
≠ atomic increment
```

---

# 2. AtomicInteger

Java provides:

```java
import java.util.concurrent.atomic.AtomicInteger;
```

Example:

```java
class Counter {

    private final AtomicInteger count =
            new AtomicInteger(0);

    public void increment() {
        count.incrementAndGet();
    }

    public int getCount() {
        return count.get();
    }
}
```

Now:

```java
count.incrementAndGet();
```

is an atomic increment operation.

---

# 3. Three Approaches

## Normal `int`

```java
int count = 0;
count++;
```

```text
No synchronization
No guaranteed cross-thread visibility
No atomicity
```

## `volatile int`

```java
volatile int count = 0;
count++;
```

```text
Visibility ✅
Ordering   ✅
Atomic ++ ❌
```

## `AtomicInteger`

```java
AtomicInteger count =
        new AtomicInteger(0);

count.incrementAndGet();
```

```text
Visibility / memory semantics ✅
Atomic operations             ✅
```

Mental model:

```text
volatile
→ "I need other threads to see changes."

AtomicInteger
→ "I need an atomic operation on this value."
```

---

# 4. Important AtomicInteger Methods

```java
get()
set()

incrementAndGet()
getAndIncrement()

decrementAndGet()
getAndDecrement()

addAndGet()
getAndAdd()

compareAndSet()
```

---

# 5. `get()`

```java
AtomicInteger count =
        new AtomicInteger(10);

int value = count.get();
```

Returns the current value.

---

# 6. `set()`

```java
count.set(20);
```

Now:

```text
count = 20
```

---

# 7. `incrementAndGet()`

```java
int result =
        count.incrementAndGet();
```

If:

```text
count = 10
```

then:

```text
count = 11
result = 11
```

Think:

```text
increment
→ return NEW value
```

---

# 8. `getAndIncrement()`

```java
int result =
        count.getAndIncrement();
```

If:

```text
count = 10
```

then:

```text
result = 10
count = 11
```

Think:

```text
return OLD value
→ then increment
```

---

# 9. General Pattern

```text
xxxAndGet()
→ perform operation
→ return NEW value

getAndXxx()
→ return OLD value
→ perform operation
```

Examples:

```java
incrementAndGet()
getAndIncrement()

addAndGet()
getAndAdd()

decrementAndGet()
getAndDecrement()
```

---

# 10. `addAndGet()`

```java
AtomicInteger balance =
        new AtomicInteger(1000);

int result =
        balance.addAndGet(500);
```

After:

```text
balance = 1500
result = 1500
```

---

# 11. `getAndAdd()`

```java
int result =
        balance.getAndAdd(500);
```

If:

```text
balance = 1000
```

then:

```text
result = 1000
balance = 1500
```

---

# 12. What Is CAS?

CAS means:

> **Compare-And-Set**

The basic idea:

```text
"I believe the current value is X.

If it is still X,
change it to Y.

Otherwise,
do not change it."
```

---

# 13. Simple CAS Example

Suppose:

```text
count = 10
```

We want:

```text
10 → 11
```

Conceptually:

```text
Compare current value with 10
        ↓
Is current value still 10?
        ↓
      YES
        ↓
Change 10 → 11
```

If another thread has already changed it:

```text
count = 15
```

then:

```text
Compare current 15 with expected 10
        ↓
Not equal
        ↓
CAS fails
```

---

# 14. `compareAndSet()`

```java
AtomicInteger count =
        new AtomicInteger(10);

boolean success =
        count.compareAndSet(10, 11);
```

If current value is `10`:

```text
success = true
count = 11
```

If current value is `20`:

```text
success = false
count = 20
```

Conceptually:

```java
if (currentValue == expectedValue) {
    currentValue = newValue;
    return true;
}

return false;
```

The actual compare-and-update operation is atomic.

---

# 15. Why CAS Is Powerful

CAS combines:

```text
COMPARE
+
UPDATE
```

into an atomic operation.

```text
        ┌──────────────────────┐
        │ Compare current with │
        │ expected value       │
        └──────────┬───────────┘
                   ↓
              Equal?
             /                 YES       NO
            ↓         ↓
         Update     Fail
```

---

# 16. CAS Retry Loop

Conceptually, increment can be implemented as:

```java
while (true) {

    int current = count.get();

    int next = current + 1;

    if (count.compareAndSet(current, next)) {
        break;
    }
}
```

If another thread changes the value before our CAS:

```text
CAS fails
   ↓
read latest value
   ↓
calculate again
   ↓
try again
```

---

# 17. CAS Under Contention

Suppose:

```text
count = 10
```

Thread A:

```text
current = 10
next = 11
```

Thread B:

```text
current = 10
next = 11
```

Thread A:

```text
CAS(10, 11)
→ SUCCESS
```

Now:

```text
count = 11
```

Thread B:

```text
CAS(10, 11)
→ FAIL
```

Thread B retries:

```text
get() → 11
next → 12
CAS(11, 12)
→ SUCCESS
```

Final:

```text
count = 12
```

---

# 18. CAS Is Optimistic

Locking says:

```text
"I will prevent others from entering."
```

CAS says:

```text
"I'll try the operation.
If someone changed the value first,
I'll detect that and retry."
```

Therefore CAS is often described as an optimistic approach.

---

# 19. Does CAS Mean No Waiting?

Not necessarily.

A thread may repeatedly retry:

```text
CAS fails
   ↓
read again
   ↓
calculate again
   ↓
CAS again
   ↓
fail
   ↓
retry
```

Under heavy contention, repeated CAS failures can become expensive.

So:

```text
CAS
≠
contention disappears
```

---

# 20. Lock-Based vs CAS-Based

## Lock-based

```text
Thread A
   ↓
LOCK
   ↓
critical section
   ↓
UNLOCK

Other threads
   ↓
may block
```

## CAS-based

```text
Thread A → attempt CAS
Thread B → attempt CAS
Thread C → attempt CAS

Winner
   ↓
success

Loser
   ↓
retry
```

A CAS operation does not acquire a Java monitor.

---

# 21. AtomicInteger Is Not a Replacement for `synchronized`

AtomicInteger is excellent for:

```text
counter
sequence number
simple numeric state
metrics
statistics
```

But consider:

```java
if (balance >= amount) {
    balance -= amount;
    recordTransaction();
    updateAudit();
}
```

A single AtomicInteger cannot automatically make this entire business operation consistent.

You may need:

```text
transaction
locking
careful coordination
database consistency
```

depending on the architecture.

---

# 22. AtomicInteger and Bank Withdrawal

This is NOT automatically safe:

```java
AtomicInteger balance =
        new AtomicInteger(10000);

if (balance.get() >= amount) {
    balance.addAndGet(-amount);
}
```

Possible interleaving:

```text
Thread A
get() → 10000

Thread B
get() → 10000

Thread A
addAndGet(-4000)

Thread B
addAndGet(-8000)
```

Both can pass the check.

The problem is:

```text
CHECK + UPDATE
```

must be coordinated as one atomic operation.

---

# 23. CAS-Based Conditional Withdrawal

CAS can be used:

```java
while (true) {

    int current = balance.get();

    if (current < amount) {
        return false;
    }

    int updated = current - amount;

    if (balance.compareAndSet(current, updated)) {
        return true;
    }
}
```

Mental model:

```text
READ
 ↓
CHECK
 ↓
CALCULATE
 ↓
CAS
 ↓
SUCCESS → done
FAILURE → retry
```

---

# 24. Why CAS Solves This Example

Initial:

```text
balance = 10000
```

Thread A:

```text
current = 10000
updated = 6000
```

Thread B:

```text
current = 10000
updated = 2000
```

Thread A:

```text
CAS(10000, 6000)
→ SUCCESS
```

Thread B:

```text
CAS(10000, 2000)
→ FAIL
```

Thread B retries and sees:

```text
balance = 6000
```

Then:

```text
6000 < 8000
```

Withdrawal fails.

The balance never becomes negative.

---

# 25. AtomicInteger Is Still JVM-Local

Suppose:

```text
Pod 1 → JVM 1
Pod 2 → JVM 2
```

Each has its own:

```java
AtomicInteger balance;
```

These are different objects:

```text
JVM 1
  ↓
AtomicInteger A

JVM 2
  ↓
AtomicInteger B
```

CAS only coordinates access to the atomic variable in its JVM.

Therefore:

```text
AtomicInteger
      ↓
JVM-local
```

Just like:

```text
synchronized
      ↓
JVM-local
```

It does not automatically coordinate multiple Spring Boot pods.

---

# 26. AtomicInteger vs Distributed State

For:

```text
local in-memory counter
```

AtomicInteger is appropriate.

For:

```text
authoritative account balance
```

in a distributed financial application, the consistency boundary generally needs a shared durable system such as the database.

Example:

```text
Pod 1 ─┐
Pod 2 ─┼──→ PostgreSQL
Pod 3 ─┘
```

Possible approaches:

```text
database transactions
row-level locking
optimistic locking
serialization by account
```

depending on the architecture.

---

# 27. AtomicLong

For `long`:

```java
AtomicLong counter =
        new AtomicLong(0);

counter.incrementAndGet();
```

Useful for:

```text
large counters
sequence values
statistics
IDs
```

---

# 28. AtomicReference

Atomic operations aren't limited to numbers.

```java
AtomicReference<String> state =
        new AtomicReference<>("NEW");
```

Then:

```java
state.compareAndSet(
    "NEW",
    "PROCESSING"
);
```

This can provide an atomic state transition.

---

# 29. AtomicReference and State Machines

Suppose:

```text
NEW
 ↓
PROCESSING
 ↓
COMPLETED
```

Represent state:

```java
AtomicReference<State> state =
        new AtomicReference<>(State.NEW);
```

Then:

```java
boolean changed =
    state.compareAndSet(
        State.NEW,
        State.PROCESSING
    );
```

If another thread already changed:

```text
NEW → CANCELLED
```

then:

```text
CAS(NEW, PROCESSING)
→ FAIL
```

This prevents an invalid concurrent transition.

---

# 30. `compareAndSet()` Return Value

```java
boolean success =
    value.compareAndSet(expected, newValue);
```

Returns:

```text
true
→ update happened

false
→ current value was not equal to expected
```

This return value enables CAS retry loops.

---

# 31. CAS and Performance

Potential advantages:

```text
No traditional monitor lock
No blocking for the atomic operation itself
Good for small/simple shared state
```

But never say:

> "CAS is always faster than synchronized."

Performance depends on:

```text
contention
number of threads
CPU architecture
critical-section size
amount of work
JVM implementation
```

Under heavy contention, repeated CAS failures can become expensive.

---

# 32. ABA Problem

An advanced CAS issue is the **ABA problem**.

Suppose Thread A reads:

```text
A
```

Thread B changes:

```text
A → B → A
```

Thread A later performs:

```text
CAS(A, C)
```

It sees:

```text
current = A
```

and thinks:

> "Nothing changed."

But something did happen:

```text
A → B → A
```

The value returned to A, but the state changed in between.

This is the:

```text
ABA problem
```

For advanced cases, Java provides:

```java
AtomicStampedReference
```

which can associate a version/stamp with the reference.

For interviews, understand the concept; you usually won't need to implement ABA handling.

---

# 33. Interview Question — What Is CAS?

### Strong Answer

> CAS, or Compare-And-Set, is an atomic operation that updates a value only if its current value matches an expected value. If another thread has already changed the value, the CAS fails, and the algorithm can retry using the latest value.

---

# 34. Interview Question — How Does AtomicInteger Increment Work?

### Strong Answer

> Conceptually, `AtomicInteger.incrementAndGet()` can be understood using a CAS loop. It reads the current value, calculates the next value, and attempts a compare-and-set. If another thread changed the value first, the CAS fails and the operation retries. The actual implementation is provided by the JDK and may use lower-level atomic primitives.

---

# 35. Interview Question — AtomicInteger vs synchronized

### Strong Answer

> `synchronized` provides mutual exclusion around a critical section and also establishes visibility guarantees. `AtomicInteger` provides atomic operations on a single integer using non-blocking atomic mechanisms such as CAS. AtomicInteger is useful for simple shared numeric state, whereas synchronized is more appropriate when multiple operations or multiple pieces of state need to be protected together.

---

# 36. Interview Question — Can AtomicInteger Replace synchronized?

### Strong Answer

> Not generally. AtomicInteger can replace synchronization for certain simple atomic state updates, such as incrementing a counter, but it does not automatically protect larger critical sections involving multiple variables or multiple business operations.

---

# 37. Interview Question — Why Doesn't AtomicInteger Solve Multi-JVM Concurrency?

### Strong Answer

> AtomicInteger operates on an in-memory variable inside a JVM. If the application has multiple pods, each JVM has its own AtomicInteger instance. Therefore it cannot coordinate updates across JVMs. For distributed shared state such as an account balance, I would use a shared consistency mechanism such as database transactions or appropriate distributed coordination.

---

# 38. Coding Exercise — Atomic Counter

Implement:

```java
class Counter {

    private final AtomicInteger count =
            new AtomicInteger(0);

    public void increment() {
        // implement
    }

    public int getCount() {
        // implement
    }
}
```

Test with:

```text
2 threads
100,000 increments each
```

Expected:

```text
200000
```

---

# 39. Coding Exercise — `compareAndSet`

Start:

```java
AtomicInteger value =
        new AtomicInteger(10);
```

Try:

```java
System.out.println(
    value.compareAndSet(10, 20)
);
```

Expected:

```text
true
```

Then:

```java
System.out.println(value.get());
```

Expected:

```text
20
```

Now:

```java
System.out.println(
    value.compareAndSet(10, 30)
);
```

Expected:

```text
false
```

because:

```text
expected = 10
current = 20
```

---

# 40. Coding Exercise — Implement Increment Using CAS

Try:

```java
void increment() {

    while (true) {

        int current = count.get();

        int next = current + 1;

        if (count.compareAndSet(current, next)) {
            return;
        }
    }
}
```

Understand every line.

This connects:

```text
AtomicInteger
+
CAS
+
race conditions
+
retry
```

---

# 41. Coding Exercise — Concurrent Withdrawal ⭐⭐⭐⭐⭐

Implement:

```java
class Account {

    private final AtomicInteger balance;

    Account(int balance) {
        this.balance =
            new AtomicInteger(balance);
    }

    boolean withdraw(int amount) {

        // Implement using CAS.
    }

    int getBalance() {
        return balance.get();
    }
}
```

Test:

```text
Initial balance = 10000

Thread A → withdraw 4000
Thread B → withdraw 8000
```

Requirements:

```text
Exactly one withdrawal succeeds.
Balance must never become negative.
```

Think:

```text
get
 ↓
check
 ↓
calculate
 ↓
compareAndSet
 ↓
success → done
failure → retry
```

---

# 42. Final Mental Model

```text
Race Condition
      ↓
Multiple threads interfere
      ↓
Need coordination
      │
      ├── synchronized
      │      ↓
      │   monitor
      │      ↓
      │   mutual exclusion
      │
      ├── volatile
      │      ↓
      │   visibility + ordering
      │
      └── Atomic/CAS
             ↓
        atomic update
             ↓
            CAS
             ↓
    compare expected value
             ↓
       update if unchanged
             ↓
       retry if necessary
```

---

# 43. Where We Are

```text
Thread Basics                    ✅
Thread Lifecycle                 ✅
Race Conditions                  ✅
synchronized                     ✅
volatile                         ✅
AtomicInteger                    ✅
CAS                              ✅
AtomicReference                  ✅
```

Next:

```text
ReentrantLock
    ↓
tryLock()
    ↓
lockInterruptibly()
    ↓
unlock()
    ↓
fairness
    ↓
Condition
```

Then:

```text
ExecutorService
    ↓
Thread Pools
    ↓
Callable
    ↓
Future
    ↓
CompletableFuture
    ↓
Async Programming
```

## Key distinction

```text
volatile
→ visibility

AtomicInteger
→ atomic operation

CAS
→ compare + conditional atomic update

synchronized
→ mutual exclusion + visibility
```
