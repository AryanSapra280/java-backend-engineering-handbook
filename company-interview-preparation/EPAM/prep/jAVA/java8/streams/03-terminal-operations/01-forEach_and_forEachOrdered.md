# Terminal Operations — Part 1: `forEach()` and `forEachOrdered()`

Now we enter the **terminal-operation section**. These are important because terminal operations actually trigger the Stream pipeline.

---

## 1. `forEach()`

### What does it do?

`forEach()` performs an action for every element.

```java
List<Integer> numbers = List.of(1, 2, 3, 4);

numbers.stream()
       .forEach(n -> System.out.println(n));
```

Output:

```text
1
2
3
4
```

Signature:

```java
void forEach(Consumer<? super T> action)
```

So:

```text
Stream<T>
   ↓
forEach(Consumer<T>)
   ↓
void
```

It is:

- **Terminal**
- **Eager once invoked**
- **Does not return a Stream**
- **Not short-circuiting**

---

# 2. Why is `forEach()` terminal?

Consider:

```java
numbers.stream()
       .filter(n -> n > 2)
       .forEach(System.out::println);
```

The pipeline is:

```text
Source
 ↓
filter()
 ↓
forEach() ← terminal
```

When `forEach()` is called, the stream starts consuming elements.

Without it:

```java
numbers.stream()
       .filter(n -> n > 2);
```

nothing needs to execute.

---

# 3. `forEach()` with parallel streams ⭐⭐⭐

This is where interviewers like to test you.

```java
List<Integer> numbers =
    List.of(1, 2, 3, 4, 5, 6);

numbers.parallelStream()
       .forEach(System.out::println);
```

You **cannot rely on encounter order**.

You might see:

```text
4
6
1
5
2
3
```

or another order.

Why?

Because different elements can be processed by different threads.

Conceptually:

```text
             parallelStream()
                  |
       -----------------------
       |          |          |
    thread-1   thread-2   thread-3
       |          |          |
      1,2        3,4        5,6
```

The threads execute independently.

---

# 4. `forEachOrdered()`

Java provides:

```java
forEachOrdered()
```

when you need encounter order.

```java
numbers.parallelStream()
       .forEachOrdered(System.out::println);
```

Output:

```text
1
2
3
4
5
6
```

assuming the source has encounter order.

---

# 5. `forEach()` vs `forEachOrdered()`

| | `forEach()` | `forEachOrdered()` |
|---|---|---|
| Terminal | Yes | Yes |
| Sequential stream | Usually encounter order | Encounter order |
| Parallel ordered stream | Order **not guaranteed** | Encounter order preserved |
| Parallelism | More freedom | More coordination |
| Return | `void` | `void` |

### Important:

Don't say:

> "`forEach()` always gives random order."

Better:

> **For a parallel stream, `forEach()` does not guarantee encounter order.**

---

# 6. Why does `forEachOrdered()` potentially reduce parallel performance?

Suppose we have:

```text
1 2 3 4 5 6 7 8
```

Parallel processing can independently process:

```text
Thread A → 1 2
Thread B → 3 4
Thread C → 5 6
Thread D → 7 8
```

With `forEach()`:

```text
A ──┐
B ──┼──> output whenever ready
C ──┤
D ──┘
```

With `forEachOrdered()`:

```text
A ──┐
B ──┼──> coordinate → 1 2 3 4 5 6 7 8
C ──┤
D ──┘
```

The actual implementation is more sophisticated than this, but this is the correct interview mental model.

**Parallel execution is still possible**, but preserving order introduces coordination.

---

# 7. Very important: `forEach()` and shared state 🚨

Bad:

```java
List<Integer> result = new ArrayList<>();

numbers.parallelStream()
       .forEach(result::add);
```

`ArrayList` isn't thread-safe.

Multiple threads may call:

```java
result.add(...)
```

concurrently.

This can cause race conditions and incorrect results.

### Better

Let the Stream API perform the collection:

```java
List<Integer> result =
    numbers.parallelStream()
           .toList();
```

This is one of the reasons you generally prefer **collectors / stream results over manually mutating shared collections**.

---

# 8. Side effects inside `forEach()`

This is another senior-level consideration.

You can do:

```java
numbers.forEach(n -> database.save(n));
```

but now your stream operation is performing external side effects.

With parallel:

```java
numbers.parallelStream()
       .forEach(n -> database.save(n));
```

you could suddenly have multiple concurrent database calls.

That might:

- overload the DB
- violate ordering assumptions
- cause transaction issues
- create concurrency problems

So don't assume:

> "I'll just add `parallelStream()` and make it faster."

---

# 9. Complexity

For `n` elements:

```text
forEach()
```

generally:

```text
Time: O(n)
```

assuming the Consumer is O(1).

But remember:

```java
.forEach(n -> database.save(n))
```

isn't really O(n) in practical runtime terms if each DB call is expensive.

The action's cost matters.

Space overhead is generally:

```text
O(1)
```

for the operation itself, excluding the underlying pipeline/source and any state created by your Consumer.

---

# 10. `forEach()` vs `map()`

Another common question.

### `map()`

```java
List<Integer> result =
    numbers.stream()
           .map(n -> n * 2)
           .toList();
```

Transforms data:

```text
1 → 2
2 → 4
3 → 6
```

### `forEach()`

```java
numbers.stream()
       .forEach(n -> System.out.println(n));
```

Performs an action and ends the pipeline.

So:

```text
map()
→ transformation

forEach()
→ terminal side-effect/action
```

---

# 11. `forEach()` vs `peek()`

You've just learned `peek()`.

This distinction should be crystal clear:

```java
stream
    .peek(...)
    .filter(...)
    .map(...)
    .forEach(...);
```

Pipeline:

```text
peek()   → intermediate
filter() → intermediate
map()    → intermediate
forEach()→ terminal
```

`peek()`:

```text
observe while pipeline continues
```

`forEach()`:

```text
consume stream and finish
```

---

# 12. Interview questions

### Q1. Is `forEach()` intermediate or terminal?

**Terminal.**

---

### Q2. Does `forEach()` preserve order?

For a sequential ordered stream, elements are normally processed in encounter order.

For a parallel stream, `forEach()` **does not guarantee encounter order**.

---

### Q3. How do you preserve order in a parallel stream?

```java
parallelStream()
    .forEachOrdered(...);
```

---

### Q4. Does `forEachOrdered()` make the stream sequential?

**No.**

The stream can still perform work in parallel, but encounter-order constraints introduce coordination.

---

### Q5. Why is this dangerous?

```java
List<Integer> list = new ArrayList<>();

numbers.parallelStream()
       .forEach(list::add);
```

Because multiple threads can mutate the non-thread-safe `ArrayList`.

---

# 🔥 Interview takeaway

Remember these three lines:

```text
forEach()
→ terminal
→ parallel execution doesn't guarantee encounter order

forEachOrdered()
→ terminal
→ preserves encounter order

peek()
→ intermediate
→ mainly for observation/debugging
```

---

# Next: `findFirst()` vs `findAny()` ⭐⭐⭐

This is **more important for interviews** because it combines:

- terminal operations
- short-circuiting
- `Optional`
- encounter order
- parallel streams
- performance trade-offs

The key question will be:

> **Why can `findAny()` be faster than `findFirst()` on a parallel stream?**