# Single Responsibility Principle (SRP)

# Part 4 - Implementation

> **Learning Goal:** Learn how to identify SRP violations in existing code and refactor them into maintainable, loosely coupled, and production-ready software using Java, Spring Boot, and modern backend engineering practices.

---

# Introduction

Understanding the Single Responsibility Principle is valuable.

Applying it to real software is even more valuable.

In real projects, you rarely get the opportunity to design everything from scratch.

Instead, you inherit an existing codebase that has evolved over months or years. Features have been added rapidly, deadlines have been tight, and responsibilities have gradually accumulated inside the same class.

The goal of SRP implementation is not simply to write smaller classes.

The goal is to separate responsibilities so that each class has one clear purpose and one reason to change.

In this chapter, we'll start with a poorly designed implementation and progressively refactor it into a clean, production-ready design.

---

# Step 1 - The Bad Design

Suppose we're building a Book Invoice application.

A beginner might write something like this.

```java
public class Invoice {

    private Book book;
    private int quantity;

    public Invoice(Book book, int quantity) {
        this.book = book;
        this.quantity = quantity;
    }

    public double calculateTotal() {
        return book.getPrice() * quantity;
    }

    public void printInvoice() {
        System.out.println("Printing Invoice...");
    }

    public void saveToDatabase() {
        System.out.println("Saving Invoice...");
    }

    public void sendEmail() {
        System.out.println("Sending Email...");
    }
}
```

At first glance, this looks convenient.

Everything related to invoices is inside one class.

However, this design violates SRP.

---

# Step 2 - Identify the Responsibilities

Let's identify the different responsibilities.

```text
Invoice

│

├── Calculate Total

├── Print Invoice

├── Save Invoice

└── Send Email
```

Ask yourself:

Why would this class change?

Possible answers:

- Pricing rules change.
- Invoice layout changes.
- Database changes.
- Email provider changes.

These are independent reasons.

Therefore, the class has multiple responsibilities.

---

# Step 3 - Refactor the Design

Instead of one large class, separate the responsibilities.

```text
                 Invoice
                     │
         Stores Invoice Data
                     │

        ┌────────────┼─────────────┐

        ▼            ▼             ▼

InvoiceCalculator InvoicePrinter InvoiceRepository

                                      │

                                      ▼

                               EmailService
```

Each class now has a single responsibility.

---

# Step 4 - Refactored Java Implementation

## Invoice

```java
public class Invoice {

    private Book book;
    private int quantity;

    public Invoice(Book book, int quantity) {
        this.book = book;
        this.quantity = quantity;
    }

    public Book getBook() {
        return book;
    }

    public int getQuantity() {
        return quantity;
    }
}
```

---

## InvoiceCalculator

```java
public class InvoiceCalculator {

    public double calculateTotal(Invoice invoice) {
        return invoice.getBook().getPrice() * invoice.getQuantity();
    }

}
```

---

## InvoicePrinter

```java
public class InvoicePrinter {

    public void print(Invoice invoice) {
        System.out.println("Printing Invoice...");
    }

}
```

---

## InvoiceRepository

```java
public class InvoiceRepository {

    public void save(Invoice invoice) {
        System.out.println("Saving Invoice...");
    }

}
```

---

## EmailService

```java
public class EmailService {

    public void sendInvoice(Invoice invoice) {
        System.out.println("Sending Email...");
    }

}
```

Now each class has one reason to change.

---

# Step 5 - Spring Boot Implementation

SRP naturally appears in Spring Boot applications.

Example structure:

```text
CustomerController

↓

CustomerService

↓

CustomerRepository

↓

Database
```

Example:

```java
@RestController
@RequestMapping("/customers")
public class CustomerController {

    private final CustomerService customerService;

    @PostMapping
    public Customer createCustomer(@RequestBody CustomerRequest request) {
        return customerService.createCustomer(request);
    }

}
```

The Controller is responsible only for handling HTTP requests.

---

```java
@Service
public class CustomerService {

    public Customer createCustomer(CustomerRequest request) {

        // Business Logic

    }

}
```

The Service contains business rules.

---

```java
@Repository
public class CustomerRepository {

    public Customer save(Customer customer) {

        // Database Logic

    }

}
```

The Repository handles persistence.

Each layer has a single responsibility.

---

# Step 6 - SRP in Event-Driven Systems

Consider an order placement workflow.

Instead of one service performing every task,

publish an event.

```text
Order Service

↓

Order Created Event

↓

Kafka

↓

Inventory Consumer

↓

Notification Consumer

↓

Analytics Consumer
```

Each consumer owns one responsibility.

---

# Step 7 - SRP in Batch Processing

Spring Batch itself follows SRP.

```text
Reader

↓

Processor

↓

Writer
```

Reader

↓

Reads Data

Processor

↓

Transforms Data

Writer

↓

Persists Data

Each component changes independently.

---

# Step 8 - Production Example

Consider your PF contribution processing system.

```text
Employer Upload

↓

Upload API

↓

Validation Service

↓

Contribution Service

↓

Interest Calculation Service

↓

Ledger Service

↓

Notification Service
```

Each service owns a single business capability.

Imagine changing the interest calculation logic.

Only the Interest Calculation Service changes.

Everything else remains untouched.

That is SRP in production.

---

# Step 9 - Refactoring Checklist

Whenever you review a class, ask:

- Does this class have more than one reason to change?
- Does it communicate with unrelated systems?
- Does it mix business logic with infrastructure logic?
- Can another engineer describe its responsibility in one sentence?
- Is testing this class difficult because of unrelated dependencies?

If the answer is "Yes", consider refactoring.

---

# Common Refactoring Mistakes

❌ Creating one class per method.

❌ Splitting classes without understanding responsibilities.

❌ Moving business logic into Controllers.

❌ Creating Utility classes containing unrelated methods.

❌ Confusing SRP with "small classes."

Remember:

Small classes do not automatically satisfy SRP.

Focused responsibilities do.

---

# Key Takeaways

- Implementation begins by identifying responsibilities, not writing code.
- Refactoring is the primary way SRP is introduced into existing systems.
- Modern frameworks like Spring Boot naturally encourage SRP through layered architecture.
- Event-driven systems and batch processing also demonstrate SRP by assigning one responsibility to each component.
- The goal of SRP is not to reduce the number of lines of code but to reduce the number of reasons a class needs to change.

---

# What's Next?

Now that you've learned how to identify and implement SRP, the final chapter focuses on interview preparation.

We'll cover:

- Frequently asked interview questions
- Real-world scenarios
- Common misconceptions
- Revision cheat sheet
- Practice exercises
- Design challenges