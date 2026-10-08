# Collections → `List`

Good. We finished the `Collection` abstraction. Now we move **one step only**: `List`.

The goal here is not to memorize “List allows duplicates.” We want to understand **why List exists, what contract it provides, how implementations achieve it, and when you'd choose each implementation.**

---

## 1. The problem `List` solves

Suppose your application receives transactions:

```java
Transaction t1;
Transaction t2;
Transaction t3;
Transaction t4;
```

You need to maintain them in the **same sequence in which they arrived**.

You also may need:

- duplicates
- positional access
- insertion at a particular position
- removal by position
- iteration in a predictable sequence

An ordinary `Set` doesn't express this requirement because uniqueness is its primary semantic.

A `Map` doesn't express it because you're not storing key → value associations.

We need an abstraction representing:

> **An ordered sequence of elements where positional access is meaningful and duplicate elements can exist.**

That's `List`.

---

# 2. What does `List` guarantee?

`List` extends `Collection`.

```text
Iterable
   ↓
Collection
   ↓
List
```

The important characteristics are:

### Ordering

If you do:

```java
List<String> users = new ArrayList<>();

users.add("Aryan");
users.add("Rahul");
users.add("Amit");
```

iteration gives:

```text
Aryan
Rahul
Amit
```

The list maintains an ordering of its elements.

### Duplicates

This is valid:

```java
users.add("Aryan");
users.add("Aryan");
```

Result:

```text
[Aryan, Aryan]
```

### Positional access

```java
users.get(1);
```

A `List` has positions/indexes.

### Positional insertion/removal

```java
users.add(1, "Raj");
users.remove(1);
```

So the `List` contract is richer than the generic `Collection` contract.

---

# 3. Important distinction: ordered vs sorted

This is an excellent interview trap.

If I say:

```java
List<Integer> numbers =
        new ArrayList<>();

numbers.add(50);
numbers.add(10);
numbers.add(30);
```

the list contains:

```text
50, 10, 30
```

It is **ordered**, but it is not **sorted**.

### Ordering

Means the elements have a defined sequence/position.

### Sorting

Means elements are arranged according to some ordering criterion:

```text
10, 30, 50
```

So:

> **List provides ordering semantics; List itself does not mean sorted.**

---

# 4. Why is `List` an interface?

Suppose your business logic says:

> "I need a sequence of transactions."

You don't necessarily care whether that sequence is implemented using:

- dynamic array
- linked nodes
- another List implementation

So you write:

```java
List<Transaction> transactions;
```

rather than:

```java
ArrayList<Transaction> transactions;
```

This gives you abstraction.

For example:

```java
void processTransactions(List<Transaction> transactions) {
    // business logic
}
```

The caller can provide:

```java
ArrayList<Transaction>
```

or:

```java
LinkedList<Transaction>
```

without changing the method contract.

---

# 5. Main List implementations

For our preparation, the important hierarchy is:

```text
                  List
                    │
       ┌────────────┼────────────┐
       │            │            │
 ArrayList      LinkedList      Vector
                                  │
                                  ↓
                                Stack
```

We'll study them in this order:

```text
List
 ↓
ArrayList
 ↓
Dynamic array implementation
 ↓
LinkedList
 ↓
Singly linked-list implementation
 ↓
Vector
 ↓
Stack
 ↓
Comparisons
```

---

# 6. ArrayList — the problem it solves

Imagine using a normal array:

```java
String[] users = new String[3];
```

You have:

```text
capacity = 3
```

After:

```java
users[0] = "Aryan";
users[1] = "Rahul";
users[2] = "Amit";
```

the array is full.

Now you want:

```java
"Raj"
```

An ordinary Java array cannot grow.

You'd have to:

1. Allocate a larger array.
2. Copy the old elements.
3. Put the new element in it.
4. Replace the old array.

That's cumbersome.

`ArrayList` solves this by providing a **dynamically resizable array abstraction**.

---

# 7. ArrayList internal model

Conceptually, think:

```text
ArrayList
   |
   ↓
Object[]
```

For:

```java
List<String> users = new ArrayList<>();

users.add("Aryan");
users.add("Rahul");
users.add("Amit");
```

the internal representation is conceptually:

```text
backing array

index
  0        1        2
┌──────┬──────┬──────┬──────┬──────┐
│Aryan │Rahul │Amit  │ null │ null │
└──────┴──────┴──────┴──────┴──────┘
              ↑
            size = 3
```

The array may have capacity greater than the number of elements.

This gives us our first important distinction.

---

# 8. Size vs Capacity

Suppose:

```text
capacity = 10
size = 3
```

The internal array conceptually looks like:

```text
[A][B][C][ ][ ][ ][ ][ ][ ][ ]
 ↑────────↑
   3 elements

 ↑─────────────────────────────↑
           capacity = 10
```

### Size

Number of elements currently stored.

```java
list.size();
```

returns:

```text
3
```

### Capacity

How many elements the current backing array can hold before it needs to grow.

This is an **internal implementation concept**, not something exposed directly by the `List` interface.

---

# 9. Why `ArrayList.get()` is O(1)

Suppose:

```java
users.get(2);
```

Because the elements are stored in an array, the implementation can access the element at index 2 directly.

Conceptually:

```text
base address
      +
index × element-size
```

So there is no need to walk:

```text
Aryan → Rahul → Amit
```

The array provides direct indexed access.

Therefore:

```text
get(index) → O(1)
```

This is one of the biggest reasons ArrayList is useful.

---

# 10. ArrayList `set()`

Suppose:

```java
users.set(1, "Raj");
```

We already know where index 1 is.

Conceptually:

```text
Before:

[Aryan][Rahul][Amit]

After:

[Aryan][Raj][Amit]
```

No shifting is required.

Therefore:

```text
set(index, value) → O(1)
```

assuming the index is valid.

---

# 11. ArrayList `add(element)`

Now:

```java
users.add("Raj");
```

If there is free capacity:

```text
[Aryan][Rahul][Amit][ ][ ]
                         ↑
                      insert here
```

The new element can be placed directly at the end.

That's approximately:

```text
O(1)
```

But there's an important problem.

---

# 12. What happens when capacity is full?

Suppose:

```text
capacity = 4
size = 4
```

```text
[A][B][C][D]
```

Now:

```java
list.add("E");
```

There isn't room.

The implementation must grow the backing array.

Conceptually:

```text
OLD

[A][B][C][D]

        ↓

allocate larger array

[A][B][C][D][ ][ ][ ]

        ↓

copy elements

[A][B][C][D][E][ ][ ]
```

So the resize operation involves copying existing elements.

If there are `n` elements:

```text
copy → O(n)
```

---

# 13. Then is `add()` O(1) or O(n)?

This is a classic interview question.

The answer is:

> **Amortized O(1).**

Why?

Most calls are:

```text
O(1)
```

Occasionally, one call causes:

```text
resize → O(n)
```

But across a large sequence of additions, the average cost per addition remains constant.

So:

```text
Normal add       → O(1)
Resize           → O(n)
Amortized add    → O(1)
```

Don't say:

> "ArrayList add is always O(1)."

That's not accurate.

---

# 14. ArrayList insertion at an index

Now consider:

```java
list.add(2, "X");
```

Suppose:

```text
[A][B][C][D][E]
```

We need:

```text
[A][B][X][C][D][E]
```

The elements after index 2 need to move:

```text
C → right
D → right
E → right
```

So the number of elements shifted can be proportional to `n`.

Therefore:

```text
add(index, element) → O(n)
```

in the general case.

---

# 15. Removing from ArrayList

Suppose:

```java
list.remove(2);
```

From:

```text
[A][B][C][D][E]
```

we want:

```text
[A][B][D][E]
```

The elements after the removed element shift left:

```text
D ←
E ←
```

Therefore:

```text
remove(index) → O(n)
```

in the general case.

Removing the last element is a special case and can be O(1).

---

# 16. Searching

Suppose:

```java
list.contains("Rahul");
```

ArrayList doesn't inherently have a hash index.

It generally searches sequentially:

```text
Aryan → Rahul → Amit → ...
```

So:

```text
contains() → O(n)
```

Likewise:

```text
indexOf() → O(n)
```

---

# 17. ArrayList complexity

Now you should understand **why**, rather than just memorizing:

| Operation | Complexity |
|---|---:|
| `get(index)` | O(1) |
| `set(index)` | O(1) |
| `add(element)` | Amortized O(1) |
| `add(index, element)` | O(n) |
| `remove(index)` | O(n) |
| `contains()` | O(n) |
| `indexOf()` | O(n) |

---

# 18. Implementing a Dynamic Array

Now let's actually implement the idea.

This is a simplified educational implementation, not a replacement for production `ArrayList`.

```java
public class MyArrayList<E> {

    private Object[] elements;
    private int size;

    public MyArrayList() {
        elements = new Object[10];
    }

    public void add(E element) {

        if (size == elements.length) {
            resize();
        }

        elements[size] = element;
        size++;
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
    public E get(int index) {

        checkIndex(index);

        return (E) elements[index];
    }

    public E set(int index, E element) {

        checkIndex(index);

        @SuppressWarnings("unchecked")
        E oldValue = (E) elements[index];

        elements[index] = element;

        return oldValue;
    }

    public int size() {
        return size;
    }

    private void checkIndex(int index) {

        if (index < 0 || index >= size) {
            throw new IndexOutOfBoundsException(
                    "Index: " + index
            );
        }
    }
}
```

---

# 19. What did we actually implement?

The important logic is:

```java
if (size == elements.length) {
    resize();
}
```

This represents the entire **dynamic array idea**.

Then:

```java
elements[size] = element;
size++;
```

stores the element.

And:

```java
Object[] newElements =
        new Object[newCapacity];
```

creates the larger storage.

Then:

```java
System.arraycopy(...)
```

copies the old elements.

This is exactly the mechanism you need to understand when an interviewer asks:

> **"How would you implement ArrayList?"**

---

# 20. Production use cases for ArrayList

ArrayList is a good candidate when:

### Use case 1 — Read-heavy indexed data

```java
List<User> users;
```

where you frequently do:

```java
users.get(index);
```

### Use case 2 — Append-heavy workload

For example:

```java
List<Transaction> transactions;
```

where you're mostly doing:

```java
transactions.add(transaction);
```

### Use case 3 — API response collections

For example:

```java
List<UserResponse> users;
```

for a REST response.

### Use case 4 — In-memory processing

For example:

```java
List<Transaction> transactions =
        fetchTransactions();

transactions.stream()
        .filter(...)
        .map(...)
        .collect(...);
```

ArrayList is often a natural choice.

---

# 21. Production consideration: ArrayList is not thread-safe

This is important.

Consider:

```java
List<Integer> list =
        new ArrayList<>();
```

and two threads:

```text
Thread A
   |
   +---- add()

Thread B
   |
   +---- add()
```

ArrayList doesn't synchronize concurrent modifications.

You therefore cannot treat it as a thread-safe shared mutable collection.

Later we'll study:

```text
Collections.synchronizedList()
CopyOnWriteArrayList
ConcurrentHashMap
BlockingQueue
```

and understand when each is appropriate.

---

# 22. Another production consideration: pre-sizing

Suppose you know you'll load approximately 1 million records.

Instead of:

```java
new ArrayList<>();
```

you can sometimes provide an initial capacity:

```java
new ArrayList<>(1_000_000);
```

Why?

It can reduce repeated growth/resizing while populating the list.

But don't blindly preallocate enormous arrays either.

You trade:

```text
fewer reallocations
```

against:

```text
more memory reserved upfront
```

This is exactly the sort of **production trade-off** we'll discuss throughout this preparation.

---

# 23. Interview questions you should now be able to answer

### Basic

**Q1. What is List?**

An ordered collection abstraction supporting positional access and allowing duplicate elements.

**Q2. Is List sorted?**

No. Ordered does not mean sorted.

**Q3. Why is ArrayList `get()` O(1)?**

Because it uses an array-like backing structure providing direct indexed access.

**Q4. Why is ArrayList `add()` amortized O(1)?**

Because most additions are constant-time, while occasional resizing costs O(n).

**Q5. Why is inserting into the middle O(n)?**

Because subsequent elements need to be shifted.

**Q6. Size vs capacity?**

Size = number of stored elements.

Capacity = current backing-array storage available before growth.

---

# 24. Senior-level follow-up

If the interviewer says:

> "I have a service processing 10 million objects. Would you always use ArrayList?"

Don't answer automatically.

You should think:

```text
What operations?
       ↓
Random access?
       ↓
Append?
       ↓
Middle insertion?
       ↓
Memory constraints?
       ↓
Concurrency?
       ↓
Expected size?
       ↓
Read/write ratio?
```

Then choose the structure based on the workload.

That's the mindset we're building.

---

# 25. Where we stop

We have now covered:

```text
List
 ↓
Problem it solves                         ✅
List contract                             ✅
Ordering vs sorting                       ✅
ArrayList                                 ✅
Internal backing array                    ✅
Size vs capacity                          ✅
get()                                     ✅
set()                                     ✅
add()                                     ✅
Resizing                                  ✅
Amortized complexity                      ✅
Insertion                                 ✅
Removal                                   ✅
Searching                                 ✅
Dynamic-array implementation              ✅
Production use cases                      ✅
Production considerations                 ✅
```

### We do NOT move to LinkedList yet.

That's deliberate.

Next topic is:

# **LinkedList**

And we'll follow exactly the same format:

**Problem → why ArrayList isn't ideal in some workloads → linked-list concept → node structure → singly linked list implementation → doubly linked list → Java's `LinkedList` → insertion/removal → traversal → complexity → production use cases → why LinkedList is often misunderstood → interview coding problems → production questions.**

Then **only after LinkedList is properly complete** will we move to Vector/Stack and finish the List section.