# CompletableFuture — EPAM Interview Level 🔥

This is one of the **highest-value remaining Java topics** for your interview because it combines concurrency, async programming, exception handling, thread pools, and your Spring Boot work.

We'll keep it practical.

---

# 1. Problem: `Future` is too restrictive

With `Future`:

```java
Future<FraudResult> fraud =
        executor.submit(() -> fraudService.check());

FraudResult result = fraud.get();
```

The problem is that `get()` blocks.

And if you have multiple async operations:

```text
Fraud Service
Account Service
Notification Service
```

you start getting:

```java
get()
get()
get()
try/catch
submit()
get()
```

It's difficult to compose.

### Answer

`CompletableFuture` allows us to build an **asynchronous computation pipeline**.

Think:

```text
id="j6xk7n"
Task
 ↓
transform
 ↓
transform
 ↓
combine
 ↓
handle error
 ↓
result
```

---

# 2. `supplyAsync()` vs `runAsync()`

### Problem

What if I want to execute something asynchronously?

If it **returns a value**:

```java
CompletableFuture<Integer> future =
        CompletableFuture.supplyAsync(() -> {
            return 100;
        });
```

If it **doesn't return a value**:

```java
CompletableFuture<Void> future =
        CompletableFuture.runAsync(() -> {
            sendNotification();
        });
```

### Remember

```text
supplyAsync → produces result
runAsync    → no result
```

---

# 3. Where does the task execute?

This is an important interview follow-up.

If you do:

```java
CompletableFuture.supplyAsync(() -> process());
```

without specifying an executor, it normally uses:

```text
ForkJoinPool.commonPool()
```

So:

```text
CompletableFuture
       ↓
common ForkJoinPool
       ↓
worker thread
```

### But in production

You often want:

```java
Executor executor = ...;

CompletableFuture.supplyAsync(
    () -> process(),
    executor
);
```

Now you control the thread pool.

---

# 4. Problem: Two downstream calls are independent

Imagine your Payment Service needs:

```text
Fraud Service
Account Service
```

Neither depends on the other.

Sequential:

```text
Fraud → 500ms
   ↓
Account → 400ms

Total ≈ 900ms
```

With CompletableFuture:

```text
          ┌── Fraud   500ms ──┐
Payment ──┤                   ├── combine
          └── Account 400ms ──┘
```

Total is approximately:

```text
max(500, 400) = 500ms
```

ignoring overhead.

Code:

```java
CompletableFuture<FraudResult> fraud =
    CompletableFuture.supplyAsync(
        () -> fraudService.check(),
        executor
    );

CompletableFuture<AccountResult> account =
    CompletableFuture.supplyAsync(
        () -> accountService.check(),
        executor
    );
```

Now we need to combine them.

---

# 5. `thenApply()` — transform the result

Suppose:

```java
CompletableFuture<String> future =
    CompletableFuture.supplyAsync(() -> "Aryan");
```

You want:

```text
"Aryan"
   ↓
"Aryan Sapra"
```

Use:

```java
CompletableFuture<String> result =
    future.thenApply(name -> name + " Sapra");
```

### Mental model

```text
T
 ↓
Function<T,R>
 ↓
R
```

So:

```text
thenApply = map/transform
```

This is very similar conceptually to `Stream.map()`.

---

# 6. Problem: What if the next operation itself is asynchronous?

Suppose:

```text
Get customer
    ↓
Call recommendation service
```

And recommendation service itself returns:

```java
CompletableFuture<Recommendation>
```

You could accidentally create:

```text
CompletableFuture<CompletableFuture<Recommendation>>
```

That's ugly.

### Answer: `thenCompose()`

```java
CompletableFuture<Recommendation> result =
    customerFuture.thenCompose(
        customer -> recommendationService.get(customer)
    );
```

### Mental model

```text
thenApply
T → R
```

while:

```text
thenCompose
T → CompletableFuture<R>
```

And it **flattens** the nested Future.

---

# 7. Interview question: `thenApply` vs `thenCompose`

### Answer

> `thenApply` is used when the next operation returns a normal value. `thenCompose` is used when the next operation itself returns a CompletableFuture, allowing us to flatten the nested asynchronous result.

Example:

```java
thenApply(x -> transform(x))
```

versus:

```java
thenCompose(x -> asyncOperation(x))
```

This is a **must-know distinction**.

---

# 8. Problem: Two independent futures need to be combined

We have:

```java
CompletableFuture<FraudResult> fraud;
CompletableFuture<AccountResult> account;
```

We want:

```text
FraudResult + AccountResult
       ↓
PaymentDecision
```

Use:

```java
CompletableFuture<PaymentDecision> decision =
    fraud.thenCombine(
        account,
        (fraudResult, accountResult) ->
            createDecision(fraudResult, accountResult)
    );
```

### Mental model

```text
Future A ──┐
           ├── combine(A,B) → result
Future B ──┘
```

---

# 9. `thenCombine()` vs `thenCompose()`

This is another good interview question.

### `thenCompose`

Dependency:

```text
A → B
```

B needs A.

```text
Customer
   ↓
Recommendations
```

### `thenCombine`

Independent:

```text
A ──┐
    ├── C
B ──┘
```

Example:

```text
Fraud ──┐
        ├── Payment Decision
Account ─┘
```

🔥 Very important mental model.

---

# 10. `thenAccept()`

Suppose you don't need to return another value.

```java
future.thenAccept(result -> {
    System.out.println(result);
});
```

Conceptually:

```text
T
 ↓
Consumer<T>
 ↓
Void
```

So:

```text
thenApply  → transform and return
thenAccept → consume result
```

---

# 11. `thenRun()`

Suppose you don't care about the previous result at all.

```java
future.thenRun(() -> {
    log.info("Payment processing completed");
});
```

It doesn't receive the previous result.

Think:

```text
previous stage
      ↓
   completed
      ↓
run another action
```

So:

```text
thenApply  → needs result, produces result
thenAccept → needs result, produces nothing
thenRun    → doesn't need result, produces nothing
```

---

# 12. Problem: What happens if an async operation fails?

Suppose:

```java
CompletableFuture<Integer> future =
    CompletableFuture.supplyAsync(() -> {
        throw new RuntimeException("Payment failed");
    });
```

We need error handling.

This is where:

```text
exceptionally
handle
whenComplete
```

become important.

---

# 13. `exceptionally()`

Use it to provide a fallback when the pipeline fails.

```java
CompletableFuture<Integer> result =
    future.exceptionally(ex -> {
        log.error("Payment failed", ex);
        return 0;
    });
```

Conceptually:

```text
success → normal result

failure
   ↓
exceptionally
   ↓
fallback result
```

The important point:

`exceptionally()` can **recover** by producing a replacement result.

---

# 14. `handle()`

`handle()` receives both:

```text
result
exception
```

Example:

```java
CompletableFuture<Integer> result =
    future.handle((value, ex) -> {

        if (ex != null) {
            return 0;
        }

        return value;
    });
```

So:

```text
handle
 ↓
success → value + null
failure → null + exception
```

### Difference

`exceptionally()`:

> Handle failure.

`handle()`:

> Handle both success and failure and produce a new result.

---

# 15. `whenComplete()`

This is usually used for **observation/side effects**, not recovery.

```java
future.whenComplete((result, ex) -> {

    if (ex != null) {
        log.error("Failed", ex);
    } else {
        log.info("Success: {}", result);
    }
});
```

It doesn't normally transform the result.

Mental model:

```text
whenComplete
      ↓
observe completion
      ↓
logging / metrics / cleanup
```

---

# 16. The interview comparison

Memorize this:

| Method | Purpose |
|---|---|
| `exceptionally` | Recover from exception |
| `handle` | Handle success + failure and transform |
| `whenComplete` | Observe completion / side effects |

Example:

```text
exceptionally
→ "If it fails, give me a fallback."

handle
→ "Regardless of success/failure, let me create the next result."

whenComplete
→ "Tell me what happened; I'll log/measure it."
```

---

# 17. Problem: Wait for multiple futures

Suppose you have:

```text
Future 1
Future 2
Future 3
Future 4
```

and you want:

> "Continue only when all are completed."

Use:

```java
CompletableFuture.allOf(
    future1,
    future2,
    future3,
    future4
);
```

It returns:

```java
CompletableFuture<Void>
```

Notice: it doesn't directly give you a `List` of results.

You generally retrieve the individual results after completion.

---

# 18. `anyOf()`

Suppose you have:

```text
Service A
Service B
Service C
```

and:

> "I only need the first completed response."

Use:

```java
CompletableFuture<Object> result =
    CompletableFuture.anyOf(
        futureA,
        futureB,
        futureC
    );
```

Mental model:

```text
A ── 500ms ──┐
B ── 200ms ──┼──→ first completed
C ── 800ms ──┘
```

---

# 19. Problem: `get()` vs `join()`

This is **very likely to be asked**.

Both wait for completion.

### `get()`

```java
future.get();
```

throws checked exceptions such as:

```text
InterruptedException
ExecutionException
```

### `join()`

```java
future.join();
```

throws unchecked:

```text
CompletionException
```

So:

```text
get()
→ checked exception handling

join()
→ unchecked CompletionException
```

### Interview answer

> Both `get()` and `join()` wait for the CompletableFuture to complete. `get()` exposes checked exceptions such as InterruptedException and ExecutionException, while `join()` wraps failures in CompletionException and is often more convenient inside CompletableFuture pipelines.

---

# 20. Problem: Why did my CompletableFuture use the common pool?

You write:

```java
CompletableFuture.supplyAsync(
    () -> callService()
);
```

You didn't specify an executor.

So CompletableFuture normally uses:

```text
ForkJoinPool.commonPool()
```

### Why can this be dangerous?

Suppose:

```text
100 requests
 ↓
each calls blocking DB/HTTP operation
 ↓
common pool
```

You may occupy common-pool workers with blocking operations.

That's why for application-specific workloads, especially blocking I/O, you often define a dedicated executor:

```java
CompletableFuture.supplyAsync(
    () -> callService(),
    paymentExecutor
);
```

---

# 21. Your Payment Service scenario

Let's put everything together.

Suppose:

```text
Payment Service
     |
     +---- Fraud Service
     |
     +---- Account Service
```

Both are independent.

We can do:

```java
CompletableFuture<FraudResult> fraudFuture =
    CompletableFuture.supplyAsync(
        () -> fraudService.check(payment),
        paymentExecutor
    );

CompletableFuture<AccountResult> accountFuture =
    CompletableFuture.supplyAsync(
        () -> accountService.check(payment),
        paymentExecutor
    );

CompletableFuture<PaymentDecision> decision =
    fraudFuture.thenCombine(
        accountFuture,
        (fraud, account) ->
            createDecision(fraud, account)
    );
```

The flow becomes:

```text
                  Payment
                     |
              ┌──────┴──────┐
              ↓             ↓
           Fraud         Account
              ↓             ↓
              └──────┬──────┘
                     ↓
               thenCombine
                     ↓
             PaymentDecision
```

That's a **real production use case** for CompletableFuture.

---

# 22. Now add exception handling

Suppose Fraud Service fails.

We might do:

```java
CompletableFuture<FraudResult> fraudFuture =
    CompletableFuture.supplyAsync(
        () -> fraudService.check(payment),
        paymentExecutor
    ).exceptionally(ex -> {
        log.error("Fraud check failed", ex);
        return FraudResult.failed();
    });
```

Then:

```text
Fraud failure
    ↓
exceptionally
    ↓
fallback FraudResult
    ↓
thenCombine
```

But there's an important architectural question:

> **Should we actually fallback?**

For something security-sensitive like fraud detection, blindly converting failure into "fraud check passed" could be dangerous.

A better design might be:

```text
Fraud unavailable
      ↓
Payment cannot safely continue
      ↓
controlled failure / retry / circuit breaker
```

This is why **exception handling + resilience patterns + business semantics** must be considered together.

---

# 23. `thenApply` vs `thenApplyAsync`

Another interview question.

```java
future.thenApply(...)
```

versus:

```java
future.thenApplyAsync(...)
```

The `Async` variant schedules the continuation asynchronously, using the default async executor if you don't provide one, or the executor you explicitly provide.

You can do:

```java
future.thenApplyAsync(
    result -> transform(result),
    paymentExecutor
);
```

### Important

Don't memorize:

> "`thenApply` always runs on the same thread."

That's too simplistic.

The execution behavior depends on completion timing and executor mechanics. For interview purposes, remember:

- `thenApply` = non-`Async` continuation
- `thenApplyAsync` = asynchronous continuation, optionally on supplied executor
- explicitly supply your executor when you need predictable application-specific execution.

---

# 24. One EPAM scenario

### Interviewer:

> "You need to call two independent downstream services concurrently. How would you implement it?"

### Strong answer:

> "I'd use CompletableFuture with a dedicated executor. I'd start both calls independently using `supplyAsync`, because they're independent. Then I'd use `thenCombine` to combine their results. I'd define timeout and exception handling explicitly, and I wouldn't use the common ForkJoinPool blindly for blocking I/O. The executor would be bounded according to the workload and downstream capacity."

That's a **Senior Engineer answer**.

---

# 25. One more important production issue: timeout

Never think:

```text
async = automatically safe
```

If downstream takes forever:

```text
Payment
  ↓
CompletableFuture
  ↓
Downstream
  ↓
WAIT FOREVER
```

You need a timeout strategy.

Modern CompletableFuture supports:

```java
orTimeout(...)
```

and:

```java
completeOnTimeout(...)
```

Conceptually:

```text
orTimeout
→ fail if deadline exceeded

completeOnTimeout
→ provide fallback/default value
```

For your interview, understand the distinction; we don't need to go deeply into every API variation.

---

# 26. Final mental model

You should now see the entire progression:

```text
Thread
  ↓
ExecutorService
  ↓
Future
  ↓
Problem: blocking get()
  ↓
CompletableFuture
  ↓
supplyAsync
  ↓
thenApply / thenCompose
  ↓
thenCombine
  ↓
allOf / anyOf
  ↓
exceptionally / handle / whenComplete
  ↓
custom Executor
  ↓
timeout / cancellation
  ↓
Spring @Async
```

And the **most important distinctions** for EPAM:

```text
thenApply
→ T → R

thenCompose
→ T → CompletableFuture<R>
→ flatten

thenCombine
→ Future A + Future B
→ combine independent results

exceptionally
→ recover from failure

handle
→ success + failure → new result

whenComplete
→ observe completion

get
→ blocking + checked exceptions

join
→ blocking + CompletionException

cancel(true)
→ interruption request, not forced termination
```

Next, we should do **Spring concurrency: `@Async`, `TaskExecutor`, custom thread pools, how Spring request threads interact with async threads, transaction boundaries, exception propagation, and your `CompletableFuture` implementation**. That's the missing bridge between **Java concurrency theory and your actual Spring Boot experience**.