You're right. **I made the same mistake twice.** Sorry about that.

I will **not invoke Deep Research again** for this interview prep. We'll continue directly in the chat, exactly as before.

# Multithreading — Q1

### What is the difference between a process and a thread?

**Process**

* Independent execution unit.
* Has its own memory/address space.
* Communication between processes is relatively expensive.
* Failure of one process normally doesn't directly corrupt another process's memory.

**Thread**

* Lightweight execution unit inside a process.
* Threads in the same process share heap and other process resources.
* Each thread has its own stack and execution state.
* Communication is easier because memory is shared—but that creates synchronization problems.

### Interview answer

> A process provides an isolated execution environment, while threads are execution units within a process that share resources such as heap memory. Threads are cheaper to create and communicate with, but shared memory introduces problems such as race conditions and visibility issues.

---

# Q2. What is the difference between `start()` and `run()`?

This is a classic trap.

```java
Thread t = new Thread(() -> {
    System.out.println("Hello");
});

t.start();
```

`start()` asks the JVM to start a **new thread**, which eventually invokes `run()`.

But:

```java
t.run();
```

is just a normal method call.

It executes on the **current thread**.

### Remember

```text
start()
   ↓
new thread
   ↓
run()
```

Whereas:

```text
run()
   ↓
same current thread
```

---

# Q3. What happens if `start()` is called twice?

```java
Thread t = new Thread(...);

t.start();
t.start();
```

This results in:

```text
IllegalThreadStateException
```

A Java `Thread` instance can be started only once.

---

# Q4. What are the main thread states?

Java's `Thread.State` contains:

```text
NEW
RUNNABLE
BLOCKED
WAITING
TIMED_WAITING
TERMINATED
```

### Example

```text
NEW
 ↓ start()
RUNNABLE
 ↓
running / executing
 ↓
WAITING / BLOCKED / TIMED_WAITING
 ↓
RUNNABLE
 ↓
TERMINATED
```

Important nuance:

Java's `RUNNABLE` includes threads that are actually running **and** threads that are ready to run, depending on scheduling.

---

# Q5. What is a race condition?

A race condition occurs when multiple threads access shared state concurrently and the result depends on the timing/interleaving of those operations.

Example:

```java
class Counter {
    int count = 0;

    void increment() {
        count++;
    }
}
```

You might think:

```text
count++
```

is one operation.

It isn't.

Conceptually:

```text
read count
add 1
write count
```

Two threads can do:

```text
Thread A: read 10
Thread B: read 10

Thread A: write 11
Thread B: write 11
```

Expected:

```text
12
```

Actual:

```text
11
```

That's a race condition.

---

# Q6. How do you fix it?

### Option 1 — synchronized

```java
synchronized void increment() {
    count++;
}
```

### Option 2 — AtomicInteger

```java
AtomicInteger count = new AtomicInteger();

count.incrementAndGet();
```

### Option 3 — Lock

```java
lock.lock();

try {
    count++;
} finally {
    lock.unlock();
}
```

Which one you choose depends on the problem.

---

# Q7. What does `synchronized` actually provide?

Two major things:

### 1. Mutual exclusion

Only one thread at a time can execute the protected critical section for the same monitor.

### 2. Memory visibility / ordering

Entering and exiting a monitor establishes happens-before relationships.

Example:

```java
synchronized (lock) {
    sharedValue = 100;
}
```

Another thread acquiring the **same lock** after the first thread releases it can see the updated state.

So don't describe `synchronized` merely as:

> "It prevents two threads from executing together."

A stronger answer is:

> `synchronized` provides mutual exclusion and establishes memory-visibility guarantees through monitor happens-before semantics.

---

# Q8. What is `volatile`?

`volatile` is primarily about **visibility and ordering**, not mutual exclusion.

```java
volatile boolean running = true;
```

Thread A:

```java
while (running) {
    // work
}
```

Thread B:

```java
running = false;
```

Without proper synchronization, Thread A isn't guaranteed to observe the update promptly.

With `volatile`, writes to `running` become visible to other threads reading it.

---

# Q9. Is `volatile` enough for `count++`?

**No.**

This:

```java
volatile int count;

count++;
```

is still unsafe.

Because:

```text
read
+
write
```

is a compound operation.

Two threads can still interfere.

Use:

```java
AtomicInteger count = new AtomicInteger();

count.incrementAndGet();
```

or appropriate locking.

### Interview one-liner

> `volatile` guarantees visibility, but it does not make compound operations atomic.

🔥 Remember this.

---

# Q10. `volatile` vs `synchronized`

|                            | `volatile`  | `synchronized`    |
| -------------------------- | ----------- | ----------------- |
| Visibility                 | Yes         | Yes               |
| Mutual exclusion           | No          | Yes               |
| Atomic compound operations | No          | Yes               |
| Locking                    | No          | Yes               |
| Typical use                | state flags | critical sections |

Example where volatile works well:

```java
private volatile boolean shutdown;
```

Example requiring synchronization:

```java
balance = balance - withdrawal;
```

because multiple operations must be coordinated.

---

# Q11. What is the Java Memory Model?

The Java Memory Model defines how threads interact with memory and, importantly, what guarantees Java provides regarding:

* visibility
* ordering
* atomicity

The compiler/JVM/CPU are allowed to reorder operations as long as single-threaded behavior remains correct.

But in multithreaded code, without proper synchronization, you cannot assume another thread will observe operations in the order you wrote them.

That's why mechanisms such as:

```text
synchronized
volatile
Lock
Atomic classes
concurrent collections
```

matter.

---

# Q12. What is happens-before?

This is a **senior-level interview question**.

A happens-before relationship means that the effects of one action are guaranteed to be visible/ordered before another action according to the Java Memory Model.

Examples include:

### Unlock → subsequent lock

If Thread A releases a monitor and Thread B subsequently acquires that same monitor, A's actions before the unlock happen-before B's actions after the lock.

### Volatile write → subsequent volatile read

A write to a volatile variable happens-before a subsequent read of that same variable.

### Thread start

Actions before:

```java
thread.start();
```

happen-before actions in the started thread.

### Thread termination/join

Actions performed by a thread happen-before another thread successfully returns from:

```java
thread.join();
```

### Why interviewers care

Because "thread safety" isn't just about locks.

It's fundamentally about **atomicity + visibility + ordering**.

---

# Q13. What is deadlock?

Deadlock occurs when threads wait indefinitely for resources held by each other.

Classic example:

```java
Thread A:
lock1 → acquired
lock2 → waiting

Thread B:
lock2 → acquired
lock1 → waiting
```

Neither can proceed.

```text
A owns lock1 → wants lock2
B owns lock2 → wants lock1
```

💥 Deadlock.

---

# Q14. How do you prevent deadlocks?

Common strategies:

### 1. Consistent lock ordering

Always acquire:

```text
lock1 → lock2
```

Never:

```text
lock2 → lock1
```

### 2. Try-lock with timeout

```java
if (lock.tryLock(1, TimeUnit.SECONDS)) {
    try {
        // work
    } finally {
        lock.unlock();
    }
}
```

### 3. Reduce lock scope

Don't hold a lock while doing:

```text
HTTP call
DB call
Kafka call
```

if you can avoid it.

### 4. Avoid unnecessary nested locks

---

# Q15. Deadlock vs starvation vs livelock?

### Deadlock

Threads are blocked waiting for each other.

### Starvation

A thread doesn't get sufficient access to a resource because other threads continually get it.

### Livelock

Threads aren't blocked, but they're repeatedly reacting to each other and making no useful progress.

Think:

```text
Deadlock  → nobody moves
Starvation → one doesn't get a chance
Livelock   → everybody moves, nobody progresses
```

---

# Q16. What is `AtomicInteger`?

`AtomicInteger` provides atomic operations without requiring explicit locking for those operations.

```java
AtomicInteger counter = new AtomicInteger(0);

counter.incrementAndGet();
counter.decrementAndGet();

int value = counter.get();
```

It uses low-level atomic mechanisms such as CAS.

### CAS

Compare-And-Set conceptually:

```text
if current == expected
    replace with new value
else
    fail/retry
```

This is useful for lock-free/low-lock algorithms.

---

# Q17. `AtomicInteger` vs `synchronized`?

Don't answer:

> AtomicInteger is always faster.

That's too simplistic.

### AtomicInteger

Good for simple atomic state transitions:

```java
counter.incrementAndGet();
```

### synchronized

Better when multiple variables/operations must change as **one consistent transaction**.

Example:

```java
synchronized void transfer(Account from, Account to, int amount) {
    from.balance -= amount;
    to.balance += amount;
}
```

An `AtomicInteger` alone doesn't make this multi-object operation atomic.

---

# Q18. What is `ReentrantLock`?

`ReentrantLock` is an explicit locking mechanism from `java.util.concurrent.locks`.

```java
Lock lock = new ReentrantLock();

lock.lock();

try {
    // critical section
} finally {
    lock.unlock();
}
```

### Why `finally`?

Because if an exception occurs and you don't unlock:

```text
lock remains held
       ↓
other threads may wait indefinitely
```

---

# Q19. `synchronized` vs `ReentrantLock`

`ReentrantLock` provides capabilities beyond basic `synchronized`, such as:

* `tryLock()`
* timed lock acquisition
* interruptible lock acquisition
* optional fairness policy
* multiple `Condition`s

Example:

```java
if (lock.tryLock(2, TimeUnit.SECONDS)) {
    try {
        // work
    } finally {
        lock.unlock();
    }
}
```

### When would you choose synchronized?

If you simply need mutual exclusion, `synchronized` is often simpler and less error-prone.

Don't use `ReentrantLock` merely because it sounds more advanced.

---

# Q20. What is `ConcurrentHashMap`?

A thread-safe map designed for concurrent access.

```java
ConcurrentHashMap<String, Integer> map =
        new ConcurrentHashMap<>();
```

Unlike legacy `Hashtable`, modern `ConcurrentHashMap` doesn't use one global lock around the entire map.

Modern implementations use a combination of:

* CAS
* fine-grained synchronization around affected bins
* volatile state/accesses

This allows much greater concurrency.

### Important

`ConcurrentHashMap` does **not allow null keys or null values**.

---

# Q21. Why not use `HashMap` with multiple threads?

Because concurrent structural modifications can lead to incorrect behavior and data races.

This is unsafe:

```java
Map<String, Integer> map = new HashMap<>();

// multiple threads modifying map
```

You can use:

```java
ConcurrentHashMap
```

or externally synchronize appropriately.

---

# Q22. Is `ConcurrentHashMap.get()` enough to make compound operations safe?

No.

This is a common trap.

```java
if (!map.containsKey(key)) {
    map.put(key, value);
}
```

The combination isn't atomic.

Another thread can modify the map between the two calls.

Use atomic APIs:

```java
map.putIfAbsent(key, value);
```

or:

```java
map.computeIfAbsent(key, k -> createValue(k));
```

That's the kind of answer that separates basic knowledge from production-level concurrency knowledge.

---

# Q23. What is ExecutorService?

Instead of manually creating threads:

```java
new Thread(...).start();
```

you can submit tasks to an executor.

```java
ExecutorService executor =
        Executors.newFixedThreadPool(10);

executor.submit(() -> doWork());

executor.shutdown();
```

The executor manages:

* worker threads
* task submission
* task execution
* shutdown

This separates:

```text
What work should execute?
        from
How should threads execute it?
```

---

# Q24. What is `ThreadPoolExecutor`?

It's the configurable implementation behind many executor configurations.

Important parameters include:

```text
corePoolSize
maximumPoolSize
keepAliveTime
workQueue
threadFactory
rejectedExecutionHandler
```

Conceptually:

```text
Task
 ↓
Is worker available?
 ↓
queue
 ↓
create more workers if appropriate
 ↓
reject if saturated
```

This is much more important for a senior interview than simply memorizing:

```java
Executors.newFixedThreadPool()
```

---

# Q25. CPU-bound vs I/O-bound thread pools?

### CPU-bound

Examples:

```text
image processing
encryption
complex calculations
compression
```

You generally don't want thousands of threads competing for a small number of CPU cores.

A common starting point is around:

```text
number of CPUs
```

or sometimes slightly above it, then benchmark.

### I/O-bound

Examples:

```text
DB calls
HTTP calls
file I/O
```

Threads spend substantial time waiting.

You can have more concurrency, but the correct number depends on:

* latency
* downstream capacity
* CPU
* memory
* connection pools
* workload

### Don't say:

> I/O pool should always be CPU × 2.

There is no universal magic formula.

---

# Q26. What happens if a thread pool is exhausted?

Suppose:

```text
10 worker threads
1000 tasks
```

The tasks typically enter the work queue depending on the executor configuration.

If both workers and queue capacity are exhausted, the executor invokes its rejection policy.

Common policies:

```text
AbortPolicy
CallerRunsPolicy
DiscardPolicy
DiscardOldestPolicy
```

### Production insight

`CallerRunsPolicy` can provide a form of natural backpressure because the submitting thread executes the task itself.

But you need to understand the latency consequences before choosing it.

---

# Q27. `Runnable` vs `Callable`

### Runnable

```java
Runnable task = () -> {
    System.out.println("Hello");
};
```

Doesn't return a result.

### Callable

```java
Callable<Integer> task = () -> {
    return 42;
};
```

Can:

* return a result
* throw checked exceptions

Submit:

```java
Future<Integer> future = executor.submit(task);
```

---

# Q28. What is Future?

`Future` represents the result of an asynchronous computation.

```java
Future<Integer> future =
        executor.submit(() -> calculate());
```

You can do:

```java
Integer result = future.get();
```

### Problem?

`get()` can block.

That's one reason `CompletableFuture` became important for composing asynchronous workflows.

---

# Q29. `Future` vs `CompletableFuture`

### Future

Mostly:

```text
submit
   ↓
wait
   ↓
get
```

### CompletableFuture

Allows composition:

```text
A
 ↓
thenApply
 ↓
B
 ↓
thenCompose
 ↓
C
```

And combining:

```text
A ──┐
    ├── thenCombine → C
B ──┘
```

And structured error handling:

```text
exceptionally
handle
whenComplete
```

---

# Q30. CountDownLatch vs CyclicBarrier

### CountDownLatch

One-time countdown.

Example:

```text
Main thread waits
       ↓
Worker A ── countDown()
Worker B ── countDown()
Worker C ── countDown()
       ↓
count = 0
       ↓
Main continues
```

Once it reaches zero, it cannot be reset.

### CyclicBarrier

Multiple threads wait for each other at a barrier.

```text
A ──┐
B ──┼── barrier
C ──┘
     ↓
all continue
```

It can be reused.

### Easy memory trick

```text
CountDownLatch → wait for events
CyclicBarrier  → wait for threads
```

---

# Q31. What is Semaphore?

A semaphore controls how many threads can access a resource concurrently.

Example:

```java
Semaphore semaphore = new Semaphore(10);
```

At most 10 permits can be acquired simultaneously.

This is very useful for **limiting concurrency**.

For example:

```text
1000 requests
     ↓
Semaphore(20)
     ↓
only 20 expensive operations concurrently
```

This is different from a lock:

```text
Lock → generally one owner at a time
Semaphore(20) → up to 20 permits
```

---

# Q32. Producer-consumer problem

Classic concurrency scenario.

Producer:

```text
produces messages
      ↓
queue
      ↓
consumer processes
```

Don't build this with:

```java
ArrayList
```

and manually synchronize everything unless there's a very specific reason.

Use:

```java
BlockingQueue
```

Example:

```java
BlockingQueue<String> queue =
        new ArrayBlockingQueue<>(100);
```

Producer:

```java
queue.put(message);
```

Consumer:

```java
String message = queue.take();
```

The queue handles the blocking coordination.

---

# Q33. Why is a bounded queue important?

Imagine an API receiving:

```text
10,000 requests/sec
```

but your downstream system can process:

```text
1,000/sec
```

If you allow an unlimited queue:

```text
requests
   ↓
huge queue
   ↓
memory grows
   ↓
latency grows
   ↓
OOM
```

A bounded queue gives you a chance to apply:

```text
backpressure
rejection
rate limiting
load shedding
```

This is extremely relevant in microservices.

---

# Q34. How would you gracefully shut down an ExecutorService?

Don't simply kill worker threads.

Use:

```java
executor.shutdown();
```

This prevents new tasks from being submitted while allowing already submitted tasks to complete.

Then:

```java
if (!executor.awaitTermination(30, TimeUnit.SECONDS)) {
    executor.shutdownNow();
}
```

But `shutdownNow()` is an **interrupt request**, not a guarantee that tasks immediately stop.

Tasks must cooperate with interruption.

---

# Q35. Production scenario: API becomes slow under heavy load. You discover your application creates a new thread for every incoming task. What would you investigate?

I'd investigate:

1. Thread count
2. Thread pool configuration
3. Queue size
4. Task rejection
5. CPU utilization
6. GC pressure
7. DB connection pool
8. HTTP connection pool
9. Downstream latency
10. Thread dumps

Then determine whether the bottleneck is:

```text
CPU
DB
network
locks
queue saturation
GC
downstream service
```

A senior engineer shouldn't immediately say:

> "Increase the thread pool."

That can make the problem worse.

If the DB allows only 50 connections and you increase workers from 100 to 1000, you may simply create more waiting and contention.

---

# Q36. How do you diagnose deadlocks in production?

Take a **thread dump**.

Look for:

```text
BLOCKED
waiting to acquire lock
locked by another thread
```

Tools commonly used include:

```text
jstack
JDK Mission Control
VisualVM
thread dumps from container/JVM tooling
```

You trace the lock ownership cycle:

```text
Thread A → waiting for Lock B
Thread B → waiting for Lock A
```

Then fix the locking strategy.

---

# 🔥 Multithreading scenarios you absolutely should be able to answer

### Scenario 1

> Two users simultaneously book the last available seat. How do you prevent both from succeeding?

Possible approaches:

* database row locking
* optimistic locking/version column
* atomic update such as `UPDATE ... WHERE available = true`
* transaction boundary
* unique constraint depending on design

Don't solve a database concurrency problem purely with a Java `synchronized` block if you're running multiple application instances.

That's a **very important microservices lesson**.

---

### Scenario 2

> Two instances of your microservice receive the same request simultaneously. Can `synchronized` solve it?

**No.**

`synchronized` only coordinates threads within the same JVM.

For multiple instances you need something shared, such as:

```text
database constraint
database lock
distributed lock
idempotency key
Kafka partitioning
atomic Redis operation
```

depending on the actual problem.

---

### Scenario 3

> You have 500 concurrent HTTP requests and every request makes a slow DB call. What would you inspect?

Think:

```text
500 requests
 ↓
thread/virtual-thread concurrency
 ↓
DB connection pool
 ↓
DB max connections
 ↓
query latency
 ↓
locks
 ↓
indexes
 ↓
transaction duration
```

The thread pool may not be the root problem.

---

## 🔥 The 10 concurrency statements to memorize

1. **`volatile` gives visibility, not compound-operation atomicity.**
2. **`synchronized` gives mutual exclusion and memory visibility guarantees.**
3. **`AtomicInteger` is good for atomic single-variable operations.**
4. **`ConcurrentHashMap` is for concurrent map access; it doesn't make arbitrary multi-step logic atomic.**
5. **`start()` creates/schedules a new thread; `run()` is just a method call.**
6. **`Future.get()` can block; `CompletableFuture` supports composition.**
7. **`CountDownLatch` is one-shot; `CyclicBarrier` is reusable.**
8. **`Semaphore` limits concurrent access using permits.**
9. **A bounded queue helps provide backpressure.**
10. **For distributed concurrency, JVM-level locks like `synchronized` are insufficient.**

Next up is **Spring Core + Spring Boot**, where we'll hit the questions that are especially important for your experience level: **IoC/DI internals, bean lifecycle, `@Component` vs `@Bean`, constructor injection, circular dependencies, `@Lazy`, scopes, proxies, auto-configuration, embedded Tomcat, profiles, configuration, Actuator, and `@Transactional` foundations.**
