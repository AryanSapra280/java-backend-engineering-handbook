# Java Interview Notes

## 1. Why is Java not 100% Object-Oriented?

Java is not considered a 100% object-oriented language because it supports **8 primitive data types**:

- `byte`
- `short`
- `int`
- `long`
- `float`
- `double`
- `char`
- `boolean`

Primitive types are not objects.

However, Java provides **wrapper classes** for these primitive types when we need to work with them as objects:

| Primitive | Wrapper |
|---|---|
| `byte` | `Byte` |
| `short` | `Short` |
| `int` | `Integer` |
| `long` | `Long` |
| `float` | `Float` |
| `double` | `Double` |
| `char` | `Character` |
| `boolean` | `Boolean` |

---

## 2. Why doesn't Java support pointers?

Java does not expose pointers directly to programmers mainly because:

1. **Pointers can be unsafe** and can provide direct access to memory.
2. Java's **JVM manages memory allocation and garbage collection**, so programmers do not need direct control over memory addresses or manual memory deallocation.

This makes Java safer and simpler than languages that expose raw pointers.

---

## 3. What is the JIT Compiler in Java?

**JIT (Just-In-Time) Compiler** is a part of the JVM that improves Java application performance by compiling frequently executed Java bytecode into **native machine code at runtime**.

Instead of repeatedly interpreting the same bytecode, the JVM can identify frequently executed code and compile it into optimized native code.

---

## 4. Why are Strings immutable in Java?

`String` objects are immutable, meaning their value cannot be changed after creation.

### Reason 1: String Pool

Java maintains a **String Pool** where String literals can be shared.

For example:

```java
String a = "Hello";
String b = "Hello";
```

Both can refer to the same pooled String object.

If Strings were mutable, changing the value through one reference could unexpectedly affect other references.

Therefore, immutability makes String pooling safe.

### Reason 2: Security

Strings are commonly used for sensitive information such as:

- File paths
- Network connections
- Database connection information
- Class names
- Configuration values

If a String could be modified after it was created, its value could change unexpectedly after validation or authorization.

Immutability prevents this.

---

## 5. What is a Marker Interface?

A **marker interface** is an interface that does not declare methods or fields. It is used to indicate that a class has a particular capability or satisfies a particular contract.

### Examples

```java
Serializable
Cloneable
```

### Why are marker interfaces useful?

Even though they do not contain methods, they provide **type information** to Java and its runtime mechanisms.

For example:

- `Serializable` indicates that objects of the class can participate in Java serialization.
- `Cloneable` indicates that the class permits cloning through `Object.clone()`.

The marker interface itself does not implement the behavior; it **marks the class as having a particular capability**.

---

## 6. Can we override a private or static method?

### Private method

**No, a private method cannot be overridden.**

A private method is accessible only within the class in which it is declared. Therefore, the child class cannot override it.

```java
class Parent {
    private void show() {
        System.out.println("Parent");
    }
}
```

A method with the same name in the child class would be a separate method, not an overridden method.

### Static method

**No, static methods are not overridden.**

If a child class declares a static method with the same signature, it is called **method hiding**.

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

Here, `Child.show()` hides `Parent.show()` rather than overriding it.

---

## 7. Does `finally` always execute in Java?

**No, `finally` does not execute in every possible situation.**

Normally, `finally` executes whether an exception occurs or not.

However, there are situations where it may not execute, for example:

### `System.exit()`

```java
try {
    System.exit(0);
} finally {
    System.out.println("Finally");
}
```

The JVM terminates, so the `finally` block does not get a chance to execute.

### JVM/process termination

If the JVM terminates unexpectedly, for example because of a severe system failure, `finally` may not execute.

---

# 8. What methods are contained in the `Object` class?

`java.lang.Object` is the **root class of the Java class hierarchy**. Every Java class directly or indirectly extends `Object`.

Important methods include:

### 1. `clone()`

```java
protected Object clone() throws CloneNotSupportedException
```

Creates and returns a copy of the object.

The default implementation performs a **shallow copy**.

A shallow copy creates a new object, but references held by the original object are copied as references.

```text
Original Object
      |
      +-----> Address A

Cloned Object
      |
      +-----> Address A
```

So both objects can refer to the same mutable nested object.

> Note: `Object.clone()` creates a **new object**. It does not simply return the same reference.

---

### 2. `equals(Object obj)`

```java
public boolean equals(Object obj)
```

Compares an object with another object for equality.

The default implementation in `Object` behaves like reference equality unless the class overrides it.

---

### 3. `finalize()`

Historically:

```java
protected void finalize() throws Throwable
```

It was intended to be called before an object was reclaimed by the garbage collector.

**Important modern Java note:** finalization has been deprecated and should not be relied upon for resource management.

---

### 4. `getClass()`

```java
public final Class<?> getClass()
```

Returns the runtime class of the object.

Example:

```java
Employee employee = new Employee();

System.out.println(employee.getClass());
```

---

### 5. `hashCode()`

```java
public int hashCode()
```

Returns an integer hash code for the object.

It is commonly used by hash-based collections such as:

```java
HashMap
HashSet
```

It is related to `equals()`: if two objects are equal according to `equals()`, they must return the same `hashCode()`.

---

### 6. `wait()`, `notify()`, and `notifyAll()`

These methods are used for **thread coordination** and are defined in `Object`.

- `wait()` → causes the current thread to wait.
- `notify()` → wakes one waiting thread.
- `notifyAll()` → wakes all waiting threads.

They are generally used together with synchronization/monitor ownership.

---

# 9. How can you make a class immutable?

An **immutable class** is a class whose object's state **cannot be changed after the object has been created**.

## General Rules

1. Declare the class as `final` so it cannot be extended.
2. Make all fields `private`.
3. Do not provide setter methods.
4. Make fields `final` so they cannot be reassigned after construction.
5. For mutable fields, make a defensive/deep copy when storing them in the constructor.
6. For mutable fields, return a defensive/deep copy from getter methods instead of returning the actual internal reference.

---

## 9.1 Why should the class be `final`?

```java
public final class Employee {
    // ...
}
```

Making the class `final` prevents inheritance.

This is important because a subclass could otherwise override methods and introduce behavior that breaks the immutability contract.

```text
Employee
   |
   | final
   ↓
Cannot be extended
   ↓
Methods cannot be overridden
   ↓
Immutability cannot be bypassed through inheritance
```

### Important interview point

`final` does **not** make the object's fields immutable.

It protects the class from being extended in a way that could violate its intended behavior.

---

## 9.2 Why make a defensive/deep copy in the constructor?

This is necessary when the class contains a **mutable object**.

Consider:

```java
public final class Employee {

    private final Address address;

    public Employee(Address address) {
        this.address = address;
    }
}
```

Now:

```java
Address address = new Address("Delhi");

Employee employee = new Employee(address);
```

Both references point to the same `Address` object:

```text
address ──────────────┐
                      ↓
                  Address A
                      ↑
                      |
employee.address ─────┘
```

The caller can now do:

```java
address.setCity("Mumbai");
```

This also changes the Address stored inside `employee`.

Therefore, `Employee` is not truly immutable.

### Solution: defensive copy

Instead of:

```java
this.address = address;
```

create a new object:

```java
this.address = new Address(address);
```

Now:

```text
Original:

address ─────────► Address A


Employee:

employee.address ─► Address B
                      ↑
                   copy of A
```

Changing the original:

```java
address.setCity("Mumbai");
```

does not change `employee.address`.

### Key idea

> **Constructor defensive copying prevents the caller from modifying the immutable object's internal state through the original reference that was passed into the constructor.**

---

## 9.3 Why return a defensive/deep copy from a getter?

Even if the constructor is protected, the getter can accidentally expose the internal mutable object.

Suppose:

```java
public Address getAddress() {
    return address;
}
```

Then:

```java
Employee employee = new Employee(address);

Address a = employee.getAddress();

a.setCity("Mumbai");
```

The getter returned the **actual internal reference**.

So:

```text
employee
   |
   +-----> Address A
               ↑
               |
               a
```

The caller can modify `Address A`, which means the internal state of `Employee` can be changed.

### Solution: return a copy

```java
public Address getAddress() {
    return new Address(address);
}
```

Now:

```text
employee
   |
   +-----> Address A


a ─────────► Address B
               ↑
            copy of A
```

When the caller executes:

```java
a.setCity("Mumbai");
```

only `Address B` changes.

The internal `Address A` inside `Employee` remains unchanged.

### Key idea

> **Getter defensive copying prevents the caller from modifying the immutable object's internal state through a reference returned by the getter.**

---

# Constructor Copy vs Getter Copy

This is the easiest way to remember the difference:

| Protection | What does it protect against? |
|---|---|
| Defensive copy in constructor | Caller modifying the object through the **original reference passed in** |
| Defensive copy in getter | Caller modifying the object through the **reference returned by the getter** |
| `final class` | Subclassing and inheritance-based violations of immutability |
| `private` fields | Direct access to internal state |
| No setters | Modification through setter methods |
| `final` fields | Reassignment of fields after construction |

---

## Complete Example

```java
public final class Employee {

    private final String name;
    private final Address address;

    public Employee(String name, Address address) {
        this.name = name;
        this.address = new Address(address); // defensive copy
    }

    public String getName() {
        return name;
    }

    public Address getAddress() {
        return new Address(address); // defensive copy
    }
}
```

### Remember

```text
final class
    ↓
prevents inheritance

private fields
    ↓
prevents direct access

no setters
    ↓
prevents modification through setters

final fields
    ↓
prevents reassignment

constructor defensive copy
    ↓
protects from the original reference

getter defensive copy
    ↓
protects from the returned reference
```

### Important correction

Do not think:

> "Make mutable fields final, therefore they become immutable."

Instead remember:

> **`final` prevents the field reference from being reassigned. It does not make the referenced object immutable.**

For example:

```java
private final Address address;
```

The reference cannot be changed to another `Address`, but the `Address` object itself may still be mutable.

That is why mutable fields require **defensive copying**.

---

# 10. What is a Singleton Class in Java?

A **Singleton** is a design pattern that ensures that only **one instance of a class is created within a given JVM/application context**, and provides a way to access that instance.

Example concept:

```text
Application
     |
     ↓
 Singleton Class
     |
     ↓
 Only one instance
```

> For a complete implementation and different Singleton approaches, refer to the Singleton Design Pattern notes.
