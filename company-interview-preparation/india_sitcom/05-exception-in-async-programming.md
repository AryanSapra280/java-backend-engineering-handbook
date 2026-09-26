# 8.27 `CompletableFuture` Exception Handling — `exceptionally()`

Perfect. Let's continue the theory **without changing your current code**. 👍

The next thing we need to understand is:

> **What happens when the PSP call throws an exception?**

---

## 1. Make our PSP fail

Currently:

```java
public PspResponse authorize(Payment payment) {

    System.out.println(
            "PSP call started on thread: "
                    + Thread.currentThread().getName()
    );

    try {
        Thread.sleep(2000);
    } catch (InterruptedException e) {
        Thread.currentThread().interrupt();
        throw new RuntimeException("PSP call interrupted", e);
    }

    System.out.println(
            "PSP call completed on thread: "
                    + Thread.currentThread().getName()
    );

    return new PspResponse(
            payment.getPaymentId(),
            "SUCCESS"
    );
}
```

Temporarily change it to:

```java
public PspResponse authorize(Payment payment) {

    System.out.println(
            "PSP call started on thread: "
                    + Thread.currentThread().getName()
    );

    throw new RuntimeException("PSP unavailable");
}
```

Now:

```java
CompletableFuture<PspResponse> future =
        CompletableFuture.supplyAsync(
                () -> pspClient.authorize(payment),
                paymentExecuter.paymentExecutor()
        );
```

The PSP throws:

```text
PSP unavailable
```

What happens to the `CompletableFuture`?

It becomes:

```text
COMPLETED EXCEPTIONALLY
```

It does **not** contain a `PspResponse`.

---

# 2. What happens when you call `join()`?

Your code currently has:

```java
PspResponse pspResponse = future.join();
```

Because the future completed exceptionally, `join()` throws:

```text
CompletionException
```

Conceptually:

```text
PSP
 ↓
RuntimeException
 ↓
CompletableFuture
 ↓
completed exceptionally
 ↓
join()
 ↓
CompletionException
```

This is an important point:

### `supplyAsync()` doesn't throw the exception directly to the calling thread.

The exception is captured by the `CompletableFuture`.

Then when you retrieve the result using:

```java
join()
```

the exception is propagated as a `CompletionException`.

---

# 3. Now introduce `exceptionally()`

Instead of:

```java
CompletableFuture<PspResponse> future =
        CompletableFuture.supplyAsync(
                () -> pspClient.authorize(payment),
                paymentExecuter.paymentExecutor()
        );
```

we can attach:

```java
.exceptionally(ex -> {
    System.out.println("PSP failed: " + ex.getMessage());

    return new PspResponse(
            payment.getPaymentId(),
            "FAILED"
    );
});
```

Complete example:

```java
CompletableFuture<PspResponse> future =
        CompletableFuture
                .supplyAsync(
                        () -> pspClient.authorize(payment),
                        paymentExecuter.paymentExecutor()
                )
                .exceptionally(ex -> {

                    System.out.println(
                            "PSP failed: " + ex.getMessage()
                    );

                    return new PspResponse(
                            payment.getPaymentId(),
                            "FAILED"
                    );
                });
```

Now the pipeline is:

```text
PSP call
   |
   | exception
   ↓
exceptionally()
   |
   ↓
PspResponse("FAILED")
```

So `exceptionally()` provides a **fallback result**.

---

# 4. Why does `exceptionally()` return `PspResponse`?

Because our original future is:

```java
CompletableFuture<PspResponse>
```

Therefore:

```java
.exceptionally(ex -> ...)
```

needs to produce a `PspResponse`.

For example:

```java
.exceptionally(ex -> {
    return new PspResponse(
        payment.getPaymentId(),
        "FAILED"
    );
});
```

The resulting future is still:

```text
CompletableFuture<PspResponse>
```

---

# 5. This is the important mental model

Think of:

```java
exceptionally()
```

as:

> **"If something went wrong upstream, recover by producing a fallback value."**

For example:

```text
Normal path:

PSP
 ↓
PspResponse
 ↓
SUCCESS
```

Failure path:

```text
PSP
 ↓
Exception
 ↓
exceptionally()
 ↓
Fallback PspResponse
 ↓
FAILED
```

---

# 6. But be careful with payment systems ⚠️

We shouldn't blindly do this:

```java
.exceptionally(ex ->
    new PspResponse(payment.getPaymentId(), "FAILED")
);
```

because:

> **PSP timeout ≠ payment definitely failed.**

Imagine:

```text
Our service
     |
     | payment request
     ↓
PSP
     |
     | Payment actually processed
     ↓
SUCCESS
```

But then the response gets lost:

```text
PSP SUCCESS
     X
     |
     ↓
Our service
```

Our service sees:

```text
timeout
```

If we immediately mark:

```text
FAILED
```

we could potentially have:

```text
Our DB: FAILED
PSP:     SUCCESS
```

🔥 That's a **real payment-system consistency problem**.

We'll come back to this when we cover:

* timeout
* retry
* reconciliation
* idempotency
* circuit breaker

This is exactly why we're building the project instead of just learning syntax.

---

# 7. `exceptionally()` only handles failures

Suppose:

```java
CompletableFuture<PspResponse> future =
        CompletableFuture
                .supplyAsync(
                        () -> pspClient.authorize(payment),
                        paymentExecuter.paymentExecutor()
                )
                .thenApply(response -> {

                    System.out.println(
                            "PSP status = " + response.status()
                    );

                    return response;
                })
                .exceptionally(ex -> {

                    System.out.println(
                            "Something went wrong: "
                                    + ex.getMessage()
                    );

                    return new PspResponse(
                            payment.getPaymentId(),
                            "FAILED"
                    );
                });
```

The flow is:

```text
supplyAsync()
      ↓
PSP
      ↓
thenApply()
      ↓
exceptionally()
```

If everything succeeds:

```text
PSP
 ↓
thenApply()
 ↓
result
```

`exceptionally()` is not invoked.

If PSP fails:

```text
PSP
 ↓
EXCEPTION
 ↓
thenApply() skipped
 ↓
exceptionally()
 ↓
fallback
```

---

# 8. Important interview question

### What happens to downstream stages after an exception?

Suppose:

```java
CompletableFuture
    .supplyAsync(...)
    .thenApply(...)
    .thenApply(...)
    .exceptionally(...);
```

If the first operation fails:

```text
supplyAsync()
      ↓
   EXCEPTION
      ↓
thenApply() ❌
      ↓
thenApply() ❌
      ↓
exceptionally() ✅
```

The exceptional state propagates down the chain until something handles it.

---

# 9. `exceptionally()` vs normal `try-catch`

This:

```java
try {
    PspResponse response = pspClient.authorize(payment);
} catch (Exception e) {
    // handle
}
```

handles a **synchronous** call.

But with:

```java
CompletableFuture.supplyAsync(...)
```

the operation happens asynchronously.

So we can attach asynchronous error handling:

```java
future.exceptionally(ex -> {
    // recovery
    return fallback;
});
```

That's one of the main reasons these methods exist.

---

# 10. One very important distinction

`exceptionally()` is primarily a **recovery mechanism**.

For example:

```java
.exceptionally(ex -> defaultResponse)
```

means:

> "If something failed, give me an alternative result."

But sometimes you don't want to recover.

You just want to **observe/log the exception and let the failure continue**.

That's where we'll use:

```java
whenComplete()
```

And if you want to transform both success and failure into another result:

```java
handle()
```

These three are worth knowing very clearly:

```text
exceptionally()
    ↓
Recover from failure

whenComplete()
    ↓
Observe success/failure

handle()
    ↓
Process success/failure and produce a new result
```

---

## Your current interview cheat sheet

| Method            | Purpose                                                             |
| ----------------- | ------------------------------------------------------------------- |
| `exceptionally()` | Handle/recover from an exception                                    |
| `handle()`        | Handle both success and failure and return a new result             |
| `whenComplete()`  | Perform an action after success/failure without changing the result |

### Example:

```java
.exceptionally(ex -> fallback)
```

```java
.handle((result, ex) -> newResult)
```

```java
.whenComplete((result, ex) -> logSomething)
```

---

# Next: `handle()` vs `whenComplete()` 🔥

This is where interviewers commonly test whether you actually understand `CompletableFuture` rather than just memorizing method names.

We'll use the **same payment/PSP example** and see exactly what happens to the returned `CompletableFuture` in each case.

# 8.28 `handle()` vs `whenComplete()`

Now we're at one of the **most useful `CompletableFuture` interview areas**.

We already know:

```text
exceptionally()
→ recover from failure
```

Now compare:

```text
handle()
whenComplete()
```

The easiest way to understand them is through our PSP call.

---

## 1. `handle()` — process success OR failure

Suppose:

```java
CompletableFuture<PspResponse> future =
        CompletableFuture.supplyAsync(
                () -> pspClient.authorize(payment),
                paymentExecuter.paymentExecutor()
        );
```

We can do:

```java
CompletableFuture<String> result =
        future.handle((response, exception) -> {

            if (exception != null) {
                return "PSP_FAILED";
            }

            return response.status();
        });
```

Notice that `handle()` receives **two parameters**:

```java
(response, exception)
```

So it can inspect both outcomes.

### Success

```text
PSP
 ↓
PspResponse
 ↓
handle()
 ↓
"SUCCESS"
```

### Failure

```text
PSP
 ↓
Exception
 ↓
handle()
 ↓
"PSP_FAILED"
```

---

# 2. Why does `handle()` return a new result?

Because `handle()` is a **transformation operation**.

Suppose:

```java
CompletableFuture<PspResponse>
```

and:

```java
.handle((response, exception) -> ...)
```

returns:

```java
CompletableFuture<String>
```

For example:

```java
CompletableFuture<String> result =
        future.handle((response, exception) -> {

            if (exception != null) {
                return "FAILED";
            }

            return response.status();
        });
```

So:

```text
CompletableFuture<PspResponse>
             ↓
          handle()
             ↓
CompletableFuture<String>
```

---

# 3. Compare `handle()` with `thenApply()`

This is useful.

### `thenApply()`

Primarily processes the **successful result**:

```java
future.thenApply(response -> response.status());
```

Conceptually:

```text
SUCCESS → transform
FAILURE → exception propagates
```

### `handle()`

Processes **both**:

```java
future.handle((response, exception) -> {

    if (exception != null) {
        return "FAILED";
    }

    return response.status();
});
```

Conceptually:

```text
SUCCESS → transform
FAILURE → transform/recover
```

---

# 4. Now `whenComplete()`

`whenComplete()` is different.

Suppose we want to log the outcome:

```java
CompletableFuture<PspResponse> result =
        future.whenComplete((response, exception) -> {

            if (exception != null) {
                log.error("PSP failed", exception);
            } else {
                log.info(
                        "PSP completed with status {}",
                        response.status()
                );
            }
        });
```

Here we're basically saying:

> "When this future finishes, let me perform some side effect."

For example:

* logging
* metrics
* auditing
* tracing
* cleanup

---

# 5. The critical difference

`whenComplete()` **doesn't normally transform the result**.

Suppose:

```java
CompletableFuture<PspResponse> future
```

Then:

```java
CompletableFuture<PspResponse> result =
        future.whenComplete(...);
```

The type remains:

```text
CompletableFuture<PspResponse>
```

Whereas:

```java
CompletableFuture<String> result =
        future.handle(...);
```

changes the result.

---

# 6. Payment example

Imagine PSP succeeds:

```text
id="payment-123"
status="SUCCESS"
```

### `whenComplete()`

```java
future.whenComplete((response, exception) -> {

    log.info(
        "Payment {} completed",
        response.paymentId()
    );

});
```

The original response remains:

```text
PspResponse
```

You're just observing it.

---

### `handle()`

```java
CompletableFuture<PaymentStatus> paymentStatus =
        future.handle((response, exception) -> {

            if (exception != null) {
                return PaymentStatus.FAILED;
            }

            return "SUCCESS".equals(response.status())
                    ? PaymentStatus.SUCCESS
                    : PaymentStatus.FAILED;
        });
```

Now you've **transformed the result** into:

```text
PaymentStatus
```

---

# 7. Very important: `whenComplete()` does NOT mean "handle the exception"

This is a common mistake.

You might write:

```java
future.whenComplete((response, exception) -> {

    if (exception != null) {
        log.error("PSP failed", exception);
    }
});
```

You logged the exception.

But unless another stage handles/recoveries from it, the future can still be **exceptionally completed**.

Think:

```text
PSP
 ↓
Exception
 ↓
whenComplete()
 ↓
log exception
 ↓
exception still exists
```

`whenComplete()` is primarily for **observation/side effects**, not recovery.

---

# 8. Compare all three now

This is the table I want you to remember:

| Method            | Success     | Failure         | Changes result?      |
| ----------------- | ----------- | --------------- | -------------------- |
| `exceptionally()` | Doesn't run | Handles/recover | Yes, fallback result |
| `handle()`        | Handles     | Handles         | Yes                  |
| `whenComplete()`  | Observes    | Observes        | No                   |

---

# 9. Simple mental model

Think about a restaurant order.

### `exceptionally()`

> "If the order fails, give me a replacement meal."

```text
Failure → replacement
```

### `handle()`

> "Whether the order succeeds or fails, decide what final status I should return."

```text
Success → status
Failure → status
```

### `whenComplete()`

> "Whatever happens, I want to record it in my logbook."

```text
Success → log
Failure → log
```

That's the easiest way to remember the difference.

---

# 10. Interview question

### Q: What's the difference between `handle()` and `whenComplete()`?

A strong answer:

> "`handle()` is used when I want to process both the successful result and the exception and transform them into another result. `whenComplete()` is mainly used for side effects such as logging, metrics or cleanup, while preserving the original result or exception."

That's enough for the first answer.

If they dig deeper, you can demonstrate:

```java
handle((result, exception) -> newResult)
```

versus:

```java
whenComplete((result, exception) -> logging)
```

---

# 11. Our payment flow so far

We now understand:

```text
                   CompletableFuture
                         │
                         ↓
                    supplyAsync()
                         │
                         ↓
                       PSP
                         │
                ┌────────┴────────┐
                │                 │
             SUCCESS           FAILURE
                │                 │
                ↓                 ↓
          thenApply()        exceptionally()
                │                 │
                ↓                 ↓
          transform          recover
```

And independently:

```text
PSP result
    ↓
whenComplete()
    ↓
logging / metrics / cleanup
```

while:

```text
PSP result
    ↓
handle()
    ↓
new transformed result
```

---

# 12. Next: `CompletableFuture` timeout 🔥

Now we're going to introduce a **very realistic payment failure**:

```text
Our Payment Service
       |
       ↓
      PSP
       |
       |................. 30 seconds
       |................. 60 seconds
       ↓
     ????
```

An external payment provider can be slow or unavailable.

We'll learn:

```java
orTimeout()
completeOnTimeout()
```

and, more importantly, the **payment-system question**:

> If the PSP times out, can we safely mark the payment `FAILED`?

The answer is **not necessarily**, and that's where timeout + idempotency + reconciliation start connecting into a real payment architecture.
