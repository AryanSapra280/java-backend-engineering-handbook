# `limit()` and `skip()`

These look simple, but they become interesting when you combine them with **ordering, stateful operations, and parallel streams**.

---

# 1. `limit()` — Take the First N Elements

## Problem

Suppose:

```java
List<Integer> numbers =
        List.of(10, 20, 30, 40, 50);
```

We only want the first 3:

```java
List<Integer> result =
        numbers.stream()
               .limit(3)
               .toList();
```

Result:

```text
[10, 20, 30]
```

So:

> `limit(n)` allows at most `n` elements to pass downstream.

---

# 2. Why `limit()` Exists

It's especially useful when you don't need the entire dataset.

For example:

```java
employees.stream()
         .limit(10)
         .toList();
```

means:

> Give me only the first 10 employees.

It is also extremely useful with **infinite streams**:

```java
Stream.iterate(1, n -> n + 1)
      .limit(10)
      .forEach(System.out::println);
```

Without `limit()`, the stream is infinite.

---

# 3. Internal Working

Suppose:

```java
1 2 3 4 5 6 7 8
```

and:

```java
.limit(3)
```

Conceptually:

```text
1 → emit
2 → emit
3 → emit

4 → stop
5 → stop
...
```

Because `limit()` is a **short-circuiting intermediate operation**.

It doesn't need to consume the entire stream in a sequential pipeline.

---

# 4. `limit()` Is Stateful or Stateless?

This one requires a little nuance.

For interview purposes, think of `limit()` as a **stateful short-circuiting operation** because it needs to track how many elements have passed:

```text
count = 0

element → count 1 → pass
element → count 2 → pass
element → count 3 → pass

stop
```

But unlike `sorted()` or `distinct()`, it does **not need to retain all previous elements**.

So:

```text
sorted()
    stateful + buffers many elements

distinct()
    stateful + remembers seen elements

limit()
    stateful/short-circuiting + only tracks count
```

That's an important distinction.

---

# 5. Complexity

For a sequential stream:

```java
numbers.stream()
       .limit(10)
       .toList();
```

If the first 10 elements are readily available:

```text
Time ≈ O(10)
```

more generally:

```text
O(min(n, limit))
```

for a straightforward source.

Additional state:

```text
O(1)
```

apart from the output collection.

---

# 6. `skip()` — Ignore the First N Elements

Now the opposite.

```java
List<Integer> result =
        numbers.stream()
               .skip(2)
               .toList();
```

Given:

```text
[10, 20, 30, 40, 50]
```

Result:

```text
[30, 40, 50]
```

Mental model:

```text
10 → discard
20 → discard

30 → emit
40 → emit
50 → emit
```

---

# 7. Why `skip()` Exists

The classic use case is **pagination**.

Suppose:

```text
page = 3
pageSize = 20
```

The conceptual calculation is:

```text
offset = (page - 1) × pageSize
       = 40
```

Then:

```java
stream.skip(40)
      .limit(20)
```

means:

```text
skip first 40
take next 20
```

However, in a real backend application, you generally **shouldn't load the entire database table into Java and then use Stream `skip()`**.

Instead, use database pagination:

```sql
OFFSET 40
LIMIT 20
```

or preferably keyset/cursor pagination for large datasets.

That's an important production distinction.

---

# 8. `skip()` + `limit()` = Slice

This is very common:

```java
List<Integer> page =
        numbers.stream()
               .skip(20)
               .limit(10)
               .toList();
```

Conceptually:

```text
Input:
0 1 2 3 ... 19 20 21 ... 29 30 ...

skip(20)
             ↓
20 21 22 ... 29 30 ...

limit(10)
             ↓
20 21 22 ... 29
```

So:

```text
skip(offset)
limit(pageSize)
```

is the basic Stream equivalent of taking a page/slice.

---

# 9. Operation Ordering Matters

Consider:

```java
numbers.stream()
       .filter(n -> n % 2 == 0)
       .limit(3)
```

This means:

> Find the first 3 even numbers.

For:

```text
1 2 3 4 5 6 7 8
```

Result:

```text
2 4 6
```

But:

```java
numbers.stream()
       .limit(3)
       .filter(n -> n % 2 == 0)
```

means:

> Look only at the first 3 numbers, then keep the even ones.

Result:

```text
2
```

So:

```text
filter → limit
```

and:

```text
limit → filter
```

are **not equivalent**.

---

# 10. `skip()` + `filter()`

Same idea.

```java
numbers.stream()
       .filter(n -> n % 2 == 0)
       .skip(2)
```

means:

> Skip the first two even numbers.

Whereas:

```java
numbers.stream()
       .skip(2)
       .filter(n -> n % 2 == 0)
```

means:

> Skip the first two elements, then find evens.

Very different.

---

# ⭐ 11. Parallel Streams — `limit()`

This is where things get interesting.

Suppose:

```java
numbers.parallelStream()
       .limit(10)
       .toList();
```

Imagine the stream is split:

```text
                Input
                  ↓
          split into chunks
          /       |       \
        T1        T2       T3
```

If this is an **ordered stream**, Java needs to determine the **first 10 elements according to encounter order**.

It cannot simply say:

```text
T1 → give me 10
T2 → give me 10
T3 → give me 10
```

because those aren't necessarily the first 10 globally.

It needs coordination.

---

# 12. Why Ordered Parallel `limit()` Can Be Expensive

Suppose encounter order is:

```text
1 2 3 4 5 6 7 8 9 10 ...
```

but partitions are:

```text
T1 → 1 2 3 4
T2 → 5 6 7 8
T3 → 9 10 11 12
```

To produce:

```text
1..10
```

the framework needs to respect partition ordering.

Therefore:

> `limit()` can become significantly more expensive on an ordered parallel stream than on a sequential stream.

Especially when the underlying source is large or poorly splittable.

---

# 13. What About Unordered Parallel Streams?

Suppose:

```java
numbers.parallelStream()
       .unordered()
       .limit(10)
       .toList();
```

Now you are telling the Stream framework:

> I don't care about encounter order.

That gives the implementation more freedom.

It can potentially take any 10 elements from the parallel processing.

For example:

```text
Thread 1 → some elements
Thread 2 → some elements
Thread 3 → some elements

Take whichever 10 become available
```

This can make `limit()` substantially easier to parallelize.

But the result is no longer guaranteed to represent the first 10 elements of the original encounter order.

---

# 14. Parallel `skip()`

Now consider:

```java
numbers.parallelStream()
       .skip(1000)
       .toList();
```

For an **ordered** stream:

> Java needs to identify which elements are the first 1000 according to encounter order and exclude them.

That requires coordination across partitions.

So ordered parallel `skip()` can also be expensive.

With:

```java
.unordered()
.skip(1000)
```

the implementation has more freedom because encounter order isn't required.

---

# 15. Very Important Interview Point

If interviewer asks:

> Why can `limit()` and `skip()` be expensive on parallel streams?

Strong answer:

> On an ordered parallel stream, `limit()` and `skip()` must respect encounter order. Since data is processed in different partitions, the framework may need coordination between partitions to determine which elements belong to the first or skipped portion. This can reduce the benefit of parallelism.

That's a strong senior-level answer.

---

# 16. `limit()` + `sorted()`

Consider:

```java
numbers.stream()
       .sorted()
       .limit(10)
```

This means:

> Sort everything, then take the first 10.

You might think:

```text
limit(10)
```

means only 10 elements need sorting.

**No.**

Because:

```text
sorted()
   ↓
needs global ordering
   ↓
limit()
```

The sorting operation generally has to consider the entire input.

---

# 17. `limit()` Before `sorted()`

Now:

```java
numbers.stream()
       .limit(10)
       .sorted()
```

means:

> Take the first 10 elements, then sort those 10.

Completely different result.

Example:

```text
Input:
5 4 3 2 1 100 200
```

### sorted → limit

```text
sorted:
1 2 3 4 5 100 200

limit(3):
1 2 3
```

### limit → sorted

```text
limit(3):
5 4 3

sorted:
3 4 5
```

So operation ordering is critical.

---

# 18. Production Pagination

You might see:

```java
repository.findAll()
           .stream()
           .skip(offset)
           .limit(size)
```

For a tiny in-memory collection, that's fine.

For:

```text
10 million database rows
```

this is bad architecture.

You're doing:

```text
Database
   ↓
load huge dataset
   ↓
Java heap
   ↓
skip millions
   ↓
take 20
```

Instead:

```text
Database
   ↓
query only required rows
   ↓
20 records
   ↓
Java
```

For Spring Data JPA, use:

```java
Pageable pageable =
        PageRequest.of(page, size);

repository.findAll(pageable);
```

And for very large datasets, consider **keyset/cursor pagination** rather than large offsets.

This connects directly to the pagination topic you were asked about in your interviews.

---

# 19. `limit()` With Infinite Streams

This is one of the best examples of why `limit()` exists.

```java
List<Integer> numbers =
        Stream.iterate(1, n -> n + 1)
              .limit(10)
              .toList();
```

Without:

```java
.limit(10)
```

the stream never terminates.

So:

```text
iterate()
    ↓
potentially infinite
    ↓
limit(10)
    ↓
finite
    ↓
terminal operation
```

---

# 20. `skip()` With Infinite Streams

This also works:

```java
Stream.iterate(1, n -> n + 1)
      .skip(100)
      .limit(10)
      .forEach(System.out::println);
```

Result:

```text
101
102
...
110
```

Notice:

```text
skip(100)
+
limit(10)
```

allows us to select a finite window from an infinite sequence.

---

# 21. Complexity

### `limit(k)`

For sequential processing:

```text
Time ≈ O(min(n, k))
Additional state ≈ O(1)
```

excluding the output storage.

### `skip(k)`

It generally has to traverse/consume the skipped elements:

```text
Time ≈ O(min(n, k) + remaining elements consumed)
```

For a terminal operation like `toList()`, you're ultimately processing the elements after the skipped region too.

The important point is:

> `skip(1_000_000)` does not magically jump to element 1,000,001 for an arbitrary Stream source.

A Stream isn't necessarily an indexed data structure.

---

# 22. Interview Questions

### Q1. Is `limit()` intermediate or terminal?

Intermediate.

It is also **short-circuiting**.

### Q2. Is `skip()` short-circuiting?

No. `skip()` itself doesn't limit the eventual number of elements.

### Q3. What is the difference?

```text
limit(n)
    maximum n elements are allowed through

skip(n)
    first n elements are discarded
```

### Q4. Why is `limit()` useful with infinite streams?

It makes an otherwise infinite pipeline finite.

### Q5. Why can ordered parallel `limit()` be expensive?

Because encounter order must be respected across parallel partitions.

### Q6. Why can `unordered()` help?

It removes the requirement to preserve encounter order, giving the implementation more freedom to select elements in parallel.

### Q7. Which is correct for "first 10 sorted elements"?

```java
.sorted()
.limit(10)
```

Not:

```java
.limit(10)
.sorted()
```

because the latter means **sort the original first 10**, not get the first 10 after global sorting.

---

# Lock This In

```text
limit(n)
─────────
Take at most n elements.

Short-circuiting:
YES

State:
Tracks count, but doesn't need to retain
all previous elements.

Parallel:
Ordered → coordination may be expensive
Unordered → more freedom


skip(n)
────────
Discard first n elements.

Short-circuiting:
NO

Parallel:
Ordered → must respect encounter order
Unordered → more freedom


Common:
skip(offset)
limit(pageSize)

But:
For DB pagination → use database pagination,
not loading everything into Java and then skip().
```

### The key interview distinction

```text
filter → select
map    → transform
flatMap → transform + flatten
distinct → remove duplicates
sorted → globally order
limit → take first N
skip → discard first N
```

**Next: `peek()`** — short topic, but there is a very common interview trap: people use `peek()` for business logic because it "looks like `forEach`." We'll cover why that's dangerous, laziness, debugging, and what happens in parallel streams.