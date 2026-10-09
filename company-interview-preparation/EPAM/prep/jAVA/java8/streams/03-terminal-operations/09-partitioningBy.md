# `partitioningBy()` ⭐⭐⭐

This one is much simpler than `groupingBy()`.

The key idea:

> **`partitioningBy()` divides elements into exactly two groups: `true` and `false`.**

---

## 1. Basic example

Suppose:

```java id="3m5h9e"
List<Integer> numbers = List.of(1, 2, 3, 4, 5, 6);
```

We want to separate even and odd numbers:

```java id="8kq5l2"
Map<Boolean, List<Integer>> result =
    numbers.stream()
           .collect(
               Collectors.partitioningBy(n -> n % 2 == 0)
           );
```

Conceptually:

```text id="8u9x0k"
true  → [2, 4, 6]
false → [1, 3, 5]
```

The key is always:

```text
Boolean
```

---

# 2. `partitioningBy()` vs `groupingBy()` ⭐⭐⭐⭐⭐

This is the main interview question.

### `groupingBy()`

Can create **many groups**:

```java id="5s9y2d"
employees.stream()
         .collect(
             groupingBy(Employee::getDepartment)
         );
```

Result:

```text id="9tq3c0"
IT       → [...]
HR       → [...]
Finance  → [...]
Sales    → [...]
```

Potentially unlimited categories.

---

### `partitioningBy()`

Creates exactly two logical groups:

```java id="1cn7s9"
employees.stream()
         .collect(
             partitioningBy(Employee::isActive)
         );
```

Result:

```text id="v9g3cn"
true  → active employees
false → inactive employees
```

### Mental model

```text id="4x6j1r"
groupingBy()
    ↓
many categories

partitioningBy()
    ↓
TRUE / FALSE
```

---

# 3. Practical example

Payment system:

> Separate successful and failed payments.

```java id="8ydw0f"
Map<Boolean, List<Payment>> result =
    payments.stream()
            .collect(
                Collectors.partitioningBy(
                    Payment::isSuccessful
                )
            );
```

Now:

```text id="v4r3p1"
true  → successful payments
false → failed payments
```

This is much more expressive than manually creating two lists.

---

# 4. `partitioningBy()` with downstream collectors ⭐⭐⭐⭐

Just like `groupingBy()`, you can provide a downstream collector.

For example:

> Count successful and failed payments.

```java id="j4r7m2"
Map<Boolean, Long> counts =
    payments.stream()
            .collect(
                Collectors.partitioningBy(
                    Payment::isSuccessful,
                    Collectors.counting()
                )
            );
```

Result:

```text id="5b2v0j"
true  → 950
false → 50
```

---

# 5. `partitioningBy()` + `mapping()`

Suppose you want payment IDs:

```java id="q6h4x8"
Map<Boolean, List<String>> result =
    payments.stream()
            .collect(
                Collectors.partitioningBy(
                    Payment::isSuccessful,
                    Collectors.mapping(
                        Payment::getId,
                        Collectors.toList()
                    )
                )
            );
```

Result:

```text id="d4f8n1"
true  → [P1, P2, P5]
false → [P3, P4]
```

Same downstream collector concept we saw with `groupingBy()`.

---

# 6. `partitioningBy()` + `summingInt()`

For example:

> Calculate total amount of successful vs failed payments.

```java id="7j3q0v"
Map<Boolean, Integer> totals =
    payments.stream()
            .collect(
                Collectors.partitioningBy(
                    Payment::isSuccessful,
                    Collectors.summingInt(Payment::getAmount)
                )
            );
```

Result:

```text id="b2c6x9"
true  → 500000
false → 25000
```

---

# 7. Why use `partitioningBy()` instead of `groupingBy()`?

You technically could do:

```java id="n1x8w3"
groupingBy(Payment::isSuccessful)
```

but `partitioningBy()` communicates the intent much more clearly:

> **There are exactly two logical partitions based on a boolean condition.**

That's the important design/readability point.

---

# 8. Parallel streams ⭐⭐⭐

Yes, it works with parallel streams:

```java id="m9k2v7"
Map<Boolean, List<Payment>> result =
    payments.parallelStream()
            .collect(
                Collectors.partitioningBy(
                    Payment::isSuccessful
                )
            );
```

Conceptually:

```text id="q2x7k4"
             Payments
                 ↓
            split partitions
           /      |       \
        Part A  Part B   Part C
           ↓       ↓        ↓
        true/    true/    true/
        false    false    false
           \       |       /
            \      |      /
              combine
                 ↓
       true → [...]
       false → [...]
```

Again, don't assume:

```text id="k3n8p6"
parallel = automatically faster
```

There is partitioning and combining overhead.

For a small collection, sequential processing may be faster.

---

# 9. Complexity

For `n` elements:

```text id="r7f2m5"
Time: O(n)
Space: O(n)
```

You're categorizing every element and storing the results.

With a downstream collector, the downstream operation's cost also matters.

---

# 10. Interview questions

### Q1. What does `partitioningBy()` do?

> It partitions stream elements into two groups based on a predicate: `true` and `false`.

### Q2. Return type?

Usually:

```java id="e6c1p8"
Map<Boolean, List<T>>
```

or another value type when a downstream collector is supplied.

### Q3. `partitioningBy()` vs `groupingBy()`?

> `partitioningBy()` creates two boolean partitions, while `groupingBy()` can create multiple groups based on a classifier.

### Q4. Can you use downstream collectors?

Yes:

```java id="z9d4r1"
partitioningBy(
    predicate,
    counting()
)
```

or:

```java id="y7k3p2"
partitioningBy(
    predicate,
    mapping(...)
)
```

---

# 🧠 Lock this in

```text id="m6v2q8"
groupingBy()
     ↓
Employee → Department
     ↓
IT / HR / Finance / Sales / ...


partitioningBy()
     ↓
Employee → isActive()
     ↓
true / false
```

---

## Next: Downstream collectors ⭐⭐⭐⭐

We'll quickly cover the important ones:

```text
counting()
mapping()
joining()
summingInt()
averagingInt()
summarizingInt()
collectingAndThen()
```

The goal here isn't to memorize every `Collectors` method. It's to understand **how to combine collectors**, because that's what makes `groupingBy()` and `partitioningBy()` powerful.