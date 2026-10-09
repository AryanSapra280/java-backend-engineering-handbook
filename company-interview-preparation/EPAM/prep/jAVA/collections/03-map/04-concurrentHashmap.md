# ConcurrentHashMap ⭐⭐⭐⭐⭐

Now we are at the **important part** of Map.

The key interview progression is:

```text
HashMap
  ↓
not thread-safe

Collections.synchronizedMap()
  ↓
thread-safe but coarse-grained locking

ConcurrentHashMap
  ↓
designed for concurrent access
```

We need to understand **why ConcurrentHashMap exists**, not just memorize that it is thread-safe.

---

## 1. The Problem

Imagine a Spring Boot application.

You have hundreds of HTTP requests being processed concurrently:

```text
Request 1 ──┐
Request 2 ──┤
Request 3 ──┤
Request 4 ──┼──> shared Map
Request 5 ──┤
Request 6 ──┘
```

Suppose the application maintains:

```java
Map<String, User> users;
```

Multiple threads need to:

```java
users.get(id);
users.put(id, user);
users.remove(id);
```

Using:

```java
HashMap
```

is unsafe if the Map is concurrently modified.

So we need a Map specifically designed for concurrent access.

That's:

```java
ConcurrentHashMap
```

---

# 2. Why not simply synchronize HashMap?

We could do:

```java
Map<String, User> users =
    Collections.synchronizedMap(new HashMap<>());
```

This gives us thread-safe individual Map operations.

But imagine:

```text
Thread A → get()
Thread B → put()
Thread C → get()
Thread D → put()
```

A synchronized wrapper uses synchronization around operations.

Conceptually:

```text
                Map lock
                   |
       +-----------+-----------+
       |           |           |
   Thread A    Thread B    Thread C
       |           |           |
      get()       put()       get()
```

Only one operation can hold that lock at a time.

So concurrency can become limited.

---

# 3. What does ConcurrentHashMap try to achieve?

We want:

```text
Thread A ──┐
Thread B ──┼──> ConcurrentHashMap
Thread C ──┤
Thread D ──┘
```

with as much independent concurrent work as possible.

The design therefore avoids simply putting:

```java
synchronized
```

around the entire Map.

Modern `ConcurrentHashMap` uses a combination of:

```text
CAS
+
fine-grained synchronization
+
careful internal coordination
```

The important mental model is:

> **Don't think "one big lock." Think "coordinate only the parts that actually need coordination."**

---

# 4. ConcurrentHashMap declaration

```java
import java.util.concurrent.ConcurrentHashMap;

ConcurrentHashMap<String, User> users =
        new ConcurrentHashMap<>();
```

Or:

```java
Map<String, User> users =
        new ConcurrentHashMap<>();
```

This is often preferable because the variable is declared against the interface.

---

# 5. Internal mental model

You already understand HashMap's structure:

```text
HashMap

table[]
   |
   +-- bucket
   +-- bucket
   +-- bucket
   +-- bucket
```

ConcurrentHashMap also organizes entries around hash-table buckets, but coordinates concurrent modifications using mechanisms designed for concurrency.

Conceptually:

```text
ConcurrentHashMap
        |
        ↓
     table[]
        |
   +----+----+----+
   |    |    |    |
 bucket bucket bucket
   |    |    |
   ↓    ↓    ↓
  A     B    C
```

If different threads are working on independent portions of the structure, they don't necessarily need to block each other with one global lock.

---

# 6. CAS — the key concept

You need to understand **CAS** because it is fundamental to ConcurrentHashMap.

CAS means:

> **Compare And Swap**

Imagine we have:

```text
value = null
```

Thread A wants to insert:

```text
A
```

CAS conceptually says:

```text
"If the value is still null,
 replace it with A."
```

Atomic operation:

```text
expected = null
newValue = A
```

If the current value is still:

```text
null
```

then:

```text
null → A
```

succeeds.

If another thread changed it first:

```text
null → B
```

then Thread A's CAS fails.

---

# 7. Why is CAS useful?

Without atomic coordination:

```text
Thread A                    Thread B

read null                   read null

write A                     write B
```

Both threads believe they won.

With CAS:

```text
Thread A                    Thread B

CAS(null → A)               CAS(null → B)

SUCCESS                     FAIL
```

Only one succeeds.

The other thread can then re-evaluate the current state.

This is one of the mechanisms ConcurrentHashMap uses for efficient concurrent updates.

---

# 8. Does ConcurrentHashMap use only CAS?

No.

This is an important interview nuance.

Don't say:

> "ConcurrentHashMap is lock-free."

That's incorrect.

Modern ConcurrentHashMap uses **CAS where appropriate and synchronized locking for certain bucket-level update operations**.

Think:

```text
Simple uncontended update
        ↓
      CAS

More complex bucket modification
        ↓
synchronized on relevant node/bucket structure
```

The goal is **fine-grained coordination**, not "no locks whatsoever."

---

# 9. `get()` is especially important

Consider:

```java
map.get("Java");
```

A major design goal is that reads can happen concurrently without acquiring a global lock.

Conceptually:

```text
Thread A → get(A)
Thread B → get(B)
Thread C → get(C)
Thread D → get(D)
```

can proceed concurrently.

This is one reason ConcurrentHashMap works well for read-heavy workloads.

---

# 10. `put()`

Suppose:

```java
map.put("Java", 100);
```

Conceptually:

```text
"Java"
   ↓
hash
   ↓
bucket
   ↓
inspect bucket
```

If the relevant location is empty, an atomic mechanism such as CAS can be used to install the new node.

If the bucket already contains entries, the update may require synchronization around the relevant bucket structure.

The important point:

```text
NOT:

entire map locked

BUT:

only the necessary portion is coordinated
```

---

# 11. Why this is better than one global lock

Imagine:

```text
Bucket 1 → Thread A
Bucket 5 → Thread B
Bucket 10 → Thread C
```

With a global lock:

```text
A gets lock
B waits
C waits
```

With fine-grained coordination:

```text
A → bucket 1
B → bucket 5
C → bucket 10
```

independent work can proceed concurrently when the operations don't conflict.

That's the fundamental concurrency advantage.

---

# 12. `putIfAbsent()` ⭐⭐⭐⭐⭐

This is one of the most important methods to know.

Suppose you want:

> Insert only if the key doesn't already exist.

A naïve approach:

```java
if (!map.containsKey(key)) {
    map.put(key, value);
}
```

Looks fine.

But under concurrency, it's broken.

### Thread A

```text
containsKey()
→ false
```

### Thread B

```text
containsKey()
→ false
```

Then:

```text
Thread A → put()
Thread B → put()
```

Both threads observed the old state.

That's a race condition.

---

# 13. Use `putIfAbsent()`

Instead:

```java
map.putIfAbsent(key, value);
```

This expresses the operation as one concurrent Map operation:

```text
if absent
    insert
```

rather than:

```text
check
+
separate insert
```

Example:

```java
ConcurrentHashMap<String, User> users =
        new ConcurrentHashMap<>();

users.putIfAbsent("101", user);
```

This is a very good interview example of **atomic compound Map behavior**.

---

# 14. `computeIfAbsent()` ⭐⭐⭐⭐⭐

Another extremely important method:

```java
map.computeIfAbsent(key, function);
```

Example:

```java
ConcurrentHashMap<String, User> cache =
        new ConcurrentHashMap<>();

User user = cache.computeIfAbsent(
    userId,
    id -> loadUserFromDatabase(id)
);
```

Mental model:

```text
Does userId exist?
       |
    +--+--+
    |     |
   yes    no
    |     |
 return  compute
 existing   |
           store
```

This is extremely useful for:

- caches
- lazy initialization
- grouping
- expensive object creation

---

# 15. Why `computeIfAbsent()` is better than manual check

Bad concurrent pattern:

```java
if (!map.containsKey(key)) {
    map.put(key, createValue());
}
```

Potentially:

```text
Thread A → sees absent
Thread B → sees absent

both create value
```

With:

```java
map.computeIfAbsent(key, k -> createValue());
```

the Map provides the appropriate atomic coordination for the mapping operation.

---

# 16. `merge()`

Very useful for counters.

Suppose:

```java
ConcurrentHashMap<String, Integer> counts =
        new ConcurrentHashMap<>();
```

Instead of:

```java
Integer count = counts.get(word);

if (count == null) {
    counts.put(word, 1);
} else {
    counts.put(word, count + 1);
}
```

use:

```java
counts.merge(
    word,
    1,
    Integer::sum
);
```

Conceptually:

```text
word doesn't exist
       ↓
insert 1

word exists
       ↓
oldValue + 1
```

This is extremely useful in concurrent frequency counting.

---

# 17. Why ConcurrentHashMap doesn't allow null

This is an important interview question.

```java
ConcurrentHashMap<String, String> map =
        new ConcurrentHashMap<>();

map.put(null, "Java");
```

Not allowed.

Likewise:

```java
map.put("Java", null);
```

Not allowed.

Why?

Because `null` creates ambiguity around lookup.

Suppose:

```java
map.get("Java")
```

returns:

```text
null
```

Does that mean:

```text
A. key doesn't exist
```

or:

```text
B. key exists and value is null
```

ConcurrentHashMap avoids this ambiguity by disallowing null keys and values.

This is especially useful in concurrent algorithms where a `null` result can naturally represent absence.

---

# 18. ConcurrentHashMap iteration

Another major difference from HashMap.

ConcurrentHashMap iterators are **weakly consistent**.

That means they:

- do not normally throw `ConcurrentModificationException` merely because another thread modifies the Map
- can observe some modifications occurring during iteration
- do not represent a fixed snapshot

Example:

```text
Thread A                  Thread B

iterate()
                           put("D", 4)
continue
```

The iteration can continue.

But you shouldn't assume:

> "The iterator sees exactly the Map state at one instant."

It doesn't provide that snapshot guarantee.

---

# 19. HashMap vs ConcurrentHashMap

This is worth putting directly in your notes:

| | HashMap | ConcurrentHashMap |
|---|---|---|
| Thread-safe | ❌ | ✅ |
| Concurrent reads | Not designed for concurrent mutation | ✅ |
| Concurrent writes | ❌ | ✅ |
| Null key | ✅ | ❌ |
| Null value | ✅ | ❌ |
| Iterator | Fail-fast, best effort | Weakly consistent |
| Concurrency mechanism | None | CAS + fine-grained synchronization |
| Atomic operations | Basic Map operations | `putIfAbsent`, `compute`, `merge`, etc. |

---

# 20. ConcurrentHashMap vs synchronizedMap

This is probably the **most important comparison**.

### `synchronizedMap`

```java
Map<String, User> map =
    Collections.synchronizedMap(new HashMap<>());
```

Think:

```text
               ONE LOCK
                  |
       +----------+----------+
       |          |          |
     get()       put()     remove()
```

Simple, but concurrency can be limited.

### `ConcurrentHashMap`

```java
ConcurrentHashMap<String, User>
```

Think:

```text
       ConcurrentHashMap
              |
       +------+------+------+
       |      |      |      |
    bucket  bucket bucket bucket
       |      |      |      |
       ↓      ↓      ↓      ↓
     work   work   work   work
```

with CAS and localized synchronization where necessary.

---

# 21. Production use cases

### In-memory cache

```java
ConcurrentHashMap<String, User> cache;
```

Multiple request threads can access it.

### Request deduplication

```java
ConcurrentHashMap<String, ProcessingStatus> requests;
```

### Concurrent counters

```java
ConcurrentHashMap<String, LongAdder> counters;
```

### Shared metadata

```java
ConcurrentHashMap<String, ServiceInfo> services;
```

### Lazy initialization

```java
map.computeIfAbsent(key, this::createObject);
```

---

# 22. One important warning

`ConcurrentHashMap` makes **Map operations** thread-safe.

It does NOT magically make your entire business logic thread-safe.

For example:

```java
if (map.get(id) == null) {
    doSomething();
    map.put(id, value);
}
```

can still have a race because your business operation is:

```text
get
+
doSomething
+
put
```

Use appropriate atomic Map operations or explicit synchronization depending on the business requirement.

---

# 23. What you should write in your notes

Don't write everything above. Your final notes should be:

```text id="0f3r9k"
CONCURRENT HASHMAP
──────────────────────────────

Purpose:
Thread-safe Map designed for concurrent access.

Why?
HashMap is not thread-safe.
synchronizedMap uses coarse-grained synchronization.

Internal idea:
Hash table + CAS + fine-grained synchronization.

Reads:
Designed to allow concurrent reads without a global lock.

Writes:
CAS where appropriate; synchronized coordination for
more complex bucket updates.

Important methods:
putIfAbsent()
computeIfAbsent()
compute()
computeIfPresent()
merge()
replace()

Null:
No null keys
No null values

Iterator:
Weakly consistent
Doesn't fail merely because of concurrent modification
Not a snapshot

Use cases:
• concurrent cache
• shared application state
• request deduplication
• counters
• lazy initialization

Key comparison:
HashMap        → not thread-safe
synchronizedMap → thread-safe, coarse-grained
ConcurrentHashMap → thread-safe, designed for high concurrency
```

### The one sentence to remember

> **ConcurrentHashMap provides thread-safe concurrent Map access using a combination of CAS and fine-grained synchronization rather than simply putting one global lock around the entire Map.**

That is the level of understanding I want you to carry into the EPAM interview.