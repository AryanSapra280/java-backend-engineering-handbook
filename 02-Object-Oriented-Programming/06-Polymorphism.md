# Polymorphism

> **Learning Goal:** Understand why polymorphism exists, how it enables flexible and extensible software, and how it is used in Java, Spring Boot, and enterprise applications.

---

# Why do we need Polymorphism?

Imagine you are designing Amazon's payment system.

Initially, the application supports only Credit Card payments.

```java
CreditCardPayment payment = new CreditCardPayment();
payment.pay();
```

A few months later, the business adds:

* UPI
* PayPal
* Wallet
* Net Banking

Without polymorphism, the client code becomes:

```java
if(paymentType == CREDIT_CARD){
    ...
}
else if(paymentType == UPI){
    ...
}
else if(paymentType == WALLET){
    ...
}
```

Every new payment method forces modifications to existing code.

Engineers wanted a better approach.

Instead of changing the client every time, they wanted the implementation to decide the behavior.

That is why polymorphism exists.

---

# First Principles

Polymorphism allows us to write code against a common abstraction while letting different objects provide their own implementation.

Think of a TV Remote.

You press:

```
Volume Up
```

The remote does not care which TV brand it controls.

Sony, Samsung or LG—all respond differently to the same operation.

The operation remains the same.

The implementation changes.

That is polymorphism.

---

# Definition

Polymorphism is the ability of different objects to respond differently to the same operation through a common interface or parent type.

It answers the design question:

> **"How can I use different implementations through one common abstraction?"**

---

# Engineering Perspective

Polymorphism reduces coupling.

Instead of depending on concrete classes, the client depends on behavior.

```
Client

↓

PaymentService

↓

CardPayment
UPIPayment
WalletPayment
```

The client never changes when new implementations are introduced.

---

# Types of Polymorphism

Java supports two major forms.

## 1. Runtime (Dynamic) Polymorphism ⭐⭐⭐⭐⭐

Implemented using **Method Overriding**.

Decision happens during runtime.

---

## 2. Compile-Time (Static) Polymorphism

Implemented using **Method Overloading**.

Decision happens during compilation.

For interviews, runtime polymorphism is significantly more important.

---

# Runtime Polymorphism

Suppose we have:

```java
abstract class Animal {
    abstract void makeSound();
}
```

Implementations:

```java
class Dog extends Animal {

    @Override
    void makeSound() {
        System.out.println("Bark");
    }
}

class Cat extends Animal {

    @Override
    void makeSound() {
        System.out.println("Meow");
    }
}
```

Client:

```java
Animal animal = new Dog();

animal.makeSound();
```

Compile Time:

```
Animal
```

Runtime:

```
Dog
```

Dog's implementation executes.

This is Runtime Polymorphism.

---

# Method Overriding

Method overriding occurs when a child class provides its own implementation of a parent method.

Rules:

* Method name must be identical.
* Parameters must be identical.
* Return type must be compatible.
* Access level cannot become more restrictive.
* `private`, `static`, and `final` methods cannot be overridden.

Example:

```java
class Vehicle {

    void start() {
        System.out.println("Vehicle Started");
    }

}

class Car extends Vehicle {

    @Override
    void start() {
        System.out.println("Car Started");
    }

}
```

---

# Compile-Time Polymorphism

Achieved using Method Overloading.

Example:

```java
print(int number)

print(String text)

print(int a, int b)
```

The compiler decides which method to call.

---

# Method Overloading Rules

Methods may differ by:

* Number of parameters
* Parameter types
* Parameter order

Changing only the return type is **not** overloading.

Incorrect:

```java
calculate()
```

returns `int`

and

```java
calculate()
```

returns `double`

This does not compile.

---

# Runtime vs Compile-Time Polymorphism

| Runtime           | Compile-Time       |
| ----------------- | ------------------ |
| Method Overriding | Method Overloading |
| Runtime Decision  | Compiler Decision  |
| Dynamic Dispatch  | Static Binding     |
| More Flexible     | Better Performance |

---

# Dynamic Method Dispatch

This is one of the most important interview topics.

Consider:

```java
Animal animal = new Dog();

animal.makeSound();
```

The compiler checks:

* Does Animal declare `makeSound()`?

Yes.

During runtime:

The JVM checks the actual object.

```
Dog
```

Therefore:

```
Dog.makeSound()
```

is executed.

This runtime selection is called **Dynamic Method Dispatch**.

---

# Java Perspective

One of the most common examples:

```java
List<String> list = new ArrayList<>();
```

Why not?

```java
ArrayList<String> list = new ArrayList<>();
```

Because tomorrow we can write:

```java
List<String> list = new LinkedList<>();
```

The client code remains unchanged.

This is programming to an abstraction using polymorphism.

---

# Spring Boot Perspective

Dependency Injection relies heavily on polymorphism.

Example:

```java
@Autowired
private NotificationService notificationService;
```

Runtime may inject:

* EmailNotificationService
* SMSNotificationService
* SlackNotificationService

Your service simply calls:

```java
notificationService.send();
```

The implementation changes without changing client code.

---

# Enterprise Example

Suppose your banking application has:

```
InterestCalculator
```

Implementations:

* SavingsInterestCalculator
* LoanInterestCalculator
* PFInterestCalculator

Client:

```java
calculator.calculate();
```

The scheduler does not care which calculator it receives.

Each implementation performs its own calculation.

This is exactly how enterprise systems remain extensible.

---

# Example from My Current Project

In a Provident Fund system, different transaction processors may exist.

```
TransactionProcessor
```

Implementations:

* MonthlyContributionProcessor
* TransferInProcessor
* TransferOutProcessor
* InterestPostingProcessor

The scheduler invokes:

```java
processor.process();
```

Each processor executes different business logic while exposing the same operation.

---

# Relationship with Other OOP Principles

## Encapsulation

Protects object state.

---

## Abstraction

Defines what operations are available.

---

## Inheritance

Creates the relationship between objects.

---

## Polymorphism

Uses those related objects through one common type.

---

# Frequently Confused Concepts

## Method Overloading vs Method Overriding

| Overloading            | Overriding                   |
| ---------------------- | ---------------------------- |
| Same class             | Parent & Child               |
| Compile-Time           | Runtime                      |
| Parameters must differ | Parameters must remain same  |
| Improves readability   | Enables runtime polymorphism |

---

## Abstraction vs Polymorphism

| Abstraction           | Polymorphism                |
| --------------------- | --------------------------- |
| Defines the contract  | Uses the contract           |
| Hides implementation  | Chooses implementation      |
| Focuses on API design | Focuses on runtime behavior |

---

## Inheritance vs Polymorphism

| Inheritance              | Polymorphism                       |
| ------------------------ | ---------------------------------- |
| Establishes relationship | Uses relationship                  |
| "IS-A" hierarchy         | Same operation, different behavior |

---

# Common Mistakes

❌ Confusing method overloading with polymorphism during interviews.

❌ Depending on concrete classes instead of abstractions.

❌ Writing long `if-else` chains instead of using polymorphism.

❌ Forgetting that Java does not support user-defined operator overloading.

❌ Believing overriding changes method signatures.

---

# FAANG Interview Perspective

Interviewers are rarely interested in the textbook definition.

They want to know:

* Why polymorphism exists.
* How it reduces coupling.
* How Spring uses it.
* How Dependency Injection relies on it.
* How it supports the Open/Closed Principle.
* When polymorphism is preferable to conditional logic.

---

# Interview Questions & Answers

## 1. What is polymorphism?

**Answer**

Polymorphism is the ability to invoke the same operation on different objects through a common abstraction while allowing each object to provide its own implementation.

---

## 2. Why does polymorphism exist?

**Answer**

It allows client code to remain unchanged while new implementations are introduced, improving flexibility, extensibility, and maintainability.

---

## 3. What is runtime polymorphism?

**Answer**

Runtime polymorphism occurs when the JVM decides which overridden method to execute based on the actual object present during runtime.

---

## 4. Difference between overloading and overriding?

**Answer**

Overloading is resolved during compilation and requires different parameter lists.

Overriding is resolved during runtime and requires identical method signatures.

---

## 5. Why do Spring applications rely heavily on polymorphism?

**Answer**

Spring injects implementations through interfaces, allowing business logic to depend on abstractions instead of concrete implementations.

---

## 6. Why do we write?

```java
List<String> list = new ArrayList<>();
```

instead of

```java
ArrayList<String> list = new ArrayList<>();
```

**Answer**

The client depends on the `List` abstraction rather than a specific implementation. This allows the implementation to change without affecting client code.

---

## 7. Can static methods be overridden?

**Answer**

No.

Static methods belong to the class, not the object. They are hidden, not overridden.

---

## 8. Can private methods be overridden?

**Answer**

No.

Private methods are not inherited by child classes.

---

## 9. Is method overloading true polymorphism?

**Answer**

It is compile-time (static) polymorphism, but in Java interviews, runtime polymorphism through overriding is generally considered more important.

---

# Revision Notes

* Polymorphism means one operation, many implementations.
* Runtime polymorphism uses method overriding.
* Compile-time polymorphism uses method overloading.
* Runtime dispatch is decided by the JVM.
* Program to abstractions, not implementations.
* Dependency Injection heavily relies on polymorphism.
* Avoid long conditional statements when polymorphism models the problem naturally.

---

# Senior Engineer Takeaways

* Design APIs around abstractions.
* Depend on interfaces instead of implementations.
* Use polymorphism to eliminate large conditional blocks.
* Runtime polymorphism improves extensibility.
* Good polymorphic designs naturally support the Open/Closed Principle.

---

# Engineering Checklist

Before introducing polymorphism, ask:

☐ Do multiple implementations exist?

☐ Can the client depend on an abstraction instead of a concrete class?

☐ Am I replacing conditional logic with object-oriented behavior?

☐ Will new implementations be added in the future?

☐ Does polymorphism make the design easier to extend?

---

# Key Takeaways

* Abstraction defines **what** can be done.
* Inheritance builds the relationship.
* Polymorphism allows the same operation to behave differently based on the actual object.
* Modern Java and Spring applications rely heavily on runtime polymorphism.
* Polymorphism is one of the primary tools for building scalable, extensible software.
