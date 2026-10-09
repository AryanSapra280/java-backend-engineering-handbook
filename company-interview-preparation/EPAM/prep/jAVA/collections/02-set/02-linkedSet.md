# LinkedHashSet — Deep Dive

We continue exactly in sequence:

```text
Set
 ├── HashSet          ✅
 ├── LinkedHashSet    ← NOW
 └── TreeSet
```

The key question is:

> **What if I want Set's uniqueness, but I also care about the order in which elements were inserted?**

---

# 1. Problem

We already know:

```java
Set<Integer> set = new HashSet<>();

set.add(30);
set.add(10);
set.add(20);
```

HashSet guarantees uniqueness, but **does not guarantee iteration order**.

Suppose your requirement is:

> "Don't allow duplicates, but when I iterate, give me elements in the same order in which they were added."

For example:

```text
Input:

30
10
20
10
```

Desired iteration:

```text
30
10
20
```

The second `10` is ignored, but the original insertion order is retained.

That's exactly the problem `LinkedHashSet` solves.

---

# 2. Solution

Use:

```java
Set<Integer> set =
    new LinkedHashSet<>();
```

Then:

```java
set.add(30);
set.add(10);
set.add(20);
set.add(10);
```

Iteration gives:

```text
30
10
20
```

So:

```text
LinkedHashSet
     =
HashSet-style uniqueness
     +
insertion-order tracking
```

---

# 3. The important internal idea

This is the key mental model:

```text
HashSet
   ↓
HashMap
   ↓
hash buckets
```

while:

```text
LinkedHashSet
   ↓
LinkedHashMap
   ↓
hash buckets
   +
linked ordering
```

Conceptually:

```text
HashSet
  └── HashMap

LinkedHashSet
  └── LinkedHashMap
```

This is the same fundamental strategy we saw earlier:

> Reuse hash-based key management, then add another structure to maintain ordering.

---

# 4. What does "linked" mean?

It does **not** mean that LinkedHashSet is a normal LinkedList.

This is important.

It still uses hashing for lookup.

But entries are additionally connected to maintain insertion order.

Imagine:

```text
Hash buckets:

Bucket 0 → A
Bucket 1 → C → D
Bucket 2 → B
```

The physical bucket arrangement doesn't represent insertion order.

So LinkedHashSet maintains another logical ordering:

```text
A ⇄ B ⇄ C ⇄ D
```

If insertion happened as:

```text
A
B
C
D
```

iteration can follow that linked ordering.

---

# 5. Two structures are involved

Think of LinkedHashSet as having two concerns:

### Concern 1 — uniqueness / fast lookup

```text
hash table
```

### Concern 2 — iteration order

```text
linked ordering structure
```

So conceptually:

```text
                  LinkedHashSet
                       │
              ┌────────┴────────┐
              │                 │
         Hash structure      Linked order
              │                 │
              ↓                 ↓
          uniqueness        insertion order
```

That's the reason it has more memory overhead than HashSet.

---

# 6. Example

```java
LinkedHashSet<String> set =
    new LinkedHashSet<>();

set.add("Java");
set.add("Spring");
set.add("Kafka");
set.add("Java");
```

Insertion sequence:

```text
Java
Spring
Kafka
Java
```

The final Set contains:

```text
Java
Spring
Kafka
```

Iteration:

```java
for (String value : set) {
    System.out.println(value);
}
```

produces:

```text
Java
Spring
Kafka
```

The duplicate doesn't create another ordering entry.

---

# 7. What happens internally on `add()`?

Suppose:

```java
set.add("Java");
```

Conceptually:

```text
"Java"
   ↓
hashCode()
   ↓
bucket calculation
   ↓
find bucket
   ↓
does equal element already exist?
     │
   ┌─┴─┐
  yes no
   │   │
   ↓   ↓
reject insert
       │
       ↓
   add to hash structure
       +
   link into insertion-order chain
```

So uniqueness is still determined by:

```text
hashCode()
+
equals()
```

The same rules we learned for HashSet still apply.

---

# 8. Duplicate insertion

Suppose:

```java
set.add("Java");
set.add("Spring");
set.add("Java");
```

On the second `"Java"`:

```text
hashCode()
   ↓
same relevant bucket
   ↓
equals()
   ↓
already exists
```

Therefore:

```text
No new element
No new insertion-order position
```

So the order remains:

```text
Java
Spring
```

It doesn't become:

```text
Java
Spring
Java
```

and it doesn't move Java to the end.

---

# 9. Important distinction — insertion order vs access order

By default, LinkedHashSet maintains:

> **Insertion order.**

Suppose:

```java
set.add("A");
set.add("B");
set.add("C");
```

Then:

```java
set.contains("A");
```

doesn't change the iteration order.

Still:

```text
A
B
C
```

Likewise, simply reading an element doesn't move it.

This is different from some `LinkedHashMap` configurations that can maintain **access order**.

For `LinkedHashSet`, the important interview answer is:

> **It maintains insertion order.**

---

# 10. Why not simply sort the HashSet?

Because sorting and insertion ordering are different requirements.

Suppose:

```text
Inserted:

50
10
30
20
```

Insertion order:

```text
50
10
30
20
```

Sorted order:

```text
10
20
30
50
```

HashSet:

```text
no guaranteed iteration order
```

LinkedHashSet:

```text
50
10
30
20
```

TreeSet:

```text
10
20
30
50
```

So:

```text
HashSet       → uniqueness
LinkedHashSet → uniqueness + insertion order
TreeSet       → uniqueness + sorted order
```

---

# 11. Complexity

For normal hash distribution:

| Operation | LinkedHashSet |
|---|---:|
| `add()` | Average O(1) |
| `contains()` | Average O(1) |
| `remove()` | Average O(1) |
| Iteration | O(n) |

Compared with HashSet:

```text
HashSet
  → O(1) average lookup

LinkedHashSet
  → O(1) average lookup
  → preserves insertion order
```

The additional ordering structure means additional memory overhead.

---

# 12. Why is iteration particularly useful?

Suppose we have:

```java
Set<String> supportedFormats =
    new LinkedHashSet<>();

supportedFormats.add("PDF");
supportedFormats.add("CSV");
supportedFormats.add("JSON");
```

Later:

```java
for (String format : supportedFormats) {
    ...
}
```

We know the iteration follows:

```text
PDF
CSV
JSON
```

This can be useful when the order in which options were configured or received matters.

---

# 13. Production use case — remove duplicates while preserving order

This is one of the most practical uses.

Suppose:

```java
List<String> values = List.of(
    "Java",
    "Spring",
    "Java",
    "Kafka",
    "Spring"
);
```

You want:

```text
Java
Spring
Kafka
```

while preserving the first occurrence.

You can do:

```java
Set<String> unique =
    new LinkedHashSet<>(values);
```

Now:

```text
Java
Spring
Kafka
```

This is a very common pattern.

---

# 14. Another production use case — request processing

Suppose an API receives:

```text
requestedFields:

id
name
email
name
address
email
```

You want to:

1. remove duplicates
2. preserve the order requested by the client

Then:

```java
Set<String> fields =
    new LinkedHashSet<>(requestedFields);
```

gives:

```text
id
name
email
address
```

This is a good example because neither pure HashSet nor TreeSet matches the requirement perfectly.

---

# 15. Memory trade-off

HashSet essentially needs:

```text
hash structure
```

LinkedHashSet needs:

```text
hash structure
+
ordering links
```

Conceptually:

```text
HashSet:

[A]
[B]
[C]


LinkedHashSet:

[A] ⇄ [B] ⇄ [C]
```

So LinkedHashSet uses more memory.

This gives us another production trade-off:

```text
Need order?
    ↓
pay additional memory/maintenance cost
```

If ordering isn't needed, HashSet is simpler.

---

# 16. LinkedHashSet vs HashSet

| Feature | HashSet | LinkedHashSet |
|---|---|---|
| Duplicates | Not allowed | Not allowed |
| Hash-based lookup | Yes | Yes |
| Average `add()` | O(1) | O(1) |
| Average `contains()` | O(1) | O(1) |
| Insertion order | Not guaranteed | Guaranteed |
| Memory overhead | Lower | Higher |
| Internal ordering links | No | Yes |

The decision:

```text
Only uniqueness?
     ↓
HashSet

Uniqueness + insertion order?
     ↓
LinkedHashSet
```

---

# 17. LinkedHashSet vs TreeSet

This is another important distinction.

Suppose we insert:

```text
30
10
20
```

### LinkedHashSet

```text
30
10
20
```

because it preserves insertion order.

### TreeSet

```text
10
20
30
```

because it maintains sorted order.

So:

```text
LinkedHashSet
    ↓
insertion order
```

while:

```text
TreeSet
    ↓
sorted order
```

Don't confuse the two.

---

# 18. What determines equality?

Exactly the same principle as HashSet:

```text
hashCode()
+
equals()
```

For example:

```java
class User {

    private final Long id;

    // equals + hashCode based on id
}
```

Then:

```java
Set<User> users =
    new LinkedHashSet<>();
```

will treat two Users with the same logical identity as duplicates **if** `equals()` and `hashCode()` correctly define that identity.

So LinkedHashSet doesn't eliminate the need to understand:

```text
equals/hashCode
```

It depends on them.

---

# 19. Mutable element warning still applies

This problem from HashSet carries directly into LinkedHashSet.

Suppose:

```java
class User {

    int id;

    @Override
    public int hashCode() {
        return Integer.hashCode(id);
    }
}
```

Then:

```java
User user = new User(10);

Set<User> users =
    new LinkedHashSet<>();

users.add(user);
```

Then:

```java
user.id = 20;
```

Now the hash identity changed after insertion.

This can make operations such as:

```java
users.contains(user);
users.remove(user);
```

behave unexpectedly.

So the same production rule applies:

> Don't mutate fields used by `equals()`/`hashCode()` while the object is stored in a hash-based Set.

---

# 20. Can LinkedHashSet contain null?

Yes.

For example:

```java
Set<String> set =
    new LinkedHashSet<>();

set.add(null);
set.add("Java");
```

It can contain the null element.

You shouldn't confuse this with:

```text
ArrayDeque
```

which does not permit null.

---

# 21. Is LinkedHashSet thread-safe?

No.

Like HashSet:

```java
LinkedHashSet
```

is not automatically thread-safe.

If multiple threads modify it concurrently, you need an appropriate concurrency strategy.

For example, a synchronized wrapper can be used in suitable scenarios:

```java
Set<String> set =
    Collections.synchronizedSet(
        new LinkedHashSet<>()
    );
```

But remember our earlier lesson:

> Synchronizing individual collection operations doesn't automatically make an arbitrary sequence of operations atomic.

---

# 22. Interview question — "How does LinkedHashSet maintain insertion order?"

Strong answer:

> "`LinkedHashSet` uses hash-based storage for uniqueness and additionally maintains links between entries so iteration can follow insertion order. In the Java implementation, this behavior is based on the linked ordering maintained by the underlying linked hash map structure."

That's much better than:

> "It uses a LinkedList."

**Don't say LinkedHashSet is implemented as HashSet + LinkedList.**

The implementation relationship is better understood as:

```text
LinkedHashSet
      ↓
LinkedHashMap
      ↓
HashMap functionality
      +
linked ordering
```

---

# 23. Interview question — "Does LinkedHashSet have O(1) lookup?"

Under normal hash distribution:

```text
add       → O(1) average
contains  → O(1) average
remove    → O(1) average
```

while also maintaining insertion order.

---

# 24. Interview question — "Why not use TreeSet if I need order?"

Ask:

> **What order?**

If the requirement is:

```text
order elements according to insertion
```

use:

```text
LinkedHashSet
```

If the requirement is:

```text
sort elements
```

use:

```text
TreeSet
```

That's the key distinction.

---

# 25. Senior-level design question

Imagine an API receives:

```text
["PAYMENT", "REFUND", "PAYMENT", "TRANSFER"]
```

Requirement:

> Remove duplicate event types while preserving the first-seen order.

What would you use?

```java
Set<String> types =
    new LinkedHashSet<>(eventTypes);
```

Why?

```text
Uniqueness      → Set
First-seen order → LinkedHashSet
Fast membership → hash-based structure
```

That's exactly how you should approach collection questions in production:

```text
Requirement
    ↓
Data structure properties
    ↓
Implementation
```

rather than memorizing collection names.

---

# 26. Final mental model

Keep this picture:

```text
                    SET
                     │
          ┌──────────┼──────────┐
          │          │          │
       HashSet   LinkedHashSet  TreeSet
          │          │          │
          ↓          ↓          ↓
      Hash-based  Hash-based   Tree-based
      uniqueness  uniqueness    uniqueness
          │          │          │
          ↓          ↓          ↓
       no order   insertion     sorted
                   order        order
```

And:

```text
HashSet
   ↓
HashMap

LinkedHashSet
   ↓
LinkedHashMap
```

---

# Set progress

```text
Set
 │
 ├── HashSet          ✅
 │     ├── Hashing
 │     ├── Buckets
 │     ├── Collision
 │     ├── equals/hashCode
 │     ├── Resize
 │     └── Treeification
 │
 ├── LinkedHashSet    ✅
 │     ├── Hash-based uniqueness
 │     ├── Insertion order
 │     └── Linked ordering
 │
 └── TreeSet          ← NEXT
```

## Next: TreeSet

This will introduce a **different mechanism entirely**:

```text
HashSet
   → hash-based
   → average O(1)

TreeSet
   → tree-based
   → O(log n)
   → sorted elements
```

We'll then need to understand **Comparable vs Comparator**, natural ordering, custom sorting, `first()`, `last()`, range operations like `subSet()`, and the very important question:

> **How can TreeSet decide that two objects are "duplicates" even when `equals()` says they are different?**