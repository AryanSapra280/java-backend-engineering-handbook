# Strategy Design Pattern

## 1. Intent

The **Strategy Design Pattern** defines a family of algorithms/behaviors, encapsulates each one separately, and makes them interchangeable at runtime.

> **Separate "what algorithm/behavior to use" from the object that needs to use it.**

## 2. Problem

Suppose a payment system supports UPI, Card, PayPal and Wallet. A naive implementation grows into a large `if-else`/`switch` containing different payment algorithms. As behaviors grow, the service becomes difficult to maintain, test and extend.

## 3. Naive Solution

```java
public class PaymentService {
    public void pay(PaymentType type, double amount) {
        if (type == PaymentType.UPI) {
            // UPI logic
        } else if (type == PaymentType.CARD) {
            // Card logic
        } else if (type == PaymentType.PAYPAL) {
            // PayPal logic
        }
    }
}
```

This is fine for tiny/simple variation. It becomes a design smell when the branches represent substantial, independently changing algorithms.

## 4. What Smell / Problem Appears?

- Growing `if-else`/`switch` logic
- Service knows every concrete behavior
- Multiple responsibilities in one class
- Existing class must change whenever a new behavior is added
- Difficult unit testing
- Increasing dependencies
- Manual object creation can bypass Spring dependency injection

## 5. Core Idea

Define a common abstraction and encapsulate each behavior separately.

```java
public interface PaymentStrategy {
    void pay(double amount);
}
```

```java
public class UPIPaymentStrategy implements PaymentStrategy {
    @Override
    public void pay(double amount) {
        System.out.println("Processing UPI payment");
    }
}
```

```java
public class CardPaymentStrategy implements PaymentStrategy {
    @Override
    public void pay(double amount) {
        System.out.println("Processing Card payment");
    }
}
```

The caller works with `PaymentStrategy`, not concrete implementations.

## 6. Structure

```text
Context
   |
   ↓
Strategy Interface
   |
   +---- Strategy A
   +---- Strategy B
   +---- Strategy C
```

### Strategy
Defines the common contract.

### Concrete Strategy
Implements one specific behavior.

### Context
Uses the Strategy through the abstraction.

```java
public class PaymentService {
    private final PaymentStrategy strategy;

    public PaymentService(PaymentStrategy strategy) {
        this.strategy = strategy;
    }

    public void pay(double amount) {
        strategy.pay(amount);
    }
}
```

## 7. Java Implementation

```java
public interface PaymentStrategy {
    void pay(double amount);
}
```

```java
public class UPIPaymentStrategy implements PaymentStrategy {
    @Override
    public void pay(double amount) {
        System.out.println("Processing UPI payment of " + amount);
    }
}
```

```java
public class CardPaymentStrategy implements PaymentStrategy {
    @Override
    public void pay(double amount) {
        System.out.println("Processing Card payment of " + amount);
    }
}
```

```java
public class PaymentService {
    private final PaymentStrategy strategy;

    public PaymentService(PaymentStrategy strategy) {
        this.strategy = strategy;
    }

    public void pay(double amount) {
        strategy.pay(amount);
    }
}
```

Usage:

```java
PaymentStrategy strategy = new UPIPaymentStrategy();
PaymentService service = new PaymentService(strategy);
service.pay(500);
```

The `PaymentService` does not change when the Strategy changes.

## 8. How to Recognize It

Look for:

1. Multiple ways of performing the same conceptual operation.
2. A growing `if-else`/`switch` based on behavior.
3. Algorithms that can change independently.
4. New implementations being added frequently.
5. A common operation with multiple implementations.

Examples:

```text
Payment → UPI / Card / PayPal
Pricing → Regular / Premium / Corporate
Notification → Email / SMS / Push
File processing → CSV / JSON / XML
Parking → Nearest Spot / Cheapest Spot / First Available
```

## 9. When to Use

Use Strategy when:

- You have interchangeable algorithms.
- The behavior varies independently of the Context.
- Conditional logic is growing because of different algorithms.
- You want each algorithm independently testable.
- New behaviors are expected to be added.
- The Context should depend on an abstraction.

## 10. When NOT to Use

Do not create a Strategy for every `if`.

```java
if (age > 18) {
    // ...
}
```

does not automatically need Strategy.

If there are only two tiny branches and the behavior is trivial, introducing multiple Strategy classes can be over-engineering.

The key question is:

> **Do these branches represent meaningful interchangeable behaviors/algorithms?**

## 11. SOLID Connection

### Single Responsibility Principle

Each Strategy owns one behavior.

```text
UPIPaymentStrategy → UPI behavior
CardPaymentStrategy → Card behavior
```

### Open/Closed Principle

Adding a new Strategy can be done by adding a new implementation rather than changing the Context.

### Dependency Inversion Principle

The Context depends on:

```java
PaymentStrategy
```

rather than:

```java
UPIPaymentStrategy
```

The high-level code depends on an abstraction.

## 12. Composition vs Inheritance

Strategy primarily uses **composition**.

```text
PaymentService
      |
      | HAS-A
      ↓
PaymentStrategy
```

```java
public class PaymentService {
    private final PaymentStrategy strategy;

    public PaymentService(PaymentStrategy strategy) {
        this.strategy = strategy;
    }
}
```

The Context does not become a subclass of each behavior. It is composed with a Strategy.

This makes behavior replaceable.

## 13. Strategy vs State

Both often have a similar class structure, but the intent differs.

### Strategy

> Which algorithm/behavior should I use?

Example:

```text
Payment
 ↓
UPI Strategy / Card Strategy
```

### State

> What is the object's current state, and how should its behavior change because of that state?

Example:

```text
Order
 ↓
PLACED → SHIPPED → DELIVERED
```

Mental model:

```text
STRATEGY = choose HOW to do something
STATE    = current state determines behavior
```

## 14. Strategy vs Factory

### Strategy

Encapsulates interchangeable behavior.

```text
PaymentStrategy
   +-- UPI
   +-- Card
   +-- PayPal
```

### Factory

Usually focuses on object creation/selection.

```text
PaymentFactory
      ↓
create(type)
      ↓
object
```

They can work together:

```text
Request
  ↓
Factory / Registry
  ↓
Strategy
  ↓
execute()
```

In Spring, however, the container can create the Strategy objects, so a Registry may only need to select an already-existing bean.

## 15. Strategy Pattern in Spring Boot

A textbook implementation might use:

```java
if (type == UPI) {
    strategy = new UPIPaymentStrategy();
} else if (type == CARD) {
    strategy = new CardPaymentStrategy();
}
```

In Spring Boot, a better production-oriented approach is often:

```text
Spring creates and manages Strategy beans
              ↓
Registry receives all Strategies
              ↓
Registry builds lookup Map
              ↓
Runtime request selects existing Strategy
              ↓
Strategy executes behavior
```

### 15.1 Make Strategies Spring Beans

```java
@Component
public class UPIPaymentStrategy implements PaymentStrategy {

    @Override
    public PaymentType getType() {
        return PaymentType.UPI;
    }

    @Override
    public void pay(double amount) {
        System.out.println("Processing UPI payment");
    }
}
```

```java
@Component
public class CardPaymentStrategy implements PaymentStrategy {

    @Override
    public PaymentType getType() {
        return PaymentType.CARD;
    }

    @Override
    public void pay(double amount) {
        System.out.println("Processing Card payment");
    }
}
```

## 16. Why Not Just Use if-else-if in Spring?

A naive Spring implementation would still be tightly coupled:

```java
public class PaymentService {
    public void pay(PaymentType type, double amount) {
        if (type == PaymentType.UPI) {
            new UPIPaymentStrategy().pay(amount);
        } else if (type == PaymentType.CARD) {
            new CardPaymentStrategy().pay(amount);
        }
    }
}
```

Problems:

1. `PaymentService` knows every Strategy.
2. `PaymentService` creates Strategy objects.
3. Manual `new` bypasses Spring dependency injection.
4. Adding a Strategy requires changing the service.
5. Strategy dependencies become difficult to wire manually.

For example:

```java
@Component
public class UPIPaymentStrategy implements PaymentStrategy {

    private final UpiGateway gateway;

    public UPIPaymentStrategy(UpiGateway gateway) {
        this.gateway = gateway;
    }
}
```

If the service manually does `new UPIPaymentStrategy(...)`, it is taking over object creation that Spring should manage.

## 17. Let Spring Inject All Strategies

Spring can inject all beans implementing an interface:

```java
@Service
public class PaymentStrategyRegistry {

    private final List<PaymentStrategy> strategies;

    public PaymentStrategyRegistry(
            List<PaymentStrategy> strategies) {
        this.strategies = strategies;
    }
}
```

If Spring has:

```text
UPIPaymentStrategy
CardPaymentStrategy
PayPalPaymentStrategy
```

then Spring can inject:

```text
List<PaymentStrategy>

[
    UPIPaymentStrategy object,
    CardPaymentStrategy object,
    PayPalPaymentStrategy object
]
```

Spring is responsible for creating and wiring the beans. The Registry is responsible for selecting the correct one.

## 18. Build a Strategy Registry

Define a business key:

```java
public enum PaymentType {
    UPI,
    CARD,
    PAYPAL
}
```

Strategy interface:

```java
public interface PaymentStrategy {
    PaymentType getType();
    void pay(double amount);
}
```

UPI:

```java
@Component
public class UPIPaymentStrategy implements PaymentStrategy {

    @Override
    public PaymentType getType() {
        return PaymentType.UPI;
    }

    @Override
    public void pay(double amount) {
        System.out.println("Processing UPI payment");
    }
}
```

Card:

```java
@Component
public class CardPaymentStrategy implements PaymentStrategy {

    @Override
    public PaymentType getType() {
        return PaymentType.CARD;
    }

    @Override
    public void pay(double amount) {
        System.out.println("Processing Card payment");
    }
}
```

## 19. Convert the List into a Map

```java
@Service
public class PaymentStrategyRegistry {

    private final Map<PaymentType, PaymentStrategy>
            paymentStrategyMap;

    public PaymentStrategyRegistry(
            List<PaymentStrategy> strategies) {

        this.paymentStrategyMap =
                strategies.stream()
                        .collect(Collectors.toMap(
                                PaymentStrategy::getType,
                                Function.identity()
                        ));
    }

    public PaymentStrategy getStrategy(
            PaymentType type) {

        PaymentStrategy strategy =
                paymentStrategyMap.get(type);

        if (strategy == null) {
            throw new IllegalArgumentException(
                    "Unsupported payment type: " + type);
        }

        return strategy;
    }
}
```

## 20. Understanding `toMap()`

This:

```java
strategies.stream()
    .collect(Collectors.toMap(
        PaymentStrategy::getType,
        Function.identity()
    ));
```

is essentially the same as:

```java
Map<PaymentType, PaymentStrategy> map =
        new HashMap<>();

for (PaymentStrategy strategy : strategies) {

    PaymentType key = strategy.getType();
    PaymentStrategy value = strategy;

    map.put(key, value);
}
```

### `PaymentStrategy::getType`

Equivalent to:

```java
strategy -> strategy.getType()
```

It determines the Map key.

### `Function.identity()`

Equivalent to:

```java
strategy -> strategy
```

It returns the Strategy object itself, so it becomes the Map value.

Therefore:

```text
UPIPaymentStrategy
       ↓ getType()
      UPI
```

produces:

```text
UPI → UPIPaymentStrategy object
```

Final map:

```text
UPI    → UPIPaymentStrategy
CARD   → CardPaymentStrategy
PAYPAL → PayPalPaymentStrategy
```

Runtime lookup:

```java
PaymentStrategy strategy =
        paymentStrategyMap.get(PaymentType.UPI);
```

returns the existing `UPIPaymentStrategy` bean.

## 21. Complete Spring Runtime Flow

### Application Startup

```text
Spring starts
     ↓
Component scanning
     ↓
Finds Strategy beans
     ↓
Creates Strategy objects
     ↓
Finds PaymentStrategyRegistry
     ↓
Sees List<PaymentStrategy> dependency
     ↓
Collects all matching Strategy beans
     ↓
Injects List<PaymentStrategy>
     ↓
Registry constructor runs
     ↓
List is converted into Map
     ↓
Registry is ready
```

Map:

```text
UPI    → UPI Strategy bean
CARD   → Card Strategy bean
PAYPAL → PayPal Strategy bean
```

### Runtime Request

Suppose:

```http
POST /payments?type=UPI&amount=500
```

Flow:

```text
HTTP Request
     ↓
Embedded Server
     ↓
DispatcherServlet
     ↓
PaymentController
     ↓
PaymentService
     ↓
PaymentStrategyRegistry
     ↓
paymentStrategyMap.get(UPI)
     ↓
UPIPaymentStrategy bean
     ↓
strategy.pay(500)
     ↓
UPI Gateway / Business Logic
     ↓
Response
```

## 22. Startup vs Runtime

### Startup

Spring is doing:

```text
Scan Components
      ↓
Create Strategy Beans
      ↓
Find all PaymentStrategy beans
      ↓
Inject List<PaymentStrategy>
      ↓
Create Registry
      ↓
Registry builds Map
```

### Runtime

The request does:

```text
Request
   ↓
Controller
   ↓
Service
   ↓
Registry
   ↓
Map.get(type)
   ↓
Existing Strategy Bean
   ↓
execute()
```

With the default Spring singleton scope, the same Strategy bean instance is normally reused across requests.

## 23. Who Does What?

This is the most useful Spring mental model:

```text
                 SPRING
                   |
                   | Creates + wires
                   ↓
          Strategy Implementations
                   |
                   | Injects all implementations
                   ↓
                REGISTRY
                   |
                   | Builds lookup map
                   ↓
             Map<Key, Strategy>
                   |
                   | Runtime lookup
                   ↓
                SERVICE
                   |
                   | execute()
                   ↓
                STRATEGY
                   |
                   ↓
             Business Logic
```

### Spring

> Which Strategy objects exist, and how should their dependencies be wired?

### Registry

> Given this business key, which existing Strategy should I return?

### Strategy

> How should this operation actually be performed?

## 24. Adding a New Strategy

Add:

```java
@Component
public class WalletPaymentStrategy
        implements PaymentStrategy {

    @Override
    public PaymentType getType() {
        return PaymentType.WALLET;
    }

    @Override
    public void pay(double amount) {
        System.out.println("Processing Wallet payment");
    }
}
```

You do not need to modify the Controller, Service, or Registry selection logic.

Spring discovers the bean, injects it into the Strategy list, and the Registry includes:

```text
WALLET → WalletPaymentStrategy
```

This is the practical Spring version of making Strategy extensible.

## 25. Duplicate Keys

If two Strategies return the same key:

```text
UPIPaymentStrategy       → UPI
AnotherUPIStrategy       → UPI
```

then:

```java
Collectors.toMap(
    PaymentStrategy::getType,
    Function.identity()
)
```

throws an exception because duplicate Map keys are not allowed by this overload.

This is often desirable because two Strategies claiming the same business key is usually a configuration/design error.

If duplicates are intentionally allowed:

```java
Collectors.toMap(
    PaymentStrategy::getType,
    Function.identity(),
    (existing, replacement) -> replacement
)
```

This keeps the replacement value, but should not be used blindly because it can hide configuration mistakes.

## 26. Real-World Examples

### Payment

```text
UPI
Card
PayPal
Wallet
NetBanking
```

### Notification

```text
Email
SMS
Push
WhatsApp
```

### Pricing

```text
RegularCustomerPricing
PremiumCustomerPricing
CorporatePricing
FestivalPricing
```

### Tax Calculation

```text
IndiaTaxStrategy
USATaxStrategy
UKTaxStrategy
```

### File Processing

```text
CSVProcessor
JSONProcessor
XMLProcessor
ParquetProcessor
```

### Loan Calculation

```text
HomeLoanStrategy
PersonalLoanStrategy
CarLoanStrategy
```

## 27. LLD Examples

Strategy commonly appears in:

- Parking Lot
- Payment System
- Notification System
- E-commerce pricing
- Ride-sharing fare calculation
- Food delivery pricing
- Tax calculation
- File processing
- Authentication
- Discount calculation
- Shipping calculation
- Route selection
- Compression
- Sorting

### Parking Lot Example

```text
Parking Spot Selection
        |
        +-- NearestSpotStrategy
        +-- CheapestSpotStrategy
        +-- FirstAvailableSpotStrategy
```

Pricing could also use Strategy:

```text
Pricing
   |
   +-- HourlyPricingStrategy
   +-- WeekendPricingStrategy
   +-- DynamicPricingStrategy
```

## 28. Interview Questions

### Q1. What is Strategy Pattern?

> Strategy Pattern defines a family of interchangeable algorithms, encapsulates each algorithm behind a common interface, and allows the behavior to be selected independently from the Context that uses it.

### Q2. Why use Strategy instead of if-else?

Because Strategy:

- separates behaviors
- reduces conditional complexity
- improves maintainability
- improves testability
- supports extensibility
- follows composition

### Q3. Is Strategy based on inheritance?

The implementations implement a common interface, but the Context uses **composition**.

```text
Context HAS-A Strategy
```

### Q4. Strategy vs State?

Strategy selects an algorithm/behavior. State represents the current state of an object and changes behavior because of that state.

### Q5. Strategy vs Factory?

Factory focuses on object creation/selection. Strategy encapsulates interchangeable behavior. They can be used together.

### Q6. How would you implement Strategy in Spring Boot?

> Define a Strategy interface, make each implementation a Spring bean, inject all implementations as a List or Map, build a registry keyed by a business identifier, and retrieve the appropriate Strategy at runtime.

### Q7. Why inject `List<PaymentStrategy>`?

Because Spring can automatically provide all beans implementing the interface.

### Q8. Why convert the List into a Map?

To make runtime selection a direct lookup:

```java
map.get(type)
```

instead of repeatedly searching through the list.

## 29. Common Mistakes

### Mistake 1: Creating Strategy objects inside the Context

Bad:

```java
if (type == UPI) {
    new UPIPaymentStrategy();
}
```

In Spring, prefer letting Spring manage the Strategy beans.

### Mistake 2: Making every `if` a Strategy

Not every conditional needs Strategy. The branches should represent meaningful interchangeable behaviors.

### Mistake 3: Creating a giant Strategy

If the Strategy implementation still contains all the original `if-else` logic, the design has not actually separated the behavior.

### Mistake 4: Putting business logic in the Registry

The Registry should primarily answer:

```text
Which Strategy?
```

The Strategy should answer:

```text
How do we perform the behavior?
```

### Mistake 5: Confusing Strategy with Factory

Remember:

```text
Factory:
"Which object should I create?"

Strategy:
"How should I perform this behavior?"
```

### Mistake 6: Over-engineering

Do not introduce Strategy, Factory and Registry simply because you know the patterns. Use them when the problem actually contains meaningful variation.

## 30. Final Mental Model

Without Strategy:

```text
PaymentService
   |
   +-- if UPI
   +-- else if CARD
   +-- else if PAYPAL
   +-- else if WALLET
```

The service knows everything.

With Strategy:

```text
                    PaymentService
                          |
                          ↓
                  PaymentStrategy
                          |
             +------------+------------+
             |            |            |
             ↓            ↓            ↓
           UPI           CARD        PAYPAL
        Strategy       Strategy      Strategy
```

The service only knows the abstraction.

### Textbook Strategy

```text
Context
   |
   ↓
Strategy
   |
   +-- Concrete Strategy A
   +-- Concrete Strategy B
   +-- Concrete Strategy C
```

### Spring Strategy

```text
                    Spring
                      |
             creates + manages
                      |
          +-----------+-----------+
          |           |           |
          ↓           ↓           ↓
        UPI          CARD       PAYPAL
      Strategy      Strategy    Strategy
          \           |           /
           \          |          /
            +---------+---------+
                      |
                      ↓
             List<PaymentStrategy>
                      |
                      ↓
             Strategy Registry
                      |
                      ↓
          Map<PaymentType, Strategy>
                      |
                      ↓
                Runtime lookup
                      |
                      ↓
             Selected Strategy
                      |
                      ↓
                  execute()
```

## 31. Senior Engineer Mental Model

When solving an LLD problem, do not start by asking:

> "Which Design Pattern should I use?"

Instead ask:

### Question 1

> Is there behavior that varies?

Example:

```text
UPI vs Card
```

### Question 2

> Is that variation substantial enough to deserve separate classes?

If yes, Strategy may be appropriate.

### Question 3

> Does the Context need to know the concrete implementation?

Ideally, no. It should depend on the interface.

### Question 4

> How will the Strategy be selected?

Possible mechanisms:

```text
Simple caller selection
Factory
Registry
Spring List/Map injection
Configuration
```

### Question 5

> Who creates the Strategy?

Plain Java:

```text
Caller / Factory
```

Spring:

```text
Spring Container
```

### Question 6

> Who decides which Strategy to execute?

Usually the caller/service/registry, depending on the architecture.

## 32. One-Line Mental Model

> **Strategy = encapsulate different ways of doing the same thing behind a common interface, then choose the behavior without changing the object that uses it.**

In Spring:

> **Let Spring create and manage the Strategy beans, inject all implementations, build a registry/map for runtime lookup, and let the service execute the selected Strategy.**

## 33. Strategy Pattern Checklist

```text
[ ] Do I have multiple algorithms/behaviors?

[ ] Are those behaviors interchangeable?

[ ] Are they substantial enough to deserve separate classes?

[ ] Can I define a common interface?

[ ] Can the Context depend on the interface instead of implementations?

[ ] Can I remove a growing if-else/switch?

[ ] Does each Strategy have one clear responsibility?

[ ] How will the Strategy be selected?

[ ] If using Spring, can the implementations be Spring beans?

[ ] Can Spring inject List<Strategy> or Map<Key, Strategy>?

[ ] Should I build a Registry?

[ ] Is a Factory actually needed, or does Spring already create the objects?

[ ] Am I over-engineering the problem?
```

## 34. Final Interview Summary

> "Strategy Pattern is useful when we have multiple interchangeable ways of performing an operation. I define a common Strategy interface and move each algorithm into its own implementation. The Context depends only on the interface, so it doesn't need to know the concrete implementations. This removes growing conditional logic and makes the system easier to extend and test.
>
> In Spring Boot, I would typically make the Strategy implementations Spring beans. Spring can inject all implementations as a List or Map. I can build a registry keyed by something like an enum or business identifier and perform a runtime lookup. Spring handles object creation and dependency injection, while the registry handles strategy selection and the strategy itself handles the business behavior."


---

# 37. Spring Injection: What Actually Happens?

A common question is:

> "If all my Strategies are Spring Beans, why doesn't Spring itself automatically choose the correct Strategy at runtime?"

The answer is:

> **Spring manages dependency injection, but Spring does not know your business rule for choosing a Strategy.**

Spring knows:

```text
There are 3 beans implementing PaymentStrategy.
```

Spring does **not** automatically know:

```text
If paymentType = UPI → choose UPIPaymentStrategy
If paymentType = CARD → choose CardPaymentStrategy
```

That mapping is **business/application logic**.

So we usually build a Registry.

---

## 37.1 What Spring Knows

Suppose we have:

```java
@Component
public class UPIPaymentStrategy
        implements PaymentStrategy {
}
```

```java
@Component
public class CardPaymentStrategy
        implements PaymentStrategy {
}
```

```java
@Component
public class PayPalPaymentStrategy
        implements PaymentStrategy {
}
```

Spring's container knows:

```text
PaymentStrategy beans:

1. UPIPaymentStrategy
2. CardPaymentStrategy
3. PayPalPaymentStrategy
```

If we ask Spring for:

```java
List<PaymentStrategy>
```

Spring can inject all of them.

---

## 37.2 What Spring Does NOT Know

Suppose the request contains:

```text
paymentType = CARD
```

Spring does not automatically infer:

```text
CARD → CardPaymentStrategy
```

because that is our application's business mapping.

We explicitly define it:

```java
@Override
public PaymentType getType() {
    return PaymentType.CARD;
}
```

Then our Registry can build:

```text
CARD → CardPaymentStrategy
```

---

# 38. Who Calls the Registry?

The flow is:

```text
Client
   ↓
Controller
   ↓
PaymentService
   ↓
PaymentStrategyRegistry
   ↓
Selected PaymentStrategy
   ↓
execute()
```

For example:

```java
@RestController
public class PaymentController {

    private final PaymentService paymentService;

    public PaymentController(
            PaymentService paymentService) {

        this.paymentService = paymentService;
    }

    @PostMapping("/payments")
    public void pay(
            @RequestParam PaymentType type,
            @RequestParam double amount) {

        paymentService.pay(type, amount);
    }
}
```

The Service calls the Registry:

```java
@Service
public class PaymentService {

    private final PaymentStrategyRegistry registry;

    public PaymentService(
            PaymentStrategyRegistry registry) {

        this.registry = registry;
    }

    public void pay(
            PaymentType type,
            double amount) {

        PaymentStrategy strategy =
                registry.getStrategy(type);

        strategy.pay(amount);
    }
}
```

Important:

> **Spring does not call the Registry because it knows that it is a Strategy Registry.**

Spring creates and injects the Registry because it is a Spring bean.

Your application code calls:

```java
registry.getStrategy(type);
```

when a business request arrives.

---

# 39. Who Builds the HashMap?

Another important question is:

> "Who is calling the Registry and sending it the HashMap?"

There are actually two separate operations.

## Step 1 — Spring injects the List

Spring creates all Strategy beans.

Then it sees:

```java
public PaymentStrategyRegistry(
        List<PaymentStrategy> strategies)
```

Spring automatically constructs the list:

```text
strategies
    |
    +-- UPIPaymentStrategy object
    +-- CardPaymentStrategy object
    +-- PayPalPaymentStrategy object
```

and passes that List to the Registry constructor.

## Step 2 — Registry builds the Map

Inside our own constructor:

```java
public PaymentStrategyRegistry(
        List<PaymentStrategy> strategies) {

    this.paymentStrategyMap =
            strategies.stream()
                    .collect(Collectors.toMap(
                            PaymentStrategy::getType,
                            Function.identity()
                    ));
}
```

**Our Registry code** builds the HashMap.

Spring does not know that we want a payment-type-to-strategy map.

Conceptually:

```java
Map<PaymentType, PaymentStrategy> map =
        new HashMap<>();

for (PaymentStrategy strategy : strategies) {

    PaymentType key =
            strategy.getType();

    map.put(key, strategy);
}
```

So:

```text
Spring
  |
  | injects List
  ↓
Registry
  |
  | builds Map
  ↓
Map<PaymentType, PaymentStrategy>
```

---

# 40. Understanding `Collectors.toMap()` Completely

Consider:

```java
this.paymentStrategyMap =
        strategies.stream()
                .collect(Collectors.toMap(
                        PaymentStrategy::getType,
                        Function.identity()
                ));
```

Suppose:

```text
strategies = [
    UPI Strategy,
    Card Strategy,
    PayPal Strategy
]
```

### `PaymentStrategy::getType`

This is equivalent to:

```java
strategy -> strategy.getType()
```

It determines the **key**.

### `Function.identity()`

This is equivalent to:

```java
strategy -> strategy
```

It determines the **value**.

Therefore:

```text
UPI    → UPI Strategy
CARD   → Card Strategy
PAYPAL → PayPal Strategy
```

The full expression is essentially:

```java
Map<PaymentType, PaymentStrategy> map =
        new HashMap<>();

for (PaymentStrategy strategy : strategies) {

    PaymentType type =
            strategy.getType();

    map.put(type, strategy);
}
```

---

# 41. Complete Spring Startup Flow

At application startup:

```text
1. Spring starts
        ↓
2. Component scanning happens
        ↓
3. Spring discovers the Strategy classes
        ↓
4. Spring creates Strategy bean instances
        ↓
5. Spring discovers PaymentStrategyRegistry
        ↓
6. Registry requires List<PaymentStrategy>
        ↓
7. Spring finds ALL PaymentStrategy beans
        ↓
8. Spring creates the List
        ↓
9. Spring calls Registry constructor
        ↓
10. Registry receives the List
        ↓
11. Registry code builds the Map
        ↓
12. Registry stores the Map
        ↓
13. Application finishes startup
```

The key distinction is:

```text
Spring
  ↓
creates + injects objects

Registry
  ↓
builds business-key lookup map
```

---

# 42. Complete Runtime Request Flow

Suppose the application is already running.

The client sends:

```http
POST /payments
paymentType=CARD
amount=500
```

The runtime flow is:

```text
Client
  ↓
HTTP Request
  ↓
DispatcherServlet
  ↓
PaymentController
  ↓
PaymentService
  ↓
PaymentStrategyRegistry
  ↓
paymentStrategyMap.get(CARD)
  ↓
CardPaymentStrategy object
  ↓
strategy.pay(500)
  ↓
Card business logic
  ↓
Response
```

Notice what does **NOT** happen:

```text
Spring does NOT search all beans for every request.
Spring does NOT create a new CardPaymentStrategy for every request.
Spring does NOT decide that CARD means CardPaymentStrategy.
```

Instead:

```text
Spring startup
    ↓
creates + wires objects

Application runtime
    ↓
Registry performs lookup
```

---

# 43. Spring `List<Strategy>` Injection Is Not Runtime Strategy Selection

This distinction is extremely important.

When we write:

```java
public PaymentStrategyRegistry(
        List<PaymentStrategy> strategies)
```

we are saying:

> "Spring, give me all beans that implement `PaymentStrategy`."

We are NOT saying:

> "Spring, choose the Strategy based on my business key."

Spring provides the objects.

Our code decides how to organize and select them.

Therefore:

```text
Dependency Injection
        ≠
Business Strategy Selection
```

They are separate responsibilities.

---

# 44. Who Does What?

A strong mental model is:

```text
                 SPRING
                   |
                   | Creates + wires
                   ↓
          Strategy Implementations
                   |
                   | Injects all implementations
                   ↓
                REGISTRY
                   |
                   | Builds lookup map
                   ↓
             Map<Key, Strategy>
                   |
                   | Runtime lookup
                   ↓
                SERVICE
                   |
                   | execute()
                   ↓
                STRATEGY
                   |
                   ↓
             Business Logic
```

### Spring

> "Which Strategy objects exist, and how should their dependencies be wired?"

### Registry

> "Given this business key, which existing Strategy should I return?"

### Strategy

> "How should this operation actually be performed?"

---

# 45. Runtime Lookup vs Object Creation

This is an important distinction between Factory and Registry.

### Factory

> "Which object should I create?"

```text
type
 ↓
Factory
 ↓
new object
```

### Registry

> "Which already-existing object should I use?"

```text
type
 ↓
Registry
 ↓
existing Strategy
```

In Spring:

```text
Spring
  ↓
creates Strategy beans

Registry
  ↓
selects existing Strategy bean

Strategy
  ↓
executes behavior
```

Therefore, do not automatically call every Strategy Registry a Factory.

---

# 46. What If We Inject `Map` Directly?

Spring can also inject a Map of beans:

```java
private final Map<String, PaymentStrategy> strategies;

public PaymentService(
        Map<String, PaymentStrategy> strategies) {

    this.strategies = strategies;
}
```

Spring can provide keys based on bean names:

```text
"upiPaymentStrategy"    → UPI Strategy
"cardPaymentStrategy"   → Card Strategy
"payPalPaymentStrategy" → PayPal Strategy
```

This is useful when bean names are the lookup keys.

However, if the business key is:

```text
PaymentType.UPI
PaymentType.CARD
PaymentType.PAYPAL
```

then explicitly building:

```java
Map<PaymentType, PaymentStrategy>
```

is often clearer because the Map represents the actual business mapping.

---

# 47. Strategy Without Spring vs Strategy With Spring

## Without Spring

```text
Controller
   ↓
Service
   ↓
Factory
   ↓
new UPIStrategy()
```

The application explicitly creates objects.

## With Spring

```text
Application Startup
        ↓
Spring creates all Strategies
        ↓
Spring injects them
        ↓
Registry builds lookup map

Runtime
        ↓
Controller
        ↓
Service
        ↓
Registry
        ↓
Existing Strategy Bean
```

Spring removes object-creation and wiring boilerplate.

It does **not** remove the need for business selection logic.

---

# 48. Senior Engineer Takeaway

The most important mental model is:

> **Strategy Pattern solves behavior variation. Spring Dependency Injection solves object creation and wiring. A Registry solves runtime lookup/selection.**

These are three different concerns.

```text
                    STRATEGY PATTERN
                          |
                          ↓
              "How can behavior vary?"
                          |
                          ↓
                Strategy Interface
                          |
             +------------+------------+
             |            |            |
             ↓            ↓            ↓
           UPI          CARD         PAYPAL


                    SPRING DI
                          |
                          ↓
              "Who creates the objects?"
                          |
                          ↓
                  Spring Container
                          |
                          ↓
                 Strategy Beans


                    REGISTRY
                          |
                          ↓
             "Which one do I use?"
                          |
                          ↓
             Map<PaymentType, Strategy>
                          |
                          ↓
                    map.get(type)
```

This separation is one of the most useful things to understand when moving from textbook Design Patterns to real Spring Boot LLD/code.

---

# 49. Questions You Should Be Able to Answer

### 1. Why can't Spring automatically decide which Strategy to use?

Because Spring knows bean types and dependencies, but it does not know the application's business mapping between a runtime value such as `PaymentType.CARD` and a particular Strategy.

### 2. Who creates the Strategy objects?

Spring creates them when they are Spring-managed beans such as `@Component`.

### 3. Who injects `List<PaymentStrategy>`?

Spring's dependency injection container.

### 4. Who builds the `HashMap`?

Our Registry code, typically in its constructor or initialization logic.

### 5. Who calls the Registry?

Our application code, usually the Service layer, during request processing.

### 6. Who selects the Strategy?

The Registry performs the business-key lookup.

### 7. Who executes the actual behavior?

The selected Concrete Strategy.

### 8. Does Spring create a new Strategy for every request?

Not by default. Spring beans are singleton-scoped by default, so the same bean instance is normally reused.

### 9. Does the Registry create Strategy objects?

No. In this design it receives existing Spring-managed Strategy objects and indexes them.

### 10. Why use a Map?

To make runtime selection simple and direct:

```java
strategyMap.get(paymentType);
```

---

# 50. Final Spring Strategy Diagram

```text
                         APPLICATION STARTUP
                                |
                                ↓
                         Spring Container
                                |
               +----------------+----------------+
               |                |                |
               ↓                ↓                ↓
          UPI Bean          Card Bean        PayPal Bean
               \                |                /
                \               |               /
                 +--------------+--------------+
                                |
                                ↓
                    List<PaymentStrategy>
                                |
                                ↓
                    PaymentStrategyRegistry
                                |
                                | build map
                                ↓
                 Map<PaymentType, PaymentStrategy>
                                |
                 +--------------+--------------+
                 |              |              |
                UPI            CARD          PAYPAL
                 |              |              |
                 ↓              ↓              ↓
                UPI            Card          PayPal
              Strategy       Strategy       Strategy


                         RUNTIME REQUEST
                                |
                                ↓
                            Controller
                                |
                                ↓
                             Service
                                |
                                ↓
                            Registry
                                |
                         map.get(type)
                                |
                                ↓
                       Selected Strategy
                                |
                                ↓
                            execute()
                                |
                                ↓
                         Business Logic
                                |
                                ↓
                            Response
```
For implementation check: https://github.com/AryanSapra280/design-patterns/tree/develop/designpatterns/src/main/java/com/designpatterns/designpatterns/designPatterns/strategy