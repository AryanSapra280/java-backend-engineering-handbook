# Spring Core — AOP Part 2: Spring Proxies ⭐⭐⭐⭐⭐

Now we get to the **most important AOP concept for your interview**:

> **How does Spring actually intercept a method call?**

The answer is:

**Spring creates a proxy around the target bean.**

---

# 1. What is a Spring Proxy?

A proxy is an object that **stands in front of the actual Spring bean** and can intercept method calls before delegating them to the real object.

Suppose we have:

```java
@Service
public class PaymentService {

    public void processPayment() {
        System.out.println("Processing payment");
    }
}
```

You might think:

```text
Caller
  ↓
PaymentService
```

But with AOP:

```text
Caller
  ↓
Spring Proxy
  ↓
PaymentService
```

The caller usually interacts with the **proxy**, not directly with the target object.

The proxy can perform additional work:

```text
Caller
  ↓
Proxy
  ↓
Logging
  ↓
Security
  ↓
Transaction
  ↓
Target method
```

---

# 2. Why does Spring create proxies?

Because Spring needs a place to intercept method calls.

For example:

```java
@Transactional
public void transferMoney() {
    // business logic
}
```

Spring needs to do something conceptually like:

```text
method called
      ↓
start transaction
      ↓
execute method
      ↓
commit
```

Your method itself doesn't contain:

```java
beginTransaction();
commitTransaction();
```

So Spring uses a proxy/interceptor mechanism around the bean.

### Interview answer

> **Spring creates proxies to intercept method calls and apply cross-cutting behavior such as transactions, security, caching, logging, and other AOP advice without modifying the target business code.**

---

# 3. What actually gets injected?

This is an important interview question.

Suppose:

```java
@Service
public class PaymentService {
}
```

and:

```java
@Service
public class OrderService {

    private final PaymentService paymentService;

    public OrderService(PaymentService paymentService) {
        this.paymentService = paymentService;
    }
}
```

If `PaymentService` has no proxy-related behavior, the injected object is effectively the actual bean.

But if Spring needs a proxy:

```java
@Transactional
@Service
public class PaymentService {
}
```

then conceptually:

```text
ApplicationContext
       │
       ├── Target PaymentService
       │
       └── Proxy PaymentService
                ↑
                │
        injected into OrderService
```

So:

```java
paymentService.processPayment();
```

can actually mean:

```text
OrderService
     ↓
PaymentService proxy
     ↓
Transaction interceptor
     ↓
Target PaymentService
     ↓
processPayment()
```

---

# 4. Two major proxy mechanisms

Spring commonly uses:

```text
JDK Dynamic Proxy
        OR
CGLIB-based proxy
```

The choice matters.

---

# 5. JDK Dynamic Proxy

JDK dynamic proxies work through **interfaces**.

Suppose:

```java
public interface PaymentService {

    void processPayment();
}
```

Implementation:

```java
@Service
public class PaymentServiceImpl
        implements PaymentService {

    public void processPayment() {
        System.out.println("Payment");
    }
}
```

Spring can create a JDK proxy implementing:

```text
PaymentService
```

Conceptually:

```text
PaymentService
      ↑
JDK Proxy
      ↓
PaymentServiceImpl
```

The caller interacts through the interface.

---

# 6. How does JDK proxy work conceptually?

Java provides:

```java
java.lang.reflect.Proxy
```

and:

```java
InvocationHandler
```

Conceptually:

```java
public Object invoke(
        Object proxy,
        Method method,
        Object[] args) {

    // before logic

    Object result =
        method.invoke(target, args);

    // after logic

    return result;
}
```

So:

```text
Caller
  ↓
JDK Proxy
  ↓
InvocationHandler
  ↓
Target method
```

Spring builds its own interceptor chain around this mechanism.

---

# 7. CGLIB Proxy

Now suppose you have:

```java
@Service
public class PaymentService {

    public void processPayment() {
    }
}
```

There is no interface.

Spring can use a **class-based proxy**, commonly backed by CGLIB-generated subclassing.

Conceptually:

```text
PaymentServiceProxy
       extends
PaymentService
```

So:

```text
Caller
  ↓
PaymentServiceProxy
  ↓
PaymentService
```

The proxy overrides/intercepts eligible methods and delegates through Spring's interceptor chain.

---

# 8. JDK Proxy vs CGLIB

🔥 **Very common interview question.**

| JDK Dynamic Proxy | CGLIB |
|---|---|
| Interface-based | Class/subclass-based |
| Uses Java proxy mechanism | Generates subclass |
| Requires an interface for the proxied contract | Can proxy concrete classes |
| Proxy implements interfaces | Proxy extends target class |
| Method calls through interface | Method interception through subclass |

The simplest interview answer:

> **JDK dynamic proxies proxy interfaces, while CGLIB creates a subclass-based proxy for concrete classes.**

---

# 9. Which proxy does Spring use?

This is where you need to be careful in a modern Spring interview.

Don't give an oversimplified answer like:

> "Spring always uses JDK proxies if an interface exists."

The exact behavior depends on Spring configuration and the mechanism being used.

For example, Spring's AOP proxying can be configured with:

```properties
spring.aop.proxy-target-class=true
```

In Spring Boot, this setting controls whether class-based proxies are preferred for auto-configured AOP infrastructure.

If class-based proxying is enabled:

```text
proxy-target-class=true
        ↓
CGLIB/class-based proxy
```

Otherwise, where applicable, interface-based JDK proxies can be used.

---

# 🔥 Interview Question

### "Why might `proxy-target-class=true` be used?"

Because you may want Spring to create a **class-based proxy** instead of relying on an interface-based JDK proxy.

For example:

```java
@Service
public class PaymentService {
    
    public void processPayment() {
    }
}
```

There is no interface.

A class-based proxy can still intercept the method.

---

# 10. Important CGLIB limitation

Because CGLIB-style proxying works through subclassing, methods that cannot be overridden cannot be intercepted in the normal way.

For example:

```java
public final void processPayment() {
}
```

A subclass cannot override a `final` method.

Therefore, don't assume:

> "Every method can always be intercepted."

This is an important practical proxy limitation.

---

# 11. The most important concept: Proxy vs Target

Suppose:

```java
@Service
public class PaymentService {

    @Transactional
    public void processPayment() {
    }
}
```

Think of Spring creating:

```text
                 Spring
                   │
        ┌──────────┴──────────┐
        ↓                     ↓
     Proxy                 Target
        │                     │
        │              PaymentService
        │
 Transaction interceptor
        │
        └──────────→ target
```

When another bean does:

```java
paymentService.processPayment();
```

the call goes:

```text
Caller
  ↓
Proxy
  ↓
Transaction interceptor
  ↓
Target
  ↓
processPayment()
```

That is the foundation for understanding `@Transactional`.

---

# 12. Why does self-invocation break Spring AOP?

🔥🔥 **EXTREMELY IMPORTANT**

Suppose:

```java
@Service
public class PaymentService {

    public void processPayment() {
        validatePayment();
    }

    @Transactional
    public void validatePayment() {
        // database operation
    }
}
```

Now:

```java
processPayment()
```

calls:

```java
validatePayment();
```

You might expect:

```text
processPayment()
      ↓
Spring proxy
      ↓
@Transactional
      ↓
validatePayment()
```

But that's **not what happens**.

---

# 13. What actually happens?

The call is:

```java
this.validatePayment();
```

implicitly.

So:

```text
Proxy
  ↓
processPayment()
  ↓
this.validatePayment()
  ↓
Target method
```

The second call happens **inside the target object itself**.

It does not go back through the proxy.

Therefore:

```text
No proxy
   ↓
No interceptor
   ↓
@Transactional advice isn't applied through that proxy call
```

This is called:

> **Self-invocation**

---

# 14. Visualize it

Normal external call:

```text
OrderService
     │
     ↓
PaymentService Proxy
     │
     ↓
Transaction Interceptor
     │
     ↓
PaymentService Target
```

Self-invocation:

```text
PaymentService Target
       │
       ↓
this.validatePayment()
       │
       ↓
PaymentService Target
```

The proxy is completely bypassed.

---

# 🔥 Interview Question

### "Why does self-invocation bypass Spring AOP?"

Interview-ready answer:

> Spring AOP is proxy-based. External callers invoke the Spring-managed proxy, which applies the relevant interceptors before delegating to the target. When a method inside the target calls another method using `this`, the call happens directly on the target object and doesn't pass through the proxy, so the proxy-based advice is not triggered.

That's the answer you want to remember.

---

# 15. How can we solve self-invocation?

There are several approaches.

### Best approach: separate responsibilities

Instead of:

```java
class PaymentService {

    public void process() {
        validate();
    }

    @Transactional
    public void validate() {
    }
}
```

Move the transactional operation to another bean:

```java
@Service
public class PaymentValidationService {

    @Transactional
    public void validate() {
    }
}
```

Then:

```java
@Service
public class PaymentService {

    private final PaymentValidationService validationService;

    public PaymentService(
            PaymentValidationService validationService) {

        this.validationService = validationService;
    }

    public void process() {

        validationService.validate();
    }
}
```

Now:

```text
PaymentService
      ↓
Spring Proxy
      ↓
PaymentValidationService
      ↓
Transaction interceptor
      ↓
validate()
```

This is generally the cleanest solution.

---

# 16. Can we inject ourselves?

Technically, you can structure code so that calls go through the proxied bean reference, but **self-injection is generally not the preferred design**.

You might see:

```java
@Autowired
private PaymentService self;
```

and:

```java
self.validatePayment();
```

Now the call may go through the proxy.

But this introduces unnecessary complexity and makes the design harder to reason about.

**Prefer extracting the responsibility into another bean.**

---

# 17. Another workaround: `AopContext`

Spring provides:

```java
AopContext.currentProxy()
```

when configured appropriately.

Conceptually:

```java
((PaymentService) AopContext.currentProxy())
        .validatePayment();
```

This forces the call through the current proxy.

But again:

> **Don't make this your first solution.**

It couples your business code to Spring AOP infrastructure.

For production design, restructuring the beans is usually cleaner.

---

# 18. Why is this important beyond `@Transactional`?

Because the same proxy principle affects many Spring features.

For example:

```text
@Transactional
@Cacheable
@Async
@PreAuthorize
custom @Aspect
```

If the behavior is implemented through Spring proxy-based interception, a direct internal call can bypass the proxy.

That's why you should develop this mental model:

```text
Spring feature
      ↓
Interceptor
      ↓
Proxy
      ↓
Target
```

If you bypass the proxy:

```text
Target
  ↓
this.method()
  ↓
Target
```

the interceptor doesn't get the opportunity to run.

---

# 19. Practical Production Example

Suppose you have:

```java
@Service
public class PaymentService {

    @Transactional
    public void createPayment() {

        paymentRepository.save(...);

        audit();
    }

    @Transactional(propagation = Propagation.REQUIRES_NEW)
    public void audit() {

        auditRepository.save(...);
    }
}
```

You might expect:

```text
createPayment()
     ↓
Transaction T1
     ↓
audit()
     ↓
Transaction T2
```

But because:

```java
audit();
```

is self-invocation:

```text
createPayment()
     ↓
T1
     ↓
this.audit()
     ↓
NO proxy
     ↓
REQUIRES_NEW isn't applied through the proxy
```

So the expected new transaction **doesn't get created through Spring's proxy mechanism**.

This is a classic senior interview trap.

---

# 20. Proxy-based Spring AOP mental model

This is the model you should carry into every interview:

```text
                   Spring Container
                         │
                         ↓
                   Target Bean
                         │
                  create/wrap
                         ↓
                    Spring Proxy
                         │
              ┌──────────┼──────────┐
              ↓          ↓          ↓
          Logging     Security   Transaction
              │          │          │
              └──────────┼──────────┘
                         ↓
                    Target Method
```

External call:

```text
Caller
  ↓
Proxy
  ↓
Interceptors
  ↓
Target
```

Internal `this.method()` call:

```text
Target
  ↓
this.method()
  ↓
Target
```

**No proxy → no proxy-based advice.**

---

# 🔥 Senior Interview Rapid Fire

### Q: What is a Spring proxy?

> An object created by Spring that wraps a target bean and intercepts method calls to apply cross-cutting behavior.

### Q: Why does Spring use proxies?

> To apply behavior such as transactions, security, caching, and AOP advice without putting that infrastructure logic directly into business methods.

### Q: JDK proxy vs CGLIB?

> JDK proxies are interface-based; CGLIB/class-based proxies use subclassing to proxy concrete classes.

### Q: Why can a final method be problematic with CGLIB?

> A subclass cannot override a final method, so normal subclass-based interception cannot be applied to it.

### Q: What is self-invocation?

> A method in a target object calls another method on the same object, typically through `this`.

### Q: Why does self-invocation bypass AOP?

> Because the call doesn't pass through the Spring proxy.

### Q: Best solution?

> Usually move the separately advised operation into another Spring bean so the call crosses the proxy boundary.

---

# The critical bridge to Transactions

We now have:

```text
AOP
 ↓
Aspect
 ↓
Pointcut
 ↓
Advice
 ↓
Proxy
 ↓
JDK / CGLIB
 ↓
Self-invocation
```

Now we're perfectly positioned for the next section:

# **`@Transactional` — How Spring Transactions Actually Work ⭐⭐⭐⭐⭐**

We'll go from:

```text
@Transactional
      ↓
Proxy
      ↓
TransactionInterceptor
      ↓
TransactionManager
      ↓
Database transaction
      ↓
Commit / Rollback
```

Then we'll cover **transaction boundaries → propagation → isolation → rollback → read-only → exceptions → async → Kafka → common production mistakes**, exactly in the path you defined.