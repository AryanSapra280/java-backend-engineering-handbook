# E. Polymorphism — Answers

### 74. What is polymorphism?

**Polymorphism means "one interface/reference, multiple forms."**

In Java, the same method call can behave differently depending on the actual object.

```java
Animal a = new Dog();
a.sound();   // Dog's sound()
```

The reference type is `Animal`, but the actual object is `Dog`.

---

### 75. What are the types of polymorphism in Java?

Two commonly discussed types:

1. **Compile-time polymorphism** → method overloading
2. **Runtime polymorphism** → method overriding

```text
Polymorphism
├── Compile-time → Overloading
└── Runtime      → Overriding
```

---

### 76. What is compile-time polymorphism?

Compile-time polymorphism occurs when the compiler determines which method to call during compilation.

The main example is **method overloading**.

```java
void print(int x) {}
void print(String x) {}
```

The compiler determines the appropriate method based on the arguments.

---

### 77. What is runtime polymorphism?

Runtime polymorphism occurs when the method implementation to execute is determined based on the **actual object at runtime**.

It is achieved through **method overriding**.

```java
class Animal {
    void sound() {
        System.out.println("Animal");
    }
}

class Dog extends Animal {
    @Override
    void sound() {
        System.out.println("Dog");
    }
}

Animal a = new Dog();
a.sound(); // Dog
```

---

### 78. What is method overloading?

Method overloading means having multiple methods with the **same name but different parameter lists** within a class/inheritance context.

```java
void add(int a, int b) {}

void add(int a, int b, int c) {}

void add(double a, double b) {}
```

The parameter list must differ.

Changing only the return type is **not** overloading.

---

### 79. What is method overriding?

Method overriding occurs when a subclass provides its own implementation of an inherited, overridable instance method with a compatible signature.

```java
class Animal {
    void sound() {
        System.out.println("Animal");
    }
}

class Dog extends Animal {
    @Override
    void sound() {
        System.out.println("Dog");
    }
}
```

The subclass version executes for a `Dog` object.

---

### 80. What is dynamic method dispatch?

**Dynamic method dispatch is the mechanism by which Java selects an overridden instance method at runtime based on the actual object.**

```java
Animal animal = new Dog();
animal.sound();
```

Although the reference is `Animal`, the actual object is `Dog`, so `Dog.sound()` executes.

---

### 81. How does Java achieve runtime polymorphism?

Through:

1. Inheritance
2. Method overriding
3. A superclass/interface reference referring to a subclass object.

Example:

```java
Animal animal = new Dog();
animal.sound();
```

The compiler verifies that `sound()` exists in `Animal`, while runtime dispatch selects the implementation belonging to the actual object.

---

### 82. When is an overloaded method resolved?

**At compile time.**

The compiler examines:

* Number of arguments
* Argument types
* Applicable conversions
* Method signatures
* Most-specific applicable method

Example:

```java
void print(int x) {}
void print(String x) {}

print(10);
```

The compiler selects `print(int)`.

---

### 83. When is an overridden method resolved?

The **implementation of an overridden instance method is selected at runtime**, based on the actual object's class.

```java
Animal a = new Dog();
a.sound();
```

`Dog.sound()` executes.

The compiler still checks the method against the **reference type**.

---

### 84. What happens internally when a superclass reference points to a subclass object?

Consider:

```java
Animal a = new Dog();
```

There are two different concepts:

```text
Reference type              Actual object
     Animal                    Dog
       │                        │
       └──────────────►─────────┘
```

The reference variable `a` is typed as `Animal`, so at compile time you can directly access only members available through `Animal`.

But the object created is a `Dog`.

Therefore, for an overridden instance method:

```java
a.sound();
```

Java uses runtime dispatch to execute `Dog.sound()`.

---

### 85. How does JVM determine which overridden method should execute?

At a high level, JVM uses **runtime method dispatch** based on the actual object's class.

```java
Animal a = new Dog();
a.sound();
```

Conceptually:

```text
Reference type → Animal
                  ↓
Runtime object → Dog
                  ↓
Find overridden implementation
                  ↓
Dog.sound()
```

The exact JVM implementation is not something Java code should depend on. HotSpot, for example, can use mechanisms such as **virtual method tables and runtime optimizations**, but the Java-level guarantee is simply that the most specific applicable overridden instance method is invoked.

---

### 86. Can static methods be overridden?

**No.**

Static methods belong to the **class**, not to individual objects.

They are **hidden**, not overridden.

```java
class Parent {
    static void print() {
        System.out.println("Parent");
    }
}

class Child extends Parent {
    static void print() {
        System.out.println("Child");
    }
}
```

This is method hiding.

---

### 87. If static methods cannot be overridden, what happens when a subclass defines a static method with the same signature?

The subclass **hides** the superclass static method.

The method selected depends on the **reference/class type**, not the runtime object.

```java
class Parent {
    static void print() {
        System.out.println("Parent");
    }
}

class Child extends Parent {
    static void print() {
        System.out.println("Child");
    }
}

Parent p = new Child();
p.print();
```

Output:

```text
Parent
```

Because `print()` is static and is resolved based on the reference type.

---

### 88. What is method hiding?

**Method hiding occurs when a subclass declares a static method with the same signature as a static method in its superclass.**

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

This is hiding, not overriding.

---

### 89. Method overriding vs method hiding?

| Overriding                  | Hiding                        |
| --------------------------- | ----------------------------- |
| Applies to instance methods | Applies to static methods     |
| Runtime polymorphism        | No runtime polymorphism       |
| Based on actual object      | Based on reference/class type |
| Dynamic dispatch            | Static/class-based selection  |
| `@Override` can be used     | It is not overriding          |

Example:

```java
Parent p = new Child();

p.instanceMethod(); // Child implementation
p.staticMethod();   // Parent implementation
```

---

### 90. Can private methods be overridden?

**No.**

Private methods are not inherited by subclasses and therefore cannot be overridden.

```java
class Parent {
    private void test() {}
}

class Child extends Parent {
    private void test() {}
}
```

The `test()` methods are two independent methods, not an overridden pair.

---

### 91. Can final methods be overridden?

**No.**

`final` prevents overriding.

```java
class Parent {
    final void print() {}
}

class Child extends Parent {
    // void print() {} // Compile-time error
}
```

---

### 92. Can constructors be overridden?

**No.**

Constructors cannot be overridden.

They can, however, be **overloaded**.

```java
class Employee {

    Employee() {}

    Employee(String name) {}
}
```

---

### 93. Why can't constructors be overridden?

Because constructors are **not inherited**.

Overriding requires:

```text
Parent method
     ↓ inherited
Child provides replacement
```

Constructors don't participate in inheritance. They are specifically responsible for initializing an object of their own class.

A subclass constructor can invoke a superclass constructor using:

```java
super();
```

but that is **constructor invocation**, not overriding.

---

### 94. Can an overloaded method differ only by return type?

**No.**

This is invalid:

```java
int getValue() {
    return 10;
}

String getValue() {
    return "10";
}
```

The parameter lists are identical.

---

### 95. Why doesn't return type alone participate in method overloading?

Because the compiler must determine the method to invoke from the **method invocation expression**.

Consider:

```java
getValue();
```

There is no argument information that could distinguish:

```java
int getValue()
```

from:

```java
String getValue()
```

Allowing return type alone would create ambiguity, especially when the return value is ignored.

Therefore, Java requires overloading to differ in the **parameter list**.

---

### 96. How does Java choose between overloaded methods?

The compiler performs **overload resolution**.

Broadly, it:

1. Finds methods with the matching name.
2. Determines which are applicable to the supplied arguments.
3. Considers allowed conversions.
4. Selects the **most specific applicable method**.

For example:

```java
void test(int x) {}
void test(long x) {}

test(10);
```

`int` is an exact match, so:

```java
test(int)
```

is selected.

---

### 97. What happens when overloaded methods involve primitive widening?

Primitive widening can be used during overload resolution.

```java
void test(long x) {
    System.out.println("long");
}

void test(double x) {
    System.out.println("double");
}

int x = 10;
test(x);
```

`int → long` and `int → double` are both widening conversions.

Java chooses the **more specific applicable conversion**, so `test(long)` is selected.

Important common widening sequence:

```text
byte → short → int → long → float → double
```

Also:

```text
char → int → long → float → double
```

---

### 98. What happens when overloaded methods involve boxing?

Boxing converts a primitive into its wrapper type.

```java
void test(int x) {
    System.out.println("int");
}

void test(Integer x) {
    System.out.println("Integer");
}

test(10);
```

The `int` version is selected because an **exact primitive match is preferred over boxing**.

---

### 99. What happens when overloaded methods involve varargs?

Varargs are considered **after normal fixed-arity possibilities**.

```java
void test(int x) {
    System.out.println("int");
}

void test(int... x) {
    System.out.println("varargs");
}

test(10);
```

The fixed-arity `int` method wins.

If no applicable fixed-arity method exists:

```java
void test(int... x) {}

test(10, 20, 30);
```

the varargs method can be selected.

---

### 100. Which has priority during overload resolution: widening, boxing or varargs?

For the common case where alternatives are otherwise applicable, the preference is:

```text
1. Exact match
2. Primitive widening
3. Boxing / unboxing
4. Varargs
```

A classic example:

```java
void test(long x) {}
void test(Integer x) {}
void test(int... x) {}

test(10);
```

`int → long` is primitive widening, while `int → Integer` is boxing.

Therefore:

```java
test(long)
```

is selected.

**Important:** Java's actual overload resolution is governed by its phased applicability rules, so don't memorize this as a universal ranking of every possible conversion combination. The key interview rule is that **varargs is considered last**, and a method applicable without boxing/varargs is generally preferred to one requiring those later phases.

---

### 101. What happens when `null` is passed to overloaded methods?

`null` can be passed to a reference type, but not to a primitive type.

Example:

```java
void test(String s) {
    System.out.println("String");
}

void test(Object o) {
    System.out.println("Object");
}

test(null);
```

Both are applicable, but `String` is more specific than `Object`.

Therefore:

```text
String
```

is selected.

---

### 102. What happens if two overloaded methods are equally specific for `null`?

The call becomes **ambiguous**, resulting in a compile-time error.

Example:

```java
void test(String s) {}
void test(Integer i) {}

test(null);
```

Both `String` and `Integer` can accept `null`, but neither is a subtype of the other.

Therefore the compiler cannot choose.

```text
String
   ↘
    ?  ← null
   ↗
Integer
```

Result:

```text
Compile-time error: reference to test is ambiguous
```

### Key revision point

For overloaded methods, remember:

```text
Overloading  → compile time
Overriding   → runtime
Static       → hiding, not overriding
Private      → cannot override
Final        → cannot override
Constructor  → cannot override

Overload resolution:
exact match → widening → boxing → varargs
```
