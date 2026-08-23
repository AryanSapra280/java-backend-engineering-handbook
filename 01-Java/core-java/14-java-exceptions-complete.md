# 14 — Java Exceptions

> Interview-focused Java Exceptions notes. This chapter covers the complete Java-level exception flow. Spring-specific exception handling such as `@ControllerAdvice`, `@ExceptionHandler`, `@Valid`, and `@Validated` is intentionally parked for the Spring phase.

---

# 1. What Is an Exception?

An exception represents an abnormal condition during program execution that interrupts the normal flow.

Example:

```java
int result = 10 / 0;
```

This causes:

```text
ArithmeticException
```

Java creates/throws an exception instead of continuing normal execution.

Mental model:

```text
Normal execution
      ↓
Abnormal condition
      ↓
Exception
      ↓
Normal flow interrupted
      ↓
Handle OR propagate
```

---

# 2. Exception Hierarchy ⭐

The important hierarchy is:

```text
Object
   ↓
Throwable
   ├── Error
   │
   └── Exception
       ├── RuntimeException
       │   ├── NullPointerException
       │   ├── IllegalArgumentException
       │   ├── IllegalStateException
       │   └── ...
       │
       └── Checked Exceptions
           ├── IOException
           ├── SQLException
           └── ...
```

Remember:

```text
Throwable
 ├── Error
 └── Exception
      ├── RuntimeException
      └── checked exceptions
```

---

# 3. `Error` vs `Exception`

## Error

`Error` generally represents serious JVM/system-level problems that application code is not normally expected to recover from.

Examples:

```text
OutOfMemoryError
StackOverflowError
```

Normally, application code should not build its normal recovery strategy around catching `Error`.

For example, this is generally not appropriate:

```java
try {
    // application logic
} catch (OutOfMemoryError e) {
    // try to continue application normally
}
```

---

## Exception

`Exception` represents conditions that application code may reasonably handle, propagate, translate, or recover from.

Examples:

```text
IOException
SQLException
IllegalArgumentException
NullPointerException
```

---

# 4. Checked vs Unchecked Exceptions ⭐⭐⭐

The key hierarchy is:

```text
Exception
├── RuntimeException
│     ↓
│   Unchecked
│
└── Other Exception subclasses
      ↓
    Checked
```

## Checked Exception

A checked exception is one the compiler requires you to **handle or declare**.

Example:

```java
void readFile() throws IOException {
    // ...
}
```

The caller must either handle it:

```java
try {
    readFile();
} catch (IOException e) {
    // handle
}
```

or declare it further:

```java
void process() throws IOException {
    readFile();
}
```

The compiler enforces this.

---

## Unchecked Exception

`RuntimeException` and its subclasses are unchecked.

Example:

```java
void process(String name) {
    System.out.println(name.length());
}
```

If:

```text
name = null
```

then:

```text
NullPointerException
```

can occur.

The compiler does not force:

```java
throws NullPointerException
```

---

# 5. Why Does Java Have Checked Exceptions?

The design idea is that some failures are expected enough that the caller should explicitly decide what to do.

Examples:

```text
File doesn't exist
Network I/O fails
Database operation fails
```

The caller may need to decide:

```text
retry?
fallback?
return error?
propagate?
```

Java can therefore force the caller to acknowledge these possibilities.

---

# 6. Why Are Runtime Exceptions Unchecked?

Runtime exceptions commonly indicate:

```text
programming error
invalid argument
invalid state
unexpected programming condition
```

Examples:

```java
list.get(100);
```

may produce:

```text
IndexOutOfBoundsException
```

or:

```java
method(null);
```

may result in:

```text
NullPointerException
```

or:

```java
if (age < 0) {
    throw new IllegalArgumentException("Age cannot be negative");
}
```

However, do NOT memorize:

```text
checked = recoverable
unchecked = unrecoverable
```

That is too simplistic.

The safest interview distinction is:

> **Checked exceptions are compiler-enforced; unchecked exceptions are not.**

Both can technically be caught and handled.

---

# 7. `throw` vs `throws` ⭐

## `throw`

`throw` actually throws an exception.

```java
throw new IllegalArgumentException(
    "Age cannot be negative"
);
```

Think:

```text
throw
 ↓
actually perform the throw
```

---

## `throws`

`throws` declares that a method may propagate an exception.

```java
void readFile() throws IOException {
}
```

Think:

```text
throws
 ↓
method declaration
 ↓
"I may propagate this exception"
```

---

## Together

```java
void processFile(String path) throws IOException {

    if (path == null) {
        throw new IllegalArgumentException(
            "Path cannot be null"
        );
    }

    // file processing
}
```

Here:

```text
throws IOException
    → declaration

throw new IllegalArgumentException(...)
    → actually throws an exception
```

---

# 8. Custom Exceptions ⭐

Backend applications often have meaningful domain failures:

```text
Payment not found
Payment already processed
Insufficient balance
Invalid payment state
```

Instead of throwing generic:

```java
throw new RuntimeException(...);
```

everywhere, you can create meaningful exceptions.

Example:

```java
public class PaymentNotFoundException
        extends RuntimeException {

    public PaymentNotFoundException(String message) {
        super(message);
    }
}
```

Usage:

```java
Payment payment = repository.findById(id)
    .orElseThrow(() ->
        new PaymentNotFoundException(
            "Payment not found: " + id
        )
    );
```

This gives the failure a meaningful application/domain identity.

---

# 9. Should Every Exception Have Its Own Class?

No.

Don't create dozens of exception classes without a meaningful distinction.

Create a custom exception when the application needs to distinguish the failure for something such as:

```text
different handling
logging
API response mapping
retry behavior
business behavior
domain semantics
```

For example:

```text
PaymentNotFoundException
PaymentAlreadyProcessedException
InvalidPaymentStateException
```

may be useful.

But unnecessary exception classes create noise.

---

# 10. Exception Propagation ⭐

Consider a backend call chain:

```text
Controller
    ↓
Service
    ↓
Repository
```

Suppose the Repository throws an exception:

```text
DatabaseException
```

The exception can propagate upward:

```text
Repository
    ↓
Service
    ↓
Controller / appropriate boundary
```

Every layer does NOT need a `try/catch`.

---

# 11. When Should You Catch?

The important design principle:

> **Don't catch an exception merely because you can. Catch it when that layer can meaningfully handle, recover, translate, or add useful context.**

Bad pattern:

```java
try {
    repository.save(payment);
} catch (Exception e) {
    // nothing useful can be done
}
```

This can hide failures.

Better:

```text
Repository
    ↓
exception
    ↓
Service
    ↓
Can service recover/translate/add context?
    ├── yes → handle appropriately
    └── no  → propagate
```

---

# 12. Exception Translation

A lower-level implementation exception does not always need to leak into the business layer.

For example:

```text
Database/JPA exception
        ↓
Data access layer
        ↓
application/domain exception
```

Or:

```text
Payment gateway SDK exception
        ↓
PaymentGatewayException
```

The goal can be to avoid coupling business logic to low-level implementation details.

---

# 13. Production Scenario — Invalid Input

Suppose:

```text
POST /payments
```

Request:

```json
{
  "amount": -500
}
```

This is an input/business validation problem, not a database failure.

At the Java level, you might conceptually represent the failure with:

```java
throw new IllegalArgumentException(
    "Amount must be positive"
);
```

Spring validation annotations such as `@Valid` and `@Validated` are intentionally covered later in the Spring phase.

---

# 14. Production Scenario — Database Failure

Suppose:

```text
Client
  ↓
PaymentService
  ↓
Database
```

The client sends a valid request, but the database connection times out.

This is different from invalid input:

```text
valid request
    ↓
application
    ↓
database unavailable
```

You should not expose low-level details such as:

```text
SQLException: connection timeout...
```

directly as an API contract.

Instead, the exception can propagate to an appropriate boundary where the application can translate it into the correct external response.

Spring-specific global API exception handling will be covered later.

---

# 15. `try / catch / finally`

Basic structure:

```java
try {
    // risky code
} catch (Exception e) {
    // handle exception
} finally {
    // cleanup
}
```

Mental model:

```text
try
 ↓
exception?
 ├── NO → finally
 │
 └── YES
      ↓
   matching catch
      ↓
   finally
```

`finally` normally executes whether the operation succeeds or an exception occurs.

---

# 16. No Exception

```java
try {
    int x = 10 / 2;
    System.out.println(x);
} catch (ArithmeticException e) {
    System.out.println("Error");
} finally {
    System.out.println("Cleanup");
}
```

Flow:

```text
try
 ↓
10 / 2 succeeds
 ↓
catch skipped
 ↓
finally
```

Output:

```text
5
Cleanup
```

---

# 17. Exception Occurs

```java
try {
    int x = 10 / 0;
} catch (ArithmeticException e) {
    System.out.println("Division error");
} finally {
    System.out.println("Cleanup");
}
```

Flow:

```text
try
 ↓
ArithmeticException
 ↓
matching catch
 ↓
finally
```

---

# 18. Multiple Catch Blocks

```java
try {
    // risky code
} catch (ArithmeticException e) {
    // ...
} catch (NullPointerException e) {
    // ...
} catch (Exception e) {
    // ...
}
```

Java selects the **first matching catch block**.

---

# 19. Catch Ordering ⭐

Specific exceptions must come before general exceptions.

Invalid:

```java
try {
    // ...
} catch (Exception e) {
    // general
} catch (NullPointerException e) {
    // specific
}
```

Why?

```text
Exception
   ↑
NullPointerException
```

`Exception` already catches `NullPointerException`.

Correct:

```java
try {
    // ...
} catch (NullPointerException e) {
    // specific
} catch (Exception e) {
    // general
}
```

Rule:

```text
Specific
   ↓
General
```

---

# 20. Multi-Catch

When multiple exceptions require exactly the same handling:

```java
try {
    // ...
} catch (IOException | SQLException e) {
    log.error("Operation failed", e);
}
```

This avoids duplicated handling code.

Use it only when the handling is genuinely the same.

---

# 21. What Happens After Catch?

Consider:

```java
try {
    riskyOperation();
} catch (Exception e) {
    recover();
}

continueProcessing();
```

If `riskyOperation()` throws:

```text
try
 ↓
exception
 ↓
catch
 ↓
continueProcessing()
```

Java does not return to the exact line that failed inside the `try`.

After the catch completes, execution continues after the try/catch structure.

---

# 22. `finally`

`finally` is generally used for cleanup that should happen regardless of success or failure.

Historically:

```java
Connection connection = null;

try {
    connection = getConnection();
    // use connection
} finally {
    if (connection != null) {
        connection.close();
    }
}
```

The goal:

```text
Acquire
  ↓
Use
  ↓
Success OR failure
  ↓
Cleanup
```

For `AutoCloseable` resources, modern Java generally prefers try-with-resources.

---

# 23. `return` + `finally` ⭐

Consider:

```java
static int test() {
    try {
        return 10;
    } finally {
        System.out.println("finally");
    }
}
```

Flow:

```text
try
 ↓
prepare return 10
 ↓
finally executes
 ↓
return 10
```

Output:

```text
finally
```

Returned value:

```text
10
```

So `finally` executes even when `try` has a return.

---

# 24. `return` Inside `finally` — Dangerous ⭐⭐⭐

```java
static int test() {
    try {
        return 10;
    } finally {
        return 20;
    }
}
```

Result:

```text
20
```

The return in `finally` overrides the earlier return.

Avoid returning from `finally`.

---

# 25. `finally` Can Suppress an Exception

```java
static void test() {
    try {
        throw new RuntimeException("original");
    } finally {
        return;
    }
}
```

The return in `finally` suppresses the original exception.

Mental model:

```text
try
 ↓
RuntimeException
 ↓
finally
 ↓
return
 ↓
original exception lost
```

Interview-quality statement:

> A `return` inside `finally` can override a return value or suppress an exception from `try`/`catch`, so it should generally be avoided.

---

# 26. Does `finally` Always Execute?

The safe answer:

> `finally` normally executes whether the `try` succeeds, throws, or returns.

Don't say it literally executes under every possible circumstance.

For example:

```java
System.exit(0);
```

terminates the JVM before normal execution can continue.

So:

> `finally` normally executes unless execution is terminated before it can run.

---

# 27. Nested Try/Catch

Possible:

```java
try {
    try {
        riskyOperation();
    } catch (IOException e) {
        handleIO();
    }
} catch (Exception e) {
    handleOther();
}
```

But don't introduce nesting without a reason.

Prefer clear exception boundaries.

---

# 28. Try-with-Resources ⭐⭐⭐

Before try-with-resources:

```java
FileInputStream input = null;

try {
    input = new FileInputStream("data.txt");
    // use input
} finally {
    if (input != null) {
        input.close();
    }
}
```

Modern Java:

```java
try (FileInputStream input =
         new FileInputStream("data.txt")) {

    // use input
}
```

Java automatically closes the resource.

Benefits:

```text
less boilerplate
less chance of forgetting cleanup
clearer resource lifecycle
safer exception handling
```

---

# 29. `AutoCloseable`

The resource used by try-with-resources must implement:

```java
AutoCloseable
```

or a related interface such as `Closeable`.

Conceptually:

```text
AutoCloseable
     ↑
Closeable
     ↑
FileInputStream
```

Java knows that it can call:

```java
close()
```

---

# 30. Try-with-Resources Flow

```java
try (FileInputStream input =
         new FileInputStream("data.txt")) {

    process(input);
}
```

Conceptually:

```text
create resource
      ↓
execute body
      ↓
close resource
      ↓
continue / catch / finally
```

The resource is closed when leaving the try-with-resources block, including when leaving because of an exception.

---

# 31. Multiple Resources

```java
try (
    FileInputStream input =
        new FileInputStream("input.txt");

    FileOutputStream output =
        new FileOutputStream("output.txt")
) {
    copy(input, output);
}
```

Resources are closed in **reverse order of declaration**.

Created:

```text
input
 ↓
output
```

Closed:

```text
output
 ↓
input
```

This is useful when resources have dependencies.

---

# 32. Try-with-Resources + Catch

```java
try (FileInputStream input =
         new FileInputStream("data.txt")) {

    process(input);

} catch (IOException e) {

    // handle
}
```

Conceptually:

```text
create resource
      ↓
try body
      ↓
resource cleanup
      ↓
matching catch
```

---

# 33. Try-with-Resources + Finally

```java
try (FileInputStream input =
         new FileInputStream("data.txt")) {

    process(input);

} catch (IOException e) {

    handle(e);

} finally {

    additionalCleanup();
}
```

The resource is still managed automatically.

`finally` can be used for other cleanup/logic.

---

# 34. Suppressed Exceptions ⭐⭐

Suppose:

```text
main operation throws Exception A
```

and then while closing the resource:

```text
close() throws Exception B
```

Try-with-resources keeps the main operation's exception as primary:

```text
Exception A
    ↓
PRIMARY
```

and records the close exception as suppressed:

```text
Exception B
    ↓
SUPPRESSED
```

You can inspect them:

```java
catch (Exception e) {
    Throwable[] suppressed = e.getSuppressed();
}
```

Mental model:

```text
Main operation
      ↓
Exception A  ← primary

Resource close
      ↓
Exception B  ← suppressed
```

---

# 35. `finally` vs Try-with-Resources

Manual cleanup:

```java
try {
    resource.use();
} finally {
    resource.close();
}
```

Modern approach:

```java
try (Resource resource = createResource()) {
    resource.use();
}
```

Prefer:

> **try-with-resources for resources implementing `AutoCloseable`.**

Use `finally` for cleanup or actions that aren't better represented as a managed resource.

---

# 36. Production Scenario — File Processing

Requirement:

> Read a file and guarantee that its handle is closed even if processing fails.

Use:

```java
try (InputStream input = openFile()) {
    process(input);
}
```

Instead of manually relying on:

```java
input.close();
```

---

# 37. Production Scenario — JDBC Resources

When directly working with JDBC:

```java
try (
    Connection connection = dataSource.getConnection();
    PreparedStatement statement =
        connection.prepareStatement(sql);
    ResultSet resultSet =
        statement.executeQuery()
) {
    // process result
}
```

The resources are automatically closed.

In Spring applications, Spring commonly manages much of this lifecycle for you, but understanding the underlying Java mechanism is still important.

---

# 38. Production Scenario — Exception Boundary

Suppose:

```text
Controller
    ↓
PaymentService
    ↓
PaymentRepository
```

The repository gets a database timeout.

Do not automatically add:

```java
try {
    repository.save(payment);
} catch (Exception e) {
    // empty
}
```

Instead ask:

```text
Can this layer:
    recover?
    retry?
    translate?
    add useful context?
```

If not, allow propagation to the appropriate boundary.

Spring-specific centralized API handling will be covered later.

---

# 39. Common Bad Patterns

## Catching everything

```java
try {
    process();
} catch (Exception e) {
    // ignore
}
```

Problems:

```text
errors disappear
debugging becomes difficult
failure semantics are lost
callers may believe the operation succeeded
```

---

## Catching and rethrowing without value

```java
try {
    process();
} catch (Exception e) {
    throw e;
}
```

If you're not adding context, translating, recovering, or changing behavior, this catch may be unnecessary.

---

## Returning from finally

```java
try {
    return 10;
} finally {
    return 20;
}
```

Avoid.

---

# 40. Complete Exception Mental Model

```text
                         Throwable
                            │
                 ┌──────────┴──────────┐
                 ↓                     ↓
               Error               Exception
                                       │
                         ┌─────────────┴─────────────┐
                         ↓                           ↓
                  RuntimeException              Checked
                         ↓                           ↓
                    Unchecked                    Compiler
                                                   enforced
```

Exception handling:

```text
                    try
                     │
             ┌───────┴────────┐
             │                │
          success          exception
             │                │
             │                ↓
             │          matching catch
             │                │
             └───────┬────────┘
                     ↓
                  finally
                     ↓
                  continue
```

Resource management:

```text
try-with-resources
        ↓
create resource
        ↓
use resource
        ↓
automatic close
        ↓
catch/finally/continue
```

---

# 41. Important Rules to Remember

```text
1. Throwable
      ├── Error
      └── Exception

2. RuntimeException → unchecked

3. Other Exception subclasses → checked

4. throw
   → actually throws an exception

5. throws
   → declares possible propagation

6. Specific catch before general catch

7. finally normally executes even with return

8. Never return from finally

9. try-with-resources is preferred for AutoCloseable resources

10. Multiple resources close in reverse declaration order

11. Try-with-resources can retain close failures
    as suppressed exceptions

12. Don't catch exceptions unless the layer can
    meaningfully handle, recover, translate,
    or add useful context

13. Custom exceptions should represent meaningful
    application/domain distinctions

14. Don't expose low-level implementation exceptions
    as your external API contract
```

---

# 42. Interview Questions

## Fundamentals

1. What is an exception?
2. Explain the Java exception hierarchy.
3. Difference between `Error` and `Exception`.
4. Difference between checked and unchecked exceptions.
5. Why does Java have checked exceptions?
6. Is every unchecked exception unrecoverable?
7. Difference between `throw` and `throws`.

## Custom exceptions

8. When would you create a custom exception?
9. Should every business error have a separate exception?
10. Why might you translate a low-level exception into a domain exception?

## Propagation

11. What is exception propagation?
12. Should every service method have try/catch?
13. When should you catch an exception?
14. Why is `catch (Exception e)` often problematic?

## `try/catch/finally`

15. What happens when no exception occurs?
16. What happens when an exception occurs?
17. How does Java select among multiple catch blocks?
18. Why must specific catches come before general catches?
19. What is multi-catch?
20. Does `finally` execute when `try` returns?
21. What happens if `finally` also returns?
22. Can `finally` suppress an exception?
23. Does `finally` literally always execute?

## Try-with-resources

24. What is try-with-resources?
25. What is `AutoCloseable`?
26. Why is try-with-resources better than manual cleanup?
27. Can you declare multiple resources?
28. In what order are multiple resources closed?
29. What happens if both the body and `close()` throw?
30. What are suppressed exceptions?
31. How do you inspect suppressed exceptions?

---

# 43. Spring Topics — PARKED

These are intentionally NOT part of Java Exceptions:

```text
@RestControllerAdvice
@ExceptionHandler
@Valid
@Validated
Bean Validation
HTTP status mapping
API error response design
Spring exception translation
```

They belong in the Spring phase.

---

# 44. Future Advanced Java Exception Topics — PARKED

Lower priority for current SDE-2 preparation:

```text
deep JVM exception internals
bytecode-level exception tables
custom JVM Error recovery
advanced exception performance analysis
```

Don't spend interview-prep time here unless a specific interview requires it.

---

# Final Interview Summary

If asked:

> "Explain exception handling in Java."

A strong answer:

> "Java exceptions derive from Throwable, which has Error and Exception branches. RuntimeException and its subclasses are unchecked, while other Exception subclasses are generally checked and must be handled or declared. `throw` actually raises an exception, while `throws` declares that a method may propagate one. Exceptions can propagate through application layers until a layer can meaningfully handle or translate them. `try/catch/finally` controls exception handling and cleanup, while try-with-resources provides automatic cleanup for AutoCloseable resources. With multiple resources, they are closed in reverse declaration order, and close failures can be recorded as suppressed exceptions."

---

# Chapter Status

```text
Java Exceptions
├── Hierarchy                    ✅
├── Error vs Exception           ✅
├── Checked vs Unchecked         ✅
├── throw vs throws              ✅
├── Custom Exceptions            ✅
├── Exception Propagation        ✅
├── try/catch                    ✅
├── Multiple catch               ✅
├── Catch ordering               ✅
├── Multi-catch                  ✅
├── finally                      ✅
├── return + finally             ✅
├── Try-with-resources            ✅
├── AutoCloseable                ✅
├── Multiple resources           ✅
├── Suppressed exceptions        ✅
└── Production scenarios         ✅

JAVA EXCEPTIONS = DONE ✅
```
