# STREAM API — 2. STREAM PIPELINE & INTERNAL EXECUTION

This is a **very important EPAM topic**. Before we touch `filter()`, `map()`, or `flatMap()`, you need to understand what actually happens when a Stream pipeline executes.

---

## 2.1 Problem

Consider:

```java
List<Integer> numbers = List.of(1, 2, 3, 4, 5);

numbers.stream()
       .filter(n -> n % 2 == 0)
       .map(n -> n * 10)
       .forEach(System.out::println);
```

An interviewer can ask:

> "What happens internally when this code executes?"

A weak answer:

> "It filters, then maps, then prints."

A stronger answer needs to explain:

```text
source
  ↓
stream pipeline
  ↓
intermediate operations
  ↓
terminal operation
  ↓
actual traversal
```

And most importantly:

> **The intermediate operations don't immediately process the data.**

---

# 2.2 Stream Pipeline

A Stream pipeline consists of three major parts:

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
       .filter(n -> n % 2 == 0)
       .map(n -> n * 10)
       .forEach(System.out::println);
```

Breakdown:

```text
numbers
   ↓
stream()                  ← source
   ↓
filter()                  ← intermediate
   ↓
map()                     ← intermediate
   ↓
forEach()                 ← terminal
```

---

# 2.3 Source

The source provides the elements.

Examples:

```java
list.stream()
```

```java
Arrays.stream(array)
```

```java
Stream.of(1, 2, 3)
```

```java
IntStream.rangeClosed(1, 100)
```

```java
Stream.iterate(...)
```

```java
Stream.generate(...)
```

The source is where the pipeline gets its data.

---

# 2.4 Intermediate Operations

Examples:

```java
filter()
map()
flatMap()
distinct()
sorted()
limit()
skip()
```

They transform or constrain the pipeline.

The critical property:

> **Intermediate operations are lazy.**

They don't normally execute when you call them.

---

# 2.5 Terminal Operation

Examples:

```java
forEach()
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
```

The terminal operation **consumes the Stream** and triggers the pipeline.

For example:

```java
numbers.stream()
       .filter(n -> n % 2 == 0)
       .map(n -> n * 10)
       .forEach(System.out::println);
```

`forEach()` is the terminal operation.

---

# 2.6 Lazy Evaluation

This is one of your HR questions:

> **Why are Streams lazy?**

Consider:

```java
numbers.stream()
       .filter(n -> {
           System.out.println("filter: " + n);
           return n % 2 == 0;
       });
```

What happens?

Nothing is printed.

Why?

Because:

```java
filter()
```

is an intermediate operation.

The pipeline has been **described**, but hasn't been consumed.

---

# 2.7 Add a Terminal Operation

Now:

```java
numbers.stream()
       .filter(n -> {
           System.out.println("filter: " + n);
           return n % 2 == 0;
       })
       .forEach(System.out::println);
```

Now execution occurs.

Output:

```text
filter: 1
filter: 2
2
filter: 3
filter: 4
4
filter: 5
```

The exact important point is:

```text
No terminal operation
    ↓
No traversal

Terminal operation
    ↓
Pipeline executes
```

---

# 2.8 Why is laziness useful?

This is where the concept becomes practical.

Suppose:

```java
numbers.stream()
       .filter(...)
       .map(...)
       .findFirst();
```

The Stream doesn't necessarily need to process every element.

Once it has found the required first element, it can stop.

For example:

```java
List<Integer> numbers =
        List.of(1, 3, 5, 8, 10, 12);
```

Pipeline:

```java
Optional<Integer> result =
        numbers.stream()
               .filter(n -> n % 2 == 0)
               .findFirst();
```

Execution can conceptually be:

```text
1 → filter → false
3 → filter → false
5 → filter → false
8 → filter → true
             ↓
          findFirst
             ↓
            STOP
```

It doesn't need to inspect:

```text
10
12
```

This is called **short-circuiting**.

We'll study short-circuiting in detail later.

---

# 2.9 Very Important: Streams are not simply "multiple loops"

Many developers mentally imagine:

```java
stream.filter(...)
      .map(...)
      .forEach(...);
```

as:

```text
loop through everything
   ↓
create filtered collection
   ↓
loop through filtered collection
   ↓
create mapped collection
   ↓
loop through mapped collection
   ↓
forEach
```

That is **not the right mental model**.

For many pipelines, processing can be thought of as element-by-element:

```text
Element 1
 ↓
filter
 ↓
map
 ↓
terminal

Element 2
 ↓
filter
 ↓
map
 ↓
terminal
```

rather than:

```text
ALL elements
 ↓
filter ALL
 ↓
map ALL
 ↓
terminal ALL
```

This distinction is extremely important.

---

# 2.10 Example — element-by-element processing

Consider:

```java
List<Integer> numbers =
        List.of(1, 2, 3);
```

Pipeline:

```java
numbers.stream()
       .filter(n -> {
           System.out.println("filter " + n);
           return n % 2 == 0;
       })
       .map(n -> {
           System.out.println("map " + n);
           return n * 10;
       })
       .forEach(n ->
           System.out.println("result " + n)
       );
```

Conceptually:

```text
1
 ↓
filter(1)
 ↓
rejected

2
 ↓
filter(2)
 ↓
accepted
 ↓
map(2)
 ↓
20
 ↓
forEach(20)

3
 ↓
filter(3)
 ↓
rejected
```

Notice:

`map()` doesn't need to wait for `filter()` to process **all** elements.

Once an element survives the filter, it can move forward.

---

# 2.11 Stream Fusion

The previous behavior is related to an important optimization/concept often called **operation fusion**.

Instead of materializing intermediate collections after every operation, the pipeline can process elements through multiple stages.

Conceptually:

```text
source
  ↓
filter
  ↓
map
  ↓
terminal
```

becomes one processing pipeline.

This reduces unnecessary intermediate storage.

Don't say:

> "Streams always create no intermediate objects."

That's too strong.

Some operations and implementations can require buffering/state, particularly stateful operations such as:

```java
sorted()
distinct()
```

We'll cover that distinction.

---

# 2.12 Stateless vs Stateful Operations

This is an important advanced Stream concept.

## Stateless operations

Examples:

```java
filter()
map()
mapToInt()
```

Generally, processing one element doesn't require knowing all other elements.

For example:

```java
filter(n -> n % 2 == 0)
```

To decide whether `8` passes, you only need `8`.

You don't need:

```text
1,2,3,4,5,6,7,9,10...
```

---

# 2.13 Stateful Operations

Examples:

```java
sorted()
distinct()
```

These need information about multiple elements.

### `sorted()`

Suppose:

```java
Stream.of(5, 1, 4, 2, 3)
      .sorted()
```

To produce the smallest value first, the operation needs enough information about the input to establish ordering.

Conceptually:

```text
5 1 4 2 3
     ↓
collect/buffer as needed
     ↓
sort
     ↓
1 2 3 4 5
```

So `sorted()` is stateful.

---

# 2.14 Why this matters for performance

Compare:

```java
stream.filter(...)
```

with:

```java
stream.sorted(...)
```

`filter()` can process an element immediately.

`sorted()` may need to retain elements and establish ordering.

Therefore:

```text
filter()
→ generally stateless
→ easy to process incrementally

sorted()
→ stateful
→ may require buffering
→ more memory
→ potentially more expensive
```

This becomes especially important with **parallel streams**.

---

# 2.15 Stream Reuse

A Stream is generally **single-use**.

Example:

```java
Stream<Integer> stream =
        Stream.of(1, 2, 3, 4);

stream.count();

stream.forEach(System.out::println);
```

The second terminal operation will fail because the Stream has already been consumed.

You'll typically get:

```text
IllegalStateException
stream has already been operated upon or closed
```

Mental model:

```text
Collection
   ↓
can create Stream A
can create Stream B
can create Stream C

Stream A
   ↓
terminal operation
   ↓
consumed
```

---

# 2.16 Collection can be reused

This is one reason Collection and Stream are fundamentally different.

```java
List<Integer> numbers =
        List.of(1, 2, 3, 4);
```

You can:

```java
numbers.stream().count();

numbers.stream().forEach(...);

numbers.stream().filter(...);
```

because every call creates a **new Stream**.

But:

```java
Stream<Integer> stream =
        numbers.stream();
```

should not be reused after a terminal operation.

---

# 2.17 Streams generally don't modify the source

Suppose:

```java
List<Integer> numbers =
        new ArrayList<>(List.of(1, 2, 3, 4));
```

Then:

```java
List<Integer> result =
        numbers.stream()
               .filter(n -> n % 2 == 0)
               .toList();
```

The source is still:

```text
1 2 3 4
```

and:

```text
result
→ 2 4
```

The Stream pipeline processes the source; it doesn't inherently mean "modify the collection."

---

# 2.18 Side effects — important production issue

Consider:

```java
List<Integer> result = new ArrayList<>();

numbers.stream()
       .filter(n -> n % 2 == 0)
       .forEach(result::add);
```

This can work sequentially, but introducing external mutable state into Stream pipelines makes reasoning harder.

It becomes especially dangerous with parallel streams.

For example:

```java
List<Integer> result = new ArrayList<>();

numbers.parallelStream()
       .forEach(result::add);
```

`ArrayList` isn't thread-safe.

This can cause incorrect behavior.

Prefer:

```java
List<Integer> result =
        numbers.parallelStream()
               .filter(n -> n % 2 == 0)
               .toList();
```

The Stream API can manage the collection process appropriately.

We'll go much deeper into this when we reach `collect()` and parallel streams.

---

# 2.19 Encounter Order

Some Streams have an **encounter order**.

For example:

```java
List<Integer> numbers =
        List.of(1, 2, 3, 4, 5);
```

Its natural encounter order is:

```text
1 → 2 → 3 → 4 → 5
```

Operations such as:

```java
forEach()
findFirst()
```

can interact with this ordering.

This becomes particularly important when using:

```java
parallelStream()
```

because parallel processing introduces ordering/performance trade-offs.

We'll cover this deeply with `findFirst()` vs `findAny()` when we reach terminal operations.

---

# 2.20 Short-circuiting

Some operations don't need the entire Stream.

Examples:

```text
findFirst()
findAny()
anyMatch()
allMatch()
noneMatch()
limit()
```

Example:

```java
boolean exists =
        numbers.stream()
               .anyMatch(n -> n > 100);
```

Suppose:

```text
1
2
3
150
4
5
```

Once `150` is found:

```text
anyMatch
   ↓
true
   ↓
STOP
```

There is no reason to inspect the remaining elements.

---

# 2.21 Short-circuiting + laziness

This is where two concepts combine:

```text
Lazy
 +
Short-circuiting
 =
Potentially avoid unnecessary processing
```

Example:

```java
numbers.stream()
       .filter(...)
       .map(...)
       .findFirst();
```

The pipeline can process only as many elements as necessary to produce the first result.

This is one of the major reasons Streams can express efficient processing pipelines.

---

# 2.22 Terminal operation consumes the Stream

This is a key rule:

```text
Intermediate operations
→ build pipeline

Terminal operation
→ starts/consumes pipeline
```

For example:

```java
Stream<Integer> stream =
        numbers.stream()
               .filter(n -> n > 2);
```

At this point, the pipeline has been constructed.

Then:

```java
stream.count();
```

consumes it.

You can't subsequently do:

```java
stream.forEach(...);
```

on the same Stream.

---

# 2.23 Production mental model

When you see:

```java
source
    .filter(...)
    .map(...)
    .flatMap(...)
    .sorted(...)
    .limit(...)
    .collect(...);
```

don't mentally see a chain of temporary Lists.

Think:

```text
             STREAM PIPELINE

Source
  │
  ▼
[ filter ]
  │
  ▼
[ map ]
  │
  ▼
[ flatMap ]
  │
  ▼
[ sorted ]
  │
  ▼
[ limit ]
  │
  ▼
Terminal operation
```

Then ask:

```text
Which operations are lazy?
Which are stateless?
Which are stateful?
Can the pipeline short-circuit?
Does ordering matter?
Is the stream sequential or parallel?
Are there side effects?
```

That's the **Senior Engineer level mental model** we're aiming for.

---

# 2.24 Interview questions naturally covered here

### What is a Stream?

A Stream is an abstraction for declaratively processing a sequence of elements through a pipeline of operations.

---

### Stream vs Collection?

```text
Collection
→ stores/manages data

Stream
→ processes data

Collection
→ can be traversed multiple times

Stream
→ generally single-use

Collection
→ eager data structure

Stream
→ lazy processing pipeline
```

---

### Why are Streams lazy?

Because intermediate operations don't immediately process elements. Processing is triggered when a terminal operation consumes the pipeline.

Benefits include:

```text
short-circuiting
avoiding unnecessary work
pipeline composition
efficient element-by-element processing
```

---

### Intermediate vs terminal operation?

```text
Intermediate
→ returns Stream
→ lazy

Examples:
filter
map
flatMap
sorted
distinct

Terminal
→ produces final result/side effect
→ triggers processing

Examples:
collect
reduce
forEach
count
findFirst
```

---

### Can you reuse a Stream?

No. After a terminal operation, the Stream is consumed.

Create a new Stream from the source instead.

---

### What is a stateful Stream operation?

An operation that may need information about multiple elements rather than processing each element independently.

Examples:

```text
sorted()
distinct()
```

---

### What is short-circuiting?

An operation can stop processing once its result is determined.

Examples:

```text
findFirst()
findAny()
anyMatch()
allMatch()
noneMatch()
limit()
```

---

# Notes — Stream Pipeline

```text
STREAM PIPELINE

Source
  ↓
Intermediate operations
  ↓
Terminal operation


Intermediate:
- lazy
- return Stream
- filter
- map
- flatMap
- distinct
- sorted
- limit
- skip

Terminal:
- trigger execution
- consume Stream
- forEach
- collect
- reduce
- count
- findFirst
- findAny
- match operations


Lazy:
Intermediate operations don't execute immediately.

Short-circuiting:
Can stop before processing all elements.

Stateless:
filter()
map()

Stateful:
sorted()
distinct()

Stream:
- generally single-use
- doesn't inherently modify source
- pipeline processing
- can exploit short-circuiting
```

---

# NEXT TOPIC — `filter()`

Now we're finally ready to learn our **first major Stream operation** properly.

We'll cover:

```text
filter()
  ↓
Problem it solves
  ↓
Predicate connection
  ↓
Internal execution
  ↓
Element-by-element processing
  ↓
Multiple filters
  ↓
Filtering objects
  ↓
Primitive filtering
  ↓
filter + short-circuiting
  ↓
Complexity
  ↓
Production considerations
  ↓
EPAM coding questions
```

Then we'll move to **`map()`**.