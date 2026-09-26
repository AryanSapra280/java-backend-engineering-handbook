# H. Constructors — Answers

### 138. What is a constructor?

A **constructor is a special member of a class used to initialize an object when it is created**.

```java
class Employee {
    private String name;

    Employee(String name) {
        this.name = name;
    }
}
```

It has:

* Same name as the class.
* No return type, not even `void`.
* It executes when an object is created.

---

### 139. Why do we need constructors?

Constructors are used to **initialize an object's initial state** and establish its required invariants.

```java
Employee e = new Employee("Aryan");
```

The constructor can ensure that the object starts with valid data.

```java
class Employee {
    private final String name;

    Employee(String name) {
        if (name == null || name.isBlank()) {
            throw new IllegalArgumentException();
        }
        this.name = name;
    }
}
```

---

### 140. What happens if you don't define a constructor?

If you don't declare **any constructor**, the compiler provides a **default constructor** with the same accessibility as the class.

For example:

```java
class Employee {
}
```

Conceptually becomes:

```java
class Employee {
    Employee() {
        super();
    }
}
```

Important: if you define **any constructor yourself**, Java does not automatically provide the no-argument constructor.

```java
class Employee {
    Employee(String name) {
    }
}

Employee e = new Employee(); // Compile-time error
```

---

### 141. Can constructors be overloaded?

**Yes.**

A class can have multiple constructors with different parameter lists.

```java
class Employee {

    Employee() {
    }

    Employee(String name) {
    }

    Employee(String name, int age) {
    }
}
```

Constructor overloading is resolved at **compile time**.

---

### 142. Can constructors be inherited?

**No.**

Constructors are **not inherited** by subclasses.

```java
class Parent {
    Parent(String name) {
    }
}

class Child extends Parent {
}
```

`Child` does not inherit `Parent(String)`.

The child must invoke an accessible parent constructor:

```java
class Child extends Parent {

    Child() {
        super("Aryan");
    }
}
```

---

### 143. Can constructors be overridden?

**No.**

Constructors cannot be overridden because they are not inherited.

They can only be **overloaded** within the same class.

---

### 144. Can a constructor be private?

**Yes.**

```java
class Employee {

    private Employee() {
    }
}
```

A private constructor can only be called from within the class (subject to Java's access rules).

Therefore:

```java
new Employee(); // Compile-time error from outside
```

---

### 145. When would you use a private constructor?

Common uses include:

### 1. Singleton pattern

```java
class Singleton {

    private static final Singleton INSTANCE = new Singleton();

    private Singleton() {
    }

    public static Singleton getInstance() {
        return INSTANCE;
    }
}
```

### 2. Utility classes

```java
final class MathUtils {

    private MathUtils() {
    }

    static int square(int x) {
        return x * x;
    }
}
```

This prevents creating unnecessary instances.

### 3. Controlling object creation

A class may expose factory methods instead:

```java
class User {

    private User(String name) {
    }

    public static User create(String name) {
        return new User(name);
    }
}
```

This allows the class to control or validate how objects are created.

---

### 146. What is constructor chaining?

**Constructor chaining means one constructor invokes another constructor, either within the same class or in the superclass.**

Within the same class:

```java
class Employee {

    Employee() {
        this("Unknown");
    }

    Employee(String name) {
        System.out.println(name);
    }
}
```

Across inheritance:

```java
class Parent {
    Parent() {
    }
}

class Child extends Parent {
    Child() {
        super();
    }
}
```

---

### 147. What is `this()`?

`this()` invokes **another constructor of the same class**.

```java
class Employee {

    Employee() {
        this("Unknown");
    }

    Employee(String name) {
        System.out.println(name);
    }
}
```

Here:

```java
this("Unknown");
```

calls the second constructor.

---

### 148. What is `super()`?

`super()` invokes the **constructor of the immediate superclass**.

```java
class Parent {

    Parent() {
        System.out.println("Parent");
    }
}

class Child extends Parent {

    Child() {
        super();
        System.out.println("Child");
    }
}
```

When `new Child()` is executed, the superclass constructor runs first.

---

### 149. Where must `this()` or `super()` appear inside a constructor?

If explicitly used, `this()` or `super()` must be the **first statement** in the constructor.

Valid:

```java
Child() {
    super();
    System.out.println("Child");
}
```

Invalid:

```java
Child() {
    System.out.println("Child");
    super(); // Compile-time error
}
```

---

### 150. Can `this()` and `super()` both be called explicitly in the same constructor?

**No.**

Both must be the first statement, so they cannot both appear explicitly.

```java
Child() {
    this();   // first statement
    super();  // unreachable/invalid
}
```

However, constructor chaining can happen indirectly:

```java
Child() {
    this(10);
}

Child(int x) {
    super();
}
```

Here `Child()` calls another child constructor, which eventually calls the superclass constructor.

---

### 151. What happens if you don't explicitly call `super()`?

The compiler implicitly inserts:

```java
super();
```

as the first statement, **provided the superclass has an accessible no-argument constructor**.

Example:

```java
class Parent {
    Parent() {
    }
}

class Child extends Parent {

    Child() {
        // compiler implicitly inserts super();
    }
}
```

But if the parent only has:

```java
Parent(String name) {
}
```

then:

```java
class Child extends Parent {
    Child() {
    }
}
```

causes a compilation error because Java tries to call `super()`, which doesn't exist.

---

### 152. What happens during superclass constructor execution?

When creating a subclass object, superclass construction happens before subclass construction.

Example:

```java
class Parent {

    int x;

    Parent() {
        x = 10;
    }
}

class Child extends Parent {

    int y;

    Child() {
        y = 20;
    }
}
```

When:

```java
new Child();
```

the superclass constructor initializes the **Parent portion/state** before the `Child` constructor initializes the subclass-specific state.

The object itself is allocated before constructor execution, and Java's initialization process ensures superclass initialization occurs before subclass initialization.

---

### 153. What is the order of constructor execution in inheritance?

Consider:

```java
class A {
    A() {
        System.out.println("A");
    }
}

class B extends A {
    B() {
        System.out.println("B");
    }
}

class C extends B {
    C() {
        System.out.println("C");
    }
}
```

```java
new C();
```

Output:

```text
A
B
C
```

So constructor execution proceeds:

```text
Object
  ↓
A
  ↓
B
  ↓
C
```

More completely, object creation involves superclass initialization first, including instance initialization and constructor execution, before moving down to the subclass.

---

### 154. What happens if a superclass constructor calls an overridable method?

The **subclass's overridden method can execute**, even though the subclass constructor has not yet run.

Example:

```java
class Parent {

    Parent() {
        print();
    }

    void print() {
        System.out.println("Parent");
    }
}

class Child extends Parent {

    private String name = "Aryan";

    @Override
    void print() {
        System.out.println(name);
    }
}
```

Now:

```java
new Child();
```

The `Parent` constructor calls `print()`, but runtime dispatch can invoke:

```java
Child.print()
```

At that point, the `Child` instance fields have not yet completed subclass initialization.

Therefore `name` may still have its default value:

```text
null
```

rather than `"Aryan"`.

---

### 155. Why is calling an overridable method from a constructor dangerous?

Because the subclass portion of the object **has not finished initialization**.

Example:

```java
class Parent {

    Parent() {
        process();
    }

    void process() {
    }
}

class Child extends Parent {

    private String value = "Hello";

    @Override
    void process() {
        System.out.println(value);
    }
}
```

During `Parent()`:

```text
Parent constructor
      ↓
Child.process()
      ↓
Child fields not initialized yet
```

So the overridden method can observe incomplete/default state.

It can also:

* Produce incorrect results.
* Throw exceptions.
* Call methods that depend on uninitialized subclass state.

**Interview rule:**

> Avoid calling overridable instance methods from constructors.

---

### 156. Can a constructor throw an exception?

**Yes.**

Constructors can declare and throw both checked and unchecked exceptions.

```java
class Employee {

    Employee(String name) throws Exception {
        if (name == null) {
            throw new Exception("Invalid name");
        }
    }
}
```

They can also throw unchecked exceptions without declaring them:

```java
Employee() {
    throw new IllegalArgumentException();
}
```

---

### 157. What happens to the object if its constructor throws an exception?

The object is **not successfully constructed** and the `new` expression fails.

```java
Employee e = new Employee();
```

If the constructor throws:

```text
new Employee()
      ↓
object allocation
      ↓
initialization
      ↓
constructor throws exception
      ↓
new expression fails
      ↓
no successfully constructed Employee returned
```

If no reachable reference to the partially initialized object remains, it can eventually become eligible for GC.

---

### 158. Can constructors be recursive?

**Indirectly, constructor chaining can create recursion**, but Java must not allow a constructor invocation cycle.

For example:

```java
class Test {

    Test() {
        this(10);
    }

    Test(int x) {
        this();
    }
}
```

This creates:

```text
Test()
 ↓
Test(int)
 ↓
Test()
 ↓
...
```

The compiler detects the constructor cycle and reports a **compile-time error**.

---

### 159. What happens if constructor chaining creates a cycle?

It results in a **compile-time error** because Java does not allow cyclic constructor invocation.

Example:

```java
class Test {

    Test() {
        this(10);
    }

    Test(int x) {
        this();
    }
}
```

The compiler detects:

```text
Test()
  ↓
Test(int)
  ↓
Test()
  ↓
cycle
```

and rejects the code.

### Key revision points

```text
Constructor
├── Initializes object
├── No return type
├── Can be overloaded
├── Not inherited
├── Not overridden
├── Can be private
├── this() → same-class constructor
└── super() → superclass constructor

Constructor chaining
├── this()/super() must be first statement
├── Cannot explicitly use both in one constructor
├── Missing super() → compiler inserts super()
└── Constructor cycles → compile-time error

Inheritance construction
Object → Parent → Child

Avoid:
Superclass constructor → overridable method
because subclass state may not be initialized yet.
```
