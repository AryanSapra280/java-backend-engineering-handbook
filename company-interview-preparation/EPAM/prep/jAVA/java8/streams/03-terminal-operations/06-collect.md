# `collect()` ⭐⭐⭐⭐⭐

Now we reach the **second major Stream operation after `reduce()`**.

The key difference:

> **`reduce()` combines elements into a value. `collect()` accumulates elements into a result container.**

---

## 1. Basic `collect()`

Suppose:

```java
List<Integer> numbers = List.of(1, 2, 3, 4, 5);
```

We want a `List`:

```java
List<Integer> result =
    numbers.stream()
           .collect(Collectors.toList());
```

Result:

```text
[1, 2, 3, 4, 5]
```

Today, you can also simply write:

```java
List<Integer> result =
    numbers.stream()
           .toList();
```

We'll discuss the difference later.

---

# 2. What is `collect()` actually doing?

Think:

```text
Stream
  ↓
elements
  ↓
accumulate
  ↓
mutable result container
```

For example:

```text
1 → List
2 → List
3 → List
4 → List
5 → List
```

Final:

```text
[1,2,3,4,5]
```

Unlike `reduce()`, the result is typically a **mutable container being accumulated into**.

---

# 3. The `Collector`

The important thing is:

```java
collect(Collector)
```

For example:

```java
Collectors.toList()
```

`Collectors` provides many predefined collectors:

```java
Collectors.toList()
Collectors.toSet()
Collectors.toMap()
Collectors.groupingBy()
Collectors.partitioningBy()
Collectors.joining()
Collectors.counting()
Collectors.mapping()
```

So:

```java
stream.collect(...)
```

is basically saying:

> "Tell me how you want to accumulate this stream."

---

# 4. `collect()` vs `reduce()` ⭐⭐⭐⭐⭐

This distinction is extremely important.

### `reduce()`

```java
int sum =
    numbers.stream()
           .reduce(0, Integer::sum);
```

Conceptually:

```text
1 + 2 + 3 + 4 + 5
          ↓
         15
```

### `collect()`

```java
List<Integer> result =
    numbers.stream()
           .collect(Collectors.toList());
```

Conceptually:

```text
1 ─┐
2 ─┤
3 ─┼──→ List
4 ─┤
5 ─┘
```

So:

```text
reduce()
→ many values → one value

collect()
→ many values → result container
```

---

# 5. Collect into a Set

```java
Set<Integer> result =
    numbers.stream()
           .collect(Collectors.toSet());
```

Duplicates are removed according to Set semantics.

```java
List<Integer> numbers =
    List.of(1, 2, 2, 3, 3, 3);

Set<Integer> result =
    numbers.stream()
           .collect(Collectors.toSet());
```

Result conceptually:

```text
[1, 2, 3]
```

---

# 6. Filtering + collecting

This is one of the most common real-world patterns:

```java
List<String> activeUsers =
    users.stream()
         .filter(User::isActive)
         .map(User::getName)
         .collect(Collectors.toList());
```

Pipeline:

```text
users
 ↓
filter active
 ↓
map → name
 ↓
collect → List<String>
```

This pattern appears constantly in production code.

---

# 7. How does a Collector work internally? ⭐⭐⭐

A `Collector` conceptually defines four things:

```text
Supplier
Accumulator
Combiner
Finisher
```

For example, imagine collecting into a `List`.

### Supplier

Creates the result container:

```java
() -> new ArrayList<>()
```

### Accumulator

Adds an element:

```java
(list, element) -> list.add(element)
```

### Combiner

Combines partial containers:

```java
(list1, list2) -> {
    list1.addAll(list2);
    return list1;
}
```

### Finisher

Converts the intermediate container into the final result if necessary.

For a basic `toList()` collector, the finisher can effectively be identity-like.

Mental model:

```text
Supplier
   ↓
create container

Accumulator
   ↓
add elements

Parallel?
   ↓
multiple containers
   ↓
Combiner
   ↓
merge containers

Finisher
   ↓
final result
```

---

# 8. Parallel `collect()` ⭐⭐⭐⭐⭐

This is important because you specifically wanted parallel behavior for every operation.

Suppose:

```java
List<Integer> result =
    numbers.parallelStream()
           .collect(Collectors.toList());
```

Conceptually:

```text
                 [1 2 3 4 5 6]
                       ↓
                  split source
                  /          \
             [1 2 3]       [4 5 6]
                ↓              ↓
             List A          List B
                ↓              ↓
                 \            /
                  \          /
                   combine
                      ↓
                final List
```

The framework can create **separate accumulation containers** for different partitions.

It doesn't mean every thread is simultaneously doing:

```java
sameList.add(...)
```

which would be dangerous.

Instead, partial results can be accumulated independently and then combined.

---

# 9. Why is this better than manually using `ArrayList`?

Bad:

```java
List<Integer> result = new ArrayList<>();

numbers.parallelStream()
       .forEach(result::add);
```

You are manually sharing mutable state between threads.

Potential race condition.

Better:

```java
List<Integer> result =
    numbers.parallelStream()
           .collect(Collectors.toList());
```

The collector knows how to manage the accumulation process.

This is one of the biggest reasons to prefer **collectors over external mutable state**.

---

# 10. `Collector` characteristics

At senior level, know these terms:

```text
CONCURRENT
UNORDERED
IDENTITY_FINISH
```

You don't need to memorize the entire Collector implementation, but understand what they mean.

### `CONCURRENT`

The collector can support concurrent accumulation into the same result container under appropriate conditions.

### `UNORDERED`

The collector doesn't care about encounter order.

### `IDENTITY_FINISH`

The accumulation type and final result type are effectively the same, so no additional finishing transformation is required.

You'll encounter these concepts when discussing advanced parallel collectors.

---

# 11. `collect()` is not necessarily only for Lists

This is where it becomes powerful.

You can produce:

```text
List
Set
Map
String
Integer/Long counts
grouped Map
partitioned Map
custom objects
```

For example:

### String

```java
String result =
    names.stream()
         .collect(Collectors.joining(", "));
```

Result:

```text
Alice, Bob, Charlie
```

### Map

```java
Map<String, Integer> result =
    employees.stream()
             .collect(
                 Collectors.toMap(
                     Employee::getName,
                     Employee::getSalary
                 )
             );
```

### Grouping

```java
Map<String, List<Employee>> result =
    employees.stream()
             .collect(
                 Collectors.groupingBy(Employee::getDepartment)
             );
```

We'll go deeply into each of these next.

---

# 12. `collect()` vs `forEach()` ⭐⭐⭐

Bad approach:

```java
List<String> names = new ArrayList<>();

users.stream()
     .forEach(user -> names.add(user.getName()));
```

You are using `forEach()` to mutate an external collection.

Better:

```java
List<String> names =
    users.stream()
         .map(User::getName)
         .collect(Collectors.toList());
```

Or:

```java
List<String> names =
    users.stream()
         .map(User::getName)
         .toList();
```

The second approach expresses the actual intent:

> "Transform these elements into a List."

---

# 13. `collect()` vs `toList()`

You'll frequently see both:

```java
stream.collect(Collectors.toList())
```

and:

```java
stream.toList()
```

They aren't exactly identical.

### `Collectors.toList()`

```java
List<T> result =
    stream.collect(Collectors.toList());
```

The exact mutability/type guarantees are deliberately weaker.

### `Stream.toList()`

```java
List<T> result =
    stream.toList();
```

The resulting list is **unmodifiable**.

Therefore:

```java
result.add(...)
```

should not be used with the result of `Stream.toList()`.

This distinction is worth remembering for interviews.

---

# 14. Practical example — payment processing

Imagine:

```java
List<Payment> payments;
```

Get successful payment IDs:

```java
List<String> successfulPaymentIds =
    payments.stream()
            .filter(Payment::isSuccessful)
            .map(Payment::getId)
            .toList();
```

This is cleaner than:

```java
List<String> ids = new ArrayList<>();

payments.forEach(payment -> {
    if (payment.isSuccessful()) {
        ids.add(payment.getId());
    }
});
```

The Stream version expresses the **data transformation** directly.

---

# 15. Complexity

For:

```java
stream.collect(Collectors.toList())
```

with `n` elements:

```text
Time: O(n)
```

assuming normal O(1) accumulation.

Space:

```text
O(n)
```

because you're creating a result containing the elements.

Compare that with:

```java
stream.reduce(...)
```

for a simple numeric sum:

```text
Space: O(1)
```

This is another conceptual difference.

---

# 16. Parallel considerations

Parallel collection isn't automatically faster.

There is overhead from:

```text
splitting
   ↓
multiple accumulation containers
   ↓
combining
```

For:

```java
[1,2,3,4,5,6]
```

parallelism is usually pointless.

For a **huge dataset with sufficiently expensive processing**, parallel collection can potentially help.

Also, order matters.

For ordered streams, collectors may need to preserve encounter-order semantics depending on the collector.

If order doesn't matter and the collector supports unordered processing, more parallel freedom may be available.

---

# 🔥 Interview questions

### Q1. What is `collect()`?

> A terminal operation that uses a `Collector` to accumulate stream elements into a result.

### Q2. `reduce()` vs `collect()`?

> `reduce()` combines elements into a single value; `collect()` accumulates elements into a result container or other final representation.

### Q3. Why shouldn't I use `forEach()` to populate an external `ArrayList`?

Because it introduces external mutable state and becomes unsafe with parallel streams.

### Q4. What are the main components of a Collector?

```text
Supplier
Accumulator
Combiner
Finisher
```

### Q5. What happens during parallel collection?

> Each partition can accumulate into its own intermediate result, and the framework combines those partial results.

### Q6. `stream.toList()` vs `Collectors.toList()`?

> `Stream.toList()` returns an unmodifiable List, while `Collectors.toList()` does not provide the same unmodifiable guarantee.

---

# 🧠 Mental model to keep

```text
reduce()
    ↓
"Combine values"

collect()
    ↓
"Build a result"

collect(toList())
    ↓
List

collect(toSet())
    ↓
Set

collect(toMap())
    ↓
Map

collect(groupingBy())
    ↓
Map<K, List<V>>
```

And **next is `toList()` / `toSet()` briefly, then `toMap()`**.

`toMap()` is especially important because of the **duplicate-key problem and merge function**, which is a very common interview trap.