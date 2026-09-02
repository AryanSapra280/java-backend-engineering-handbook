# Java 8 — Stream Collectors

## 1. What Is `collect()`?

`collect()` is a **terminal Stream operation** used to accumulate Stream elements into a final result.

Example:

```java
List<Integer> result =
    numbers.stream()
           .filter(x -> x % 2 == 0)
           .collect(Collectors.toList());
```

Mental model:

```text
Stream
   ↓
collect()
   ↓
Collector / recipe
   ↓
Final result
```

---

# 2. `collect()` vs `Collectors`

This distinction is important.

### `collect()`

```java
stream.collect(...)
```

is a **terminal Stream operation**.

### `Collectors`

```java
Collectors.toList()
Collectors.toSet()
Collectors.toMap()
Collectors.groupingBy()
```

provides predefined **Collector implementations / recipes**.

Example:

```java
stream.collect(
    Collectors.toList()
);
```

Think:

```text
Stream
  ↓
collect()
  ↓
"How should I accumulate these elements?"
  ↓
Collectors.toList()
  ↓
List
```

---

# 3. `Collectors.toList()` ⭐⭐⭐

Java 8 style:

```java
List<Integer> result =
    numbers.stream()
           .filter(x -> x % 2 == 0)
           .collect(Collectors.toList());
```

For:

```text
[1,2,3,4,5,6]
```

result:

```text
[2,4,6]
```

The Collector tells `collect()`:

> Accumulate the elements into a List.

---

# 4. `Collectors.toSet()`

If you want a Set:

```java
Set<Integer> result =
    numbers.stream()
           .collect(Collectors.toSet());
```

Example:

```java
List<Integer> numbers =
    List.of(1, 2, 2, 3, 3, 4);

Set<Integer> result =
    numbers.stream()
           .collect(Collectors.toSet());
```

Result conceptually:

```text
[1,2,3,4]
```

Duplicates are removed because the result is a Set.

---

# 5. `toSet()` and `equals()` / `hashCode()`

When collecting objects into a Set, the Set's equality semantics matter.

For example:

```java
employees.stream()
         .collect(Collectors.toSet());
```

Whether two Employee objects are considered duplicates depends on their equality definition.

This connects to Collections:

```text
HashSet
   ↓
equals() + hashCode()

Collectors.toSet()
   ↓
Set semantics
   ↓
equals() + hashCode()
```

---

# 6. `Collectors.toMap()` ⭐⭐⭐

This is one of the most important Collectors for interviews.

Suppose:

```java
class Employee {

    private int id;
    private String name;

    public int getId() {
        return id;
    }

    public String getName() {
        return name;
    }
}
```

And:

```java
List<Employee> employees;
```

We want:

```text
employee ID → employee name
```

Use:

```java
Map<Integer, String> employeeMap =
    employees.stream()
             .collect(
                 Collectors.toMap(
                     Employee::getId,
                     Employee::getName
                 )
             );
```

Conceptually:

```text
Employee
    ↓
getId()
    ↓
KEY

Employee
    ↓
getName()
    ↓
VALUE

        ↓

Map<Integer, String>
```

---

# 7. `toMap()` Basic Signature

Conceptually:

```java
Collectors.toMap(
    keyMapper,
    valueMapper
)
```

The two functions are:

```text
Object → Key
Object → Value
```

Example:

```java
Collectors.toMap(
    Employee::getId,
    Employee::getName
)
```

means:

```text
Employee → ID
Employee → Name
```

---

# 8. `toMap()` Duplicate Key Problem ⭐⭐⭐

This is a very common interview question.

Suppose:

```text
Employee A → ID 101
Employee B → ID 102
Employee C → ID 101
```

Now:

```java
Collectors.toMap(
    Employee::getId,
    Employee::getName
)
```

has a problem.

Two employees have:

```text
ID = 101
```

The Collector doesn't know which value should be stored.

This can result in:

```text
IllegalStateException
```

due to the duplicate key.

---

# 9. Handling Duplicate Keys

Provide a merge function:

```java
Map<Integer, String> employeeMap =
    employees.stream()
             .collect(
                 Collectors.toMap(
                     Employee::getId,
                     Employee::getName,
                     (oldValue, newValue) -> oldValue
                 )
             );
```

This means:

```text
duplicate key
      ↓
keep old value
```

Alternatively:

```java
(oldValue, newValue) -> newValue
```

means:

```text
duplicate key
      ↓
keep new value
```

---

# 10. `toMap()` With Merge Function

Mental model:

```java
Collectors.toMap(
    keyMapper,
    valueMapper,
    mergeFunction
)
```

Think:

```text
Object
   ↓
keyMapper
   ↓
KEY

Object
   ↓
valueMapper
   ↓
VALUE

Duplicate KEY?
   ↓
mergeFunction
   ↓
choose value
```

This is an important interview pattern.

---

# 11. `Collectors.joining()`

Used to combine Strings into one String.

Example:

```java
List<String> names =
    List.of("Aryan", "Rahul", "Amit");

String result =
    names.stream()
         .collect(Collectors.joining());
```

Result:

```text
AryanRahulAmit
```

---

# 12. `joining()` With Delimiter

```java
String result =
    names.stream()
         .collect(
             Collectors.joining(", ")
         );
```

Result:

```text
Aryan, Rahul, Amit
```

---

# 13. `joining()` With Prefix and Suffix

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
[Aryan, Rahul, Amit]
```

Conceptually:

```text
joining(
    delimiter,
    prefix,
    suffix
)
```

---

# 14. `Collectors.groupingBy()` ⭐⭐⭐

One of the most important Collectors.

Suppose:

```java
class Employee {

    private String department;

    public String getDepartment() {
        return department;
    }
}
```

And:

```java
List<Employee> employees;
```

We want:

```text
department → employees
```

Use:

```java
Map<String, List<Employee>> employeesByDepartment =
    employees.stream()
             .collect(
                 Collectors.groupingBy(
                     Employee::getDepartment
                 )
             );
```

---

# 15. `groupingBy()` Mental Model

This:

```java
Collectors.groupingBy(
    Employee::getDepartment
)
```

means:

> Group employees according to the department returned by `getDepartment()`.

Conceptually:

```text
Employee
    ↓
getDepartment()
    ↓
GROUP KEY
    ↓
Map<Key, List<Employee>>
```

Example:

```text
IT
 ├── Employee A
 ├── Employee B

HR
 ├── Employee C

Finance
 ├── Employee D
 ├── Employee E
```

Result:

```java
Map<String, List<Employee>>
```

---

# 16. Grouping by Another Property

You can group by any property.

For example:

```java
Map<String, List<Employee>> employeesByLocation =
    employees.stream()
             .collect(
                 Collectors.groupingBy(
                     Employee::getLocation
                 )
             );
```

Result conceptually:

```text
Bangalore → [...]
Noida     → [...]
Delhi     → [...]
```

---

# 17. `groupingBy()` With `counting()` ⭐⭐⭐

Suppose you don't want the employees.

You want:

```text
department → employee count
```

Use:

```java
Map<String, Long> countByDepartment =
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
IT       → 25
Finance  → 12
HR       → 8
```

This is a very common interview pattern.

---

# 18. Why Is `counting()` Nested?

Look at:

```java
Collectors.groupingBy(
    Employee::getDepartment,
    Collectors.counting()
)
```

There are two levels:

```text
groupingBy()
    ↓
create groups

counting()
    ↓
within each group, count elements
```

Mental model:

```text
Employees
    ↓
group by department
    ↓
for each department
    ↓
count employees
```

---

# 19. `groupingBy()` With `mapping()`

Suppose we want:

```text
department → employee names
```

instead of:

```text
department → Employee objects
```

Use:

```java
Map<String, List<String>> namesByDepartment =
    employees.stream()
             .collect(
                 Collectors.groupingBy(
                     Employee::getDepartment,
                     Collectors.mapping(
                         Employee::getName,
                         Collectors.toList()
                     )
                 )
             );
```

Conceptually:

```text
Employee
   ↓
group by department
   ↓
extract name
   ↓
collect names into List
```

Result:

```text
IT
 ├── Aryan
 ├── Rahul

HR
 ├── Amit
```

---

# 20. `Collectors.partitioningBy()` ⭐⭐

`partitioningBy()` divides elements into exactly two groups based on a Predicate.

Example:

```java
Map<Boolean, List<Integer>> result =
    numbers.stream()
           .collect(
               Collectors.partitioningBy(
                   x -> x % 2 == 0
               )
           );
```

Conceptually:

```text
true
 ↓
even numbers

false
 ↓
odd numbers
```

For:

```text
[1,2,3,4,5,6]
```

result:

```text
true  → [2,4,6]
false → [1,3,5]
```

---

# 21. `groupingBy()` vs `partitioningBy()` ⭐⭐⭐

## `groupingBy()`

Can create many groups.

Example:

```java
Collectors.groupingBy(
    Employee::getDepartment
)
```

Possible groups:

```text
IT
HR
Finance
Sales
Operations
...
```

Result:

```java
Map<String, List<Employee>>
```

---

## `partitioningBy()`

Creates two logical groups:

```text
true
false
```

Example:

```java
Collectors.partitioningBy(
    e -> e.getSalary() > 1000000
)
```

Result:

```java
Map<Boolean, List<Employee>>
```

---

## Easy Memory Trick

```text
groupingBy()
    → MANY categories

partitioningBy()
    → TWO categories
```

---

# 22. `Collectors.summingInt()`

Suppose you want total salaries.

```java
int totalSalary =
    employees.stream()
             .collect(
                 Collectors.summingInt(
                     Employee::getSalary
                 )
             );
```

Mental model:

```text
Employee
   ↓
salary
   ↓
sum
```

There are also:

```java
Collectors.summingLong()
Collectors.summingDouble()
```

---

# 23. `Collectors.averagingInt()`

Calculate average salary:

```java
double averageSalary =
    employees.stream()
             .collect(
                 Collectors.averagingInt(
                     Employee::getSalary
                 )
             );
```

Also available:

```java
Collectors.averagingLong()
Collectors.averagingDouble()
```

---

# 24. `Collectors.summarizingInt()`

If you want multiple statistics:

```java
IntSummaryStatistics stats =
    employees.stream()
             .collect(
                 Collectors.summarizingInt(
                     Employee::getSalary
                 )
             );
```

Then:

```java
stats.getCount();
stats.getSum();
stats.getMin();
stats.getMax();
stats.getAverage();
```

This gives you:

```text
count
sum
min
max
average
```

all together.

This is useful to know, but lower priority than:

```text
toMap()
groupingBy()
partitioningBy()
joining()
```

for your interview.

---

# 25. Most Important Collectors for MUFG

Don't try to memorize every Collector.

## ⭐⭐⭐ Must Know

```java
Collectors.toList()
Collectors.toSet()
Collectors.toMap()
Collectors.groupingBy()
Collectors.partitioningBy()
Collectors.joining()
```

## ⭐⭐ Know Conceptually

```java
Collectors.counting()
Collectors.mapping()
Collectors.summingInt()
Collectors.averagingInt()
```

## 🟢 Recognize

```java
Collectors.summarizingInt()
```

---

# 26. Complete Example

Suppose:

```java
List<Employee> employees;
```

We want employees earning more than ₹10 LPA, grouped by department.

```java
Map<String, List<Employee>> result =
    employees.stream()
             .filter(e -> e.getSalary() > 1000000)
             .collect(
                 Collectors.groupingBy(
                     Employee::getDepartment
                 )
             );
```

Pipeline:

```text
Employees
    ↓
filter salary > 10L
    ↓
group by department
    ↓
Map<String, List<Employee>>
```

This combines concepts we already learned:

```text
filter()
    ↓
Intermediate operation

collect()
    ↓
Terminal operation

groupingBy()
    ↓
Collector
```

---

# 27. Another Important Example

Count employees earning more than ₹10 LPA by department:

```java
Map<String, Long> result =
    employees.stream()
             .filter(e -> e.getSalary() > 1000000)
             .collect(
                 Collectors.groupingBy(
                     Employee::getDepartment,
                     Collectors.counting()
                 )
             );
```

Pipeline:

```text
Employees
    ↓
filter
    ↓
groupBy department
    ↓
count each group
    ↓
Map<String, Long>
```

---

# 28. `collect()` vs `reduce()`

This is an important interview distinction.

## `reduce()`

Usually combines elements into a single result:

```java
int sum =
    numbers.stream()
           .reduce(0, Integer::sum);
```

Conceptually:

```text
many values
    ↓
combine
    ↓
one value
```

---

## `collect()`

Designed for mutable accumulation into result containers/structures using a Collector.

Example:

```java
List<Integer> result =
    numbers.stream()
           .collect(Collectors.toList());
```

or:

```java
Map<String, List<Employee>> result =
    employees.stream()
             .collect(
                 Collectors.groupingBy(
                     Employee::getDepartment
                 )
             );
```

Mental model:

```text
reduce()
    → combine values

collect()
    → accumulate/build a result structure
```

Don't oversimplify this into "reduce is immutable and collect is mutable"; the important interview distinction is their intended reduction/accumulation models and Collector behavior.

---

# 29. Interview Questions

## Basic

1. What is `collect()`?
2. What is `Collectors`?
3. What is the difference between `collect()` and `Collectors`?
4. How do you collect a Stream into a List?
5. How do you collect a Stream into a Set?
6. What does `joining()` do?

---

## Important

7. How does `Collectors.toMap()` work?
8. What happens if `toMap()` encounters duplicate keys?
9. How do you handle duplicate keys?
10. What is `groupingBy()`?
11. What type does simple `groupingBy()` usually produce?
12. Difference between `groupingBy()` and `partitioningBy()`?
13. How would you count employees by department?
14. How would you group employees by department and return only their names?
15. How would you calculate total salary?
16. How would you calculate average salary?
17. What does `joining()` do?
18. What is the purpose of a merge function in `toMap()`?
19. Difference between `reduce()` and `collect()`?

---

# 30. Coding Practice

## Q1 — Basic

Given:

```java
List<Integer> numbers =
    List.of(1, 2, 2, 3, 3, 4);
```

Convert it into:

```java
Set<Integer>
```

using `Collectors`.

Expected:

```text
[1,2,3,4]
```

---

## Q2 — Basic

Given:

```java
List<String> names =
    List.of("Aryan", "Rahul", "Amit");
```

Create:

```text
Aryan, Rahul, Amit
```

using:

```java
Collectors.joining()
```

---

## Q3 — Medium

Given:

```java
class Employee {

    int id;
    String name;

    public int getId() {
        return id;
    }

    public String getName() {
        return name;
    }
}
```

Create:

```java
Map<Integer, String>
```

where:

```text
employee ID → employee name
```

using:

```java
Collectors.toMap()
```

---

## Q4 — Important

What happens if two Employees have the same ID?

Modify your solution so that:

```text
duplicate ID
    ↓
keep the old employee name
```

---

## Q5 — Medium

Given:

```java
List<Employee> employees;
```

Group employees by department.

Expected type:

```java
Map<String, List<Employee>>
```

---

## Q6 — Important

Count employees in each department.

Expected:

```java
Map<String, Long>
```

Hint:

```java
Collectors.groupingBy(
    ...,
    Collectors.counting()
)
```

---

## Q7 — Medium

Partition employees into:

```text
salary > 10 LPA
salary <= 10 LPA
```

using:

```java
Collectors.partitioningBy()
```

---

## Q8 — Medium

Create:

```java
Map<String, List<String>>
```

where:

```text
department → employee names
```

Use:

```java
groupingBy()
mapping()
toList()
```

---

# 31. Quick Revision Table

| Collector | Purpose | Typical Result |
|---|---|---|
| `toList()` | Collect into List | `List<T>` |
| `toSet()` | Collect into Set | `Set<T>` |
| `toMap()` | Create Map | `Map<K,V>` |
| `groupingBy()` | Group elements | `Map<K,List<T>>` |
| `partitioningBy()` | Split into two groups | `Map<Boolean,List<T>>` |
| `joining()` | Combine Strings | `String` |
| `counting()` | Count | `Long` |
| `mapping()` | Transform inside another Collector | Depends |
| `summingInt()` | Sum ints | `Integer` |
| `averagingInt()` | Average ints | `Double` |
| `summarizingInt()` | Multiple statistics | `IntSummaryStatistics` |

---

# 32. Final Mental Model

```text
Stream
  ↓
collect()
  ↓
Collector
  ↓
result
```

### Common recipes

```text
toList()
    → List

toSet()
    → Set

toMap()
    → Map

groupingBy()
    → MANY groups

partitioningBy()
    → TWO groups

joining()
    → String
```

### Most important interview patterns

```java
// List
stream.collect(Collectors.toList());

// Set
stream.collect(Collectors.toSet());

// Map
stream.collect(
    Collectors.toMap(
        keyMapper,
        valueMapper
    )
);

// Map with duplicate-key handling
stream.collect(
    Collectors.toMap(
        keyMapper,
        valueMapper,
        (oldValue, newValue) -> oldValue
    )
);

// Group
stream.collect(
    Collectors.groupingBy(
        classifier
    )
);

// Group + count
stream.collect(
    Collectors.groupingBy(
        classifier,
        Collectors.counting()
    )
);

// Partition
stream.collect(
    Collectors.partitioningBy(
        predicate
    )
);

// Join
stream.collect(
    Collectors.joining(", ")
);
```

---

# 33. Stream Learning Progress

```text
Stream Fundamentals                 ✅
Stream Sources                      ✅
Intermediate Operations             ✅
Terminal Operations                 ✅
Collectors                          ✅

        ↓

Optional                            ← NEXT

        ↓

Java 8 Complete

        ↓

Multithreading / Concurrency
```

**Java 8 — Collectors = DONE ✅**