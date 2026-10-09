# `mapToInt()` / `mapToLong()` / `mapToDouble()`

This is the next important Stream topic. The key interview idea is **primitive streams vs object streams**.

---

## 1. Problem — Why does `mapToInt()` exist?

Suppose we have:

```java
List<Employee> employees = ...;
```

and every employee has:

```java
int salary;
```

We want the total salary.

You could write:

```java
int total =
        employees.stream()
                 .map(Employee::getSalary)
                 .reduce(0, Integer::sum);
```

This works.

But there is a more natural numeric Stream API:

```java
int total =
        employees.stream()
                 .mapToInt(Employee::getSalary)
                 .sum();
```

That's what `mapToInt()` is for.

---

# 2. Concept

Normal `map()` produces an **object Stream**:

```java
Stream<Integer>
```

whereas:

```java
mapToInt()
```

produces:

```java
IntStream
```

Similarly:

```text
mapToLong()    → LongStream
mapToDouble()  → DoubleStream
```

Think:

```text
map()
    T → R
    Stream<R>

mapToInt()
    T → int
    IntStream

mapToLong()
    T → long
    LongStream

mapToDouble()
    T → double
    DoubleStream
```

---

# 3. The Important Difference

Consider:

```java
List<String> numbers =
        List.of("10", "20", "30");
```

### Using `map()`

```java
List<Integer> result =
        numbers.stream()
               .map(Integer::parseInt)
               .toList();
```

The resulting type is:

```text
Stream<Integer>
```

That's an **object stream** containing `Integer` objects.

---

### Using `mapToInt()`

```java
int sum =
        numbers.stream()
               .mapToInt(Integer::parseInt)
               .sum();
```

The intermediate stream is:

```text
IntStream
```

and `sum()` directly calculates the primitive `int` sum.

---

# 4. Why Primitive Streams?

Java has primitive types:

```text
int
long
double
```

and wrapper classes:

```text
Integer
Long
Double
```

Generics cannot directly use primitives:

```java
List<int>        // ❌
Stream<int>      // ❌
```

So:

```java
Stream<Integer>
```

uses boxed `Integer` objects.

Primitive streams solve this for numeric stream processing:

```java
IntStream
LongStream
DoubleStream
```

---

# 5. Boxing and Unboxing

This is where the interview depth comes in.

Suppose:

```java
Stream<Integer>
```

contains:

```text
Integer
Integer
Integer
```

If you need an `int`, Java may need to **unbox**:

```text
Integer → int
```

Similarly, when an `int` needs to become an `Integer`:

```text
int → Integer
```

that's **boxing**.

For large numeric pipelines, repeated boxing/unboxing can create unnecessary object overhead.

Primitive streams avoid that for the stream's numeric representation.

---

# 6. Practical Example

Suppose:

```java
List<Integer> numbers =
        List.of(10, 20, 30, 40, 50);
```

### `map()`

```java
List<Integer> doubled =
        numbers.stream()
               .map(n -> n * 2)
               .toList();
```

Result:

```text
[20, 40, 60, 80, 100]
```

Type:

```text
Stream<Integer>
```

---

### `mapToInt()`

```java
int sum =
        numbers.stream()
               .mapToInt(n -> n * 2)
               .sum();
```

Result:

```text
600
```

Type of the intermediate stream:

```text
IntStream
```

---

# 7. Why Does `mapToInt()` Take a Different Functional Interface?

This is another good interview question.

`map()` expects:

```java
Function<T, R>
```

For example:

```java
Function<Employee, Integer>
```

But `mapToInt()` expects:

```java
ToIntFunction<T>
```

Conceptually:

```text
T → int
```

For example:

```java
ToIntFunction<Employee>
```

means:

```text
Employee → int
```

So:

```java
.map(Employee::getSalary)
```

uses the normal `Function` machinery conceptually as:

```text
Employee → Integer
```

while:

```java
.mapToInt(Employee::getSalary)
```

means:

```text
Employee → int
```

That's the distinction.

---

# 8. Example With Employees

Suppose:

```java
class Employee {

    private String name;
    private int salary;

    public int getSalary() {
        return salary;
    }

    public String getName() {
        return name;
    }
}
```

Now:

### Total salary

```java
int totalSalary =
        employees.stream()
                 .mapToInt(Employee::getSalary)
                 .sum();
```

### Maximum salary

```java
OptionalInt maxSalary =
        employees.stream()
                 .mapToInt(Employee::getSalary)
                 .max();
```

### Minimum salary

```java
OptionalInt minSalary =
        employees.stream()
                 .mapToInt(Employee::getSalary)
                 .min();
```

### Average salary

```java
OptionalDouble averageSalary =
        employees.stream()
                 .mapToInt(Employee::getSalary)
                 .average();
```

Notice something important:

```text
max()     → OptionalInt
min()     → OptionalInt
average() → OptionalDouble
```

Why?

Because the stream might be empty.

---

# 9. `sum()` Is Different

For:

```java
IntStream.empty().sum()
```

the result is:

```text
0
```

Whereas:

```java
IntStream.empty().max()
```

can't return an actual maximum.

Therefore:

```java
OptionalInt
```

is returned.

This is the same idea we've seen with `Optional<T>`—the result may not exist.

---

# 10. Very Important: `map()` vs `mapToInt()`

Suppose:

```java
List<Employee> employees = ...;
```

### Version 1

```java
List<Integer> salaries =
        employees.stream()
                 .map(Employee::getSalary)
                 .toList();
```

Pipeline:

```text
Employee
   ↓
Integer
   ↓
Stream<Integer>
```

### Version 2

```java
int total =
        employees.stream()
                 .mapToInt(Employee::getSalary)
                 .sum();
```

Pipeline:

```text
Employee
   ↓
int
   ↓
IntStream
   ↓
sum()
```

So don't think:

> "`mapToInt()` is just another syntax for `map()`."

It's specifically a transition from an object stream to a **primitive `IntStream`**.

---

# 11. When Should You Use Which?

### You're transforming to another object

Use:

```java
map()
```

Example:

```java
.map(Employee::getName)
```

```text
Employee → String
```

---

### You're doing integer numeric processing

Use:

```java
mapToInt()
```

Example:

```java
.mapToInt(Employee::getSalary)
```

---

### You're doing long numeric processing

Use:

```java
mapToLong()
```

Example:

```java
.mapToLong(Transaction::getAmount)
```

---

### You're doing double processing

Use:

```java
mapToDouble()
```

Example:

```java
.mapToDouble(Product::getPrice)
```

---

# 12. Real Production Example

Imagine a payment service:

```java
class Payment {
    private long amount;

    public long getAmount() {
        return amount;
    }
}
```

Suppose you want total transaction value.

```java
long total =
        payments.stream()
                .mapToLong(Payment::getAmount)
                .sum();
```

That's much clearer than:

```java
long total =
        payments.stream()
                .map(Payment::getAmount)
                .reduce(0L, Long::sum);
```

Both can be valid, but the first expresses the numeric operation directly.

For a ledger/payment system, `long` is also often preferable to `double` for monetary amounts when the value is represented in the smallest currency unit, such as paise/cents.

---

# 13. `mapToInt()` Can Be Followed by `boxed()`

This is an important trick.

Suppose:

```java
IntStream stream =
        numbers.stream()
               .mapToInt(Integer::intValue);
```

You can convert back:

```java
Stream<Integer> boxed =
        stream.boxed();
```

So:

```text
Stream<Integer>
      ↓
mapToInt()
      ↓
IntStream
      ↓
boxed()
      ↓
Stream<Integer>
```

Example:

```java
List<Integer> result =
        numbers.stream()
               .mapToInt(n -> n * 2)
               .boxed()
               .toList();
```

Result:

```text
[20, 40, 60, 80]
```

Why would you do this?

Because you've performed primitive numeric processing but eventually need an object stream or `List<Integer>`.

---

# 14. Primitive Stream → Object Stream

You can also use:

```java
mapToObj()
```

Example:

```java
List<String> result =
        IntStream.rangeClosed(1, 5)
                 .mapToObj(n -> "Number-" + n)
                 .toList();
```

Result:

```text
["Number-1", "Number-2", "Number-3", "Number-4", "Number-5"]
```

Pipeline:

```text
IntStream
   ↓
mapToObj()
   ↓
Stream<String>
```

This is useful when you've started with primitive numeric processing and then want to create objects.

---

# 15. A Very Important Interview Trap

Suppose interviewer asks:

> What's the difference between `map()` and `mapToInt()`?

Don't answer merely:

> "`mapToInt()` is faster."

That's too simplistic.

Better:

> "`map()` transforms elements into another reference type and returns `Stream<R>`, while `mapToInt()` transforms elements into primitive `int` values and returns `IntStream`. Primitive streams provide numeric terminal operations such as `sum`, `average`, `min`, and `max`, and avoid representing each stream element as an `Integer` object."

That's a much stronger answer.

---

# 16. Complexity

For:

```java
employees.stream()
         .mapToInt(Employee::getSalary)
         .sum();
```

with `n` employees:

```text
Time: O(n)
```

assuming:

```java
getSalary()
```

is O(1).

Additional algorithmic state for `sum()`:

```text
O(1)
```

because we only need a running sum.

For:

```java
.toList()
```

the result requires:

```text
O(n)
```

space.

---

# 17. Practical EPAM Coding Patterns

### Sum

```java
int sum =
        numbers.stream()
               .mapToInt(Integer::intValue)
               .sum();
```

### Average

```java
double avg =
        numbers.stream()
               .mapToInt(Integer::intValue)
               .average()
               .orElse(0.0);
```

### Maximum

```java
int max =
        numbers.stream()
               .mapToInt(Integer::intValue)
               .max()
               .orElse(0);
```

### Minimum

```java
int min =
        numbers.stream()
               .mapToInt(Integer::intValue)
               .min()
               .orElse(0);
```

### Sum of even numbers

This directly connects to the HR question:

```java
int sum =
        IntStream.rangeClosed(1, 100)
                 .filter(n -> n % 2 == 0)
                 .sum();
```

Result:

```text
2550
```

Notice we don't even need `mapToInt()` here.

Why?

Because:

```java
IntStream.rangeClosed(...)
```

already gives us an `IntStream`.

---

# 18. Big Mental Model

Keep this picture:

```text
                 OBJECT STREAM
                     |
                     | map()
                     ↓
                Stream<R>


                 OBJECT STREAM
                     |
                     | mapToInt()
                     ↓
                  IntStream
                     |
          ┌──────────┼──────────┐
          ↓          ↓          ↓
        sum()      avg()      max()
```

And:

```text
Stream<T>
   |
   | mapToLong()
   ↓
LongStream
```

```text
Stream<T>
   |
   | mapToDouble()
   ↓
DoubleStream
```

---

# 19. Interview Questions You Should Know

### Q1. Why do primitive streams exist?

To support primitive numeric processing without representing every value as a wrapper object and to provide specialized numeric operations.

### Q2. What does `mapToInt()` return?

```java
IntStream
```

### Q3. What functional interface does `mapToInt()` use?

Conceptually:

```java
ToIntFunction<T>
```

which represents:

```text
T → int
```

### Q4. What is the difference between `Stream<Integer>` and `IntStream`?

```text
Stream<Integer>
    object/reference stream

IntStream
    primitive int stream
```

### Q5. How do you convert `IntStream` to `Stream<Integer>`?

```java
.boxed()
```

### Q6. How do you convert an `IntStream` to `Stream<String>`?

```java
.mapToObj(...)
```

### Q7. Why does `average()` return `OptionalDouble`?

Because the stream may be empty and therefore may have no average.

---

## Notes — lock this in

```text
map()
─────
T → R
Returns Stream<R>

mapToInt()
──────────
T → int
Returns IntStream

mapToLong()
───────────
T → long
Returns LongStream

mapToDouble()
─────────────
T → double
Returns DoubleStream

Primitive streams provide:
    sum()
    average()
    min()
    max()
    count()

Convert:
    IntStream → Stream<Integer>
    .boxed()

    IntStream → Stream<R>
    .mapToObj(...)
```

### Next topic: `flatMap()`

This one is **very important for interviews** because it tests whether you really understand the difference between **one-to-one transformation (`map`) and one-to-many transformation + flattening (`flatMap`)**. We'll do nested lists, `List<List<T>>`, strings, `Optional.flatMap()`, and a real-world employee/order example.