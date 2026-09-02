# Java Concurrency — 08: ExecutorService & Thread Pools

> Thread pools are one of the most important practical parts of Java concurrency for backend development. Instead of creating a new thread for every task, we reuse a controlled number of worker threads.

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
      ↓
ExecutorService & Thread Pools
```

---

# 2. Why Not `new Thread()` for Every Task?

A naïve implementation could do:

```java
new Thread(() -> processRequest()).start();
```

for every request.

If 10,000 requests arrive:

```text
10,000 requests
      ↓
10,000 threads
      ↓
huge memory consumption
      ↓
context switching
      ↓
CPU contention
      ↓
application instability
```

Thread creation has a cost.

Instead:

```text
Tasks
 ↓
Thread Pool
 ↓
Reusable Worker Threads
```

Example:

```text
              Thread Pool
             /    |    \
            T1    T2    T3
```

Tasks are submitted to the pool and worker threads execute them.

---

# 3. What Is a Thread Pool?

A thread pool is a collection of reusable worker threads.

Instead of:

```text
Task 1 → create Thread → execute → destroy
Task 2 → create Thread → execute → destroy
Task 3 → create Thread → execute → destroy
```

we have:

```text
Create workers once
       ↓
reuse workers
       ↓
Task 1
Task 2
Task 3
Task 4
...
```

Benefits:

```text
thread reuse
resource control
bounded concurrency
better throughput
backpressure
lifecycle management
```

---

# 4. Executor

Java provides the `Executor` abstraction:

```java
Executor
```

It has a simple method:

```java
void execute(Runnable command);
```

Its purpose is essentially:

> Execute this task.

Example:

```java
Executor executor = ...;

executor.execute(() -> {
    System.out.println("Task running");
});
```

The caller doesn't have to manually create a thread.

---

# 5. ExecutorService

`ExecutorService` extends `Executor`.

It adds task-management and lifecycle functionality.

Important methods include:

```text
execute()
submit()
shutdown()
shutdownNow()
awaitTermination()
```

Mental model:

```text
Executor
   ↓
execute tasks

ExecutorService
   ↓
execute tasks
   ↓
submit tasks
   ↓
manage executor lifecycle
   ↓
Future
   ↓
shutdown
```

---

# 6. Fixed Thread Pool

Example:

```java
ExecutorService executor =
        Executors.newFixedThreadPool(3);
```

Conceptually:

```text
                ExecutorService
                      |
              Thread Pool = 3
             /      |      \
            T1      T2      T3
```

There are three worker threads available.

---

# 7. Ten Tasks on a Pool of Three

Suppose:

```java
ExecutorService executor =
        Executors.newFixedThreadPool(3);
```

and we submit 10 tasks.

Do all 10 execute simultaneously?

**No.**

Only up to 3 tasks can actively execute at the same time.

Conceptually:

```text
              Thread Pool
          ┌────┬────┬────┐
          │ T1 │ T2 │ T3 │
          └────┴────┴────┘
             ↓    ↓    ↓
          executing

Waiting tasks:

T4
T5
T6
T7
T8
T9
T10
 ↓
queue
```

When T1 finishes:

```text
T1 finishes
     ↓
worker becomes free
     ↓
take next task from queue
     ↓
T4 executes
```

The worker thread is reused.

---

# 8. Practical Example

```java
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

public class Main {

    public static void main(String[] args) {

        ExecutorService executor =
                Executors.newFixedThreadPool(3);

        for (int i = 1; i <= 10; i++) {

            int taskId = i;

            executor.execute(() -> {

                System.out.println(
                    "Task " + taskId +
                    " executed by " +
                    Thread.currentThread().getName()
                );
            });
        }

        executor.shutdown();
    }
}
```

You may see output such as:

```text
Task 1 → pool-1-thread-1
Task 2 → pool-1-thread-2
Task 3 → pool-1-thread-3
Task 4 → pool-1-thread-1
Task 5 → pool-1-thread-2
...
```

The important observation:

> The same worker threads are reused.

---

# 9. `execute()` vs `submit()`

This is an important interview question.

## `execute()`

```java
executor.execute(() -> {
    System.out.println("Hello");
});
```

Accepts:

```text
Runnable
```

It does not return a result.

---

## `submit()`

```java
Future<?> future =
        executor.submit(() -> {
            System.out.println("Hello");
        });
```

Returns:

```text
Future
```

The `Future` allows you to track the submitted task.

---

# 10. `Callable`

`Callable<T>` is useful when a task needs to return a result.

Example:

```java
Callable<Integer> task = () -> {

    return 10 + 20;
};
```

Submit:

```java
Future<Integer> future =
        executor.submit(task);
```

Later:

```java
Integer result =
        future.get();
```

Result:

```text
30
```

Mental model:

```text
Runnable
   ↓
does work
   ↓
no result

Callable<T>
   ↓
does work
   ↓
returns T
   ↓
Future<T>
```

---

# 11. ThreadPoolExecutor

`Executors.newFixedThreadPool()` is a convenience factory.

Underneath, Java uses `ThreadPoolExecutor`.

You can create one directly:

```java
ThreadPoolExecutor executor =
        new ThreadPoolExecutor(
            3,
            5,
            60,
            TimeUnit.SECONDS,
            new LinkedBlockingQueue<>(100)
        );
```

Parameters:

```text
3
↓
corePoolSize

5
↓
maximumPoolSize

60 seconds
↓
keepAliveTime

LinkedBlockingQueue<>(100)
↓
work queue
```

---

# 12. Core Pool Size

Suppose:

```text
corePoolSize = 3
```

The executor maintains up to three core worker threads as needed.

Conceptually:

```text
Thread Pool

T1
T2
T3
```

These are the core workers.

---

# 13. Maximum Pool Size

Suppose:

```text
corePoolSize = 3
maximumPoolSize = 5
```

The pool can grow beyond the core size up to:

```text
5 workers
```

But:

> The pool does not immediately create five threads.

The work queue affects when additional threads are created.

---

# 14. The Critical ThreadPoolExecutor Algorithm

Suppose:

```text
corePoolSize = 3
maximumPoolSize = 5
queueCapacity = 2
```

When a task arrives:

### Step 1 — Core Workers

If:

```text
current workers < corePoolSize
```

the executor creates a worker.

So:

```text
Task 1 → Thread 1
Task 2 → Thread 2
Task 3 → Thread 3
```

Now:

```text
workers = 3
```

---

### Step 2 — Queue

Another task arrives.

The core pool is already full.

The executor tries to queue the task:

```text
T4 → queue
T5 → queue
```

Now:

```text
workers = 3
queue = 2
```

---

### Step 3 — Queue Full

Another task arrives.

The queue is full.

If:

```text
workers < maximumPoolSize
```

the executor can create another worker.

```text
T6 → Thread 4
```

Now:

```text
workers = 4
queue = 2
```

---

### Step 4 — Grow Again

Another task:

```text
T7 → Thread 5
```

Now:

```text
workers = 5
queue = 2
```

Maximum reached.

---

### Step 5 — Saturation

Another task arrives:

```text
T8
```

Now:

```text
workers = 5
queue = full
maximum reached
```

Therefore:

```text
REJECT
```

The `RejectedExecutionHandler` determines what happens.

---

# 15. Core Execution Flow

For:

```text
core = 3
max = 5
queue = 2
```

The mental model is:

```text
Task arrives
      ↓
workers < 3?
   YES ↓
create worker
      ↓
execute task

If workers == 3
      ↓
try queue
      ↓
queue has space?
   YES ↓
put task in queue

If queue is full
      ↓
workers < 5?
   YES ↓
create additional worker

If workers == 5
and queue full
      ↓
REJECT TASK
```

This is one of the most important `ThreadPoolExecutor` concepts.

---

# 16. Why This Matters in Spring Boot

Suppose you configure:

```text
corePoolSize = 8
maxPoolSize = 10
queueCapacity = 20
```

Conceptually:

```text
8 core workers
     ↓
20 tasks can wait in queue
     ↓
pool can grow up to 10 workers
     ↓
10 workers + full queue
     ↓
rejection policy
```

This is directly relevant when configuring asynchronous processing or Spring Batch workers.

---

# 17. `Executors.newFixedThreadPool()`

When you write:

```java
Executors.newFixedThreadPool(3);
```

you are using a convenience factory.

Conceptually:

```text
Executors
   ↓
factory
   ↓
ThreadPoolExecutor
```

So `Executors` is not a completely different threading mechanism.

---

# 18. Why Explicit `ThreadPoolExecutor` Can Be Useful

You can control:

```text
corePoolSize
maximumPoolSize
keepAliveTime
work queue
thread factory
rejection policy
```

Example:

```java
ThreadPoolExecutor executor =
        new ThreadPoolExecutor(
            3,
            5,
            60,
            TimeUnit.SECONDS,
            new ArrayBlockingQueue<>(100),
            new ThreadPoolExecutor.CallerRunsPolicy()
        );
```

This provides much more control over production behavior.

---

# 19. Rejection Policies

When the executor cannot accept another task, a `RejectedExecutionHandler` decides what happens.

Java provides common policies.

---

# 20. AbortPolicy

Default policy:

```java
new ThreadPoolExecutor.AbortPolicy()
```

It throws:

```text
RejectedExecutionException
```

Flow:

```text
task
 ↓
pool full
 ↓
RejectedExecutionException
```

---

# 21. CallerRunsPolicy

```java
new ThreadPoolExecutor.CallerRunsPolicy()
```

Instead of immediately rejecting:

```text
pool full
   ↓
caller executes task
```

For example:

```text
Request Thread
      ↓
submit task
      ↓
pool full
      ↓
Request Thread executes task itself
```

This can provide a form of backpressure.

But it also means the caller may perform expensive work itself.

---

# 22. DiscardPolicy

```java
new ThreadPoolExecutor.DiscardPolicy()
```

The task is silently discarded.

```text
pool full
   ↓
task discarded
```

This can be dangerous for business-critical tasks.

For financial processing, silently dropping a task would generally require very careful consideration.

---

# 23. DiscardOldestPolicy

```java
new ThreadPoolExecutor.DiscardOldestPolicy()
```

Conceptually:

```text
queue full
   ↓
discard oldest queued task
   ↓
try submitting new task
```

Again, this may be inappropriate for important financial workflows.

---

# 24. Rejection Policy Interview Question

### Question

> What happens when a `ThreadPoolExecutor` is saturated?

Strong answer:

> "When the core and maximum worker limits are reached and the work queue cannot accept another task, the executor invokes its `RejectedExecutionHandler`. The default `AbortPolicy` throws `RejectedExecutionException`, while alternatives such as `CallerRunsPolicy` can provide backpressure or different fallback behavior."

---

# 25. `shutdown()`

After using an executor:

```java
executor.shutdown();
```

This means:

> Stop accepting new tasks, but allow already submitted tasks to complete.

Conceptually:

```text
shutdown()
   ↓
no new tasks
   ↓
existing tasks continue
   ↓
workers eventually terminate
```

---

# 26. `shutdownNow()`

```java
executor.shutdownNow();
```

This attempts to stop the executor by:

```text
interrupting worker threads
+
returning tasks that never started
```

Important:

> `shutdownNow()` does not guarantee that currently running tasks immediately stop.

Tasks need to respond appropriately to interruption.

---

# 27. `shutdown()` vs `shutdownNow()`

```text
shutdown()
→ graceful shutdown
→ don't accept new tasks
→ finish submitted tasks

shutdownNow()
→ attempt immediate shutdown
→ interrupt running workers
→ return queued tasks
```

Strong interview answer:

> "`shutdown()` is generally used for graceful shutdown, while `shutdownNow()` attempts to interrupt running tasks and returns tasks that were waiting in the queue."

---

# 28. `awaitTermination()`

After:

```java
executor.shutdown();
```

you can wait:

```java
executor.awaitTermination(
    30,
    TimeUnit.SECONDS
);
```

This means:

```text
shutdown
   ↓
wait up to 30 seconds
   ↓
did workers terminate?
```

Typical pattern:

```java
executor.shutdown();

try {

    if (!executor.awaitTermination(
            30,
            TimeUnit.SECONDS)) {

        executor.shutdownNow();
    }

} catch (InterruptedException e) {

    executor.shutdownNow();

    Thread.currentThread().interrupt();
}
```

---

# 29. Why Thread Pools Are Important

Without pooling:

```text
Task
 ↓
create thread
 ↓
execute
 ↓
destroy
```

Repeated thousands of times.

With a pool:

```text
Create workers once
      ↓
reuse workers
      ↓
Task 1
Task 2
Task 3
Task 4
...
```

Benefits:

```text
thread reuse
resource control
bounded concurrency
better throughput
backpressure
lifecycle management
```

---

# 30. Thread Pools Are Not Automatically Good

A badly configured pool can cause:

```text
too many threads
      ↓
CPU contention
      ↓
context switching
      ↓
memory pressure
```

Or:

```text
queue too large
      ↓
tasks wait too long
      ↓
latency increases
      ↓
memory usage increases
```

Or:

```text
queue too small
      ↓
rejections
```

Thread-pool sizing is therefore an architectural decision.

---

# 31. CPU-Bound vs I/O-Bound

## CPU-Bound Work

Examples:

```text
complex calculations
compression
encryption
large in-memory transformations
```

CPU is the primary bottleneck.

You generally don't want an unnecessarily huge number of threads.

---

## I/O-Bound Work

Examples:

```text
database calls
HTTP calls
file I/O
network calls
```

A worker can spend significant time waiting for I/O.

Therefore, more concurrency may be appropriate than the number of CPU cores, depending on workload and system constraints.

Important:

> There is no universal rule such as "threads = CPU cores × 2."

Measure the actual workload.

---

# 32. Your Spring Batch Example

You previously worked with:

```text
corePoolSize = 8
maxPoolSize = 10
queueCapacity = 20
```

Imagine 100 tasks arrive.

Conceptually:

```text
First 8
 ↓
8 workers

Next 20
 ↓
queue

Additional tasks
 ↓
workers can grow
 ↓
up to 10

Once:
10 workers
+
20 queued
 ↓
additional submissions
 ↓
RejectedExecutionHandler
```

Now the configuration becomes much easier to reason about.

---

# 33. Backpressure

This is particularly relevant to distributed systems.

Imagine:

```text
Producer
   ↓
100,000 tasks/sec

Consumer pool
   ↓
1,000 tasks/sec
```

If you allow unlimited queuing:

```text
queue
 ↓
1M
 ↓
10M
 ↓
memory pressure
```

Eventually the application can become unstable.

A bounded queue can force the system to say:

```text
"I cannot accept more work right now."
```

This is **backpressure**.

---

# 34. Connection to Kafka

Kafka:

```text
Producer
   ↓
Topic / Partition
   ↓
Consumer
```

Thread pool:

```text
Producer / submitter
   ↓
Work Queue
   ↓
Worker Threads
```

Both involve controlling how work flows through a system.

But they are fundamentally different:

```text
Kafka queue
→ durable distributed messaging

Executor queue
→ in-memory JVM task queue
```

The executor queue is not a replacement for Kafka.

---

# 35. Connection to Kubernetes

Suppose:

```text
Pod A
  ↓
ThreadPoolExecutor
  ↓
10 workers
```

and:

```text
Pod B
  ↓
ThreadPoolExecutor
  ↓
10 workers
```

You do NOT have:

```text
one global pool of 20
```

You have:

```text
Pod A → 10 local workers
Pod B → 10 local workers
```

Thread pools are JVM-local.

---

# 36. Practical Exercise

Write:

```java
ExecutorService executor =
        Executors.newFixedThreadPool(3);
```

Submit 10 tasks.

Each task should:

```java
Thread.sleep(2000);
```

and print:

```text
START task X
END task X
Thread name
```

Observe:

```text
3 tasks
→ execute

7 tasks
→ wait

when one finishes
→ next task starts
```

This is one of the best ways to understand thread pools.

---

# 37. Important Interview Trap

### Question

> If `corePoolSize = 8` and `maximumPoolSize = 20`, will 20 threads immediately be created?

Answer:

**No.**

The executor creates core workers as needed.

Once the core size is reached, additional tasks are normally queued first.

Only when the queue cannot accept more tasks does the executor create additional workers, up to the maximum.

---

# 38. Another Interview Trap

### Question

> If I have `maximumPoolSize = 20`, why not just use an unbounded queue?

Because an unbounded queue can prevent the pool from growing beyond the core size under the normal `ThreadPoolExecutor` algorithm.

For example:

```text
core = 8
max = 20
unbounded queue
```

After 8 workers are active:

```text
additional tasks
       ↓
queue
```

The queue keeps accepting tasks.

Therefore:

```text
workers may remain = 8
```

and the maximum of 20 may effectively never be reached.

---

# 39. Thread Pool Sizing Mental Model

Think about:

```text
                 Workload
                    ↓
            CPU-bound / I/O-bound
                    ↓
              Pool size
                    ↓
              Queue capacity
                    ↓
          Rejection / backpressure
```

Don't choose values blindly.

Consider:

```text
CPU
memory
task duration
I/O wait
arrival rate
service time
downstream capacity
latency requirements
```

---

# 40. MUFG-Level Scenario

### Question

> You have a Spring Boot service receiving requests that trigger database operations. Would you create a new Thread for every request?

Strong answer:

> "I would generally avoid creating an unbounded number of threads per request. I would use an appropriately sized executor or framework-managed thread pool, with a bounded queue and an appropriate rejection/backpressure strategy. The sizing would depend on whether the workload is CPU-bound or I/O-bound and on the capacity of downstream systems such as the database."

---

# 41. MUFG-Level Scenario

### Question

> Your executor has 8 core threads, 10 maximum threads and queue capacity 20. What happens when 50 tasks arrive?

Answer:

```text
First tasks
→ create up to 8 core workers

Next tasks
→ queue up to 20

When queue is full
→ create additional workers
→ up to 10 total

After:
10 workers + 20 queued
→ additional tasks are rejected
→ RejectedExecutionHandler decides behavior
```

The exact number executing/queued at a particular instant depends on task completion timing, but the capacity model is:

```text
10 active workers maximum
+
20 queued tasks
```

---

# 42. ExecutorService vs ThreadPoolExecutor

```text
ExecutorService
      ↓
interface
      ↓
defines executor lifecycle/task APIs

ThreadPoolExecutor
      ↓
concrete implementation
      ↓
configurable thread pool
```

Mental model:

```text
ExecutorService
       ↑
ThreadPoolExecutor
```

You typically program against the interface:

```java
ExecutorService executor = ...;
```

while using `ThreadPoolExecutor` when you need detailed configuration or inspection.

---

# 43. Next: Callable + Future

We've now understood:

```text
Executor
      ↓
ExecutorService
      ↓
Thread Pool
      ↓
ThreadPoolExecutor
      ↓
Core Pool
      ↓
Maximum Pool
      ↓
Queue
      ↓
RejectedExecutionHandler
      ↓
Shutdown
```

Now we need:

```text
Runnable
   ↓
does work
   ↓
no result

Callable<T>
   ↓
does work
   ↓
returns result
   ↓
Future<T>
```

Example:

```java
Callable<Integer> task = () -> {

    return 10 + 20;
};
```

Submit:

```java
Future<Integer> future =
        executor.submit(task);
```

Get:

```java
Integer result =
        future.get();
```

Result:

```text
30
```

This leads directly into an important concept:

> **What exactly happens when you call `future.get()`?**

That will lead us into:

```text
blocking
     ↓
Future limitations
     ↓
CompletableFuture
     ↓
non-blocking composition
     ↓
async programming
```

---

# 44. Interview Checkpoint

Before moving on, make sure you can explain these without memorizing:

1. Why do we use a thread pool?
2. What happens when 10 tasks are submitted to a pool of 3 workers?
3. What is the work queue?
4. `corePoolSize` vs `maximumPoolSize`?
5. When does the pool create threads beyond the core size?
6. What happens when both pool and queue are full?
7. What is `RejectedExecutionHandler`?
8. `shutdown()` vs `shutdownNow()`?
9. Why can an unbounded queue be dangerous?
10. Why is a thread pool JVM-local?
11. CPU-bound vs I/O-bound — why does it affect pool sizing?
12. Why can `CallerRunsPolicy` provide backpressure?
13. Why can an unbounded queue prevent growth toward `maximumPoolSize`?
14. Why is `ThreadPoolExecutor` important even if you normally use `ExecutorService`?
15. Why shouldn't we blindly choose thread-pool sizes?

---

# 45. Final Mental Model

```text
Task
 ↓
ExecutorService
 ↓
ThreadPoolExecutor
 ↓
Are core workers available?
 ↓
YES → create/use worker
 ↓
NO
 ↓
Try queue
 ↓
Queue available?
 ↓
YES → wait in queue
 ↓
NO
 ↓
Can create worker up to maximum?
 ↓
YES → create additional worker
 ↓
NO
 ↓
RejectedExecutionHandler
```

And remember:

```text
Executor
→ execute tasks

ExecutorService
→ execute + submit + lifecycle

ThreadPoolExecutor
→ configurable implementation

Runnable
→ task without result

Callable
→ task with result

Future
→ handle/track asynchronous result
```

Next topic:

# Callable + Future

The most important question will be:

> **If `submit()` is asynchronous, why does `future.get()` make the caller wait?**

That distinction is the bridge from basic thread pools to **real asynchronous programming with `CompletableFuture`**.
