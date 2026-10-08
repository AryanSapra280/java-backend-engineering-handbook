# `map()` — Transforming Every Element

This is the next core Stream operation. Since we already covered `Function<T,R>` and lambdas, `map()` is where that knowledge becomes very practical.

---

## 1. Problem — Why do we need `map()`?

Suppose:

```java
List<String> names = List.of("aryan", "rahul", "amit");
```

You want:

```text
ARYAN
RAHUL
AMIT
```

You aren't filtering anything.

You aren't removing anything.

You want to **transform every element**.

That's exactly what `map()` does.

---

# 2. Concept

`map()` means:

> **Take each element and transform it into another element.**

Basic syntax:

```java
stream.map(function)
```

Its signature is essentially:

```java
<R> Stream<R> map(Function<? super T, ? extends R> mapper)
```

The important part for interviews is:

```text
T → R
```

That's exactly our `Function<T, R>`.

For example:

```java
Function<String, String>
```

means:

```text
String → String
```

And:

```java
Function<Employee, String>
```

means:

```text
Employee → String
```

---

# 3. Simple Example

```java
List<String> names =
        List.of("aryan", "rahul", "amit");

List<String> result =
        names.stream()
             .map(name -> name.toUpperCase())
             .toList();
```

Result:

```text
[ARYAN, RAHUL, AMIT]
```

Could also use a method reference:

```java
List<String> result =
        names.stream()
             .map(String::toUpperCase)
             .toList();
```

Because:

```java
name -> name.toUpperCase()
```

is equivalent to:

```java
String::toUpperCase
```

---

# 4. Internal Working

This is VERY important for interviews.

Suppose:

```java
List<Integer> numbers = List.of(1, 2, 3, 4);
```

Pipeline:

```java
numbers.stream()
       .map(n -> n * 10)
       .toList();
```

Mentally think:

```text
Input       map()             Output

1      →    1 * 10      →      10
2      →    2 * 10      →      20
3      →    3 * 10      →      30
4      →    4 * 10      →      40
```

So:

```text
Stream<Integer>
      ↓
map(n -> n * 10)
      ↓
Stream<Integer>
```

`map()` does **not** modify the original collection.

It produces transformed elements for the downstream operation.

---

# 5. Why is `map()` Lazy?

Remember our pipeline model:

```text
Source
  ↓
filter()
  ↓
map()
  ↓
terminal operation
```

Neither `filter()` nor `map()` actually starts processing when you create them.

Example:

```java
numbers.stream()
       .map(n -> {
           System.out.println("Mapping " + n);
           return n * 10;
       });
```

Nothing prints.

Why?

Because:

```java
map()
```

is an **intermediate operation**.

There is no terminal operation.

Add:

```java
numbers.stream()
       .map(n -> {
           System.out.println("Mapping " + n);
           return n * 10;
       })
       .toList();
```

Now it executes.

---

# 6. `map()` Usually Preserves Element Count

This is a useful mental model.

Suppose:

```text
Input:
[1, 2, 3, 4]
```

After:

```java
.map(n -> n * 2)
```

you get:

```text
[2, 4, 6, 8]
```

Four elements entered.

Four elements came out.

But the **type/value can change**.

For example:

```java
List<String> names = List.of("Aryan", "Rahul", "Amit");

List<Integer> lengths =
        names.stream()
             .map(String::length)
             .toList();
```

Result:

```text
[5, 5, 4]
```

Here:

```text
String → Integer
```

So:

```java
Stream<String>
```

becomes:

```java
Stream<Integer>
```

This is one of the most important properties of `map()`.

---

# 7. `Employee → String`

This is extremely common in interviews.

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

And:

```java
List<Employee> employees = ...;
```

You want only employee names.

```java
List<String> names =
        employees.stream()
                 .map(Employee::getName)
                 .toList();
```

Conceptually:

```text
Employee
   ↓
getName()
   ↓
String
```

So:

```text
Stream<Employee>
        ↓
map(Employee::getName)
        ↓
Stream<String>
```

---

# 8. `map()` Can Change Type

This is worth remembering for EPAM.

Example:

```java
List<String> numbers = List.of("10", "20", "30");
```

Convert them to integers:

```java
List<Integer> result =
        numbers.stream()
               .map(Integer::parseInt)
               .toList();
```

Result:

```text
[10, 20, 30]
```

Here:

```text
String → Integer
```

Another:

```java
List<String> names = List.of("Java", "Spring", "Kafka");

List<Integer> lengths =
        names.stream()
             .map(String::length)
             .toList();
```

Here:

```text
String → Integer
```

---

# 9. Multiple `map()` Operations

You can chain transformations.

```java
List<String> names =
        List.of("aryan", "rahul", "amit");

List<Integer> result =
        names.stream()
             .map(String::toUpperCase)
             .map(String::length)
             .toList();
```

Trace:

```text
"aryan"
   ↓
"ARYAN"
   ↓
5

"rahul"
   ↓
"RAHUL"
   ↓
5

"amit"
   ↓
"AMIT"
   ↓
4
```

Result:

```text
[5, 5, 4]
```

The pipeline is:

```text
Stream<String>
      ↓
map(String → String)
      ↓
Stream<String>
      ↓
map(String → Integer)
      ↓
Stream<Integer>
```

---

# 10. `filter()` + `map()`

This combination is **extremely common** in interviews.

Suppose:

```java
List<Employee> employees = ...;
```

Requirement:

> Get names of employees earning more than ₹50,000.

```java
List<String> names =
        employees.stream()
                 .filter(e -> e.getSalary() > 50000)
                 .map(Employee::getName)
                 .toList();
```

Pipeline:

```text
Employee
   ↓
filter salary > 50000
   ↓
Employee
   ↓
map Employee → name
   ↓
String
```

This is a perfect example of:

```text
filter = select
map    = transform
```

---

# 11. Why Filter Before Map?

Compare:

### Approach A

```java
employees.stream()
         .filter(e -> e.getSalary() > 50000)
         .map(Employee::getName)
         .toList();
```

### Approach B

```java
employees.stream()
         .map(Employee::getName)
         .filter(name -> ...)
         .toList();
```

If the filtering condition can be applied to the original object, **filtering first is generally better**, especially when the transformation is expensive.

Suppose there are:

```text
1,000,000 employees
```

and only:

```text
10,000 employees
```

earn > ₹50,000.

Then:

```text
filter first
    ↓
1,000,000 → 10,000
    ↓
map only 10,000
```

Instead of:

```text
map 1,000,000
    ↓
then filter
```

So an important production principle is:

> **Reduce the number of elements as early as possible before expensive transformations.**

But don't blindly reorder operations—the operations must remain semantically equivalent.

---

# 12. `map()` + Terminal Operation

You don't necessarily have to collect the result.

Example:

```java
int totalLength =
        names.stream()
             .mapToInt(String::length)
             .sum();
```

We'll study `mapToInt()` separately next.

Or:

```java
long count =
        names.stream()
             .map(String::toUpperCase)
             .count();
```

The terminal operation determines what ultimately happens to the mapped values.

---

# 13. Important: `map()` Doesn't Flatten

This is where `map()` vs `flatMap()` becomes critical.

Suppose:

```java
List<List<String>> users = List.of(
        List.of("a", "b"),
        List.of("c", "d")
);
```

If you use:

```java
users.stream()
     .map(list -> list.stream())
```

you get approximately:

```text
Stream<Stream<String>>
```

You're transforming:

```text
List<String> → Stream<String>
```

You now have nested streams.

`flatMap()` exists to flatten this.

We'll cover that later in detail.

So remember:

```text
map:
A → B

flatMap:
A → Stream<B>
then flatten
```

---

# 14. `map()` vs `filter()`

Very common interview question.

| Operation | Purpose | Output |
|---|---|---|
| `filter()` | Select elements | Same type |
| `map()` | Transform elements | Potentially different type |

Example:

```java
.filter(n -> n > 10)
```

means:

```text
"Should this element remain?"
```

Whereas:

```java
.map(n -> n * 2)
```

means:

```text
"What should this element become?"
```

Mental shortcut:

```text
filter → keep/remove
map    → transform
```

---

# 15. `map()` vs `mapToInt()`

Both transform elements, but their resulting stream type differs.

### `map()`

```java
Stream<Integer>
```

Example:

```java
Stream<Integer> result =
        numbers.stream()
               .map(n -> n * 2);
```

### `mapToInt()`

```java
IntStream
```

Example:

```java
IntStream result =
        employees.stream()
                 .mapToInt(Employee::getSalary);
```

Why is that useful?

Because `IntStream` has primitive-specialized operations:

```java
sum()
average()
min()
max()
```

and avoids unnecessary boxing in numeric pipelines.

We'll go deep into this next.

---

# 16. Internal Execution Trace

Consider:

```java
List<Integer> numbers =
        List.of(1, 2, 3, 4, 5);

int result =
        numbers.stream()
               .filter(n -> n % 2 == 0)
               .map(n -> n * 10)
               .findFirst()
               .orElse(0);
```

Don't imagine:

```text
List
 ↓
filter → creates another List
 ↓
map → creates another List
 ↓
findFirst
```

Conceptually, Stream processing can flow element-by-element.

For `1`:

```text
1
 ↓
filter → false
 ↓
discard
```

`2`:

```text
2
 ↓
filter → true
 ↓
map → 20
 ↓
findFirst → 20
```

Now `findFirst()` has what it needs, so processing can stop.

This demonstrates the power of:

**lazy + fused pipeline + short-circuiting terminal operation.**

---

# 17. Complexity

For:

```java
list.stream()
    .map(x -> transformation(x))
    .toList();
```

If there are `n` elements and transformation is `O(1)`:

```text
Time:  O(n)
```

The mapper itself doesn't need to maintain significant state:

```text
Additional processing state: O(1)
```

But if you collect the results:

```java
.toList()
```

the resulting collection requires:

```text
O(n)
```

space.

So don't simply say:

> "map is O(1)."

That's a common interview mistake.

Better answer:

> "`map()` processes each element once, so the transformation is O(n) overall for n elements, assuming the mapper itself is O(1). Additional mapper state is generally O(1), while collecting the transformed output requires O(n) space."

That's a **Senior-level answer**.

---

# 18. Production Considerations

### 1. Avoid side effects

Bad:

```java
.map(e -> {
    database.save(e);
    return e;
})
```

Now your transformation has a side effect.

This becomes especially dangerous with parallel streams.

Prefer:

```text
Stream pipeline → transformation
Side effects → explicit application/service logic
```

---

### 2. Avoid expensive transformation before filtering

Prefer:

```java
.filter(...)
.map(expensiveTransformation)
```

when logically valid.

---

### 3. Don't use streams just to look clever

This:

```java
employees.stream()
         .map(Employee::getName)
         .toList();
```

is clean.

But forcing a complicated multi-step business workflow into one enormous stream can become difficult to debug and maintain.

---

# 19. Senior Interview Questions

Now that the concept is understood, these are the questions I'd expect EPAM/TCS-style interviewers to ask:

### Q1. What does `map()` do?

> It transforms each element of a stream using a `Function<T,R>` and produces a new stream containing the transformed elements.

---

### Q2. Is `map()` intermediate or terminal?

**Intermediate.**

Therefore it is lazy.

---

### Q3. Can `map()` change the type?

**Yes.**

```java
Stream<Employee>
```

can become:

```java
Stream<String>
```

with:

```java
.map(Employee::getName)
```

---

### Q4. Difference between `map()` and `filter()`?

```text
filter → decides whether an element remains
map    → transforms an element
```

---

### Q5. Difference between `map()` and `flatMap()`?

```text
map:
A → B

flatMap:
A → Stream<B>
then flatten all streams
```

We'll do `flatMap()` properly later.

---

### Q6. Why would you put `filter()` before `map()`?

To reduce the number of elements entering potentially expensive transformations, when the operations can safely be reordered.

---

### Q7. Is `map()` lazy?

Yes. It doesn't execute until a terminal operation triggers the pipeline.

---

### Q8. Does `map()` modify the original collection?

No. The stream pipeline produces transformed elements; the original collection isn't inherently modified.

---

### Q9. What happens if the mapper returns `null`?

An object stream can contain `null`, so `map()` can produce null elements. But downstream operations may not handle them safely—for example:

```java
.map(Employee::getName)
.map(String::toUpperCase)
```

will fail if `getName()` returns `null`.

---

# 20. One Interview-Level Example

This combines everything we've learned:

```java
List<Employee> employees = ...;

List<String> highPaidEmployeeNames =
        employees.stream()
                 .filter(e -> e.getSalary() > 100000)
                 .map(Employee::getName)
                 .map(String::toUpperCase)
                 .toList();
```

Think about the types at every stage:

```text
employees
    ↓
Stream<Employee>
    ↓
filter(...)
    ↓
Stream<Employee>
    ↓
map(Employee::getName)
    ↓
Stream<String>
    ↓
map(String::toUpperCase)
    ↓
Stream<String>
    ↓
toList()
    ↓
List<String>
```

That ability to **track the type through every pipeline stage** is extremely important for Stream API coding interviews.

---

# Notes to remember

```text
map()
─────
Purpose:
    Transform every element.

Signature:
    Stream<R> map(Function<? super T, ? extends R> mapper)

Input:
    T

Output:
    R

Example:
    employees.stream()
             .map(Employee::getName)

    Employee → String

Properties:
    Intermediate
    Lazy
    Stateless
    Can change element type
    Usually preserves element count

filter:
    T → boolean
    selects elements

map:
    T → R
    transforms elements

flatMap:
    T → Stream<R>
    transforms + flattens

Complexity:
    O(n) overall if mapper is O(1)

Common production rule:
    Filter early when logically possible,
    then perform expensive transformations.
```

## Next: `mapToInt()` / `mapToLong()` / `mapToDouble()`

This is an important jump because it connects **Streams + primitive types + boxing/unboxing + performance + `IntStream.sum()/average()/max()`**, and EPAM can absolutely ask why you'd use `mapToInt()` instead of `map()`.