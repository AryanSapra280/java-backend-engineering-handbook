# Single Responsibility Principle (SRP)

# Part 3 - Engineering Application

> **Learning Goal:** Learn how the Single Responsibility Principle is applied in real-world software engineering. Understand how modern backend applications, Spring Boot, layered architectures, microservices, event-driven systems, and production software naturally follow SRP to improve maintainability, scalability, and testability.

---

# Introduction

So far, we've understood the motivation and theory behind SRP.

However, software engineering is not about memorizing principles.

It's about applying them.

One interesting fact is that many engineers use SRP every day without even realizing it.

Whenever we separate Controllers, Services, Repositories, Kafka Consumers, or Notification Services, we are applying the Single Responsibility Principle.

This chapter explores how SRP appears naturally in modern software systems.

---

# SRP in Layered Architecture

One of the most common examples of SRP is the Layered Architecture used in enterprise applications.

A typical Spring Boot application looks like this:

```text
               HTTP Request
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

Each layer has one responsibility.

| Layer | Responsibility |
|--------|---------------|
| Controller | Handle HTTP requests and responses |
| Service | Execute business logic |
| Repository | Access the database |
| Database | Store data |

Each layer changes for different reasons.

This is SRP in action.

---

# Example: E-Commerce Order Flow

Suppose a customer places an order.

Without SRP:

```text
OrderController

↓

Validate Request

↓

Calculate Discount

↓

Save Order

↓

Update Inventory

↓

Send Email

↓

Generate Invoice

↓

Return Response
```

The controller is doing everything.

It has multiple responsibilities.

---

After applying SRP:

```text
                OrderController
                        │
                        ▼
                OrderService
        ┌────────┼─────────┐
        ▼        ▼         ▼
InventoryService PaymentService NotificationService
        │
        ▼
OrderRepository
```

Every component now has a focused responsibility.

The controller only coordinates the request.

---

# SRP in Spring Boot

Spring Boot naturally encourages SRP through stereotypes.

## Controller

Responsible for:

- Receiving requests
- Validating input
- Returning responses

Not responsible for business logic.

---

## Service

Responsible for:

- Business rules
- Calculations
- Validation
- Workflow

Not responsible for persistence.

---

## Repository

Responsible for:

- CRUD operations
- Database interaction

Not responsible for business decisions.

---

## Entity

Responsible only for representing domain data.

Not responsible for:

- Sending emails
- Calling APIs
- Business workflows

---

# Example

Good Design

```text
CustomerController

↓

CustomerService

↓

CustomerRepository
```

Bad Design

```text
CustomerController

↓

Validate

↓

Save

↓

Send Email

↓

Generate PDF

↓

Update Cache

↓

Publish Kafka Event
```

Controllers should never become God Classes.

---

# SRP in Microservices

Microservices are essentially SRP applied at the service level.

Instead of assigning one responsibility to a class,

we assign one business capability to an entire service.

Example:

```text
Order Service

↓

Handles Orders

-------------------

Payment Service

↓

Handles Payments

-------------------

Inventory Service

↓

Handles Inventory

-------------------

Notification Service

↓

Handles Notifications
```

Each service has one responsibility.

This improves scalability and independent deployment.

---

# SRP in Event-Driven Architecture

Consider an Order Placed event.

```text
Order Created

        │

        ▼

Kafka Topic

        │

 ┌──────┼────────┐

 ▼      ▼        ▼

Inventory

Notification

Analytics
```

Each consumer performs one responsibility.

Inventory Consumer

↓

Update Stock

Notification Consumer

↓

Send Email

Analytics Consumer

↓

Generate Reports

Each consumer changes independently.

---

# SRP in Batch Processing

Imagine a monthly interest calculation job.

Instead of one batch class doing everything,

a well-designed batch application separates responsibilities.

```text
InterestReader

↓

InterestProcessor

↓

InterestWriter
```

Reader

↓

Reads data

Processor

↓

Calculates Interest

Writer

↓

Stores Results

This separation follows SRP.

---

# SRP in Your Project (EdgeVerve Example)

Consider the PF contribution processing workflow.

```text
Employer Upload

↓

Upload API

↓

Contribution Validator

↓

Contribution Processor

↓

Interest Calculator

↓

Ledger Updater

↓

Notification Service
```

Notice that every component has one responsibility.

Changing interest calculation does not affect notifications.

Changing notifications does not affect ledger updates.

Changing validation does not affect persistence.

This is exactly why enterprise applications remain maintainable.

---

# SRP and Design Patterns

Many design patterns help enforce SRP.

Examples include:

| Design Pattern | Responsibility |
|----------------|---------------|
| Strategy | Separate algorithms |
| Command | Separate operations |
| Factory | Separate object creation |
| Builder | Separate object construction |
| Observer | Separate event handling |
| Facade | Separate complex interactions |

SOLID and Design Patterns complement each other.

---

# SRP and Clean Architecture

Clean Architecture is heavily influenced by SRP.

Example:

```text
Presentation Layer

↓

Application Layer

↓

Domain Layer

↓

Infrastructure Layer
```

Each layer has a well-defined responsibility.

This improves maintainability and testability.

---

# Engineering Checklist

Ask yourself the following questions while designing software.

- Does this class have more than one reason to change?
- Is business logic mixed with infrastructure logic?
- Is this class becoming a God Class?
- Can I describe this class using one sentence?
- If requirements change, will only one responsibility be affected?
- Does this class depend on unrelated modules?

If any answer is "No", reconsider the design.

---

# Common SRP Violations in Enterprise Projects

Avoid these anti-patterns:

❌ Fat Controllers

❌ God Services

❌ Utility classes with unrelated methods

❌ Entities containing business logic

❌ Repository classes performing calculations

❌ One Kafka consumer handling multiple business events

❌ Batch jobs combining reading, processing, and writing logic

---

# Engineering Best Practices

- Keep Controllers thin.
- Keep Services focused.
- One business capability per service.
- One purpose per Kafka Consumer.
- One responsibility per Batch Component.
- Prefer composition over large utility classes.
- Refactor continuously as responsibilities evolve.

---

# Transition to Implementation

So far, we've seen **where** SRP appears in real software systems.

The next chapter focuses on **how** to implement SRP using Java and Spring Boot.

We'll start with a poorly designed implementation, identify the SRP violations, and progressively refactor it into a clean, modular, and production-ready solution.