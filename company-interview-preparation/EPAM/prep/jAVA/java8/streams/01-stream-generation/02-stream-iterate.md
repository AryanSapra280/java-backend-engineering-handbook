# STREAM API — 1.6 `Stream.iterate()`

## 1. Problem

Sometimes we don't have an existing collection or array.

Instead, we have a **starting value and a rule for generating the next value**.

For example:

```text
1
2
3
4
5
6
...
```

The rule is:

```text
next = current + 1
```

Or:

```text
1
2
4
8
16
32
...
```

The rule is:

```text
next = current * 2
```

This is exactly the kind of problem `Stream.iterate()` solves.

---

# 2. Why `Stream.iterate()` exists

`IntStream.range()` is excellent when you already know:

```text
start
end
```

For example:

```java
IntStream.rangeClosed(1, 100);
```

But what if the sequence isn't simply an integer range?

For example:

```text
1 → 2 → 4 → 8 → 16 → 32
```

or:

```text
1 → 3 → 6 → 10 → 15
```

You need a **generation rule**.

That's where:

```java
Stream.iterate()
```

comes in.

---

# 3. Core concept

The basic form is:

```java
Stream.iterate(seed, nextFunction)
```

Example:

```java
Stream.iterate(1, n -> n + 1)
```

Break it down:

```text
seed
 ↓
1
 ↓
n -> n + 1
 ↓
2
 ↓
n -> n + 1
 ↓
3
 ↓
n -> n + 1
 ↓
4
 ↓
...
```

So:

```java
Stream.iterate(1, n -> n + 1)
```

means:

> Start with `1`, and repeatedly apply `n -> n + 1` to generate the next element.

---

# 4. What is the type?

If you write:

```java
Stream.iterate(1, n -> n + 1)
```

the result is:

```java
Stream<Integer>
```

Notice this is **not**:

```java
IntStream
```

It is:

```java
Stream<Integer>
```

because the standard two-argument `iterate()` is defined on `Stream<T>`.

This means the values are boxed `Integer` objects.

---

# 5. The two arguments

```java
Stream.iterate(
    seed,
    nextFunction
)
```

### First argument — seed

The initial value.

```java
1
```

### Second argument — UnaryOperator

A function that receives the current value and produces the next value.

```java
n -> n + 1
```

The functional interface involved is:

```java
UnaryOperator<T>
```

Remember what we learned earlier:

```text
UnaryOperator<T>
        ↓
T → T
```

Therefore:

```java
n -> n + 1
```

takes:

```text
Integer
```

and returns:

```text
Integer
```

---

# 6. Internal execution model

Suppose:

```java
Stream.iterate(1, n -> n * 2)
```

Conceptually:

```text
seed = 1

current = 1
emit 1

next = 1 * 2
current = 2
emit 2

next = 2 * 2
current = 4
emit 4

next = 4 * 2
current = 8
emit 8
```

Therefore:

```text
1, 2, 4, 8, 16, 32, ...
```

The important mental model is:

```text
current value
      ↓
apply function
      ↓
next value
      ↓
apply function
      ↓
next value
      ↓
...
```

---

# 7. Very important: `iterate()` is potentially infinite

This is one of the most important interview points.

Consider:

```java
Stream.iterate(1, n -> n + 1);
```

There is no termination condition.

Therefore:

```text
1, 2, 3, 4, 5, 6, ...
```

continues indefinitely.

So this is dangerous:

```java
Stream.iterate(1, n -> n + 1)
      .forEach(System.out::println);
```

It will keep producing values.

You normally need something to limit or terminate the stream.

---

# 8. `limit()`

The most common way is:

```java
Stream.iterate(1, n -> n + 1)
      .limit(10)
      .forEach(System.out::println);
```

Output:

```text
1
2
3
4
5
6
7
8
9
10
```

Now the pipeline is finite.

Mental model:

```text
iterate()
   ↓
1,2,3,4,5,6,...
   ↓
limit(10)
   ↓
1,2,3,4,5,6,7,8,9,10
```

---

# 9. `iterate()` + `limit()` for interview coding

Suppose interviewer asks:

> Generate numbers 1 to 200 using `Stream.iterate()`.

You can write:

```java
Stream.iterate(1, n -> n + 1)
      .limit(200)
      .forEach(System.out::println);
```

But there's an important distinction.

If the interviewer asks:

> Generate numbers 1 to 200.

The simpler choice is:

```java
IntStream.rangeClosed(1, 200);
```

If they specifically ask:

> Use `Stream.iterate()`.

Then:

```java
Stream.iterate(1, n -> n + 1)
      .limit(200);
```

---

# 10. `iterate()` with more interesting sequences

This is where `iterate()` becomes more useful than `range()`.

### Powers of 2

```java
Stream.iterate(1, n -> n * 2)
      .limit(10)
      .forEach(System.out::println);
```

Output:

```text
1
2
4
8
16
32
64
128
256
512
```

---

### Odd numbers

```java
Stream.iterate(1, n -> n + 2)
      .limit(10)
      .forEach(System.out::println);
```

Output:

```text
1
3
5
7
9
11
13
15
17
19
```

---

### Multiples of 5

```java
Stream.iterate(5, n -> n + 5)
      .limit(10)
      .forEach(System.out::println);
```

Output:

```text
5
10
15
20
25
30
35
40
45
50
```

---

# 11. Java 9 — three-argument `iterate()`

This is very important.

Java 9 introduced:

```java
Stream.iterate(
    seed,
    predicate,
    nextFunction
)
```

Syntax:

```java
Stream.iterate(
    seed,
    condition,
    nextValue
)
```

For example:

```java
Stream.iterate(
        1,
        n -> n <= 10,
        n -> n + 1
)
.forEach(System.out::println);
```

Output:

```text
1
2
3
4
5
6
7
8
9
10
```

---

# 12. Why was the three-argument version introduced?

Previously, you often needed:

```java
Stream.iterate(1, n -> n + 1)
      .limit(10);
```

Now you can express a stopping condition directly:

```java
Stream.iterate(
        1,
        n -> n <= 10,
        n -> n + 1
);
```

Mental model:

```text
seed
 ↓
check predicate
 ↓
if true → emit
 ↓
generate next
 ↓
check predicate
 ↓
...
 ↓
predicate false
 ↓
STOP
```

---

# 13. Example with a condition

Generate numbers until they reach 100:

```java
Stream.iterate(
        1,
        n -> n <= 100,
        n -> n + 1
)
.forEach(System.out::println);
```

This produces:

```text
1 ... 100
```

Notice the condition:

```java
n -> n <= 100
```

controls whether the next element is accepted.

---

# 14. Difference between `limit()` and predicate-based `iterate()`

### Using `limit()`

```java
Stream.iterate(1, n -> n + 1)
      .limit(100);
```

You're saying:

> Give me exactly the first 100 generated values.

### Using three-argument `iterate()`

```java
Stream.iterate(
        1,
        n -> n <= 100,
        n -> n + 1
);
```

You're saying:

> Keep generating while the condition is true.

That's a conceptual difference worth remembering.

---

# 15. Another important example

Consider:

```java
Stream.iterate(
        2,
        n -> n <= 100,
        n -> n * 2
)
.forEach(System.out::println);
```

Let's trace:

```text
2
 ↓
4
 ↓
8
 ↓
16
 ↓
32
 ↓
64
 ↓
128
```

But `128 <= 100` is false.

So output is:

```text
2
4
8
16
32
64
```

The `128` is **not emitted** because the predicate fails before that value becomes part of the stream.

This distinction is useful when reasoning about the three-argument form.

---

# 16. `iterate()` vs `range()`

This is an important interviewer comparison.

### `range()`

```java
IntStream.rangeClosed(1, 100);
```

Best when:

```text
simple numeric sequence
known boundaries
primitive int processing
```

### `iterate()`

```java
Stream.iterate(1, n -> n * 2)
```

Best when:

```text
next value depends on previous value
custom sequence generation
potentially infinite sequence
```

Mental model:

```text
range()
→ "Give me numbers in this interval."

iterate()
→ "Start here and tell me how to produce the next value."
```

---

# 17. `iterate()` vs `generate()`

We'll study `generate()` next, but understand the fundamental distinction now.

### `iterate()`

The next value depends on the **previous value**:

```java
Stream.iterate(1, n -> n + 2)
```

```text
1 → 3 → 5 → 7 → 9
```

### `generate()`

Each value comes from a **Supplier**, with no required relationship to the previous value:

```java
Stream.generate(Math::random)
```

Conceptually:

```text
random()
random()
random()
random()
...
```

So:

```text
iterate()
→ state/progression

generate()
→ independent value generation
```

We'll go deep into `generate()` separately.

---

# 18. Complexity

Suppose:

```java
Stream.iterate(1, n -> n + 1)
      .limit(n);
```

If we consume `n` elements:

```text
Time: O(n)
```

The stream itself is lazy.

It doesn't generate all values upfront.

With:

```java
.limit(10)
```

only the required values need to be generated.

---

# 19. Production considerations

### Don't accidentally create an unbounded pipeline

Bad:

```java
Stream.iterate(1, n -> n + 1)
      .forEach(...);
```

unless you intentionally have some external termination mechanism.

Usually:

```java
.limit(...)
```

or Java 9's predicate form is needed.

---

### Prefer simpler APIs when available

Don't use:

```java
Stream.iterate(1, n -> n + 1)
      .limit(100);
```

just to show off Streams if:

```java
IntStream.rangeClosed(1, 100);
```

expresses the requirement better.

Production code should favor the **clearest abstraction**, not the most complicated one.

---

# 20. Interview questions naturally covered by this topic

### What is `Stream.iterate()`?

A Stream creation mechanism where a seed value is repeatedly transformed using a `UnaryOperator` to produce subsequent values.

---

### Why can `iterate()` be infinite?

Because the two-argument form:

```java
Stream.iterate(seed, nextFunction)
```

has no built-in termination condition.

---

### How do you make it finite?

```java
.limit(n)
```

or use the Java 9 three-argument form:

```java
Stream.iterate(seed, predicate, nextFunction)
```

---

### What functional interface does the second argument represent?

```java
UnaryOperator<T>
```

because:

```text
T → T
```

---

### `iterate()` vs `range()`?

```text
range()
→ fixed numeric range

iterate()
→ generated sequence based on previous value
```

---

### Generate powers of 2 using Streams.

```java
Stream.iterate(1, n -> n * 2)
      .limit(10);
```

---

### Generate 1–100 using `iterate()`.

```java
Stream.iterate(
        1,
        n -> n <= 100,
        n -> n + 1
);
```

---

# Notes — `Stream.iterate()`

```text
Stream.iterate(seed, nextFunction)

seed
→ first element

nextFunction
→ UnaryOperator<T>
→ generates next element from current element

Example:
Stream.iterate(1, n -> n + 1)

→ 1,2,3,4,5,...

Two-argument form
→ potentially infinite

Make finite:
.limit(n)

Java 9:
Stream.iterate(seed, predicate, nextFunction)

→ generates while predicate is true

Use iterate()
→ when next value depends on previous value

Use range()
→ simple numeric range
```

---

# NEXT TOPIC — `Stream.generate()`

We'll now do the same depth for:

```java
Stream.generate(Supplier<T>)
```

including **why `Supplier` fits here, how values are generated, why it is infinite, `limit()`, `iterate()` vs `generate()`, practical examples, and interview traps.**