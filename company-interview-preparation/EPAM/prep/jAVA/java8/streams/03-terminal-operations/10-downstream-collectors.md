Absolutely. Let's do a **fast interview-focused pass** over all three sections—enough to answer EPAM questions without going excessively deep.

# 1. Downstream Collectors

The key idea:

```text
groupingBy()
      ↓
downstream collector
      ↓
what should happen inside each group?
```

---

### `counting()`

Count elements in each group.

```java
Map<String, Long> count =
    employees.stream()
             .collect(groupingBy(
                 Employee::getDepartment,
                 counting()
             ));
```

```text
IT → 5
HR → 3
```

Think:

> **group + count**

---

### `mapping()`

Transform elements **inside each group**.

```java
Map<String, List<String>> names =
    employees.stream()
             .collect(groupingBy(
                 Employee::getDepartment,
                 mapping(Employee::getName, toList())
             ));
```

```text
IT → [Amit, Rahul]
HR → [John, Mike]
```

Think:

> **group + transform + collect**

---

### `joining()`

Combine strings.

```java
String result =
    names.stream()
         .collect(joining(", "));
```

Result:

```text
"Amit, Rahul, John"
```

Can also use prefix/suffix:

```java
joining(", ", "[", "]")
```

→

```text
"[Amit, Rahul, John]"
```

Very useful with `mapping()`:

```java
Map<String, String> namesByDept =
    employees.stream()
             .collect(groupingBy(
                 Employee::getDepartment,
                 mapping(
                     Employee::getName,
                     joining(", ")
                 )
             ));
```

---

### `summingInt()`

Sum a field.

```java
Map<String, Integer> salary =
    employees.stream()
             .collect(groupingBy(
                 Employee::getDepartment,
                 summingInt(Employee::getSalary)
             ));
```

```text
IT → 500000
HR → 300000
```

---

### `averagingInt()`

Average a field.

```java
Map<String, Double> average =
    employees.stream()
             .collect(groupingBy(
                 Employee::getDepartment,
                 averagingInt(Employee::getSalary)
             ));
```

Returns `Double`.

---

### `summarizingInt()`

This gives you **multiple statistics at once**.

```java
Map<String, IntSummaryStatistics> stats =
    employees.stream()
             .collect(groupingBy(
                 Employee::getDepartment,
                 summarizingInt(Employee::getSalary)
             ));
```

Then:

```java
stats.get("IT").getCount();
stats.get("IT").getSum();
stats.get("IT").getMin();
stats.get("IT").getMax();
stats.get("IT").getAverage();
```

Instead of traversing the group multiple times for count/sum/min/max/average.

---

### `collectingAndThen()` ⭐

This means:

> **Perform a collector, then apply another transformation to its final result.**

Example:

```java
List<String> result =
    employees.stream()
             .collect(collectingAndThen(
                 toList(),
                 Collections::unmodifiableList
             ));
```

Conceptually:

```text
stream
  ↓
toList()
  ↓
List
  ↓
Collections.unmodifiableList()
  ↓
final result
```

Another common example:

```java
Map<String, Employee> highestPaid =
    employees.stream()
             .collect(groupingBy(
                 Employee::getDepartment,
                 collectingAndThen(
                     maxBy(Comparator.comparingInt(Employee::getSalary)),
                     Optional::get
                 )
             ));
```

You use it when:

> "I know how to collect the data, but I want to transform the final collected result."

---

# 2. Optional + Streams

## What is `Optional`?

`Optional<T>` represents:

```text
value exists
      OR
value doesn't exist
```

Instead of:

```java
Employee employee = findEmployee();
if (employee != null) {
    ...
}
```

you can have:

```java
Optional<Employee> employee = findEmployee();
```

---

## `Optional.map()`

Transforms the value if present.

```java
Optional<String> name =
    Optional.of(employee)
            .map(Employee::getName);
```

Mental model:

```text
Optional<Employee>
       ↓ map()
Optional<String>
```

Similar to Stream `map()` conceptually:

```text
Stream<T> → Stream<R>

Optional<T> → Optional<R>
```

---

## `Optional.filter()`

Keeps the value only if the condition passes.

```java
Optional<Employee> result =
    Optional.of(employee)
            .filter(Employee::isActive);
```

If inactive:

```text
Optional.empty()
```

---

## `Optional.flatMap()` ⭐

Used when your mapping function **already returns Optional**.

Suppose:

```java
Employee → Optional<Address>
```

Don't do:

```java
Optional<Optional<Address>>
```

Instead:

```java
Optional<Address> address =
    employeeOptional.flatMap(Employee::getAddress);
```

Mental model:

```text
map:
Optional<T> → Optional<Optional<R>>

flatMap:
Optional<T> → Optional<R>
```

Exactly the same flattening idea you've already learned with Stream `flatMap()`.

---

# `orElse()` vs `orElseGet()` ⭐⭐⭐⭐⭐

Very common interview question.

### `orElse()`

```java
String name =
    optional.orElse(getDefaultName());
```

The default expression is evaluated **even if the Optional contains a value**.

### `orElseGet()`

```java
String name =
    optional.orElseGet(() -> getDefaultName());
```

The supplier executes **only when the Optional is empty**.

So:

```text
orElse()
    → default is evaluated eagerly

orElseGet()
    → default is evaluated lazily
```

This matters if:

```java
getDefaultName()
```

is expensive or has side effects.

---

# `orElseThrow()`

Instead of:

```java
if (optional.isEmpty()) {
    throw new RuntimeException();
}
```

you can:

```java
Employee employee =
    optional.orElseThrow();
```

Or custom exception:

```java
Employee employee =
    optional.orElseThrow(
        () -> new EmployeeNotFoundException()
    );
```

---

# Stream + Optional ⭐⭐⭐

Classic example:

```java
Optional<Employee> highest =
    employees.stream()
             .filter(Employee::isActive)
             .max(Comparator.comparingInt(Employee::getSalary));
```

Then:

```java
Employee employee =
    highest.orElseThrow(EmployeeNotFoundException::new);
```

This is a very clean production pattern.

---

# 3. Parallel Streams ⭐⭐⭐⭐⭐

Now the important final section.

## How does parallel stream work?

```java
numbers.parallelStream()
```

allows the stream framework to divide the source into pieces.

Conceptually:

```text
             [1 2 3 4 5 6 7 8]
                      ↓
                 split source
              /      |       \
          [1 2]   [3 4]   [5 6 7 8]
             ↓       ↓        ↓
          thread   thread    thread
              \       |       /
               combine results
```

Java commonly uses the **ForkJoinPool common pool** for parallel stream operations unless the execution context/framework setup changes how work is submitted.

---

# ForkJoinPool

The common pool uses worker threads to execute tasks.

Mental model:

```text
Main thread
    ↓
submit stream work
    ↓
ForkJoinPool
   / | \
 W1 W2 W3
   \ | /
   combine
```

It's designed for **divide-and-conquer work**.

---

# Stateless vs Stateful operations

### Stateless

Each element can be processed independently.

Examples:

```text
filter()
map()
mapToInt()
```

```text
element A → independently process
element B → independently process
element C → independently process
```

These generally parallelize well.

### Stateful

The operation needs information about multiple elements.

Examples:

```text
sorted()
distinct()
limit()
skip()
```

These may require coordination/buffering.

Therefore parallel execution can become more expensive.

---

# Ordered vs unordered

An ordered stream has an **encounter order**.

Example:

```java
List.of(1,2,3,4,5)
```

For:

```java
parallelStream().forEach(...)
```

execution order isn't guaranteed.

But:

```java
parallelStream().forEachOrdered(...)
```

preserves encounter order.

Similarly:

```text
findFirst()
→ must respect first element

findAny()
→ doesn't require first element
```

That's why `findAny()` can have more parallel freedom.

---

# Thread safety 🚨

Parallel stream does NOT automatically make your code thread-safe.

Bad:

```java
List<Integer> result = new ArrayList<>();

numbers.parallelStream()
       .forEach(result::add);
```

Multiple threads modify the same `ArrayList`.

Potential race condition.

Better:

```java
List<Integer> result =
    numbers.parallelStream()
           .toList();
```

Let the Stream framework manage the result accumulation.

---

# Side effects

Avoid:

```java
parallelStream()
    .map(...)
    .peek(x -> sharedCounter++)
```

or:

```java
parallelStream()
    .forEach(x -> database.save(x));
```

unless you have deliberately designed for concurrency.

Parallel streams mean your lambda may run concurrently.

---

# `findFirst()` vs `findAny()`

You've already understood this:

```text
findFirst()
→ first according to encounter order

findAny()
→ any matching element
```

With parallel execution:

```text
findFirst()
→ more coordination

findAny()
→ more freedom
```

So `findAny()` **can** be faster, but don't say it is *always* faster.

---

# `forEach()` vs `forEachOrdered()`

```java
parallelStream()
    .forEach(...)
```

→ order not guaranteed.

```java
parallelStream()
    .forEachOrdered(...)
```

→ encounter order preserved.

Ordering constraints can reduce the benefit of parallelism.

---

# When should you NOT use parallel streams? ⭐⭐⭐⭐⭐

This is probably more useful than memorizing the implementation.

Avoid or be cautious when:

### 1. Small dataset

```text
100 elements
```

Parallelization overhead can exceed the benefit.

### 2. I/O-heavy operations

```java
parallelStream()
    .forEach(x -> database.save(x));
```

You're not magically getting a better architecture.

You can overload the DB.

### 3. Shared mutable state

```java
parallelStream()
    .forEach(x -> sharedList.add(x));
```

Dangerous.

### 4. Order is important

If your algorithm requires strict encounter ordering, parallelism may provide little benefit.

### 5. Cheap operations

If your operation is:

```java
.map(x -> x * 2)
```

on a small collection, parallel overhead isn't worth it.

### 6. Blocking operations

Parallel streams use the common ForkJoinPool, so blocking tasks can interfere with other work using that pool.

---

# 🔥 EPAM final mental model

If they ask:

> **"When would you use parallel streams?"**

A strong answer:

> "I'd consider parallel streams for large datasets where the processing is CPU-intensive, stateless, independent per element, and the reduction/collection is safely parallelizable. I would avoid them for small datasets, blocking I/O, shared mutable state, or when strict ordering is important."

And remember:

```text
Parallel Stream
      ↓
split source
      ↓
process partitions concurrently
      ↓
combine results
```

### Your Stream API is now essentially complete. ✅

The remaining work is the **coding-pattern section**, which is the part I'd prioritize before moving to Generics:

```text
Frequency counting
Duplicates
Grouping
Sorting
Top N
Second highest
Max/min
Flattening
String problems
Employee problems
Map transformations
```

Those are where we'll turn everything you've learned into **EPAM-style coding solutions**.