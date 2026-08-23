# 09 — Composition vs Inheritance

## Definition

**Inheritance** represents an **IS-A** relationship where one class derives from another and inherits its behavior/contract.

```java
class Animal {
    void eat() {}
}

class Dog extends Animal {
}
```

`Dog IS-A Animal`.

**Composition** represents a **HAS-A** relationship where one object contains or uses another object to provide behavior.

```java
class Engine {
    void start() {}
}

class Car {
    private Engine engine;

    Car(Engine engine) {
        this.engine = engine;
    }
}
```

`Car HAS-A Engine`.

---

## Core Difference

```text
Inheritance
    ↓
IS-A
    ↓
reuse through class hierarchy

Composition
    ↓
HAS-A
    ↓
reuse through object collaboration
```

The first-principles question is:

> Is this a genuine, stable IS-A relationship, or do I simply need another object to provide behavior?

---

## Payment Example

Inheritance:

```java
class Payment {
    void process() {}
}

class CardPayment extends Payment {}
class UpiPayment extends Payment {}
class WalletPayment extends Payment {}
```

If payment behavior can vary independently and new payment methods may be added, composition is often more flexible:

```java
interface PaymentProcessor {
    void process();
}

class UpiProcessor implements PaymentProcessor {}
class CardProcessor implements PaymentProcessor {}
class WalletProcessor implements PaymentProcessor {}

class PaymentService {
    private PaymentProcessor processor;

    PaymentService(PaymentProcessor processor) {
        this.processor = processor;
    }

    void processPayment() {
        processor.process();
    }
}
```

Conceptually:

```text
PaymentService
      │
      │ HAS-A
      ↓
PaymentProcessor
      │
 ┌────┼─────┐
 ↓    ↓     ↓
UPI  Card  Wallet
```

### Interview-quality reasoning

> I'd prefer composition because the payment behavior is likely to vary independently of the payment service. Composition lets the service depend on the `PaymentProcessor` abstraction and inject different implementations, making the system more extensible and allowing behavior to change without modifying the consumer. Inheritance would couple the variation to the class hierarchy.

### Important correction

It is not correct to say inheritance makes extension impossible. A consumer can create another subclass such as:

```java
class CryptoPayment extends Payment {}
```

The stronger argument is **coupling, rigidity, and behavior being tied to the hierarchy**.

---

## Strategy Pattern Connection

Don't jump directly to the pattern name.

Reason:

```text
What varies?
      ↓
Payment processing behavior

How isolate it?
      ↓
Composition

How should the consumer depend on it?
      ↓
Interface

What pattern does this resemble?
      ↓
Strategy Pattern
```

This is especially important for scenario-based interviews.

---

# Composition vs Aggregation

### Composition

A strong HAS-A relationship where the contained object's lifecycle is owned by the containing object.

```java
class Car {
    private final Engine engine;

    Car() {
        this.engine = new Engine();
    }
}
```

Think:

```text
Car
 │
 └── owns → Engine
```

### Aggregation

A weaker HAS-A relationship where the contained object can exist independently.

```java
class Employee {}

class Department {
    private List<Employee> employees;

    Department(List<Employee> employees) {
        this.employees = employees;
    }
}
```

Think:

```text
Department
    │
    └── has → Employee
```

### Easy distinction

Ask:

> **Who owns the lifecycle?**

Composition:
`Container owns component → lifecycle is tied to container`

Aggregation:
`Container references component → component has an independent lifecycle`

Examples:

- House → Room: generally composition
- University → Student: generally aggregation
- Order → OrderItem: generally composition
- Team → Player: generally aggregation

---

## Important Nuance: Composition Does NOT Mean `new`

Do not memorize:

> Composition means the parent must create the child with `new`.

That's an oversimplification.

This is valid:

```java
class Car {
    private final Engine engine;

    Car(Engine engine) {
        this.engine = engine;
    }
}
```

Whether the relationship is composition depends on **domain ownership/lifecycle semantics**, not simply where `new` appears.

This is especially relevant to Spring dependency injection:

```java
class PaymentService {
    private final PaymentProcessor processor;

    PaymentService(PaymentProcessor processor) {
        this.processor = processor;
    }
}
```

Spring can manage the processor bean while `PaymentService` is composed with that processor.

---

# Key Principle

> **Favor composition over inheritance when you need to assemble behavior, vary behavior independently, or avoid unnecessary coupling to a class hierarchy.**

But:

> **Do not avoid inheritance when a genuine, stable IS-A relationship exists.**

---

# Interview Decision Framework

```text
1. What relationship exists?
       ↓
   IS-A or HAS-A?

2. What is expected to vary?
       ↓
   Type or behavior?

3. Does behavior need to change independently?
       ↓
   If yes → composition is attractive.

4. Would inheritance create tight coupling?
       ↓
   If yes → prefer composition.

5. Is there a genuine stable subtype relationship?
       ↓
   If yes → inheritance may be appropriate.

6. Is behavior injected behind an interface?
       ↓
   Strategy/delegation may be the resulting pattern.
```

---

# Connection to Interview Preparation

A weak answer jumps to:

> "I'll use Strategy Pattern."

A stronger answer builds:

```text
Requirement
    ↓
What varies?
    ↓
Design relationship
    ↓
Composition / inheritance
    ↓
Abstraction
    ↓
Trade-offs
    ↓
Pattern name
```

This is the reasoning style we want for SDE-2/SDE-3 scenario questions.

---

# Java Core Track

```text
9. Composition vs Inheritance     ← DONE
10. Object Methods
    ├── equals()                  ← DONE
    ├── hashCode()                ← DONE
    └── toString()                ← NEXT
11. String / String Pool
12. Wrapper Classes
13. Collections
14. Generics
15. Exception Handling
16. Java 8+
17. Records / Sealed Classes
18. Modern Java Features

Collections deliberately comes only after this track.
```
