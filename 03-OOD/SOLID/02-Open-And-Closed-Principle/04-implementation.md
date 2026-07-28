# Open-Closed Principle (OCP)

# Part 4 - Implementation

> **Learning Goal:** Learn how to implement the Open-Closed Principle in Java and Spring Boot by progressively refactoring rigid, modification-heavy code into an extensible architecture using interfaces, polymorphism, dependency injection, and design patterns.

---

# Introduction

Understanding OCP is only the first step.

The real challenge is implementing it in production software.

Most legacy systems begin with simple conditional statements.

As business requirements grow, these conditionals become larger and more difficult to maintain.

In this chapter, we'll gradually refactor such a system into a scalable, production-ready design.

---

# Step 1 - The Initial Design

Suppose our application initially supports only Card payments.

```java
public class PaymentProcessor {

    public void processPayment(String paymentType) {

        if ("CARD".equals(paymentType)) {
            System.out.println("Processing Card Payment");
        }

    }

}
```

This implementation works.

At this stage, there is no immediate problem.

---

# Step 2 - Business Growth

Business now supports:

- Card
- UPI
- PayPal

The implementation becomes:

```java
public class PaymentProcessor {

    public void processPayment(String paymentType) {

        if ("CARD".equals(paymentType)) {

            System.out.println("Card Payment");

        } else if ("UPI".equals(paymentType)) {

            System.out.println("UPI Payment");

        } else if ("PAYPAL".equals(paymentType)) {

            System.out.println("PayPal Payment");

        }

    }

}
```

The application still works.

However, every new payment method requires modifying this class.

---

# Problems with the Current Design

Adding a new payment provider means:

- Modifying tested code
- Re-running regression tests
- Increasing merge conflicts
- Growing if-else chains
- Violating OCP

The system is **closed for extension** and **open for modification**, which is exactly the opposite of what we want.

---

# Step 3 - Introduce an Abstraction

Create a common contract.

```java
public interface PaymentProcessor {

    void processPayment();

}
```

Now every payment method implements the same interface.

---

## Card Payment

```java
public class CardPayment implements PaymentProcessor {

    @Override
    public void processPayment() {
        System.out.println("Processing Card Payment");
    }

}
```

---

## UPI Payment

```java
public class UpiPayment implements PaymentProcessor {

    @Override
    public void processPayment() {
        System.out.println("Processing UPI Payment");
    }

}
```

---

## PayPal Payment

```java
public class PaypalPayment implements PaymentProcessor {

    @Override
    public void processPayment() {
        System.out.println("Processing PayPal Payment");
    }

}
```

Notice that every new payment provider is simply another implementation.

No existing implementation changes.

---

# Step 4 - Client Code

Instead of depending on concrete implementations,

the client depends on the abstraction.

```java
public class CheckoutService {

    private final PaymentProcessor paymentProcessor;

    public CheckoutService(PaymentProcessor paymentProcessor) {
        this.paymentProcessor = paymentProcessor;
    }

    public void checkout() {
        paymentProcessor.processPayment();
    }

}
```

This is the essence of OCP.

The client never changes when new payment methods are introduced.

---

# Step 5 - Spring Boot Implementation

Spring Boot makes OCP extremely natural.

## Interface

```java
public interface PaymentProcessor {

    void processPayment();

}
```

---

## Implementation

```java
@Service
public class CardPaymentProcessor implements PaymentProcessor {

    @Override
    public void processPayment() {

        System.out.println("Card Payment");

    }

}
```

---

## Another Implementation

```java
@Service
public class UpiPaymentProcessor implements PaymentProcessor {

    @Override
    public void processPayment() {

        System.out.println("UPI Payment");

    }

}
```

---

## Service Layer

```java
@Service
public class CheckoutService {

    private final PaymentProcessor paymentProcessor;

    public CheckoutService(PaymentProcessor paymentProcessor) {
        this.paymentProcessor = paymentProcessor;
    }

    public void checkout() {
        paymentProcessor.processPayment();
    }

}
```

Spring injects the required implementation.

Business logic never changes.

---

# Step 6 - Strategy Pattern

This implementation is actually the Strategy Pattern.

```text
Checkout Service

↓

PaymentProcessor

        ▲

 ┌──────┼───────────┐

 ▼      ▼           ▼

Card   UPI      PayPal
```

Each payment algorithm is a strategy.

Adding another strategy requires creating another implementation.

No modification.

Only extension.

---

# Step 7 - Factory Pattern

Suppose users choose a payment method dynamically.

Instead of using if-else inside the service,

use a factory.

```text
Checkout Service

↓

Payment Factory

        │

        ▼

PaymentProcessor
```

The factory decides which implementation to return.

The checkout logic remains unchanged.

---

# Step 8 - Production Example

Consider a notification system.

Initially,

Only Email exists.

```text
NotificationService

↓

Email
```

Business later requires:

- SMS
- WhatsApp
- Slack
- Push Notifications

Instead of modifying `NotificationService` repeatedly,

design:

```text
NotificationSender

         ▲

 ┌───────┼──────────┐

 ▼       ▼          ▼

Email   SMS     WhatsApp
```

The service now supports unlimited notification providers.

---

# Step 9 - OCP in Spring Framework

Spring itself follows OCP extensively.

Examples include:

- Dependency Injection
- BeanFactory
- ApplicationContext
- Spring Security AuthenticationProvider
- Spring Data Repository
- Message Converters
- View Resolvers

Instead of modifying the framework,

developers extend it by providing new implementations.

This is why Spring is highly extensible.

---

# Step 10 - Refactoring Journey

```text
Business Requirement

↓

if-else

↓

Growing Conditionals

↓

Introduce Interface

↓

Implement Polymorphism

↓

Dependency Injection

↓

Strategy Pattern

↓

Factory Pattern

↓

Production Ready Design
```

---

# Refactoring Checklist

When reviewing code, ask yourself:

- Am I modifying existing classes repeatedly?
- Am I adding more if-else statements?
- Can this variation become an interface?
- Can polymorphism replace this condition?
- Can dependency injection remove this dependency?
- Am I protecting stable code?

If several answers are "No",

consider applying OCP.

---

# Common Implementation Mistakes

❌ Creating unnecessary abstractions too early.

❌ Using inheritance when composition is sufficient.

❌ Replacing every if-else with interfaces.

❌ Over-engineering simple applications.

❌ Violating SRP while trying to satisfy OCP.

Remember:

OCP is a design principle, not a rule that must be applied everywhere.

Apply it where future extension is likely.

---

# Key Takeaways

- OCP encourages extension instead of modification.
- Interfaces and polymorphism are the primary implementation techniques.
- Spring Boot naturally supports OCP through dependency injection.
- Strategy and Factory are the most common design patterns implementing OCP.
- The objective is not to eliminate changes but to protect stable code from unnecessary modifications.
- Well-designed systems evolve by adding new implementations rather than editing existing ones.

---

# What's Next?

Now that you've implemented OCP using Java, Spring Boot, interfaces, and design patterns, the final chapter focuses on interview preparation.

We'll cover:

- Frequently asked interview questions
- Common misconceptions
- Scenario-based problems
- OCP cheat sheet
- Revision notes
- Engineering checklist