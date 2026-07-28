# Open-Closed Principle (OCP)

# Part 3 - Engineering Application

> **Learning Goal:** Learn how the Open-Closed Principle is applied in modern software engineering. Understand how enterprise applications, Spring Boot, microservices, event-driven systems, and design patterns leverage OCP to build scalable, extensible, and maintainable software.

---

# Introduction

The Open-Closed Principle is much more than an interview question.

It is one of the most widely used principles in enterprise software development.

Whenever software allows you to add new functionality without modifying existing, stable code, OCP is being applied.

Modern frameworks like Spring Boot, Hibernate, Kafka, and many design patterns are heavily based on this principle.

As a backend engineer, you'll use OCP almost every day—often without consciously thinking about it.

---

# OCP in Layered Architecture

A typical backend application follows a layered architecture.

```text
                 Client
                    │
                    ▼
            Controller Layer
                    │
                    ▼
             Service Layer
                    │
                    ▼
          Repository Layer
                    │
                    ▼
                Database
```

Each layer has a clearly defined responsibility.

Suppose tomorrow the database changes from MySQL to PostgreSQL.

Ideally,

Only the Repository Layer changes.

Controllers and Services remain untouched.

The application is **extended** by changing the persistence implementation rather than modifying business logic.

---

# OCP in Spring Boot

Spring Boot naturally encourages OCP through dependency injection.

Consider an Order Service.

Instead of depending on a concrete implementation,

it depends on an abstraction.

```text
               OrderService
                     │
                     ▼
          PaymentProcessor
                (Interface)

       ▲          ▲          ▲

CardPayment  UpiPayment  PaypalPayment
```

Whenever a new payment provider is introduced,

we simply create another implementation.

```text
ApplePayProcessor
```

No changes are required inside `OrderService`.

---

# OCP Through Dependency Injection

Spring's Dependency Injection is one of the best examples of OCP.

Example:

```java
@Service
public class OrderService {

    private final PaymentProcessor paymentProcessor;

    public OrderService(PaymentProcessor paymentProcessor) {
        this.paymentProcessor = paymentProcessor;
    }

}
```

Notice something important.

`OrderService` never depends on:

- CardPayment
- UpiPayment
- PaypalPayment

Instead, it depends on the interface.

Spring injects the appropriate implementation at runtime.

This allows the application to grow without modifying existing business logic.

---

# OCP in Authentication Systems

Suppose an application initially supports:

- Username & Password

Business later requires:

- Google Login
- GitHub Login
- Microsoft Login
- SAML
- OAuth2

A poor design looks like:

```text
AuthenticationService

↓

if(Google)

↓

else if(GitHub)

↓

else if(Microsoft)

↓

else if(...)
```

Every new provider modifies existing code.

---

A better design uses OCP.

```text
AuthenticationProvider

            ▲

 ┌──────────┼──────────┐

 ▼          ▼          ▼

Google   GitHub   Microsoft
```

Adding another provider requires creating another implementation.

---

# OCP in Notification Systems

Imagine sending notifications.

Initially,

Only Email is supported.

Later,

Business wants:

- SMS
- WhatsApp
- Push Notification
- Slack
- Microsoft Teams

Poor design:

```text
NotificationService

↓

if(email)

↓

else if(sms)

↓

else if(push)
```

Better design:

```text
NotificationSender

          ▲

 ┌────────┼───────────┐

 ▼        ▼           ▼

Email    SMS      Push
```

Now the system grows through extension.

---

# OCP in Payment Gateways

Most payment systems support multiple providers.

Example:

```text
PaymentGateway

↓

Stripe

↓

Razorpay

↓

PayPal

↓

Adyen

↓

WorldPay
```

When another provider is added,

the gateway interface remains unchanged.

Only a new implementation is introduced.

---

# OCP in Cloud Storage

Many enterprise applications support different storage providers.

```text
StorageService

         ▲

 ┌───────┼────────┐

 ▼       ▼        ▼

AWS S3 Azure GCS
```

If tomorrow the organisation moves from AWS S3 to Azure,

the business logic remains unchanged.

Only another implementation is configured.

---

# OCP in Event-Driven Systems

Suppose an order is placed.

Instead of modifying the Order Service every time another business requirement appears,

publish an event.

```text
Order Service

↓

OrderCreated Event

↓

Kafka Topic

↓

Inventory Consumer

↓

Notification Consumer

↓

Analytics Consumer

↓

Fraud Detection Consumer
```

Need another business capability?

Create another consumer.

No changes to the Order Service.

---

# OCP in Microservices

Microservices naturally support OCP.

Example:

```text
Order Service

↓

Payment Service

↓

Inventory Service

↓

Notification Service
```

Suppose the business introduces

"Loyalty Points."

Instead of modifying Order Service,

create

```text
Loyalty Service
```

The architecture grows through extension.

---

# OCP in Rules Engines

Many financial systems use configurable rules.

Instead of writing:

```java
if(customerType == GOLD)

else if(customerType == SILVER)

else if(customerType == PLATINUM)
```

Enterprise systems define rules separately.

```text
DiscountRule

         ▲

 ┌───────┼──────────┐

 ▼       ▼          ▼

Gold  Silver  Platinum
```

Each rule becomes independently extensible.

---

# OCP and Design Patterns

Many GoF Design Patterns exist primarily to implement OCP.

| Pattern | How It Supports OCP |
|----------|---------------------|
| Strategy | Add new algorithms without changing existing code |
| Factory | Add new object types without modifying clients |
| Decorator | Add behaviour dynamically |
| Command | Add new commands without changing invokers |
| Template Method | Allow subclasses to customise behaviour |

When studying Design Patterns later, you'll notice that OCP is one of the major motivations behind them.

---

# OCP in Your Project (EdgeVerve Example)

Consider a contribution processing workflow.

Initially,

Only one contribution calculation algorithm exists.

Later,

Business introduces:

- Equal Share
- Percentage Based
- Fixed Amount
- Weighted Distribution

Instead of writing:

```text
if(equalShare)

else if(percentage)

else if(weighted)
```

A better design would look like:

```text
DistributionStrategy

          ▲

 ┌────────┼──────────┐

 ▼        ▼          ▼

Equal  Percentage  Weighted
```

Whenever another distribution algorithm is introduced,

a new implementation is added.

The existing engine remains unchanged.

This is a real-world example of OCP in financial software.

---

# Engineering Checklist

Ask yourself the following questions.

- Does adding a feature require modifying existing classes?
- Am I repeatedly adding new `if-else` conditions?
- Can I introduce new behaviour by creating another implementation?
- Does my code depend on abstractions?
- Are stable components protected from frequent changes?

If the answer is "No",

consider applying OCP.

---

# Common OCP Violations

Watch out for these anti-patterns.

❌ Large switch statements

❌ Long if-else chains

❌ Feature flags inside business logic

❌ Direct dependency on concrete implementations

❌ Repeated modifications to stable classes

❌ Monolithic services handling every variation

---

# Engineering Best Practices

- Program to interfaces, not implementations.
- Prefer composition over inheritance.
- Use dependency injection.
- Isolate business variations.
- Keep stable code untouched.
- Extend behaviour through new implementations.
- Avoid modifying well-tested components whenever possible.

---

# Transition to Implementation

Understanding where OCP appears in software engineering is only the beginning.

In the next chapter, we'll implement OCP using Java and Spring Boot.

We'll start with a poor implementation using `if-else` statements, identify the design issues, and progressively refactor it into a clean, extensible architecture using interfaces, polymorphism, and dependency injection.