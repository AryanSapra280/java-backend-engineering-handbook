# PART 3 — HASHMAP INTERNALS 🔥🔥🔥

This is one of the **highest-value Core Java topics** for your EPAM interview.

Don't memorize isolated facts. The goal is to be able to explain this entire chain confidently:

```text
put(key, value)
      ↓
hashCode()
      ↓
hash spreading
      ↓
bucket index
      ↓
Node<K,V>
      ↓
collision
      ↓
equals()
      ↓
linked list / tree
      ↓
load factor
      ↓
resize
      ↓
rehashing
```

---

# 1. What is HashMap?

### Interview question

> How does HashMap work internally?

### Strong answer

> HashMap stores key-value pairs using a hash table. Internally, it maintains an array of buckets. For a key, HashMap calculates a hash from the key's `hashCode()`, uses that hash to determine a bucket index, and stores the key-value entry there.
>
> If multiple keys map to the same bucket, a collision occurs. HashMap handles collisions using a linked structure, and in modern Java implementations, sufficiently large collision chains can be converted into a balanced tree.
>
> During lookup, HashMap uses the hash to locate the bucket and then uses `equals()` to identify the exact key.

That is the **30-second answer**.

Now let's go deep.

---

# 2. What does HashMap actually contain?

Conceptually:

```text
HashMap
   |
   ↓
Node[] table
```

Each bucket can contain an entry.

A simplified Node looks conceptually like:

```java
class Node<K,V> {
    int hash;
    K key;
    V value;
    Node<K,V> next;
}
```

So:

```text
table
 ┌─────┐
0│     │
 ├─────┤
1│Node │──► Node ──► Node
 ├─────┤
2│     │
 ├─────┤
3│Node │
 ├─────┤
4│     │
 └─────┘
```

---

# 3. Let's execute `put()`

Suppose:

```java
Map<String, Integer> map = new HashMap<>();

map.put("Aryan", 100);
```

What happens?

## Step 1 — Calculate hash

HashMap obtains:

```java
"Aryan".hashCode()
```

The exact hash value isn't important for interview understanding.

Then HashMap applies hash spreading/mixing.

Conceptually:

```text
key
 ↓
hashCode()
 ↓
spread/mix hash
```

---

# 4. Why does HashMap spread the hash?

Because poor distribution of hash codes can cause collisions.

HashMap uses bits of the hash to improve distribution before calculating the bucket.

In modern Java's implementation, conceptually you'll encounter:

```java
h ^ (h >>> 16)
```

The purpose is to mix higher bits into lower bits.

You don't need to memorize the exact source expression as much as the reason:

> HashMap spreads the hash so that information from higher bits contributes to bucket selection and helps reduce clustering.

---

# 5. How is the bucket index calculated?

A very important implementation detail.

If table capacity is a power of two, HashMap can use:

```java
index = (n - 1) & hash;
```

where:

```text
n = table length
```

For example:

```text
capacity = 16
```

Then:

```text
index = (16 - 1) & hash
      = 15 & hash
```

This is efficient.

---

# 6. Why power-of-two capacity?

Because:

```text
n - 1
```

becomes a bit mask.

For:

```text
n = 16
```

binary:

```text
16     = 10000
15     = 01111
```

Therefore:

```text
hash & 01111
```

quickly extracts the relevant lower bits.

### Interview answer

> HashMap uses power-of-two table capacities so bucket index calculation can efficiently use `(n - 1) & hash`, and resizing can also efficiently redistribute entries.

---

# 7. First insertion

Suppose:

```java
map.put("Aryan", 100);
```

After calculating the bucket:

```text
table[index]
      |
      ↓
   Node
   ├── hash
   ├── key = Aryan
   ├── value = 100
   └── next = null
```

Simple.

---

# 8. What if another key goes to the same bucket?

Suppose:

```text
"Aryan" → bucket 5
"Rahul" → bucket 5
```

This is a **collision**.

HashMap cannot replace Aryan simply because the bucket is the same.

It checks whether the key itself is equal.

Conceptually:

```text
bucket 5
   |
   ↓
[Aryan,100] → [Rahul,200]
```

This is collision handling.

---

# 9. Why both hashCode and equals?

This is one of the most important questions.

Suppose:

```text
key A
hash = 100
```

and:

```text
key B
hash = 100
```

They land in the same bucket.

HashMap then needs to determine:

> Is this the same key?

That's where `equals()` comes in.

Conceptually:

```text
hashCode()
    ↓
find candidate bucket
    ↓
compare hash
    ↓
equals()
    ↓
same key?
```

So:

> `hashCode()` narrows down where to look. `equals()` determines logical key equality.

---

# 10. Can two unequal objects have the same hashCode?

YES.

Example conceptually:

```text
A.hashCode() = 100
B.hashCode() = 100

A.equals(B) = false
```

That's a collision.

This is completely valid.

But this must never happen:

```text
A.equals(B) = true

A.hashCode() != B.hashCode()
```

That violates the contract.

---

# 11. The classic interview question

### Q:

> If two objects have the same hashCode, are they necessarily equal?

### Answer:

**No.**

Same hash code can mean collision.

But:

> If two objects are equal according to `equals()`, they must have the same hash code.

This distinction is extremely important.

---

# 12. What happens when the same key is inserted?

Suppose:

```java
map.put("Aryan", 100);
map.put("Aryan", 200);
```

HashMap finds the existing key.

Conceptually:

```text
hash matches
     ↓
equals() == true
     ↓
existing entry found
     ↓
value replaced
```

Final:

```text
Aryan → 200
```

The size remains:

```text
1
```

---

# 13. Collision chain

Suppose:

```text
Bucket 4
   |
   ↓
Node A → Node B → Node C
```

Lookup:

```java
map.get(key)
```

HashMap first finds bucket 4.

Then it checks nodes.

Conceptually:

```text
hash matches A?
    ↓
equals?
    ↓
no

hash matches B?
    ↓
equals?
    ↓
yes

return B.value
```

Without good hashing, lookup can become slower.

This is why collision management matters.

---

# 14. Treeification 🔥🔥

Modern Java HashMap doesn't simply allow a huge linked list forever.

When a bucket's collision chain becomes sufficiently large, it can be converted into a tree structure.

The important thresholds are:

```text
TREEIFY_THRESHOLD = 8
UNTREEIFY_THRESHOLD = 6
MIN_TREEIFY_CAPACITY = 64
```

### Important nuance

Don't say:

> "At 8 elements, HashMap always converts the bucket into a tree."

That's incomplete.

If the table is too small, HashMap may **resize instead of treeifying**.

Treeification requires the table to have reached the minimum capacity of **64**.

---

# 15. Why treeification?

A long linked list can make lookup approach:

```text
O(n)
```

A balanced tree can improve lookup toward:

```text
O(log n)
```

under the tree-bin conditions.

So conceptually:

```text
Few collisions:

Node → Node → Node


Many collisions:

       Node
      /    \
   Node    Node
   /          \
 Node         Node
```

The actual implementation is more nuanced, but that's the interview-level mental model.

---

# 16. Why doesn't HashMap immediately treeify at 8?

This is a **great follow-up question**.

Suppose the table is small.

Instead of converting one bucket into a tree, HashMap may prefer:

```text
resize table
```

because increasing the number of buckets may distribute the entries and reduce collisions naturally.

Hence:

```text
collision increases
       ↓
table small?
       ↓
resize
       ↓
otherwise
       ↓
treeify
```

---

# 17. Load factor 🔥

Default HashMap load factor:

```text
0.75
```

Suppose capacity:

```text
16
```

Threshold:

```text
16 × 0.75 = 12
```

So approximately when the number of entries reaches the threshold, HashMap resizes.

### Interview answer

> Load factor controls how full the hash table is allowed to become before resizing. The default is 0.75, which is a tradeoff between memory usage and collision frequency.

---

# 18. Why not load factor 1.0?

Because the table could become more densely populated.

That can increase collisions.

But a lower load factor isn't automatically better either.

For example:

```text
0.5
```

means more buckets and potentially more memory.

So:

```text
lower load factor
→ fewer collisions
→ more memory

higher load factor
→ more compact
→ potentially more collisions
```

`0.75` is a practical compromise.

---

# 19. Capacity vs size vs threshold

Interviewers sometimes deliberately mix these up.

### Size

Number of mappings currently stored.

Example:

```java
map.size()
```

could be:

```text
100
```

### Capacity

Number of buckets in the internal table.

Example:

```text
16
32
64
128
...
```

### Threshold

Approximate number of entries at which resizing occurs.

Conceptually:

```text
threshold = capacity × loadFactor
```

Example:

```text
capacity = 16
loadFactor = 0.75

threshold = 12
```

---

# 20. Resize

Suppose:

```text
capacity = 16
threshold = 12
```

and we cross the threshold.

HashMap grows the table.

Conceptually:

```text
16
 ↓
32
```

Then entries need to be redistributed across the new bucket array.

This is why resizing is expensive.

### Complexity

A resize involves processing existing entries:

```text
O(n)
```

Therefore, an individual insertion can occasionally cost O(n).

But normal insertion remains:

```text
O(1) average/amortized
```

---

# 21. Does HashMap recalculate every object's hashCode during resize?

This is a subtle question.

Don't explain it as:

> "HashMap calls the object's `hashCode()` again for every entry."

The implementation already stores the hash in the node and uses optimized redistribution logic during resize.

The important interview-level idea is:

> Existing entries are redistributed into the expanded table based on their stored hash and the new capacity.

---

# 22. Why resize from 16 → 32?

Because HashMap uses power-of-two capacities.

Common progression:

```text
16
32
64
128
256
...
```

This makes redistribution efficient.

An important implementation detail:

When capacity doubles, an existing entry generally either:

```text
stays at the same index
```

or moves by:

```text
oldCapacity
```

This is an optimization made possible by the power-of-two design.

You don't need to implement this from memory unless specifically asked.

---

# 23. Initial capacity

You can specify:

```java
Map<String, Integer> map =
        new HashMap<>(100);
```

But be careful:

> The constructor argument is related to initial sizing, not necessarily exactly "100 buckets" in the way beginners often assume.

HashMap internally rounds table capacity to an appropriate power-of-two size when the table is allocated.

---

# 24. Mutable keys 🔥🔥🔥

This is one of the **best HashMap interview traps**.

Consider:

```java
class Employee {

    int id;
    String name;

    // equals/hashCode use id
}
```

Then:

```java
Employee e = new Employee(1, "Aryan");

Map<Employee, String> map = new HashMap<>();

map.put(e, "Developer");
```

Now:

```java
e.id = 2;
```

If `hashCode()` depends on `id`, the hash has changed.

But the entry is still physically sitting in the bucket determined by the **old hash**.

Now:

```java
map.get(e);
```

may return:

```text
null
```

even though the exact object reference is still present in the map.

---

# 25. Why?

Initially:

```text
Employee(id=1)
      ↓
hash = H1
      ↓
bucket 5
```

After mutation:

```text
Employee(id=2)
      ↓
hash = H2
      ↓
bucket 12
```

But the entry is still sitting in:

```text
bucket 5
```

Lookup searches:

```text
bucket 12
```

and doesn't find it.

### Interview answer

> Keys used in hash-based collections should be effectively immutable with respect to the fields used by `equals()` and `hashCode()`.

This is why immutable objects such as `String` make excellent HashMap keys.

---

# 26. Can I use a mutable object as a HashMap key?

Technically yes.

But it is dangerous if fields participating in equality/hash calculation can change while the object is a key.

This is a fantastic interview distinction:

> **Possible** doesn't mean **safe**.

---

# 27. HashMap with null

Can HashMap have null?

Yes.

```java
Map<String, Integer> map = new HashMap<>();

map.put(null, 100);
map.put("A", null);
```

HashMap permits:

```text
one null key
multiple null values
```

Why only one null key?

Because keys must be unique.

---

# 28. HashSet and HashMap connection

This is often asked:

> How does HashSet ensure uniqueness?

Conceptually HashSet uses a HashMap internally.

When you do:

```java
set.add("Java");
```

the element becomes a key in the backing map.

Conceptually:

```text
HashSet
   |
   ↓
HashMap
   |
   ├── "Java" → PRESENT
   ├── "Spring" → PRESENT
   └── "Kafka" → PRESENT
```

The value isn't the important part; the key provides uniqueness.

---

# 29. HashMap vs Hashtable vs ConcurrentHashMap

### HashMap

```text
not thread-safe
allows null key
allows null values
```

### Hashtable

```text
synchronized
no null key
no null values
legacy
```

### ConcurrentHashMap

```text
thread-safe
doesn't allow null keys/values
designed for concurrent access
supports atomic operations
```

### Why doesn't ConcurrentHashMap allow null?

This is a good conceptual question.

In concurrent code, `null` can create ambiguity between:

```text
key absent
```

and:

```text
key present with null value
```

ConcurrentHashMap avoids that ambiguity by disallowing null keys and values.

---

# 30. ConcurrentHashMap internals

You need to be careful here because older interview explanations often say:

> "ConcurrentHashMap uses segments."

That was true for older Java 7-era implementations.

Modern Java implementations do **not** use the old fixed-segment design.

Modern ConcurrentHashMap uses a combination of:

- CAS
- volatile reads/writes
- synchronized blocks around specific update operations/bins when needed
- internal bucket/node structures

The important point:

> It avoids locking the entire map for ordinary concurrent operations.

---

# 31. Why is ConcurrentHashMap better than `Collections.synchronizedMap()`?

Consider:

```java
Map<K,V> map =
    Collections.synchronizedMap(new HashMap<>());
```

This provides synchronized access around map operations.

But concurrency can be more restrictive because synchronization is broader.

ConcurrentHashMap is specifically designed for concurrent access and provides better scalability for many concurrent read/write scenarios.

Also, it provides useful atomic operations:

```java
putIfAbsent()
computeIfAbsent()
compute()
merge()
```

---

# 32. Atomic compound operation

Consider:

```java
if (!map.containsKey(key)) {
    map.put(key, value);
}
```

Is this atomic?

**No.**

Two threads can do:

```text
Thread A:
containsKey → false

Thread B:
containsKey → false

Thread A:
put

Thread B:
put
```

Instead:

```java
map.putIfAbsent(key, value);
```

expresses the operation atomically.

This is an important bridge between:

```text
Collections
       ↓
HashMap
       ↓
Concurrency
```

---

# 33. `computeIfAbsent()` — very useful

Suppose:

```java
Map<String, List<String>> map = new HashMap<>();
```

You want to add a value to a list.

Without `computeIfAbsent()`:

```java
List<String> list = map.get(key);

if (list == null) {
    list = new ArrayList<>();
    map.put(key, list);
}

list.add(value);
```

With:

```java
map.computeIfAbsent(
    key,
    k -> new ArrayList<>()
).add(value);
```

This pattern becomes extremely useful in:

- grouping
- graph construction
- indexes
- caches
- frequency/grouping logic

And Streams' `groupingBy()` essentially gives you a higher-level way to express similar grouping logic.

---

# 🔥 34. HashMap interview coding

## Problem 1 — Two Sum

Given:

```text
[2, 7, 11, 15]
target = 9
```

Return:

```text
[0, 1]
```

### Clean solution

```java
public static int[] twoSum(int[] nums, int target) {

    Map<Integer, Integer> map = new HashMap<>();

    for (int i = 0; i < nums.length; i++) {

        int complement = target - nums[i];

        if (map.containsKey(complement)) {
            return new int[] {
                map.get(complement),
                i
            };
        }

        map.put(nums[i], i);
    }

    return new int[0];
}
```

### Complexity

```text
Time:  O(n)
Space: O(n)
```

### Why HashMap?

We need:

```text
value → index
```

so that we can find the complement in O(1) average time.

---

# 35. Two Sum interviewer follow-up

### Q:

Why not use nested loops?

That gives:

```text
O(n²)
```

HashMap reduces it to:

```text
O(n)
```

### Q:

What if duplicate values exist?

The map stores the appropriate previously seen index, and we check the complement before inserting the current value.

Example:

```text
[3, 3]
target = 6
```

At index 0:

```text
complement = 3
not found
put(3, 0)
```

At index 1:

```text
complement = 3
found → return [0,1]
```

---

# 36. Problem 2 — Group Anagrams

Input:

```text
["eat", "tea", "tan", "ate", "nat", "bat"]
```

Output conceptually:

```text
[
    ["eat", "tea", "ate"],
    ["tan", "nat"],
    ["bat"]
]
```

### HashMap approach

Use the sorted characters as the key.

```java
public static List<List<String>> groupAnagrams(
        String[] words) {

    Map<String, List<String>> map = new HashMap<>();

    for (String word : words) {

        char[] chars = word.toCharArray();

        Arrays.sort(chars);

        String key = new String(chars);

        map.computeIfAbsent(
                key,
                k -> new ArrayList<>()
        ).add(word);
    }

    return new ArrayList<>(map.values());
}
```

Example:

```text
eat → aet
tea → aet
ate → aet

tan → ant
nat → ant
```

So:

```text
aet → [eat, tea, ate]
ant → [tan, nat]
```

This is a very good example of:

```text
HashMap
+
computeIfAbsent()
```

---

# 37. One more important coding pattern — first non-repeating

We've already seen it, but now understand why the collection choices matter:

```java
Map<Character, Integer> frequency =
        new LinkedHashMap<>();
```

Why not HashMap?

Because we need the first character **in original order**.

This is the sort of explanation interviewers appreciate.

---

# 38. EPAM-style rapid-fire

You should now be able to answer these without hesitation:

### What is the average lookup complexity of HashMap?

> O(1) average.

### What happens during a collision?

> Multiple entries occupy the same bucket and are searched using their hash and equality; sufficiently large bins can be treeified.

### What is default load factor?

> 0.75.

### Why 0.75?

> Practical tradeoff between memory usage and collision frequency.

### Default initial capacity?

> 16, conceptually for the default HashMap configuration once the table is allocated.

### Treeification threshold?

> 8 nodes in a bin, subject to the minimum table capacity condition.

### Minimum capacity for treeification?

> 64.

### Untreeification threshold?

> 6.

### Can unequal objects have the same hash code?

> Yes.

### Can equal objects have different hash codes?

> No, that violates the contract.

### Why are mutable keys dangerous?

> Mutating fields used by `hashCode()`/`equals()` can make the key effectively unreachable in the map.

### Why power-of-two capacity?

> Efficient bucket calculation and optimized redistribution during resize.

### HashMap thread-safe?

> No.

### ConcurrentHashMap thread-safe?

> Yes.

### Can ConcurrentHashMap contain null?

> No null keys or values.

### HashMap null?

> One null key and multiple null values are allowed.

### HashSet internally?

> HashMap-based.

### TreeMap ordering?

> Keys sorted by natural ordering or Comparator.

### LinkedHashMap?

> HashMap-like lookup with predictable insertion/access ordering.

### PriorityQueue?

> Heap-based priority queue; head is the highest-priority element according to its ordering.

### Does PriorityQueue iteration return sorted elements?

> No. Repeated `poll()` gives priority order.

---

# 🎯 The HashMap mental model you need for EPAM

If the interviewer says:

> **"Explain HashMap internals."**

Don't dump 20 disconnected facts.

Say this:

> "HashMap maintains an array of buckets. When I put a key-value pair, it obtains the key's hashCode and applies hash spreading, then calculates a bucket index using the table capacity. If the bucket is empty, the node is inserted. If there is a collision, HashMap compares the hash and then uses equals to determine whether the key already exists; otherwise another node is added to that bucket. In modern Java, if a collision bin becomes sufficiently large and the table is large enough, it can be treeified. HashMap resizes when the number of entries crosses the load-factor threshold, with a default load factor of 0.75. The table uses power-of-two capacities to make bucket calculation and resizing efficient."

Then **stop**.

Let the interviewer ask the next question.

If they ask:

> "Why 0.75?"

You answer.

If:

> "Why treeify?"

You answer.

If:

> "What happens if key changes?"

You answer.

If:

> "What happens during resize?"

You answer.

That's how you demonstrate actual understanding instead of reciting everything.

---

## 🔥 Part 3 takeaway

The five things I want you to be rock-solid on are:

```text
1. hashCode() → bucket selection

2. equals() → exact key comparison

3. collision → linked nodes → possible treeification

4. load factor → resize

5. mutable key → dangerous HashMap behavior
```

And the **most important code patterns** are:

```java
map.getOrDefault(key, 0) + 1
```

```java
map.putIfAbsent(key, value)
```

```java
map.computeIfAbsent(key, k -> new ArrayList<>())
```

```java
map.merge(key, 1, Integer::sum)
```

```java
new LinkedHashSet<>(list)
```

```java
PriorityQueue + HashMap
```

These will repeatedly appear when we get into **Streams and coding**.

---

# PART 4 — Java 8 Functional Programming 🔥🔥

Next we'll go into:

```text
Lambda
   ↓
Functional Interface
   ↓
Predicate
Function
Consumer
Supplier
   ↓
UnaryOperator / BinaryOperator
   ↓
Method References
   ↓
effectively final
   ↓
this inside lambda
   ↓
default/static interface methods
```

And I'll connect each one to **actual Stream coding**, because memorizing `Predicate<T>` definitions without knowing where you'd use it won't help much in your EPAM round.