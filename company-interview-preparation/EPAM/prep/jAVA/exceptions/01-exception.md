# Exception Handling — EPAM Interview Level 🔥

This is the right next topic because exception questions can start from basic Java theory and quickly turn into **Spring + DB + microservice production scenarios**.

We'll build it from Java → Spring → Microservices.

---

## 1. What is an exception?

An exception represents an abnormal condition during program execution that disrupts the normal flow.

Example:

```java
int result = 10 / 0;
```

This throws:

```text
ArithmeticException
```

The important interview point is that Java represents exceptions as objects.

---

# 2. Exception hierarchy

The hierarchy you should remember:

```text
Object
  ↓
Throwable
  ├── Error
  │     ├── OutOfMemoryError
  │     └── StackOverflowError
  │
  └── Exception
        ├── RuntimeException
        │     ├── NullPointerException
        │     ├── IllegalArgumentException
        │     └── ArithmeticException
        │
        └── Other checked exceptions
              ├── IOException
              ├── SQLException
              └── ...
```

### `Error`

Generally represents serious JVM/system-level problems.

Example:

```java
OutOfMemoryError
StackOverflowError
```

You generally **don't catch these to continue normal application processing**.

### `Exception`

Represents conditions that application code can potentially handle.

---

# 3. Checked vs unchecked exceptions

This is a **must-know interview question**.

## Checked exception

Compiler forces you to handle or declare it.

Example:

```java
public void readFile() throws IOException {
    Files.readString(Path.of("data.txt"));
}
```

You must either:

```java
try {
    ...
} catch (IOException e) {
    ...
}
```

or:

```java
throws IOException
```

Examples:

```text
IOException
SQLException
ClassNotFoundException
```

---

## Unchecked exception

Subclass of `RuntimeException`.

Compiler doesn't force handling.

```java
public void process(String name) {
    if (name == null) {
        throw new IllegalArgumentException("name required");
    }
}
```

Examples:

```text
NullPointerException
IllegalArgumentException
IllegalStateException
ArithmeticException
```

### Interview answer

> Checked exceptions are enforced by the compiler and generally represent conditions the caller may reasonably recover from. Unchecked exceptions extend RuntimeException and aren't required to be caught or declared.

Don't say:

> "Checked exceptions are compile-time exceptions."

That's inaccurate.

The **exception happens at runtime**; the compiler simply enforces handling.

---

# 4. `throw` vs `throws`

Very common.

### `throw`

Actually throws an exception.

```java
if (amount <= 0) {
    throw new IllegalArgumentException("Invalid amount");
}
```

### `throws`

Declares that a method may propagate an exception.

```java
public void read() throws IOException {
    ...
}
```

Think:

```text
throw  → actually throw
throws → declare possibility
```

---

# 5. How does exception propagation work?

This becomes important for your microservice questions.

Suppose:

```java
public void controller() {
    service();
}

public void service() {
    repository();
}

public void repository() {
    throw new RuntimeException("DB failed");
}
```

Execution:

```text
Controller
    ↓
Service
    ↓
Repository
    ↓
Exception occurs
```

If repository doesn't catch it:

```text
Repository
    ↑
    exception propagates
    ↑
Service
    ↑
    exception propagates
    ↑
Controller
```

Java walks back up the call stack looking for a matching `catch`.

This is called **exception propagation**.

---

# 6. What happens if nobody catches it?

Eventually the exception reaches the thread boundary.

For a normal Java thread:

```text
main()
  ↓
methodA()
  ↓
methodB()
  ↓
Exception
```

If nobody handles it, the thread terminates and Java's uncaught-exception handling mechanism gets involved.

In a Spring Boot HTTP application, however, there's another layer.

That's where things become interesting.

---

# 7. Spring Boot request flow

Imagine:

```text
Client
  ↓
HTTP Request
  ↓
Controller
  ↓
Service
  ↓
Repository
  ↓
PostgreSQL
```

Suppose DB fails.

```text
PostgreSQL
    ↓
SQLException / JDBC exception
    ↓
Repository
    ↓
Service
    ↓
Controller
```

You **don't necessarily want the repository to catch it and return some random response**.

Instead, the exception can propagate toward Spring's web exception handling infrastructure.

Then you can have:

```java
@RestControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(CustomerNotFoundException.class)
    public ResponseEntity<ErrorResponse> handle(
            CustomerNotFoundException ex) {

        return ResponseEntity
                .status(HttpStatus.NOT_FOUND)
                .body(new ErrorResponse(
                        "CUSTOMER_NOT_FOUND",
                        ex.getMessage()
                ));
    }
}
```

So your architecture becomes:

```text
                 ┌─────────────────────┐
                 │ Global Exception    │
                 │ Handler              │
                 └──────────┬──────────┘
                            ↑
Client → Controller → Service → Repository → DB
```

---

# 8. Why use `@RestControllerAdvice`?

Instead of doing this everywhere:

```java
try {
    service.process();
} catch (...) {
    ...
}
```

you centralize API exception handling.

For example:

```text
CustomerNotFoundException → 404
ValidationException       → 400
UnauthorizedException     → 401
ForbiddenException        → 403
BusinessException         → 422/400
UnexpectedException       → 500
```

This gives you a **consistent API contract**.

---

# 9. DB exception — important scenario

Suppose:

```java
customerRepository.save(customer);
```

fails because of a database constraint.

The low-level database/driver exception can be translated by Spring's data-access infrastructure into Spring's `DataAccessException` hierarchy.

Conceptually:

```text
Database
   ↓
JDBC / driver exception
   ↓
Spring Data access layer
   ↓
DataAccessException
   ↓
Service
   ↓
Global Exception Handler
   ↓
HTTP response
```

This is one reason Spring's data-access abstraction is useful.

You generally don't want every service to be tightly coupled to every vendor-specific database exception.

---

# 10. Should repository catch DB exceptions?

Usually **not just to rethrow them**.

Bad:

```java
try {
    return repository.save(customer);
} catch (Exception e) {
    throw new RuntimeException("Something failed");
}
```

You've now potentially destroyed useful information.

Better:

```text
Repository
    ↓
allow meaningful data-access exception to propagate
    ↓
Service decides whether business handling is needed
    ↓
API layer converts it to appropriate response
```

However, if you need to translate a low-level exception into a **meaningful business/domain exception**, that's different.

For example:

```java
catch (DataIntegrityViolationException e) {
    throw new CustomerAlreadyExistsException(
        "Customer already exists", e);
}
```

Notice the second argument:

```java
e
```

That leads to our next important concept.

---

# 11. Exception chaining

Suppose database throws:

```text
DataIntegrityViolationException
```

You want to expose:

```text
CustomerAlreadyExistsException
```

but don't want to lose the original cause.

```java
throw new CustomerAlreadyExistsException(
    "Customer already exists",
    e
);
```

Now:

```text
CustomerAlreadyExistsException
        ↓
      cause
        ↓
DataIntegrityViolationException
        ↓
original DB exception
```

You can inspect it with:

```java
ex.getCause()
```

This is called **exception chaining**.

### Why is it useful?

Because your application can expose a meaningful abstraction while preserving the original cause for debugging.

---

# 12. A very realistic microservice scenario

Suppose your Payment service calls Fraud service.

```text
Payment Service
       ↓
HTTP
       ↓
Fraud Service
       ↓
Database
```

Fraud service's DB fails.

What happens?

Potentially:

```text
Fraud DB
   ↓
DataAccessException
   ↓
Fraud Service
   ↓
HTTP 500
   ↓
Payment Service receives 500
```

Payment service's HTTP client may translate that into something like:

```text
FeignException
WebClientResponseException
RestClientResponseException
```

depending on the client.

Then:

```text
Fraud Service
      ↓
    500
      ↓
Payment Service
      ↓
client exception
      ↓
Payment Service handling
      ↓
customer response
```

And **this is where your interviewer may ask:**

> "Would you return the downstream 500 directly to your customer?"

Usually, **no**.

You should translate downstream failures into an appropriate contract for your own API.

For example:

```json
{
  "code": "PAYMENT_PROCESSING_UNAVAILABLE",
  "message": "Payment could not be processed at this time",
  "correlationId": "abc-123"
}
```

You don't want to leak:

```text
Postgres connection refused
SQLState 08001
internal service hostname
stack trace
```

to the customer.

---

# 13. Exception ≠ HTTP status

This distinction is important.

An exception is an **application/runtime event**.

HTTP status is an **API communication contract**.

For example:

```text
CustomerNotFoundException
        ↓
HTTP 404
```

or:

```text
DownstreamTimeoutException
        ↓
HTTP 503
```

The mapping is an application design decision.

---

# 14. What about validation?

Suppose:

```http
POST /payments
```

contains:

```json
{
    "amount": -100
}
```

Bean validation might generate a validation exception.

Spring's web layer can translate this into something like:

```http
400 Bad Request
```

Your global handler can standardize the response:

```json
{
    "code": "VALIDATION_ERROR",
    "message": "Invalid request",
    "errors": [
        {
            "field": "amount",
            "message": "must be greater than 0"
        }
    ]
}
```

Again:

```text
Exception
   ↓
Exception Handler
   ↓
API Error Contract
   ↓
Customer
```

---

# 15. The BIG mistake: `catch (Exception e)` everywhere

Interviewers love this.

Bad:

```java
try {
    paymentService.process();
} catch (Exception e) {
    return "Payment failed";
}
```

Why?

### Problem 1 — hides the real failure

### Problem 2 — destroys useful exception information

### Problem 3 — can hide programming bugs

For example:

```java
NullPointerException
```

gets treated exactly like:

```text
BusinessException
```

### Problem 4 — incorrect HTTP semantics

Everything might become:

```text
500
```

even when the correct response should be:

```text
400
404
409
503
```

---

# 16. What should you catch?

Catch an exception when you can **actually do something meaningful**.

For example:

```java
try {
    paymentService.process();
} catch (InsufficientBalanceException e) {
    // meaningful business response
}
```

Or translate an infrastructure exception:

```java
catch (DataIntegrityViolationException e) {
    throw new CustomerAlreadyExistsException(
        "Customer already exists", e);
}
```

But don't do:

```java
catch (Exception e) {
    throw new RuntimeException(e);
}
```

at every layer.

---

# 17. `finally`

`finally` generally executes regardless of whether an exception occurs.

```java
try {
    process();
} catch (Exception e) {
    handle(e);
} finally {
    cleanup();
}
```

Typical use:

```text
cleanup resources
release locks
close resources
```

But modern Java usually prefers **try-with-resources** for closeable resources.

---

# 18. Try-with-resources

Instead of:

```java
InputStream input = null;

try {
    input = new FileInputStream("data.txt");
    ...
} finally {
    if (input != null) {
        input.close();
    }
}
```

use:

```java
try (InputStream input =
         new FileInputStream("data.txt")) {

    ...
}
```

Java automatically closes the resource.

This is worth knowing, but we don't need to go deep into it for EPAM.

---

# 19. Exception + Transaction — VERY important for Spring

Consider:

```java
@Transactional
public void transfer() {

    debitAccount();

    creditAccount();
}
```

Suppose:

```text
debitAccount()
      ↓
success

creditAccount()
      ↓
exception
```

If the transaction is rolled back, the DB changes made within that transaction are rolled back.

For Spring's default rollback behavior, **unchecked exceptions (`RuntimeException` and `Error`) trigger rollback by default**, while checked exceptions don't automatically trigger rollback.

This is a very common interview question.

You can explicitly configure rollback behavior:

```java
@Transactional(rollbackFor = SomeCheckedException.class)
```

---

# 20. Now connect everything together

This is the scenario I want you to be able to explain in EPAM.

### Customer requests payment

```text
Client
  ↓
POST /payments
  ↓
PaymentController
  ↓
PaymentService
  ↓
PaymentRepository
  ↓
PostgreSQL
```

### DB failure

```text
PostgreSQL
    ↓
DB/driver exception
    ↓
Spring Data exception translation
    ↓
DataAccessException
    ↓
Service
    ↓
GlobalExceptionHandler
    ↓
HTTP 500/appropriate status
    ↓
Customer
```

### Downstream failure

```text
Payment Service
      ↓
Fraud Service
      ↓
timeout
      ↓
HTTP client exception
      ↓
Payment Service
      ↓
exception translation
      ↓
GlobalExceptionHandler
      ↓
HTTP 503
      ↓
Customer
```

### And in a real production system:

```text
Exception
   +
Correlation ID
   +
Structured logs
   +
Metrics
   +
Tracing
   +
Appropriate HTTP response
```

That's the **Senior Software Engineer-level perspective**.

---

# Interview questions you should now be ready for

### Java

1. What is the exception hierarchy?
2. Checked vs unchecked?
3. `throw` vs `throws`?
4. How does exception propagation work?
5. What happens if an exception isn't caught?
6. Why use custom exceptions?
7. What is exception chaining?
8. Why shouldn't we catch `Exception` everywhere?
9. What is try-with-resources?
10. What happens in `finally`?

### Spring

11. How does `@ControllerAdvice` work?
12. Difference between `@ExceptionHandler` and `@ControllerAdvice`?
13. How does Spring translate DB exceptions?
14. What happens when repository throws an exception?
15. How does `@Transactional` interact with exceptions?
16. Which exceptions trigger rollback by default?

### Microservices

17. What happens when a downstream service throws an exception?
18. Would you propagate the downstream exception directly?
19. How do you convert downstream failure into your API response?
20. What status code would you return for timeout/unavailable downstream?
21. How do you prevent internal exception details from reaching customers?
22. How would you debug an exception across multiple microservices?

That last group is **exactly where your previous TCS interview style comes in**.

---

## One mental model to retain

Don't think:

> "Exception = try/catch."

Think:

```text
                 ┌───────────────┐
                 │   Customer    │
                 └───────┬───────┘
                         ↓
                    Controller
                         ↓
                      Service
                    ↙         ↘
              Repository     Downstream
                   ↓             ↓
                  DB          Other MS
                   ↓             ↓
               Exception     Exception
                    ↘           ↙
                     Exception
                       Handler
                          ↓
                    Error Contract
                          ↓
                       Customer
```

And **each layer should handle the exception only when it has a meaningful responsibility to do so**.

Next, we should go into **Multithreading & Concurrency**. That's the much bigger EPAM area, and we'll start from the Java thread model, lifecycle, race conditions, `synchronized`, `volatile`, atomicity/visibility, locks, and then naturally move into **ExecutorService → Future → CompletableFuture → Spring `@Async`**.