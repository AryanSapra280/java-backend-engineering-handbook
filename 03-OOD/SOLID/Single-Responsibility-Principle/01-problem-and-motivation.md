# Single Responsibility Principle (SRP)

# Part 1 - Problem & Motivation

> **Learning Goal:** Understand why the Single Responsibility Principle exists, what problems it solves, and how poor responsibility assignment leads to tightly coupled, fragile, and difficult-to-maintain software systems.

---

# Introduction

Imagine you've been asked to build a simple invoice system.

Initially, the requirement is straightforward.

> "Calculate the total amount for an invoice."

So you create a simple `Invoice` class.

```java
public class Invoice {

    public double calculateTotal() {
        // ...
    }

}
```

Everything looks clean.

A week later, another requirement arrives.

> "Print the invoice."

You add another method.

```java
public void printInvoice() {

}
```

Another week passes.

> "Save the invoice to the database."

You add another method.

```java
public void saveInvoice() {

}
```

Business keeps growing.

Soon your Invoice class also contains

```java
sendEmail();

exportPdf();

generateQRCode();

uploadInvoice();

sendNotification();

calculateGST();

calculateDiscount();

calculateRewardPoints();
```

The class continues growing.

Every new feature finds a place inside the same class because

> "It is related to invoices."

Months later, another developer opens this file.

They find a class containing hundreds of lines of code with multiple unrelated responsibilities.

Changing one feature risks breaking another.

Testing becomes difficult.

Understanding the code takes hours.

Welcome to one of the most common design problems in object-oriented software.

---

# The Real Problem

The problem is **not** that the class has many methods.

The problem is that the class has **multiple independent reasons to change**.

For example,

The Invoice class may change because:

- Tax calculation changes.
- Printing format changes.
- Database technology changes.
- Email template changes.
- PDF generation changes.
- Business discount rules change.

Notice something important.

Each of these changes originates from a **different stakeholder**.

Different teams.

Different business requirements.

Different release cycles.

Yet all changes require modifying the same class.

This is exactly what the Single Responsibility Principle tries to prevent.

---

# Understanding "Responsibility"

One of the biggest misconceptions about SRP is believing that

> A class should do only one thing.

That is **not** what SRP says.

Instead,

> A class should have **one responsibility**.

A responsibility represents a single reason for the class to change.

A class may perform multiple related operations as long as they all contribute toward fulfilling the same responsibility.

For example,

An Invoice class may contain

```java
calculateSubtotal();

calculateTax();

calculateDiscount();

calculateFinalAmount();
```

Although there are multiple methods,

they all contribute toward a single responsibility:

> **Invoice calculation.**

This follows SRP.

---

# Code Smell

One of the easiest ways to identify an SRP violation is by recognizing common code smells.

## God Class

A God Class tries to do everything.

Example

```text
Invoice

│

├── Calculate Price

├── Save Invoice

├── Print Invoice

├── Send Email

├── Generate PDF

├── Upload Invoice

├── Export CSV

├── Notify Customer
```

Symptoms

- Hundreds of lines of code
- Large number of dependencies
- Difficult to understand
- Difficult to test
- Frequently modified
- Merge conflicts
- High coupling

Whenever you encounter a God Class,

SRP is usually being violated.

---

# Real-Life Analogy

Imagine a restaurant.

One employee

- Takes orders
- Cooks food
- Washes dishes
- Collects payment
- Cleans tables
- Manages inventory

Would the restaurant function efficiently?

No.

Instead,

Responsibilities are divided.

```text
Waiter

↓

Take Orders

Chef

↓

Prepare Food

Cashier

↓

Collect Payment

Cleaner

↓

Maintain Cleanliness
```

Each role has one clear responsibility.

Software should be designed in the same way.

---

# Why Was SRP Introduced?

As software systems grow,

requirements continuously evolve.

Without proper responsibility separation,

every modification increases the risk of introducing bugs.

SRP was introduced to reduce this risk by encouraging classes that are focused, cohesive, and easier to understand.

Instead of creating large multi-purpose classes,

we create smaller classes with clearly defined responsibilities.

---

# Consequences of Violating SRP

Ignoring SRP often leads to:

- High coupling
- Low cohesion
- Difficult testing
- Frequent merge conflicts
- Large classes
- Difficult debugging
- Poor readability
- Slow feature development
- Fear of modifying existing code

As projects scale,

these problems become increasingly expensive.

---

# Benefits of Following SRP

Applying SRP leads to:

- Highly cohesive classes
- Easier maintenance
- Better readability
- Improved unit testing
- Lower coupling
- Better reusability
- Easier refactoring
- Cleaner architecture

Each class becomes responsible for one well-defined part of the system.

---

# Relationship with Clean Code

The Single Responsibility Principle is one of the foundations of Clean Code.

Many Clean Code practices naturally emerge from following SRP.

Examples include:

- Small classes
- Small methods
- High cohesion
- Clear naming
- Better readability
- Easier maintenance

SRP encourages writing software that is easier for humans to understand.

---

# Transition to Part 2

Now that we understand the motivation behind the Single Responsibility Principle,

the next step is understanding its formal definition, the meaning of "one reason to change," how responsibilities are identified, and how to design classes that correctly follow SRP.

In the next chapter, we'll explore the theory and design behind SRP in depth.