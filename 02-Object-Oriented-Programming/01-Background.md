# Background of Object-Oriented Programming (OOP)

> **Learning Goal:** Understand why Object-Oriented Programming exists, what problems it solves, and how classes and objects model real-world systems.

---

# Why do we need Object-Oriented Programming?

Programming exists to solve real-world problems.

As software systems became larger, organizing code as a collection of functions became difficult.

Problems with procedural programming:

* Difficult to maintain large codebases
* Poor ownership of data
* Tight coupling between functions
* Difficult to extend existing features
* Hard to model real-world entities

Object-Oriented Programming solves these problems by organizing software around **objects** instead of functions.

An object combines:

* **State (Data)**
* **Behavior (Actions)**

This makes software closer to how humans naturally think about the real world.

---

# Definition

Object-Oriented Programming (OOP) is a programming paradigm that models software using **objects**.

Each object represents a real-world entity and encapsulates:

* State (Attributes)
* Behavior (Methods)

Objects collaborate with one another to solve business problems.

---

# Real-World Analogy

Imagine a Bank.

Instead of thinking:

* deposit()
* withdraw()
* transfer()

We think about:

* Customer
* Account
* Loan
* Employee

Each object owns its own responsibilities.

For example:

**Account**

State

* Account Number
* Balance
* Status

Behavior

* Deposit
* Withdraw
* Freeze
* Close

This makes the design easier to understand and maintain.

---

# Building Blocks of OOP

## 1. Class

A class is a **blueprint** used to create objects.

Example:

```
Employee
---------
id
name
salary

work()
takeLeave()
```

A class does not represent a real employee.

It only defines what every employee should look like.

---

## 2. Object

An object is an **instance of a class**.

Example

```
Employee emp1

id = 101

name = Aryan

salary = 100000
```

Multiple objects can be created from the same class.

---

## 3. Attributes

Attributes represent the **state** of an object.

Example

```
Account

balance

status

accountType
```

The values of attributes change during the object's lifetime.

---

## 4. Methods

Methods represent the **behavior** of an object.

Example

```
deposit()

withdraw()

freeze()

close()
```

Methods operate on the object's state.

---

# State vs Behavior

Example: Car

### State

* Color
* Fuel
* Speed
* Gear

### Behavior

* Start
* Stop
* Accelerate
* Brake

An object is incomplete without both state and behavior.

---

# Why Objects Instead of Functions?

Procedural Programming

```
withdraw(account)

deposit(account)

freeze(account)
```

Object-Oriented Programming

```
account.withdraw()

account.deposit()

account.freeze()
```

In OOP, the object owns the behavior.

This improves encapsulation and makes the design easier to maintain.

---

# Class vs Object

| Class                          | Object                   |
| ------------------------------ | ------------------------ |
| Blueprint                      | Real instance            |
| Defines structure              | Represents a real entity |
| No object-specific state       | Has its own state        |
| One class creates many objects | Created from a class     |

---

# Java Example

```java
public class BankAccount {

    private String accountNumber;
    private double balance;

    public void deposit(double amount) {
        balance += amount;
    }

    public void withdraw(double amount) {
        balance -= amount;
    }
}
```

---

# Real-World Example in Spring Boot

Every Spring Bean is an object.

Examples

* OrderService
* PaymentService
* UserRepository
* NotificationService

Although we don't create them using `new`, the Spring container creates and manages these objects.

---

# Interview Perspective

Common Questions

1. What is Object-Oriented Programming?
2. Why does OOP exist?
3. Difference between Class and Object?
4. Difference between Attributes and Methods?
5. Why should behavior belong inside an object?

---

# Common Mistakes

❌ Thinking a class is just a collection of fields.

❌ Treating objects as database tables.

❌ Ignoring object behavior.

❌ Starting implementation before understanding the domain.

---

# What I Learned

* OOP models software using real-world entities.
* Every object has state and behavior.
* A class is a blueprint; an object is an instance of that blueprint.
* Methods should operate on an object's own state.
* Good object-oriented design starts with identifying responsibilities before writing code.

---

# Revision Notes

* OOP organizes software around objects.
* Objects contain state and behavior.
* Classes are blueprints used to create objects.
* Attributes represent state.
* Methods represent behavior.
* Objects collaborate to solve business problems.
* Focus on responsibilities before implementation.

---

# Key Takeaway

> **An object is not a bag of data. It is a real-world entity that owns its state, behavior, and business rules.**
