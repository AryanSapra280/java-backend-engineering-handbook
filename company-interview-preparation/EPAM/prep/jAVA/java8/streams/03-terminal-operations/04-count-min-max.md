# `count()`, `min()`, `max()` ⭐⭐

These are straightforward, but there are a few interview traps around **empty streams, comparators, and parallel execution**.

---

## 1. `count()`

`count()` tells you how many elements are in the stream.

```java
long count =
    numbers.stream()
           .filter(n -> n > 10)
           .count();
```

If:

```text
[5, 12, 15, 20]
```

then:

```text
count = 3
```

because:

```text
12, 15, 20
```

match.

### Important

Return type is:

```java
long
```

not `int`.

```java
long count = stream.count();
```

---

## 2. `count()` is terminal

```java
numbers.stream()
       .filter(n -> n > 10)
       .count();
```

The `count()` triggers the entire pipeline.

Without it:

```java
numbers.stream()
       .filter(n -> n > 10);
```

nothing is consumed.

---

## 3. Complexity

Generally:

```text
Time: O(n)
Space: O(1)
```

If there are intermediate operations, their cost is included.

Example:

```java
numbers.stream()
       .filter(...)
       .map(...)
       .count();
```

Each relevant element flows through the pipeline.

---

# 4. `min()`

`min()` returns the smallest element according to a comparator.

```java
Optional<Integer> min =
    numbers.stream()
           .min(Integer::compare);
```

Why `Optional<Integer>`?

Because the stream might be empty.

```java
List<Integer> numbers = List.of();

Optional<Integer> min =
    numbers.stream()
           .min(Integer::compare);
```

Result:

```text
Optional.empty
```

So:

```java
Integer value =
    numbers.stream()
           .min(Integer::compare)
           .orElse(0);
```

---

# 5. `max()`

Same idea:

```java
Optional<Integer> max =
    numbers.stream()
           .max(Integer::compare);
```

For:

```text
[10, 50, 20, 5]
```

you get:

```text
Optional[50]
```

---

# 6. Objects + Comparator ⭐

This is where it becomes more practical.

Suppose:

```java
class Employee {
    String name;
    int salary;
}
```

Find lowest salary:

```java
Optional<Employee> lowest =
    employees.stream()
             .min(Comparator.comparingInt(Employee::getSalary));
```

Highest:

```java
Optional<Employee> highest =
    employees.stream()
             .max(Comparator.comparingInt(Employee::getSalary));
```

This is much better than manually maintaining:

```java
Employee highest = null;

for (Employee e : employees) {
    if (highest == null || e.getSalary() > highest.getSalary()) {
        highest = e;
    }
}
```

---

# 7. `min()` / `max()` are NOT sorting

This is important.

You don't need to sort the entire stream:

```java
employees.stream()
         .sorted(Comparator.comparingInt(Employee::getSalary))
         .findFirst();
```

to find the minimum.

That's wasteful.

Use:

```java
employees.stream()
         .min(Comparator.comparingInt(Employee::getSalary));
```

### Complexity

`min()` / `max()`:

```text
O(n)
```

Sorting:

```text
O(n log n)
```

So if you only need the minimum/maximum:

> **Don't sort just to find an extreme value.**

---

# 8. Parallel streams ⭐⭐⭐

`count()`, `min()`, and `max()` can work well with parallel streams because the computation can be performed on partitions.

For example:

```text
[10, 5, 30, 2, 50, 8]
```

could conceptually become:

```text
Thread 1 → [10,5,30] → local min = 5
Thread 2 → [2,50,8]  → local min = 2

                ↓

        combine → min(5,2)
                ↓
                   2
```

Similarly for `max()`:

```text
Thread 1 → local max = 30
Thread 2 → local max = 50

combine → 50
```

This is one reason reduction-style operations can parallelize nicely.

---

# 9. Does `min()` preserve order?

Suppose:

```java
[5, 2, 2, 10]
```

The minimum value is `2`.

There may be multiple objects with the same minimum according to the comparator.

If you're working with objects and care **which equal-minimum object** gets returned, don't assume arbitrary tie behavior without considering stream ordering and comparator semantics.

For normal numeric minimum, this isn't usually an issue.

---

# 10. `count()` with parallel stream

```java
long count =
    numbers.parallelStream()
           .filter(n -> n > 100)
           .count();
```

Conceptually:

```text
Partition 1 → local count = 10
Partition 2 → local count = 7
Partition 3 → local count = 12

                     ↓
                 combine

                     ↓
                    29
```

The framework can calculate partial results and combine them.

Unlike `forEach()`, you don't have to maintain a shared counter.

### Bad

```java
AtomicLong counter = new AtomicLong();

numbers.parallelStream()
       .forEach(n -> counter.incrementAndGet());
```

### Better

```java
long count =
    numbers.parallelStream()
           .count();
```

Let the Stream framework handle the aggregation.

---

# 11. Quick comparison

| Operation | Returns | Short-circuit? | Typical complexity |
|---|---|---:|---:|
| `count()` | `long` | No | O(n) |
| `min()` | `Optional<T>` | No* | O(n) |
| `max()` | `Optional<T>` | No* | O(n) |

`min()`/`max()` generally need to inspect the elements to know the true minimum/maximum, so they cannot stop early merely because they've found a "good enough" value.

---

## 🔥 Interview points

### Why does `min()` return `Optional<T>`?

Because the stream can be empty.

### Why is `count()` `long`?

Because the number of elements can exceed the range of `int`.

### Which is better for finding maximum?

```java
max(...)
```

rather than:

```java
sorted(...)
.findFirst()
```

because `max()` is O(n), while sorting is typically O(n log n).

### Can these work with parallel streams?

Yes. They can use partition-local calculations and combine the results.

---

# Next: `reduce()` ⭐⭐⭐⭐⭐

This is the **first really important terminal operation** in the remaining Stream section.

We'll cover:

```text
reduce()
 ↓
identity
 ↓
accumulator
 ↓
combiner
 ↓
why combiner matters in parallel streams
 ↓
sum/product/max/custom reduction
 ↓
reduce vs collect
```

The **identity + accumulator + combiner** part is especially important for your EPAM interview.