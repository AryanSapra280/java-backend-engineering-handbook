# `distinct()` — Removing Duplicates

Now we continue the Stream API exactly from where we left off.

---

## 1. Problem — Why do we need `distinct()`?

Suppose:

```java
List<Integer> numbers =
        List.of(10, 20, 10, 30, 20, 40, 30);
```

We want:

```text
[10, 20, 30, 40]
```

We could manually use a `Set`, but Stream already provides:

```java
distinct()
```

So:

```java
List<Integer> result =
        numbers.stream()
               .distinct()
               .toList();
```

Result:

```text
[10, 20, 30, 40]
```

---

# 2. Concept

`distinct()` removes duplicate elements according to the stream's equality semantics.

For normal objects, this fundamentally depends on:

```java
equals()
hashCode()
```

Mental model:

```text
Input:
A A B C C D

distinct()

Output:
A B C D
```

---

# 3. Important — `distinct()` Is Stateful

Remember our earlier distinction:

### Stateless

```text
filter()
map()
```

Each element can generally be processed independently.

### Stateful

```text
distinct()
sorted()
```

These need information about **other elements**.

For example, when processing:

```text
10
```

`distinct()` needs to know:

> "Have I already seen 10?"

So conceptually it maintains a set of previously seen elements:

```text
Seen:
{}

10 → not seen → emit → {10}

20 → not seen → emit → {10,20}

10 → already seen → discard

30 → not seen → emit → {10,20,30}
```

That's why `distinct()` is stateful.

---

# 4. How Does It Know Something Is a Duplicate?

This is where your Collections knowledge connects directly.

Consider:

```java
List<String> names =
        List.of("Aryan", "Rahul", "Aryan");
```

`String` already correctly implements:

```java
equals()
hashCode()
```

So:

```java
names.stream()
     .distinct()
     .toList();
```

gives:

```text
[Aryan, Rahul]
```

Conceptually, Stream maintains a set of seen values.

So your understanding of:

```text
HashSet
   ↓
hashCode()
   ↓
equals()
```

is directly relevant here.

---

# 5. Custom Objects — VERY IMPORTANT

Suppose:

```java
class Employee {
    private int id;
    private String name;

    // constructor/getters
}
```

And:

```java
List<Employee> employees =
        List.of(
            new Employee(1, "Aryan"),
            new Employee(2, "Rahul"),
            new Employee(1, "Aryan")
        );
```

Now:

```java
employees.stream()
         .distinct()
         .toList();
```

Will it necessarily remove the second employee?

**No.**

If `Employee` hasn't overridden `equals()` and `hashCode()`, two separate objects are generally considered different objects.

Even though:

```text
id = 1
name = Aryan
```

is the same.

---

# 6. Why `equals()` / `hashCode()` Matter

Suppose we implement:

```java
class Employee {

    private int id;
    private String name;

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (!(o instanceof Employee e)) return false;

        return id == e.id &&
               Objects.equals(name, e.name);
    }

    @Override
    public int hashCode() {
        return Objects.hash(id, name);
    }
}
```

Now:

```java
employees.stream()
         .distinct()
         .toList();
```

can identify logically equal employees as duplicates.

This is a **very good interview connection**:

> `distinct()` relies on object equality semantics, so incorrect `equals()`/`hashCode()` implementations can produce unexpected results.

---

# 7. `distinct()` Preserves Encounter Order

For an ordered sequential stream:

```java
List<Integer> numbers =
        List.of(3, 1, 3, 2, 1, 4);
```

Then:

```java
numbers.stream()
       .distinct()
       .toList();
```

gives:

```text
[3, 1, 2, 4]
```

Notice:

- first `3` survives
- second `3` is removed
- first `1` survives
- second `1` is removed
- `2` survives
- `4` survives

So conceptually:

```text
3 → keep
1 → keep
3 → discard
2 → keep
1 → discard
4 → keep
```

The first occurrence is retained.

---

# 8. `filter()` + `distinct()`

Very common pattern.

Suppose:

```java
List<Employee> employees = ...;
```

Requirement:

> Get unique names of employees whose salary is above 50,000.

```java
List<String> names =
        employees.stream()
                 .filter(e -> e.getSalary() > 50000)
                 .map(Employee::getName)
                 .distinct()
                 .toList();
```

Pipeline:

```text
Employee
   ↓
filter salary
   ↓
Employee
   ↓
map → String
   ↓
distinct
   ↓
unique String
```

Notice the order.

If we want unique **names**, `distinct()` should happen after `map(Employee::getName)`.

---

# 9. Example of Why Ordering Matters

Suppose:

```text
Employee 1 → Aryan → 80k
Employee 2 → Aryan → 60k
Employee 3 → Rahul → 70k
```

This:

```java
employees.stream()
         .distinct()
         .map(Employee::getName)
```

checks uniqueness of **Employee objects**.

Whereas:

```java
employees.stream()
         .map(Employee::getName)
         .distinct()
```

checks uniqueness of **String names**.

These are different requirements.

Very important in interviews.

---

# 10. `distinct()` Is Intermediate and Lazy

This:

```java
numbers.stream()
       .distinct();
```

doesn't actually process the numbers yet.

Because:

```text
distinct()
```

is an intermediate operation.

You need:

```java
numbers.stream()
       .distinct()
       .toList();
```

for execution.

---

# 11. Complexity

For:

```java
numbers.stream()
       .distinct()
       .toList();
```

with `n` elements:

Typical expected time:

```text
O(n)
```

because duplicate detection is generally hash-based.

Additional state:

```text
O(n)
```

in the worst case because it may need to remember every distinct element.

This is an important difference from `filter()`.

### `filter()`

```text
Additional state ≈ O(1)
```

### `distinct()`

```text
Additional state ≈ O(n)
```

because it needs to remember what it has already encountered.

---

# 12. Production Consideration — Memory

Imagine:

```text
10 million records
```

and almost every record is unique.

Then `distinct()` potentially has to retain information about millions of values.

So don't think:

> "It's just one Stream operation, therefore it is memory-efficient."

Streams don't magically eliminate algorithmic memory requirements.

---

# 13. `distinct()` vs `Set`

You could write:

```java
Set<Integer> result =
        new HashSet<>(numbers);
```

But there is an important difference.

`distinct()` works naturally **inside a pipeline**:

```java
numbers.stream()
       .filter(...)
       .map(...)
       .distinct()
       .limit(10)
       .toList();
```

You can perform all those operations declaratively.

---

# 14. `distinct()` + `limit()`

This is an interesting pipeline:

```java
numbers.stream()
       .distinct()
       .limit(5)
       .toList();
```

Requirement:

> Get the first five unique values.

Example:

```text
Input:
1 2 1 3 2 4 5 6

distinct:
1 2 3 4 5 6

limit(5):
1 2 3 4 5
```

This is different from:

```java
numbers.stream()
       .limit(5)
       .distinct()
       .toList();
```

Here:

```text
limit first 5:
1 2 1 3 2

distinct:
1 2 3
```

So **operation ordering changes the result**.

That's a classic interview point.

---

# 15. `distinct()` With Objects

This is another practical interview question.

Suppose:

```java
class User {
    private String email;
}
```

Requirement:

> Remove duplicate users based on email.

Simply doing:

```java
users.stream()
     .distinct()
```

only works correctly if `equals()`/`hashCode()` define equality based on email.

If the existing equality contract doesn't match the business requirement, you may instead do something like:

```java
users.stream()
     .collect(Collectors.toMap(
         User::getEmail,
         Function.identity(),
         (first, second) -> first
     ))
     .values()
     .stream()
     .toList();
```

We'll cover `toMap()` properly later.

The important lesson:

> `distinct()` uses the object's equality definition. It doesn't know your business definition of "duplicate."

---

# 16. Parallel Stream — Important Nuance

With:

```java
numbers.parallelStream()
       .distinct()
```

the implementation has to coordinate duplicate detection across parallel processing.

This can make `distinct()` relatively expensive in parallel streams, especially when encounter order must be preserved.

So don't automatically assume:

```text
parallelStream + distinct = faster
```

It can actually be worse depending on the workload.

We'll cover this properly when we reach **parallel streams**.

---

# 17. Interview Questions

### Q1. Is `distinct()` intermediate or terminal?

**Intermediate.**

### Q2. Is `distinct()` stateless or stateful?

**Stateful.**

Because it needs to remember previously encountered elements.

### Q3. What does `distinct()` use for duplicate detection?

For object streams, it relies on the equality semantics of the elements—effectively `equals()`/`hashCode()`.

### Q4. What's the typical complexity?

```text
Time: O(n) expected
Space: O(n) worst case
```

### Q5. Does `distinct()` preserve order?

For an ordered stream, the first occurrence is retained and encounter order is preserved.

### Q6. Why can `distinct()` be expensive?

Because it may need to maintain a set of previously seen elements.

### Q7. Will this remove duplicate custom objects?

```java
employees.stream()
         .distinct();
```

Only according to the custom object's `equals()`/`hashCode()` implementation.

---

# Mental Model

Lock this in:

```text
filter()
    ↓
stateless
    ↓
"Should I keep this element?"

map()
    ↓
stateless
    ↓
"What should this element become?"

distinct()
    ↓
stateful
    ↓
"Have I already seen this element?"
```

And:

```text
Input:
A B A C B D

seen = {}

A → keep → {A}
B → keep → {A,B}
A → discard
C → keep → {A,B,C}
B → discard
D → keep → {A,B,C,D}

Output:
A B C D
```

### Next: `sorted()`

We'll cover **natural ordering vs Comparator, Comparable vs Comparator, statefulness, why `sorted()` may need to see the entire stream, custom object sorting, multi-field sorting, and `sorted()` vs database `ORDER BY`**.