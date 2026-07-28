# Single Responsibility Principle (SRP)

# Part 2 - Theory & Design

> **Learning Goal:** Develop a deep understanding of the Single Responsibility Principle by learning its formal definition, understanding what a "responsibility" really means, identifying SRP violations, and designing highly cohesive object-oriented systems.

---

# Introduction

In the previous chapter, we explored the motivation behind the Single Responsibility Principle.

We saw how software gradually becomes difficult to maintain when multiple unrelated responsibilities are placed inside a single class.

Now it's time to formally understand what SRP actually means.

Most developers memorize the statement:

> **"A class should have only one reason to change."**

Unfortunately, many stop there.

The real challenge is understanding:

- What is a responsibility?
- What is a reason to change?
- How do we identify responsibilities?
- How do we know whether a class violates SRP?

This chapter answers these questions.

---

# Formal Definition

The Single Responsibility Principle was introduced by **Robert C. Martin (Uncle Bob)**.

> **A class should have one, and only one, reason to change.**

Notice something important.

It does **not** say:

- One method
- One function
- One feature
- One line of business logic

It says:

> **One reason to change.**

Understanding this sentence is the key to mastering SRP.

---

# What is a Responsibility?

A responsibility is a **single area of concern** or **business responsibility** assigned to a class.

Think of responsibility as the answer to the question:

> **"Why does this class exist?"**

If the answer contains multiple unrelated reasons, the class probably violates SRP.

---

## Example

Consider an `Invoice` class.

Responsibilities may include:

- Calculate invoice total
- Print invoice
- Save invoice
- Send invoice email

Although all methods involve invoices, they represent different responsibilities.

Why?

Because each one changes for a different reason.

---

# Understanding "Reason to Change"

This is the most important concept in SRP.

A class should change only when **its primary responsibility changes**.

Consider this class:

```java
class Invoice {

    calculateTotal();

    printInvoice();

    saveInvoice();

}
```

Now imagine the following changes.

### Scenario 1

The government changes GST rules.

Which method changes?

```
calculateTotal()
```

---

### Scenario 2

The company changes invoice format.

Which method changes?

```
printInvoice()
```

---

### Scenario 3

The company migrates from MySQL to PostgreSQL.

Which method changes?

```
saveInvoice()
```

---

Notice something.

Each requirement comes from a completely different source.

Different teams.

Different business rules.

Different release cycles.

Yet they all modify the same class.

That is an SRP violation.

---

# Responsibility vs Functionality

One of the biggest misconceptions is:

> "A class should perform only one operation."

This is incorrect.

Consider:

```java
InvoiceCalculator

calculateSubtotal()

calculateTax()

calculateDiscount()

calculateGrandTotal()
```

There are multiple methods.

Yet every method contributes to a single responsibility:

**Invoice Calculation.**

Therefore, this class follows SRP.

---

# Cohesion

SRP is closely related to another important software design concept:

## Cohesion

Cohesion measures how closely related the responsibilities inside a class are.

### High Cohesion

All methods work together toward one responsibility.

```text
InvoiceCalculator

↓

Subtotal

↓

Tax

↓

Discount

↓

Final Amount
```

Everything belongs together.

---

### Low Cohesion

Methods perform unrelated work.

```text
Invoice

↓

Calculate

↓

Print

↓

Save

↓

Email

↓

Export PDF
```

These methods do not contribute to one responsibility.

The class has low cohesion.

---

# Coupling vs Cohesion

These two concepts are often confused.

## Cohesion

Relationship **inside** a class.

Question:

> Do the methods belong together?

Higher cohesion is desirable.

---

## Coupling

Relationship **between** classes.

Question:

> How dependent are different classes on one another?

Lower coupling is desirable.

A well-designed system generally has:

- High Cohesion
- Low Coupling

SRP primarily improves **cohesion** and indirectly reduces **coupling**.

---

# Identifying Responsibilities

When designing a class, ask yourself:

- Why does this class exist?
- Who requests changes to this class?
- What business capability does it represent?
- Which stakeholder owns this logic?

If you identify multiple independent answers, split the class.

---

# Refactoring Using SRP

Before

```text
Invoice

├── Calculate

├── Print

├── Save

├── Email
```

After

```text
InvoiceCalculator

↓

InvoicePrinter

↓

InvoiceRepository

↓

InvoiceEmailService
```

Now every class has one responsibility.

Each class changes independently.

---

# UML Representation

### Before Applying SRP

```text
+-------------------------+
| Invoice                 |
+-------------------------+
| calculateTotal()        |
| printInvoice()          |
| saveInvoice()           |
| sendEmail()             |
+-------------------------+
```

---

### After Applying SRP

```text
+----------------------+
| InvoiceCalculator    |
+----------------------+
| calculateTotal()     |
+----------------------+

        │

        ▼

+----------------------+
| InvoicePrinter       |
+----------------------+
| printInvoice()       |
+----------------------+

        │

        ▼

+----------------------+
| InvoiceRepository    |
+----------------------+
| saveInvoice()        |
+----------------------+

        │

        ▼

+----------------------+
| EmailService         |
+----------------------+
| sendInvoice()        |
+----------------------+
```

Notice how responsibilities become easier to understand.

---

# Advantages of SRP

Applying SRP provides several benefits.

- Easier maintenance
- Better readability
- Higher cohesion
- Easier testing
- Reduced merge conflicts
- Better code reuse
- Simpler debugging
- Cleaner architecture

---

# Common Misconceptions

## ❌ One Class = One Method

Incorrect.

A class can contain multiple methods.

The methods must contribute toward one responsibility.

---

## ❌ One Class = One Feature

Incorrect.

A feature often requires multiple collaborating classes.

---

## ❌ SRP Creates Too Many Classes

Sometimes.

Small focused classes are generally easier to maintain than one large God Class.

Quality is more important than reducing the number of files.

---

# SRP and Object-Oriented Design

SRP is one of the foundations of good object-oriented design.

It encourages engineers to think in terms of:

- Responsibilities
- Business capabilities
- Collaboration
- Separation of concerns

Instead of writing classes that "do everything."

---

# Transition to Engineering Application

Understanding SRP conceptually is only the beginning.

The next chapter explores how software engineers apply SRP in real-world backend systems, Spring Boot applications, layered architectures, microservices, and production environments.

We'll see how this principle influences the way professional software is designed every day.