# Spring Concurrency — `@Async`, TaskExecutor & Production Scenarios 🔥

Now we bridge the gap between **Java concurrency** and what actually happens in your **Spring Boot microservice**.

This is exactly the answer you want when EPAM asks:

> **"Have you worked with multithreading in Spring?"**

---

# 1. Problem: I want a Spring method to execute asynchronously

Suppose:

```java
@Service
public class PaymentService {

    public void processPayment() {
        // expensive work
    }
}
```

Normally:

```text
HTTP thread
   ↓
processPayment()
   ↓
wait
   ↓
response
```

We can make it asynchronous:

```java
@Async
public void processPayment() {
    // expensive work
}
```

and enable it:

```java
@EnableAsync
@SpringBootApplication
public class Application {
}
```

Now conceptually:

```text
HTTP Thread
    ↓
calls @Async method
    ↓
Task submitted to executor
    ↓
HTTP thread can continue
                     ↓
                 Worker Thread
                     ↓
                 processPayment()
```

---

# 2. Problem: Does `@Async` automatically create a new thread?

Not exactly.

This is an important interview distinction.

`@Async` delegates execution to a **Spring-managed executor**.

If you don't explicitly configure one, Spring uses its default async executor behavior, which may not be what you want for production.

So a better production design is:

```java
@Bean
public Executor paymentExecutor() {

    ThreadPoolTaskExecutor executor =
            new ThreadPoolTaskExecutor();

    executor.setCorePoolSize(8);
    executor.setMaxPoolSize(10);
    executor.setQueueCapacity(20);
    executor.setThreadNamePrefix("payment-");

    executor.initialize();

    return executor;
}
```

Then:

```java
@Async("paymentExecutor")
public void processPayment() {
    ...
}
```

Now you know exactly which pool handles that work.

---

# 3. Problem: Why not use one executor for everything?

Imagine:

```text
Application
   |
   +-- Payment tasks
   +-- Email tasks
   +-- Report generation
   +-- DB-heavy tasks
```

If everything uses the same pool:

```text
Report generation
      ↓
consumes all threads
      ↓
Payment tasks wait
```

That's **resource contention**.

A production system may use separate pools for workloads with different characteristics:

```text
paymentExecutor
emailExecutor
reportExecutor
```

You don't always need separate pools, but you should understand **why pool isolation can matter**.

---

# 4. Problem: What happens to the HTTP thread?

This is a very important distinction.

Suppose:

```java
@PostMapping("/payment")
public ResponseEntity<?> payment() {

    paymentService.processAsync();

    return ResponseEntity.ok().build();
}
```

and:

```java
@Async("paymentExecutor")
public void processAsync() {
    ...
}
```

Flow:

```text
HTTP request thread
       |
       | calls async method
       ↓
submit task
       |
       ↓
returns
       |
       ↓
HTTP response

Meanwhile:

paymentExecutor
       ↓
worker thread
       ↓
processAsync()
```

So the HTTP thread isn't performing the actual async work.

---

# 5. Problem: What if I return `CompletableFuture`?

This is much more useful for APIs where the caller needs the result.

```java
@Async("paymentExecutor")
public CompletableFuture<PaymentResult> processPayment() {

    PaymentResult result = process();

    return CompletableFuture.completedFuture(result);
}
```

Controller:

```java
CompletableFuture<PaymentResult> result =
        paymentService.processPayment();

return result;
```

Now Spring can work with the asynchronous result.

Conceptually:

```text
HTTP request
    ↓
async operation
    ↓
CompletableFuture
    ↓
eventual result
    ↓
HTTP response
```

---

# 6. Problem: `@Async` + CompletableFuture — why use both?

They solve slightly different things.

### `@Async`

Tells Spring:

> Execute this method asynchronously using an executor.

### `CompletableFuture`

Represents/composes the asynchronous result.

So:

```java
@Async("paymentExecutor")
public CompletableFuture<Result> process() {
    ...
}
```

means:

```text
Spring
 ↓
run method asynchronously

CompletableFuture
 ↓
represent the eventual result
```

---

# 7. Problem: What if `@Async` method throws an exception?

This depends on the return type.

Suppose:

```java
@Async
public void process() {

    throw new RuntimeException("Payment failed");
}
```

The caller doesn't receive that exception through a normal return path because the method is asynchronous.

For `void` async methods, Spring provides an `AsyncUncaughtExceptionHandler` mechanism for uncaught exceptions.

But if:

```java
@Async
public CompletableFuture<Result> process() {
    ...
}
```

the exception can be represented in the `CompletableFuture` and handled through the normal CompletableFuture mechanisms:

```java
future.exceptionally(...)
future.handle(...)
```

### Interview answer

> For `@Async void` methods, the caller cannot receive the exception through the return value, so Spring's async exception handling mechanism is used. For `CompletableFuture`-returning methods, the failure can be represented in the future and composed/handled using CompletableFuture APIs.

---

# 8. Problem: Why does `@Async` sometimes appear not to work?

🔥 **Very common Spring interview question.**

Consider:

```java
@Service
public class PaymentService {

    public void process() {
        processAsync();
    }

    @Async
    public void processAsync() {
        ...
    }
}
```

You might expect:

```text
process()
   ↓
new thread
```

But it may execute synchronously.

Why?

### Self-invocation.

Spring's `@Async` relies on the Spring proxy mechanism.

Conceptually:

```text
Caller
  ↓
Spring Proxy
  ↓
Async Executor
  ↓
Actual Bean
```

But:

```text
Bean
 ↓
this.processAsync()
```

bypasses the proxy.

So the async interception doesn't happen.

### Solution

Put the async method in another Spring bean:

```text
PaymentService
      ↓
AsyncPaymentService
      ↓
@Async
```

Or otherwise invoke it through the Spring-managed proxy.

---

# 9. Problem: Does `@Async` work on private methods?

Generally, no.

For the normal proxy-based mechanism, the method needs to be eligible for proxy interception; a `private` method can't be intercepted in the normal way.

So:

```java
@Async
private void process() {}
```

is not how you should use it.

For interviews:

> `@Async` works through Spring's proxy/interceptor mechanism, so self-invocation and method visibility/proxy limitations matter.

---

# 10. Problem: What happens to `@Transactional` with `@Async`?

🔥 **Very important Senior-level question.**

Suppose:

```java
@Transactional
public void processPayment() {

    savePayment();

    notificationService.sendAsync();
}
```

and:

```java
@Async
public void sendAsync() {
    ...
}
```

Do they run in the same transaction?

**No.**

The async method runs on a different thread.

The original thread's transaction context does not simply transfer to the new thread.

Think:

```text
Thread A
   |
@Transactional
   |
DB operations
   |
calls @Async
   |
   +--------------------+
                        |
                   Thread B
                        |
                    @Async
                        |
                 separate execution
```

This is extremely important.

### Interview answer

> Spring transactions are generally thread-bound. When execution moves to another thread through `@Async`, the original transaction context does not propagate automatically. If the async method needs its own transaction, it can define its own transactional boundary.

---

# 11. Problem: What about `ThreadLocal`?

Same fundamental issue.

Suppose:

```java
ThreadLocal<String> userContext;
```

Thread A:

```text
userContext = "Aryan"
```

Then:

```text
Thread A
   ↓
@Async
   ↓
Thread B
```

Thread B does not automatically get the same ordinary `ThreadLocal` value.

This becomes important for:

- authentication context
- correlation IDs
- MDC logging context
- tenant IDs
- request context

You need an explicit context propagation mechanism when required.

---

# 12. Production scenario: Correlation ID

Imagine:

```text
Request
Correlation-ID: abc-123
```

Then:

```text
HTTP Thread
     ↓
Payment Service
     ↓
@Async
     ↓
Worker Thread
```

If your logging context is thread-local, the worker thread may not automatically contain:

```text
abc-123
```

Then your logs become difficult to correlate.

A mature system therefore considers:

```text
Correlation ID
      ↓
context propagation
      ↓
async worker
      ↓
logs/traces
```

This is exactly the sort of practical point that can distinguish a Senior Engineer answer.

---

# 13. Problem: What happens when the executor is exhausted?

Suppose:

```text
core = 8
max = 10
queue = 20
```

Eventually:

```text
8 workers busy
    ↓
20 queued
    ↓
2 additional workers
    ↓
10 workers busy
    ↓
queue full
    ↓
rejection
```

Now your application must have a deliberate policy.

Possibilities:

```text
reject
caller-runs
retry
shed load
return controlled failure
```

And importantly:

> Increasing the thread pool indefinitely is not the solution.

Because downstream dependencies may become the bottleneck.

---

# 14. Problem: Async + DB

Suppose:

```text
Executor = 50 threads
DB pool = 10 connections
```

Your 50 async tasks all eventually call the DB.

You can get:

```text
50 application threads
       ↓
10 DB connections
       ↓
40 waiting
```

So concurrency must be designed across the entire stack:

```text
HTTP concurrency
       ↓
Application executor
       ↓
DB connection pool
       ↓
Database capacity
```

This is an excellent answer when someone asks:

> "How do you decide thread pool size?"

---

# 15. Problem: Should I use `@Async` for everything?

**No.**

It's appropriate when asynchronous execution provides a real benefit.

Good examples:

```text
send notification
background processing
independent expensive operation
non-critical post-processing
parallel independent operations
```

But don't blindly make everything async.

It adds:

- thread management
- error handling complexity
- context propagation issues
- transaction-boundary issues
- monitoring complexity
- ordering concerns

---

# 16. Java concurrency vs Spring concurrency

This is the exact distinction you wanted.

## Java

You directly manage:

```text
Thread
Runnable
Callable
ExecutorService
Future
CompletableFuture
synchronized
Lock
Atomic*
```

Example:

```java
ExecutorService executor =
        Executors.newFixedThreadPool(10);
```

## Spring

Spring provides abstractions/integration:

```text
@Async
TaskExecutor
ThreadPoolTaskExecutor
Spring-managed lifecycle
transaction/proxy integration
```

Example:

```java
@Async("paymentExecutor")
public CompletableFuture<Result> process() {
    ...
}
```

Underneath, you're **still using Java threads and executors**.

Spring doesn't replace Java concurrency.

It manages/integrates it into the Spring application model.

---

# 17. EPAM question: "Have you worked on multithreaded applications?"

Here's how I'd answer in your situation:

> "Yes. In Spring Boot microservices, requests are already processed concurrently by the servlet thread pool, so singleton Spring beans can be accessed by multiple threads. I keep service beans stateless wherever possible. For explicit asynchronous processing, I've worked with ExecutorService/CompletableFuture and Spring's TaskExecutor or `@Async`. I configure bounded thread pools based on workload and downstream capacity. I also consider exception propagation, transaction boundaries, interruption/cancellation, and context propagation when work moves to another thread."

That's a **much stronger answer than simply saying "yes, I used CompletableFuture."**

---

# 18. One complete production flow

Imagine your Payment microservice:

```text
                    Client
                      |
                      ↓
              HTTP request thread
                      |
                      ↓
              Payment Controller
                      |
                      ↓
              Payment Service
                  /       \
                 /         \
                ↓           ↓
          Fraud Service  Account Service
             async          async
                \           /
                 \         /
                  ↓       ↓
                 combine results
                       |
                       ↓
                Payment decision
                       |
                       ↓
                     DB
                       |
                       ↓
                    response
```

Now add the infrastructure:

```text
HTTP Thread Pool
       ↓
Payment Executor
       ↓
CompletableFuture
       ↓
Downstream services
       ↓
DB Connection Pool
       ↓
Database
```

And if something fails:

```text
Downstream timeout
       ↓
CompletableFuture exception
       ↓
exceptionally / handle
       ↓
business decision
       ↓
Global Exception Handler
       ↓
customer-safe HTTP response
```

That is the mental model I want you to carry into the interview.

---

# Concurrency section status

At this point we've covered the important path:

```text
Thread
 ↓
Race Condition
 ↓
synchronized
 ↓
volatile
 ↓
AtomicInteger / CAS
 ↓
Deadlock / starvation
 ↓
Thread safety
 ↓
ExecutorService
 ↓
ThreadPoolExecutor
 ↓
Future
 ↓
Thread interruption
 ↓
CompletableFuture
 ↓
Spring @Async
 ↓
TaskExecutor
 ↓
Transactions + async
 ↓
Context propagation
```

### Remaining Java after this

I'd now move to:

**1. JVM Internals** → practical interview level  
**2. Garbage Collection**  
**3. Serialization**  
**4. Testing**  
**5. Selected Design Patterns — including Facade, Strategy, Factory, Builder, Proxy, Adapter, Decorator**

Then we can transition into the **Spring/Microservices/System Design revision**, where your Java concurrency knowledge will directly connect to architecture and schema design.