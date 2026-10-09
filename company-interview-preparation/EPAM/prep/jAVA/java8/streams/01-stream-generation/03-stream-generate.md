# STREAM API — 1.7 `Stream.generate()`

We now move to the next **Stream creation mechanism**.

---

## 1. Problem

Suppose you need a Stream where values are produced dynamically, but there is **no previous value that determines the next value**.

For example:

```text
random
random
random
random
...
```

Each value can be generated independently.

Or:

```text
UUID
UUID
UUID
UUID
...
```

There is no sequence like:

```text
1 → 2 → 3 → 4
```

Instead, every element is obtained by **calling a value-producing function**.

That's the problem `Stream.generate()` solves.

---

# 2. Why `generate()` exists

We learned:

```java
Stream.iterate(seed, nextFunction)
```

`iterate()` means:

> Start with a value and use the previous value to calculate the next value.

For example:

```java
Stream.iterate(1, n -> n * 2)
```

produces:

```text
1 → 2 → 4 → 8 → 16 → ...
```

But sometimes there is no dependency between values.

For example:

```text
random value
random value
random value
```

So Java provides:

```java
Stream.generate(Supplier<T>)
```

---

# 3. Core concept

Syntax:

```java
Stream.generate(supplier)
```

The argument is a:

```java
Supplier<T>
```

Remember from Functional Interfaces:

```text
Supplier<T>
     ↓
() → T
```

It takes **nothing** and produces a value.

For example:

```java
Stream.generate(() -> Math.random())
```

The Supplier is:

```java
() -> Math.random()
```

Every time the Stream needs another element, the Supplier is invoked.

---

# 4. Internal mental model

Consider:

```java
Stream.generate(() -> Math.random())
```

Think:

```text
call Supplier
     ↓
0.42
     ↓
emit

call Supplier
     ↓
0.17
     ↓
emit

call Supplier
     ↓
0.81
     ↓
emit

...
```

There is **no dependency**:

```text
0.42 → 0.17
```

The second value doesn't come from the first.

Each value is generated independently by the Supplier.

---

# 5. Basic example

```java
Stream.generate(() -> Math.random())
      .limit(5)
      .forEach(System.out::println);
```

Possible output:

```text
0.382
0.921
0.144
0.673
0.517
```

The actual numbers will obviously vary.

---

# 6. Why `limit()` is extremely important

The basic form:

```java
Stream.generate(() -> Math.random())
```

is **unbounded**.

There is no natural stopping point.

So this:

```java
Stream.generate(() -> Math.random())
      .forEach(System.out::println);
```

is effectively an endless operation.

Normally:

```java
Stream.generate(...)
      .limit(10)
```

is used when you want a finite number of generated elements.

---

# 7. `generate()` with UUID

A practical example:

```java
Stream.generate(UUID::randomUUID)
      .limit(5)
      .forEach(System.out::println);
```

Conceptually:

```text
UUID #1
UUID #2
UUID #3
UUID #4
UUID #5
```

This demonstrates why `Supplier<T>` is appropriate:

```text
Supplier
   ↓
generate a new value
```

---

# 8. `generate()` with a constant

Consider:

```java
Stream.generate(() -> "JAVA")
      .limit(5)
      .forEach(System.out::println);
```

Output:

```text
JAVA
JAVA
JAVA
JAVA
JAVA
```

The Supplier is invoked repeatedly.

It doesn't mean the Stream stores `"JAVA"` five times beforehand.

Conceptually:

```text
Supplier.get()
 ↓
JAVA

Supplier.get()
 ↓
JAVA

Supplier.get()
 ↓
JAVA
```

---

# 9. `generate()` with objects

Suppose we have:

```java
class Employee {
    private String name;

    Employee(String name) {
        this.name = name;
    }
}
```

We could generate employees:

```java
Stream.generate(() -> new Employee("Unknown"))
      .limit(10);
```

Each Supplier invocation creates a new `Employee` object.

This distinction matters:

```java
Stream.generate(() -> new Employee("Unknown"))
```

creates a **new object per invocation**.

Whereas:

```java
Employee employee = new Employee("Unknown");

Stream.generate(() -> employee)
```

returns a reference to the **same object** repeatedly.

That's a subtle but useful production/interview point.

---

# 10. `generate()` vs `iterate()`

This is one of the most important comparisons.

## `iterate()`

```java
Stream.iterate(
    1,
    n -> n + 1
)
```

Relationship:

```text
current
   ↓
function
   ↓
next
```

Example:

```text
1 → 2 → 3 → 4 → 5
```

The next value depends on the previous value.

---

## `generate()`

```java
Stream.generate(Math::random)
```

Relationship:

```text
Supplier → value
Supplier → value
Supplier → value
```

Example:

```text
random
random
random
random
```

There is no required relationship between consecutive values.

### Mental model

```text
iterate()
→ "What comes next?"

generate()
→ "Give me another value."
```

---

# 11. `generate()` vs `range()`

Another useful comparison:

### `range()`

```java
IntStream.rangeClosed(1, 10)
```

Produces a deterministic numeric range:

```text
1 2 3 4 5 6 7 8 9 10
```

### `generate()`

```java
Stream.generate(Math::random)
      .limit(10)
```

Produces values based on a Supplier:

```text
random random random ...
```

So:

```text
range()
→ predefined numerical range

iterate()
→ sequence based on previous value

generate()
→ independently generated values
```

---

# 12. Why does `generate()` use `Supplier`?

This directly connects to the Functional Interface section we already learned.

`Supplier<T>` means:

```java
T get();
```

No input.

Only output.

`generate()` needs exactly that behavior:

```text
Stream needs next element
        ↓
call Supplier.get()
        ↓
receive element
```

Therefore:

```java
Stream.generate(Supplier<T>)
```

is a natural API design.

---

# 13. Practical example — random numbers

Generate 5 random integers between 1 and 100:

```java
Stream.generate(() -> ThreadLocalRandom.current().nextInt(1, 101))
      .limit(5)
      .forEach(System.out::println);
```

Possible output:

```text
37
82
14
91
53
```

Notice:

```java
ThreadLocalRandom.current().nextInt(1, 101)
```

generates values in:

```text
1 <= n < 101
```

therefore:

```text
1–100
```

---

# 14. Practical example — repeatedly generate timestamps

You could technically do:

```java
Stream.generate(Instant::now)
      .limit(5)
      .forEach(System.out::println);
```

Each Supplier invocation calls:

```java
Instant.now()
```

and therefore produces the current instant at that point.

This demonstrates that `generate()` is useful when the value is obtained from an external/dynamic source.

---

# 15. Is `generate()` lazy?

Yes.

Consider:

```java
Stream.generate(() -> {
    System.out.println("Generating...");
    return Math.random();
});
```

Nothing happens merely because the Stream was created.

You need a terminal operation.

For example:

```java
Stream.generate(() -> {
    System.out.println("Generating...");
    return Math.random();
})
.limit(3)
.forEach(System.out::println);
```

Now the Supplier is invoked as elements are requested.

Conceptually:

```text
create pipeline
      ↓
nothing generated yet
      ↓
terminal operation
      ↓
request element
      ↓
Supplier.get()
      ↓
request next
      ↓
Supplier.get()
```

This connects directly to the **lazy Stream execution model** we will study more deeply.

---

# 16. Complexity

Suppose:

```java
Stream.generate(supplier)
      .limit(n)
```

and the Supplier itself takes O(1).

Then:

```text
Time:  O(n)
Space: O(1)
```

for a simple terminal operation that doesn't accumulate all results.

If you collect the values:

```java
List<Double> values =
    Stream.generate(Math::random)
          .limit(n)
          .toList();
```

then storing the result requires:

```text
Space: O(n)
```

The Stream itself doesn't inherently mean O(n) memory.

The **terminal operation and pipeline** determine whether results are accumulated.

---

# 17. Production considerations

### Don't use an unbounded generated Stream accidentally

Bad:

```java
Stream.generate(...)
      .collect(Collectors.toList());
```

This has no natural termination.

You need:

```java
.limit(...)
```

or another terminating mechanism.

---

### Be careful with expensive Suppliers

Suppose:

```java
Stream.generate(() -> expensiveDatabaseCall())
      .limit(1000);
```

Now you're potentially doing:

```text
1000 database calls
```

The Stream API doesn't magically make the underlying operation efficient.

Always consider the cost of the Supplier.

---

### Side effects inside Supplier

Avoid using:

```java
Stream.generate(() -> {
    updateDatabase();
    return getSomething();
})
```

unless the side effect is explicitly intended and carefully controlled.

Streams are generally easier to reason about when the pipeline remains focused on transformation rather than hidden side effects.

---

# 18. Interview questions naturally covered

### What is `Stream.generate()`?

It creates a Stream whose elements are supplied by repeatedly invoking a `Supplier`.

---

### Which functional interface does it use?

```java
Supplier<T>
```

because:

```text
Supplier
() → T
```

---

### Is `generate()` finite?

Not by default.

It is potentially infinite.

Use:

```java
.limit(n)
```

to constrain it.

---

### Difference between `iterate()` and `generate()`?

```text
iterate()
→ next value depends on previous value

generate()
→ each value is obtained independently from Supplier
```

---

### Generate five random numbers.

```java
Stream.generate(Math::random)
      .limit(5)
      .forEach(System.out::println);
```

---

### Why can't you simply use `forEach()` on an unbounded generated Stream?

Because:

```java
Stream.generate(...)
      .forEach(...)
```

has no natural termination and can continue indefinitely.

---

# Notes — `Stream.generate()`

```text
Stream.generate(Supplier<T>)

Supplier:
() → T

generate()
→ repeatedly calls Supplier
→ each value can be independent
→ potentially infinite
→ lazy

Example:

Stream.generate(Math::random)
      .limit(5);

iterate()
→ next value depends on previous value

generate()
→ next value comes from Supplier

Use limit() to make generated Stream finite.
```

---

# STREAM CREATION — COMPLETE

We now have:

```text
Collection.stream()
        ↓
Stream.of()
        ↓
Arrays.stream()
        ↓
IntStream.of()
        ↓
IntStream.range()
        ↓
IntStream.rangeClosed()
        ↓
Stream.iterate()
        ↓
Stream.generate()
```

The next topic is **not another random API**.

# NEXT: STREAM PIPELINE & INTERNAL EXECUTION

We'll now build the mental model that makes almost every Stream interview question easier:

```text
Source
  ↓
Stream
  ↓
Intermediate operations
  ↓
Terminal operation
```

Then we'll go deep into:

- **lazy evaluation**
- how elements actually flow through the pipeline
- why intermediate operations don't execute immediately
- **stateless vs stateful** operations
- short-circuiting
- stream reuse
- encounter order
- why Streams don't modify the source
- what actually triggers execution

This is the foundation before we start `filter()`, `map()`, and `flatMap()`.