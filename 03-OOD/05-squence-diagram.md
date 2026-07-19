# Sequence Diagram

## What is a Sequence Diagram?

A **Sequence Diagram** is a UML behavioral diagram that represents **how different objects interact with each other over time** to accomplish a particular use case.

Unlike a Class Diagram, which shows the **static structure** of a system, a Sequence Diagram shows the **dynamic behavior** of a system.

It answers questions like:

* Who starts the interaction?
* Which object calls which object?
* In what order do interactions happen?
* Which object is responsible for what?
* When does an object become active?

---

# Class Diagram vs Sequence Diagram

| Class Diagram            | Sequence Diagram             |
| ------------------------ | ---------------------------- |
| Static View              | Dynamic View                 |
| Shows structure          | Shows behavior               |
| Focuses on classes       | Focuses on interactions      |
| Represents relationships | Represents message flow      |
| Used before coding       | Used to model execution flow |

Visualizing the difference:

```text
Class Diagram

Customer -------- ATM -------- Account


Sequence Diagram

Customer
    |
    | withdrawCash()
    v
ATM
    |
    | validate()
    v
Account
    |
    | approved
    ^
ATM
    |
    | dispenseCash()
    v
CashDispenser
```

---

# Why Do We Need Sequence Diagrams?

Suppose we have the following classes:

```text
Customer
ATM
Transaction
Account
CashDispenser
```

A Class Diagram tells us these classes exist.

But it doesn't answer:

* Who creates Transaction?
* Who validates the account?
* Who tells the cash dispenser to release cash?
* What happens if validation fails?

A Sequence Diagram answers these questions by showing the order of communication.

---

# Elements of a Sequence Diagram

## 1. Lifeline

A lifeline represents the existence of an object during an interaction.

```text
+-----------+
| Customer  |
+-----------+
      |
      |
      |
      |
```

Every participant has one lifeline.

Examples:

* User
* ATM
* Payment Service
* Database
* Notification Service

---

## 2. Activation Bar

An activation bar represents the period during which an object is actively executing an operation.

```text
+--------+
|  ATM   |
+--------+
    |
   ████
   ████
    |
```

Think of it as:

> "The object is currently busy performing some work."

---

## 3. Messages

Objects communicate by sending messages.

### Synchronous Message

The sender waits until the receiver finishes processing.

```text
Customer ---------> ATM
```

Java example:

```java
account.validate();
```

The caller waits for `validate()` to complete.

---

### Asynchronous Message

The sender continues execution without waiting.

```text
NotificationService ----->> Kafka
```

Examples:

* Kafka
* RabbitMQ
* Email sending
* SMS
* Push notifications

---

### Return Message

Represents the response after a synchronous call.

```text
Customer ---------> ATM

Customer <--------- ATM
```

Usually drawn using a dashed arrow.

---

### Create Message

Represents creation of a new object.

```text
ATM ---------> Transaction
                <<create>>
```

Equivalent Java:

```java
Transaction tx = new Transaction();
```

---

### Destroy Message

Represents the destruction of an object.

```text
Transaction
      |
      |
      X
```

---

### Lost Message

The receiver is unknown.

```text
Object ---------> ○
```

---

### Found Message

The sender is unknown.

```text
○ ---------> Object
```

---

# Fragment Frames

Fragments help represent conditions, loops, and reusable sequences.

---

## ALT (Alternative)

Represents **if-else**.

```text
+--------------------------------------+
| ALT                                  |
|--------------------------------------|
| [Balance >= Amount]                  |
|    Dispense Cash                     |
|--------------------------------------|
| [Else]                               |
|    Show "Insufficient Balance"        |
+--------------------------------------+
```

Equivalent Java:

```java
if(balance >= amount){
    dispenseCash();
}else{
    showError();
}
```

---

## OPT (Optional)

Represents **if**.

```text
+-----------------------------+
| OPT                         |
|-----------------------------|
| [User Requested Receipt]    |
|    Print Receipt            |
+-----------------------------+
```

Equivalent Java:

```java
if(printReceipt){
    printReceipt();
}
```

---

## LOOP

Represents repetition.

```text
+-------------------------+
| LOOP                    |
|-------------------------|
| Retry Password          |
| Maximum 3 Attempts      |
+-------------------------+
```

Equivalent Java:

```java
while(attempts < 3){
    authenticate();
}
```

---

## REF

Represents reuse of another sequence diagram.

```text
+----------------------+
| REF                  |
| Validate Payment     |
+----------------------+
```

Useful for large systems.

---

# ATM Withdrawal Example

Participants:

* Customer
* ATM
* Transaction
* Account
* CashDispenser

```text
Customer
    |
    | withdraw(amount)
    v
ATM
    |
    | createTransaction()
    v
Transaction
    |
    | validate(amount)
    v
Account
    |
    | balanceAvailable
    ^
Transaction
    |
    | approved
    ^
ATM
    |
    | dispenseCash()
    v
CashDispenser
    |
    | cashReleased
    ^
ATM
    |
    | Display Success
    v
Customer
```

---

# Login Flow Example (Spring Boot)

```text
User
 |
 | POST /login
 v
LoginController
 |
 | authenticate()
 v
AuthenticationService
 |
 | findByEmail()
 v
UserRepository
 |
 | SELECT User
 v
Database
 ^
 |
UserRepository
 ^
 |
AuthenticationService
 |
 | generateToken()
 v
JWTService
 ^
 |
AuthenticationService
 ^
 |
LoginController
 |
 | HTTP 200 + JWT
 v
User
```

---

# Spring Boot Request Flow

A typical backend request follows this sequence:

```text
Client
   |
   v
Controller
   |
   v
Service
   |
   v
Repository
   |
   v
Database
   ^
Repository
   ^
Service
   ^
Controller
   ^
Client
```

Every API request you build follows this interaction pattern.

---

# Best Practices

* Start with the actor.
* Keep only meaningful interactions.
* Show major business operations.
* Avoid getters/setters in diagrams.
* Use fragments for conditions and loops.
* Keep diagrams readable.

---

# Common Interview Mistakes

❌ Confusing Class Diagram with Sequence Diagram.

❌ Missing return messages.

❌ Showing every internal helper method.

❌ Putting business logic in the wrong object.

❌ Ignoring alternate scenarios.

---

# Interview Questions & Answers

## 1. Why is a Sequence Diagram called a dynamic diagram?

Because it shows **how objects interact over time**. It focuses on the execution flow of a use case rather than the system's structure.

---

## 2. Difference between Class Diagram and Sequence Diagram?

* **Class Diagram** shows the static structure (classes and relationships).
* **Sequence Diagram** shows the runtime interaction and order of communication between objects.

---

## 3. What is a Lifeline?

A lifeline represents the existence of an object during an interaction. Every participant in a sequence diagram has its own lifeline.

---

## 4. What is an Activation Bar?

An activation bar represents the period during which an object is actively executing an operation or processing a message.

---

## 5. Is a Java method call synchronous or asynchronous?

A normal Java method call is **synchronous** because the caller waits until the method completes before continuing execution.

---

## 6. Give an example of an asynchronous call in Spring Boot.

Examples include:

* Publishing an event to Kafka
* Sending an email asynchronously
* Executing an `@Async` method
* Publishing a message to RabbitMQ

In all these cases, the caller continues execution without waiting for completion.

---

## 7. Which relationship does Controller → Service represent?

It is a **dependency relationship** because the controller uses the service to perform business logic but does not inherit from or own it.

---

## 8. Draw the sequence for Login.

```text
User
 ↓
LoginController
 ↓
AuthenticationService
 ↓
UserRepository
 ↓
Database
 ↑
UserRepository
 ↑
AuthenticationService
 ↓
JWTService
 ↑
AuthenticationService
 ↑
LoginController
 ↑
User
```

---

## 9. Draw the sequence for Food Ordering.

```text
Customer
 ↓
OrderController
 ↓
OrderService
 ↓
RestaurantService
 ↓
PaymentService
 ↓
NotificationService
 ↑
Customer
```

---

## 10. Draw the sequence for UPI Payment.

```text
User
 ↓
UPI App
 ↓
Bank Server
 ↓
NPCI
 ↓
Receiver Bank
 ↑
NPCI
 ↑
Bank Server
 ↑
UPI App
 ↑
User
```

---

## 11. Suppose validation fails halfway. Which object should handle rollback?

The **business service or transaction manager** should handle rollback because it coordinates the entire operation and maintains consistency.

For example, in Spring Boot, this is commonly managed using `@Transactional`, ensuring that if any step fails, all previous database changes are rolled back automatically.

---

# Key Takeaways

* A Sequence Diagram models the **behavior** of a system.
* It represents interactions between objects over time.
* Focus on **who talks to whom** and **in what order**.
* Method calls are typically synchronous.
* Event-driven systems often use asynchronous communication.
* Fragment frames (`alt`, `opt`, `loop`, `ref`) help model real-world scenarios.
* Sequence diagrams bridge the gap between requirements and implementation and are frequently used in Low-Level Design interviews.
