# `findFirst()` vs `findAny()` ⭐⭐⭐

This is a **very common Java Stream interview topic**, especially because it exposes whether you understand **encounter order + short-circuiting + parallel streams**.

---

## 1. `findFirst()`

`findFirst()` returns the **first element according to the stream's encounter order**.

```java
List<Integer> numbers = List.of(10, 20, 30, 40, 50);

Optional<Integer> result =
    numbers.stream()
           .filter(n -> n > 25)
           .findFirst();

System.out.println(result);
```

Result:

```text
Optional[30]
```

Because:

```text
10 → no
20 → no
30 → YES ← first matching element
40 → ...
50 → ...
```

Signature:

```java
Optional<T> findFirst()
```

---

# 2. Why does it return `Optional`?

What if nothing matches?

```java
Optional<Integer> result =
    numbers.stream()
           .filter(n -> n > 100)
           .findFirst();
```

Result:

```text
Optional.empty
```

This avoids returning `null`.

So you can safely do:

```java
result.ifPresent(System.out::println);
```

or:

```java
Integer value = result.orElse(0);
```

---

# 3. `findFirst()` is short-circuiting

This is important.

```java
numbers.stream()
       .filter(n -> n > 25)
       .findFirst();
```

Once `30` is found, the stream doesn't need to continue searching for the first match.

Conceptually:

```text
10 → filter ❌
20 → filter ❌
30 → filter ✅ → STOP
40 → not needed
50 → not needed
```

So `findFirst()` is a **short-circuiting terminal operation**.

---

# 4. `findAny()`

`findAny()` returns **some element** matching the pipeline.

```java
Optional<Integer> result =
    numbers.stream()
           .filter(n -> n > 25)
           .findAny();
```

For a sequential stream, you will commonly see:

```text
Optional[30]
```

But don't build your code around that assumption.

The contract of `findAny()` does **not require the first matching element**.

Its purpose is to allow the implementation more freedom.

---

# 5. The big difference ⭐⭐⭐

### `findFirst()`

```text
Give me the FIRST matching element.
```

### `findAny()`

```text
Give me ANY matching element.
```

That's the entire conceptual difference.

---

# 6. Why is `findAny()` interesting with parallel streams?

Suppose:

```java
List<Integer> numbers =
    List.of(1, 2, 3, 4, 5, 6, 7, 8);
```

And:

```java
numbers.parallelStream()
       .filter(n -> n % 2 == 0)
       .findAny();
```

Conceptually:

```text
Partition A → [1,2]
Partition B → [3,4]
Partition C → [5,6]
Partition D → [7,8]
```

Multiple threads can search simultaneously.

Suppose:

```text
Thread A → finds 2
Thread B → finds 4
Thread C → finds 6
Thread D → finds 8
```

`findAny()` can essentially say:

> "I don't care which one. Give me a valid match."

So once a valid match is available, the operation has much more freedom to finish.

Result could be:

```text
2
```

or:

```text
4
```

or:

```text
6
```

or:

```text
8
```

depending on execution.

---

# 7. Why is `findFirst()` potentially more expensive?

With:

```java
parallelStream()
    .filter(...)
    .findFirst();
```

the system must respect **encounter order**.

Suppose:

```text
Original:

[1, 2, 3, 4, 5, 6, 7, 8]
```

and:

```text
2, 4, 6, 8
```

match.

Suppose different threads discover:

```text
Thread A → 2
Thread B → 4
Thread C → 6
Thread D → 8
```

Even if Thread C finds `6` first, `findFirst()` cannot simply return `6`.

It needs to determine whether there is an earlier matching element.

There could be:

```text
2
```

earlier in encounter order.

So ordered parallel `findFirst()` may require more coordination.

---

# 8. `findAny()` vs `findFirst()` in parallel

This is the interview answer:

| | `findFirst()` | `findAny()` |
|---|---|---|
| Returns | First matching element | Any matching element |
| Short-circuiting | Yes | Yes |
| Returns Optional | Yes | Yes |
| Respects encounter order | Yes, for ordered streams | Not required |
| Parallel freedom | Lower | Higher |
| Potential parallel performance | More coordination | Potentially better |

### The key sentence:

> **`findAny()` can be more efficient in parallel because it doesn't need to preserve encounter order.**

Don't say "`findAny()` is always faster."

It **can** be faster depending on the workload and stream characteristics.

---

# 9. Sequential stream — important nuance

Consider:

```java
numbers.stream()
       .filter(...)
       .findAny();
```

You'll commonly get the first matching element.

But you should **not rely on that as a contract**.

If your business requirement is:

> "I need the first element."

Use:

```java
findFirst()
```

If your requirement is:

> "I only need any valid matching element."

Use:

```java
findAny()
```

This is a great example of choosing an API based on **business semantics**, not just implementation behavior.

---

# 10. Practical production example

Imagine you need to know:

> "Does any server have capacity?"

You don't care which server.

```java
Optional<Server> server =
    servers.parallelStream()
           .filter(Server::hasCapacity)
           .findAny();
```

That's a good use case.

You don't need:

```text
the first server
```

You just need:

```text
any available server
```

---

# 11. Another example: first transaction

Suppose transactions are chronologically ordered:

```java
transactions.stream()
            .filter(Transaction::isFailed)
            .findFirst();
```

Here you **do care about order**.

You want:

> First failed transaction.

Therefore:

```java
findFirst()
```

is correct.

Using:

```java
findAny()
```

would violate the business requirement.

---

# 12. `findFirst()` and unordered streams

Here's a subtle interview point.

For an **ordered stream**, `findFirst()` has clear encounter-order semantics.

If you explicitly make a stream unordered:

```java
numbers.parallelStream()
       .unordered()
       .filter(...)
       .findFirst();
```

you have removed the ordering constraint.

The important principle is:

> Stream ordering is part of the stream's semantics, and `unordered()` gives the implementation more freedom.

Don't use `unordered()` casually just for performance if your application depends on order.

---

# 13. Complexity

For a sequential stream, both can potentially stop early.

If the matching element occurs after `k` elements:

```text
findFirst() → roughly O(k)
findAny()   → potentially O(k) sequentially
```

In parallel streams, exact performance depends heavily on:

- source
- partitioning
- number of threads
- position of matches
- ordered vs unordered stream
- cost of the predicate

So don't give a simplistic "`findAny()` is O(1)" answer.

It isn't.

---

# 14. Common interview trap

### Question:

```java
List<Integer> list =
    List.of(1, 3, 5, 8, 10, 12);

Optional<Integer> result =
    list.parallelStream()
        .filter(n -> n % 2 == 0)
        .findAny();
```

What can `result` contain?

Potentially:

```text
8
10
12
```

Any matching element.

Not necessarily `8`.

---

### But:

```java
list.parallelStream()
    .filter(n -> n % 2 == 0)
    .findFirst();
```

For this ordered list, the answer must be:

```text
8
```

because `8` is the first matching element in encounter order.

---

# 15. One more important connection

You've now seen three different ways streams can stop early:

```text
limit()
takeWhile()
findFirst()/findAny()
```

And later you'll see:

```text
anyMatch()
allMatch()
noneMatch()
```

These are all related to **short-circuiting**, but their semantics are different.

---

## 🔥 Interview takeaway

Memorize this:

> **`findFirst()` guarantees the first matching element according to encounter order, while `findAny()` can return any matching element and therefore provides more freedom for parallel execution. Both are short-circuiting terminal operations and return `Optional<T>`.**

### Next → `anyMatch()`, `allMatch()`, `noneMatch()`

These three are extremely useful because they introduce **boolean short-circuiting** and have interesting differences in **parallel execution**.