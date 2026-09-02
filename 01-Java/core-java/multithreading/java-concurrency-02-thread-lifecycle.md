# Java Multithreading & Concurrency — 02: Thread Lifecycle & Thread States

## 1. Java's Official Thread States

Java provides six official values in `Thread.State`:

```text
NEW
RUNNABLE
BLOCKED
WAITING
TIMED_WAITING
TERMINATED
```

A simplified lifecycle:

```text
                 start()
NEW ─────────────────────────→ RUNNABLE
                                │
                    ┌───────────┼────────────┐
                    ↓           ↓            ↓
                 BLOCKED      WAITING    TIMED_WAITING
                    │           │            │
                    └───────────┴────────────┘
                                ↓
                            RUNNABLE
                                ↓
                           TERMINATED
```

### Important interview detail ⭐⭐⭐

Java does **not** have a separate official `RUNNING` state.

When a Java thread is actually executing, its state is still reported as:

```text
RUNNABLE
```

`RUNNABLE` can represent a thread that is ready to run as well as one that is currently running.

---

# 2. `NEW`

When a Thread object is created:

```java
Thread t1 = new Thread(() -> {
    System.out.println("Hello");
});
```

the thread is in:

```text
NEW
```

At this point:

```text
Thread object exists
        ↓
Thread has not started
```

You can check:

```java
System.out.println(t1.getState());
```

Output:

```text
NEW
```

---

# 3. Calling `start()`

When:

```java
t1.start();
```

is called, the thread becomes eligible for execution.

Conceptually:

```text
NEW
 ↓
start()
 ↓
RUNNABLE
```

The JVM/OS scheduler determines when it actually gets CPU time.

---

# 4. What Does `RUNNABLE` Mean?

This is one of the most important details.

`RUNNABLE` means the thread is:

> Ready to run or currently running in the JVM.

It does **not necessarily mean the CPU is executing it at this exact moment**.

For example, with:

```text
4 threads
1 CPU core
```

only one thread can execute at an instant, while multiple threads may be eligible to run.

Therefore:

```text
RUNNABLE
=
eligible to run / actually running
```

Java combines those concepts into one state.

---

# 5. Why Doesn't Java Have a RUNNING State?

Java's `Thread.State` abstraction does not distinguish:

```text
ready to execute
```

from:

```text
currently executing
```

at the Java thread-state level.

The scheduler and operating system handle the actual CPU scheduling.

Therefore:

```text
Java Thread.State
        ↓
RUNNABLE
```

can represent both:

```text
ready to run
```

and:

```text
currently running
```

---

# 6. `BLOCKED`

A thread becomes:

```text
BLOCKED
```

when it is waiting to acquire a **monitor lock**.

This commonly happens with:

```java
synchronized
```

Example:

```java
synchronized (lock) {
    // critical section
}
```

Suppose Thread 1 owns the lock:

```text
Thread 1
    ↓
acquires lock
    ↓
inside synchronized block
```

Then Thread 2 tries:

```java
synchronized (lock) {
    // ...
}
```

Thread 2 cannot acquire the monitor while Thread 1 owns it.

Therefore:

```text
Thread 2
    ↓
BLOCKED
    ↓
waiting to acquire lock
```

---

# 7. Example of BLOCKED

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

        System.out.println("T2 acquired lock");

    }

});
```

If:

```java
t1.start();
```

gets the lock first and then:

```java
t2.start();
```

Thread 2 may become:

```text
BLOCKED
```

while waiting for the monitor.

---

# 8. `BLOCKED` vs `WAITING` ⭐⭐⭐

These are very commonly confused.

## BLOCKED

Means:

> I am waiting to acquire a monitor lock.

Typical example:

```java
synchronized (lock)
```

when another thread already owns the monitor.

---

## WAITING

Means:

> I am waiting indefinitely for another thread/action to allow me to continue.

Examples include:

```java
Object.wait()
Thread.join()
LockSupport.park()
```

without a timeout.

### Easy mental model

```text
BLOCKED
→ waiting for a lock

WAITING
→ waiting for another thread/action
```

---

# 9. `WAITING`

A thread enters `WAITING` when it waits indefinitely for another thread or action.

For example:

```java
t1.join();
```

If `t1` is still running, the calling thread can enter:

```text
WAITING
```

because it is waiting for `t1` to terminate.

---

# 10. `wait()`

Another important example:

```java
synchronized (lock) {

    lock.wait();

}
```

The thread enters:

```text
WAITING
```

until another thread performs something such as:

```java
lock.notify();
```

or:

```java
lock.notifyAll();
```

This will be studied more deeply when we cover thread coordination and synchronization.

---

# 11. `TIMED_WAITING`

`TIMED_WAITING` is similar to `WAITING`, except the thread waits for a **specified maximum amount of time**.

Examples:

```java
Thread.sleep(2000);
```

```java
t1.join(2000);
```

```java
lock.wait(2000);
```

These can put the thread into:

```text
TIMED_WAITING
```

### Mental model

```text
WAITING
→ wait indefinitely

TIMED_WAITING
→ wait for a specified time
```

---

# 12. `sleep()` and TIMED_WAITING

Remember:

```java
Thread.sleep(5000);
```

The current thread enters:

```text
TIMED_WAITING
```

for approximately that duration.

Important:

```java
synchronized (lock) {
    Thread.sleep(5000);
}
```

The thread **continues holding the monitor** while sleeping.

So:

```text
sleep()
→ TIMED_WAITING
→ does NOT release a synchronized monitor lock
```

---

# 13. `TERMINATED`

When the thread's `run()` method completes:

```text
RUNNABLE
   ↓
TERMINATED
```

Example:

```java
Thread t1 = new Thread(() -> {
    System.out.println("Hello");
});

t1.start();
```

Eventually:

```text
run() finishes
     ↓
TERMINATED
```

You can check:

```java
System.out.println(t1.getState());
```

after completion.

Output:

```text
TERMINATED
```

---

# 14. Can a TERMINATED Thread Be Restarted? ⭐⭐⭐

No.

Suppose:

```java
Thread t1 = new Thread(task);

t1.start();
```

After it finishes:

```text
TERMINATED
```

You cannot do:

```java
t1.start();
```

again.

This results in:

```text
IllegalThreadStateException
```

If you need another execution, create a new Thread object:

```java
Thread t2 = new Thread(task);
t2.start();
```

### Mental model

```text
Thread object
    ↓
NEW
    ↓
start()
    ↓
execution
    ↓
TERMINATED
```

A terminated Thread cannot transition back to `NEW`.

---

# 15. Complete Lifecycle

A simplified lifecycle:

```text
             new Thread()
                  │
                  ↓
                NEW
                  │
                start()
                  │
                  ↓
              RUNNABLE
             /    |     \
            /     |      \
           ↓      ↓       ↓
      BLOCKED  WAITING  TIMED_WAITING
           \      |       /
            \     |      /
             \    |     /
              ↓   ↓    ↓
               RUNNABLE
                  │
             run() ends
                  │
                  ↓
             TERMINATED
```

The key is to understand **why** a thread enters each state.

---

# 16. RUNNABLE → BLOCKED

Suppose:

```java
synchronized (lock) {
    // critical section
}
```

Thread A owns the lock.

Thread B attempts to enter:

```java
synchronized (lock) {
    // critical section
}
```

Thread B can transition:

```text
RUNNABLE
   ↓
tries to acquire monitor
   ↓
lock unavailable
   ↓
BLOCKED
```

When the lock becomes available, Thread B can become eligible to run again:

```text
BLOCKED
   ↓
RUNNABLE
```

---

# 17. RUNNABLE → WAITING

Suppose:

```java
t1.join();
```

The calling thread waits for t1.

Conceptually:

```text
RUNNABLE
   ↓
join()
   ↓
WAITING
   ↓
t1 finishes
   ↓
RUNNABLE
```

---

# 18. RUNNABLE → TIMED_WAITING

Example:

```java
Thread.sleep(3000);
```

Conceptually:

```text
RUNNABLE
   ↓
sleep(3000)
   ↓
TIMED_WAITING
   ↓
time expires
   ↓
RUNNABLE
```

---

# 19. Checking Thread State

Java provides:

```java
thread.getState()
```

Example:

```java
Thread t1 = new Thread(() -> {

    try {
        Thread.sleep(3000);
    } catch (InterruptedException e) {
        Thread.currentThread().interrupt();
    }

});

System.out.println(
    "Before start: " + t1.getState()
);

t1.start();

System.out.println(
    "After start: " + t1.getState()
);
```

Possible output:

```text
Before start: NEW
After start: RUNNABLE
```

Because scheduling is nondeterministic, exact observations can vary.

---

# 20. State Observations Are Snapshots

Consider:

```java
t1.start();
System.out.println(t1.getState());
```

You may expect:

```text
RUNNABLE
```

But the thread could execute very quickly and reach another state before `getState()` is evaluated.

Similarly, a thread may transition between states immediately after you observe it.

Therefore:

> `getState()` is a snapshot of the thread state at that moment, not a guarantee about what happens next.

This is important when debugging concurrent applications.

---

# 21. Practical Lifecycle Exercise

Run:

```java
public class Main {

    public static void main(String[] args)
            throws InterruptedException {

        Thread worker = new Thread(() -> {

            try {

                Thread.sleep(3000);

            } catch (InterruptedException e) {

                Thread.currentThread().interrupt();
            }

        });

        System.out.println(
            "Before start: " +
            worker.getState()
        );

        worker.start();

        System.out.println(
            "After start: " +
            worker.getState()
        );

        Thread.sleep(500);

        System.out.println(
            "While sleeping: " +
            worker.getState()
        );

        worker.join();

        System.out.println(
            "After completion: " +
            worker.getState()
        );
    }
}
```

Conceptually you should observe:

```text
Before start:
NEW

After start:
RUNNABLE

While sleeping:
TIMED_WAITING

After completion:
TERMINATED
```

Exact timing can make individual observations vary.

---

# 22. Practical BLOCKED Exercise

Try creating a blocked thread:

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

        System.out.println("T2 got the lock");

    }

});

t1.start();

Thread.sleep(100);

t2.start();

Thread.sleep(100);

System.out.println(
    "T2 state: " + t2.getState()
);
```

You should be able to observe:

```text
T2 state: BLOCKED
```

Why?

```text
T1
 ↓
owns lock
 ↓
sleeping
 ↓
still owns lock

T2
 ↓
tries synchronized(lock)
 ↓
lock unavailable
 ↓
BLOCKED
```

This is the connection between thread lifecycle and synchronization.

---

# 23. State Cheat Sheet ⭐⭐⭐

| State | Meaning |
|---|---|
| `NEW` | Thread created but not started |
| `RUNNABLE` | Ready to run or currently executing |
| `BLOCKED` | Waiting to acquire a monitor lock |
| `WAITING` | Waiting indefinitely for another action/thread |
| `TIMED_WAITING` | Waiting for a specified maximum time |
| `TERMINATED` | Execution completed |

---

# 24. Three States You Must Not Confuse

## BLOCKED

```text
"I want the lock, but someone else has it."
```

Example:

```java
synchronized (lock)
```

when another thread owns the monitor.

---

## WAITING

```text
"I'm waiting for another thread/action."
```

Examples:

```java
wait()
join()
park()
```

without a timeout.

---

## TIMED_WAITING

```text
"I'm waiting, but only for a limited time."
```

Examples:

```java
sleep()
wait(timeout)
join(timeout)
```

---

# 25. Interview Questions

## Basic

1. What are the six states of a Java thread?
2. What is the `NEW` state?
3. What happens when `start()` is called?
4. What does `RUNNABLE` mean?
5. What is the `TERMINATED` state?

## Important ⭐⭐⭐

6. Does Java have a separate `RUNNING` state?

7. What is the difference between `RUNNABLE` and `BLOCKED`?

8. What is the difference between `BLOCKED` and `WAITING`?

9. What is the difference between `WAITING` and `TIMED_WAITING`?

10. Which state does `Thread.sleep()` cause?

11. Which state can a thread enter when waiting for `join()` without a timeout?

12. Can a thread in `TERMINATED` state be started again?

13. What happens if you call `start()` twice on the same Thread?

14. How does a thread enter `BLOCKED` state?

15. Does `sleep()` release a synchronized monitor lock?

16. What does `thread.getState()` return?

17. Why shouldn't you assume that `getState()` will remain the same immediately after observing it?

---

# 26. Strong Interview Answers

### Q: Does Java have a RUNNING state?

> No. Java's `Thread.State` enum has six states and does not have a separate RUNNING state. A thread that is executing or ready to execute is represented by `RUNNABLE`.

### Q: BLOCKED vs WAITING?

> BLOCKED means the thread is waiting to acquire a monitor lock, typically because another thread owns it. WAITING means the thread is waiting indefinitely for another thread or action, such as after `wait()` or `join()` without a timeout.

### Q: WAITING vs TIMED_WAITING?

> WAITING means indefinite waiting. TIMED_WAITING means waiting for a specified maximum duration.

### Q: Can a terminated Thread be restarted?

> No. A Thread object can be started only once. Calling `start()` again after termination throws `IllegalThreadStateException`.

---

# 27. MUFG Interview Focus

For your MUFG interview, the most important concepts from this section are:

```text
⭐⭐⭐ Thread.State
⭐⭐⭐ RUNNABLE vs BLOCKED
⭐⭐⭐ BLOCKED vs WAITING
⭐⭐⭐ WAITING vs TIMED_WAITING
⭐⭐⭐ sleep()
⭐⭐⭐ join()
⭐⭐⭐ synchronized → BLOCKED
⭐⭐⭐ TERMINATED
⭐⭐⭐ Thread cannot be restarted
```

These are foundational for the next topic:

```text
Thread Lifecycle
      ↓
Race Conditions
      ↓
synchronized
      ↓
Locks
      ↓
Atomic Classes
      ↓
Concurrency
```

---

# 28. Where We Go Next

```text
01. Thread Basics                    ✅
02. Thread Lifecycle                 ✅
        ↓
03. Race Conditions                  ← NEXT ⭐⭐⭐⭐⭐
        ↓
04. synchronized                     ⭐⭐⭐⭐⭐
        ↓
05. volatile                         ⭐⭐⭐
        ↓
06. Atomic Classes                   ⭐⭐⭐⭐
        ↓
07. Locks / ReentrantLock            ⭐⭐⭐⭐
        ↓
08. ExecutorService                  ⭐⭐⭐⭐⭐
        ↓
09. Thread Pools                     ⭐⭐⭐⭐⭐
        ↓
10. Future                           ⭐⭐⭐⭐
        ↓
11. CompletableFuture                ⭐⭐⭐⭐⭐
        ↓
12. Async Programming                ⭐⭐⭐⭐⭐
        ↓
13. Concurrent Collections           ⭐⭐⭐⭐
        ↓
14. Deadlock / Starvation / Livelock ⭐⭐⭐⭐
        ↓
15. Concurrency Coding Problems      ⭐⭐⭐⭐⭐
```

# Final Mental Model

```text
NEW
→ Thread object exists, but hasn't started.

RUNNABLE
→ Thread can execute / is executing.

BLOCKED
→ Thread needs a monitor lock.

WAITING
→ Thread waits indefinitely for another action/thread.

TIMED_WAITING
→ Thread waits for a limited time.

TERMINATED
→ Thread execution is complete.
```

The key question to ask whenever you see a thread state is:

> **"What exactly is this thread waiting for?"**

```text
Waiting for CPU / execution opportunity
→ RUNNABLE

Waiting for a monitor lock
→ BLOCKED

Waiting indefinitely for another action
→ WAITING

Waiting with a time limit
→ TIMED_WAITING

Execution finished
→ TERMINATED
```
