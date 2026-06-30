# Generalization

> **Learning Goal:** Understand how to identify common characteristics among multiple classes and extract them into a shared parent abstraction to improve code reuse, maintainability, and extensibility.

---

# Why do we need Generalization?

As software systems grow, different classes often begin to share common attributes and behaviors.

Without generalization, these common features are duplicated across multiple classes, increasing maintenance effort and the likelihood of inconsistencies.

Generalization solves this by identifying common behavior and moving it into a common parent class.

---

# Definition

Generalization is the design process of identifying common attributes and behaviors among multiple classes and extracting them into a more general parent class.

It helps create cleaner object models by reducing duplication and encouraging reuse.

---

# First Principles

Imagine designing a zoo management system.

Initially, you create:

* Dog
* Cat
* Horse

After some time, you realize all of them can:

* eat()
* sleep()
* breathe()

Instead of repeating these methods in every class, you introduce a parent class.

```text
             Animal

      eat()

      sleep()

      breathe()

        ▲

   ┌────┴────┐

 Dog   Cat   Horse
```

The process of discovering these common characteristics and moving them into `Animal` is called **Generalization**.

---

# Generalization vs Specialization

Generalization and Specialization are opposite design activities.

### Generalization

Moves upward by identifying common behavior.

```text
Dog

Cat

Horse

      ↓

Animal
```

---

### Specialization

Moves downward by creating more specific classes.

```text
Animal

      ↓

Dog

      ↓

GoldenRetriever
```

---

# Generalization vs Inheritance

These concepts are closely related but not identical.

| Generalization               | Inheritance                          |
| ---------------------------- | ------------------------------------ |
| Design activity              | Programming mechanism                |
| Finds common characteristics | Implements parent-child relationship |
| Happens during design        | Happens during implementation        |
| Answers "What is common?"    | Uses `extends` in Java               |

Generalization is the thought process.

Inheritance is how Java implements that design.

---

# Enterprise Example

Consider a banking system.

Initially, separate transaction classes exist:

* MonthlyContribution
* TransferIn
* TransferOut

Each class contains:

* transactionId
* memberId
* amount
* process()

Instead of duplicating these members, we create:

```text
            Transaction

      transactionId

      memberId

      amount

      process()

           ▲

 ┌─────────┼─────────┐

MonthlyContribution

TransferIn

TransferOut
```

This is generalization.

---

# Relationship with Polymorphism

Generalization enables polymorphism.

Example:

```java
Shape shape;

shape = new Circle();

shape = new Rectangle();
```

Since `Circle` and `Rectangle` were generalized into `Shape`, one reference can represent multiple concrete objects.

---

# Relationship with the Open/Closed Principle

Generalization supports the Open/Closed Principle.

Suppose:

```text
Vehicle

↓

Car

Truck
```

Tomorrow a new requirement arrives:

```
Bike
```

We simply create:

```text
Bike extends Vehicle
```

No existing classes need modification.

The system becomes open for extension and closed for modification.

---

# Advantages

* Reduces duplicate code.
* Improves maintainability.
* Encourages reuse.
* Simplifies future extensions.
* Naturally supports polymorphism.

---

# Common Mistakes

❌ Thinking Generalization and Inheritance are the same concept.

❌ Creating parent classes without identifying true common behavior.

❌ Generalizing too early before understanding the domain.

❌ Building inheritance hierarchies only to reuse code.

---

# Interview Questions and Answers

### 1. What is Generalization?

**Answer:**

Generalization is the design process of identifying common characteristics among multiple classes and extracting them into a common parent abstraction.

---

### 2. What is the difference between Generalization and Inheritance?

**Answer:**

Generalization is the design concept of finding commonality, while inheritance is the programming mechanism used to implement that relationship.

---

### 3. How does Generalization support Polymorphism?

**Answer:**

By introducing a common parent type, different child objects can be treated uniformly through the parent reference.

---

### 4. How does Generalization support the Open/Closed Principle?

**Answer:**

New subclasses can be added without modifying existing parent or sibling classes, making the system easier to extend.

---

# Revision Notes

* Generalization identifies common behavior.
* Inheritance implements generalization.
* Specialization is the opposite of generalization.
* Generalization promotes reuse, maintainability, and extensibility.
* Good generalization creates meaningful parent abstractions.

---

# Key Takeaways

* Generalization is a design activity.
* Inheritance is a Java implementation mechanism.
* Always identify common behavior before introducing a parent class.
* Generalization naturally enables polymorphism.

---

# Engineering Checklist

Before creating a parent class, ask:

☐ Do these classes genuinely share common behavior?

☐ Am I reducing duplication or creating unnecessary abstraction?

☐ Will future subclasses naturally fit into this hierarchy?

☐ Does the parent represent a meaningful business concept?

☐ Am I introducing a parent only for code reuse?
