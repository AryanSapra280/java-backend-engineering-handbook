🔥 Let's start properly. We'll treat this like **5-year Java backend interview preparation**, not beginner Java.

# Module 1 — Core Java

## Topic 1: `HashMap` — Deep Dive

This is one of the **highest-value Java interview topics** for your Cognizant round.

By the end, you should be able to answer:

> "Explain HashMap internally."

in **2–4 minutes**, and then survive 5–10 follow-up questions.

---

# 1. First: What is HashMap?

`HashMap<K,V>` stores data as **key-value pairs**.

```java
Map<Integer, String> map = new HashMap<>();

map.put(101, "Aryan");
map.put(102, "Rahul");
map.put(103, "Amit");
```

Conceptually:

```text
Key       Value
----------------
101  →    Aryan
102  →    Rahul
103  →    Amit
```

The important property is:

> HashMap uses the **key's hash** to determine where the entry should be stored.

That's what gives average **O(1)** lookup, insertion and deletion.

---

# 2. What actually happens during `put()`?

Suppose:

```java
map.put(101, "Aryan");
```

Conceptually:

```text
             put(101, "Aryan")
                     │
                     ↓
             calculate hash
                     │
                     ↓
             find bucket
                     │
                     ↓
          Is bucket empty?
             /          \
           YES           NO
            │             │
            ↓             ↓
       store node      collision
                          │
                          ↓
                   compare keys
                          │
                   ┌──────┴──────┐
                   ↓             ↓
                same key      different
                   │             │
                 update       new node
```

Now let's go deeper.

---

# 3. Internal structure

Conceptually, HashMap has an array:

```text
Node<K,V>[] table
```

Imagine:

```text
Bucket
  0     → null
  1     → Node
  2     → null
  3     → Node
  4     → Node
  5     → null
  6     → null
  7     → Node
```

Each bucket can contain entries.

An entry is conceptually:

```java
Node<K,V> {
    int hash;
    K key;
    V value;
    Node<K,V> next;
}
```

So:

```text
Bucket 3
   ↓
Node
 ├── hash
 ├── key
 ├── value
 └── next
       ↓
      Node
       ↓
      Node
```

This `next` reference is how collisions can form a chain.

---

# 4. What is a collision?

Suppose:

```text
Key A → bucket 5
Key B → bucket 5
```

Both keys want the same bucket.

That's a **hash collision**.

```text
Bucket 5
   ↓
Node(A)
   ↓
Node(B)
   ↓
Node(C)
```

HashMap must therefore distinguish between:

```text
same bucket
```

and

```text
same key
```

These are NOT the same thing.

---

# 5. `hashCode()` is NOT the key itself

Suppose:

```java
String a = "ABC";
```

Java calculates:

```java
a.hashCode()
```

The hash is an integer.

HashMap then uses that hash to determine a bucket.

Conceptually:

```text
key
 ↓
hashCode()
 ↓
hash
 ↓
bucket index
```

---

# 6. How is bucket index calculated?

This is where interviewers may go deeper.

In Java 8 HashMap, the table size is maintained as a **power of two**.

Conceptually the index calculation is:

```java
index = (n - 1) & hash;
```

where:

```text
n = table length
```

For example:

```text
table length = 16
```

Then:

```text
n - 1 = 15
```

Binary:

```text
15 = 1111
```

So:

```text
index = hash & 1111
```

This efficiently produces an index between:

```text
0 → 15
```

---

# 7. Why power of 2?

This is a classic interview follow-up.

Because:

```text
(n - 1) & hash
```

efficiently distributes hash bits across the available buckets.

For a power-of-two table size:

```text
16 → 10000
15 → 01111
```

Therefore:

```text
hash & 01111
```

selects the lower bits efficiently.

You don't need to memorize the implementation line blindly.

Understand the reason.

---

# 8. But Java does something with the hash first

This is another deeper question.

HashMap doesn't simply use:

```java
key.hashCode()
```

directly.

It performs a hash spreading operation conceptually similar to:

```java
h ^ (h >>> 16)
```

Why?

Because bucket selection relies heavily on lower bits when the table size is a power of two.

Mixing higher bits into lower bits can improve distribution.

Conceptually:

```text
Original hash
     ↓
mix/spread bits
     ↓
bucket calculation
```

This helps reduce poor distribution for certain hash codes.

---

# 9. What happens if two keys have the same hash?

Suppose:

```text
key1.hashCode() = 100
key2.hashCode() = 100
```

That doesn't mean:

```java
key1.equals(key2)
```

is necessarily true.

HashMap therefore does:

```text
hash match?
    ↓
equals()?
```

Conceptually:

```text
Same bucket
    ↓
compare hash
    ↓
compare key using equals()
```

This is why both methods matter.

---

# 10. `equals()` + `hashCode()` contract

🔥🔥🔥 **MEMORIZE THIS CONCEPTUALLY.**

If:

```java
a.equals(b) == true
```

then:

```java
a.hashCode() == b.hashCode()
```

must be true.

But the reverse isn't required.

Meaning:

```text
same hashCode
     ≠
same object
```

Example:

```text
Object A → hash = 100
Object B → hash = 100
```

They can still be different objects.

That's a collision.

---

# 11. Why is HashMap average O(1)?

Because normally:

```text
key
 ↓
hash
 ↓
bucket
 ↓
entry
```

So you don't search the entire map.

Average:

```text
put()    → O(1)
get()    → O(1)
remove() → O(1)
```

But that's **average-case**, not guaranteed O(1).

---

# 12. What about collisions?

Before Java 8, heavy collisions could result in a linked list:

```text
Bucket
  ↓
Node
  ↓
Node
  ↓
Node
  ↓
Node
```

Lookup could degrade toward:

```text
O(n)
```

Java 8 introduced treeification of sufficiently large collision chains.

Conceptually:

```text
Before

Bucket
  ↓
A → B → C → D → E
```

can become something conceptually like:

```text
             C
           /   \
          A     D
           \     \
            B     E
```

a red-black tree.

Then lookup can become approximately:

```text
O(log n)
```

within that heavily-collided bucket.

---

# 13. Treeification threshold

This is a potential interviewer trap.

Important implementation constants in Java 8 include:

```text
TREEIFY_THRESHOLD = 8
UNTREEIFY_THRESHOLD = 6
MIN_TREEIFY_CAPACITY = 64
```

But don't simply say:

> "After 8 elements HashMap always becomes a tree."

That's incomplete.

Treeification also depends on the table capacity; if the table is too small, HashMap prefers resizing rather than treeifying.

For an interview, say:

> "When a bucket becomes sufficiently collision-heavy, HashMap can convert the linked structure into a red-black tree. In Java 8 the treeification threshold is 8, but HashMap also considers table capacity and may resize instead."

That's a **5-year-level answer**.

---

# 14. HashMap resizing

🔥🔥🔥 Very important.

Suppose:

```text
initial capacity = 16
load factor = 0.75
```

Threshold:

```text
16 × 0.75 = 12
```

When the number of entries crosses the threshold, HashMap resizes.

Conceptually:

```text
16 buckets
    ↓
resize
    ↓
32 buckets
```

Then entries are redistributed according to the new table size.

---

# 15. Why resize?

Because as the map becomes too full:

```text
more entries
     ↓
more collisions
     ↓
longer chains
     ↓
slower lookup
```

Resizing increases the number of buckets:

```text
16 → 32 → 64 → 128 → ...
```

reducing collision density.

---

# 16. What is load factor?

Default load factor:

```text
0.75
```

It represents how full the HashMap is allowed to become before resizing.

For:

```text
capacity = 16
load factor = 0.75
```

threshold:

```text
12
```

So approximately after reaching the threshold, resizing occurs.

---

# 17. Why not use load factor = 1?

Because you could reduce resizing and memory overhead.

But:

```text
higher load factor
      ↓
more entries per bucket
      ↓
more collisions
      ↓
potentially slower operations
```

So it's a trade-off:

```text
Memory
   ↕
Collision / performance
```

---

# 18. Initial capacity vs size

This is another common mistake.

If:

```java
new HashMap<>(100);
```

that doesn't mean:

> "I can store exactly 100 elements."

Capacity refers to the underlying table sizing behavior, not the number of entries.

And Java's resizing behavior has implementation nuances, including lazy table initialization.

For interview purposes:

> **Size = number of mappings currently stored. Capacity = number of buckets allocated/targeted by the table.**

---

# 19. What happens during `get()`?

Suppose:

```java
map.get(101);
```

Conceptually:

```text
101
 ↓
hashCode()
 ↓
hash spreading
 ↓
bucket index
 ↓
bucket
 ↓
compare hash/key
 ↓
equals()
 ↓
value
```

So:

```text
get()
```

doesn't scan every entry.

---

# 20. What if key doesn't exist?

Example:

```java
map.get(999);
```

HashMap finds the appropriate bucket.

Then:

```text
bucket
 ↓
compare first node
 ↓
not equal
 ↓
next node
 ↓
not equal
 ↓
...
 ↓
null
```

Then:

```java
null
```

is returned, unless you've stored a null value and need to distinguish presence with `containsKey()`.

---

# 21. What happens with `null` key?

Important.

`HashMap` permits:

```java
map.put(null, "Aryan");
```

and multiple null values.

There can be only one mapping for the `null` key.

For example:

```java
map.put(null, "A");
map.put(null, "B");
```

Result:

```text
null → B
```

The second `put` replaces the first mapping.

---

# 22. Is HashMap thread-safe?

❌ No.

This is a very important interview answer.

If multiple threads modify a normal HashMap concurrently, you cannot rely on it for thread-safe behavior.

For concurrent access, depending on the use case:

```java
ConcurrentHashMap
```

may be appropriate.

---

# 23. HashMap vs ConcurrentHashMap

| HashMap                                                    | ConcurrentHashMap               |
| ---------------------------------------------------------- | ------------------------------- |
| Not thread-safe                                            | Designed for concurrent access  |
| Allows null key/value                                      | Doesn't allow null keys/values  |
| Suitable for single-threaded / externally synchronized use | Suitable for concurrent access  |
| No concurrency coordination                                | Provides concurrency mechanisms |

And don't say:

> ConcurrentHashMap simply locks the entire map.

Modern implementations use much finer-grained concurrency mechanisms; understanding that distinction is useful.

---

# 24. The BIG interview trap: Mutable keys

Consider:

```java
Map<Employee, String> map = new HashMap<>();
```

Suppose `Employee.hashCode()` depends on:

```java
employee.id
```

You do:

```java
map.put(employee, "Developer");
```

Then change:

```java
employee.id = 200;
```

Now the object's hash can change.

You effectively have:

```text
Original insertion

hash(employee)
     ↓
bucket 5


After mutation

hash(employee)
     ↓
bucket 12
```

But the actual entry is still sitting in bucket 5.

Then:

```java
map.get(employee)
```

may fail to find the entry.

🔥 Therefore:

> Keys used in hash-based collections should generally have stable equality/hash-code-relevant state while they're being used as keys.

This is an excellent production-level concept.

---

# 25. Your 3-minute interview answer

If Cognizant asks:

> **"Explain HashMap internals."**

You can say:

> "`HashMap` stores key-value pairs using an internal bucket array. When we insert a key, HashMap obtains the key's hash code, performs hash spreading, and uses the hash along with the table size to determine the bucket. If the bucket is empty, the entry is stored there. If there's a collision, HashMap compares the hash and then the keys using `equals()` to determine whether it's an existing key or a different key."
>
> "In Java 8, collision-heavy buckets can be converted from a linked structure into a red-black tree, which improves lookup in those buckets from linear behavior toward logarithmic behavior. HashMap also resizes when its size crosses the threshold determined by capacity and load factor; the default load factor is 0.75."
>
> "Average `put`, `get`, and `remove` are O(1), although poor hash distribution or heavy collisions can degrade performance. HashMap isn't thread-safe, so for concurrent access we may use `ConcurrentHashMap` depending on the use case."

🔥 That's already a strong answer.

---

# 26. Follow-up questions you MUST be ready for

After that answer, the interviewer can attack from several directions:

### Level 1

**Q1. Why does HashMap use `hashCode()`?**

**Q2. Why do we need `equals()` if we already have `hashCode()`?**

**Q3. What is collision?**

**Q4. What is load factor?**

**Q5. What is the default load factor?**

---

### Level 2

**Q6. Why is HashMap O(1)?**

**Q7. What happens when two keys have the same hash?**

**Q8. What happens when two objects have the same hash but aren't equal?**

**Q9. How does resizing work?**

**Q10. Why does Java 8 convert buckets into trees?**

---

### Level 3

**Q11. Why is HashMap capacity generally a power of two?**

**Q12. How does `(n - 1) & hash` work?**

**Q13. Why does HashMap spread/mix the hash?**

**Q14. What happens if a key's fields used by `hashCode()` change after insertion?**

**Q15. Why doesn't ConcurrentHashMap allow null?**

---

# 27. One thing I want you to avoid

Don't memorize this:

> "HashMap uses array + linked list + red-black tree."

That's only **20% of the answer**.

Your mental model should be:

```text
                  HashMap
                     │
       ┌─────────────┼──────────────┐
       ↓             ↓              ↓
    Hashing       Collision       Resizing
       │             │              │
       ↓             ↓              ↓
 hashCode()      equals()       Load Factor
       │             │              │
       ↓             ↓              ↓
 bucket index   List / Tree     Capacity × LF
```

And then:

```text
                    HashMap
                       │
              ┌────────┴────────┐
              ↓                 ↓
            Normal           Concurrent
              ↓                 ↓
           HashMap       ConcurrentHashMap
```

---

# 🧪 Your first interview exercise

Don't look back at the explanation now.

Imagine I'm the Cognizant interviewer:

> **"You have a `HashMap<Employee, String>`. Explain what happens internally when I execute `map.put(employee, "Developer")`, and then later execute `map.get(employee)`. Also tell me what happens if two Employee objects have the same hash code but are not equal."**

**Answer this in your own words as if you're in the interview.**

I'll evaluate your answer at a **5-year-experience level**, correct it, and then take you into the next layer: **HashSet → ConcurrentHashMap → `equals/hashCode` traps → Collections internals → Java 8**.
