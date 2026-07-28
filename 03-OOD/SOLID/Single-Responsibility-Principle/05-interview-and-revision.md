# Single Responsibility Principle (SRP)

# Part 5 - Interview & Revision

> **Learning Goal:** Reinforce your understanding of the Single Responsibility Principle through interview questions, revision notes, practical scenarios, common pitfalls, and engineering checklists. By the end of this chapter, you should be able to confidently explain, identify, and apply SRP in real-world software systems.

---

# 1. Quick Revision

## Definition

> **A class should have one, and only one, reason to change.**

Remember:

SRP does **not** mean:

- One method
- One feature
- One line of code

It means:

**One responsibility.**

---

## Core Idea

Ask yourself:

> **"Why does this class exist?"**

If the answer contains multiple unrelated responsibilities, the class probably violates SRP.

---

## One Responsibility vs One Method

❌ Wrong

```
One Class

↓

One Method
```

---

✅ Correct

```
One Class

↓

One Responsibility

↓

Many Related Methods
```

Example

```
InvoiceCalculator

↓

calculateSubtotal()

↓

calculateTax()

↓

calculateDiscount()

↓

calculateFinalAmount()
```

This class still follows SRP because every method contributes to invoice calculation.

---

# 2. Interview Questions

## Beginner Level

### What is the Single Responsibility Principle?

### Who introduced SRP?

### Why is SRP important?

### What is meant by "one reason to change"?

### Does SRP mean one method per class?

### Can a class contain multiple methods and still satisfy SRP?

### What problems does SRP solve?

---

## Intermediate Level

### How do you identify an SRP violation?

### What is cohesion?

### What is coupling?

### How does SRP improve maintainability?

### How does SRP improve testability?

### Explain SRP using a real-world example.

### Explain SRP using the Invoice example.

### Difference between SRP and Separation of Concerns?

---

## Advanced Level

### How is SRP applied in Spring Boot?

### How does SRP influence Microservices?

### Does every microservice represent SRP?

### Can SRP increase the number of classes?

### Is having many small classes always a good design?

### Explain SRP violations in enterprise applications.

### How would you refactor a God Class?

### Explain SRP in your current project.

---

# 3. Scenario-Based Questions

## Scenario 1

You find the following class:

```
Employee

↓

calculateSalary()

↓

saveEmployee()

↓

printPayslip()

↓

sendEmail()

↓

generateReport()
```

Question:

Identify the SRP violations.

How would you refactor it?

---

## Scenario 2

A Spring Boot Controller:

- Validates requests
- Calculates discounts
- Saves orders
- Sends emails
- Publishes Kafka events

Question:

Does it follow SRP?

What responsibilities should be extracted?

---

## Scenario 3

A Kafka Consumer:

- Updates inventory
- Sends notifications
- Generates reports

Question:

Would you keep everything inside one consumer?

Explain your reasoning.

---

## Scenario 4

A Service contains:

- Business Logic
- SQL Queries
- Email Logic
- PDF Generation

Question:

How would you redesign this module?

---

# 4. Common Misconceptions

## ❌ SRP means one method per class.

Incorrect.

A class can have many methods.

All methods should contribute to one responsibility.

---

## ❌ Smaller classes always satisfy SRP.

Incorrect.

A small class can still perform unrelated work.

---

## ❌ SRP increases unnecessary complexity.

Incorrect.

SRP often increases the number of classes but reduces overall system complexity.

---

## ❌ SRP only applies to classes.

Incorrect.

The same idea applies to:

- Packages
- Components
- Modules
- Microservices
- Teams

---

# 5. Code Smells That Suggest SRP Violations

Watch out for these warning signs:

- God Class
- Large Class
- Long Methods
- Feature Envy
- Shotgun Surgery
- Divergent Change
- Too Many Dependencies
- Low Cohesion

If you observe these, SRP may be violated.

---

# 6. Refactoring Checklist

Whenever you review a class, ask:

- Does this class have multiple reasons to change?
- Can I describe its purpose in one sentence?
- Does it mix business and infrastructure logic?
- Is it difficult to test?
- Does it frequently change?
- Does it depend on unrelated classes?
- Would another engineer immediately understand its purpose?

If several answers are **No**, consider refactoring.

---

# 7. Engineering Best Practices

- Prefer high cohesion.
- Prefer low coupling.
- Keep Controllers thin.
- Keep Services focused.
- Keep Repositories responsible only for persistence.
- Separate business logic from infrastructure.
- Refactor continuously.
- Avoid God Classes.
- Think in responsibilities, not methods.

---

# 8. Revision Cheat Sheet

```
SRP

↓

One Responsibility

↓

One Reason to Change

↓

High Cohesion

↓

Low Coupling

↓

Easy Testing

↓

Easy Maintenance

↓

Easy Extension
```

---

## Responsibility Identification

Ask:

```
Why does this class exist?
```

If you answer:

```
Calculate

Print

Save

Email
```

↓

Split the class.

---

## Refactoring Journey

```
Large Class

↓

Identify Responsibilities

↓

Separate Responsibilities

↓

Small Focused Classes

↓

Better Design
```

---

# 9. Relationship with Other SOLID Principles

SRP naturally supports the remaining SOLID principles.

```
SRP

↓

Smaller Classes

↓

OCP becomes easier

↓

LSP becomes safer

↓

ISP becomes natural

↓

DIP becomes simpler
```

Good responsibility separation makes every other SOLID principle easier to apply.

---

# 10. Practice Exercises

## Easy

Refactor:

- Invoice
- Employee
- Student

---

## Medium

Design using SRP:

- ATM
- Library Management System
- Parking Lot

---

## Advanced

Refactor an existing Spring Boot application by identifying:

- Fat Controllers
- God Services
- Large Repositories
- Utility Classes
- Kafka Consumers
- Batch Jobs

Explain why each refactoring improves maintainability.

---

# 11. Key Takeaways

- SRP is about **responsibility**, not the number of methods.
- A class should have **one reason to change**.
- High cohesion is the primary goal of SRP.
- SRP reduces maintenance cost, improves readability, and makes software easier to test.
- Modern backend architectures naturally apply SRP through layers, services, and modular design.
- Refactoring is the practical way to introduce SRP into existing codebases.
- Mastering SRP lays the foundation for the remaining SOLID principles.

---

# 12. What's Next?

In the next chapter, we'll explore the **Open-Closed Principle (OCP)**.

You'll learn how to design software that is:

- **Open for Extension**
- **Closed for Modification**

We'll see why constantly modifying existing code is risky and how abstractions, polymorphism, and design patterns allow us to add new features without changing stable, tested code.