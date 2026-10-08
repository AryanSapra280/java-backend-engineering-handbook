
# Spring Core — Part A: IoC / Dependency Injection

For every topic, prepare in this order:

```text
Concept
   ↓
Interview-ready answer
   ↓
Example
   ↓
Interview follow-up
   ↓
Practical / implementation question
   ↓
Trap / deeper question
```

---

# Q1. What is Spring?

## Interview Answer

**Spring is a Java framework used to build enterprise applications. Its core feature is IoC/Dependency Injection, which allows Spring to manage object creation and dependencies instead of the application creating and wiring objects manually.**

Spring also provides modules/features for:

- Web applications
- Data access
- Security
- Transactions
- AOP
- Messaging
- REST APIs
- Integration

The key idea is:

> **Spring separates object creation/configuration from business logic.**

## Without Spring

```java
class OrderService {

    private PaymentService paymentService =
            new PaymentService();

}
```

Here `OrderService` is responsible for creating its dependency.

This creates tight coupling.

## With Spring

```java
@Service
class OrderService {

    private final PaymentService paymentService;

    public OrderService(PaymentService paymentService) {
        this.paymentService = paymentService;
    }
}
```

Spring creates `PaymentService` and provides it to `OrderService`.

## Interview Follow-up

### Q: What is the biggest advantage of Spring?

The major advantage is **loose coupling and better testability**.

The class depends on an abstraction/dependency rather than controlling how that dependency is created.

For example:

```java
class OrderService {

    private final PaymentService paymentService;

    OrderService(PaymentService paymentService) {
        this.paymentService = paymentService;
    }
}
```

The service doesn't care whether Spring created:

```text
UpiPaymentService
CardPaymentService
MockPaymentService
```

It only depends on the required contract.

---

# Q2. Why do we need Spring?

Consider:

```java
class OrderService {

    PaymentService paymentService =
            new PaymentService();

    NotificationService notificationService =
            new NotificationService();

    OrderRepository repository =
            new OrderRepository();
}
```

Now `OrderService` is tightly coupled to concrete implementations.

Suppose tomorrow:

```text
PaymentService
      ↓
UpiPaymentService
```

The business class may need to change.

Spring moves object creation and wiring outside the business class.

```text
              Spring Container
                    |
        +-----------+-----------+
        |           |           |
 PaymentService Repository Notification
        |
        +------ injected ------+
                    |
               OrderService
```

## Interview Answer

> **Spring helps separate object creation and dependency management from business logic, which improves loose coupling, testability, maintainability and flexibility.**

---

# Q3. What is IoC?

**IoC = Inversion of Control.**

Normally your application controls object creation:

```java
PaymentService payment =
        new PaymentService();
```

Your code decides:

> "I need this object, so I will create it."

With IoC, control is transferred to the Spring container.

```text
Spring Container
      ↓
Creates objects
      ↓
Manages objects
      ↓
Resolves dependencies
      ↓
Injects dependencies
      ↓
Manages lifecycle
```

## Interview Answer

> **IoC means the control of object creation, dependency management and lifecycle is transferred from application code to the framework/container.**

---

## Interview Trap: IoC vs DI

IoC and DI are related but not exactly the same.

```text
IoC
 ↓
Broad design principle
 ↓
Control is inverted
 ↓
DI
 ↓
Dependencies are supplied externally
```

### Interview Answer

> **IoC is the broader principle of transferring control to the framework, while Dependency Injection is one mechanism used to achieve that inversion of control.**

---

# Q4. What is Dependency Injection?

## Interview Answer

> **Dependency Injection is a mechanism where an object's dependencies are provided to it from outside instead of the object creating those dependencies itself.**

Example:

```java
class OrderService {

    private final PaymentService paymentService;

    public OrderService(PaymentService paymentService) {
        this.paymentService = paymentService;
    }
}
```

`OrderService` depends on `PaymentService`.

Instead of:

```java
new PaymentService();
```

inside `OrderService`, the dependency is provided externally.

That is Dependency Injection.

---

# Q5. IoC vs DI

| IoC | DI |
|---|---|
| Broad design principle | Specific mechanism/technique |
| Control is transferred to framework/container | Dependencies are supplied externally |
| Describes inversion of control | Describes how dependencies are provided |
| DI is one way to achieve IoC | Spring heavily uses DI |

### Interview Answer

> **IoC is the principle of transferring control, while Dependency Injection is one mechanism through which that control inversion is achieved.**

---

# Q6. What are the types of Dependency Injection?

There are three commonly discussed types:

1. Constructor Injection
2. Setter Injection
3. Field Injection

---

## 1. Constructor Injection

```java
@Service
class OrderService {

    private final PaymentService paymentService;

    OrderService(PaymentService paymentService) {
        this.paymentService = paymentService;
    }
}
```

Dependencies are provided through the constructor.

---

## 2. Setter Injection

```java
@Service
class OrderService {

    private PaymentService paymentService;

    @Autowired
    public void setPaymentService(
            PaymentService paymentService) {

        this.paymentService = paymentService;
    }
}
```

The dependency is provided through a setter.

Useful when the dependency is optional or can be changed after object creation.

---

## 3. Field Injection

```java
@Service
class OrderService {

    @Autowired
    private PaymentService paymentService;
}
```

Spring injects directly into the field.

Although supported, field injection is generally discouraged in modern applications.

---

# Q7. Why is Constructor Injection Preferred?

This is an important interview question.

## 1. Dependencies can be `final`

```java
private final PaymentService paymentService;
```

The dependency cannot accidentally be replaced after construction.

---

## 2. Dependencies are explicit

```java
OrderService(
    PaymentService paymentService,
    OrderRepository repository,
    NotificationService notificationService
)
```

Anyone looking at the constructor can immediately understand what the class requires.

---

## 3. Better testability

You can create the class without starting Spring.

```java
PaymentService paymentService =
        mock(PaymentService.class);

OrderService service =
        new OrderService(paymentService);
```

No Spring container is required.

---

## 4. Prevents partially initialized objects

Required dependencies must be provided when the object is constructed.

---

## 5. Circular dependencies become visible

Suppose:

```text
A → B
B → A
```

Constructor injection exposes this dependency cycle during startup.

---

## 6. Supports immutability

Required dependencies can be stored in `final` fields.

---

## Practical Interview Question

### Q: How would you unit-test a constructor-injected service without starting Spring?

Use normal Java object construction and mock the dependencies.

```java
@ExtendWith(MockitoExtension.class)
class OrderServiceTest {

    @Mock
    private PaymentService paymentService;

    @InjectMocks
    private OrderService orderService;
}
```

Or manually:

```java
PaymentService paymentService =
        mock(PaymentService.class);

OrderService orderService =
        new OrderService(paymentService);
```

### Interview Point

> Constructor injection makes unit testing easier because the class does not depend on the Spring container to receive its dependencies.

---

# Q8. What is `@Autowired`?

`@Autowired` tells Spring to resolve and inject a suitable bean.

Example:

```java
@Service
class OrderService {

    private final PaymentService paymentService;

    @Autowired
    public OrderService(
            PaymentService paymentService) {

        this.paymentService = paymentService;
    }
}
```

---

## Important Modern Spring Detail

If a class has only **one constructor**, `@Autowired` is generally not required.

```java
@Service
class OrderService {

    private final PaymentService paymentService;

    public OrderService(
            PaymentService paymentService) {

        this.paymentService = paymentService;
    }
}
```

Spring can use the single constructor automatically.

---

# Q9. How does Spring resolve `@Autowired`?

Suppose:

```java
public OrderService(
        PaymentService paymentService) {
}
```

Spring has to determine which bean should be injected.

Conceptually:

```text
Dependency requested
       ↓
Find candidate beans
       ↓
Match by type
       ↓
Any ambiguity?
   ↙           ↘
 No            Yes
 ↓              ↓
Inject       @Qualifier /
             @Primary /
             other resolution rules
```

For a single dependency, Spring primarily resolves candidates by type.

---

## Scenario 1 — One Matching Bean

```java
@Service
class UpiPaymentService
        implements PaymentService {
}
```

Then:

```java
OrderService(PaymentService paymentService)
```

has one matching candidate.

Spring injects it.

---

## Scenario 2 — No Matching Bean

If no suitable bean exists, Spring cannot satisfy the required dependency and application startup generally fails with a `NoSuchBeanDefinitionException` or related dependency-resolution error.

---

## Scenario 3 — Multiple Matching Beans

Suppose:

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

Now:

```java
OrderService(PaymentService paymentService)
```

has multiple candidates.

Spring cannot determine which one to use unless the ambiguity is resolved.

This typically results in:

```text
NoUniqueBeanDefinitionException
```

---

# Q10. What happens if multiple beans of the same type exist?

Suppose:

```java
interface PaymentService {
}
```

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

Now:

```java
public OrderService(
        PaymentService paymentService) {

    this.paymentService = paymentService;
}
```

Spring sees:

```text
PaymentService
      |
  +---+---+
  |       |
 UPI     CARD
```

There are multiple candidates.

Spring needs additional information.

---

# Q11. How does `@Qualifier` solve multiple-bean ambiguity?

Use:

```java
public OrderService(
        @Qualifier("upiPaymentService")
        PaymentService paymentService) {

    this.paymentService = paymentService;
}
```

Now Spring specifically selects the bean named:

```text
upiPaymentService
```

---

## Interview Answer

> **`@Qualifier` tells Spring which specific bean should be selected when multiple candidates exist.**

---

## Practical Question

### Q: You have three `PaymentService` implementations. How would you inject only the Card implementation?

```java
public OrderService(
        @Qualifier("cardPaymentService")
        PaymentService paymentService) {

    this.paymentService = paymentService;
}
```

---

# Q12. How does `@Primary` solve multiple-bean ambiguity?

Mark one implementation as the default:

```java
@Primary
@Service
class UpiPaymentService
        implements PaymentService {
}
```

Now when Spring sees:

```java
PaymentService paymentService
```

and multiple candidates exist, `UpiPaymentService` becomes the preferred candidate.

---

# Q13. `@Qualifier` vs `@Primary`

### `@Primary`

Means:

> "Use this bean as the default candidate when multiple candidates exist."

Example:

```java
@Primary
@Service
class UpiPaymentService
        implements PaymentService {
}
```

### `@Qualifier`

Means:

> "I specifically want this bean."

Example:

```java
@Qualifier("cardPaymentService")
PaymentService paymentService
```

### Which is more explicit?

`@Qualifier`.

### Practical rule

Use `@Primary` when one implementation is the **general/default choice**.

Use `@Qualifier` when a specific consumer needs a **specific implementation**.

---

## Important Follow-up

### Q: What happens if a `@Primary` bean and a `@Qualifier` are both involved?

The explicit qualifier can select the specifically qualified bean.

For example:

```java
@Primary
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

If we write:

```java
public OrderService(
        @Qualifier("cardPaymentService")
        PaymentService paymentService) {
}
```

the qualifier selects `CardPaymentService`, even though UPI is marked `@Primary`.

### Interview takeaway

> **`@Primary` provides a default preference; `@Qualifier` provides explicit selection.**

---

# Q14. Can Spring inject `List<PaymentService>`?

Yes.

Suppose:

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

```java
@Service
class WalletPaymentService
        implements PaymentService {
}
```

We can write:

```java
@Service
class PaymentProcessor {

    private final List<PaymentService> paymentServices;

    PaymentProcessor(
            List<PaymentService> paymentServices) {

        this.paymentServices = paymentServices;
    }
}
```

Spring injects all beans that match `PaymentService`.

Conceptually:

```text
List<PaymentService>

[ UPI, CARD, WALLET ]
```

---

## Interview Follow-up

### Q: How does Spring know which beans belong in the List?

Spring examines the generic type of the injection point:

```java
List<PaymentService>
```

It looks for all Spring beans that are assignable to `PaymentService` and injects those matching beans into the collection.

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

Both match `PaymentService`, so both are included.

### Important Point

The collection injection is based on the **element type**, not the variable name.

---

# Q15. Can Spring inject `Set<PaymentService>`?

Yes.

```java
@Service
class PaymentProcessor {

    private final Set<PaymentService> paymentServices;

    PaymentProcessor(
            Set<PaymentService> paymentServices) {

        this.paymentServices = paymentServices;
    }
}
```

Spring injects all matching `PaymentService` beans into the Set.

### When is Set useful?

Use a `Set` when you want collection semantics where duplicate elements are not meaningful and you don't need List-specific indexing.

Use `List` when ordering or positional access matters.

---

# Q16. Can Spring inject `Map<String, PaymentService>`?

Yes.

```java
@Service
class PaymentProcessor {

    private final Map<String, PaymentService> paymentServices;

    PaymentProcessor(
            Map<String, PaymentService> paymentServices) {

        this.paymentServices = paymentServices;
    }
}
```

Spring can inject all matching `PaymentService` beans into the Map.

---

# Q17. What becomes the key in `Map<String, PaymentService>`?

The Map key is normally the **Spring bean name**.

For:

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

the default bean names are typically:

```text
upiPaymentService
cardPaymentService
```

So the Map is conceptually:

```text
{
    "upiPaymentService"  → UpiPaymentService,
    "cardPaymentService" → CardPaymentService
}
```

Then:

```java
PaymentService service =
        paymentServices.get("upiPaymentService");
```

can retrieve the UPI strategy.

---

# Q18. When is List/Map injection preferable to `@Qualifier`?

Use `@Qualifier` when the consumer needs **one known implementation**.

Example:

```java
OrderService(
    @Qualifier("upiPaymentService")
    PaymentService paymentService
)
```

Use `List` when the application needs to **iterate over all implementations**.

Example:

```java
List<PaymentService>
```

Use `Map` when the application needs to **select an implementation dynamically** using a key.

Example:

```java
Map<String, PaymentService>
```

### Practical comparison

```text
One specific implementation
        ↓
@Qualifier

All implementations
        ↓
List / Set

Dynamic selection
        ↓
Map
```

---

# Q19. Practical Design — Strategy Pattern with Spring DI

### Problem

Design a payment system supporting:

```text
UPI
CARD
NET_BANKING
```

Each payment type has its own implementation.

The design should allow adding another payment type without creating a huge `if-else` chain.

---

## Step 1 — Create interface

```java
public interface PaymentStrategy {

    void pay(double amount);
}
```

---

## Step 2 — Implement strategies

```java
@Service
class UpiPaymentStrategy
        implements PaymentStrategy {

    @Override
    public void pay(double amount) {
        System.out.println("Processing UPI");
    }
}
```

```java
@Service
class CardPaymentStrategy
        implements PaymentStrategy {

    @Override
    public void pay(double amount) {
        System.out.println("Processing Card");
    }
}
```

```java
@Service
class NetBankingPaymentStrategy
        implements PaymentStrategy {

    @Override
    public void pay(double amount) {
        System.out.println("Processing Net Banking");
    }
}
```

---

## Step 3 — Inject all strategies

```java
@Service
class PaymentProcessor {

    private final Map<String, PaymentStrategy> strategies;

    PaymentProcessor(
            Map<String, PaymentStrategy> strategies) {

        this.strategies = strategies;
    }
}
```

---

## Step 4 — Select strategy

Because the default Map keys are bean names, one approach is:

```java
public void process(
        String paymentType,
        double amount) {

    PaymentStrategy strategy =
            strategies.get(
                paymentType + "PaymentStrategy"
            );

    if (strategy == null) {
        throw new IllegalArgumentException(
                "Unsupported payment type");
    }

    strategy.pay(amount);
}
```

In production, a cleaner approach is to explicitly associate each strategy with a business key rather than relying blindly on generated bean names.

---

## EPAM Follow-up

### Q: Why is this better than a large switch?

Because the business service depends on the abstraction:

```java
PaymentStrategy
```

rather than concrete classes.

Adding:

```text
WalletPaymentStrategy
```

doesn't require modifying a large conditional block in the processor.

This follows the **Open/Closed Principle** more closely.

---

# Q20. Practical Question — How would you select an implementation based on runtime input?

Suppose the API receives:

```json
{
    "paymentType": "UPI",
    "amount": 5000
}
```

We need:

```text
UPI
 ↓
UpiPaymentStrategy

CARD
 ↓
CardPaymentStrategy

NET_BANKING
 ↓
NetBankingPaymentStrategy
```

A practical design is:

```text
Request
  ↓
PaymentController
  ↓
PaymentProcessor
  ↓
Map<String, PaymentStrategy>
  ↓
Correct Strategy
```

The Map can provide O(1)-average lookup by key.

The important architectural idea is:

> **The controller should not contain payment-specific implementation logic. The strategy selection belongs in an appropriate service/strategy layer.**

---

# Q21. Practical Question — What if I add a fourth implementation tomorrow?

Suppose we add:

```java
@Service
class WalletPaymentStrategy
        implements PaymentStrategy {
}
```

Spring automatically discovers the new bean through component scanning.

If we inject:

```java
List<PaymentStrategy>
```

or:

```java
Map<String, PaymentStrategy>
```

the new implementation becomes part of the injected collection.

### Interview Answer

> **Spring's collection injection allows new implementations to participate automatically as Spring beans, without changing the dependency-injection wiring code.**

The business logic may still need a business-key mapping depending on how strategy selection is designed.

---

# Q22. Practical Question — How would you unit-test a service using DI?

Suppose:

```java
@Service
class OrderService {

    private final PaymentService paymentService;

    OrderService(PaymentService paymentService) {
        this.paymentService = paymentService;
    }

    public void placeOrder() {
        paymentService.pay(100);
    }
}
```

We can test without Spring:

```java
@ExtendWith(MockitoExtension.class)
class OrderServiceTest {

    @Mock
    PaymentService paymentService;

    @InjectMocks
    OrderService orderService;

    @Test
    void shouldPlaceOrder() {

        orderService.placeOrder();

        verify(paymentService)
                .pay(100);
    }
}
```

### Why is this easy?

Because constructor injection makes the dependency explicit.

The test can provide a mock directly.

---

# Q23. What is Circular Dependency?

Suppose:

```text
OrderService
     ↓
PaymentService
     ↓
OrderService
```

This means:

```text
A → B
B → A
```

Example:

```java
@Service
class OrderService {

    private final PaymentService paymentService;

    OrderService(PaymentService paymentService) {
        this.paymentService = paymentService;
    }
}
```

```java
@Service
class PaymentService {

    private final OrderService orderService;

    PaymentService(OrderService orderService) {
        this.orderService = orderService;
    }
}
```

Neither object can be constructed first.

---

# Q24. Why does Constructor Injection expose Circular Dependency?

Spring tries:

```text
Create OrderService
      ↓
Needs PaymentService
      ↓
Create PaymentService
      ↓
Needs OrderService
      ↓
OrderService isn't completely created
      ↓
Circular dependency
```

Constructor injection requires the dependency to exist before the object can be constructed.

Therefore the cycle becomes immediately visible.

### Interview Answer

> **Constructor injection exposes circular dependencies because both objects require each other before either can be fully constructed.**

---

# Q25. How would you fix Circular Dependency?

### Preferred solution

**Redesign the dependency graph.**

For example:

```text
OrderService → PaymentService
PaymentService → OrderService
```

may indicate that responsibilities are mixed.

Extract common functionality:

```text
OrderService ──→ PaymentService

OrderService ──→ PaymentHistoryService
PaymentService ──→ PaymentHistoryService
```

or introduce another appropriate abstraction.

### Other technical options

Depending on the situation:

- `@Lazy`
- Setter injection
- Refactoring common logic into another service

But these should not automatically be treated as the best architectural solution.

### Strong interview answer

> **I would first remove the circular dependency by redesigning the responsibilities. `@Lazy` or setter injection can technically break certain cycles, but hiding a design problem is generally less desirable than removing the cycle.**

---

# Q26. What does `@Lazy` do?

Normally singleton beans are eagerly initialized during ApplicationContext startup.

With:

```java
@Lazy
@Service
class PaymentService {
}
```

Spring delays initialization until the bean is actually needed.

Conceptually:

```text
Application starts
      ↓
PaymentService not created yet
      ↓
First required access
      ↓
PaymentService created
```

---

## Practical Uses

`@Lazy` can be useful for:

- Expensive initialization
- Delaying unnecessary bean creation
- Breaking certain dependency cycles
- Reducing startup work

### Important Trap

Do not say:

> "`@Lazy` is the correct way to fix every circular dependency."

Instead:

> **Use `@Lazy` when appropriate, but first check whether the dependency graph itself should be redesigned.**

---

# Q27. Can Spring inject an interface?

Yes — but Spring does not instantiate the interface itself.

For example:

```java
interface PaymentService {
}
```

and:

```java
@Service
class UpiPaymentService
        implements PaymentService {
}
```

Spring creates:

```text
UpiPaymentService
```

and can inject it wherever:

```java
PaymentService
```

is required.

### Important distinction

Spring resolves the dependency using the **interface type**, but the actual object is a concrete implementation.

---

# Q28. Can Spring instantiate an interface?

No.

An interface cannot be directly instantiated:

```java
new PaymentService(); // impossible
```

There must be a concrete implementation or some other supported bean-producing mechanism.

For example:

```java
@Service
class UpiPaymentService
        implements PaymentService {
}
```

Spring can create the concrete class and inject it through the interface.

---

# Q29. Can we have multiple beans of the same class?

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

There are now two distinct Spring beans.

Conceptually:

```text
payment1 → PaymentService instance #1

payment2 → PaymentService instance #2
```

They are different bean instances even though they have the same Java class.

---

# Q30. How would you inject one of two beans of the same class?

Use a qualifier/name.

For example:

```java
@Bean
PaymentService upiPaymentService() {
    return new PaymentService();
}

@Bean
PaymentService cardPaymentService() {
    return new PaymentService();
}
```

Then:

```java
public OrderService(
        @Qualifier("upiPaymentService")
        PaymentService paymentService) {
}
```

The qualifier selects the desired bean.

---

# Q31. What happens if no bean is found?

Suppose:

```java
@Service
class OrderService {

    OrderService(PaymentService paymentService) {
    }
}
```

but there is no suitable `PaymentService` bean.

Spring cannot satisfy the required dependency.

Application startup generally fails with a dependency-resolution error such as:

```text
NoSuchBeanDefinitionException
```

or a related bean-creation exception wrapping the underlying problem.

---

## Practical Debugging Approach

If this happens in a real application, check:

```text
1. Is the implementation annotated with @Component/@Service?
             ↓
2. Is it inside the component-scan package?
             ↓
3. Is the correct profile/configuration active?
             ↓
4. Is the bean conditional?
             ↓
5. Is bean creation itself failing?
             ↓
6. Is there a configuration/startup error?
```

---

# Q32. What happens if two beans match?

Suppose:

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

and:

```java
OrderService(PaymentService paymentService)
```

Spring has multiple candidates.

If there is no valid resolution mechanism, startup fails with:

```text
NoUniqueBeanDefinitionException
```

Fix using:

```text
@Qualifier
```

or:

```text
@Primary
```

or redesign the injection point to use:

```text
List<PaymentService>
```

or:

```text
Map<String, PaymentService>
```

depending on the requirement.

---

# Q33. What happens if you add a new implementation and suddenly the application fails?

Suppose yesterday:

```text
PaymentService
     ↓
UPI
```

Today you add:

```text
PaymentService
     ↓
UPI
CARD
```

and an existing class has:

```java
OrderService(PaymentService paymentService)
```

The previously unique dependency is now ambiguous.

Spring may fail at startup with:

```text
NoUniqueBeanDefinitionException
```

### Production debugging approach

Check:

```text
New implementation added?
        ↓
Is it a Spring bean?
        ↓
Does it implement the same interface?
        ↓
Are existing injection points ambiguous?
        ↓
Use @Primary / @Qualifier / collection injection
```

This is a very practical Spring debugging scenario.

---

# Q34. Practical Question — Design a Payment Service

### Requirement

Create:

```text
PaymentController
        ↓
PaymentProcessor
        ↓
PaymentStrategy
      /   |    \
    UPI CARD NET_BANKING
```

Requirements:

- Use Spring DI.
- Use constructor injection.
- Use an interface.
- Have multiple implementations.
- Avoid a large `if-else` or `switch`.
- Make it easy to add a new payment strategy.
- Keep it unit-testable.

### Design

```java
public interface PaymentStrategy {

    void pay(double amount);
}
```

Implement:

```java
@Service
class UpiPaymentStrategy
        implements PaymentStrategy {

    @Override
    public void pay(double amount) {
        // UPI logic
    }
}
```

```java
@Service
class CardPaymentStrategy
        implements PaymentStrategy {

    @Override
    public void pay(double amount) {
        // Card logic
    }
}
```

Then:

```java
@Service
class PaymentProcessor {

    private final List<PaymentStrategy> strategies;

    PaymentProcessor(
            List<PaymentStrategy> strategies) {

        this.strategies = strategies;
    }
}
```

Or use:

```java
Map<String, PaymentStrategy>
```

when dynamic selection is required.

### Why this design?

Because:

```text
Business service
       ↓
Interface
       ↓
Multiple implementations
```

The processor doesn't directly create:

```java
new UpiPaymentStrategy();
new CardPaymentStrategy();
```

Spring controls the dependencies.

---

# Q35. What is the difference between `new` and Spring DI?

Without DI:

```java
PaymentService payment =
        new PaymentService();
```

The class controls object creation.

With DI:

```java
OrderService(
        PaymentService paymentService
)
```

Spring provides the dependency.

### Important distinction

Using `new` is not inherently bad.

The problem is when a business class directly constructs dependencies that should be externally managed/configured.

For example:

```java
class OrderService {

    PaymentService payment =
            new PaymentService();
}
```

creates tight coupling.

---

# Q36. Can `@Autowired` be used on a private field?

Yes.

For example:

```java
@Autowired
private PaymentService paymentService;
```

Spring can perform field injection even when the field is private.

However, private field injection is generally less preferred than constructor injection because:

- Dependencies are hidden.
- Fields cannot naturally be `final`.
- Unit testing becomes less convenient.
- The class can appear constructible without its required dependencies.

### Preferred approach

```java
private final PaymentService paymentService;

OrderService(PaymentService paymentService) {
    this.paymentService = paymentService;
}
```

---

# Q37. Can Spring inject a `final` field?

Spring cannot perform normal field injection into a `final` field because the field must be assigned during construction.

Therefore:

```java
@Autowired
private final PaymentService paymentService;
```

is not the normal approach.

Instead use constructor injection:

```java
private final PaymentService paymentService;

OrderService(PaymentService paymentService) {
    this.paymentService = paymentService;
}
```

This is another reason constructor injection is preferred.

---

# Q38. Practical Question — Are Spring Singleton Beans Thread-Safe?

No.

Spring singleton means:

> **One bean instance per ApplicationContext.**

It does not mean:

> **The object is automatically thread-safe.**

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

Multiple HTTP requests can execute:

```text
Thread 1 ─┐
Thread 2 ─┼──→ same PaymentService instance
Thread 3 ─┘
```

All can access:

```java
count
```

concurrently.

That can create race conditions.

---

## Better design

Keep singleton services stateless:

```java
@Service
class PaymentService {

    public PaymentResponse process(
            PaymentRequest request) {

        int amount = request.getAmount();

        // local variables
        // business logic

        return ...;
    }
}
```

Local variables are thread-confined to the executing method invocation.

### Interview Answer

> **Spring singleton scope and thread safety are different concepts. Spring manages the bean scope, but the developer is responsible for making shared mutable state thread-safe.**

---

# Q39. Is Spring Singleton the same as Singleton Design Pattern?

No.

### Singleton Design Pattern

Generally means:

> Only one instance according to the pattern's implementation within its relevant JVM/classloader context.

### Spring Singleton

Means:

> **One bean instance per Spring ApplicationContext.**

For example:

```text
ApplicationContext A
        ↓
PaymentService instance #1

ApplicationContext B
        ↓
PaymentService instance #2
```

Therefore Spring singleton does not necessarily mean one object for the entire JVM.

---

# Q40. Is Spring creating one object globally for the entire JVM?

No.

Spring singleton is scoped to the **ApplicationContext**.

Multiple ApplicationContexts can have separate instances.

```text
JVM
 |
 +--- ApplicationContext A
 |        |
 |        +--- PaymentService #1
 |
 +--- ApplicationContext B
          |
          +--- PaymentService #2
```

### Interview Answer

> **Spring singleton is container-scoped, not JVM-global.**

---

# Q41. Practical Interview Scenario — Application Startup Failure

### Scenario

You deploy a new version of the application.

The application fails during startup with:

```text
NoUniqueBeanDefinitionException
```

### How would you debug it?

First identify the dependency from the stack trace.

Then check:

```text
Which interface/type?
        ↓
How many beans implement it?
        ↓
Was a new implementation recently added?
        ↓
Is there @Primary?
        ↓
Is there @Qualifier?
        ↓
Should this dependency actually be a List/Map?
```

For example:

```java
OrderService(PaymentService paymentService)
```

may now have:

```text
UPI
CARD
WALLET
```

instead of only:

```text
UPI
```

Then decide whether the requirement is:

```text
One default implementation
        ↓
@Primary

Specific implementation
        ↓
@Qualifier

All implementations
        ↓
List / Set

Dynamic strategy selection
        ↓
Map
```

---

# Q42. Practical Interview Scenario — Service Not Found

### Scenario

You create:

```java
@Service
class PaymentService {
}
```

but another class gets:

```text
NoSuchBeanDefinitionException
```

### What would you check?

#### 1. Component scanning

Is `PaymentService` inside the package scanned by Spring?

#### 2. Correct annotation

Is the class actually annotated:

```java
@Service
```

or:

```java
@Component
```

#### 3. Configuration/profile

Is the bean conditionally enabled or disabled?

#### 4. Bean creation

Did the bean itself fail during initialization?

#### 5. Application context

Is the class being loaded into the same Spring ApplicationContext?

### Interview Answer

> **I would first verify component scanning and bean registration, then check profiles/conditional configuration and finally inspect the startup stack trace to determine whether bean creation itself failed.**

---

# Q43. Practical Interview Scenario — Three Payment Implementations

### Requirement

You have:

```text
PaymentService
      |
 +----+----+
 |    |    |
UPI CARD WALLET
```

The interviewer asks:

> "For one service, I always want UPI. For another service, I always want Card. For a third service, I want all payment implementations. How would you implement it?"

### Answer

For the first:

```java
OrderService(
    @Qualifier("upiPaymentService")
    PaymentService paymentService
)
```

For the second:

```java
RefundService(
    @Qualifier("cardPaymentService")
    PaymentService paymentService
)
```

For the third:

```java
PaymentProcessor(
    List<PaymentService> paymentServices
)
```

or:

```java
PaymentProcessor(
    Map<String, PaymentService> paymentServices
)
```

depending on whether dynamic lookup is required.

---

# Q44. Practical Interview Scenario — New Payment Type

### Requirement

Today:

```text
UPI
CARD
```

Tomorrow:

```text
UPI
CARD
WALLET
```

### Good design

Create:

```java
@Service
class WalletPaymentService
        implements PaymentService {
}
```

If the consuming service uses:

```java
List<PaymentService>
```

or:

```java
Map<String, PaymentService>
```

Spring automatically includes the new bean in the collection.

### Interview Answer

> **I would use interface-based strategies managed by Spring and inject the implementations as a collection. This avoids hard-coding object creation and makes the system easier to extend.**

---

# Q45. Rapid-Fire Interview Questions

## Can Spring inject an interface?

Yes.

Spring injects a concrete implementation that satisfies the interface.

---

## Can Spring inject multiple implementations?

Yes.

Use:

```java
List<PaymentService>
```

```java
Set<PaymentService>
```

or:

```java
Map<String, PaymentService>
```

---

## Can Spring inject a List?

Yes.

It injects all matching beans.

---

## Can Spring inject a Set?

Yes.

It injects all matching beans into the Set.

---

## Can Spring inject a Map?

Yes.

For:

```java
Map<String, PaymentService>
```

the keys are normally Spring bean names.

---

## What happens if no matching bean exists?

A required dependency cannot be resolved and startup generally fails with a `NoSuchBeanDefinitionException` or related dependency-resolution error.

---

## What happens if multiple beans match?

Spring reports ambiguity, typically through:

```text
NoUniqueBeanDefinitionException
```

unless the candidates are resolved using mechanisms such as:

```text
@Primary
@Qualifier
```

or the dependency is changed to a collection.

---

## `@Primary` vs `@Qualifier`?

```text
@Primary
→ default/preferred candidate

@Qualifier
→ explicitly selected candidate
```

---

## Can two beans have the same Java class?

Yes.

Two different bean definitions can create two different instances of the same class.

---

## Can Spring instantiate an interface?

No.

A concrete implementation is required.

---

## Can a singleton bean depend on a prototype bean?

Yes.

But direct injection normally resolves the prototype when the singleton is created, so the singleton does not automatically receive a new prototype instance on every method call.

Use mechanisms such as:

```java
ObjectProvider<PrototypeBean>
```

when a fresh prototype instance is required.

---

## Are Spring singleton beans thread-safe?

No.

Singleton means one shared instance per ApplicationContext, not automatic thread safety.

---

## Why is constructor injection preferred?

Because it provides:

- Explicit dependencies
- `final` fields
- Better testability
- Better immutability
- Earlier detection of circular dependencies
- Prevention of partially initialized required dependencies

---

# EPAM Practical Coding Checklist

Before considering Spring IoC/DI complete, you should be able to write these **without looking at notes**:

### 1. Basic Constructor Injection

```text
OrderService
    ↓
PaymentService
```

Implement it using constructor injection.

---

### 2. Multiple Implementations

```text
PaymentService
    ├── UPI
    └── CARD
```

Resolve ambiguity using:

```text
@Qualifier
```

and:

```text
@Primary
```

---

### 3. Collection Injection

Write:

```java
List<PaymentService>
```

and:

```java
Set<PaymentService>
```

injection.

---

### 4. Map Injection

Write:

```java
Map<String, PaymentService>
```

and explain what the keys represent.

---

### 5. Strategy Pattern

Implement:

```text
PaymentStrategy
    ├── UPI
    ├── CARD
    └── NET_BANKING
```

and select the strategy dynamically.

---

### 6. Unit Testing

Test a constructor-injected service using Mockito without starting Spring.

---

### 7. Circular Dependency

Create a circular dependency and explain why constructor injection exposes it.

Then explain how you would redesign it.

---

### 8. Debugging

Be able to diagnose:

```text
NoSuchBeanDefinitionException
```

and:

```text
NoUniqueBeanDefinitionException
```

from the Spring startup logs.

---

# Final Mental Model

You should be able to explain Spring IoC/DI like this:

```text
                Spring Container
                       |
              Creates/manages beans
                       |
                       ↓
             Resolves dependencies
                       |
          +------------+------------+
          |            |            |
       Service      Repository   Strategy
          |            |            |
          +------------+------------+
                       |
                       ↓
                 Business Code
```

The business class should focus on **business logic**, while Spring manages:

```text
Object creation
      ↓
Dependency resolution
      ↓
Dependency injection
      ↓
Bean lifecycle
      ↓
Bean scope
```

And when multiple implementations exist:

```text
One specific implementation
        ↓
@Qualifier

Default implementation
        ↓
@Primary

All implementations
        ↓
List / Set

Dynamic strategy selection
        ↓
Map
```

### Core Interview Statement

> **Spring IoC/DI removes responsibility for dependency creation from business classes. The container manages bean creation, resolves dependencies, injects them, and manages their lifecycle. Constructor injection is generally preferred because it makes dependencies explicit, supports immutability and testability, and exposes design problems such as circular dependencies early.**

## Part A Complete

```text
Spring
   ↓
IoC
   ↓
DI
   ↓
Types of DI
   ↓
Constructor Injection
   ↓
@Autowired
   ↓
Dependency Resolution
   ↓
Multiple Beans
   ↓
@Qualifier / @Primary
   ↓
List / Set / Map Injection
   ↓
Strategy Pattern
   ↓
Circular Dependency
   ↓
@Lazy
   ↓
Practical DI / Debugging
   ↓
Unit Testing
   ↓
Singleton Thread-Safety
```

**Next: Spring Core — Part B: Spring Beans**

We will cover:

```text
Spring Bean
@Component
@Service
@Repository
@Controller
@RestController
Component Scanning
@Bean
@Configuration
Bean Naming
Bean Scopes
Singleton
Prototype
Request / Session / Application
Singleton + Prototype
Bean lifecycle
Bean creation
```