Absolutely. We move **exactly to Spring AOP** now. 🔥

# Spring Core — AOP ⭐⭐⭐⭐

AOP is important because it connects directly to **Spring proxies and `@Transactional`**, which we'll cover next.

We'll go in this order:

```text
AOP Fundamentals
   ↓
Aspect
   ↓
Join Point
   ↓
Pointcut
   ↓
Advice
   ├── Before
   ├── After
   ├── AfterReturning
   ├── AfterThrowing
   └── Around
   ↓
Spring Proxy
   ↓
JDK Dynamic Proxy
   ↓
CGLIB
   ↓
Self-invocation
   ↓
Practical AOP
```

---

# 1. What is AOP?

**AOP = Aspect-Oriented Programming.**

It is a programming technique used to separate **cross-cutting concerns** from the application's core business logic.

### What is a cross-cutting concern?

A concern that is needed across many unrelated parts of the application.

Examples:

```text
Logging
Security
Transactions
Auditing
Metrics
Tracing
Caching
```

Imagine:

```java
public void transferMoney() {

    log.info("Transfer started");

    authenticateUser();

    beginTransaction();

    // business logic

    commitTransaction();

    log.info("Transfer completed");
}
```

Now imagine having this logic in:

```text
PaymentService
OrderService
UserService
LoanService
AccountService
```

You would duplicate:

```text
logging
security
transaction
metrics
```

everywhere.

That's a problem.

---

# 2. What problem does AOP solve?

Without AOP:

```text
PaymentService
 ├── business logic
 ├── logging
 ├── security
 ├── transaction
 └── metrics

OrderService
 ├── business logic
 ├── logging
 ├── security
 ├── transaction
 └── metrics
```

Business logic becomes mixed with infrastructure concerns.

With AOP:

```text
                 Logging Aspect
                      │
                 Security Aspect
                      │
                Transaction Aspect
                      │
                      ↓
              Business Service
```

The business class focuses on business logic.

---

# 3. What is a Cross-Cutting Concern?

### Interview answer

> A cross-cutting concern is functionality that affects multiple parts of an application but is not part of the core business logic of those components.

Examples:

- Logging
- Authentication/authorization
- Transactions
- Monitoring
- Auditing
- Caching

---

# 🔥 Practical Example

Suppose you have:

```java
@Service
public class PaymentService {

    public void processPayment() {
        // payment logic
    }
}
```

You want to log execution time for every service method.

Without AOP:

```java
public void processPayment() {

    long start = System.currentTimeMillis();

    // payment logic

    long end = System.currentTimeMillis();

    log.info("Execution time = {}", end - start);
}
```

Then you repeat it everywhere.

With AOP, you can define the logging once and apply it to multiple methods.

---

# 4. What is an Aspect?

An **Aspect** is a module containing cross-cutting behavior.

For example:

```java
@Aspect
@Component
public class LoggingAspect {

}
```

The aspect can contain:

```text
When should something execute?
        ↓
Pointcut

What should execute?
        ↓
Advice
```

So:

```text
Aspect
 ├── Pointcut
 └── Advice
```

### Interview answer

> An Aspect encapsulates a cross-cutting concern such as logging, security, auditing, or transaction behavior.

---

# 5. What is a Join Point?

A **Join Point** is a point during program execution where additional behavior can be applied.

In Spring AOP, the important practical limitation is:

> **Spring AOP primarily works with method execution join points.**

For example:

```java
public void processPayment() {
}
```

The execution of this method is a join point that Spring AOP can intercept.

Think:

```text
processPayment()
      ↓
method execution
      ↑
   Join Point
```

### Important interview distinction

AspectJ supports many kinds of join points, including things such as:

```text
method execution
method call
field access
constructor execution
```

But **Spring AOP is proxy-based and focuses on method execution**.

🔥 This distinction is a good senior-level interview point.

---

# 6. What is a Pointcut?

A **Pointcut defines which join points should be intercepted.**

Suppose you have:

```text
PaymentService
OrderService
UserService
```

You may want logging for all service methods.

You could define a pointcut such as:

```java
@Pointcut("execution(* com.example.service..*(..))")
public void serviceMethods() {
}
```

This says approximately:

> Match method executions under `com.example.service` and its subpackages.

So:

```text
Join points
     ↓
Many method executions

Pointcut
     ↓
selects specific ones
```

### Interview answer

> A pointcut is an expression that determines which join points an advice should be applied to.

---

# 7. Join Point vs Pointcut

This is a common interview question.

### Join Point

A possible interception point.

Example:

```java
PaymentService.processPayment()
```

### Pointcut

The rule used to select join points.

Example:

```text
all methods inside service package
```

Think:

```text
Join Points
────────────────────────────
A()
B()
C()
D()
E()

             ↓ Pointcut

A()
C()
E()
```

The pointcut selects which join points should receive advice.

---

# 8. What is Advice?

**Advice is the actual code that executes at the selected join points.**

For example:

```java
@Before("serviceMethods()")
public void logBefore() {
    System.out.println("Method started");
}
```

Here:

```text
Pointcut
    ↓
serviceMethods()

Advice
    ↓
logBefore()
```

So:

> **Pointcut = where**
>
> **Advice = what**

This is one of the most important AOP concepts.

---

# 9. Types of Advice

Spring AOP provides several advice types:

```text
@Before
@After
@AfterReturning
@AfterThrowing
@Around
```

Let's understand each practically.

---

# 10. `@Before`

Runs **before the target method executes**.

```java
@Before("serviceMethods()")
public void before() {
    System.out.println("Before method");
}
```

Flow:

```text
Client
  ↓
Before Advice
  ↓
Target Method
```

Example use cases:

```text
Logging
Authorization checks
Request validation
```

---

# 11. `@After`

Runs after the method execution completes, whether it completes normally or throws an exception.

```java
@After("serviceMethods()")
public void after() {
    System.out.println("Method completed");
}
```

Conceptually:

```text
Target method
     ↓
returns OR throws
     ↓
@After
```

It is similar to a `finally`-style cleanup concept.

---

# 12. `@AfterReturning`

Runs **only when the target method successfully returns**.

```java
@AfterReturning(
    pointcut = "serviceMethods()",
    returning = "result"
)
public void afterReturning(Object result) {

    System.out.println("Result = " + result);
}
```

Flow:

```text
Target method
     ↓
success
     ↓
return value
     ↓
@AfterReturning
```

If the method throws an exception:

```text
@AfterReturning
       ↓
NOT executed
```

---

# 13. `@AfterThrowing`

Runs when the target method throws an exception.

```java
@AfterThrowing(
    pointcut = "serviceMethods()",
    throwing = "ex"
)
public void afterThrowing(Exception ex) {

    System.out.println("Exception = " + ex);
}
```

Flow:

```text
Target method
     ↓
exception
     ↓
@AfterThrowing
```

Useful for:

```text
Error logging
Auditing
Metrics
Alerting
```

---

# 14. `@Around` ⭐⭐⭐⭐⭐

This is the **most powerful advice type**.

It can execute:

```text
before method
        ↓
method
        ↓
after method
```

Example:

```java
@Around("serviceMethods()")
public Object around(ProceedingJoinPoint joinPoint)
        throws Throwable {

    long start = System.currentTimeMillis();

    Object result = joinPoint.proceed();

    long end = System.currentTimeMillis();

    System.out.println(
        "Execution time = " + (end - start)
    );

    return result;
}
```

The important line is:

```java
joinPoint.proceed();
```

That means:

> Continue execution to the intercepted target method.

---

# 🔥 What happens if you don't call `proceed()`?

This is an excellent interview trap.

Suppose:

```java
@Around("serviceMethods()")
public Object around(ProceedingJoinPoint joinPoint) {

    System.out.println("Before");

    return null;
}
```

You **never call**:

```java
joinPoint.proceed();
```

Therefore the target method may never execute.

Flow becomes:

```text
Client
  ↓
Around Advice
  ↓
return
  X
Target method never executes
```

That's why `@Around` must be handled carefully.

---

# 15. Why is `@Around` powerful?

Because it can:

### Execute before

```java
logBefore();
```

### Decide whether the method executes

```java
if (allowed) {
    joinPoint.proceed();
}
```

### Modify arguments

It can potentially proceed with different arguments depending on the interception mechanism/design.

### Modify the return value

```java
Object result = joinPoint.proceed();

return modify(result);
```

### Handle exceptions

```java
try {
    return joinPoint.proceed();
} catch (Exception e) {
    // handle/log
    throw e;
}
```

### Measure execution time

```java
long start = ...
Object result = joinPoint.proceed();
long end = ...
```

---

# 16. Complete Advice Flow

Suppose:

```java
@Around(...)
```

and:

```java
@Before(...)
```

etc.

Conceptually:

```text
                 Client
                   │
                   ↓
              Spring Proxy
                   │
                   ↓
             @Around - before
                   │
                   ↓
                @Before
                   │
                   ↓
              Target Method
                   │
          ┌────────┴────────┐
          ↓                 ↓
       success            exception
          ↓                 ↓
 @AfterReturning       @AfterThrowing
          │                 │
          └────────┬────────┘
                   ↓
                @After
                   ↓
             @Around - after
                   ↓
                Client
```

The exact nesting/order can depend on multiple aspects and their ordering, so don't memorize this as a universal ordering rule. The key idea is understanding what each advice type guarantees.

---

# 17. Complete Practical Example

```java
@Aspect
@Component
public class LoggingAspect {

    @Around(
        "execution(* com.example.service..*(..))"
    )
    public Object logExecution(
            ProceedingJoinPoint joinPoint)
            throws Throwable {

        String method =
            joinPoint.getSignature().toShortString();

        long start = System.currentTimeMillis();

        try {

            System.out.println(
                "Starting: " + method
            );

            Object result =
                joinPoint.proceed();

            return result;

        } finally {

            long time =
                System.currentTimeMillis() - start;

            System.out.println(
                "Completed: " + method +
                " in " + time + " ms"
            );
        }
    }
}
```

Now:

```java
@Service
public class PaymentService {

    public void processPayment() {

        System.out.println("Processing payment");
    }
}
```

When:

```java
paymentService.processPayment();
```

the conceptual flow is:

```text
Caller
   ↓
Spring Proxy
   ↓
LoggingAspect
   ↓
joinPoint.proceed()
   ↓
PaymentService.processPayment()
   ↓
return
   ↓
LoggingAspect
   ↓
Caller
```

**The service doesn't contain the logging code.**

That's the real value of AOP.

---

# 🔥 Senior Interview Question

### "Where would you use AOP in a real application?"

Good answer:

> I would use AOP for cross-cutting concerns that apply consistently across multiple components, such as logging, execution-time metrics, auditing, authorization checks, tracing, and transaction management. I would avoid putting core business rules into AOP because that can make the application flow harder to understand and debug.

That's an important distinction.

**AOP is not an excuse to hide business logic.**

---

# 18. AOP vs OOP

### OOP

Organizes code around:

```text
Objects
Classes
Responsibilities
```

### AOP

Organizes cross-cutting behavior around:

```text
Aspects
Pointcuts
Advice
```

They complement each other.

```text
OOP
 ↓
Business responsibilities

AOP
 ↓
Cross-cutting responsibilities
```

---

# 19. Spring AOP vs AspectJ

Another useful interview distinction.

### Spring AOP

Primarily:

```text
Proxy-based
Runtime interception
Method execution
```

### AspectJ

More powerful weaving model:

```text
Compile-time weaving
Load-time weaving
More join-point types
```

For normal Spring applications, you'll commonly encounter:

```text
Spring AOP
   ↓
Proxy
   ↓
Method interception
```

---

# 🔥 The key concepts you should now know

```text
Cross-cutting concern
        ↓
       Aspect
        ↓
     Pointcut
        ↓
Selects join points
        ↓
      Advice
        ↓
 ┌──────┼─────────┐
 ↓      ↓         ↓
Before  After    Around
        ↓
AfterReturning
AfterThrowing
```

And remember the simplest interview definitions:

> **Aspect** → encapsulates cross-cutting behavior.

> **Join Point** → a point during execution where interception can occur; Spring AOP primarily deals with method execution.

> **Pointcut** → selects which join points should be intercepted.

> **Advice** → code that executes at selected join points.

> **`@Around`** → wraps the target invocation and controls whether/how `proceed()` is called.

---

### Next in the exact path: **Spring Proxy**

This is where AOP becomes really important for your EPAM interview:

```text
Why does Spring need a proxy?
        ↓
How does proxy interception work?
        ↓
JDK Dynamic Proxy
        ↓
CGLIB
        ↓
How @Transactional uses the proxy
        ↓
Self-invocation problem
```

That is the part we should go deep on next.