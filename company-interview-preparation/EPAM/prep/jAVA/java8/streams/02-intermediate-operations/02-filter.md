# STREAM API — 3. `filter()`

Now we start the **actual Stream operations**.

`filter()` is one of the most important operations because it connects directly to the **`Predicate` functional interface** we already learned.

---

## 3.1 Problem

Suppose we have:

```java
List<Integer> numbers =
        List.of(10, 15, 20, 25, 30);
```

We want only the even numbers:

```text
10
20
30
```

Traditional approach:

```java
List<Integer> result = new ArrayList<>();

for (Integer n : numbers) {
    if (n % 2 == 0) {
        result.add(n);
    }
}
```

With Streams:

```java
List<Integer> result =
        numbers.stream()
               .filter(n -> n % 2 == 0)
               .toList();
```

---

# 3.2 Why does `filter()` exist?

The purpose of `filter()` is:

> **Keep elements that satisfy a condition and discard the rest.**

The condition is represented by a `Predicate`.

Remember:

```text
Predicate<T>
      ↓
T → boolean
```

So:

```java
n -> n % 2 == 0
```

is effectively:

```text
Integer → boolean
```

Therefore it can be passed to:

```java
filter()
```

---

# 3.3 Method signature

Conceptually:

```java
Stream<T> filter(Predicate<? super T> predicate)
```

The important part is:

```java
filter(Predicate)
```

It receives a condition and returns another Stream.

Therefore:

```java
numbers.stream()
       .filter(...)
```

is still a:

```text
Stream<Integer>
```

This is why we can continue chaining:

```java
.filter(...)
.map(...)
.sorted(...)
```

---

# 3.4 How `filter()` works internally

Suppose:

```java
List<Integer> numbers =
        List.of(1, 2, 3, 4, 5);
```

and:

```java
numbers.stream()
       .filter(n -> n % 2 == 0)
       .forEach(System.out::println);
```

Conceptually:

```text
1
 ↓
predicate
 ↓
false
 ↓
discard

2
 ↓
predicate
 ↓
true
 ↓
pass downstream

3
 ↓
predicate
 ↓
false
 ↓
discard

4
 ↓
predicate
 ↓
true
 ↓
pass downstream

5
 ↓
predicate
 ↓
false
 ↓
discard
```

Output:

```text
2
4
```

The important mental model:

> `filter()` does not transform the element. It decides whether the element is allowed to continue through the pipeline.

---

# 3.5 `filter()` is lazy

Consider:

```java
numbers.stream()
       .filter(n -> {
           System.out.println("Checking " + n);
           return n % 2 == 0;
       });
```

Nothing happens yet.

Why?

Because `filter()` is an **intermediate operation**.

Execution starts when a terminal operation appears:

```java
numbers.stream()
       .filter(n -> {
           System.out.println("Checking " + n);
           return n % 2 == 0;
       })
       .forEach(System.out::println);
```

Now the predicate executes.

---

# 3.6 `filter()` is stateless

`filter()` is considered a **stateless intermediate operation**.

For:

```java
n -> n % 2 == 0
```

the decision for `10` doesn't require knowing:

```text
1
2
3
4
5
6
7
8
9
11
12
...
```

It only needs:

```text
10
```

Therefore:

```text
filter()
→ stateless
→ element can generally be evaluated independently
```

This becomes particularly important when we later discuss **parallel streams**.

---

# 3.7 Multiple `filter()` operations

You can chain filters:

```java
List<Integer> result =
        numbers.stream()
               .filter(n -> n > 10)
               .filter(n -> n % 2 == 0)
               .toList();
```

Conceptually:

```text
numbers
   ↓
n > 10
   ↓
even
   ↓
result
```

For:

```text
5, 10, 12, 15, 20, 24
```

first filter:

```text
12, 15, 20, 24
```

second filter:

```text
12, 20, 24
```

---

# 3.8 Multiple conditions inside one filter

You could also write:

```java
.filter(n -> n > 10 && n % 2 == 0)
```

instead of:

```java
.filter(n -> n > 10)
.filter(n -> n % 2 == 0)
```

Both can express the same logical condition.

But separate filters can sometimes improve readability:

```java
.filter(n -> n > 10)
.filter(n -> n % 2 == 0)
```

The choice should be based on clarity rather than blindly preferring one style.

---

# 3.9 `Predicate` composition

Because `filter()` uses `Predicate`, we can use Predicate's methods:

```java
Predicate<Integer> greaterThan10 =
        n -> n > 10;

Predicate<Integer> even =
        n -> n % 2 == 0;
```

Combine them:

```java
Predicate<Integer> condition =
        greaterThan10.and(even);
```

Then:

```java
numbers.stream()
       .filter(condition)
       .toList();
```

This connects directly back to the functional-interface topic.

---

# 3.10 `Predicate.or()`

```java
Predicate<Integer> condition =
        n -> n < 10;
```

Another:

```java
Predicate<Integer> even =
        n -> n % 2 == 0;
```

Combine:

```java
condition.or(even);
```

Meaning:

```text
n < 10
OR
n is even
```

---

# 3.11 `Predicate.negate()`

Suppose:

```java
Predicate<Integer> even =
        n -> n % 2 == 0;
```

You can invert it:

```java
Predicate<Integer> odd =
        even.negate();
```

Then:

```java
numbers.stream()
       .filter(odd)
       .toList();
```

This is a useful connection between **Functional Interfaces → Predicate → Streams**.

---

# 3.12 Filtering objects

This is much more likely in a real interview.

Suppose:

```java
class Employee {

    private String name;
    private String department;
    private int salary;

    // constructor/getters
}
```

We have:

```java
List<Employee> employees;
```

Find employees earning more than ₹50,000:

```java
List<Employee> result =
        employees.stream()
                 .filter(e -> e.getSalary() > 50_000)
                 .toList();
```

The predicate is:

```java
e -> e.getSalary() > 50_000
```

which means:

```text
Employee → boolean
```

---

# 3.13 Multiple employee conditions

For example:

> Find employees from the IT department earning more than ₹50,000.

```java
List<Employee> result =
        employees.stream()
                 .filter(e -> e.getDepartment().equals("IT"))
                 .filter(e -> e.getSalary() > 50_000)
                 .toList();
```

Or:

```java
List<Employee> result =
        employees.stream()
                 .filter(e ->
                     e.getDepartment().equals("IT")
                     && e.getSalary() > 50_000
                 )
                 .toList();
```

---

# 3.14 Filtering Strings

Find names starting with `"A"`:

```java
List<String> result =
        names.stream()
             .filter(name -> name.startsWith("A"))
             .toList();
```

Find names longer than five characters:

```java
names.stream()
     .filter(name -> name.length() > 5)
     .toList();
```

Find non-empty strings:

```java
names.stream()
     .filter(name -> !name.isEmpty())
     .toList();
```

---

# 3.15 Filtering `null`

Suppose:

```java
List<String> names =
        Arrays.asList("Java", null, "Spring", null, "Kafka");
```

We can remove nulls:

```java
List<String> result =
        names.stream()
             .filter(Objects::nonNull)
             .toList();
```

Or:

```java
.filter(name -> name != null)
```

Method reference:

```java
Objects::nonNull
```

is effectively a predicate:

```text
String → boolean
```

---

# 3.16 Filtering with `Optional`-style conditions

For example:

```java
employees.stream()
         .filter(Objects::nonNull)
         .filter(e -> e.getSalary() > 50_000)
         .toList();
```

This can prevent a `NullPointerException` caused by null employee references.

However, if:

```java
e.getSalary()
```

itself can somehow involve nullable data, that needs separate handling.

Don't confuse:

```java
.filter(Objects::nonNull)
```

with general null-safety for every field.

---

# 3.17 `filter()` + `map()`

This is where Stream pipelines start becoming powerful.

Suppose:

> Find employees earning more than ₹50,000 and return their names.

```java
List<String> names =
        employees.stream()
                 .filter(e -> e.getSalary() > 50_000)
                 .map(Employee::getName)
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
map getName
   ↓
String
   ↓
List<String>
```

Notice the difference:

```text
filter
→ decides whether element continues

map
→ transforms element
```

We'll go very deep into `map()` next.

---

# 3.18 `filter()` + `findFirst()`

Suppose:

```java
Optional<Employee> employee =
        employees.stream()
                 .filter(e -> e.getSalary() > 100_000)
                 .findFirst();
```

Because `findFirst()` is short-circuiting, processing can stop once the first matching employee is found.

Conceptually:

```text
Employee 1 → reject
Employee 2 → reject
Employee 3 → accept
             ↓
          findFirst
             ↓
            STOP
```

This demonstrates how:

```text
filter()
+
lazy execution
+
short-circuiting
```

work together.

---

# 3.19 `filter()` + `anyMatch()`

Suppose:

> Does any employee earn more than ₹1 lakh?

```java
boolean exists =
        employees.stream()
                 .anyMatch(e -> e.getSalary() > 100_000);
```

This is often better than:

```java
employees.stream()
         .filter(e -> e.getSalary() > 100_000)
         .count() > 0;
```

Why?

`anyMatch()` is explicitly designed to answer an existence question and can short-circuit.

Once one employee matches:

```text
true
```

the pipeline can stop.

---

# 3.20 `filter()` + `count()`

Count employees earning above ₹50,000:

```java
long count =
        employees.stream()
                 .filter(e -> e.getSalary() > 50_000)
                 .count();
```

Pipeline:

```text
Employee
 ↓
salary > 50000?
 ↓
yes → count
no  → discard
```

---

# 3.21 Complexity

For:

```java
numbers.stream()
       .filter(condition)
```

if there are `n` elements:

```text
Time: O(n)
```

because each element may need to be evaluated.

The filter itself generally requires:

```text
Space: O(1)
```

additional state, assuming the predicate itself doesn't allocate/store data.

But if you do:

```java
.toList()
```

then the resulting collection can require:

```text
O(k)
```

where `k` is the number of elements that pass the filter.

So don't simply say:

> "Stream filter uses O(n) space."

The space depends on what the **rest of the pipeline** does.

---

# 3.22 Production consideration — filter early

Consider:

```java
employees.stream()
         .map(this::expensiveTransformation)
         .filter(e -> e.getSalary() > 50_000)
```

versus:

```java
employees.stream()
         .filter(e -> e.getSalary() > 50_000)
         .map(this::expensiveTransformation)
```

If the filter removes many elements, filtering first can avoid unnecessary expensive transformations.

Mental model:

```text
Cheap selective operation
        ↓
Expensive operation
```

can often be better.

But don't blindly optimize every pipeline—measure and preserve readability.

---

# 3.23 Production consideration — avoid side effects in predicates

Bad:

```java
.filter(e -> {
    auditDatabase(e);
    return e.getSalary() > 50_000;
})
```

Now your filtering operation has a database side effect.

This creates problems with:

- retries
- parallel execution
- reasoning about behavior
- testing
- performance

Prefer:

```java
.filter(e -> e.getSalary() > 50_000)
```

and handle side effects separately where appropriate.

---

# 3.24 Production consideration — expensive predicates

This:

```java
.filter(e -> expensiveServiceCall(e))
```

can become very expensive if you have:

```text
1 million employees
```

You might accidentally make:

```text
1 million service calls
```

A Stream doesn't automatically make an expensive predicate efficient.

Always think about the cost of the operation you're putting inside the pipeline.

---

# 3.25 Interview-level distinction: `filter()` vs `map()`

This is a very common question.

### `filter()`

```java
.filter(e -> e.getSalary() > 50_000)
```

Input:

```text
Employee
```

Output:

```text
Employee
```

Potentially fewer elements.

Conceptually:

```text
T → T
```

---

### `map()`

```java
.map(Employee::getName)
```

Input:

```text
Employee
```

Output:

```text
String
```

Generally same number of elements unless later operations change it.

Conceptually:

```text
T → R
```

So:

```text
filter
→ select

map
→ transform
```

Remember this sentence for interviews:

> **`filter()` changes the number of elements; `map()` changes the form/type/value of each element.**

---

# 3.26 Interview-level distinction: `filter()` vs `distinct()`

```java
.filter(...)
```

uses a condition you provide.

```java
.distinct()
```

removes duplicates based on equality semantics.

For example:

```java
Stream.of(1, 2, 2, 3, 3, 3)
      .distinct()
```

produces:

```text
1 2 3
```

`distinct()` is stateful because it needs to remember what it has already seen.

We'll cover it later.

---

# 3.27 Interview-level distinction: `filter()` vs `takeWhile()`

These are **not interchangeable**.

For an ordered stream:

```java
Stream.of(2, 4, 6, 7, 8, 10)
```

`filter(n -> n % 2 == 0)`:

```text
2 4 6 8 10
```

It keeps checking the entire stream.

But:

```java
.takeWhile(n -> n % 2 == 0)
```

takes elements **while the condition remains true**:

```text
2 4 6
```

Once `7` fails:

```text
STOP
```

We'll study `takeWhile()` later.

---

# 3.28 Hands-on mental execution

Consider:

```java
List<Integer> numbers =
        List.of(5, 12, 7, 20, 3, 30);

List<Integer> result =
        numbers.stream()
               .filter(n -> n > 10)
               .filter(n -> n % 2 == 0)
               .toList();
```

Process:

```text
5
 ↓
>10? NO
 ↓
discard

12
 ↓
>10? YES
 ↓
even? YES
 ↓
keep

7
 ↓
>10? NO
 ↓
discard

20
 ↓
>10? YES
 ↓
even? YES
 ↓
keep

3
 ↓
>10? NO
 ↓
discard

30
 ↓
>10? YES
 ↓
even? YES
 ↓
keep
```

Result:

```text
[12, 20, 30]
```

---

# 3.29 Notes — `filter()`

```text
filter(Predicate<T>)

Purpose:
→ select elements satisfying a condition

Predicate:
T → boolean

Example:
.filter(n -> n % 2 == 0)

Intermediate operation:
→ lazy
→ returns Stream

Stateless:
→ decision generally depends only on current element

filter:
→ can reduce number of elements
→ does not transform element type

Example:
Employee → Employee

filter + map:
Employee
 ↓
filter salary
 ↓
Employee
 ↓
map getName
 ↓
String

Complexity:
Time: O(n)
Additional filter state: generally O(1)

Production:
→ filter early when it avoids expensive work
→ avoid side effects
→ be careful with expensive predicates
```

---

# EPAM mental checklist for `filter()`

When you see:

```java
.stream()
.filter(...)
```

immediately think:

```text
1. Predicate
2. Intermediate
3. Lazy
4. Stateless
5. Selects/removes elements
6. Doesn't transform type
7. Can participate in short-circuiting
8. O(n)
9. Avoid expensive/side-effect predicates
```

---

# NEXT TOPIC — `map()`

This is the next major operation and **even more important for your Stream coding**.

We'll cover:

```text
map()
 ↓
Problem
 ↓
Function<T,R>
 ↓
Internal element transformation
 ↓
Type transformation
 ↓
Object → Object
 ↓
Object → String
 ↓
Object → Integer
 ↓
method references
 ↓
map vs filter
 ↓
map vs mapToInt
 ↓
chaining multiple maps
 ↓
production considerations
 ↓
interview coding
```

Then we'll move into **`mapToInt()`**, where primitive streams, boxing, and numerical Stream problems start becoming much clearer.