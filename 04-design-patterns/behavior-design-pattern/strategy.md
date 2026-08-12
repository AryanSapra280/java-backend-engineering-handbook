# Strategy Design Pattern

> **Category:** Behavioral Design Pattern  
> **Primary Goal:** Encapsulate interchangeable algorithms/behaviors and make them independently replaceable.

---

# 1. Intent

The **Strategy Design Pattern** defines a family of interchangeable algorithms, encapsulates each algorithm in its own class, and allows the algorithm to be selected independently from the object that uses it.

### In simple words

> **When a class can perform the same operation in multiple different ways, move those different ways into separate Strategy classes.**

The class using the strategy should not need to know the implementation details of the algorithm.

### Mental model

```text
Context
   |
   | uses
   ↓
Strategy
   |
   +---- Strategy A
   +---- Strategy B
   +---- Strategy C
```

The:

```text
Context  → knows WHAT needs to be done
Strategy → knows HOW it should be done
```

---

# 2. Problem

Consider an e-commerce application.

We need to calculate shipping charges for an order.

The application supports:

- Standard Shipping
- Express Shipping
- Same-Day Shipping

A naive implementation might put all the logic inside `OrderService`.

```java
public class OrderService {

    public double calculateShipping(
            Order order,
            String shippingType) {

        if (shippingType.equals("STANDARD")) {

            return 50;

        } else if (shippingType.equals("EXPRESS")) {

            return 150;

        } else if (shippingType.equals("SAME_DAY")) {

            return 300;
        }

        throw new IllegalArgumentException(
                "Unsupported shipping type");
    }
}
```

This works initially.

But imagine shipping rules become complicated.

```text
Standard:
    base price = 50
    free above ₹1000

Express:
    base price = 150
    additional charge based on distance

Same Day:
    base price = 300
    additional charge based on distance/time
```

Now `OrderService` starts becoming responsible for all of these algorithms.

---

# 3. Naive Solution

Let's make the problem more realistic.

```java
public class OrderService {

    public double calculateShipping(
            Order order,
            String shippingType) {

        if (shippingType.equals("STANDARD")) {

            double cost = 50;

            if (order.getTotalAmount() > 1000) {
                cost = 0;
            }

            return cost;
        }

        else if (shippingType.equals("EXPRESS")) {

            double cost = 150;

            if (order.getDistance() > 20) {
                cost += 100;
            }

            return cost;
        }

        else if (shippingType.equals("SAME_DAY")) {

            double cost = 300;

            if (order.getDistance() > 10) {
                cost += 200;
            }

            return cost;
        }

        throw new IllegalArgumentException(
                "Unsupported shipping type");
    }
}
```

At this point:

```text
OrderService
    |
    +-- Standard Shipping Algorithm
    |
    +-- Express Shipping Algorithm
    |
    +-- Same-Day Shipping Algorithm
```

Now suppose we add:

```text
International Shipping
Drone Delivery
Store Pickup
Premium Delivery
```

The class keeps growing.

---

# 4. What Smell / Problem Appears?

## 4.1 Large Conditional Logic

We have:

```java
if (...)
else if (...)
else if (...)
else if (...)
else if (...)
```

The conditional itself isn't automatically bad.

The problem is that each branch contains a **different algorithm**.

---

## 4.2 Single Responsibility Violation

`OrderService` is now responsible for:

```text
Order processing
+
Standard shipping calculation
+
Express shipping calculation
+
Same-day shipping calculation
+
Future shipping calculations
```

These responsibilities can change independently.

---

## 4.3 Open/Closed Principle Problem

Every time we introduce a new shipping algorithm:

```text
Add International Shipping
        ↓
Modify OrderService
```

Then:

```text
Add Drone Delivery
        ↓
Modify OrderService
```

The class is continuously being modified.

---

## 4.4 Testing Becomes Harder

Testing one algorithm requires creating the entire `OrderService` and testing through conditional branches.

Instead, we'd ideally like:

```text
Test StandardShippingStrategy independently
Test ExpressShippingStrategy independently
Test SameDayShippingStrategy independently
```

---

## 4.5 High Coupling

`OrderService` knows implementation details of every shipping algorithm.

```text
OrderService
    |
    +--> Standard logic
    +--> Express logic
    +--> Same Day logic
```

This creates unnecessary coupling.

---

# 5. Core Idea

Identify the behavior that changes:

```text
Shipping Calculation
```

Then ask:

> "Are there multiple ways to perform this operation?"

Yes:

```text
Standard
Express
Same Day
```

So extract the behavior.

```text
ShippingStrategy
       |
       +---- StandardShippingStrategy
       |
       +---- ExpressShippingStrategy
       |
       +---- SameDayShippingStrategy
```

Now `OrderService` doesn't implement shipping algorithms.

It simply delegates to the selected strategy.

## Before

```text
OrderService
     |
     +-- Standard algorithm
     +-- Express algorithm
     +-- Same Day algorithm
```

## After

```text
OrderService
     |
     +---- ShippingStrategy
                |
                +---- Standard
                +---- Express
                +---- Same Day
```

---

# 6. Structure

The Strategy Pattern usually contains three major components.

## 6.1 Strategy

Defines the common operation.

```java
public interface ShippingStrategy {

    double calculateShipping(Order order);
}
```

---

## 6.2 Concrete Strategies

Each strategy implements one algorithm.

```text
ShippingStrategy
       |
       +---- StandardShippingStrategy
       +---- ExpressShippingStrategy
       +---- SameDayShippingStrategy
```

---

## 6.3 Context

The Context uses the strategy.

```text
OrderService
     |
     +---- ShippingStrategy
```

The Context doesn't care which implementation it receives.

---

## UML-style representation

```text
                +----------------------+
                |   ShippingStrategy   |
                +----------------------+
                | + calculateShipping |
                +----------+-----------+
                           ^
                           |
             +-------------+-------------+
             |             |             |
             |             |             |
+-------------------+ +----------------+ +---------------------+
| StandardShipping  | | ExpressShipping | | SameDayShipping     |
| Strategy          | | Strategy        | | Strategy            |
+-------------------+ +----------------+ +---------------------+

                +----------------------+
                |     OrderService     |
                +----------------------+
                | - strategy          |
                +----------------------+
                | + calculateShipping |
                +----------------------+
```

---

# 7. Java Implementation

## 7.1 Order

```java
public class Order {

    private final double totalAmount;
    private final double distance;

    public Order(
            double totalAmount,
            double distance) {

        this.totalAmount = totalAmount;
        this.distance = distance;
    }

    public double getTotalAmount() {
        return totalAmount;
    }

    public double getDistance() {
        return distance;
    }
}
```

---

## 7.2 Strategy Interface

```java
public interface ShippingStrategy {

    double calculateShipping(Order order);
}
```

This interface represents the common behavior.

Every shipping strategy must answer:

```text
calculateShipping(Order)
```

---

## 7.3 Standard Shipping Strategy

```java
public class StandardShippingStrategy
        implements ShippingStrategy {

    @Override
    public double calculateShipping(Order order) {

        if (order.getTotalAmount() >= 1000) {
            return 0;
        }

        return 50;
    }
}
```

---

## 7.4 Express Shipping Strategy

```java
public class ExpressShippingStrategy
        implements ShippingStrategy {

    @Override
    public double calculateShipping(Order order) {

        double cost = 150;

        if (order.getDistance() > 20) {
            cost += 100;
        }

        return cost;
    }
}
```

---

## 7.5 Same-Day Shipping Strategy

```java
public class SameDayShippingStrategy
        implements ShippingStrategy {

    @Override
    public double calculateShipping(Order order) {

        double cost = 300;

        if (order.getDistance() > 10) {
            cost += 200;
        }

        return cost;
    }
}
```

---

## 7.6 Context

```java
public class OrderService {

    private final ShippingStrategy shippingStrategy;

    public OrderService(
            ShippingStrategy shippingStrategy) {

        this.shippingStrategy = shippingStrategy;
    }

    public double calculateShipping(Order order) {

        return shippingStrategy.calculateShipping(order);
    }
}
```

Notice that `OrderService` doesn't know:

```text
Standard
Express
Same Day
```

It only knows:

```java
ShippingStrategy
```

This is the decoupling we wanted.

---

## 7.7 Client Code

```java
public class Main {

    public static void main(String[] args) {

        Order order =
                new Order(1500, 15);

        ShippingStrategy strategy =
                new StandardShippingStrategy();

        OrderService orderService =
                new OrderService(strategy);

        double shippingCost =
                orderService.calculateShipping(order);

        System.out.println(
                "Shipping cost: " + shippingCost);
    }
}
```

Output:

```text
Shipping cost: 0.0
```

Because the order is above ₹1000.

---

## 7.8 Changing the Strategy

Change:

```java
ShippingStrategy strategy =
        new StandardShippingStrategy();
```

to:

```java
ShippingStrategy strategy =
        new ExpressShippingStrategy();
```

The `OrderService` doesn't change.

That's the core benefit.

---

## 7.9 Runtime Strategy Selection

A strategy can be selected at runtime.

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

The selected strategy may come from:

- user input
- configuration
- business rules
- another service
- a Factory
- dependency injection

The important part is that the Context does not need to know the internal algorithm.

---

# 8. How to Recognize It

When reading an LLD problem, ask these questions.

## Question 1

> Do I have multiple ways of doing the same thing?

Examples:

```text
Payment
    UPI
    Card
    PayPal
```

```text
Notification
    Email
    SMS
    Push
```

```text
Routing
    Fastest
    Cheapest
    Shortest
```

```text
Pricing
    Normal
    Festival
    Membership
    Surge
```

```text
Parking Allocation
    Nearest
    First Available
    Best Fit
```

If yes, Strategy may be appropriate.

---

## Question 2

> Is the behavior likely to change independently?

For example:

```text
Shipping rules change frequently.

Order processing does not.
```

That is a strong Strategy signal.

---

## Question 3

> Is there a growing conditional containing different algorithms?

For example:

```java
if (type == A) {
    // 50 lines
}
else if (type == B) {
    // 80 lines
}
else if (type == C) {
    // 100 lines
}
```

This is a strong candidate.

---

## Question 4

> Could I describe the variations as "different ways of doing X"?

Examples:

```text
Different ways of calculating price
Different ways of allocating parking
Different ways of routing
Different ways of paying
Different ways of sending notifications
```

If yes, think Strategy.

---

# 9. When to Use

Use Strategy when:

## 9.1 Multiple Algorithms Exist

```text
Sorting:
    QuickSort
    MergeSort
    HeapSort
```

---

## 9.2 Algorithms Change Independently

For example:

```text
Pricing algorithm
```

changes frequently while:

```text
Order processing
```

remains stable.

---

## 9.3 Conditional Logic Is Growing

Instead of:

```java
if (...)
else if (...)
else if (...)
else if (...)
```

extract each algorithm.

---

## 9.4 Runtime Behavior Selection Is Required

For example:

```text
User chooses:
    UPI

System chooses:
    UPIStrategy
```

---

## 9.5 You Want to Add Algorithms Without Modifying Existing Code

For example:

```text
Current:
    Standard
    Express

Later:
    Drone
    International
    Premium
```

New strategies can be added.

---

# 10. When NOT to Use

Strategy should not be used just because it exists.

## 10.1 Only One Algorithm Exists

If there is only:

```java
calculatePrice()
```

and there is no meaningful variation, don't create:

```text
PriceStrategy
DefaultPriceStrategy
PriceContext
Factory
```

That is unnecessary abstraction.

---

## 10.2 Tiny Stable Conditional

This is perfectly acceptable:

```java
if (isPremium) {
    return 20;
}

return 10;
```

Creating a Strategy hierarchy for this may make the code worse.

---

## 10.3 Variations Are Not Actually the Same Behavior

Don't force unrelated things into:

```java
SomeStrategy
```

just to satisfy a pattern.

Strategies should represent variations of the same conceptual operation.

---

## 10.4 Overengineering

Avoid creating:

```text
Strategy
StrategyFactory
StrategyManager
StrategyResolver
StrategyProvider
StrategyRegistry
```

when there are only two trivial implementations.

Patterns should reduce complexity, not create it.

---

# 11. SOLID Connection

Strategy has strong connections to SOLID.

## 11.1 Single Responsibility Principle

Each Strategy has one responsibility.

```text
StandardShippingStrategy
    → standard shipping calculation
```

```text
ExpressShippingStrategy
    → express shipping calculation
```

Instead of:

```text
OrderService
    → order processing
    → standard calculation
    → express calculation
    → same-day calculation
```

---

## 11.2 Open/Closed Principle

We can add:

```java
public class DroneShippingStrategy
        implements ShippingStrategy {

    @Override
    public double calculateShipping(Order order) {
        return 500;
    }
}
```

without changing:

```java
OrderService
```

This is a major benefit.

---

## 11.3 Dependency Inversion Principle

`OrderService` depends on:

```java
ShippingStrategy
```

not:

```java
StandardShippingStrategy
```

Therefore:

```text
OrderService
      |
      ↓
ShippingStrategy
      ↑
      |
Concrete strategies
```

The high-level module depends on an abstraction.

---

## 11.4 Dependency Injection

The strategy is injected through the constructor:

```java
public OrderService(
        ShippingStrategy shippingStrategy) {

    this.shippingStrategy = shippingStrategy;
}
```

This also makes testing easier.

For example:

```java
ShippingStrategy fakeStrategy =
        order -> 100;

OrderService service =
        new OrderService(fakeStrategy);
```

---

# 12. Composition vs Inheritance

Strategy primarily uses **composition**.

## Inheritance

Inheritance represents:

```text
IS-A
```

Example:

```text
Dog IS-A Animal
```

```java
class Dog extends Animal {
}
```

---

## Strategy

Strategy represents:

```text
HAS-A
```

Example:

```text
OrderService HAS-A ShippingStrategy
```

```java
class OrderService {

    private final ShippingStrategy strategy;

}
```

This is composition.

---

## Why composition is useful

With inheritance:

```text
OrderService
    |
    +-- StandardOrderService
    +-- ExpressOrderService
    +-- SameDayOrderService
```

Behavior is tied to the class hierarchy.

With Strategy:

```text
OrderService
     |
     +---- ShippingStrategy
```

and:

```text
StandardStrategy
ExpressStrategy
SameDayStrategy
```

The behavior can be replaced independently.

### Mental model

```text
Inheritance
    → change behavior through subclass

Strategy
    → change behavior through composition
```

---

# 13. Strategy vs State

This is one of the most commonly confused pairs.

Structurally, they can look almost identical.

Both may have:

```java
interface Behavior {

    void execute();
}
```

with multiple implementations.

The difference is the **intent**.

---

## Strategy

Question:

> **Which algorithm should I use?**

Example:

```text
Payment
    |
    +-- UPI
    +-- Card
    +-- PayPal
```

The behavior is selected because we want a particular algorithm.

---

## State

Question:

> **What should I do because my current state is X?**

Example:

```text
Order

NEW
 ↓
PAID
 ↓
SHIPPED
 ↓
DELIVERED
```

The behavior changes because the object's state changes.

---

## Strategy

```text
Client chooses behavior
          ↓
Strategy
```

## State

```text
Object state changes
          ↓
Behavior changes
```

---

## Example

### Strategy

```java
PaymentService service =
        new PaymentService(
                new UPIPaymentStrategy());
```

We intentionally selected UPI.

### State

```java
order.setState(
        new ShippedState());
```

The order's state determines what operations are allowed.

---

## Quick distinction

```text
Strategy → "How should I do this?"

State    → "What should I do in my current state?"
```

---

# 14. Strategy vs Factory

Strategy and Factory solve different problems.

## Strategy

Responsible for:

> **Encapsulating behavior.**

```text
PaymentStrategy
    |
    +-- UPI
    +-- Card
    +-- PayPal
```

---

## Factory

Responsible for:

> **Creating/selecting an object.**

```text
PaymentType
     |
     ↓
PaymentStrategyFactory
     |
     +-- UPI Strategy
     +-- Card Strategy
     +-- PayPal Strategy
```

---

## They can work together

A common design is:

```text
Client
  |
  ↓
Factory
  |
  ↓
Strategy
  |
  ↓
execute()
```

Example:

```java
public class ShippingStrategyFactory {

    public static ShippingStrategy getStrategy(
            String type) {

        switch (type) {

            case "STANDARD":
                return new StandardShippingStrategy();

            case "EXPRESS":
                return new ExpressShippingStrategy();

            case "SAME_DAY":
                return new SameDayShippingStrategy();

            default:
                throw new IllegalArgumentException(
                        "Unsupported shipping type");
        }
    }
}
```

Client:

```java
ShippingStrategy strategy =
        ShippingStrategyFactory
                .getStrategy("EXPRESS");

OrderService service =
        new OrderService(strategy);
```

Factory handles:

```text
Which object?
```

Strategy handles:

```text
How should the behavior execute?
```

---

# 15. Real-World Examples

## Payment

```text
PaymentStrategy
    ├── UPI
    ├── Credit Card
    ├── Debit Card
    └── PayPal
```

## Notification

```text
NotificationStrategy
    ├── Email
    ├── SMS
    └── Push
```

## Routing

```text
RoutingStrategy
    ├── ShortestRoute
    ├── FastestRoute
    └── CheapestRoute
```

## Discount

```text
DiscountStrategy
    ├── NoDiscount
    ├── FestivalDiscount
    ├── MembershipDiscount
    └── BulkDiscount
```

## Compression

```text
CompressionStrategy
    ├── Zip
    ├── GZip
    └── Brotli
```

## Storage

```text
StorageStrategy
    ├── LocalStorage
    ├── S3Storage
    └── AzureStorage
```

---

# 16. LLD Examples

Strategy appears frequently in LLD problems.

## 16.1 Parking Lot

One strong candidate is:

```text
Parking Spot Allocation
```

Different algorithms:

```text
Nearest Spot
First Available
Best Fit
```

Design:

```text
ParkingSpotAllocationStrategy
            |
            +---- NearestSpotStrategy
            +---- FirstAvailableStrategy
            +---- BestFitStrategy
```

Example interface:

```java
public interface ParkingSpotAllocationStrategy {

    ParkingSpot findSpot(
            Vehicle vehicle,
            List<ParkingSpot> spots);
}
```

Example implementation:

```java
public class FirstAvailableStrategy
        implements ParkingSpotAllocationStrategy {

    @Override
    public ParkingSpot findSpot(
            Vehicle vehicle,
            List<ParkingSpot> spots) {

        for (ParkingSpot spot : spots) {

            if (spot.isAvailable()
                    && spot.canFit(vehicle)) {

                return spot;
            }
        }

        return null;
    }
}
```

Context:

```java
public class ParkingLot {

    private final ParkingSpotAllocationStrategy strategy;

    public ParkingLot(
            ParkingSpotAllocationStrategy strategy) {

        this.strategy = strategy;
    }

    public ParkingSpot findSpot(
            Vehicle vehicle,
            List<ParkingSpot> spots) {

        return strategy.findSpot(vehicle, spots);
    }
}
```

The important reasoning is:

> Parking spot allocation has multiple interchangeable algorithms, so isolate that changing behavior behind a Strategy abstraction.

---

## 16.2 Payment System

```text
PaymentStrategy
    ├── UPI
    ├── Card
    └── PayPal
```

---

## 16.3 Ride Sharing

Driver matching:

```text
DriverMatchingStrategy
    ├── NearestDriver
    ├── HighestRatedDriver
    └── CheapestDriver
```

---

## 16.4 E-commerce Pricing

```text
PricingStrategy
    ├── RegularPricing
    ├── FestivalPricing
    ├── MembershipPricing
    └── DynamicPricing
```

---

## 16.5 Food Delivery

Delivery assignment:

```text
DeliveryAssignmentStrategy
    ├── NearestDeliveryPartner
    ├── LeastBusyPartner
    └── HighestRatedPartner
```

---

# 17. Interview Questions

## Q1. What is Strategy Pattern?

> Strategy is a behavioral design pattern used when multiple interchangeable algorithms exist for the same operation. Each algorithm is encapsulated behind a common interface, and the Context delegates the operation to the selected strategy.

---

## Q2. Why not just use if-else?

> If the conditional logic is small and stable, if-else is perfectly fine. Strategy becomes useful when the branches represent substantial algorithms that change independently, are reused, or are expected to grow.

---

## Q3. What principles does Strategy support?

Mainly:

```text
Single Responsibility Principle
Open/Closed Principle
Dependency Inversion Principle
```

---

## Q4. Does Strategy use composition or inheritance?

Composition.

```java
class Context {

    private Strategy strategy;
}
```

---

## Q5. Can Strategy be selected at runtime?

Yes.

```java
PaymentStrategy strategy;

if (paymentType.equals("UPI")) {
    strategy = new UPIStrategy();
} else {
    strategy = new CardStrategy();
}
```

The selection mechanism itself may be extracted into a Factory or another component.

---

## Q6. Difference between Strategy and State?

```text
Strategy → different algorithms
State    → different behavior based on current state
```

---

## Q7. Can Factory and Strategy be used together?

Yes.

Factory can select/create the appropriate Strategy.

```text
Factory
   ↓
Strategy
   ↓
execute()
```

---

## Q8. Does Strategy always require an interface?

Not strictly.

The important concept is the abstraction that allows interchangeable behavior.

In Java, an interface is usually the cleanest implementation.

---

## Q9. Biggest benefit?

Decoupling changing algorithms from stable business logic.

---

## Q10. Biggest disadvantage?

More classes and abstractions.

Strategy can be overengineering when the behavior is trivial and unlikely to change.

---

# 18. Common Mistakes

## Mistake 1 — "Strategy = Interface + Multiple Classes"

Not enough.

The implementations must represent **interchangeable behavior**.

---

## Mistake 2 — Using Strategy for Every If-Else

Don't.

```java
if (age > 18) {
    ...
}
```

doesn't automatically mean Strategy.

Ask:

> Is this actually a different algorithm?

---

## Mistake 3 — Keeping Algorithms Inside Context

Bad:

```java
class PaymentService {

    private PaymentStrategy strategy;

    public void pay() {

        if (strategy instanceof UPIStrategy) {
            // UPI algorithm
        }

        if (strategy instanceof CardStrategy) {
            // Card algorithm
        }
    }
}
```

Now the Context still knows concrete implementations.

You have defeated the purpose of Strategy.

Prefer:

```java
strategy.pay();
```

---

## Mistake 4 — Strategy Selection Everywhere

Bad:

```java
if (type.equals("UPI")) {
    new UPIStrategy();
}

if (type.equals("CARD")) {
    new CardStrategy();
}
```

repeated across multiple services.

Instead, centralize selection using:

```text
Factory
Registry
Dependency Injection
Configuration
```

depending on the problem.

---

## Mistake 5 — Confusing Strategy with State

Remember:

```text
Strategy:
    Which algorithm should I use?

State:
    What should I do because I am currently in this state?
```

---

## Mistake 6 — Creating Too Many Layers

Avoid unnecessary abstractions such as:

```text
Strategy
StrategyFactory
StrategyManager
StrategyResolver
StrategyProvider
StrategyRegistry
```

unless the system actually requires them.

---

## Mistake 7 — Ignoring the Context

Strategy is not just a collection of algorithms.

The Context is important because it owns the stable workflow and delegates the variable part.

---

# 19. Final Mental Model

The easiest way to remember Strategy:

```text
                    PROBLEM
                       |
                       ↓
             Multiple ways to do X
                       |
                       ↓
             Do these algorithms
             change independently?
                       |
                      YES
                       |
                       ↓
             Extract the behavior
                       |
                       ↓
              Strategy Interface
                       |
          +------------+------------+
          |            |            |
          ↓            ↓            ↓
      Strategy A   Strategy B   Strategy C
          \            |            /
           \           |           /
            +----------+----------+
                       |
                       ↓
                    Context
```

The Context should know:

```text
WHAT needs to happen
```

The Strategy should know:

```text
HOW it happens
```

---

# The One Sentence to Remember

> **Use Strategy when a behavior or algorithm can vary independently and you want those variations to be interchangeable.**

---

# The One Question to Ask in LLD

Whenever you read a problem, ask:

> **"Do I have multiple ways of performing the same operation?"**

If yes, investigate Strategy.

Examples:

```text
How do I pay?
    → UPI / Card / PayPal

How do I calculate shipping?
    → Standard / Express / Same Day

How do I find a parking spot?
    → Nearest / First Available / Best Fit

How do I calculate discount?
    → Regular / Festival / Membership

How do I find a driver?
    → Nearest / Highest Rated / Least Busy
```

---

# Strategy Pattern — Quick Revision

```text
CATEGORY
    Behavioral

PURPOSE
    Encapsulate interchangeable algorithms

KEY IDEA
    Separate changing behavior from stable logic

MAIN COMPONENTS
    1. Strategy
    2. Concrete Strategy
    3. Context

USES
    Multiple algorithms
    Runtime behavior selection
    Growing conditional logic
    Independently changing behavior

MAIN SOLID CONNECTION
    SRP
    OCP
    DIP

DESIGN STYLE
    Composition

KEY DIFFERENCE
    Strategy → choose algorithm
    State    → behavior changes with state
    Factory  → creates/selects object

CORE QUESTION
    "Do I have multiple ways of performing the same operation?"
```

---

# Strategy Pattern in One Diagram

```text
                         CLIENT
                           |
                           | chooses
                           ↓
                  +-------------------+
                  |     Strategy      |
                  +-------------------+
                           ^
                           |
              +------------+------------+
              |            |            |
              |            |            |
      +---------------+ +---------+ +-------------+
      | Strategy A    | |Strategy B| | Strategy C |
      +---------------+ +---------+ +-------------+
              ^            ^            ^
              |            |            |
              +------------+------------+
                           |
                           |
                    +-------------+
                    |   Context   |
                    +-------------+
                    |             |
                    | uses        |
                    | Strategy    |
                    +-------------+
```

---

# Final Takeaway

Do not memorize:

> "Strategy is an interface with multiple implementations."

Memorize the **reasoning**:

```text
Something is changing
        ↓
The changing part is a behavior/algorithm
        ↓
There are multiple ways to perform it
        ↓
Those ways can vary independently
        ↓
Extract them behind an abstraction
        ↓
Let the Context delegate to the selected Strategy
```

That reasoning is what will allow you to recognize Strategy in an unfamiliar LLD problem.

---

# Next Pattern

After Strategy, continue with:

```text
1. Strategy   ✓
2. Factory
3. Builder
4. Singleton
5. State
```

After all five patterns are understood, apply them to the **Parking Lot LLD** and identify which patterns are genuinely useful rather than forcing every pattern into the design.
