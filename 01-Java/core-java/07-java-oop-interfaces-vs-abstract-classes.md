# Java OOP — Interfaces vs Abstract Classes

## Status

🟢 COMPLETED / PARKED

This document captures the **full discussion**, including the questions I raised, your reasoning, corrections, practical examples, interview-level conclusions, and the doubts that came up during the discussion.

The goal is not just revision. It is meant to be a future reference for understanding **why** we choose interfaces or abstract classes in real Java design.

---

# 1. Definition

**An interface defines a contract/capability that a class agrees to provide, while an abstract class provides a common base that can combine abstract behavior, concrete behavior, state, constructors, and shared implementation.**

Do not reduce this to:

> Interface = abstraction  
> Abstract class = partial abstraction

That is too shallow for SDE-2/SDE-3 interviews.

The better question is:

> **What behavior and state are actually shared, and what relationship exists between the classes?**

---

# 2. Starting From the Payment Problem

We used the BharatPe interview question as the practical starting point:

> “Walk me through how you would design a simple Java class hierarchy for handling different payment types.”

Suppose we have:

```text
UPI
Card
Wallet
```

and all payments need:

```java
process()
```

A natural abstraction is:

```java
interface Payment {

    void process();
}
```

Then:

```java
class UpiPayment implements Payment {

    @Override
    public void process() {
        // UPI processing
    }
}

class CardPayment implements Payment {

    @Override
    public void process() {
        // Card processing
    }
}

class WalletPayment implements Payment {

    @Override
    public void process() {
        // Wallet processing
    }
}
```

The consumer can depend on:

```java
Payment payment;
```

rather than knowing the concrete implementation.

This gives us polymorphism.

---

# 3. Why We Initially Chose an Interface

Your initial reasoning was:

> “I can choose abstract class or interface because I am abstracting a method and that abstract method will be exposed to the consumer.”

You then identified an important concern:

> A payment implementation might need other unrelated features, and Java allows a class to extend only one class.

That led us toward an interface.

This is good reasoning, but we refined it.

A stronger interview answer is:

> **“I'd prefer an interface when the payment types primarily need to follow a common contract and may need to implement other unrelated capabilities. Interfaces allow a class to implement multiple contracts. If I had meaningful shared state or common implementation that should be reused across payment types, I would consider an abstract class.”**

---

# 4. Important Correction: Abstract Classes Are NOT 100% Abstract

One of the first corrections we made:

You initially thought an abstract class was effectively 100% abstract.

That is incorrect.

An abstract class can contain:

```text
- abstract methods
- concrete methods
- instance fields
- constructors
- protected helpers
- common state
- common implementation
```

Example:

```java
abstract class Payment {

    abstract void process();

    void validate() {
        System.out.println("Common validation");
    }
}
```

A child only needs to provide the missing abstract behavior:

```java
class UpiPayment extends Payment {

    @Override
    void process() {
        System.out.println("UPI payment");
    }
}
```

Therefore:

> **An abstract class is not “100% abstract.” It can be a partially implemented base class.**

---

# 5. Your Important Follow-Up Question

You then asked a much better question:

> **“If interfaces can also have methods and default implementations, why do we need abstract classes?”**

This is an excellent question because modern Java interfaces can contain implementations.

The answer requires separating:

```text
Behavior
vs
State
```

---

# 6. Interfaces Can Provide Behavior

Modern Java interfaces can provide default behavior.

```java
interface Payment {

    void process();

    default void validate() {
        System.out.println("Common validation");
    }
}
```

Then:

```java
class UpiPayment implements Payment {

    @Override
    public void process() {
        System.out.println("UPI");
    }
}
```

`UpiPayment` gets the default `validate()` implementation.

So your observation was correct:

> **An interface can absolutely provide shared behavior.**

Therefore, “interfaces don't contain implementation” is outdated Java knowledge.

---

# 7. The Major Difference: Per-Object State

Now consider:

```java
abstract class Payment {

    protected String transactionId;
    protected BigDecimal amount;

    protected Payment(
            String transactionId,
            BigDecimal amount) {

        this.transactionId = transactionId;
        this.amount = amount;
    }

    abstract void process();

    protected void validate() {
        // common validation
    }
}
```

Now every payment object can have its own:

```text
transactionId
amount
```

For example:

```text
Payment 1
transactionId = T100
amount = 500

Payment 2
transactionId = T101
amount = 1000
```

The abstract class can define the shared instance state and initialize it through a constructor.

This is one of the strongest reasons to use an abstract class.

---

# 8. Why Interface Fields Are Different

You asked, essentially:

> “But can't an interface also have fields?”

It can have fields, but interface fields are implicitly:

```text
public
static
final
```

Example:

```java
interface Payment {

    int MAX_AMOUNT = 100000;
}
```

is effectively:

```java
public static final int MAX_AMOUNT = 100000;
```

That is a constant.

It is NOT per-object mutable state.

You cannot use an interface field to represent:

```text
Payment 1 → amount = 500
Payment 2 → amount = 1000
```

as instance state.

This distinction is important:

```text
Interface field
    ↓
shared constant

Abstract-class field
    ↓
per-object instance state
```

---

# 9. Constructors — Another Major Difference

An abstract class can define constructors:

```java
abstract class Payment {

    protected final String transactionId;

    protected Payment(String transactionId) {
        this.transactionId = transactionId;
    }
}
```

Then:

```java
class UpiPayment extends Payment {

    UpiPayment(String transactionId) {
        super(transactionId);
    }
}
```

The base class can therefore enforce initialization of common state.

An interface does not have constructors because it does not represent an object whose instance state needs to be initialized through a base-class constructor.

---

# 10. Protected State and Helper Methods

An abstract class can provide protected implementation details:

```java
abstract class Payment {

    protected BigDecimal amount;

    protected boolean isValidAmount() {
        return amount.signum() > 0;
    }

    abstract void process();
}
```

Subclasses can directly use the shared state and helper:

```java
class CardPayment extends Payment {

    @Override
    void process() {

        if (!isValidAmount()) {
            throw new IllegalArgumentException();
        }

        // Card-specific processing
    }
}
```

This is useful when the subclasses genuinely form a common implementation hierarchy.

---

# 11. The Correct Mental Model

### Interface

Think:

> **“What contract or capability does this class provide?”**

Example:

```java
interface Refundable {

    void refund();
}
```

Then:

```java
class CardPayment
        implements Payment, Refundable {
}
```

Another unrelated class could also implement it:

```java
class Subscription
        implements Refundable {
}
```

Both have the capability:

```text
refund()
```

without needing to share a common class hierarchy.

---

### Abstract class

Think:

> **“What common base implementation/state do these closely related classes share?”**

Example:

```java
abstract class Payment {

    protected final String transactionId;
    protected final BigDecimal amount;

    protected Payment(
            String transactionId,
            BigDecimal amount) {

        this.transactionId = transactionId;
        this.amount = amount;
    }

    protected void validate() {
        // common validation
    }

    abstract void process();
}
```

---

# 12. Multiple Interfaces vs Single Class Inheritance

Java allows:

```java
class CardPayment
        implements Payment, Refundable, Auditable {
}
```

A class can implement multiple interfaces.

But it cannot extend multiple classes:

```java
class CardPayment
        extends Payment, Transaction { // ❌
}
```

This is one reason interfaces are useful for composing independent capabilities.

Think:

```text
CardPayment
    │
    ├── Payment
    ├── Refundable
    └── Auditable
```

Each interface represents a capability/contract.

---

# 13. Can Interfaces Have Implementations?

Yes.

Modern Java interfaces can contain:

```text
abstract methods
default methods
static methods
private methods
```

So this statement is outdated:

> “Interfaces only contain abstract methods.”

---

# 14. Default Methods

Example:

```java
interface Payment {

    void process();

    default void audit() {
        System.out.println("Default audit");
    }
}
```

An implementation can inherit the default:

```java
class UpiPayment implements Payment {

    @Override
    public void process() {
        System.out.println("UPI");
    }
}
```

Or override it:

```java
class UpiPayment implements Payment {

    @Override
    public void process() {
        System.out.println("UPI");
    }

    @Override
    public void audit() {
        System.out.println("UPI-specific audit");
    }
}
```

---

# 15. Why Were Default Methods Introduced?

This is a valuable interview question.

Suppose an interface already has many implementations:

```java
interface Payment {

    void process();
}
```

Hundreds of classes may implement it.

Now suppose we want to add:

```java
void validate();
```

If it is abstract, every implementation must implement it immediately.

That can make API evolution difficult.

A default method allows:

```java
interface Payment {

    void process();

    default void validate() {
        // default implementation
    }
}
```

Existing implementations can inherit the new behavior.

Therefore:

> **Default methods help evolve interfaces while reducing the need to immediately modify every existing implementation.**

---

# 16. Default Method Diamond Problem

This was one of our important practical discussions.

Consider:

```java
interface A {

    default void process() {
        System.out.println("A");
    }
}
```

and:

```java
interface B {

    default void process() {
        System.out.println("B");
    }
}
```

Then:

```java
class Payment implements A, B {
}
```

What should:

```java
payment.process();
```

execute?

Java cannot arbitrarily choose:

```text
A.process()
or
B.process()
```

Therefore `Payment` must resolve the conflict.

---

# 17. Resolving the Diamond Conflict

```java
class Payment implements A, B {

    @Override
    public void process() {
        A.super.process();
    }
}
```

Or:

```java
@Override
public void process() {
    B.super.process();
}
```

Or the class can provide completely new behavior:

```java
@Override
public void process() {
    System.out.println("Payment-specific behavior");
}
```

---

# 18. Your Important Doubt: What Does `A.super` Mean?

You asked:

> **“Interface objects are not created, so how is `A.super.process()` working? Is `super` representing the instance?”**

This was a very important distinction.

There is no:

```java
new A(); // ❌
```

`A.super` does NOT mean:

> “the A object”

Instead, it means roughly:

> **“For the current object, explicitly invoke the default implementation supplied by interface A.”**

There is still only one object:

```text
Payment object
     ↑
     │
  payment
```

There is no separate A object and no separate B object.

Conceptually:

```text
                 Payment object
                       │
              implements A and B
                  /                          ↓           ↓
        A default method   B default method
                 \           /
                  \         /
                   Payment
                   chooses
                     ↓
              A.super.process()
```

So:

> **`A.super` is not an interface instance. It is a way to explicitly select A's default implementation in the context of the current object.**

---

# 19. `super` in Classes

We then connected this to normal class inheritance.

Example:

```java
class GrandParent {

    void process() {
        System.out.println("GrandParent");
    }
}

class Parent extends GrandParent {

    @Override
    void process() {
        System.out.println("Parent");
    }
}

class Child extends Parent {

    @Override
    void process() {
        System.out.println("Child");
    }

    void callParent() {
        super.process();
    }
}
```

Here:

```java
super.process();
```

means:

```text
Child
  │
  │ super.process()
  ↓
Parent.process()
```

`super` refers to the **immediate superclass**.

---

# 20. Can `super` Skip a Generation?

You correctly observed:

> “Even in class also super gives immediate parent.”

Correct.

You cannot write:

```java
GrandParent.super.process(); // ❌
```

from `Child` to directly bypass `Parent`.

If `Parent` itself calls:

```java
super.process();
```

then the chain can continue:

```text
Child.process()
      ↓
super.process()
      ↓
Parent.process()
      ↓
super.process()
      ↓
GrandParent.process()
```

So `super` is not a “jump to any ancestor” mechanism.

---

# 21. Interface Super-Method Refinement

Interfaces have a related but slightly different syntax:

```java
A.super.process();
```

The important interview-level rule is:

> **`X.super.method()` explicitly selects a default implementation from a directly relevant superinterface; it is not a general mechanism for jumping to an arbitrary ancestor interface.**

Example:

```java
interface A {

    default void process() {
        System.out.println("A");
    }
}

interface B extends A {

    @Override
    default void process() {
        System.out.println("B");
    }
}

class Payment implements B {

    @Override
    public void process() {
        B.super.process(); // ✅
    }
}
```

Here:

```text
Payment
   ↓ implements
B
   ↓ extends
A
```

`B.super.process()` selects B's default implementation.

You cannot use:

```java
A.super.process(); // ❌
```

as a mechanism for skipping B and directly selecting A's overridden default.

---

# 22. Interface Static Methods

Interfaces can contain static methods:

```java
interface Payment {

    static boolean isValidAmount(int amount) {
        return amount > 0;
    }
}
```

Call it using the interface:

```java
Payment.isValidAmount(100);
```

Not:

```java
payment.isValidAmount(100); // ❌
```

The static method belongs to the interface itself.

It does not participate in runtime polymorphism like an instance method.

---

# 23. Private Interface Methods

Interfaces can also have private helper methods.

Example:

```java
interface Payment {

    default void validate() {
        validateAmount();
    }

    private void validateAmount() {
        // helper logic
    }
}
```

The private helper exists for the interface's own implementation.

It is not part of the public contract and cannot be directly called by implementing classes.

---

# 24. Interface Fields

Interface fields are implicitly:

```text
public
static
final
```

Example:

```java
interface Payment {

    int MAX_AMOUNT = 100000;
}
```

Equivalent conceptually to:

```java
public static final int MAX_AMOUNT = 100000;
```

Therefore interface fields are constants, not per-object mutable state.

---

# 25. Functional Interfaces

## Definition

**A functional interface is an interface with exactly one abstract method.**

Example:

```java
@FunctionalInterface
interface PaymentValidator {

    boolean validate(Payment payment);
}
```

It can be used with a lambda:

```java
PaymentValidator validator =
        payment -> payment.getAmount() > 0;
```

Common JDK functional interfaces include:

```text
Predicate<T>
Function<T,R>
Consumer<T>
Supplier<T>
Runnable
Comparator<T>
```

We will cover these properly in the Java 8+ section.

---

# 26. `@FunctionalInterface`

The annotation tells the compiler that the interface is intended to have exactly one abstract method.

```java
@FunctionalInterface
interface PaymentValidator {

    boolean validate(Payment payment);
}
```

If another abstract method is added:

```java
boolean validate(Payment payment);
void audit(); // ❌
```

the compiler rejects the interface.

This protects the functional-interface contract.

Important:

A functional interface can still have default/static/private methods. The restriction is on the number of **abstract methods**.

---

# 27. Interface Inheritance

Interfaces can extend other interfaces.

```java
interface Payment {

    void process();
}
```

Then:

```java
interface RefundablePayment extends Payment {

    void refund();
}
```

A class implementing `RefundablePayment` must satisfy both contracts:

```java
class CardPayment implements RefundablePayment {

    @Override
    public void process() {
    }

    @Override
    public void refund() {
    }
}
```

Interfaces can also extend multiple interfaces:

```java
interface SecurePayment
        extends Payment, Auditable, Refundable {
}
```

This is another important difference from class inheritance.

---

# 28. Practical Capability Design

Suppose:

```text
UPI
Card
Wallet
```

all support:

```text
process()
```

but only Card and Wallet support:

```text
refund()
```

A good design could be:

```java
interface Payment {

    void process();
}

interface Refundable {

    void refund();
}
```

Then:

```java
class UpiPayment implements Payment {
}

class CardPayment
        implements Payment, Refundable {
}

class WalletPayment
        implements Payment, Refundable {
}
```

The important design principle is:

> **Model independent capabilities independently when they are genuinely independent.**

Don't force every payment type into one giant interface:

```java
interface Payment {

    void process();
    void refund();
    void chargeback();
    void audit();
    void settle();
    // ...
}
```

if not every payment supports all of those operations.

---

# 29. Interface + Abstract Class Together

They are not mutually exclusive.

A strong design can use both.

```java
interface Payment {

    void process();
}

interface Refundable {

    void refund();
}

abstract class BasePayment implements Payment {

    protected final String transactionId;
    protected final BigDecimal amount;

    protected BasePayment(
            String transactionId,
            BigDecimal amount) {

        this.transactionId = transactionId;
        this.amount = amount;
    }

    protected void validate() {
        // common validation
    }
}

class CardPayment
        extends BasePayment
        implements Refundable {

    @Override
    public void process() {
        validate();
        // card processing
    }

    @Override
    public void refund() {
        // refund
    }
}
```

Now:

```text
Payment
   ↓
common contract

Refundable
   ↓
independent capability

BasePayment
   ↓
shared state + implementation

CardPayment
   ↓
concrete behavior
```

This is much closer to real-world design than treating interface and abstract class as mutually exclusive choices.

---

# 30. When to Choose an Interface

Choose an interface when:

- You primarily need a contract.
- You are modeling a capability.
- Multiple unrelated classes can provide that capability.
- A class may need multiple capabilities.
- You don't need shared per-object state.
- Implementations may already extend another class.
- You want to decouple consumers from implementations.
- You expect multiple implementations.

Example:

```java
interface Retryable {

    void retry();
}
```

---

# 31. When to Consider an Abstract Class

Consider an abstract class when:

- Classes form a meaningful common family.
- They share instance state.
- They share substantial implementation.
- You need constructors.
- You need protected helper methods/state.
- You want to enforce part of the implementation.
- The subclasses naturally belong to one base-class hierarchy.

Example:

```java
abstract class Payment {

    protected final String transactionId;

    protected Payment(String transactionId) {
        this.transactionId = transactionId;
    }

    protected void validate() {
    }

    abstract void process();
}
```

---

# 32. The Important Interview Question

If asked:

> **“Why did you use an interface instead of an abstract class?”**

Do NOT answer:

> “Interface is for abstraction and abstract class is for partial abstraction.”

Instead start with:

> **“What behavior and state are actually shared?”**

Then reason:

```text
Only contract/capability?
        ↓
Interface

Shared implementation?
        ↓
Could consider abstract class

Shared instance state + constructors?
        ↓
Abstract class becomes stronger candidate

Multiple independent capabilities?
        ↓
Interfaces become stronger candidate
```

---

# 33. Don't Over-Interface Everything

Another important design lesson:

Do not create an interface for every class mechanically.

For example:

```text
PaymentService
PaymentServiceImpl
```

doesn't automatically mean an interface is required.

An interface should provide meaningful value such as:

```text
- polymorphism
- decoupling
- multiple implementations
- capability composition
- dependency inversion
- clear API contract
```

The goal is not:

> “Every service must have an interface.”

The goal is:

> **“Introduce an abstraction where it provides design value.”**

---

# 34. Spring Connection

This topic connects directly to Spring.

For example:

```java
interface PaymentService {

    void process();
}
```

Implementation:

```java
@Service
class UpiPaymentService
        implements PaymentService {

    @Override
    public void process() {
    }
}
```

A consumer can depend on:

```java
private final PaymentService paymentService;
```

instead of:

```java
private final UpiPaymentService paymentService;
```

This connects:

```text
Interface
    ↓
Polymorphism
    ↓
Dependency Inversion
    ↓
Dependency Injection
    ↓
Spring
```

We'll revisit this deeply in the Spring section.

---

# 35. Important Interview Traps

### Trap 1

> “Interfaces cannot have implementation.”

❌ Outdated.

Modern interfaces can have:

- default methods
- static methods
- private methods

### Trap 2

> “Abstract classes are 100% abstract.”

❌ Incorrect.

They can contain concrete methods and state.

### Trap 3

> “Interface fields are instance fields.”

❌ No.

They are implicitly:

```text
public static final
```

### Trap 4

> “`A.super.process()` creates an A object.”

❌ No.

There is still only the implementing object's instance.

It explicitly selects A's default implementation.

### Trap 5

> “`super` can skip to any ancestor.”

❌ No.

For classes, `super` refers to the immediate superclass.

### Trap 6

> “Interface is always better than abstract class.”

❌ No.

The choice depends on state, shared implementation, contracts, capabilities, and hierarchy.

### Trap 7

> “A functional interface cannot have default methods.”

❌ It can.

It simply must have exactly one abstract method.

### Trap 8

> “Every service should have an interface.”

❌ Not automatically.

The abstraction should provide real design value.

---

# 36. Interview-Ready Comparison

| Aspect | Interface | Abstract Class |
|---|---|---|
| Primary purpose | Contract / capability | Common base |
| Multiple inheritance | Multiple interfaces | One superclass |
| Instance state | ❌ | ✅ |
| Constructors | ❌ | ✅ |
| Abstract methods | ✅ | ✅ |
| Concrete methods | ✅ via default methods | ✅ |
| Static methods | ✅ | ✅ |
| Private helper methods | ✅ | ✅ |
| Protected instance state | ❌ | ✅ |
| Multiple capabilities | ✅ | Limited by single class inheritance |
| Good for unrelated classes | ✅ | Usually not |
| Good for shared state | ❌ | ✅ |
| Good for shared implementation | Possible via defaults | Stronger fit |

The table is useful for revision, but in interviews always explain the **reasoning**, not just the table.

---

# 37. First-Principles Decision Framework

This is the framework we want to reuse in LLD and HLD:

```text
                  Requirement
                       ↓
             What is actually shared?
                       ↓
          ┌────────────┴────────────┐
          ↓                         ↓
       Contract                 State + behavior
          ↓                         ↓
      Interface               Abstract class
          ↓                         ↓
 Multiple capabilities       Common base hierarchy
```

And sometimes:

```text
Interface
    +
Abstract base class
    +
Concrete implementation
```

is the best design.

---

# 38. Why This Matters for Your Interviews

This topic is especially relevant because your target roles mention:

```text
Strong OOP
Advanced Java
LLD
Spring
Microservices
Distributed systems
```

Interviewers are less interested in hearing:

> “Interface is abstraction.”

They want to see whether you can reason about:

```text
requirements
   ↓
abstraction
   ↓
trade-offs
   ↓
Java language features
   ↓
extensibility
   ↓
maintainability
```

That is the same first-principles approach we used in the payment-processing exercise.

---

# 39. Current OOP Progress

```text
Classes / Objects / Constructors    🟢
Encapsulation                       🟢
Inheritance                        🟢
Polymorphism                       🟢
Interfaces                         🟢
Abstract Classes                   🟢
```

Next OOP/Java topics will be selected based on interview ROI rather than simply following textbook order.

---

# 40. Topics Intentionally Deferred

We do NOT need to go deeper into these right now:

- bytecode-level interface method resolution
- JVM invocation instructions
- obscure JLS inheritance edge cases
- compiler-generated synthetic methods
- bridge methods
- advanced reflection details

These can be revisited if an interview exposes a specific gap.

---

# 41. Key Takeaways

1. **Interface = contract/capability.**
2. **Abstract class = common base with possible shared state and implementation.**
3. Interfaces can have default, static, and private methods.
4. Abstract classes can contain both abstract and concrete methods.
5. Interface fields are constants (`public static final`).
6. Abstract classes can have instance fields and constructors.
7. A class can implement multiple interfaces but extend only one class.
8. Default methods can create conflicts when multiple interfaces provide the same method.
9. `A.super.process()` does not create an interface object.
10. `A.super.process()` selects A's default implementation for the current implementing object.
11. `super` is not a mechanism for skipping arbitrary inheritance levels.
12. Functional interfaces have exactly one abstract method.
13. Interfaces can extend multiple interfaces.
14. Interface and abstract class can be used together.
15. Don't introduce abstractions mechanically.
16. Don't choose a pattern before understanding the requirement.
17. For interviews, explain **why** you chose the abstraction.

---

# 42. The Most Important Mental Model

If you remember only this:

```text
INTERFACE
"What can this object DO?"
        ↓
Contract / capability
        ↓
Can combine multiple interfaces


ABSTRACT CLASS
"What common implementation/state
does this family of objects SHARE?"
        ↓
Common base
        ↓
Single class inheritance
```

And when the two overlap:

```text
              Interface
                  +
           Abstract Base
                  +
          Concrete Class
```

can be a completely valid design.

---

## Next Topic

We will move forward rather than over-studying interfaces.

The next Java/OOP area should focus on **`equals()`, `hashCode()`, and object identity**, because this connects directly to:

```text
Objects
   ↓
Collections
   ↓
HashMap / HashSet
   ↓
Caching
   ↓
Real-world bugs
   ↓
Interview questions
```

This will also give us another opportunity to use the same first-principles approach rather than memorizing rules.
