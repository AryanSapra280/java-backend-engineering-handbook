# UML Cheatsheet

> **Goal:** Revise all important UML diagrams in under 10 minutes.

---

# UML Design Flow

```text
Business Requirement
        │
        ▼
Use Case Diagram
        │
        ▼
Class Diagram
        │
        ▼
Sequence Diagram
        │
        ▼
Activity Diagram
        │
        ▼
State Machine Diagram
        │
        ▼
Java Implementation
```

---

# 1. Use Case Diagram

### Purpose

Captures **functional requirements**.

### Answers

> **What should the system do?**

### Focus

- Actors
- Features
- User interactions
- System boundary

### Example

```text
Customer

↓

Withdraw Cash
```

### Interview Keyword

**Requirements Gathering**

---

# 2. Class Diagram

### Purpose

Models the **static structure** of the system.

### Answers

> **Which classes implement these features?**

### Focus

- Classes
- Attributes
- Methods
- Relationships

### Example

```text
Customer

ATM

Account

Transaction
```

### Interview Keyword

**Static Structure**

---

# 3. Sequence Diagram

### Purpose

Shows how classes collaborate at runtime.

### Answers

> **How do objects interact to complete one use case?**

### Focus

- Objects
- Messages
- Method calls
- Order of execution

### Example

```text
Customer

↓

ATM

↓

Account

↓

ATM

↓

Customer
```

### Interview Keyword

**Object Interaction**

---

# 4. Activity Diagram

### Purpose

Models the business workflow.

### Answers

> **What activities happen step by step?**

### Focus

- Workflow
- Decisions
- Parallel execution
- Business process

### Example

```text
Insert Card

↓

Validate PIN

↓

Withdraw Cash

↓

Print Receipt
```

### Interview Keyword

**Workflow**

---

# 5. State Machine Diagram

### Purpose

Models the lifecycle of a single object.

### Answers

> **How does an object change its state over time?**

### Focus

- States
- Events
- Transitions
- Guards
- Actions

### Example

```text
Placed

↓

Paid

↓

Shipped

↓

Delivered
```

### Interview Keyword

**Lifecycle**

---

# Static vs Dynamic

| Diagram | Type |
|----------|------|
| Use Case | Functional |
| Class | Static |
| Sequence | Dynamic |
| Activity | Dynamic |
| State Machine | Dynamic |

---

# One-Line Difference

| Diagram | One Line |
|----------|----------|
| Use Case | What features exist? |
| Class | Which classes exist? |
| Sequence | Who talks to whom? |
| Activity | What happens next? |
| State Machine | Which state is the object in? |

---

# Think Like an Interviewer

## Use Case

```
Customer

↓

Withdraw Cash
```

Think:

> Features

---

## Class

```
Customer

ATM

Account
```

Think:

> Structure

---

## Sequence

```
Customer

↓

ATM

↓

Account
```

Think:

> Communication

---

## Activity

```
Login

↓

Validate

↓

Generate JWT
```

Think:

> Workflow

---

## State Machine

```
Pending

↓

Paid

↓

Shipped
```

Think:

> Lifecycle

---

# Which UML Diagram Should I Use?

| Situation | Diagram |
|-----------|----------|
| Gather requirements | Use Case |
| Design classes | Class |
| Explain API flow | Sequence |
| Explain business process | Activity |
| Explain object lifecycle | State Machine |

---

# Spring Boot Mapping

| Spring Component | UML Diagram |
|------------------|-------------|
| API Requirements | Use Case |
| Entity + Service + Repository | Class |
| Controller → Service → Repository | Sequence |
| Request Processing Flow | Activity |
| OrderStatus Enum | State Machine |

---

# Real-World Examples

| System | Best UML Diagram |
|---------|------------------|
| ATM Features | Use Case |
| ATM Design | Class |
| ATM Cash Withdrawal | Sequence |
| ATM Business Flow | Activity |
| ATM Card Lifecycle | State Machine |

---

# Memory Trick

```text
Use Case

↓

WHAT should be built?

↓

Class

↓

WHO builds it?

↓

Sequence

↓

WHO talks to WHO?

↓

Activity

↓

WHAT happens?

↓

State Machine

↓

WHAT STATE is the object in?
```

---

# Common Interview Mistakes

❌ Using Sequence Diagram instead of Activity Diagram.

❌ Treating State Diagram like a Flowchart.

❌ Showing classes inside Activity Diagram.

❌ Forgetting relationships in Class Diagram.

❌ Thinking Sequence Diagram is only about API calls.

---

# Interview Answers (30 Seconds)

### Use Case Diagram

Captures the functional requirements of the system by identifying actors and the use cases they interact with.

---

### Class Diagram

Represents the static structure of the system by showing classes, attributes, methods, and relationships.

---

### Sequence Diagram

Represents how objects communicate with each other over time to accomplish a specific use case.

---

### Activity Diagram

Represents the workflow of a business process by showing activities, decisions, loops, and parallel execution.

---

### State Machine Diagram

Represents the lifecycle of a single object by showing its states, transitions, events, and guard conditions.

---

# Final Memory Trick

```text
Use Case
↓

"What?"

Class
↓

"Who?"

Sequence
↓

"Who talks?"

Activity
↓

"What happens?"

State Machine
↓

"What state?"
```