## Hashtable

This is a **short topic**. Don't spend too much preparation time here; the important Map topic after this is `ConcurrentHashMap`.

### 1. What is Hashtable?

`Hashtable` is a **legacy, synchronized Map implementation**.

```java
Map<String, Integer> map = new Hashtable<>();

map.put("Java", 100);
```

Conceptually:

```text
Hashtable
   ↓
hash-based structure
   +
synchronized operations
```

It predates the modern Collections Framework (`Hashtable` is from JDK 1.0).

---

### 2. Hashtable vs HashMap

This is the main thing to remember.

| | HashMap | Hashtable |
|---|---|---|
| Thread-safe | ❌ | ✅ |
| Synchronization | None | Synchronized |
| Null key | ✅ | ❌ |
| Null value | ✅ | ❌ |
| Legacy | No | Yes |
| Performance under concurrency | — | Can be limited by synchronization |
| Recommended for new code | Yes, when appropriate | Generally no |

Example:

```java
HashMap<String, String> map = new HashMap<>();

map.put(null, "value");      // allowed
map.put("Java", null);       // allowed
```

But:

```java
Hashtable<String, String> table = new Hashtable<>();

table.put(null, "value");    // NullPointerException
table.put("Java", null);     // NullPointerException
```

---

### 3. Why is Hashtable thread-safe?

Its methods are synchronized.

Conceptually:

```java
public synchronized V put(K key, V value) {
    ...
}
```

So if multiple threads access it:

```text
Thread A ──┐
           │
Thread B ──┼──> Hashtable
           │
Thread C ──┘
```

access is coordinated through synchronization.

But there's a downside:

```text
multiple threads
       ↓
same synchronization mechanism
       ↓
more contention
       ↓
less concurrency
```

That's one reason modern applications generally don't choose Hashtable for concurrent workloads.

---

### 4. Why was Hashtable replaced?

Modern Java provides better choices depending on the requirement:

```text
Single-threaded
      ↓
HashMap


Need ordering
      ↓
LinkedHashMap


Need sorted keys
      ↓
TreeMap


Need concurrent access
      ↓
ConcurrentHashMap
```

So Hashtable is mainly important because an interviewer may ask:

> "What's the difference between HashMap and Hashtable?"

You should be able to answer it quickly.

---

# The important Map topic now: ConcurrentHashMap ⭐⭐⭐⭐⭐

This is where we should slow down again.

For your EPAM interview, I would treat `ConcurrentHashMap` as **much more important than Hashtable** because it connects directly to your Java concurrency preparation.

We'll cover it properly:

```text
ConcurrentHashMap
        ↓
Why HashMap fails with concurrency
        ↓
Why synchronizedMap has limitations
        ↓
Internal structure
        ↓
CAS
        ↓
locking during updates
        ↓
put()
        ↓
get()
        ↓
putIfAbsent()
        ↓
computeIfAbsent()
        ↓
merge()
        ↓
null restriction
        ↓
weakly consistent iterator
        ↓
size()/mappingCount()
        ↓
production use cases
        ↓
interview coding questions
```

This is the point where **Collections starts connecting directly into Concurrency**, so we'll go deep rather than treating it as another simple implementation comparison.