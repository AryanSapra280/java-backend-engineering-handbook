# Java 8 — Method References

## 1. What is a Method Reference?

A method reference lets us refer to an **existing method** instead of writing a Lambda that simply calls that method.

Example:

### Lambda

```java
List<String> names =
    List.of("Aryan", "Rahul", "Amit");

names.forEach(
    name -> System.out.println(name)
);
```

### Method Reference

```java
names.forEach(System.out::println);
```

The method reference is:

```java
System.out::println
```

### Mental model

```text
Lambda:

x -> someMethod(x)

Method reference:

SomeClass::someMethod
```

A method reference is essentially a concise way to express behavior that already exists as a method.

---

# 2. Why `::`?

The syntax is:

```java
ClassName::methodName
```

or:

```java
object::methodName
```

or:

```java
ClassName::new
```

The `::` means:

> Use this existing method/constructor as the behavior expected by the target functional interface.

---

# 3. Method Reference Is NOT a Method Call

This is important.

```java
System.out.println("Hello");
```

is a method invocation.

It immediately executes the method.

But:

```java
System.out::println
```

is a method reference.

It represents the method and can be used where a compatible functional interface is expected.

Example:

```java
Consumer<String> printer =
    System.out::println;
```

The actual invocation happens when:

```java
printer.accept("Hello");
```

is called.

Mental model:

```text
System.out::println
       ↓
method reference
       ↓
Consumer<String>
       ↓
printer.accept("Hello")
       ↓
println("Hello")
```

---

# 4. Four Types of Method References

There are four important categories:

```text
1. Static method
2. Instance method of a particular object
3. Instance method of an arbitrary object of a particular type
4. Constructor
```

---

# 5. Static Method Reference

Suppose:

```java
class MathUtil {

    static int add(int a, int b) {
        return a + b;
    }
}
```

### Lambda

```java
BiFunction<Integer, Integer, Integer> add =
    (a, b) -> MathUtil.add(a, b);
```

### Method Reference

```java
BiFunction<Integer, Integer, Integer> add =
    MathUtil::add;
```

Mental model:

```text
(a, b) -> MathUtil.add(a, b)

              ↓

MathUtil::add
```

Because `add()` is static:

```java
ClassName::staticMethod
```

---

# 6. Instance Method of a Particular Object

Suppose:

```java
String prefix = "Payment: ";
```

We can use its instance method:

```java
Function<String, String> addPrefix =
    prefix::concat;
```

Here:

```text
prefix
  ↓
specific object
  ↓
concat()
```

General form:

```java
object::instanceMethod
```

The object is already known.

---

# 7. `System.out::println`

This is one of the most common examples.

`System.out` is a particular `PrintStream` object.

Therefore:

```java
System.out::println
```

is an instance-method reference on a specific object.

Equivalent Lambda:

```java
x -> System.out.println(x)
```

So:

```java
Consumer<String> printer =
    System.out::println;
```

is equivalent to:

```java
Consumer<String> printer =
    x -> System.out.println(x);
```

---

# 8. Instance Method of an Arbitrary Object

This one is slightly more subtle and important.

Suppose:

```java
List<String> names =
    List.of("Aryan", "Rahul", "Amit");
```

We want to sort ignoring case.

### Lambda

```java
names.sort(
    (a, b) -> a.compareToIgnoreCase(b)
);
```

### Method Reference

```java
names.sort(
    String::compareToIgnoreCase
);
```

But `compareToIgnoreCase()` is an instance method.

Which object is being used?

The first argument becomes the object on which the method is called.

Conceptually:

```java
String::compareToIgnoreCase
```

means:

```java
(a, b) -> a.compareToIgnoreCase(b)
```

Mental model:

```text
ClassName::instanceMethod

        ↓

(a, b) -> a.instanceMethod(b)
```

---

# 9. Another Arbitrary-Object Example

Suppose:

```java
List<String> names =
    List.of("Aryan", "Rahul", "Amit");
```

Lambda:

```java
names.forEach(
    name -> name.toUpperCase()
);
```

Method reference:

```java
names.forEach(
    String::toUpperCase
);
```

Conceptually:

```text
name -> name.toUpperCase()

          ↓

String::toUpperCase
```

The String object is supplied by the functional interface invocation.

---

# 10. Constructor Reference

You can also reference constructors.

Suppose:

```java
class Payment {

    public Payment() {
    }
}
```

### Lambda

```java
Supplier<Payment> supplier =
    () -> new Payment();
```

### Constructor Reference

```java
Supplier<Payment> supplier =
    Payment::new;
```

Mental model:

```text
() -> new Payment()

       ↓

Payment::new
```

This is called a **constructor reference**.

---

# 11. Constructor With Parameters

Suppose:

```java
class Payment {

    private final BigDecimal amount;

    public Payment(BigDecimal amount) {
        this.amount = amount;
    }
}
```

Lambda:

```java
Function<BigDecimal, Payment> creator =
    amount -> new Payment(amount);
```

Method reference:

```java
Function<BigDecimal, Payment> creator =
    Payment::new;
```

Java looks at the target type:

```text
Function<BigDecimal, Payment>
```

which means:

```text
Input  → BigDecimal
Output → Payment
```

It then finds the compatible constructor:

```text
Payment(BigDecimal)
```

---

# 12. The Four Forms — Memorize This

```text
1. Static method

ClassName::staticMethod


2. Particular object

object::instanceMethod


3. Arbitrary object of a type

ClassName::instanceMethod


4. Constructor

ClassName::new
```

Examples:

```java
MathUtil::add

System.out::println

String::toUpperCase

String::compareToIgnoreCase

Payment::new
```

---

# 13. How to Determine Whether a Method Reference Is Valid

The most important technique:

> **Look at the target functional interface first.**

Example:

```java
Consumer<String> printer =
    System.out::println;
```

`Consumer<String>` requires:

```text
accept(String)
```

`System.out.println(String)` exists.

Therefore the method reference is compatible.

---

Another:

```java
Function<String, Integer> length =
    String::length;
```

`Function<String, Integer>` requires:

```text
String → Integer
```

`String.length()` gives:

```text
String → int
```

Java can box the `int` into `Integer`, so this is compatible.

---

# 14. Method Reference vs Lambda

Method reference:

```java
System.out::println
```

Lambda:

```java
x -> System.out.println(x)
```

A method reference is generally cleaner when the Lambda simply forwards to an existing method.

For example:

```java
x -> x.toUpperCase()
```

can become:

```java
String::toUpperCase
```

But don't force method references when there is actual logic.

For example:

```java
x -> x * 2
```

doesn't have an obvious equivalent method reference.

---

# 15. Important Arbitrary-Object Rule ⭐⭐⭐

Consider:

```java
String::compareToIgnoreCase
```

Equivalent:

```java
(a, b) -> a.compareToIgnoreCase(b)
```

The **first Lambda argument becomes the receiver object**.

It is NOT:

```java
(a, b) -> b.compareToIgnoreCase(a)
```

General mental model:

```text
ClassName::instanceMethod

        ↓

(a, remainingArguments)
        ↓
a.instanceMethod(remainingArguments)
```

This is one of the method-reference forms worth understanding rather than simply memorizing.

---

# 16. Method References and Functional Interfaces

Everything connects:

```text
Functional Interface
       ↓
defines target type

Lambda
       ↓
provides behavior

Method Reference
       ↓
provides an existing method as behavior
```

Example:

```java
Function<String, Integer> length =
    String::length;
```

The target type tells Java how to interpret the method reference.

---

# 17. Production Example — Payment Amount

Suppose:

```java
class Payment {

    private BigDecimal amount;

    public BigDecimal getAmount() {
        return amount;
    }
}
```

With a Lambda:

```java
Function<Payment, BigDecimal> amount =
    payment -> payment.getAmount();
```

With a method reference:

```java
Function<Payment, BigDecimal> amount =
    Payment::getAmount;
```

This becomes especially useful with Streams:

```java
payments.stream()
        .map(Payment::getAmount)
        .toList();
```

---

# 18. Common Places You'll See Method References

## `forEach`

```java
list.forEach(System.out::println);
```

## Streams

```java
payments.stream()
        .map(Payment::getAmount)
        .toList();
```

## Sorting

```java
names.sort(String::compareToIgnoreCase);
```

## Object Creation

```java
Supplier<Payment> supplier =
    Payment::new;
```

---

# 19. Method Reference vs Method Invocation

### Method reference

```java
System.out::println
```

Means:

> Here is the behavior represented by this method.

### Method invocation

```java
System.out.println("Hello");
```

Means:

> Execute this method now.

Remember:

```text
:: → reference

() → invocation
```

This isn't literally true for every syntactic detail, but it is a useful interview mental model.

---

# 20. Is Every Lambda Replaceable With a Method Reference?

**No.**

A method reference works when an existing method already represents the required behavior.

Example:

```java
x -> x.toUpperCase()
```

can become:

```java
String::toUpperCase
```

But:

```java
x -> x * 2
```

doesn't have an obvious equivalent method reference.

The Lambda contains custom logic rather than simply forwarding to an existing method.

---

# 21. Important Interview Question

### What is the difference between:

```java
System.out::println
```

and:

```java
System.out.println("Hello");
```

Answer:

```text
System.out::println
→ method reference
→ represents behavior
→ does not invoke the method immediately

System.out.println("Hello")
→ method invocation
→ actually executes the method
```

---

# 22. Important Interview Question

Given:

```java
Function<String, Integer> f =
    String::length;
```

How does Java know this is valid?

Because the target type:

```text
Function<String, Integer>
```

requires:

```text
String → Integer
```

and:

```java
String.length()
```

is compatible:

```text
String → int
```

with boxing from `int` to `Integer`.

Therefore:

```java
String::length
```

can satisfy the functional interface.

---

# 23. Quick Cheat Sheet

```text
METHOD REFERENCES
=================

Static method:
ClassName::method

Particular object's instance method:
object::method

Arbitrary object's instance method:
ClassName::instanceMethod

Constructor:
ClassName::new
```

Examples:

```java
MathUtil::add

System.out::println

String::toUpperCase

String::compareToIgnoreCase

Payment::new
```

---

# 24. Equivalent Lambda Examples

```java
x -> System.out.println(x)
```

↓

```java
System.out::println
```

---

```java
x -> x.toUpperCase()
```

↓

```java
String::toUpperCase
```

---

```java
(a, b) -> a.compareToIgnoreCase(b)
```

↓

```java
String::compareToIgnoreCase
```

---

```java
() -> new Payment()
```

↓

```java
Payment::new
```

---

```java
payment -> payment.getAmount()
```

↓

```java
Payment::getAmount
```

---

# 25. Coding Practice

## Q1 — Basic

Convert:

```java
Consumer<String> printer =
    x -> System.out.println(x);
```

to a method reference.

---

## Q2

Convert:

```java
Function<String, Integer> length =
    x -> x.length();
```

to a method reference.

---

## Q3

Convert:

```java
Function<Payment, BigDecimal> amount =
    payment -> payment.getAmount();
```

to a method reference.

---

## Q4 — Medium

Convert:

```java
names.sort(
    (a, b) -> a.compareToIgnoreCase(b)
);
```

to a method reference.

Explain why the first argument becomes the object on which the method is called.

---

## Q5 — Constructor

Convert:

```java
Supplier<Payment> creator =
    () -> new Payment();
```

to a constructor reference.

---

## Q6 — Reasoning

Explain the difference between:

```java
String::toUpperCase
```

and:

```java
someString.toUpperCase()
```

---

## Q7 — Interview Level

Given:

```java
Function<String, Integer> f =
    String::length;
```

Explain exactly how Java determines that `String::length` is compatible with the target functional interface.

---

# 26. Interview Questions

1. What is a method reference?
2. Why use a method reference instead of a Lambda?
3. What does `::` mean?
4. Is a method reference a method invocation?
5. What are the four types of method references?
6. Explain static method references.
7. Explain instance method references on a particular object.
8. Explain instance method references on an arbitrary object.
9. Explain constructor references.
10. What is the difference between `object::method` and `ClassName::method`?
11. How does target typing apply to method references?
12. Can every Lambda be converted into a method reference?
13. Explain `String::compareToIgnoreCase`.
14. Explain why `String::length` can be assigned to `Function<String, Integer>`.
15. What is the difference between `System.out::println` and `System.out.println()`?
16. Where are method references commonly used in Streams?
17. How are constructor references used with `Supplier` or `Function`?

---

# 27. Final Revision Cheat Sheet

```text
METHOD REFERENCE
================

Purpose:
Use an existing method as behavior.

Syntax:

Static:
ClassName::staticMethod

Specific object:
object::instanceMethod

Arbitrary object:
ClassName::instanceMethod

Constructor:
ClassName::new
```

Examples:

```java
MathUtil::add
System.out::println
String::toUpperCase
String::compareToIgnoreCase
Payment::new
Payment::getAmount
```

Key relationship:

```text
Functional Interface
        ↓
target type

Method Reference
        ↓
existing method as behavior
```

Key rule:

```text
Lambda:
x -> object.method(x)

Method reference:
object::method
```

For an arbitrary object:

```text
Lambda:
(a, b) -> a.method(b)

Method reference:
Type::method
```

---

# 28. Learning Progress

```text
Lambda + Functional Interfaces     ✅
Method References                  ✅
        ↓
Streams                            ← NEXT
        ↓
Collectors
        ↓
Optional
        ↓
Date/Time API
        ↓
Modern Java
```

**Java 8 — Method References = DONE ✅**