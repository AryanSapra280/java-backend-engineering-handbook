# Java Concurrency — 09: Callable & Future

> `Callable` and `Future` build on `ExecutorService` and thread pools. They allow us to submit work that produces a result and retrieve that result later.

---

## 1. Runnable vs Callable

### Runnable

```java
Runnable task = () -> {
    System.out.println("Processing...");
};
```

A `Runnable` represents a task that does not return a result.

Its method is:

```java
void run();
```

### Callable

```java
Callable<Integer> task = () -> {
    return 10 + 20;
};
```

Its method is:

```java
T call() throws Exception;
```

Mental model:

```text
Runnable
   ↓
run()
   ↓
no return value

Callable<T>
   ↓
call()
   ↓
returns T
```

---

## 2. Callable with ExecutorService

```java
ExecutorService executor =
        Executors.newFixedThreadPool(2);

Callable<Integer> task = () -> {

    System.out.println("Calculating...");

    return 10 + 20;
};

Future<Integer> future =
        executor.submit(task);
```

`submit()` returns a `Future`.

Think of `Future` as:

> "I don't have the result yet, but I have a handle through which I can retrieve it later."

---

## 3. Getting the Result

```java
Integer result = future.get();

System.out.println(result);
```

Output:

```text
30
```

But `get()` has an important property: **it can block**.

Suppose the task takes 10 seconds:

```java
Callable<Integer> task = () -> {

    Thread.sleep(10000);

    return 30;
};
```

Then:

```java
Future<Integer> future =
        executor.submit(task);

System.out.println("Before get");

Integer result = future.get();

System.out.println("After get");
```

Flow:

```text
submit()
   ↓
task starts asynchronously
   ↓
"Before get"
   ↓
future.get()
   ↓
WAIT
   ↓
10 seconds
   ↓
result available
   ↓
"After get"
```

Therefore:

```text
submit()
→ asynchronous execution relative to caller

future.get()
→ blocking result retrieval if result isn't ready
```

---

## 4. The First Big Limitation of Future

Consider:

```java
Future<Integer> future =
        executor.submit(task);

Integer result =
        future.get();

doSomething(result);
```

Although the task was submitted to another thread, the current thread waits at:

```java
future.get();
```

Conceptually:

```text
Caller Thread
      |
      | submit()
      ↓
Worker Thread
      |
      | performs task
      ↓
    result

Caller Thread
      |
      | future.get()
      ↓
    WAITING
```

This isn't necessarily bad. Sometimes you need the result.

The problem is that `Future` makes composition of multiple asynchronous operations cumbersome.

---

## 5. `isDone()`

You can check whether the task has completed:

```java
if (future.isDone()) {
    System.out.println("Task completed");
}
```

Important:

```text
isDone()
→ does NOT wait
```

It simply checks the current state.

Example:

```java
Future<Integer> future =
        executor.submit(() -> {

            Thread.sleep(5000);

            return 100;
        });

System.out.println(future.isDone());

Thread.sleep(6000);

System.out.println(future.isDone());
```

Likely:

```text
false
true
```

---

## 6. `isCancelled()`

You can check whether the task was cancelled:

```java
future.isCancelled();
```

Example:

```java
if (future.isCancelled()) {
    System.out.println("Task was cancelled");
}
```

---

## 7. Cancelling a Future

You can call:

```java
future.cancel(true);
```

The `true` means:

> If the task is already running, attempt to interrupt its executing thread.

Example:

```java
Future<Integer> future =
        executor.submit(() -> {

            while (true) {

                System.out.println("Working...");

                Thread.sleep(1000);
            }
        });

Thread.sleep(3000);

future.cancel(true);
```

The worker thread receives an interruption request.

Important:

> `interrupt()` is cooperative. It does not forcibly kill a thread.

---

## 8. `cancel(false)` vs `cancel(true)`

```java
future.cancel(false);
```

means:

> Don't interrupt the task if it is already running.

```java
future.cancel(true);
```

means:

> Attempt to interrupt the running task.

Mental model:

```text
cancel(false)
→ don't interrupt running task

cancel(true)
→ request interruption
```

---

## 9. Important Interview Trap

### Does `future.cancel(true)` kill the thread?

**No.**

It requests interruption.

The task needs to respond appropriately.

For example:

```java
while (!Thread.currentThread().isInterrupted()) {
    process();
}
```

Or code may respond to:

```java
InterruptedException
```

appropriately.

---

## 10. Timeout with `get()`

You can specify a maximum waiting time:

```java
future.get(
    2,
    TimeUnit.SECONDS
);
```

Meaning:

> Wait at most 2 seconds for the result.

Example:

```java
try {

    Integer result =
            future.get(
                2,
                TimeUnit.SECONDS
            );

} catch (TimeoutException e) {

    System.out.println(
        "Task took too long"
    );
}
```

This is different from:

```java
future.get();
```

which can wait indefinitely.

---

## 11. Exceptions in Callable

Suppose:

```java
Callable<Integer> task = () -> {

    throw new RuntimeException(
        "Something failed"
    );
};
```

Submit:

```java
Future<Integer> future =
        executor.submit(task);
```

When you call:

```java
future.get();
```

the failure is reported through:

```java
ExecutionException
```

Example:

```java
try {

    Integer result = future.get();

} catch (ExecutionException e) {

    System.out.println(
        "Task failed: " +
        e.getCause()
    );
}
```

Mental model:

```text
Worker Thread
      ↓
exception
      ↓
Future
      ↓
future.get()
      ↓
ExecutionException
```

---

## 12. Common Future Exceptions

When working with:

```java
future.get();
```

you commonly deal with:

```text
InterruptedException
ExecutionException
TimeoutException
```

Example:

```java
try {

    Integer result =
            future.get(
                2,
                TimeUnit.SECONDS
            );

} catch (InterruptedException e) {

    Thread.currentThread().interrupt();

} catch (ExecutionException e) {

    System.out.println(
        "Task failed: " +
        e.getCause()
    );

} catch (TimeoutException e) {

    System.out.println(
        "Task timed out"
    );
}
```

---

## 13. Future Doesn't Give You Great Composition

Suppose you need:

```text
Get customer
     ↓
Get account
     ↓
Get balance
     ↓
Calculate result
```

With `Future`, you may end up doing:

```java
Future<Customer> customerFuture =
        executor.submit(this::getCustomer);

Future<Account> accountFuture =
        executor.submit(this::getAccount);

Customer customer =
        customerFuture.get();

Account account =
        accountFuture.get();

Future<Balance> balanceFuture =
        executor.submit(
            () -> getBalance(account)
        );

Balance balance =
        balanceFuture.get();
```

This involves explicit coordination and potentially lots of blocking.

---

## 14. Combining Futures

Suppose:

```text
Service A → result A
Service B → result B
```

and you want:

```text
A + B
 ↓
final result
```

With `Future`, you generally have to manually:

```text
submit A
submit B
get A
get B
combine
```

There is no natural fluent composition model.

This is one reason `CompletableFuture` is important.

---

## 15. Future Mental Model

Remember:

```text
submit()
   ↓
task executes asynchronously
   ↓
Future<T>
   ↓
represents eventual result
```

Then:

```java
future.get();
```

means:

> "Give me the result. If it isn't ready, I'll wait."

Therefore:

```text
Future
→ asynchronous task handle

future.get()
→ blocking retrieval
```

---

## 16. Practical Example

```java
import java.util.concurrent.*;

public class Main {

    public static void main(String[] args)
            throws Exception {

        ExecutorService executor =
                Executors.newFixedThreadPool(2);

        Callable<Integer> task = () -> {

            System.out.println(
                "Task running on: " +
                Thread.currentThread().getName()
            );

            Thread.sleep(3000);

            return 100;
        };

        Future<Integer> future =
                executor.submit(task);

        System.out.println("Task submitted");

        System.out.println(
            "Is done? " +
            future.isDone()
        );

        Integer result =
                future.get();

        System.out.println(
            "Result = " + result
        );

        executor.shutdown();
    }
}
```

Expected conceptual flow:

```text
Task submitted
Is done? false
       ↓
3 seconds
       ↓
Result = 100
```

---

## 17. MUFG Interview Question

### What is the difference between Runnable and Callable?

Strong answer:

> "`Runnable` represents a task that doesn't return a result and its `run()` method returns void. `Callable<T>` returns a result of type T and its `call()` method can throw checked exceptions. Callable is typically submitted through ExecutorService and returns a Future representing the eventual result."

---

## 18. MUFG Interview Question

### Is `ExecutorService.submit()` synchronous or asynchronous?

Strong answer:

> "The task execution is asynchronous relative to the submitting thread. `submit()` returns a Future while the task executes on a worker thread. However, calling `future.get()` can block the submitting thread until the result is available."

---

## 19. MUFG Interview Question

### What are the limitations of Future?

Strong answer:

> "`Future` lets us represent and retrieve the result of an asynchronous computation, but its composition capabilities are limited. We often end up using blocking `get()` calls to coordinate dependent operations, and combining multiple asynchronous operations or handling asynchronous exceptions is cumbersome. `CompletableFuture` addresses these limitations with non-blocking composition and callback-style APIs."

---

## 20. Important Distinction

Do not confuse:

```text
Asynchronous execution
```

with:

```text
Non-blocking result handling
```

For example:

```java
Future<Integer> future =
        executor.submit(task);
```

The task can execute asynchronously.

But:

```java
future.get();
```

can block the current thread.

Therefore:

```text
submit()
→ asynchronous execution

get()
→ potentially blocking
```

This distinction becomes central to `CompletableFuture`.

---

## 21. The Big Picture

We've now reached:

```text
Thread
   ↓
ExecutorService
   ↓
Thread Pool
   ↓
submit()
   ↓
Callable
   ↓
Future<T>
   ↓
future.get()
   ↓
BLOCKING
```

And now:

```text
Future
   ↓
limitations
   ↓
CompletableFuture
```

---

## 22. Next Topic — CompletableFuture

We'll start from the basics:

```java
CompletableFuture
```

and understand:

```text
runAsync()
supplyAsync()
      ↓
thenApply()
thenAccept()
thenRun()
      ↓
thenCompose()
thenCombine()
      ↓
exceptionally()
handle()
whenComplete()
      ↓
allOf()
anyOf()
      ↓
custom Executor
      ↓
async vs blocking
```

Then we'll build a realistic backend example:

```text
                    Request
                       ↓
              Transaction Service
                 /     |                      /      |                      ↓       ↓        ↓
          Account    Balance   Interest
           Service    Service   Service
               \       |       /
                \      |      /
                 ↓     ↓     ↓
                 Combine Results
                       ↓
                  Final Response
```

The key question:

> How can I start multiple independent operations concurrently and then combine their results without manually blocking on each Future?

That's where `CompletableFuture` becomes powerful.

---

## 23. Interview Checkpoint

Make sure you can explain:

1. What is `Callable`?
2. `Runnable` vs `Callable`?
3. What does `ExecutorService.submit()` return?
4. What is a `Future`?
5. Does `submit()` block?
6. Does `future.get()` block?
7. What does `isDone()` do?
8. What does `cancel(true)` do?
9. Does `cancel(true)` kill a thread?
10. `cancel(false)` vs `cancel(true)`?
11. How do you retrieve a Future result with a timeout?
12. What happens if the Callable throws an exception?
13. What is `ExecutionException`?
14. What are the limitations of Future?
15. Why is `CompletableFuture` better for composition?

---

## 24. Final Mental Model

```text
Runnable
   ↓
run()
   ↓
no result

Callable<T>
   ↓
call()
   ↓
returns T

ExecutorService
   ↓
submit(Callable)
   ↓
Future<T>
   ↓
eventual result
```

And:

```text
Future
   ↓
get()
   ↓
wait if necessary
   ↓
result
```

Therefore:

```text
Future
→ represents an asynchronous computation

get()
→ retrieves its result and may block

CompletableFuture
→ asynchronous computation + composition
```

Next:

# CompletableFuture — Part 1
