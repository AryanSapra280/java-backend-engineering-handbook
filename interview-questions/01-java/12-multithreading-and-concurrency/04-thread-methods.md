# D. Thread Methods

Now we move to **Section D — Thread Methods**, questions **2382–2410**. This section is especially important because interviewers frequently test the differences between **`sleep()` vs `wait()` vs `join()`**, and then go deep into **`interrupt()` and interrupt status**. 

---

# 2382. What does `sleep()` do?

`Thread.sleep()` pauses the **currently executing thread** for at least the specified amount of time, subject to scheduler and system timing.

Example:

```java
Thread.sleep(2000);
```

The current thread enters:

```text
TIMED_WAITING
```

for approximately 2 seconds.

Example:

```java
System.out.println("Before");

Thread.sleep(2000);

System.out.println("After");
```

Conceptually:

```text
Thread
  ↓
Before
  ↓
sleep(2 sec)
  ↓
TIMED_WAITING
  ↓
approximately 2 sec
  ↓
RUNNABLE
  ↓
After
```

### Important

`sleep()`:

* Applies to the **current thread**
* Does **not release monitors**
* Causes `TIMED_WAITING`
* Can be interrupted

The method declares:

```java
throws InterruptedException
```

so you need to handle or propagate that exception.

---

# 2383. Does `Thread.sleep()` release a lock?

**No. This is extremely important.**

Suppose:

```java
synchronized (lock) {

    Thread.sleep(5000);

}
```

The thread sleeps, but it **continues to own the monitor**.

```text
Thread A
   ↓
acquires lock
   ↓
sleep(5 sec)
   ↓
still owns lock
   ↓
wakes up
   ↓
releases lock
```

Another thread trying:

```java
synchronized (lock)
```

will have to wait for the lock.

It can therefore enter `BLOCKED`.

### Compare with `wait()`

```text
sleep()
→ does NOT release monitor

wait()
→ releases monitor
```

This distinction is one of the most frequently asked Java concurrency questions.

---

# 2384. What happens when a thread calls `sleep()` while holding a monitor?

Consider:

```java
Object lock = new Object();

Thread t1 = new Thread(() -> {

    synchronized (lock) {

        System.out.println("T1 acquired lock");

        try {
            Thread.sleep(5000);
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        }
    }
});
```

While `t1` sleeps:

```text
T1
 ↓
owns monitor
 ↓
TIMED_WAITING
 ↓
still owns monitor
```

If `t2` executes:

```java
synchronized (lock) {
    // ...
}
```

then:

```text
T2
 ↓
tries monitor
 ↓
monitor owned by T1
 ↓
BLOCKED
```

### Important interview scenario

> "Thread A holds a synchronized lock and sleeps for 10 seconds. What happens to Thread B trying to acquire the same lock?"

Answer:

> Thread A enters `TIMED_WAITING` but retains the monitor. Thread B becomes `BLOCKED` waiting for that monitor.

---

# 2385. What is `join()`?

`join()` allows one thread to **wait for another thread to terminate**.

Example:

```java
Thread worker = new Thread(() -> {
    System.out.println("Doing work...");
});

worker.start();

worker.join();

System.out.println("Worker finished");
```

The thread calling:

```java
worker.join();
```

waits until `worker` terminates.

Conceptually:

```text
Main Thread
   |
   | start worker
   ↓
Worker Thread ─────→ does work ─────→ TERMINATED
   ↑
   |
   | main waits
   |
Main Thread
```

Once the worker finishes:

```text
Main
 ↓
continues
```

---

# 2386. Why would you use `join()`?

You use `join()` when the current thread must wait for another thread to finish before continuing.

### Example

Suppose:

```text
Thread 1 → Load configuration
Thread 2 → Load database data
Main     → Start application
```

You might need:

```text
Wait until Thread 1 completes
Wait until Thread 2 completes
Then continue
```

Using:

```java
t1.join();
t2.join();
```

provides that coordination.

### Another example

```java
Thread worker = new Thread(() -> {
    // expensive calculation
});

worker.start();

worker.join();

System.out.println("Result can now be used");
```

### Key idea

> `join()` establishes a dependency: **"I cannot continue until that thread terminates."**

---

# 2387. What happens when `threadA.join()` is called?

Suppose:

```java
Thread threadA = new Thread(task);

threadA.start();

threadA.join();
```

The **thread that calls `join()`** waits.

This is a subtle point.

If `main()` calls:

```java
threadA.join();
```

then:

```text
main
 ↓
WAITING
```

while:

```text
threadA
 ↓
RUNNABLE
```

When `threadA` terminates:

```text
threadA → TERMINATED
       ↓
main → RUNNABLE
```

Then `main` continues after `join()`.

### Important

`join()` does **not mean the target thread waits**.

The **calling thread waits for the target thread**.

---

# 2388. What is the difference between `join()` and `sleep()`?

| `sleep()`                                   | `join()`                                                           |
| ------------------------------------------- | ------------------------------------------------------------------ |
| Pauses current thread for a time            | Current thread waits for another thread                            |
| Time-based                                  | Completion-based                                                   |
| `TIMED_WAITING`                             | `WAITING` when no timeout                                          |
| Doesn't release monitor                     | `join()` itself doesn't acquire/release your application's monitor |
| Can be interrupted                          | Can be interrupted                                                 |
| Doesn't care about another thread finishing | Specifically waits for target thread                               |

Example:

```java
Thread.sleep(5000);
```

means:

> "Pause me for approximately 5 seconds."

Whereas:

```java
worker.join();
```

means:

> "Wait until `worker` finishes."

### Timed join

```java
worker.join(5000);
```

means:

> "Wait for `worker` for at most approximately 5 seconds."

That produces `TIMED_WAITING` while waiting.

---

# 2389. What happens if `join()` is called on the current thread?

This is a classic interview question.

Suppose:

```java
Thread.currentThread().join();
```

The current thread waits for **itself to terminate**.

But it cannot terminate while it is waiting for itself.

So this effectively causes the thread to wait indefinitely, barring interruption or other exceptional circumstances.

Conceptually:

```text
Thread A
   ↓
join(Thread A)
   ↓
waits for Thread A to terminate
   ↓
Thread A can't finish because it's waiting
```

This is effectively a self-wait/deadlock-like situation.

### Interview answer

> Calling `join()` on the current thread causes it to wait for itself to terminate, so it will not normally make progress unless it is interrupted.

---

# 2390. What happens when `interrupt()` is called?

This is one of the **most important Java concurrency concepts**.

Calling:

```java
thread.interrupt();
```

does **not forcibly kill the thread**.

Instead, it requests/interposes an **interruption signal** on that thread.

What happens next depends on what the target thread is doing.

There are two major cases.

### Case 1 — Thread is in an interruptible blocking operation

For example:

```java
Thread.sleep(...)
```

or:

```java
wait()
```

or:

```java
join()
```

The operation can throw:

```text
InterruptedException
```

and the interrupt status is cleared when that exception is thrown.

---

### Case 2 — Thread is doing normal computation

If the thread isn't currently in an interruptible blocking operation, `interrupt()` generally sets its interrupt status.

The thread must cooperate by checking the status or otherwise responding appropriately.

Example:

```java
while (!Thread.currentThread().isInterrupted()) {
    doWork();
}
```

### Mental model

Think of:

```java
thread.interrupt();
```

as:

> **"Please stop/wake up/cooperate with cancellation."**

not:

> **"Kill this thread immediately."**

---

# 2391. Does `interrupt()` forcibly terminate a thread?

**No.**

This is a very common interview trap.

```java
thread.interrupt();
```

does not forcibly terminate the target thread.

The thread must generally cooperate.

For example:

```java
while (!Thread.currentThread().isInterrupted()) {
    doWork();
}
```

Once interrupted:

```text
interrupt()
   ↓
interrupt status set
   ↓
loop detects it
   ↓
thread exits
```

### Why is this design useful?

Forcibly killing a thread could leave shared state inconsistent.

Cooperative cancellation allows the thread to:

* Clean up resources
* Release application-level resources
* Roll back/finish appropriate work
* Exit safely

---

# 2392. What is the interrupted status of a thread?

Every thread has an **interruption status flag**.

You can check it with:

```java
thread.isInterrupted();
```

or:

```java
Thread.currentThread().isInterrupted();
```

Suppose:

```java
Thread.currentThread().interrupt();
```

Then:

```java
Thread.currentThread().isInterrupted()
```

returns:

```text
true
```

until the status is cleared.

### Important distinction

There are two commonly confused methods:

```java
isInterrupted()
```

and:

```java
Thread.interrupted()
```

We'll cover their exact difference below.

---

# 2393. What happens when `interrupt()` is called on a sleeping thread?

Suppose:

```java
try {
    Thread.sleep(10000);
} catch (InterruptedException e) {
    System.out.println("Interrupted");
}
```

Another thread calls:

```java
worker.interrupt();
```

The sleeping thread is awakened from `sleep()` and:

```text
InterruptedException
```

is thrown.

Conceptually:

```text
worker
 ↓
sleep(10 sec)
 ↓
TIMED_WAITING
        ↑
        |
    interrupt()
        |
        ↓
InterruptedException
```

### Important

When `InterruptedException` is thrown by `sleep()`, the thread's interrupt status is **cleared**.

Therefore, if you catch it and want higher-level code to know the thread was interrupted, a common pattern is:

```java
catch (InterruptedException e) {
    Thread.currentThread().interrupt();
}
```

We'll discuss why this is important in question 2395/2396 and later production scenarios.

---

# 2394. What happens when `interrupt()` is called on a waiting thread?

Suppose:

```java
synchronized (lock) {
    lock.wait();
}
```

The thread is in:

```text
WAITING
```

Another thread calls:

```java
thread.interrupt();
```

The waiting thread is awakened and:

```text
InterruptedException
```

is thrown.

Conceptually:

```text
WAITING
   ↓
interrupt()
   ↓
InterruptedException
   ↓
continues through exception handling
```

The interrupt status is cleared when `InterruptedException` is thrown.

---

# 2395. What happens when `interrupt()` is called on a thread that is doing normal computation?

Suppose:

```java
while (true) {
    calculate();
}
```

and another thread calls:

```java
worker.interrupt();
```

If the worker isn't currently in an interruptible blocking operation, `interrupt()` doesn't automatically stop it.

Instead, its interrupt status is set.

The worker must check it.

Example:

```java
while (!Thread.currentThread().isInterrupted()) {
    calculate();
}
```

Eventually:

```text
interrupt()
   ↓
interrupt flag = true
   ↓
loop checks flag
   ↓
loop exits
```

### This is cooperative cancellation.

---

# 2396. What is the difference between `Thread.interrupted()` and `isInterrupted()`?

🔥 **Very important interview question.**

Both check interruption status, but there is one critical difference.

### `isInterrupted()`

Instance method:

```java
thread.isInterrupted();
```

It **does not clear** the interrupt status.

Example:

```java
thread.interrupt();

System.out.println(thread.isInterrupted()); // true
System.out.println(thread.isInterrupted()); // true
```

---

### `Thread.interrupted()`

Static method:

```java
Thread.interrupted();
```

It checks the **current thread's** interrupt status and **clears it**.

Example:

```java
Thread.currentThread().interrupt();

System.out.println(Thread.interrupted()); // true
System.out.println(Thread.interrupted()); // false
```

### Comparison

| Method                   | Checks          | Clears flag? |
| ------------------------ | --------------- | ------------ |
| `thread.isInterrupted()` | Specific thread | ❌ No         |
| `Thread.interrupted()`   | Current thread  | ✅ Yes        |

### ⭐ Memorize this

> **`isInterrupted()` = check only.**
> **`Thread.interrupted()` = check + clear.**

---

# 2397. Does calling `Thread.interrupted()` clear the interrupt flag?

**Yes.**

Example:

```java
Thread.currentThread().interrupt();

System.out.println(
    Thread.interrupted()
); // true

System.out.println(
    Thread.interrupted()
); // false
```

First call:

```text
flag = true
   ↓
Thread.interrupted()
   ↓
returns true
   ↓
clears flag
```

Second call:

```text
flag = false
```

so it returns:

```text
false
```

---

# 2398. Does calling `isInterrupted()` clear the interrupt flag?

**No.**

Example:

```java
Thread.currentThread().interrupt();

Thread current = Thread.currentThread();

System.out.println(current.isInterrupted()); // true
System.out.println(current.isInterrupted()); // true
```

The flag remains set.

So:

```text
isInterrupted()
→ read status

Thread.interrupted()
→ read + clear status
```

---

# 🟡 Should Know

Now we move into the slightly more advanced thread methods.

---

# 2399. What is `yield()`?

`Thread.yield()` is a hint to the scheduler that the current thread is willing to yield its current execution opportunity.

Example:

```java
Thread.yield();
```

Conceptually:

```text
Thread A running
      ↓
    yield()
      ↓
"I am willing to let another runnable thread run"
```

But it is only a **hint**.

The scheduler may:

* Switch to another thread
* Continue running the current thread
* Treat the hint differently depending on OS/JVM behavior

---

# 2400. Is `yield()` guaranteed to pause the current thread?

**No.**

This is another interview trap.

```java
Thread.yield();
```

doesn't guarantee:

```text
current thread stops
```

It is only a scheduling hint.

The JVM/OS may decide that the same thread should continue.

Therefore, don't use `yield()` as a synchronization mechanism.

---

# 2401. Is `yield()` useful in production code?

Usually, you should **not depend on `yield()` for correctness or synchronization**.

It's a hint, not a guarantee.

Bad design:

```java
while (!condition) {
    Thread.yield();
}
```

This doesn't establish a proper synchronization/visibility protocol.

Instead, use appropriate mechanisms such as:

* `BlockingQueue`
* `CountDownLatch`
* `Lock`
* `Condition`
* `CompletableFuture`
* `wait/notify`
* Atomic/volatile mechanisms where appropriate

### Interview answer

> "`yield()` is only a scheduler hint and has no strong guarantee about when or whether the current thread will stop executing. Therefore, production synchronization should not depend on it."

---

# 2402. What is thread priority?

Thread priority is a scheduling-related property associated with a Java thread.

You can set it using:

```java
thread.setPriority(...);
```

and retrieve it using:

```java
thread.getPriority();
```

Example:

```java
thread.setPriority(Thread.MAX_PRIORITY);
```

### Important

Priority is a **hint to the scheduler**, not a guarantee of execution order.

---

# 2403. What are Java thread priority levels?

Java defines:

```java
Thread.MIN_PRIORITY
Thread.NORM_PRIORITY
Thread.MAX_PRIORITY
```

with values:

```text
MIN_PRIORITY  = 1
NORM_PRIORITY = 5
MAX_PRIORITY  = 10
```

So the valid Java priority range is:

```text
1 → 10
```

Default priority is generally:

```text
5
```

for a normally created thread unless inherited differently from its creator.

---

# 2404. Does thread priority guarantee execution order?

**No.**

Suppose:

```java
t1.setPriority(Thread.MAX_PRIORITY);
t2.setPriority(Thread.MIN_PRIORITY);
```

You cannot conclude:

```text
t1 definitely runs first
```

or:

```text
t1 gets 10× CPU time
```

The actual scheduling behavior depends on the JVM and operating system.

### Therefore

Never design correctness around:

```java
setPriority()
```

---

# 2405. Can thread priority be relied upon for synchronization?

**No.**

Synchronization should use proper concurrency mechanisms.

Don't do:

```text
Thread A priority = 10
Thread B priority = 1

therefore A must finish first
```

That's not a valid synchronization guarantee.

Use:

```text
join()
CountDownLatch
Future
CompletableFuture
Lock
BlockingQueue
```

etc., depending on the problem.

---

# 2406. What is a daemon thread?

A **daemon thread** is a background thread that does not keep the JVM alive by itself.

Example:

```java
Thread worker = new Thread(() -> {
    while (true) {
        // background work
    }
});

worker.setDaemon(true);
worker.start();
```

If all non-daemon threads have terminated, the JVM can shut down even if daemon threads are still running.

### Conceptual model

```text
JVM
│
├── Non-daemon Thread
│
├── Non-daemon Thread
│
└── Daemon Thread
```

As long as a non-daemon thread exists, the JVM remains alive.

Once all non-daemon threads finish:

```text
No non-daemon threads
        ↓
JVM can terminate
        ↓
Daemon threads don't prevent shutdown
```

---

# 2407. What happens when all non-daemon threads terminate?

The JVM can terminate.

Even if daemon threads are still running:

```text
Daemon Thread A → running
Daemon Thread B → running

Non-daemon threads → none
```

the JVM doesn't remain alive merely because those daemon threads exist.

Their execution can therefore be abruptly stopped as the JVM shuts down.

### Important

Daemon threads shouldn't normally be used for work that **must complete**, such as:

* Critical data persistence
* Important business transactions
* Required cleanup

---

# 2408. Can a daemon thread prevent JVM shutdown?

**No.**

That's essentially the defining property.

```text
All non-daemon threads terminate
          ↓
JVM may shut down
          ↓
daemon threads do not prevent it
```

---

# 2409. How do you create a daemon thread?

Call:

```java
thread.setDaemon(true);
```

**before** calling:

```java
thread.start();
```

Example:

```java
Thread t = new Thread(() -> {
    // background task
});

t.setDaemon(true);
t.start();
```

---

# 2410. Can `setDaemon()` be called after a thread has started?

**No.**

You must call:

```java
setDaemon(true);
```

before:

```java
start();
```

Example:

```java
Thread t = new Thread(task);

t.start();

t.setDaemon(true); // IllegalThreadStateException
```

Once a thread has started, its daemon status cannot be changed.

---

# 🔥 Extremely Important Comparison: `sleep()` vs `wait()` vs `join()`

This is something I strongly recommend memorizing:

|                    | `sleep()`            | `wait()`            | `join()`                                                                                |
| ------------------ | -------------------- | ------------------- | --------------------------------------------------------------------------------------- |
| Defined on         | `Thread`             | `Object`            | `Thread`                                                                                |
| Purpose            | Pause current thread | Thread coordination | Wait for another thread                                                                 |
| Releases monitor?  | ❌ No                 | ✅ Yes               | It doesn't release an application monitor held by the caller merely because of `join()` |
| Timeout available? | ✅                    | ✅                   | ✅                                                                                       |
| Interruptible?     | ✅                    | ✅                   | ✅                                                                                       |
| No-timeout state   | N/A                  | `WAITING`           | `WAITING`                                                                               |
| Timed state        | `TIMED_WAITING`      | `TIMED_WAITING`     | `TIMED_WAITING`                                                                         |

### The simplest mental model

```text
sleep()
→ "Pause me."

wait()
→ "I can't continue until someone signals the condition."

join()
→ "I can't continue until that thread finishes."
```

---

# 🔥 `interrupt()` Mental Model

Don't think:

```text
interrupt()
→ kill thread
```

Think:

```text
                 interrupt()
                     ↓
              interruption request
                     ↓
          ┌──────────┴──────────┐
          ↓                     ↓
   Thread is blocked       Normal computation
   in interruptible op
          ↓                     ↓
InterruptedException       interrupt flag set
          ↓                     ↓
      cooperate           check flag
```

And remember:

```text
sleep()
wait()
join()
   ↓
InterruptedException
   ↓
interrupt status cleared
```

while:

```text
isInterrupted()
→ doesn't clear

Thread.interrupted()
→ clears
```

---

# 🔥 Interview Deep-Dive Scenario

**Interviewer:**

> "I have a thread doing a long-running loop. I call `interrupt()` but the thread doesn't stop. Why?"

Strong answer:

> "`interrupt()` is cooperative cancellation; it doesn't forcibly terminate a thread. If the thread is doing normal computation rather than an interruptible blocking operation, the interrupt generally sets its interrupt status. The thread needs to check that status and exit or otherwise respond to the interruption."

Example:

```java
while (!Thread.currentThread().isInterrupted()) {
    performWork();
}
```

And if the work catches `InterruptedException`:

```java
catch (InterruptedException e) {
    Thread.currentThread().interrupt();
    // cleanup / exit
}
```

---

# 🧠 Section D Final Mental Map

```text
Thread Methods
│
├── sleep()
│     ├── pauses current thread
│     ├── TIMED_WAITING
│     ├── doesn't release monitor
│     └── interruptible
│
├── join()
│     ├── wait for another thread
│     ├── WAITING / TIMED_WAITING
│     └── interruptible
│
├── interrupt()
│     ├── NOT force-stop
│     ├── cooperative cancellation
│     ├── blocking operation → InterruptedException
│     └── normal computation → interrupt status
│
├── yield()
│     └── scheduler hint
│
├── Priority
│     └── scheduling hint, not guarantee
│
└── Daemon
      └── doesn't keep JVM alive
```

### ⭐ Five things to absolutely remember

1. **`sleep()` does not release a lock.**
2. **`wait()` releases the monitor it is waiting on.**
3. **`join()` makes the calling thread wait for another thread to terminate.**
4. **`interrupt()` does not kill a thread.**
5. **`Thread.interrupted()` clears the interrupt flag; `isInterrupted()` doesn't.**

Next is **Section E — `synchronized` and Intrinsic Locks**, which is a **major interview section**. We'll cover **monitors, intrinsic locks, synchronized methods vs blocks, instance vs static synchronization, reentrancy, private lock objects, and then JVM-level monitor concepts**.
