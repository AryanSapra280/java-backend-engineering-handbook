# 8.24 Next: `CompletableFuture` Composition

Great! Let's continue. 🔥

Now that you've **actually run `supplyAsync()` with your custom executor**, we'll learn how to build asynchronous pipelines without immediately calling `join()`.

We'll focus on these four first:

```text
supplyAsync()
    ↓
thenApply()
    ↓
thenAccept()
    ↓
thenCompose()
```

Then we'll handle:

```text
exceptionally()
handle()
whenComplete()
```

### First concept: `thenApply()`

Think of it as:

> **"When the previous async operation finishes, transform its result."**

Example:

```java
CompletableFuture<PspResponse> future =
        CompletableFuture.supplyAsync(
                () -> pspClient.authorize(payment),
                paymentExecuter.paymentExecutor()
        );

CompletableFuture<String> statusFuture =
        future.thenApply(response -> response.status());
```

Flow:

```text
PSP call
   ↓
PspResponse
   ↓
thenApply()
   ↓
String status
```

So:

```java
CompletableFuture<PspResponse>
```

becomes:

```java
CompletableFuture<String>
```

The important rule:

> **`thenApply()` is used when you receive a result and transform it into another result.**

We'll now apply that directly to your payment flow rather than using abstract examples.

# 8.25 `thenApply()` — Transforming the PSP Result

Let's connect this directly to **our payment service**.

Right now you have:

```java
CompletableFuture<PspResponse> future =
        CompletableFuture.supplyAsync(
                () -> pspClient.authorize(payment),
                paymentExecuter.paymentExecutor()
        );
```

The result type is:

```text id="h6y7d4"
CompletableFuture<PspResponse>
```

Meaning:

> "At some point, I'll have a `PspResponse`."

---

## 1. What if we only want the PSP status?

We can do:

```java id="y1q6rb"
CompletableFuture<String> statusFuture =
        future.thenApply(response -> response.status());
```

Now the flow is:

```text id="c5n8kq"
PSP
 ↓
PspResponse
 ↓
thenApply()
 ↓
response.status()
 ↓
String
```

So the type changes:

```text id="2z7g9k"
CompletableFuture<PspResponse>
              ↓
          thenApply()
              ↓
CompletableFuture<String>
```

This is the fundamental purpose of `thenApply()`.

---

# 2. Apply it to our payment

We can write:

```java id="6g3q6q"
CompletableFuture<String> statusFuture =
        CompletableFuture
                .supplyAsync(
                        () -> pspClient.authorize(payment),
                        paymentExecuter.paymentExecutor()
                )
                .thenApply(response -> response.status());
```

Then:

```java id="3b3n4w"
String status = statusFuture.join();
```

And:

```java id="r9q8hl"
if ("SUCCESS".equals(status)) {
    payment.setStatus(PaymentStatus.SUCCESS);
} else {
    payment.setStatus(PaymentStatus.FAILED);
}
```

---

# 3. But we're still using `join()`

Yes.

This:

```java id="z8x5jj"
String status = statusFuture.join();
```

still blocks.

We're **not trying to make the whole API non-blocking yet**.

We're learning how to compose asynchronous operations.

That's an important distinction.

---

# 4. `thenApply()` can do more than extract a field

Suppose your PSP returns:

```java
PspResponse
```

and you want to convert it to your own domain object:

```java
PaymentStatus
```

You could do:

```java id="l0r8m4"
CompletableFuture<PaymentStatus> statusFuture =
        CompletableFuture
                .supplyAsync(
                        () -> pspClient.authorize(payment),
                        paymentExecuter.paymentExecutor()
                )
                .thenApply(response -> {

                    if ("SUCCESS".equals(response.status())) {
                        return PaymentStatus.SUCCESS;
                    }

                    return PaymentStatus.FAILED;
                });
```

Now:

```text id="ujh7c2"
CompletableFuture<PspResponse>
             ↓
        thenApply()
             ↓
CompletableFuture<PaymentStatus>
```

Then:

```java id="i5t1b8"
PaymentStatus status = statusFuture.join();

payment.setStatus(status);
paymentRepository.save(payment);
```

---

# 5. The key interview rule

Remember this:

> **`thenApply()` is for synchronous transformation of the previous result.**

For example:

```java id="4g0f8d"
CompletableFuture<A>
        ↓
thenApply(A → B)
        ↓
CompletableFuture<B>
```

Examples:

```java id="j8i1l0"
PspResponse → String
```

```java id="k5v9l2"
PspResponse → PaymentStatus
```

```java id="e3v7x1"
Payment → PaymentResponse
```

---

# 6. Now compare it with `thenAccept()`

Suppose you **don't need to produce another result**.

You simply want to perform an action.

For example:

```java id="x2s8k1"
future.thenAccept(response -> {

    System.out.println(
            "PSP status = " + response.status()
    );

});
```

The difference:

### `thenApply`

```java id="k8r2y0"
.thenApply(response -> response.status())
```

produces:

```text id="c5r1zx"
CompletableFuture<String>
```

### `thenAccept`

```java id="9t4qwb"
.thenAccept(response -> {
    System.out.println(response.status());
});
```

produces:

```text id="z1k6pq"
CompletableFuture<Void>
```

Because we're not returning anything.

---

# 7. Think of them like this

### `thenApply`

> "I have a result. Transform it."

```text id="4h7q3w"
A → B
```

### `thenAccept`

> "I have a result. Do something with it."

```text id="8m1v6x"
A → void
```

### `thenRun`

> "I don't care about the previous result. Just run something afterward."

```java id="7c4n2p"
future.thenRun(() -> {
    System.out.println("PSP operation finished");
});
```

So:

```text id="j5v8s3"
thenApply  → transform result
thenAccept → consume result
thenRun    → just execute something
```

That's a very useful interview cheat sheet.

---

# 8. Now comes `thenCompose()` — VERY important

This is where many candidates get confused.

Suppose after the PSP succeeds, we need to call another asynchronous service:

```text id="4xq0zn"
PSP authorization
       ↓
Ledger service
```

And suppose our ledger client is asynchronous:

```java
CompletableFuture<LedgerResponse> updateLedger(Payment payment)
```

Now we have:

```java
CompletableFuture<PspResponse>
```

and after PSP completes, we want:

```java
CompletableFuture<LedgerResponse>
```

We could try:

```java
.thenApply(response ->
    ledgerClient.updateLedger(payment)
)
```

But that creates:

```text id="z5p9x2"
CompletableFuture<CompletableFuture<LedgerResponse>>
```

😵

That's called a **nested CompletableFuture**.

---

# 9. `thenCompose()` solves this

Instead:

```java id="n7x2q4"
CompletableFuture<LedgerResponse> ledgerFuture =
        pspFuture.thenCompose(
                response -> ledgerClient.updateLedger(payment)
        );
```

Now:

```text id="w2c8k1"
PSP Future
    ↓
PspResponse
    ↓
thenCompose()
    ↓
Ledger Future
```

The resulting type is simply:

```text id="v6n1p3"
CompletableFuture<LedgerResponse>
```

not:

```text id="z8k4q1"
CompletableFuture<CompletableFuture<LedgerResponse>>
```

---

# 10. The easiest way to remember it

### `thenApply`

If your function returns a **normal value**:

```java
A → B
```

use:

```java
thenApply()
```

---

### `thenCompose`

If your function returns **another CompletableFuture**:

```java
A → CompletableFuture<B>
```

use:

```java
thenCompose()
```

That's the key.

---

# 11. Payment example

Imagine:

```java id="a4r9w2"
CompletableFuture<PspResponse> pspFuture =
        CompletableFuture.supplyAsync(
                () -> pspClient.authorize(payment),
                paymentExecuter.paymentExecutor()
        );
```

Then:

```java id="n8q5v3"
CompletableFuture<LedgerResponse> ledgerFuture =
        pspFuture.thenCompose(
                response -> ledgerClient.updateLedger(payment)
        );
```

The complete flow becomes:

```text id="h8s4k2"
PSP
 ↓
PspResponse
 ↓
thenCompose()
 ↓
Ledger Service
 ↓
LedgerResponse
```

That's a **realistic payment workflow** and exactly the kind of thing you can discuss in an interview.

---

# 12. Your interview cheat sheet

| Method          | Use when                                          | Result                           |
| --------------- | ------------------------------------------------- | -------------------------------- |
| `supplyAsync()` | Start async work that returns a value             | `CompletableFuture<T>`           |
| `thenApply()`   | Transform a result                                | `CompletableFuture<U>`           |
| `thenAccept()`  | Consume result, return nothing                    | `CompletableFuture<Void>`        |
| `thenRun()`     | Run something after completion, don't need result | `CompletableFuture<Void>`        |
| `thenCompose()` | Chain another async operation                     | Flattened `CompletableFuture<U>` |

The two you should **really remember**:

```text id="s0j4v8"
thenApply
    ↓
A → B
```

versus:

```text id="q7k3m1"
thenCompose
    ↓
A → CompletableFuture<B>
```

---

## Next: Exception Handling 🔥

Now we're going to make your fake PSP **throw an exception**.

Then you'll see why these are different:

```java
exceptionally()
handle()
whenComplete()
```

And we'll answer a very common interview question:

> **"How do you handle exceptions in `CompletableFuture`?"**

After that, we'll add a **PSP timeout**, because for a payment service, an external dependency that hangs for 30 seconds is a much more interesting problem than a simple exception.
