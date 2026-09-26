# C. Abstraction — Answers

### 36. What is abstraction?

**Abstraction is the process of exposing only the essential details of an object while hiding unnecessary implementation details.**

Example:

```java
interface Payment {
    void pay(double amount);
}
```

The caller knows **what** operation is available (`pay()`), but doesn't need to know how payment is processed internally.

---

### 37. Why do we need abstraction?

Abstraction helps us:

* Hide implementation complexity.
* Expose only relevant functionality.
* Reduce coupling between components.
* Make code easier to change and maintain.
* Define clear contracts/APIs.
* Allow different implementations behind the same interface.

Example:

```java
Payment payment = new UpiPayment();
payment.pay(1000);
```

The caller depends on `Payment`, not the implementation details of UPI processing.

---

### 38. How is abstraction achieved in Java?

Primarily through:

### 1. Abstract classes

```java
abstract class Vehicle {
    abstract void start();

    void stop() {
        System.out.println("Stopping");
    }
}
```

### 2. Interfaces

```java
interface Payment {
    void pay(double amount);
}
```

Interfaces are generally used to define contracts, while abstract classes can provide both abstraction and shared implementation/state.

---

### 39. What is the difference between abstraction and encapsulation?

**Abstraction = what to expose/hide.**

**Encapsulation = how to protect and control state/implementation.**

Example:

```java
class BankAccount {

    private double balance;  // Encapsulation

    public void withdraw(double amount) {  // Abstraction of operation
        // implementation
    }
}
```

Think:

> **Abstraction → hides complexity.**
> **Encapsulation → protects internal state and controls access.**

They are related but not the same concept.

---

### 40. Can abstraction be achieved without an abstract class?

**Yes.**

The most common way is through an **interface**.

```java
interface Payment {
    void pay(double amount);
}

class UpiPayment implements Payment {

    @Override
    public void pay(double amount) {
        System.out.println("Processing UPI payment");
    }
}
```

The interface provides the abstraction without requiring an abstract class.

Abstraction can also be achieved through normal classes by exposing a simple public API and hiding implementation details.

---

### 41. Can an abstract class have concrete methods?

**Yes.**

An abstract class can contain both abstract and concrete methods.

```java
abstract class Animal {

    abstract void sound();

    void eat() {
        System.out.println("Eating");
    }
}
```

Here:

* `sound()` → abstract method
* `eat()` → concrete method

This allows an abstract class to provide **common implementation** while leaving some behavior to subclasses.

---

### 42. Can an abstract class have variables?

**Yes.**

It can contain:

* instance variables
* static variables
* final variables
* non-final variables

Example:

```java
abstract class Employee {

    protected String name;
    private int age;
    static int count;
    final String company = "ABC";
}
```

There is no requirement that variables in an abstract class must be `final` or `static`.

---

### 43. Can an abstract class have constructors?

**Yes.**

```java
abstract class Animal {

    Animal() {
        System.out.println("Animal constructor");
    }
}
```

A subclass constructor invokes it:

```java
class Dog extends Animal {

    Dog() {
        super();
    }
}
```

Even though you cannot directly instantiate `Animal`, its constructor participates in **initializing the `Animal` portion of a subclass object**.

---

### 44. Why does an abstract class need a constructor if it cannot be instantiated?

Because the constructor is used when a **subclass object is created**.

```java
abstract class Animal {

    String name;

    Animal(String name) {
        this.name = name;
    }
}

class Dog extends Animal {

    Dog(String name) {
        super(name);
    }
}
```

When:

```java
Dog d = new Dog("Bruno");
```

the `Animal` constructor executes as part of constructing the `Dog` object.

Conceptually:

```text
new Dog()
   ↓
Dog constructor
   ↓
Animal constructor
   ↓
Animal state initialized
   ↓
Dog state initialized
```

---

### 45. Can an abstract class contain static methods?

**Yes.**

```java
abstract class Utility {

    static void print() {
        System.out.println("Hello");
    }
}
```

You can call:

```java
Utility.print();
```

Static methods belong to the class, so they don't require an object.

Also, a static method **cannot be abstract**, because `abstract` requires implementation through overriding, while static methods are associated with the class rather than an instance.

---

### 46. Can an abstract class contain final methods?

**Yes.**

```java
abstract class Animal {

    final void breathe() {
        System.out.println("Breathing");
    }

    abstract void sound();
}
```

A subclass must implement `sound()`, but cannot override `breathe()`.

```java
class Dog extends Animal {

    @Override
    void sound() {
        System.out.println("Bark");
    }
}
```

A `final` method can therefore have a concrete implementation inside an abstract class.

---

### 47. Can an abstract class contain private methods?

**Yes.**

```java
abstract class Animal {

    private void log() {
        System.out.println("Logging");
    }

    abstract void sound();
}
```

A private method is accessible only inside the declaring class.

It cannot be overridden by subclasses.

Therefore, a method cannot meaningfully be both:

```java
private abstract void method();
```

because an abstract method must be implemented by a subclass, while a private method isn't accessible to subclasses.

---

### 48. Can an abstract class have zero abstract methods?

**Yes.**

This is completely valid:

```java
abstract class Utility {

    void print() {
        System.out.println("Hello");
    }
}
```

The class is abstract because the designer explicitly prevents direct instantiation.

There is **no requirement that an abstract class must contain an abstract method**.

---

### 49. Can an abstract class be declared `final`?

**No.**

These modifiers conflict:

```java
abstract final class A {
}
```

Why?

* `abstract` → class must be subclassed to create a concrete implementation.
* `final` → class cannot be subclassed.

Therefore, the two intentions contradict each other.

---

### 50. Why would you create an abstract class that has no abstract methods?

To **prevent direct instantiation** while still providing shared implementation/state to subclasses.

For example:

```java
abstract class BaseProcessor {

    protected void log(String message) {
        System.out.println(message);
    }

    protected void validate() {
        // common validation
    }
}
```

Subclasses can reuse these methods:

```java
class PaymentProcessor extends BaseProcessor {
}
```

The designer may want `BaseProcessor` to exist only as a **base type**, not as an independent object.

Another reason is to communicate a design constraint:

> "This class is intended only to be extended."

---

### 51. When would you choose an abstract class over an interface?

Choose an **abstract class** when closely related classes need to share:

* Common state.
* Common implementation.
* Constructors.
* Protected methods.
* Non-public members.
* A common base-class relationship.

Example:

```java
abstract class Employee {

    protected String name;

    Employee(String name) {
        this.name = name;
    }

    void login() {
        System.out.println("Login");
    }

    abstract void performWork();
}
```

Different employee types can inherit the common state and behavior.

Choose an **interface** when you primarily want to define a **contract/capability** that potentially unrelated classes can implement.

```java
interface Flyable {
    void fly();
}
```

A `Bird`, `Drone`, and `Airplane` can all implement `Flyable` despite not sharing a meaningful class hierarchy.

### Key interview distinction

| Abstract Class                                       | Interface                                                     |
| ---------------------------------------------------- | ------------------------------------------------------------- |
| Represents a common base/abstraction                 | Represents a contract/capability                              |
| Can have instance state                              | Fields are implicitly `public static final`                   |
| Can have constructors                                | No constructors                                               |
| Can have concrete methods                            | Can have `default`, `static`, and `private` methods           |
| A class can extend only one class                    | A class can implement multiple interfaces                     |
| Can have `protected`/package-private/private members | Interface methods/fields have specific interface access rules |

**Interview shortcut:**

> If I need **shared state + shared implementation + inheritance**, an abstract class is usually appropriate.
> If I need a **contract/capability that multiple unrelated classes can implement**, an interface is usually more appropriate.
