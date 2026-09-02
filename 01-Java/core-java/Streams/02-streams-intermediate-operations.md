# Java 8 — Stream Intermediate Operations

## 1. What Is an Intermediate Operation?

An intermediate operation:

- takes a Stream
- returns another Stream
- builds the Stream pipeline
- is generally lazy
- does not trigger execution by itself

Example:

```java
numbers.stream()
       .filter(x -> x > 10)
       .map(x -> x * 2);
```

Both:

```java
filter()
map()
```

are intermediate operations.

Nothing actually happens until a terminal operation such as:

```java
.toList()
```

is called.

---

# 2. Intermediate Operation Mental Model

Think of an intermediate operation as:

> "Add another processing step to the pipeline."

Example:

```java
numbers.stream()
       .filter(...)
       .map(...)
       .sorted();
```

Conceptually:

```text
Stream
  ↓
filter step
  ↓
map step
  ↓
sorted step
```

The pipeline is being built.

Execution happens when a terminal operation is eventually called.

---

# 3. `filter()` ⭐⭐⭐

`filter()` answers:

> Which elements should survive?

It takes a:

```text
Predicate<T>
```

which has the shape:

```text
T → boolean
```

Example:

```java
List<Integer> numbers =
    List.of(1, 2, 3, 4, 5, 6);

List<Integer> result =
    numbers.stream()
           .filter(x -> x % 2 == 0)
           .toList();
```

Result:

```text
[2, 4, 6]
```

Mental model:

```text
1 → false → discard
2 → true  → keep
3 → false → discard
4 → true  → keep
5 → false → discard
6 → true  → keep
```

---

# 4. Important Property of `filter()`

`filter()` does **not transform** an element.

It only decides:

```text
KEEP
```

or:

```text
DISCARD
```

Therefore, conceptually:

```text
Input Stream<T>
       ↓
    filter()
       ↓
Output Stream<T>
```

The element type normally stays the same.

---

# 5. `filter()` With Objects

Suppose:

```java
class Employee {

    private String name;
    private int salary;

    public String getName() {
        return name;
    }

    public int getSalary() {
        return salary;
    }
}
```

You can filter employees:

```java
employees.stream()
         .filter(e -> e.getSalary() > 100000)
         .toList();
```

The Stream still contains:

```text
Employee
```

You simply removed employees that didn't satisfy the condition.

---

# 6. Multiple `filter()` Operations

You can chain multiple filters:

```java
employees.stream()
         .filter(e -> e.getSalary() > 100000)
         .filter(e -> e.getAge() < 30)
         .toList();
```

Conceptually:

```text
employees
    ↓
salary > 100000
    ↓
age < 30
    ↓
result
```

You could also combine the conditions:

```java
employees.stream()
         .filter(e ->
             e.getSalary() > 100000 &&
             e.getAge() < 30
         )
         .toList();
```

Both approaches are valid.

---

# 7. `map()` ⭐⭐⭐

`map()` answers:

> How should each element be transformed?

It takes a:

```text
Function<T, R>
```

which has the shape:

```text
T → R
```

Example:

```java
List<Integer> numbers =
    List.of(1, 2, 3, 4);

List<Integer> result =
    numbers.stream()
           .map(x -> x * 10)
           .toList();
```

Result:

```text
[10, 20, 30, 40]
```

Mental model:

```text
1 → 10
2 → 20
3 → 30
4 → 40
```

---

# 8. `map()` Can Change the Type

This is extremely important.

Suppose:

```java
List<String> names =
    List.of("Aryan", "Rahul", "Amit");
```

We want the lengths:

```java
List<Integer> lengths =
    names.stream()
         .map(String::length)
         .toList();
```

Input:

```text
Stream<String>
```

Output:

```text
Stream<Integer>
```

Mental model:

```text
String
  ↓
map()
  ↓
Integer
```

This is why the signature uses:

```text
Function<T, R>
```

instead of:

```text
Function<T, T>
```

The output type can be different from the input type.

---

# 9. `filter()` vs `map()` ⭐⭐⭐

Very common interview question.

## `filter()`

Answers:

> Which elements should remain?

Conceptually:

```text
T → boolean
```

Example:

```java
.filter(x -> x > 10)
```

It selects/removes elements.

---

## `map()`

Answers:

> What should each element become?

Conceptually:

```text
T → R
```

Example:

```java
.map(x -> x * 2)
```

It transforms elements.

---

## Easy Memory Trick

```text
filter → WHICH elements?

map    → WHAT should each element become?
```

---

# 10. Production Example — Payment

Suppose:

```java
class Payment {

    BigDecimal getAmount() {
        // ...
    }
}
```

You want all payment amounts:

```java
List<BigDecimal> amounts =
    payments.stream()
            .map(Payment::getAmount)
            .toList();
```

Input:

```text
Payment
```

Output:

```text
BigDecimal
```

This is a very common real-world use of `map()`.

---

# 11. `filter()` + `map()` ⭐⭐⭐

One of the most common Stream patterns:

```java
List<BigDecimal> highValueAmounts =
    payments.stream()
            .filter(p ->
                p.getAmount()
                 .compareTo(new BigDecimal("10000")) > 0
            )
            .map(Payment::getAmount)
            .toList();
```

Conceptually:

```text
Payments
   ↓
filter high-value payments
   ↓
Payment objects
   ↓
extract amount
   ↓
BigDecimal values
```

Remember:

```text
filter → selects
map    → transforms
```

---

# 12. `flatMap()` ⭐⭐⭐

`flatMap()` becomes important when one element contains multiple elements.

Suppose:

```java
List<List<Integer>> numbers =
    List.of(
        List.of(1, 2),
        List.of(3, 4),
        List.of(5, 6)
    );
```

Structure:

```text
[
    [1, 2],
    [3, 4],
    [5, 6]
]
```

We want:

```text
[1, 2, 3, 4, 5, 6]
```

---

# 13. What Happens With `map()`?

If we do:

```java
numbers.stream()
       .map(list -> list.stream());
```

we conceptually get:

```text
Stream<Stream<Integer>>
```

Why?

Each:

```text
List<Integer>
```

is transformed into:

```text
Stream<Integer>
```

So:

```text
Stream<List<Integer>>
       ↓ map
Stream<Stream<Integer>>
```

That's usually not what we want.

---

# 14. `flatMap()` Solves This

```java
List<Integer> result =
    numbers.stream()
           .flatMap(list -> list.stream())
           .toList();
```

Result:

```text
[1, 2, 3, 4, 5, 6]
```

Conceptually:

```text
[1,2] ─┐
       │
[3,4] ─┼──→ [1,2,3,4,5,6]
       │
[5,6] ─┘
```

The nested Streams are flattened into one Stream.

---

# 15. `map()` vs `flatMap()` ⭐⭐⭐

### `map()`

```text
one input
   ↓
one output
```

Example:

```java
.map(x -> x * 2)
```

---

### `flatMap()`

```text
one input
   ↓
multiple/nested outputs
   ↓
flatten into one Stream
```

Example:

```java
.flatMap(list -> list.stream())
```

---

# 16. Production Example — Customers and Accounts

Suppose:

```java
class Customer {

    List<Account> getAccounts() {
        // ...
    }
}
```

And:

```java
List<Customer> customers;
```

You want all Accounts from all Customers.

Using:

```java
customers.stream()
         .map(Customer::getAccounts)
```

gives:

```text
Stream<List<Account>>
```

But we want:

```text
Stream<Account>
```

Use:

```java
customers.stream()
         .flatMap(
             customer ->
                 customer.getAccounts().stream()
         );
```

Mental model:

```text
Customer
   ↓
List<Account>
   ↓
flatten
   ↓
Account
```

This is a very realistic backend use case.

---

# 17. `distinct()`

`distinct()` removes duplicate elements.

Example:

```java
List<Integer> numbers =
    List.of(1, 2, 2, 3, 3, 3, 4);

List<Integer> result =
    numbers.stream()
           .distinct()
           .toList();
```

Result:

```text
[1, 2, 3, 4]
```

---

# 18. `distinct()` and Equality

For objects, `distinct()` depends on equality semantics.

Conceptually:

```text
distinct()
    ↓
determine duplicates
    ↓
equals() / hashCode()
```

Suppose:

```java
class Employee {
    int id;
    String name;
}
```

If you use:

```java
employees.stream()
         .distinct()
         .toList();
```

whether two Employee objects are considered duplicates depends on how equality is defined.

If `equals()` and `hashCode()` are not implemented appropriately, logically identical objects may not be considered duplicates.

This connects directly to Collections:

```text
HashSet / HashMap
        ↓
equals() + hashCode()

Stream.distinct()
        ↓
equality semantics
```

---

# 19. `sorted()`

`sorted()` sorts elements.

Example:

```java
List<Integer> numbers =
    List.of(5, 2, 4, 1, 3);

List<Integer> result =
    numbers.stream()
           .sorted()
           .toList();
```

Result:

```text
[1, 2, 3, 4, 5]
```

The no-argument version uses natural ordering.

This connects directly to:

```text
Comparable
compareTo()
```

which we already studied.

---

# 20. `sorted()` With Comparator

You can provide a Comparator:

```java
employees.stream()
         .sorted(
             Comparator.comparing(Employee::getSalary)
         )
         .toList();
```

For descending order:

```java
employees.stream()
         .sorted(
             Comparator.comparing(Employee::getSalary)
                       .reversed()
         )
         .toList();
```

This connects directly to:

```text
Comparable
    ↓
natural ordering

Comparator
    ↓
custom ordering
```

---

# 21. `limit()`

`limit(n)` keeps only the first `n` elements.

Example:

```java
List<Integer> result =
    numbers.stream()
           .limit(3)
           .toList();
```

For:

```text
[1, 2, 3, 4, 5]
```

result:

```text
[1, 2, 3]
```

---

# 22. `limit()` and Infinite Streams

`limit()` is especially useful with potentially infinite Streams.

Example:

```java
Stream.generate(() -> "Hello")
      .limit(5)
      .toList();
```

Result:

```text
[Hello, Hello, Hello, Hello, Hello]
```

Without `limit()`, an infinite Stream has no natural end.

---

# 23. `skip()`

`skip(n)` skips the first `n` elements.

Example:

```java
List<Integer> result =
    numbers.stream()
           .skip(2)
           .toList();
```

For:

```text
[1, 2, 3, 4, 5]
```

result:

```text
[3, 4, 5]
```

Mental model:

```text
[1,2,3,4,5]
    ↓ skip 2
[3,4,5]
```

---

# 24. `skip()` + `limit()`

They can be combined:

```java
numbers.stream()
       .skip(2)
       .limit(3)
       .toList();
```

For:

```text
[1,2,3,4,5,6]
```

result:

```text
[3,4,5]
```

Conceptually:

```text
[1,2,3,4,5,6]
      ↓
skip 2
      ↓
[3,4,5,6]
      ↓
limit 3
      ↓
[3,4,5]
```

This resembles offset/limit style pagination, although for large database datasets, database-level pagination is generally preferable to loading everything into memory first.

---

# 25. `peek()`

`peek()` allows you to observe elements as they pass through the pipeline.

Example:

```java
numbers.stream()
       .filter(x -> x > 2)
       .peek(x ->
           System.out.println("After filter: " + x)
       )
       .map(x -> x * 10)
       .toList();
```

It is primarily useful for debugging or inspecting a pipeline.

---

# 26. Don't Use `peek()` for Business Logic

Avoid:

```java
numbers.stream()
       .peek(x -> saveToDatabase(x))
       .toList();
```

Why?

Because `peek()` is:

- intermediate
- lazy
- intended mainly for observation/debugging

Its execution depends on the pipeline actually being consumed.

Prefer an explicit operation when business side effects are required.

---

# 27. Java 9 — `takeWhile()`

Since we're also covering modern Java, know that Java 9 added:

```java
takeWhile()
```

Example:

```java
Stream.of(1, 2, 3, 4, 1, 5)
      .takeWhile(x -> x < 4)
      .toList();
```

Result:

```text
[1, 2, 3]
```

Important:

`takeWhile()` keeps taking elements **while the condition remains true**.

At:

```text
4
```

the condition becomes false, so it stops.

```text
1 → true
2 → true
3 → true
4 → false → STOP
```

It does not continue searching later elements.

---

# 28. Java 9 — `dropWhile()`

Opposite idea:

```java
Stream.of(1, 2, 3, 4, 1, 5)
      .dropWhile(x -> x < 4)
      .toList();
```

Result:

```text
[4, 1, 5]
```

It drops elements while the condition is true.

Once the first element fails the condition, the remaining elements are kept.

`dropWhile()` was also added in Java 9.

---

# 29. Intermediate Operations by Category

## Filtering

```text
filter()
distinct()
```

## Transformation

```text
map()
flatMap()
```

## Ordering

```text
sorted()
```

## Slicing / Limiting

```text
limit()
skip()
takeWhile()    // Java 9+
dropWhile()    // Java 9+
```

## Inspection

```text
peek()
```

---

# 30. Intermediate Operations Return Streams

This is why chaining works:

```java
numbers.stream()
       .filter(...)
       .map(...)
       .sorted()
       .limit(5);
```

Conceptually:

```text
Stream
  ↓
filter()
  ↓
Stream
  ↓
map()
  ↓
Stream
  ↓
sorted()
  ↓
Stream
  ↓
limit()
  ↓
Stream
```

No final result has been produced yet.

A terminal operation is required.

---

# 31. Laziness + Intermediate Operations ⭐⭐⭐

Consider:

```java
numbers.stream()
       .filter(x -> x > 10)
       .map(x -> x * 2);
```

This creates a pipeline:

```text
Stream
  ↓
filter step
  ↓
map step
```

But it does not execute the processing yet.

When we add:

```java
.toList();
```

the terminal operation triggers execution.

Mental model:

```text
Intermediate operation
        ↓
"Add another step"

Terminal operation
        ↓
"Execute the pipeline"
```

---

# 32. Short-Circuiting

Some Stream operations can stop processing early.

For example:

```java
numbers.stream()
       .filter(x -> x > 100)
       .findFirst();
```

Once the first matching element is found, there is no need to continue searching.

Similarly:

```java
numbers.stream()
       .anyMatch(x -> x > 100);
```

can stop as soon as it finds one matching element.

These are terminal operations and will be covered in detail in the Terminal Operations section.

---

# 33. Complete Pipeline Example

Given:

```java
List<Integer> numbers =
    List.of(5, 2, 8, 2, 10, 4, 6);
```

Run:

```java
List<Integer> result =
    numbers.stream()
           .filter(x -> x > 3)
           .distinct()
           .sorted()
           .map(x -> x * 10)
           .limit(3)
           .toList();
```

Step-by-step conceptually:

```text
Original:
[5,2,8,2,10,4,6]

filter > 3:
[5,8,10,4,6]

distinct:
[5,8,10,4,6]

sorted:
[4,5,6,8,10]

map × 10:
[40,50,60,80,100]

limit 3:
[40,50,60]
```

Final:

```text
[40,50,60]
```

---

# 34. Most Important Distinctions

```text
filter
→ Which elements should remain?

map
→ What should each element become?

flatMap
→ Flatten nested results

distinct
→ Remove duplicates

sorted
→ Order elements

limit
→ Keep first N

skip
→ Ignore first N

peek
→ Observe/debug
```

---

# 35. Functional Interface Connection

The operations connect directly to what we learned earlier.

### `filter()`

```text
Predicate<T>
```

Example:

```java
.filter(x -> x > 10)
```

Shape:

```text
T → boolean
```

---

### `map()`

```text
Function<T, R>
```

Example:

```java
.map(x -> x * 2)
```

Shape:

```text
T → R
```

---

### `flatMap()`

Conceptually:

```text
Function<T, Stream<R>>
```

Example:

```java
.flatMap(list -> list.stream())
```

---

### `peek()`

```text
Consumer<T>
```

Example:

```java
.peek(System.out::println)
```

Shape:

```text
T → void
```

---

# 36. Interview Questions

## Basic

1. What is an intermediate operation?
2. Why are intermediate operations lazy?
3. What does `filter()` do?
4. What does `map()` do?
5. What does `distinct()` do?
6. What does `sorted()` do?
7. What does `limit()` do?
8. What does `skip()` do?
9. What is `peek()`?

---

## Important

10. Difference between `filter()` and `map()`?
11. Difference between `map()` and `flatMap()`?
12. Why can `map()` produce `Stream<Stream<T>>`?
13. How does `flatMap()` solve that?
14. How does `distinct()` determine duplicates?
15. Why do `equals()` and `hashCode()` matter for `distinct()`?
16. How is `sorted()` related to `Comparable` and `Comparator`?
17. Can you sort a Stream using a custom Comparator?
18. Why is `peek()` generally not recommended for business side effects?
19. What does lazy evaluation mean?
20. Why can short-circuiting improve Stream processing?
21. What is the difference between `limit()` and `skip()`?
22. What is the difference between `takeWhile()` and `filter()`?
23. What is the difference between `dropWhile()` and `filter()`?

---

# 37. Coding Practice — Basic

## Q1

Given:

```java
List<Integer> numbers =
    List.of(1, 2, 3, 4, 5, 6);
```

Return only even numbers.

Expected:

```text
[2,4,6]
```

---

## Q2

Given:

```java
List<Integer> numbers =
    List.of(1, 2, 3, 4, 5);
```

Multiply every number by 10.

Expected:

```text
[10,20,30,40,50]
```

---

## Q3

Given:

```java
List<Integer> numbers =
    List.of(1, 2, 2, 3, 3, 4);
```

Remove duplicates.

Expected:

```text
[1,2,3,4]
```

---

## Q4

Sort:

```java
List<Integer> numbers =
    List.of(5, 1, 4, 2, 3);
```

in descending order.

---

# 38. Coding Practice — Medium

## Q5

Given:

```java
List<Integer> numbers =
    List.of(1, 2, 3, 4, 5, 6, 7, 8);
```

Return:

```text
[30,40,50]
```

using Stream operations.

Think carefully about the order of:

```text
filter
map
skip
limit
```

---

## Q6

Given:

```java
List<List<Integer>> numbers =
    List.of(
        List.of(1, 2),
        List.of(3, 4),
        List.of(5, 6)
    );
```

Flatten into:

```text
[1,2,3,4,5,6]
```

Use `flatMap()`.

---

# 39. Coding Practice — Production Style

## Q7

Given:

```java
class Customer {

    List<Account> getAccounts() {
        // ...
    }
}
```

and:

```java
List<Customer> customers;
```

Return all Accounts from all Customers using `flatMap()`.

---

## Q8 — Reasoning ⭐⭐

What's the difference between:

```java
customers.stream()
         .map(Customer::getAccounts)
```

and:

```java
customers.stream()
         .flatMap(
             customer ->
                 customer.getAccounts().stream()
         )
```

What are the resulting Stream types?

---

# 40. Final Cheat Sheet

```text
STREAM INTERMEDIATE OPERATIONS
==============================

Filtering:
filter()
distinct()

Transformation:
map()
flatMap()

Ordering:
sorted()

Slicing:
limit()
skip()

Java 9:
takeWhile()
dropWhile()

Inspection:
peek()
```

### Functional Interface Connection

```text
filter()
   ↓
Predicate<T>

map()
   ↓
Function<T,R>

flatMap()
   ↓
Function<T, Stream<R>>

peek()
   ↓
Consumer<T>
```

### Mental Model

```text
filter   → WHICH elements?

map      → WHAT should each element become?

flatMap  → FLATTEN nested results

distinct → REMOVE duplicates

sorted   → ORDER elements

limit    → KEEP first N

skip     → IGNORE first N

peek     → OBSERVE / DEBUG
```

---

# 41. Stream Learning Progress

```text
Stream Fundamentals                 ✅
Stream Source Methods               ✅
Stream Intermediate Operations      ✅
        ↓
Terminal Operations                 ← NEXT
        ↓
forEach()
toList()
collect()
reduce()
count()
findFirst()
findAny()
anyMatch()
allMatch()
noneMatch()
min()
max()
        ↓
Collectors
        ↓
Stream Coding Practice
        ↓
Stream Interview Questions
```

**Java 8 — Stream Intermediate Operations = DONE ✅**