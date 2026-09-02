# Java Concurrency — 05: `volatile`

> `volatile` is primarily about **visibility and ordering** between threads. It does **not** provide mutual exclusion and does **not** make general compound operations atomic.

---

# 1. The Problem: Visibility

Consider:

```java
class Worker {

    private boolean running = true;

    void stop() {
        running = false;
    }

    void work() {
        while (running) {
            // do work
        }

        System.out.println("Stopped");
    }
}
```

We have:

```text
Thread 1 → work()
Thread 2 → stop()
```

Thread 2 executes:

```java
running = false;
```

We expect Thread 1 to observe the change.

But without an appropriate Java Memory Model synchronization mechanism, there is no guarantee that the write is observed by the other thread in the way required for correctness.

This is a **visibility problem**.

---

# 2. Add `volatile`

```java
class Worker {

    private volatile boolean running = true;

    void stop() {
        running = false;
    }

    void work() {

        while (running) {
            // do work
        }

        System.out.println("Stopped");
    }
}
```

Now accesses to `running` have the visibility and ordering guarantees defined for volatile variables by the Java Memory Model.

Mental model:

```text
Thread 2
   ↓
volatile write
running = false
   ↓
Thread 1
   ↓
volatile read
running
   ↓
observes the write
```

---

# 3. The Most Important Rule

A useful mental model is:

> **Use `volatile` when threads need to reliably observe changes to shared state, but the operation does not require mutual exclusion or a compound atomic update.**

---

# 4. `volatile` Does NOT Mean "Thread-Safe"

Classic interview trap:

```java
private volatile int count = 0;

void increment() {
    count++;
}
```

Is this thread-safe?

**No.**

Because:

```text
count++
```

is still conceptually:

```text
READ count
    ↓
ADD 1
    ↓
WRITE count
```

Two threads can interleave those steps.

Therefore:

```text
volatile
→ visibility + ordering

volatile
≠ atomic increment
```

---

# 5. Example of Lost Update

Initial:

```text
count = 0
```

Thread A:

```text
READ 0
```

Thread B:

```text
READ 0
```

Thread A:

```text
WRITE 1
```

Thread B:

```text
WRITE 1
```

Final:

```text
1
```

Expected:

```text
2
```

Even though `count` is volatile.

So:

```text
Visibility
    ≠
Atomicity
```

---

# 6. `volatile` vs `synchronized`

| Feature | `volatile` | `synchronized` |
|---|---|---|
| Visibility | ✅ | ✅ |
| Ordering guarantees | ✅ | ✅ |
| Mutual exclusion | ❌ | ✅ |
| Monitor/lock | ❌ | ✅ |
| Blocks competing threads | ❌ | ✅ |
| `count++` automatically safe | ❌ | ✅ inside protected section |
| Good for simple flags | ✅ | Can be |
| Protects compound critical section | ❌ | ✅ |

Mental model:

```text
volatile
→ "Other threads need to see this state change."

synchronized
→ "Only one thread at a time should execute this critical section,
   and changes need to be visible."
```

---

# 7. `volatile` Does Not Lock

With:

```java
synchronized (lock) {
    // ...
}
```

if Thread A owns the monitor:

```text
Thread B
   ↓
tries lock
   ↓
BLOCKED
```

With:

```java
volatile boolean running;
```

there is no monitor lock.

Multiple threads can access the variable concurrently according to the Java Memory Model's volatile semantics.

---

# 8. Classic Use Case — Shutdown Flag

A common pattern:

```java
class Worker implements Runnable {

    private volatile boolean running = true;

    public void shutdown() {
        running = false;
    }

    @Override
    public void run() {

        while (running) {
            processNextTask();
        }
    }

    private void processNextTask() {
        System.out.println("Processing...");
    }
}
```

One thread runs the worker:

```java
worker.run();
```

Another thread requests shutdown:

```java
worker.shutdown();
```

The important requirement is:

```text
Thread A
→ repeatedly reads state

Thread B
→ changes state

Thread A
→ must reliably observe the change
```

There is no need to make the flag update a compound read-modify-write operation.

---

# 9. Why `volatile` Is Good for a Flag

Consider:

```java
private volatile boolean running = true;
```

The worker does:

```java
while (running) {
    doWork();
}
```

Another thread does:

```java
running = false;
```

The operation is essentially a simple state signal:

```text
RUNNING
   ↓
STOP REQUESTED
```

This is a common use case for `volatile`.

---

# 10. Why `volatile` Does Not Fix the Bank Example

Suppose:

```java
private volatile int balance;
```

and:

```java
void withdraw(int amount) {

    if (balance >= amount) {
        balance -= amount;
    }
}
```

Still unsafe.

Why?

Because the business operation is:

```text
CHECK balance
      +
UPDATE balance
```

Two threads can still do:

```text
Thread A → READ balance
Thread B → READ balance

Thread A → UPDATE
Thread B → UPDATE
```

`volatile` does not combine those operations into one atomic critical section.

Possible mechanisms include:

```text
synchronized
Lock
Atomic/CAS-based design
Database transaction/locking
```

depending on where the authoritative state lives.

---

# 11. `volatile` and `volatile int`

```java
volatile int value;
```

Volatile reads and writes of the variable have the required visibility/ordering semantics.

For example:

```java
value = 10;
```

and:

```java
int x = value;
```

use volatile access semantics.

But:

```java
value++;
```

is still a compound read-modify-write operation and is not automatically atomic.

---

# 12. `volatile long`

Modern Java guarantees atomic reads/writes for `long` and `double` variables.

`volatile` additionally provides the volatile visibility/ordering semantics.

Still:

```java
volatile long count;

count++;
```

is **not** an atomic increment.

The issue is:

```text
READ
+
MODIFY
+
WRITE
```

not simply the size of the variable.

---

# 13. `volatile` and Ordering

`volatile` also provides ordering guarantees under the Java Memory Model.

Example:

```java
class Example {

    private int data;
    private volatile boolean ready;

    void writer() {

        data = 42;
        ready = true;
    }

    void reader() {

        if (ready) {
            System.out.println(data);
        }
    }
}
```

The important relationship is:

```text
writer:
data = 42
     ↓
ready = true  ← volatile write
```

and:

```text
reader:
volatile read of ready
     ↓
ready == true
     ↓
read data
```

If the volatile read observes the volatile write, the Java Memory Model establishes the relevant happens-before relationship.

Therefore the earlier write to `data` is visible to the reader through that synchronization relationship.

This is a classic **publication/signaling** pattern.

---

# 14. Don't Reduce `volatile` to "RAM"

A common beginner explanation is:

> "`volatile` makes the value go directly to RAM."

That is an oversimplification.

A better mental model is:

```text
volatile
→ establishes specific visibility and ordering guarantees
  under the Java Memory Model
```

The JVM and hardware can still use caches and optimizations. `volatile` defines synchronization semantics that the implementation must respect.

---

# 15. `volatile` Reference ≠ Thread-Safe Object

Consider:

```java
private volatile List<String> names;
```

This makes the **reference** volatile.

It does not make the `List` itself thread-safe.

For example:

```java
names.add("Aryan");
```

is still a mutation of the list object.

Mental model:

```text
volatile reference
        ≠
thread-safe object
```

For concurrent collections, Java provides classes such as:

```text
ConcurrentHashMap
CopyOnWriteArrayList
BlockingQueue
```

which we'll study later.

---

# 16. Example of a Volatile Reference

```java
private volatile Config config;
```

Thread A:

```java
config = new Config(...);
```

Thread B:

```java
Config current = config;
```

The volatile reference provides the visibility/ordering semantics for publication of the reference.

However, whether an object is safely published depends on how it is constructed and whether its state is properly initialized before publication.

---

# 17. Immutable Objects + Volatile Reference

A useful pattern is replacing an immutable object rather than mutating shared state.

Example:

```java
private volatile Config config;
```

Build a new configuration:

```text
old Config
     ↓
new immutable Config
     ↓
volatile reference replaced
```

Readers can observe the new reference using the volatile semantics.

This can be useful for read-mostly configuration/state.

---

# 18. Why `volatile` Is Not Enough for Compound Operations

Consider:

```java
volatile int count;
```

Safe simple access:

```java
int x = count;
```

or:

```java
count = 10;
```

But:

```java
count++;
```

is not atomic.

Likewise:

```java
if (count > 0) {
    count--;
}
```

is not automatically safe.

And:

```java
if (balance >= amount) {
    balance -= amount;
}
```

is not automatically safe.

The general rule:

```text
Simple visibility-sensitive read/write
→ volatile may be appropriate

Check + update
Read + modify + write
Multiple related fields
→ need stronger coordination
```

---

# 19. Why Not Use `volatile` Everywhere?

Interview question:

> If volatile provides visibility, why don't we use volatile instead of synchronized?

Strong answer:

> Visibility and mutual exclusion solve different problems. `volatile` provides visibility and ordering guarantees for accesses to the variable, but it does not protect a compound operation or critical section from concurrent modification. `synchronized` additionally provides mutual exclusion.

---

# 20. Interview Question — `count++`

### Question

Is this thread-safe?

```java
private volatile int count = 0;

void increment() {
    count++;
}
```

### Answer

No.

`count++` is a read-modify-write operation. Multiple threads can read the same value and overwrite each other's updates. `volatile` provides visibility and ordering, not atomic increment.

For atomic increments, use an atomic class such as:

```java
AtomicInteger
```

or another appropriate synchronization mechanism.

---

# 21. Interview Question — Real Use Case

### Question

Give a real-world use case for `volatile`.

### Strong answer

> A simple shutdown or state flag is a common use case. A worker thread can repeatedly check a volatile `running` flag while another thread sets it to false to request shutdown. The operation primarily requires reliable visibility of the state change rather than mutual exclusion.

---

# 22. Interview Question — Bank Withdrawal

### Question

Why isn't `volatile` enough for a bank withdrawal?

### Strong answer

> Because the withdrawal is not a simple variable read or write. It is a compound operation: check whether the balance is sufficient and then update the balance. Two threads can both pass the check before either updates the value. `volatile` provides visibility but not mutual exclusion or atomicity of the whole check-and-update operation.

---

# 23. `volatile` vs `AtomicInteger`

Suppose:

```java
volatile int count;
```

You need:

```java
count++;
```

`volatile` is not enough.

Instead:

```java
AtomicInteger count = new AtomicInteger();
```

Then:

```java
count.incrementAndGet();
```

Now the increment is atomic.

Mental model:

```text
volatile
→ visibility + ordering

AtomicInteger
→ atomic operations
→ compare-and-set
→ increment/decrement
→ without traditional synchronized locking
```

---

# 24. `volatile` vs `AtomicInteger`

| Feature | `volatile int` | `AtomicInteger` |
|---|---:|---:|
| Visibility | ✅ | ✅ |
| Simple read/write | ✅ | ✅ |
| Atomic increment | ❌ | ✅ |
| `compareAndSet()` | ❌ | ✅ |
| Mutual exclusion | ❌ | ❌ |
| Monitor lock | ❌ | ❌ |

`AtomicInteger` is not simply "volatile with extra methods." It provides atomic operations using JVM/hardware-supported atomic mechanisms such as CAS.

---

# 25. Important Mental Model

```text
                 CONCURRENCY
                      │
          ┌───────────┴───────────┐
          ↓                       ↓
     Visibility             Mutual Exclusion
          │                       │
       volatile              synchronized
          │                       │
          │                  monitor/lock
          │                       │
          └──────────┬────────────┘
                     ↓
              Choose based on
               the problem
```

More precisely:

```text
volatile
→ visibility + ordering
→ no mutual exclusion
→ compound operations still need protection

synchronized
→ mutual exclusion
→ visibility
→ protected critical section
```

---

# 26. Coding Exercise — Shutdown Flag

Write:

```java
class Worker {

    private volatile boolean running = true;

    public void shutdown() {
        // stop the worker
    }

    public void work() {

        while (/* condition */) {
            System.out.println("Working...");
        }

        System.out.println("Stopped");
    }
}
```

Create:

```text
Thread 1 → worker.work()
Thread 2 → after 2 seconds, worker.shutdown()
```

Expected:

```text
Working...
Working...
Working...
...
Stopped
```

---

# 27. Coding Exercise — Prove `volatile` Doesn't Make `++` Atomic

Create:

```java
class Counter {

    private volatile int count = 0;

    public void increment() {
        count++;
    }

    public int getCount() {
        return count;
    }
}
```

Run two threads with:

```text
100,000 increments each
```

Expected mathematically:

```text
200000
```

But a lower value can occur.

Then ask:

> If volatile makes changes visible, why can updates still be lost?

Answer:

```text
Visibility
≠
Atomicity
```

---

# 28. Coding Exercise — Publication

Try:

```java
class Config {

    final int port;
    final String host;

    Config(int port, String host) {
        this.port = port;
        this.host = host;
    }
}
```

Then:

```java
private volatile Config config;
```

Experiment with one thread publishing:

```java
config = new Config(8080, "localhost");
```

and another thread reading:

```java
Config current = config;
```

The goal is to understand:

```text
volatile reference
→ publication/visibility semantics
```

---

# 29. Interview Questions Checklist

## Basic

1. What is `volatile`?
2. What problem does `volatile` solve?
3. Does `volatile` provide visibility?
4. Does `volatile` provide mutual exclusion?
5. Does `volatile` make `count++` thread-safe?

## Medium

6. Give a real-world use case for `volatile`.
7. Why is a shutdown flag a good use case?
8. Difference between `volatile` and `synchronized`?
9. Difference between visibility and atomicity?
10. Does `volatile` make an object thread-safe?
11. Does `volatile List<String>` make the list thread-safe?

## Advanced

12. Explain the happens-before relationship created by a volatile write/read.
13. Why can `volatile` solve visibility but not lost updates?
14. Why isn't `volatile` enough for bank withdrawal?
15. Can a volatile reference help publish an immutable object?
16. Why might `AtomicInteger` be preferred over `volatile int` for a counter?

---

# 30. MUFG Interview Answer

If asked:

> What is the difference between volatile and synchronized?

A strong answer:

> "`volatile` is primarily used when we need visibility and ordering guarantees for a shared variable without mutual exclusion. It doesn't make compound operations such as `count++` atomic. `synchronized`, on the other hand, provides mutual exclusion through a monitor and also establishes memory visibility guarantees, so it can protect a critical section containing multiple operations."

---

# 31. Where We Are

```text
Thread Basics                    ✅
Thread Lifecycle                 ✅
Race Conditions                  ✅
synchronized                     ✅
volatile                         ✅
```

Next:

```text
AtomicInteger / AtomicLong
        ↓
CAS (Compare-And-Set)
        ↓
Atomic operations
        ↓
Lock-free concepts
        ↓
ReentrantLock
        ↓
ExecutorService
        ↓
Thread Pools
        ↓
Future
        ↓
CompletableFuture
        ↓
Async Programming
```

The next topic, **AtomicInteger + CAS**, connects directly to the question:

> "If I don't want to use `synchronized`, but I need an atomic increment, how can Java do that?"
