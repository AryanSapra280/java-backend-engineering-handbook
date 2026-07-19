# Activity Diagram

## What is an Activity Diagram?

An **Activity Diagram** is a UML behavioral diagram that models the **workflow of a system**.

It focuses on **activities (tasks)**, **decisions**, **parallel execution**, and **overall business flow**.

Unlike a Sequence Diagram, it does **not** focus on which object performs an action.

Instead, it focuses on **what activities happen and in what order**.

---

# Why Do We Need an Activity Diagram?

Suppose we want to model ATM Cash Withdrawal.

A Sequence Diagram tells us:

```
Customer
    ↓
ATM
    ↓
Account
    ↓
Cash Dispenser
```

It explains **which object communicates with which object.**

However, sometimes we don't care about the objects.

Instead, we simply want to understand the workflow.

```
Start

↓

Insert Card

↓

Enter PIN

↓

Validate PIN

↓

Select Withdraw

↓

Enter Amount

↓

Check Balance

↓

Dispense Cash

↓

Print Receipt

↓

End
```

This is exactly what an Activity Diagram represents.

---

# Purpose of an Activity Diagram

Activity Diagrams are used to:

- Model business workflows.
- Describe system processes.
- Represent decision making.
- Show concurrent execution.
- Describe end-to-end functionality.

Think of it as an advanced flowchart.

---

# Activity Diagram vs Sequence Diagram

| Sequence Diagram | Activity Diagram |
|------------------|------------------|
| Focuses on Objects | Focuses on Activities |
| Shows message passing | Shows workflow |
| Dynamic behavior | Functional behavior |
| Shows who performs the action | Shows what action happens |
| Used in object interactions | Used for business processes |

---

# Class Diagram vs Sequence Diagram vs Activity Diagram

## Class Diagram

Answers:

> **Who exists?**

Example

```
Customer

ATM

Account

Transaction
```

---

## Sequence Diagram

Answers:

> **Who talks to whom?**

```
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

---

## Activity Diagram

Answers:

> **What happens step by step?**

```
Insert Card

↓

Validate PIN

↓

Withdraw Cash

↓

Dispense Cash

↓

Print Receipt
```

Notice that objects disappear.

Only activities remain.

---

# Components of an Activity Diagram

## 1. Initial Node

Represents the start of the workflow.

```
●
```

Every activity diagram starts with an initial node.

---

## 2. Action

Represents a task performed by the system.

```
+--------------------+
| Validate PIN       |
+--------------------+
```

Examples:

- Login
- Validate Payment
- Send Email
- Generate Report

---

## 3. Control Flow

Represents the order of execution.

```
Login

↓

Validate User

↓

Generate JWT
```

---

## 4. Object Flow

Represents movement of data or objects.

```
Customer

↓

ATM

↓

Transaction

↓

Account
```

Unlike Control Flow, this tracks the data being passed.

---

## 5. Decision Node

Represents an if-else condition.

```
             Balance Available?

            /                 \

         Yes                   No

         |                     |

Dispense Cash          Show Error
```

Equivalent Java

```java
if(balance >= amount){
    dispenseCash();
}else{
    showError();
}
```

---

## 6. Merge Node

Combines multiple decision branches into one.

```
        Success

           \

            \

             Continue

            /

           /

        Retry
```

Equivalent Java

```java
if(success){
}

else{
}

continueExecution();
```

---

## 7. Fork

Represents parallel execution.

```
Generate Invoice

        |

       Fork

      /     \

Send Email   Update Inventory
```

Equivalent Java concept

```
Thread A

Thread B
```

or

```
CompletableFuture
```

---

## 8. Join

Waits for parallel tasks to finish.

```
Send Email

        \

         Join

        /

Update Inventory

↓

Complete Order
```

Nothing continues until both activities finish.

---

## 9. Flow Final

Stops only the current flow.

```
Current Branch Ends
```

Other branches may continue.

---

## 10. Activity Final

Ends the entire activity.

```
◎
```

Everything stops.

---

# ATM Withdrawal Example

```
                ●
                │
                ▼
        Insert Card
                │
                ▼
           Enter PIN
                │
                ▼
          Validate PIN
                │
                ▼
        Select Withdraw
                │
                ▼
         Enter Amount
                │
                ▼
        Balance Available?
          /             \
       Yes               No
        │                 │
        ▼                 ▼
 Dispense Cash      Show Error
        │                 │
        └────────┬────────┘
                 ▼
          Print Receipt
                 │
                 ▼
                 ◎
```

---

# Login Flow Example

```
          ●
          │
          ▼
      Enter Email
          │
          ▼
     Enter Password
          │
          ▼
   Validate Credentials
          │
          ▼
 Credentials Valid?
     /             \
   Yes             No
    │               │
    ▼               ▼
Generate JWT   Show Error
    │               │
    └───────┬───────┘
            ▼
      Return Response
            │
            ◎
```

---

# Parallel Execution Example

Suppose an order is placed.

After payment succeeds, two tasks can happen simultaneously.

```
                Payment Success
                      │
                     Fork
                   /      \
                  ▼        ▼
        Send Email   Update Inventory
                  \        /
                   \      /
                    Join
                      │
                      ▼
               Complete Order
```

This is one of the biggest advantages of Activity Diagrams over simple flowcharts.

---

# Sequence Diagram vs Activity Diagram

## Sequence Diagram

```
Customer

↓

Controller

↓

Service

↓

Repository

↓

Database
```

Focus:

Who communicates?

---

## Activity Diagram

```
Receive Request

↓

Validate Request

↓

Save Data

↓

Return Response
```

Focus:

What happens?

---

# Interview Tips

When should you use an Activity Diagram?

Use it when:

- Modeling business processes.
- Explaining workflows.
- Showing parallel execution.
- Showing decisions.
- Explaining algorithms.

Use a Sequence Diagram when:

- Showing interaction between objects.
- Explaining message passing.
- Designing object collaboration.

---

# Common Interview Mistakes

❌ Confusing Activity Diagram with Flowchart.

❌ Showing classes instead of activities.

❌ Ignoring decision nodes.

❌ Forgetting parallel execution.

❌ Missing start/end nodes.

---

# Interview Questions & Answers

## 1. What is an Activity Diagram?

An Activity Diagram is a UML behavioral diagram that models the workflow of a system by representing activities, decisions, loops, and parallel execution.

---

## 2. Difference between Activity Diagram and Sequence Diagram?

A Sequence Diagram focuses on interactions between objects through message passing.

An Activity Diagram focuses on the workflow of activities irrespective of which object performs them.

---

## 3. Difference between Activity Diagram and Flowchart?

A Flowchart models a generic algorithm or process.

An Activity Diagram is an object-oriented UML diagram that additionally supports concurrency, synchronization, object flows, and system behavior.

---

## 4. What is a Decision Node?

A Decision Node represents branching based on a condition.

Equivalent Java:

```java
if(condition){
    ...
}else{
    ...
}
```

---

## 5. What is a Merge Node?

A Merge Node combines multiple alternative paths into a single execution path.

---

## 6. What is the difference between Fork and Join?

Fork splits one execution into multiple parallel executions.

Join waits for all parallel executions to complete before continuing.

---

## 7. Give a real-world example where Fork and Join are useful.

After a customer places an order:

- Send confirmation email.
- Update inventory.
- Generate invoice.

These tasks can execute in parallel.

The order is marked complete only after all three finish.

---

## 8. Can Activity Diagrams represent parallel execution?

Yes.

Fork and Join nodes are specifically designed to model concurrent workflows.

---

## 9. Which UML diagram would you choose to explain a login process?

If I want to explain **the workflow**, I would use an Activity Diagram.

If I want to explain **how LoginController, AuthenticationService, Repository, and Database communicate**, I would use a Sequence Diagram.

---

# Key Takeaways

- **Use Case Diagram** → What should the system do?
- **Class Diagram** → Which classes implement those features?
- **Sequence Diagram** → How do those classes interact?
- **Activity Diagram** → What is the overall workflow of the process?

Remember this progression:

```
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
Java Code
```

Each UML diagram answers a different design question, and together they provide a complete view of the system from requirements to implementation.