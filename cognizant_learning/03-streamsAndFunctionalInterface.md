Let's go! 🔥 **Java 8 is next**, and for this Cognizant role I want you to get beyond “I know streams” into **I can solve problems and explain the trade-offs**.

# Module 1.4 — Java 8: Lambda, Functional Interfaces & Streams

For a 5-year candidate, the interviewer can start with:

> "What are the features introduced in Java 8?"

and quickly move to:

> "Given this employee list, solve these five problems using streams."

So we'll learn both.

---

# 1. What did Java 8 introduce?

The major things you should know:

```text
Java 8
 │
 ├── Lambda expressions
 ├── Functional interfaces
 ├── Stream API
 ├── Method references
 ├── Optional
 ├── Default/static methods in interfaces
 └── New Date/Time API
```

For this interview, the biggest ones are:

**Lambda + Functional Interface + Streams + Optional.**

---

# 2. Functional Interface

A functional interface has **exactly one abstract method**.

Example:

```java
@FunctionalInterface
interface Calculator {
    int calculate(int a, int b);
}
```

Because there's only one abstract method, we can represent its implementation using a lambda.

```java
Calculator addition =
        (a, b) -> a + b;
```

Then:

```java
System.out.println(addition.calculate(10, 20));
```

Output:

```text
30
```

---

# 3. Why do we need Lambda?

Before Java 8:

```java
Calculator addition = new Calculator() {
    @Override
    public int calculate(int a, int b) {
        return a + b;
    }
};
```

Java 8:

```java
Calculator addition =
        (a, b) -> a + b;
```

The lambda makes it possible to pass **behavior as a value**.

Think:

```text
Traditional Java

data → method

Java 8

data + behavior
```

---

# 4. Common Functional Interfaces

You should know these very well:

| Interface           | Input | Output  |
| ------------------- | ----- | ------- |
| `Predicate<T>`      | T     | boolean |
| `Function<T,R>`     | T     | R       |
| `Consumer<T>`       | T     | void    |
| `Supplier<T>`       | none  | T       |
| `UnaryOperator<T>`  | T     | T       |
| `BinaryOperator<T>` | T,T   | T       |

---

# 5. Predicate

Used when you want to test a condition.

```java
Predicate<Integer> isEven =
        n -> n % 2 == 0;
```

Then:

```java
isEven.test(10);
```

returns:

```text
true
```

Think:

```text
T → boolean
```

This becomes extremely relevant to `filter()`.

---

# 6. Function

Transforms one type into another.

```java
Function<String, Integer> length =
        s -> s.length();
```

Think:

```text
T → R
```

This maps directly to:

```java
stream.map(...)
```

---

# 7. Consumer

Consumes a value but returns nothing.

```java
Consumer<String> printer =
        s -> System.out.println(s);
```

Think:

```text
T → void
```

Used heavily with:

```java
forEach()
```

---

# 8. Supplier

Produces a value.

```java
Supplier<Double> random =
        () -> Math.random();
```

Think:

```text
() → T
```

Very important when understanding:

```java
Optional.orElseGet(...)
```

because the supplier can defer computation.

---

# 9. Method Reference

Instead of:

```java
employees.forEach(
    employee -> System.out.println(employee)
);
```

you can write:

```java
employees.forEach(System.out::println);
```

Common forms:

```text
Class::staticMethod
object::instanceMethod
Class::instanceMethod
Class::new
```

Example:

```java
List<String> names =
        employees.stream()
                 .map(Employee::getName)
                 .toList();
```

---

# 10. Now Streams 🔥

A Stream is **not a collection**.

This distinction is important.

A collection stores data.

A stream represents a **pipeline for processing data**.

Think:

```text
Collection
    ↓
Stream
    ↓
Operations
    ↓
Result
```

Example:

```java
List<Integer> numbers =
        Arrays.asList(1, 2, 3, 4, 5);

List<Integer> result =
        numbers.stream()
               .filter(n -> n % 2 == 0)
               .toList();
```

Result:

```text
[2, 4]
```

---

# 11. Stream pipeline

Think of every Stream problem as:

```text
SOURCE
  ↓
INTERMEDIATE OPERATIONS
  ↓
TERMINAL OPERATION
```

Example:

```java
numbers.stream()
       .filter(...)
       .map(...)
       .sorted(...)
       .collect(...);
```

Here:

```text
Source       → numbers.stream()

Intermediate → filter
               map
               sorted

Terminal     → collect
```

---

# 12. Intermediate vs Terminal

### Intermediate operations

Return another Stream.

Examples:

```text
filter()
map()
flatMap()
sorted()
distinct()
limit()
skip()
```

They are generally **lazy**.

### Terminal operations

Actually trigger stream processing.

Examples:

```text
collect()
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

---

# 13. Lazy evaluation 🔥

Consider:

```java
numbers.stream()
       .filter(n -> {
           System.out.println(n);
           return n > 2;
       });
```

Nothing necessarily gets printed merely because you've defined this pipeline.

Why?

Because `filter()` is intermediate.

You need a terminal operation:

```java
numbers.stream()
       .filter(n -> {
           System.out.println(n);
           return n > 2;
       })
       .toList();
```

Now the pipeline executes.

Mental model:

```text
stream()
   ↓
build pipeline
   ↓
lazy
   ↓
terminal operation
   ↓
execution
```

---

# 14. `filter()`

`filter()` keeps elements satisfying a predicate.

```java
List<Integer> even =
    numbers.stream()
           .filter(n -> n % 2 == 0)
           .toList();
```

Conceptually:

```text
1 → reject
2 → keep
3 → reject
4 → keep
5 → reject
```

---

# 15. `map()`

`map()` transforms each element.

```java
List<String> names =
    employees.stream()
             .map(Employee::getName)
             .toList();
```

Input:

```text
Employee
Employee
Employee
```

Output:

```text
String
String
String
```

Think:

```text
T → R
```

---

# 16. `filter()` vs `map()`

Extremely simple mental model:

```text
filter()
   ↓
removes / keeps elements

map()
   ↓
transforms elements
```

Example:

```java
numbers.stream()
       .filter(n -> n > 10)
       .map(n -> n * 2)
```

First:

```text
filter
```

Then:

```text
transform
```

---

# 17. `flatMap()` 🔥🔥🔥

This is one of the most common Java 8 interview questions.

Suppose:

```java
List<List<Integer>> numbers =
    Arrays.asList(
        Arrays.asList(1, 2),
        Arrays.asList(3, 4),
        Arrays.asList(5, 6)
    );
```

With `map()`:

```java
numbers.stream()
       .map(list -> list.stream())
```

You effectively get:

```text
Stream<Stream<Integer>>
```

That's nested.

With:

```java
numbers.stream()
       .flatMap(List::stream)
```

you get:

```text
Stream<Integer>
```

Result:

```text
1 2 3 4 5 6
```

Mental model:

```text
map

[A,B] [C,D] [E,F]
   ↓
Stream<List<Integer>>
```

versus:

```text
flatMap

[A,B] [C,D] [E,F]
       ↓
A B C D E F
```

---

# 18. Real interview example for `flatMap()`

Suppose:

```java
class Employee {
    String name;
    List<String> skills;
}
```

Data:

```text
Aryan → [Java, Kafka]
Rahul → [Java, Spring]
Amit  → [Kafka, Docker]
```

Question:

> Find all unique skills.

Answer:

```java
Set<String> skills =
    employees.stream()
             .flatMap(e -> e.getSkills().stream())
             .collect(Collectors.toSet());
```

Flow:

```text
Employee
   ↓
List<Skill>
   ↓
flatMap()
   ↓
individual Skill
   ↓
distinct / Set
```

This is exactly the type of question I expect in a Java interview.

---

# 19. `distinct()`

```java
List<Integer> result =
    numbers.stream()
           .distinct()
           .toList();
```

It removes duplicates according to equality semantics.

For custom objects, that means your `equals()`/`hashCode()` design matters.

Notice how this connects directly to our previous topic!

```text
Java 8
  ↓
distinct()
  ↓
equals/hashCode
```

---

# 20. `sorted()`

Natural ordering:

```java
numbers.stream()
       .sorted()
       .toList();
```

Custom:

```java
employees.stream()
         .sorted(
             Comparator.comparing(Employee::getSalary)
                       .reversed()
         )
         .toList();
```

Again:

```text
Streams
  ↓
Comparator
```

---

# 21. `limit()` and `skip()`

Example:

```java
numbers.stream()
       .limit(5)
```

takes the first five elements.

```java
numbers.stream()
       .skip(5)
```

skips the first five.

Useful for:

* top N
* pagination-like processing
* limiting expensive work

But don't confuse in-memory stream pagination with **database pagination**.

If your database has 100 million rows, doing:

```java
repository.findAll()
```

and then:

```java
stream().skip(...)
```

is not a scalable database pagination strategy.

🔥 That's a production-level distinction.

---

# 22. `reduce()`

Used to combine elements into a single result.

Example:

```java
int sum =
    numbers.stream()
           .reduce(0, (a, b) -> a + b);
```

Conceptually:

```text
0 + 1
 ↓
1 + 2
 ↓
3 + 3
 ↓
6 + 4
 ↓
10 + 5
 ↓
15
```

Think:

```text
many values
    ↓
one value
```

Examples:

* sum
* product
* max
* min
* combining objects

---

# 23. `collect()`

`collect()` is generally used to accumulate stream results into a collection or another result structure.

Example:

```java
List<String> names =
    employees.stream()
             .map(Employee::getName)
             .collect(Collectors.toList());
```

Modern Java can also use:

```java
.toList();
```

depending on the required semantics.

---

# 24. `groupingBy()` 🔥

Extremely common interview question.

Suppose:

```text
Employee
name
department
salary
```

Question:

> Group employees by department.

Answer:

```java
Map<String, List<Employee>> result =
    employees.stream()
             .collect(
                 Collectors.groupingBy(
                     Employee::getDepartment
                 )
             );
```

Result conceptually:

```text
IT
 ├── Aryan
 ├── Rahul
 └── Amit

HR
 ├── Priya
 └── Neha
```

---

# 25. Grouping + counting

Question:

> Count employees in each department.

```java
Map<String, Long> count =
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
IT → 3
HR → 2
```

---

# 26. Grouping + max salary

🔥🔥🔥

Question:

> Find the highest-paid employee in each department.

```java
Map<String, Optional<Employee>> result =
    employees.stream()
             .collect(
                 Collectors.groupingBy(
                     Employee::getDepartment,
                     Collectors.maxBy(
                         Comparator.comparing(
                             Employee::getSalary
                         )
                     )
                 )
             );
```

That's a very interview-worthy problem.

---

# 27. `partitioningBy()`

Difference from `groupingBy()`:

`partitioningBy()` divides data into **two groups based on a boolean condition**.

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

Conceptually:

```text
true
 ↓
salary > 100000

false
 ↓
salary <= 100000
```

Think:

```text
groupingBy()
   ↓
many possible groups

partitioningBy()
   ↓
true / false
```

---

# 28. Convert List → Map

Very common.

```java
Map<Integer, Employee> employeeMap =
    employees.stream()
             .collect(
                 Collectors.toMap(
                     Employee::getId,
                     employee -> employee
                 )
             );
```

Can simplify:

```java
Collectors.toMap(
    Employee::getId,
    Function.identity()
)
```

---

# 29. BIG trap: duplicate keys

Suppose two Employees have the same ID.

This:

```java
Collectors.toMap(
    Employee::getId,
    Function.identity()
)
```

can throw an exception because the key isn't unique.

You need a merge function:

```java
Collectors.toMap(
    Employee::getId,
    Function.identity(),
    (existing, replacement) -> replacement
)
```

Now you're explicitly defining what happens when duplicate IDs occur.

🔥 This is a good production-level question.

---

# 30. `anyMatch`, `allMatch`, `noneMatch`

These are extremely useful.

### anyMatch

```java
boolean exists =
    employees.stream()
             .anyMatch(e -> e.getSalary() > 200000);
```

Means:

> Does at least one satisfy the condition?

### allMatch

```java
employees.stream()
         .allMatch(e -> e.getSalary() > 50000);
```

Means:

> Do all satisfy it?

### noneMatch

```java
employees.stream()
         .noneMatch(e -> e.getSalary() < 0);
```

Means:

> Does none satisfy it?

These can short-circuit.

---

# 31. `findFirst()` vs `findAny()`

```java
stream.findFirst()
```

returns the first element according to encounter order where applicable.

`findAny()` allows the implementation more freedom, particularly useful in parallel processing.

This distinction becomes more interesting with:

```text
parallelStream()
```

---

# 32. `parallelStream()` 🔥

This is a **trap**.

An interviewer might ask:

> "If streams are useful, why not always use parallelStream?"

Because parallelism isn't free.

Conceptually:

```text
Normal stream

A → B → C → D → E
```

Parallel:

```text
          Stream
        /   |   \
       A    B    C
       |    |    |
    Thread Thread Thread
```

There are costs:

* splitting work
* thread scheduling
* synchronization/coordination
* merging results
* contention
* common ForkJoinPool usage
* overhead for small tasks

Therefore:

> `parallelStream()` can be slower than a sequential stream for small datasets or cheap operations.

---

# 33. When parallelism can be dangerous

Suppose:

```java
employees.parallelStream()
         .forEach(e -> database.save(e));
```

🚨 Don't assume this is a good optimization.

Now you've potentially created:

```text
many parallel tasks
       ↓
database
       ↓
connection pool exhausted
       ↓
contention
       ↓
performance degradation
```

You must consider the **downstream bottleneck**.

This is exactly how you should think about your ETL/batch systems too.

---

# 34. Streams don't magically improve algorithmic complexity

Suppose:

```java
list.stream()
    .filter(...)
    .collect(...)
```

doesn't magically make the algorithm faster.

Streams primarily provide:

* declarative processing
* composability
* readability
* lazy evaluation
* potential parallel processing

You still need to understand the underlying complexity.

---

# 35. Stream vs Collection

Interview answer:

> "A Collection is primarily a data structure used to store and manage elements, whereas a Stream is an abstraction for processing elements through a pipeline. A stream doesn't generally store the data itself, and stream operations can be lazy until a terminal operation is invoked."

Excellent.

---

# 36. A very important interview problem

Let's build one.

Given:

```java
class Employee {
    private int id;
    private String name;
    private String department;
    private double salary;
}
```

Question:

> Find the second-highest distinct salary.

A possible approach:

```java
Optional<Double> secondHighest =
    employees.stream()
             .map(Employee::getSalary)
             .distinct()
             .sorted(Comparator.reverseOrder())
             .skip(1)
             .findFirst();
```

Flow:

```text
Employees
   ↓
salary
   ↓
distinct
   ↓
descending sort
   ↓
skip highest
   ↓
find second
```

But here's your 5-year-level follow-up:

> **What is the complexity?**

The sort makes it roughly:

```text
O(n log n)
```

If the interviewer asks whether we can improve it, you should start thinking about a one-pass approach / maintaining the top two distinct values rather than automatically sorting the entire dataset.

That's the mindset I want.

---

# 37. One more important interview question

> **What's wrong with this?**

```java
employees.stream()
         .filter(e -> e.getDepartment().equals("IT"))
         .forEach(e -> database.save(e));
```

Nothing is inherently syntactically wrong.

But production concerns include:

```text
Database call per employee
        ↓
N database calls
        ↓
potentially poor throughput
```

Depending on the use case, you might prefer:

```text
batch insert
bulk operation
JDBC batching
appropriate transaction boundaries
```

Again:

**Java 8 knowledge + backend engineering knowledge.**

---

# 🧠 Your Java 8 mental map

You should now see:

```text
                    STREAM
                       │
            ┌──────────┴──────────┐
            ↓                     ↓
      Intermediate             Terminal
            │                     │
      ┌─────┼─────┐         ┌─────┼──────┐
      ↓     ↓     ↓         ↓     ↓      ↓
   filter  map  flatMap   collect reduce count
      │     │     │         │
      ↓     ↓     ↓         ↓
   Predicate Function      groupingBy
                           partitioningBy
```

And:

```text
Functional Interfaces

Predicate → T → boolean
Function  → T → R
Consumer  → T → void
Supplier  → () → T
```

---

# 🎯 Now let's test you

I'm going to switch into **interviewer mode**.

### Question 1 — Easy/medium

Given:

```java
List<Integer> numbers =
    Arrays.asList(10, 5, 20, 5, 30, 10, 40);
```

Using Java 8 Streams, return the **unique even numbers sorted in descending order**.

Then answer:

**Q2. What is the difference between `map()` and `flatMap()`?**

And finally:

**Q3. Why can `parallelStream()` make an application slower rather than faster?**

Answer all three as if you're in the Cognizant interview. I'll evaluate them, then we'll go into **Optional + advanced Stream questions**, and after that we'll move to the **JVM + Multithreading section**.

Perfect. 🔥 Let's move to the next **high-value Java area: `Optional` + advanced Stream behavior**, and then we'll hit **JVM + Multithreading**.

# Module 1.5 — Optional + Advanced Streams

At 5 years, the interviewer isn't really checking whether you know `Optional.ofNullable()`. They're checking whether you understand **when it should and shouldn't be used**.

---

# 1. Why was Optional introduced?

Before Java 8:

```java
Employee employee = findEmployee(id);

if (employee != null) {
    System.out.println(employee.getName());
}
```

Without careful checks:

```java
employee.getDepartment().getManager().getName();
```

can produce:

```text
NullPointerException
```

`Optional<T>` represents:

```text
value exists
     OR
value doesn't exist
```

Conceptually:

```text
Optional<Employee>

       ┌───────────────┐
       │               │
       ↓               ↓
   Employee          empty
```

---

# 2. Creating Optional

### `Optional.of()`

```java
Optional<String> name =
    Optional.of("Aryan");
```

The value **must not be null**.

```java
Optional.of(null);
```

❌ `NullPointerException`

---

### `Optional.ofNullable()`

```java
Optional<String> name =
    Optional.ofNullable(value);
```

If `value == null`:

```text
Optional.empty()
```

Use this when the source may return null.

---

### `Optional.empty()`

```java
Optional<String> name =
    Optional.empty();
```

Explicitly represents absence.

---

# 3. `isPresent()`

```java
if (name.isPresent()) {
    System.out.println(name.get());
}
```

This works, but don't overuse it.

If your code becomes:

```java
if (optional.isPresent()) {
    ...
}
```

everywhere, you're often just recreating old null-checking patterns.

---

# 4. `orElse()`

```java
String name =
    optional.orElse("Unknown");
```

If value exists:

```text
value → returned
```

If empty:

```text
"Unknown" → returned
```

---

# 5. BIG interview question: `orElse()` vs `orElseGet()`

This is worth knowing precisely.

Suppose:

```java
String result =
    optional.orElse(getDefault());
```

`getDefault()` can be evaluated **even when the Optional already contains a value**.

With:

```java
String result =
    optional.orElseGet(() -> getDefault());
```

the supplier is evaluated only when the Optional is empty.

Conceptually:

```text
orElse()
    ↓
evaluate argument
    ↓
return value/default
```

versus:

```text
orElseGet()
    ↓
is value present?
   /       \
 yes       no
  ↓         ↓
value    call supplier
```

### Interview answer

> "`orElse()` receives an already evaluated argument, while `orElseGet()` takes a Supplier and can lazily calculate the fallback. Therefore `orElseGet()` is preferable when the fallback operation is expensive or has side effects."

🔥 Excellent answer.

---

# 6. `orElseThrow()`

Instead of:

```java
if (!optional.isPresent()) {
    throw new EmployeeNotFoundException();
}
```

you can do:

```java
Employee employee =
    optional.orElseThrow(
        EmployeeNotFoundException::new
    );
```

Very common in Spring services.

---

# 7. `map()` on Optional

Suppose:

```java
Optional<Employee> employee;
```

and:

```java
employee.map(Employee::getName);
```

Result:

```text
Optional<Employee>
       ↓
map()
       ↓
Optional<String>
```

The important idea:

> Optional's `map()` transforms the contained value if it exists.

---

# 8. Optional `flatMap()`

Suppose:

```java
Employee
   ↓
getDepartment()
   ↓
Optional<Department>
```

If:

```java
employee.map(Employee::getDepartment)
```

and `getDepartment()` already returns Optional, you can end up with:

```text
Optional<Optional<Department>>
```

That's where:

```java
flatMap()
```

helps:

```text
Optional<Employee>
        ↓
flatMap()
        ↓
Optional<Department>
```

Same fundamental idea as Stream `flatMap()`:

> **Flatten nested structures.**

---

# 9. Should Optional be used everywhere?

❌ No.

This is an important senior-level discussion.

Generally, `Optional` is particularly useful as a **return type when absence is a valid outcome**.

For example:

```java
Optional<Employee> findById(Long id)
```

But using Optional indiscriminately for:

```java
Optional<String> name;
```

as every field can add complexity and overhead.

Also, don't generally make it your default choice for every method parameter.

The goal is:

> **Make absence explicit where it improves the API.**

Not:

> "Use Optional everywhere."

---

# 10. Optional + Spring Data JPA

You may see:

```java
Optional<Employee> employee =
    employeeRepository.findById(id);
```

Then:

```java
Employee employee =
    employeeRepository.findById(id)
        .orElseThrow(EmployeeNotFoundException::new);
```

That's clean service-layer code.

---

# 11. Advanced Streams — short circuiting

Some stream operations don't necessarily process every element.

For example:

```java
numbers.stream()
       .anyMatch(n -> n > 100);
```

Suppose:

```text
1 2 3 101 200 300
```

Once `101` is found:

```text
1 → no
2 → no
3 → no
101 → YES
```

The stream can stop.

This is called **short-circuiting**.

Operations to know:

```text
anyMatch()
allMatch()
noneMatch()
findFirst()
findAny()
limit()
```

---

# 12. Why ordering matters

Consider:

```java
numbers.stream()
       .filter(...)
       .map(...)
       .limit(10)
```

versus:

```java
numbers.stream()
       .limit(10)
       .filter(...)
       .map(...)
```

These can produce different results.

Example:

```text
numbers:
1 2 3 4 5 6 7 8 9 10 11 12
```

If you:

```text
filter even
↓
limit 3
```

you get:

```text
2 4 6
```

But:

```text
limit 3
↓
filter even
```

gives:

```text
2 4
```

🔥 Stream pipelines aren't just a bag of operations. **Order matters.**

---

# 13. Stream processing is often fused

Consider:

```java
numbers.stream()
       .filter(n -> n > 10)
       .map(n -> n * 2)
       .filter(n -> n < 100)
       .toList();
```

Conceptually, you don't need to imagine:

```text
filter → creates entire intermediate list
       ↓
map → creates another list
       ↓
filter → creates another list
```

Streams can process elements through the pipeline efficiently, often in a fused/lazy manner:

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

This is one reason streams can be expressive without necessarily materializing every intermediate result.

---

# 14. `findFirst()` vs `findAny()`

Sequential stream:

```java
findFirst()
```

is straightforward.

With parallel streams, `findFirst()` has to respect encounter order where an order exists.

`findAny()` doesn't require that same first-element guarantee and can therefore give the implementation more flexibility.

So:

```text
findFirst()
   ↓
ordering matters

findAny()
   ↓
any matching element is acceptable
```

---

# 15. Stateful vs stateless operations

This is a useful advanced distinction.

### Stateless

Each element can be processed independently:

```text
filter()
map()
peek()
```

### Stateful

The operation may need information about other elements:

```text
sorted()
distinct()
```

For example:

```java
stream.sorted()
```

needs to consider the elements collectively.

This can affect memory and parallel execution characteristics.

---

# 16. `peek()` — interview trap

You may see:

```java
numbers.stream()
       .filter(...)
       .peek(System.out::println)
       .map(...)
       .toList();
```

`peek()` is mainly intended for **debugging/observing elements as they pass through the pipeline**, not for business logic.

Don't do:

```java
.peek(e -> database.save(e))
```

as your business-processing mechanism.

---

# 17. Stream side effects

Avoid code like:

```java
List<String> result = new ArrayList<>();

employees.stream()
         .filter(...)
         .forEach(e -> result.add(e.getName()));
```

You can simply:

```java
List<String> result =
    employees.stream()
             .filter(...)
             .map(Employee::getName)
             .toList();
```

Why?

The second version is:

* declarative
* easier to reason about
* less dependent on mutable external state
* safer for parallelization

---

# 18. Parallel Stream + shared mutable state

Danger:

```java
List<Integer> result = new ArrayList<>();

numbers.parallelStream()
       .forEach(result::add);
```

`ArrayList` isn't thread-safe.

You can get incorrect behavior.

Even replacing it with a synchronized collection doesn't automatically mean the design is good; synchronization can become a bottleneck.

Prefer stream collectors where appropriate:

```java
List<Integer> result =
    numbers.parallelStream()
           .filter(...)
           .toList();
```

---

# 19. `Collectors.groupingBy()` vs `groupingByConcurrent()`

Interesting 5-year-level question.

For parallel processing, Java provides:

```java
Collectors.groupingByConcurrent(...)
```

which can produce a concurrent map and may be appropriate for certain parallel workloads.

But again:

> Don't choose it simply because the word "Concurrent" sounds faster.

You need to consider:

* workload
* contention
* ordering requirements
* downstream collector
* parallel overhead

---

# 20. A realistic interview exercise

Suppose:

```java
class Employee {
    int id;
    String name;
    String department;
    double salary;
}
```

The interviewer asks:

### Q1

Find employees earning more than 100,000.

```text
filter()
```

### Q2

Return only their names.

```text
filter()
→ map()
```

### Q3

Sort them by salary descending.

```text
filter()
→ sorted()
```

### Q4

Group them by department.

```text
groupingBy()
```

### Q5

Find the highest salary per department.

```text
groupingBy()
→ maxBy()
```

### Q6

Return a Map:

```text
employeeId → employee
```

```text
toMap()
```

### Q7

What if employee IDs aren't unique?

Use the merge function.

This chain of questions is **very realistic for a 5-year Java interview**.

---

# 🚨 Now we're moving into a VERY important area

We've covered:

```text
Java
 ├── OOP
 ├── HashMap
 ├── HashSet
 ├── ConcurrentHashMap
 ├── Collections
 ├── Java 8
 │    ├── Lambda
 │    ├── Functional interfaces
 │    ├── Streams
 │    └── Optional
```

The next topic is:

# Module 1.6 — JVM + Multithreading 🔥🔥🔥🔥🔥

This is where I'll go much deeper.

We'll cover:

### JVM

```text
JVM
 ├── Heap
 │    ├── Young Generation
 │    └── Old Generation
 ├── Stack
 ├── Metaspace
 ├── PC Register
 └── Native Method Stack
```

Then:

* How Java code actually executes
* Class loading
* JIT
* Heap vs Stack
* GC
* Memory leaks in Java
* OutOfMemoryError
* StackOverflowError
* Strong/weak references

### Multithreading

Then we'll build:

```text
Thread
  ↓
Runnable
  ↓
Callable
  ↓
Future
  ↓
ExecutorService
  ↓
ThreadPoolExecutor
  ↓
synchronized
  ↓
volatile
  ↓
AtomicInteger
  ↓
CAS
  ↓
Lock
  ↓
ReadWriteLock
  ↓
Deadlock
  ↓
Race Condition
  ↓
CompletableFuture
```

And **this section is particularly important for you**, because the JD explicitly asks for multithreading and your backend/batch work gives you real examples to connect the theory to.

Next, we'll start with **JVM memory architecture**, then I'll hit you with the classic interview question:

> **"What exactly happens in JVM memory when you execute `Employee e = new Employee()`?"**
