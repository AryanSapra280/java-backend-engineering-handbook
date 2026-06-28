# Encapsulation

> **Learning Goal:** Understand why encapsulation exists, the engineering problem it solves, and how to design objects that protect their own business rules.

---

# Why do we need Encapsulation?

As software systems grow, multiple developers and services interact with the same objects.

If every object exposes its internal state directly, any code can modify it without validation, leading to invalid states and inconsistent business behavior.

Encapsulation solves this problem by ensuring that an object's state changes only through controlled operations that enforce business rules.

---

# Problem Statement

Consider a `BankAccount` object.

Without encapsulation:

```java
account.balance = -1000;
account.balance = 100000000;
```

Nothing prevents another developer from assigning invalid values.

Business rules are completely bypassed.

Instead, the object should expose meaningful behaviors:

```java
account.deposit(1000);
account.withdraw(500);
```

The object validates every operation before changing its state.

---

# Definition

Encapsulation is the process of bundling an object's state (data) and behavior (methods) together while protecting the object's internal state by allowing modifications only through controlled operations.

The primary goal of encapsulation is **not data hiding**.

The primary goal is **protecting business rules and maintaining object integrity.**

---

# Encapsulation vs Data Hiding

Data Hiding is the objective.

Encapsulation is one technique used to achieve that objective.

```text
Data Hiding
      │
      ▼
Encapsulation
      │
      ▼
Controlled Access to Object State
```

---

# Why "private + Getters/Setters" is NOT Enough

Many developers define encapsulation as:

* Make fields private.
* Generate getters and setters.

This is incomplete.

Example:

```java
private double balance;

public void setBalance(double balance) {
    this.balance = balance;
}
```

The following is still possible:

```java
account.setBalance(-1000);
```

The object's business rules remain unprotected.

Private fields alone do not guarantee good encapsulation.

---

# Good API Design

Instead of exposing setters:

```java
setBalance()

setStatus()
```

Expose business operations:

```java
deposit()

withdraw()

freeze()

activate()

close()

transfer()
```

Each operation validates business rules before modifying the object's state.

---

# Java Example

```java
public class BankAccount {

    private double balance;

    public void deposit(double amount) {
        // Validate amount
        // Update balance
    }

    public void withdraw(double amount) {
        // Validate balance
        // Validate account status
        // Update balance
    }
}
```

Notice that the balance cannot be modified directly.

---

# Spring Boot Perspective

Encapsulation is heavily used in enterprise applications.

Instead of exposing:

```java
user.setEmail(email);
```

A richer domain model prefers:

```java
user.changeEmail(email);
```

This allows the object to perform:

* Email format validation
* Duplicate email checks
* Audit logging
* Domain event publishing
* Email verification

The object protects its own consistency.

---

# Enterprise Example

Consider an Interest Calculation Service.

Instead of:

```java
interest.setAmount(amount);
```

Prefer:

```java
interest.applyInterest(rate);
```

The object validates:

* Interest rate
* Effective date
* Product rules
* Regulatory constraints

---

# Advantages

* Protects business rules.
* Prevents invalid object states.
* Reduces coupling.
* Improves maintainability.
* Makes objects responsible for their own data.
* Improves readability by exposing meaningful business operations.

---

# Common Mistakes

❌ Thinking encapsulation simply means making variables private.

❌ Generating getters and setters for every field.

❌ Allowing direct modification of business-critical data.

❌ Exposing mutable collections directly.

---

# Interview Questions

1. What is encapsulation?

2. Why does encapsulation exist?

3. Why are getters and setters not sufficient?

4. Should every field have a getter and setter?

5. How does encapsulation protect business rules?

6. Explain encapsulation using a banking example.

---

# Revision Notes

* Encapsulation bundles data and behavior together.
* The goal is to protect business rules.
* Objects should expose behaviors, not unnecessary state.
* Private fields are an implementation detail, not the definition of encapsulation.
* Good APIs express business operations rather than field modifications.

---

# Key Takeaway

> **Encapsulation ensures that an object's state changes only through controlled behaviors that enforce business rules.**
