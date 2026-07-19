# Part 5 - Interview & Revision

> **Learning Goal:** Consolidate everything learned in this chapter and prepare to confidently discuss, design, and apply Use Case Diagrams in interviews, system design discussions, and real-world software projects.

---

# 1. Interview Expectations

Interviewers generally do not expect you to memorize UML syntax.

Instead, they evaluate whether you can:

- Understand business requirements.
- Identify the system's actors.
- Discover the major business capabilities.
- Model the system correctly.
- Differentiate between Include and Extend relationships.
- Explain your design decisions.
- Connect requirements to software design and implementation.

A good Use Case Diagram demonstrates your ability to think from a business perspective before writing code.

---

# 2. Frequently Asked Interview Questions

## Beginner Level

### What is a Use Case Diagram?

### Why do we use Use Case Diagrams?

### What are Actors?

### What is a Use Case?

### What is a System Boundary?

### What is the purpose of Include?

### What is the purpose of Extend?

### Difference between Include and Extend?

### What is Generalization?

### Difference between Functional and Non-Functional Requirements?

---

## Intermediate Level

### When should you create a Use Case Diagram?

### Can one Actor participate in multiple Use Cases?

### Can multiple Actors use the same Use Case?

### Can a Use Case call another Use Case?

### When should you split a Use Case Diagram?

### What are common mistakes while drawing Use Case Diagrams?

### Can a Use Case Diagram generate Java classes?

### How does a Use Case Diagram help architects?

### How does a Use Case influence API design?

### How does a Use Case influence database design?

---

## Advanced Level

### Design a Use Case Diagram for:

- Online Food Delivery
- ATM System
- BookMyShow
- WhatsApp
- Netflix
- Amazon
- Uber
- Parking Lot
- Hospital Management System
- Library Management System

Interviewers may ask you to explain your assumptions and justify your modeling decisions.

---

# 3. Scenario-Based Questions

Try solving these without looking at the answers.

## Scenario 1

Design a Use Case Diagram for an Online Shopping System.

Questions:

- Who are the actors?
- What are the primary Use Cases?
- Which Use Cases are mandatory?
- Which Use Cases are optional?

---

## Scenario 2

Design a Use Case Diagram for an Online Banking Application.

Identify:

- Customers
- Bank Employees
- Payment Gateway
- Notification Service
- Administrator

---

## Scenario 3

Design a Use Case Diagram for a Movie Ticket Booking System.

Think about:

- Booking Tickets
- Cancelling Tickets
- Making Payments
- Applying Coupons
- Viewing Booking History

---

# 4. Common Mistakes

Avoid the following mistakes:

- Designing screens instead of business goals.
- Naming Use Cases using nouns instead of verbs.
- Including implementation details.
- Forgetting the System Boundary.
- Using Include instead of Extend.
- Ignoring external systems.
- Creating one huge diagram for the entire application.
- Treating Actors as classes or database tables.
- Jumping directly to Class Diagrams without understanding requirements.

---

# 5. Best Practices

- Focus on business goals rather than UI screens.
- Keep diagrams simple and readable.
- Use verb phrases for Use Cases.
- Keep Actors outside the System Boundary.
- Validate diagrams with business stakeholders.
- Create separate diagrams for different modules when required.
- Use Include only for mandatory reusable behavior.
- Use Extend only for optional behavior.
- Review the diagram before moving to design.

---

# 6. Exercises

## Easy

Design a Use Case Diagram for:

- ATM
- Library
- Student Management System

---

## Medium

Design a Use Case Diagram for:

- Food Delivery
- Hotel Booking
- Online Banking

---

## Advanced

Design complete Use Case Diagrams for:

- Amazon
- Netflix
- Uber
- BookMyShow
- WhatsApp
- LinkedIn
- Hospital Management System

For each exercise, identify:

- Actors
- Use Cases
- Relationships
- System Boundary
- Business Capabilities

---

# 7. Revision Cheat Sheet

## Remember

```text
Actor
↓
Who interacts with the system?

Use Case
↓
What does the system do?

System Boundary
↓
What belongs to the system?

Association
↓
Who performs what?

Include
↓
Always executed

Extend
↓
Optional behavior

Generalization
↓
Inheritance between Actors or Use Cases
```

---

## Include vs Extend

| Include | Extend |
|----------|---------|
| Mandatory | Optional |
| Always Executed | Executed only under certain conditions |
| Promotes Reusability | Adds Optional Functionality |
| Base Use Case Depends on It | Base Use Case Works Independently |

---

## Design Flow

```text
Business Requirement
        │
        ▼
Actors
        │
        ▼
Use Cases
        │
        ▼
System Boundary
        │
        ▼
Relationships
        │
        ▼
Validate with Stakeholders
        │
        ▼
Software Design
```

---

# 8. Engineering Checklist

Before finalizing a Use Case Diagram, verify the following:

- Business requirements are clearly understood.
- All external actors are identified.
- Every Use Case represents a business goal.
- Actor names are meaningful.
- Use Case names start with verbs.
- The System Boundary is clearly defined.
- Include and Extend relationships are used correctly.
- The diagram is easy to understand.
- The diagram has been reviewed with stakeholders.
- The diagram is ready for the next design phase.

---

# 9. Key Takeaways

- Use Case Diagrams capture functional requirements from a business perspective.
- They help bridge the gap between business stakeholders and technical teams.
- They guide software design but do not define implementation.
- A good Use Case Diagram improves communication, reduces ambiguity, and leads to better software architecture.
- Understanding the "why" behind each Use Case is more important than memorizing UML notation.
- Use Case Diagrams are the foundation for subsequent artifacts such as Class Diagrams, Sequence Diagrams, Activity Diagrams, and software implementation.

---

# 10. Related Chapters

Continue your learning with:

- Class Diagram
- Object Diagram
- Sequence Diagram
- Activity Diagram
- State Machine Diagram
- Communication Diagram
- SOLID Principles
- Design Patterns
- Domain-Driven Design (DDD)
- System Design