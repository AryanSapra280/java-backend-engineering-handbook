# Engineering Journal – Use Case Diagram

---

# What Changed in My Thinking Today?

Today I realized that software design does not begin with classes.

It begins with understanding the business.

Before identifying objects, I must first understand who uses the system and what they are trying to achieve.

---

# Biggest Mindset Shift

Old thinking:

```text
Requirement

↓

Classes

↓

Code
```

New thinking:

```text
Requirement

↓

Actors

↓

Business Goals

↓

Use Cases

↓

Classes

↓

Code
```

This is the correct OOAD approach.

---

# My Biggest Learning

A Use Case Diagram models business interactions, not software.

Its purpose is to understand the problem rather than design the solution.

---

# Mistakes I Made Today

## 1. Confused actors with actions.

I initially wrote:

* Make Payment
* Refund
* Send Email

These are not actors.

Actors are entities.

Correct examples:

* Payment Gateway
* Email Service
* Maps Service

---

## 2. Needed to think more carefully about Include vs Extend.

I learned to ask one question:

"Does this happen every time?"

If yes → Include.

If it depends on conditions → Extend.

---

# Rules I Want to Remember

1. Never jump directly to classes.

2. Identify actors first.

3. Use Cases are actions.

4. Actors are entities.

5. Think from the user's perspective.

6. Separate analysis from design.

---

# My Personal Framework

Whenever I receive a new LLD problem:

Step 1

Understand the business.

↓

Step 2

Identify actors.

↓

Step 3

Identify business goals.

↓

Step 4

Draw Use Cases.

↓

Step 5

Move to Class Diagram.

---

# Interview Mindset

If an interviewer says:

"Design a Hotel Management System"

I should not immediately create classes.

Instead I should ask:

* Who are the actors?
* What functionality does the system support?
* What are the major business goals?

Only after that should I start identifying classes.

---

# Confidence

Today I can confidently:

✅ Identify actors.

✅ Separate primary and secondary actors.

✅ Identify use cases.

✅ Understand Include and Extend relationships.

Need more practice:

* Designing large systems.
* Identifying optional versus mandatory use cases.

---

# Mentor Feedback

Progress over the last few days:

Day 1

Objects first.

↓

Day 4

Business first.

This is a major improvement.

I am beginning to think like a software designer instead of immediately thinking like a programmer.

---

# Tomorrow's Goal

Learn Class Diagrams.

Focus on converting business requirements and use cases into software classes and relationships.

# Quick Revision

Requirements
↓
Actors
↓
Use Cases
↓
Classes
↓
Relationships
↓
Java