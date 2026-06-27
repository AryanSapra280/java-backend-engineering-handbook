# Week 1 - Day 1

## Engineering Journal

---

# What Changed in My Thinking Today

### 1. Classes are NOT database tables.

Earlier, whenever I thought about a class, my first instinct was to think about fields.

Today I realized that a class represents a real-world entity, not a database record.

A real-world entity owns:

* State
* Behavior
* Business Rules

This is probably the biggest mindset shift of today.

---

### 2. Responsibilities come before fields.

Earlier my design process was:

Problem

↓

Fields

↓

Methods

Today my design process became:

Problem

↓

Responsibilities

↓

Objects

↓

Relationships

↓

Methods

↓

Fields

↓

Java Code

I should consciously follow this order every time I design a system.

---

### 3. LLD is NOT about writing Java classes.

I used to think Low-Level Design meant creating classes.

Today I realized:

LLD starts much earlier.

It starts by understanding the domain.

The Java code is only the implementation.

---

### 4. OOD happens before OOP.

This distinction became much clearer today.

OOD = Thinking

OOP = Coding

If I jump directly into IntelliJ, I'm skipping the most important engineering step.

---

### 5. Objects own behavior.

Objects are not containers of data.

Objects should perform actions on their own data.

Instead of:

withdraw(account)

Think:

account.withdraw()

This is much closer to real-world modeling.

---

### 6. Good software models the real world.

Instead of thinking about:

Functions

Tables

Database

I should think about:

Customer

Order

Book

Account

Parking Slot

Vehicle

Loan

These become my objects.

---

# Mistakes I Made Today

### Mistake 1

While designing Book and Member, I focused mostly on fields.

I forgot that objects exist because they have responsibilities.

Lesson:

Always ask:

"What does this object do?"

before asking

"What data does it have?"

---

### Mistake 2

While designing Laptop, I assigned behaviors like Click() to Laptop.

I learned that I should think carefully about ownership.

Click belongs more naturally to a Mouse or Trackpad.

Lesson:

Ask:

"Who is actually responsible for performing this action?"

---

### Mistake 3

While solving design exercises, I initially thought there must be one correct answer.

Today's discussion about a frozen bank account taught me that LLD is often driven by business requirements.

There are multiple valid designs if I clearly state my assumptions.

---

# Improvements I Noticed

✔ I naturally started identifying behaviors before fields.

This happened during the Laptop exercise.

That means my thinking is already changing.

---

✔ I started questioning responsibilities.

Instead of blindly accepting classes, I began asking:

Who owns this behavior?

This is how experienced engineers think.

---

✔ I understood that objects collaborate.

Objects do not exist in isolation.

A Loan collaborates with Book and Member.

A Parking Ticket collaborates with Vehicle and Parking Spot.

Everything is connected.

---

# Things I Should Remember Every Time I Solve an LLD Problem

1. Never start with Java.

2. Gather requirements first.

3. Identify actors.

4. Identify nouns.

5. Identify responsibilities.

6. Build relationships.

7. Only then think about Java classes.

---

# Questions I Should Continuously Ask Myself

Why does this object exist?

Why should this behavior belong here?

Can another object own this responsibility better?

Am I modeling the real world or simply designing database tables?

What business rule am I protecting?

---

# Golden Rules Learned Today

Rule 1

Behavior before Fields.

---

Rule 2

Objects own responsibilities.

---

Rule 3

OOD is language independent.

---

Rule 4

There is rarely one perfect design.

Always explain assumptions.

---

Rule 5

Classes represent real-world entities.

Not database tables.

---

# Things I Need To Improve

I still tend to think about data first.

I need to practice identifying responsibilities naturally.

I should continue asking:

"What does this object do?"

instead of

"What fields does this object have?"

---

# Biggest Takeaway of the Day

Today I stopped thinking like someone writing Java classes.

I started thinking like someone modeling software.

This is the foundation of Low-Level Design.

---

# Mentor's Feedback

Strengths

* Curious and asks "why".
* Comfortable discussing ideas.
* Quickly adapts when challenged.
* Already showing signs of object-oriented thinking.

Areas to Improve

* Go deeper into responsibility assignment.
* Think about ownership before implementation.
* Get comfortable stating assumptions instead of searching for a single correct answer.

---

# Tomorrow's Goal

Master Encapsulation.

Not by memorizing:

"private variables"

Instead understand:

"What engineering problem did encapsulation solve?"
