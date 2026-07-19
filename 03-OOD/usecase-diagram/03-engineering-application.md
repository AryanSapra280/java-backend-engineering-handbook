# Part 3 - Engineering Application

> **Learning Goal:** Learn how Use Case Diagrams are used by Business Analysts, Architects, Developers, and QA teams to transform business requirements into a well-designed software system.

---

## 1. Introduction

- Why engineering application matters
- Use Case Diagram is a communication artifact, not a coding artifact

---

## 2. Role in the Software Development Lifecycle (SDLC)

- Requirement Gathering
- Requirement Analysis
- Solution Design
- Development
- Testing
- Deployment
- Maintenance

---

## 3. How Companies Use Use Case Diagrams

- Business Analyst
- Product Manager
- Solution Architect
- Technical Lead
- Developers
- QA Engineers
- Business Stakeholders

---

## 4. From Business Requirement to System Design

Explain how a business requirement evolves into a software solution.

Business Requirement
↓
Use Case Diagram
↓
Business Capabilities
↓
Modules
↓
Domain Model
↓
APIs
↓
Architecture
↓
Implementation

---

## 5. Discovering Business Capabilities

Example:

Food Delivery

- Browse Restaurants
- Place Order
- Make Payment
- Track Order

Each capability becomes a candidate module.

---

## 6. Discovering System Modules

Show how Use Cases naturally help divide the application into modules.

Example:

Restaurant Module

Order Module

Payment Module

Notification Module

User Module

---

## 7. Discovering Domain Models

How Use Cases help identify important business entities.

Example:

Customer

Order

Restaurant

Menu

Payment

Delivery Partner

Coupon

Review

---

## 8. Discovering APIs

Transform business goals into API candidates.

Example:

Place Order

↓

POST /orders

Cancel Order

↓

POST /orders/{id}/cancel

Track Order

↓

GET /orders/{id}

---

## 9. Influence on Microservice Design

Explain how business capabilities often become independent services.

Restaurant Service

Order Service

Payment Service

Inventory Service

Notification Service

Tracking Service

---

## 10. Design Decisions

- REST vs Event
- Sync vs Async
- Single Service vs Multiple Services
- Shared Database vs Independent Database

---

## 11. Common Engineering Mistakes

- Designing classes before understanding business requirements
- Skipping Use Cases
- Mixing technical details with business requirements
- Creating one Use Case for every UI screen
- Ignoring external systems
- Ignoring alternate flows

---

## 12. Transition to Implementation

Briefly explain how engineering artifacts finally become code.

This chapter intentionally stops before implementation.

The next chapter demonstrates how these engineering decisions are translated into Java, Spring Boot, APIs, and production-ready applications.