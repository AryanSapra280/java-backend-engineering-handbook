Absolutely — let's continue in order. We just finished `findFirst()` / `findAny()`.

# Next: `anyMatch()`, `allMatch()`, `noneMatch()` ⭐⭐⭐

These are **boolean terminal operations** and all three are **short-circuiting**.

### 1. `anyMatch()`

> "Does **at least one** element satisfy the condition?"

```java
boolean result =
    numbers.stream()
           .anyMatch(n -> n > 100);
```

If it finds one match:

```text
10 → no
20 → no
150 → YES → STOP
```

Result:

```java
true
```

It doesn't need to inspect the remaining elements.

---

### 2. `allMatch()`

> "Do **all** elements satisfy the condition?"

```java
boolean result =
    numbers.stream()
           .allMatch(n -> n > 0);
```

If it finds one failure:

```text
10 → yes
20 → yes
-5 → NO → STOP
```

Result:

```java
false
```

So:

```text
anyMatch → stop when TRUE
allMatch → stop when FALSE
```

---

### 3. `noneMatch()`

> "Does **no** element satisfy the condition?"

```java
boolean result =
    numbers.stream()
           .noneMatch(n -> n < 0);
```

If it finds a negative number:

```text
10 → no
20 → no
-5 → MATCH → STOP
```

Result:

```java
false
```

Think of it as:

```text
noneMatch(P)
    ≈
!anyMatch(P)
```

---

## The easiest way to remember all three

| Operation | Question | Stops when |
|---|---|---|
| `anyMatch()` | Is **at least one** true? | Finds `true` |
| `allMatch()` | Are **all** true? | Finds `false` |
| `noneMatch()` | Is **zero** true? | Finds `true` |

---

# Parallel streams ⭐⭐⭐

All three can execute predicates concurrently.

For:

```java
numbers.parallelStream()
       .anyMatch(n -> n > 100);
```

multiple partitions can search simultaneously.

Once a matching element is found, the overall result can become `true`, although other parallel tasks may already be in progress.

Similarly:

```text
anyMatch  → looking for TRUE
allMatch  → looking for FALSE
noneMatch → looking for TRUE
```

The important point is:

> **Short-circuiting in a parallel stream does not mean every other thread instantly stops at the exact same moment.**

Some work may already have been started.

---

## Practical examples

### Check whether any employee is a manager

```java
boolean hasManager =
    employees.stream()
             .anyMatch(Employee::isManager);
```

### Validate that all payments are positive

```java
boolean valid =
    payments.stream()
            .allMatch(p -> p.getAmount() > 0);
```

### Check that no account is blocked

```java
boolean safe =
    accounts.stream()
            .noneMatch(Account::isBlocked);
```

These are much cleaner than manually maintaining boolean variables.

---

# Complexity

Worst case, all three can inspect every element:

```text
Time: O(n)
Space: O(1)
```

But because they're short-circuiting, they can finish much earlier.

For example:

```java
anyMatch(...)
```

with the first element matching:

```text
O(1) effectively
```

---

# ⚠️ Important: empty streams

This is a classic interview question.

```java
Stream.empty().anyMatch(x -> true)
```

→ `false`

Because there isn't any matching element.

```java
Stream.empty().allMatch(x -> false)
```

→ `true`

This surprises people.

Why?

Because there is **no counterexample** to the statement "all elements satisfy the condition."

And:

```java
Stream.empty().noneMatch(x -> true)
```

→ `true`

Because there are no elements matching the predicate.

So:

```text
empty.anyMatch()  → false
empty.allMatch()  → true
empty.noneMatch() → true
```

---

# 🔥 Interview takeaway

Remember:

> **`anyMatch`, `allMatch`, and `noneMatch` are short-circuiting terminal operations returning boolean. `anyMatch` stops on a match, `allMatch` stops on a failure, and `noneMatch` stops when a match is found.**

---

## Next topics

After these, we have:

```text
count()
   ↓
min() / max()
   ↓
reduce() ⭐⭐⭐
   ↓
collect() ⭐⭐⭐
   ↓
toMap()
   ↓
groupingBy() ⭐⭐⭐
   ↓
partitioningBy()
   ↓
downstream collectors
   ↓
Optional + Streams
   ↓
Parallel Streams
   ↓
Stream coding patterns
```

**The next one is `count()`, then `min()` / `max()`.**