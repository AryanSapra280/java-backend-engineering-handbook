# B. Lambda Expressions — 1404–1445

### 1404. 🟢 What is a lambda expression?

A **lambda expression** is a concise way of representing a block of behavior that can be passed around as a value.

Syntax:

```java
(parameters) -> expression
```

or:

```java
(parameters) -> {
    statements;
}
```

Example:

```java
Runnable r = () -> System.out.println("Hello");
```

Here:

```text
()                         → parameters
->                         → lambda operator
System.out.println(...)    → body
```

A lambda itself does not have a name like a normal method.

It is primarily used as an implementation of a **functional interface**.

---

### 1405. 🟢 Why were lambda expressions introduced?

Mainly to reduce the verbosity of passing behavior.

Before Java 8:

```java
Runnable r = new Runnable() {
    @Override
    public void run() {
        System.out.println("Running");
    }
};
```

Java 8:

```java
Runnable r = () -> System.out.println("Running");
```

They are especially useful with:

* Streams
* Collections
* Callbacks
* Functional interfaces
* Asynchronous APIs

Example:

```java
numbers.stream()
       .filter(x -> x > 10)
       .map(x -> x * 2)
       .forEach(System.out::println);
```

---

### 1406. 🟢 What is the syntax of a lambda expression?

General syntax:

```java
(parameters) -> expression
```

Example:

```java
(int x, int y) -> x + y
```

Or with a block:

```java
(int x, int y) -> {
    int result = x + y;
    return result;
}
```

Common variations:

```java
() -> 10
```

```java
x -> x * 2
```

```java
(x, y) -> x + y
```

```java
(x, y) -> {
    return x + y;
}
```

---

### 1407. 🟢 Explain:

```java
(x, y) -> x + y
```

This is a lambda with:

```text
x, y      → parameters
->        → separates parameters from body
x + y     → expression/body
```

For example, if the target type is:

```java
@FunctionalInterface
interface Calculator {
    int calculate(int x, int y);
}
```

then:

```java
Calculator c = (x, y) -> x + y;
```

Calling:

```java
c.calculate(10, 20);
```

returns:

```text
30
```

The types of `x` and `y` are inferred from the target functional interface:

```text
x → int
y → int
```

---

### 1408. 🟢 What does the `->` operator represent?

`->` separates the **lambda parameters** from the **lambda body**.

```java
(x, y) -> x + y
```

means approximately:

```text
parameters → implementation
```

It is called the **lambda operator**.

It is not an operator that performs a normal runtime calculation like `+` or `==`.

---

### 1409. 🟢 Can a lambda have zero parameters?

**Yes.**

Use empty parentheses:

```java
() -> System.out.println("Hello");
```

Example:

```java
Runnable r = () -> System.out.println("Running");
```

The corresponding functional method is:

```java
void run();
```

which has no parameters.

---

### 1410. 🟢 Can a lambda have one parameter?

**Yes.**

```java
x -> x * 2
```

Parentheses can normally be omitted when there is exactly one parameter and its type is implicit.

For example:

```java
Function<Integer, Integer> f = x -> x * 2;
```

You can also write:

```java
Function<Integer, Integer> f = (x) -> x * 2;
```

Both are valid.

---

### 1411. 🟢 When can parentheses around a single lambda parameter be omitted?

When:

1. There is exactly **one parameter**
2. Its type is being inferred
3. You aren't explicitly declaring its type

Valid:

```java
x -> x * 2
```

Valid:

```java
(x) -> x * 2
```

Also valid:

```java
(int x) -> x * 2
```

But:

```java
int x -> x * 2
```

is invalid syntax.

---

### 1412. 🟢 Can a lambda have multiple parameters?

**Yes.**

For multiple parameters, parentheses are required.

```java
(x, y) -> x + y
```

With explicit types:

```java
(int x, int y) -> x + y
```

Example:

```java
BinaryOperator<Integer> add =
        (x, y) -> x + y;
```

---

### 1413. 🟢 Can a lambda contain multiple statements?

**Yes.**

Use a block body:

```java
(x, y) -> {
    int sum = x + y;
    System.out.println(sum);
    return sum;
}
```

A lambda with multiple statements requires `{}`.

---

### 1414. 🟢 When are braces required in a lambda?

Braces are required when the lambda has a **block body containing multiple statements**.

Expression body:

```java
x -> x * 2
```

Block body:

```java
x -> {
    int result = x * 2;
    return result;
}
```

You can also use braces for a single statement:

```java
x -> {
    System.out.println(x);
}
```

but then it becomes a statement body rather than an expression body.

---

### 1415. 🟢 When is `return` required inside a lambda?

For an **expression body**, you don't write `return`.

```java
x -> x * 2
```

For a **block body**, if the functional method returns a value, you need `return`.

```java
x -> {
    return x * 2;
}
```

For a `void` functional method:

```java
x -> {
    System.out.println(x);
}
```

no `return` is required.

### Trap

This is invalid:

```java
x -> {
    x * 2;
}
```

if the target functional interface expects a return value.

---

### 1416. 🟡 Can a lambda expression have an explicit parameter type?

**Yes.**

```java
(int x, int y) -> x + y
```

For example:

```java
BinaryOperator<Integer> add =
        (Integer x, Integer y) -> x + y;
```

But you generally don't need to specify the types because they can be inferred from the target type.

```java
BinaryOperator<Integer> add =
        (x, y) -> x + y;
```

### Important rule

If you explicitly specify parameter types, **all parameters must have their types specified**.

---

### 1417. 🟡 Can you mix implicit and explicit parameter types in the same lambda?

**No.**

Invalid:

```java
(x, int y) -> x + y
```

You must use either:

```java
(x, y) -> x + y
```

or:

```java
(int x, int y) -> x + y
```

You cannot mix the two styles.

---

### 1418. 🟡 What is target typing in lambda expressions?

A lambda doesn't independently determine its parameter and return types.

Its type comes from the **target context**, usually a functional interface.

Example:

```java
Function<Integer, Integer> f = x -> x * 2;
```

The compiler knows:

```text
Function<Integer, Integer>
        ↓
T = Integer
R = Integer
```

Therefore:

```text
x → Integer
return → Integer
```

Another example:

```java
Predicate<String> p = s -> s.length() > 5;
```

The compiler knows `s` is a `String` because the target type is:

```java
Predicate<String>
```

### Key idea

```text
Lambda
   ↓
Target functional interface
   ↓
Compiler infers parameter/return types
```

---

### 1419. 🟢 Can a lambda exist without a target type?

**Generally, no.**

A lambda needs a **target functional interface type** to determine what method it implements.

For example:

```java
Runnable r = () -> System.out.println("Hello");
```

Here `Runnable` provides the target type.

Similarly:

```java
Predicate<Integer> p = x -> x > 10;
```

But you cannot normally write:

```java
var p = x -> x > 10;
```

because `var` does not provide a target functional-interface type.

---

### 1420. 🔴 Why does this fail?

```java
var operation = (x, y) -> x + y;
```

Because `var` requires the initializer to have a **standalone type** that can be inferred.

A lambda doesn't have a standalone type like:

```text
LambdaType
```

Instead, it needs a target functional interface.

For example:

```java
BinaryOperator<Integer> operation =
        (x, y) -> x + y;
```

Now the compiler knows the target type:

```text
BinaryOperator<Integer>
```

and therefore knows:

```text
x → Integer
y → Integer
return → Integer
```

So:

```java
var operation = (x, y) -> x + y;
```

fails, while:

```java
var operation =
        (BinaryOperator<Integer>) (x, y) -> x + y;
```

can work because the cast supplies the target type.

---

### 1421. 🟡 How does the compiler determine the parameter types of a lambda?

It uses the **target functional interface**.

Example:

```java
Predicate<String> p = s -> s.length() > 5;
```

The compiler examines:

```java
Predicate<T>
```

and its abstract method:

```java
boolean test(T t);
```

Since:

```text
T = String
```

the lambda parameter becomes:

```text
s → String
```

Similarly:

```java
BiFunction<Integer, Integer, Long> f =
        (x, y) -> (long) x + y;
```

The compiler knows:

```text
x → Integer
y → Integer
result → Long
```

Target typing is therefore essential to lambda type inference.

---

### 1422. 🟡 Can lambda parameters have annotations?

**Yes**, provided the annotation is applicable to the parameter.

Example:

```java
(@NonNull String name) -> name.length()
```

If using explicit parameter types, annotate accordingly:

```java
(@Nullable String name) -> ...
```

You may also need parentheses because the parameter is explicitly typed.

The annotation's `@Target` determines whether it can legally be used on lambda parameters.

---

### 1423. 🟡 Can a lambda throw checked exceptions?

**Yes, but only if the target functional interface permits that checked exception.**

Example:

```java
@FunctionalInterface
interface Task {
    void execute() throws IOException;
}
```

Then:

```java
Task task = () -> {
    throw new IOException();
};
```

is valid.

But:

```java
Runnable task = () -> {
    throw new IOException(); // compile error
};
```

because:

```java
Runnable.run()
```

does not declare `IOException`.

---

### 1424. 🔴 What determines whether a lambda can throw a checked exception?

The **throws clause of the functional interface's abstract method** determines this.

For example:

```java
interface Task {
    void execute() throws IOException;
}
```

allows:

```java
Task t = () -> {
    throw new IOException();
};
```

But:

```java
Runnable r = () -> {
    throw new IOException(); // invalid
};
```

because:

```java
void run();
```

doesn't declare `IOException`.

### Important

The lambda itself doesn't independently declare a throws contract.

The **target method's contract** determines what checked exceptions are permitted.

---

### 1425. 🟡 Can a lambda capture local variables?

**Yes.**

Example:

```java
int multiplier = 10;

Function<Integer, Integer> f =
        x -> x * multiplier;
```

The lambda captures `multiplier`.

Captured local variables must be:

> **final or effectively final**

---

### 1426. 🟢 What does "effectively final" mean?

A local variable is **effectively final** if it isn't explicitly declared `final` but is assigned only once.

Example:

```java
int x = 10;

Runnable r = () -> System.out.println(x);
```

`x` is effectively final.

You don't need:

```java
final int x = 10;
```

But this makes it no longer effectively final:

```java
int x = 10;
x = 20;
```

Therefore:

```java
Runnable r = () -> System.out.println(x);
```

would not compile if `x` is being captured.

---

### 1427. 🟢 Why must captured local variables be final or effectively final?

Because local variables normally live in a method's stack frame, while a lambda may outlive that method invocation.

Example:

```java
Runnable createTask() {
    int x = 10;

    return () -> System.out.println(x);
}
```

The method returns, but the lambda may execute later.

The lambda therefore cannot depend on a mutable stack-local variable whose lifetime has ended.

Conceptually, Java captures the **value** needed by the lambda rather than giving the lambda a normal mutable alias to the local variable.

The effectively-final rule makes this model safe and predictable.

---

### 1428. 🔴 Why can't a lambda modify a captured local variable?

This is invalid:

```java
int count = 0;

Runnable r = () -> {
    count++;
};
```

because `count` would no longer be effectively final.

The important distinction is:

```text
Local variable itself → cannot be reassigned
Object referenced by local variable → may be mutable
```

Java's lambda capture semantics capture local variables by value.

It does not provide a mutable shared local-variable cell.

---

### 1429. 🟡 Can a lambda modify a mutable object referenced by a captured variable?

**Yes.**

Example:

```java
List<Integer> list = new ArrayList<>();

Consumer<Integer> consumer =
        x -> list.add(x);
```

This is valid.

Why?

The variable:

```java
list
```

still refers to the same `ArrayList` object.

We're not reassigning `list`.

We're modifying the **object it references**.

So:

```java
list = new ArrayList<>();
```

would violate effective-final capture.

But:

```java
list.add(10);
```

doesn't.

---

### 1430. 🔴 Explain the difference between modifying the variable and modifying the object referenced by the variable.

Consider:

```java
List<String> names = new ArrayList<>();
```

### Modifying the variable

```java
names = new ArrayList<>();
```

The reference stored in `names` changes.

Therefore the variable is no longer effectively final.

### Modifying the object

```java
names.add("Aryan");
```

The reference still points to the same object.

Only the object's state changes.

Therefore this is allowed inside a lambda:

```java
List<String> names = new ArrayList<>();

Runnable r = () -> names.add("Aryan");
```

Think:

```text
names ──────────→ ArrayList object
  │
  │
  X cannot change reference
  │
  └── object itself can potentially mutate
```

---

### 1431. 🟡 Can a lambda access instance variables?

**Yes.**

```java
class Employee {

    private int salary = 100;

    void process() {
        Runnable r = () -> {
            System.out.println(salary);
        };

        r.run();
    }
}
```

The lambda can access the enclosing object's instance state.

You can also explicitly write:

```java
() -> System.out.println(this.salary)
```

---

### 1432. 🟡 Can a lambda access static variables?

**Yes.**

```java
class Example {

    static int count = 10;

    void test() {
        Runnable r = () ->
                System.out.println(count);
    }
}
```

The lambda can access static members according to the normal Java access rules.

---

### 1433. 🟡 What does `this` mean inside a lambda?

Inside a lambda, `this` refers to the **`this` of the enclosing context**.

Example:

```java
class Employee {

    String name = "Aryan";

    void test() {
        Runnable r = () ->
                System.out.println(this.name);
    }
}
```

Here:

```java
this
```

refers to the `Employee` object.

A lambda **does not create a new `this`**.

---

### 1434. 🟢 How is `this` inside a lambda different from `this` inside an anonymous inner class?

This is a very common interview question.

### Lambda

```java
class Example {

    void test() {

        Runnable r = () -> {
            System.out.println(this);
        };
    }
}
```

`this` refers to the enclosing `Example` object.

### Anonymous class

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

Therefore:

```text
Lambda
this → enclosing object

Anonymous class
this → anonymous class object
```

---

### 1435. 🔴 Why does a lambda not introduce a new `this` scope?

A lambda is designed to represent **behavior**, not introduce a new object/class scope in the same way an anonymous class does.

Therefore Java deliberately defines:

```java
this
```

inside a lambda as referring to the enclosing instance.

This allows code like:

```java
button.setHandler(() -> this.handle());
```

to naturally refer to the enclosing object's method.

With an anonymous class, `this` would refer to the anonymous object, so you would need:

```java
Example.this.handle();
```

if you wanted the enclosing object.

---

### 1436. 🟡 Can a lambda access `super`?

**Yes**, when the enclosing context permits it.

For example:

```java
class Parent {
    void show() {
        System.out.println("Parent");
    }
}

class Child extends Parent {

    void test() {
        Runnable r = () -> super.show();
        r.run();
    }
}
```

Here:

```java
super.show()
```

refers to the superclass implementation of the enclosing `Child` object.

Again, the lambda doesn't create its own independent `super` context.

---

### 1437. 🔴 How does variable capture work conceptually?

Consider:

```java
void test() {

    int x = 10;

    Runnable r = () ->
            System.out.println(x);
}
```

The lambda needs access to `x` even though `test()` may finish before `r.run()` occurs.

Conceptually, the lambda captures the value of `x`.

Think:

```text
Method local:
x = 10

        ↓ capture

Lambda object/behavior
        ↓
captured value = 10
```

This is why the local variable must be final/effectively final.

### Important distinction

For local variables, think **capture of value**, not capture of a mutable stack variable.

For instance fields, the lambda can access the enclosing object (`this`) and therefore its current state.

---

### 1438. 🔴 What happens if a captured variable is modified after the lambda is created?

For a local variable, this is prohibited by the compiler.

Example:

```java
int x = 10;

Runnable r = () -> System.out.println(x);

x = 20; // compile error
```

Because `x` is no longer effectively final.

The reason is that Java does not support a lambda capturing a local variable as a mutable shared variable.

If you need mutable shared state, you need an appropriate mutable object/container, while also considering thread-safety if multiple threads are involved.

---

### 1439. 🟡 Are lambda expressions objects?

The safest interview answer is:

> A lambda expression can evaluate to an object that is an instance of a functional interface, but the language does not require every lambda to be implemented as a conventional anonymous class object.

Example:

```java
Runnable r = () -> System.out.println("Hello");
```

`r` is a reference whose runtime object implements `Runnable`.

You can do:

```java
System.out.println(r instanceof Runnable);
```

which is `true`.

But don't explain lambdas simply as:

> "The compiler creates an anonymous inner class."

That is an outdated/inaccurate simplification for modern Java.

---

### 1440. 🔴 How are lambda expressions represented at runtime?

Java commonly uses:

```text
invokedynamic
```

to implement lambda expressions.

The compiler generally does **not** generate a separate anonymous-class `.class` file for every lambda expression.

Conceptually:

```text
Lambda source
    ↓
javac
    ↓
invokedynamic instruction
    ↓
Lambda metafactory/runtime linkage
    ↓
functional-interface instance
```

This gives the JVM more flexibility to decide how the lambda should be represented and optimized.

---

### 1441. 🔴 What is `invokedynamic`?

`invokedynamic` is a JVM bytecode instruction introduced in Java 7 that supports **dynamic method invocation/linkage**.

Java 8 uses it heavily for lambda implementation.

Conceptually:

```text
invokedynamic
       ↓
bootstrap method
       ↓
runtime linkage
       ↓
appropriate call site / target
```

For lambdas, the JVM uses a bootstrap mechanism associated with the lambda metafactory to create/link the required functional-interface behavior.

### Interview point

`invokedynamic` allows the JVM/runtime to determine the implementation strategy instead of requiring the compiler to hard-code a particular generated class structure.

---

### 1442. 🔴 Why did Java use `invokedynamic` for lambdas instead of generating an anonymous class for every lambda?

There are several advantages.

### 1. Less generated class-file overhead

Generating a separate class for every lambda could create many additional classes.

### 2. Runtime optimization freedom

`invokedynamic` lets the runtime choose an efficient implementation strategy.

### 3. Better JVM optimization opportunities

The JVM can potentially optimize lambda instances, call sites, and allocations.

### 4. Future flexibility

The language/compiler doesn't have to permanently commit to a particular generated-class representation.

Conceptually:

```text
Anonymous class approach
Lambda
  ↓
Generate class
  ↓
Load class
  ↓
Create object

invokedynamic approach
Lambda
  ↓
invokedynamic
  ↓
runtime linkage
  ↓
optimized implementation
```

This is one reason a lambda should **not** simply be described as "syntactic sugar for an anonymous class."

---

### 1443. 🔴 What is `LambdaMetafactory`?

`LambdaMetafactory` is part of the JDK's method-handle infrastructure and provides mechanisms used by the JVM to create/link implementations for lambda expressions.

It is located in:

```java
java.lang.invoke.LambdaMetafactory
```

Conceptually:

```text
lambda
   ↓
invokedynamic
   ↓
bootstrap method
   ↓
LambdaMetafactory
   ↓
functional interface implementation
```

It helps turn the method/behavior information associated with a lambda into a call site capable of producing the required functional-interface instance.

### Interview answer

> `LambdaMetafactory` is a runtime mechanism used by Java's `invokedynamic`-based lambda implementation to create functional-interface instances from lambda implementation information.

---

### 1444. 🔴 Does every lambda expression create a new object every time it executes?

**Not necessarily.**

This is an important trap.

Consider:

```java
Runnable r = () -> System.out.println("Hello");
```

The JVM/runtime may reuse an instance when the lambda has **no captured state**.

But a capturing lambda can require different instances because each instance may carry different captured values.

Example:

```java
Runnable create(int x) {
    return () -> System.out.println(x);
}
```

Here the lambda captures `x`.

Calling:

```java
create(10);
create(20);
```

requires behavior associated with different captured values.

### Important

Don't state:

> Every lambda creates an object.

or:

> No lambda creates an object.

The runtime has implementation and optimization freedom.

---

### 1445. 🔴 Can the JVM optimize lambda allocation?

**Yes.**

The `invokedynamic` design gives the JVM flexibility to optimize lambda implementation.

Depending on the lambda and runtime circumstances, the JVM may optimize:

* Object allocation
* Instance reuse
* Call-site linkage
* Method invocation
* Inlining
* Capturing/non-capturing lambda representation

For example, a **non-capturing lambda**:

```java
Runnable r = () -> System.out.println("Hello");
```

doesn't need to store any captured state, so reuse can be possible.

A **capturing lambda**:

```java
int x = 10;

Runnable r = () -> System.out.println(x);
```

has captured state, so its representation has additional information associated with `x`.

### Interview takeaway

> Don't reason about lambda performance by assuming "one lambda = one anonymous object allocation." Java's `invokedynamic` and JIT give the runtime significant freedom to optimize lambda creation, linkage, and invocation.
