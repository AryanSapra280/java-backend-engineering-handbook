# `peek()` — Stream API

This one is **short but very interview-relevant** because `peek()` is commonly misused.

---

## 1. What problem does `peek()` solve?

Sometimes you want to **observe elements while they are flowing through a Stream pipeline** without changing them.

For example, while debugging:

```java
List<Integer> result =
    numbers.stream()
           .filter(n -> n > 10)
           .peek(n -> System.out.println("After filter: " + n))
           .map(n -> n * 2)
           .peek(n -> System.out.println("After map: " + n))
           .toList();
```

If:

```java
numbers = [5, 12, 15]
```

You can mentally see:

```text
5  → filter ❌

12 → filter ✅ → peek(12) → map(24) → peek(24)
15 → filter ✅ → peek(15) → map(30) → peek(30)
```

Result:

```text
[24, 30]
```

---

# 2. What exactly is `peek()`?

Signature:

```java
Stream<T> peek(Consumer<? super T> action)
```

So:

```text
peek()
 ↓
Consumer<T>
 ↓
T → void
```

It is an **intermediate operation**.

Therefore:

```java
stream.peek(...)
```

does **not execute immediately**.

Example:

```java
numbers.stream()
       .peek(n -> System.out.println(n));
```

Nothing necessarily gets printed.

Why?

Because there is **no terminal operation**.

Add:

```java
numbers.stream()
       .peek(n -> System.out.println(n))
       .toList();
```

Now the pipeline executes.

---

# 3. `peek()` does NOT transform elements

Compare:

### `map()`

```java
numbers.stream()
       .map(n -> n * 2)
```

Changes:

```text
1 → 2
2 → 4
3 → 6
```

### `peek()`

```java
numbers.stream()
       .peek(n -> System.out.println(n))
```

Elements remain:

```text
1 → 1
2 → 2
3 → 3
```

`peek()` only **observes** them.

---

# 4. `peek()` vs `forEach()`

Very important interview question.

| | `peek()` | `forEach()` |
|---|---|---|
| Type | Intermediate | Terminal |
| Lazy? | Yes | No |
| Returns | Stream | void |
| Purpose | Observe pipeline elements | Final action |
| Can continue pipeline? | Yes | No |

Example:

```java
numbers.stream()
       .peek(System.out::println)
       .filter(n -> n > 10)
       .toList();
```

Valid.

But:

```java
numbers.stream()
       .forEach(System.out::println)
       .filter(...);
```

Invalid because `forEach()` terminates the stream.

---

# 5. The biggest interview trap 🚨

Don't use `peek()` for important business logic.

Bad:

```java
orders.stream()
      .peek(order -> database.save(order))
      .filter(Order::isValid)
      .toList();
```

This is a bad design.

Why?

Because `peek()` is intended primarily for **observation/debugging**, not as a business-processing mechanism.

Better:

```java
orders.stream()
      .filter(Order::isValid)
      .map(order -> {
          database.save(order);
          return order;
      })
      .toList();
```

Even better in real production code: keep database writes outside a stream pipeline when the operation is inherently side-effectful.

---

# 6. Why is using side effects inside `peek()` dangerous?

Consider:

```java
List<Integer> result =
    numbers.stream()
           .peek(n -> counter++)
           .toList();
```

You are making the stream pipeline responsible for modifying external state.

Problems:

### 1. Laziness

Without terminal operation:

```java
numbers.stream()
       .peek(n -> counter++);
```

`counter` doesn't change.

### 2. Parallel execution

With:

```java
numbers.parallelStream()
       .peek(n -> counter++)
       .toList();
```

multiple threads can execute the consumer.

Now:

```java
counter++
```

is not atomic.

You can get a race condition.

---

# 7. `peek()` with parallel streams ⭐⭐⭐

This is particularly important for your EPAM preparation.

Suppose:

```java
numbers.parallelStream()
       .peek(n -> System.out.println(
           Thread.currentThread().getName() + " : " + n
       ))
       .toList();
```

The `peek()` action may execute on **different worker threads**.

Conceptually:

```text
main thread
   ↓
partition A

ForkJoinPool worker
   ↓
partition B

ForkJoinPool worker
   ↓
partition C
```

So the output could be:

```text
ForkJoinPool.commonPool-worker-1 : 6
main : 3
ForkJoinPool.commonPool-worker-3 : 9
ForkJoinPool.commonPool-worker-1 : 2
...
```

Don't assume:

```text
1
2
3
4
5
6
```

---

# 8. What about encounter order?

Suppose:

```java
List<Integer> numbers = List.of(1, 2, 3, 4, 5);

numbers.parallelStream()
       .peek(System.out::println)
       .toList();
```

The **result of `toList()` can preserve encounter order**, because the stream is ordered.

But the execution of `peek()` itself can happen concurrently, so **the printed/logged order is not something you should rely on**.

This distinction is very important:

```text
Result ordering
        ≠
Execution ordering
```

For example:

```java
parallelStream()
    .map(...)
    .toList();
```

can produce an ordered result.

But:

```java
parallelStream()
    .peek(...)
```

doesn't mean the `peek()` actions happen sequentially.

---

# 9. `forEachOrdered()`?

If you really need encounter-order processing:

```java
numbers.parallelStream()
       .forEachOrdered(System.out::println);
```

This preserves encounter order.

But there's a tradeoff:

```text
More ordering constraints
        ↓
Less freedom for parallel execution
        ↓
Potentially less parallel benefit
```

So don't blindly use parallel streams + ordering.

---

# 10. Complexity

For:

```java
numbers.stream()
       .peek(...)
       .toList();
```

If there are `n` elements:

```text
Time: O(n)
```

assuming the `peek` action itself is O(1).

Additional stream state:

```text
O(1)
```

But if your peek action does:

```java
peek(n -> database.save(n))
```

then the database operation dominates the cost.

So always remember:

> **Stream operation complexity + cost of the lambda/action.**

---

# 11. Practical debugging example

This is where `peek()` is genuinely useful.

```java
List<String> result =
    users.stream()
         .filter(User::isActive)
         .peek(user -> log.info("Active user: {}", user.getId()))
         .map(User::getEmail)
         .peek(email -> log.info("Email: {}", email))
         .filter(email -> email.endsWith("@company.com"))
         .toList();
```

You can understand what's happening at each stage:

```text
users
  ↓
filter active
  ↓
peek
  ↓
map → email
  ↓
peek
  ↓
filter company email
  ↓
result
```

This is a good debugging use.

---

# 12. Important subtle point: `peek()` isn't a guaranteed business hook

Don't say in an interview:

> "`peek()` always executes for every element."

That's too strong.

The correct mental model is:

> `peek()` is a lazy intermediate operation whose action runs as elements are consumed by the terminal operation, and it should not be relied upon for essential side effects.

That's the senior-level answer.

---

# 13. Interview questions you should be ready for

### Q1. Is `peek()` intermediate or terminal?

**Intermediate and lazy.**

---

### Q2. Why doesn't this print anything?

```java
numbers.stream()
       .peek(System.out::println);
```

Because there is no terminal operation.

---

### Q3. Difference between `peek()` and `forEach()`?

```text
peek   → intermediate → returns Stream
forEach → terminal → returns void
```

---

### Q4. Difference between `peek()` and `map()`?

```text
peek → observes element
map  → transforms element
```

---

### Q5. Can `peek()` modify the stream element?

The `Consumer` can mutate a **mutable object**, but that's a side effect and should generally be avoided.

For example:

```java
.peek(user -> user.setStatus("ACTIVE"))
```

technically possible, but poor stream design if that mutation is core business logic.

---

### Q6. What happens with `peek()` in a parallel stream?

The action may execute concurrently on multiple threads, so:

- don't assume execution order
- don't use unsafe shared mutable state
- logging may interleave
- synchronization/atomic structures may be required if side effects are unavoidable

---

## Interview takeaway

Remember this one sentence:

> **`peek()` is a lazy intermediate operation primarily intended for observing/debugging elements as they flow through a stream; it should not be used as the primary mechanism for business side effects, especially in parallel streams.**

---

# Next: `takeWhile()` and `dropWhile()` ⭐

These are **Java 9+ operations** and are particularly interesting because they behave differently from `filter()`.

We'll cover:

```text
takeWhile()
dropWhile()
       ↓
ordered vs unordered streams
       ↓
difference from filter()
       ↓
parallel-stream behavior
       ↓
practical examples
```