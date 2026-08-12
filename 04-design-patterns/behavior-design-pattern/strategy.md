# Strategy Design Pattern

> **Category:** Behavioral Design Pattern  
> **Primary Goal:** Encapsulate interchangeable algorithms/behaviors and make them independently replaceable.

---

# 1. Intent

The **Strategy Design Pattern** defines a family of interchangeable algorithms, encapsulates each algorithm in its own class, and allows the algorithm to be selected independently from the object that uses it.

### In simple words

> **When a class can perform the same operation in multiple different ways, move those different ways into separate Strategy classes.**

The class using the strategy should not need to know the implementation details of the algorithm.

### Mental model

```text
Context
   |
   | uses
   ↓
Strategy
   |
   +---- Strategy A
   +---- Strategy B
   +---- Strategy C