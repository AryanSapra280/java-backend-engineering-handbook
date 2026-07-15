# UML Class Diagram

## What is a Class Diagram?

A **Class Diagram** is a UML (Unified Modeling Language) diagram that represents the **static structure** of a software system.

It describes:

- Classes
- Attributes
- Methods
- Relationships between classes

Unlike sequence or activity diagrams, a class diagram focuses on **what the system is**, not **how it behaves**.

Think of it as the **blueprint** before writing Java code.

---

# Why Class Diagrams?

Class diagrams help us:

- Design object-oriented software.
- Visualize system structure.
- Identify responsibilities of classes.
- Define relationships between objects.
- Convert design directly into code.
- Improve communication among developers.

---

# Structure of a Class

Every class consists of three sections.

```
+----------------------+
|      Movie           |
+----------------------+
| - title : String     |
| - duration : int     |
| - genre : String     |
+----------------------+
| + play()             |
| + stop()             |
| + getDuration()      |
+----------------------+
```

### Sections

1. Class Name
2. Attributes
3. Methods (Operations)

---

# Access Modifiers

| Symbol | Visibility |
|---------|------------|
| + | Public |
| - | Private |
| # | Protected |

Example

```
+ login()

- password

# calculateInterest()
```

---

# Types of Relationships

## 1. Inheritance (IS-A)

Represents an inheritance relationship.

Example:

```
Vehicle
   ▲
   |
 Car
```

Java

```java
class Vehicle {}

class Car extends Vehicle {}
```

Use when a child **is a** parent.

Example:

- Car is a Vehicle
- Dog is an Animal

---

## 2. Association

Represents communication between two independent objects.

```
Doctor -------- Patient
```

Examples

- Teacher teaches Student
- Customer places Order

Neither object owns the other.

---

## 3. Aggregation (Weak HAS-A)

Aggregation represents a weak ownership.

```
Department ◇──── Employee
```

Characteristics

- Child can exist independently.
- Parent does not control child's lifecycle.

Examples

- Team → Player
- Department → Employee
- Library → Books

If the department is deleted,
employees still exist.

---

## 4. Composition (Strong HAS-A)

Composition represents strong ownership.

```
House ◆──── Room
```

Characteristics

- Child cannot exist independently.
- Parent owns child's lifecycle.

Examples

- House → Room
- Order → OrderItem
- Car → Engine (depending on the design)

If the Order is deleted,
OrderItems are deleted.

---

## 5. Dependency (USES-A)

Represents temporary usage.

```
PaymentService -----> EmailService
```

Java Example

```java
public class PaymentService {

    private EmailService emailService;

    public PaymentService(EmailService emailService){
        this.emailService = emailService;
    }
}
```

Spring Dependency Injection is a good example of dependency.

---

# Choosing the Correct Relationship

Use this decision tree.

```
Is it an "IS-A" relationship?

        Yes
         |
   Inheritance

         No
         |
Can child exist without parent?

        No
         |
    Composition

        Yes
         |
Does parent own it?

      Partially
         |
   Aggregation

Otherwise

Dependency / Association
```

---

# Association vs Aggregation vs Composition

| Feature | Association | Aggregation | Composition |
|----------|------------|-------------|-------------|
| Ownership | No | Weak | Strong |
| Lifecycle Dependency | No | No | Yes |
| Can Child Exist Alone? | Yes | Yes | No |
| Example | Doctor-Patient | Team-Player | Order-OrderItem |

---

# Class Diagram in Spring Boot

Typical layered architecture:

```
Controller
     |
     ▼
Service
     |
     ▼
Repository
     |
     ▼
Database
```

Relationships:

- Controller depends on Service.
- Service depends on Repository.
- Repository communicates with Database.

Notice that:

- Service does **not** inherit Repository.
- Controller does **not** inherit Service.

They use **Dependency** instead.

---

# Interview Tips

When solving LLD problems:

1. Gather requirements.
2. Identify classes.
3. Identify responsibilities.
4. Decide relationships.
5. Draw the class diagram.
6. Only then start coding.

---

# Common Interview Mistakes

❌ Using inheritance everywhere.

❌ Confusing aggregation and composition.

❌ Ignoring object responsibilities.

❌ One class handling too many tasks (God Object).

❌ Jumping directly into Java code.

---

# Key Takeaways

- A class diagram represents the **static view** of a system.
- It is the blueprint for object-oriented design.
- Focus on responsibilities before implementation.
- Understand relationships deeply:
    - Inheritance → IS-A
    - Association → Uses
    - Aggregation → Weak HAS-A
    - Composition → Strong HAS-A
    - Dependency → Temporary USES-A
- In real-world Spring Boot applications, dependency is the most common relationship.