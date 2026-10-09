# ExecutorService — EPAM Interview Level 🔥

Now we move from **"what is concurrency?"** to **"how do I actually manage threads in a Java application?"**

The core problem ExecutorService solves is:

> **Don't create/manage threads manually. Give tasks to a controlled pool of reusable worker threads.**

---

# 1. Problem: I have 1,000 tasks. Should I create 1,000 threads?

Suppose:

```java
for (int i = 0; i < 1000; i++) {
    new Thread(() -> process()).start();
}
```

Technically possible.

But it's a bad production design.

### Why?

Every thread has overhead:

```text
Thread creation
     ↓
Memory allocation
     ↓
Scheduling
     ↓
Context switching
```

And now you have potentially:

```text
1000 threads
     ↓
CPU contention
     ↓
memory pressure
     ↓
context switching
     ↓
poor performance
```

### Answer

Use a thread pool:

```text
1000 Tasks
    ↓
ExecutorService
    ↓
Queue
    ↓
10 Worker Threads
```

The workers are reused.

---

# 2. What is ExecutorService?

`ExecutorService` is an abstraction for **submitting and managing asynchronous tasks using a pool of threads**.

Example:

```java
ExecutorService executor =
        Executors.newFixedThreadPool(5);

executor.submit(() -> processPayment());
```

Conceptually:

```text
                ExecutorService
                      |
              ┌───────┴───────┐
              ↓               ↓
            Queue          Workers
                            ↓ ↓ ↓
                           T1 T2 T3
```

You submit tasks.

The executor decides when a worker executes them.

---

# 3. Problem: What's the difference between `execute()` and `submit()`?

Very common EPAM question.

### `execute()`

```java
executor.execute(() -> process());
```

Accepts a `Runnable`.

It doesn't return a result.

### `submit()`

```java
Future<?> future =
        executor.submit(() -> process());
```

Can accept:

```text
Runnable
Callable
```

and returns:

```text
Future
```

### Important difference

`submit()` gives you a mechanism to retrieve:

- result
- exception
- completion state

while `execute()` is simply fire-and-execute.

---

# 4. Problem: My task needs to return a result

Suppose:

```java
int calculatePayment() {
    return 100;
}
```

You cannot use `Runnable` for a return value.

Use `Callable<T>`:

```java
Callable<Integer> task = () -> {
    return 100;
};
```

Then:

```java
Future<Integer> future =
        executor.submit(task);
```

Later:

```java
Integer result = future.get();
```

---

# 5. What is `Future`?

A `Future` represents the **result of an asynchronous computation**.

Think:

```text
submit task
     ↓
Future
     ↓
task executing...
     ↓
result available
```

Example:

```java
Future<Integer> future =
        executor.submit(() -> {
            Thread.sleep(1000);
            return 100;
        });

System.out.println("Doing other work...");

Integer result = future.get();
```

`future.get()` waits if the result isn't ready.

---

# 6. Problem: Isn't `future.get()` blocking?

**Yes.**

This is extremely important.

Suppose:

```java
Future<Integer> future =
        executor.submit(() -> slowOperation());

Integer result = future.get();
```

The calling thread waits.

So:

```text
Main/request thread
       ↓
future.get()
       ↓
WAIT
       ↓
worker finishes
       ↓
result
```

Therefore, simply using `ExecutorService` doesn't automatically make your request non-blocking.

🔥 Interview answer:

> `submit()` is asynchronous, but `Future.get()` is blocking. If the caller immediately calls `get()`, it can still block the calling thread until the task completes.

---

# 7. Problem: What if I don't want to wait forever?

Use:

```java
future.get(2, TimeUnit.SECONDS);
```

Now:

```text
Task finishes < 2 sec → result
Task takes > 2 sec   → TimeoutException
```

This is important for microservices.

For example:

```text
Payment Service
      ↓
Fraud Service
      ↓
timeout
```

You don't want a request thread waiting indefinitely.

---

# 8. Problem: What happens if the Callable throws an exception?

Suppose:

```java
Future<Integer> future =
    executor.submit(() -> {
        throw new RuntimeException("DB failed");
    });
```

The exception doesn't simply get thrown on the submitting thread immediately.

It is associated with the task's `Future`.

When you call:

```java
future.get();
```

you get:

```text
ExecutionException
       ↓
cause
       ↓
RuntimeException("DB failed")
```

So:

```java
try {
    future.get();
} catch (ExecutionException e) {
    Throwable cause = e.getCause();
}
```

### Interview point

> Exceptions thrown by a task submitted through `submit()` are captured and become available through the Future; `get()` typically throws `ExecutionException` wrapping the original cause.

---

# 9. Problem: `execute()` vs `submit()` exception behavior

This is a nice interviewer trap.

With:

```java
executor.execute(() -> {
    throw new RuntimeException("failed");
});
```

there is no `Future` to retrieve the exception from.

The exception is handled through the thread's uncaught-exception mechanism.

With:

```java
executor.submit(() -> {
    throw new RuntimeException("failed");
});
```

the exception is captured in the returned `Future`.

So don't say:

> "submit throws the exception."

More accurately:

> The task's exception is captured by the Future and observed through `get()` as an `ExecutionException`.

---

# 10. Problem: What happens when all threads are busy?

Suppose:

```java
ExecutorService executor =
    Executors.newFixedThreadPool(3);
```

You submit:

```text
Task 1 → Worker 1
Task 2 → Worker 2
Task 3 → Worker 3
Task 4 → ?
Task 5 → ?
```

The additional tasks wait in the executor's queue.

Conceptually:

```text
             Executor
                 |
       ┌─────────┴─────────┐
       ↓                   ↓
   Worker Threads         Queue
    T1 T2 T3             T4 T5 T6...
```

This is one reason thread pools are useful: they **control concurrency**.

---

# 11. Problem: What happens if the queue becomes full?

Now we're getting into `ThreadPoolExecutor`.

Its important components are:

```text
corePoolSize
maximumPoolSize
workQueue
keepAliveTime
RejectedExecutionHandler
```

Example:

```java
new ThreadPoolExecutor(
    5,
    10,
    60,
    TimeUnit.SECONDS,
    new ArrayBlockingQueue<>(100)
);
```

Interpretation:

```text
Core threads       = 5
Maximum threads    = 10
Queue capacity     = 100
Idle timeout       = 60 sec
```

---

# 12. How does ThreadPoolExecutor decide what to do?

This is **very interview-worthy**.

Suppose:

```text
corePoolSize = 5
maxPoolSize  = 10
queue        = 100
```

When tasks arrive:

### Step 1

If fewer than 5 workers exist:

```text
Create core worker
```

### Step 2

Once 5 core workers exist:

```text
Put new tasks into queue
```

### Step 3

If queue becomes full:

```text
Create additional workers
```

up to:

```text
maxPoolSize = 10
```

### Step 4

If:

```text
10 workers busy
+
queue full
```

then:

```text
RejectedExecutionHandler
```

comes into play.

This is a **very strong Senior Engineer topic** because it connects directly to production load handling.

---

# 13. Problem: Why is an unbounded queue dangerous?

Suppose your application receives tasks faster than workers can process them.

```text
Incoming tasks
      ↓↓↓↓↓↓↓↓↓
    Queue
    1000
    5000
    50000
    500000
```

Eventually:

```text
memory pressure
      ↓
OOM risk
```

So queue capacity is not just a configuration detail.

It's part of **backpressure/load management**.

---

# 14. Problem: What is `RejectedExecutionHandler`?

When the executor cannot accept a task, it needs a rejection policy.

Common policies:

```text
AbortPolicy
CallerRunsPolicy
DiscardPolicy
DiscardOldestPolicy
```

### AbortPolicy

Throws:

```text
RejectedExecutionException
```

This is the default for `ThreadPoolExecutor`.

### CallerRunsPolicy

The submitting thread executes the task itself.

This can naturally slow down the producer.

Example:

```text
Executor overloaded
       ↓
caller executes task
       ↓
caller becomes slower
       ↓
fewer tasks submitted
```

This can provide a form of backpressure.

### DiscardPolicy

Silently discards the task.

Dangerous if the task is important.

### DiscardOldestPolicy

Removes the oldest queued task and attempts to submit the new one.

---

# 15. Problem: How many threads should I create?

There isn't one magic number.

It depends on workload.

### CPU-bound

Examples:

```text
calculations
compression
CPU-heavy transformations
```

Usually keep thread count relatively close to available CPU cores.

### I/O-bound

Examples:

```text
DB calls
HTTP calls
file/network I/O
```

Can often benefit from more threads because threads spend time waiting.

But don't blindly say:

> "CPU cores × 2."

That's only a starting heuristic.

### Senior answer

> Thread-pool sizing depends on workload characteristics, CPU availability, task latency, downstream capacity, and acceptable concurrency. For I/O-heavy work I may use more workers than CPU cores, but I would validate the configuration using metrics and load testing.

That's a much better answer.

---

# 16. Problem: Should I use `Executors.newFixedThreadPool()` in production?

You can, but understand what it creates.

For example:

```java
Executors.newFixedThreadPool(10);
```

uses a fixed number of workers and an effectively unbounded queue.

For production systems where you need **explicit capacity control**, it's often preferable to configure `ThreadPoolExecutor` directly with:

```text
bounded queue
+
max threads
+
rejection policy
```

This gives you explicit backpressure behavior.

---

# 17. Problem: What happens when I forget to shut down ExecutorService?

Suppose:

```java
ExecutorService executor =
    Executors.newFixedThreadPool(10);
```

and never shut it down.

Those worker threads can keep the executor alive.

You should manage its lifecycle:

```java
executor.shutdown();
```

If necessary:

```java
executor.shutdownNow();
```

Difference:

### `shutdown()`

Stops accepting new tasks and allows already submitted tasks to complete.

### `shutdownNow()`

Attempts to interrupt running tasks and returns tasks that haven't started.

Important:

> `shutdownNow()` does not magically kill threads. It uses interruption.

---

# 18. Microservice scenario — this is where it becomes real

Imagine your Payment Service:

```text
POST /payment
       ↓
PaymentController
       ↓
PaymentService
       ↓
ExecutorService
       ↓
Fraud check
```

You configure:

```text
10 worker threads
queue = 100
```

Suddenly traffic spikes.

```text
Incoming requests
       ↓
       ↓
10 workers busy
       ↓
100 tasks queued
       ↓
next task rejected
```

Now you need to decide:

```text
What should happen?
```

Possible answers:

- reject quickly
- execute in caller thread
- return a controlled failure
- use retry where appropriate
- apply rate limiting/backpressure
- scale service horizontally
- increase capacity only if downstream can handle it

This is why concurrency isn't just a Java topic.

It's a **system design concern**.

---

# 19. Spring connection

In Spring Boot, instead of manually doing:

```java
Executors.newFixedThreadPool(...)
```

you'll often configure a Spring-managed executor:

```java
@Bean
public Executor paymentExecutor() {
    ThreadPoolTaskExecutor executor =
        new ThreadPoolTaskExecutor();

    executor.setCorePoolSize(8);
    executor.setMaxPoolSize(10);
    executor.setQueueCapacity(20);

    executor.initialize();

    return executor;
}
```

Then asynchronous work can use that executor.

This leads to:

```text
Java ExecutorService
        ↓
Spring TaskExecutor
        ↓
@Async
```

We'll cover this properly after `Future` and `CompletableFuture`.

---

# 20. EPAM-style scenario

### Interviewer:

> "Your service receives 1000 requests and each request has a task that takes 2 seconds. Would you create a new thread for every request?"

### Strong answer:

> "No. I would use a bounded thread pool so that concurrency is controlled. The executor would have a configured number of worker threads and a bounded queue. If the queue is exhausted, I'd define a rejection policy rather than allowing unlimited tasks to accumulate. The exact pool size would depend on whether the workload is CPU-bound or I/O-bound and also on downstream capacity such as DB connection pool size."

🔥 That's the level you want.

---

# 21. One very important production relationship

Don't configure this independently:

```text
Executor = 100 threads
Database pool = 10 connections
```

and assume everything will be faster.

You could end up with:

```text
100 application threads
       ↓
10 DB connections
       ↓
90 threads waiting
```

You've just created contention.

So think:

```text
HTTP thread pool
       ↓
Async executor
       ↓
DB connection pool
       ↓
Database
```

All these concurrency limits need to be considered together.

---

# What you should remember

```text
ExecutorService
     ↓
manages worker threads

execute()
     ↓
Runnable, no Future

submit()
     ↓
Runnable/Callable
     ↓
Future

Future.get()
     ↓
BLOCKING

ThreadPoolExecutor
     ↓
corePoolSize
maxPoolSize
queue
rejection policy
keepAlive

Production
     ↓
bounded concurrency
backpressure
downstream capacity
graceful shutdown
```

## Next: `Future` → `CompletableFuture` 🔥

This is where we'll answer the really common questions:

> **Why was `Future` introduced?**

> **What are the limitations of Future?**

> **Why do we need CompletableFuture?**

> **Difference between `thenApply`, `thenCompose`, and `thenCombine`?**

> **What happens when a CompletableFuture throws an exception?**

> **`exceptionally` vs `handle` vs `whenComplete`?**

> **How do you run two downstream calls concurrently and combine their results?**

And then we'll connect that directly to your **Spring Boot Payment microservice + custom executor** experience.