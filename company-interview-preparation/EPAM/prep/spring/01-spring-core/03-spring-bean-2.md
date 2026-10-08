# Spring Core — Part C: Dependency Injection + Bean Lifecycle

This section goes deeper into how Spring resolves dependencies, handles multiple beans, manages circular dependencies, and controls the complete bean lifecycle.

---

# Part C — Dependency Injection & Autowiring

## Q21. How does `@Autowired` work?

### Interview Answer

`@Autowired` tells Spring to resolve and inject a dependency from the ApplicationContext.

The basic resolution process is:

```text
Injection point
      ↓
Find beans matching required type
      ↓
Exactly one?
   ↓       ↓
  Yes      No
   ↓       ↓
Inject   Resolve ambiguity
          ↓
     @Qualifier / @Primary
```

Example:

```java
@Service
public class OrderService {

    private final PaymentService paymentService;

    public OrderService(PaymentService paymentService) {
        this.paymentService = paymentService;
    }
}
```

Spring searches for a bean compatible with:

```java
PaymentService
```

and injects it.

### If no matching bean exists

Spring cannot satisfy the dependency and typically throws:

```text
NoSuchBeanDefinitionException
```

or a related `NoSuchBeanDefinitionException`/unsatisfied-dependency failure during context creation.

### If multiple beans match

Spring may throw:

```text
NoUniqueBeanDefinitionException
```

unless the ambiguity is resolved.

---

# Q22. How does Spring resolve multiple beans of the same type?

Suppose:

```java
public interface PaymentService {
}
```

and:

```java
@Service
class UpiPaymentService implements PaymentService {
}
```

```java
@Service
class CardPaymentService implements PaymentService {
}
```

Now:

```java
@Service
class OrderService {

    private final PaymentService paymentService;

    OrderService(PaymentService paymentService) {
        this.paymentService = paymentService;
    }
}
```

There are two candidates:

```text
PaymentService
      |
      +---- UpiPaymentService
      |
      +---- CardPaymentService
```

Spring cannot determine which one should be injected.

This produces an ambiguity.

We can resolve it using:

```text
@Qualifier
@Primary
```

---

# Q23. What is `@Qualifier`?

`@Qualifier` tells Spring exactly which bean should be injected when multiple candidates exist.

Example:

```java
@Service("upiPaymentService")
class UpiPaymentService implements PaymentService {
}
```

Then:

```java
@Service
class OrderService {

    private final PaymentService paymentService;

    OrderService(
        @Qualifier("upiPaymentService")
        PaymentService paymentService) {

        this.paymentService = paymentService;
    }
}
```

Spring selects:

```text
upiPaymentService
```

instead of trying to choose between all `PaymentService` implementations.

### Interview Answer

> `@Qualifier` provides a more specific identifier for dependency resolution when multiple beans match the required type.

---

# Q24. What is `@Primary`?

`@Primary` tells Spring:

> "If multiple beans match this type and no more specific qualifier is provided, prefer this bean."

Example:

```java
@Primary
@Service
class UpiPaymentService implements PaymentService {
}
```

Now:

```java
OrderService(PaymentService paymentService)
```

will normally receive `UpiPaymentService`.

---

# Q25. `@Primary` vs `@Qualifier`

This is a common interview question.

### `@Primary`

Defines the **default/preferred candidate**.

```java
@Primary
@Service
class UpiPaymentService implements PaymentService {
}
```

### `@Qualifier`

Makes the dependency selection **specific**.

```java
OrderService(
    @Qualifier("cardPaymentService")
    PaymentService paymentService
)
```

Think:

```text
@Primary
   ↓
"default choice"

@Qualifier
   ↓
"give me this exact candidate"
```

### Interview Answer

> `@Primary` is useful when one bean should be the default candidate, while `@Qualifier` is useful when a particular injection point needs a specific bean.

---

# Q26. Which one has more specific selection semantics: `@Primary` or `@Qualifier`?

`@Qualifier` is more specific.

Suppose:

```java
@Primary
@Service
class UpiPaymentService implements PaymentService {
}
```

and:

```java
@Service
class CardPaymentService implements PaymentService {
}
```

If we write:

```java
PaymentService paymentService
```

Spring chooses the primary bean.

But if we write:

```java
@Qualifier("cardPaymentService")
PaymentService paymentService
```

we explicitly ask for the card implementation.

So:

```text
Type
 ↓
@Primary → default candidate
 ↓
@Qualifier → specific candidate
```

---

# Q27. Can Spring inject all implementations of an interface?

Yes.

For example:

```java
public OrderService(
        List<PaymentService> paymentServices) {

    this.paymentServices = paymentServices;
}
```

If we have:

```text
UpiPaymentService
CardPaymentService
WalletPaymentService
```

Spring can inject all of them into the list.

Conceptually:

```text
List<PaymentService>

    ↓

[UPI, CARD, WALLET]
```

This is extremely useful for implementing strategy-style designs.

---

# Q28. Can Spring inject a `Set` of beans?

Yes.

```java
public OrderService(
        Set<PaymentService> paymentServices) {
}
```

Spring finds all matching beans and injects them into the set.

The important idea is:

> Spring resolves collection elements based on their required bean type.

---

# Q29. Can Spring inject a `Map<String, PaymentService>`?

Yes.

```java
public OrderService(
        Map<String, PaymentService> paymentServices) {
}
```

Spring finds all `PaymentService` beans and uses their bean names as the map keys.

For example:

```text
Map<String, PaymentService>

"upiPaymentService"    → UpiPaymentService
"cardPaymentService"   → CardPaymentService
"walletPaymentService" → WalletPaymentService
```

This is useful when the application needs dynamic selection.

Example:

```java
PaymentService service =
        paymentServices.get(paymentType);
```

---

# Q30. Why is collection injection useful in real applications?

Consider a payment system:

```text
PaymentService
     |
     +---- UPI
     +---- CARD
     +---- WALLET
     +---- NET_BANKING
```

Instead of writing:

```java
if (type.equals("UPI")) {
    new UpiPaymentService();
}
else if (type.equals("CARD")) {
    new CardPaymentService();
}
```

we can let Spring inject all implementations.

```java
@Service
class PaymentProcessor {

    private final Map<String, PaymentService> services;

    PaymentProcessor(
        Map<String, PaymentService> services) {

        this.services = services;
    }

    public void process(
            String type,
            PaymentRequest request) {

        PaymentService service =
                services.get(type);

        service.pay(request);
    }
}
```

This gives us:

```text
Spring DI
   ↓
All strategies registered automatically
   ↓
Map
   ↓
Runtime selection
```

Adding another implementation can become much easier.

---

# Q31. How does Spring know what to put into `List<PaymentService>`?

Spring looks at the generic element type:

```java
List<PaymentService>
```

The required element type is:

```java
PaymentService
```

Spring finds all beans assignable to that type.

For example:

```java
@Service
class UpiPaymentService
        implements PaymentService {
}
```

```java
@Service
class CardPaymentService
        implements PaymentService {
}
```

Both become candidates for:

```java
List<PaymentService>
```

---

# Q32. Can Spring inject optional dependencies?

Yes.

Sometimes a dependency is optional rather than mandatory.

One approach is:

```java
@Autowired(required = false)
private PaymentService paymentService;
```

Another, often cleaner approach, is:

```java
Optional<PaymentService>
```

Example:

```java
@Service
class OrderService {

    private final Optional<PaymentService> paymentService;

    OrderService(
        Optional<PaymentService> paymentService) {

        this.paymentService = paymentService;
    }
}
```

Then:

```java
if (paymentService.isPresent()) {
    // use it
}
```

You can also use `ObjectProvider` when you need lazy/on-demand access.

### Interview Point

For constructor-based design, `Optional<T>` or `ObjectProvider<T>` is generally clearer than making a required constructor dependency silently optional.

---

# Q33. What is a Circular Dependency?

A circular dependency occurs when beans depend on each other directly or indirectly.

Simple example:

```text
A → B
B → A
```

Code:

```java
@Service
class A {

    private final B b;

    A(B b) {
        this.b = b;
    }
}
```

```java
@Service
class B {

    private final A a;

    B(A a) {
        this.a = a;
    }
}
```

Spring tries:

```text
Create A
   ↓
A needs B
   ↓
Create B
   ↓
B needs A
   ↓
A is not fully created
```

The dependency cycle cannot be satisfied through normal constructor creation.

---

# Q34. Why does constructor injection expose circular dependencies?

Constructor injection requires the dependency to exist **before the object can be constructed**.

For:

```java
A(B b)
```

Spring cannot create `A` until it has a `B`.

But:

```java
B(A a)
```

means Spring cannot create `B` until it has an `A`.

Therefore:

```text
A
 ↓ needs B
B
 ↓ needs A
A
 ↓
cycle
```

Constructor injection makes this dependency cycle explicit and causes startup failure rather than allowing partially initialized objects.

---

# Q35. What happens with circular dependency during application startup?

With constructor-based circular dependencies, Spring cannot complete bean creation.

The application context fails to initialize and Spring reports a circular dependency problem.

Conceptually:

```text
Application startup
       ↓
Create A
       ↓
Need B
       ↓
Create B
       ↓
Need A
       ↓
Circular dependency
       ↓
Context startup failure
```

---

# Q36. What is the best solution for a circular dependency?

The best solution is usually:

> **Redesign the classes and remove the circular dependency.**

For example:

```text
Bad:

A → B
B → A
```

Maybe both classes contain responsibilities that should actually belong to another component:

```text
A → C
B → C
```

or:

```text
A → B
```

with `B` no longer depending on `A`.

### Practical interview answer

> I would first treat circular dependency as a design smell and refactor the responsibilities. I wouldn't immediately use `@Lazy` just to hide the cycle.

---

# Q37. How can we resolve a circular dependency if refactoring isn't immediately possible?

Possible mechanisms include:

```text
@Lazy
Setter injection
Other lookup mechanisms
```

For example:

```java
@Service
class A {

    private final B b;

    A(@Lazy B b) {
        this.b = b;
    }
}
```

`@Lazy` can defer creation of the dependency and can break certain cycles.

However:

> **`@Lazy` is a technical workaround, not generally the preferred architectural solution.**

---

# Q38. What does `@Lazy` do?

Normally singleton beans are eagerly created during ApplicationContext startup.

For example:

```java
@Service
class PaymentService {
}
```

is normally initialized during startup.

With:

```java
@Lazy
@Service
class PaymentService {
}
```

Spring delays its initialization until the bean is actually needed.

Conceptually:

```text
Normal:

Application startup
      ↓
Create PaymentService
      ↓
Application ready


@Lazy:

Application startup
      ↓
Don't create PaymentService yet
      ↓
Application ready
      ↓
First actual request
      ↓
Create PaymentService
```

---

# Q39. When is `@Lazy` useful?

Common cases:

### 1. Expensive initialization

If creating a bean takes significant time/resources:

```java
@Lazy
@Service
class ExpensiveService {
}
```

### 2. Rarely used functionality

If a component is not needed for most application executions, lazy initialization can defer its creation.

### 3. Certain circular dependencies

`@Lazy` can break certain dependency cycles.

But again:

> **Don't use `@Lazy` as the first solution to a circular dependency. Refactoring is usually better.**

---

# Part D — Bean Lifecycle

Now we move into one of the most important Spring Core interview areas.

---

# Q40. What is the Spring Bean Lifecycle?

A simplified lifecycle is:

```text
BeanDefinition
      ↓
Instantiate bean
      ↓
Populate dependencies
      ↓
Aware callbacks
      ↓
BeanPostProcessor
(before initialization)
      ↓
@PostConstruct
      ↓
InitializingBean / init-method
      ↓
BeanPostProcessor
(after initialization)
      ↓
Bean ready
      ↓
Application runs
      ↓
ApplicationContext shutdown
      ↓
@PreDestroy
      ↓
DisposableBean / destroy-method
```

The exact internal lifecycle has more details, but this is the interview-level mental model you should know.

---

# Q41. Where does Dependency Injection happen in the Bean Lifecycle?

After Spring instantiates the bean, it populates/injects its dependencies.

Conceptually:

```text
Instantiate object
      ↓
Inject dependencies
      ↓
Initialization callbacks
      ↓
Bean ready
```

For example:

```java
@Service
class PaymentService {

    private final ConfigService config;

    PaymentService(ConfigService config) {
        this.config = config;
    }
}
```

Spring resolves `ConfigService` and supplies it while constructing `PaymentService`.

For field/setter injection, dependency population happens after the object itself has been instantiated.

---

# Q42. What is `@PostConstruct`?

`@PostConstruct` marks a method that Spring invokes during bean initialization after dependency injection has occurred.

Example:

```java
@Service
class PaymentService {

    private final ConfigService config;

    PaymentService(ConfigService config) {
        this.config = config;
    }

    @PostConstruct
    void init() {
        // dependencies are available here
    }
}
```

Conceptually:

```text
Constructor
    ↓
Dependency injection
    ↓
@PostConstruct
    ↓
Bean ready
```

---

# Q43. Why use `@PostConstruct` instead of the constructor?

The constructor is primarily for **constructing the object and establishing required dependencies**.

`@PostConstruct` is useful when initialization logic needs the bean's dependencies to already be available.

Example:

```java
@Service
class PaymentService {

    private final ConfigService config;

    PaymentService(ConfigService config) {
        this.config = config;
    }

    @PostConstruct
    void init() {
        String mode = config.getPaymentMode();
    }
}
```

The `config` dependency is available during `@PostConstruct`.

---

# Q44. Constructor vs `@PostConstruct`

Think:

```text
Constructor
    ↓
"Can I construct this object?"

@PostConstruct
    ↓
"Now that dependencies are available,
can I initialize it?"
```

### Constructor

Good for:

- Required dependencies
- Establishing valid object state
- Basic construction

### `@PostConstruct`

Good for:

- Initialization requiring injected dependencies
- Preparing internal state
- Performing initialization logic

---

# Q45. What is `@PreDestroy`?

`@PreDestroy` marks a method that Spring invokes before destroying a managed bean.

Example:

```java
@Service
class PaymentService {

    @PreDestroy
    void cleanup() {
        // cleanup resources
    }
}
```

Conceptually:

```text
Application shutdown
      ↓
Bean destruction
      ↓
@PreDestroy
      ↓
cleanup
```

It is primarily relevant to beans whose destruction is managed by Spring, especially singleton beans.

---

# Q46. What is `InitializingBean`?

`InitializingBean` is a Spring-specific lifecycle interface.

Example:

```java
@Service
class PaymentService
        implements InitializingBean {

    @Override
    public void afterPropertiesSet() {
        // initialization
    }
}
```

Spring calls:

```java
afterPropertiesSet()
```

during bean initialization.

---

# Q47. What is `DisposableBean`?

`DisposableBean` is the destruction counterpart to `InitializingBean`.

Example:

```java
@Service
class PaymentService
        implements DisposableBean {

    @Override
    public void destroy() {
        // cleanup
    }
}
```

Spring calls `destroy()` during bean destruction.

---

# Q48. `@PostConstruct` vs `InitializingBean`

Both are used for initialization.

### `@PostConstruct`

```java
@PostConstruct
void init() {
}
```

Standard lifecycle annotation and generally preferred when you don't want your business class tightly coupled to a Spring-specific lifecycle interface.

### `InitializingBean`

```java
implements InitializingBean
```

Spring-specific interface.

### Interview Answer

> `@PostConstruct` is a standard lifecycle annotation, while `InitializingBean` is a Spring-specific lifecycle callback interface. In application code, `@PostConstruct` is often preferred because it avoids coupling the class directly to a Spring interface.

---

# Q49. `@PreDestroy` vs `DisposableBean`

Same idea for destruction.

```text
@PreDestroy
      vs
DisposableBean.destroy()
```

`@PreDestroy` is generally preferable when you want lifecycle behavior without making the class implement a Spring-specific interface.

---

# Q50. What is a `BeanPostProcessor`?

🔥 **Very important interview question.**

A `BeanPostProcessor` allows Spring to process bean instances before and after their initialization.

Conceptually:

```text
Bean instantiated
      ↓
Dependencies populated
      ↓
postProcessBeforeInitialization()
      ↓
@PostConstruct
      ↓
InitializingBean / init-method
      ↓
postProcessAfterInitialization()
      ↓
Bean ready
```

A `BeanPostProcessor` works with **actual bean instances**.

---

# Q51. Why is `BeanPostProcessor` important?

Spring itself uses this infrastructure heavily.

It provides extension points that allow Spring to:

- Inspect beans
- Modify beans
- Wrap beans
- Create proxies
- Apply framework behavior

This is closely related to how features such as Spring AOP are integrated into managed beans.

### Interview Answer

> `BeanPostProcessor` is an extension mechanism that allows Spring to process bean instances before and after initialization. Spring uses it internally for many framework features, including proxy-related processing.

---

# Q52. Can we create our own `BeanPostProcessor`?

Yes.

Example:

```java
@Component
public class MyBeanPostProcessor
        implements BeanPostProcessor {

    @Override
    public Object postProcessBeforeInitialization(
            Object bean,
            String beanName) {

        return bean;
    }

    @Override
    public Object postProcessAfterInitialization(
            Object bean,
            String beanName) {

        return bean;
    }
}
```

Spring calls these methods for beans during the lifecycle.

---

# Q53. What is `BeanFactoryPostProcessor`?

This is different from `BeanPostProcessor`.

A `BeanFactoryPostProcessor` works with **BeanDefinitions**, not normal bean instances.

Remember:

```text
BeanFactoryPostProcessor
        ↓
BeanDefinition / metadata


BeanPostProcessor
        ↓
Actual bean instance
```

---

# Q54. `BeanPostProcessor` vs `BeanFactoryPostProcessor`

🔥 Very common interview comparison.

| `BeanFactoryPostProcessor` | `BeanPostProcessor` |
|---|---|
| Works with bean definitions/metadata | Works with bean instances |
| Runs before normal bean creation | Processes instances during lifecycle |
| Can modify bean configuration metadata | Can modify/wrap bean objects |
| Definition-level processing | Instance-level processing |

### Easy memory trick

```text
BeanFactoryPostProcessor
        ↓
"What should Spring create?"

BeanPostProcessor
        ↓
"Spring created it — what should I do with it?"
```

---

# Q55. What is a `BeanDefinition`?

Before Spring creates the actual object, it has metadata describing the bean.

Conceptually:

```text
BeanDefinition
    |
    +── Bean name
    +── Bean class
    +── Scope
    +── Lazy initialization
    +── Dependencies
    +── Initialization method
    +── Destruction method
    +── Other configuration
```

Spring uses this information to create and manage the bean.

Example:

```java
@Service
class PaymentService {
}
```

Spring discovers this class and creates bean metadata describing how it should be managed.

---

# Q56. Why is `BeanDefinition` important?

Because Spring doesn't simply maintain a list of already-created Java objects.

It maintains metadata describing how beans should be created and managed.

This allows Spring to control:

```text
Creation
Dependency injection
Scope
Lifecycle
Initialization
Destruction
Lazy loading
```

This is also why `BeanFactoryPostProcessor` can work with bean definitions before the actual bean instances are created.

---

# Q57. What happens if a Bean throws an exception during initialization?

Suppose:

```java
@Service
class PaymentService {

    @PostConstruct
    void init() {

        throw new RuntimeException(
            "Initialization failed"
        );
    }
}
```

Spring cannot successfully initialize the bean.

If that bean is required during startup, ApplicationContext initialization can fail.

Conceptually:

```text
Application startup
      ↓
Create PaymentService
      ↓
@PostConstruct
      ↓
Exception
      ↓
Bean creation failure
      ↓
ApplicationContext startup failure
```

### Production Point

Initialization logic should therefore be used carefully.

Don't put unnecessary long-running or failure-prone operations into every singleton's startup lifecycle unless the application genuinely needs them before becoming ready.

---

# Q58. What happens when the ApplicationContext is closed?

For managed singleton beans, Spring executes destruction callbacks.

Conceptually:

```text
ApplicationContext.close()
          ↓
Destroy singleton beans
          ↓
@PreDestroy
          ↓
DisposableBean
          ↓
destroy-method
```

This is useful for cleanup such as:

- Closing resources
- Stopping background workers
- Releasing connections
- Cleaning up application-managed resources

---

# Q59. Does `@PreDestroy` always run?

It runs when Spring gets an opportunity to properly destroy the managed bean as part of the ApplicationContext lifecycle.

You should not treat it as a guarantee for every possible JVM termination scenario.

For example, an abrupt process termination is different from a normal Spring context shutdown.

### Interview Answer

> `@PreDestroy` is part of Spring's managed bean destruction lifecycle. It is invoked during normal context shutdown, but it should not be treated as a guarantee for abrupt JVM/process termination.

---

# Q60. Can a prototype bean use `@PreDestroy`?

A prototype bean can define destruction callbacks, but Spring does not manage the complete destruction lifecycle of prototype instances after handing them to the caller in the same way it does for singleton beans.

Therefore, relying on Spring to automatically perform prototype cleanup is incorrect.

---

# Practical EPAM Scenario 1 — Two Payment Implementations

You have:

```java
@Service
class UpiPaymentService
        implements PaymentService {
}
```

and:

```java
@Service
class CardPaymentService
        implements PaymentService {
}
```

Then:

```java
OrderService(PaymentService paymentService)
```

fails.

### Why?

Two beans match:

```text
PaymentService
   ├── UpiPaymentService
   └── CardPaymentService
```

### Solutions

Specific implementation:

```java
@Qualifier("upiPaymentService")
```

Default implementation:

```java
@Primary
```

All implementations:

```java
List<PaymentService>
```

Dynamic selection:

```java
Map<String, PaymentService>
```

---

# Practical EPAM Scenario 2 — Singleton + Prototype

```java
@Scope("prototype")
@Component
class PaymentContext {
}
```

```java
@Service
class PaymentService {

    private final PaymentContext context;

    PaymentService(PaymentContext context) {
        this.context = context;
    }
}
```

### Interviewer asks:

"Will I get a new `PaymentContext` for every call to `PaymentService.process()`?"

### Answer:

**No.**

The singleton `PaymentService` receives a prototype instance when the singleton is created.

```text
PaymentService created
       ↓
PaymentContext #1
       ↓
stored inside singleton
       ↓
process()
       ↓
same #1
```

For a fresh instance:

```java
ObjectProvider<PaymentContext>
```

should be used.

---

# Practical EPAM Scenario 3 — Circular Dependency

Suppose:

```text
PaymentService → FraudService
FraudService → PaymentService
```

### Bad solution

Immediately add:

```java
@Lazy
```

and move on.

### Better answer

First investigate the design.

Maybe:

```text
PaymentService
      ↓
FraudService
```

is enough, and `FraudService` shouldn't call `PaymentService`.

Or common logic could move into:

```text
PaymentRulesService
```

so:

```text
PaymentService → PaymentRulesService
FraudService   → PaymentRulesService
```

The goal is to remove the cycle.

---

# Practical EPAM Scenario 4 — Why `@PostConstruct`?

Suppose:

```java
@Service
class PaymentService {

    private final ConfigService config;

    PaymentService(ConfigService config) {
        this.config = config;
    }

    @PostConstruct
    void init() {
        // load configuration
    }
}
```

Interviewer asks:

> Why not do this in the constructor?

Answer:

> The constructor should primarily establish the object's required dependencies and valid initial state. `@PostConstruct` is useful for initialization logic that should run after dependency injection has completed.

---

# Practical EPAM Scenario 5 — BeanPostProcessor

Interviewer asks:

> "What is the practical purpose of a BeanPostProcessor?"

Answer:

> A `BeanPostProcessor` provides lifecycle extension points around bean initialization. It can inspect or wrap bean instances before and after initialization. Spring uses this infrastructure internally for framework behavior such as proxy-related processing.

Remember:

```text
BeanPostProcessor
       ↓
Bean instance
```

not:

```text
BeanDefinition
```

---

# Practical EPAM Scenario 6 — BeanFactoryPostProcessor

Interviewer asks:

> "What is the difference between BeanFactoryPostProcessor and BeanPostProcessor?"

Answer:

> `BeanFactoryPostProcessor` works with bean definitions before normal bean instances are created, while `BeanPostProcessor` works with actual bean instances during their lifecycle.

Easy memory:

```text
BFPP → Definition
BPP  → Instance
```

---

# Rapid-Fire Questions

## Can an interface itself be a Spring Bean?

Not directly.

Spring needs a concrete object to instantiate.

An implementation must exist.

---

## Can an abstract class be a Spring Bean?

Spring cannot instantiate the abstract class itself.

A concrete implementation is required.

---

## Can we have two beans of the same class?

Yes.

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

These are two different bean definitions and normally two different instances.

---

## Can `@Autowired` be used on a private field?

Yes.

For example:

```java
@Autowired
private PaymentService paymentService;
```

Spring can perform field injection through reflection.

However, constructor injection is generally preferred because dependencies become explicit and can be stored in `final` fields.

---

## Can Spring inject a `final` field using field injection?

A normal `final` field cannot be assigned through ordinary field injection after construction.

That's one reason constructor injection works well:

```java
private final PaymentService paymentService;

public OrderService(
        PaymentService paymentService) {

    this.paymentService = paymentService;
}
```

The dependency is supplied through the constructor and the field can remain immutable.

---

## What happens if no bean is found?

Spring cannot satisfy the dependency and application context creation typically fails with an unsatisfied dependency / `NoSuchBeanDefinitionException`.

---

## What happens if two beans match?

Spring reports an ambiguity, typically resulting in:

```text
NoUniqueBeanDefinitionException
```

unless the candidates are resolved using mechanisms such as:

```text
@Qualifier
@Primary
```

or collection injection.

---

## Are Spring singleton beans thread-safe?

No.

Singleton means:

```text
one shared instance per ApplicationContext
```

It does not mean:

```text
thread-safe
```

Avoid mutable request-specific state in singleton services.

---

## Is Spring creating one singleton object globally for the entire JVM?

No.

Spring's singleton means:

> **One bean instance per ApplicationContext.**

Different ApplicationContexts can have separate instances.

---

## Does Spring always eagerly create singleton beans?

Normally singleton beans are eagerly initialized during ApplicationContext startup.

`@Lazy` can defer initialization.

---

## Does Spring completely manage prototype destruction?

No.

Spring manages creation and initialization of prototype instances, but does not manage their complete destruction lifecycle after handing them to the caller in the same way as singleton beans.

---

# Final Spring Core Mental Model

You should now be able to visualize Spring Core like this:

```text
                    Spring IoC Container
                           |
                    BeanDefinition
                           |
             +-------------+-------------+
             |                           |
       Component Scan                  @Bean
             |                           |
             +-------------+-------------+
                           |
                    Bean Instantiation
                           |
                    Dependency Injection
                           |
              +------------+------------+
              |                         |
        @Autowired                 Constructor
              |
      +-------+--------+
      |                |
 @Qualifier        @Primary
      |
      +----------------+
      |
 List / Set / Map
      |
      ↓
 Bean Lifecycle
      |
      +── @PostConstruct
      |
      +── InitializingBean
      |
      +── BeanPostProcessor
      |
      ↓
   Bean Ready
      |
      ↓
 Application Running
      |
      ↓
 Context Shutdown
      |
      +── @PreDestroy
      |
      +── DisposableBean
      |
      ↓
 Cleanup
```

And the two most important processor distinctions:

```text
BeanFactoryPostProcessor
        ↓
BeanDefinition / metadata
        ↓
"What should Spring create?"


BeanPostProcessor
        ↓
Bean instance
        ↓
"How should Spring process this bean?"
```

---

# Spring Core — Part C Complete

```text
@Autowired
   ↓
Dependency Resolution
   ↓
@Qualifier
   ↓
@Primary
   ↓
List / Set / Map Injection
   ↓
Optional Dependencies
   ↓
Circular Dependency
   ↓
@Lazy
   ↓
Bean Lifecycle
   ↓
@PostConstruct
   ↓
InitializingBean
   ↓
BeanPostProcessor
   ↓
BeanFactoryPostProcessor
   ↓
BeanDefinition
   ↓
@PreDestroy
   ↓
DisposableBean
   ↓
ApplicationContext Shutdown
```

**Spring Core is now complete.**

The next major section is **Spring Boot**, where the focus moves from the underlying Spring container to how Spring Boot actually starts and configures a production application:

```text
@SpringBootApplication
Auto-configuration
Application startup
Starters
Embedded Tomcat
application.properties / YAML
Profiles
@ConfigurationProperties
Actuator
Health checks
Logging
Exception handling
Validation
External configuration
```