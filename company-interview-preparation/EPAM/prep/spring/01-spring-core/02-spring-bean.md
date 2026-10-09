# Spring Core — Part B: Spring Beans

This section covers how Spring creates, registers, manages, scopes, and injects beans, with practical EPAM-level follow-ups.

---

# Q11. What is a Spring Bean?

## Interview Answer

> **A Spring Bean is an object that is created, configured, and managed by the Spring IoC container.**

Example:

```java
@Service
public class PaymentService {
}
```

Spring discovers this class and creates an instance managed by the Spring container.

Another bean can then receive it:

```java
@Service
public class OrderService {

    private final PaymentService paymentService;

    public OrderService(PaymentService paymentService) {
        this.paymentService = paymentService;
    }
}
```

Both objects are managed by Spring.

---

## Important Distinction

Not every Java object is a Spring Bean.

```java
PaymentService service = new PaymentService();
```

This is simply a normal Java object.

Whereas:

```java
@Service
class PaymentService {
}
```

when discovered by Spring becomes a Spring Bean.

### Interview Follow-up

**Q: Does using `new` always mean the object is not a Spring Bean?**

If you manually create an object using `new`, Spring does not automatically manage that object as a bean.

For example:

```java
PaymentService service = new PaymentService();
```

Spring does not automatically apply its normal bean lifecycle, dependency injection, scopes, AOP proxying, etc. to that manually created instance.

---

# Q12. How does Spring create a Bean?

Suppose:

```java
@Service
public class PaymentService {
}
```

At a high level:

```text
Spring Boot starts
      ↓
ApplicationContext created
      ↓
Component scanning
      ↓
PaymentService discovered
      ↓
BeanDefinition registered
      ↓
Spring creates object
      ↓
Dependencies injected
      ↓
Bean lifecycle callbacks
      ↓
Bean becomes ready
```

The important concept is:

> **Spring first has metadata describing how the bean should be created, represented by a BeanDefinition, and then the IoC container uses that metadata to create and manage the bean.**

---

## Practical Interview Question

**Q: Does Spring immediately create the object when it discovers the class?**

Not necessarily.

Spring first registers bean metadata/definition. Whether the actual instance is created immediately depends on factors such as bean scope and lazy initialization.

For normal singleton beans, Spring generally creates them eagerly during ApplicationContext startup.

---

# Q13. What is `@Component`?

`@Component` tells Spring:

> **Detect this class during component scanning and register it as a Spring Bean.**

Example:

```java
@Component
public class EmailSender {
}
```

If the package is scanned, Spring registers `EmailSender` as a bean.

---

# Q14. What are `@Service`, `@Repository`, and `@Controller`?

They are specialized stereotype annotations built on top of `@Component`.

Conceptually:

```text
@Component
   ├── @Service
   ├── @Repository
   └── @Controller
```

### `@Service`

Used for business/service logic:

```java
@Service
class PaymentService {
}
```

### `@Repository`

Used for persistence/data-access components:

```java
@Repository
class PaymentRepository {
}
```

`@Repository` also participates in Spring's persistence exception-translation mechanism.

### `@Controller`

Used for Spring MVC controllers:

```java
@Controller
class PaymentController {
}
```

### `@RestController`

Commonly used for REST APIs.

It combines:

```java
@Controller
@ResponseBody
```

Conceptually:

```java
@RestController
class PaymentController {
}
```

---

## Interview Follow-up

**Q: Are `@Service`, `@Repository`, and `@Controller` completely different from `@Component`?**

They are specialized stereotypes built on the `@Component` model.

Their purpose is to communicate the role of the class, and some have additional framework semantics.

The important example is:

```text
@Repository
```

which participates in persistence exception translation.

---

# Q15. What is Component Scanning?

Spring needs to discover classes annotated with:

```java
@Component
@Service
@Repository
@Controller
```

Component scanning searches configured packages for these classes.

In Spring Boot:

```java
@SpringBootApplication
public class Application {
}
```

component scanning is enabled as part of the application configuration.

Typical structure:

```text
com.example
 ├── Application
 ├── service
 │     └── PaymentService
 ├── repository
 │     └── PaymentRepository
 └── controller
       └── PaymentController
```

These classes can be discovered because they are under the appropriate scan hierarchy.

---

## Practical Interview Question

**Q: Your `PaymentService` has `@Service`, but Spring says no bean exists. What would you check?**

Check:

```text
1. Is @Service/@Component present?
       ↓
2. Is the package being scanned?
       ↓
3. Is the class outside the component-scan hierarchy?
       ↓
4. Is a custom @ComponentScan restricting packages?
       ↓
5. Is the bean disabled by profile/conditional configuration?
       ↓
6. Did bean creation itself fail?
```

For example:

```text
com.example
   └── Application

com.other
   └── PaymentService
```

If `com.other` is not included in component scanning, Spring may not discover the service.

---

# Q16. What is Bean Naming?

When Spring registers a component:

```java
@Service
class PaymentService {
}
```

Spring normally creates a bean name based on the class name:

```text
PaymentService
      ↓
paymentService
```

The default naming convention generally decapitalizes the class name.

You can explicitly provide a name:

```java
@Service("upiPaymentService")
class UpiPaymentService implements PaymentService {
}
```

Now the bean name is:

```text
upiPaymentService
```

---

## Practical Question

**Q: Why does bean naming matter?**

It becomes important when:

- Using `@Qualifier`
- Injecting a `Map<String, BeanType>`
- Multiple beans of the same type exist
- You need explicit bean identification

For example:

```java
@Qualifier("upiPaymentService")
PaymentService paymentService
```

---

# Q17. `@Component` vs `@Bean`

This is a very common interview question.

## `@Component`

Placed on a class:

```java
@Component
class PaymentService {
}
```

Spring discovers it through component scanning.

## `@Bean`

Placed on a method:

```java
@Configuration
class AppConfig {

    @Bean
    PaymentService paymentService() {
        return new PaymentService();
    }
}
```

Here you explicitly tell Spring how to create the bean.

### Simple distinction

```text
@Component
    ↓
Spring discovers the class

@Bean
    ↓
You explicitly provide the object creation method
```

---

# Q18. When would you use `@Bean` instead of `@Component`?

A common case is a **third-party class** that you cannot modify.

For example:

```java
ObjectMapper objectMapper = new ObjectMapper();
```

You cannot add:

```java
@Component
```

to Jackson's `ObjectMapper` class.

So:

```java
@Configuration
class AppConfig {

    @Bean
    ObjectMapper objectMapper() {
        return new ObjectMapper();
    }
}
```

Now Spring manages the `ObjectMapper`.

---

## Typical use cases for `@Bean`

- Third-party classes
- Custom object creation
- Complex initialization
- Custom configuration
- Multiple differently configured instances

---

## Practical Interview Question

**Q: When would you choose `@Bean` over `@Component`?**

### Interview Answer

> I would use `@Component` when I control the class and want Spring to discover it through component scanning. I would use `@Bean` when I need explicit control over object creation, especially for third-party classes or objects requiring custom construction/configuration.

---

# Q19. What is `@Configuration`?

`@Configuration` indicates that a class contains Spring bean configuration.

Example:

```java
@Configuration
public class AppConfig {

    @Bean
    public PaymentService paymentService() {
        return new PaymentService();
    }
}
```

Spring processes the configuration and registers the returned object as a bean.

---

# Q20. `@Configuration` vs `@Component`

Both can be Spring-managed components, but they communicate different intent.

### `@Component`

General-purpose Spring-managed component.

```java
@Component
class PaymentService {
}
```

### `@Configuration`

Used specifically for configuration and bean definitions.

```java
@Configuration
class AppConfig {

    @Bean
    PaymentService paymentService() {
        return new PaymentService();
    }
}
```

### Interview Answer

> **`@Component` is a general-purpose Spring component, while `@Configuration` is specifically intended for configuration classes containing bean definitions.**

---

# Q21. What happens when `@Bean` methods call each other?

Consider:

```java
@Configuration
class AppConfig {

    @Bean
    PaymentService paymentService() {
        return new PaymentService(repository());
    }

    @Bean
    PaymentRepository repository() {
        return new PaymentRepository();
    }
}
```

With a normal full `@Configuration` class, Spring gives special semantics to the configuration class so that calls between `@Bean` methods can resolve the managed bean rather than simply creating an unmanaged duplicate through an ordinary Java method call.

Conceptually:

```text
paymentService()
      ↓
repository()
      ↓
Spring-managed repository bean
```

This is one reason `@Configuration` has special semantics compared with a plain component containing `@Bean` methods.

---

# Q22. What is Full vs Lite Configuration?

This is a useful deeper interview question.

A class annotated with:

```java
@Configuration
```

is treated as a configuration class with special handling for its `@Bean` methods.

A regular component containing `@Bean` methods does not necessarily receive the same full configuration semantics.

Example:

```java
@Component
class AppConfig {

    @Bean
    PaymentRepository repository() {
        return new PaymentRepository();
    }
}
```

This is different from:

```java
@Configuration
class AppConfig {

    @Bean
    PaymentRepository repository() {
        return new PaymentRepository();
    }
}
```

### Interview Point

> **Full `@Configuration` has special handling for inter-bean method calls, while lite configuration does not provide the same semantics.**

For EPAM, understand the concept rather than memorizing implementation details.

---

# Q23. What are Spring Bean Scopes?

Bean scope determines how Spring manages bean instances.

Important scopes include:

```text
Singleton
Prototype
Request
Session
Application
WebSocket
```

For backend interviews, understand these especially well:

```text
Singleton
Prototype
```

---

# Q24. What is Singleton Scope?

Singleton is the **default Spring bean scope**.

```java
@Service
class PaymentService {
}
```

Normally Spring creates one instance of this bean per `ApplicationContext`.

Conceptually:

```text
ApplicationContext
       |
       └── PaymentService instance #1
```

Multiple consumers receive the same bean instance.

---

# Q25. Is Spring Singleton the Same as Singleton Design Pattern?

No.

This is an important interview trap.

### Singleton Design Pattern

Generally aims to restrict a class to one instance within its relevant runtime scope.

### Spring Singleton

Means:

> **One bean instance per Spring ApplicationContext.**

For example:

```text
ApplicationContext A
      ↓
PaymentService #1

ApplicationContext B
      ↓
PaymentService #2
```

Therefore:

> **Spring singleton does not mean one object for the entire JVM.**

---

# Q26. Is a Spring Singleton Bean Thread-Safe?

**No, not automatically.**

Singleton means:

```text
One shared object
```

It does not mean:

```text
Thread-safe object
```

Suppose:

```java
@Service
class PaymentService {

    private int count = 0;

    public void process() {
        count++;
    }
}
```

Multiple request threads can access the same bean:

```text
Thread 1 ─┐
Thread 2 ─┼──> Same PaymentService
Thread 3 ─┘
```

Therefore `count++` can have race conditions.

### Better design

Keep singleton services stateless:

```java
@Service
class PaymentService {

    public PaymentResponse process(
            PaymentRequest request) {

        int amount = request.getAmount();

        // business logic

        return ...;
    }
}
```

Local variables belong to the individual method execution.

### Interview Answer

> **Singleton scope and thread safety are different concepts. Spring controls the bean's scope, but it does not automatically make mutable shared state thread-safe.**

---

# Q27. What is Prototype Scope?

Prototype means Spring creates a new bean instance whenever the bean is requested from the container.

Example:

```java
@Component
@Scope("prototype")
class ReportGenerator {
}
```

If we request it twice:

```java
ReportGenerator r1 =
        context.getBean(ReportGenerator.class);

ReportGenerator r2 =
        context.getBean(ReportGenerator.class);
```

then:

```java
r1 != r2
```

Conceptually:

```text
getBean()
   ↓
Instance #1

getBean()
   ↓
Instance #2

getBean()
   ↓
Instance #3
```

---

# Q28. Singleton vs Prototype

| Singleton | Prototype |
|---|---|
| Default scope | Must be explicitly configured |
| One instance per ApplicationContext | New instance when requested |
| Shared between consumers | Separate instances |
| Common for stateless services | Useful for stateful objects |
| Container manages singleton lifecycle | Destruction lifecycle is not managed in the same complete way |

---

# Q29. What happens if a Singleton depends on a Prototype?

This is a **very important Spring interview trap**.

Suppose:

```java
@Component
@Scope("prototype")
class PaymentContext {
}
```

and:

```java
@Service
class PaymentService {

    private final PaymentContext context;

    PaymentService(PaymentContext context) {
        this.context = context;
    }
}
```

You might think:

> Every time `PaymentService` uses `context`, Spring will create a new `PaymentContext`.

That is **not what happens**.

Spring creates the singleton once.

During singleton creation, it obtains a prototype instance:

```text
PaymentService created
       ↓
PaymentContext #1 injected
       ↓
PaymentService stores #1
```

Later:

```text
PaymentService.process()
       ↓
same PaymentContext #1
```

The prototype is **not automatically recreated on every method call**.

---

# Q30. How can a Singleton get a new Prototype instance every time?

Use `ObjectProvider`.

```java
@Service
class PaymentService {

    private final ObjectProvider<PaymentContext> provider;

    PaymentService(
            ObjectProvider<PaymentContext> provider) {

        this.provider = provider;
    }

    public void process() {

        PaymentContext context =
                provider.getObject();

        // use fresh prototype
    }
}
```

Each call to:

```java
provider.getObject()
```

can obtain a new prototype instance.

Another mechanism is `@Lookup`, but `ObjectProvider` is important to understand for interviews.

---

# Q31. What are Request, Session, Application and WebSocket scopes?

These are primarily web-related scopes.

### Request Scope

One bean instance per HTTP request.

```text
Request 1 → Bean #1
Request 2 → Bean #2
Request 3 → Bean #3
```

### Session Scope

One bean instance per HTTP session.

```text
Session A → Bean #1
Session B → Bean #2
```

### Application Scope

One bean instance associated with the web application's `ServletContext`.

### WebSocket Scope

One bean instance associated with a WebSocket session.

### Interview Point

For normal backend service beans, singleton is usually the common default.

---

# Q32. What happens if you inject a Request-Scoped Bean into a Singleton?

This is a deeper practical question.

A singleton lives much longer than an individual HTTP request.

Spring therefore needs a mechanism that allows the singleton to interact with the request-scoped object associated with the **current request**, rather than simply capturing one request object permanently.

Spring can use scoped proxies or other scope-aware mechanisms.

Conceptually:

```text
Singleton Service
      ↓
Scoped Proxy
      ↓
Current HTTP Request's Bean
```

### Interview Point

> **Different bean scopes require Spring to handle lifecycle and lookup appropriately when a shorter-lived bean is used by a longer-lived bean.**

---

# Q33. Does Spring manage Prototype Destruction?

Not in the same complete way as singleton beans.

Spring creates and initializes the prototype bean, but after handing it to the requesting code, the container generally does not manage its complete destruction lifecycle.

Therefore, if a prototype object owns resources requiring cleanup, the application may need to handle that cleanup.

### Interview Answer

> **Spring manages creation and initialization of prototype beans, but does not provide the same complete destruction management that it provides for singleton beans.**

---

# Q34. Can we have multiple beans of the same class?

Yes.

For example:

```java
@Configuration
class AppConfig {

    @Bean
    PaymentService payment1() {
        return new PaymentService();
    }

    @Bean
    PaymentService payment2() {
        return new PaymentService();
    }
}
```

Spring now has:

```text
payment1 → PaymentService instance #1
payment2 → PaymentService instance #2
```

These are two different bean definitions and two different instances.

---

# Q35. How do you inject a specific bean when multiple beans have the same class?

Use a qualifier.

```java
@Configuration
class AppConfig {

    @Bean
    PaymentService payment1() {
        return new PaymentService();
    }

    @Bean
    PaymentService payment2() {
        return new PaymentService();
    }
}
```

Then:

```java
OrderService(
    @Qualifier("payment1")
    PaymentService paymentService
)
```

Spring selects the bean named `payment1`.

---

# Q36. What happens if two `@Bean` methods return the same interface type?

Example:

```java
@Configuration
class AppConfig {

    @Bean
    PaymentService upiPaymentService() {
        return new UpiPaymentService();
    }

    @Bean
    PaymentService cardPaymentService() {
        return new CardPaymentService();
    }
}
```

Spring now has:

```text
PaymentService
   ├── upiPaymentService
   └── cardPaymentService
```

If we write:

```java
OrderService(PaymentService paymentService)
```

the dependency is ambiguous.

Use:

```java
@Qualifier("upiPaymentService")
```

or:

```java
@Primary
```

depending on the requirement.

---

# Q37. Practical Question — Configure a Third-Party Object

### Requirement

You want Spring to manage:

```java
ObjectMapper
```

but you cannot annotate the third-party class.

Use:

```java
@Configuration
class JacksonConfig {

    @Bean
    ObjectMapper objectMapper() {

        ObjectMapper mapper =
                new ObjectMapper();

        // custom configuration

        return mapper;
    }
}
```

Now Spring manages the returned `ObjectMapper`.

### Interview Follow-up

**Why not use `@Component`?**

Because you generally cannot modify the third-party class to add the annotation.

`@Bean` gives explicit control over construction.

---

# Q38. Practical Question — Two Different Configurations of the Same Class

Suppose you need two `ObjectMapper` beans:

```text
Default ObjectMapper
Audit ObjectMapper
```

You can define:

```java
@Configuration
class Config {

    @Bean
    ObjectMapper defaultObjectMapper() {
        return new ObjectMapper();
    }

    @Bean
    ObjectMapper auditObjectMapper() {
        return new ObjectMapper();
    }
}
```

Then select one using:

```java
@Qualifier("auditObjectMapper")
```

This is a practical reason for explicit `@Bean` definitions.

---

# Q39. Practical Debugging — Bean Not Found

### Scenario

You add:

```java
@Service
class PaymentService {
}
```

but startup fails because Spring cannot find it.

### Debugging sequence

```text
@Service present?
       ↓
Package scanned?
       ↓
Custom @ComponentScan?
       ↓
Correct ApplicationContext?
       ↓
Profile active?
       ↓
Conditional configuration?
       ↓
Bean initialization failing?
```

### Interview Answer

> **I would first verify bean registration and component scanning, then check profiles/conditional configuration and finally inspect the startup stack trace to determine whether the bean was discovered but failed during creation.**

---

# Q40. Practical Debugging — Bean Created Twice?

Suppose you accidentally configure the same logical dependency through both:

```java
@Component
class PaymentService {
}
```

and:

```java
@Configuration
class Config {

    @Bean
    PaymentService paymentService() {
        return new PaymentService();
    }
}
```

Spring can end up with multiple beans representing the same type.

Then an injection point such as:

```java
OrderService(PaymentService paymentService)
```

may become ambiguous.

### Debugging approach

Check:

- Component scanning
- `@Component`
- `@Bean`
- Imported configuration
- Explicit bean definitions
- Bean names

---

# Q41. Practical Interview Question — Singleton Service Design

### Bad Design

```java
@Service
class PaymentService {

    private PaymentRequest request;

    public void process(PaymentRequest request) {
        this.request = request;

        // processing
    }
}
```

Because `PaymentService` is normally a singleton, multiple requests can modify the same instance field.

### Better Design

```java
@Service
class PaymentService {

    public void process(PaymentRequest request) {

        // use request locally
    }
}
```

### Interview Answer

> **Spring singleton services should generally be stateless. Request-specific data should be kept in method-local variables or appropriate request-scoped objects rather than mutable singleton fields.**

---

# Q42. Practical Design Question — Payment Bean Architecture

Design:

```text
PaymentController
       ↓
PaymentProcessor
       ↓
PaymentService
    /    |     \
  UPI   CARD   WALLET
```

Requirements:

- All implementations should be Spring beans.
- The processor should not manually create implementations.
- Adding a new payment type should be easy.
- The processor should be unit-testable.

### Solution

Define:

```java
public interface PaymentService {

    void pay(double amount);
}
```

Implement:

```java
@Service
class UpiPaymentService implements PaymentService {

    public void pay(double amount) {
        // UPI
    }
}
```

```java
@Service
class CardPaymentService implements PaymentService {

    public void pay(double amount) {
        // CARD
    }
}
```

```java
@Service
class WalletPaymentService implements PaymentService {

    public void pay(double amount) {
        // WALLET
    }
}
```

Then inject them into the processor:

```java
@Service
class PaymentProcessor {

    private final List<PaymentService> services;

    PaymentProcessor(List<PaymentService> services) {
        this.services = services;
    }
}
```

Or use:

```java
Map<String, PaymentService>
```

if dynamic lookup is required.

---

# Q43. How does Spring know which beans belong in a collection?

For:

```java
List<PaymentService>
```

Spring identifies the required element type:

```text
PaymentService
```

and finds all beans assignable to that type.

For:

```java
Set<PaymentService>
```

the same matching principle applies.

For:

```java
Map<String, PaymentService>
```

Spring collects matching beans and uses their bean names as the String keys.

---

# Q44. What is the difference between List, Set and Map injection?

### List

Use when you want all implementations and ordering/position can matter.

```java
List<PaymentService>
```

### Set

Use when you want all implementations with set semantics.

```java
Set<PaymentService>
```

### Map

Use when you want to select an implementation using a key.

```java
Map<String, PaymentService>
```

Conceptually:

```text
One specific bean
      ↓
@Qualifier

All beans
      ↓
List / Set

Dynamic lookup
      ↓
Map
```

---

# Q45. Practical EPAM Question — Add a New Implementation

Suppose the application currently has:

```text
UPI
CARD
```

and tomorrow we add:

```text
WALLET
```

We create:

```java
@Service
class WalletPaymentService
        implements PaymentService {
}
```

If an existing consumer uses:

```java
List<PaymentService>
```

or:

```java
Map<String, PaymentService>
```

Spring can automatically include the new bean in the collection.

### Interview Answer

> **Because the implementations are discovered as Spring beans and injected by their common type, adding a new implementation can automatically make it available to collection injection. The selection/business-key logic still needs to be designed appropriately.**

---

# Q46. What happens if a Bean throws an exception during initialization?

Suppose:

```java
@Service
class PaymentService {

    @PostConstruct
    void init() {

        throw new RuntimeException(
                "Initialization failed");
    }
}
```

Spring cannot successfully initialize that bean.

If the bean is required during application startup, the ApplicationContext startup can fail.

### Practical Production Point

Initialization code should therefore:

- Fail fast when required configuration is invalid.
- Avoid unnecessary expensive work.
- Handle external-resource initialization carefully.
- Produce useful logs.

---

# Q47. What happens when the Spring ApplicationContext is closed?

For singleton beans, Spring executes destruction callbacks such as:

```java
@PreDestroy
```

or configured destroy methods.

Conceptually:

```text
Application shutdown
       ↓
Destroy singleton beans
       ↓
@PreDestroy
       ↓
Cleanup
```

Prototype destruction is different because Spring does not manage the complete destruction lifecycle of prototype instances after handing them to the caller.

---

# Q48. Is every Spring Bean created at application startup?

No.

For normal singleton beans, Spring generally creates them eagerly during ApplicationContext startup.

But lazy initialization can defer creation:

```java
@Lazy
@Service
class PaymentService {
}
```

Prototype beans are created when requested from the container rather than as one eagerly created singleton.

---

# Q49. Does Spring Bean Scope determine thread safety?

No.

Scope determines **lifetime/instance behavior**.

Thread safety is a separate concern.

For example:

```text
Singleton
   ↓
One shared instance
   ↓
Potential concurrent access
   ↓
Developer must ensure thread safety
```

A prototype bean is not automatically thread-safe either.

### Interview Answer

> **Bean scope determines how Spring manages instances; thread safety depends on the object's state and synchronization/design.**

---

# Q50. Can a Spring Bean be created manually using `new`?

Yes, Java allows it:

```java
PaymentService service =
        new PaymentService();
```

But that manually created object is not automatically a Spring-managed bean.

Therefore it won't automatically receive normal Spring-managed features such as:

- Dependency injection
- Bean lifecycle callbacks
- Spring AOP proxying
- Declarative transaction interception
- Spring-managed scope behavior

### Interview Point

> **If Spring needs to manage an object's lifecycle and dependencies, let the container create/manage it rather than manually constructing it in business code.**

---

# EPAM Practical Interview Scenarios

## Scenario 1 — `@Service` Not Found

```text
Application startup fails:
NoSuchBeanDefinitionException
```

Answer:

```text
Check annotation
      ↓
Check component scanning
      ↓
Check package structure
      ↓
Check profiles/conditions
      ↓
Check bean creation errors
```

---

## Scenario 2 — New Implementation Breaks Existing Application

You add:

```java
@Service
class WalletPaymentService
        implements PaymentService {
}
```

and startup now fails.

Reason:

```text
Previously:
PaymentService → one candidate

Now:
PaymentService → UPI + CARD + WALLET
```

Existing injection points may become ambiguous.

Fix according to requirement:

```text
Specific → @Qualifier

Default → @Primary

All → List / Set

Dynamic → Map
```

---

## Scenario 3 — Prototype Doesn't Produce New Object

You have:

```java
@Scope("prototype")
@Component
class PaymentContext {
}
```

injected directly into:

```java
@Service
class PaymentService {
}
```

You expect a new object every method call.

That expectation is wrong.

The prototype is resolved when the singleton is created.

For a new instance on demand:

```java
ObjectProvider<PaymentContext>
```

---

## Scenario 4 — Singleton Has Request Data

You find:

```java
@Service
class PaymentService {

    private PaymentRequest request;
}
```

This is suspicious because the service is normally singleton-scoped.

Multiple requests may access the same instance.

Recommended approach:

```java
public void process(PaymentRequest request) {
    // request remains local to invocation
}
```

---

# Rapid-Fire Spring Bean Questions

### What is a Spring Bean?

An object managed by the Spring IoC container.

### What is the default bean scope?

Singleton.

### Does singleton mean one instance for the JVM?

No. One instance per ApplicationContext.

### Is a singleton bean automatically thread-safe?

No.

### What does `@Component` do?

Registers a class as a candidate for component scanning and Spring bean creation.

### What does `@Service` do?

Provides a service-layer stereotype on top of the component model.

### What does `@Repository` do?

Marks a persistence component and participates in persistence exception translation.

### What does `@Controller` do?

Marks a Spring MVC controller.

### What does `@RestController` do?

Combines controller semantics with response-body behavior.

### What does `@Bean` do?

Explicitly declares a bean-producing method.

### When would you use `@Bean`?

For third-party classes, custom construction, or explicit bean configuration.

### What does `@Configuration` do?

Marks a configuration class containing bean definitions.

### Can two beans have the same class?

Yes.

### Can two beans have the same type?

Yes.

### What happens if one dependency matches multiple beans?

Spring needs the ambiguity resolved using mechanisms such as `@Qualifier`, `@Primary`, or collection injection.

### What is prototype scope?

A new instance is created whenever the prototype bean is requested from the container.

### Does injecting a prototype into a singleton create a new prototype every method call?

No.

### How can we obtain a fresh prototype instance?

Use `ObjectProvider`, `ObjectFactory`, `@Lookup`, or another appropriate lookup mechanism.

### Does Spring fully manage prototype destruction?

No.

### What is component scanning?

The process by which Spring discovers component classes in configured packages.

### Why might `@Service` not be discovered?

It may be outside the component-scan range or otherwise excluded by configuration.

---
## Q. What is the purpose of `@Repository`? What is exception translation?

### Interview Answer

`@Repository` is a specialized Spring stereotype used for the **persistence/data-access layer**.

One important additional feature is that Spring's persistence exception-translation mechanism can translate **database/persistence-technology-specific exceptions** such as JDBC or Hibernate exceptions into Spring's common `DataAccessException` hierarchy.

For example:

```text
Database
   ↓
JDBC / Hibernate
   ↓
Technology-specific exception
   ↓
Spring exception translation
   ↓
DataAccessException
   ↓
Service layer
```

For example, a database constraint violation may ultimately be exposed as:

```java
DataIntegrityViolationException
```

instead of requiring the service layer to depend directly on:

```java
SQLException
Hibernate ConstraintViolationException
```

### Important clarification

`@Repository` itself does **not mean that we catch the exception inside the repository**.

Instead, `@Repository` marks the class as a persistence component so that Spring's exception-translation infrastructure can apply.

The exception can then propagate to the service layer:

```java
@Service
public class UserService {

    public void register(User user) {
        try {
            userRepository.save(user);
        } catch (DataIntegrityViolationException e) {
            throw new UserAlreadyExistsException(
                "User already exists"
            );
        }
    }
}
```

The service layer can then convert the technical persistence problem into a business-level exception if required.

### What about Spring Data repositories?

With:

```java
public interface UserRepository
        extends JpaRepository<User, Long> {
}
```

we don't write the repository implementation ourselves. Spring Data creates the implementation/proxy.

Persistence exceptions can still participate in Spring's exception-translation mechanism.

### Interview-ready one-liner

> **`@Repository` marks a persistence component and enables it to participate in Spring's persistence exception-translation mechanism, which converts technology-specific exceptions such as JDBC/Hibernate exceptions into Spring's `DataAccessException` hierarchy.**

### Example Flow

```text
PostgreSQL
    ↓
SQLException
    ↓
Hibernate/JPA
    ↓
Persistence exception
    ↓
Spring exception translation
    ↓
DataIntegrityViolationException
    ↓
Service layer
    ↓
Business exception / appropriate response
```

### Key distinction

```text
@Component
    ↓
Generic Spring component

@Service
    ↓
Business/service layer

@Repository
    ↓
Persistence/data-access layer
    +
    Exception translation semantics

@Controller
    ↓
Web/MVC layer
```

So the key reason to remember for interviews is:

> **`@Repository` is not just a naming convention. It identifies the persistence layer and participates in exception translation.**

# EPAM Hands-On Checklist

Before moving to the next Spring topic, be able to implement and explain all of these:

### 1. Create a Spring Bean

```java
@Service
class PaymentService {
}
```

### 2. Create a Bean Using `@Bean`

```java
@Configuration
class Config {

    @Bean
    ObjectMapper objectMapper() {
        return new ObjectMapper();
    }
}
```

### 3. Create Multiple Beans of the Same Type

```java
@Bean
PaymentService payment1() {
    return new PaymentService();
}

@Bean
PaymentService payment2() {
    return new PaymentService();
}
```

### 4. Resolve Them

```java
@Qualifier("payment1")
```

### 5. Inject All Implementations

```java
List<PaymentService>
```

### 6. Dynamic Lookup

```java
Map<String, PaymentService>
```

### 7. Prototype Scope

```java
@Scope("prototype")
```

### 8. Prototype Inside Singleton

Understand why direct injection doesn't create a new prototype on every method call.

### 9. Fresh Prototype Lookup

```java
ObjectProvider<PrototypeService>
```

### 10. Debug Bean Creation

Be able to diagnose:

```text
NoSuchBeanDefinitionException
NoUniqueBeanDefinitionException
BeanCreationException
```

---

# Final Mental Model

Think about Spring Beans like this:

```text
                 Spring Container
                       |
                 BeanDefinition
                       |
              +--------+--------+
              |                 |
        Component Scan       @Bean
              |                 |
              +--------+--------+
                       |
                 Bean Creation
                       |
               Dependency Injection
                       |
                 Bean Lifecycle
                       |
                  Bean Scope
                 /           \
          Singleton         Prototype
```

And remember the practical decisions:

```text
Need one specific implementation?
        ↓
@Qualifier

Need a default implementation?
        ↓
@Primary

Need all implementations?
        ↓
List / Set

Need dynamic implementation lookup?
        ↓
Map

Need a third-party/custom object?
        ↓
@Bean

Need a normal application component?
        ↓
@Component / @Service / @Repository / @Controller

Need a new prototype for every lookup?
        ↓
ObjectProvider / lookup mechanism
```

# Part B Complete

```text
Spring Bean
    ↓
@Component
    ↓
@Service / @Repository / @Controller
    ↓
Component Scanning
    ↓
Bean Naming
    ↓
@Bean
    ↓
@Configuration
    ↓
Full vs Lite Configuration
    ↓
Bean Scopes
    ↓
Singleton
    ↓
Prototype
    ↓
Web Scopes
    ↓
Singleton + Prototype Trap
    ↓
ObjectProvider
    ↓
Bean Creation / Debugging
    ↓
Thread Safety
```

**Next: Spring Core — Part C: Dependency Injection + Bean Lifecycle**

This will go deeper into:

```text
@Autowired resolution
Collection injection
Optional dependencies
Circular dependencies
@Lazy
BeanDefinition
Bean lifecycle
Aware interfaces
@PostConstruct
InitializingBean
BeanPostProcessor
BeanFactoryPostProcessor
@PreDestroy
Prototype destruction
```