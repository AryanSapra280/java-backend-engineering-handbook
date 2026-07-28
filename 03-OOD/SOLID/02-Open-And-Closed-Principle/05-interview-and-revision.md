# Open-Closed Principle (OCP)

# Part 5 - Interview & Revision

> **Learning Goal:** Master the Open-Closed Principle from an interview perspective. Learn how to identify OCP violations, answer conceptual and scenario-based questions, recognise real-world applications, and revise the principle quickly before technical interviews.

---

# 1. Quick Revision

## Definition

> **Software entities should be open for extension but closed for modification.**

Coined by:

**Bertrand Meyer (1988)**

Popularized as part of SOLID by:

**Robert C. Martin (Uncle Bob)**

---

## Core Idea

Instead of changing existing code whenever a new requirement arrives,

**extend the system by adding new implementations.**

Think:

```text
Old Behaviour

        │

        ▼

Keep Stable

        │

        ▼

Add New Behaviour
```

---

## Open vs Closed

### Closed for Modification

Stable and tested code should remain unchanged.

Avoid repeatedly modifying:

- Classes
- Methods
- Modules
- Components

---

### Open for Extension

New behaviour should be introduced through:

- New Classes
- New Implementations
- New Modules
- New Services

without changing existing code.

---

# 2. Mental Model

Whenever a business requirement changes,

do **not** ask:

> Which class should I modify?

Instead ask:

> Which new implementation should I create?

That single shift in thinking is the essence of OCP.

---

# 3. Before vs After OCP

## Without OCP

```text
PaymentService

↓

if(Card)

↓

else if(UPI)

↓

else if(PayPal)

↓

else if(...)
```

Every feature requires modifying existing code.

---

## With OCP

```text
                PaymentProcessor

                       ▲

        ┌──────────────┼──────────────┐

        ▼              ▼              ▼

      Card           UPI          PayPal
```

Adding another payment method requires only another implementation.

---

# 4. Interview Questions

## Beginner Level

### What is the Open-Closed Principle?

---

### Who introduced OCP?

---

### What does "Open for Extension" mean?

---

### What does "Closed for Modification" mean?

---

### Why is OCP important?

---

### Does OCP eliminate modifications completely?

---

### Does OCP always require inheritance?

---

## Intermediate Level

### How does polymorphism support OCP?

### Explain OCP using a Payment System.

### Explain OCP using Spring Boot.

### Difference between SRP and OCP.

### Explain OCP using a Notification System.

### How do interfaces help implement OCP?

### Why are long if-else chains considered an OCP violation?

---

## Advanced Level

### Explain OCP in Microservices.

### Explain OCP in Kafka.

### Explain OCP in your current project.

### Which Spring Framework components follow OCP?

### How do Strategy and Factory Patterns support OCP?

### Can overusing OCP become a problem?

### How would you refactor a legacy application violating OCP?

---

# 5. Scenario-Based Questions

## Scenario 1

You have:

```text
DiscountService

↓

if(GOLD)

↓

else if(SILVER)

↓

else if(PLATINUM)
```

Business introduces:

DIAMOND customers.

Question:

How would you redesign the system using OCP?

---

## Scenario 2

A NotificationService contains:

- Email
- SMS
- WhatsApp
- Push Notification

implemented using one large switch statement.

Question:

Identify the OCP violation.

How would you redesign it?

---

## Scenario 3

An AuthenticationService supports:

- Username
- Google
- GitHub
- Microsoft

using multiple if statements.

Question:

Would interfaces improve the design?

Explain.

---

## Scenario 4

Your Spring Boot application supports only AWS S3.

Business wants Azure Blob Storage.

Question:

Would you modify StorageService?

Or create another implementation?

Explain.

---

# 6. Common Misconceptions

## ❌ OCP means code never changes.

Incorrect.

Stable components change less frequently.

New behaviour is introduced through extension.

---

## ❌ OCP always uses inheritance.

Incorrect.

Modern Java prefers:

- Interfaces
- Composition
- Dependency Injection

---

## ❌ Every if statement violates OCP.

Incorrect.

Simple conditions are acceptable.

Repeated business variations usually indicate an opportunity for OCP.

---

## ❌ OCP should be applied everywhere.

Incorrect.

Applying OCP unnecessarily leads to over-engineering.

Use it where future extension is likely.

---

# 7. Code Smells Indicating OCP Violations

Watch for:

- Long if-else chains
- Large switch statements
- Frequent modifications to the same class
- Hardcoded business rules
- Tight coupling with concrete implementations
- Feature flags scattered throughout business logic

---

# 8. Engineering Best Practices

- Program to interfaces, not implementations.
- Prefer composition over inheritance.
- Use dependency injection.
- Keep business variations isolated.
- Avoid editing stable classes.
- Refactor conditionals into polymorphism when appropriate.
- Design for expected extension points.

---

# 9. OCP in Real Projects

Examples where OCP naturally appears:

## Payment Systems

```text
PaymentProcessor

↓

Card

↓

UPI

↓

PayPal

↓

Apple Pay
```

---

## Notification Systems

```text
NotificationSender

↓

Email

↓

SMS

↓

Push

↓

WhatsApp
```

---

## Authentication Providers

```text
AuthenticationProvider

↓

Google

↓

GitHub

↓

Microsoft

↓

SAML
```

---

## Cloud Storage

```text
StorageProvider

↓

AWS S3

↓

Azure Blob

↓

Google Cloud Storage
```

---

## Distribution Engine (Your Project)

```text
DistributionStrategy

↓

Equal Share

↓

Percentage

↓

Weighted

↓

Future Strategies...
```

Instead of modifying the engine,

new algorithms are added independently.

---

# 10. OCP and Other SOLID Principles

```text
SRP

↓

Smaller Focused Classes

↓

OCP

↓

Easy Extension

↓

LSP

↓

Reliable Polymorphism

↓

ISP

↓

Smaller Interfaces

↓

DIP

↓

Depend on Abstractions
```

Notice that the SOLID principles reinforce one another.

Good OCP is difficult without good SRP and DIP.

---

# 11. OCP and Design Patterns

| Design Pattern | Why It Uses OCP |
|---------------|-----------------|
| Strategy | Add new algorithms without modifying clients |
| Factory | Create new object types without changing callers |
| Decorator | Extend behaviour dynamically |
| Command | Add new commands without changing invokers |
| Template Method | Allow subclasses to customise specific steps |
| Chain of Responsibility | Add new handlers without modifying existing chain |

Understanding OCP makes these patterns much easier to understand.

---

# 12. Revision Cheat Sheet

```
Business Change

↓

Don't Modify

↓

Extend

↓

Abstraction

↓

Interface

↓

Polymorphism

↓

Dependency Injection

↓

Strategy Pattern

↓

Factory Pattern

↓

Production Ready Design
```

---

## OCP Decision Tree

```text
New Requirement?

       │

       ▼

Can I create a new implementation?

       │

 ┌─────┴─────┐

 │           │

Yes         No

 │           │

 ▼           ▼

Use OCP   Consider Refactoring
```

---

# 13. Practice Exercises

## Easy

Apply OCP to:

- Shape Calculator
- Payment Processor
- Tax Calculator

---

## Medium

Design using OCP:

- Notification System
- Authentication System
- Shipping Provider

---

## Advanced

Refactor a Spring Boot application by replacing:

- switch statements
- if-else chains
- hardcoded business rules

with interfaces, dependency injection, and polymorphism.

Explain why your design is more extensible.

---

# 14. Key Takeaways

- OCP encourages extending software rather than modifying existing, stable code.
- Interfaces and abstractions are the foundation of OCP.
- Polymorphism enables interchangeable behaviour.
- Spring Boot naturally supports OCP through dependency injection.
- Strategy and Factory are common design patterns implementing OCP.
- Apply OCP where future variations are expected—not everywhere.
- The goal is to minimise risk while allowing software to evolve.

---

# 15. What's Next?

In the next chapter, we'll study the **Liskov Substitution Principle (LSP)**.

You'll learn one of the most misunderstood SOLID principles:

> **Can every subclass truly replace its parent without breaking the application?**

We'll explore behavioural contracts, inheritance correctness, design-by-contract concepts, and real-world Java examples that explain why simply using `extends` is not enough.