# Java Concurrency — CompletableFuture Fundamentals

## 1. Why CompletableFuture Exists

`Future` gives us a handle to an asynchronous computation, but retrieving results with `get()` can block and composing multiple asynchronous operations is cumbersome.

```text
ExecutorService
      ↓
submit()
      ↓
Callable
      ↓
Future<T>
```

`CompletableFuture` adds fluent composition:

```text
"When this operation completes,
 continue with this operation."
```

---

## 2. What Is CompletableFuture?

`CompletableFuture<T>` represents an asynchronous computation that eventually produces `T` or completes exceptionally.

```java
CompletableFuture<String> future =
        CompletableFuture.supplyAsync(() -> "Aryan");
```

It provides APIs for:

- chaining
- transforming
- combining
- exception handling
- manual completion

---

## 3. completedFuture()

Creates an already-completed future.

```java
CompletableFuture<Integer> future =
        CompletableFuture.completedFuture(100);
```

Useful when an API expects `CompletableFuture<T>` but you already have `T`.

---

## 4. runAsync()

Use when the task does not produce a result.

```java
CompletableFuture<Void> future =
        CompletableFuture.runAsync(() -> {
            System.out.println("Running asynchronously");
        });
```

Mental model:

```text
runAsync()
   ↓
perform work
   ↓
no result
   ↓
CompletableFuture<Void>
```

---

## 5. supplyAsync()

Use when the task produces a result.

```java
CompletableFuture<Integer> future =
        CompletableFuture.supplyAsync(() -> {
            return 10 + 20;
        });
```

Mental model:

```text
runAsync()   → no result
supplyAsync() → result
```

Useful shortcut:

```text
Runnable   ↔ runAsync()
Callable<T> ↔ supplyAsync()
```

This is a mental model, not exact API equivalence.

---

## 6. What Thread Executes supplyAsync()?

The asynchronous computation still needs a thread.

By default, async CompletableFuture methods use the default async execution facility, commonly the `ForkJoinPool.commonPool()`.

```text
CompletableFuture
      ↓
Executor
      ↓
Worker Thread
      ↓
Task
```

Therefore:

> Async does not mean "no thread."

---

## 7. Custom Executor

You can explicitly provide an executor.

```java
ExecutorService executor =
        Executors.newFixedThreadPool(5);

CompletableFuture<Integer> future =
        CompletableFuture.supplyAsync(
            () -> 100,
            executor
        );
```

Continuation:

```java
future.thenApplyAsync(
    value -> value * 2,
    executor
);
```

Custom executors can isolate workloads, for example request processing versus heavy computation.

---

## 8. get() and join()

You can retrieve a result using:

```java
future.get();
```

or:

```java
future.join();
```

Both can wait/block.

Important distinction:

```text
get()
→ checked exceptions such as
  InterruptedException, ExecutionException, TimeoutException

join()
→ exceptional completion is typically reported
  through CompletionException
```

Do not say CompletableFuture is "always non-blocking."

Better:

> CompletableFuture supports non-blocking composition, but `get()` and `join()` can still block.

---

## 9. thenApply()

`thenApply()` transforms the previous result.

```java
CompletableFuture<String> future =
        CompletableFuture.supplyAsync(() -> "Aryan");

CompletableFuture<Integer> length =
        future.thenApply(name -> name.length());
```

Type shape:

```text
CompletableFuture<A>
       ↓
   thenApply
       ↓
CompletableFuture<B>
```

where:

```text
A → B
```

Example:

```text
Account → AccountNumber
```

```java
accountFuture.thenApply(
    account -> account.getAccountNumber()
);
```

---

## 10. thenAccept()

Consumes the previous result without producing a meaningful new result.

```java
future.thenAccept(value -> {
    System.out.println(value);
});
```

Type shape:

```text
A → void
```

Mental model:

```text
thenApply  → transform
thenAccept → consume
```

---

## 11. thenRun()

Runs an action after completion without needing the previous result.

```java
future.thenRun(() -> {
    System.out.println("Done");
});
```

Mental model:

```text
thenRun
→ previous result is irrelevant
→ execute an action afterward
```

---

## 12. Compare the Three

```text
thenApply
→ receives result
→ transforms it
→ produces another result

thenAccept
→ receives result
→ consumes it
→ no meaningful result

thenRun
→ does not receive result
→ runs an action
→ no meaningful result
```

Examples:

```java
future.thenApply(x -> x * 2);
```

```java
future.thenAccept(x -> System.out.println(x));
```

```java
future.thenRun(() -> System.out.println("Done"));
```

---

## 13. thenApply() vs thenApplyAsync()

Do not say:

> thenApply() always runs on the caller thread.

The non-async continuation may execute in the thread that completes the previous stage.

```java
future.thenApply(value -> value * 2);
```

With:

```java
future.thenApplyAsync(value -> value * 2);
```

the continuation is scheduled asynchronously using the default async execution facility unless an executor is supplied.

Custom executor:

```java
future.thenApplyAsync(
    value -> value * 2,
    executor
);
```

Mental model:

```text
thenApply()
→ continuation may run in completing thread

thenApplyAsync()
→ schedule continuation asynchronously
```

---

## 14. Why Custom Executors Matter

Suppose:

```text
HTTP request work
      ↓
Executor A
```

and:

```text
Heavy computation
      ↓
Executor B
```

Using separate executors can prevent one workload from consuming resources needed by another.

Do not use `Async` everywhere automatically; additional scheduling and thread-pool contention can have costs.

---

## 15. Pipeline Mental Model

```java
CompletableFuture.supplyAsync(() -> getAccount())
    .thenApply(account -> account.getId())
    .thenAccept(id -> log(id));
```

Conceptually:

```text
Start async operation
        ↓
Account
        ↓
extract ID
        ↓
consume ID
```

This avoids manually calling `get()` between every stage.

---

## 16. CompletableFuture Does Not Automatically Remove Blocking

This:

```java
CompletableFuture<Integer> future =
        CompletableFuture.supplyAsync(() -> 100);
```

is asynchronous.

This:

```java
Integer result = future.get();
```

can block.

Therefore:

```text
CompletableFuture composition
→ can be non-blocking

get()/join()
→ blocking wait
```

---

## 17. Backend Example

```java
CompletableFuture<Account> accountFuture =
        getAccount(accountId);

CompletableFuture<String> accountNumber =
        accountFuture.thenApply(
            Account::getAccountNumber
        );

accountNumber.thenAccept(
    number -> audit(number)
);
```

Flow:

```text
getAccount()
     ↓
Future<Account>
     ↓
thenApply()
     ↓
Future<String>
     ↓
thenAccept()
     ↓
audit
```

---

## 18. MUFG Interview Questions

### What is CompletableFuture?

> `CompletableFuture` represents the eventual result of an asynchronous computation and provides APIs to compose, transform, combine, and handle exceptional completion of asynchronous operations.

### runAsync() vs supplyAsync()?

> `runAsync` is for asynchronous work without a result; `supplyAsync` is for asynchronous work that produces a result.

### thenApply() vs thenAccept()?

> `thenApply` transforms the previous result and returns another CompletableFuture; `thenAccept` consumes the result without producing a meaningful result.

### thenAccept() vs thenRun()?

> `thenAccept` receives the previous result; `thenRun` does not need the previous result and simply executes an action afterward.

### Does thenApply() always create a new thread?

> No. The non-async continuation may execute in the thread that completes the previous stage. `thenApplyAsync` schedules the continuation asynchronously.

### Is CompletableFuture non-blocking?

> It supports non-blocking composition, but operations such as `get()` and `join()` can block.

---

## 19. One Rule

```text
A → B
    ↓
thenApply

A → void
    ↓
thenAccept

() → void
    ↓
thenRun
```

---

## 20. Final Mental Model

```text
                    CompletableFuture
                           |
              ┌────────────┼────────────┐
              ↓            ↓            ↓
         runAsync()   supplyAsync()   completedFuture()
              ↓            ↓
          no result      result
                           |
                           ↓
                      thenApply()
                           ↓
                      transform
                           |
                           ↓
                      thenAccept()
                           ↓
                       consume
                           |
                           ↓
                       thenRun()
                           ↓
                        action
```

Key takeaway:

```text
Future:
"Give me the result."

CompletableFuture:
"When the result is ready,
 continue with this operation."
```

Next: `thenCompose()`, `thenCombine()`, exception handling, `allOf()`, `anyOf()`, timeouts, and a complete backend example.
