Exactly. Since we already covered **LinkedHashSet** and **TreeSet**, we don't need to repeat the underlying data structures from scratch. We just need to transfer that understanding from **Set → Map**, then capture the Map-specific interview points.

# LinkedHashMap

## 1. Problem

`HashMap` gives fast lookup:

```java
Map<String, Integer> map = new HashMap<>();
```

Expected:

```text
put/get/remove → O(1)
```

But:

> **HashMap does not guarantee iteration order.**

Suppose you need:

```text
Insert:
A
B
C
D

Iterate:
A
B
C
D
```

while still wanting HashMap-style lookup.

That's the problem `LinkedHashMap` solves.

---

# 2. Core idea

```java
Map<String, Integer> map = new LinkedHashMap<>();
```

It combines:

```text
Hashing
+
Linked ordering
```

Mental model:

```text
LinkedHashMap
      |
      +----------------+
      |                |
   HashMap         Linked list
      |                |
 fast lookup       ordering
```

So:

```text
HashMap
→ key lookup

LinkedHashMap
→ key lookup + predictable iteration order
```

---

# 3. Internal structure

This is the important part.

`LinkedHashMap` extends `HashMap`.

Its entries maintain additional links for ordering.

Conceptually:

```java
class Entry<K,V> extends HashMap.Node<K,V> {
    Entry<K,V> before;
    Entry<K,V> after;
}
```

So an entry conceptually has:

```text
hash
key
value
next
before
after
```

The `next` relationship is related to the HashMap bucket structure.

The `before/after` links maintain the iteration order.

---

# 4. Mental picture

Suppose:

```java
map.put("A", 10);
map.put("B", 20);
map.put("C", 30);
```

Hash buckets might look something like:

```text
Hash table:

bucket 2 → A
bucket 5 → C
bucket 7 → B
```

Notice the bucket positions don't necessarily represent insertion order.

Separately, LinkedHashMap maintains:

```text
head
 ↓
A ⇄ B ⇄ C
             ↓
            tail
```

Iteration follows:

```text
A → B → C
```

rather than walking buckets and exposing their arbitrary order.

---

# 5. Default ordering: insertion order

By default:

```java
LinkedHashMap<String, Integer> map =
        new LinkedHashMap<>();

map.put("A", 1);
map.put("B", 2);
map.put("C", 3);
```

Iteration:

```text
A
B
C
```

If you update:

```java
map.put("B", 200);
```

you haven't inserted a new key.

So the order remains:

```text
A
B
C
```

---

# 6. Duplicate key does not create another ordering node

This is important.

```java
map.put("A", 10);
map.put("B", 20);
map.put("A", 100);
```

Final:

```text
A → 100
B → 20
```

Order:

```text
A
B
```

The second `put("A")` updates the existing mapping.

---

# 7. Access-order mode ⭐

This is the Map-specific feature you should definitely know.

LinkedHashMap has a constructor:

```java
new LinkedHashMap<>(
    initialCapacity,
    loadFactor,
    accessOrder
);
```

If:

```java
accessOrder = true
```

the linked ordering can represent **access order** rather than insertion order.

Example:

```java
LinkedHashMap<String, Integer> map =
        new LinkedHashMap<>(16, 0.75f, true);

map.put("A", 1);
map.put("B", 2);
map.put("C", 3);
```

Initially:

```text
A → B → C
```

Now:

```java
map.get("A");
```

Because `"A"` was accessed, it moves toward the end.

Order becomes:

```text
B → C → A
```

Then:

```java
map.get("B");
```

becomes:

```text
C → A → B
```

This is extremely useful for implementing an **LRU cache**.

---

# 8. LRU cache connection ⭐⭐⭐⭐⭐

LRU means:

> Least Recently Used.

Suppose capacity is 3:

```text
A B C
```

Access:

```java
get(A)
```

Now:

```text
B C A
```

Access:

```java
get(B)
```

Now:

```text
C A B
```

The first entry:

```text
C
```

is now the least recently used.

LinkedHashMap provides exactly the ordering mechanism needed for this.

---

# 9. `removeEldestEntry()`

This is another **very high-value interview point**.

LinkedHashMap provides:

```java
protected boolean removeEldestEntry(
        Map.Entry<K,V> eldest)
```

You can override it to automatically remove the oldest entry.

Example:

```java
class LruCache<K, V>
        extends LinkedHashMap<K, V> {

    private final int capacity;

    public LruCache(int capacity) {
        super(16, 0.75f, true);
        this.capacity = capacity;
    }

    @Override
    protected boolean removeEldestEntry(
            Map.Entry<K, V> eldest) {

        return size() > capacity;
    }
}
```

Now:

```java
LruCache<Integer, String> cache =
        new LruCache<>(3);

cache.put(1, "A");
cache.put(2, "B");
cache.put(3, "C");
```

Then:

```java
cache.get(1);
```

Order becomes approximately:

```text
2 → 3 → 1
```

Then:

```java
cache.put(4, "D");
```

The eldest entry:

```text
2
```

can be removed.

Result:

```text
3 → 1 → 4
```

This is a **classic Java interview coding question**.

---

# 10. Complexity

Because LinkedHashMap still uses hashing:

| Operation | Expected |
|---|---:|
| `put()` | O(1) |
| `get()` | O(1) |
| `remove()` | O(1) |
| `containsKey()` | O(1) |
| Iteration | O(n) |

There is some additional memory overhead because of the ordering links.

---

# 11. Production considerations

Use `LinkedHashMap` when you need:

```text
Fast hash lookup
+
Predictable iteration order
```

Typical examples:

### Preserve input order

```text
request fields
configuration properties
processed records
```

### LRU-style in-memory cache

```text
recently accessed items
```

### Deterministic output

Useful when generating:

```text
reports
serialized data
API responses
```

where stable iteration order is desirable.

---

# 12. LinkedHashMap vs HashMap

| | HashMap | LinkedHashMap |
|---|---|---|
| Hash lookup | Expected O(1) | Expected O(1) |
| Ordering | None guaranteed | Predictable |
| Default order | — | Insertion order |
| Access-order mode | ❌ | ✅ |
| Memory | Lower | Higher |
| LRU implementation | Not naturally suited | Excellent fit |
| Internal structure | Hash table | Hash table + links |

### Mental model

```text
HashMap
   ↓
hashing
   ↓
fast lookup


LinkedHashMap
   ↓
hashing + linked ordering
   ↓
fast lookup + predictable iteration
```

---

# TreeMap

Now this one is also mostly familiar because we already covered **TreeSet → TreeMap → Red-Black Tree**.

The Map-specific version is simply:

> `TreeMap` stores **key-value pairs sorted according to the keys**.

---

## 1. Problem

HashMap:

```text
A → 10
C → 30
B → 20
```

doesn't guarantee sorted iteration.

Suppose we need:

```text
A → 10
B → 20
C → 30
```

and also want efficient operations such as:

```java
firstKey()
lastKey()
floorKey()
ceilingKey()
lowerKey()
higherKey()
subMap()
headMap()
tailMap()
```

That's where TreeMap comes in.

---

# 2. Internal structure

Just like TreeSet:

```text
TreeMap
   ↓
Red-Black Tree
   ↓
nodes ordered by key
```

Conceptually:

```text
             50
           /    \
         30      70
        /  \    /  \
      20   40  60   80
```

Each node contains:

```text
key
value
left
right
parent
color
```

The key determines the ordering.

---

# 3. Ordering

TreeMap requires keys to be comparable.

Either the key implements:

```java
Comparable
```

or you provide:

```java
Comparator
```

Example:

```java
TreeMap<Integer, String> map =
        new TreeMap<>();

map.put(30, "C");
map.put(10, "A");
map.put(20, "B");
```

Iteration:

```text
10 → A
20 → B
30 → C
```

The insertion order doesn't matter.

The **key ordering** determines iteration order.

---

# 4. Complexity

Because it's a balanced Red-Black Tree:

| Operation | Complexity |
|---|---:|
| `put()` | O(log n) |
| `get()` | O(log n) |
| `remove()` | O(log n) |
| `containsKey()` | O(log n) |
| iteration | O(n) |

This is the major contrast with HashMap.

```text
HashMap
→ expected O(1)

TreeMap
→ O(log n)
```

But TreeMap gives you **ordering and range operations**.

---

# 5. The extremely important `compareTo()` rule

This is exactly the same concept we discussed with TreeSet.

TreeMap determines key equivalence using comparison.

Suppose:

```java
Comparator<Employee> comparator =
        Comparator.comparing(Employee::getSalary);
```

Then:

```text
Employee A salary = 100
Employee B salary = 100
```

The comparator returns:

```text
0
```

TreeMap considers them equivalent for its ordering purposes.

So you can accidentally lose one mapping:

```java
map.put(employeeA, "A");
map.put(employeeB, "B");
```

if the comparator says:

```text
compare(employeeA, employeeB) == 0
```

even though:

```java
employeeA.equals(employeeB) == false
```

Therefore:

> A TreeMap's comparator should generally be consistent with `equals()` unless you intentionally want a different equivalence relation.

---

# 6. TreeMap's killer feature: range queries

This is where TreeMap is much more than "a sorted Map."

Suppose:

```java
TreeMap<Integer, String> map = new TreeMap<>();

map.put(10, "A");
map.put(20, "B");
map.put(30, "C");
map.put(40, "D");
map.put(50, "E");
```

### `firstKey()`

```java
map.firstKey();
```

returns:

```text
10
```

### `lastKey()`

```java
map.lastKey();
```

returns:

```text
50
```

### `higherKey(20)`

```text
30
```

### `lowerKey(20)`

```text
10
```

### `ceilingKey(25)`

```text
30
```

### `floorKey(25)`

```text
20
```

These are extremely useful when working with ordered data.

---

# 7. `subMap()`

```java
map.subMap(20, 50);
```

gives the mappings in the specified range according to TreeMap's range semantics.

Similarly:

```java
map.headMap(30);
```

and:

```java
map.tailMap(30);
```

are useful for range-based processing.

This is one reason you wouldn't simply replace every TreeMap with HashMap + sorting afterward.

---

# 8. HashMap vs LinkedHashMap vs TreeMap

This is the **final Map selection table** you should put in your notes:

| Requirement | Use |
|---|---|
| Fast lookup, no ordering requirement | `HashMap` |
| Fast lookup + insertion order | `LinkedHashMap` |
| Fast lookup + access order | `LinkedHashMap` |
| LRU-style cache | `LinkedHashMap` |
| Sorted keys | `TreeMap` |
| Range queries | `TreeMap` |
| `firstKey`, `lastKey`, `floorKey`, etc. | `TreeMap` |

Mental model:

```text
                    Map
                     |
       +-------------+-------------+
       |             |             |
    HashMap     LinkedHashMap    TreeMap
       |             |             |
   Hash table   Hash + links    Red-Black Tree
       |             |             |
   expected O(1) expected O(1)     O(log n)
                     |
              insertion/access
                  ordering
```

---

## What you should add to your notes

Because we've already deeply covered the Set versions, **don't duplicate pages of material**.

For `LinkedHashMap`, note these **5 things**:

```text
1. HashMap + doubly linked ordering structure
2. Default = insertion order
3. accessOrder=true = access order
4. removeEldestEntry() → LRU cache
5. Expected O(1), extra memory for links
```

For `TreeMap`, note these **5 things**:

```text
1. Backed by Red-Black Tree
2. Keys sorted by Comparable/Comparator
3. put/get/remove = O(log n)
4. compare() == 0 means equivalent keys for TreeMap
5. Navigable/range operations:
   firstKey, lastKey, floorKey, ceilingKey,
   lowerKey, higherKey, subMap, headMap, tailMap
```

**That completes HashMap → LinkedHashMap → TreeMap.**

The next Map topic should be **Hashtable**, but it's much smaller/less important than what we've done. After that, we should spend serious time on **ConcurrentHashMap**, because *that* is the concurrency-heavy Map topic worth going deep on for EPAM.