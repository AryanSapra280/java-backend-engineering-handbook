# D. Inheritance — Answers

### 52. What is inheritance?

**Inheritance is a mechanism where one class acquires the properties and behavior of another class.**

```java
class Animal {
    void eat() {
        System.out.println("Eating");
    }
}

class Dog extends Animal {
    void bark() {
        System.out.println("Barking");
    }
}
```

`Dog` inherits `eat()` from `Animal`.

---

### 53. Why is inheritance used?

Inheritance is used for:

* Code reuse.
* Representing an **IS-A** relationship.
* Method overriding and runtime polymorphism.
* Creating a common base abstraction.
* Extending existing behavior.

However, inheritance should not be used merely for code reuse; the subclass should genuinely represent a specialized form of the parent.

---

### 54. What types of inheritance does Java support?

For **classes**, Java supports:

1. **Single inheritance**

   ```text
   A
   |
   B
   ```

2. **Multilevel inheritance**

   ```text
   A
   |
   B
   |
   C
   ```

3. **Hierarchical inheritance**

   ```text
       A
      / \
     B   C
   ```

Java does **not** support multiple or hybrid inheritance of classes.

Through interfaces, Java can model multiple-inheritance-like type relationships.

---

### 55. Does Java support multiple inheritance?

**Java does not support multiple inheritance of classes.**

This is invalid:

```java
class C extends A, B {
}
```

However, a class can implement multiple interfaces:

```java
class C implements A, B {
}
```

So Java supports **multiple inheritance of type through interfaces**, but not multiple inheritance of class implementation.

---

### 56. Why doesn't Java support multiple inheritance of classes?

Primarily to avoid **ambiguity and complexity**, especially the diamond problem.

Suppose:

```text
      A
     / \
    B   C
     \ /
      D
```

If both `B` and `C` inherit/override a method from `A`, which implementation should `D` inherit?

Java avoids this ambiguity by allowing a class to extend only one class.

---

### 57. What is the diamond problem?

The **diamond problem** occurs when a class inherits from two classes that both inherit from the same superclass.

The inheritance structure forms a diamond:

```text
       A
      / \
     B   C
      \ /
       D
```

If `A` has a method and both `B` and `C` provide their own implementations, `D` may face ambiguity about which implementation to use.

---

### 58. Explain the diamond problem with an example.

Imagine a language allowed:

```java
class A {
    void print() {
        System.out.println("A");
    }
}

class B extends A {
    @Override
    void print() {
        System.out.println("B");
    }
}

class C extends A {
    @Override
    void print() {
        System.out.println("C");
    }
}
```

Now suppose:

```java
class D extends B, C {
}
```

What should happen here?

```java
D d = new D();
d.print();
```

Should it print:

```text
B
```

or:

```text
C
```

There is no obvious answer.

That's the **diamond problem**.

---

### 59. How does Java avoid the diamond problem?

Java prevents the problem at the class level by allowing:

```java
class D extends B
```

but not:

```java
class D extends B, C
```

For interfaces, Java has explicit rules for resolving default-method conflicts, and the implementing class can override the conflicting method.

---

### 60. Can Java achieve multiple inheritance using interfaces?

**Yes, in terms of implementing multiple types.**

```java
interface A {
    void methodA();
}

interface B {
    void methodB();
}

class C implements A, B {

    public void methodA() {
    }

    public void methodB() {
    }
}
```

A class can implement multiple interfaces:

```java
class C implements A, B, D {
}
```

This provides multiple inheritance of **type/contracts**, without multiple inheritance of classes.

---

### 61. What happens when two interfaces contain the same default method?

If a class implements both interfaces and both provide the same default method, there is a conflict.

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
```

Then:

```java
class C implements A, B {
}
```

This causes a **compile-time error** because Java cannot choose between `A.print()` and `B.print()`.

---

### 62. What happens when a class implements two interfaces having conflicting default methods?

The class must resolve the conflict by **overriding the method**.

```java
class C implements A, B {

    @Override
    public void print() {
        System.out.println("C");
    }
}
```

Otherwise, compilation fails.

---

### 63. How do you resolve a default-method conflict?

Override the method in the implementing class.

You can also explicitly choose one interface's default implementation using:

```java
InterfaceName.super.method();
```

Example:

```java
class C implements A, B {

    @Override
    public void print() {
        A.super.print();
    }
}
```

Now `A`'s default implementation is used.

You can also combine behavior:

```java
class C implements A, B {

    @Override
    public void print() {
        A.super.print();
        B.super.print();
    }
}
```

---

### 64. What is an IS-A relationship?

**IS-A means one type is a specialized form of another type.**

It is represented using inheritance.

```java
class Animal {
}

class Dog extends Animal {
}
```

A `Dog` **IS-A** `Animal`.

Therefore:

```java
Dog dog = new Dog();
Animal animal = dog;
```

This is valid.

---

### 65. What is a HAS-A relationship?

**HAS-A represents a composition/association relationship where one object contains or uses another object.**

Example:

```java
class Engine {
}

class Car {
    private Engine engine;
}
```

A `Car` **HAS-A** `Engine`.

---

### 66. How does inheritance represent an IS-A relationship?

Inheritance establishes a parent-child type relationship.

```java
class Vehicle {
}

class Car extends Vehicle {
}
```

Therefore:

```text
Car IS-A Vehicle
```

This allows:

```java
Vehicle v = new Car();
```

This is also the basis for **polymorphism**.

---

### 67. How does composition represent a HAS-A relationship?

Composition means an object contains another object as part of its implementation.

```java
class Engine {
    void start() {
    }
}

class Car {

    private final Engine engine;

    Car(Engine engine) {
        this.engine = engine;
    }
}
```

Here:

```text
Car
 |
 └── Engine
```

Therefore:

> Car HAS-A Engine.

The `Car` doesn't need to inherit from `Engine`.

---

### 68. What is the difference between inheritance and composition?

| Inheritance                      | Composition                     |
| -------------------------------- | ------------------------------- |
| IS-A                             | HAS-A                           |
| `extends`                        | Object reference/field          |
| Strong parent-child relationship | Objects collaborate             |
| Creates tighter coupling         | Usually looser coupling         |
| Behavior inherited               | Behavior delegated              |
| Compile-time class hierarchy     | Can be more flexible at runtime |

Example:

```java
class Dog extends Animal
```

vs.

```java
class Car {
    private Engine engine;
}
```

---

### 69. Why is composition often preferred over inheritance?

Because composition generally provides **more flexibility and lower coupling**.

With inheritance:

```java
class Car extends Vehicle
```

`Car` becomes strongly dependent on the design and behavior of `Vehicle`.

With composition:

```java
class Car {
    private Engine engine;
}
```

you can change or replace the `Engine` implementation more easily.

Composition also allows behavior to be assembled from multiple components without creating complicated inheritance hierarchies.

This is often summarized as:

> **Favor composition over inheritance.**

It doesn't mean inheritance is bad. Inheritance is appropriate when there is a genuine, stable **IS-A** relationship.

---

### 70. What problems can arise from deep inheritance hierarchies?

For example:

```text
A
|
B
|
C
|
D
|
E
|
F
```

Problems include:

* Difficult-to-understand behavior.
* Changes in parent classes can affect many subclasses.
* Tight coupling.
* Method overriding becomes difficult to trace.
* More difficult testing and debugging.
* Fragile base-class problems.
* Difficult maintenance.
* Subclasses may inherit behavior they don't actually need.

---

### 71. What is tight coupling in inheritance?

Inheritance creates coupling because a subclass depends on the **structure and behavior of its parent class**.

Example:

```java
class Parent {
    protected int value;
}

class Child extends Parent {
    void process() {
        value++;
    }
}
```

`Child` directly depends on the parent's internal field.

If `Parent` changes:

```java
private int value;
```

or changes how `value` should be managed, `Child` may need modification.

This dependency between parent and child is an example of tight coupling.

---

### 72. Give an example where inheritance looks appropriate initially but composition would be better.

Imagine:

```java
class Employee {
    void work() {
    }
}

class Developer extends Employee {
}

class Manager extends Employee {
}
```

Initially this looks fine.

But later you discover employees can have different capabilities:

* Coding
* Managing
* Testing
* Designing
* Mentoring

If you keep adding inheritance:

```text
Employee
 ├── Developer
 ├── Manager
 ├── Tester
 ├── DeveloperManager
 ├── DeveloperTester
 └── ManagerTester
```

the hierarchy becomes complicated.

Composition can model capabilities:

```java
interface Coding {
    void code();
}

interface Managing {
    void manage();
}
```

Then:

```java
class Employee {

    private Coding coding;
    private Managing managing;
}
```

Now capabilities can be combined without creating subclasses for every combination.

---

### 73. How would you refactor a large inheritance hierarchy?

A practical approach:

**1. Identify why inheritance exists.**

Determine whether each relationship is genuinely **IS-A** or is inheritance being used only for code reuse.

**2. Find duplicated/shared behavior.**

Move reusable behavior into separate components/services.

**3. Extract interfaces where appropriate.**

Define capabilities/contracts:

```java
interface PaymentProcessor {
    void process();
}
```

**4. Replace inheritance with composition.**

Instead of:

```java
class A extends B
```

consider:

```java
class A {
    private B b;
}
```

and delegate:

```java
b.process();
```

**5. Keep inheritance where polymorphic substitution is genuinely required.**

Don't blindly eliminate all inheritance.

### Interview answer:

> "I would first identify whether the inheritance relationships represent genuine IS-A relationships or are being used primarily for code reuse. For relationships that are not true IS-A relationships, I would extract common behavior into components or interfaces and use composition and delegation. I would also reduce deep hierarchies and keep inheritance only where substitutability and polymorphism provide a clear benefit."
