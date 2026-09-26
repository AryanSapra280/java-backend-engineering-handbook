# ⚙️ Spring AOP — Part 1

This one is usually **easy marks** if you understand the terminology and the proxy concept.

---

## 1. What is AOP?

**AOP = Aspect-Oriented Programming.**

It is used to separate **cross-cutting concerns** from business logic.

Examples:

```text
Logging
Security
Transactions
Auditing
Metrics
Caching
```

Without AOP:

```java
public void transfer() {
    log.info("start");

    authenticate();

    beginTransaction();

    // business logic

    commitTransaction();

    log.info("end");
}
```

With AOP:

```java
public void transfer() {
    // only business logic
}
```

Cross-cutting behavior is handled separately.

### Interview answer

> AOP allows us to modularize cross-cutting concerns such as logging, security and transactions so that they don't have to be repeated throughout business logic.

---

# 2. What is a cross-cutting concern?

A concern that affects **multiple parts of an application**.

Example:

```text
Service A → logging
Service B → logging
Service C → logging
Service D → logging
```

Instead of implementing logging separately everywhere, AOP can centralize it.

Common examples:

```text
Logging
Security
Transactions
Auditing
Caching
Performance monitoring
```

---

# 3. What is an Aspect?

An **Aspect** contains the cross-cutting logic.

Example:

```java
@Aspect
@Component
public class LoggingAspect {

}
```

It can define:

```text
When should something run?
What should run?
```

For example:

```java
@Before(...)
public void log() {
    ...
}
```

---

# 4. What is a Join Point?

A **Join Point** is a point during program execution where an aspect can potentially be applied.

In Spring AOP, the important practical join points are **method executions**.

Example:

```java
public void createAccount() {
}
```

The execution of this method can be a join point.

### Interview trap

Full AspectJ supports more kinds of join points, but **Spring AOP is proxy-based and primarily works with method execution join points**.

---

# 5. What is a Pointcut?

A **Pointcut defines which join points should be intercepted.**

Example:

```java
@Pointcut("execution(* com.example.service.*.*(..))")
public void serviceMethods() {
}
```

Meaning roughly:

> Match methods in the service package.

Think:

```text
Join Point → possible interception point

Pointcut → which ones do I actually select?
```

---

# 6. What is Advice?

**Advice is the actual code that runs at the selected join points.**

Common types:

```text
@Before
@After
@AfterReturning
@AfterThrowing
@Around
```

So:

```text
Aspect
 ├── Pointcut → WHERE?
 └── Advice   → WHAT?
```

---

# 7. `@Before`

Runs **before** the target method.

```java
@Before("execution(* com.example.service.*.*(..))")
public void before() {
    System.out.println("Before method");
}
```

Flow:

```text
Request
  ↓
@Before
  ↓
Target method
```

Use cases:

```text
Logging
Validation
Security checks
```

---

# 8. `@After`

Runs after the method execution finishes, regardless of whether it returns normally or throws an exception.

```java
@After("execution(* com.example.service.*.*(..))")
public void after() {
    System.out.println("Method completed");
}
```

Conceptually similar to a `finally` block.

---

# 9. `@AfterReturning`

Runs only when the method completes successfully.

```java
@AfterReturning(
    pointcut = "execution(* com.example.service.*.*(..))",
    returning = "result"
)
public void afterReturning(Object result) {
    System.out.println(result);
}
```

Flow:

```text
Method
  ↓
Success
  ↓
@AfterReturning
```

---

# 10. `@AfterThrowing`

Runs when the target method throws an exception.

```java
@AfterThrowing(
    pointcut = "execution(* com.example.service.*.*(..))",
    throwing = "ex"
)
public void afterThrowing(Exception ex) {
    log.error("Error", ex);
}
```

Useful for:

```text
Error logging
Auditing failures
Metrics
```

---

# 11. What is `@Around`?

This is the **most powerful advice**.

It can execute code:

```text
before
+
after
```

and can even control whether the target method executes.

Example:

```java
@Around("execution(* com.example.service.*.*(..))")
public Object around(ProceedingJoinPoint pjp)
        throws Throwable {

    long start = System.currentTimeMillis();

    Object result = pjp.proceed();

    long end = System.currentTimeMillis();

    log.info("Time = {}", end - start);

    return result;
}
```

Flow:

```text
        @Around
       /       \
before         after
      \       /
      target method
```

---

# 12. What is `ProceedingJoinPoint`?

Used with `@Around`.

```java
pjp.proceed();
```

means:

> Continue execution to the intercepted target method.

If you don't call:

```java
pjp.proceed();
```

the target method may **never execute**.

This is a classic interview question.

---

# 13. Can `@Around` modify arguments?

Yes.

Conceptually:

```java
Object[] args = pjp.getArgs();
```

You can modify arguments and proceed with modified arguments:

```java
pjp.proceed(modifiedArgs);
```

It can also modify the return value.

That's why `@Around` is more powerful than the other advice types.

---

# 14. What is `@Pointcut`?

Instead of repeating the expression:

```java
execution(* com.example.service.*.*(..))
```

define it once:

```java
@Pointcut("execution(* com.example.service.*.*(..))")
public void serviceMethods() {}
```

Then:

```java
@Before("serviceMethods()")
public void before() {
}
```

and:

```java
@After("serviceMethods()")
public void after() {
}
```

Cleaner and reusable.

---

# 15. Example: Logging Aspect

```java
@Aspect
@Component
public class LoggingAspect {

    @Around("execution(* com.example.service.*.*(..))")
    public Object logExecution(
            ProceedingJoinPoint pjp) throws Throwable {

        long start = System.currentTimeMillis();

        try {
            return pjp.proceed();
        } finally {
            long time =
                System.currentTimeMillis() - start;

            System.out.println(
                pjp.getSignature() +
                " took " + time + " ms"
            );
        }
    }
}
```

Now:

```java
accountService.createAccount();
```

automatically gets logging/timing behavior.

---

# 16. How does Spring AOP actually work?

This is the **most important concept**.

Spring AOP generally uses **proxies**.

Suppose:

```java
@Service
public class PaymentService {

    public void pay() {
        // business logic
    }
}
```

You inject:

```java
PaymentService service;
```

What you interact with may actually be:

```text
Caller
  ↓
Spring Proxy
  ↓
Aspect
  ↓
Actual PaymentService
```

So:

```text
service.pay()
```

may actually mean:

```text
Proxy.pay()
    ↓
AOP logic
    ↓
Real object.pay()
```

---

# 17. Why is this important?

Because it explains one of the biggest Spring interview traps:

# Self-invocation

Suppose:

```java
@Service
public class PaymentService {

    public void methodA() {
        methodB();
    }

    @Transactional
    public void methodB() {
    }
}
```

You might expect:

```text
methodA
 ↓
@Transactional methodB
```

to activate the transaction.

But:

```java
methodB();
```

is effectively:

```java
this.methodB();
```

The call doesn't go through the Spring proxy.

Therefore the proxy-based advice may **not execute**.

---

# 18. How do you solve self-invocation?

Common approaches:

### Move the method to another Spring bean

```text
Service A
   ↓
Service B
   ↓
@Transactional method
```

Now the call crosses the proxy.

### Or inject/use the proxied bean appropriately

But the cleanest solution is usually:

> Separate responsibilities into another bean rather than relying on self-invocation.

---

# 19. JDK Proxy vs CGLIB Proxy

Another common interview question.

### JDK Dynamic Proxy

Works through an **interface**.

Example:

```java
interface PaymentService {
    void pay();
}
```

Spring can create a proxy implementing the interface.

```text
Caller
 ↓
JDK Proxy
 ↓
Implementation
```

### CGLIB

Creates a subclass of the target class.

```text
PaymentService
      ↑
CGLIB subclass proxy
```

Modern Spring commonly uses class-based proxying when configured/appropriate.

### Important limitation

CGLIB subclass proxies cannot advise methods that cannot be overridden, such as:

```text
final methods
```

And classes that cannot be subclassed present limitations as well.

---

# 20. How is `@Transactional` related to AOP?

This is a **very important TCS/interview question**.

When you write:

```java
@Transactional
public void transfer() {
}
```

Spring can create a proxy around the bean.

Conceptually:

```text
Caller
 ↓
Spring Proxy
 ↓
Transaction interceptor
 ↓
Begin transaction
 ↓
transfer()
 ↓
Commit / rollback
```

So `@Transactional` is commonly implemented using **Spring AOP/proxy-based interception**.

---

# 21. Can Spring AOP intercept private methods?

Generally **no**, not through Spring's normal proxy-based AOP.

Why?

The proxy intercepts calls made through the proxy, and private methods cannot be overridden/intercepted in the same way.

Similarly, self-invocation is a problem.

### Remember:

```text
public method through proxy → AOP works

private method → no normal Spring proxy interception

self-invocation → bypasses proxy
```

---

# 22. Where is AOP useful in real projects?

Very common:

### Logging

```text
method start
method end
execution time
```

### Auditing

```text
Who changed what?
When?
```

### Transactions

```java
@Transactional
```

### Security

```java
@PreAuthorize(...)
```

### Performance monitoring

```text
API/service execution time
```

### Caching

```java
@Cacheable
```

---

# 🔥 AOP interview rapid-fire

| Question                     | Answer                                     |
| ---------------------------- | ------------------------------------------ |
| AOP?                         | Separates cross-cutting concerns           |
| Aspect?                      | Module containing cross-cutting logic      |
| Join Point?                  | Execution point where advice can apply     |
| Pointcut?                    | Selects join points                        |
| Advice?                      | Code executed at selected join points      |
| `@Before`?                   | Before method                              |
| `@After`?                    | After method, success or exception         |
| `@AfterReturning`?           | Only successful return                     |
| `@AfterThrowing`?            | When exception is thrown                   |
| `@Around`?                   | Controls before/after and target execution |
| `proceed()`?                 | Continue to target method                  |
| Spring AOP mechanism?        | Proxy-based                                |
| Self-invocation?             | Bypasses proxy                             |
| JDK proxy?                   | Interface-based                            |
| CGLIB?                       | Class/subclass-based                       |
| `@Transactional`?            | Commonly implemented through Spring AOP    |
| Private method interception? | Not through normal Spring proxy AOP        |

---

# ⚡ One answer worth memorizing

If they ask:

> **"Explain how `@Transactional` works internally."**

Say:

> "`@Transactional` is processed by Spring and typically implemented using a proxy. When a caller invokes the transactional method through the proxy, a transaction interceptor starts the transaction, invokes the target method, and then commits or rolls back based on the outcome. Because this is proxy-based, self-invocation can bypass the transactional interceptor."

That's a **very strong interview answer**.

---

## ✅ Spring AOP DONE

Next up:

# 🚀 Spring Batch

This is especially important for you because of your project experience. We'll cover **Job → Step → ItemReader → ItemProcessor → ItemWriter → chunk → transaction → skip/retry → partitioning → parallel processing → restartability**, but at interview speed.
