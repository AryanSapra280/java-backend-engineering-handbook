# Java 8 — Stream Terminal Operations

## 1. What Is a Terminal Operation?

A terminal operation **ends the Stream pipeline** and produces a final result or performs a side effect.

A typical Stream pipeline:

```text
SOURCE
   ↓
Intermediate Operations
   ↓
Terminal Operation
   ↓
RESULT
```

Example:

```java
List<Integer> result =
    numbers.stream()
           .filter(x -> x % 2 == 0)
           .map(x -> x * 10)
           .toList();
```

Here:

```text
stream()
    → source

filter()
    → intermediate

map()
    → intermediate

toList()
    → terminal
```

---

# 2. Important Properties

Terminal operations:

- trigger Stream execution
- consume the Stream
- produce a final result or side effect
- cannot be followed by another operation on the same Stream

Example:

```java
Stream<Integer> stream =
    numbers.stream();

long count = stream.count();

List<Integer> result =
    stream.toList();    // ERROR
```

The Stream has already been consumed.

You must create a new Stream:

```java
numbers.stream()
       .toList();
```

---

# 3. `forEach()` ⭐⭐⭐

`forEach()` performs an action for every element.

Example:

```java
List<Integer> numbers =
    List.of(1, 2, 3, 4, 5);

numbers.stream()
       .forEach(System.out::println);
```

Output:

```text
1
2
3
4
5
```

It takes a:

```text
Consumer<T>
```

because:

```text
T → void
```

Example:

```java
.forEach(x -> System.out.println(x))
```

or:

```java
.forEach(System.out::println)
```

---

# 4. `forEach()` With Parallel Streams

With:

```java
numbers.parallelStream()
       .forEach(System.out::println);
```

the order of output is **not guaranteed**.

Possible output:

```text
3
4
1
5
2
```

The exact order depends on parallel execution.

Therefore:

```text
forEach()
    ↓
Order is not guaranteed for parallel Streams
```

---

# 5. `forEachOrdered()`

`forEachOrdered()` performs the action while respecting the Stream's **encounter order**, when an encounter order exists.

Example:

```java
numbers.parallelStream()
       .forEachOrdered(System.out::println);
```

For an ordered source such as a List, the output follows:

```text
1
2
3
4
5
```

### Important

`forEachOrdered()` does **not** mean:

```text
sort()
```

It respects the existing encounter order.

Example:

```java
List<Integer> numbers =
    List.of(5, 2, 4, 1, 3);

numbers.stream()
       .forEachOrdered(System.out::println);
```

Output:

```text
5
2
4
1
3
```

It does not sort the values.

---

# 6. `forEach()` vs `forEachOrdered()`

```text
forEach()
    → process every element
    → parallel Stream does not guarantee encounter order

forEachOrdered()
    → process every element
    → preserve encounter order when one exists
```

With a parallel Stream:

```text
forEach()
    → more freedom for parallel execution

forEachOrdered()
    → additional coordination may be required
      to preserve encounter order
```

Important:

```text
ordering guarantee
       ≠
thread safety
```

`forEachOrdered()` does not make arbitrary mutable objects thread-safe.

---

# 7. `toList()` ⭐⭐⭐

`toList()` collects Stream elements into a List.

Example:

```java
List<Integer> result =
    numbers.stream()
           .filter(x -> x % 2 == 0)
           .toList();
```

Result:

```text
[2, 4]
```

### Java Version

`Stream.toList()` was introduced in **Java 16**.

Therefore, when working with Java 8 code, you will commonly see:

```java
.collect(Collectors.toList())
```

Modern Java:

```java
.toList()
```

Java 8 style:

```java
.collect(Collectors.toList())
```

Know both for interviews.

---

# 8. `count()` ⭐⭐⭐

`count()` returns the number of elements.

Example:

```java
long count =
    numbers.stream()
           .filter(x -> x % 2 == 0)
           .count();
```

For:

```text
[1,2,3,4,5]
```

result:

```text
2
```

### Important

The return type is:

```java
long
```

not:

```java
int
```

So:

```java
long count = stream.count();
```

---

# 9. `min()` ⭐⭐⭐

Finds the minimum element.

Example:

```java
Optional<Integer> min =
    numbers.stream()
           .min(Integer::compareTo);
```

For:

```text
[5,2,8,1,4]
```

the result represents:

```text
Optional[1]
```

### Why Optional?

The Stream might be empty.

For:

```java
List<Integer> numbers =
    List.of();
```

there is no minimum.

Therefore:

```text
min()
   ↓
Optional<T>
```

---

# 10. `max()`

Similar to `min()`:

```java
Optional<Integer> max =
    numbers.stream()
           .max(Integer::compareTo);
```

For:

```text
[5,2,8,1,4]
```

result:

```text
Optional[8]
```

Again:

```text
max()
   ↓
Optional<T>
```

because the Stream might be empty.

---

# 11. `findFirst()` ⭐⭐⭐

Returns the first element according to encounter order.

Example:

```java
Optional<Integer> first =
    numbers.stream()
           .findFirst();
```

For:

```text
[10,20,30,40]
```

result:

```text
Optional[10]
```

If the Stream is empty:

```text
Optional.empty()
```

Therefore:

```text
findFirst()
    ↓
Optional<T>
```

---

# 12. Why Does `findFirst()` Return Optional?

Consider:

```java
List<Integer> numbers =
    List.of();
```

Now:

```java
numbers.stream()
       .findFirst();
```

There is no first element.

Returning:

```java
null
```

would require null handling.

Instead Java returns:

```java
Optional<Integer>
```

which can represent:

```text
value exists
```

or:

```text
value doesn't exist
```

---

# 13. `findAny()` ⭐⭐⭐

Returns **some element** from the Stream.

Example:

```java
Optional<Integer> result =
    numbers.stream()
           .findAny();
```

The important semantic difference:

```text
findFirst()
    → specifically asks for the first element

findAny()
    → asks for any element
```

With a sequential Stream, `findAny()` will often appear to return the first element, but you should **not rely on that as its guarantee**.

---

# 14. `findFirst()` vs `findAny()`

| Operation | Meaning |
|---|---|
| `findFirst()` | First element according to encounter order |
| `findAny()` | Any element |

`findAny()` can be useful when order does not matter, especially with parallel Streams.

Why?

Because the implementation has more freedom to return a suitable element rather than coordinating to determine the first element.

Mental model:

```text
findFirst()
    → "I specifically need the first."

findAny()
    → "I just need one."
```

---

# 15. `anyMatch()` ⭐⭐⭐

Answers:

> Does **at least one** element satisfy this condition?

Example:

```java
boolean result =
    numbers.stream()
           .anyMatch(x -> x > 10);
```

If at least one number is greater than 10:

```text
true
```

Otherwise:

```text
false
```

It takes:

```text
Predicate<T>
```

because:

```text
T → boolean
```

---

# 16. `allMatch()`

Answers:

> Do **all** elements satisfy the condition?

Example:

```java
boolean result =
    numbers.stream()
           .allMatch(x -> x > 0);
```

For:

```text
[1,2,3,4,5]
```

result:

```text
true
```

But:

```text
[1,2,-3,4]
```

gives:

```text
false
```

---

# 17. `noneMatch()`

Answers:

> Does **no** element satisfy the condition?

Example:

```java
boolean result =
    numbers.stream()
           .noneMatch(x -> x < 0);
```

For:

```text
[1,2,3,4]
```

result:

```text
true
```

For:

```text
[1,2,-3,4]
```

result:

```text
false
```

---

# 18. The Three Match Operations ⭐⭐⭐

Memorize this:

```text
anyMatch()
    → at least ONE

allMatch()
    → EVERY element

noneMatch()
    → ZERO
```

Example:

```java
anyMatch(x -> x > 100)
```

means:

```text
"Is there at least one?"
```

---

```java
allMatch(x -> x > 0)
```

means:

```text
"Are all of them positive?"
```

---

```java
noneMatch(x -> x < 0)
```

means:

```text
"Are there no negative values?"
```

---

# 19. Match Operations Are Short-Circuiting ⭐⭐⭐

This connects directly to our earlier discussion of **lazy evaluation**.

Consider:

```java
numbers.stream()
       .anyMatch(x -> x > 100);
```

As soon as Java finds a matching element, it can stop.

It does not need to inspect the remaining elements.

Similarly:

```java
allMatch(...)
```

can stop when it finds the first element that fails the condition.

And:

```java
noneMatch(...)
```

can stop when it finds an element that satisfies the condition.

Mental model:

```text
anyMatch()
    ↓
first TRUE → STOP

allMatch()
    ↓
first FALSE → STOP

noneMatch()
    ↓
first TRUE → STOP
```

---

# 20. `reduce()` ⭐⭐⭐

`reduce()` combines multiple Stream elements into **one result**.

Example:

```java
List<Integer> numbers =
    List.of(1, 2, 3, 4, 5);

int sum =
    numbers.stream()
           .reduce(0, (a, b) -> a + b);
```

Result:

```text
15
```

Conceptually:

```text
0 + 1 = 1
1 + 2 = 3
3 + 3 = 6
6 + 4 = 10
10 + 5 = 15
```

Mental model:

```text
many elements
      ↓
    reduce()
      ↓
one result
```

---

# 21. `reduce()` and `BinaryOperator`

This:

```java
(a, b) -> a + b
```

has the shape:

```text
T, T → T
```

which corresponds to:

```text
BinaryOperator<T>
```

Therefore:

```java
numbers.stream()
       .reduce(0, (a, b) -> a + b);
```

uses a BinaryOperator to combine elements.

This connects directly with Functional Interfaces.

---

# 22. `reduce()` for Multiplication

```java
int result =
    numbers.stream()
           .reduce(1, (a, b) -> a * b);
```

For:

```text
[1,2,3,4,5]
```

result:

```text
120
```

Conceptually:

```text
1 × 1 = 1
1 × 2 = 2
2 × 3 = 6
6 × 4 = 24
24 × 5 = 120
```

---

# 23. Identity in `reduce()`

Consider:

```java
numbers.stream()
       .reduce(0, (a, b) -> a + b);
```

The:

```text
0
```

is the **identity value**.

For addition:

```text
0 + x = x
```

For multiplication:

```text
1 × x = x
```

Therefore:

```java
.reduce(0, Integer::sum)
```

is appropriate for addition.

And:

```java
.reduce(1, (a, b) -> a * b)
```

is appropriate for multiplication.

The identity should actually be an identity for the operation.

---

# 24. `reduce()` Without Identity

You can also write:

```java
Optional<Integer> sum =
    numbers.stream()
           .reduce((a, b) -> a + b);
```

Notice the return type:

```text
Optional<Integer>
```

Why?

Because the Stream could be empty.

Therefore:

```text
reduce(identity, ...)
    → T

reduce(...)
    → Optional<T>
```

---

# 25. `map()` vs `reduce()` ⭐⭐⭐

These are very different.

### `map()`

Transforms each element:

```java
.map(x -> x * 2)
```

Conceptually:

```text
[1,2,3]
   ↓
[2,4,6]
```

Many elements remain many elements.

---

### `reduce()`

Combines elements:

```java
.reduce(0, Integer::sum)
```

Conceptually:

```text
[1,2,3]
   ↓
6
```

Many elements become one result.

Remember:

```text
map    → transform each

reduce → combine many into one
```

---

# 26. `collect()` ⭐⭐⭐

`collect()` is one of the most important terminal operations.

Java 8 style:

```java
List<Integer> result =
    numbers.stream()
           .filter(x -> x % 2 == 0)
           .collect(Collectors.toList());
```

Modern Java:

```java
List<Integer> result =
    numbers.stream()
           .filter(x -> x % 2 == 0)
           .toList();
```

`collect()` is much more powerful because it works with the **Collectors API**.

Examples:

```text
Collectors.toList()
Collectors.toSet()
Collectors.toMap()
Collectors.groupingBy()
Collectors.partitioningBy()
Collectors.joining()
```

We will cover these separately.

---

# 27. `toList()` vs `collect(Collectors.toList())`

### Modern Java

```java
stream.toList();
```

### Java 8

```java
stream.collect(Collectors.toList());
```

Important:

```text
Stream.toList()
    → Java 16+

Collectors.toList()
    → available in Java 8
```

Because your interview preparation includes Java 8, you should recognize both.

---

# 28. `forEachOrdered()`

`forEachOrdered()` is another terminal operation.

Example:

```java
numbers.stream()
       .forEachOrdered(System.out::println);
```

For an ordered sequential Stream:

```text
1
2
3
4
5
```

With a parallel ordered Stream:

```java
numbers.parallelStream()
       .forEachOrdered(System.out::println);
```

the terminal action respects encounter order.

---

# 29. `forEach()` vs `forEachOrdered()`

### `forEach()`

```java
numbers.parallelStream()
       .forEach(System.out::println);
```

does not guarantee encounter order.

### `forEachOrdered()`

```java
numbers.parallelStream()
       .forEachOrdered(System.out::println);
```

preserves encounter order when the Stream has one.

Important:

```text
forEachOrdered()
    ≠ sort()

forEachOrdered()
    ≠ thread-safe

forEachOrdered()
    ≠ necessarily sequential
```

A parallel Stream can still perform processing concurrently; maintaining ordered terminal consumption can require additional coordination.

---

# 30. Terminal Operations — Categories

## Iteration / Side Effects

```text
forEach()
forEachOrdered()
```

## Collection

```text
toList()
collect()
```

## Reduction

```text
reduce()
count()
min()
max()
```

## Finding

```text
findFirst()
findAny()
```

## Matching

```text
anyMatch()
allMatch()
noneMatch()
```

---

# 31. Terminal Operation Return Types

| Operation | Return Type |
|---|---|
| `forEach()` | `void` |
| `forEachOrdered()` | `void` |
| `toList()` | `List<T>` |
| `count()` | `long` |
| `min()` | `Optional<T>` |
| `max()` | `Optional<T>` |
| `findFirst()` | `Optional<T>` |
| `findAny()` | `Optional<T>` |
| `anyMatch()` | `boolean` |
| `allMatch()` | `boolean` |
| `noneMatch()` | `boolean` |
| `reduce(identity, ...)` | `T` |
| `reduce(...)` | `Optional<T>` |
| `collect(...)` | Depends on Collector |

---

# 32. One Complete Stream Pipeline

```java
List<Integer> numbers =
    List.of(5, 2, 8, 2, 10, 4, 6);

List<Integer> result =
    numbers.stream()
           .filter(x -> x > 3)
           .distinct()
           .sorted()
           .map(x -> x * 10)
           .limit(3)
           .toList();
```

Conceptually:

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

toList:
RESULT
```

Here:

```java
.toList()
```

is the terminal operation.

---

# 33. Important Interview Questions

## Basic

1. What is a terminal operation?
2. Why is it called terminal?
3. What happens after a terminal operation?
4. Can you reuse a Stream after a terminal operation?
5. What does `forEach()` do?
6. What does `count()` return?
7. What does `findFirst()` return?
8. What does `findAny()` return?
9. What does `reduce()` do?
10. What does `collect()` do?

---

## Important

11. Difference between `forEach()` and `forEachOrdered()`?
12. Difference between `findFirst()` and `findAny()`?
13. Difference between `anyMatch()`, `allMatch()`, and `noneMatch()`?
14. Why do match operations short-circuit?
15. Why does `findFirst()` return `Optional`?
16. Why does `min()` return `Optional`?
17. Why does `reduce()` sometimes return `Optional`?
18. Difference between `reduce()` and `collect()`?
19. Difference between `map()` and `reduce()`?
20. Difference between `Stream.toList()` and `Collectors.toList()`?
21. Why does `count()` return `long`?
22. Does `forEachOrdered()` make a parallel Stream sequential?
23. Does `forEachOrdered()` make your code thread-safe?

---

# 34. Coding Practice — Basic

Given:

```java
List<Integer> numbers =
    List.of(1, 2, 3, 4, 5);
```

### Q1

Print every number using a Stream.

### Q2

Count how many even numbers exist.

Expected:

```text
2
```

### Q3

Find the first number greater than 3.

### Q4

Check whether any number is greater than 10.

### Q5

Check whether all numbers are positive.

### Q6

Check whether none of the numbers are negative.

---

# 35. Coding Practice — Medium

### Q7

Calculate the sum using:

```java
reduce()
```

For:

```text
[1,2,3,4,5]
```

Expected:

```text
15
```

### Q8

Calculate the product using `reduce()`.

Expected:

```text
120
```

### Q9

Find the maximum value using:

```java
max()
```

### Q10

Find the minimum value using:

```java
min()
```

---

# 36. Coding Practice — Interview Style

Given:

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

and:

```java
List<Employee> employees;
```

### Q11

Check whether any employee earns more than ₹20 LPA.

### Q12

Check whether all employees earn more than ₹5 LPA.

### Q13

Find the highest salary.

### Q14

Find the first employee whose salary is greater than ₹10 LPA.

### Q15

Count employees earning more than ₹10 LPA.

---

# 37. Reasoning Practice

### Q16

Why are terminal operations called terminal operations?

### Q17

What happens here?

```java
Stream<Integer> stream =
    numbers.stream();

stream.count();

stream.toList();
```

### Q18

Why does:

```java
findFirst()
```

return:

```java
Optional<T>
```

while:

```java
count()
```

returns:

```java
long
```

### Q19

Why can `anyMatch()` stop before processing every element?

### Q20

Why might `findAny()` be preferable to `findFirst()` in parallel processing when ordering does not matter?

---

# 38. Final Cheat Sheet

```text
TERMINAL OPERATIONS
===================

Iteration:
forEach()
forEachOrdered()

Collection:
toList()
collect()

Reduction:
reduce()
count()
min()
max()

Finding:
findFirst()
findAny()

Matching:
anyMatch()
allMatch()
noneMatch()
```

### Mental Model

```text
forEach
    → do something with every element

forEachOrdered
    → do something with every element in encounter order

toList
    → give me a List

collect
    → accumulate into a chosen result

count
    → how many?

min/max
    → smallest/largest

findFirst
    → give me the first

findAny
    → give me any one

anyMatch
    → is there at least one?

allMatch
    → do all satisfy?

noneMatch
    → does none satisfy?

reduce
    → combine many into one
```

---

# 39. Stream Learning Progress

```text
Stream Fundamentals                 ✅
Stream Source Methods               ✅
Stream Intermediate Operations      ✅
Stream Terminal Operations          ✅
        ↓
Collectors                         ← NEXT
        ↓
Optional
        ↓
Stream Coding Practice
        ↓
Stream Interview Questions
```

**Java 8 — Stream Terminal Operations = DONE ✅**