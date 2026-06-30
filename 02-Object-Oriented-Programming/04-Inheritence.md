# Inheritance

> **Learning Goal:** Understand why inheritance exists, when to use it, when to avoid it, and how to identify correct inheritance relationships in object-oriented design.

---

# Why do we need Inheritance?

As software systems grow, multiple classes often share common attributes and behaviors.

Without inheritance, developers duplicate code across similar classes, making systems harder to maintain and extend.

Inheritance allows common behavior to be defined once in a parent class and reused by specialized child classes.

However, inheritance should only be used when there is a genuine **IS-A** relationship.

---

# Problem Statement

Imagine designing different types of vehicles.

Without inheritance:

* Car has `start()`, `stop()`, `engineNumber`
* Bike has `start()`, `stop()`, `engineNumber`
* Truck has `start()`, `stop()`, `engineNumber`

The same behavior is duplicated.

Instead:

```text
             Vehicle
        ----------------
        start()
        stop()
        engineNumber

          /      |      \
       Car     Bike    Truck
```

Each child class reuses the common behavior while adding its own specialization.

---

# Definition

Inheritance is an OOP mechanism that allows one class to extend another class, inheriting its accessible state and behavior while specializing it for more specific requirements.

Inheritance models an **IS-A** relationship.

---

# The IS-A Relationship

Inheritance should only be used when the child is a specialized version of the parent.

### Good Examples

* Dog IS AN Animal
* Car IS A Vehicle
* Circle IS A Shape
* SavingsAccount IS A BankAccount

### Incorrect Examples

* Engine IS A Car ❌
* Customer IS A Bank ❌
* Keyboard IS A Laptop ❌
* Book IS A Library ❌

These are **HAS-A** relationships, not inheritance.

---

# HAS-A vs IS-A

## IS-A (Inheritance)

```text
Dog
   ↓
Animal
```

A Dog is an Animal.

---

## HAS-A (Composition)

```text
Car
   ↓
Engine
```

A Car has an Engine.

Composition models collaboration, not classification.

---

# Java Example

```java
public class Vehicle {

    public void start() {
    }

    public void stop() {
    }
}

public class Car extends Vehicle {

    public void openSunroof() {
    }
}
```

`Car` automatically inherits the common behavior from `Vehicle`.

---

# Types of Inheritance

## Single Inheritance

One child extends one parent.

```
Vehicle
   ↓
Car
```

Supported in Java.

---

## Multilevel Inheritance

```
Vehicle
   ↓
FuelVehicle
   ↓
Car
```

Supported in Java.

---

## Hierarchical Inheritance

```
        Vehicle
      /    |    \
    Car   Bike  Truck
```

Supported in Java.

---

## Multiple Inheritance

A child inherits from multiple parent classes.

Java **does not support** multiple inheritance through classes because of the **Diamond Problem**.

Java achieves similar behavior using interfaces.

---

## Hybrid Inheritance

Combination of multiple inheritance types.

Achieved in Java through interfaces.

---

# Why Java Does Not Support Multiple Inheritance

Suppose:

```java
class A {
    void print() {}
}

class B {
    void print() {}
}
```

If Java allowed:

```java
class C extends A, B
```

Which `print()` should be executed?

This ambiguity is known as the **Diamond Problem**.

To avoid this ambiguity, Java allows only single inheritance for classes.

---

# Enterprise Example

In banking systems:

```
Account
   ↓
SavingsAccount

Account
   ↓
CurrentAccount

Account
   ↓
SalaryAccount
```

Each specialized account inherits common account behavior.

However:

```
Customer
   ↓
Account
```

is incorrect.

A Customer **has** Accounts.

---

# Spring Boot Perspective

Spring uses inheritance where specialization naturally exists.

Example:

```
RuntimeException
        ↓
DataAccessException
        ↓
DuplicateKeyException
```

Each child exception specializes the parent.

However, most business components in Spring use **composition** instead of inheritance.

---

# Advantages

* Promotes code reuse.
* Eliminates duplicate implementations.
* Encourages specialization.
* Simplifies maintenance.
* Makes common behavior centralized.

---

# Disadvantages

* Creates tight coupling between parent and child.
* Incorrect hierarchies become difficult to change.
* Deep inheritance trees reduce readability.
* Changes in parent classes may affect all children.

---

# Favor Composition over Inheritance

Modern software design generally prefers **composition** unless inheritance is the most natural model.

Instead of:

```
LoanService extends InterestCalculationService
```

Use:

```
LoanService
      │
      ▼
uses
      │
InterestCalculationService
```

This reduces coupling and improves flexibility.

---

# Frequently Confused Concepts

## IS-A vs HAS-A

| IS-A             | HAS-A             |
| ---------------- | ----------------- |
| Inheritance      | Composition       |
| Classification   | Collaboration     |
| Dog is an Animal | Car has an Engine |

---

## Composition vs Inheritance

| Composition                | Inheritance                          |
| -------------------------- | ------------------------------------ |
| Flexible                   | Tightly Coupled                      |
| Runtime collaboration      | Compile-time hierarchy               |
| Preferred in modern design | Use only for true IS-A relationships |

---

# Engineering Decision Framework

Before using inheritance, ask:

* Is this truly an **IS-A** relationship?
* Can the child safely replace the parent?
* Am I using inheritance only for code reuse?
* Would composition solve the problem better?
* Will this hierarchy still make sense if requirements change?

If any answer raises concern, reconsider the design.

---

# Interview Questions and Answers

### 1. What is inheritance?

**Answer:**

Inheritance is an OOP mechanism that allows a child class to inherit accessible state and behavior from a parent class, enabling code reuse and specialization through a true IS-A relationship.

---

### 2. What is an IS-A relationship?

**Answer:**

An IS-A relationship means the child is a specialized form of the parent and can logically replace the parent.

Example:

* Car is a Vehicle.
* Dog is an Animal.

---

### 3. Difference between IS-A and HAS-A?

**Answer:**

IS-A models inheritance.

HAS-A models composition.

Use inheritance for specialization and composition for collaboration.

---

### 4. Why doesn't Java support multiple inheritance?

**Answer:**

Because multiple inheritance through classes introduces ambiguity (Diamond Problem). Java avoids this by allowing only one parent class while supporting multiple interfaces.

---

### 5. Why is composition preferred over inheritance?

**Answer:**

Composition reduces coupling, improves flexibility, allows runtime collaboration, and avoids incorrect inheritance hierarchies.

---

### 6. Should inheritance be used only for code reuse?

**Answer:**

No.

Code reuse alone is not a sufficient reason.

Inheritance should model a true IS-A relationship.

---

### 7. Give one real-world example of inheritance.

**Answer:**

SavingsAccount extends BankAccount because a SavingsAccount is a specialized type of BankAccount.

---

# Common Mistakes

❌ Using inheritance only to reuse code.

❌ Confusing IS-A with HAS-A.

❌ Creating deep inheritance hierarchies.

❌ Ignoring composition as an alternative.

❌ Violating the Liskov Substitution Principle.

---

# Revision Notes

* Inheritance models IS-A relationships.
* Use inheritance only for specialization.
* HAS-A relationships should use composition.
* Java supports single, multilevel, and hierarchical inheritance.
* Java avoids multiple inheritance of classes because of the Diamond Problem.
* Prefer composition unless inheritance is the natural choice.

---

# Key Takeaways

* Inheritance classifies related objects.
* Composition connects collaborating objects.
* Code reuse is a benefit of inheritance—not its primary purpose.
* Always validate the relationship before introducing inheritance.

---

# Engineering Checklist

Before using inheritance:

☐ Is this a true IS-A relationship?

☐ Can the child replace the parent safely?

☐ Am I inheriting because of behavior rather than code reuse?

☐ Would composition provide a cleaner solution?

☐ Will this hierarchy remain valid as requirements evolve?
