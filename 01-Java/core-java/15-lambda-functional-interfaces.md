# Java 8 — Lambda Expressions & Functional Interfaces

## 1. Why Lambdas?

Before Java 8, behavior was commonly passed using anonymous classes:

```java
Runnable runnable = new Runnable() {
    @Override
    public void run() {
        System.out.println("Hello");
    }
};
```

Java 8 lets us write:

```java
Runnable runnable =
    () -> System.out.println("Hello");
```

### Core idea

> A Lambda represents behavior that can be passed around as a value.

Mental model:

```text
Before Java 8:

object
  ↓
anonymous class
  ↓
method implementation


Java 8:

lambda
  ↓
behavior
```

---

# 2. Lambda Syntax

General syntax:

```java
(parameters) -> expression
```

or:

```java
(parameters) -> {
    // multiple statements
}
```

### No parameters

```java
() -> System.out.println("Hello")
```

### One parameter

```java
x -> System.out.println(x)
```

Parentheses around one parameter are optional when the parameter type is omitted.

This is also valid:

```java
(x) -> System.out.println(x)
```

### Multiple parameters

```java
(a, b) -> a + b
```

### Multiple statements

```java
(a, b) -> {
    int result = a + b;
    System.out.println(result);
    return result;
}
```

---

# 3. Expression Lambda vs Block Lambda

### Expression Lambda

```java
(a, b) -> a + b
```

The expression result is implicitly returned.

### Block Lambda

```java
(a, b) -> {
    return a + b;
}
```

When using `{}`, an explicit `return` is required if the functional method has a non-void return type.

These are equivalent:

```java
(a, b) -> a + b
```

```java
(a, b) -> {
    return a + b;
}
```

---

# 4. A Lambda Does Not Have a Type By Itself

Consider:

```java
(a, b) -> a + b
```

How does Java know:

```text
How many parameters?
What are their types?
What is the return type?
Which method is being implemented?
```

A Lambda gets this information from its **target type**.

The target type is normally a **functional interface**.

Example:

```java
@FunctionalInterface
interface Calculator {
    int calculate(int a, int b);
}

Calculator calculator =
    (a, b) -> a + b;
```

Java can infer:

```text
a      → int
b      → int
return → int
```

Mental model:

```text
Lambda
  ↓
target type
  ↓
Functional Interface
  ↓
one abstract method
  ↓
parameter + return contract
```

---

# 5. Functional Interface

A functional interface has exactly **one abstract method**.

Example:

```java
@FunctionalInterface
interface Operation {
    int execute(int a, int b);
}
```

Different Lambdas can provide different implementations:

```java
Operation add =
    (a, b) -> a + b;

Operation subtract =
    (a, b) -> a - b;

Operation multiply =
    (a, b) -> a * b;
```

Mental model:

```text
Functional Interface
        ↓
defines the contract / shape

Lambda
        ↓
provides the behavior
```

---

# 6. Your Runnable Example

You used:

```java
Runnable runnable =
    () -> {
        System.out.println("i am a thread");
    };

runnable.run();
```

This works because `Runnable` has one abstract method:

```java
void run();
```

Therefore Java understands:

```text
Runnable
   ↓
run()
   ↓
() -> { ... }
```

Important:

> Lambda is not specifically a threading feature. `Runnable` is simply one functional interface that can be implemented using a Lambda.

---

# 7. Your Custom Functional Interface Example

You created:

```java
interface Functional {
    int cal(int a, int b);
}
```

Then:

```java
Functional add =
    (x, y) -> x + y;

Functional sub =
    (x, y) -> y - x;

Functional mul =
    (x, y) -> x * y;
```

The same interface contract supports different behaviors:

```text
Functional
    │
    ├── add → x + y
    ├── sub → y - x
    └── mul → x * y
```

Usage:

```java
int a = 10;
int b = 15;

System.out.println(add.cal(a, b));
System.out.println(sub.cal(a, b));
System.out.println(mul.cal(a, b));
```

Your subtraction:

```java
(x, y) -> y - x
```

therefore gives:

```text
15 - 10 = 5
```

If normal `a - b` was intended:

```java
(x, y) -> x - y
```

---

# 8. `@FunctionalInterface`

You don't technically need the annotation.

This can be functional:

```java
interface Calculator {
    int calculate(int a, int b);
}
```

But this is preferable:

```java
@FunctionalInterface
interface Calculator {
    int calculate(int a, int b);
}
```

The annotation:

- documents intent
- gives a compile-time check
- prevents someone from accidentally adding another abstract method

For example:

```java
@FunctionalInterface
interface Calculator {

    int calculate(int a, int b);

    void reset();   // Compile-time error
}
```

---

# 9. Can a Functional Interface Have Other Methods?

Yes.

The requirement is:

> Exactly **one abstract method**.

It can also have:

- default methods
- static methods
- applicable methods inherited from `Object`

Example:

```java
@FunctionalInterface
interface Calculator {

    int calculate(int a, int b);

    default void log() {
        System.out.println("Calculator");
    }

    static void info() {
        System.out.println("Utility");
    }
}
```

It is still functional because:

```text
Abstract methods = 1
```

---

# 10. Target Typing ⭐⭐⭐

This was an important clarification from our discussion.

Valid:

```java
Functional operation =
    (a, b) -> a + b;
```

Invalid:

```java
var operation =
    (a, b) -> a + b;
```

### Why?

The Lambda itself doesn't tell Java:

```text
Am I implementing Runnable?
Calculator?
Predicate?
Function?
Some custom interface?
```

Java needs a target functional-interface type.

With:

```java
Functional operation =
    (a, b) -> a + b;
```

Java knows:

```text
Lambda
  ↓
Functional
  ↓
cal(int, int)
```

Therefore it knows:

```text
2 parameters
both int
return int
```

But with:

```java
var operation =
    (a, b) -> a + b;
```

there is no target functional-interface type.

### Important mental model

```text
Lambda
   ↓
needs target type
   ↓
Functional Interface
   ↓
defines parameter count/types + return type
```

---

# 11. Why `var` Works Elsewhere

This works:

```java
var number = 10;
```

because Java can infer:

```text
10 → int
```

This works:

```java
var name = "Aryan";
```

because:

```text
"Aryan" → String
```

But:

```java
var operation =
    (a, b) -> a + b;
```

doesn't work because a Lambda requires a target functional-interface type.

---

# Built-in Functional Interfaces

Java provides common functional interfaces in:

```java
java.util.function
```

The four most important are:

```text
Predicate<T>       → T → boolean

Function<T,R>      → T → R

Consumer<T>        → T → void

Supplier<T>        → () → T
```

### Quick memory trick

```text
Predicate  → asks a question
Function   → transforms something
Consumer   → performs an action
Supplier   → supplies something
```

---

# 12. Predicate<T>

Use `Predicate` when your Lambda answers a **yes/no question**.

Method:

```java
boolean test(T t);
```

Example:

```java
Predicate<Integer> isEven =
    x -> x % 2 == 0;
```

Usage:

```java
System.out.println(isEven.test(10)); // true
System.out.println(isEven.test(7));  // false
```

Mental model:

```text
Input
  ↓
Predicate
  ↓
true / false
```

---

# 13. Predicate Production Example

Suppose:

```java
Predicate<Payment> isHighValue =
    payment ->
        payment.getAmount()
               .compareTo(new BigDecimal("10000")) > 0;
```

Then:

```java
if (isHighValue.test(payment)) {
    // high-value payment
}
```

Later, this becomes useful with Streams:

```java
payments.stream()
        .filter(isHighValue)
        .toList();
```

`filter()` expects a Predicate.

---

# 14. Predicate Composition

Predicates can be combined.

```java
Predicate<Integer> isEven =
    x -> x % 2 == 0;

Predicate<Integer> greaterThan10 =
    x -> x > 10;
```

### `and()`

```java
Predicate<Integer> condition =
    isEven.and(greaterThan10);
```

Now:

```java
condition.test(12); // true
condition.test(8);  // false
condition.test(15); // false
```

### `or()`

```java
Predicate<Integer> condition =
    isEven.or(greaterThan10);
```

### `negate()`

```java
Predicate<Integer> isOdd =
    isEven.negate();
```

Mental model:

```text
Predicate
   ↓
and()
or()
negate()
   ↓
larger reusable conditions
```

---

# 15. Function<T,R>

Use `Function` when you want to:

> Take something and transform it into something else.

Method:

```java
R apply(T t);
```

Example:

```java
Function<Integer, Integer> square =
    x -> x * x;
```

Usage:

```java
int result = square.apply(5);
```

Result:

```text
25
```

Mental model:

```text
T
 ↓
Function
 ↓
R
```

---

# 16. Function Input and Output Can Differ

They don't have to be the same type.

Example:

```java
Function<String, Integer> length =
    s -> s.length();
```

Here:

```text
String
  ↓
Function
  ↓
Integer
```

Usage:

```java
length.apply("Aryan"); // 5
```

---

# 17. Function Production Example

```java
Function<Payment, BigDecimal> getAmount =
    payment -> payment.getAmount();
```

Later, Method References can shorten this:

```java
Function<Payment, BigDecimal> getAmount =
    Payment::getAmount;
```

Method References are covered separately.

---

# 18. Function Composition

Suppose:

```java
Function<Integer, Integer> multiplyBy2 =
    x -> x * 2;

Function<Integer, Integer> add10 =
    x -> x + 10;
```

Using:

```java
Function<Integer, Integer> combined =
    multiplyBy2.andThen(add10);
```

For input `5`:

```text
5
 ↓
multiplyBy2
 ↓
10
 ↓
add10
 ↓
20
```

Therefore:

```java
combined.apply(5); // 20
```

---

# 19. `andThen()` vs `compose()`

### `andThen()`

```java
f1.andThen(f2)
```

means:

```text
input
 ↓
f1
 ↓
f2
 ↓
output
```

### `compose()`

```java
f2.compose(f1)
```

also means:

```text
input
 ↓
f1
 ↓
f2
 ↓
output
```

The important distinction is how the functions are expressed from the current function's perspective.

Example:

```java
Function<Integer, Integer> multiplyBy2 =
    x -> x * 2;

Function<Integer, Integer> add10 =
    x -> x + 10;

add10.compose(multiplyBy2);
```

means:

```text
multiplyBy2
     ↓
add10
```

---

# 20. Consumer<T>

Use `Consumer` when you:

> Take something and perform an action, but don't return a result.

Method:

```java
void accept(T t);
```

Example:

```java
Consumer<String> printer =
    x -> System.out.println(x);
```

Usage:

```java
printer.accept("Hello");
```

Mental model:

```text
Input
  ↓
Consumer
  ↓
action
  ↓
nothing returned
```

---

# 21. Consumer Production Example

```java
Consumer<Payment> processPayment =
    payment -> payment.process();
```

Usage:

```java
processPayment.accept(payment);
```

Common Stream usage:

```java
payments.forEach(
    payment -> System.out.println(payment)
);
```

`forEach()` accepts a Consumer.

---

# 22. Supplier<T>

Supplier is different.

It:

> Takes nothing and supplies a value.

Method:

```java
T get();
```

Example:

```java
Supplier<Double> randomNumber =
    () -> Math.random();
```

Usage:

```java
Double value = randomNumber.get();
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
Supplier<String> idGenerator =
    () -> UUID.randomUUID().toString();
```

---

# 23. The Four Together

For:

```java
Integer number = 10;
```

### Predicate

```java
Predicate<Integer> isEven =
    x -> x % 2 == 0;
```

```text
10 → true
```

### Function

```java
Function<Integer, Integer> square =
    x -> x * x;
```

```text
10 → 100
```

### Consumer

```java
Consumer<Integer> printer =
    x -> System.out.println(x);
```

```text
10 → action
```

### Supplier

```java
Supplier<Integer> supplier =
    () -> 10;
```

```text
nothing → 10
```

---

# 24. Recognition Rule ⭐⭐⭐

When you see a functional interface, ask:

### Does it answer a question?

```text
Predicate
```

### Does it transform something?

```text
Function
```

### Does it perform an action?

```text
Consumer
```

### Does it provide/create something?

```text
Supplier
```

This is one of the easiest ways to remember them during an interview.

---

# 25. BiPredicate

Sometimes one input isn't enough.

```java
BiPredicate<Integer, Integer> greater =
    (a, b) -> a > b;
```

Mental model:

```text
T + U → boolean
```

Method:

```java
boolean test(T t, U u);
```

---

# 26. BiFunction

Two inputs, one output.

```java
BiFunction<Integer, Integer, Integer> add =
    (a, b) -> a + b;
```

Generic form:

```text
BiFunction<T, U, R>
```

Mental model:

```text
T + U → R
```

---

# 27. BiConsumer

Two inputs, no return value.

```java
BiConsumer<String, Integer> print =
    (name, age) ->
        System.out.println(name + " " + age);
```

Mental model:

```text
T + U → void
```

---

# 28. Primitive Specializations

Generic functional interfaces may involve wrapper types:

```java
Function<Integer, Integer>
```

which can involve boxing/unboxing.

Java provides primitive-specialized interfaces:

```text
IntPredicate
IntFunction
IntConsumer
IntSupplier
```

Example:

```java
IntPredicate isEven =
    x -> x % 2 == 0;
```

These can avoid unnecessary boxing/unboxing.

You do not need to memorize every primitive specialization.

Interview-level understanding:

> Primitive specializations exist for performance reasons when working with primitive values.

---

# 29. Why Not One Generic Functional Interface?

Because the contracts are different.

```text
Predicate
T → boolean

Function
T → R

Consumer
T → void

Supplier
() → T
```

They represent fundamentally different kinds of behavior.

---

# 30. Behavior as a Parameter ⭐⭐⭐

Traditional code often passes data:

```java
processPayment(payment);
```

Functional programming allows behavior to be passed too:

```java
processPayment(
    payment,
    p -> calculateFee(p)
);
```

Conceptually:

```text
Data
 ↓
Payment

Behavior
 ↓
Function<Payment, BigDecimal>
```

This is one of the major ideas behind Java's functional features.

---

# 31. Production Scenario — Payment Fee

Suppose the service supports different fee calculation behavior.

```java
Function<Payment, BigDecimal> feeCalculator =
    payment ->
        payment.getAmount()
               .multiply(new BigDecimal("0.02"));
```

Another behavior:

```java
feeCalculator =
    payment ->
        payment.getAmount()
               .multiply(new BigDecimal("0.01"));
```

Then:

```java
BigDecimal fee =
    feeCalculator.apply(payment);
```

The calling code doesn't need to know how the fee is calculated.

This is behavior being passed into the system.

---

# 32. Production Scenario — Payment Eligibility

```java
Predicate<Payment> verified =
    Payment::isVerified;

Predicate<Payment> withinLimit =
    payment ->
        payment.getAmount()
               .compareTo(LIMIT) <= 0;

Predicate<Payment> eligible =
    verified.and(withinLimit);
```

Then:

```java
if (eligible.test(payment)) {
    process(payment);
}
```

Individual business rules can be composed.

---

# 33. Lambda + Streams Connection

Later we'll write:

```java
numbers.stream()
       .filter(x -> x > 10)
       .map(x -> x * 2)
       .forEach(x -> System.out.println(x));
```

Now the functional-interface types become clear:

```text
filter()
   ↓
Predicate<T>
   ↓
x -> x > 10

map()
   ↓
Function<T,R>
   ↓
x -> x * 2

forEach()
   ↓
Consumer<T>
   ↓
x -> System.out.println(x)
```

This is why our learning order is:

```text
Lambda
   ↓
Functional Interfaces
   ↓
Method References
   ↓
Streams
```

---

# 34. Common Interview Traps

## Trap 1 — Lambda with `var`

```java
var operation =
    (a, b) -> a + b;
```

Invalid because there is no target functional-interface type.

---

## Trap 2 — Predicate vs Function

```text
Predicate<T>   → boolean
Function<T, R> → R
```

---

## Trap 3 — Consumer vs Supplier

```text
Consumer<T>
→ receives T
→ returns nothing

Supplier<T>
→ receives nothing
→ returns T
```

---

## Trap 4 — `@FunctionalInterface`

It does not create Lambda support.

It provides a compile-time check and documents intent.

---

## Trap 5 — Functional interface means one method

More accurately:

> A functional interface has exactly one **abstract** method.

It may also have default and static methods.

---

# 35. Coding Practice — Basic

### Q1

Convert:

```java
Runnable task = new Runnable() {
    @Override
    public void run() {
        System.out.println("Processing payment");
    }
};
```

to a Lambda.

### Q2

Given:

```java
@FunctionalInterface
interface Operation {
    int execute(int a, int b);
}
```

Create Lambdas for:

- addition
- subtraction
- multiplication

---

# 36. Coding Practice — Medium

### Q3

Given:

```java
class Payment {
    private BigDecimal amount;

    public BigDecimal getAmount() {
        return amount;
    }
}
```

Write:

```java
Predicate<Payment>
Function<Payment, BigDecimal>
Consumer<Payment>
```

for:

- amount > 10,000
- extract amount
- print amount

### Q4

Create:

```java
Predicate<Integer> isEven
Predicate<Integer> greaterThan10
```

Combine them using `and()`.

Test:

```text
8
12
15
```

### Q5

Create:

```java
Function<Integer, Integer> multiplyBy2
Function<Integer, Integer> add10
```

Use `andThen()` so:

```text
5 → 10 → 20
```

---

# 37. Coding Practice — Debugging / Reasoning

### Q6

Why is this invalid?

```java
var operation =
    (a, b) -> a + b;
```

How would you fix it?

### Q7

What do these return?

```java
Predicate<Integer> p1 = x -> x > 10;
Predicate<Integer> p2 = x -> x % 2 == 0;

Predicate<Integer> p3 = p1.and(p2);

p3.test(12);
p3.test(15);
p3.test(8);
```

### Q8

Explain the difference:

```java
Function<Integer, Integer>
```

vs

```java
Predicate<Integer>
```

### Q9

Explain the execution order:

```java
Function<Integer, Integer> f1 =
    x -> x * 2;

Function<Integer, Integer> f2 =
    x -> x + 10;

f1.andThen(f2).apply(5);
```

---

# 38. Interview Questions

## Basic

1. What is a Lambda expression?
2. Why were Lambdas introduced?
3. What is Lambda syntax?
4. What is a functional interface?
5. What is the relationship between Lambda and functional interface?
6. What does `@FunctionalInterface` do?
7. Can a functional interface have default methods?
8. Can it have static methods?
9. Can it have more than one method?

## Target Typing

10. Why does a Lambda need a target type?
11. Why does this fail?

```java
var x = () -> System.out.println("Hello");
```

12. Why does this work?

```java
Runnable x =
    () -> System.out.println("Hello");
```

13. How does Java infer Lambda parameter types?

## Built-in Functional Interfaces

14. What is Predicate?
15. What is Function?
16. What is Consumer?
17. What is Supplier?
18. Predicate vs Function?
19. Consumer vs Supplier?
20. What is BiPredicate?
21. What is BiFunction?
22. What is BiConsumer?
23. Why do primitive-specialized interfaces exist?

## Composition

24. How do you combine Predicates?
25. Difference between `and()`, `or()`, and `negate()`?
26. What is Function `andThen()`?
27. What is Function `compose()`?
28. Difference between `andThen()` and `compose()`?

## Design / Production

29. When would you pass behavior as a method parameter?
30. Where would Predicate be useful in a backend application?
31. Where would Function be useful?
32. Where would Consumer be useful?
33. Where would Supplier be useful?
34. When can passing behavior reduce conditional logic?
35. When might a normal class/Strategy implementation be preferable to a Lambda?

---

# 39. Final Revision Cheat Sheet

```text
LAMBDA
(parameters) -> expression


FUNCTIONAL INTERFACE
Exactly ONE abstract method


TARGET TYPING
Lambda needs a target functional-interface type

Functional f = (a,b) -> a+b;  ✓

var f = (a,b) -> a+b;         ✗


PREDICATE
T → boolean
test(T)


FUNCTION
T → R
apply(T)


CONSUMER
T → void
accept(T)


SUPPLIER
() → T
get()


BI-PREDICATE
T + U → boolean


BI-FUNCTION
T + U → R


BI-CONSUMER
T + U → void


PREDICATE COMPOSITION
and()
or()
negate()


FUNCTION COMPOSITION
andThen()
compose()


PRIMITIVE SPECIALIZATIONS
IntPredicate
IntFunction
IntConsumer
IntSupplier
...
```

---

# 40. Next Topic

```text
Lambda + Functional Interfaces     ✅
        ↓
Method References                 ← NEXT
        ↓
Streams
        ↓
Collectors
        ↓
Optional
        ↓
Date/Time API
        ↓
Modern Java
```

**Java 8 — Lambda + Functional Interfaces = DONE**