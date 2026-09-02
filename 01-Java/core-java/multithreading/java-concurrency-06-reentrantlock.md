# Java Concurrency — 07: ReentrantLock

> `ReentrantLock` provides mutual exclusion like `synchronized`, but gives us more explicit control over locking.

---

# 1. Where We Are

```text
Thread Basics
      ↓
Thread Lifecycle
      ↓
Race Conditions
      ↓
synchronized
      ↓
volatile
      ↓
AtomicInteger + CAS
      ↓
ReentrantLock
```

The progression:

```text
synchronized
→ protect a critical section using a monitor

volatile
→ visibility + ordering

AtomicInteger
→ atomic operations on a variable

CAS
→ update a value only if it is still what I expect

ReentrantLock
→ explicit and flexible locking
```

---

# 2. Basic `synchronized`

```java
public synchronized void withdraw(int amount) {

    if (balance >= amount) {
        balance -= amount;
    }
}
```

Java automatically handles:

```text
acquire lock
    ↓
execute method
    ↓
release lock
```

If an exception occurs, the monitor is still released when leaving the synchronized method/block.

This makes `synchronized` convenient.

---

# 3. ReentrantLock

Import:

```java
import java.util.concurrent.locks.ReentrantLock;
```

Basic usage:

```java
class Account {

    private final ReentrantLock lock =
            new ReentrantLock();

    private int balance = 10000;

    public void withdraw(int amount) {

        lock.lock();

        try {

            if (balance >= amount) {
                balance -= amount;
            }

        } finally {

            lock.unlock();
        }
    }
}
```

The standard pattern is:

```java
lock.lock();

try {
    // critical section
} finally {
    lock.unlock();
}
```

---

# 4. Why `finally`?

Suppose:

```java
lock.lock();

try {
    process();

} finally {
    lock.unlock();
}
```

If `process()` throws an exception, `finally` still executes.

Without `finally`:

```java
lock.lock();

process();

lock.unlock();
```

If `process()` throws:

```text
process()
   ↓
Exception
   ↓
lock.unlock() never executes
```

The lock may remain held.

Therefore:

```text
lock()
   ↓
try
   ↓
finally
   ↓
unlock()
```

is the standard pattern.

---

# 5. Why Is It Called "Reentrant"?

**Reentrant means the same thread can acquire the same lock multiple times.**

Example:

```java
class Demo {

    private final ReentrantLock lock =
            new ReentrantLock();

    void methodA() {

        lock.lock();

        try {
            methodB();

        } finally {
            lock.unlock();
        }
    }

    void methodB() {

        lock.lock();

        try {
            System.out.println("Inside B");

        } finally {
            lock.unlock();
        }
    }
}
```

Suppose Thread A calls:

```text
methodA()
```

It acquires the lock.

Then `methodA()` calls `methodB()`.

`methodB()` tries to acquire the same lock.

Would Thread A deadlock itself?

No.

Because the lock is reentrant.

The same thread can acquire it again.

---

# 6. Lock Hold Count

You can mentally model reentrancy using a hold count.

First acquisition:

```text
Thread A
lock.lock()

hold count = 1
```

Second acquisition:

```text
Thread A
lock.lock()

hold count = 2
```

Then:

```java
lock.unlock();
```

reduces:

```text
2 → 1
```

Another:

```java
lock.unlock();
```

reduces:

```text
1 → 0
```

Only when the hold count reaches zero can another thread acquire the lock.

Mental model:

```text
lock()
→ hold count + 1

unlock()
→ hold count - 1

hold count == 0
→ lock becomes available
```

---

# 7. Important Rule

If a thread calls:

```java
lock.lock();
lock.lock();
```

it needs:

```java
lock.unlock();
lock.unlock();
```

The acquisitions and releases must match.

---

# 8. `ReentrantLock` vs `synchronized`

| Feature | `synchronized` | `ReentrantLock` |
|---|---|---|
| Mutual exclusion | Yes | Yes |
| Visibility | Yes | Yes |
| Reentrant | Yes | Yes |
| Manual lock/unlock | No | Yes |
| `tryLock()` | No | Yes |
| Timed lock attempt | No | Yes |
| Interruptible acquisition | Limited | Yes |
| Fairness option | No direct control | Yes |
| Multiple `Condition`s | No | Yes |

Big difference:

> `synchronized` is simpler and automatically managed, while `ReentrantLock` provides explicit and more flexible lock control.

---

# 9. `tryLock()`

With:

```java
lock.lock();
```

if another thread owns the lock:

```text
Thread A
   ↓
owns lock

Thread B
   ↓
lock.lock()
   ↓
waits
```

With:

```java
if (lock.tryLock()) {

    try {
        process();

    } finally {
        lock.unlock();
    }
}
```

Thread B can ask:

> "Can I get the lock right now?"

If yes:

```text
true
```

If no:

```text
false
```

Instead of waiting indefinitely.

---

# 10. Example of `tryLock()`

```java
if (lock.tryLock()) {

    try {
        System.out.println("Got lock");
        process();

    } finally {
        lock.unlock();
    }

} else {

    System.out.println(
        "Could not acquire lock"
    );
}
```

Flow:

```text
Try lock
   ↓
success → process
failure → fallback
```

---

# 11. Timed `tryLock()`

You can specify a maximum waiting time:

```java
if (lock.tryLock(2, TimeUnit.SECONDS)) {

    try {
        process();

    } finally {
        lock.unlock();
    }

} else {

    System.out.println(
        "Could not acquire lock within 2 seconds"
    );
}
```

Flow:

```text
try for up to 2 seconds
       ↓
success → execute
failure → fallback
```

This can be useful when you don't want a thread to wait indefinitely.

---

# 12. Why `tryLock()` Matters

Suppose a request needs a local lock.

Instead of:

```text
wait forever
```

you can:

```text
attempt lock
    ↓
not available
    ↓
don't block indefinitely
    ↓
retry / queue / fallback / return
```

This can be useful when latency matters.

However:

> `ReentrantLock` is still JVM-local.

---

# 13. `lockInterruptibly()`

Normally:

```java
lock.lock();
```

waits for the lock.

`ReentrantLock` also provides:

```java
lock.lockInterruptibly();
```

This allows a waiting thread to respond to interruption.

Example:

```java
try {

    lock.lockInterruptibly();

    try {
        process();

    } finally {
        lock.unlock();
    }

} catch (InterruptedException e) {

    Thread.currentThread().interrupt();

    System.out.println(
        "Thread was interrupted while waiting"
    );
}
```

---

# 14. `lock()` vs `lockInterruptibly()`

### `lock()`

```java
lock.lock();
```

The thread waits for the lock.

### `lockInterruptibly()`

```java
lock.lockInterruptibly();
```

The thread waits, but interruption can cause acquisition to stop with `InterruptedException`.

Mental model:

```text
lock()
→ wait until available

lockInterruptibly()
→ wait until available OR interruption
```

---

# 15. Fairness

`ReentrantLock` can optionally be created as a fair lock:

```java
ReentrantLock lock =
        new ReentrantLock(true);
```

The `true` requests fairness.

Conceptually:

```text
Thread A waiting
Thread B waiting
Thread C waiting
```

A fair lock attempts to give waiting threads access in a more orderly manner.

But:

> Fairness can have a performance cost.

Don't say:

> "Fair lock is always better."

It depends on the workload.

---

# 16. Non-Fair Lock

Default:

```java
ReentrantLock lock =
        new ReentrantLock();
```

This is non-fair.

A later-arriving thread may sometimes acquire the lock before an already-waiting thread.

Why is non-fair the default?

Because non-fair locking can provide better throughput in many workloads.

---

# 17. Fair vs Non-Fair

```text
Fair
→ more orderly access
→ potentially lower starvation risk
→ potentially lower throughput

Non-fair
→ potentially better throughput
→ less strict ordering
```

Fairness does not mean absolute scheduling guarantees.

---

# 18. `Condition`

One of the most important advanced features of `ReentrantLock` is `Condition`.

With `synchronized`, we have:

```java
wait();
notify();
notifyAll();
```

With `ReentrantLock`, we can create:

```java
Condition condition =
        lock.newCondition();
```

Then:

```java
condition.await();
condition.signal();
condition.signalAll();
```

This provides explicit waiting/signaling.

---

# 19. Producer-Consumer Example

Imagine:

```text
Producer
   ↓
adds item
   ↓
Queue

Consumer
   ↓
takes item
```

If the queue is empty:

```text
Consumer
   ↓
wait
```

When producer adds an item:

```text
Producer
   ↓
signal
   ↓
Consumer wakes
```

A `Condition` can express this relationship.

---

# 20. Basic Condition Example

```java
import java.util.LinkedList;
import java.util.Queue;
import java.util.concurrent.locks.Condition;
import java.util.concurrent.locks.ReentrantLock;

class Buffer {

    private final ReentrantLock lock =
            new ReentrantLock();

    private final Condition notEmpty =
            lock.newCondition();

    private final Queue<String> queue =
            new LinkedList<>();

    public String take()
            throws InterruptedException {

        lock.lock();

        try {

            while (queue.isEmpty()) {
                notEmpty.await();
            }

            return queue.poll();

        } finally {

            lock.unlock();
        }
    }

    public void put(String value) {

        lock.lock();

        try {

            queue.offer(value);

            notEmpty.signal();

        } finally {

            lock.unlock();
        }
    }
}
```

Flow:

```text
Consumer
   ↓
queue empty?
   ↓
await()
   ↓
releases lock while waiting
```

Producer:

```text
put()
   ↓
add item
   ↓
signal()
```

Consumer wakes and eventually reacquires the lock.

---

# 21. Important: `await()` Releases the Lock

When:

```java
condition.await();
```

is called:

```text
Thread
   ↓
holds lock
   ↓
await()
   ↓
lock released
   ↓
thread waits
```

Why release the lock?

Because otherwise the producer could never acquire the lock to add an item.

When signaled:

```text
signal()
   ↓
waiting thread becomes eligible
   ↓
must reacquire lock
   ↓
continues
```

---

# 22. Why `while`, Not `if`?

Use:

```java
while (queue.isEmpty()) {
    notEmpty.await();
}
```

not simply:

```java
if (queue.isEmpty()) {
    notEmpty.await();
}
```

When a thread wakes up, it should re-check the condition.

Correct pattern:

```text
wait
 ↓
wake
 ↓
re-check condition
 ↓
condition still false?
 ↓
wait again
```

This is a fundamental concurrency pattern.

---

# 23. `signal()` vs `signalAll()`

```java
condition.signal();
```

signals one waiting thread.

```java
condition.signalAll();
```

signals all waiting threads.

Mental model:

```text
signal()
→ wake one

signalAll()
→ wake all
```

Which is correct depends on the coordination problem.

---

# 24. `synchronized` vs `ReentrantLock` — Interview Answer

Question:

> Why would you use ReentrantLock instead of synchronized?

Strong answer:

> "`synchronized` is simpler and automatically releases the monitor when the synchronized block or method exits. `ReentrantLock` provides more explicit control, such as `tryLock`, timed lock acquisition, interruptible acquisition, configurable fairness, and multiple Condition objects. I would prefer `synchronized` when simple mutual exclusion is sufficient, and use `ReentrantLock` when I actually need those additional capabilities."

---

# 25. Important Mistake

Never do:

```java
lock.lock();

process();

lock.unlock();
```

without protection.

Instead:

```java
lock.lock();

try {
    process();

} finally {
    lock.unlock();
}
```

Because:

```text
process()
   ↓
RuntimeException
   ↓
lock.unlock() may never execute
```

Potential result:

```text
Lock remains held
      ↓
Other threads can't acquire it
      ↓
Potential application problem
```

---

# 26. ReentrantLock Is JVM-Local

This is extremely important for backend interviews.

Suppose:

```text
Kubernetes

Pod A → JVM A
Pod B → JVM B
Pod C → JVM C
```

Each has:

```java
ReentrantLock lock =
        new ReentrantLock();
```

You actually have:

```text
JVM A → Lock A
JVM B → Lock B
JVM C → Lock C
```

These are separate locks.

Therefore:

```text
ReentrantLock
       ↓
JVM-local
```

It cannot solve distributed locking across pods.

For distributed coordination, you need something such as:

```text
Database locking
Redis-based distributed lock
ZooKeeper / etcd
other distributed coordination mechanism
```

depending on the system.

---

# 27. ReentrantLock vs AtomicInteger

Don't confuse these.

## AtomicInteger

```text
atomic operation
↓
single piece of state
↓
CAS
```

Example:

```java
count.incrementAndGet();
```

## ReentrantLock

```text
mutual exclusion
↓
critical section
↓
multiple operations / state
```

Example:

```java
if (balance >= amount) {

    balance -= amount;

    saveTransaction();

    updateAudit();
}
```

The entire operation may need coordination.

---

# 28. ReentrantLock vs `synchronized`

## `synchronized`

```java
synchronized (lock) {
    process();
}
```

Advantages:

```text
Simple
Automatic release
Less boilerplate
Harder to forget unlock
```

## `ReentrantLock`

```java
lock.lock();

try {
    process();

} finally {
    lock.unlock();
}
```

Advantages:

```text
tryLock()
timed acquisition
interruptible acquisition
fairness option
Conditions
explicit lock control
```

---

# 29. Coding Exercise — Basic ReentrantLock

Implement:

```java
import java.util.concurrent.locks.ReentrantLock;

class Counter {

    private int count = 0;

    private final ReentrantLock lock =
            new ReentrantLock();

    public void increment() {

        // Protect count using the lock.
    }

    public int getCount() {
        return count;
    }
}
```

Run:

```text
2 threads
100,000 increments each
```

Expected:

```text
200000
```

---

# 30. Coding Exercise — `tryLock()`

Create two threads.

Thread A:

```text
acquire lock
sleep for 5 seconds
release
```

Thread B:

```text
tryLock()
```

Observe:

```text
false
```

Then experiment with:

```java
tryLock(2, TimeUnit.SECONDS)
```

and see what happens when Thread A releases the lock before/after those 2 seconds.

---

# 31. Coding Exercise — Reentrant Behavior

Create:

```java
void methodA() {

    lock.lock();

    try {
        methodB();

    } finally {
        lock.unlock();
    }
}

void methodB() {

    lock.lock();

    try {
        System.out.println("Inside B");

    } finally {
        lock.unlock();
    }
}
```

Verify that the same thread can acquire the lock twice.

Understand the hold count:

```text
methodA → hold count 1
methodB → hold count 2
methodB unlock → 1
methodA unlock → 0
```

---

# 32. Interview Questions

## Basic

1. What is `ReentrantLock`?
2. Why is it called reentrant?
3. How is it different from `synchronized`?
4. Why should `unlock()` be inside `finally`?
5. What does `tryLock()` do?

## Medium

6. Difference between `lock()` and `tryLock()`?
7. What is `lockInterruptibly()`?
8. What is a fair lock?
9. What is a `Condition`?
10. Difference between `await()` and `signal()`?
11. Why should `await()` generally be used inside a `while` loop?

## Advanced

12. Does `ReentrantLock` work across JVMs?
13. What happens if a thread acquires a ReentrantLock twice?
14. What happens if it unlocks only once?
15. Why might a fair lock have lower throughput?
16. How would you avoid waiting indefinitely for a lock?
17. `AtomicInteger` vs `ReentrantLock` — when would you choose each?

---

# 33. MUFG-Level Question

### Question

> You have multiple threads processing transactions. Why would you use `ReentrantLock` instead of `synchronized`?

Strong answer:

> "If simple mutual exclusion is enough, I would normally prefer synchronized because it is simpler and automatically manages the monitor. I would choose ReentrantLock when I need features such as timed lock acquisition using tryLock, interruptible acquisition, fairness, or Conditions for more sophisticated thread coordination."

---

# 34. MUFG-Level Question

### Question

> Can ReentrantLock solve concurrency when your application has multiple Kubernetes pods?

Answer:

> "No. ReentrantLock is JVM-local. Each pod has its own JVM and therefore its own lock instance. For coordination across pods, I would need a distributed consistency mechanism such as database locking or an appropriate distributed locking/coordination solution."

---

# 35. Final Mental Model

```text
synchronized
     ↓
simple mutual exclusion
     ↓
automatic monitor management

ReentrantLock
     ↓
explicit mutual exclusion
     ↓
more control
     ├── tryLock()
     ├── timed lock
     ├── interruptible lock
     ├── fairness
     └── Condition

AtomicInteger
     ↓
single-value atomic operations
     ↓
CAS
```

Remember:

```text
synchronized
→ simple lock

ReentrantLock
→ flexible lock

AtomicInteger
→ atomic state update

volatile
→ visibility + ordering
```

---

# 36. Where We Are

```text
Thread lifecycle                  ✅
Race conditions                  ✅
synchronized                     ✅
volatile                         ✅
AtomicInteger                    ✅
CAS                              ✅
ReentrantLock                    ✅
```

Next:

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

This is especially important for backend interviews because instead of manually doing:

```java
new Thread(...).start();
```

real backend applications generally manage concurrent work using:

```text
ExecutorService
       ↓
ThreadPoolExecutor
       ↓
worker threads
       ↓
tasks
```

That is the next major concurrency topic.
