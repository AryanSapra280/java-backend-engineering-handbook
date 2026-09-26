Yes 😄 — **I overdid it.** You asked to continue the interview prep, not start a separate research project. We don't need Deep Research for this. I'll keep going directly, and I'll only distinguish TCS-reported questions when I have a source.

# Next Topic: Java 8 — Deep Interview Q&A

Recent TCS interview reports specifically mention **Java 8 features, functional interfaces, lambdas, Streams, `Comparable`/`Comparator`, and Stream coding problems**, so this section is highly relevant rather than random Java trivia.

---

## 1. What are the major features introduced in Java 8?

### Interview answer

The major Java 8 features are:

1. **Lambda expressions**
2. **Functional interfaces**
3. **Stream API**
4. **Method references**
5. **Default and static methods in interfaces**
6. **Optional**
7. **New Date/Time API — `java.time`**
8. **CompletableFuture**
9. **Collectors API**
10. Improvements to `Map`, `Collection`, etc.

The biggest change was that Java became much more friendly toward **functional-style programming**, while remaining object-oriented.

Example:

```java
List<String> names = List.of("John", "David", "Alex");

names.stream()
     .filter(name -> name.startsWith("A"))
     .forEach(System.out::println);
```

---

# 2. What is a functional interface?

A **functional interface** is an interface containing **exactly one abstract method**.

It can have:

* one abstract method
* multiple `default` methods
* multiple `static` methods

Example:

```java
@FunctionalInterface
interface Calculator {
    int calculate(int a, int b);
}
```

Usage:

```java
Calculator add = (a, b) -> a + b;

System.out.println(add.calculate(10, 20));
```

### Important trap

This is valid:

```java
@FunctionalInterface
interface Test {
    void execute();

    default void log() {
        System.out.println("log");
    }

    static void print() {
        System.out.println("print");
    }
}
```

Because `log()` and `print()` aren't abstract methods.

### Common built-in functional interfaces

| Interface           | Method            | Purpose   |
| ------------------- | ----------------- | --------- |
| `Predicate<T>`      | `boolean test(T)` | condition |
| `Function<T,R>`     | `R apply(T)`      | transform |
| `Consumer<T>`       | `void accept(T)`  | consume   |
| `Supplier<T>`       | `T get()`         | supply    |
| `UnaryOperator<T>`  | `T apply(T)`      | T → T     |
| `BinaryOperator<T>` | `T apply(T,T)`    | T,T → T   |

---

# 3. Lambda expression — what actually happens?

Consider:

```java
(a, b) -> a + b
```

A lambda isn't itself a functional interface.

It is an implementation of the **target functional interface**.

```java
Calculator c = (a, b) -> a + b;
```

The compiler knows the target type is `Calculator`, so it knows the lambda must implement:

```java
int calculate(int a, int b);
```

### Follow-up: Can a lambda exist without a functional interface?

Not as a standalone value.

This doesn't make sense:

```java
var x = (a, b) -> a + b; // invalid
```

The compiler needs a target functional-interface type.

---

# 4. What is the difference between lambda and anonymous inner class?

### Anonymous class

```java
Runnable r = new Runnable() {
    @Override
    public void run() {
        System.out.println("Hello");
    }
};
```

### Lambda

```java
Runnable r = () -> System.out.println("Hello");
```

Lambda is more concise and is primarily intended for functional interfaces.

### Important interview trap: `this`

Inside an anonymous class:

```java
this
```

refers to the anonymous-class object.

Inside a lambda:

```java
this
```

refers to the enclosing object.

That's a very good follow-up question.

---

# 5. What is a method reference?

Method reference is shorthand for a lambda when you're simply calling an existing method.

Instead of:

```java
names.forEach(name -> System.out.println(name));
```

you can write:

```java
names.forEach(System.out::println);
```

Types:

```java
ClassName::staticMethod
object::instanceMethod
ClassName::instanceMethod
ClassName::new
```

Example constructor reference:

```java
Supplier<List<String>> supplier = ArrayList::new;
```

Equivalent conceptually to:

```java
Supplier<List<String>> supplier = () -> new ArrayList<>();
```

---

# 6. What is Stream API?

This is **very important**.

A Stream is a pipeline for processing data.

It doesn't store data itself.

Example:

```java
List<Integer> numbers = List.of(1, 2, 3, 4, 5, 6);

List<Integer> result =
        numbers.stream()
               .filter(n -> n % 2 == 0)
               .map(n -> n * 10)
               .toList();
```

Conceptually:

```text
Collection
   ↓
stream()
   ↓
filter()
   ↓
map()
   ↓
toList()
```

---

# 7. Collection vs Stream?

This is a classic question.

### Collection

A Collection is a **data structure that stores data**.

```java
List<Integer> numbers = ...
```

### Stream

A Stream is a **pipeline for processing data**.

```java
numbers.stream()
       .filter(...)
       .map(...)
```

A Stream generally doesn't modify the original collection.

```java
List<Integer> numbers = List.of(1, 2, 3);

List<Integer> result =
    numbers.stream()
           .map(x -> x * 2)
           .toList();
```

`numbers` remains unchanged.

---

# 8. What are intermediate and terminal operations?

Extremely important.

### Intermediate operations

They return another Stream.

Examples:

```java
filter()
map()
flatMap()
distinct()
sorted()
limit()
skip()
peek()
```

Example:

```java
stream
    .filter(...)
    .map(...)
    .sorted();
```

### Terminal operations

They produce the final result or side effect.

Examples:

```java
collect()
toList()
forEach()
reduce()
count()
min()
max()
findFirst()
findAny()
anyMatch()
allMatch()
noneMatch()
```

Example:

```java
long count =
    numbers.stream()
           .filter(n -> n > 10)
           .count();
```

---

# 9. What does lazy evaluation mean in Streams?

This is a **very common senior-level follow-up**.

Intermediate operations aren't normally executed immediately.

Example:

```java
Stream<Integer> stream =
    numbers.stream()
           .filter(n -> {
               System.out.println("filter " + n);
               return n > 3;
           });
```

At this point, filtering hasn't actually processed the elements.

Execution starts when you call a terminal operation:

```java
stream.count();
```

### Why lazy?

Because Java can optimize the pipeline and avoid unnecessary work.

For example:

```java
numbers.stream()
       .filter(...)
       .map(...)
       .findFirst();
```

It doesn't necessarily process the entire collection.

It can stop once `findFirst()` has its answer.

---

# 10. `map()` vs `flatMap()`

**Very important.**

### `map`

One input → one output.

```java
List<String> names = List.of("John", "Alex");

List<Integer> lengths =
    names.stream()
         .map(String::length)
         .toList();
```

Result:

```text
[4, 4]
```

---

### `flatMap`

Used when each input produces another collection/stream and you want to flatten them.

```java
List<List<Integer>> numbers =
    List.of(
        List.of(1, 2),
        List.of(3, 4),
        List.of(5, 6)
    );
```

Using `map`:

```java
numbers.stream()
       .map(list -> list.stream())
```

You get:

```text
Stream<Stream<Integer>>
```

Using `flatMap`:

```java
numbers.stream()
       .flatMap(List::stream)
       .toList();
```

Result:

```text
[1, 2, 3, 4, 5, 6]
```

### Interview sentence

> `map` transforms each element independently, while `flatMap` transforms and flattens nested structures into a single stream.

---

# 11. `filter()` vs `map()`

Easy but often asked.

`filter` decides **whether an element remains**.

```java
.filter(x -> x > 10)
```

`map` decides **what the element becomes**.

```java
.map(x -> x * 2)
```

Example:

```java
List<Integer> result =
    numbers.stream()
           .filter(x -> x > 10)
           .map(x -> x * 2)
           .toList();
```

---

# 12. `map()` vs `reduce()`

`map` transforms every element.

```java
[1,2,3,4]
    ↓ map(x -> x * 2)
[2,4,6,8]
```

`reduce` combines elements into a single result.

```java
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
1 + 2 + 3 + 4
```

---

# 13. `collect()` vs `reduce()`

Another good interview question.

### `reduce`

Usually combines elements into **one value**.

```java
int sum = numbers.stream()
                 .reduce(0, Integer::sum);
```

### `collect`

Usually accumulates elements into a mutable result structure.

```java
List<Integer> result =
    numbers.stream()
           .filter(x -> x > 10)
           .collect(Collectors.toList());
```

`collect()` is especially useful for:

```java
groupingBy
partitioningBy
toMap
joining
toList
toSet
```

---

# 14. `groupingBy()` — very important

Suppose:

```java
class Employee {
    String name;
    String department;
    int salary;
}
```

Question:

> Group employees by department.

```java
Map<String, List<Employee>> result =
    employees.stream()
             .collect(Collectors.groupingBy(Employee::getDepartment));
```

Result conceptually:

```text
IT       -> [John, Alex]
HR       -> [David, Mike]
Finance  -> [Sara]
```

### Follow-up

Find average salary per department:

```java
Map<String, Double> result =
    employees.stream()
             .collect(Collectors.groupingBy(
                 Employee::getDepartment,
                 Collectors.averagingInt(Employee::getSalary)
             ));
```

This is exactly the kind of Stream coding that appears in reported TCS interviews.

---

# 15. `partitioningBy()` vs `groupingBy()`

`partitioningBy` creates exactly two logical groups based on a predicate:

```java
Map<Boolean, List<Employee>> result =
    employees.stream()
             .collect(Collectors.partitioningBy(
                 e -> e.getSalary() > 100000
             ));
```

Result:

```text
true  -> salary > 100000
false -> salary <= 100000
```

`groupingBy` can create many groups:

```java
groupingBy(Employee::getDepartment)
```

---

# 16. `Collectors.toMap()` — famous trap

Suppose:

```java
employees.stream()
    .collect(Collectors.toMap(
        Employee::getDepartment,
        Employee::getName
    ));
```

What if two employees belong to the same department?

You can get:

```text
IllegalStateException: Duplicate key
```

Handle duplicates:

```java
.collect(Collectors.toMap(
    Employee::getDepartment,
    Employee::getName,
    (existing, replacement) -> existing
));
```

This is a very useful production-level detail.

---

# 17. Find highest salary employee

```java
Employee highest =
    employees.stream()
             .max(Comparator.comparingInt(Employee::getSalary))
             .orElse(null);
```

### Highest salary by department

```java
Map<String, Optional<Employee>> result =
    employees.stream()
             .collect(Collectors.groupingBy(
                 Employee::getDepartment,
                 Collectors.maxBy(
                     Comparator.comparingInt(Employee::getSalary)
                 )
             ));
```

If you want the employee directly rather than `Optional<Employee>`, you can use a downstream collector such as `collectingAndThen`.

---

# 18. `Comparable` vs `Comparator`

This has appeared in recent TCS reports.

### Comparable

Defines the object's **natural ordering**.

```java
class Employee implements Comparable<Employee> {

    @Override
    public int compareTo(Employee other) {
        return Integer.compare(this.salary, other.salary);
    }
}
```

Usage:

```java
Collections.sort(employees);
```

### Comparator

Defines ordering externally.

```java
Comparator<Employee> byName =
    Comparator.comparing(Employee::getName);
```

Then:

```java
employees.sort(byName);
```

### Interview answer

> Comparable is used when the class itself defines its natural ordering. Comparator is used when we want external, custom, or multiple sorting strategies.

---

# 19. `orElse()` vs `orElseGet()`

Very common trap.

```java
Optional<String> name = Optional.of("John");

name.orElse(getDefault());
```

`getDefault()` can be evaluated **even though the Optional already contains a value**.

With:

```java
name.orElseGet(() -> getDefault());
```

the supplier is evaluated only when the Optional is empty.

So:

```text
orElse     → eager
orElseGet  → lazy
```

This matters if the fallback operation is expensive or has side effects.

---

# 20. Should Optional be used everywhere?

No.

Good:

```java
public Optional<Employee> findEmployee(Long id)
```

It clearly communicates that the result may not exist.

But blindly using:

```java
Optional<String> name;
```

as entity fields, DTO fields, method parameters, etc. is generally not a good design choice.

Also don't do:

```java
Optional.ofNullable(x).get();
```

without checking presence. You're basically hiding a potential `NoSuchElementException`.

---

# 21. What is a parallel stream?

```java
numbers.parallelStream()
       .filter(...)
       .map(...)
       .toList();
```

It allows stream processing to execute in parallel.

By default, parallel streams commonly use the **common ForkJoinPool**.

### Is parallel stream always faster?

**No.**

For small collections:

```text
parallelization overhead > computation benefit
```

For I/O-heavy operations, parallel streams can also be problematic because you're consuming shared worker threads with blocking operations.

### Good interview answer

> Parallel streams can help CPU-intensive, sufficiently large, independent workloads, but they shouldn't be used blindly. Ordering, shared mutable state, synchronization, workload size, and blocking operations all affect whether parallelization is beneficial.

---

# 22. What is `CompletableFuture`?

Java 8 introduced `CompletableFuture` for asynchronous and composable operations.

Example:

```java
CompletableFuture
    .supplyAsync(() -> getUser())
    .thenApply(user -> user.getName())
    .thenAccept(System.out::println);
```

Conceptually:

```text
getUser()
   ↓
getName()
   ↓
print
```

---

# 23. `thenApply()` vs `thenCompose()`

**Very important.**

### `thenApply`

Used for transformation.

```java
CompletableFuture<User> userFuture = ...;

CompletableFuture<String> result =
    userFuture.thenApply(User::getName);
```

One future produces another value.

```text
Future<User>
     ↓
Future<String>
```

### `thenCompose`

Used when the next operation itself returns a `CompletableFuture`.

```java
CompletableFuture<User> userFuture = ...;

CompletableFuture<Address> addressFuture =
    userFuture.thenCompose(user ->
        getAddressAsync(user.getId())
    );
```

Without `thenCompose`, you'd effectively get:

```text
CompletableFuture<CompletableFuture<Address>>
```

`thenCompose` flattens it.

### Easy memory trick

```text
thenApply   → map
thenCompose → flatMap
```

---

# 24. `thenCombine()`?

Used to combine **two independent asynchronous operations**.

```java
CompletableFuture<User> user =
    getUserAsync();

CompletableFuture<Account> account =
    getAccountAsync();

CompletableFuture<Result> result =
    user.thenCombine(
        account,
        (u, a) -> createResult(u, a)
    );
```

Both can execute independently, then their results are combined.

---

# 25. How do you handle exceptions in CompletableFuture?

Three important methods:

### `exceptionally`

Recover from an exception:

```java
future.exceptionally(ex -> defaultValue);
```

### `handle`

Handle both success and failure:

```java
future.handle((result, ex) -> {
    if (ex != null) {
        return fallback;
    }
    return result;
});
```

### `whenComplete`

Observe completion but generally don't transform the result:

```java
future.whenComplete((result, ex) -> {
    // logging
});
```

---

# 26. What's wrong with this?

```java
CompletableFuture.supplyAsync(() -> callDatabase());
```

Nothing inherently wrong, **but the default executor matters**.

If you're doing blocking database/network calls using the common ForkJoinPool, you can exhaust shared worker threads.

For blocking workloads, consider a properly sized, bounded application executor:

```java
ExecutorService executor = ...;

CompletableFuture.supplyAsync(
    () -> callDatabase(),
    executor
);
```

This becomes particularly important in microservices.

---

# 27. Java 8 interface default methods

Before Java 8, adding a method to an interface could break all implementations.

Java 8 introduced:

```java
default void log() {
    System.out.println("default");
}
```

Now existing implementations don't necessarily need to implement it.

### What if two interfaces have the same default method?

```java
interface A {
    default void test() {}
}

interface B {
    default void test() {}
}

class C implements A, B {
    @Override
    public void test() {
        A.super.test();
    }
}
```

The class must resolve the conflict.

---

# 28. Java 8 Date/Time API

Old:

```java
Date
Calendar
SimpleDateFormat
```

Java 8 introduced:

```java
LocalDate
LocalTime
LocalDateTime
ZonedDateTime
Instant
Duration
Period
DateTimeFormatter
```

Example:

```java
LocalDate today = LocalDate.now();

LocalDate nextWeek = today.plusWeeks(1);
```

For distributed systems, `Instant` is particularly useful because it represents a point on the UTC timeline.

---

# Java 8 — interview traps to remember

Before moving to Java 11:

* `Stream` ≠ Collection
* Streams are generally **lazy**
* Intermediate operation ≠ terminal operation
* `map` ≠ `flatMap`
* `map` ≠ `reduce`
* `collect` is for accumulation
* `groupingBy` ≠ `partitioningBy`
* `toMap` can fail on duplicate keys
* `orElse` is eager
* `orElseGet` is lazy
* `parallelStream()` is not automatically faster
* `thenApply` ≈ transformation
* `thenCompose` ≈ flattening async operations
* `thenCombine` combines independent futures
* Don't blindly use common ForkJoinPool for blocking work
* `Comparable` = natural ordering
* `Comparator` = external/custom ordering

**Next topic: Java 11 → Java 17 → Java 21**, with the questions that matter in a 4–9 year Java interview—especially **records, sealed classes, pattern matching, strong encapsulation, virtual threads, and the Java 21 concurrency model**.
