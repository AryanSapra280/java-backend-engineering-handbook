# Java Multithreading & Concurrency — 01: Thread Basics

## 1. Process vs Thread

### Process
A process is a running program. Examples include Chrome, IntelliJ IDEA, PostgreSQL, and a Spring Boot application.

A process generally has its own memory space and resources.

### Thread
A thread is an execution path inside a process.

```text
Java Process
│
├── Main Thread
├── Worker Thread 1
├── Worker Thread 2
├── Worker Thread 3
└── ...
```

Threads in the same process share process-level resources and memory.

---

## 2. Why Multiple Threads?

Sequential:

```text
Task A → wait → Task B → wait → Task C
```

Concurrent:

```text
Thread 1 → Task A
Thread 2 → Task B
Thread 3 → Task C
```

Multithreading can improve responsiveness and throughput, especially for workloads involving waiting/I/O, but threads have overhead.

---

## 3. Is Multithreading Always Faster?

No. Thread creation, context switching, memory, synchronization, and coordination all have costs.

Creating thousands of threads unnecessarily can make an application slower.

This is why Java provides:

```text
ExecutorService
Thread Pools
```

---

## 4. Concurrency vs Parallelism

### Concurrency
Multiple tasks are in progress during overlapping periods.

### Parallelism
Multiple tasks are actually executing at the same time, typically on multiple CPU cores.

```text
Concurrency
→ dealing with multiple tasks

Parallelism
→ executing multiple tasks simultaneously
```

---

## 5. Creating a Thread by Extending Thread

```java
class MyThread extends Thread {

    @Override
    public void run() {
        System.out.println("Running...");
    }
}

public class Main {
    public static void main(String[] args) {
        MyThread thread = new MyThread();
        thread.start();
    }
}
```

---

## 6. `start()` vs `run()` ⭐⭐⭐

```java
thread.run();
```

is a normal method call and executes on the current thread.

```java
thread.start();
```

starts a new thread of execution, which then executes `run()`.

```text
run()
 ↓
normal method call
 ↓
same thread

start()
 ↓
new thread starts
 ↓
new thread executes run()
```

Remember:

```java
thread.run();      // Normal method call
thread.start();    // Starts a new thread
```

---

## 7. Proving `start()` vs `run()`

```java
class MyThread extends Thread {

    @Override
    public void run() {
        System.out.println(
            "Running in: " +
            Thread.currentThread().getName()
        );
    }
}

public class Main {
    public static void main(String[] args) {

        System.out.println(
            "Main: " +
            Thread.currentThread().getName()
        );

        MyThread thread = new MyThread();

        thread.run();   // try start() here instead
    }
}
```

`run()` executes on the main thread. Replacing it with `start()` lets a different thread execute `run()`.

---

## 8. `Thread.currentThread()`

```java
Thread.currentThread()
```

returns the currently executing thread.

Example:

```java
System.out.println(
    Thread.currentThread().getName()
);
```

A thread can be named:

```java
MyThread thread = new MyThread();
thread.setName("PaymentWorker");
thread.start();
```

Thread names are useful for debugging.

---

## 9. Using Runnable

```java
class MyTask implements Runnable {

    @Override
    public void run() {
        System.out.println("Task running");
    }
}

Thread thread = new Thread(new MyTask());
thread.start();
```

### Why prefer Runnable?

Java supports single inheritance. Extending `Thread` uses the superclass slot.

With `Runnable`:

```java
class MyTask extends SomeOtherClass
        implements Runnable {
}
```

More importantly:

```text
Runnable
→ what work should be done?

Thread
→ mechanism executing the work
```

This separation becomes especially important with `ExecutorService` and thread pools.

---

## 10. Runnable + Lambda

`Runnable` is a functional interface with:

```java
void run();
```

Therefore:

```java
Runnable task =
    () -> System.out.println("Running");

Thread thread = new Thread(task);
thread.start();
```

Or:

```java
new Thread(
    () -> System.out.println("Running")
).start();
```

The compiler knows the lambda represents `Runnable` because the constructor expects a `Runnable`.

```text
Thread constructor
        ↓
expects Runnable
        ↓
Runnable.run()
        ↓
run() has zero parameters
        ↓
() -> ...
```

---

## 11. Practical Example — Two Threads

```java
public class Main {

    public static void main(String[] args) {

        Runnable task = () -> {

            for (int i = 1; i <= 5; i++) {

                System.out.println(
                    Thread.currentThread().getName()
                    + " : "
                    + i
                );
            }
        };

        Thread t1 = new Thread(task, "Worker-1");
        Thread t2 = new Thread(task, "Worker-2");

        t1.start();
        t2.start();
    }
}
```

Do not expect output to always be in the same order. Thread scheduling is not deterministic.

---

## 12. `Thread.sleep()`

`sleep()` pauses the currently executing thread for approximately the specified duration.

```java
System.out.println("Start");

Thread.sleep(2000);

System.out.println("End");
```

Mental model:

```text
Current Thread
    ↓
Start
    ↓
sleep(2000)
    ↓
PAUSED
    ↓
~2 seconds
    ↓
End
```

`sleep()` is static and means the currently executing thread sleeps.

---

## 13. Does `sleep()` Release a Lock?

No.

```java
synchronized (lock) {
    Thread.sleep(5000);
}
```

The thread continues holding the monitor lock while sleeping.

```text
sleep()
→ pauses the thread
→ does NOT release a synchronized monitor lock
```

---

## 14. `sleep()` and InterruptedException

A sleeping thread can be interrupted:

```java
try {
    Thread.sleep(2000);
} catch (InterruptedException e) {
    Thread.currentThread().interrupt();
}
```

`InterruptedException` can occur when an interruptible blocking operation is interrupted.

---

## 15. `join()`

Suppose:

```java
Thread t1 = new Thread(task);
t1.start();
```

The current thread does not automatically wait for `t1`.

To wait:

```java
t1.join();
```

Meaning:

> The current thread waits until `t1` terminates.

---

## 16. Example of join()

```java
Thread t1 = new Thread(() -> {

    System.out.println("Worker started");

    try {
        Thread.sleep(3000);
    } catch (InterruptedException e) {
        Thread.currentThread().interrupt();
    }

    System.out.println("Worker finished");

});

t1.start();

t1.join();

System.out.println("Main finished");
```

Flow:

```text
Main
 |
 | start t1
 ↓
t1 running
 |
 | join()
 ↓
Main waits
 |
 ↓
t1 finishes
 |
 ↓
Main continues
```

`join()` does not stop the target thread. It makes the calling thread wait.

---

## 17. `join()` With Two Threads

Concurrent start:

```java
t1.start();
t2.start();

t1.join();
t2.join();
```

Both threads can execute concurrently; the main thread waits until both have finished.

Sequential start:

```java
t1.start();
t1.join();

t2.start();
t2.join();
```

Here t2 starts only after t1 finishes.

Therefore, if you want t2 to start only after t1:

```java
t1.start();
t1.join();
t2.start();
```

---

## 18. `interrupt()`

```java
worker.interrupt();
```

does **not** forcibly kill the thread.

It sends an interruption request/signal. The thread must cooperate by handling `InterruptedException` or checking its interrupted status.

---

## 19. Interrupting a Sleeping Thread

```java
Thread worker = new Thread(() -> {

    try {
        Thread.sleep(10000);
    } catch (InterruptedException e) {
        System.out.println("I was interrupted!");
    }

});

worker.start();
```

Another thread can call:

```java
worker.interrupt();
```

The sleeping thread can receive `InterruptedException` and wake up.

---

## 20. Cooperative Cancellation

```java
Thread worker = new Thread(() -> {

    while (!Thread.currentThread().isInterrupted()) {
        System.out.println("Working...");
    }

    System.out.println("Stopping...");
});

worker.start();
```

Then:

```java
worker.interrupt();
```

Flow:

```text
Working
Working
Working
   ↓
interrupt()
   ↓
interrupted status
   ↓
loop detects interruption
   ↓
loop exits
```

This is cooperative cancellation.

---

## 21. `InterruptedException` and Restoring the Flag

A common pattern:

```java
try {
    Thread.sleep(5000);
} catch (InterruptedException e) {
    Thread.currentThread().interrupt();
}
```

When `sleep()` responds to interruption by throwing `InterruptedException`, the interrupted status is cleared as part of that mechanism.

If the current layer is not going to fully handle cancellation, restoring the status lets higher-level code detect the interruption.

---

## 22. `sleep()` vs `join()` vs `interrupt()`

| Method | Meaning |
|---|---|
| `sleep()` | Pause the currently executing thread |
| `join()` | Current thread waits for another thread to finish |
| `interrupt()` | Request another thread to respond to interruption/cancellation |

Mental model:

```text
sleep()
→ "I need to pause."

join()
→ "I need to wait for you."

interrupt()
→ "Please respond to this cancellation/interruption request."
```

---

## 23. `join()` and Locks

`join()` makes the calling thread wait for the target thread. It does not mean "release all locks currently held by the calling thread."

Potential deadlock:

```java
synchronized (lock) {
    t1.start();
    t1.join();
}
```

If `t1` also needs `lock`:

```java
synchronized (lock) {
    // work
}
```

then:

```text
Main thread
    ↓
acquires lock
    ↓
starts t1
    ↓
join()
    ↓
waits for t1

t1
 ↓
tries to acquire lock
 ↓
BLOCKED
```

Therefore:

```text
Main waits for t1
      ↑
      |
t1 waits for lock
      ↑
      |
Main owns lock
```

This is a deadlock.

---

## 24. `interrupt()` vs `stop()`

Do not use the deprecated:

```java
Thread.stop()
```

as a normal cancellation mechanism.

Prefer cooperative interruption:

```java
thread.interrupt();
```

and make the worker respond appropriately.

---

# Interview Questions

## Basic

1. What is a process?
2. What is a thread?
3. What is the difference between a process and a thread?
4. What is concurrency?
5. What is parallelism?

## Important

6. What is the difference between `thread.start()` and `thread.run()`?
7. Why would you prefer `Runnable` over extending `Thread`?
8. What does `Thread.currentThread()` return?
9. What does `Thread.sleep()` do?
10. Does `sleep()` release a synchronized lock?
11. What does `join()` do?
12. If `t1.start(); t2.start();` are called, is their execution order guaranteed?
13. How can you make t2 start only after t1 finishes?
14. Does `interrupt()` kill a thread?
15. What is cooperative cancellation?
16. Why might you call `Thread.currentThread().interrupt()` inside a catch block for `InterruptedException`?
17. What can go wrong with `synchronized(lock) { t1.start(); t1.join(); }` if t1 also needs `lock`?

---

# Coding Practice

## Q1 — Basic

Create a thread using `Runnable` that prints:

```text
Hello from worker
```

## Q2 — Basic

Create two threads:

```text
Worker-1
Worker-2
```

Each should print numbers from 1 to 5.

Observe whether the ordering is deterministic.

## Q3 — Medium

Create a worker thread that:

1. Prints `"Working"`
2. Sleeps for 5 seconds
3. Prints `"Finished"`

From the main thread, interrupt it after 1 second.

Observe what happens.

## Q4 — Medium

Create two threads and make the main thread wait until both have finished:

```java
t1.start();
t2.start();

t1.join();
t2.join();
```

## Q5 — Interview ⭐⭐⭐

Write a program where:

```text
Thread 1
    ↓
does some work
    ↓
finishes

Thread 2
    ↓
starts only after Thread 1 finishes
```

Use `join()`.

## Q6 — Interview ⭐⭐⭐

Create a worker loop that can be stopped using `interrupt()` without forcibly killing the thread.

---

# Key Takeaways

```text
Process
→ running program

Thread
→ execution path inside a process

start()
→ starts a new thread

run()
→ normal method call

Runnable
→ represents the task/work

sleep()
→ pauses current thread

join()
→ current thread waits for another thread

interrupt()
→ interruption/cancellation request

InterruptedException
→ can occur when interruptible blocking operations are interrupted
```

Most important distinction:

```text
start()
    ↓
"Start this work concurrently."

join()
    ↓
"I'll wait until this thread finishes."

interrupt()
    ↓
"Please respond to this cancellation/interruption request."
```

---

# MUFG Interview Focus

The high-value topics coming next are:

```text
Race conditions
synchronized
AtomicInteger
ConcurrentHashMap
ExecutorService
ThreadPool
Future
CompletableFuture
```

These connect directly to resume claims around:

- Multithreading
- Concurrency
- Asynchronous processing
- Event-driven architecture
- Performance optimization

---

# Where We Go Next

```text
Thread Basics                         ✅
sleep() / join() / interrupt()        ✅
    ↓
Thread Lifecycle                      ← NEXT
    ↓
Race Conditions                       ⭐⭐⭐
    ↓
synchronized                          ⭐⭐⭐
    ↓
volatile                              ⭐⭐⭐
    ↓
Atomic Classes                        ⭐⭐⭐
    ↓
Locks / ReentrantLock                 ⭐⭐⭐
    ↓
ExecutorService                       ⭐⭐⭐
    ↓
Thread Pools                          ⭐⭐⭐
    ↓
Future                                ⭐⭐⭐
    ↓
CompletableFuture                     ⭐⭐⭐⭐⭐
    ↓
Async Programming                     ⭐⭐⭐⭐⭐
    ↓
Concurrent Collections                ⭐⭐⭐
    ↓
Deadlock / Starvation / Livelock      ⭐⭐⭐
    ↓
Concurrency Coding Problems           ⭐⭐⭐⭐⭐
```

## Final Mental Model

```text
Thread
    ↓
executes a task

Runnable
    ↓
represents the task

start()
    ↓
starts concurrent execution

sleep()
    ↓
pause current thread

join()
    ↓
wait for another thread to finish

interrupt()
    ↓
request interruption/cancellation

ExecutorService
    ↓
manage threads efficiently

CompletableFuture
    ↓
compose asynchronous operations
```
