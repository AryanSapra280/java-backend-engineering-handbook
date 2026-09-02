# Spring Core — Interview Revision Notes

> Interview-focused notes for rapid revision.  
> **P0:** must know deeply enough for follow-ups.  
> **P1:** know concept + common follow-ups.  
> **Later:** useful, but not worth spending Monday-prep time on.

---

## 1. IoC — Inversion of Control

### Core idea

Without Spring, application code often creates its own dependencies:

```java
class PaymentService {
    private PaymentRepository repository =
            new PaymentRepository();
}
```

`PaymentService` controls dependency creation.

With Spring:

```java
@Service
class PaymentService {
    private final PaymentRepository repository;

    PaymentService(PaymentRepository repository) {
        this.repository = repository;
    }
}
```

Spring creates/manages the objects and supplies the dependency.

### Mental model

```text
Without Spring:
Application code → creates objects

With Spring:
Spring IoC Container
    → creates objects
    → manages objects
    → resolves dependencies
    → injects dependencies
```

### Interview answer

> IoC means that responsibility for creating and managing application objects is transferred from application code to the Spring container.

---

## 2. Dependency Injection

A class has a dependency when it needs another object.

```text
PaymentService
      ↓
PaymentRepository
```

Instead of:

```java
new PaymentRepository()
```

inside `PaymentService`, the dependency is supplied from outside.

```java
class PaymentService {
    private final PaymentRepository repository;

    PaymentService(PaymentRepository repository) {
        this.repository = repository;
    }
}
```

### Interview answer

> Dependency Injection is a mechanism where an object's dependencies are provided to it from outside rather than the object creating those dependencies itself.

---

## 3. IoC vs DI

```text
IoC
 └─ broader principle
    └─ control over object creation/management is inverted

DI
 └─ mechanism
    └─ dependencies are supplied from outside
```

### Interview answer

> IoC is the broader principle of transferring control of object creation and management to a container or framework. Dependency Injection is one of the primary mechanisms used to achieve IoC.

---

## 4. Why Dependency Injection?

### Without DI

```java
class ContributionService {
    private TransactionService transactionService =
            new TransactionService();
}
```

Problems:

- Tight coupling
- Harder testing
- Implementation is fixed inside the class
- Harder replacement/configuration

### With DI

```java
class ContributionService {
    private final TransactionService transactionService;

    ContributionService(TransactionService transactionService) {
        this.transactionService = transactionService;
    }
}
```

Production:

```text
ContributionService → RealTransactionService
```

Test:

```text
ContributionService → MockTransactionService
```

### Benefits

- Loose coupling
- Better testability
- Explicit dependencies
- Easier replacement/configuration
- Cleaner object construction

---

# 5. Types of Dependency Injection

Commonly discussed types:

1. Constructor Injection — **preferred**
2. Setter Injection
3. Field Injection

---

## 5.1 Constructor Injection ⭐ P0

```java
@Service
public class PaymentService {

    private final PaymentRepository repository;

    public PaymentService(PaymentRepository repository) {
        this.repository = repository;
    }
}
```

### Why preferred?

1. Dependencies are explicit.
2. Mandatory dependencies are enforced at construction.
3. Dependencies can be `final`.
4. Unit testing is easy without starting Spring.
5. Avoids partially initialized objects.

### Interview answer

> I prefer constructor injection because dependencies are explicit, mandatory dependencies are enforced at construction time, fields can remain final, and the class is easier to unit test without depending on the Spring container.

---

## 5.2 Field Injection

```java
@Service
public class PaymentService {

    @Autowired
    private PaymentRepository repository;
}
```

It works, but is generally not preferred.

### Problems

- Dependencies are less visible.
- Injected fields cannot normally be `final`.
- Standalone unit testing is less clean.
- Construction and dependency injection are separated.

### Interview answer

> Field injection works, but constructor injection is generally preferred because it makes dependencies explicit, allows final fields, enforces mandatory dependencies during construction, and improves testability.

---

## 5.3 Setter Injection

```java
@Service
public class PaymentService {

    private PaymentRepository repository;

    @Autowired
    public void setRepository(PaymentRepository repository) {
        this.repository = repository;
    }
}
```

Useful when a dependency is optional or intentionally configurable after construction.

For normal mandatory dependencies:

```text
Constructor Injection > Setter Injection
```

---

# 6. Spring Bean ⭐ P0

A Spring Bean is:

> An object that is instantiated, configured, and managed by the Spring IoC container.

Example:

```java
@Service
public class ContributionService {
}
```

Spring can discover the class, create an instance, and manage that instance.

### Important distinction

```java
ContributionService service =
        new ContributionService();
```

creates a normal Java object.

It is not automatically a Spring-managed bean.

### Interview answer

> A Spring Bean is an object whose lifecycle and dependencies are managed by the Spring IoC container.

---

# 7. Component Scanning ⭐ P0

Spring can discover classes using component stereotypes:

```java
@Component
@Service
@Repository
@Controller
@RestController
```

Conceptually:

```text
Application startup
       ↓
Component scanning
       ↓
Find component classes
       ↓
Register bean definitions
       ↓
Create/manage beans
```

### Interview answer

> Component scanning is the mechanism through which Spring searches configured packages for component classes and registers them as beans.

---

# 8. Component Stereotypes

## `@Component`

Generic Spring-managed component:

```java
@Component
public class PaymentValidator {
}
```

## `@Service`

Represents business/service logic:

```java
@Service
public class ContributionService {
}
```

## `@Repository`

Represents persistence/data-access components:

```java
@Repository
public class ContributionRepository {
}
```

`@Repository` also participates in Spring's persistence exception translation.

## `@Controller`

Used for Spring MVC controllers.

## `@RestController`

Used for REST controllers.

Conceptually:

```text
@Component
   ├── @Service
   ├── @Repository
   └── @Controller
```

### Interview trap

Do not say `@Service` and `@Component` are completely different mechanisms.

They both participate in Spring's component model; `@Service` communicates the intended role.

---

# 9. ApplicationContext ⭐ P0

`ApplicationContext` is Spring's IoC container/context.

Think:

```text
ApplicationContext
       ├── ContributionService
       ├── TransactionService
       ├── AccountService
       ├── Repository
       └── Other beans
```

It manages beans and provides services such as:

- Dependency Injection
- Bean lifecycle management
- Configuration
- Application events
- Resource/message support

### Interview answer

> `ApplicationContext` is Spring's IoC container that manages application beans and provides services such as dependency injection, bean lifecycle management, configuration, and application events.

---

# 10. How Spring Wires an Application

High-level mental model:

```text
Spring Boot starts
       ↓
ApplicationContext created
       ↓
Configuration read
       ↓
Component scanning
       ↓
Bean definitions registered
       ↓
Beans created
       ↓
Dependencies resolved
       ↓
Dependencies injected
       ↓
Beans initialized
       ↓
Application ready
```

This is an interview-level mental model, not an exact source-code sequence for every configuration path.

---

# 11. `@Configuration` + `@Bean` ⭐ P0

Instead of relying on component scanning:

```java
@Service
public class PaymentService {
}
```

you can explicitly register an object:

```java
@Configuration
public class AppConfig {

    @Bean
    public PaymentService paymentService() {
        return new PaymentService();
    }
}
```

The object returned by the `@Bean` method is managed by Spring.

---

# 12. `@Component` vs `@Bean` ⭐ P0

### `@Component`

Placed on a class:

```java
@Component
public class PaymentValidator {
}
```

Spring discovers it through component scanning.

### `@Bean`

Placed on a configuration method:

```java
@Bean
public PaymentValidator paymentValidator() {
    return new PaymentValidator();
}
```

The method explicitly tells Spring what object to manage.

### Easy memory

```text
@Component
→ "Spring, discover this class."

@Bean
→ "Spring, manage this object I am explicitly creating."
```

### When to use `@Bean`

Especially useful when:

- You need explicit configuration.
- You need to customize object creation.
- The class comes from a third-party library.
- You cannot modify the class to add `@Component`.

Example:

```java
@Configuration
public class AppConfig {

    @Bean
    public ObjectMapper objectMapper() {
        return new ObjectMapper();
    }
}
```

---

# 13. Dependency Graph

Spring manages a graph of dependencies.

Example:

```text
ContributionController
        ↓
ContributionService
        ↓
TransactionService
        ↓
TransactionRepository
```

Instead of manually doing:

```java
new TransactionRepository();
new TransactionService(repository);
new ContributionService(transactionService);
new ContributionController(service);
```

Spring resolves and injects the required objects.

This is the practical combination of:

```text
IoC + DI + ApplicationContext + Beans
```

---

# 14. Circular Dependencies ⭐ P0

Example:

```text
A → B
↑   ↓
└───┘
```

```java
@Service
class A {
    A(B b) {}
}
```

```java
@Service
class B {
    B(A a) {}
}
```

Each requires the other to be constructed.

Constructor injection makes this problem visible and can cause bean creation to fail.

### Interview answer

> Circular dependencies generally indicate a design problem. Constructor injection makes such cycles visible because neither object can be constructed without the other.

---

# 15. Bean Lifecycle ⭐ P0

High-level lifecycle:

```text
Bean definition
      ↓
Instantiation
      ↓
Dependency Injection
      ↓
Initialization
      ↓
Bean ready
      ↓
Application uses bean
      ↓
Application shutdown
      ↓
Destruction
```

More detailed interview-level view:

```text
Instantiation
      ↓
Dependency Injection
      ↓
Aware callbacks (when applicable)
      ↓
BeanPostProcessor - before initialization
      ↓
@PostConstruct
      ↓
InitializingBean / init-method
      ↓
BeanPostProcessor - after initialization
      ↓
READY
      ↓
@PreDestroy
      ↓
DisposableBean / destroy-method
```

You do not need to memorize every callback for Monday.

---

# 16. `@PostConstruct` ⭐ P0

Example:

```java
@Service
public class AccountService {

    private final AccountRepository repository;

    public AccountService(AccountRepository repository) {
        this.repository = repository;
    }

    @PostConstruct
    public void initialize() {
        // repository is available here
    }
}
```

It runs after construction and dependency injection, during bean initialization.

### Mental model

```text
Constructor
    ↓
Dependency injection
    ↓
@PostConstruct
    ↓
Bean ready
```

### Interview answer

> `@PostConstruct` is used for initialization logic that should run after the bean has been constructed and its dependencies have been injected.

---

# 17. `@PreDestroy` ⭐ P0

Example:

```java
@Service
public class AccountService {

    @PreDestroy
    public void cleanup() {
        // cleanup
    }
}
```

It runs during normal ApplicationContext shutdown before the bean is destroyed.

### Important caveat

Do not say "`@PreDestroy` always executes."

If the JVM crashes or is abruptly terminated, Spring may not get the opportunity to perform normal shutdown.

```text
Graceful shutdown
    ↓
@PreDestroy can execute

Abrupt termination/crash
    ↓
May not execute
```

### Interview answer

> `@PreDestroy` is a lifecycle callback used for cleanup before a Spring-managed bean is destroyed during ApplicationContext shutdown.

---

# 18. Singleton Scope ⭐ P0

Spring beans are singleton-scoped by default.

Within a particular `ApplicationContext`, Spring normally maintains one instance for that bean definition.

```text
ApplicationContext
        ↓
AccountService
  ONE INSTANCE
        ↑
   ┌────┼────┐
   ↓    ↓    ↓
 Req1 Req2 Req3
```

### Critical point

**Spring singleton != Java singleton.**

Spring manages the singleton within the scope of an `ApplicationContext`.

---

# 19. Singleton Does NOT Mean Thread-Safe ⭐ P0

This is a major interview trap.

```java
@Service
public class CounterService {

    private int count = 0;

    public void increment() {
        count++;
    }
}
```

Because the service is normally singleton-scoped:

```text
Thread 1 ──┐
Thread 2 ──┼──> SAME CounterService
Thread 3 ──┘
```

The mutable field is shared.

`count++` is not an atomic operation.

A race condition is possible.

### Interview answer

> Spring's singleton scope means one bean instance per ApplicationContext; it does not make the bean thread-safe. If the singleton contains mutable shared state, concurrent requests can still cause race conditions.

### Better service design

Prefer stateless services:

```java
@Service
public class ContributionService {

    public void process(String accountId) {
        String currentAccountId = accountId;
        // local/request-specific state
    }
}
```

Avoid request-specific mutable state in singleton service fields.

---

# 20. Prototype Scope

Example:

```java
@Scope("prototype")
@Component
public class ReportGenerator {
}
```

Prototype scope means Spring creates a new instance when the bean is requested from the container.

```text
Request from container #1
       ↓
ReportGenerator #1

Request from container #2
       ↓
ReportGenerator #2
```

### Singleton vs Prototype

```text
Singleton
→ one instance per ApplicationContext

Prototype
→ new instance when requested from the container
```

Spring manages creation/configuration of prototype beans, but does not generally manage their complete destruction lifecycle in the same way as singleton beans.

---

# 21. Request Scope

In web applications:

```java
@Scope("request")
```

A bean instance is associated with an HTTP request.

```text
HTTP Request 1
      ↓
Bean #1

HTTP Request 2
      ↓
Bean #2
```

Know the concept; do not spend time on implementation internals right now.

---

# 22. Common Spring Scopes

| Scope | Meaning |
|---|---|
| singleton | One instance per ApplicationContext |
| prototype | New instance when requested from container |
| request | One instance per HTTP request |
| session | One instance per HTTP session |

Typical backend components such as `@Service`, `@Repository`, and controllers are normally singleton-scoped unless configured otherwise.

---

# 23. BeanPostProcessor ⭐ P1

`BeanPostProcessor` provides hooks to process beans during their lifecycle.

Conceptually:

```text
Create Bean
    ↓
BeanPostProcessor
(before initialization)
    ↓
@PostConstruct
    ↓
BeanPostProcessor
(after initialization)
    ↓
Final bean
```

A processor can inspect/modify a bean or return a wrapper/proxy.

### Why it matters

It helps explain framework features involving:

- AOP
- Proxies
- `@Transactional`
- Other bean transformations

### Interview answer

> BeanPostProcessor is an extension mechanism that allows Spring to process beans before and after initialization and can, for example, modify or wrap a bean.

---

# 24. Why Proxies Matter — Preview

This becomes important in AOP and transactions.

Conceptually:

```text
Application
    ↓
Spring Proxy
    ↓
Actual Bean
```

The proxy can perform framework logic around a method call.

For example:

```java
@Transactional
public void transfer() {
}
```

A proxy can participate in transaction handling around the method.

We will cover this properly in the AOP/transaction section.

---

# 25. PF Application Connection

Imagine part of the PF backend:

```text
ContributionController
        ↓
ContributionService
        ↓
TransactionService
        ↓
TransactionRepository
```

Spring can manage this dependency graph.

Example:

```java
@RestController
public class ContributionController {

    private final ContributionService service;

    public ContributionController(ContributionService service) {
        this.service = service;
    }
}
```

```java
@Service
public class ContributionService {

    private final TransactionService transactionService;

    public ContributionService(TransactionService transactionService) {
        this.transactionService = transactionService;
    }
}
```

Spring resolves:

```text
ContributionController
        ↓
ContributionService
        ↓
TransactionService
```

This is directly connected to how a typical Spring microservice is structured.

---

# 26. Interview Question Bank

## IoC / DI

### Q1. What is IoC?

> IoC means that responsibility for creating and managing application objects is transferred from application code to the Spring container.

### Q2. What is Dependency Injection?

> DI is the mechanism where dependencies are provided to an object from outside rather than the object creating them itself.

### Q3. IoC vs DI?

> IoC is the broader principle; DI is one of the primary mechanisms used to implement that principle.

### Q4. Why constructor injection?

> It makes dependencies explicit, enforces mandatory dependencies during construction, allows final fields, improves testability, and avoids partially initialized objects.

### Q5. Why is field injection discouraged?

> Dependencies are hidden, fields cannot normally be final, and standalone unit testing becomes less clean.

---

## Beans / ApplicationContext

### Q6. What is a Spring Bean?

> An object whose lifecycle and dependencies are managed by the Spring IoC container.

### Q7. How does Spring discover beans?

> Through component scanning of stereotypes such as `@Component`, `@Service`, `@Repository`, and `@Controller`, as well as explicit configuration through `@Bean`.

### Q8. What is ApplicationContext?

> Spring's IoC container that manages beans, dependency injection, lifecycle, configuration, and application-level services.

### Q9. `@Component` vs `@Bean`?

> `@Component` lets Spring discover a class through component scanning, while `@Bean` explicitly registers an object returned from a configuration method.

### Q10. When would you use `@Bean`?

> When I need explicit configuration or when registering a class I don't control, such as a third-party library.

---

## Lifecycle / Scopes

### Q11. Explain the Spring bean lifecycle.

> Spring creates the bean, injects dependencies, runs initialization callbacks such as `@PostConstruct`, and makes the bean available for use. During normal context shutdown, destruction callbacks such as `@PreDestroy` can run before the bean is destroyed.

### Q12. When does `@PostConstruct` execute?

> After construction and dependency injection, during bean initialization.

### Q13. When does `@PreDestroy` execute?

> During normal ApplicationContext shutdown, before bean destruction.

### Q14. Are singleton beans thread-safe?

> No. Singleton means one instance per ApplicationContext, not thread safety. Mutable shared state can still cause race conditions.

### Q15. What is prototype scope?

> It causes Spring to create a new bean instance when the bean is requested from the container.

### Q16. What does BeanPostProcessor do?

> It provides hooks to process beans before and after initialization and can modify or wrap beans.

---

# 27. High-Probability Follow-Up Chains

## Chain 1 — DI

```text
What is DI?
    ↓
Why constructor injection?
    ↓
Why not field injection?
    ↓
What if there are two implementations?
    ↓
@Primary vs @Qualifier
```

## Chain 2 — Bean

```text
What is a bean?
    ↓
How does Spring discover it?
    ↓
What is component scanning?
    ↓
@Component vs @Bean
    ↓
What is ApplicationContext?
```

## Chain 3 — Singleton

```text
Default scope?
    ↓
Singleton
    ↓
Is it thread-safe?
    ↓
No
    ↓
What happens with shared mutable state?
    ↓
Race condition
```

## Chain 4 — Lifecycle

```text
Bean lifecycle?
    ↓
@PostConstruct?
    ↓
@PreDestroy?
    ↓
BeanPostProcessor?
    ↓
Why do proxies matter?
    ↓
AOP / @Transactional
```

---

# 28. Don't Study These Deeply Before Monday

Useful later, but low return right now:

- BeanFactory internals
- BeanDefinition internals
- BeanDefinitionRegistry
- InstantiationAwareBeanPostProcessor
- SmartInstantiationAwareBeanPostProcessor
- Custom component scanner implementation
- Every obscure lifecycle callback
- Exact Spring source-code internals

Know the names if encountered, but don't spend valuable interview time here.

---

# 29. 60-Second Spring Core Summary

If asked:

> "Give me a high-level explanation of how Spring manages your application."

Think:

```text
Spring Boot
    ↓
ApplicationContext
    ↓
Component Scanning / @Bean
    ↓
Bean Definitions
    ↓
Bean Creation
    ↓
Dependency Resolution
    ↓
Constructor Injection
    ↓
Initialization
    ↓
Bean Ready
```

During runtime:

```text
Application
    ↓
Spring-managed beans
    ├── services
    ├── repositories
    ├── controllers
    └── other components
```

During shutdown:

```text
ApplicationContext shutdown
        ↓
@PreDestroy
        ↓
Bean destruction
```

---

# 30. Final Mental Model

Remember these six statements:

```text
1. IoC
   = Spring takes control of object creation/management.

2. DI
   = Spring provides dependencies to objects.

3. Bean
   = object managed by Spring.

4. ApplicationContext
   = Spring's IoC container/context.

5. Constructor injection
   = preferred way to inject mandatory dependencies.

6. Singleton
   = one instance per ApplicationContext,
     NOT automatically thread-safe.
```

Next:

```text
AOP
  ↓
Proxies
  ↓
JDK Dynamic Proxy vs CGLIB
  ↓
@Aspect / Advice / Pointcut
  ↓
@Around
  ↓
Self-invocation
  ↓
@Transactional
  ↓
Rollback
  ↓
Propagation
  ↓
Isolation
```

These are the next high-priority Spring Core interview topics.
