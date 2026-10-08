# TreeSet — Deep Dive

Now we finish the three major `Set` implementations:

```text
Set
 ├── HashSet          ✅
 ├── LinkedHashSet    ✅
 └── TreeSet          ← NOW
```

The key conceptual shift is:

> **HashSet answers "Is this element already present?" using hashing. TreeSet answers it using ordering/comparison.**

---

# 1. Problem — What if uniqueness AND sorting are required?

Suppose you have transaction amounts:

```java
List<Integer> amounts =
    List.of(500, 100, 300, 100, 200);
```

Requirements:

1. No duplicates
2. Always maintain sorted order

Expected:

```text
100
200
300
500
```

A `HashSet` gives uniqueness:

```java
new HashSet<>(amounts);
```

but doesn't guarantee sorted iteration.

A `LinkedHashSet` preserves insertion order:

```text
500
100
300
200
```

but doesn't sort.

We need:

> **Unique + sorted**

That's what `TreeSet` provides.

---

# 2. Solution

```java
Set<Integer> amounts =
    new TreeSet<>();

amounts.add(500);
amounts.add(100);
amounts.add(300);
amounts.add(100);
amounts.add(200);
```

Iteration:

```text
100
200
300
500
```

So:

```text
TreeSet
   ↓
unique elements
   +
sorted order
```

---

# 3. Fundamental difference from HashSet

HashSet:

```text id="h1"
element
   ↓
hashCode()
   ↓
bucket
   ↓
equals()
```

TreeSet:

```text id="h2"
element
   ↓
comparison
   ↓
tree position
```

So TreeSet doesn't need hash buckets to organize the elements.

Its organization is based on a **sorted tree structure**.

---

# 4. What is TreeSet internally?

`TreeSet` is backed by a `TreeMap`.

Conceptually:

```text
TreeSet
   ↓
TreeMap
   ↓
Red-Black Tree
```

This is similar to what we saw with:

```text
HashSet → HashMap
LinkedHashSet → LinkedHashMap
```

So remember:

```text
HashSet
    → HashMap

LinkedHashSet
    → LinkedHashMap

TreeSet
    → TreeMap
```

And `TreeMap` uses a **Red-Black Tree**, a self-balancing binary search tree.

We'll go deeper into TreeMap when we reach Map.

---

# 5. What is a Binary Search Tree?

Suppose we insert:

```text
50
30
70
20
40
60
80
```

A binary search tree conceptually looks like:

```text
              50
            /    \
          30      70
         /  \    /  \
       20   40  60   80
```

The rule is:

```text
left subtree  < node
right subtree > node
```

So if we're searching for:

```text
60
```

we don't examine every element.

Start:

```text
50
```

60 > 50:

```text
        50
          \
           70
```

60 < 70:

```text
        70
       /
      60
```

Found.

That's logarithmic behavior when the tree remains balanced.

---

# 6. Why does TreeSet need a balanced tree?

Imagine inserting:

```text
10
20
30
40
50
60
```

into a simple binary search tree.

You could end up with:

```text
10
  \
   20
     \
      30
        \
         40
           \
            50
              \
               60
```

That's basically a linked list.

Search becomes:

```text
O(n)
```

That's not what we want.

A Red-Black Tree maintains approximate balance.

Conceptually:

```text
          30
        /    \
      20      50
     /       /  \
   10       40   60
```

Now operations remain:

```text
O(log n)
```

---

# 7. TreeSet complexity

Under the Red-Black Tree structure:

| Operation | Complexity |
|---|---:|
| `add()` | O(log n) |
| `remove()` | O(log n) |
| `contains()` | O(log n) |
| `first()` | O(log n) / implementation details |
| `last()` | O(log n) / implementation details |
| iteration | O(n) |

Compare this with HashSet:

```text
HashSet
add       → average O(1)
contains  → average O(1)
remove    → average O(1)
```

versus:

```text
TreeSet
add       → O(log n)
contains  → O(log n)
remove    → O(log n)
```

So why would we choose TreeSet?

Because we're getting **sorted order and ordered/range operations**.

---

# 8. The most important question: How does TreeSet compare elements?

Consider:

```java
TreeSet<Integer> numbers =
    new TreeSet<>();
```

Java already knows how to compare Integers.

For:

```text
10
20
30
```

it knows:

```text
10 < 20 < 30
```

But what about:

```java
class Employee {
    int id;
    String name;
}
```

What does this mean?

```text
Employee A < Employee B
```

Java needs a rule.

There are two primary mechanisms:

```text
Comparable
Comparator
```

---

# 9. Comparable — natural ordering

Suppose:

```java
class Employee implements Comparable<Employee> {

    private int id;
    private String name;

    @Override
    public int compareTo(Employee other) {
        return Integer.compare(this.id, other.id);
    }
}
```

Now:

```java
TreeSet<Employee> employees =
    new TreeSet<>();
```

When inserting employees:

```java
employees.add(e1);
employees.add(e2);
employees.add(e3);
```

TreeSet uses:

```java
e1.compareTo(e2)
```

to determine their ordering.

So:

```text
Employee
   ↓
Comparable
   ↓
compareTo()
   ↓
TreeSet ordering
```

---

# 10. What does `compareTo()` return?

The important contract is **sign**, not exact number.

```text
compareTo(other) < 0
    ↓
this comes before other

compareTo(other) == 0
    ↓
same ordering position

compareTo(other) > 0
    ↓
this comes after other
```

For example:

```java
return Integer.compare(this.id, other.id);
```

If:

```text
this.id = 10
other.id = 20
```

then:

```text
compareTo() < 0
```

If:

```text
10 vs 10
```

then:

```text
0
```

---

# 11. Comparator — external/custom ordering

What if we want to sort employees by salary instead of ID?

We might not want to change the Employee's natural ordering.

We can provide a Comparator:

```java
Comparator<Employee> bySalary =
    Comparator.comparing(Employee::getSalary);
```

Then:

```java
TreeSet<Employee> employees =
    new TreeSet<>(bySalary);
```

Now TreeSet uses:

```text
salary
```

to determine ordering.

So:

```text
Comparable
    → object's natural ordering

Comparator
    → externally supplied/custom ordering
```

---

# 12. Why is Comparator powerful?

Suppose the same Employee objects need different orderings:

```text
by ID
by salary
by name
by joining date
```

We don't want four different Employee classes.

We can create:

```java
Comparator<Employee> byId =
    Comparator.comparing(Employee::getId);

Comparator<Employee> bySalary =
    Comparator.comparing(Employee::getSalary);

Comparator<Employee> byName =
    Comparator.comparing(Employee::getName);
```

Then:

```java
new TreeSet<>(byId);
new TreeSet<>(bySalary);
new TreeSet<>(byName);
```

Same objects, different ordering policies.

---

# 13. The BIG TreeSet interview trap

Consider:

```java
class Employee {

    int id;
    String name;
}
```

Suppose:

```java
Comparator<Employee> bySalary =
    Comparator.comparing(Employee::getSalary);
```

Now:

```java
TreeSet<Employee> employees =
    new TreeSet<>(bySalary);
```

Suppose:

```text
Employee A → salary 100000
Employee B → salary 100000
```

but:

```text
A.id = 1
B.id = 2
```

Comparator says:

```text
compare(A, B) == 0
```

TreeSet may therefore treat them as the **same element for ordering/set purposes**.

So B may not be added.

This is a critical distinction.

---

# 14. TreeSet uniqueness is based on comparison

For HashSet, we learned:

```text
hashCode()
+
equals()
```

For TreeSet, uniqueness is determined through:

```text
compareTo()
```

or:

```text
Comparator.compare()
```

If comparison returns:

```text
0
```

TreeSet treats the elements as equivalent in its ordering.

Therefore:

```text
HashSet
   → equality semantics

TreeSet
   → ordering/comparison semantics
```

This is one of the most important differences between the two.

---

# 15. Example

```java
TreeSet<String> set =
    new TreeSet<>();

set.add("java");
set.add("JAVA");
```

Default String ordering is case-sensitive.

So:

```text
"JAVA"
"java"
```

are distinct according to their comparison.

But suppose:

```java
TreeSet<String> set =
    new TreeSet<>(String.CASE_INSENSITIVE_ORDER);
```

Then:

```java
set.add("java");
set.add("JAVA");
```

The comparator can return:

```text
0
```

So the second value is considered equivalent for the Set.

This demonstrates:

> **TreeSet's notion of uniqueness can be determined by its comparator.**

---

# 16. `compareTo() == 0` vs `equals() == true`

This leads to an important Java design concern.

Ideally, the natural ordering should be **consistent with equals**.

That means:

```text
a.compareTo(b) == 0
```

should generally correspond to:

```text
a.equals(b)
```

when designing a value type.

But Java doesn't universally enforce this.

You can have:

```text
compareTo() == 0
equals() == false
```

And TreeSet can still treat them as equivalent for Set purposes.

---

# 17. Production example

Suppose you have:

```java
class Transaction {
    String transactionId;
    BigDecimal amount;
}
```

You create:

```java
TreeSet<Transaction> transactions =
    new TreeSet<>(
        Comparator.comparing(Transaction::getAmount)
    );
```

Now two transactions:

```text
T1 → ₹500
T2 → ₹500
```

compare as:

```text
0
```

Potentially only one remains in the Set.

That's probably **not** what you want.

You may instead need a tie-breaker:

```java
Comparator<Transaction> comparator =
    Comparator.comparing(Transaction::getAmount)
              .thenComparing(Transaction::getTransactionId);
```

Now:

```text
amount
   ↓
if equal
   ↓
transactionId
```

So two transactions with the same amount can still be distinct.

This is a very practical production-level use of Comparator composition.

---

# 18. TreeSet range operations

This is another reason TreeSet exists.

Suppose:

```java
TreeSet<Integer> numbers =
    new TreeSet<>();

numbers.add(10);
numbers.add(20);
numbers.add(30);
numbers.add(40);
numbers.add(50);
```

You can ask:

```java
numbers.first();
```

Result:

```text
10
```

and:

```java
numbers.last();
```

Result:

```text
50
```

More interestingly:

```java
numbers.lower(30);
```

returns:

```text
20
```

```java
numbers.higher(30);
```

returns:

```text
40
```

You can also use range views:

```java
numbers.subSet(20, 50);
```

which conceptually represents:

```text
20
30
40
```

These ordered operations are a major reason to choose TreeSet instead of HashSet.

---

# 19. `lower`, `higher`, `floor`, `ceiling`

These are worth knowing.

Suppose:

```text
10 20 30 40 50
```

Query:

```text
x = 30
```

### `lower(30)`

Strictly less than 30:

```text
20
```

### `floor(30)`

Less than or equal to 30:

```text
30
```

### `higher(30)`

Strictly greater than 30:

```text
40
```

### `ceiling(30)`

Greater than or equal to 30:

```text
30
```

Think:

```text
lower  → <
floor   → ≤
higher  → >
ceiling → ≥
```

These are very useful in range/search problems.

---

# 20. `subSet`, `headSet`, `tailSet`

Suppose:

```java
TreeSet<Integer> set =
    new TreeSet<>(
        List.of(10, 20, 30, 40, 50)
    );
```

Then:

```java
set.headSet(30);
```

gives values before 30:

```text
10
20
```

```java
set.tailSet(30);
```

gives:

```text
30
40
50
```

And:

```java
set.subSet(20, 50);
```

conceptually gives:

```text
20
30
40
```

The exact boundary behavior matters; the default `subSet(from, to)` is typically **from inclusive, to exclusive**.

You can also explicitly specify inclusiveness with the overload that accepts booleans.

---

# 21. Null handling

Unlike HashSet:

```java
HashSet
```

can contain a null element.

TreeSet generally cannot use `null` with its natural ordering because it needs to compare elements.

For example:

```java
TreeSet<Integer> set =
    new TreeSet<>();

set.add(null);
```

will result in a `NullPointerException` in the usual natural-ordering case.

If a comparator explicitly defines how null should be handled, behavior can be designed differently, but for interview purposes:

> **Don't assume TreeSet accepts null.**

---

# 22. TreeSet vs HashSet

Now the comparison becomes much clearer.

| Feature | HashSet | TreeSet |
|---|---|---|
| Internal concept | Hash table | Red-Black Tree |
| Backed by | HashMap | TreeMap |
| Ordering | None guaranteed | Sorted |
| Average add | O(1) | O(log n) |
| Contains | Average O(1) | O(log n) |
| Duplicate determination | `hashCode` + `equals` | comparison |
| Range operations | No | Yes |
| `first/last` | Not naturally ordered | Yes |

So don't ask:

> "Which is faster?"

Ask:

> **"Do I need ordering?"**

If no:

```text
HashSet
```

If yes:

```text
TreeSet
```

---

# 23. TreeSet vs LinkedHashSet

This is another common interview question.

### LinkedHashSet

```text
30
10
20
```

preserves:

```text
30
10
20
```

### TreeSet

```text
30
10
20
```

produces:

```text
10
20
30
```

Therefore:

```text
LinkedHashSet → insertion order
TreeSet       → sorted order
```

---

# 24. Custom sorting example

Suppose:

```java
class Employee {
    private String name;
    private int salary;

    // getters
}
```

We want highest salary first.

```java
Comparator<Employee> bySalaryDesc =
    Comparator.comparingInt(Employee::getSalary)
              .reversed();

TreeSet<Employee> employees =
    new TreeSet<>(bySalaryDesc);
```

But there's a problem.

Two employees can have:

```text
salary = 100000
```

Then:

```text
compare() == 0
```

and TreeSet may treat one as equivalent to the other.

So use a tie-breaker:

```java
Comparator<Employee> comparator =
    Comparator.comparingInt(Employee::getSalary)
              .reversed()
              .thenComparing(Employee::getName);
```

Now:

```text
salary
   ↓
same salary?
   ↓
compare name
```

This is an excellent example of **why Comparator design matters**, not just how to write it.

---

# 25. Implement a simplified TreeSet?

In an interview, you normally wouldn't implement a full Red-Black Tree unless explicitly asked.

But you should understand the fundamental BST insertion.

For:

```text
50, 30, 70, 20
```

Start:

```text
50
```

Insert 30:

```text
  50
 /
30
```

Insert 70:

```text
  50
 /  \
30   70
```

Insert 20:

```text
    50
   /  \
 30   70
 /
20
```

The search rule is:

```text
if value < current
    go left
else
    go right
```

A production TreeSet additionally needs balancing, which is why Java uses a Red-Black Tree through TreeMap.

---

# 26. Why Red-Black Tree?

The goal is to maintain approximately balanced height.

Without balancing:

```text
10
  \
   20
     \
      30
        \
         40
```

height becomes:

```text
O(n)
```

With balancing:

```text
      20
     /  \
   10    30
           \
            40
```

height stays logarithmic.

Therefore:

```text
search
insert
delete
```

remain:

```text
O(log n)
```

We'll go much deeper into Red-Black Tree mechanics when we cover `TreeMap`.

---

# 27. Interview question — "What if compareTo returns 0?"

This is one you should answer precisely:

> "`TreeSet` treats the elements as equivalent according to its ordering, so the second element may not be inserted. Therefore, comparison returning zero effectively determines uniqueness within that TreeSet."

---

# 28. Interview question — "Can TreeSet contain objects without Comparable?"

Yes, **if you provide a Comparator**.

Without either:

```text
Comparable
```

or:

```text
Comparator
```

TreeSet won't know how to order arbitrary objects.

For example:

```java
TreeSet<Employee> set =
    new TreeSet<>();
```

if Employee doesn't implement Comparable and no Comparator is supplied, attempting to add elements will lead to a `ClassCastException`.

With:

```java
new TreeSet<>(employeeComparator)
```

it knows how to compare them.

---

# 29. Interview question — Comparable vs Comparator

### Comparable

Defined inside the class:

```java
class Employee
    implements Comparable<Employee>
```

with:

```java
compareTo()
```

Represents:

> **Natural ordering**

### Comparator

Separate object:

```java
Comparator<Employee>
```

with:

```java
compare()
```

Represents:

> **Custom/external ordering**

Mental model:

```text
Comparable
   → "How does this object naturally compare?"

Comparator
   → "How do I want to compare these objects for this particular use?"
```

---

# 30. The complete Set mental model

Now you have all three:

```text
                         SET
                          │
          ┌───────────────┼────────────────┐
          │               │                │
       HashSet      LinkedHashSet       TreeSet
          │               │                │
       HashMap       LinkedHashMap       TreeMap
          │               │                │
       hashing       hashing + links     tree
          │               │                │
     uniqueness     uniqueness +        uniqueness +
                    insertion order      sorted order
          │               │                │
      avg O(1)          avg O(1)          O(log n)
```

This is the **core Set decision tree**.

---

# 31. Production decision

When you encounter a requirement, think:

### "I just need unique values."

```text
HashSet
```

### "I need unique values and preserve insertion order."

```text
LinkedHashSet
```

### "I need unique values maintained in sorted order and range queries."

```text
TreeSet
```

That's much better than memorizing three class names independently.

---

# Set section status

```text
Set
 │
 ├── HashSet          ✅
 │    ├── Hashing
 │    ├── Buckets
 │    ├── Collisions
 │    ├── equals/hashCode
 │    ├── Resize
 │    └── Treeification
 │
 ├── LinkedHashSet    ✅
 │    └── Insertion order
 │
 └── TreeSet          ✅
      ├── TreeMap backing
      ├── Red-Black Tree
      ├── Comparable
      ├── Comparator
      ├── Sorted order
      └── Range operations
```

### Next in the roadmap: `Set` consolidation

Before we move to **Map**, we should lock down the highest-value Set interview concepts:

1. `equals()` / `hashCode()` contract in depth
2. `Comparable` vs `Comparator`
3. HashSet vs LinkedHashSet vs TreeSet
4. Duplicate detection with custom objects
5. Mutable keys/elements
6. Comparator returning `0`
7. Fail-fast behavior
8. Set coding problems

Then we move into **Map → HashMap**, which is one of the most important topics in your entire Java interview preparation.