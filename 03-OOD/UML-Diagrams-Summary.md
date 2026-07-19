# UML Design Flow: Use Case Diagram → Class Diagram → Sequence Diagram

## Introduction

When designing an object-oriented system, we don't start by writing code.

Instead, we progressively refine the design using different UML diagrams.

Each diagram answers a different question about the system.

The overall design flow looks like this:

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

# Step 1: Use Case Diagram

## Purpose

A **Use Case Diagram** captures the **functional requirements** of the system.

It answers the question:

> **What should the system do, and who will use it?**

---

## It Identifies

* Primary Actors
* Secondary Actors
* Functionalities (Use Cases)
* System Boundary
* User interactions with the system
* Include / Extend relationships (when applicable)

---

## ATM Example

```text
                +---------------------------+
Customer -----> | Withdraw Cash             |
                | Deposit Cash              |
                | Check Balance             |
Bank Server --> | Verify Transaction        |
                +---------------------------+
```

Notice that there are **no classes** here.

We are only discussing:

* Features
* Users
* Interactions

---

# Step 2: Class Diagram

## Purpose

Once we know the system's features, we identify the objects responsible for implementing them.

A **Class Diagram** answers:

> **Which classes are required to implement these use cases?**

---

## It Identifies

* Classes
* Attributes
* Methods
* Relationships
* Responsibilities

---

## ATM Example

From the "Withdraw Cash" use case, we identify the following classes:

```text
Customer

ATM

Transaction

Account

CashDispenser

Bank
```

Example class:

```text
+----------------------+
| ATM                  |
+----------------------+
| - id                 |
| - location           |
+----------------------+
| + withdrawCash()     |
| + depositCash()      |
| + checkBalance()     |
+----------------------+
```

Relationships:

```text
ATM --------> Transaction

ATM --------> Account

ATM --------> CashDispenser
```

The Class Diagram represents the **static structure** of the system.

---

# Step 3: Sequence Diagram

## Purpose

A Sequence Diagram explains how the classes collaborate to complete **one specific use case**.

It answers:

> **How do the identified classes interact to perform a feature?**

---

## ATM Withdrawal Flow

```text
Customer
    |
withdrawCash()
    |
    ▼
ATM
    |
createTransaction()
    |
    ▼
Transaction
    |
validate()
    |
    ▼
Account
    |
approved
    ▲
Transaction
    |
dispenseCash()
    |
    ▼
CashDispenser
    |
cashReleased
    ▲
ATM
    |
Display Success
    |
    ▼
Customer
```

Unlike the Class Diagram, the Sequence Diagram focuses on:

* Order of execution
* Method calls
* Message passing
* Runtime behavior

---

# How These Diagrams Connect

Suppose the requirement is:

> Customer wants to withdraw cash.

---

## Use Case Diagram

Defines the feature.

```text
Actor

Customer

↓

Withdraw Cash
```

Question answered:

> **What functionality should the system provide?**

---

## Class Diagram

Defines the objects responsible for implementing that feature.

```text
Customer

ATM

Transaction

Account

CashDispenser
```

Question answered:

> **Who is responsible for implementing the feature?**

---

## Sequence Diagram

Shows how those objects collaborate.

```text
Customer

↓

ATM

↓

Transaction

↓

Account

↓

CashDispenser

↓

Customer
```

Question answered:

> **How is the feature executed?**

---

# House Construction Analogy

Understanding UML becomes much easier with this analogy.

---

## Use Case Diagram

The client says:

"I want"

* Three bedrooms
* Two bathrooms
* One kitchen
* Parking

These are the **requirements**.

---

## Class Diagram

The architect designs the components.

```text
House

Room

Kitchen

Bathroom

Garage
```

These represent the structure.

---

## Sequence Diagram

Someone enters the house.

```text
Person

↓

Main Door

↓

Living Room

↓

Kitchen

↓

Bedroom
```

This represents the movement (behavior).

The structure of the house never changes.

Only the sequence of interactions changes.

---

# Complete UML Design Journey

```text
Business Requirement

Customer wants to withdraw cash

                │
                ▼

Use Case Diagram
─────────────────────────────

Actor

Customer

Use Case

Withdraw Cash

                │
                ▼

Class Diagram
─────────────────────────────

Customer

ATM

Transaction

Account

CashDispenser

                │
                ▼

Sequence Diagram
─────────────────────────────

Customer

↓

ATM

↓

Transaction

↓

Account

↓

CashDispenser

↓

Customer

                │
                ▼

Java Code
─────────────────────────────

atm.withdrawCash();

transaction.validate();

account.debit();

cashDispenser.dispense();
```

---

# Static vs Dynamic View

| Diagram          | View            | Purpose                             |
| ---------------- | --------------- | ----------------------------------- |
| Use Case Diagram | Functional View | Captures requirements               |
| Class Diagram    | Static View     | Defines structure and relationships |
| Sequence Diagram | Dynamic View    | Defines runtime interaction         |

---

# Key Differences

| Use Case Diagram          | Class Diagram            | Sequence Diagram     |
| ------------------------- | ------------------------ | -------------------- |
| Focuses on features       | Focuses on structure     | Focuses on behavior  |
| Shows actors              | Shows classes            | Shows objects        |
| Defines requirements      | Defines responsibilities | Defines message flow |
| No implementation details | Static design            | Runtime execution    |
| Starting point            | Design blueprint         | Execution blueprint  |

---

# Interview Answer

**Q. Explain the relationship between Use Case Diagram, Class Diagram, and Sequence Diagram.**

**Answer:**

A Use Case Diagram captures the functional requirements of the system by identifying the actors and the features they interact with. Based on these use cases, we identify the required classes, their responsibilities, attributes, methods, and relationships using a Class Diagram. Finally, for each important use case, we create a Sequence Diagram to show how those classes collaborate over time through message exchanges to accomplish that functionality. This progression takes us from requirements to design and finally to implementation.

---

# Key Takeaways

* Design begins with **requirements**, not code.
* **Use Case Diagram** answers: **What should the system do?**
* **Class Diagram** answers: **Which classes will implement those features?**
* **Sequence Diagram** answers: **How do those classes interact at runtime?**
* Together, these diagrams form a natural progression from **requirements → structure → behavior → implementation**.
