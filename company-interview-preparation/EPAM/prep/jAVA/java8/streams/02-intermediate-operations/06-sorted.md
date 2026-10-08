# `sorted()` — Sorting a Stream

This is the next **stateful** Stream operation, and it's particularly important because sorting becomes interesting with **parallel streams**.

---

## 1. Problem — Why `sorted()`?

Suppose:

```java id="1d9w6n"
List<Integer> numbers =
        List.of(5, 2, 8, 1, 3);
```

We want:

```text
[1, 2, 3, 5, 8]
```

Using Stream:

```java id="f7y7xj"
List<Integer> result =
        numbers.stream()
               .sorted()
               .toList();
```

`sorted()` sorts elements according to their **natural ordering**.

For `Integer`:

```text
1 < 2 < 3 < 5 < 8
```

---

# 2. Why Is `sorted()` Stateful?

This is the important part.

Compare:

```java id="e2p0j9"
.filter(n -> n > 5)
```

The filter can process:

```text
1 → immediately know whether to keep it
2 → immediately know
3 → immediately know
```

It doesn't need to know about other elements.

But suppose we receive:

```text id="x3r2a5"
5
2
8
1
3
```

When `sorted()` receives:

```text
5
```

can it immediately emit `5`?

**No.**

Because maybe later it receives:

```text
1
```

So it has to gather/coordinate the elements before it can produce the correctly sorted result.

That's why:

```text id="r5q3eq"
sorted()
```

is a **stateful intermediate operation**.

---

# 3. Internal Mental Model

Conceptually:

```text id="n7x7m2"
Input:
5 2 8 1 3

        ↓

buffer elements

        ↓

[5, 2, 8, 1, 3]

        ↓

sort

        ↓

[1, 2, 3, 5, 8]

        ↓

downstream operation
```

This is fundamentally different from:

```text id="3e9z7s"
filter()
map()
```

which can process elements progressively.

---

# 4. Natural Ordering

When you write:

```java id="f75vch"
numbers.stream()
       .sorted()
```

Java uses the elements' **natural ordering**.

For objects, this generally means they implement:

```java id="s2u5i9"
Comparable<T>
```

Example:

```java id="4x8d3v"
class Employee implements Comparable<Employee> {

    private int salary;

    @Override
    public int compareTo(Employee other) {
        return Integer.compare(this.salary, other.salary);
    }
}
```

Now:

```java id="6a0y0c"
employees.stream()
         .sorted()
         .toList();
```

sorts according to `compareTo()`.

---

# 5. `Comparable` vs `Comparator`

This distinction is **very important**.

### Comparable

The class defines its natural ordering:

```java id="x2q1ki"
class Employee implements Comparable<Employee>
```

and:

```java id="r9d1z6"
compareTo()
```

defines the ordering.

Think:

```text id="8fdr0x"
Employee
   ↓
"How should employees naturally be ordered?"
   ↓
compareTo()
```

---

### Comparator

The caller defines the ordering:

```java id="7a0qzi"
employees.stream()
         .sorted(Comparator.comparing(Employee::getSalary))
```

You can now choose different orderings without modifying `Employee`.

For example:

```java id="ydn8ap"
.sorted(Comparator.comparing(Employee::getName))
```

or:

```java id="o5x2ab"
.sorted(Comparator.comparing(Employee::getSalary))
```

or descending:

```java id="2skm6c"
.sorted(Comparator.comparing(Employee::getSalary)
                  .reversed())
```

---

# 6. `sorted()` Has Two Forms

### Natural ordering

```java id="t5n8lw"
stream.sorted()
```

Uses `Comparable`.

### Custom ordering

```java id="d3bqz9"
stream.sorted(comparator)
```

Uses the supplied `Comparator`.

For example:

```java id="qwr5r8"
List<String> names =
        List.of("Aryan", "Rahul", "Amit", "Zoya");

List<String> result =
        names.stream()
             .sorted(Comparator.comparing(String::length))
             .toList();
```

This sorts by string length.

---

# 7. Multiple Sorting Conditions

Very common interview coding question.

Suppose:

```java id="p3y0tr"
class Employee {
    String department;
    int salary;
    String name;
}
```

Requirement:

> Sort employees by salary descending, and if salary is equal, sort by name.

```java id="m1jv5r"
employees.stream()
         .sorted(
             Comparator.comparing(Employee::getSalary)
                       .reversed()
                       .thenComparing(Employee::getName)
         )
         .toList();
```

Conceptually:

```text id="cnp0zh"
salary DESC
      ↓
if equal
      ↓
name ASC
```

This is an excellent EPAM-style Stream question.

---

# 8. `sorted()` + `filter()`

Suppose:

```text id="t7h5n6"
1 million employees
```

but only:

```text id="f8d2xk"
10,000 employees
```

satisfy:

```java id="d1w0er"
salary > 100000
```

Prefer:

```java id="cm1s47"
employees.stream()
         .filter(e -> e.getSalary() > 100000)
         .sorted(Comparator.comparing(Employee::getSalary))
         .toList();
```

rather than:

```java id="6t7y1d"
employees.stream()
         .sorted(Comparator.comparing(Employee::getSalary))
         .filter(e -> e.getSalary() > 100000)
         .toList();
```

Why?

Because sorting is expensive.

You want to reduce the dataset **before** sorting when the semantics allow it.

Mental model:

```text id="xv3qwk"
1,000,000 employees
        ↓
filter
        ↓
10,000 employees
        ↓
sort 10,000
```

rather than:

```text id="9e1k3p"
1,000,000 employees
        ↓
sort 1,000,000
        ↓
filter
```

---

# 9. `sorted()` + `limit()`

This is another important interview scenario.

Requirement:

> Find the top 5 highest-paid employees.

You might write:

```java id="l5d6r2"
employees.stream()
         .sorted(
             Comparator.comparing(Employee::getSalary)
                       .reversed()
         )
         .limit(5)
         .toList();
```

This is correct.

But algorithmically, you're potentially sorting the entire dataset just to get 5 elements.

For very large datasets, a **bounded priority queue / top-K algorithm** can be more efficient than a full sort.

So in production:

```text id="l1c8qg"
Need top 5
    ↓
Full sort
    ↓
Potentially O(n log n)

Top-K algorithm
    ↓
Potentially O(n log k)
```

where:

```text id="o4z9bm"
k = 5
```

This is a great **senior-level optimization discussion**.

---

# 10. Complexity

For `n` elements:

Typical comparison-based sorting complexity:

```text id="x3s2yz"
O(n log n)
```

The exact implementation details shouldn't be your main interview answer unless specifically asked.

Memory can be:

```text id="i8f9u2"
O(n)
```

because sorting is stateful and may need to buffer elements.

The important interview distinction is:

```text id="qj4z3c"
filter → O(n), stateless
map    → O(n), stateless

sorted → O(n log n), stateful
```

---

# ⭐ 11. Parallel Stream Behavior

Now the part you specifically asked me to include for every operation.

Consider:

```java id="x5w9s8"
List<Integer> result =
        numbers.parallelStream()
               .sorted()
               .toList();
```

Can each thread simply sort its own chunk and return it?

Not enough.

Imagine:

```text id="l9e0m8"
Thread 1:
[8, 2, 7]

Thread 2:
[1, 9, 3]

Thread 3:
[4, 6, 5]
```

Each thread can locally sort:

```text id="s2g4jx"
T1 → [2, 7, 8]
T2 → [1, 3, 9]
T3 → [4, 5, 6]
```

But the final answer still needs to be:

```text id="1j5g0j"
[1, 2, 3, 4, 5, 6, 7, 8, 9]
```

So parallel sorting requires **coordination/merging of the partitions**.

Conceptually:

```text id="7k1zrq"
                    Input
                      ↓
                split partitions
                /      |       \
               T1      T2       T3
               ↓       ↓        ↓
             sort     sort     sort
               \       |        /
                \      |       /
                 merge/coordinate
                       ↓
                 globally sorted
```

That's why `sorted()` isn't a trivially parallel operation.

---

# 12. Ordered vs Unordered Parallel Stream

Suppose:

```java id="p7n2cy"
numbers.parallelStream()
       .sorted()
       .toList();
```

The stream is ordered, and the result must respect the sorted ordering.

There's significant coordination involved.

If you're dealing with an unordered stream:

```java id="j3h6w0"
numbers.parallelStream()
       .unordered()
       .sorted()
```

you still need a globally sorted result.

So `unordered()` doesn't magically make sorting cheap.

The fundamental requirement:

> **A globally sorted result requires global coordination.**

---

# 13. Does Parallel `sorted()` Always Make It Faster?

**No.**

For:

```text id="k5h7p0"
100 elements
```

parallelization overhead can easily exceed the benefit.

For a very large dataset with sufficiently expensive sorting and enough CPU cores, parallel sorting can potentially help.

So your interview answer should be:

> "`sorted()` can be parallelized, but it is stateful and requires coordination across partitions. Whether a parallel stream is faster depends on dataset size, comparator cost, available CPU, ordering requirements and overhead."

That's a strong answer.

---

# 14. Comparator and Parallel Streams

Be careful with this:

```java id="7s6x6a"
.sorted((a, b) -> {
    someSharedState.modify();
    return ...;
})
```

Comparators should be:

- deterministic
- consistent
- preferably stateless
- free of unsafe shared mutable state

A comparator with shared mutable state can cause incorrect behavior, especially when parallel execution is involved.

---

# 15. `sorted()` and `distinct()` Together

Example:

```java id="axk2vr"
numbers.stream()
       .distinct()
       .sorted()
       .toList();
```

Pipeline:

```text id="9d1t5p"
5 2 5 1 3 2
      ↓
distinct
      ↓
5 2 1 3
      ↓
sorted
      ↓
1 2 3 5
```

Could you reverse them?

```java id="h0jv6z"
numbers.stream()
       .sorted()
       .distinct()
       .toList();
```

Yes, result is also:

```text id="h4c0t1"
1 2 3 5
```

But performance characteristics differ.

You generally don't need to sort duplicate values that will eventually be removed.

For example:

```text id="5ojrzw"
1,000,000 values
↓
100 unique values
```

Doing:

```text id="3i7q0p"
distinct → 100
sorted → 100
```

can be dramatically better than:

```text id="lq6fhy"
sorted → 1,000,000
distinct → 100
```

So:

> **When operations are semantically reorderable, eliminate data before expensive stateful operations.**

---

# 16. Production Example

Imagine you're building a payment reporting API:

```java id="x9f4cu"
List<Payment> payments = ...;
```

Requirement:

> Give me the 10 largest successful payments.

A straightforward implementation:

```java id="z7m3ax"
List<Payment> topPayments =
        payments.stream()
                .filter(p -> p.isSuccessful())
                .sorted(
                    Comparator.comparing(Payment::getAmount)
                              .reversed()
                )
                .limit(10)
                .toList();
```

Pipeline:

```text id="j99g1j"
Payments
   ↓
successful only
   ↓
sort descending by amount
   ↓
top 10
```

This is perfectly reasonable for a moderate dataset.

For millions of records, however, you'd ask:

> Why am I loading/sorting millions of payments in the application?

Maybe the database should do:

```sql
ORDER BY amount DESC
LIMIT 10
```

or use an appropriate index.

**Stream API doesn't replace database query optimization.**

That's a very good production-level observation.

---

# 17. Interview Questions

### Q1. Why is `sorted()` stateful?

Because it cannot determine an element's final position without considering other elements.

### Q2. What's the complexity?

Typically:

```text
O(n log n)
```

for comparison-based sorting.

### Q3. What does `sorted()` without arguments use?

Natural ordering, generally through `Comparable`.

### Q4. How do you sort using a custom condition?

```java
.sorted(Comparator.comparing(Employee::getSalary))
```

### Q5. How do you sort descending?

```java
.sorted(
    Comparator.comparing(Employee::getSalary)
              .reversed()
)
```

### Q6. Multiple sorting conditions?

```java
.sorted(
    Comparator.comparing(Employee::getSalary)
              .thenComparing(Employee::getName)
)
```

### Q7. Does `parallelStream().sorted()` simply sort each thread's partition?

No. Local sorting can happen in parallel, but a globally sorted result requires coordination/merging.

### Q8. Is parallel sorting always faster?

No. Dataset size, CPU availability, comparator cost, ordering, and parallelization/merge overhead all matter.

### Q9. Why filter before sorting?

To reduce the number of elements that need to participate in the expensive stateful sorting operation.

---

# Lock this in

```text id="5a7o4z"
sorted()
────────

Purpose:
    Sort stream elements.

Forms:
    sorted()
        → natural ordering

    sorted(Comparator)
        → custom ordering

Properties:
    Intermediate
    Lazy
    Stateful
    Typically O(n log n)

Requires:
    Comparable OR Comparator

Parallel:
    Can execute in parallel
    BUT requires global coordination
    for globally sorted result

Production:
    Filter before sorting when possible.
    For huge datasets, consider DB ORDER BY,
    indexes, or top-K algorithms.

Mental model:

filter/map:
    element-by-element

sorted:
    need to see/coordinate across elements
    ↓
    sort
    ↓
    emit downstream
```

**Next: `limit()` and `skip()`** — these are deceptively simple, but the **parallel + ordering behavior is very interview-worthy**, especially why `limit()`/`skip()` can become expensive on ordered parallel streams.