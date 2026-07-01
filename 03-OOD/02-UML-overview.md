# UML Overview

> **Learning Goal:** Understand what UML is, why it exists, where it fits into the software development lifecycle, and how it helps engineers communicate software designs before implementation.

---

# Why does UML exist?

Imagine joining a new engineering team.

A senior engineer starts explaining the architecture using only words.

After thirty minutes, you are still trying to visualize how the system works.

Now imagine the same engineer draws:

```text
Customer
     │
     ▼
Order API
     │
     ▼
Order Service
     │
 ┌───┴────────┐
 ▼            ▼
Inventory   Payment
```

Within a minute, the system becomes much easier to understand.

That is why UML exists.

---

# The Problem UML Solves

Large software systems become difficult to explain using only documentation or code.

Engineers need a common visual language that everyone can understand.

UML provides that language.

Instead of describing a system in paragraphs, engineers communicate using diagrams.

---

# What is UML?

**Unified Modeling Language (UML)** is a standardized visual language used to analyze, design, document, and communicate software systems.

It is **not** a programming language.

It is a design language.

---

# Where UML Fits in SDLC

Software is not built by immediately writing Java code.

A typical workflow looks like this:

```text
Business Requirement
        │
        ▼
Requirement Gathering
        │
        ▼
Object-Oriented Analysis (OOA)
        │
        ▼
Object-Oriented Design (OOD)
        │
        ▼
UML Modeling
        │
        ▼
Java Implementation
        │
        ▼
Testing
        │
        ▼
Deployment
        │
        ▼
Maintenance
```

UML acts as the blueprint before implementation.

---

# UML in Object-Oriented Design

OOA helps us understand the problem.

OOD helps us design the solution.

UML helps us communicate that solution.

Think of the flow as:

```text
Requirements
      │
      ▼
Identify Objects
      │
      ▼
Design Relationships
      │
      ▼
Draw UML
      │
      ▼
Implement in Java
```

---

# UML Building Blocks

UML consists of three major building blocks:

* Things
* Relationships
* Diagrams

For interview preparation, focus primarily on **Relationships** and **Diagrams**.

Theoretical classification is less important than knowing when and how to use each diagram.

---

# Structural vs Behavioral Modeling

UML diagrams can broadly be divided into two categories.

## Structural Modeling

Answers the question:

> **What exists in the system?**

Examples:

* Classes
* Interfaces
* Components
* Nodes
* Actors

Think of structural diagrams as describing the **nouns** of the system.

---

## Behavioral Modeling

Answers the question:

> **How does the system behave?**

Examples:

* Sequence Diagrams
* Activity Diagrams
* State Diagrams

Think of behavioral diagrams as describing the **verbs** of the system.

---

# Most Important UML Elements

## Class

Represents the blueprint of an object.

Contains:

* Attributes
* Methods

---

## Interface

Represents a contract.

Defines behavior without implementation.

---

## Actor

Represents an external entity interacting with the system.

Examples:

* Customer
* Admin
* Rider
* Librarian

---

## Use Case

Represents a user's goal.

Examples:

* Borrow Book
* Place Order
* Book Ticket

---

## Component

Represents a logical module.

Examples:

* Payment Service
* Inventory Service
* Notification Service

---

## Node

Represents physical infrastructure.

Examples:

* Application Server
* Database Server
* Kubernetes Pod

Nodes are more commonly used in High-Level Design than Low-Level Design.

---

# UML Diagrams You Must Master

For LLD interviews, focus on these four diagrams.

## 1. Class Diagram ⭐⭐⭐⭐⭐

Shows:

* Classes
* Attributes
* Methods
* Relationships

---

## 2. Use Case Diagram ⭐⭐⭐⭐☆

Shows:

* Actors
* System functionality

---

## 3. Sequence Diagram ⭐⭐⭐⭐⭐

Shows:

* Object interactions
* Message flow
* Execution order

---

## 4. Activity Diagram ⭐⭐⭐⭐☆

Shows:

* Workflow
* Business process
* Decision flow

---

# UML in Enterprise Applications

Before implementing a new feature, teams typically:

* Gather requirements
* Discuss possible designs
* Sketch UML diagrams on a whiteboard
* Review the design
* Begin implementation

UML serves as a communication tool rather than an implementation language.

---

# Mapping UML to Java

| UML         | Java                                     |
| ----------- | ---------------------------------------- |
| Class       | `class`                                  |
| Interface   | `interface`                              |
| Inheritance | `extends`                                |
| Realization | `implements`                             |
| Association | Object Reference                         |
| Composition | Member Object                            |
| Dependency  | Method Parameter / Constructor Injection |

---

# My Current Project Connection

In a Provident Fund application:

A **Class Diagram** may contain:

* Member
* Contribution
* Account
* Transaction

A **Sequence Diagram** may describe:

Employer → Kafka → Contribution Service → Validator → Database

Different diagrams answer different design questions.

---

# Common Mistakes

❌ Thinking UML is a programming language.

❌ Memorizing symbols without understanding their purpose.

❌ Using one diagram for every design problem.

❌ Skipping UML and jumping directly into code.

---

# Interview Questions

### What is UML?

UML is a standardized visual language used to model, communicate, and document software designs.

---

### Is UML a programming language?

No.

It is a visual modeling language.

---

### Why is UML important?

It improves communication, simplifies complex systems, and provides a common blueprint before implementation.

---

### Which UML diagrams are most important for LLD interviews?

* Class Diagram
* Sequence Diagram
* Use Case Diagram
* Activity Diagram

---

# Revision Notes

* UML is a visual design language.
* UML is used before implementation.
* Structural diagrams describe what exists.
* Behavioral diagrams describe how the system behaves.
* Master Class, Sequence, Use Case, and Activity diagrams.

---

# Engineering Checklist

Before writing code, ask yourself:

☐ Do I understand the requirements?

☐ Can I identify the major objects?

☐ Would drawing a UML diagram simplify the discussion?

☐ Which UML diagram best represents this problem?

☐ Have I communicated the design clearly before implementation?

---

# Key Takeaways

* UML is the blueprint of software.
* UML communicates ideas, not code.
* Different UML diagrams answer different design questions.
* Strong software engineers can explain systems visually before implementing them.
