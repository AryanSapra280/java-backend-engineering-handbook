# `groupingBy()` ⭐⭐⭐⭐⭐

This is one of the **most important Stream API operations for coding interviews**.

The basic idea:

> **Take elements and group them by some key.**

---

## 1. Basic example

Suppose we have employees:

```java
Employee("A", "IT")
Employee("B", "HR")
Employee("C", "IT")
Employee("D", "Finance")
```

We want:

```text
IT      → [A, C]
HR      → [B]
Finance → [D]
```

Use:

```java
Map<String, List<Employee>> result =
    employees.stream()
             .collect(
                 Collectors.groupingBy(Employee::getDepartment)
             );
```

The mental model:

```text
Employee
   ↓
extract department
   ↓
group by department
   ↓
Map<Department, List<Employee>>
```

---

# 2. What does `groupingBy()` actually produce?

This:

```java
Collectors.groupingBy(Employee::getDepartment)
```

conceptually means:

```text
KEY:
    department

VALUE:
    List of employees belonging to that department
```

So the resulting type is:

```java
Map<String, List<Employee>>
```

This type is **very important** to recognize in interviews.

---

# 3. Why not `toMap()`?

Suppose:

```text
A → IT
B → IT
```

With `toMap()`:

```text
IT → ?
```

You have multiple employees for the same key, so you need a merge function.

But with `groupingBy()`:

```text
IT → [A, B]
```

That's exactly what grouping is designed for.

### Mental model

```text
toMap()
   ↓
Key → Value

groupingBy()
   ↓
Key → Collection of Values
```

---

# 4. Grouping by salary range

You don't have to group only by a direct field.

For example:

```java
Map<String, List<Employee>> result =
    employees.stream()
             .collect(
                 Collectors.groupingBy(
                     e -> e.getSalary() > 100000
                          ? "HIGH"
                          : "LOW"
                 )
             );
```

Result:

```text
HIGH → [Employee1, Employee3]
LOW  → [Employee2, Employee4]
```

The classifier function determines the key.

---

# 5. `groupingBy()` has a classifier

The first argument:

```java
Employee::getDepartment
```

is called the **classifier**.

It answers:

> "Which group does this element belong to?"

Conceptually:

```text
Employee A → classifier → IT
Employee B → classifier → HR
Employee C → classifier → IT
```

Then:

```text
IT → [A,C]
HR → [B]
```

---

# 6. `groupingBy()` + `counting()` ⭐⭐⭐⭐⭐

This is a **very common interview pattern**.

Suppose:

```text
IT → 3 employees
HR → 5 employees
Finance → 2 employees
```

You don't need the actual employee lists.

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
IT       → 3
HR       → 5
Finance  → 2
```

This is called a **downstream collector**.

Mental model:

```text
groupingBy(department,
           counting())

             ↓

department
     ↓
group
     ↓
count elements
```

---

# 7. `groupingBy()` + `mapping()` ⭐⭐⭐⭐

Suppose you want:

```text
IT → [Aryan, Rahul]
HR → [John, Mike]
```

rather than entire Employee objects.

```java
Map<String, List<String>> result =
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

So:

```text
groupingBy()
     ↓
department

mapping()
     ↓
extract name

toList()
     ↓
collect names
```

This pattern is worth knowing.

---

# 8. `groupingBy()` + `maxBy()` ⭐⭐⭐⭐⭐

Very interview-friendly:

> Find the highest-paid employee in each department.

```java
Map<String, Optional<Employee>> result =
    employees.stream()
             .collect(
                 Collectors.groupingBy(
                     Employee::getDepartment,
                     Collectors.maxBy(
                         Comparator.comparingInt(Employee::getSalary)
                     )
                 )
             );
```

Result conceptually:

```text
IT      → Optional(Employee with highest salary)
HR      → Optional(Employee with highest salary)
Finance → Optional(Employee with highest salary)
```

Why `Optional<Employee>`?

Because `maxBy()` itself returns an `Optional`.

---

# 9. `groupingBy()` + `summingInt()`

Suppose you want:

> Total salary per department.

```java
Map<String, Integer> totalSalary =
    employees.stream()
             .collect(
                 Collectors.groupingBy(
                     Employee::getDepartment,
                     Collectors.summingInt(Employee::getSalary)
                 )
             );
```

Result:

```text
IT      → 500000
HR      → 300000
Finance → 200000
```

This is another very common coding question.

---

# 10. Nested grouping ⭐⭐⭐⭐⭐

Suppose you want:

```text
Department
    ↓
Location
    ↓
Employees
```

You can do:

```java
Map<String, Map<String, List<Employee>>> result =
    employees.stream()
             .collect(
                 Collectors.groupingBy(
                     Employee::getDepartment,
                     Collectors.groupingBy(Employee::getLocation)
                 )
             );
```

Result:

```text
IT
 ├── Bangalore → [A, B]
 └── Pune      → [C]

HR
 ├── Bangalore → [D]
 └── Mumbai    → [E]
```

The important thing is recognizing the resulting type:

```java
Map<String, Map<String, List<Employee>>>
```

---

# 11. `groupingBy()` with parallel streams ⭐⭐⭐⭐⭐

You specifically wanted this for every operation.

You can do:

```java
Map<String, List<Employee>> result =
    employees.parallelStream()
             .collect(
                 Collectors.groupingBy(Employee::getDepartment)
             );
```

Conceptually:

```text
                    Employees
                       ↓
                  split partitions
                  /      |       \
                 /       |        \
             Part A    Part B    Part C
                ↓         ↓         ↓
             local      local      local
             groups     groups     groups
                \         |         /
                 \        |        /
                    combine
                       ↓
                final grouping map
```

Each partition can build partial groups and then the framework combines them.

### But:

Parallel grouping is **not automatically faster**.

There is overhead from:

```text
partitioning
+
creating partial maps
+
combining maps
```

For a small collection:

```java
employees.parallelStream()
```

could actually be slower than:

```java
employees.stream()
```

---

# 12. `groupingBy()` vs `groupingByConcurrent()` ⭐⭐⭐

This is a good senior-level interview distinction.

Normal:

```java
Collectors.groupingBy(...)
```

produces a regular Map and the parallel implementation can combine partial maps.

There is also:

```java
Collectors.groupingByConcurrent(...)
```

which is designed for concurrent grouping and produces a `ConcurrentMap`.

Example:

```java
ConcurrentMap<String, List<Employee>> result =
    employees.parallelStream()
             .collect(
                 Collectors.groupingByConcurrent(
                     Employee::getDepartment
                 )
             );
```

The important point isn't:

> "`groupingByConcurrent()` is always faster."

Instead:

> **It is designed for concurrent accumulation and can avoid some of the map-merging structure of ordinary `groupingBy()` when the stream is suitable for concurrent collection.**

Ordering semantics also differ, so don't substitute it blindly.

---

# 13. Complexity

For ordinary grouping with a hash-based map:

```text
Expected time: O(n)
Space: O(n)
```

Why O(n) space?

Because you're storing the groups and their elements.

With:

```java
groupingBy(...)
```

every element ends up in some group.

The downstream collector can change the practical cost.

For example:

```java
groupingBy(..., counting())
```

doesn't need to retain every element in the final grouped value; it only needs counts.

---

# 14. Very common coding patterns

You should recognize these immediately:

### Group employees by department

```java
groupingBy(Employee::getDepartment)
```

### Count employees per department

```java
groupingBy(
    Employee::getDepartment,
    counting()
)
```

### Sum salaries per department

```java
groupingBy(
    Employee::getDepartment,
    summingInt(Employee::getSalary)
)
```

### Names per department

```java
groupingBy(
    Employee::getDepartment,
    mapping(Employee::getName, toList())
)
```

### Highest salary per department

```java
groupingBy(
    Employee::getDepartment,
    maxBy(Comparator.comparingInt(Employee::getSalary))
)
```

### Department → location → employees

```java
groupingBy(
    Employee::getDepartment,
    groupingBy(Employee::getLocation)
)
```

---

# 🔥 EPAM interview mental model

If they give you:

> "Group employees by department and count them."

Immediately think:

```java
employees.stream()
         .collect(
             groupingBy(
                 Employee::getDepartment,
                 counting()
             )
         );
```

If they say:

> "Group by department and find highest salary."

Think:

```text
groupingBy
    +
maxBy
```

If they say:

> "Group by department but only return employee names."

Think:

```text
groupingBy
    +
mapping
    +
toList
```

This **`groupingBy + downstream collector` pattern** is one of the most valuable Stream API patterns to know.

---

## Next

We've now covered the big collector:

```text
collect()
  ↓
toList / toSet
  ↓
toMap
  ↓
groupingBy
```

Next is **`partitioningBy()`**, which is simpler:

```text
groupingBy → potentially MANY groups
partitioningBy → exactly TWO groups: true / false
```

Then we'll cover the remaining downstream collectors (`joining`, `mapping`, `counting`, `summarizing`, `collectingAndThen`) and move toward **Optional + Streams and parallel streams**.