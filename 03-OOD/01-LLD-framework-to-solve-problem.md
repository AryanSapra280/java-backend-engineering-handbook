# Low-Level Design (LLD) Framework

> **Purpose:** A repeatable framework for solving any Low-Level Design (LLD) problem in interviews or real-world software engineering. Instead of memorizing solutions, follow this process to derive the design systematically.

---

# The Engineering Mindset

Never start coding immediately.

Always think in this order:

```text
Understand the Problem
        ↓
Analyze the Domain
        ↓
Identify Objects
        ↓
Assign Responsibilities
        ↓
Define Relationships
        ↓
Apply OOP Principles
        ↓
Apply SOLID Principles
        ↓
Use Design Patterns (Only if Needed)
        ↓
Draw UML
        ↓
Implement in Java
        ↓
Review & Refactor
```

The biggest mistake candidates make is jumping directly to Java classes.

---

# Step 1 — Requirement Gathering

## Functional Requirements

Ask:

* What should the system do?
* Who are the actors?
* What actions can they perform?
* What are the expected outputs?

Example:

Library System

* Borrow Book
* Return Book
* Search Book
* Reserve Book

---

## Non-Functional Requirements

Ask:

* Expected scale?
* Performance expectations?
* Availability requirements?
* Security requirements?
* Concurrency concerns?
* Maintainability requirements?

Example:

Parking Lot

* Multiple entry gates
* Concurrent parking
* Fast slot allocation

---

## Clarify Assumptions

Never assume.

Ask questions like:

* Are multiple users allowed?
* Can an operation fail?
* What are the edge cases?
* Are there limits?
* What should happen during invalid operations?

---

# Step 2 — Identify Objects

Identify nouns from the problem statement.

Example:

Online Food Delivery

* Customer
* Restaurant
* Menu
* Order
* Rider
* Payment
* Address
* Notification

These become candidate classes.

---

# Step 3 — Define Responsibilities

For every object ask:

* Why does this object exist?
* What information does it own?
* What actions should it perform?
* What business rules belong to it?

Example:

Order

Responsibilities

* Place Order
* Cancel Order
* Calculate Total
* Track Status

Avoid creating "God Classes."

Every class should have a clear purpose.

---

# Step 4 — Identify Relationships

Ask:

## IS-A?

Use Inheritance.

Example:

```text
SavingsAccount IS-A BankAccount
```

---

## HAS-A?

Use Composition.

Example:

```text
Car HAS-A Engine
```

---

## USES-A?

Use Association/Dependency.

Example:

```text
OrderService USES PaymentService
```

---

# Step 5 — Apply OOP Principles

For every important class ask:

## Encapsulation

* Can someone put this object into an invalid state?
* Should I expose behavior instead of setters?

---

## Abstraction

* What should clients know?
* What implementation details can remain hidden?

---

## Inheritance

* Is there a genuine IS-A relationship?

---

## Polymorphism

* Will multiple implementations exist?
* Can I program to an abstraction?

---

# Step 6 — Apply SOLID Principles

Before implementation, verify:

### SRP

Does every class have one responsibility?

---

### OCP

Can new features be added without modifying existing code?

---

### LSP

Can every child safely replace its parent?

---

### ISP

Are interfaces small and focused?

---

### DIP

Are high-level modules depending on abstractions instead of implementations?

---

# Step 7 — Decide Whether Design Patterns Are Needed

Only introduce patterns if they solve a real problem.

Examples:

| Problem                             | Pattern   |
| ----------------------------------- | --------- |
| Multiple object creation strategies | Factory   |
| Multiple algorithms                 | Strategy  |
| Event notification                  | Observer  |
| Single shared instance              | Singleton |
| Complex object creation             | Builder   |

Never use patterns just because you know them.

---

# Step 8 — Draw UML

Before writing code, draw:

* Class Diagram
* Sequence Diagram
* Activity Diagram (if required)
* State Diagram (if useful)

Focus on communication before implementation.

---

# Step 9 — Implement in Java

Recommended order:

1. Enums
2. Interfaces
3. Abstract Classes
4. Domain Models
5. Services
6. Repositories
7. Controllers (if applicable)

Write behavior before optimization.

---

# Step 10 — Validate the Design

Ask yourself:

* Can another developer understand this easily?
* Are responsibilities well distributed?
* Is coupling low?
* Is cohesion high?
* Can new features be added easily?
* Is testing straightforward?

---

# LLD Interview Framework

When solving an interview problem:

## 1. Clarify Requirements

Never begin coding immediately.

---

## 2. Identify Actors

Who interacts with the system?

---

## 3. Identify Core Objects

Convert nouns into candidate classes.

---

## 4. Assign Responsibilities

Every class should own its own business rules.

---

## 5. Identify Relationships

IS-A

HAS-A

USES-A

---

## 6. Draw UML

Start simple.

Refine gradually.

---

## 7. Explain Design Decisions

Interviewers care more about **why** than **what**.

Explain:

* Why inheritance?
* Why composition?
* Why interface?
* Why this design pattern?

---

## 8. Discuss Future Scalability

Mention:

* Extensibility
* Maintainability
* Testability
* Future requirements

---

# Engineering Checklist

Before finalizing any design:

☐ Have I understood the requirements?

☐ Have I identified all major objects?

☐ Does every class have one responsibility?

☐ Are relationships correctly modeled?

☐ Am I exposing behavior instead of state?

☐ Am I programming to abstractions?

☐ Would composition be better than inheritance?

☐ Does this design follow SOLID?

☐ Is any design pattern genuinely required?

☐ Can this design evolve without major modifications?

---

# Common Mistakes

❌ Jumping directly into coding.

❌ Designing classes before understanding requirements.

❌ Creating unnecessary inheritance hierarchies.

❌ Using design patterns without a problem to solve.

❌ Ignoring object responsibilities.

❌ Depending on concrete implementations.

❌ Creating God Objects.

---

# Mental Model

```text
Requirements
        ↓
Objects
        ↓
Responsibilities
        ↓
Relationships
        ↓
OOP
        ↓
SOLID
        ↓
Design Patterns
        ↓
UML
        ↓
Java Implementation
```

---

# Golden Rules

1. Understand before designing.
2. Design before coding.
3. Model the business domain, not the database.
4. Prefer composition over inheritance.
5. Program to abstractions, not implementations.
6. Keep responsibilities focused.
7. Let objects protect their own state.
8. Introduce design patterns only when they solve a real problem.
9. Explain every design decision with a reason.
10. Optimize for maintainability before optimization.

---

# Final Takeaway

A strong LLD engineer does not memorize solutions.

A strong LLD engineer follows a consistent design process that transforms requirements into maintainable, extensible, and well-structured software.
