# Engineering Journal — Day 03

## Topic: Abstraction

---

# What Changed in My Thinking Today?

Before today, I thought abstraction simply meant hiding implementation using interfaces or abstract classes.

Today I realized that abstraction is fundamentally about reducing complexity by exposing only the operations a client needs.

The focus is not on hiding for security—it is on simplifying interaction.

---

# Biggest Mindset Shift

Yesterday:

Encapsulation protects an object's business rules.

Today:

Abstraction decides what the outside world should know.

These concepts solve different problems but work together in good software design.

---

# My Biggest Learning

Interfaces are not created just because Java provides them.

They are introduced to reduce coupling and allow clients to depend on behavior rather than implementation.

Creating unnecessary interfaces adds complexity instead of reducing it.

---

# Important Design Question

Whenever I think about introducing an interface, I should ask:

* Will there be multiple implementations?
* Should clients remain independent of implementation?
* Does an abstraction improve flexibility?

If the answer is "No", an interface may not be necessary.

---

# New Connection I Made

One design can involve multiple OOP principles together.

Example:

PaymentService (Interface)
↓
Abstraction

CardPaymentService implements PaymentService
↓
Inheritance

PaymentService service = new CardPaymentService();
service.pay();
↓
Polymorphism

This helped me understand that OOP principles are connected rather than isolated topics.

---

# Mistakes I Was Making

I believed abstraction only meant using interfaces.

Today I learned that interfaces and abstract classes are simply Java tools used to implement the design principle of abstraction.

The principle exists independently of the programming language.

---

# Mentor Feedback

### Strengths

* Asked deeper "Why?" questions instead of memorizing definitions.
* Connected abstraction with interfaces and enterprise applications.
* Started relating multiple OOP principles together.
* Questioned when to use interfaces versus abstract classes.

---

### Areas to Improve

* Continue practicing interface vs abstract class decisions.
* Learn to identify abstraction in existing Spring Boot code.
* Focus on design decisions before language features.

---

# Rules I Want to Remember

1. Reduce complexity, don't expose it.
2. Program to abstractions, not implementations.
3. Don't create interfaces without a clear design need.
4. Hide unnecessary implementation details.
5. Keep APIs simple and meaningful.

---

# Questions I Should Ask During Design

* What should the client know?
* What implementation details can remain hidden?
* Will another implementation exist in the future?
* Am I exposing behavior or implementation?

---

# Confidence

Today's Confidence: ____ / 10

---

# Tomorrow's Goal

Understand Inheritance as a mechanism for modeling "IS-A" relationships while avoiding the common mistake of using inheritance where composition is a better choice.
