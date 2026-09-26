# B. Creating and Starting Threads

We’ll continue exactly section-by-section. This section is **very important for interviews** because interviewers often start with `Thread` vs `Runnable` and then drill into **`start()` vs `run()`**, thread lifecycle, and exception handling.

The question set here covers **2332–2356**. 

---

## 2332. What are the different ways to create a thread in Java?

There are several approaches, but conceptually the important ones are:

### 1. Extend `Thread`

```java
class MyThread extends Thread {

    @Override
    public void run() {
        System.out.println("Running...");
    }
}
```

Then:

```java
MyThread t = new MyThread();
t.start();
```

---

### 2. Implement `Runnable`

```java
class MyTask implements Runnable {

    @Override
    public void run() {
        System.out.println("Running...");
    }
}
```

Then:

```java
Thread t = new Thread(new MyTask());
t.start();
```

Or using a lambda:

```java
Thread t = new Thread(() -> {
    System.out.println("Running...");
});

t.start();
```

---

### 3. Use `Callable` with an executor

`Callable` is useful when the task needs to **return a result** or throw checked exceptions.

```java
ExecutorService executor = Executors.newFixedThreadPool(2);

Future<Integer> future = executor.submit(() -> {
    return 10 + 20;
});
```

Technically, you're submitting a task to an executor rather than manually creating the thread.

---

### 4. Use an Executor/Thread Pool

In real backend applications, this is generally much more common than manually creating threads.

```java
ExecutorService executor =
        Executors.newFixedThreadPool(10);

executor.submit(() -> {
    System.out.println("Task running");
});
```

The important distinction is:

> **Thread represents execution; Runnable/Callable represents work; Executor manages how that work gets executed.**

That's a very useful mental model.

---

# 2333. How do you create a thread by extending `Thread`?

You create a subclass of `Thread` and override `run()`.

```java
class MyThread extends Thread {

    @Override
    public void run() {
        System.out.println(
            "Executing in: " +
            Thread.currentThread().getName()
        );
    }
}
```

Then:

```java
public class Main {

    public static void main(String[] args) {

        MyThread thread = new MyThread();

        thread.start();
    }
}
```

### What happens?

```text
main()
  |
  ↓
new MyThread()
  |
  ↓
thread.start()
  |
  ↓
JVM creates/schedules execution
  |
  ↓
run() executes
```

### Important

You don't normally call:

```java
thread.run();
```

if your intention is to start a new thread.

We'll see why shortly.

---

# 2334. How do you create a thread using `Runnable`?

You implement `Runnable`.

```java
class MyTask implements Runnable {

    @Override
    public void run() {
        System.out.println("Task running");
    }
}
```

Then pass the task to a `Thread`.

```java
Runnable task = new MyTask();

Thread thread = new Thread(task);

thread.start();
```

Or more commonly:

```java
Thread thread = new Thread(() -> {
    System.out.println("Task running");
});

thread.start();
```

### Architecture

Notice the separation:

```text
Runnable
   ↓
represents WHAT to execute

Thread
   ↓
represents execution mechanism
```

This separation becomes even more important when using executors:

```text
Runnable / Callable
       ↓
ExecutorService
       ↓
Thread Pool
       ↓
Worker Thread
```

---

# 2335. What is the difference between extending `Thread` and implementing `Runnable`?

This is a **very common interview question**.

### Extending `Thread`

```java
class MyThread extends Thread {
    @Override
    public void run() {
    }
}
```

Your class **is a Thread**.

### Implementing `Runnable`

```java
class MyTask implements Runnable {
    @Override
    public void run() {
    }
}
```

Your class **represents a task that can be executed by a thread**.

### Key difference

Java supports **single inheritance**.

If you do:

```java
class MyTask extends Thread
```

you cannot extend another class.

But:

```java
class MyTask extends SomeBusinessClass
        implements Runnable
```

is perfectly valid.

### Comparison

| `extends Thread`                      | `implements Runnable`         |
| ------------------------------------- | ----------------------------- |
| Class becomes a Thread                | Class represents a task       |
| Uses inheritance                      | Uses composition              |
| Can't extend another class            | Can extend another class      |
| Couples task with execution mechanism | Separates task from execution |
| Less flexible                         | More flexible                 |

---

# 2336. Why is implementing `Runnable` generally preferred over extending `Thread`?

There are several reasons.

### 1. Java only supports single class inheritance

Suppose:

```java
class PaymentTask extends Thread
```

You can no longer do:

```java
class PaymentTask extends PaymentService
```

because Java doesn't allow multiple class inheritance.

With `Runnable`:

```java
class PaymentTask extends PaymentService
                  implements Runnable
```

you can do both.

---

### 2. Separation of responsibility

`Runnable` represents:

> "What work should be done?"

`Thread` represents:

> "What executes the work?"

That's cleaner design.

---

### 3. Works naturally with executors

```java
executor.submit(task);
```

You don't need to care which thread executes it.

---

### 4. Thread reuse

With raw `Thread`:

```text
Task
 ↓
Thread
 ↓
finishes
```

A `Thread` generally isn't reused after termination.

With an executor:

```text
Task A ─┐
Task B ─┼→ Thread Pool
Task C ─┘
```

Worker threads can execute many tasks.

### Interview-ready answer

> "Runnable is generally preferred because it separates the task from the execution mechanism, avoids consuming the single inheritance slot, and works naturally with ExecutorService and thread pools."

---

# 2337. Can a Java class extend `Thread` and another class simultaneously?

**No.**

Java doesn't support multiple class inheritance.

This is invalid:

```java
class MyClass extends Thread, SomeClass {
}
```

However, you can implement multiple interfaces:

```java
class MyClass extends SomeClass
        implements Runnable, Serializable {
}
```

So:

```text
One superclass
+
Multiple interfaces
```

---

# 2338. Can the same `Runnable` object be submitted to multiple threads?

**Yes.**

Example:

```java
Runnable task = () -> {
    System.out.println(
        Thread.currentThread().getName()
    );
};

Thread t1 = new Thread(task);
Thread t2 = new Thread(task);

t1.start();
t2.start();
```

Both threads execute the same `Runnable` object's `run()` method.

```text
             Runnable object
              /           \
             ↓             ↓
         Thread 1       Thread 2
             ↓             ↓
          run()           run()
```

### But here's the important part

If the `Runnable` contains mutable instance state:

```java
class Task implements Runnable {

    private int count;

    @Override
    public void run() {
        count++;
    }
}
```

and the same object is shared between threads, then `count` becomes shared mutable state.

You may now have a race condition.

So:

> Sharing a Runnable is allowed, but its state must be designed for concurrent access.

---

# 2339. What happens if the same `Thread` object is started twice?

Example:

```java
Thread t = new Thread(() -> {
    System.out.println("Hello");
});

t.start();
t.start();
```

The second `start()` is invalid.

A `Thread` can only be started **once**.

---

# 2340. What exception is thrown when a thread is started more than once?

Java throws:

```text
IllegalThreadStateException
```

Example:

```java
Thread t = new Thread(() -> {
    System.out.println("Hello");
});

t.start();

t.start(); // IllegalThreadStateException
```

### Why?

A `Thread` has a lifecycle.

Once it has been started, you cannot transition that same `Thread` object back into a new execution.

Even after termination:

```text
NEW
 ↓
RUNNABLE
 ↓
TERMINATED
```

you cannot do:

```java
t.start();
```

again.

---

# 2341. What is the difference between `start()` and `run()`?

🔥 **Extremely important.**

### `start()`

Starts a new thread of execution.

```java
Thread t = new Thread(() -> {
    System.out.println("Worker");
});

t.start();
```

The JVM/underlying runtime arranges for the new thread to execute `run()`.

Conceptually:

```text
main thread
     |
     | start()
     ↓
new thread
     |
     ↓
run()
```

---

### `run()`

`run()` is just a method.

If you call:

```java
t.run();
```

you are simply invoking the method from the **current thread**.

No new thread is created.

```text
main thread
     |
     | run()
     ↓
run() executes
     ↓
still main thread
```

### Example

```java
Thread t = new Thread(() -> {
    System.out.println(
        Thread.currentThread().getName()
    );
});

t.run();
```

Output could be:

```text
main
```

But:

```java
t.start();
```

could print something like:

```text
Thread-0
```

### ⭐ Memorize this

> **`start()` creates/schedules new concurrent execution; `run()` contains the task logic and calling it directly does not create a new thread.**

---

# 2342. What happens internally when `start()` is called?

At a conceptual level:

```java
thread.start();
```

causes the JVM to request that the thread begin execution.

The important sequence is:

```text
Thread object
     ↓
start()
     ↓
Thread transitions from NEW
     ↓
JVM/runtime arranges execution
     ↓
OS schedules platform thread
     ↓
run() is invoked
```

For a traditional platform thread, the JVM works with an OS thread that the operating system schedules.

The exact internal implementation details are JVM/platform dependent, so don't overstate a particular native implementation in an interview.

### Important rule

`start()` must only be called once on a particular `Thread` object.

---

# 2343. What happens if you directly call `run()`?

Example:

```java
Thread t = new Thread(() -> {
    System.out.println("Worker");
});

t.run();
```

`run()` executes like an ordinary method call.

No new thread is created.

If called from `main()`:

```text
main thread
    |
    ↓
t.run()
    |
    ↓
task executes
    |
    ↓
main thread continues
```

Therefore:

```java
t.run();
```

does **not** give you multithreading.

---

# 2344. Does calling `run()` create a new thread?

**No.**

This is one of the most common beginner mistakes.

```java
Thread t = new Thread(task);

t.run();
```

means:

> "Call the `run()` method."

Whereas:

```java
t.start();
```

means:

> "Start the thread so that its `run()` method can execute in that thread."

---

# 2345. Can `run()` be called manually?

**Yes.**

There's nothing preventing you from doing:

```java
t.run();
```

But it behaves as a normal method call.

It doesn't start the thread.

For example:

```java
Thread t = new Thread(() -> {
    System.out.println(Thread.currentThread().getName());
});

t.run();
```

The task runs in whichever thread called `run()`.

### Interview trap

**Q:** "Can `run()` be called manually?"

**A:** Yes.

**Q:** "Will it create another thread?"

**A:** No.

---

# 2346. Can `start()` be overridden?

**Technically, yes**, because `Thread.start()` is not final.

But **you generally should not override it**.

Why?

Because `start()` has special semantics associated with starting the thread. Overriding it can break the expected lifecycle behavior.

Example:

```java
class MyThread extends Thread {

    @Override
    public void start() {
        System.out.println("Custom start");
    }

    @Override
    public void run() {
        System.out.println("Run");
    }
}
```

Now:

```java
new MyThread().start();
```

may only execute your custom method and never actually start the thread.

### Interview takeaway

> `start()` can be overridden, but overriding it is generally a bad design choice. `run()` is the method intended to contain the thread's work.

---

# 2347. Can `run()` be overloaded?

**Yes.**

Remember:

### Override

Same signature:

```java
@Override
public void run() {
}
```

### Overload

Different parameters:

```java
public void run() {
}

public void run(String name) {
}
```

Both are legal.

---

# 2348. What happens if you overload `run()` instead of overriding it?

This is a classic trick.

Suppose:

```java
class MyThread extends Thread {

    public void run(String name) {
        System.out.println(name);
    }
}
```

You did **not override**:

```java
Thread.run()
```

because the signature is different.

Now:

```java
MyThread t = new MyThread();
t.start();
```

What happens?

The JVM invokes the inherited `run()` method.

Since you haven't overridden `Thread.run()`, your:

```java
run(String name)
```

isn't automatically called.

### Important

The thread mechanism looks for:

```java
run()
```

not:

```java
run(String)
```

So if you want your thread logic to execute, override:

```java
@Override
public void run()
```

---

# 2349. Can a thread be started from inside another thread?

**Yes.**

There is nothing special about the main thread in this regard.

Example:

```java
Thread child = new Thread(() -> {
    System.out.println("Child");
});

Thread parent = new Thread(() -> {
    child.start();
});

parent.start();
```

Conceptually:

```text
Main
 ↓
Parent Thread
 ↓
Child Thread
```

A thread can create and start another thread.

---

# 2350. Can a thread start another thread?

**Yes.**

Same concept as above.

A thread is allowed to create and start other threads, provided it has the necessary permissions/resources.

There isn't a rule saying:

> Only `main()` can start threads.

---

# 2351. What happens if an exception escapes from a thread's `run()` method?

Suppose:

```java
Thread t = new Thread(() -> {
    throw new RuntimeException("Something went wrong");
});

t.start();
```

If the exception escapes the thread's `run()` method:

1. The thread terminates.
2. The uncaught exception handling mechanism is invoked.
3. The exception can be reported/logged.
4. The thread does not continue executing after the uncaught exception.

Typically you'll see something like:

```text
Exception in thread "Thread-0"
java.lang.RuntimeException: Something went wrong
```

### Important

The exception belongs to that thread's execution.

It doesn't automatically get thrown into the thread that called `start()`.

---

# 2352. Does an exception in one thread terminate the entire JVM?

**Generally, no.**

Suppose:

```text
Thread A → exception → terminates

Thread B → continues
Thread C → continues
Main    → continues
```

An uncaught exception normally terminates **that thread**, not the entire JVM.

However, if the failed thread was the last remaining **non-daemon thread**, the JVM may subsequently terminate because there are no non-daemon threads left.

So distinguish:

```text
Uncaught exception
        ↓
terminates that thread
        ↓
Does JVM terminate?
        ↓
depends on whether other non-daemon threads remain
```

---

# 2353. How can you handle uncaught exceptions in a thread?

You can use:

```java
Thread.UncaughtExceptionHandler
```

Example:

```java
Thread thread = new Thread(() -> {
    throw new RuntimeException("Failure");
});

thread.setUncaughtExceptionHandler(
    (t, e) -> {
        System.out.println(
            "Thread: " + t.getName()
        );
        System.out.println(
            "Exception: " + e.getMessage()
        );
    }
);

thread.start();
```

This handler gets invoked when an exception escapes the thread's `run()` method.

---

# 2354. What is `UncaughtExceptionHandler`?

It's an interface used to handle an exception that **escapes a thread's `run()` method** and wasn't otherwise caught.

Signature:

```java
public interface UncaughtExceptionHandler {

    void uncaughtException(
        Thread t,
        Throwable e
    );
}
```

You can configure it:

```java
thread.setUncaughtExceptionHandler(
    (t, e) -> {
        // logging / handling
    }
);
```

### Why useful?

In production systems, it can help with:

* Logging
* Alerting
* Diagnostics
* Capturing thread name
* Recording unexpected failures

### Important limitation

It is for **uncaught exceptions**.

It doesn't replace normal exception handling inside the task.

---

# 2355. What is the difference between setting an uncaught exception handler on a thread and on the `Thread` class?

There are two levels.

### Thread-specific handler

```java
thread.setUncaughtExceptionHandler(handler);
```

This applies only to that particular thread.

### Default handler

```java
Thread.setDefaultUncaughtExceptionHandler(handler);
```

This acts as the default handler for threads that don't have their own specific handler.

Conceptually:

```text
Specific Thread Handler
        ↓
used first

Default Handler
        ↓
fallback
```

---

# 2356. What happens if both a thread-level and default uncaught exception handler are configured?

The **thread-specific handler takes precedence**.

For example:

```java
Thread.setDefaultUncaughtExceptionHandler(
    (t, e) -> System.out.println("DEFAULT")
);

Thread thread = new Thread(() -> {
    throw new RuntimeException();
});

thread.setUncaughtExceptionHandler(
    (t, e) -> System.out.println("THREAD")
);

thread.start();
```

The thread-specific handler is used:

```text
THREAD
```

not:

```text
DEFAULT
```

### Resolution order

Think:

```text
Exception escapes run()
        ↓
Does thread have handler?
       / \
     YES  NO
      ↓    ↓
 Thread   Default
 Handler  Handler
```

If neither is available, the JVM's default uncaught-exception handling behavior is used.

---

# 🔥 Section B — Most Important Interview Traps

These are the ones I would **definitely practice verbally**:

### Trap 1

**Q: `start()` vs `run()`?**

```text
start()
→ starts new thread execution
→ eventually invokes run()

run()
→ normal method call
→ no new thread
```

---

### Trap 2

**Q: Can the same Thread object be started twice?**

```text
NO
↓
IllegalThreadStateException
```

---

### Trap 3

**Q: Can the same Runnable be used by multiple threads?**

```text
YES
```

But shared mutable state inside that Runnable must be protected.

---

### Trap 4

**Q: Can run() be overloaded?**

```text
YES
```

But only:

```java
run()
```

is the method associated with the `Thread` execution mechanism.

---

### Trap 5

**Q: Does an uncaught exception in one thread kill the JVM?**

Generally:

```text
No
↓
that thread terminates
```

If it was the last non-daemon thread, the JVM can then shut down.

---

### Trap 6

**Q: Which uncaught exception handler wins?**

```text
Thread-specific handler
        ↓
      wins
        ↓
Default handler
```

---

## 🧠 One mental model to retain

Don't think of `Thread` and `Runnable` as the same thing.

Think:

```text
             WORK
              │
       Runnable / Callable
              │
              ↓
        HOW TO EXECUTE?
              │
              ↓
       ExecutorService
              │
              ↓
        Thread Pool
              │
              ↓
           Threads
```

For low-level learning:

```text
Runnable
   ↓
Thread
   ↓
start()
   ↓
new execution
   ↓
run()
```

That distinction will make **ExecutorService, ThreadPoolExecutor, Callable, Future and CompletableFuture** much easier later.

**Next: Section C — Thread Lifecycle and Thread States (2357–2381)**. This is especially important because interviewers love drilling into **NEW → RUNNABLE → BLOCKED / WAITING / TIMED_WAITING → TERMINATED**, and asking you to distinguish **BLOCKED vs WAITING**.
