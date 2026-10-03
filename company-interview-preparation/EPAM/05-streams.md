# PART 5 — STREAMS 🔥🔥🔥🔥

This is the **most important Java 8 section for your EPAM round**.

Your source specifically prioritizes lazy evaluation, `map`, `flatMap`, `filter`, collectors, `groupingBy`, `partitioningBy`, `toMap`, `reduce`, short-circuiting, parallel streams, `Optional`, and Stream coding. Pasted markdown (2)(1)

The goal is not to memorize Stream syntax. You should be able to **look at a problem and construct the pipeline yourself**.

---

# 1. What is a Stream?

### Interview question

> What is a Stream in Java?

### Strong answer

> A Stream is a sequence of elements that supports declarative processing through operations such as filtering, mapping, sorting, aggregation and collection. A Stream does not store data itself; it processes data from a source such as a Collection, array, or generated source.

For example:

```java
List<Integer> numbers =
        Arrays.asList(1, 2, 3, 4, 5);

List<Integer> result =
        numbers.stream()
               .filter(n -> n % 2 == 0)
               .map(n -> n * n)
               .collect(Collectors.toList());
```

Result:

```text
[4, 16]
```

The original list is not modified.

---

# 2. Collection vs Stream 🔥

This is a common question.

### Collection

A Collection:

- stores data
- can generally be traversed multiple times
- represents a data structure

### Stream

A Stream:

- processes data
- is generally consumed once
- supports declarative pipelines
- is lazy for intermediate operations

Think:

```text
Collection
    ↓
DATA

Stream
    ↓
PROCESSING
```

A very good interview sentence:

> A Collection is primarily about storing and managing data, whereas a Stream is about describing a computation over data.

---

# 3. Stream lifecycle

A Stream pipeline generally looks like:

```text
Source
  ↓
Intermediate operations
  ↓
Intermediate operations
  ↓
Terminal operation
```

Example:

```java
numbers.stream()              // source
       .filter(n -> n > 10)   // intermediate
       .map(n -> n * 2)       // intermediate
       .collect(Collectors.toList()); // terminal
```

---

# 4. Intermediate vs Terminal operations 🔥

## Intermediate operations

Return another Stream.

Examples:

```text
filter
map
flatMap
distinct
sorted
peek
limit
skip
```

They are generally **lazy**.

---

## Terminal operations

Produce a final result or side effect.

Examples:

```text
collect
reduce
forEach
count
min
max
findFirst
findAny
anyMatch
allMatch
noneMatch
```

Once the terminal operation executes, the Stream is consumed.

---

# 5. Lazy evaluation 🔥🔥🔥

This is a favorite follow-up.

Consider:

```java
List<Integer> numbers =
        Arrays.asList(1, 2, 3, 4, 5);

numbers.stream()
       .filter(n -> {
           System.out.println("filter: " + n);
           return n > 2;
       });
```

What happens?

**Nothing is printed.**

Why?

Because `filter()` is an intermediate operation and Stream processing is lazy.

Now:

```java
numbers.stream()
       .filter(n -> {
           System.out.println("filter: " + n);
           return n > 2;
       })
       .collect(Collectors.toList());
```

Now execution happens.

### Interview answer

> Intermediate operations build the pipeline but don't normally execute it immediately. A terminal operation triggers traversal of the source and execution of the pipeline.

This lazy behavior is explicitly identified as a P0 Stream concept in your source. Pasted markdown (2)(1)

---

# 6. Why is laziness useful?

Suppose:

```java
numbers.stream()
       .filter(...)
       .map(...)
       .limit(5)
       .collect(...);
```

Because execution is lazy, Stream processing can often avoid processing elements that aren't necessary.

This becomes especially important with:

```text
findFirst()
findAny()
anyMatch()
limit()
```

These can **short-circuit**.

---

# 7. The most important Stream concept: pipeline fusion

Suppose:

```java
numbers.stream()
       .filter(n -> n > 10)
       .map(n -> n * 2)
       .filter(n -> n < 100)
       .collect(Collectors.toList());
```

A beginner may imagine:

```text
filter entire list
      ↓
create new list
      ↓
map entire list
      ↓
create another list
      ↓
filter again
```

That's not the right mental model.

Streams can process elements through the pipeline as they travel through it.

Conceptually:

```text
element
   ↓
filter
   ↓
map
   ↓
filter
   ↓
next element
```

This is one reason Streams can express multi-stage processing efficiently.

---

# 8. `filter()` 🔥

`filter()` takes a Predicate.

```java
List<Integer> evenNumbers =
        numbers.stream()
               .filter(n -> n % 2 == 0)
               .collect(Collectors.toList());
```

Result:

```text
[2, 4]
```

### Important

`filter()` does not modify the source Collection.

---

# 9. `map()` 🔥🔥

`map()` is for **one-to-one transformation**.

```java
List<String> names =
        Arrays.asList("Aryan", "Rahul", "John");

List<Integer> lengths =
        names.stream()
             .map(String::length)
             .collect(Collectors.toList());
```

Result:

```text
[5, 5, 4]
```

Conceptually:

```text
String
   ↓
Function<String,Integer>
   ↓
Integer
```

Your source explicitly describes `map` as a one-to-one transformation. Pasted markdown (2)(1)

---

# 10. `map()` does NOT flatten

This leads directly to `flatMap()`.

Suppose:

```java
List<List<Integer>> numbers =
        Arrays.asList(
            Arrays.asList(1, 2),
            Arrays.asList(3, 4),
            Arrays.asList(5, 6)
        );
```

If you do:

```java
numbers.stream()
       .map(List::stream)
```

you get:

```text
Stream<Stream<Integer>>
```

You haven't flattened anything.

---

# 11. `flatMap()` 🔥🔥🔥

`flatMap()` performs:

> one-to-many transformation + flattening.

```java
List<Integer> result =
        numbers.stream()
               .flatMap(List::stream)
               .collect(Collectors.toList());
```

Result:

```text
[1, 2, 3, 4, 5, 6]
```

Mental model:

```text
map:

[A, B, C]
   ↓
[Stream, Stream, Stream]


flatMap:

[A, B, C]
   ↓
A B C
   ↓
single Stream
```

Your source explicitly identifies `map vs flatMap` as a P0 SDE-2 concept and uses nested collection flattening as the coding scenario. Pasted markdown (2)(1)

---

# 12. The interview question: map vs flatMap

### Strong answer

> `map` transforms each element independently and produces one output element per input element. `flatMap` is useful when each input element produces multiple elements or another Stream, and it flattens those nested Streams into a single Stream.

Example:

```java
List<List<String>> accounts;
```

If each account has transactions:

```text
Account 1 → [T1,T2]
Account 2 → [T3,T4]
Account 3 → [T5]
```

`flatMap()` gives:

```text
T1,T2,T3,T4,T5
```

This is a **very practical backend scenario**.

---

# 13. `distinct()`

Removes duplicates according to equality semantics.

```java
List<Integer> result =
        numbers.stream()
               .distinct()
               .collect(Collectors.toList());
```

Input:

```text
[1,2,2,3,3,3,4]
```

Output:

```text
[1,2,3,4]
```

### Important interview point

For objects, `distinct()` depends on:

```text
equals()
hashCode()
```

So your Part 1 knowledge comes back here.

---

# 14. `sorted()`

Natural ordering:

```java
List<Integer> result =
        numbers.stream()
               .sorted()
               .collect(Collectors.toList());
```

Custom ordering:

```java
employees.stream()
         .sorted(
             Comparator.comparingInt(Employee::getSalary)
                       .reversed()
         )
```

### Important deeper point

`sorted()` is a **stateful** operation.

Why?

It generally needs to see the relevant elements before producing the sorted result.

For a huge stream:

```text
millions of records
        ↓
sorted()
        ↓
memory / processing cost
```

Your source specifically calls out the performance implications of `sorted()` on huge streams. Pasted markdown (2)(1)

---

# 15. `limit()` and `skip()`

```java
numbers.stream()
       .limit(5)
```

takes at most 5 elements.

```java
numbers.stream()
       .skip(5)
```

ignores the first 5.

Example:

```java
Arrays.asList(1,2,3,4,5,6)
       .stream()
       .skip(2)
       .limit(3)
```

Result:

```text
[3,4,5]
```

---

# 16. Important backend trap: `skip/limit` is NOT database pagination

Suppose your DB has:

```text
100 million records
```

and you do:

```java
repository.findAll()
          .stream()
          .skip(50_000_000)
          .limit(100);
```

You've potentially already loaded/processed an enormous amount of data.

For database pagination, use:

- DB `LIMIT/OFFSET`
- keyset pagination
- Spring Data pagination
- appropriate indexes

Your source specifically calls out the distinction between Stream `limit/skip` and actual database pagination. Pasted markdown (2)(1)

---

# 17. `peek()` ⚠️

`peek()` is mainly intended for debugging/observing a pipeline.

```java
numbers.stream()
       .filter(n -> n > 2)
       .peek(n -> System.out.println("After filter: " + n))
       .map(n -> n * 2)
       .collect(Collectors.toList());
```

### Don't use it for business logic.

Bad:

```java
stream.peek(order -> order.setStatus("PAID"))
```

Why?

Because:

- lazy execution
- side effects
- difficult reasoning
- debugging complexity

Your source explicitly flags `peek` as a lower-priority operation and warns against using it as business logic. Pasted markdown (2)

---

# 18. `reduce()` 🔥

`reduce()` combines multiple elements into one result.

Example:

```java
List<Integer> numbers =
        Arrays.asList(1, 2, 3, 4);

int sum =
        numbers.stream()
               .reduce(0, Integer::sum);
```

Result:

```text
10
```

Conceptually:

```text
0 + 1
  + 2
  + 3
  + 4
= 10
```

---

# 19. Three important reduce concepts

### Identity

Initial value:

```java
0
```

### Accumulator

How the current result combines with an element:

```java
Integer::sum
```

### Combiner

Important particularly for parallel reduction.

Conceptually:

```java
reduce(
    identity,
    accumulator,
    combiner
)
```

Example:

```java
int sum =
    numbers.parallelStream()
           .reduce(
               0,
               Integer::sum,
               Integer::sum
           );
```

---

# 20. Why must parallel reduction be associative?

This is a deeper interview question.

Suppose:

```text
[a,b,c,d]
```

Parallel processing might produce:

```text
(a + b) + (c + d)
```

instead of:

```text
((a + b) + c) + d
```

For addition:

```text
(a+b)+(c+d)
```

and:

```text
((a+b)+c)+d
```

produce the same result.

Therefore addition is associative.

But if your reduction operation depends on ordering or has incompatible side effects, parallel execution can produce incorrect results.

### Interview sentence

> A reduction intended for parallel execution should use an associative, compatible accumulation strategy so that partial results can be safely combined.

---

# 21. `reduce()` vs `collect()` 🔥

This is a common question.

### `reduce()`

Usually produces a single combined value:

```java
int sum =
    numbers.stream()
           .reduce(0, Integer::sum);
```

### `collect()`

Usually accumulates into a result container:

```java
List<Integer> result =
    numbers.stream()
           .filter(n -> n > 10)
           .collect(Collectors.toList());
```

Your source explicitly distinguishes `reduce` as combining values and `collect` as accumulating into mutable result structures. Pasted text (2)

---

# 22. Primitive Streams 🔥

Don't unnecessarily box primitives.

Instead of:

```java
int sum =
    numbers.stream()
           .reduce(0, Integer::sum);
```

you can use:

```java
int sum =
    numbers.stream()
           .mapToInt(Integer::intValue)
           .sum();
```

This uses:

```text
IntStream
```

Other primitive streams:

```text
IntStream
LongStream
DoubleStream
```

This can avoid unnecessary boxing/unboxing.

---

# 23. `collect()` 🔥🔥

One of the most important terminal operations.

```java
List<Integer> result =
        numbers.stream()
               .filter(n -> n % 2 == 0)
               .collect(Collectors.toList());
```

Common collectors:

```text
toList
toSet
toMap
groupingBy
partitioningBy
joining
counting
summingInt
averagingInt
maxBy
minBy
mapping
```

---

# 24. `groupingBy()` 🔥🔥🔥

This is **must-know Stream coding**.

Suppose:

```java
class Employee {
    String name;
    String department;
    int salary;

    // getters
}
```

Group employees by department:

```java
Map<String, List<Employee>> result =
        employees.stream()
                 .collect(
                     Collectors.groupingBy(
                         Employee::getDepartment
                     )
                 );
```

Conceptually:

```text
IT
 → [Aryan, Rahul]

HR
 → [John, Mike]

Finance
 → [Sara]
```

Your source marks `groupingBy` as P0 and specifically asks for account/transaction grouping scenarios. Pasted markdown (2)(1)

---

# 25. `groupingBy()` + counting

Question:

> Count employees in each department.

```java
Map<String, Long> result =
        employees.stream()
                 .collect(
                     Collectors.groupingBy(
                         Employee::getDepartment,
                         Collectors.counting()
                     )
                 );
```

Result:

```text
IT       → 5
HR       → 3
Finance  → 4
```

---

# 26. `groupingBy()` + averaging

Average salary by department:

```java
Map<String, Double> result =
        employees.stream()
                 .collect(
                     Collectors.groupingBy(
                         Employee::getDepartment,
                         Collectors.averagingInt(
                             Employee::getSalary
                         )
                     )
                 );
```

This is a very common interview coding pattern.

---

# 27. `groupingBy()` + summing

Total salary per department:

```java
Map<String, Integer> result =
        employees.stream()
                 .collect(
                     Collectors.groupingBy(
                         Employee::getDepartment,
                         Collectors.summingInt(
                             Employee::getSalary
                         )
                     )
                 );
```

Think:

```text
groupingBy
    +
downstream collector
```

This pattern is extremely important.

---

# 28. `partitioningBy()` 🔥

`partitioningBy()` is for splitting into two groups based on a boolean condition.

Example:

```java
Map<Boolean, List<Employee>> result =
        employees.stream()
                 .collect(
                     Collectors.partitioningBy(
                         e -> e.getSalary() > 100000
                     )
                 );
```

Result:

```text
true  → salary > 100000
false → salary <= 100000
```

### Difference

```text
groupingBy
→ potentially many groups

partitioningBy
→ true / false
```

Your source explicitly highlights this distinction. Pasted markdown (2)(1)

---

# 29. `toMap()` — THE FAMOUS TRAP 🔥🔥

Suppose:

```java
Map<String, String> result =
        employees.stream()
                 .collect(
                     Collectors.toMap(
                         Employee::getDepartment,
                         Employee::getName
                     )
                 );
```

What if:

```text
IT → Aryan
IT → Rahul
```

Now there are duplicate keys.

You'll get:

```text
IllegalStateException
```

because `toMap()` needs a strategy for resolving duplicate keys.

---

# 30. `toMap()` with merge function

Keep the first:

```java
Map<String, String> result =
        employees.stream()
                 .collect(
                     Collectors.toMap(
                         Employee::getDepartment,
                         Employee::getName,
                         (existing, replacement) -> existing
                     )
                 );
```

Keep the second:

```java
(existing, replacement) -> replacement
```

This is **very important interview knowledge**. Your source explicitly flags duplicate-key handling as a P0 concept. Pasted text (2)

---

# 31. `joining()`

Given:

```java
List<String> names =
        Arrays.asList("Aryan", "Rahul", "John");
```

Do:

```java
String result =
        names.stream()
             .collect(Collectors.joining(", "));
```

Result:

```text
Aryan, Rahul, John
```

You can also specify prefix/suffix:

```java
String result =
        names.stream()
             .collect(
                 Collectors.joining(
                     ", ",
                     "[",
                     "]"
                 )
             );
```

Result:

```text
[Aryan, Rahul, John]
```

---

# 32. `max()` / `min()`

Highest-paid employee:

```java
Employee highest =
        employees.stream()
                 .max(
                     Comparator.comparingInt(
                         Employee::getSalary
                     )
                 )
                 .orElse(null);
```

Lowest:

```java
Employee lowest =
        employees.stream()
                 .min(
                     Comparator.comparingInt(
                         Employee::getSalary
                     )
                 )
                 .orElse(null);
```

---

# 33. `maxBy()` / `minBy()`

These are downstream collectors.

Highest salary per department:

```java
Map<String, Optional<Employee>> result =
        employees.stream()
                 .collect(
                     Collectors.groupingBy(
                         Employee::getDepartment,
                         Collectors.maxBy(
                             Comparator.comparingInt(
                                 Employee::getSalary
                             )
                         )
                     )
                 );
```

This is a **high-value EPAM-style problem**.

Your source contains exactly this problem. Pasted text(20260930-224841)

---

# 34. Removing the Optional

If you need:

```text
Map<String, Employee>
```

rather than:

```text
Map<String, Optional<Employee>>
```

you can use `collectingAndThen()`:

```java
Map<String, Employee> result =
        employees.stream()
                 .collect(
                     Collectors.groupingBy(
                         Employee::getDepartment,
                         Collectors.collectingAndThen(
                             Collectors.maxBy(
                                 Comparator.comparingInt(
                                     Employee::getSalary
                                 )
                             ),
                             Optional::orElseThrow
                         )
                     )
                 );
```

### Important clarification

If a department could theoretically have no employees, `orElseThrow()` must be considered appropriately. In normal `groupingBy`, groups arise from existing employees, so this generally isn't an issue.

---

# 35. `findFirst()` vs `findAny()`

Both return:

```java
Optional<T>
```

### `findFirst()`

Returns the first element according to encounter order when applicable.

```java
Optional<Integer> result =
        numbers.stream()
               .filter(n -> n > 10)
               .findFirst();
```

### `findAny()`

Returns any matching element.

This can be more flexible for parallel processing.

---

# 36. Match operations 🔥

### `anyMatch`

```java
boolean exists =
        numbers.stream()
               .anyMatch(n -> n > 100);
```

Stops as soon as it finds a match.

### `allMatch`

```java
boolean allPositive =
        numbers.stream()
               .allMatch(n -> n > 0);
```

Can stop when it finds a violation.

### `noneMatch`

```java
boolean noneNegative =
        numbers.stream()
               .noneMatch(n -> n < 0);
```

These are **short-circuiting terminal operations**. Your source specifically highlights them. Pasted markdown (2)(1)

---

# 37. Why `anyMatch()` can be better than filter + count

Bad approach:

```java
boolean exists =
        numbers.stream()
               .filter(n -> n > 100)
               .count() > 0;
```

This may process more elements than necessary.

Better:

```java
boolean exists =
        numbers.stream()
               .anyMatch(n -> n > 100);
```

Once a match is found:

```text
STOP
```

---

# 38. Stream reuse 🔥

This is a classic trap.

```java
Stream<Integer> stream =
        numbers.stream();

stream.count();

stream.forEach(System.out::println);
```

The second operation fails because the Stream has already been consumed.

You'll typically get:

```text
IllegalStateException
```

### Correct

Create another Stream:

```java
numbers.stream().count();

numbers.stream().forEach(System.out::println);
```

### Interview answer

> A Stream represents a consumable pipeline and generally cannot be reused after a terminal operation.

Your source explicitly lists Stream reuse as a P0 concept. Pasted markdown (2)

---

# 39. Stream does not modify the source

Example:

```java
List<Integer> numbers =
        Arrays.asList(1, 2, 3, 4);

List<Integer> even =
        numbers.stream()
               .filter(n -> n % 2 == 0)
               .collect(Collectors.toList());
```

`numbers` remains:

```text
[1,2,3,4]
```

and:

```text
even = [2,4]
```

Unless your lambda explicitly mutates an object, the Stream operations themselves don't mutate the source collection.

---

# 40. Side effects ⚠️

Avoid:

```java
List<Integer> result = new ArrayList<>();

numbers.stream()
       .filter(n -> n % 2 == 0)
       .forEach(result::add);
```

It can work sequentially, but you're introducing external mutable state.

Worse:

```java
List<Integer> result =
        new ArrayList<>();

numbers.parallelStream()
       .forEach(result::add);
```

Now you have a concurrency problem.

Prefer:

```java
List<Integer> result =
        numbers.stream()
               .filter(n -> n % 2 == 0)
               .collect(Collectors.toList());
```

### Interview sentence

> Stream pipelines are easier to reason about when operations are stateless and avoid external mutable state.

---

# 41. Parallel Streams 🔥

```java
numbers.parallelStream()
```

can process work in parallel.

But:

> Parallel Stream does NOT automatically mean faster.

Potential overhead:

```text
task splitting
thread scheduling
coordination
combining results
context switching
```

For small collections:

```text
parallel overhead > computation benefit
```

---

# 42. Which thread pool does parallel Stream use?

By default, parallel streams use the common:

```text
ForkJoinPool
```

specifically the common pool.

This is important for backend applications.

---

# 43. Why can parallelStream() be dangerous in backend code?

Suppose your Stream performs blocking I/O:

```java
orders.parallelStream()
      .map(order -> callExternalService(order))
```

Now common-pool worker threads can become blocked waiting for network calls.

This can affect unrelated parallel-stream workloads sharing the pool.

### Strong interview answer

> Parallel streams are most appropriate for sufficiently large, CPU-bound, independent workloads. I would be cautious with blocking I/O, shared mutable state, small datasets, and latency-sensitive backend code.

Your source explicitly highlights common ForkJoinPool usage, work splitting, overhead, shared state, and blocking risks. Pasted markdown (2)

---

# 44. The Stream coding patterns you MUST memorize

Now let's get to the part most relevant to your EPAM round.

---

## Coding 1 — Even numbers

```java
List<Integer> result =
        numbers.stream()
               .filter(n -> n % 2 == 0)
               .collect(Collectors.toList());
```

---

## Coding 2 — Squares of even numbers

```java
List<Integer> result =
        numbers.stream()
               .filter(n -> n % 2 == 0)
               .map(n -> n * n)
               .collect(Collectors.toList());
```

Pipeline:

```text
filter
  ↓
map
  ↓
collect
```

---

# Coding 3 — Sort employees by ID

Ascending:

```java
List<Employee> result =
        employees.stream()
                 .sorted(
                     Comparator.comparingInt(
                         Employee::getId
                     )
                 )
                 .collect(Collectors.toList());
```

Descending:

```java
List<Employee> result =
        employees.stream()
                 .sorted(
                     Comparator.comparingInt(
                         Employee::getId
                     )
                     .reversed()
                 )
                 .collect(Collectors.toList());
```

---

# Coding 4 — Highest-paid employee

```java
Employee result =
        employees.stream()
                 .max(
                     Comparator.comparingInt(
                         Employee::getSalary
                     )
                 )
                 .orElse(null);
```

---

# Coding 5 — Second-highest salary

This is a very common interview variation.

```java
Optional<Integer> secondHighest =
        employees.stream()
                 .map(Employee::getSalary)
                 .distinct()
                 .sorted(Comparator.reverseOrder())
                 .skip(1)
                 .findFirst();
```

Why `distinct()`?

Suppose:

```text
100000
90000
90000
80000
```

Without distinct:

```text
highest = 100000
second = 90000
```

That's fine here, but if the interviewer asks for the **second distinct highest salary**, `distinct()` makes the intent explicit.

---

# Coding 6 — Group by department

```java
Map<String, List<Employee>> result =
        employees.stream()
                 .collect(
                     Collectors.groupingBy(
                         Employee::getDepartment
                     )
                 );
```

---

# Coding 7 — Count employees per department

```java
Map<String, Long> result =
        employees.stream()
                 .collect(
                     Collectors.groupingBy(
                         Employee::getDepartment,
                         Collectors.counting()
                     )
                 );
```

---

# Coding 8 — Average salary per department

```java
Map<String, Double> result =
        employees.stream()
                 .collect(
                     Collectors.groupingBy(
                         Employee::getDepartment,
                         Collectors.averagingInt(
                             Employee::getSalary
                         )
                     )
                 );
```

---

# Coding 9 — Highest salary per department 🔥🔥

```java
Map<String, Optional<Employee>> result =
        employees.stream()
                 .collect(
                     Collectors.groupingBy(
                         Employee::getDepartment,
                         Collectors.maxBy(
                             Comparator.comparingInt(
                                 Employee::getSalary
                             )
                         )
                     )
                 );
```

This is exactly the kind of nested collector problem you need to be comfortable with. Pasted text(20260930-224841)

---

# Coding 10 — Flatten nested lists 🔥

Input:

```java
List<List<Integer>> input =
        Arrays.asList(
            Arrays.asList(1, 2),
            Arrays.asList(3, 4),
            Arrays.asList(5, 6)
        );
```

Solution:

```java
List<Integer> result =
        input.stream()
             .flatMap(List::stream)
             .collect(Collectors.toList());
```

Output:

```text
[1,2,3,4,5,6]
```

---

# Coding 11 — Word frequency

Input:

```text
"java spring java kafka spring java"
```

```java
Map<String, Long> frequency =
        Arrays.stream(input.split("\\s+"))
              .collect(
                  Collectors.groupingBy(
                      Function.identity(),
                      Collectors.counting()
                  )
              );
```

Result:

```text
java   → 3
spring → 2
kafka  → 1
```

This is a **must-practice pattern**.

---

# Coding 12 — First non-repeating character 🔥🔥🔥

Input:

```text
swiss
```

Output:

```text
w
```

A clean Stream solution:

```java
Character result =
        input.chars()
             .mapToObj(c -> (char) c)
             .collect(
                 Collectors.groupingBy(
                     Function.identity(),
                     LinkedHashMap::new,
                     Collectors.counting()
                 )
             )
             .entrySet()
             .stream()
             .filter(entry -> entry.getValue() == 1)
             .map(Map.Entry::getKey)
             .findFirst()
             .orElse(null);
```

### Why `LinkedHashMap`?

This is the important part.

`groupingBy()` with a normal HashMap doesn't guarantee encounter order.

We need:

```text
frequency + original order
```

So:

```java
LinkedHashMap::new
```

is used.

This exact pattern exists in your source. Pasted text(20260930-224841)

---

# Coding 13 — Find duplicate elements

```java
Set<Integer> seen = new HashSet<>();

Set<Integer> duplicates =
        numbers.stream()
               .filter(n -> !seen.add(n))
               .collect(Collectors.toSet());
```

For:

```text
[1,2,3,4,2,5,1]
```

result:

```text
[1,2]
```

The source includes this exact pattern. Pasted markdown(20260830-173503)

### Interview caveat

This uses external mutable state (`seen`), so it isn't a pure/stateless Stream pipeline. It's acceptable as a coding solution in some interviews, but if discussing production-quality parallel streams, call out that this approach is unsuitable for parallel execution without proper concurrency handling.

---

# Coding 14 — Convert strings to uppercase

```java
List<String> result =
        names.stream()
             .map(String::toUpperCase)
             .collect(Collectors.toList());
```

This is a textbook method-reference + map question. Pasted markdown(20260830-173503)

---

# Coding 15 — Sum numbers

```java
int sum =
        numbers.stream()
               .mapToInt(Integer::intValue)
               .sum();
```

For:

```text
[1,2,3,4,5]
```

result:

```text
15
```

The source uses this exact primitive-stream approach. Pasted markdown(20260830-173503)

---

# Coding 16 — Check whether any value satisfies a condition

```java
boolean exists =
        numbers.stream()
               .anyMatch(n -> n > 100);
```

Remember:

```text
anyMatch → at least one
allMatch → every element
noneMatch → no elements
```

---

# Coding 17 — Group strings by length

```java
Map<Integer, List<String>> result =
        words.stream()
             .collect(
                 Collectors.groupingBy(
                     String::length
                 )
             );
```

Example:

```text
Java   → length 4
Stream → length 6
API    → length 3
```

Again, this is directly represented in the source coding material. Pasted markdown(20260830-173503)

---

# 45. A VERY important EPAM coding strategy

Suppose they give you:

> "Given employees, print employee names whose salary is greater than 100000, sorted by salary descending."

Don't panic.

Break it into English:

```text
employees
   ↓
salary > 100000
   ↓
sort descending
   ↓
get name
   ↓
collect
```

Then translate:

```java
List<String> result =
        employees.stream()
                 .filter(e -> e.getSalary() > 100000)
                 .sorted(
                     Comparator.comparingInt(
                         Employee::getSalary
                     ).reversed()
                 )
                 .map(Employee::getName)
                 .collect(Collectors.toList());
```

This is the skill you need.

---

# 46. Think in pipeline verbs

When reading a problem, identify these words:

| Problem says | Think |
|---|---|
| only employees with... | `filter` |
| convert/extract | `map` |
| nested lists | `flatMap` |
| remove duplicates | `distinct` |
| order/sort | `sorted` |
| first N | `limit` |
| ignore first N | `skip` |
| total | `sum` / `reduce` |
| group by | `groupingBy` |
| split into yes/no | `partitioningBy` |
| create Map | `toMap` |
| concatenate | `joining` |
| highest/lowest | `max` / `min` |
| check at least one | `anyMatch` |
| check all | `allMatch` |
| check none | `noneMatch` |
| first matching | `findFirst` |

This is **far more useful** than memorizing 30 Stream examples.

---

# 47. Optional 🔥

Streams frequently return Optional.

For example:

```java
Optional<Employee> highest =
        employees.stream()
                 .max(
                     Comparator.comparingInt(
                         Employee::getSalary
                     )
                 );
```

Why?

Because there might be no employee.

Instead of returning `null`, Java can explicitly represent:

```text
value exists
```

or:

```text
value absent
```

---

# 48. `Optional.of()` vs `ofNullable()`

```java
Optional.of(value)
```

requires non-null value.

If:

```java
value == null
```

it throws `NullPointerException`.

Use:

```java
Optional.ofNullable(value)
```

if the value may be null.

```java
Optional.empty()
```

represents absence explicitly.

---

# 49. `orElse()` vs `orElseGet()` 🔥

This is a common trap.

```java
optional.orElse(expensiveOperation());
```

The argument can be evaluated even when the Optional already contains a value.

Whereas:

```java
optional.orElseGet(
    () -> expensiveOperation()
);
```

supplies the fallback lazily.

### Think:

```text
orElse
→ eager fallback expression

orElseGet
→ Supplier, lazy fallback computation
```

This connects directly to the `Supplier` section from Part 4.

---

# 50. `Optional.map()` vs `Optional.flatMap()`

Suppose:

```java
Optional<String> name =
        Optional.of("Aryan");
```

You can:

```java
Optional<Integer> length =
        name.map(String::length);
```

Result:

```text
Optional[5]
```

But suppose your mapper already returns Optional:

```java
Optional<Address> findAddress(String name)
```

If you use:

```java
Optional<Optional<Address>>
```

you've created nesting.

Instead:

```java
name.flatMap(this::findAddress)
```

produces:

```text
Optional<Address>
```

Same conceptual reason as Stream `map` vs `flatMap`:

```text
map
→ transformation

flatMap
→ transformation + flattening
```

Your source specifically includes Optional `map` vs `flatMap` as a key follow-up. Pasted markdown (2)

---

# 51. Optional `get()` trap

This:

```java
optional.get();
```

can throw:

```text
NoSuchElementException
```

if empty.

Prefer:

```java
optional.orElse(null)
```

or:

```java
optional.orElseThrow(...)
```

depending on the desired semantics.

---

# 🔥 52. EPAM-style coding problem

Let's combine almost everything.

### Problem

Given employees:

```text
Employee(name, department, salary)
```

Find the **highest-paid employee from each department**.

### Solution

```java
Map<String, Optional<Employee>> result =
        employees.stream()
                 .collect(
                     Collectors.groupingBy(
                         Employee::getDepartment,
                         Collectors.maxBy(
                             Comparator.comparingInt(
                                 Employee::getSalary
                             )
                         )
                     )
                 );
```

### Explain it aloud:

> "I'm grouping employees by department. For each department, the downstream collector finds the employee with the maximum salary using `maxBy` and a salary comparator. Since `maxBy` returns an Optional, the resulting map has `Optional<Employee>` values."

That's an **excellent interview explanation**.

---

# 53. Another EPAM-style problem

### Problem

Given:

```java
List<List<String>> accountTransactions
```

Return all unique transaction IDs.

Solution:

```java
List<String> result =
        accountTransactions.stream()
                           .flatMap(List::stream)
                           .distinct()
                           .collect(Collectors.toList());
```

Translate the requirement:

```text
nested collections
       ↓
flatMap

unique
       ↓
distinct

return List
       ↓
collect(toList)
```

That's how you should construct Stream solutions.

---

# 54. The Stream traps you MUST know

### Trap 1

```java
stream.filter(...)
```

doesn't execute immediately.

### Trap 2

A Stream generally cannot be reused after a terminal operation.

### Trap 3

`map()` doesn't flatten.

### Trap 4

`flatMap()` is used for nested/one-to-many transformation.

### Trap 5

`toMap()` can throw `IllegalStateException` for duplicate keys unless you provide a merge function.

### Trap 6

`parallelStream()` isn't automatically faster.

### Trap 7

`peek()` isn't intended for business logic.

### Trap 8

`skip/limit` isn't a replacement for DB pagination.

### Trap 9

`distinct()` relies on equality semantics.

### Trap 10

`sorted()` can be expensive for large datasets.

### Trap 11

External mutable state inside Stream operations can cause correctness/concurrency problems.

---

# 55. The complete Stream mental model

You want this in your head:

```text
                         STREAM
                            |
                ┌───────────┴───────────┐
                ↓                       ↓
            Intermediate             Terminal
                |                       |
        ┌───────┼────────┐       ┌──────┼─────────┐
        ↓       ↓        ↓       ↓      ↓         ↓
     filter    map    flatMap   collect reduce   match
     sorted  distinct  limit    count   max      find
      skip    peek
                |
                ↓
             LAZY
                |
                ↓
        terminal triggers
           execution
```

Then collectors:

```text
collect()
   |
   ├── toList
   ├── toSet
   ├── toMap
   ├── groupingBy
   ├── partitioningBy
   ├── joining
   ├── counting
   ├── summing
   ├── averaging
   ├── maxBy
   └── minBy
```

---

# 🎯 The 10 Stream questions I would expect you to answer confidently

1. **What is a Stream?**
2. **Collection vs Stream?**
3. **Intermediate vs terminal operation?**
4. **Why are intermediate operations lazy?**
5. **`map` vs `flatMap`?**
6. **`reduce` vs `collect`?**
7. **`groupingBy` vs `partitioningBy`?**
8. **What happens with duplicate keys in `toMap`?**
9. **Why can `parallelStream()` be dangerous/slower?**
10. **Can you write employee grouping/aggregation problems using Streams?**

If you can do those plus the coding patterns above, your Stream foundation is strong.

### One final thing

Your source's broader Java 8 checklist also includes deeper internals such as **Spliterator, Sink, `tryAdvance`, `trySplit`, ForkJoinPool/work stealing, primitive streams, Stream performance, tricky output questions, and production scenarios**. Pasted markdown(20260919-020506)

I would **not** dump all of those into this first Stream pass. For your 45-minute EPAM round, the coding/API layer above is the priority. We'll revisit the deeper internals as interview follow-ups.

**Next: PART 6 — Exceptions 🔥**

We'll cover the hierarchy, checked vs unchecked, `throw`/`throws`, propagation, custom exceptions, overriding rules, try-with-resources, suppressed exceptions, and the nasty `finally`/return traps—with coding questions rather than just definitions.