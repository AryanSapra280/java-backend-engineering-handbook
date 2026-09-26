Absolutely. Let's make **Async Programming in Java** a proper study topic that you can directly note down for the interview.

The most important thing is that you don't mix up these four concepts:

```text
1. Multithreading
2. Asynchronous execution
3. CompletableFuture
4. Asynchronous/non-blocking I/O
```

They are related, but **not the same thing**.

---

# ASYNC PROGRAMMING IN JAVA — INTERVIEW NOTES

## 1. What is asynchronous programming?

In synchronous execution:

```text
Task A
  ↓
wait
  ↓
Task A completes
  ↓
Task B
  ↓
wait
  ↓
Task B completes
```

The caller waits for each operation.

Example:

```java
Payment payment = paymentService.createPayment();

FraudResult fraud = fraudService.check(payment);

Notification notification =
        notificationService.send(payment);
```

If:

```text
createPayment = 100 ms
fraud         = 500 ms
notification  = 200 ms
```

Total can be approximately:

```text
100 + 500 + 200 = 800 ms
```

---

## 2. What does asynchronous execution mean?

It means:

> The caller can start an operation without waiting for that operation to complete before continuing with other work.

For example:

```java
CompletableFuture<Payment> payment =
        createPaymentAsync();

CompletableFuture<FraudResult> fraud =
        checkFraudAsync();
```

Both operations can be in progress concurrently.

---

# 3. Async does NOT automatically mean parallel

This is important.

### Asynchronous

Means:

> I don't necessarily wait for the operation immediately.

### Parallel

Means:

> Multiple pieces of work are actually executing simultaneously.

You can have asynchronous execution that isn't truly parallel depending on the executor/scheduling model.

For interview purposes:

```text
Async ≠ automatically parallel
```

---

# 4. Async does NOT automatically mean non-blocking

This is probably the **most important distinction** for your interview.

Consider:

```java
CompletableFuture.supplyAsync(() -> {

    return restTemplate.getForObject(
        "http://fraud-service",
        FraudResult.class
    );

});
```

The caller can continue asynchronously.

But the executor thread running the lambda may be **blocked** while `RestTemplate` waits for the HTTP response.

So:

```text
Caller thread
     |
     | released
     ↓
Executor thread
     |
     | blocking HTTP call
     ↓
Fraud Service
```

This is asynchronous execution, but the I/O itself is blocking.

---

# 5. What is a Future?

Before `CompletableFuture`, Java had:

```java
Future<T>
```

Example:

```java
ExecutorService executor =
        Executors.newFixedThreadPool(5);

Future<String> future =
        executor.submit(() -> {

            return "Payment successful";

        });
```

You can retrieve:

```java
String result = future.get();
```

But:

```java
future.get();
```

is blocking.

The caller waits until the result is available.

---

# 6. Why CompletableFuture?

`CompletableFuture` improves upon basic `Future` by allowing you to:

* compose asynchronous operations
* chain operations
* combine multiple futures
* handle exceptions
* apply timeouts
* explicitly complete futures
* build asynchronous workflows

Example:

```java
CompletableFuture
        .supplyAsync(() -> getPayment())
        .thenApply(payment -> convert(payment))
        .thenAccept(result -> sendResponse(result));
```

Instead of manually calling:

```java
future.get();
```

after every operation.

---

# 7. Creating a CompletableFuture

## `supplyAsync()`

Used when the asynchronous task returns a value.

```java
CompletableFuture<String> future =
        CompletableFuture.supplyAsync(() -> {

            return "SUCCESS";

        });
```

Result:

```text
CompletableFuture<String>
```

---

## `runAsync()`

Used when the task doesn't return a result.

```java
CompletableFuture<Void> future =
        CompletableFuture.runAsync(() -> {

            sendNotification();

        });
```

Think:

```text
supplyAsync → produces result

runAsync → just performs action
```

---

# 8. Where does the async task execute?

If you don't provide an executor:

```java
CompletableFuture.supplyAsync(() -> {
    ...
});
```

Java uses its default asynchronous execution facility, commonly the `ForkJoinPool.commonPool()` for these methods.

You can explicitly provide your own executor:

```java
ExecutorService executor =
        Executors.newFixedThreadPool(10);

CompletableFuture.supplyAsync(
    () -> callPaymentService(),
    executor
);
```

This is often preferable for backend applications because you control the resources used for that workload.

---

# 9. Why use a custom executor?

Imagine your application has:

```text
Payment processing
Fraud checks
Email
Reporting
```

If everything uses the same shared pool, one workload can consume resources needed by another.

For example:

```text
1000 slow email tasks
        ↓
executor exhausted
        ↓
payment task waits
```

You can isolate workloads:

```text
Payment Executor
   ↓
10 threads

Notification Executor
   ↓
5 threads

Reporting Executor
   ↓
3 threads
```

This is related to the **bulkhead pattern** we'll cover later.

---

# 10. `thenApply()`

Use `thenApply()` to transform a successful result.

```java
CompletableFuture<Payment> paymentFuture =
        createPayment();

CompletableFuture<String> idFuture =
        paymentFuture.thenApply(
            payment -> payment.getPaymentId()
        );
```

Conceptually:

```text
Future<Payment>
      ↓
thenApply()
      ↓
Future<String>
```

It is similar to:

```text
map
```

in functional programming.

---

# 11. `thenAccept()`

Use it when you want to consume the result but don't need another result.

```java
paymentFuture.thenAccept(payment -> {

    System.out.println(payment.getPaymentId());

});
```

Conceptually:

```text
Future<Payment>
      ↓
consumer
      ↓
Future<Void>
```

---

# 12. `thenRun()`

Use when you don't care about the previous result.

```java
paymentFuture.thenRun(() -> {

    System.out.println("Payment processing completed");

});
```

Difference:

```text
thenApply
→ uses result + produces result

thenAccept
→ uses result + doesn't produce meaningful result

thenRun
→ doesn't use result + doesn't produce result
```

---

# 13. `thenCompose()`

🔥 **Very important interview question.**

Suppose:

```java
CompletableFuture<Payment> payment =
        createPaymentAsync();
```

and:

```java
CompletableFuture<FraudResult> fraud =
        checkFraudAsync(payment);
```

The second operation depends on the first.

You write:

```java
payment.thenCompose(
    p -> checkFraudAsync(p)
);
```

Why not `thenApply()`?

Because:

```java
thenApply()
```

would produce:

```text
CompletableFuture<CompletableFuture<FraudResult>>
```

whereas:

```text
thenCompose()
```

flattens it:

```text
CompletableFuture<FraudResult>
```

### Mental model

```text
thenApply
Future<A>
   ↓
A → B
   ↓
Future<B>
```

```text
thenCompose
Future<A>
   ↓
A → Future<B>
   ↓
Future<B>
```

---

# 14. Example: dependent asynchronous calls

Suppose:

```text
Get customer
      ↓
Get customer's wallet
      ↓
Get wallet balance
```

You could write:

```java
getCustomerAsync(customerId)
    .thenCompose(customer ->
        getWalletAsync(customer.getWalletId())
    )
    .thenCompose(wallet ->
        getBalanceAsync(wallet.getId())
    );
```

This creates an asynchronous chain.

---

# 15. `thenCombine()`

Use this when you have **independent asynchronous operations** and need both results.

Suppose:

```java
CompletableFuture<Customer> customer =
        getCustomerAsync();

CompletableFuture<FraudResult> fraud =
        checkFraudAsync();
```

Neither depends on the other.

You can do:

```java
CompletableFuture<PaymentDecision> decision =
    customer.thenCombine(
        fraud,
        (c, f) -> createDecision(c, f)
    );
```

Conceptually:

```text
Customer ────────┐
                 ├──> createDecision()
Fraud ───────────┘
```

This is often cleaner than manually calling `join()`.

---

# 16. `allOf()`

Suppose you have:

```java
CompletableFuture<A> f1;
CompletableFuture<B> f2;
CompletableFuture<C> f3;
```

You want to wait until all are complete.

```java
CompletableFuture<Void> all =
        CompletableFuture.allOf(f1, f2, f3);
```

Important:

`allOf()` returns:

```text
CompletableFuture<Void>
```

It doesn't automatically give you:

```text
List<A/B/C>
```

You have to obtain the individual results yourself.

---

# 17. Does `allOf()` block?

No.

This:

```java
CompletableFuture<Void> all =
        CompletableFuture.allOf(f1, f2, f3);
```

creates a future representing completion of all futures.

But:

```java
all.join();
```

**does block the current thread** until all complete.

This distinction is extremely important.

```text
allOf()
→ asynchronous composition

join()
→ wait for result
```

---

# 18. `allOf()` without blocking

You can compose further:

```java
return CompletableFuture.allOf(
        customerFuture,
        fraudFuture,
        merchantFuture
    )
    .thenApply(v -> {

        Customer customer = customerFuture.join();
        FraudResult fraud = fraudFuture.join();
        Merchant merchant = merchantFuture.join();

        return buildResponse(
            customer,
            fraud,
            merchant
        );
    });
```

The `thenApply` runs only after `allOf` has completed, so those futures are already complete.

For a small number of typed futures, `thenCombine()` can often express the relationship more naturally.

---

# 19. `join()` vs `get()`

Both retrieve the result and can wait.

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

This is one reason `join()` is often more convenient when composing CompletableFutures.

---

# 20. `exceptionally()`

Used for recovery from failure.

```java
CompletableFuture<String> future =
    callPaymentService()
        .exceptionally(ex -> {

            log.error("Payment failed", ex);

            return "FAILED";
        });
```

If the previous stage fails, you can return a fallback.

---

# 21. `handle()`

`handle()` receives both:

```text
result
exception
```

Example:

```java
future.handle((result, exception) -> {

    if (exception != null) {
        return "FAILED";
    }

    return result;
});
```

It can transform either success or failure into another result.

---

# 22. `whenComplete()`

Useful for observing completion.

```java
future.whenComplete((result, exception) -> {

    if (exception != null) {
        log.error("Failed", exception);
    } else {
        log.info("Success");
    }

});
```

It's commonly useful for:

* logging
* metrics
* tracing
* cleanup

without fundamentally changing the result.

---

# 23. Important difference

Remember:

```text
exceptionally
→ recover from failure

handle
→ inspect success/failure and produce another result

whenComplete
→ observe completion
```

---

# 24. Timeouts

Very important in microservices.

Suppose:

```text
Payment → Fraud Service
```

and Fraud doesn't respond.

Don't wait indefinitely.

Modern `CompletableFuture` provides:

```java
future.orTimeout(
    2,
    TimeUnit.SECONDS
);
```

This completes the future exceptionally if the timeout expires.

Another option:

```java
future.completeOnTimeout(
    fallbackResult,
    2,
    TimeUnit.SECONDS
);
```

which provides a fallback result.

---

# 25. But timeout ≠ cancellation of underlying work

🔥 Important.

Suppose:

```java
future.orTimeout(2, TimeUnit.SECONDS);
```

Your future can complete exceptionally after 2 seconds.

That doesn't automatically mean the underlying external system has stopped processing the operation.

For a payment:

```text
Your timeout
     ↓
2 seconds
     ↓
You stop waiting
```

doesn't mean:

```text
PSP stopped processing payment
```

The PSP may have already processed it.

This is why payment systems need:

```text
PROCESSING / UNKNOWN
       ↓
status inquiry
       ↓
webhook
       ↓
reconciliation
```

rather than simply:

```text
timeout → FAILED
```

---

# 26. Returning CompletableFuture from Service

Example:

```java
@Service
public class PaymentService {

    public CompletableFuture<PaymentResponse> process(
            PaymentRequest request) {

        return CompletableFuture.supplyAsync(() -> {

            return paymentProcessor.process(request);

        });
    }
}
```

Controller:

```java
@GetMapping("/payments/{id}")
public CompletableFuture<PaymentResponse> getPayment(
        @PathVariable String id) {

    return paymentService.process(id);
}
```

Spring can handle the asynchronous return value.

The UI does **not** receive a CompletableFuture.

It eventually receives:

```json
{
  "paymentId": "P123",
  "status": "SUCCESS"
}
```

---

# 27. Important: UI still waits

This is where you were asking the right question earlier.

Suppose:

```java
@GetMapping
public CompletableFuture<Response> get() {
    return service.process();
}
```

and processing takes:

```text
5 seconds
```

The browser generally sees:

```text
Loading...
```

for those 5 seconds.

Then:

```text
Future completes
      ↓
Spring generates HTTP response
      ↓
Browser receives JSON
```

So:

> **Returning CompletableFuture does not mean the frontend immediately receives a response.**

It means the **server-side request handling can be asynchronous**.

---

# 28. Immediate response design

For long-running payment processing, another architecture is:

```java
@PostMapping("/payments")
public ResponseEntity<PaymentAcceptedResponse> createPayment(
        @RequestBody PaymentRequest request) {

    PaymentAcceptedResponse response =
        paymentService.accept(request);

    return ResponseEntity
        .accepted()
        .body(response);
}
```

Response:

```http
202 Accepted
```

```json
{
    "paymentId": "P123",
    "status": "PROCESSING"
}
```

Meanwhile:

```text
Payment API
     |
     +---- save PROCESSING
     |
     +---- publish event
               |
               v
             Kafka
               |
       ┌───────┼────────┐
       ↓       ↓        ↓
     Fraud    PSP    Notification
```

The UI can later:

```http
GET /payments/P123
```

or receive an update through an appropriate push mechanism.

---

# 29. CompletableFuture vs 202 Accepted

This distinction is **excellent interview material**.

### CompletableFuture controller

```java
CompletableFuture<Response>
```

means:

> The HTTP request remains logically open until the future completes.

### `202 Accepted`

```text
HTTP 202
```

means:

> The server accepted the work but isn't claiming that the business operation is finished.

So:

```text
CompletableFuture
→ server-side execution/request handling mechanism

202 Accepted
→ API-level business contract
```

They solve different problems.

---

# 30. Synchronous vs asynchronous API

### Synchronous

```text
POST /payment
      ↓
process everything
      ↓
SUCCESS
```

Response:

```json
{
  "status": "SUCCESS"
}
```

### Asynchronous API

```text
POST /payment
      ↓
create payment
      ↓
202 PROCESSING
```

Then:

```text
GET /payment/P123
      ↓
SUCCESS
```

This is especially useful when the operation involves:

* external PSP
* fraud
* settlement
* reconciliation
* multiple downstream services
* long-running processing

---

# 31. `CompletableFuture` and exception propagation

Suppose:

```java
CompletableFuture
    .supplyAsync(() -> step1())
    .thenApply(result -> step2(result))
    .thenApply(result -> step3(result));
```

If:

```text
step1 fails
```

the dependent stages generally don't execute normally; the future becomes exceptional and the failure can be handled by a downstream exception-handling stage.

For example:

```java
.handle((result, ex) -> {

    if (ex != null) {
        return fallback();
    }

    return result;
});
```

---

# 32. What if `step2()` fails?

```text
step1
 ↓
success
 ↓
step2
 ↓
failure
 ↓
step3 doesn't execute normally
 ↓
exception handler
```

This is useful for building asynchronous workflows.

---

# 33. Async parallel processing example

Suppose payment requires:

```text
Customer service
Merchant service
Fraud service
```

These don't depend on each other.

Do:

```java
CompletableFuture<Customer> customer =
        getCustomer();

CompletableFuture<Merchant> merchant =
        getMerchant();

CompletableFuture<FraudResult> fraud =
        checkFraud();
```

Then combine.

Conceptually:

```text
             Payment
                |
      ┌─────────┼─────────┐
      ↓         ↓         ↓
   Customer  Merchant    Fraud
      |         |         |
      └─────────┼─────────┘
                ↓
          Payment decision
```

This is where async execution can actually reduce latency.

---

# 34. Async sequential dependency

Now suppose:

```text
Payment
  ↓
getCustomer()
  ↓
getWallet(customer.walletId)
  ↓
getBalance(wallet.id)
```

These operations are dependent.

Don't try to run them all in parallel.

Use:

```java
getCustomer()
    .thenCompose(customer ->
        getWallet(customer.getWalletId())
    )
    .thenCompose(wallet ->
        getBalance(wallet.getId())
    );
```

This is a good example of when **`thenCompose()`** is useful.

---

# 35. Async parallel + then combine

Real applications often have both.

Example:

```text
Get customer
      ↓
      ├──────────→ Get wallet
      |
      └──────────→ Check fraud
```

Once customer is known:

```java
getCustomer()
    .thenCompose(customer -> {

        CompletableFuture<Wallet> wallet =
            getWallet(customer.getWalletId());

        CompletableFuture<FraudResult> fraud =
            checkFraud(customer);

        return wallet.thenCombine(
            fraud,
            (w, f) -> buildDecision(customer, w, f)
        );
    });
```

That's the kind of composition you should understand.

---

# 36. Common mistake: blocking inside async code

Bad:

```java
CompletableFuture<Payment> payment =
    getPayment();

Payment p = payment.join();

return CompletableFuture
    .supplyAsync(() -> process(p));
```

You've introduced unnecessary blocking.

Prefer composing:

```java
return getPayment()
    .thenCompose(p ->
        processAsync(p)
    );
```

---

# 37. Common mistake: async everything

Don't do:

```text
Controller
 ↓
CompletableFuture
 ↓
Service
 ↓
CompletableFuture
 ↓
Repository
 ↓
CompletableFuture
 ↓
Database
```

just because "async is faster."

You need to understand:

> **What resource am I trying to free? What latency am I trying to reduce? What work can actually happen concurrently?**

If the database call is blocking, wrapping it in `CompletableFuture` just moves the blocking to another thread.

---

# 38. Blocking vs non-blocking

### Blocking

```java
String result =
    restTemplate.getForObject(...);
```

The calling thread waits.

### Async wrapper around blocking call

```java
CompletableFuture.supplyAsync(() ->
    restTemplate.getForObject(...)
);
```

The caller can continue, but an executor thread is blocked.

### Non-blocking/reactive I/O

A reactive HTTP client can avoid occupying a thread while waiting for I/O, depending on the programming model.

Conceptually:

```text
Blocking:

Thread ──────── WAIT ──────── Response


Non-blocking:

Thread ── initiate I/O ──> available for other work
                              |
                              ↓
                           response event
```

Don't claim "WebClient = always non-blocking" without considering how it's used, but in a reactive stack it is designed for non-blocking I/O.

---

# 39. Async and database operations

This is another trap.

Suppose:

```java
CompletableFuture.supplyAsync(() ->
    repository.findById(id)
);
```

If JPA/JDBC is being used:

```text
Executor thread
      |
      v
JDBC
      |
      | WAIT
      v
Database
```

The database operation is still blocking that executor thread.

Therefore, don't blindly say:

> "CompletableFuture makes database calls non-blocking."

It doesn't.

---

# 40. Async + thread pool sizing

Suppose:

```text
Thread pool = 10
```

and every task makes a 10-second blocking external call.

You can have:

```text
10 threads
 ↓
10 requests
 ↓
all waiting
```

Request 11 has to wait.

So asynchronous programming doesn't eliminate resource limits.

You need:

```text
bounded pools
timeouts
bulkheads
backpressure
rate limiting
downstream capacity
```

---

# 41. `CompletableFuture` cancellation

You can call:

```java
future.cancel(true);
```

But don't assume this magically terminates every underlying operation.

Cancellation behavior depends on the task/executor and whether the underlying operation responds to interruption/cancellation.

Again:

```text
cancel Future
≠
cancel external PSP transaction
```

Very important for payment systems.

---

# 42. `CompletableFuture` vs ExecutorService

These aren't competitors.

### ExecutorService

Primarily manages:

```text
threads
tasks
thread pool
```

### CompletableFuture

Primarily provides:

```text
asynchronous result
composition
chaining
exception handling
coordination
```

You often use both:

```java
ExecutorService executor =
    Executors.newFixedThreadPool(10);

CompletableFuture
    .supplyAsync(
        () -> callService(),
        executor
    );
```

---

# 43. `CompletableFuture` vs Spring `@Async`

Spring provides:

```java
@Async
public CompletableFuture<Result> process() {
    ...
}
```

The method can be executed asynchronously using a Spring-configured executor.

Example configuration:

```java
@Configuration
@EnableAsync
public class AsyncConfig {

    @Bean
    public Executor paymentExecutor() {

        ThreadPoolTaskExecutor executor =
            new ThreadPoolTaskExecutor();

        executor.setCorePoolSize(10);
        executor.setMaxPoolSize(20);
        executor.setQueueCapacity(100);

        executor.initialize();

        return executor;
    }
}
```

Then:

```java
@Async("paymentExecutor")
public CompletableFuture<Result> process() {

    Result result = doWork();

    return CompletableFuture.completedFuture(result);
}
```

---

# 44. Important `@Async` trap

This often doesn't work as people expect:

```java
@Service
public class PaymentService {

    public void outer() {
        inner();
    }

    @Async
    public void inner() {
        ...
    }
}
```

Why?

Because `@Async` relies on a Spring proxy.

Calling:

```java
inner();
```

from the **same object** bypasses the proxy.

This is called **self-invocation**.

A call coming through the Spring proxy is what triggers the asynchronous interceptor.

We'll revisit this when we cover Spring AOP.

---

# 45. Async exception trap

If an async method throws an exception:

```java
@Async
public void process() {

    throw new RuntimeException("failed");
}
```

the caller doesn't necessarily receive it like a normal synchronous method call.

With:

```java
@Async
public CompletableFuture<Result> process()
```

the exception can be represented by the returned future.

That's another reason returning a `CompletableFuture` can be useful when the caller needs to observe completion/failure.

---

# 46. Payment example — putting everything together

Suppose:

```text
POST /payments
```

You don't want to wait for:

```text
Fraud
PSP
Notification
Settlement
```

So:

```text
Client
   |
   | POST /payments
   ↓
Payment Service
   |
   | DB transaction
   |
   +---- Payment = PROCESSING
   |
   +---- Outbox event
   |
   ↓
202 Accepted
   |
   ↓
Client gets paymentId
```

Then asynchronously:

```text
Outbox
  ↓
Kafka
  ↓
Payment Worker
  |
  +---- Fraud
  |
  +---- PSP
  |
  +---- Notification
```

If PSP responds:

```text
SUCCESS
```

update:

```text
PROCESSING → SUCCESS
```

If PSP times out:

```text
PROCESSING → UNKNOWN
```

Then:

```text
Webhook
   OR
Status inquiry
   OR
Reconciliation
```

determines the final state.

This is a much more realistic payment architecture than simply doing:

```java
CompletableFuture.supplyAsync(() -> callPSP());
```

and assuming async solves everything.

---

# 47. What you should remember for tomorrow

### Concept 1

**CompletableFuture is not a thread.**

It represents a future result.

---

### Concept 2

**`supplyAsync()` executes work asynchronously.**

```java
CompletableFuture.supplyAsync(...)
```

---

### Concept 3

**`join()` / `get()` can block.**

```java
future.join();
```

means:

> "I need the result now."

---

### Concept 4

**Returning a CompletableFuture from a Spring controller doesn't mean the UI gets a future.**

The UI eventually gets normal HTTP:

```text
JSON
HTTP status
headers
```

---

### Concept 5

**Returning CompletableFuture doesn't mean the UI immediately gets a response.**

If the future takes 5 seconds:

```text
UI waits ~5 sec
```

before receiving the final response.

---

### Concept 6

If you want an immediate response:

```text
202 Accepted
+
paymentId
+
PROCESSING
```

and continue processing asynchronously.

---

### Concept 7

**Async ≠ non-blocking.**

```text
CompletableFuture + RestTemplate
```

can still block an executor thread.

---

### Concept 8

**Async ≠ automatically faster.**

It helps when work can be concurrent or when you need to avoid blocking a request-processing thread.

---

### Concept 9

Use:

```text
thenApply
```

for transformation.

```text
thenCompose
```

for dependent async operations.

```text
thenCombine
```

for independent futures whose results need to be combined.

```text
allOf
```

to coordinate completion of multiple futures.

---

### Concept 10

For production microservices, think beyond `CompletableFuture`:

```text
Timeout
Retry
Circuit Breaker
Bulkhead
Rate Limiting
Backpressure
Idempotency
Messaging
Outbox
```

These are what make asynchronous/distributed systems reliable.

---

# 🔥 Interview questions you should write down

1. What is asynchronous programming?
2. Async vs parallel vs non-blocking?
3. What is `Future`?
4. Why was `CompletableFuture` introduced?
5. `supplyAsync()` vs `runAsync()`?
6. Where does `CompletableFuture` execute by default?
7. Why use a custom Executor?
8. `thenApply()` vs `thenCompose()`?
9. `thenCompose()` vs `thenCombine()`?
10. `thenAccept()` vs `thenRun()`?
11. What does `allOf()` return?
12. Does `allOf()` block?
13. Does `join()` block?
14. `get()` vs `join()`?
15. `exceptionally()` vs `handle()` vs `whenComplete()`?
16. How do you implement timeout with CompletableFuture?
17. Does a CompletableFuture timeout cancel the underlying PSP operation?
18. What happens if one stage in a CompletableFuture chain fails?
19. How do you execute three independent service calls concurrently?
20. How do you execute dependent async calls?
21. Why is immediately calling `join()` after `supplyAsync()` often a poor design?
22. Does CompletableFuture make blocking JDBC/HTTP calls non-blocking?
23. CompletableFuture vs ExecutorService?
24. CompletableFuture vs Spring `@Async`?
25. How does `@Async` work internally?
26. Why does self-invocation break `@Async`?
27. If a controller returns `CompletableFuture`, what does the frontend receive?
28. Does returning CompletableFuture immediately send an HTTP response?
29. How would you design an API that immediately returns while payment processing continues?
30. Why would you use `202 Accepted`?
31. How would the UI know when asynchronous payment processing is complete?
32. How do timeout, retry and idempotency interact in an asynchronous payment system?
33. How would you prevent an asynchronous payment from being lost if the application crashes?
34. Why would you use Kafka/outbox instead of only `CompletableFuture` for long-running payment workflows?

### The single most important mental model

```text
                 ASYNC PROGRAMMING
                        |
        ┌───────────────┼────────────────┐
        ↓               ↓                ↓
   Execution        Composition       API Design
        |               |                |
 ExecutorService   CompletableFuture    202 Accepted
 @Async             thenCompose         Polling
 Thread Pool        thenCombine          WebSocket/SSE
                    allOf
                    timeout
```

**`CompletableFuture` handles asynchronous computation inside your application. It does not by itself define how the frontend should interact with a long-running operation.**

That distinction is worth remembering very clearly for tomorrow.
