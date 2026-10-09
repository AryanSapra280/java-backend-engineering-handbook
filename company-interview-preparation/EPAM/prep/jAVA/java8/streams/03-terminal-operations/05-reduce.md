# `reduce()` ⭐⭐⭐⭐⭐

This is one of the **most important Stream operations** for interviews.

The simplest mental model:

> **`reduce()` takes many elements and combines them into one result.**

For example:

```text
[1, 2, 3, 4, 5]
       ↓
    reduce
       ↓
      15
```

---

# 1. Basic `reduce()`

Suppose we want the sum:

```java
int sum =
    numbers.stream()
           .reduce(0, (a, b) -> a + b);
```

If:

```java
numbers = [1, 2, 3, 4]
```

the reduction works conceptually like:

```text
initial = 0

0 + 1 = 1
1 + 2 = 3
3 + 3 = 6
6 + 4 = 10
```

Result:

```text
10
```

---

# 2. The three important pieces

You will see this form:

```java
reduce(identity, accumulator, combiner)
```

Let's understand each one.

### Identity

```java
0
```

Starting value.

For addition:

```java
0
```

For multiplication:

```java
1
```

---

### Accumulator

```java
(a, b) -> a + b
```

It says:

> Given the current accumulated result and the next element, produce the new result.

Conceptually:

```text
accumulator(currentResult, currentElement)
```

---

### Combiner

Used primarily when the stream is processed in parallel.

```java
(a, b) -> a + b
```

It says:

> Combine two partial results.

---

# 3. The most important overloads

There are three common forms.

### Form 1

```java
Optional<T> reduce(BinaryOperator<T> accumulator)
```

Example:

```java
Optional<Integer> sum =
    numbers.stream()
           .reduce((a, b) -> a + b);
```

Because there is no identity, the stream might be empty, so the result is:

```java
Optional<Integer>
```

---

### Form 2

```java
T reduce(T identity,
         BinaryOperator<T> accumulator)
```

Example:

```java
int sum =
    numbers.stream()
           .reduce(0, (a, b) -> a + b);
```

Because we provide an identity, the result is directly:

```text
int
```

---

### Form 3 ⭐

```java
<U> U reduce(
    U identity,
    BiFunction<U, ? super T, U> accumulator,
    BinaryOperator<U> combiner
)
```

This becomes especially important when the input type and result type differ, and for parallel processing.

---

# 4. Why does `reduce()` need an identity?

Suppose:

```java
numbers = [1, 2, 3, 4]
```

For addition:

```java
reduce(0, Integer::sum)
```

The `0` is the neutral element:

```text
0 + x = x
```

For multiplication:

```java
reduce(1, (a,b) -> a*b)
```

because:

```text
1 × x = x
```

So:

| Operation | Identity |
|---|---:|
| Addition | `0` |
| Multiplication | `1` |
| String concatenation | `""` |

The identity should not change the result.

---

# 5. `Integer::sum`

You recently asked about this, so connect it here.

Instead of:

```java
.reduce(0, (a, b) -> a + b)
```

you can write:

```java
.reduce(0, Integer::sum)
```

`Integer::sum` represents:

```java
(a, b) -> a + b
```

So:

```java
int sum =
    numbers.stream()
           .reduce(0, Integer::sum);
```

---

# 6. Finding maximum using `reduce()`

You can do:

```java
int max =
    numbers.stream()
           .reduce(Integer.MIN_VALUE, Integer::max);
```

For:

```text
[10, 50, 20, 5]
```

conceptually:

```text
MIN_VALUE + 10 → 10
10 vs 50       → 50
50 vs 20       → 50
50 vs 5        → 50
```

Result:

```text
50
```

Although in practice, when you simply need the maximum, prefer:

```java
numbers.stream().max(Integer::compareTo);
```

`reduce()` is useful for understanding **general reduction**, not necessarily for replacing every specialized Stream operation.

---

# 7. Multiplication

```java
int product =
    numbers.stream()
           .reduce(1, (a, b) -> a * b);
```

For:

```text
[2, 3, 4]
```

```text
1 × 2 = 2
2 × 3 = 6
6 × 4 = 24
```

Result:

```text
24
```

---

# 8. The BIG interview topic — parallel `reduce()` ⭐⭐⭐⭐⭐

This is where the **combiner** becomes important.

Suppose:

```java
int sum =
    numbers.parallelStream()
           .reduce(
               0,
               (a, b) -> a + b,
               (a, b) -> a + b
           );
```

Imagine the data is split:

```text
[1, 2, 3, 4, 5, 6]
```

Conceptually:

```text
Partition 1 → [1,2,3]
Partition 2 → [4,5,6]
```

Thread 1:

```text
0 + 1 = 1
1 + 2 = 3
3 + 3 = 6
```

Thread 2:

```text
0 + 4 = 4
4 + 5 = 9
9 + 6 = 15
```

Now we have:

```text
partial result 1 = 6
partial result 2 = 15
```

The **combiner** combines them:

```text
6 + 15 = 21
```

So:

```text
                 [1 2 3 4 5 6]
                       ↓
                split partitions
                 ↙           ↘
              [1 2 3]      [4 5 6]
                 ↓             ↓
                 6             15
                  \           /
                   \         /
                    combine
                       ↓
                      21
```

This is the key reason the third argument exists.

---

# 9. Why can't the accumulator alone be enough?

In sequential processing:

```text
accumulator
     ↓
1 → 2 → 3 → 4 → 5
```

One running result is enough.

In parallel:

```text
        input
       /     \
  partition  partition
      ↓          ↓
   result A    result B
       \        /
        combine
```

Now you need a function that knows how to combine:

```text
result A + result B
```

That's the **combiner**.

---

# 10. The identity rule ⭐

For a valid reduction, identity must behave neutrally.

For addition:

```java
0 + x = x
```

Good.

For multiplication:

```java
1 * x = x
```

Good.

Bad example:

```java
reduce(10, (a, b) -> a + b)
```

Why?

For:

```text
[1,2,3]
```

sequentially:

```text
10 + 1 + 2 + 3 = 16
```

But with parallel partitions, each partition can start with `10`.

You could conceptually get:

```text
partition A → 10 + 1 + 2 = 13
partition B → 10 + 3 = 13

combine → 26
```

which is **wrong**.

So the identity must genuinely be an identity for the operation.

---

# 11. Associativity is VERY important

The reduction operation should generally be **associative**.

For addition:

```text
(a + b) + c
=
a + (b + c)
```

Example:

```text
(1 + 2) + 3 = 6
1 + (2 + 3) = 6
```

So addition works nicely in parallel.

Multiplication too:

```text
(a × b) × c
=
a × (b × c)
```

But subtraction:

```text
(10 - 5) - 2 = 3
```

while:

```text
10 - (5 - 2) = 7
```

Not associative.

Therefore reduction with subtraction can produce unexpected results in parallel.

### Interview-level rule:

> **For reliable parallel reduction, the reduction function should be associative and the identity should be neutral.**

---

# 12. `reduce()` vs `collect()` ⭐⭐⭐

This is another very common interview question.

### `reduce()`

Use when you are conceptually **combining values into one value**.

Examples:

```text
numbers → sum
numbers → product
numbers → maximum
numbers → minimum
```

### `collect()`

Use when you're **accumulating elements into a mutable result container**.

Examples:

```text
Stream<Employee>
      ↓
List<Employee>

Stream<Employee>
      ↓
Map<Department, List<Employee>>
```

Think:

```text
reduce()
many values → ONE value

collect()
many values → RESULT CONTAINER
```

We'll go much deeper into `collect()` next.

---

# 13. `reduce()` vs `map()`

Don't confuse them.

```java
numbers.stream()
       .map(n -> n * 2)
```

Input:

```text
[1,2,3]
```

Output:

```text
[2,4,6]
```

Still multiple elements.

But:

```java
numbers.stream()
       .reduce(0, Integer::sum)
```

Input:

```text
[1,2,3]
```

Output:

```text
6
```

So:

```text
map    → one-to-one transformation
reduce → many-to-one combination
```

---

# 14. Complexity

For a simple reduction:

```java
numbers.stream()
       .reduce(0, Integer::sum);
```

Sequential:

```text
Time: O(n)
Space: O(1)
```

Parallel:

```text
Work: O(n)
```

but wall-clock performance may improve depending on:

- dataset size
- cost of operation
- splitting overhead
- number of CPU cores
- source characteristics

Don't assume parallel is automatically faster.

---

# 15. Interview questions

### Q1. What is `reduce()`?

> A terminal operation that combines stream elements into a single result.

### Q2. Why does `reduce()` sometimes return `Optional`?

Because without an identity, an empty stream has no result.

### Q3. Why is the combiner needed?

> To combine partial results produced by different partitions during parallel reduction.

### Q4. What properties should a reduction operation have?

For reliable parallel reduction:

> **Associative operation + proper identity.**

### Q5. Why is `10` a bad identity for addition?

Because:

```text
10 + x ≠ x
```

It isn't neutral and can produce incorrect parallel results.

### Q6. `reduce()` vs `collect()`?

> `reduce()` combines values into a single result; `collect()` accumulates elements into a mutable result container.

---

# 🔥 EPAM mental model

If they show you:

```java
numbers.parallelStream()
       .reduce(0, Integer::sum, Integer::sum);
```

immediately think:

```text
identity   → 0
accumulator → add element to partial result
combiner    → add partial results
```

And:

```text
parallel
   ↓
partition
   ↓
local reductions
   ↓
combiner
   ↓
final result
```

That's the part I would **definitely know cold** for your interview.

### Next → `collect()` ⭐⭐⭐⭐⭐

This is the other major Stream API operation, and we'll connect it directly to **`toList()`, `toSet()`, `toMap()`, `groupingBy()` and parallel collection**.