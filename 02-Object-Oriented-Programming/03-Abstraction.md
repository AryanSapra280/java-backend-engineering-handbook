# Abstraction

> **Learning Goal:** Understand why abstraction exists, how it reduces complexity, and how it differs from encapsulation in object-oriented design.

---

# Why do we need Abstraction?

As software systems become larger, exposing every implementation detail to users or other developers makes the system difficult to understand and maintain.

Abstraction solves this by exposing only the operations that a user needs while hiding unnecessary implementation details.

The goal is not to hide information for security, but to **reduce complexity**.

---

# Problem Statement

Imagine driving a car.

To accelerate the car, you simply press the accelerator pedal.

You do not need to know:

* How fuel is injected.
* How pistons move.
* How the gearbox operates.
* How combustion occurs.

The car exposes a simple interface:

```text
Accelerator
        ↓
Increase Speed
```

All mechanical complexity remains hidden.

This is abstraction.

---

# Definition

Abstraction is the process of exposing only the essential behaviors of an object while hiding unnecessary implementation details.

It answers the question:

> **"What should the outside world know?"**

---

# Engineering Perspective

Abstraction exists to reduce cognitive load.

Users interact with a simplified interface instead of understanding the complete implementation.

Instead of exposing internal algorithms, objects expose meaningful business operations.

---

# Java Perspective

One common way to implement abstraction in Java is through interfaces.

```java
public interface PaymentService {

    void pay(double amount);

}
```

Different implementations provide different behaviors.

```java
class UpiPaymentService implements PaymentService

class CardPaymentService implements PaymentService

class WalletPaymentService implements PaymentService
```

The client only knows:

```java
paymentService.pay(amount);
```

It does not know how payment is processed.

---

# Abstract Class vs Interface

Both support abstraction but serve different purposes.

### Interface

Represents **capability (CAN DO).**

Example:

```java
interface PaymentService
```

Used when multiple unrelated classes provide the same behavior.

---

### Abstract Class

Represents **inheritance (IS A).**

Example:

```java
abstract class Vehicle
```

Used when child classes share common state or behavior.

---

# Enterprise Example

Spring Boot heavily relies on abstraction.

Example:

```java
userRepository.save(user);
```

The application uses the repository without knowing:

* SQL generation
* Database connection handling
* Transaction management
* ORM implementation

Spring hides these implementation details.

---

# Real World Example

Ordering food online.

The customer presses:

```text
Place Order
```

The customer does not know:

* Restaurant processing
* Rider assignment
* Route optimization
* Payment settlement
* Notification delivery

The customer interacts with a simple interface.

Complexity remains hidden.

---

# Advantages

* Reduces system complexity.
* Improves readability.
* Encourages modular design.
* Makes systems easier to extend.
* Allows implementation changes without affecting clients.

---

# Common Mistakes

❌ Thinking abstraction means making variables private.

❌ Confusing abstraction with encapsulation.

❌ Creating interfaces for every class without a design reason.

❌ Exposing implementation details unnecessarily.

---

# Frequently Confused Concepts

## Abstraction vs Encapsulation

| Abstraction                    | Encapsulation                           |
| ------------------------------ | --------------------------------------- |
| Reduces complexity             | Protects object integrity               |
| Hides implementation           | Protects business rules                 |
| Focuses on **what** is exposed | Focuses on **how** state changes safely |
| Goal: Simplicity               | Goal: Safety                            |

---

## Interface vs Abstract Class

| Interface                              | Abstract Class                |
| -------------------------------------- | ----------------------------- |
| Represents capability (CAN DO)         | Represents inheritance (IS A) |
| No instance state                      | Can contain state             |
| Multiple interfaces can be implemented | Single inheritance            |
| Defines a contract                     | Shares common implementation  |

---

# Interview Questions

1. What is abstraction?

2. Why does abstraction exist?

3. Difference between abstraction and encapsulation?

4. Can abstraction exist without encapsulation?

5. Interface vs Abstract Class?

6. Why does Spring use interfaces extensively?

7. Should every class have an interface?

---

# Revision Notes

* Abstraction reduces complexity.
* It exposes only essential behaviors.
* Interfaces and abstract classes help implement abstraction.
* Clients should depend on behavior rather than implementation.
* Abstraction and encapsulation solve different engineering problems.

---

# Key Takeaways

* Encapsulation answers **"Who can modify me?"**
* Abstraction answers **"What should users know?"**
* Interfaces define contracts.
* Abstract classes provide shared behavior.
* Good software exposes simple APIs while hiding internal complexity.
