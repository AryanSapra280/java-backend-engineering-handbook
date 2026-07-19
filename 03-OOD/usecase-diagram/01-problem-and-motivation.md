# Use Case Diagram

> **Learning Goal:** Learn how to analyze a software system from a business perspective by identifying actors, user goals, and system scope before designing classes or writing code.

---

# 1. Introduction

A **Use Case Diagram** is one of the most important UML diagrams used during the **Object-Oriented Analysis (OOA)** phase of software development.

Before writing a single line of code, software engineers must first understand:

- Who will use the system?
- What problems are they trying to solve?
- What functionality should the system provide?
- Where does the responsibility of the system begin and end?

A Use Case Diagram answers these questions by modeling the interaction between external users (actors) and the system.

Unlike Class Diagrams or Sequence Diagrams, a Use Case Diagram does **not** describe software implementation.

Instead, it describes the **business requirements** from the user's perspective.

Think of it as the blueprint that helps everyone—business analysts, architects, developers, testers, and stakeholders—agree on **what the system should do** before discussing **how it will do it**.

---

# 2. Why Does a Use Case Diagram Exist?

One of the biggest mistakes new developers make is jumping directly into implementation.

Imagine someone asks you to design an **Online Food Delivery Application**.

Many developers immediately start thinking about:

- Customer class
- Restaurant class
- Order class
- PaymentService
- Database tables

However, these are implementation details.

Before designing software, we must first understand the business.

Questions like:

- Who uses the application?
- What are they trying to accomplish?
- Which features are required?
- Which systems communicate with our application?
- What is inside our system?
- What is outside our system?

These questions cannot be answered using Java classes.

They are answered using a **Use Case Diagram**.

A Use Case Diagram forces engineers to think about the business before thinking about software.

---

# 3. First Principles

A common misconception is that a Use Case Diagram models software.

It does not.

It models **business interactions**.

The goal is to understand **what the system should accomplish**, not **how it will accomplish it**.

Think of software development as building a house.

You would never start by deciding where to place electrical wires.

Instead, you first ask:

- How many bedrooms are required?
- Where is the kitchen?
- Do we need a garage?
- Who will live in the house?

Only after understanding the requirements do you begin designing the structure.

Software follows exactly the same approach.

```
Business Requirements
        │
        ▼
   Use Case Diagram
        │
        ▼
   Domain Understanding
        │
        ▼
    Class Diagram
        │
        ▼
  Sequence Diagram
        │
        ▼
 Activity Diagram
        │
        ▼
 State Machine Diagram
        │
        ▼
 Java / Spring Boot Implementation
```

The better your understanding of the business, the better your software design.

---

# 4. Where Does It Fit in OOAD?

Object-Oriented Analysis and Design (OOAD) is usually divided into two major phases.

```
Object-Oriented Analysis (OOA)

Understand Business
        │
Identify Actors
        │
Identify User Goals
        │
Define System Scope
        │
Create Use Case Diagram

────────────────────────────────────

Object-Oriented Design (OOD)

Identify Classes
        │
Define Relationships
        │
Create Class Diagram
        │
Design Object Interactions
        │
Sequence Diagram
        │
Activity Diagram
        │
State Machine Diagram
        │
Implementation
```

Notice that **Use Case Diagrams belong entirely to the Analysis phase.**

They help us understand **what needs to be built**, not **how it should be implemented**.

---

# 5. UML Notation

Before learning the components, it is useful to understand the standard UML symbols.

| Component | UML Symbol | Purpose |
|------------|------------|----------|
| Actor | Stick Figure | Represents an external entity |
| Use Case | Oval | Represents a business goal |
| System Boundary | Rectangle | Defines system scope |
| Association | Solid Line | Actor interacts with a use case |
| Include | Dashed Arrow + `<<include>>` | Mandatory reusable behavior |
| Extend | Dashed Arrow + `<<extend>>` | Optional or conditional behavior |
| Generalization | Hollow Triangle Arrow | Inheritance between actors or use cases |

These symbols are standardized by UML and are recognized across the software industry.

---

# 6. Components of a Use Case Diagram

Every Use Case Diagram is built using a small set of components.

Understanding these components is enough to model almost any business system.

---

## 6.1 Actor

An **Actor** is any external entity that interacts with the system.

An actor is **outside** the system boundary.

An actor is **not** part of your software.

Actors can be:

- Human users
- External software systems
- Hardware devices
- Third-party services
- Government systems
- Banking systems

Examples:

- Customer
- Admin
- Restaurant
- Delivery Partner
- Payment Gateway
- Email Service
- SAP
- EPFO
- GPS Provider

### Important Rule

Actors are always **entities**, never actions.

✔ Correct

- Customer
- Payment Gateway
- Email Service

❌ Incorrect

- Make Payment
- Send Email
- Generate Invoice

Those are **use cases**, not actors.

---

## 6.2 Primary Actor

A **Primary Actor** initiates the interaction with the system.

Without the primary actor, the use case would never begin.

Examples:

- Customer
- Student
- Employee
- Employer
- Admin

Example:

```
Customer
    │
    ▼
Place Order
```

The customer starts the interaction.

---

## 6.3 Secondary Actor

A **Secondary Actor** supports the system during execution.

It usually provides a service that our application depends upon.

Examples:

- Payment Gateway
- SMS Service
- Email Service
- Notification Service
- Google Maps API
- SAP
- Kafka Consumer
- EPFO System

Example:

```
Customer
      │
      ▼
Place Order
      │
      ▼
Payment Gateway
```

The customer initiates the request.

The payment gateway only supports it.

---

## 6.4 Use Case

A **Use Case** represents a business goal that an actor wants to accomplish.

It answers the question:

> **What does the actor want from the system?**

Examples:

Library Management

- Borrow Book
- Return Book
- Search Book
- Reserve Book

Food Delivery

- Search Restaurant
- Browse Menu
- Place Order
- Cancel Order
- Track Delivery

Banking

- Deposit Money
- Withdraw Cash
- Transfer Funds
- Check Balance

### Naming Convention

Use Cases should almost always be written using **verb + noun**.

✔ Good

- Place Order
- Withdraw Cash
- Generate Statement
- Approve Loan

❌ Bad

- Order
- Money
- Statement
- Loan

A Use Case represents an action or goal—not an object.

---