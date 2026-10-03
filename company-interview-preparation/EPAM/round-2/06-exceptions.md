Absolutely — **Exceptions next.** 🔥

# 6. Exceptions — from Java basics to production

Don't learn exceptions as merely:

```java
try {
    // code
} catch (Exception e) {
}
```

For a senior backend interview, you need to understand **what actually happens when an exception is thrown, how it propagates, how resources are cleaned up, and how you design error handling in a microservice.**

---

# 1. What is an exception?

An exception represents an abnormal condition during program execution that interrupts the normal flow.

Example:

```java
int result = 10 / 0;
```

This causes:

```text
ArithmeticException
```

Conceptually:

```text
main()
  |
  v
10 / 0
  |
  v
Exception created
  |
  v
Normal execution stops
  |
  v
JVM searches for matching handler
```

---

# 2. Exception hierarchy

This is important.

```text
                    Throwable
                       |
             +---------+---------+
             |                   |
           Error              Exception
             |                   |
       serious JVM/system       |
       problems                 |
                         +------+------+
                         |             |
                  RuntimeException   Checked
                         |
                  unchecked exceptions
```

Examples:

### `Error`

```java
OutOfMemoryError
StackOverflowError
```

These generally indicate serious JVM/runtime problems.

You normally don't write:

```java
catch (OutOfMemoryError e) {
}
```

as normal application recovery logic.

---

### `Exception`

Examples:

```java
IOException
SQLException
RuntimeException
```

---

# 3. Checked vs unchecked exceptions

This is one of the **most frequently asked Java questions**.

## Checked exception

A checked exception is an exception the compiler requires you to either:

1. handle

or

2. declare.

Example:

```java
void readFile() throws IOException {
    Files.readString(path);
}
```

You could instead catch it:

```java
try {
    Files.readString(path);
} catch (IOException e) {
    // handle
}
```

---

# 4. Unchecked exception

Unchecked exceptions extend `RuntimeException`.

Examples:

```java
NullPointerException
IllegalArgumentException
IllegalStateException
IndexOutOfBoundsException
ArithmeticException
```

You aren't required to declare:

```java
void process() throws NullPointerException
```

although technically you can.

---

# 5. Why does Java have checked exceptions?

The idea behind checked exceptions is:

> Some failures are considered conditions that callers should explicitly account for.

For example:

```java
Files.readString(...)
```

may fail because:

- file doesn't exist
- permissions
- filesystem problem
- etc.

The API forces callers to acknowledge that possibility.

Conceptually:

```text
Checked exception
       ↓
Compiler says:
"Deal with this or explicitly propagate it."
```

---

# 6. Why aren't RuntimeExceptions checked?

Consider:

```java
List<String> list = null;

list.size();
```

You get:

```text
NullPointerException
```

Making every possible programming mistake a checked exception would make Java code extremely cumbersome.

Imagine:

```java
void process()
    throws NullPointerException,
           IllegalArgumentException,
           IndexOutOfBoundsException,
           ...
```

That's not useful API design.

So Java separates:

```text
Checked
→ caller is expected to consciously handle/propagate

Unchecked
→ usually programming errors, invalid state, or violations
```

But don't say:

> "Checked exceptions are always recoverable and unchecked are always programmer bugs."

That's too absolute.

Real production systems have gray areas.

---

# 7. `throw` vs `throws`

Very common interview question.

## `throw`

Actually throws an exception object.

```java
throw new IllegalArgumentException("Amount must be positive");
```

Think:

```text
throw
  ↓
perform the action
```

---

## `throws`

Declares that a method may propagate an exception.

```java
void readFile() throws IOException {
}
```

Think:

```text
throws
  ↓
method declaration / contract
```

---

# 8. Example

```java
void withdraw(BigDecimal amount) {

    if (amount.signum() <= 0) {
        throw new IllegalArgumentException(
            "Amount must be positive"
        );
    }
}
```

Here:

```java
throw
```

actually creates/throws the exception.

---

Another example:

```java
void readPaymentFile() throws IOException {
    // ...
}
```

Here:

```java
throws
```

tells the caller:

> "This method may propagate IOException."

---

# 9. Exception propagation

This is extremely important.

Suppose:

```java
void repository() {
    throw new RuntimeException("DB failure");
}

void service() {
    repository();
}

void controller() {
    service();
}
```

Execution:

```text
controller()
    ↓
service()
    ↓
repository()
    ↓
RuntimeException
```

If `repository()` doesn't catch it:

```text
repository
    ↑
service
    ↑
controller
    ↑
caller
```

The exception propagates upward until a matching handler is found.

---

# 10. Stack trace

Suppose:

```java
void repository() {
    throw new RuntimeException("DB failure");
}
```

You might see:

```text
java.lang.RuntimeException: DB failure
    at PaymentRepository.repository(...)
    at PaymentService.service(...)
    at PaymentController.controller(...)
```

This is extremely useful because it shows the call path.

That's why in production you generally want:

```java
log.error("Payment processing failed", e);
```

rather than:

```java
log.error(e.getMessage());
```

The latter can lose the stack trace.

---

# 11. `try-catch`

Basic structure:

```java
try {
    processPayment();
} catch (PaymentException e) {
    handlePaymentFailure(e);
}
```

If the exception occurs:

```text
try
 ↓
exception
 ↓
matching catch
 ↓
continue after catch
```

---

# 12. Multiple catch blocks

Example:

```java
try {
    process();
}
catch (IOException e) {
    // file problem
}
catch (SQLException e) {
    // database problem
}
catch (Exception e) {
    // generic fallback
}
```

Important:

> More specific exceptions must come before more general exceptions.

This is wrong:

```java
catch (Exception e) {
}
catch (IOException e) {
}
```

because `IOException` is already covered by `Exception`.

The compiler rejects it as unreachable.

---

# 13. Multi-catch

Java allows:

```java
try {
    process();
}
catch (IOException | SQLException e) {
    log.error("Infrastructure failure", e);
}
```

Useful when different exceptions require identical handling.

---

# 14. `finally`

`finally` generally executes whether the try succeeds or an exception occurs.

```java
try {
    process();
}
catch (Exception e) {
    handle(e);
}
finally {
    cleanup();
}
```

Conceptually:

```text
success ──────────→ finally
exception → catch → finally
```

This is traditionally used for cleanup.

---

# 15. Nasty interview question: `return` + finally

Look at this:

```java
public int test() {

    try {
        return 10;
    }
    finally {
        System.out.println("finally");
    }
}
```

What happens?

Output:

```text
finally
```

and return value:

```text
10
```

Why?

The method prepares to return `10`, but before leaving the method, the `finally` block executes.

---

# 16. The dangerous version

```java
public int test() {

    try {
        return 10;
    }
    finally {
        return 20;
    }
}
```

Result:

```text
20
```

The `finally` return overrides the earlier return.

### Never do this in production.

It's confusing and can suppress exceptions too.

---

# 17. Even worse

```java
public int test() {

    try {
        throw new RuntimeException("Failure");
    }
    finally {
        return 20;
    }
}
```

What happens?

The exception is effectively suppressed by the `return` in `finally`.

The caller gets:

```text
20
```

instead of the exception.

This is why:

> **Never return from `finally`.**

Excellent interview point.

---

# 18. `try-with-resources`

Now something much more important for backend development.

Suppose you have:

```java
InputStream input = new FileInputStream(file);

try {
    // use input
}
finally {
    input.close();
}
```

This works, but Java provides:

```java
try (InputStream input = new FileInputStream(file)) {
    // use input
}
```

This is **try-with-resources**.

The resource is automatically closed.

---

# 19. What qualifies as a resource?

The object must implement:

```java
AutoCloseable
```

For example:

```java
InputStream
OutputStream
Reader
Writer
Connection
PreparedStatement
ResultSet
```

Many Java I/O and JDBC classes implement `AutoCloseable`.

---

# 20. Multiple resources

You can write:

```java
try (
    Connection connection = dataSource.getConnection();
    PreparedStatement statement =
        connection.prepareStatement(sql);
    ResultSet resultSet =
        statement.executeQuery()
) {

    // process

}
```

When leaving the block, resources are closed.

---

# 21. What order are resources closed?

This is a common interview question.

They are closed in **reverse order of declaration**.

```java
try (
    ResourceA a = ...;
    ResourceB b = ...;
    ResourceC c = ...
) {
}
```

Closing:

```text
C
↓
B
↓
A
```

Think stack:

```text
A opened
B opened
C opened

C closes
B closes
A closes
```

This makes sense because later resources may depend on earlier ones.

---

# 22. The really important part: suppressed exceptions

Suppose:

```java
try (Resource r = ...) {

    throw new RuntimeException("Processing failed");

}
```

And while closing the resource:

```java
r.close();
```

also throws an exception.

Which exception should the caller primarily see?

The original processing exception.

The close exception becomes a **suppressed exception**.

You can inspect them:

```java
catch (Exception e) {

    for (Throwable suppressed : e.getSuppressed()) {
        System.out.println(suppressed);
    }
}
```

This is one of those details that separates a basic answer from a strong Java answer.

---

# 23. Why try-with-resources is better

Compare:

```java
try {
    process();
}
finally {
    resource.close();
}
```

with:

```java
try (Resource resource = ...) {
    process();
}
```

Try-with-resources provides:

- automatic cleanup
- correct closure ordering
- exception handling around close
- suppressed exception support
- less boilerplate

So in modern Java:

> Prefer try-with-resources for `AutoCloseable` resources.

---

# 24. Custom exceptions

In a payment system, you may have:

```java
public class PaymentNotFoundException
        extends RuntimeException {

    public PaymentNotFoundException(String paymentId) {
        super("Payment not found: " + paymentId);
    }
}
```

Then:

```java
Payment payment = repository.findById(id)
    .orElseThrow(() ->
        new PaymentNotFoundException(id)
    );
```

This communicates domain meaning much better than:

```java
throw new RuntimeException("Not found");
```

---

# 25. Exception chaining

Suppose your database layer throws:

```text
SQLException
```

You don't necessarily want your entire application to expose JDBC implementation details.

You can wrap it:

```java
try {
    repository.save(payment);
}
catch (SQLException e) {
    throw new PaymentPersistenceException(
        "Unable to save payment",
        e
    );
}
```

The second argument is the **cause**.

Conceptually:

```text
PaymentPersistenceException
          |
          | caused by
          v
      SQLException
```

Then:

```java
e.getCause()
```

can retrieve the underlying exception.

---

# 26. Why chaining matters

Bad:

```java
catch (SQLException e) {
    throw new PaymentException("Database failed");
}
```

You just threw away useful diagnostic information.

Better:

```java
catch (SQLException e) {
    throw new PaymentException("Database failed", e);
}
```

Now your logs can show:

```text
PaymentException
   caused by
SQLException
   caused by ...
```

---

# 27. Don't catch `Exception` everywhere

This is a very important production principle.

Bad:

```java
try {
    processPayment();
}
catch (Exception e) {
    return "Payment failed";
}
```

Why is this dangerous?

You may accidentally swallow:

- programming bugs
- null pointer errors
- configuration errors
- database failures
- unexpected infrastructure problems

And turn all of them into one generic result.

Instead, catch exceptions where you can **meaningfully recover or translate them**.

---

# 28. Your microservice example

Imagine:

```text
HTTP Request
     ↓
PaymentController
     ↓
PaymentService
     ↓
PaymentRepository
     ↓
MongoDB
```

Suppose MongoDB is unavailable.

You don't necessarily want:

```java
catch (Exception e)
```

at every layer.

Instead:

```text
Mongo exception
     ↓
Repository / driver
     ↓
Service translates if necessary
     ↓
Controller/global handler
     ↓
HTTP response
```

For example:

```text
Database connectivity failure
        ↓
503 Service Unavailable
```

while:

```text
Payment doesn't exist
        ↓
404 Not Found
```

and:

```text
Invalid payment request
        ↓
400 Bad Request
```

The exact mapping depends on your API contract.

---

# 29. Spring `@ControllerAdvice`

In Spring Boot, you can centralize REST exception handling.

```java
@RestControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(PaymentNotFoundException.class)
    public ResponseEntity<?> handleNotFound(
            PaymentNotFoundException e) {

        return ResponseEntity
                .status(HttpStatus.NOT_FOUND)
                .body(e.getMessage());
    }
}
```

Then your controller doesn't need:

```java
try {
   ...
}
catch (PaymentNotFoundException e) {
   ...
}
```

everywhere.

This gives you:

```text
Controller
   ↓
Service
   ↓
Exception
   ↓
@RestControllerAdvice
   ↓
HTTP response
```

We'll go much deeper into this when we reach Spring.

---

# 30. Exception vs HTTP error

Don't confuse these two.

An exception is a **Java/runtime concept**.

HTTP status is an **API protocol concept**.

For example:

```text
PaymentNotFoundException
        ↓
Global exception handler
        ↓
HTTP 404
```

The mapping is your application's decision.

There isn't a rule saying every Java exception automatically becomes a particular HTTP status.

---

# 31. Exception handling + retries

This becomes very important in microservices.

Suppose:

```text
Payment Service
      ↓
Bank Service
```

Bank service temporarily times out.

You might classify the error:

```text
Transient
   ↓
potentially retry
```

versus:

```text
Invalid payment request
   ↓
don't retry
```

For example:

```text
HTTP 400
→ usually don't retry

timeout
→ potentially retry

connection reset
→ potentially retry

business rejection
→ don't blindly retry
```

This connects directly to:

```text
Retry
Circuit Breaker
Timeout
Bulkhead
DLQ
Idempotency
```

which we'll cover under microservices/Kafka.

---

# 32. Exception handling is NOT fault tolerance

This is a very important distinction for your interviews.

If a downstream service fails:

```java
try {
    callBankService();
}
catch (Exception e) {
    return fallback();
}
```

you've handled an exception.

But that doesn't necessarily mean you've implemented proper **fault tolerance**.

Fault tolerance may involve:

```text
Timeout
+
Retry
+
Circuit Breaker
+
Bulkhead
+
Fallback
+
Observability
```

So if an interviewer asks:

> "How would you handle a failing microservice?"

Don't stop at:

> "I'll use try-catch."

That's only one small part.

---

# 33. Interview-ready answer: checked vs unchecked

If asked:

> "What is the difference between checked and unchecked exceptions?"

A strong answer:

> "Checked exceptions are exceptions other than RuntimeException and its subclasses that the compiler requires us to catch or declare. Unchecked exceptions extend RuntimeException and don't have that compile-time requirement. Checked exceptions can be useful when the caller is expected to explicitly handle or propagate a foreseeable failure, while unchecked exceptions are commonly used for programming errors, invalid arguments, or invalid application state. In production, I choose based on API semantics rather than simply categorizing every recoverable error as checked."

That's much better than:

> "Checked = compile time, unchecked = runtime."

Because **all exceptions occur at runtime**.

The difference is whether the compiler enforces handling/propagation.

---

# 34. One more interview trap

What is wrong here?

```java
try {
    process();
}
catch (Exception e) {
    e.printStackTrace();
}
```

In a backend service, this is usually insufficient.

Problems:

- no structured logging
- no useful context
- possibly wrong log destination
- exception may be swallowed
- caller may receive an incorrect success response
- no meaningful error contract

Better:

```java
catch (PaymentException e) {
    log.error(
        "Payment processing failed, paymentId={}",
        paymentId,
        e
    );

    throw e;
}
```

Or translate it appropriately.

---

# 🔥 Exception mental model

Remember this:

```text
                Throwable
                   |
          +--------+--------+
          |                 |
        Error           Exception
                            |
                   +--------+--------+
                   |                 |
             RuntimeException     Checked
                   |
             NPE, IAE, etc.
```

And:

```text
throw
  ↓
actually throws an exception

throws
  ↓
declares possible propagation
```

And:

```text
try-with-resources
        ↓
AutoCloseable
        ↓
automatic cleanup
        ↓
reverse closing order
        ↓
suppressed exceptions
```

And production:

```text
Exception
   ↓
meaningful handling/translation
   ↓
global error handler
   ↓
appropriate API response
   ↓
logs + metrics + tracing
```

---

## Quick test before Generics

What will this print?

```java
public static int test() {

    try {
        System.out.println("try");
        return 10;
    } catch (Exception e) {
        System.out.println("catch");
        return 20;
    } finally {
        System.out.println("finally");
    }
}
```

And then:

```java
System.out.println(test());
```

The output is:

```text
try
finally
10
```

Because there is no exception, `catch` doesn't execute. The `return 10` is prepared, `finally` executes, and then the method returns `10`.

**Next: Generics** — and we'll go beyond `List<String>` into **type erasure, bounded types, wildcards, PECS (`? extends` vs `? super`), generic methods, invariance, and the classic `List<Object>` vs `List<String>` interview traps.**