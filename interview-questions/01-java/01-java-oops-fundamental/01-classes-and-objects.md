# A. Classes & Objects — Answers

### 1. What is a class in Java?

A **class is a blueprint/template for creating objects**. It defines the state and behavior of objects through fields, methods, constructors, etc.

---

### 2. What is an object in Java?

An **object is an instance of a class**. It has its own state (instance fields) and can perform behavior defined by the class's methods.

---

### 3. What is the difference between a class and an object?

| Class                                    | Object                             |
| ---------------------------------------- | ---------------------------------- |
| Blueprint/template                       | Instance of a class                |
| Defines structure and behavior           | Has actual state                   |
| Does not represent a particular instance | Represents a particular entity     |
| One class can create many objects        | Each object is a separate instance |

---

### 4. How do you create an object in Java?

Typically using the `new` keyword:

```java
Employee e = new Employee();
```

The right side creates the object, and `e` stores a reference to it.

---

### 5. What happens internally when you use the `new` keyword?

Conceptually:

1. JVM allocates memory for the object.
2. Instance fields receive their default values.
3. The object's initialization occurs.
4. The constructor executes.
5. A reference to the newly created object is returned.

Example:

```java
Employee e = new Employee();
```

`new Employee()` creates and initializes the object; `e` receives its reference.

---

### 6. Where is an object stored when it is created?

Objects are generally allocated in the **heap**.

The exact memory management is JVM-dependent, but the Java specification conceptually treats objects as heap-allocated.

---

### 7. Where is the reference variable stored?

It depends on where it is declared:

* **Local variable** → typically associated with the current stack frame.
* **Instance field** → stored as part of the object on the heap.
* **Static field** → associated with the class and stored in JVM-managed class/static data structures.

Don't simply say **"references are always stored in the stack."**

---

### 8. What exactly does a reference variable contain?

A reference variable contains a **reference to an object**, not the object itself.

```java
Employee e = new Employee();
```

Conceptually:

```text
e ─────────► Employee object
```

The Java specification does not require the reference to literally be a memory address/pointer. The JVM decides how references are represented internally.

---

### 9. Can two reference variables point to the same object?

**Yes.**

```java
Employee e1 = new Employee();
Employee e2 = e1;
```

Both references point to the same object.

```text
e1 ──┐
     ├──► Employee object
e2 ──┘
```

---

### 10. Can an object exist without any reference variable pointing to it?

**Yes.**

An object can exist without a named reference pointing to it.

For example:

```java
new Employee();
```

The object is created, but there is no reference variable retaining it.

It can also temporarily become unreachable after the last reference is removed.

---

### 11. When does an object become eligible for garbage collection?

An object becomes **eligible for GC when it is no longer reachable through any live reference from GC roots**.

Example:

```java
Employee e = new Employee();
e = null;
```

If nothing else references that object, it becomes eligible for garbage collection.

**Eligible does not mean immediately collected.**

---

### 12. Can an object become eligible for GC even though a reference variable previously pointed to it?

**Yes.**

For example:

```java
Employee e = new Employee();

e = new Employee();
```

Initially:

```text
e ──► Object 1
```

After reassignment:

```text
e ──► Object 2

Object 1 ──► unreachable
```

If no other live reference points to Object 1, it becomes eligible for GC.

---

### 13. Can you create an object without using `new`?

**Yes.**

`new` is the most common mechanism, but objects can also be created through mechanisms such as:

```java
String s = "Hello";       // String literal
```

or:

```java
Employee e = Employee.class
    .getDeclaredConstructor()
    .newInstance();
```

Deserialization can also create objects without invoking the normal constructor in the usual way.

---

### 14. What are the different ways an object can be created in Java?

Common mechanisms include:

1. **Using `new`**

   ```java
   Employee e = new Employee();
   ```

2. **Using reflection**

   ```java
   Employee e = Employee.class
       .getDeclaredConstructor()
       .newInstance();
   ```

3. **Using `clone()`**

   ```java
   Employee e2 = e1.clone();
   ```

4. **Deserialization**

   ```java
   ObjectInputStream in = ...;
   Employee e = (Employee) in.readObject();
   ```

5. **String literals**

   ```java
   String s = "Hello";
   ```

6. **Factory methods**

   ```java
   Employee e = EmployeeFactory.create();
   ```

The factory method itself may internally use `new`, but from the caller's perspective object creation happens through the factory.

---

### 15. What is the difference between object identity and object state?

**Object identity** means **which particular object it is**.

**Object state** means the **current values of its fields**.

```java
Employee e1 = new Employee();
Employee e2 = new Employee();

e1.setName("Aryan");
e2.setName("Aryan");
```

Both objects may have the same state:

```text
e1 → name = "Aryan"
e2 → name = "Aryan"
```

But they have different identities:

```java
e1 == e2  // false
```

So:

> **Identity = which object**
> **State = what values the object currently has**

---

# Interviewer Follow-ups

### 16. If two variables point to the same object and you modify the object through one variable, what happens to the other?

Both references point to the **same object**, so the modification is visible through the other reference.

```java
Employee e1 = new Employee();
Employee e2 = e1;

e1.setName("Aryan");

System.out.println(e2.getName()); // Aryan
```

Because:

```text
e1 ──┐
     ├──► same object
e2 ──┘
```

---

### 17. What happens if you assign one object reference to another?

The **reference is copied**, not the object.

```java
Employee e1 = new Employee();
Employee e2 = e1;
```

Now both references point to the same object.

No new `Employee` object is created by `e2 = e1`.

---

### 18. If `Employee e1 = new Employee()` and `Employee e2 = e1`, how many objects exist?

**One object.**

```java
Employee e1 = new Employee(); // creates one object
Employee e2 = e1;             // creates no object
```

Both references point to the same object.

```text
e1 ──┐
     ├──► Employee Object #1
e2 ──┘
```

---

### 19. If `e1 = null`, what happens to the object?

The reference `e1` stops pointing to the object.

```java
Employee e1 = new Employee();

e1 = null;
```

If no other reachable reference points to that object, it becomes **eligible for GC**.

It is **not necessarily garbage-collected immediately**.

---

### 20. If a local reference goes out of scope, does the object immediately get garbage collected?

**No.**

Going out of scope only means that particular local reference is no longer accessible.

Example:

```java
void test() {
    Employee e = new Employee();
}
```

After `test()` returns, `e` is gone. If no other reachable reference points to the `Employee` object, the object becomes **eligible for GC**.

But GC:

* may happen later,
* may happen much later,
* or the JVM may terminate before collecting it.

So remember:

> **Out of scope ≠ immediately garbage collected.**
> **Unreachable ≠ immediately garbage collected.**
