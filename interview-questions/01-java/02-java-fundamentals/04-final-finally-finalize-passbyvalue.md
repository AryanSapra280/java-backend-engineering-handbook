# I. `final`, `finally` & `finalize`

### 137. What is the difference between `final`, `finally` and `finalize()`?

| `final`                                      | `finally`                                         | `finalize()`                         |
| -------------------------------------------- | ------------------------------------------------- | ------------------------------------ |
| Keyword                                      | Block                                             | Method                               |
| Used for variables, methods, classes         | Exception handling                                | Historically associated with GC      |
| Prevents reassignment/overriding/inheritance | Usually executes cleanup code after `try`/`catch` | Was intended for pre-GC cleanup      |
| Compile-time concept                         | Runtime control flow                              | Deprecated and no longer recommended |

Example:

```java
final int x = 10;

try {
    // code
} finally {
    // cleanup
}
```

`finalize()` was deprecated in Java 9 and is deprecated for removal in modern Java.

---

### 138. What does `final` mean for a variable?

A `final` variable can be **assigned only once**.

```java
final int x = 10;

x = 20; // Compile-time error
```

For a reference:

```java
final Employee e = new Employee();

e = new Employee(); // Error
```

The reference cannot be reassigned.

---

### 139. What does `final` mean for a method?

A `final` method **cannot be overridden by a subclass**.

```java
class Parent {

    final void display() {
        System.out.println("Parent");
    }
}

class Child extends Parent {

    // void display() {} // Compile-time error
}
```

The method can still be inherited and called by the subclass.

---

### 140. What does `final` mean for a class?

A `final` class **cannot be extended**.

```java
final class Employee {
}
```

This is invalid:

```java
class Manager extends Employee {
}
```

A classic example is:

```java
public final class String
```

---

### 141. Can a final variable be initialized in a constructor?

**Yes.**

This is commonly used for instance fields that must be assigned exactly once per object.

```java
class Employee {

    private final int id;

    Employee(int id) {
        this.id = id;
    }
}
```

Each constructor execution initializes `id` once.

After initialization:

```java
this.id = 20; // Error
```

---

### 142. What is a blank final variable?

A **blank final variable** is a `final` variable declared without an initial value and assigned later exactly once, typically in a constructor.

```java
class Employee {

    private final int id;

    Employee(int id) {
        this.id = id;
    }
}
```

Here:

```java
private final int id;
```

is a blank final field.

The compiler ensures it is definitely assigned before construction completes.

---

### 143. Is a final reference immutable?

**No.**

`final` prevents the **reference from being reassigned**. It does not make the referenced object immutable.

```java
final List<String> list = new ArrayList<>();

list.add("Java"); // Valid
list.add("Spring"); // Valid

// list = new ArrayList<>(); // Invalid
```

The reference stays the same, but the object can change.

---

### 144. Can the state of an object referenced by a final variable change?

**Yes.**

```java
final Employee employee = new Employee();

employee.setName("Aryan");
```

This is valid.

`final` applies to the reference:

```text
final employee
      │
      └──────► Employee object
                    │
                    └── state can change
```

It prevents:

```java
employee = anotherEmployee;
```

---

### 145. What is the difference between final reference and immutable object?

### Final reference

The reference cannot point to another object:

```java
final Employee e = new Employee();

e.setName("Aryan"); // allowed
```

### Immutable object

The object's state cannot change after construction.

For example:

```java
String s = "Java";
```

`String` itself is immutable.

You can combine both:

```java
final String s = "Java";
```

Now:

* Reference cannot be reassigned.
* String object cannot be mutated.

But these are **two separate properties**.

> `final` controls the variable/reference.
> Immutability controls the object's state.

---

### 146. What is `finally`?

`finally` is a block used with exception handling to execute cleanup code after a `try`/`catch` sequence.

```java
try {
    // operation
} catch (Exception e) {
    // handling
} finally {
    // cleanup
}
```

Typical use:

```java
finally {
    connection.close();
}
```

Although modern Java usually prefers **try-with-resources** for `AutoCloseable` resources.

---

### 147. When does `finally` execute?

Normally, `finally` executes when control leaves the associated `try`/`catch` structure, regardless of whether:

* The `try` completes normally.
* An exception occurs and is caught.
* An exception occurs and propagates.
* A `return` is executed in `try`.
* A `return` is executed in `catch`.

Example:

```java
try {
    System.out.println("try");
} finally {
    System.out.println("finally");
}
```

Output:

```text
try
finally
```

---

### 148. Are there situations where `finally` does not execute?

**Yes.**

For example:

### `System.exit()`

```java
try {
    System.out.println("try");
    System.exit(0);
} finally {
    System.out.println("finally");
}
```

`finally` is not executed because the JVM is terminated.

Other abnormal situations such as a JVM crash, forced process termination, or catastrophic system failure can also prevent it from running.

So don't say:

> "`finally` always executes."

Say:

> "`finally` normally executes when control leaves the try/catch construct, except in situations where execution of the JVM/process is terminated or otherwise cannot continue."

---

### 149. What happens when `return` exists inside `try` and `finally`?

The `finally` block executes **before the method actually returns**.

```java
public static int test() {
    try {
        return 10;
    } finally {
        System.out.println("finally");
    }
}
```

Output:

```text
finally
```

Return value:

```text
10
```

Conceptually:

```text
try
 ↓
prepare return value = 10
 ↓
finally executes
 ↓
method returns 10
```

---

### 150. What happens when `finally` itself contains `return`?

A `return` in `finally` **overrides/suppresses a pending return or exception** from `try`/`catch`.

Example:

```java
static int test() {
    try {
        return 10;
    } finally {
        return 20;
    }
}
```

Result:

```text
20
```

The `return 10` is effectively discarded.

This is why **returning from `finally` is strongly discouraged**.

It can also suppress exceptions:

```java
static void test() {
    try {
        throw new RuntimeException("Original");
    } finally {
        return;
    }
}
```

The exception is suppressed because the `finally` return completes the method normally.

---

### 151. What happens when `System.exit()` is called?

`System.exit()` requests termination of the JVM.

For example:

```java
try {
    System.exit(0);
} finally {
    System.out.println("finally");
}
```

The JVM begins shutdown, so the `finally` block is **not guaranteed to execute**.

Therefore, don't rely on `finally` for cleanup that must survive JVM termination.

---

### 152. What was `finalize()` intended for?

`finalize()` was a method inherited from `Object` that was intended to allow an object to perform cleanup before the JVM reclaimed it.

Historically:

```java
@Override
protected void finalize() throws Throwable {
    // cleanup
}
```

The JVM/GC could potentially invoke it before reclaiming an object.

However, this mechanism was unreliable and has been deprecated.

---

### 153. Why was finalization deprecated/removed from modern Java practices?

Finalization has serious problems:

* Execution time is unpredictable.
* It is not guaranteed to run promptly.
* It can introduce significant GC overhead.
* It can delay reclamation of objects.
* It can cause resurrection-related issues.
* It makes resource management nondeterministic.
* Exceptions from finalizers don't provide a reliable cleanup mechanism.
* It complicates JVM/runtime behavior.

Therefore, modern Java strongly discourages finalization.

---

### 154. Why is relying on finalization dangerous for resource management?

Suppose you open a database connection:

```java
Connection connection = ...;
```

You cannot safely say:

> "I'll close it in `finalize()`."

The GC may not run for a long time, so the database connection could remain open.

This can cause:

* Connection pool exhaustion.
* File descriptor exhaustion.
* Memory/resource pressure.
* Unpredictable application behavior.

Resource cleanup should happen **deterministically**, not when GC happens to run.

---

### 155. What should be used instead of finalization for managing resources?

Use **try-with-resources** for resources implementing `AutoCloseable`/`Closeable`.

```java
try (Connection connection = dataSource.getConnection()) {
    // use connection
}
```

When control leaves the block, Java automatically calls `close()`.

You can also implement explicit lifecycle methods:

```java
resource.close();
```

For memory management, rely on **garbage collection**, not explicit destruction.

---

# J. Pass-by-Value & Reference Semantics

### 156. Is Java pass-by-value or pass-by-reference?

**Java is strictly pass-by-value.**

This applies to both:

* Primitive arguments.
* Object references.

For objects, Java passes a **copy of the reference value**.

This distinction is extremely important.

---

### 157. Explain Java's pass-by-value behavior with primitives.

```java
static void change(int x) {
    x = 100;
}

int a = 10;

change(a);

System.out.println(a);
```

Output:

```text
10
```

Why?

```text
Caller:
a = 10

       copy value
           ↓
Method:
x = 10
```

Changing `x` doesn't change `a`.

---

### 158. Explain Java's pass-by-value behavior with object references.

Consider:

```java
class Employee {
    String name;
}
```

```java
static void change(Employee e) {
    e.name = "Aryan";
}

Employee emp = new Employee();
emp.name = "John";

change(emp);

System.out.println(emp.name);
```

Output:

```text
Aryan
```

Why?

Java copies the **reference value**:

```text
Caller                     Method

emp ─────┐
         │
         ▼
     Employee object
         ▲
         │
e ───────┘
```

Both `emp` and `e` contain references to the same object.

So modifying the object's state is visible through `emp`.

But the references themselves are separate variables.

---

### 159. Why do people sometimes incorrectly say Java is pass-by-reference?

Because when an object is passed:

```java
change(emp);
```

the method can modify the object's state, and the caller sees the modification.

That can look like pass-by-reference.

But Java actually passes:

> **A copy of the reference value.**

Evidence:

```java
static void change(Employee e) {
    e = new Employee();
}
```

Reassigning `e` doesn't change the caller's `emp`.

Therefore Java cannot be true pass-by-reference.

---

### 160. What exactly is passed when an object is supplied as a method argument?

The **value of the reference is copied into the parameter**.

```java
Employee emp = new Employee();

change(emp);
```

Conceptually:

```text
Before method:

emp ─────► Object


During method:

emp ─────► Object
e   ─────► Object
```

`emp` and `e` are separate reference variables containing the same reference value.

The object itself is **not copied**.

---

### 161. What happens if a method changes a field of an object passed as an argument?

The caller can observe the change because both references point to the same object.

```java
static void update(Employee e) {
    e.name = "Aryan";
}

Employee emp = new Employee();
emp.name = "John";

update(emp);

System.out.println(emp.name);
```

Output:

```text
Aryan
```

---

### 162. What happens if the method assigns a completely new object to the parameter?

The caller's reference does **not** change.

```java
static void change(Employee e) {
    e = new Employee();
    e.name = "New";
}

Employee emp = new Employee();
emp.name = "Old";

change(emp);

System.out.println(emp.name);
```

Output:

```text
Old
```

Inside the method:

```text
Before reassignment:

e ───► Object A
emp ─► Object A


After:

e ───► Object B
emp ─► Object A
```

The method only changed its **local copy of the reference**.

---

### 163. Can a method change the caller's reference variable?

**No.**

Because the method receives a copy of the reference.

```java
static void change(Employee e) {
    e = new Employee();
}
```

This cannot make the caller's variable point to the new object.

However, the method can mutate the object that the caller's reference points to.

---

### 164. Predict the output of a program that modifies an object and then reassigns the reference.

```java
class Employee {
    String name;
}

static void test(Employee e) {

    e.name = "Aryan";

    e = new Employee();
    e.name = "Rahul";
}
```

Caller:

```java
Employee emp = new Employee();
emp.name = "Original";

test(emp);

System.out.println(emp.name);
```

Output:

```text
Aryan
```

Step-by-step:

### Initially

```text
emp ─────► Object A
             name = "Original"
```

### Inside `test()`

```java
e.name = "Aryan";
```

Both references point to Object A:

```text
emp ──┐
      ├──► Object A
e ────┘     name = "Aryan"
```

Then:

```java
e = new Employee();
```

Now:

```text
emp ─────► Object A
             name = "Aryan"

e ───────► Object B
             name = "Rahul"
```

The caller still points to Object A.

Therefore:

```text
Aryan
```

---

### 165. How would you explain Java reference semantics to an interviewer using a simple example?

Use this:

```java
class Employee {
    String name;
}
```

```java
static void update(Employee e) {
    e.name = "Aryan";      // modifies shared object
    e = new Employee();    // changes only local reference
    e.name = "Rahul";
}

Employee emp = new Employee();
emp.name = "John";

update(emp);

System.out.println(emp.name);
```

Output:

```text
Aryan
```

Then explain:

> "Java is always pass-by-value. When an object is passed, the value being copied is the reference to that object. Therefore, both the caller and method parameter initially refer to the same object, so changing the object's fields is visible to the caller. But if I reassign the parameter to a new object, only the local copy of the reference changes; the caller's reference remains unchanged."

### The one diagram to remember

```text
Employee emp = new Employee();

        Caller                  Method
          
        emp                      e
         │                       │
         │   copied reference   │
         └──────────┬────────────┘
                    ▼
             ┌──────────────┐
             │ Employee     │
             │ name = John  │
             └──────────────┘

e.name = "Aryan"
        ↓
Same object changes
        ↓
emp.name == "Aryan"


e = new Employee()
        ↓
Only local 'e' changes

        emp ─────► Object A
        e   ─────► Object B
```

**Core rule:**

> **Java passes everything by value. For objects, the value being passed is a copy of the reference.**
