Absolutely. We’ll **park `WeakHashMap` and `IdentityHashMap`** and move on.

# Next: Java 8+ — Functional Interfaces & Lambda Expressions

This is the next section in your roadmap after Collections. Pasted markdown

And because EPAM specifically focuses heavily on **Java 8+, Streams, functional programming, and coding**, we should do this properly rather than treating lambdas as syntax.

Our sequence:

```text
Java 8+
│
├── Functional Interfaces
│   ├── What problem they solve
│   ├── @FunctionalInterface
│   ├── Predicate
│   ├── Function
│   ├── Consumer
│   ├── Supplier
│   ├── UnaryOperator
│   ├── BinaryOperator
│   └── custom functional interfaces
│
├── Lambda Expressions
│   ├── syntax
│   ├── target typing
│   ├── effectively final variables
│   ├── lambda vs anonymous class
│   └── method references
│
└── Stream API
    └── major deep-dive
```

We'll use the same pattern we've established:

**Problem → Why → Concept → Internal working → Code → Complexity → Production use → Interview questions.**

---

# 1. Functional Interface

## Problem

Before Java 8, suppose you wanted to pass behavior into a method.

For example:

> "Give me a condition, and I'll filter employees based on that condition."

Java didn't have lambda expressions.

You might create an interface:

```java
interface EmployeeFilter {
    boolean test(Employee employee);
}
```

Then:

```java
class HighSalaryFilter implements EmployeeFilter {

    @Override
    public boolean test(Employee employee) {
        return employee.getSalary() > 50000;
    }
}
```

And:

```java
filterEmployees(employees, new HighSalaryFilter());
```

This works, but for a tiny piece of behavior it's verbose.

Java 8 introduced a much cleaner way.

---

# 2. What is a Functional Interface?

A functional interface is an interface having **exactly one abstract method**.

Example:

```java
@FunctionalInterface
interface EmployeeFilter {
    boolean test(Employee employee);
}
```

It can then be represented by a lambda:

```java
EmployeeFilter filter =
        employee -> employee.getSalary() > 50000;
```

Now:

```java
filter.test(employee);
```

executes the lambda.

---

# 3. Why exactly ONE abstract method?

Because a lambda represents an implementation of **one abstract behavior**.

For:

```java
@FunctionalInterface
interface Calculator {
    int calculate(int a, int b);
}
```

we can write:

```java
Calculator addition = (a, b) -> a + b;
```

The compiler knows:

```text
(a, b) -> a + b
       ↓
calculate(int, int)
```

But imagine:

```java
interface Calculator {

    int add(int a, int b);

    int subtract(int a, int b);
}
```

Now:

```java
(a, b) -> a + b
```

Which method should it implement?

Ambiguous.

Therefore:

```text
Functional Interface
        ↓
exactly ONE abstract method
        ↓
lambda can represent it
```

---

# 4. `@FunctionalInterface`

You can explicitly tell the compiler:

```java
@FunctionalInterface
interface Calculator {
    int calculate(int a, int b);
}
```

The annotation isn't what **makes** the interface functional.

The interface is functional because it has one abstract method.

The annotation gives you compiler validation.

For example:

```java
@FunctionalInterface
interface Calculator {

    int calculate(int a, int b);

    int anotherMethod(int a, int b);
}
```

Compiler error.

That's useful because it prevents someone from accidentally changing the interface into something that is no longer functional.

---

# 5. Can Functional Interfaces have other methods?

Yes.

This is an important interview detail.

They can have:

```text
1 abstract method
+
default methods
+
static methods
+
Object methods such as equals()
```

Example:

```java
@FunctionalInterface
interface Calculator {

    int calculate(int a, int b);

    default void print() {
        System.out.println("Calculator");
    }

    static void info() {
        System.out.println("Utility");
    }
}
```

Still functional because there is only **one abstract method**:

```java
calculate()
```

---

# 6. The four most important built-in Functional Interfaces

You absolutely need these for EPAM:

```text
Predicate
Function
Consumer
Supplier
```

Mental model:

```text
Predicate → input → boolean

Function  → input → output

Consumer  → input → nothing

Supplier  → nothing → output
```

Memorize this table:

| Interface | Input | Output |
|---|---|---|
| `Predicate<T>` | T | boolean |
| `Function<T,R>` | T | R |
| `Consumer<T>` | T | void |
| `Supplier<T>` | nothing | T |

---

# 7. Predicate

Used when you want to **test a condition**.

```java
Predicate<Integer> isEven =
        n -> n % 2 == 0;
```

Usage:

```java
System.out.println(isEven.test(10));
```

Output:

```text
true
```

Method:

```java
boolean test(T t)
```

Mental model:

```text
input
  ↓
Predicate
  ↓
true / false
```

Very common in Streams:

```java
employees.stream()
         .filter(e -> e.getSalary() > 50000);
```

The lambda passed to `filter()` behaves like a `Predicate<Employee>`.

---

# 8. Function

`Function<T,R>` transforms one thing into another.

```java
Function<String, Integer> length =
        s -> s.length();
```

Usage:

```java
int result = length.apply("Java");
```

Result:

```text
4
```

Method:

```java
R apply(T t)
```

Mental model:

```text
Employee
   ↓
Function
   ↓
String
```

Example:

```java
Function<Employee, String> getName =
        Employee::getName;
```

This becomes extremely important in:

```java
map()
```

because:

```java
.map(Employee::getName)
```

is essentially using a `Function<Employee, String>`.

---

# 9. Consumer

Consumer accepts something but returns nothing.

```java
Consumer<String> printer =
        s -> System.out.println(s);
```

Usage:

```java
printer.accept("Java");
```

Method:

```java
void accept(T t)
```

Mental model:

```text
input
  ↓
Consumer
  ↓
side effect
```

Example:

```java
employees.forEach(
    employee -> System.out.println(employee.getName())
);
```

The lambda behaves like a:

```text
Consumer<Employee>
```

---

# 10. Supplier

Supplier takes no input but produces a value.

```java
Supplier<Double> random =
        () -> Math.random();
```

Usage:

```java
Double value = random.get();
```

Method:

```java
T get()
```

Mental model:

```text
nothing
   ↓
Supplier
   ↓
value
```

Production example:

```java
Supplier<UUID> idGenerator =
        UUID::randomUUID;
```

---

# 11. The four mental models

This is worth writing down exactly:

```text
Predicate
T → boolean
"Should I allow this?"

Function
T → R
"Transform this."

Consumer
T → void
"Do something with this."

Supplier
() → T
"Give me something."
```

If an interviewer gives you a lambda and asks what functional interface it represents, this mental model helps immediately.

---

# 12. Primitive-specialized versions

Java also provides specialized functional interfaces to avoid unnecessary boxing.

For example:

```java
Predicate<Integer>
```

works with `Integer`.

But:

```java
IntPredicate
```

works directly with primitive `int`.

Similarly:

```text
Function<T,R>
IntFunction<R>
ToIntFunction<T>
IntToLongFunction
IntUnaryOperator
IntBinaryOperator
```

This becomes particularly relevant when discussing **Streams and performance**.

---

## What I want you to note right now

Don't write pages yet. Put this in your notes:

```text
FUNCTIONAL INTERFACE
────────────────────────────

Interface with exactly ONE abstract method.

@FunctionalInterface
→ compiler validation
→ doesn't create the functional property

Can contain:
• one abstract method
• default methods
• static methods
• Object methods

Core interfaces:

Predicate<T>
T → boolean
test()

Function<T,R>
T → R
apply()

Consumer<T>
T → void
accept()

Supplier<T>
() → T
get()
```

**Next within this topic:** we'll go deeper into **Lambda expressions — how the compiler understands them, target typing, effectively-final variables, lambda vs anonymous class, and method references.** Then we'll enter the Stream API, where this all starts coming together.