# Use Case Diagram

> **Learning Goal:** Learn how to analyze a system from a business perspective by identifying actors, user goals, and system scope before designing classes or writing code.

---

# Why does a Use Case Diagram exist?

One of the biggest mistakes engineers make is jumping directly into implementation.

Suppose someone asks you to design an Online Food Delivery application.

Many developers immediately think about:

* Customer class
* Restaurant class
* Order class

However, before designing software, we must understand the business.

Questions like:

* Who will use this system?
* What are they trying to accomplish?
* What functionality should the system support?

A Use Case Diagram answers these questions.

---

# First Principles

A Use Case Diagram does **not** model software.

It models **business interactions**.

It helps engineers understand the problem before thinking about implementation.

Think of it as the bridge between requirements and design.

```text
Business Requirements
        │
        ▼
Use Case Diagram
        │
        ▼
Class Diagram
        │
        ▼
Sequence Diagram
        │
        ▼
Java Implementation
```

---

# What is a Use Case Diagram?

A Use Case Diagram is a high-level UML diagram that describes:

* Who interacts with the system (Actors)
* What they want to achieve (Use Cases)

It focuses entirely on the external behavior of the system.

It intentionally hides implementation details.

---

# Where Does It Fit in OOAD?

Object-Oriented Analysis

↓

Understand business requirements

↓

Identify actors

↓

Identify use cases

↓

Define system scope

↓

Object-Oriented Design

↓

Identify classes

↓

Relationships

↓

Implementation

Use Case Diagrams belong to the **Analysis** phase.

---

# Components of a Use Case Diagram

## 1. Actor

An Actor is any external entity that interacts with the system.

Actors are **outside** the system boundary.

Actors can be:

* Human users
* External systems
* Hardware devices
* Third-party services

Examples:

* Customer
* Admin
* Restaurant
* Delivery Partner
* Payment Gateway
* Email Service
* SAP System

---

## Primary Actor

A primary actor initiates the interaction.

Examples:

* Customer
* Employee
* Admin
* Employer

---

## Secondary Actor

A secondary actor supports the system.

Examples:

* Payment Gateway
* Email Service
* SMS Service
* GPS Service
* SAP

Remember:

Actors are **entities**, not actions.

Incorrect:

❌ Make Payment

Correct:

✅ Payment Gateway

---

## 2. Use Case

A Use Case represents a business goal.

Examples:

Library

* Borrow Book
* Return Book
* Search Book

Online Food Delivery

* Search Restaurant
* Place Order
* Cancel Order
* Track Order

Use Cases are typically written as **verbs** because they describe actions.

---

## 3. System Boundary

The system boundary defines what belongs to the system.

Everything inside the boundary is the responsibility of the system.

Everything outside the boundary is an actor.

```text
           Customer

               │

    +----------------------+

    |   Food Delivery App  |

    |  Search Restaurant   |

    |  Place Order         |

    |  Track Order         |

    +----------------------+
```

---

# Relationships

## Association

Represents interaction between an actor and a use case.

Example:

Customer

↓

Place Order

---

## Include (<<include>>)

Represents mandatory reusable functionality.

Question to ask:

> **Does this always happen?**

If yes → Include.

Examples:

Transfer Money

↓

Validate Account

---

Place Order

↓

Calculate Total

---

Login

↓

Validate Credentials

---

## Extend (<<extend>>)

Represents optional behavior.

Question to ask:

> **Does this happen only under certain conditions?**

If yes → Extend.

Examples:

Checkout

↓

Apply Coupon

---

Withdraw Cash

↓

Print Receipt

---

Cancel Order

↓

Refund (depends on business rules)

---

## Generalization

Represents inheritance between use cases.

Example:

Payment

↓

Pay via Savings

↓

Pay via Current Account

---

# Framework for Solving Any Use Case Diagram

## Step 1

Understand the business problem.

Do not think about Java.

---

## Step 2

Identify all actors.

Ask:

Who interacts with the system?

---

## Step 3

Separate actors into:

* Primary Actors
* Secondary Actors

---

## Step 4

Identify user goals.

Every goal becomes a Use Case.

---

## Step 5

Draw the system boundary.

Keep actors outside.

Keep use cases inside.

---

## Step 6

Identify relationships.

Ask:

Does this always happen?

→ <<include>>

Does this happen only sometimes?

→ <<extend>>

---

## Step 7

Review the scope.

Ensure every actor has at least one interaction.

---

# Mapping to Java

Use Case Diagrams do not map directly to Java.

They help discover:

* Business requirements
* Candidate classes
* APIs
* Services

Java implementation comes later.

---

# Enterprise Example

Online Food Delivery

Primary Actors

* Customer
* Restaurant
* Delivery Partner
* Admin

Secondary Actors

* Payment Gateway
* Email Service
* Maps Service
* Notification Service

Use Cases

* Search Restaurant
* Browse Menu
* Place Order
* Make Payment
* Cancel Order
* Track Delivery
* Deliver Order
* Request Refund

---

# My Current Project

Primary Actors

* Employer
* Employee
* Admin

Secondary Actors

* Kafka
* SAP
* Notification Service

Use Cases

* Upload Contribution
* Process Contribution
* Withdraw PF
* Settlement
* Generate Statement
* Retry Failed Upload

---

# Common Mistakes

❌ Confusing actors with actions.

❌ Drawing classes instead of use cases.

❌ Showing database tables.

❌ Adding implementation details.

❌ Thinking about Java too early.

❌ Using <<include>> and <<extend>> incorrectly.

---

# Interview Questions

### What is a Use Case Diagram?

A UML diagram that models user interactions and business goals from the user's perspective.

---

### Why is a Use Case Diagram created before a Class Diagram?

Because we must understand the business requirements before designing software.

---

### Difference between Actor and Object?

Actor:

External entity interacting with the system.

Object:

Internal software entity.

---

### Difference between <<include>> and <<extend>>?

Include:

Mandatory behavior.

Extend:

Optional or conditional behavior.

---

### Can another software system be an actor?

Yes.

Any external system interacting with the application is an actor.

---

# Revision Notes

* Think business before software.
* Actors are external entities.
* Use Cases are business goals.
* Use Case Diagrams belong to Object-Oriented Analysis.
* Include = Mandatory.
* Extend = Optional.
* Identify actors before identifying classes.

---

# Engineering Checklist

Before drawing a Use Case Diagram ask:

☐ Who uses the system?

☐ What are their goals?

☐ Have I identified all external systems?

☐ Are my actors entities rather than actions?

☐ Are my use cases written as actions?

☐ Have I correctly identified Include and Extend relationships?

☐ Does the diagram clearly define the system scope?

---

# Key Takeaways

A Use Case Diagram is not about classes or code.

It is about understanding the business.

Good software design always begins with understanding the people and systems that interact with your application.
