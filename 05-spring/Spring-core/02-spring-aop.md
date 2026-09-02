# Spring AOP & Proxies — Interview Revision Notes

> **Goal:** Understand AOP well enough for an SDE-2 Spring interview and build the foundation for `@Transactional`.
>
> **Priority:** P0 = important for Monday. P1 = understand the concept. Later = deeper framework internals.

---

# 1. What Problem Does AOP Solve?

Imagine many service methods need the same additional behavior:

- Logging
- Security checks
- Transaction management
- Performance measurement
- Auditing
- Metrics
- Tracing
- Caching

Without AOP, you may repeat the same code everywhere:

```java
public void transferMoney() {

    log.info("Starting");

    checkSecurity();

    beginTransaction();

    // actual business logic

    commitTransaction();

    log.info("Completed");
}
```

This creates duplication and mixes infrastructure concerns with business logic.

AOP helps separate these **cross-cutting concerns** from the core business logic.

---

# 2. What Is AOP?

AOP = **Aspect-Oriented Programming**.

It is a programming technique used to separate cross-cutting concerns from core business logic.

Conceptually:

```text
                    Cross-Cutting Concerns
                           |
          +----------------+----------------+
          |                |                |
       Logging         Security        Transactions
                           |
                           v
                     Business Logic
```

### Interview answer

> AOP is a programming technique used to separate cross-cutting concerns such as logging, security, transactions, and metrics from the application's core business logic.

---

# 3. What Is a Cross-Cutting Concern?

A cross-cutting concern is functionality that affects multiple parts of an application.

Examples:

```text
Logging
Authentication / Authorization
Transactions
Auditing
Metrics
Tracing
Caching
```

For a Provident Fund application, imagine:

```text
ContributionService
WithdrawalService
TransferService
SettlementService
```

All may need:

```text
Logging
Authorization
Transaction management
Metrics
Tracing
```

Instead of putting that code inside every service method, AOP can handle it separately.

---

# 4. Important AOP Terminology

For interviews, focus on these six terms:

| Term | Meaning |
|---|---|
| Aspect | Module containing cross-cutting logic |
| Advice | What action should execute |
| Join Point | Point during execution where advice can apply |
| Pointcut | Rule selecting join points |
| Target | Actual object being advised |
| Proxy | Object through which intercepted calls pass |

The **proxy** is particularly important because Spring AOP commonly uses proxy-based interception.

---

# 5. Aspect

An Aspect contains the cross-cutting behavior.

Example:

```java
@Aspect
@Component
public class LoggingAspect {
}
```

A complete example:

```java
@Aspect
@Component
public class LoggingAspect {

    @Around("execution(* com.example.service.*.*(..))")
    public Object logExecution(ProceedingJoinPoint joinPoint)
            throws Throwable {

        System.out.println("Before method");

        Object result = joinPoint.proceed();

        System.out.println("After method");

        return result;
    }
}
```

Don't focus on memorizing the pointcut syntax.

Understand the architecture:

```text
Aspect
  |
  +-- Pointcut → where should it apply?
  |
  +-- Advice   → what should it do?
```

---

# 6. Advice

**Advice = what action should happen.**

Common advice types:

```text
@Before
@After
@AfterReturning
@AfterThrowing
@Around
```

---

## 6.1 `@Before`

Runs before the target method.

```java
@Before("...")
public void before() {
}
```

Conceptually:

```text
Advice
  ↓
Target Method
```

---

## 6.2 `@After`

Runs after the method execution, generally regardless of whether the method completes normally or throws.

---

## 6.3 `@AfterReturning`

Runs after successful method completion.

```text
Target Method
     ↓
successful return
     ↓
@AfterReturning
```

---

## 6.4 `@AfterThrowing`

Runs when the target method throws an exception.

```text
Target Method
     ↓
Exception
     ↓
@AfterThrowing
```

---

## 6.5 `@Around` ⭐⭐⭐

`@Around` is the most powerful/common advice for understanding proxy interception.

It can execute logic:

```text
Before
  ↓
Actual Method
  ↓
After
```

Example:

```java
@Around("execution(* com.example.service.*.*(..))")
public Object around(ProceedingJoinPoint joinPoint)
        throws Throwable {

    System.out.println("Before");

    Object result = joinPoint.proceed();

    System.out.println("After");

    return result;
}
```

---

# 7. `joinPoint.proceed()` ⭐⭐⭐

This is a common interview follow-up.

```java
Object result = joinPoint.proceed();
```

means:

> Continue execution to the intercepted target method.

Flow:

```text
Application
    ↓
Proxy
    ↓
Around Advice
    ↓
joinPoint.proceed()
    ↓
Actual Method
    ↓
return
    ↓
Around Advice
    ↓
Application
```

### What if `proceed()` is not called?

If an `@Around` advice does not call `proceed()`, the target method will not execute through that advice.

Example:

```java
@Around("...")
public Object around(ProceedingJoinPoint jp) {

    System.out.println("Blocked");

    return null;
}
```

The target method is not continued.

This is one reason `@Around` is powerful: it can control whether the target method proceeds.

---

# 8. Pointcut ⭐⭐⭐

A pointcut defines **which method executions should be intercepted**.

Example:

```java
@Around("execution(* com.example.service.*.*(..))")
```

Conceptually:

```text
Which methods?
      ↓
Pointcut
      ↓
Apply Advice
```

### Interview answer

> A pointcut is a rule that selects the join points where an aspect should apply.

---

# 9. Join Point

A join point is a point during program execution where advice can potentially be applied.

For Spring AOP, the practical interview-level point is:

> Spring AOP primarily works with **method execution join points**.

You do not need to study the entire AspectJ join-point model for Monday.

---

# 10. Target Object

The **target** is the actual business object containing the method being intercepted.

Example:

```java
@Service
public class PaymentService {

    public void transfer() {
        System.out.println("Transfer");
    }
}
```

Here:

```text
PaymentService instance
        ↓
Target object
```

---

# 11. Proxy ⭐⭐⭐

This is the most important practical Spring AOP concept.

Suppose:

```java
@Service
public class PaymentService {

    public void transfer() {
        System.out.println("Transfer");
    }
}
```

Conceptually, your application does not necessarily call the target object directly.

Instead:

```text
Application
    ↓
Spring Proxy
    ↓
Actual PaymentService
```

When:

```java
paymentService.transfer();
```

is called:

```text
Application
    ↓
Proxy
    ↓
Advice
    ↓
Actual PaymentService.transfer()
```

The proxy gives Spring a place to intercept the method call.

---

# 12. Why Does Spring Use Proxies?

Spring can intercept method calls **without putting framework/infrastructure code directly inside business methods**.

For example:

```java
@Transactional
public void transfer() {
    // business logic
}
```

You don't manually write:

```java
beginTransaction();

try {
    transferLogic();
    commit();
} catch (...) {
    rollback();
}
```

Spring's transaction infrastructure can intercept the call.

Conceptually:

```text
Caller
   ↓
Spring Proxy
   |
   +--> Start transaction
   |
   +--> Call actual method
   |
   +--> Commit / Rollback
   ↓
Caller
```

This is the bridge:

```text
AOP
  ↓
Proxy
  ↓
@Transactional
  ↓
Transaction management
```

---

# 13. JDK Dynamic Proxy vs CGLIB ⭐⭐

This is a common interview question.

## JDK Dynamic Proxy

JDK dynamic proxies are interface-based.

Example:

```java
public interface PaymentService {
    void transfer();
}
```

Conceptually:

```text
PaymentService Interface
          ↑
          |
      JDK Proxy
          |
          v
PaymentService Implementation
```

The proxy implements the interface.

---

## CGLIB

CGLIB-based proxying uses subclassing.

Conceptually:

```text
PaymentService
      ↑
      |
CGLIB-generated subclass
```

The generated subclass can intercept method calls.

### Interview answer

> JDK dynamic proxies are interface-based, while CGLIB uses subclass-based proxying. Spring can use proxying to intercept method calls and apply concerns such as transactions and other AOP advice.

### Don't go too deep yet

You do not need to study:

- Bytecode generation internals
- CGLIB implementation internals
- ProxyFactory internals
- Every Spring AOP class

---

# 14. Why Proxy Knowledge Matters for `@Transactional` ⭐⭐⭐

An interviewer may ask:

> "How does `@Transactional` actually work?"

Weak answer:

> Spring automatically manages the transaction.

Better answer:

> `@Transactional` is implemented through Spring's transaction infrastructure, typically using proxy-based interception. When the caller invokes the proxied bean, the transaction interceptor can start a transaction, invoke the target method, and then commit or roll back based on the outcome.

Mental model:

```text
Caller
   ↓
Spring Proxy
   ↓
Transaction Interceptor
   ↓
Start Transaction
   ↓
Target Method
   ↓
Success? ───────→ Commit
   |
   └── Exception → Rollback (depending on rules)
```

We will study the exact rollback rules in the next section.

---

# 15. BIG TRAP — Self Invocation ⭐⭐⭐

This is one of the most important Spring interview questions.

Consider:

```java
@Service
public class PaymentService {

    public void methodA() {
        methodB();
    }

    @Transactional
    public void methodB() {
        // transactional logic
    }
}
```

You might think:

```text
methodA()
   ↓
methodB()
   ↓
Transaction starts
```

But with proxy-based AOP, that is not necessarily what happens.

Why?

Because:

```java
methodB();
```

is an internal call on the same object.

It does not go through the Spring proxy.

---

# 16. Understand Self Invocation Visually

External call:

```text
External Caller
      ↓
    Proxy
      ↓
  methodB()
      ↓
Target Object
```

Advice can apply.

Internal call:

```text
External Caller
      ↓
    Proxy
      ↓
  methodA()
      ↓
this.methodB()
      ↓
Target Object
```

The call from `methodA()` to `methodB()` is direct.

The proxy is bypassed.

Therefore proxy-based advice such as transaction interception may not apply to that internal call.

---

# 17. The Key Rule

Memorize this:

> **Spring proxy-based AOP works when the method call goes through the proxy.**

External call:

```text
Caller
  ↓
Proxy
  ↓
Method
```

AOP can apply.

Internal self-invocation:

```text
Method A
  ↓
this.methodB()
```

Proxy is bypassed.

---

# 18. How to Fix Self Invocation

The cleanest design is usually to move the advised operation into another Spring bean.

Instead of:

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

use:

```java
@Service
public class PaymentService {

    private final TransactionService transactionService;

    public PaymentService(TransactionService transactionService) {
        this.transactionService = transactionService;
    }

    public void methodA() {
        transactionService.methodB();
    }
}
```

```java
@Service
public class TransactionService {

    @Transactional
    public void methodB() {
    }
}
```

Now:

```text
PaymentService
      ↓
Spring Proxy
      ↓
TransactionService
      ↓
@Transactional method
```

The call crosses the proxy boundary.

### Interview answer

> A common solution is to move the advised method into another Spring-managed bean so the call goes through that bean's proxy.

---

# 19. PF Application Example

Imagine:

```text
ContributionController
        ↓
ContributionService
        ↓
TransactionService
        ↓
Repository
```

Suppose:

```java
@Transactional
public void processContribution() {

    // update contribution
    // update transaction
    // update balance
}
```

The controller calls the Spring-managed service:

```text
HTTP Request
     ↓
Controller
     ↓
Spring Proxy
     ↓
@Transactional Service
     ↓
Database
```

The proxy/interceptor can handle transaction behavior around the service call.

---

# 20. AOP Interview Question Bank

### Q1. What is AOP?

> AOP separates cross-cutting concerns such as logging, security, transactions, and metrics from core business logic.

### Q2. What is an Aspect?

> An aspect encapsulates a cross-cutting concern and defines behavior that should be applied to selected join points.

### Q3. What is Advice?

> Advice defines what action should execute, such as before, after, after returning, after throwing, or around a method execution.

### Q4. What is a Pointcut?

> A pointcut defines the rule used to select where an aspect should apply.

### Q5. What is a Join Point?

> A join point is a point during execution where advice can potentially be applied. In Spring AOP, method execution is the key practical case.

### Q6. Why does Spring use proxies?

> Proxies allow Spring to intercept method calls and apply cross-cutting behavior such as transaction management without putting infrastructure code directly inside business methods.

### Q7. What does `joinPoint.proceed()` do?

> It continues execution to the intercepted target method.

### Q8. What happens if `proceed()` is not called?

> The target method does not execute through that around advice.

### Q9. JDK Proxy vs CGLIB?

> JDK dynamic proxies are interface-based, while CGLIB uses subclass-based proxying.

### Q10. What is self invocation?

> Self invocation occurs when one method in a bean directly calls another method on the same bean. The internal call bypasses the Spring proxy, so proxy-based advice such as transaction interception may not apply.

### Q11. How would you fix self invocation?

> Move the advised operation to another Spring-managed bean so the call goes through the proxy.

---

# 21. High-Probability Follow-Up Chain

Expect this sequence:

```text
What is AOP?
    ↓
What is a cross-cutting concern?
    ↓
What is an Aspect?
    ↓
What is Advice?
    ↓
What is Pointcut?
    ↓
How does Spring implement AOP?
    ↓
Proxy
    ↓
JDK Proxy vs CGLIB
    ↓
How does @Transactional work?
    ↓
What about self invocation?
```

If you understand this chain, you are in a strong position for Spring AOP questions.

---

# 22. Don't Study These Deeply Before Monday

Useful later, but low return right now:

- AspectJ weaving
- Load-time weaving
- AspectJ vs Spring AOP internals
- ProxyFactory internals
- Advisor chain internals
- CGLIB bytecode details
- Custom pointcut parser internals
- Spring AOP source code

Focus on:

```text
AOP
↓
Cross-cutting concern
↓
Aspect
↓
Advice
↓
Pointcut
↓
Proxy
↓
Self invocation
↓
@Transactional
```

---

# 23. 60-Second AOP Mental Model

Remember this:

```text
Cross-Cutting Concern
        ↓
       AOP
        ↓
     Aspect
     /    \
Pointcut  Advice
   ↓        ↓
Where?    What?
     \     /
      Proxy
        ↓
   Target Method
```

For `@Around`:

```text
Proxy
  ↓
Before logic
  ↓
joinPoint.proceed()
  ↓
Actual method
  ↓
After logic
```

For transactions:

```text
Caller
  ↓
Proxy
  ↓
Transaction Interceptor
  ↓
Begin transaction
  ↓
Target method
  ↓
Commit / Rollback
```

And the critical trap:

```text
External call
→ goes through proxy
→ advice can apply

Self invocation
→ direct internal call
→ proxy bypassed
→ proxy-based advice may not apply
```

---

# 24. Next Topic — `@Transactional` ⭐⭐⭐

This is the next **high-priority** section.

We will cover:

```text
@Transactional
    ↓
How transaction actually starts
    ↓
Commit
    ↓
Rollback
    ↓
RuntimeException vs checked exception
    ↓
rollbackFor
    ↓
Propagation
    ↓
REQUIRED
    ↓
REQUIRES_NEW
    ↓
NESTED
    ↓
Isolation
    ↓
Concurrency problems
    ↓
Self-invocation
    ↓
Real PF transaction scenarios
```

**For Monday, focus heavily on `@Transactional`, propagation, rollback, and isolation.**
