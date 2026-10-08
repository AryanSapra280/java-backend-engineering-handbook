# 3. Vector

We're continuing the **List** flow in order. Vector is older than ArrayList, but it matters because interviewers often use it to test whether you understand **synchronization vs thread safety vs performance**.

---

## 1. Problem — Why did Vector exist?

Before the modern Collections Framework, Java needed a dynamically growing array structure.

A normal Java array has fixed size:

```java
int[] numbers = new int[10];
```

You can't simply add an 11th element.

So Java introduced **Vector** as a dynamically resizable array.

Conceptually:

```text
Fixed array:

[ A ][ B ][ C ][ D ]
```

If it becomes full, Vector creates a larger array:

```text
Old:
[ A ][ B ][ C ][ D ]

New:
[ A ][ B ][ C ][ D ][ E ][ _ ][ _ ][ _ ]
```

So the fundamental problem Vector solves is the same basic problem as ArrayList:

> **How do we maintain a dynamically growing array?**

---

# 2. Why does Vector still matter?

Because Vector has something ArrayList traditionally didn't:

> **Its methods are synchronized.**

For example:

```java
Vector<Integer> numbers = new Vector<>();
```

Operations such as:

```java
numbers.add(10);
numbers.get(0);
numbers.remove(0);
```

are synchronized at the method level.

Conceptually:

```java
public synchronized boolean add(E e) {
    ...
}
```

So Vector was designed for use where access to the collection needed synchronization.

---

# 3. Concept — Vector is a dynamic array

The mental model is:

```text
Vector
  ↓
dynamic array
  ↓
Object[]
```

Very similar to ArrayList.

For example:

```java
Vector<Integer> numbers =
        new Vector<>();

numbers.add(10);
numbers.add(20);
numbers.add(30);
```

Internally conceptually:

```text
elementData
   ↓
[10][20][30][_][_][_]
```

And it maintains something equivalent to:

```text
size
capacity
```

So:

```text
size = number of actual elements

capacity = size of backing array
```

---

# 4. Internal working

Suppose:

```java
Vector<Integer> v = new Vector<>(3);
```

Initially:

```text
capacity = 3
size = 0
```

After:

```java
v.add(10);
v.add(20);
v.add(30);
```

we have:

```text
capacity = 3
size = 3

[10][20][30]
```

Now:

```java
v.add(40);
```

There isn't enough capacity.

Vector must resize.

Conceptually:

```text
Old array:

[10][20][30]

        ↓ resize

New array:

[10][20][30][40][_][_]
```

The existing elements are copied into the new backing array.

---

# 5. Vector growth policy

This is a good interview follow-up.

Vector has historically supported a **capacity increment** mechanism.

You can construct it like:

```java
Vector<Integer> v =
        new Vector<>(10, 5);
```

Meaning:

```text
initial capacity = 10
capacity increment = 5
```

When expansion is needed, capacity can increase by that increment.

If capacity increment isn't specified, Vector has a growth behavior that increases capacity substantially (historically doubling).

The exact implementation details can vary by JDK, so don't make an interview answer depend on a hard-coded growth factor unless the interviewer specifically asks about a particular JDK.

The important concept is:

> **Vector dynamically expands its backing array when capacity is exhausted.**

---

# 6. Why is Vector thread-safe?

Because its individual methods are synchronized.

Conceptually:

```java
public synchronized boolean add(E e) {
    ...
}
```

and:

```java
public synchronized E get(int index) {
    ...
}
```

Therefore two threads cannot simultaneously execute two synchronized instance methods on the **same Vector object**.

Imagine:

```text
Thread A                 Thread B

add(10)                  add(20)
   │                         │
   └── lock Vector ──────────┘
```

Only one can acquire the object's monitor at a time for those synchronized methods.

---

# 7. But here's the important trap

A thread-safe collection does **not automatically make a multi-operation business operation atomic**.

For example:

```java
if (!vector.contains(value)) {
    vector.add(value);
}
```

You might think:

> "Vector is synchronized, so this is thread-safe."

Not necessarily.

Why?

Because these are two separate operations:

```text
contains()
   ↓
add()
```

Another thread can execute between them.

Example:

```text
Thread A                  Thread B

contains(X) → false

                          contains(X) → false

add(X)

                          add(X)
```

Now X exists twice.

This is a classic interview concept:

> **Thread-safe individual methods ≠ thread-safe compound operation.**

If multiple operations must behave atomically, you need external synchronization or another concurrency design.

---

# 8. Vector vs ArrayList

This is one of the most likely interview questions.

| Feature | ArrayList | Vector |
|---|---|---|
| Internal structure | Dynamic array | Dynamic array |
| Random access | O(1) | O(1) |
| Add at end | Amortized O(1) | Amortized O(1) |
| Insert middle | O(n) | O(n) |
| Thread-safe individual methods | No | Yes, synchronized |
| Synchronization overhead | Lower | Higher |
| Modern default | Yes | Generally no |

The big difference:

```text
ArrayList
    ↓
not synchronized by default

Vector
    ↓
synchronized methods
```

---

# 9. Why isn't Vector automatically the best choice for multithreading?

This is where senior-level understanding matters.

Suppose:

```text
Thread 1
Thread 2
Thread 3
Thread 4
```

all frequently access the same Vector.

Every synchronized operation needs to acquire the Vector's monitor.

Conceptually:

```text
Thread 1 ──┐
Thread 2 ──┤
Thread 3 ──┼──> Vector lock
Thread 4 ──┘
```

This can create contention.

So synchronization provides safety, but it can also reduce concurrency.

Modern concurrent applications often choose a collection based on the actual access pattern instead of simply saying:

> "Multiple threads → Vector."

For example, later we'll study:

```text
ConcurrentHashMap
CopyOnWriteArrayList
BlockingQueue
```

Each solves different concurrency problems.

---

# 10. Vector vs `Collections.synchronizedList()`

This is a good interviewer question.

You can create:

```java
List<Integer> list =
        Collections.synchronizedList(
                new ArrayList<>()
        );
```

Now the underlying ArrayList is wrapped with synchronization.

Conceptually:

```text
ArrayList
    ↓
synchronized wrapper
    ↓
thread-safe List interface
```

Vector, on the other hand, is itself an older synchronized collection implementation.

So:

```java
Vector<Integer> vector =
        new Vector<>();
```

versus:

```java
List<Integer> list =
        Collections.synchronizedList(
                new ArrayList<>()
        );
```

Both provide synchronized access to individual collection operations.

But the second approach lets you choose the underlying List implementation.

---

# 11. Important: Iteration still needs care

Consider:

```java
List<Integer> list =
        Collections.synchronizedList(
                new ArrayList<>()
        );
```

This does **not** mean you can blindly iterate without considering synchronization.

For example:

```java
synchronized (list) {
    for (Integer value : list) {
        System.out.println(value);
    }
}
```

The synchronization wrapper's documentation requires external synchronization during iteration to prevent concurrent modification races.

This is another excellent interview distinction:

> Synchronizing collection methods doesn't necessarily make an entire iteration or multi-step operation atomic.

---

# 12. Can we implement a Vector-like structure?

Yes.

A simplified version:

```java
class MyVector<E> {

    private Object[] elements;
    private int size;

    public MyVector(int capacity) {
        elements = new Object[capacity];
    }

    public synchronized void add(E element) {

        if (size == elements.length) {
            resize();
        }

        elements[size++] = element;
    }

    private void resize() {

        int newCapacity = elements.length * 2;

        Object[] newElements =
                new Object[newCapacity];

        System.arraycopy(
                elements,
                0,
                newElements,
                0,
                size
        );

        elements = newElements;
    }

    @SuppressWarnings("unchecked")
    public synchronized E get(int index) {

        if (index < 0 || index >= size) {
            throw new IndexOutOfBoundsException();
        }

        return (E) elements[index];
    }

    public synchronized int size() {
        return size;
    }
}
```

The important part is:

```java
synchronized
```

combined with:

```text
dynamic array
+
resize
+
size tracking
```

That's essentially the conceptual foundation.

---

# 13. Complexity

| Operation | Complexity |
|---|---:|
| `get(index)` | O(1) |
| `set(index)` | O(1) |
| `add(element)` | Amortized O(1) |
| `add(index, element)` | O(n) |
| `remove(index)` | O(n) |
| `contains()` | O(n) |
| Resize | O(n) |

Synchronization doesn't change the fundamental Big-O complexity, but it adds locking/coordination overhead.

---

# 14. Production considerations

### Don't use Vector just because it's thread-safe

Instead ask:

> What concurrency behavior do I actually need?

For example:

### Single-threaded list

```java
ArrayList
```

### Need synchronized List wrapper

```java
Collections.synchronizedList(...)
```

### Many reads, very few writes

Potentially:

```java
CopyOnWriteArrayList
```

### Concurrent key/value access

Potentially:

```java
ConcurrentHashMap
```

### Producer/consumer workload

Potentially:

```java
BlockingQueue
```

We'll cover these properly later.

---

# 15. Interview trap: Vector vs synchronizedList

Interviewer:

> "What's the difference between Vector and Collections.synchronizedList?"

Good answer:

> "Vector is a legacy synchronized List implementation with synchronization built into its methods. `Collections.synchronizedList()` is a wrapper that synchronizes access to an underlying List implementation such as ArrayList. Both provide synchronized individual operations, but compound operations and iteration still require appropriate external synchronization."

That's much better than:

> "Both are thread-safe."

---

# 16. Interview trap: Is Vector completely thread-safe?

Answer:

> "Its individual methods are synchronized, but that doesn't make arbitrary sequences of operations atomic. If multiple operations must form one atomic business operation, additional synchronization or a different concurrency mechanism may be required."

Example:

```java
if (!vector.contains(x)) {
    vector.add(x);
}
```

is not automatically atomic.

---

# 17. Vector vs ArrayList — interviewer follow-up

**Interviewer:**

> "If ArrayList is faster and Vector is synchronized, why not synchronize ArrayList yourself?"

You could say:

```java
Collections.synchronizedList(
    new ArrayList<>()
);
```

This is one option.

But the deeper answer is:

> The correct choice depends on the concurrency pattern. A synchronized wrapper uses one lock and can become a contention point. For high-concurrency workloads, specialized concurrent collections may provide better scalability.

This leads naturally into the collections we'll study later.

---

# 18. One more important historical point

Vector is considered a **legacy collection**.

That doesn't mean:

```text
Vector = broken
```

It means newer APIs generally provide better choices depending on requirements.

You may still encounter Vector in:

- older enterprise applications
- legacy Java codebases
- interview questions
- APIs maintained for backward compatibility

So you absolutely should understand it for an interview, but you shouldn't automatically reach for it in new code.

---

# 19. Mental model

Remember Vector like this:

```text
                  Vector
                    │
             Dynamic Array
                    │
          ┌─────────┴─────────┐
          │                   │
      Object[]            size/capacity
          │
       resize
          │
     synchronization
          │
   synchronized methods
```

And compare:

```text
ArrayList
   ↓
Dynamic array
   ↓
No synchronization

Vector
   ↓
Dynamic array
   ↓
Synchronized methods
   ↓
Legacy
```

---

# 20. Questions you should now be able to answer

1. Why was Vector introduced?
2. What is the internal data structure of Vector?
3. How does Vector resize?
4. What is the difference between size and capacity?
5. Why is Vector thread-safe?
6. Does synchronization make compound operations atomic?
7. Vector vs ArrayList?
8. Vector vs `Collections.synchronizedList()`?
9. Why can synchronization hurt performance?
10. Why would you generally avoid Vector in new code?
11. Implement a simplified Vector.
12. Is iterating over a synchronized collection automatically thread-safe?

---

## Collections progress

```text
Collection
   │
   └── List
       ├── ArrayList     ✅
       ├── LinkedList    ✅
       ├── Vector        ✅
       └── Stack        ← NEXT
```

**Next: `Stack`** — and this one is particularly important because we'll uncover why Java's legacy `Stack` isn't generally the preferred way to implement a stack, and why **Deque/ArrayDeque** is usually the modern choice.