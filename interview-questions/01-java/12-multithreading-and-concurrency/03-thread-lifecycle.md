# C. Thread Lifecycle and Thread States

This is a **very important interview section**. Interviewers often don't stop at naming the states—they'll ask **exactly what causes each transition**, especially the difference between `BLOCKED`, `WAITING`, and `TIMED_WAITING`.

The section contains questions **2357–2381**. 

---

# 2357. What are the different states of a Java thread?

Java defines **six thread states** in `Thread.State`:

```text
NEW
RUNNABLE
BLOCKED
WAITING
TIMED_WAITING
TERMINATED
```

The enum is:

```java
Thread.State
```

A simplified lifecycle looks like:

```text
                 start()
NEW ─────────────────────→ RUNNABLE
                              │
                ┌─────────────┼──────────────┐
                │             │              │
                ↓             ↓              ↓
             BLOCKED       WAITING      TIMED_WAITING
                │             │              │
                └─────────────┴──────────────┘
                              │
                              ↓
                          RUNNABLE
                              │
                              ↓
                         TERMINATED
```

### Important interview point

Java has **six states**, but there is **no separate `RUNNING` state**.

---

# 2358. Explain the Java `Thread.State` enum.

Java provides:

```java
Thread.State
```

with six constants:

```java
Thread.State.NEW
Thread.State.RUNNABLE
Thread.State.BLOCKED
Thread.State.WAITING
Thread.State.TIMED_WAITING
Thread.State.TERMINATED
```

You can inspect a thread's current state:

```java
Thread.State state = thread.getState();
```

For example:

```java
Thread t = new Thread(() -> {
    // work
});

System.out.println(t.getState());
```

Before starting:

```text
NEW
```

After starting, it can become:

```text
RUNNABLE
```

Eventually:

```text
TERMINATED
```

### Important nuance

`Thread.State` represents the **JVM-level state abstraction**. It isn't a one-to-one description of every possible OS scheduler state.

---

# 2359. What is the `NEW` state?

A thread is in `NEW` when its `Thread` object has been created but `start()` hasn't successfully started it yet.

Example:

```java
Thread t = new Thread(() -> {
    System.out.println("Hello");
});
```

At this point:

```text
Thread object exists
        ↓
Thread.State.NEW
```

You can verify:

```java
System.out.println(t.getState());
```

Output:

```text
NEW
```

Once:

```java
t.start();
```

is called successfully, the thread transitions out of `NEW`.

### Important

`NEW` doesn't mean:

> "The thread is waiting to run."

It means:

> **The thread has not yet been started.**

---

# 2360. What causes a thread to transition from `NEW` to `RUNNABLE`?

Calling:

```java
thread.start();
```

causes the thread to transition from `NEW` toward `RUNNABLE`.

Conceptually:

```text
NEW
 │
 │ start()
 ↓
RUNNABLE
```

Example:

```java
Thread t = new Thread(() -> {
    System.out.println("Hello");
});

System.out.println(t.getState()); // NEW

t.start();
```

After `start()`, the thread becomes eligible for execution.

### Important nuance

`RUNNABLE` means **eligible/runnable**, not necessarily that the CPU is executing it at that exact instant.

---

# 2361. What is the `RUNNABLE` state?

`RUNNABLE` means the thread is eligible to run and may be:

* Actually executing on a CPU, or
* Ready/runnable and waiting for CPU scheduling.

This is a major interview point.

Suppose:

```text
Thread A → currently executing
Thread B → ready to execute
```

Both can conceptually be reported as:

```text
RUNNABLE
```

Java doesn't expose a separate:

```text
RUNNING
```

state.

### Example

```java
while (true) {
    calculate();
}
```

The thread may remain:

```text
RUNNABLE
```

while the scheduler gives it CPU time.

---

# 2362. Does Java have a separate `RUNNING` thread state?

**No.**

Java has:

```text
NEW
RUNNABLE
BLOCKED
WAITING
TIMED_WAITING
TERMINATED
```

There is no:

```text
RUNNING
```

state.

The `RUNNABLE` state encompasses the JVM-level concept of a thread that is ready to run or actually running.

### Interview trap

If asked:

> "What are the six thread states?"

Don't say:

```text
NEW → READY → RUNNING → ...
```

That's a common conceptual diagram, but **those aren't the six values of `Thread.State`**.

Say:

```text
NEW
RUNNABLE
BLOCKED
WAITING
TIMED_WAITING
TERMINATED
```

---

# 2363. What is the `BLOCKED` state?

A thread is in `BLOCKED` when it is **waiting to acquire an intrinsic monitor lock**.

For example:

```java
synchronized (lock) {
    // critical section
}
```

Suppose:

```text
Thread A → owns lock
Thread B → tries synchronized(lock)
```

Thread B can't acquire the monitor.

Therefore:

```text
Thread B
   ↓
BLOCKED
```

### Important

`BLOCKED` specifically refers to waiting for a **monitor lock**.

It does not simply mean:

> "The thread isn't doing anything."

---

# 2364. When does a thread enter the `BLOCKED` state?

A thread enters `BLOCKED` when it tries to enter a `synchronized` region but another thread already owns that monitor.

Example:

```java
Object lock = new Object();

Thread t1 = new Thread(() -> {
    synchronized (lock) {
        try {
            Thread.sleep(5000);
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        }
    }
});

Thread t2 = new Thread(() -> {
    synchronized (lock) {
        System.out.println("Got lock");
    }
});
```

If `t1` owns the lock:

```text
t1
 ↓
owns lock
 ↓
sleeping while holding lock

t2
 ↓
tries synchronized(lock)
 ↓
cannot acquire monitor
 ↓
BLOCKED
```

### Very important distinction

`t1` is:

```text
TIMED_WAITING
```

because it is sleeping.

`t2` is:

```text
BLOCKED
```

because it is waiting to acquire the monitor.

This distinction is **extremely useful in production troubleshooting**.

---

# 2365. What is the `WAITING` state?

`WAITING` means the thread is waiting **indefinitely for another thread to perform some action**.

Common operations that can cause `WAITING` include:

```java
Object.wait()
```

```java
Thread.join()
```

```java
LockSupport.park()
```

when used without a timeout.

Example:

```java
synchronized (lock) {
    lock.wait();
}
```

The thread enters:

```text
WAITING
```

until it is:

* Notified
* Interrupted
* Otherwise unparked, depending on the mechanism

### Key word

**Indefinitely.**

There is no timeout specified.

---

# 2366. What causes a thread to enter `WAITING`?

Typical examples:

### `Object.wait()`

```java
synchronized (lock) {
    lock.wait();
}
```

### `Thread.join()`

```java
thread.join();
```

without a timeout.

The calling thread waits for the target thread to terminate.

### `LockSupport.park()`

```java
LockSupport.park();
```

can put the thread into `WAITING`.

So:

```text
wait()
join()
park()
   ↓
WAITING
```

assuming no timeout is involved.

---

# 2367. What is the `TIMED_WAITING` state?

`TIMED_WAITING` means a thread is waiting for a **specified maximum amount of time**.

Common examples:

```java
Thread.sleep(5000);
```

```java
thread.join(5000);
```

```java
lock.wait(5000);
```

and timed `LockSupport.parkNanos()` / `parkUntil()`.

Conceptually:

```text
WAITING
→ wait indefinitely

TIMED_WAITING
→ wait for a specified time
```

---

# 2368. What operations can cause `TIMED_WAITING`?

Important examples:

### `Thread.sleep()`

```java
Thread.sleep(1000);
```

### Timed `join()`

```java
thread.join(1000);
```

### Timed `wait()`

```java
synchronized (lock) {
    lock.wait(1000);
}
```

### Timed parking

```java
LockSupport.parkNanos(...);
```

or:

```java
LockSupport.parkUntil(...);
```

### Timed lock acquisition

Some lock APIs can also result in waiting with a timeout; the exact reported state depends on the mechanism and implementation.

---

# 2369. What is the `TERMINATED` state?

A thread is `TERMINATED` when its execution has completed.

For example:

```java
Thread t = new Thread(() -> {
    System.out.println("Hello");
});

t.start();
```

Lifecycle:

```text
NEW
 ↓
RUNNABLE
 ↓
run() completes
 ↓
TERMINATED
```

Once terminated, the thread cannot execute again.

---

# 2370. Can a terminated thread be restarted?

**No.**

Example:

```java
Thread t = new Thread(() -> {
    System.out.println("Hello");
});

t.start();
```

After it finishes:

```text
TERMINATED
```

Calling:

```java
t.start();
```

again results in:

```text
IllegalThreadStateException
```

### Remember

A `Thread` object represents a **single execution lifecycle**.

If you need another execution, create another `Thread` object or submit the task again to an executor.

---

# 2371. Explain all possible transitions between Java thread states.

This is one of the most important questions in this section.

Let's build the lifecycle carefully.

## 1. `NEW → RUNNABLE`

Calling:

```java
thread.start();
```

```text
NEW
 ↓ start()
RUNNABLE
```

---

## 2. `RUNNABLE → BLOCKED`

The thread tries to acquire a monitor already owned by another thread.

```text
RUNNABLE
   ↓
tries synchronized(lock)
   ↓
lock unavailable
   ↓
BLOCKED
```

When it acquires the monitor:

```text
BLOCKED
   ↓
RUNNABLE
```

---

## 3. `RUNNABLE → WAITING`

For example:

```java
lock.wait();
```

or:

```java
otherThread.join();
```

or:

```java
LockSupport.park();
```

```text
RUNNABLE
   ↓
WAITING
```

After notification/unpark/termination/interruption as applicable:

```text
WAITING
   ↓
RUNNABLE
```

---

## 4. `RUNNABLE → TIMED_WAITING`

For example:

```java
Thread.sleep(5000);
```

```text
RUNNABLE
   ↓
TIMED_WAITING
```

When timeout expires:

```text
TIMED_WAITING
   ↓
RUNNABLE
```

---

## 5. `RUNNABLE → TERMINATED`

When `run()` finishes:

```text
RUNNABLE
   ↓
run() completes
   ↓
TERMINATED
```

---

### Complete simplified diagram

```text
                       start()
                 ┌──────────────────┐
                 │                  ↓
              NEW ─────────────→ RUNNABLE
                                  │   │   │
                         ┌────────┘   │   └─────────┐
                         ↓            ↓             ↓
                      BLOCKED       WAITING    TIMED_WAITING
                         │            │             │
                         └────────────┴─────────────┘
                                      │
                                      ↓
                                  RUNNABLE
                                      │
                                      ↓
                                  TERMINATED
```

### Important correction

There isn't necessarily one single linear path.

A thread can repeatedly move:

```text
RUNNABLE
 ↕
BLOCKED

RUNNABLE
 ↕
WAITING

RUNNABLE
 ↕
TIMED_WAITING
```

until its `run()` method completes.

---

# 2372. What is the difference between `BLOCKED` and `WAITING`?

🔥 **Very important interview question.**

### `BLOCKED`

The thread is waiting to acquire a **monitor lock**.

Example:

```java
synchronized (lock) {
}
```

Another thread owns the monitor.

```text
Thread A → owns lock
Thread B → BLOCKED
```

### `WAITING`

The thread is waiting for **another thread to perform some action**.

Example:

```java
synchronized (lock) {
    lock.wait();
}
```

The waiting thread has voluntarily released the monitor and waits for notification/other permitted wake-up.

### Comparison

| BLOCKED                                        | WAITING                                |
| ---------------------------------------------- | -------------------------------------- |
| Waiting for monitor acquisition                | Waiting for another action             |
| Caused by lock contention                      | Caused by `wait`, `join`, `park`, etc. |
| Doesn't own the monitor it's trying to acquire | `wait()` releases the monitor          |
| Usually caused by `synchronized`               | Often explicit thread coordination     |

### ⭐ Interview answer

> "`BLOCKED` means the thread is trying to acquire an intrinsic monitor that another thread currently owns. `WAITING` means the thread is waiting indefinitely for another thread or coordination mechanism to cause it to continue, such as `wait()`, `join()`, or `park()`."

---

# 2373. What is the difference between `WAITING` and `TIMED_WAITING`?

Simple distinction:

```text
WAITING
→ indefinite wait

TIMED_WAITING
→ wait with timeout
```

Examples:

### WAITING

```java
thread.join();
```

### TIMED_WAITING

```java
thread.join(5000);
```

Similarly:

```java
lock.wait();
```

versus:

```java
lock.wait(5000);
```

And:

```java
LockSupport.park();
```

versus timed parking.

### `sleep()` is always important here

```java
Thread.sleep(5000);
```

puts the thread into:

```text
TIMED_WAITING
```

---

# 2374. What is the difference between `RUNNABLE` and actually running on a CPU?

This is a subtle but **excellent interviewer question**.

Java's `RUNNABLE` state covers both:

```text
Ready to execute
```

and:

```text
Actually executing
```

There isn't a separate Java:

```text
RUNNING
```

state.

For example:

```text
CPU Core 1 → Thread A executing

Ready queue → Thread B
```

Both Thread A and Thread B can be reported as:

```text
RUNNABLE
```

depending on when their state is observed.

### Therefore

```text
RUNNABLE
≠ guaranteed CPU execution at this exact moment
```

It means the thread isn't blocked/waiting/terminated and is eligible for execution.

---

# 2375. Can a thread move directly from `NEW` to `TERMINATED`?

From the normal Java thread lifecycle:

**No.**

The thread needs to be started before its execution can terminate.

Typical lifecycle:

```text
NEW
 ↓
start()
 ↓
RUNNABLE
 ↓
run() completes
 ↓
TERMINATED
```

There is no normal API operation that takes an unstarted thread directly to `TERMINATED`.

---

# 2376. Can a thread move from `WAITING` directly to `TERMINATED`?

**Yes, depending on what caused the waiting and what happens afterward.**

The key is that a thread in `WAITING` must first become able to continue execution.

For example:

```text
WAITING
   ↓
notification / interrupt / target completion
   ↓
RUNNABLE
   ↓
run() completes
   ↓
TERMINATED
```

From the Java state model, termination happens when the thread's execution completes; a waiting thread doesn't execute its `run()` method while remaining in `WAITING`.

So conceptually you should think:

```text
WAITING → RUNNABLE → TERMINATED
```

rather than treating `WAITING → TERMINATED` as an ordinary direct transition.

---

# 🔴 Deep / Advanced

Now let's move into the questions that separate **basic Java knowledge from production-level concurrency knowledge**.

---

# 2377. How would you diagnose an application where hundreds of threads are in `BLOCKED` state?

This is a very realistic production question.

Suppose you take a thread dump and see:

```text
300 threads → BLOCKED
```

I would immediately investigate **monitor contention**.

### Step 1 — Take thread dumps

Use tools such as:

```bash
jstack <pid>
```

or:

```bash
jcmd <pid> Thread.print
```

Take multiple dumps a few seconds apart.

---

### Step 2 — Find what they're blocked on

Look for patterns like:

```text
BLOCKED waiting for monitor
```

and identify the lock/object involved.

For example:

```text
Thread-100:
  BLOCKED
  waiting to lock <0x1234>

Thread-101:
  BLOCKED
  waiting to lock <0x1234>

Thread-102:
  BLOCKED
  waiting to lock <0x1234>
```

Now you know many threads are competing for the same monitor.

---

### Step 3 — Find the thread owning the lock

Find which thread currently owns:

```text
<0x1234>
```

Then inspect its stack trace.

Maybe it's doing:

```text
synchronized(lock)
    ↓
database call
    ↓
network call
    ↓
slow operation
```

That's a major red flag.

### Step 4 — Look for oversized critical sections

Bad:

```java
synchronized (lock) {

    updateMemory();

    callDatabase();

    callExternalAPI();

    writeFile();
}
```

The lock is being held while doing slow I/O.

Better:

```text
minimal shared-state operation
        ↓
release lock
        ↓
slow I/O outside lock
```

---

### Step 5 — Investigate lock design

Ask:

* Is one global lock protecting too much?
* Can the lock be split?
* Is there unnecessary synchronization?
* Is a synchronized collection causing contention?
* Is some slow operation occurring while holding the lock?

### Production reasoning

```text
Hundreds BLOCKED
       ↓
Monitor contention
       ↓
Identify common lock
       ↓
Identify lock owner
       ↓
Inspect owner stack
       ↓
Find why lock is held so long
       ↓
Reduce contention / redesign
```

---

# 2378. How would you distinguish lock contention from I/O waiting using a thread dump?

This is a **very useful troubleshooting skill**.

### Lock contention

You'll typically see:

```text
BLOCKED
```

and something indicating the thread is waiting to acquire a monitor.

Example conceptually:

```text
"worker-10" BLOCKED
    waiting to lock <0x1234>
```

You then identify the thread holding that lock.

---

### I/O waiting

A thread performing blocking I/O may appear:

```text
RUNNABLE
```

even though it's spending time in native/network I/O, depending on the operation and JVM/OS interaction.

The stack trace is critical.

You might see calls associated with:

```text
Socket
File I/O
JDBC driver
Network read
```

### So don't just look at the state.

Use:

```text
Thread state
+
Stack trace
+
Repeated thread dumps
+
Application metrics
```

### Example

```text
Thread A
BLOCKED
waiting for monitor
```

→ likely lock contention.

Versus:

```text
Thread B
RUNNABLE
native/socket read
```

→ potentially blocked in I/O despite being reported as `RUNNABLE`.

This is why **thread state alone isn't enough for diagnosis**.

---

# 2379. Can a Java thread remain `RUNNABLE` while it is actually waiting on native I/O?

**Yes.**

This is a very good interview question.

A Java thread performing certain native or blocking I/O operations can appear as:

```text
RUNNABLE
```

in a thread dump even though it isn't actively consuming CPU.

Why?

Because Java's `RUNNABLE` state includes execution that may be inside native code, and the JVM's state doesn't necessarily distinguish every OS-level waiting condition.

### Example

A thread might be doing:

```text
JDBC call
   ↓
socket read
   ↓
waiting for database response
```

Yet a thread dump may report:

```text
RUNNABLE
```

depending on the call stack and JVM/platform behavior.

### Therefore

Never conclude:

> "RUNNABLE = consuming CPU."

Instead inspect:

```text
RUNNABLE
+
stack trace
+
CPU metrics
+
I/O metrics
```

---

# 2380. How does the JVM map Java threads to operating-system threads?

For **platform threads**, Java threads are backed by OS threads.

Conceptually:

```text
Java Thread
     ↓
JVM
     ↓
OS Thread
     ↓
CPU
```

The OS scheduler determines when the underlying OS thread executes.

For example:

```text
Java Thread A ──→ OS Thread A ──→ CPU Core 1
Java Thread B ──→ OS Thread B ──→ CPU Core 2
```

This is why creating thousands of platform threads can be expensive.

Each involves OS-level resources.

### But modern Java adds an important distinction

Java now has:

```text
Platform Threads
        +
Virtual Threads
```

Virtual threads are not simply one OS thread per Java thread.

We'll cover this in detail in Section AE.

---

# 2381. What is the difference between platform threads and virtual threads from a lifecycle perspective?

This is a modern Java question.

### Platform thread

A platform thread is backed by an OS thread.

Conceptually:

```text
Java Platform Thread
        ↓
OS Thread
        ↓
CPU
```

### Virtual thread

A virtual thread is a lightweight Java thread managed by the JVM/runtime.

It is not permanently tied one-to-one to an OS thread.

Instead:

```text
Many Virtual Threads
        ↓
Carrier Threads
        ↓
OS Threads
        ↓
CPU
```

A virtual thread can be mounted onto a carrier thread when executing.

When it performs certain blocking operations, it can be unmounted, allowing the carrier to execute another virtual thread.

We'll go much deeper into:

* Carrier threads
* Mounting/unmounting
* Pinning
* Blocking I/O
* Virtual-thread scalability

in **Section AE**.

### Lifecycle perspective

Both still have Java-level states:

```text
NEW
RUNNABLE
BLOCKED
WAITING
TIMED_WAITING
TERMINATED
```

The major difference is **how execution is scheduled/backed underneath**.

---

# 🔥 The Most Important Part: BLOCKED vs WAITING vs TIMED_WAITING

Memorize this table:

| State           | Meaning                            | Typical example                 |
| --------------- | ---------------------------------- | ------------------------------- |
| `NEW`           | Not started                        | `new Thread(...)`               |
| `RUNNABLE`      | Ready/running from JVM perspective | CPU work                        |
| `BLOCKED`       | Waiting for monitor                | `synchronized` lock unavailable |
| `WAITING`       | Waiting indefinitely               | `wait()`, `join()`, `park()`    |
| `TIMED_WAITING` | Waiting with timeout               | `sleep()`, timed `wait()`       |
| `TERMINATED`    | Execution finished                 | `run()` completed               |

### The three commonly confused states

```text
BLOCKED
   ↓
"I want a lock."

WAITING
   ↓
"I am waiting for someone/something to signal me."

TIMED_WAITING
   ↓
"I am waiting, but only for a specified amount of time."
```

---

# 🔥 Interview Scenario

Interviewer:

> "Thread A is inside a synchronized method and calls `Thread.sleep(10 seconds)`. Thread B calls the same synchronized method. What states are the two threads in?"

Answer:

```text
Thread A
→ TIMED_WAITING
→ because of sleep()

Thread B
→ BLOCKED
→ because it is waiting to acquire A's monitor
```

And the **critical detail**:

> `sleep()` does **not** release the monitor.

So Thread B remains blocked while A sleeps.

This connects directly to the next section.

---

## 🧠 Section C Mental Model

Think of a thread as moving through **execution vs waiting**:

```text
                   start()
                      ↓
                    NEW
                      ↓
                  RUNNABLE
                /     |      \
               /      |       \
              ↓       ↓        ↓
          BLOCKED   WAITING  TIMED_WAITING
              \       |        /
               \      |       /
                └──────┴──────┘
                       ↓
                   RUNNABLE
                       ↓
                  TERMINATED
```

And remember:

> **BLOCKED = waiting for a monitor.**
> **WAITING = waiting indefinitely.**
> **TIMED_WAITING = waiting with a timeout.**
> **RUNNABLE = may be running or merely ready to run.**

Next is **Section D — Thread Methods (2382–2410)**, where we'll go deeply into **`sleep()`, `join()`, `interrupt()`, interrupt status, `Thread.interrupted()` vs `isInterrupted()`, `yield()`, daemon threads, and several interview traps around interruption**.
