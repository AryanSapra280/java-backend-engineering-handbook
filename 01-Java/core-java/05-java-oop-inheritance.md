# Java OOP — Inheritance

## Status

🟢 COMPLETED / PARKED

This document captures our inheritance discussion, including Java-specific behavior, interview traps, and deferred topics.

---

# 1. Definition

**Inheritance is a Java mechanism where one class acquires accessible behavior and structure from another class, allowing the child class to reuse or specialize the parent class.**

Java uses:

```java
class Child extends Parent {
}
```

Inheritance should generally represent an **is-a** relationship.

```text
SavingsAccount IS-A Account
```

---

# 2. What Is Actually Inherited?

It is incorrect to say that a child gets everything from its parent.

Access modifiers matter.

```java
class Parent {
    private int a;
    int b;
    protected int c;
    public int d;
}
```

A subclass cannot directly access the `private` field.

```text
private          → not directly accessible
package-private  → accessible within same package
protected        → accessible according to protected rules
public           → accessible wherever the type is accessible
```

Important:

> **Not directly accessible does not mean the parent state does not exist as part of the object.**

---

# 3. Parent and Child Parts of an Object

```java
class Account {

    private BigDecimal balance;

    public void deposit(BigDecimal amount) {
        balance = balance.add(amount);
    }
}

class SavingsAccount extends Account {

    public void addInterest(BigDecimal amount) {
        deposit(amount);
    }
}
```

Conceptually, a `SavingsAccount` object contains the state associated with its superclass as well as its own state.

```text
SavingsAccount object
┌─────────────────────────┐
│ Account portion         │
│ private balance         │
├─────────────────────────┤
│ SavingsAccount portion  │
│ child-specific state    │
└─────────────────────────┘
```

This is a conceptual model; exact JVM object layout is implementation-dependent.

---

# 4. `super`

## Definition

**`super` is used by a subclass to refer to its immediate superclass.**

Important forms:

```java
super.field
super.method()
super(...)
```

### `super.field`

```java
class Parent {
    int value = 10;
}

class Child extends Parent {
    int value = 20;

    void print() {
        System.out.println(value);
        System.out.println(super.value);
    }
}
```

Output:

```text
20
10
```

`value` refers to the child's field; `super.value` explicitly refers to the parent's field.

This is **field hiding**, not method overriding.

---

# 5. `super.method()`

```java
class Parent {
    void print() {
        System.out.println("Parent");
    }
}

class Child extends Parent {

    @Override
    void print() {
        System.out.println("Child");
    }

    void test() {
        print();
        super.print();
    }
}
```

Output:

```text
Child
Parent
```

`print()` uses normal instance-method dispatch.

`super.print()` explicitly invokes the parent implementation.

A common pattern is to extend parent behavior:

```java
class Account {

    void validate() {
        System.out.println("Common validation");
    }
}

class LoanAccount extends Account {

    @Override
    void validate() {
        super.validate();
        System.out.println("Loan-specific validation");
    }
}
```

---

# 6. Constructor Chaining

When a child object is created, the superclass constructor is invoked as part of construction.

```java
class Parent {
    Parent() {
        System.out.println("Parent");
    }
}

class Child extends Parent {
    Child() {
        System.out.println("Child");
    }
}
```

```java
new Child();
```

prints:

```text
Parent
Child
```

If the child constructor does not explicitly invoke a superclass constructor, Java implicitly inserts:

```java
super();
```

Conceptually:

```java
Child() {
    super();
    System.out.println("Child");
}
```

Chain:

```text
new Child()
    ↓
Child()
    ↓
super()
    ↓
Parent()
    ↓
Child constructor body
```

---

# 7. Why Does the Parent Constructor Run?

A child object also contains the superclass state that needs initialization.

Conceptually:

```text
new Child()
     ↓
initialize Parent state
     ↓
Parent constructor
     ↓
initialize Child state
     ↓
Child constructor body
```

Therefore superclass construction happens before the child constructor body.

---

# 8. Parameterized Parent Constructor

```java
class Parent {
    Parent(int id) {
        System.out.println(id);
    }
}

class Child extends Parent {

    Child() {
        System.out.println("Child");
    }
}
```

This does **not compile** because Java tries to insert:

```java
super();
```

but `Parent` has no accessible no-argument constructor.

Fix:

```java
class Child extends Parent {

    Child() {
        super(100);
        System.out.println("Child");
    }
}
```

Chain:

```text
new Child()
    ↓
Child()
    ↓
super(100)
    ↓
Parent(int)
    ↓
Child constructor body
```

---

# 9. No-Argument Constructor vs Default Constructor

A no-argument constructor is any constructor with zero parameters.

A **default constructor** specifically means the no-argument constructor supplied by the compiler when the class declares no constructor.

Therefore:

```java
Parent() {
    System.out.println("Parent");
}
```

is an explicitly written no-argument constructor, not a compiler-generated default constructor.

---

# 10. Constructor Selection

If the child explicitly writes:

```java
Child() {
    super(10);
}
```

then `Parent(int)` is selected.

If there is no explicit `super(...)`, Java attempts:

```java
super();
```

Therefore:

```text
No explicit super(...)
        ↓
implicit super()
        ↓
requires accessible no-arg parent constructor

Explicit super(arguments)
        ↓
matching parent constructor
```

Constructor selection happens at compile time.

---

# 11. `this()` vs `super()`

```text
this(...)
   ↓
another constructor in the SAME class

super(...)
   ↓
constructor in the PARENT class
```

Constructor chaining ultimately reaches `Object`, directly or indirectly, because every Java class derives from `Object`.

---

# 12. Method Overriding

## Definition

**Method overriding occurs when a subclass provides its own implementation of an inherited instance method with the same signature as the superclass method.**

```java
class PaymentProcessor {

    void process() {
        System.out.println("Generic payment");
    }
}

class UpiPaymentProcessor extends PaymentProcessor {

    @Override
    void process() {
        System.out.println("UPI payment");
    }
}
```

`@Override` is recommended because the compiler verifies that the method is actually overriding a superclass method.

---

# 13. Runtime Method Dispatch

```java
PaymentProcessor processor = new UpiPaymentProcessor();

processor.process();
```

There are two types:

```text
Reference type                Actual object type
     ↓                              ↓
PaymentProcessor          UpiPaymentProcessor
```

The **reference type** determines what the compiler allows you to call.

The **actual runtime object type** determines which overridden instance method implementation executes.

```text
Compile time:
PaymentProcessor has process()
        ↓
call is valid

Runtime:
actual object = UpiPaymentProcessor
        ↓
UpiPaymentProcessor.process() executes
```

This is the foundation of runtime polymorphism / dynamic method dispatch.

---

# 14. Reference Type vs Actual Object Type

```java
PaymentProcessor processor = new UpiPaymentProcessor();
```

The compiler sees `PaymentProcessor`.

The runtime object is `UpiPaymentProcessor`.

Therefore:

```java
processor.process();   // ✅
processor.refund();    // ✅ if parent has refund()
processor.verifyUpi(); // ❌ if only child has verifyUpi()
```

The third call fails at compile time because the reference type does not expose `verifyUpi()`.

---

# 15. Upcasting

## Definition

**Upcasting means treating a child object as a reference to its parent type.**

```java
UpiPaymentProcessor upi = new UpiPaymentProcessor();

PaymentProcessor processor = upi;
```

This is safe because:

```text
UpiPaymentProcessor IS-A PaymentProcessor
```

It normally does not require an explicit cast.

---

# 16. Downcasting

## Definition

**Downcasting means converting a parent reference to a more specific child reference.**

```java
PaymentProcessor processor = new UpiPaymentProcessor();

UpiPaymentProcessor upi =
        (UpiPaymentProcessor) processor;
```

This works because the actual object really is a `UpiPaymentProcessor`.

The cast changes the compile-time type of the reference; it does not change the object.

---

# 17. Unsafe Downcasting

```java
PaymentProcessor processor = new PaymentProcessor();

UpiPaymentProcessor upi =
        (UpiPaymentProcessor) processor;
```

This compiles because the types are related, but fails at runtime with:

```text
ClassCastException
```

Rule:

> **A downcast is safe only when the actual object is compatible with the target child type.**

---

# 18. `instanceof`

Before downcasting:

```java
if (processor instanceof UpiPaymentProcessor) {
    UpiPaymentProcessor upi =
            (UpiPaymentProcessor) processor;

    upi.verifyUpi();
}
```

Modern Java also supports:

```java
if (processor instanceof UpiPaymentProcessor upi) {
    upi.verifyUpi();
}
```

Core idea:

```text
Check actual runtime type
        ↓
Downcast only when compatible
```

---

# 19. Static Methods — Method Hiding

Static methods belong to the class rather than an individual object.

```java
class Parent {

    static void show() {
        System.out.println("Parent");
    }
}

class Child extends Parent {

    static void show() {
        System.out.println("Child");
    }
}
```

This is **method hiding**, not overriding.

Static methods do not participate in runtime polymorphism like normal overridable instance methods.

---

# 20. Private Methods and Overriding

```java
class Parent {

    private void test() {
        System.out.println("Parent");
    }
}

class Child extends Parent {

    void test() {
        System.out.println("Child");
    }
}
```

The child's `test()` is **not an override** of the parent's private method.

The parent method is not accessible to the subclass.

---

# 21. Final Methods

A `final` instance method cannot be overridden.

```java
class Parent {

    final void run() {
        System.out.println("Parent");
    }
}

class Child extends Parent {

    // void run() {} // ❌ compile error
}
```

`final` prevents subclasses from replacing the implementation.

---

# 22. Field Hiding vs Method Overriding

Fields:

```java
class Parent {
    int value = 10;
}

class Child extends Parent {
    int value = 20;
}
```

This is **field hiding**.

Methods:

```java
class Parent {
    void print() {
        System.out.println("Parent");
    }
}

class Child extends Parent {

    @Override
    void print() {
        System.out.println("Child");
    }
}
```

This is **method overriding** and participates in runtime dispatch.

Mental model:

```text
Fields
  ↓
Field hiding
  ↓
Not runtime polymorphism

Overridable instance methods
  ↓
Method overriding
  ↓
Runtime polymorphism
```

---

# 23. Interview Table

| Member | Can Child Override It? | Important Behavior |
|---|---:|---|
| Normal instance method | ✅ | Runtime dispatch |
| `static` method | ❌ | Method hiding |
| `private` method | ❌ | Not accessible to subclass |
| `final` method | ❌ | Explicitly cannot be overridden |
| Field | ❌ | Field hiding, not overriding |

---

# 24. Core Mental Model

For:

```java
PaymentProcessor processor =
        new UpiPaymentProcessor();
```

think:

```text
                 processor
                     │
                     ▼
          UpiPaymentProcessor object
                     │
        ┌────────────┴────────────┐
        │                         │
 reference type              actual object
        │                         │
 PaymentProcessor          UpiPaymentProcessor
        │                         │
        ▼                         ▼
What can I call?           Which overridden
Compile time               method executes?
                           Runtime
```

This connects:

```text
Inheritance
      ↓
Overriding
      ↓
Polymorphism
      ↓
Interfaces
      ↓
Dependency Injection
      ↓
Spring
      ↓
LLD
```

---

# 25. Deferred Topics

We intentionally did not go deep into:

- JVM-specific object memory layout
- JLS-level inheritance edge cases
- Bridge methods
- Synthetic methods
- Bytecode invocation instructions
- Advanced reflection behavior
- Multiple inheritance internals

These can be revisited later if an interview requires them.

---

# 26. Interview Traps Covered

### Trap 1
“Child inherits everything from parent.”

❌ Too broad. Access modifiers matter.

### Trap 2
“Private methods are overridden.”

❌ No. They are not accessible to the subclass.

### Trap 3
“Static methods are overridden.”

❌ They are hidden.

### Trap 4
“Final means the method cannot be called.”

❌ `final` prevents overriding; it can still be called.

### Trap 5
“Downcasting always works because classes are related.”

❌ It can cause `ClassCastException`.

### Trap 6
“Reference type determines which implementation executes.”

❌ Reference type determines compile-time accessibility; runtime object type determines the overridden instance method implementation.

### Trap 7
“Fields are polymorphic like methods.”

❌ Fields are hidden; they do not participate in runtime method dispatch.

---

# 27. Senior-Level Takeaways

1. Inheritance should represent a meaningful **is-a** relationship.
2. Private state can exist in the parent portion of a child object while remaining inaccessible to the child.
3. `super` explicitly accesses the immediate superclass's field, method, or constructor.
4. Superclass construction occurs before the child constructor body.
5. If no explicit `super(...)` is present, Java attempts an implicit `super()`.
6. A parent with only parameterized constructors requires the child to explicitly invoke one.
7. Overridable instance methods participate in runtime polymorphism.
8. Reference type controls compile-time accessibility.
9. Runtime object type controls overridden instance-method dispatch.
10. Upcasting is generally safe.
11. Downcasting requires runtime compatibility and can throw `ClassCastException`.
12. `instanceof` can guard a downcast.
13. Static methods are hidden, not overridden.
14. Private methods cannot be overridden.
15. Final methods cannot be overridden.
16. Fields are hidden rather than overridden.
17. These concepts become foundational for interfaces, dependency injection, Spring, and LLD.

---

# 28. Current Preparation Status

```text
Classes / Objects / Constructors    🟢
Encapsulation                       🟢
Inheritance                        🟢
Polymorphism                       🔄 NEXT
Interfaces                         ⏳
Abstract Classes                   ⏳
equals / hashCode                  ⏳
Collections                        ⏳
Generics                           ⏳
Exceptions                         ⏳
Java 8+                            ⏳
Concurrency                        ⏳
Async Java                         ⏳
```

## Next Topic

**Polymorphism**

We will use the payment-processing problem from the BharatPe interview:

> “Design a simple Java class hierarchy for different payment types.”

The goal is to practice:

```text
Requirements
    ↓
Abstraction
    ↓
Interface / abstract class
    ↓
Concrete implementations
    ↓
Collection choice
    ↓
Extensibility
    ↓
Design pattern only when justified
```

This specifically addresses the tendency to jump directly to “Strategy Pattern” before establishing the actual requirements and object model.
