# G. Interfaces — Answers

### 116. What is an interface?

An **interface is a reference type that defines a contract that implementing classes agree to follow**.

```java
interface Payment {
    void pay(double amount);
}
```

A class implements it:

```java
class UpiPayment implements Payment {

    @Override
    public void pay(double amount) {
        System.out.println("Processing UPI");
    }
}
```

Interfaces are also important for **abstraction, polymorphism, loose coupling, and dependency inversion**.

---

### 117. Why do we use interfaces?

Interfaces are used to:

* Define contracts.
* Achieve abstraction.
* Support multiple inheritance of type.
* Enable runtime polymorphism.
* Reduce coupling.
* Make implementations replaceable.
* Improve testability.

Example:

```java
Payment payment = new UpiPayment();
```

Later:

```java
Payment payment = new CardPayment();
```

The calling code can depend on `Payment` rather than a concrete implementation.

---

### 118. Interface vs abstract class?

| Interface                                      | Abstract Class                                    |
| ---------------------------------------------- | ------------------------------------------------- |
| Defines a contract/capability                  | Defines a common base abstraction                 |
| A class can implement multiple interfaces      | A class can extend only one class                 |
| No constructors                                | Can have constructors                             |
| No instance fields                             | Can have instance fields                          |
| Fields are `public static final`               | Fields can have any permitted modifier            |
| Methods can be abstract/default/static/private | Can have abstract and concrete methods            |
| Useful for loosely coupled contracts           | Useful when subclasses share state/implementation |

Example:

```java
interface Flyable {
    void fly();
}
```

vs.

```java
abstract class Animal {
    protected String name;

    abstract void sound();

    void eat() {
        System.out.println("Eating");
    }
}
```

---

### 119. Can an interface have variables?

**Yes.**

```java
interface Config {
    int MAX_RETRIES = 3;
}
```

Interface fields are implicitly:

```java
public static final
```

So this:

```java
int MAX_RETRIES = 3;
```

is equivalent to:

```java
public static final int MAX_RETRIES = 3;
```

---

### 120. What is the default modifier of an interface variable?

Every field declared in an interface is implicitly:

```java
public static final
```

Example:

```java
interface Test {
    int VALUE = 10;
}
```

is equivalent to:

```java
interface Test {
    public static final int VALUE = 10;
}
```

Therefore, it is a **constant**, not an instance variable.

---

### 121. Can an interface have a constructor?

**No.**

Interfaces cannot be instantiated, so they don't have constructors.

```java
interface Payment {
    // Payment() {} // invalid
}
```

An implementing class has the constructor:

```java
class UpiPayment implements Payment {

    UpiPayment() {
    }
}
```

---

### 122. Can an interface have static methods?

**Yes.**

Since Java 8, interfaces can contain static methods.

```java
interface Payment {

    static boolean isValid(double amount) {
        return amount > 0;
    }
}
```

Call it using the interface name:

```java
Payment.isValid(100);
```

Static interface methods are **not inherited by implementing classes** in the same way instance methods are.

---

### 123. Can an interface have default methods?

**Yes.**

Default methods were introduced in Java 8.

```java
interface Payment {

    default void log() {
        System.out.println("Payment logged");
    }
}
```

An implementing class automatically gets the default implementation unless it overrides it.

---

### 124. Can an interface have private methods?

**Yes.**

Private interface methods were introduced in **Java 9**.

```java
interface Payment {

    default void process() {
        validate();
        System.out.println("Processing");
    }

    private void validate() {
        System.out.println("Validating");
    }
}
```

Private methods are useful for sharing implementation between default/static methods **inside the interface**.

They cannot be accessed or overridden by implementing classes.

---

### 125. Why were default methods introduced in Java 8?

Primarily to allow interfaces to **evolve without breaking existing implementations**.

Suppose Java originally had:

```java
interface Payment {
    void pay();
}
```

Many classes implement it:

```java
class UpiPayment implements Payment {
    public void pay() {}
}

class CardPayment implements Payment {
    public void pay() {}
}
```

Now suppose a new method needs to be added:

```java
void refund();
```

If it were abstract, every existing implementation would have to implement it.

A default method allows:

```java
default void refund() {
    // default implementation
}
```

Existing implementations can continue compiling without immediately implementing the new method.

---

### 126. How do default methods provide backward compatibility?

Consider an existing interface:

```java
interface Vehicle {
    void start();
}
```

Many existing classes implement it.

Java adds:

```java
default void stop() {
    System.out.println("Stopping");
}
```

Existing classes don't have to immediately implement `stop()`.

```java
class Car implements Vehicle {
    public void start() {
    }
}
```

`Car` automatically gets the default `stop()` implementation.

Therefore, adding a default method doesn't force every existing implementation to change.

---

### 127. What happens if a class inherits the same default method from two interfaces?

There is a **default-method conflict**, resulting in a compile-time error unless the class resolves it.

```java
interface A {
    default void print() {
        System.out.println("A");
    }
}

interface B {
    default void print() {
        System.out.println("B");
    }
}

class C implements A, B {
    // Compile-time error
}
```

The class must override:

```java
class C implements A, B {

    @Override
    public void print() {
        System.out.println("C");
    }
}
```

---

### 128. What happens if a superclass has a method and an interface has a default method with the same signature?

The superclass method takes precedence.

```java
class Parent {
    public void print() {
        System.out.println("Parent");
    }
}

interface A {
    default void print() {
        System.out.println("Interface");
    }
}

class Child extends Parent implements A {
}
```

Then:

```java
Child c = new Child();
c.print();
```

Output:

```text
Parent
```

---

### 129. Which one takes precedence: superclass method or interface default method?

**Superclass method takes precedence over an interface default method.**

The commonly remembered rule is:

> **Class wins over interface.**

Example:

```java
class Parent {
    public void test() {
        System.out.println("Parent");
    }
}

interface A {
    default void test() {
        System.out.println("A");
    }
}

class Child extends Parent implements A {
}
```

`Child.test()` uses `Parent.test()`.

This rule helps avoid ambiguity when an inherited class method and an interface default method provide the same behavior.

---

### 130. Can an interface extend another interface?

**Yes.**

```java
interface Animal {
    void eat();
}

interface Dog extends Animal {
    void bark();
}
```

`Dog` inherits the contract of `Animal`.

A class implementing `Dog` must satisfy both:

```java
class Labrador implements Dog {

    public void eat() {
    }

    public void bark() {
    }
}
```

---

### 131. Can an interface extend multiple interfaces?

**Yes.**

```java
interface A {
    void a();
}

interface B {
    void b();
}

interface C extends A, B {
    void c();
}
```

This is one way Java supports multiple inheritance of **type**.

---

### 132. Can a class implement multiple interfaces?

**Yes.**

```java
interface Flyable {
    void fly();
}

interface Swimmable {
    void swim();
}

class Duck implements Flyable, Swimmable {

    public void fly() {
    }

    public void swim() {
    }
}
```

A class can implement multiple interfaces.

---

### 133. Can an interface extend a class?

**No.**

An interface can only:

```java
extends interface(s)
```

It cannot:

```java
extends class
```

Example:

```java
interface A extends B {
}
```

where `B` must be an interface.

A class uses:

```java
class C extends A
```

and implements interfaces using:

```java
class C implements A
```

---

### 134. Can an interface have private static methods?

**Yes.**

Since Java 9, interfaces can have private static methods.

```java
interface Utility {

    static void process() {
        validate();
    }

    private static void validate() {
        System.out.println("Validating");
    }
}
```

The private static method can only be called from within the interface.

It cannot be called as:

```java
Utility.validate(); // invalid
```

---

### 135. Can an interface be instantiated?

**No.**

This is invalid:

```java
Payment p = new Payment();
```

because an interface doesn't provide a concrete object implementation by itself.

But you can instantiate an implementing class:

```java
Payment p = new UpiPayment();
```

---

### 136. Can an interface reference point to an implementation object?

**Yes.**

This is a fundamental use of interfaces and runtime polymorphism.

```java
Payment payment = new UpiPayment();
```

Here:

```text
Reference type → Payment
Actual object  → UpiPayment
```

You can call methods declared by `Payment`:

```java
payment.pay(1000);
```

At runtime, the overridden `UpiPayment.pay()` implementation executes.

---

### 137. Why is programming to an interface useful in backend systems?

It reduces coupling between your business logic and concrete implementations.

For example:

```java
interface PaymentService {
    void processPayment();
}
```

Implementation:

```java
class UpiPaymentService implements PaymentService {
    public void processPayment() {
    }
}
```

Your business service depends on:

```java
private final PaymentService paymentService;
```

rather than:

```java
private final UpiPaymentService paymentService;
```

This provides several benefits:

* **Loose coupling**
* Easy replacement of implementations.
* Easier unit testing/mocking.
* Better adherence to dependency inversion.
* Cleaner architecture.
* Easier extension.

For example, you can have:

```text
PaymentService
      |
 ┌────┴─────┐
 ↓          ↓
UPI       Card
```

The business layer doesn't need to know which concrete payment implementation it is using.

### Key revision points

```text
Interface
├── Contract / abstraction
├── Multiple interfaces can be implemented
├── Interface fields → public static final
├── Constructor → No
├── static methods → Yes
├── default methods → Yes (Java 8)
└── private methods → Yes (Java 9)

Default-method conflict
→ implementing class must resolve it

Superclass method vs interface default
→ superclass method wins

Interface reference
→ can point to implementation object
→ enables runtime polymorphism
```
