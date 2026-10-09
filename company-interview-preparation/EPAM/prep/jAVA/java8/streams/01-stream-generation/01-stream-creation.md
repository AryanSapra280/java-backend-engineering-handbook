Got it. From now on, **no review tasks and no asking you to send code**. I'll keep feeding you structured, note-taking-ready content, one topic at a time.

# STREAM API — 1. STREAM CREATION

## 1.1 Creating a Stream from a Collection

### Problem

You have a collection of objects and want to process them declaratively:

```java
List<Integer> numbers = List.of(10, 20, 30, 40);
```

Instead of manually iterating:

```java
for (Integer n : numbers) {
    System.out.println(n);
}
```

you can create a Stream:

```java
numbers.stream()
       .forEach(System.out::println);
```

---

## Why `stream()` exists

A `Collection` stores data.

A `Stream` represents a **pipeline for processing data**.

Think:

```text
Collection
    ↓
contains data

Stream
    ↓
processes data
```

A Stream does **not normally store another copy of the collection's elements**.

---

# 1.2 `Collection.stream()`

Syntax:

```java
collection.stream()
```

Example:

```java
List<String> names =
        List.of("Aryan", "Rahul", "Amit");

Stream<String> stream =
        names.stream();
```

You can then build a pipeline:

```java
names.stream()
     .filter(name -> name.length() > 4)
     .map(String::toUpperCase)
     .forEach(System.out::println);
```

Pipeline:

```text
List
 ↓
stream()
 ↓
filter()
 ↓
map()
 ↓
forEach()
```

---

# 1.3 Important: Stream doesn't modify the Collection

Given:

```java
List<Integer> numbers =
        List.of(1, 2, 3, 4, 5);
```

Doing:

```java
numbers.stream()
       .filter(n -> n % 2 == 0)
       .forEach(System.out::println);
```

doesn't remove odd numbers from `numbers`.

The original collection remains:

```text
1 2 3 4 5
```

The Stream is a **processing abstraction over the source**.

---

# 1.4 Stream from an Array

For arrays:

```java
int[] numbers = {1, 2, 3, 4, 5};
```

you can use:

```java
Arrays.stream(numbers)
```

Example:

```java
Arrays.stream(numbers)
      .filter(n -> n % 2 == 0)
      .forEach(System.out::println);
```

Output:

```text
2
4
```

For an object array:

```java
String[] names = {"Amit", "Rahul", "Aryan"};

Arrays.stream(names)
      .forEach(System.out::println);
```

---

# 1.5 `Stream.of()`

You can directly create a Stream from values:

```java
Stream.of("Java", "Spring", "Kafka");
```

Example:

```java
Stream.of("Java", "Spring", "Kafka")
      .forEach(System.out::println);
```

Output:

```text
Java
Spring
Kafka
```

You can also use:

```java
Stream.of(10, 20, 30, 40);
```

which produces:

```java
Stream<Integer>
```

---

# 1.6 `Stream.of()` vs `Collection.stream()`

```java
List<Integer> numbers = List.of(1, 2, 3);

numbers.stream();
```

means:

> Create a Stream from an existing collection.

Whereas:

```java
Stream.of(1, 2, 3);
```

means:

> Create a Stream directly from supplied values.

---

# 1.7 Primitive Streams

Java provides specialized streams:

```text
IntStream
LongStream
DoubleStream
```

instead of:

```text
Stream<Integer>
Stream<Long>
Stream<Double>
```

For example:

```java
IntStream.of(10, 20, 30, 40);
```

This is useful for numerical processing.

Example:

```java
int sum =
    IntStream.of(10, 20, 30, 40)
             .sum();
```

Result:

```text
100
```

---

# 1.8 Why Primitive Streams?

With:

```java
Stream<Integer>
```

values are objects:

```text
int
 ↓
Integer
```

With:

```java
IntStream
```

the stream is specialized for primitive `int`.

This can avoid unnecessary boxing/unboxing.

So:

```java
IntStream
LongStream
DoubleStream
```

are particularly useful for numerical operations.

---

# 1.9 `IntStream.range()`

Syntax:

```java
IntStream.range(startInclusive, endExclusive)
```

Example:

```java
IntStream.range(1, 5)
```

produces:

```text
1 2 3 4
```

The ending value is **exclusive**.

Mental model:

```text
range(start, end)

[start, end)
```

---

# 1.10 `IntStream.rangeClosed()`

Syntax:

```java
IntStream.rangeClosed(startInclusive, endInclusive)
```

Example:

```java
IntStream.rangeClosed(1, 5)
```

produces:

```text
1 2 3 4 5
```

Here the ending value is **inclusive**.

Mental model:

```text
rangeClosed(start, end)

[start, end]
```

---

# 1.11 `range()` vs `rangeClosed()`

| Method | Start | End |
|---|---|---|
| `range(1, 5)` | inclusive | exclusive |
| `rangeClosed(1, 5)` | inclusive | inclusive |

Therefore:

```java
IntStream.range(1, 5)
```

→ `1 2 3 4`

while:

```java
IntStream.rangeClosed(1, 5)
```

→ `1 2 3 4 5`

### Interview pattern

Generate numbers from `1` to `n`:

```java
IntStream.rangeClosed(1, n)
```

Generate numbers from `1` to `n - 1`:

```java
IntStream.range(1, n)
```

---

# 1.12 Practical Example — EPAM-style

### Problem

Generate numbers from `1` to `100`, select even numbers and calculate their sum.

```java
int sum =
    IntStream.rangeClosed(1, 100)
             .filter(n -> n % 2 == 0)
             .sum();
```

Result:

```text
2550
```

Pipeline:

```text
1 ... 100
    ↓
filter even
    ↓
2,4,6,...100
    ↓
sum()
    ↓
2550
```

Notice how our earlier topics connect:

```text
Lambda
    ↓
n -> n % 2 == 0

Functional interface
    ↓
IntPredicate

Stream
    ↓
IntStream

Intermediate operation
    ↓
filter()

Terminal operation
    ↓
sum()
```

This is exactly how we want to learn Streams: **each new API should connect back to the Java concepts we've already learned.**

---

# 1.13 Complexity

For:

```java
IntStream.rangeClosed(1, n)
         .filter(...)
         .sum();
```

assuming every number is processed:

```text
Time:  O(n)
Space: O(1)
```

The pipeline doesn't require creating a separate `List` containing all `n` integers.

---

# 1.14 Stream creation summary

```text
Collection
    ↓
collection.stream()

Array
    ↓
Arrays.stream(array)

Direct values
    ↓
Stream.of(...)

Primitive values
    ↓
IntStream.of(...)
LongStream.of(...)
DoubleStream.of(...)

Integer range
    ↓
IntStream.range(...)
IntStream.rangeClosed(...)
```

---

# Interview Questions Covered Naturally in This Topic

These are **part of Stream Creation**, not a separate question section:

### Q1. How do you create a Stream from a List?

```java
list.stream();
```

### Q2. How do you create a Stream from an array?

```java
Arrays.stream(array);
```

### Q3. How do you create a Stream directly?

```java
Stream.of(...);
```

### Q4. `range()` vs `rangeClosed()`?

```text
range       → end exclusive
rangeClosed → end inclusive
```

### Q5. Generate 1–100.

```java
IntStream.rangeClosed(1, 100);
```

### Q6. Why use `IntStream`?

For primitive `int` processing and to avoid unnecessary boxing.

### Q7. Generate and sum even numbers from 1–100.

```java
IntStream.rangeClosed(1, 100)
         .filter(n -> n % 2 == 0)
         .sum();
```

---

# NEXT TOPIC: `Stream.iterate()`

This is where we'll go deeper.

We'll cover:

```text
Why iterate() exists
        ↓
Finite vs infinite streams
        ↓
iterate(seed, UnaryOperator)
        ↓
How each next value is generated
        ↓
Java 9 three-argument iterate()
        ↓
iterate() + limit()
        ↓
iterate() + predicate
        ↓
Internal execution model
        ↓
Complexity
        ↓
Real use cases
        ↓
Interview traps
```

And we'll **not move to `generate()` until `iterate()` is genuinely complete**.