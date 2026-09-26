# O. Nested, Inner & Anonymous Classes

### 219. 🟢 What is a nested class?

A **nested class** is a class declared inside another class or interface.

```java
class Outer {
    class Inner {
    }
}
```

There are four common forms:

1. Static nested class
2. Non-static inner class
3. Local class
4. Anonymous class

The main reason to use nested classes is to **logically group a helper class with the class that uses it** and control its visibility.

---

### 220. 🟢 What is a static nested class?

A **static nested class** is a nested class declared with `static`.

```java
class Outer {

    static class Nested {
        void display() {
            System.out.println("Hello");
        }
    }
}
```

It does **not require an instance of `Outer`**.

```java
Outer.Nested obj = new Outer.Nested();
```

A static nested class can directly access only the **static members** of the outer class.

```java
class Outer {
    static int x = 10;
    int y = 20;

    static class Nested {
        void test() {
            System.out.println(x); // OK
            // System.out.println(y); // Compile error
        }
    }
}
```

However, it can access an outer instance's non-static members if it has an explicit `Outer` reference:

```java
static class Nested {
    void test(Outer outer) {
        System.out.println(outer.y);
    }
}
```

---

### 221. 🟢 What is an inner class?

Strictly speaking, an **inner class** is a **non-static nested class**.

```java
class Outer {

    class Inner {
        void display() {
            System.out.println("Hello");
        }
    }
}
```

It is associated with an instance of the enclosing class.

```java
Outer outer = new Outer();
Outer.Inner inner = outer.new Inner();
```

Because it has an enclosing instance, it can directly access instance members of `Outer`.

```java
class Outer {
    private int value = 100;

    class Inner {
        void print() {
            System.out.println(value);
        }
    }
}
```

---

### 222. 🟡 Static nested class vs inner class?

| Feature                                | Static Nested Class  | Inner Class         |
| -------------------------------------- | -------------------- | ------------------- |
| `static`                               | Yes                  | No                  |
| Requires outer object?                 | No                   | Yes                 |
| Direct access to outer instance fields | No                   | Yes                 |
| Direct access to outer static fields   | Yes                  | Yes                 |
| Has enclosing instance relationship    | No                   | Yes                 |
| Creation                               | `new Outer.Nested()` | `outer.new Inner()` |

Example:

```java
class Outer {
    static int a = 10;
    int b = 20;

    static class Nested {
        void test() {
            System.out.println(a); // OK
            // System.out.println(b); // No
        }
    }

    class Inner {
        void test() {
            System.out.println(a); // OK
            System.out.println(b); // OK
        }
    }
}
```

**Interview shortcut:**

> Static nested class is independent of an outer instance; an inner class carries an enclosing-instance relationship.

---

### 223. 🟡 Can an inner class access private members of its outer class?

**Yes.**

```java
class Outer {
    private int value = 100;

    class Inner {
        void print() {
            System.out.println(value);
        }
    }
}
```

The inner class has access to the private members of its enclosing class.

The reverse is also possible: the outer class can access private members of its nested/inner class.

```java
class Outer {
    class Inner {
        private int x = 10;
    }

    void test() {
        Inner i = new Inner();
        System.out.println(i.x); // OK
    }
}
```

This is allowed because nested classes are considered part of the same top-level class's implementation/access context.

---

### 224. 🔴 How does an inner class maintain a relationship with its enclosing object?

A non-static inner-class object has an **associated enclosing instance** of the outer class.

Conceptually:

```java
Outer outer = new Outer();
Outer.Inner inner = outer.new Inner();
```

The `inner` object remembers the particular `outer` instance with which it was created.

Therefore:

```java
class Outer {
    int value;

    Outer(int value) {
        this.value = value;
    }

    class Inner {
        void print() {
            System.out.println(value);
        }
    }
}
```

```java
Outer o1 = new Outer(10);
Outer o2 = new Outer(20);

Outer.Inner i1 = o1.new Inner();
Outer.Inner i2 = o2.new Inner();

i1.print(); // 10
i2.print(); // 20
```

### Important JVM-level point

Conceptually, the inner object carries a reference to its enclosing `Outer` instance.

The Java compiler typically represents this using a **synthetic field/reference** in the generated class file.

So you can think of it approximately as:

```text
Inner object
   |
   +---- reference ----> Outer object
```

This is why an inner class can access:

```java
Outer.this.value
```

and explicitly refer to its enclosing object:

```java
Outer.this
```

---

### 225. 🔴 Can a static nested class access non-static members of the outer class directly?

**No.**

A static nested class does not have an implicit enclosing instance.

```java
class Outer {
    int value = 100;

    static class Nested {
        void test() {
            // System.out.println(value); // Compile error
        }
    }
}
```

But it can access the member through an explicitly supplied outer object:

```java
static class Nested {
    void test(Outer outer) {
        System.out.println(outer.value);
    }
}
```

So:

```text
Static nested class
        |
        X  no implicit Outer instance
        |
        +---- cannot directly access instance members

Inner class
        |
        +---- implicit Outer instance
        |
        +---- can directly access instance members
```

---

### 226. 🟡 What is a local class?

A **local class** is a class declared inside a method, constructor, or initializer block.

```java
class Outer {

    void process() {

        class Helper {
            void calculate() {
                System.out.println("Processing");
            }
        }

        Helper h = new Helper();
        h.calculate();
    }
}
```

Its scope is limited to the block where it is declared.

A local class can access:

* Members of the enclosing class
* Local variables that are **final or effectively final**
* Parameters that are final/effectively final

Example:

```java
void process() {
    int x = 10; // effectively final

    class Helper {
        void print() {
            System.out.println(x);
        }
    }
}
```

This works.

But:

```java
void process() {
    int x = 10;

    x = 20;

    class Helper {
        void print() {
            System.out.println(x); // Compile error
        }
    }
}
```

because `x` is no longer effectively final.

---

### 227. 🟡 What is an anonymous class?

An **anonymous class** is a class without an explicit class name that is declared and instantiated at the same time.

```java
Runnable r = new Runnable() {
    @Override
    public void run() {
        System.out.println("Running");
    }
};
```

There is no class name such as:

```java
class MyRunnable implements Runnable
```

Instead, the implementation is created inline.

It can also extend a class:

```java
Animal a = new Animal() {
    @Override
    void sound() {
        System.out.println("Bark");
    }
};
```

An anonymous class:

* Has no explicit class name in source code
* Is instantiated where it is declared
* Can extend one class **or** implement one interface
* Can have fields and methods
* Can have an instance initializer
* Cannot have an explicitly declared constructor

---

### 228. 🟡 When would you use an anonymous class?

Use an anonymous class when you need a **small, one-off implementation** and giving that implementation a separate named class would add unnecessary structure.

Common historical examples:

```java
button.addActionListener(new ActionListener() {
    @Override
    public void actionPerformed(ActionEvent e) {
        System.out.println("Clicked");
    }
});
```

Another example:

```java
Thread t = new Thread(new Runnable() {
    @Override
    public void run() {
        System.out.println("Task");
    }
});
```

However, for a **functional interface**, modern Java usually prefers a lambda:

```java
Thread t = new Thread(() -> {
    System.out.println("Task");
});
```

Use a named class when the implementation is substantial, reusable, or deserves its own identity.

---

### 229. 🔴 Anonymous class vs lambda?

The biggest distinction is:

> A lambda represents a function/behavior for a **functional interface**; an anonymous class creates an actual class instance with its own class body.

Example lambda:

```java
Runnable r = () -> System.out.println("Hello");
```

Anonymous class:

```java
Runnable r = new Runnable() {
    @Override
    public void run() {
        System.out.println("Hello");
    }
};
```

Important differences:

| Feature                        | Anonymous Class                  | Lambda                               |
| ------------------------------ | -------------------------------- | ------------------------------------ |
| Functional interface required? | No                               | Yes                                  |
| Can extend a class?            | Yes                              | No                                   |
| Can implement interface?       | Yes                              | Functional interface only            |
| Can declare multiple methods?  | Yes                              | No                                   |
| Own fields?                    | Yes                              | Cannot declare fields in lambda body |
| `this`                         | Refers to anonymous-class object | Refers to enclosing object           |
| Syntax                         | Verbose                          | Concise                              |

### Very important `this` difference

Anonymous class:

```java
class Example {
    void test() {
        Runnable r = new Runnable() {
            @Override
            public void run() {
                System.out.println(this);
            }
        };
    }
}
```

Here `this` refers to the **anonymous class instance**.

Lambda:

```java
class Example {
    void test() {
        Runnable r = () -> {
            System.out.println(this);
        };
    }
}
```

Here `this` refers to the **enclosing `Example` object**.

### Interview trap

A lambda is **not simply syntactic sugar for an anonymous class**. The JVM/runtime representation and semantics differ; Java specifies lambda behavior as functional-interface instances rather than requiring anonymous-class semantics.

---

### 230. 🔴 Can an anonymous class extend a class and implement an interface?

**No, not directly in a single anonymous-class declaration.**

An anonymous class can:

```java
new SomeClass() {
    // extends SomeClass
};
```

**or**

```java
new SomeInterface() {
    // implements SomeInterface
};
```

But you cannot write something equivalent to:

```java
new SomeClass() implements SomeInterface { } // invalid
```

However, you can define a **named class** that extends a class and implements interfaces:

```java
class MyClass extends SomeClass implements SomeInterface {
}
```

and then instantiate it anonymously if appropriate through the superclass/interface type.

---

### 231. 🔴 What happens if an inner class escapes the lifetime of its outer object?

The outer object **does not become eligible for GC** as long as the inner object is reachable and the inner object holds its enclosing-instance reference.

Example:

```java
class Outer {
    private int value = 100;

    class Inner {
        void print() {
            System.out.println(value);
        }
    }

    Inner createInner() {
        return new Inner();
    }
}
```

```java
Outer outer = new Outer();

Outer.Inner inner = outer.createInner();

outer = null;
```

You might think `Outer` can now be garbage collected.

But conceptually:

```text
inner
  |
  +----> Outer
```

Therefore the `Outer` object is still reachable through `inner`.

So:

```text
outer variable = null
        ↓
Outer object is NOT necessarily GC eligible
        ↑
        |
Inner object
```

The outer object becomes eligible only when there is no reachable path to it.

---

### 232. 🔴 How can inner classes contribute to memory retention?

Because a non-static inner class can hold a reference to its enclosing object.

Consider:

```java
class LargeObject {

    byte[] hugeData = new byte[100_000_000];

    class Inner {
    }
}
```

If an `Inner` instance remains reachable:

```java
LargeObject.Inner inner = largeObject.new Inner();

largeObject = null;
```

the `LargeObject` may still be retained because:

```text
reachable Inner
      ↓
enclosing LargeObject
      ↓
hugeData
```

Therefore a relatively small inner object can indirectly keep a **large object graph** alive.

### Common real-world scenario

This can matter with:

* Long-lived listeners
* Callbacks
* Executors
* Threads
* Caches
* Android components
* Application-level registries
* Asynchronous tasks

For example:

```java
class Service {
    class Callback {
        void execute() {
            // uses Service state
        }
    }
}
```

If a long-lived component stores `Callback`, it may unintentionally retain the `Service`.

### Solution

If the inner class does **not need access to the outer instance**, make it `static`:

```java
class Service {

    static class Callback {
    }
}
```

This removes the implicit enclosing-instance reference.

**Interview takeaway:**

> Non-static inner classes can retain their outer objects. Static nested classes avoid this implicit reference and are preferable when no outer-instance state is required.

---

# P. Enums

### 233. 🟢 What is an enum?

An `enum` is a special Java type used to represent a **fixed set of named constants**.

```java
enum Status {
    PENDING,
    PROCESSING,
    COMPLETED,
    FAILED
}
```

Usage:

```java
Status status = Status.PENDING;
```

Enums provide:

* Type safety
* Fixed set of values
* Useful methods such as `values()` and `valueOf()`
* Ability to contain fields, methods, and constructors
* Ability to implement interfaces

---

### 234. 🟢 Why should enums be preferred over integer constants in many cases?

Instead of:

```java
public static final int PENDING = 1;
public static final int COMPLETED = 2;
public static final int FAILED = 3;
```

use:

```java
enum Status {
    PENDING,
    COMPLETED,
    FAILED
}
```

### Advantages

**1. Type safety**

```java
Status status = Status.PENDING;
```

You cannot accidentally assign an arbitrary integer.

With integers:

```java
int status = 999; // syntactically valid
```

even though `999` may not represent a valid status.

**2. Readability**

```java
if (status == Status.COMPLETED)
```

is much clearer than:

```java
if (status == 2)
```

**3. Behavior can be attached to constants**

```java
enum Status {
    PENDING,
    COMPLETED;

    boolean isTerminal() {
        return this == COMPLETED;
    }
}
```

**4. Works well with `switch`**

```java
switch (status) {
    case PENDING -> ...
    case COMPLETED -> ...
}
```

**5. Specialized collections**

```java
EnumSet<Status>
EnumMap<Status, String>
```

can be highly efficient because enum values come from a fixed set.

---

### 235. 🟡 Can an enum contain fields?

**Yes.**

```java
enum Status {
    PENDING("P"),
    COMPLETED("C");

    private final String code;

    Status(String code) {
        this.code = code;
    }

    public String getCode() {
        return code;
    }
}
```

Usage:

```java
System.out.println(Status.PENDING.getCode()); // P
```

The fields can be:

* `private`
* `final`
* `static` where allowed by enum initialization rules
* Other normal field types

A common pattern is to associate a database/API code with each enum constant.

---

### 236. 🟡 Can an enum contain methods?

**Yes.**

```java
enum Status {
    PENDING,
    COMPLETED;

    public boolean isTerminal() {
        return this == COMPLETED;
    }
}
```

```java
Status.COMPLETED.isTerminal(); // true
```

Enums can contain:

* Instance methods
* Static methods
* Private methods
* Abstract methods
* Overridden methods

They are much more powerful than simply being a list of constants.

---

### 237. 🟡 Can an enum have a constructor?

**Yes.**

```java
enum Status {
    PENDING("P"),
    COMPLETED("C");

    private final String code;

    Status(String code) {
        this.code = code;
    }
}
```

The constructor is called when the enum constants are initialized.

You cannot normally create enum instances yourself:

```java
new Status("X"); // compile error
```

The enum constants are the instances created by the JVM as part of enum initialization.

---

### 238. 🔴 Why are enum constructors effectively private?

Because enum instances are meant to come from the **fixed set of declared constants**.

```java
enum Status {
    PENDING,
    COMPLETED
}
```

Java prevents arbitrary external creation such as:

```java
new Status(); // impossible
```

An enum constructor cannot be `public` or `protected`; it is implicitly private.

Conceptually:

```java
enum Status {
    PENDING,
    COMPLETED;

    private Status() {
    }
}
```

The restriction ensures that code cannot create additional enum constants beyond those declared by the enum.

---

### 239. 🟡 Can an enum implement an interface?

**Yes.**

```java
interface Printable {
    void print();
}

enum Status implements Printable {
    PENDING,
    COMPLETED;

    @Override
    public void print() {
        System.out.println(this);
    }
}
```

Enums can implement **one or multiple interfaces**.

```java
enum Status implements A, B, C {
}
```

This is useful when enum constants need to participate in a common abstraction.

---

### 240. 🟡 Can an enum extend another class?

**No, not an arbitrary class.**

An enum implicitly extends `java.lang.Enum`.

Conceptually:

```java
enum Status extends Enum<Status> {
}
```

You cannot write:

```java
enum Status extends SomeClass { } // invalid
```

---

### 241. 🔴 Why can't an enum extend an arbitrary class?

Because Java's enum type already has a superclass:

```java
java.lang.Enum
```

Java classes can have only **one superclass**.

So an enum cannot simultaneously extend:

```text
Enum
  +
SomeOtherClass
```

Java instead allows enums to **implement interfaces**:

```java
enum Status implements SomeInterface {
}
```

This gives enums additional behavior without requiring multiple class inheritance.

---

### 242. 🔴 How are enum constants represented internally?

Each enum constant is essentially a **singleton instance of the enum type** created during enum class initialization.

For:

```java
enum Status {
    PENDING,
    COMPLETED
}
```

conceptually, the compiler generates something resembling:

```java
public final class Status extends Enum<Status> {

    public static final Status PENDING =
        new Status("PENDING", 0);

    public static final Status COMPLETED =
        new Status("COMPLETED", 1);

    private Status(String name, int ordinal) {
        super(name, ordinal);
    }
}
```

This is **conceptual**, not the exact generated bytecode.

Important properties:

### `name()`

```java
Status.PENDING.name()
```

returns:

```text
"PENDING"
```

### `ordinal()`

```java
Status.PENDING.ordinal()
```

returns:

```text
0
```

`ordinal()` represents declaration order.

### Important interview warning

Do **not** use `ordinal()` as a persistent database value.

Changing:

```java
PENDING,
COMPLETED,
FAILED
```

to:

```java
NEW,
PENDING,
COMPLETED,
FAILED
```

changes the ordinals.

Use an explicit field instead:

```java
enum Status {
    PENDING("P"),
    COMPLETED("C");

    private final String code;
}
```

---

### 243. 🔴 Can enum constants have different implementations of the same method?

**Yes.**

This is a powerful enum feature called **constant-specific class bodies**.

```java
enum Operation {

    ADD {
        @Override
        int apply(int a, int b) {
            return a + b;
        }
    },

    MULTIPLY {
        @Override
        int apply(int a, int b) {
            return a * b;
        }
    };

    abstract int apply(int a, int b);
}
```

Usage:

```java
Operation.ADD.apply(2, 3);      // 5
Operation.MULTIPLY.apply(2, 3); // 6
```

Each constant can provide its own implementation.

Conceptually, the constants behave as specialized implementations associated with the enum type.

This is useful for implementing **constant-specific behavior** without a large `switch`.

---

### 244. 🟡 When would you use `EnumMap` or `EnumSet`?

Use them when the elements/keys are enum values.

## `EnumSet`

A specialized `Set` implementation for enum types.

```java
enum Permission {
    READ,
    WRITE,
    DELETE
}

EnumSet<Permission> permissions =
        EnumSet.of(Permission.READ, Permission.WRITE);
```

Useful when you need a **set of enum values**.

Example:

```java
EnumSet<Permission> permissions =
        EnumSet.allOf(Permission.class);
```

It is typically implemented very efficiently using bit-oriented representations because the set of possible enum values is fixed.

---

## `EnumMap`

A specialized `Map` whose keys are enum values.

```java
EnumMap<Permission, String> descriptions =
        new EnumMap<>(Permission.class);

descriptions.put(Permission.READ, "Read access");
descriptions.put(Permission.WRITE, "Write access");
```

Use it when:

```text
Map<Enum, Value>
```

is the required data structure.

It can be more efficient and compact than a general-purpose `HashMap` for enum keys.

### Interview shortcut

```text
EnumSet → Set of enum constants

EnumMap → Map where keys are enum constants
```

Example:

```java
EnumSet<Status> statuses;
EnumMap<Status, String> messages;
```

Both are specialized specifically for enum types.
