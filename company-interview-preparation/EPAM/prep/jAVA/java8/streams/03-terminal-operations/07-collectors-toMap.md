# `toList()` / `toSet()` → then `toMap()` ⭐⭐⭐⭐⭐

We just covered `collect()`. Now let's make the common collectors practical.

---

## 1. `toList()`

The modern Java approach:

```java
List<String> names =
    employees.stream()
             .map(Employee::getName)
             .toList();
```

Example:

```text
employees
   ↓
map(name)
   ↓
["Amit", "Rahul", "John"]
```

### Important

`Stream.toList()` returns an **unmodifiable List**.

So:

```java
List<String> names = stream.toList();

names.add("X"); // UnsupportedOperationException
```

If you need a mutable list:

```java
List<String> names =
    stream.collect(Collectors.toList());
```

Don't overstate the exact implementation/type of `Collectors.toList()`; the important interview distinction is that it **doesn't guarantee an unmodifiable result**, whereas `Stream.toList()` does.

---

# 2. `toSet()`

```java
Set<String> names =
    employees.stream()
             .map(Employee::getName)
             .collect(Collectors.toSet());
```

Duplicates are removed according to Set semantics.

```text
["Amit", "Rahul", "Amit", "John"]

        ↓ toSet()

["Amit", "Rahul", "John"]
```

Remember:

```java
Collectors.toSet()
```

does **not promise a particular Set implementation or ordering**.

If you specifically need insertion order:

```java
Set<String> names =
    employees.stream()
             .map(Employee::getName)
             .collect(Collectors.toCollection(LinkedHashSet::new));
```

---

# 3. Now the important one: `toMap()` ⭐⭐⭐⭐⭐

Suppose:

```java
List<Employee> employees;
```

and we want:

```text
employee ID → employee name
```

Use:

```java
Map<Long, String> employeeMap =
    employees.stream()
             .collect(Collectors.toMap(
                 Employee::getId,
                 Employee::getName
             ));
```

Conceptually:

```text
Employee 101, Aryan
Employee 102, Rahul
Employee 103, John

        ↓

101 → Aryan
102 → Rahul
103 → John
```

---

# 4. `toMap()` has two important functions

This:

```java
Collectors.toMap(
    Employee::getId,
    Employee::getName
)
```

means:

```text
key mapper
    ↓
Employee → ID

value mapper
    ↓
Employee → Name
```

So:

```text
Employee
   ↙    ↘
 ID     Name
 ↓       ↓
Key     Value
```

---

# 5. 🚨 Duplicate-key problem

This is the big interview question.

Suppose:

```text
Employee 101 → Aryan
Employee 101 → Rahul
```

Now:

```java
employees.stream()
         .collect(Collectors.toMap(
             Employee::getId,
             Employee::getName
         ));
```

What happens?

You get:

```text
IllegalStateException
```

because two employees are trying to create:

```text
101 → ?
```

A `Map` can't have two values for the same key.

---

# 6. How do we solve duplicate keys?

Use the **third argument: merge function**.

```java
Map<Long, String> employeeMap =
    employees.stream()
             .collect(Collectors.toMap(
                 Employee::getId,
                 Employee::getName,
                 (oldValue, newValue) -> newValue
             ));
```

Now if:

```text
101 → Aryan
101 → Rahul
```

the merge function receives:

```text
oldValue = Aryan
newValue = Rahul
```

and:

```java
(oldValue, newValue) -> newValue
```

means:

```text
keep Rahul
```

---

# 7. Keep the old value

```java
(oldValue, newValue) -> oldValue
```

Example:

```text
101 → Aryan
101 → Rahul

        ↓

101 → Aryan
```

---

# 8. Combine the values

Suppose you're creating:

```text
productId → total quantity
```

You might have:

```text
Product 101 → quantity 5
Product 101 → quantity 7
```

Then:

```java
Map<Long, Integer> quantities =
    products.stream()
            .collect(Collectors.toMap(
                Product::getId,
                Product::getQuantity,
                Integer::sum
            ));
```

Result:

```text
101 → 12
```

This connects directly to the `Integer::sum` concept we discussed earlier.

Equivalent lambda:

```java
(oldValue, newValue) -> oldValue + newValue
```

---

# 9. Why does the merge function receive two values?

Because the key already exists.

Think:

```text
key = 101

existing value = 5
new value      = 7

              ↓

merge(5, 7)

              ↓

             12
```

So:

```java
Integer::sum
```

means:

```java
(a, b) -> a + b
```

The Stream API isn't magically deciding that `b` is `1`.

**Both arguments are the values associated with the duplicate key.**

---

# 10. Very practical payment example

Suppose you have payments:

```text
Payment ID    Customer    Amount
--------------------------------
P1            C1         500
P2            C2         700
P3            C1         300
```

You want:

```text
Customer → total payment
```

You could do:

```java
Map<String, Integer> totals =
    payments.stream()
            .collect(Collectors.toMap(
                Payment::getCustomerId,
                Payment::getAmount,
                Integer::sum
            ));
```

Result:

```text
C1 → 800
C2 → 700
```

This is a **very interview-friendly example**.

---

# 11. `toMap()` with parallel streams ⭐⭐⭐

Now let's connect it to your requirement.

```java
Map<Long, String> map =
    employees.parallelStream()
             .collect(Collectors.toMap(
                 Employee::getId,
                 Employee::getName
             ));
```

The stream can conceptually:

```text
partition 1 → partial Map A
partition 2 → partial Map B
partition 3 → partial Map C
                    ↓
                 combine
                    ↓
               final Map
```

The framework has to handle duplicate keys while combining partial results.

### Important:

The merge function therefore needs to be **correct for repeated/parallel merging**.

Don't put weird stateful logic inside it.

Bad:

```java
(oldValue, newValue) -> {
    globalCounter++;
    return newValue;
}
```

Now you're introducing shared mutable state.

---

# 12. Does `toMap()` guarantee Map ordering?

No.

If you need insertion order:

```java
Map<Long, String> map =
    employees.stream()
             .collect(Collectors.toMap(
                 Employee::getId,
                 Employee::getName,
                 (a, b) -> b,
                 LinkedHashMap::new
             ));
```

The fourth argument specifies the Map implementation.

Conceptually:

```text
key mapper
value mapper
merge function
map supplier
```

---

# 13. `toMap()` overloads

Know these three forms:

### Two arguments

```java
Collectors.toMap(keyMapper, valueMapper)
```

Duplicate key → exception.

### Three arguments

```java
Collectors.toMap(
    keyMapper,
    valueMapper,
    mergeFunction
)
```

Duplicate key → handled by merge function.

### Four arguments

```java
Collectors.toMap(
    keyMapper,
    valueMapper,
    mergeFunction,
    mapSupplier
)
```

You control the Map implementation.

---

# 14. `toMap()` vs `groupingBy()` ⭐⭐⭐⭐⭐

This distinction is VERY important.

Suppose:

```text
Employee
101 → IT
102 → HR
103 → IT
```

If you want:

```text
IT → [Employee101, Employee103]
HR → [Employee102]
```

Use:

```java
groupingBy()
```

Because **one key maps to multiple values**.

But if you want:

```text
101 → Employee101
102 → Employee102
103 → Employee103
```

use:

```java
toMap()
```

Mental model:

```text
toMap()
    ↓
one key → one final value
    ↓
duplicate requires merge


groupingBy()
    ↓
one key → collection of values
```

---

# 15. Complexity

For a normal hash-based map:

```text
Time: O(n) expected
Space: O(n)
```

assuming reasonably good hashing and O(1)-ish map operations.

With duplicate merging, the merge function's cost also matters.

---

# 🔥 Interview questions

### Q1. What happens when `toMap()` encounters duplicate keys?

Without a merge function:

> `IllegalStateException`.

---

### Q2. How do you keep the latest value?

```java
(oldValue, newValue) -> newValue
```

---

### Q3. How do you keep the first value?

```java
(oldValue, newValue) -> oldValue
```

---

### Q4. How do you sum duplicate values?

```java
Integer::sum
```

---

### Q5. `toMap()` vs `groupingBy()`?

> `toMap()` is appropriate when each key ultimately maps to one value, with a merge strategy if duplicates occur. `groupingBy()` naturally creates a group of values for each key.

---

## 🧠 Lock this mental model in

```text
toList()
    → List

toSet()
    → Set

toMap()
    → Key → Value
       ↓
    duplicate?
       ↓
    merge function

groupingBy()
    → Key → List<Value>
```

### Next → `groupingBy()` ⭐⭐⭐⭐⭐

This is probably the **most important collector for Stream coding questions**: grouping employees by department, counting frequencies, highest salary per department, nested grouping, and downstream collectors.