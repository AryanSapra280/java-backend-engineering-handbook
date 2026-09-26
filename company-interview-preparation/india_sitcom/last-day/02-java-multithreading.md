Absolutely. We’ll keep the same approach: **examples + implementation thinking + interviewer follow-ups + edge cases**. I’ll also make sure we cover the things that are easy to miss.

# TOPIC 2 — MULTITHREADING & CONCURRENCY

For a 5-year Java backend interview, don't stop at:

> "A thread is a lightweight process."

They can easily move from a simple question into:

```text
Thread
→ ExecutorService
→ Thread Pool
→ Race Condition
→ synchronized
→ volatile
→ AtomicInteger
→ Locks
→ ConcurrentHashMap
→ CompletableFuture
→ Spring async
→ Deadlocks
→ Thread starvation
→ async microservices
→ high-TPS payment processing
```

---

# SECTION 1 — THREAD BASICS

## Q1. What is a thread?

A thread is an independent execution path within a process.

For example:

```java
public class Demo {

    public static void main(String[] args) {

        Thread thread = new Thread(() -> {
            System.out.println("Running in another thread");
        });

        thread.start();

        System.out.println("Main thread");
    }
}
```

Important:

```java
thread.start();
```

creates/schedules a new execution path.

But:

```java
thread.run();
```

does **not** start a new thread. It simply invokes the method on the current thread.

🔥 Interview trap:

> What is the difference between `start()` and `run()`?

Answer:

```text
start()
→ asks JVM/runtime to execute the thread concurrently

run()
→ normal method invocation
```

---

# Q2. Can you call `start()` twice?

No.

```java
Thread t = new Thread(...);

t.start();
t.start();
```

The second invocation results in:

```text
IllegalThreadStateException
```

A Java `Thread` instance cannot be restarted after termination.

---

# Q3. What are the basic thread states?

You should know:

```text
NEW
 ↓
RUNNABLE
 ↓
BLOCKED / WAITING / TIMED_WAITING
 ↓
TERMINATED
```

For example:

```java
Thread.sleep(1000);
```

puts the thread into `TIMED_WAITING`.

If it is waiting to acquire a monitor:

```java
synchronized(lock) {
}
```

it can be `BLOCKED`.

---

# SECTION 2 — WHY NOT CREATE THREADS MANUALLY?

## Q4. What's wrong with this?

```java
for (int i = 0; i < 10000; i++) {
    new Thread(() -> process()).start();
}
```

You can create a huge number of threads.

Problems include:

* memory consumption
* thread creation overhead
* context switching
* scheduling overhead
* difficult lifecycle management
* potentially overwhelming downstream systems

Instead, use a **thread pool**.

---

# SECTION 3 — EXECUTOR SERVICE

## Q5. What is ExecutorService?

It separates:

```text
Task submission
```

from:

```text
Thread management
```

Example:

```java
ExecutorService executor =
        Executors.newFixedThreadPool(5);

executor.submit(() -> {
    processPayment();
});

executor.shutdown();
```

Instead of creating a new thread for every task, the executor maintains a pool.

Conceptually:

```text
              ExecutorService
                    |
        ┌───────────┼───────────┐
        ↓           ↓           ↓
     Thread 1    Thread 2    Thread 3
```

---

# Q6. `execute()` vs `submit()`?

### `execute()`

```java
executor.execute(task);
```

Used for a `Runnable`.

No `Future` result.

### `submit()`

```java
Future<?> future = executor.submit(task);
```

Returns a `Future`.

You can retrieve the result:

```java
Future<Integer> future =
        executor.submit(() -> 10 + 20);

Integer result = future.get();
```

But be careful:

```java
future.get();
```

is blocking.

---

# Q7. Why can an unbounded thread pool/queue be dangerous?

Suppose your service receives:

```text
100,000 requests
```

and each task takes time.

If tasks are simply accumulated without controlling resource usage, memory and latency can explode.

This leads to:

```text
Traffic spike
   ↓
More tasks
   ↓
Queue grows
   ↓
Memory pressure
   ↓
Latency increases
   ↓
Timeouts
   ↓
Retries
   ↓
Even more load
```

This is how systems can spiral into failure.

---

# SECTION 4 — RACE CONDITIONS

This is **extremely important** for payment/wallet systems.

## Q8. What is a race condition?

A race condition occurs when the result depends on the timing/interleaving of concurrent operations.

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

Suppose:

```text
Initial = 0
```

Two threads:

```text
Thread 1: read 0
Thread 2: read 0

Thread 1: write 1
Thread 2: write 1
```

Expected:

```text
2
```

Actual:

```text
1
```

That's a race condition.

---

# Q9. How do you fix it?

### Option 1 — synchronized

```java
public synchronized void increment() {
    count++;
}
```

Now only one thread at a time can execute that synchronized method on the same object monitor.

---

# Q10. What exactly does `synchronized` guarantee?

It provides important **mutual exclusion and memory visibility guarantees** around the monitor.

If:

```java
synchronized(lock) {
    ...
}
```

one thread owns that monitor, another thread attempting to enter the same monitor must wait.

It also establishes happens-before relationships around monitor unlock/lock operations.

---

# Q11. Is `synchronized` enough for distributed systems?

🔥🔥 Very important.

No.

Suppose:

```text
Payment Service
   |
   +--- Pod 1
   +--- Pod 2
   +--- Pod 3
```

Each pod is a separate JVM.

If you do:

```java
synchronized
```

inside Pod 1, it doesn't prevent Pod 2 from executing the same operation.

So:

```text
synchronized
```

provides **JVM-level coordination**, not distributed locking.

For distributed state, you may need:

* database transactions
* optimistic locking
* pessimistic locking
* atomic SQL updates
* distributed locks where justified
* partition ownership

This distinction is highly likely to matter in your payment interview.

---

# SECTION 5 — VOLATILE

## Q12. What is `volatile`?

`volatile` primarily provides **visibility guarantees** for a variable between threads.

Example:

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
    }
}
```

Without appropriate synchronization/visibility guarantees, another thread isn't guaranteed to observe updates promptly.

`volatile` helps ensure reads see an appropriately synchronized value.

---

# Q13. Does volatile make `count++` thread-safe?

🔥🔥 **No.**

```java
volatile int count;

count++;
```

is still:

```text
read
+
increment
+
write
```

The operation isn't atomic.

So:

```text
volatile → visibility
```

doesn't mean:

```text
volatile → atomicity
```

---

# SECTION 6 — ATOMIC CLASSES

## Q14. How would you make a counter thread-safe without synchronized?

Use:

```java
AtomicInteger
```

Example:

```java
AtomicInteger counter = new AtomicInteger(0);

counter.incrementAndGet();
```

Atomic classes provide atomic operations using low-level concurrency mechanisms.

---

# Q15. `AtomicInteger` vs `volatile int`

|                            | volatile int | AtomicInteger |
| -------------------------- | ------------ | ------------- |
| Visibility                 | Yes          | Yes           |
| Atomic increment           | No           | Yes           |
| `compareAndSet()`          | No           | Yes           |
| Compound atomic operations | Limited      | Yes           |

Example:

```java
if (counter.compareAndSet(10, 11)) {
    ...
}
```

This is useful for lock-free state transitions.

---

# SECTION 7 — LOCKS

## Q16. `synchronized` vs `ReentrantLock`

`ReentrantLock` gives you additional control.

Example:

```java
Lock lock = new ReentrantLock();

lock.lock();

try {
    process();
} finally {
    lock.unlock();
}
```

The `finally` is critical.

Otherwise an exception could leave the lock held.

---

# Q17. Why would you use ReentrantLock?

Features can include:

* `tryLock()`
* interruptible lock acquisition
* fairness option
* explicit lock/unlock
* multiple condition variables

For example:

```java
if (lock.tryLock(2, TimeUnit.SECONDS)) {
    try {
        process();
    } finally {
        lock.unlock();
    }
}
```

Instead of waiting forever for a lock, you can time out.

---

# SECTION 8 — DEADLOCK

## Q18. What is a deadlock?

Two or more threads wait forever for resources held by each other.

Example:

```java
Thread 1:

lockA.lock();

lockB.lock();
```

while:

```java
Thread 2:

lockB.lock();

lockA.lock();
```

Potentially:

```text
Thread 1 owns A
Thread 2 owns B

Thread 1 waits for B
Thread 2 waits for A

        DEADLOCK
```

---

# Q19. How do you prevent deadlocks?

A strong answer:

### 1. Consistent lock ordering

Always:

```text
A → B
```

Never:

```text
Thread 1: A → B
Thread 2: B → A
```

### 2. Avoid holding locks unnecessarily

### 3. Use `tryLock()` with timeout where appropriate

### 4. Keep critical sections small

### 5. Avoid nested locks where possible

---

# SECTION 9 — WAIT VS SLEEP

## Q20. Difference between `wait()` and `sleep()`?

### `Thread.sleep()`

```java
Thread.sleep(1000);
```

The thread sleeps for a duration.

It does **not release locks it already holds merely because it is sleeping**.

### `wait()`

```java
synchronized(lock) {
    lock.wait();
}
```

The thread waits and releases the associated monitor while waiting.

It must be used with the appropriate monitor.

---

# Q21. How does `notify()` work?

Example:

```java
synchronized (lock) {

    while (!condition) {
        lock.wait();
    }

    process();
}
```

Another thread:

```java
synchronized (lock) {
    condition = true;
    lock.notifyAll();
}
```

Use a `while` loop around the condition rather than assuming a notification guarantees the condition is true.

---

# SECTION 10 — COUNTDOWNLATCH

## Q22. What is CountDownLatch?

Useful when one thread must wait until several operations complete.

Example:

```java
CountDownLatch latch =
        new CountDownLatch(3);
```

Three workers:

```java
executor.submit(() -> {
    loadPaymentData();
    latch.countDown();
});
```

Main thread:

```java
latch.await();
```

It waits until:

```text
3 → 2 → 1 → 0
```

Then proceeds.

### Example use case

You need to load:

```text
Customer
Payment
Merchant
```

in parallel before constructing a response.

---

# Q23. CountDownLatch vs CyclicBarrier?

### CountDownLatch

Generally one-shot.

Once count reaches zero, it stays open.

### CyclicBarrier

Multiple threads can repeatedly wait for each other at a common barrier.

For most backend interviews, know this distinction rather than memorizing implementation details.

---

# SECTION 11 — CONCURRENT COLLECTIONS

## Q24. Why can't you simply use HashMap from multiple threads?

Because concurrent modifications aren't safely coordinated.

For example:

```java
Map<String, Integer> map =
        new HashMap<>();
```

Multiple threads modifying it concurrently can lead to race conditions and inconsistent behavior.

Use:

```java
ConcurrentHashMap
```

where appropriate.

---

# Q25. Is this thread-safe?

```java
ConcurrentHashMap<String, Integer> map;

map.put("count", 0);

map.put("count", map.get("count") + 1);
```

🔥 **No.**

Even though each individual operation is thread-safe:

```text
get()
put()
```

the combined operation isn't atomic.

Two threads can do:

```text
Thread 1: get → 0
Thread 2: get → 0

Thread 1: put → 1
Thread 2: put → 1
```

Use:

```java
map.compute("count", (k, v) -> v + 1);
```

or an `AtomicInteger` value depending on the use case.

---

# SECTION 12 — COMPLETABLEFUTURE

This is important because your HR specifically mentioned **async programming**.

## Q26. What is CompletableFuture?

It represents the eventual result of an asynchronous computation.

Example:

```java
CompletableFuture<String> future =
        CompletableFuture.supplyAsync(() -> {
            return "Payment processed";
        });
```

You can compose operations without manually blocking.

---

# Q27. `thenApply()` vs `thenCompose()`

🔥 Very important.

### `thenApply`

Transforms a result.

```java
CompletableFuture<String> result =
    getPayment()
        .thenApply(payment -> payment.getPaymentId());
```

Conceptually:

```text
Future<Payment>
      ↓
Future<String>
```

### `thenCompose`

Used when the next operation itself returns a Future.

Suppose:

```java
CompletableFuture<Payment> getPayment();

CompletableFuture<Merchant> getMerchant(Payment payment);
```

Then:

```java
getPayment()
    .thenCompose(payment ->
        getMerchant(payment)
    );
```

Without `thenCompose`, you can end up with:

```text
CompletableFuture<CompletableFuture<Merchant>>
```

`thenCompose()` flattens that asynchronous chain.

---

# Q28. `thenApply()` vs `thenAccept()`?

### `thenApply`

Transforms and returns a value.

```java
future.thenApply(result -> result.toUpperCase());
```

### `thenAccept`

Consumes the result and returns no meaningful result.

```java
future.thenAccept(result ->
    System.out.println(result)
);
```

---

# Q29. How do you handle exceptions in CompletableFuture?

Several mechanisms exist.

### `exceptionally`

```java
future.exceptionally(ex -> {
    log.error("Failed", ex);
    return "fallback";
});
```

### `handle`

Handles both success and failure:

```java
future.handle((result, ex) -> {

    if (ex != null) {
        return fallback();
    }

    return result;
});
```

### `whenComplete`

Useful for side effects/logging:

```java
future.whenComplete((result, ex) -> {
    log.info("Completed");
});
```

---

# Q30. What's the difference between `handle()` and `whenComplete()`?

Important distinction:

`handle()` can **transform the result**.

```java
future.handle((result, ex) -> "new result");
```

`whenComplete()` is mainly for observing completion and generally passes the original outcome onward rather than replacing it with a transformed value.

---

# SECTION 13 — CUSTOM THREAD POOL

## Q31. Why shouldn't you blindly use `CompletableFuture.supplyAsync()`?

By default, asynchronous methods such as `supplyAsync()` use the common ForkJoinPool unless you provide an executor.

You may want:

```java
ExecutorService executor =
        Executors.newFixedThreadPool(10);

CompletableFuture
    .supplyAsync(() -> callPSP(), executor);
```

Why?

Because you don't want an unrelated workload consuming the same shared pool.

This is especially important in backend systems.

---

# Q32. What happens if you use too many threads?

More threads don't necessarily mean more performance.

Eventually:

```text
More threads
    ↓
Context switching
    ↓
CPU overhead
    ↓
Memory usage
    ↓
Contention
    ↓
Performance degradation
```

Thread pool sizing depends on workload.

### CPU-bound

Usually fewer threads, often related to CPU cores.

### I/O-bound

You can often have more concurrency because threads spend time waiting for I/O.

But the actual limit is also determined by:

```text
DB connections
HTTP connections
downstream capacity
CPU
memory
latency
```

---

# SECTION 14 — ASYNC DOES NOT MEAN FASTER

🔥🔥 **Excellent interviewer question.**

> If I make everything asynchronous, will my application become faster?

No.

Suppose:

```text
Payment → Fraud
```

takes 500 ms.

Making the Java call asynchronous doesn't magically make Fraud respond in 100 ms.

It can improve:

* thread utilization
* ability to handle concurrent work
* responsiveness
* composition of independent tasks

But it doesn't necessarily reduce the underlying dependency latency.

---

# SECTION 15 — PARALLEL vs SEQUENTIAL

Suppose you need:

```text
Merchant information
Customer information
Fraud information
```

and they don't depend on each other.

Sequential:

```text
Merchant = 200ms
Customer = 200ms
Fraud = 300ms

Total ≈ 700ms
```

You might run them concurrently:

```java
CompletableFuture<Merchant> merchant =
        getMerchant();

CompletableFuture<Customer> customer =
        getCustomer();

CompletableFuture<FraudResult> fraud =
        checkFraud();

CompletableFuture.allOf(
        merchant,
        customer,
        fraud
).join();
```

Potential latency approaches the slowest operation rather than the sum, assuming the operations truly run independently and resources aren't the bottleneck.

---

# SECTION 16 — IMPORTANT PAYMENT QUESTION

## Q33. Would you process two debit requests concurrently?

Suppose wallet balance is:

```text
₹3000
```

Two requests arrive:

```text
Request A → debit ₹2000
Request B → debit ₹2000
```

Naive code:

```java
if (balance >= amount) {
    balance -= amount;
}
```

Both threads may observe:

```text
₹3000
```

and both approve.

Now you have overspending.

### Don't solve this merely with:

```java
synchronized
```

because your service might have:

```text
Pod 1
Pod 2
Pod 3
```

Instead, use database-level concurrency control.

For example:

```sql
UPDATE wallet
SET balance = balance - :amount
WHERE wallet_id = :walletId
AND balance >= :amount;
```

Then check:

```text
affected rows == 1
```

means debit succeeded.

```text
affected rows == 0
```

means insufficient balance / concurrent state prevented the debit.

This is one of the most important connections between **Java concurrency and distributed backend design**.

---

# SECTION 17 — OPTIMISTIC LOCKING

Another approach:

```java
@Entity
public class Wallet {

    @Id
    private Long id;

    private BigDecimal balance;

    @Version
    private Long version;
}
```

Suppose:

```text
Version = 5
```

Two transactions read version 5.

Transaction A updates:

```text
5 → 6
```

Transaction B still tries to update version 5.

It fails because the version has changed.

That's **optimistic locking**.

---

# SECTION 18 — PESSIMISTIC LOCKING

You can instead acquire a database lock.

Conceptually:

```sql
SELECT *
FROM wallet
WHERE wallet_id = ?
FOR UPDATE;
```

Then another transaction attempting to lock the same row may have to wait.

Use carefully because excessive locking can reduce concurrency and cause contention/deadlocks.

---

# SECTION 19 — THREAD STARVATION

## Q34. What is thread starvation?

A thread cannot get enough execution time/resources because other threads monopolize available resources.

Example:

```text
Thread Pool = 10
```

Suppose all 10 threads are blocked waiting for a slow downstream service.

Now a request that needs those threads can't execute.

This is one reason:

* timeouts
* bounded pools
* bulkheads
* asynchronous designs

matter.

---

# SECTION 20 — THREAD POOL EXHAUSTION

Very important microservices scenario.

```text
Payment Service
Thread pool = 20
```

Every request calls:

```text
Payment → Fraud Service
```

Fraud becomes slow.

Now:

```text
20 threads
 ↓
all waiting on Fraud
 ↓
new requests wait
 ↓
latency increases
 ↓
timeouts
```

If clients retry:

```text
timeouts
   ↓
retries
   ↓
more requests
   ↓
more threads blocked
   ↓
service collapses
```

This is why **timeouts + bounded resources + circuit breakers** work together.

---

# SECTION 21 — `synchronized` METHOD VS BLOCK

### Method

```java
public synchronized void process() {
}
```

locks the object's monitor.

Equivalent conceptually to synchronizing on `this`.

### Block

```java
public void process() {

    synchronized (lock) {
        criticalSection();
    }
}
```

allows you to:

* lock only a small critical section
* use a specific lock object
* reduce contention

Generally, smaller critical sections are preferable when synchronization is actually necessary.

---

# SECTION 22 — `synchronized` STATIC METHOD

🔥 Potential interview trap.

```java
public static synchronized void process() {
}
```

The lock is associated with the **Class object**, not an individual instance.

So:

```text
instance lock
```

and:

```text
class-level lock
```

are different concepts.

---

# SECTION 23 — ATOMICITY, VISIBILITY, ORDERING

You should know these three words.

### Atomicity

Operation appears indivisible.

Example:

```java
AtomicInteger.incrementAndGet()
```

### Visibility

One thread sees another thread's update.

Example:

```java
volatile
```

### Ordering

Operations happen with the ordering guarantees established by the Java Memory Model/happens-before relationships.

`synchronized`, `volatile`, thread start/join, etc. establish important happens-before relationships.

---

# SECTION 24 — `join()`

## Q35. What does `join()` do?

Suppose:

```java
Thread t = new Thread(() -> process());

t.start();

t.join();

System.out.println("Done");
```

The current thread waits until `t` terminates.

Example:

```text
Main
 |
 +--> Worker
 |
 | waits
 ↓
Worker completes
 |
 ↓
Main continues
```

---

# SECTION 25 — INTERRUPTIONS

🔥 Often missed.

What is:

```java
thread.interrupt();
```

It doesn't forcibly kill the thread.

It is a cooperative interruption signal.

A blocking operation such as `sleep()` may throw:

```java
InterruptedException
```

Good code generally preserves the interruption status if it cannot handle the interruption fully:

```java
catch (InterruptedException e) {
    Thread.currentThread().interrupt();
    // handle/propagate appropriately
}
```

---

# SECTION 26 — INTERVIEW SCENARIO

### Q36.

> You have a Java microservice receiving 1000 requests/sec. Each request makes a synchronous call to another service that takes 2 seconds. What problems can occur?

Think:

```text
1000 requests/sec
       ↓
Many concurrent requests
       ↓
Threads occupied
       ↓
Connection pool pressure
       ↓
Memory pressure
       ↓
Latency
       ↓
Timeouts
```

Then ask:

> How would you improve it?

Possible considerations:

* connection pooling
* bounded thread pools
* timeout
* circuit breaker
* bulkhead
* caching where valid
* async processing where business semantics allow
* event-driven processing
* batching
* downstream scaling
* backpressure/rate limiting

Don't automatically say:

> "Use CompletableFuture."

You first identify the bottleneck.

---

# SECTION 27 — VERY IMPORTANT: ASYNC PAYMENT

### Q37.

> The client calls payment API. Should your API wait for every downstream operation before returning?

Not necessarily.

For example:

```text
POST /payments
```

could return:

```http
202 Accepted
```

with:

```json
{
  "paymentId": "P100",
  "status": "PROCESSING"
}
```

Then:

```text
Payment API
     |
     v
Persist payment
     |
     v
Publish event
     |
     v
Async processing
     |
     +--> Fraud
     +--> PSP
     +--> Notification
```

But whether this is appropriate depends on the payment contract and consistency requirements.

---

# SECTION 28 — THE INTERVIEWER MAY TRY THIS

> If you use asynchronous processing, how do you make sure the request doesn't lose the payment?

This is where you need:

```text
Database transaction
+
Durable messaging
+
Transactional Outbox
```

Example:

```text
BEGIN TRANSACTION

INSERT payment
INSERT outbox_event

COMMIT
```

Then:

```text
Outbox Publisher
      |
      v
Kafka
      |
      v
Consumers
```

So the database state and event intent are persisted atomically.

We'll cover this deeply under **Kafka/distributed transactions**.

---

# SECTION 29 — QUESTIONS YOU SHOULD WRITE DOWN

For **Multithreading & Concurrency**, write these:

### Fundamentals

1. What is a thread?
2. `start()` vs `run()`?
3. Can you start the same Thread twice?
4. What are Java thread states?
5. Why shouldn't we create thousands of threads manually?
6. What is ExecutorService?
7. `execute()` vs `submit()`?
8. What is a thread pool?
9. What happens if the thread pool is exhausted?

### Synchronization

10. What is a race condition? Give a real example.
11. How does `synchronized` work?
12. What does `synchronized` guarantee?
13. `synchronized` method vs synchronized block?
14. Why is `synchronized` insufficient in a multi-pod microservice?
15. `synchronized` static method vs instance method?

### Java Memory Model

16. What is `volatile`?
17. Does volatile make `count++` thread-safe?
18. Atomicity vs visibility?
19. What is happens-before?
20. Why can one thread fail to immediately observe another thread's update?

### Atomic/concurrent

21. `AtomicInteger` vs `volatile int`?
22. How does `compareAndSet()` work?
23. Why is `ConcurrentHashMap` safer than HashMap?
24. Is `ConcurrentHashMap.get() + put()` atomic?
25. When would you use `computeIfAbsent()`?

### Locks

26. `synchronized` vs `ReentrantLock`?
27. What is `tryLock()`?
28. What is deadlock?
29. How do you prevent deadlock?
30. What is starvation?
31. What is livelock?

### Thread coordination

32. `wait()` vs `sleep()`?
33. `notify()` vs `notifyAll()`?
34. Why should `wait()` generally be inside a `while` condition?
35. What does `join()` do?
36. What does `interrupt()` do?
37. `CountDownLatch` vs `CyclicBarrier`?

### CompletableFuture

38. What is CompletableFuture?
39. `thenApply()` vs `thenCompose()`?
40. `thenAccept()` vs `thenRun()`?
41. `exceptionally()` vs `handle()` vs `whenComplete()`?
42. Why provide a custom Executor to CompletableFuture?
43. What happens if you use the common pool for blocking I/O?
44. How do you run multiple independent operations concurrently?
45. How do you handle timeout in CompletableFuture?

### Backend scenarios

46. Two requests debit the same wallet simultaneously. How do you prevent double spending?
47. Why doesn't `synchronized` solve the wallet problem in a Kubernetes deployment?
48. How would you design a high-TPS wallet debit?
49. A downstream service becomes slow and all your threads are blocked. What happens?
50. How do timeout, retry, circuit breaker and bulkhead work together?
51. When would you choose synchronous vs asynchronous processing?
52. Why doesn't making code asynchronous automatically make it faster?
53. How would you process 1 million independent records concurrently without creating 1 million threads?

---

# ⭐ The 8 questions I especially want you to master

If the interviewer is strong, these are the ones I'd expect them to use to distinguish someone who **knows Java** from someone who has only used Java/Spring:

> **1. `volatile` vs AtomicInteger vs synchronized — when would you use each?**

> **2. Why is `ConcurrentHashMap` thread-safe but `get() + put()` still potentially unsafe?**

> **3. Why does synchronized fail to protect a shared wallet when you have multiple service pods?**

> **4. How does thread-pool exhaustion happen in a microservice?**

> **5. What happens if all threads are blocked waiting for a downstream service?**

> **6. Why should you be careful using CompletableFuture with blocking I/O?**

> **7. How would you safely process two concurrent debit requests?**

> **8. Explain how timeout + retry + circuit breaker + bulkhead prevent cascading failure.**

These connect **Core Java → Spring → Microservices → Payments**, which is exactly the direction your interview can take.

---

## Next: Spring Core

Then we'll start **Topic 3 — Spring Core**, from:

```text
IoC
 ↓
Dependency Injection
 ↓
Bean creation
 ↓
@Component / @Service / @Repository
 ↓
@Bean
 ↓
@Configuration
 ↓
@ComponentScan
 ↓
AutoConfiguration
 ↓
Bean scopes
 ↓
Bean lifecycle
 ↓
Spring proxies
 ↓
AOP
 ↓
@Transactional
 ↓
self-invocation
 ↓
transaction propagation
 ↓
transaction boundaries
```

And I'll include the **"why does Spring actually do this?"** questions and code examples rather than just annotations.

