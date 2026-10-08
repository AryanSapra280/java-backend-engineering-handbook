# Set — Part 1: Why does Set exist?

We now move from **List → Set**. We’ll start from the problem and build all the way toward `HashSet` internals.

---

## 1. Problem — What does List fail to guarantee?

Suppose we have:

```java
List<String> emails = new ArrayList<>();

emails.add("a@gmail.com");
emails.add("b@gmail.com");
emails.add("a@gmail.com");
```

The List contains:

```text
a@gmail.com
b@gmail.com
a@gmail.com
```

That's perfectly valid because **List allows duplicates**.

But imagine your requirement is:

> "I need to store unique email addresses."

We could manually check:

```java
if (!emails.contains(email)) {
    emails.add(email);
}
```

But now every insertion involves a search.

And more importantly, **uniqueness is now application logic** rather than a property of the data structure.

This is the problem `Set` solves.

---

# 2. Solution — Set

A `Set` represents a collection that does **not allow duplicate elements**.

```java
Set<String> emails = new HashSet<>();

emails.add("a@gmail.com");
emails.add("b@gmail.com");
emails.add("a@gmail.com");
```

Conceptually:

```text
a@gmail.com
b@gmail.com
```

The second:

```java
emails.add("a@gmail.com");
```

doesn't create another element.

So the fundamental contract is:

> **A Set contains no duplicate elements.**

---

# 3. Set is an interface

Just like:

```text
List
```

we don't instantiate:

```java
new Set<>();
```

Instead:

```java
Set<String> set =
    new HashSet<>();
```

The abstraction is:

```text
Set
 ↑
HashSet
```

Other important implementations:

```text
Set
 ├── HashSet
 ├── LinkedHashSet
 └── TreeSet
```

And they have different behavior.

---

# 4. Set does NOT mean sorted

This is a common interview trap.

Suppose:

```java
Set<Integer> numbers =
    new HashSet<>();

numbers.add(30);
numbers.add(10);
numbers.add(20);
```

You should **not** assume:

```text
10
20
30
```

A HashSet does not guarantee sorted order.

The important distinction:

```text
HashSet
    → uniqueness
    → no guaranteed iteration order

LinkedHashSet
    → uniqueness
    → insertion order

TreeSet
    → uniqueness
    → sorted order
```

We'll study these individually.

---

# 5. First implementation: HashSet

We'll start with:

```java
Set<String> set =
    new HashSet<>();
```

Why HashSet?

Because it is the most important Set implementation for understanding:

- hashing
- `hashCode()`
- `equals()`
- buckets
- collisions
- duplicate detection
- HashMap internals

And this is one of the highest-value Collections topics for Java interviews.

---

# 6. Problem — How can HashSet detect duplicates efficiently?

Suppose we have:

```text
1,000,000 elements
```

When we insert:

```java
set.add(x);
```

we don't want to scan:

```text
element 1
element 2
element 3
...
element 1,000,000
```

every time.

That would be:

```text
O(n)
```

for each insertion.

Instead, HashSet uses **hashing**.

---

# 7. The core idea of hashing

Suppose:

```java
String key = "Aryan";
```

Java can calculate:

```java
key.hashCode()
```

which produces an integer.

Conceptually:

```text
"Aryan"
   ↓
hashCode()
   ↓
some integer
   ↓
bucket location
```

The hash helps determine **where the element should be stored**.

Think of buckets:

```text
Bucket 0
Bucket 1
Bucket 2
Bucket 3
Bucket 4
Bucket 5
Bucket 6
Bucket 7
```

Instead of searching the entire Set, we go approximately to the appropriate bucket.

---

# 8. HashSet's most important internal fact

Here's the interview-critical point:

> **HashSet is implemented internally using a HashMap.**

Conceptually:

```text
HashSet
   ↓
HashMap
   ↓
buckets
   ↓
nodes
```

When you do:

```java
Set<String> set =
    new HashSet<>();

set.add("Java");
```

internally HashSet essentially uses the value as a **key in a HashMap**.

Conceptually:

```text
HashMap

"Java" → PRESENT
```

The actual implementation uses an internal dummy value.

You don't need to memorize the exact private implementation field names, but you absolutely should understand:

```text
HashSet = HashMap-backed
```

---

# 9. Why HashMap?

Because HashMap already provides:

```text
hashing
+
bucket management
+
collision handling
+
equals() checking
```

HashSet doesn't need key-value semantics.

It only needs:

```text
unique keys
```

So conceptually:

```text
HashMap<K,V>

K → V
```

becomes:

```text
HashSet<E>

E → PRESENT
```

That's the mental model.

---

# 10. Now let's understand `add()`

Consider:

```java
Set<String> set =
    new HashSet<>();

set.add("Java");
```

Conceptually, the process is:

```text
"Java"
   ↓
hashCode()
   ↓
hash spreading / bucket calculation
   ↓
bucket
   ↓
check existing entries
   ↓
equals()
   ↓
insert if not duplicate
```

This is the heart of HashSet.

---

# 11. Why do we need BOTH `hashCode()` and `equals()`?

This is one of the most important Java interview concepts.

Suppose:

```java
String a = new String("Java");
String b = new String("Java");
```

They are different objects:

```text
a ≠ b
```

in terms of object identity.

But:

```java
a.equals(b)
```

returns:

```text
true
```

because their content is equal.

HashSet needs to understand:

> "Are these two values logically the same?"

That's where `equals()` comes in.

---

# 12. Why can't HashSet just use equals()?

Because doing this:

```text
compare new element
against every existing element
```

would require:

```text
O(n)
```

searching.

Hashing first narrows down the candidates.

So:

```text
hashCode()
    ↓
find likely bucket
    ↓
equals()
    ↓
confirm duplicate
```

This is the fundamental relationship:

> **`hashCode()` narrows the search; `equals()` confirms logical equality.**

---

# 13. Collision

Now suppose two different objects produce the same bucket.

For example:

```text
Object A → bucket 5
Object B → bucket 5
```

This is called a:

> **Hash collision**

Conceptually:

```text
Bucket 5

[A] → [B] → [C]
```

The fact that they land in the same bucket does **not** mean they're equal.

HashSet/HashMap must distinguish them using equality checks.

So:

```text
same hash/bucket
       ≠
equal objects
```

Very important.

---

# 14. Example with custom class

Suppose:

```java
class Employee {

    private int id;
    private String name;

    Employee(int id, String name) {
        this.id = id;
        this.name = name;
    }
}
```

Now:

```java
Set<Employee> employees =
    new HashSet<>();

employees.add(
    new Employee(1, "Aryan")
);

employees.add(
    new Employee(1, "Aryan")
);
```

Will the Set necessarily contain one element?

**No.**

Why?

Because unless we override `equals()` and `hashCode()`, the default `Object` implementations are based on object identity.

So these are two separate objects.

```text
Employee object A
        ≠
Employee object B
```

The Set can contain both.

---

# 15. Implement equals and hashCode

If our business definition is:

> Two Employees are equal if their ID is equal.

Then:

```java
class Employee {

    private int id;
    private String name;

    @Override
    public boolean equals(Object o) {

        if (this == o) {
            return true;
        }

        if (!(o instanceof Employee)) {
            return false;
        }

        Employee other = (Employee) o;

        return this.id == other.id;
    }

    @Override
    public int hashCode() {
        return Integer.hashCode(id);
    }
}
```

Now:

```java
Employee e1 =
    new Employee(1, "Aryan");

Employee e2 =
    new Employee(1, "Aryan");
```

have:

```text
e1.equals(e2) → true
```

and:

```text
e1.hashCode() == e2.hashCode()
```

Therefore HashSet recognizes them as duplicates.

---

# 16. The contract you MUST remember

If:

```java
a.equals(b)
```

is:

```text
true
```

then:

```java
a.hashCode() == b.hashCode()
```

**must be true.**

But the reverse isn't required.

You can have:

```text
a.hashCode() == b.hashCode()
```

while:

```text
a.equals(b) == false
```

That's a collision.

So:

```text
equals true
     ↓
hashCode MUST be same

hashCode same
     ↓
equals may be true OR false
```

This is fundamental.

---

# 17. What happens inside HashSet when adding?

Let's make the complete mental model.

Suppose:

```java
set.add(employee);
```

Conceptually:

```text
                 Employee
                     │
                     ↓
               hashCode()
                     │
                     ↓
               bucket index
                     │
                     ↓
           Is bucket empty?
              /          \
            YES           NO
             │             │
             ↓             ↓
          insert       compare entries
                            │
                            ↓
                         equals()
                         /     \
                       true    false
                        │        │
                        ↓        ↓
                    duplicate   collision/
                    → no add    continue
```

That's the flow you should be able to explain at a whiteboard.

---

# 18. What does `add()` return?

This is another useful interview detail.

```java
Set<String> set =
    new HashSet<>();

System.out.println(
    set.add("Java")
);
```

returns:

```text
true
```

because Java was added.

Then:

```java
System.out.println(
    set.add("Java")
);
```

returns:

```text
false
```

because Java already existed.

So:

```text
add(new element) → true
add(duplicate)   → false
```

This is useful in production code when you need to know whether an insertion actually changed the Set.

---

# 19. Complexity

Under normal/good hash distribution:

| Operation | Average |
|---|---:|
| `add()` | O(1) |
| `contains()` | O(1) |
| `remove()` | O(1) |

But don't say:

> HashSet operations are always O(1).

That's too absolute.

Performance depends on:

- hash distribution
- collisions
- resizing
- equality checks

Modern Java HashMap/HashSet implementations also have collision-management behavior that can improve worst-case bucket lookup compared with a simple linked-list bucket implementation.

We'll go **very deep into this when we reach HashMap**, including treeification and the Java 7 vs Java 8+ differences.

---

# 20. What happens when the Set gets full?

Hash-based collections maintain a relationship between:

```text
capacity
+
load factor
```

The default load factor commonly associated with HashMap/HashSet is:

```text
0.75
```

Conceptually:

```text
capacity × load factor
        ↓
threshold
```

When the number of entries exceeds the threshold:

```text
resize
```

occurs.

Example conceptually:

```text
capacity = 16
load factor = 0.75

threshold ≈ 12
```

When enough elements are added, the backing table is resized and entries are redistributed.

This is why an individual insertion can occasionally cost O(n), while average insertion remains approximately O(1).

We'll unpack the exact HashMap mechanics later.

---

# 21. Production use cases

HashSet is useful when the core requirement is:

> **Fast membership testing + uniqueness.**

Examples:

### Duplicate detection

```java
Set<String> processedIds =
    new HashSet<>();
```

Then:

```java
if (!processedIds.add(transactionId)) {
    // already processed
}
```

This is a nice pattern.

Because:

```java
add()
```

itself tells you whether the value was already present.

---

### Permissions

```java
Set<String> permissions =
    new HashSet<>();
```

Then:

```java
if (permissions.contains("PAYMENT_WRITE")) {
    // allow operation
}
```

---

### Removing duplicates

```java
List<String> values =
    List.of("A", "B", "A", "C");

Set<String> unique =
    new HashSet<>(values);
```

Now duplicates are removed.

But remember:

> You lose guaranteed insertion ordering with HashSet.

If order matters, that's where LinkedHashSet comes in.

---

# 22. Important production trap — mutable objects

Suppose:

```java
class Employee {

    int id;

    @Override
    public boolean equals(Object o) {
        ...
    }

    @Override
    public int hashCode() {
        return Integer.hashCode(id);
    }
}
```

Now:

```java
Employee e =
    new Employee(10);

Set<Employee> set =
    new HashSet<>();

set.add(e);
```

The object is placed according to:

```text
hashCode(id = 10)
```

Now imagine:

```java
e.id = 20;
```

Its hash code changes.

But the object is still physically sitting in the bucket calculated using the **old hash**.

Now:

```java
set.contains(e)
```

may fail unexpectedly.

This is a very important production issue.

---

# 23. Why is this dangerous?

The Set expects the fields participating in:

```text
equals()
hashCode()
```

to remain stable while the object is being used as a hash-based collection element.

Bad:

```java
Set<Employee> set =
    new HashSet<>();

Employee e =
    new Employee(10);

set.add(e);

e.setId(20);

set.contains(e); // problematic
```

Better:

Use immutable key fields.

For example:

```java
final class EmployeeKey {

    private final int id;

    EmployeeKey(int id) {
        this.id = id;
    }

    // equals + hashCode
}
```

This is particularly important for:

- entity identifiers
- cache keys
- map keys
- Set elements

---

# 24. Interview questions you should now be able to answer

### Q1. Why does Set exist?

To represent a collection where duplicate elements are not allowed.

### Q2. Does HashSet maintain insertion order?

No guaranteed insertion order.

### Q3. How does HashSet detect duplicates?

Conceptually:

```text
hashCode()
    ↓
bucket
    ↓
equals()
    ↓
duplicate or new element
```

### Q4. Why are both hashCode and equals required?

Hashing efficiently narrows the search; equals determines logical equality.

### Q5. Can two unequal objects have the same hashCode?

Yes. That's a hash collision.

### Q6. If two objects have the same hashCode, are they equal?

No.

### Q7. If two objects are equal, can their hashCodes differ?

No. That violates the contract.

### Q8. What happens if you override equals but not hashCode?

You can break hash-based collections because logically equal objects may end up in different buckets.

### Q9. Why can mutating an object already inside a HashSet be dangerous?

If fields participating in `hashCode()`/`equals()` change, the object's effective hash location changes while the collection still has it stored according to the old hash.

---

# 25. The most important mental model

Don't memorize:

```text
HashSet → O(1)
```

Instead remember:

```text
                    HashSet
                       │
                       ↓
                    HashMap
                       │
                       ↓
                    hashing
                       │
                       ↓
                   bucket
                       │
                 ┌─────┴─────┐
                 │           │
              empty       entries
                 │           │
                 ↓           ↓
              insert      equals()
                             │
                       ┌─────┴─────┐
                       │           │
                     equal       unequal
                       │           │
                       ↓           ↓
                   duplicate    collision/
                    → reject     continue
```

That mental model will make **HashMap internals much easier later**.

---

## Set flow

We've now established the foundation:

```text
Set
 │
 └── HashSet ← we are here
       │
       ├── uniqueness
       ├── hashing
       ├── hashCode()
       ├── equals()
       ├── buckets
       ├── collisions
       └── HashMap backing
```

### Next: HashSet implementation in much deeper detail

We'll go one level deeper before moving to `LinkedHashSet`:

- actual HashSet → HashMap relationship
- bucket calculation
- `hash()` / hash spreading
- why powers of 2 matter for HashMap capacity
- collisions
- Java 7 vs Java 8+ bucket structure
- linked list → tree conversion
- load factor and resizing
- `equals()`/`hashCode()` failure examples
- implementing a simplified HashSet
- interviewer follow-up questions

That is the part that will prepare you for the **HashMap internals question** you've encountered in interviews.