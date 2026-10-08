# List Consolidation — Part 1: ArrayList vs LinkedList

We have finished the four List implementations. Now we need to make sure you can **reason about them in an interview**, not just recall definitions.

The List section in the roadmap specifically includes implementation comparisons and coding-oriented questions. Pasted markdown

---

# 1. The real problem: How do I choose a List?

Suppose an interviewer gives you:

> "I need to store 10 million objects. Which List will you use?"

There is **no correct answer yet**.

You first ask:

```text
How will the data be accessed?
How often is it inserted?
Where are elements inserted?
How often are elements removed?
Do I need random access?
Is it shared between threads?
Do I need ordering?
```

Then choose the implementation.

That's the production mindset.

---

# 2. ArrayList vs LinkedList

Let's start from their internal structures.

### ArrayList

```text
Object[]


index
  0     1     2     3
┌───┬───┬───┬───┐
│ A │ B │ C │ D │
└───┴───┴───┴───┘
```

### LinkedList

```text
head
 ↓
[A] ⇄ [B] ⇄ [C] ⇄ [D]
                         ↑
                        tail
```

This immediately explains most of the performance differences.

---

# 3. `get(index)`

### ArrayList

```java
list.get(500000);
```

ArrayList can directly access the array position.

Therefore:

```text
O(1)
```

### LinkedList

It has to traverse:

```text
A → B → C → ... → target
```

Therefore:

```text
O(n)
```

Java optimizes the direction:

```text
if index < size/2
    start from first
else
    start from last
```

but it remains O(n).

---

# 4. Adding at the end

### ArrayList

Normally:

```java
list.add(value);
```

is:

```text
O(1) amortized
```

Occasionally resizing requires:

```text
O(n)
```

because elements have to be copied.

### LinkedList

With a tail reference:

```text
tail → newNode
tail = newNode
```

So:

```text
O(1)
```

---

# 5. Inserting in the middle

Suppose:

```text
[A][B][C][D]
```

Insert X at index 2.

### ArrayList

Must shift:

```text
[A][B][C][D]
       ↓
[A][B][X][C][D]
```

C and D need to move.

```text
O(n)
```

### LinkedList

Once the correct node is located:

```text
B ⇄ C
```

becomes:

```text
B ⇄ X ⇄ C
```

Pointer changes are:

```text
O(1)
```

But finding index 2 requires traversal:

```text
O(n)
```

Therefore:

```text
LinkedList.add(index, value)
= O(n) overall
```

This is one of the most important interview nuances.

---

# 6. So is LinkedList actually better for insertion?

Not automatically.

Suppose you say:

> "LinkedList is better because insertion is O(1)."

Interviewer:

> "How do you find the insertion point?"

You:

> "...O(n)."

Exactly.

That's why the statement:

> **"LinkedList has O(1) insertion."**

is incomplete.

The accurate statement is:

> **Once the target node is known, insertion/removal can be O(1). Finding the node by index is O(n).**

---

# 7. Memory

### ArrayList

Conceptually:

```text
Object[]
 ├── reference A
 ├── reference B
 ├── reference C
 └── reference D
```

### LinkedList

Each node contains:

```text
prev
data
next
```

So every element requires additional node/reference overhead.

Therefore LinkedList generally consumes more memory per element.

---

# 8. CPU cache locality

This is a **senior-level follow-up**.

ArrayList stores references in a contiguous backing array:

```text
[A][B][C][D][E][F]
```

This tends to provide good memory locality.

LinkedList nodes can be scattered across the heap:

```text
Node A        Node C
     Node B             Node D
```

Following:

```text
A → B → C → D
```

requires pointer chasing.

So even when Big-O looks similar, ArrayList can perform better in practice because of memory locality and lower object overhead.

---

# 9. Decision table

| Requirement | Better default |
|---|---|
| Frequent `get(index)` | ArrayList |
| Append-heavy | ArrayList |
| Memory efficiency | ArrayList |
| Cache-friendly sequential traversal | ArrayList |
| Frequent operations at beginning/end | LinkedList can work |
| Need Deque behavior | Prefer ArrayDeque |
| Already have node/reference and frequently unlink/relink | LinkedList can be useful |
| General-purpose List | ArrayList |

Notice the last two words:

> **Better default**

We're not saying one implementation is universally superior.

---

# 10. Interview question: "When would you use LinkedList?"

A strong answer:

> "I'd use LinkedList when its node-based structure actually matches the workload, such as operations at the ends or operations where I already have the relevant node. I wouldn't choose it simply because insertion is theoretically O(1), because index-based insertion still requires O(n) traversal, and LinkedList also has higher memory overhead and poorer locality."

That's the answer I want you to remember.

---

# 11. ArrayList vs Vector

Now:

```text
ArrayList
   ↓
dynamic array

Vector
   ↓
dynamic array
   +
synchronized methods
```

The underlying data structure is broadly similar.

The major difference is synchronization.

| | ArrayList | Vector |
|---|---|---|
| Dynamic array | Yes | Yes |
| Random access | O(1) | O(1) |
| Amortized append | O(1) | O(1) |
| Synchronized methods | No | Yes |
| Legacy | No | Yes |
| Typical modern choice | Yes | Usually no |

---

# 12. Why not just use Vector for concurrent code?

Because:

```text
thread safety ≠ scalability
```

Suppose 20 threads frequently access the same Vector.

They contend on the object's monitor:

```text
T1 ─┐
T2 ─┤
T3 ─┤
T4 ─┼──> Vector lock
... │
T20─┘
```

So operations are serialized around the lock.

Modern Java gives specialized concurrent collections depending on the workload.

We'll later cover:

```text
CopyOnWriteArrayList
ConcurrentHashMap
BlockingQueue
```

---

# 13. Vector vs synchronizedList

You can do:

```java
List<Integer> list =
    Collections.synchronizedList(
        new ArrayList<>()
    );
```

This gives synchronized access through a wrapper.

The conceptual difference:

```text
Vector
    ↓
synchronized collection implementation

synchronizedList
    ↓
wrapper
    ↓
underlying List
```

The wrapper approach lets you select the underlying implementation.

But remember:

```java
if (!list.contains(x)) {
    list.add(x);
}
```

is still a compound operation.

Two synchronized method calls don't magically become one atomic transaction.

---

# 14. ArrayList vs CopyOnWriteArrayList

This is an important production question.

Imagine:

```text
100 threads reading
1 thread occasionally writing
```

A normal ArrayList isn't safe for concurrent modification.

One option is:

```java
CopyOnWriteArrayList<T>
```

Its fundamental strategy is:

> **Readers operate on the current array; writes create a new copy.**

Conceptually:

```text
Current:

[A][B][C]


write D


New:

[A][B][C][D]
```

The old array can continue being used by readers that already have a reference to it.

This makes reads very attractive under read-heavy workloads.

But writes are expensive because the array has to be copied.

Therefore:

```text
Many reads
Few writes
      ↓
CopyOnWriteArrayList can be useful
```

while:

```text
Many writes
      ↓
CopyOnWriteArrayList can become very expensive
```

We'll revisit this properly when we reach concurrency.

---

# 15. Coding Problem #1 — Implement a Dynamic Array

You've already seen the simplified implementation. Now let's understand the design as an interviewer would.

### Problem

Implement something like:

```java
MyArrayList<Integer> list = new MyArrayList<>();

list.add(10);
list.add(20);

list.get(1);
```

The challenge:

> How do we grow a fixed-size array dynamically?

---

## Step 1 — Internal state

We need:

```java
private Object[] elements;
private int size;
```

Why two variables?

Because:

```text
capacity != size
```

Example:

```text
capacity = 10
size = 3
```

means:

```text
[A][B][C][_][_][_][_][_][_][_]
```

---

## Step 2 — Add

```java
public void add(E element) {

    if (size == elements.length) {
        resize();
    }

    elements[size] = element;
    size++;
}
```

---

## Step 3 — Resize

```java
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
```

The critical operation is:

```java
System.arraycopy(...)
```

because the old elements must be moved to the new backing array.

---

## Step 4 — Get

```java
@SuppressWarnings("unchecked")
public E get(int index) {

    if (index < 0 || index >= size) {
        throw new IndexOutOfBoundsException();
    }

    return (E) elements[index];
}
```

Complexity:

```text
get = O(1)
```

---

# 16. Coding Problem #2 — Implement Singly LinkedList

Now let's implement the fundamental structure.

```java
class Node<E> {

    E data;
    Node<E> next;

    Node(E data) {
        this.data = data;
    }
}
```

Then:

```java
class MyLinkedList<E> {

    private Node<E> head;
    private int size;

    public void add(E value) {

        Node<E> newNode = new Node<>(value);

        if (head == null) {
            head = newNode;
        } else {

            Node<E> current = head;

            while (current.next != null) {
                current = current.next;
            }

            current.next = newNode;
        }

        size++;
    }
}
```

But we already identified the weakness:

```text
add()
 ↓
traverse to end
 ↓
O(n)
```

So production implementation maintains:

```java
private Node<E> head;
private Node<E> tail;
```

Then:

```java
public void add(E value) {

    Node<E> node = new Node<>(value);

    if (head == null) {
        head = tail = node;
    } else {
        tail.next = node;
        tail = node;
    }

    size++;
}
```

Now append:

```text
O(1)
```

---

# 17. Coding Problem #3 — Reverse LinkedList

This is extremely common.

Given:

```text
1 → 2 → 3 → 4 → null
```

produce:

```text
4 → 3 → 2 → 1 → null
```

The key problem:

> If we change `current.next`, we could lose the rest of the list.

So we need three references:

```java
Node prev = null;
Node current = head;
Node next;
```

Then:

```java
while (current != null) {

    next = current.next;

    current.next = prev;

    prev = current;

    current = next;
}
```

Finally:

```java
head = prev;
```

---

## Let's trace it

Initially:

```text
prev = null

current
   ↓
   1 → 2 → 3 → 4
```

Save:

```java
next = current.next;
```

So:

```text
next → 2
```

Reverse:

```java
current.next = prev;
```

Now:

```text
1 → null
```

Move:

```java
prev = current;
current = next;
```

Now:

```text
prev
 ↓
1

current
 ↓
2 → 3 → 4
```

Repeat.

Eventually:

```text
prev
 ↓
4 → 3 → 2 → 1 → null

current = null
```

Then:

```java
head = prev;
```

Done.

### Complexity

```text
Time  = O(n)
Space = O(1)
```

This is a beautiful example of why understanding references matters more than memorizing the code.

---

# 18. Coding Problem #4 — Find Middle Node

Given:

```text
1 → 2 → 3 → 4 → 5
```

return:

```text
3
```

Could we first count the nodes?

Yes:

```text
count = n
middle = n / 2
```

Then traverse again.

That works:

```text
Time = O(n)
Space = O(1)
```

But we can do it in one traversal using:

### Slow and Fast pointers

```java
Node slow = head;
Node fast = head;

while (fast != null &&
       fast.next != null) {

    slow = slow.next;
    fast = fast.next.next;
}
```

When:

```text
fast
```

reaches the end:

```text
slow
```

is around the middle.

Example:

```text
1 → 2 → 3 → 4 → 5
s
f
```

After one iteration:

```text
1 → 2 → 3 → 4 → 5
    s
        f
```

After two:

```text
1 → 2 → 3 → 4 → 5
        s
                f
```

So:

```text
slow = 3
```

Complexity:

```text
O(n) time
O(1) space
```

---

# 19. Coding Problem #5 — Detect Cycle

Consider:

```text
1 → 2 → 3 → 4
        ↑     |
        └─────┘
```

There is a cycle.

A naive approach could use:

```java
Set<Node> visited
```

and detect whether we've already seen a node.

That gives:

```text
O(n) space
```

But there's a better solution.

Again:

### Slow + Fast

```java
Node slow = head;
Node fast = head;

while (fast != null &&
       fast.next != null) {

    slow = slow.next;
    fast = fast.next.next;

    if (slow == fast) {
        return true;
    }
}

return false;
```

Why does this work?

If a cycle exists:

```text
slow → one step
fast → two steps
```

Eventually fast catches slow inside the cycle.

This is **Floyd's cycle detection algorithm**.

Complexity:

```text
Time  = O(n)
Space = O(1)
```

---

# 20. Coding Problem #6 — Remove Nth Node From End

Example:

```text
1 → 2 → 3 → 4 → 5
```

Remove:

```text
2nd from end
```

Result:

```text
1 → 2 → 3 → 5
```

A common mistake is to calculate length first.

You can solve it in one pass using two pointers.

Create:

```java
Node dummy = new Node(0);
dummy.next = head;

Node first = dummy;
Node second = dummy;
```

Move `first` ahead by `n + 1`:

```java
for (int i = 0; i <= n; i++) {
    first = first.next;
}
```

Then:

```java
while (first != null) {
    first = first.next;
    second = second.next;
}
```

Now:

```text
second
   ↓
node BEFORE the node to remove
```

So:

```java
second.next = second.next.next;
```

Finally:

```java
head = dummy.next;
```

Why the dummy node?

It handles the case where the head itself must be removed.

Again:

```text
Time  = O(n)
Space = O(1)
```

---

# 21. Coding Problem #7 — Implement Stack Using Deque

This one should become automatic for you.

```java
Deque<Integer> stack = new ArrayDeque<>();
```

Push:

```java
stack.push(10);
```

Pop:

```java
stack.pop();
```

Peek:

```java
stack.peek();
```

That's enough.

Conceptually:

```text
push → addFirst
pop  → removeFirst
peek → peekFirst
```

So:

```text
Deque
 ↓
ArrayDeque
 ↓
Stack behavior
```

---

# 22. The concurrency question

Now suppose:

```java
List<Integer> list =
    new ArrayList<>();
```

and:

```text
Thread A → add()
Thread B → remove()
Thread C → iterate()
```

Can we safely do that?

**No, not simply because it's a List.**

ArrayList isn't designed for concurrent structural modification.

Possible choices depend on the workload:

```text
ArrayList
    ↓
single-threaded

synchronizedList
    ↓
simple shared List + locking

CopyOnWriteArrayList
    ↓
many readers + very few writers
```

And later:

```text
Concurrent collections
```

will give us more specialized options.

---

# 23. Very important: `ConcurrentModificationException`

Suppose:

```java
List<Integer> list =
    new ArrayList<>();

list.add(1);
list.add(2);
list.add(3);

for (Integer x : list) {

    if (x == 2) {
        list.remove(x);
    }
}
```

This can result in:

```text
ConcurrentModificationException
```

The confusing part is:

> "I'm not using multiple threads. Why is it called ConcurrentModificationException?"

Because "concurrent" here can mean the collection is structurally modified **while an iterator is active**, not necessarily that two OS threads are involved.

---

# 24. Correct way to remove during iteration

Use the Iterator:

```java
Iterator<Integer> iterator =
    list.iterator();

while (iterator.hasNext()) {

    Integer value = iterator.next();

    if (value == 2) {
        iterator.remove();
    }
}
```

The iterator's `remove()` is designed for this operation.

Alternatively:

```java
list.removeIf(x -> x == 2);
```

for suitable cases.

---

# 25. Fail-fast — important interview concept

ArrayList's iterator is generally **fail-fast**.

It tracks structural modifications through an internal modification count.

Conceptually:

```text
iterator created
      ↓
expected modification count = X

collection changes
      ↓
actual modification count = X + 1

iterator detects mismatch
      ↓
ConcurrentModificationException
```

This is primarily a bug-detection mechanism.

It is **not a concurrency guarantee**.

Don't say:

> "Fail-fast makes ArrayList thread-safe."

It doesn't.

---

# 26. Final List decision tree

When the interviewer asks:

> "Which List?"

Think:

```text
                 Need List?
                     │
                     ↓
             Need index access?
                /          \
              YES           NO
               │             │
               ↓             ↓
          ArrayList      Need deque?
                           /     \
                         YES      NO
                          │        │
                          ↓        ↓
                     ArrayDeque   Analyze
```

And if shared between threads:

```text
Shared?
  │
  ├── No → ArrayList / ArrayDeque
  │
  └── Yes
       │
       ├── Simple synchronized access
       │      → synchronizedList
       │
       ├── Many reads, rare writes
       │      → CopyOnWriteArrayList
       │
       └── Different concurrency pattern
              → specialized concurrent collection
```

---

# List section — DONE ✅

Your mental model should now be:

```text
LIST
 │
 ├── ArrayList
 │     └── dynamic array
 │         └── fast random access
 │
 ├── LinkedList
 │     └── doubly linked nodes
 │         └── O(1) link manipulation
 │             but O(n) index traversal
 │
 ├── Vector
 │     └── dynamic array
 │         └── synchronized legacy collection
 │
 └── Stack
       └── LIFO
           └── legacy
               └── prefer Deque/ArrayDeque
```

And the coding patterns you've now covered are:

```text
Dynamic Array
Linked List
Reverse Linked List
Middle of Linked List
Cycle Detection
Remove Nth Node
Stack using Deque
Iterator removal
```

### Next: `Set`

We'll now move to:

```text
Collection
   │
   ├── List ✅
   │
   ├── Set  ← NEXT
   │
   └── Queue
```

And **Set is where `equals()` + `hashCode()` becomes absolutely critical**. We'll start with the problem Set solves, then `HashSet` internally—buckets, hashing, collisions, equality, duplicate detection, and finally the production implications.