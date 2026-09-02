# Java 8 — Stream Source Methods

## 1. What Is a Stream Source?

A Stream needs a **source** — somewhere from which its elements come.

Mental model:

```text
SOURCE
  ↓
STREAM
  ↓
processing pipeline
```

Examples of sources:

```text
Collection
Array
Individual values
Generated values
Iterated values
Numeric ranges
```

---

# 2. Collection → Stream

The most common way to create a Stream is from a Collection.

Example:

```java
List<Integer> numbers =
    List.of(1, 2, 3, 4, 5);

Stream<Integer> stream =
    numbers.stream();
```

For a Set:

```java
Set<String> names =
    Set.of("Aryan", "Rahul", "Amit");

Stream<String> stream =
    names.stream();
```

### Important distinction

```text
Collection
    ↓
stores data

Stream
    ↓
processes data
```

Calling:

```java
numbers.stream()
```

does not create another List containing the same elements.

It creates a Stream pipeline over the collection's elements.

---

# 3. `stream()` vs `parallelStream()`

A Collection can provide:

```java
numbers.stream();
```

or:

```java
numbers.parallelStream();
```

Basic distinction:

```text
stream()
    ↓
sequential Stream

parallelStream()
    ↓
parallel Stream
```

Do not go deep into `parallelStream()` yet.

Concurrency, thread pools, ordering, overhead, and thread-safety concerns will be covered later with multithreading/concurrency.

---

# 4. Array → Stream

For arrays, Java provides:

```java
Arrays.stream(array)
```

Example:

```java
int[] numbers = {
    1, 2, 3, 4, 5
};

IntStream stream =
    Arrays.stream(numbers);
```

Notice:

```text
int[]
 ↓
Arrays.stream()
 ↓
IntStream
```

It does not become:

```text
Stream<Integer>
```

because Java provides specialized primitive Streams.

---

# 5. Primitive Streams

Java provides:

```text
IntStream
LongStream
DoubleStream
```

These are designed for primitive numeric values.

Examples:

```java
IntStream
LongStream
DoubleStream
```

We'll discuss the reason for primitive specializations and boxing/unboxing separately.

---

# 6. Object Arrays

For an object array:

```java
String[] names = {
    "Aryan",
    "Rahul",
    "Amit"
};
```

You can write:

```java
Stream<String> stream =
    Arrays.stream(names);
```

So:

```text
int[]       → IntStream
long[]      → LongStream
double[]    → DoubleStream

String[]    → Stream<String>
Payment[]   → Stream<Payment>
```

---

# 7. `Stream.of()`

You can directly create a Stream from values.

Example:

```java
Stream<Integer> numbers =
    Stream.of(1, 2, 3, 4, 5);
```

Another example:

```java
Stream<String> names =
    Stream.of("Aryan", "Rahul", "Amit");
```

Mental model:

```text
individual values
       ↓
   Stream.of()
       ↓
     Stream
```

---

# 8. `Stream.of()` With an Object Array

Suppose:

```java
String[] names = {
    "Aryan",
    "Rahul",
    "Amit"
};
```

This works:

```java
Stream<String> stream =
    Stream.of(names);
```

Because `Stream.of(T... values)` uses varargs, the object array can be treated as the varargs elements.

Conceptually:

```text
names
 ↓
Aryan
Rahul
Amit
 ↓
Stream<String>
```

---

# 9. Important Primitive Array Trap ⭐⭐⭐

Suppose:

```java
int[] numbers = {
    1, 2, 3
};
```

If you write:

```java
Stream.of(numbers);
```

you do **not** get:

```text
Stream<Integer>
```

You effectively get:

```text
Stream<int[]>
```

Why?

Because `int[]` is itself one object/reference value from the generic Stream API's perspective.

For primitive arrays, use:

```java
Arrays.stream(numbers);
```

which gives:

```text
IntStream
```

### Interview trap

```java
int[] arr = {1, 2, 3};

Stream.of(arr);          // Stream<int[]>

Arrays.stream(arr);      // IntStream
```

This distinction is worth remembering.

---

# 10. `Stream.empty()`

You can create an empty Stream:

```java
Stream<String> stream =
    Stream.empty();
```

It contains zero elements.

This is useful when a method needs to return a Stream but there is no data.

Example:

```java
public Stream<String> getNames(boolean available) {

    if (available) {
        return Stream.of("Aryan", "Rahul");
    }

    return Stream.empty();
}
```

Mental model:

```text
data available
    ↓
Stream<String>

no data
    ↓
empty Stream<String>
```

Returning an empty Stream is often cleaner than returning `null`.

Instead of:

```java
return null;
```

you can return:

```java
return Stream.empty();
```

This allows callers to continue using Stream operations without first checking for `null`.

---

# 11. `Stream.generate()`

`Stream.generate()` creates a Stream using a `Supplier`.

Remember:

```text
Supplier<T>
() → T
```

Example:

```java
Stream.generate(() -> "Hello");
```

Conceptually:

```text
Hello
Hello
Hello
Hello
Hello
...
```

This Stream is potentially **infinite**.

Therefore, don't try to collect it without limiting it.

For example:

```java
Stream.generate(() -> "Hello")
      .limit(5)
      .toList();
```

Result:

```text
[Hello, Hello, Hello, Hello, Hello]
```

`limit()` is an intermediate operation and will be studied later.

---

# 12. `generate()` and Supplier

This is a direct connection with Functional Interfaces.

```java
Stream.generate(
    () -> UUID.randomUUID().toString()
);
```

The Lambda:

```java
() -> UUID.randomUUID().toString()
```

has the shape:

```text
() → String
```

which matches:

```text
Supplier<String>
```

So:

```text
Stream.generate()
        ↓
expects Supplier
        ↓
Supplier generates values
```

---

# 13. `Stream.iterate()`

Another way to create a potentially infinite Stream is:

```java
Stream.iterate(...)
```

Example:

```java
Stream.iterate(
    1,
    n -> n + 1
);
```

Conceptually:

```text
1
 ↓
2
 ↓
3
 ↓
4
 ↓
5
 ↓
...
```

The first argument is the initial value.

The second argument determines how to generate the next value.

Mental model:

```text
seed
 ↓
function
 ↓
next value
 ↓
function
 ↓
next value
 ↓
...
```

Example:

```text
1 → 2
2 → 3
3 → 4
4 → 5
...
```

---

# 14. Safely Using `iterate()`

Because it can be infinite:

```java
Stream.iterate(1, n -> n + 1)
```

we commonly combine it with a limiting operation.

Example:

```java
Stream.iterate(1, n -> n + 1)
      .limit(5)
      .toList();
```

Result:

```text
[1, 2, 3, 4, 5]
```

---

# 15. `iterate()` and Functional Interfaces

The second argument:

```java
n -> n + 1
```

represents:

```text
T → T
```

Conceptually:

```text
UnaryOperator<T>
```

So:

```java
Stream.iterate(
    1,
    n -> n + 1
);
```

uses a function that takes the previous value and produces the next value.

---

# 16. `Stream.iterate()` With a Predicate — Java 9

Java 9 added another overload:

```java
Stream.iterate(
    seed,
    predicate,
    next
);
```

Example:

```java
Stream.iterate(
    1,
    n -> n <= 5,
    n -> n + 1
);
```

Conceptually:

```text
1 → condition true
2 → condition true
3 → condition true
4 → condition true
5 → condition true
6 → condition false → stop
```

Result:

```text
1, 2, 3, 4, 5
```

Important:

> The three-argument `iterate()` overload is a Java 9 addition, not a Java 8 feature.

Since we're also learning modern Java, know that it exists, but don't confuse it with the original Java 8 API.

---

# 17. `IntStream.range()`

Java provides numeric Stream sources.

Example:

```java
IntStream.range(1, 5);
```

Result:

```text
1
2
3
4
```

The ending value is **exclusive**.

Mental model:

```text
[start, end)
```

Therefore:

```java
IntStream.range(1, 5)
```

means:

```text
1 <= x < 5
```

---

# 18. `IntStream.rangeClosed()`

There is also:

```java
IntStream.rangeClosed(1, 5);
```

Result:

```text
1
2
3
4
5
```

The ending value is **inclusive**.

Mental model:

```text
[start, end]
```

Comparison:

```java
IntStream.range(1, 5)
```

```text
1 2 3 4
```

while:

```java
IntStream.rangeClosed(1, 5)
```

```text
1 2 3 4 5
```

---

# 19. Finite vs Infinite Streams

Most collection Streams are finite.

Example:

```java
List<Integer> numbers =
    List.of(1, 2, 3, 4, 5);

numbers.stream();
```

There are only five elements.

But:

```java
Stream.generate(...)
```

and:

```java
Stream.iterate(...)
```

can produce potentially infinite Streams.

Therefore operations such as:

```text
limit()
findFirst()
anyMatch()
```

become important when working with them.

---

# 20. Source Methods — Quick Comparison

| Source | Example | Result |
|---|---|---|
| Collection | `list.stream()` | `Stream<T>` |
| Collection | `list.parallelStream()` | parallel `Stream<T>` |
| Object array | `Arrays.stream(names)` | `Stream<String>` |
| `int[]` | `Arrays.stream(arr)` | `IntStream` |
| Direct values | `Stream.of(1,2,3)` | `Stream<Integer>` |
| Empty | `Stream.empty()` | empty `Stream<T>` |
| Generator | `Stream.generate(...)` | potentially infinite Stream |
| Iterator | `Stream.iterate(...)` | potentially infinite Stream |
| Numeric range | `IntStream.range(...)` | `IntStream` |
| Numeric range | `IntStream.rangeClosed(...)` | `IntStream` |

---

# 21. What You Need to Know for Interviews

Know these very well:

```java
collection.stream()
```

```java
Arrays.stream(array)
```

```java
Stream.of(...)
```

```java
Stream.empty()
```

```java
Stream.generate(...)
```

```java
Stream.iterate(...)
```

And:

```java
IntStream.range(...)
IntStream.rangeClosed(...)
```

You do not need to memorize every obscure Stream factory method.

---

# 22. Important Interview Traps

## Trap 1

```java
int[] arr = {1, 2, 3};

Stream.of(arr);
```

This gives:

```text
Stream<int[]>
```

not:

```text
Stream<Integer>
```

Use:

```java
Arrays.stream(arr);
```

for:

```text
IntStream
```

---

## Trap 2

```java
IntStream.range(1, 5)
```

does not include `5`.

It gives:

```text
1, 2, 3, 4
```

Whereas:

```java
IntStream.rangeClosed(1, 5)
```

includes `5`.

---

## Trap 3

```java
Stream.generate(...)
```

can be infinite.

Don't blindly collect it.

---

## Trap 4

```java
Stream.iterate(...)
```

can also be infinite.

Use an appropriate limiting/short-circuiting operation when necessary.

---

## Trap 5

Don't confuse:

```java
list.stream()
```

with:

```java
list
```

The List stores data.

The Stream provides a processing pipeline over the data.

---

# 23. Connection With What We've Already Learned

Our Java 8 learning sequence is now connected:

```text
Lambda
   ↓
Functional Interface
   ↓
Method Reference
   ↓
Stream Source
   ↓
Intermediate Operations
   ↓
Terminal Operations
```

Examples:

### `generate()`

```java
Stream.generate(
    () -> UUID.randomUUID().toString()
);
```

uses a:

```text
Supplier<String>
```

because:

```text
() → String
```

### `iterate()`

```java
Stream.iterate(
    1,
    n -> n + 1
);
```

uses behavior equivalent to:

```text
T → T
```

---

# 24. Coding Practice

## Q1 — Basic

Create a Stream from:

```java
List<Integer> numbers =
    List.of(1, 2, 3, 4, 5);
```

---

## Q2

Create a Stream from:

```java
String[] names = {
    "Aryan",
    "Rahul",
    "Amit"
};
```

---

## Q3 — Important

Explain the difference between:

```java
Arrays.stream(arr)
```

and:

```java
Stream.of(arr)
```

when:

```java
int[] arr = {1, 2, 3};
```

---

## Q4

What is the difference between:

```java
IntStream.range(1, 5)
```

and:

```java
IntStream.rangeClosed(1, 5)
```

---

## Q5 — Medium

What happens here?

```java
Stream<String> stream =
    Stream.generate(() -> "Hello");
```

Is it finite or infinite?

How would you safely obtain exactly five values?

---

## Q6 — Interview

Why might returning:

```java
Stream.empty()
```

be preferable to:

```java
return null;
```

when a method returns `Stream<T>`?

---

## Q7 — Functional Interface Connection

Which functional interface is involved here?

```java
Stream.generate(
    () -> UUID.randomUUID().toString()
);
```

Why?

---

## Q8 — Slightly Harder

What does this produce?

```java
Stream.iterate(1, n -> n * 2)
      .limit(5)
      .toList();
```

---

# 25. Final Revision Cheat Sheet

```text
STREAM SOURCES
==============

Collection:
collection.stream()

Parallel:
collection.parallelStream()

Array:
Arrays.stream(array)

Direct values:
Stream.of(...)

Empty:
Stream.empty()

Generated:
Stream.generate(Supplier)

Iterated:
Stream.iterate(seed, UnaryOperator)

Numeric:
IntStream.range(start, end)

Numeric inclusive:
IntStream.rangeClosed(start, end)
```

### Primitive array trap

```java
int[] arr = {1, 2, 3};

Stream.of(arr);        // Stream<int[]>

Arrays.stream(arr);    // IntStream
```

### Range trap

```java
IntStream.range(1, 5)
```

```text
1 2 3 4
```

```java
IntStream.rangeClosed(1, 5)
```

```text
1 2 3 4 5
```

### Infinite sources

```java
Stream.generate(...)
Stream.iterate(...)
```

can be infinite.

---

# 26. Next Topic

```text
Stream Fundamentals                 ✅
        ↓
Stream Source Methods               ✅
        ↓
Intermediate Operations             ← NEXT
        ↓
filter()
        ↓
map()
        ↓
flatMap()
        ↓
distinct()
        ↓
sorted()
        ↓
limit()
skip()
        ↓
peek()
        ↓
Terminal Operations
```

**Stream Source Methods = DONE ✅**