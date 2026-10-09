# Collections → List → LinkedList

We finished **ArrayList**, so we continue exactly in sequence with **LinkedList**. The key is not memorizing methods—we want to understand *why LinkedList exists and when its internal structure actually helps.*

Pasted markdown

---

## 2. LinkedList

### 1. Problem — Why do we need LinkedList?

Suppose we have:

```text
[A] [B] [C] [D] [E]
```

This is an `ArrayList`.

Now imagine inserting `X` at index `2`:

```text
[A] [B] [X] [C] [D] [E]
```

With an ArrayList, elements after index `2` have to move:

```text
C → D
D → E
E → new position
```

So insertion in the middle costs **O(n)**.

What if our application performs **lots of insertions/removals**?

We want a structure where we don't have to shift all the following elements.

That's the problem LinkedList tries to solve.

---

# 2. Solution — Linked nodes

Instead of storing elements next to each other in an array:

```text
[A][B][C][D]
```

we store them as separate nodes connected to each other:

```text
[A | next] → [B | next] → [C | next] → [D | null]
```

Each node contains:

```text
data
+
reference to another node
```

Conceptually:

```java
class Node<E> {
    E item;
    Node<E> next;
}
```

So if we insert `X` between B and C:

Before:

```text
A → B → C → D
```

After:

```text
A → B → X → C → D
```

We only need to change references:

```text
B.next = X
X.next = C
```

We don't move C and D.

That's the fundamental idea.

---

# 3. Concept — What is LinkedList?

Java's:

```java
LinkedList<E>
```

implements:

```text
List
Deque
Queue
```

So it can behave both as a List and as a Deque/Queue.

The important implementation detail is that Java's `LinkedList` is a **doubly linked list**.

Conceptually:

```text
null
  ↑
prev
  |
[A] ⇄ [B] ⇄ [C] ⇄ [D]
                   |
                  next
                   ↓
                  null
```

Each node has:

```text
previous node
current value
next node
```

---

# 4. Internal Working

Conceptually, Java's node looks like:

```java
private static class Node<E> {
    E item;
    Node<E> next;
    Node<E> prev;

    Node(Node<E> prev, E element, Node<E> next) {
        this.item = element;
        this.next = next;
        this.prev = prev;
    }
}
```

And the LinkedList maintains references roughly like:

```text
first
  ↓
[A] ⇄ [B] ⇄ [C] ⇄ [D]
                         ↑
                        last
```

So instead of maintaining:

```java
Object[] elements;
```

like ArrayList, it maintains links between nodes.

---

# 5. Why doubly linked?

You could have a singly linked list:

```text
A → B → C → D
```

To go from C back to B, you can't directly do it.

With a doubly linked list:

```text
A ⇄ B ⇄ C ⇄ D
```

you can traverse both directions.

This is especially useful when removing a node.

Suppose:

```text
A ⇄ B ⇄ C ⇄ D
```

Remove C.

We can reconnect:

```text
B.next = D
D.prev = B
```

Result:

```text
A ⇄ B ⇄ D
```

No shifting.

---

# 6. The most important interview point: insertion is NOT automatically O(1)

This is where interviewers often catch people.

Someone may say:

> "LinkedList insertion is O(1)."

That's incomplete.

Consider:

```java
list.add(500000, value);
```

First Java has to **find the node at index 500000**.

That traversal costs:

```text
O(n)
```

Once the correct node has been located, actually linking the new node is:

```text
O(1)
```

So:

### If you already have the node/reference:

```text
Insertion = O(1)
```

### If you only have an index:

```text
Finding position = O(n)
Insertion itself = O(1)

Total = O(n)
```

This distinction is **very important**.

---

# 7. How does `get(index)` work?

ArrayList:

```java
list.get(500000);
```

can directly jump to the array position:

```text
base_address + index
```

Therefore:

```text
O(1)
```

LinkedList cannot do that.

It has to walk through nodes:

```text
A → B → C → D → E → ...
```

For:

```java
list.get(500000);
```

it traverses until it reaches that node.

Therefore:

```text
O(n)
```

But Java's LinkedList has an optimization.

Because it is doubly linked, it can decide whether to start from:

```text
first
```

or:

```text
last
```

For example, if you're requesting an index near the end:

```text
index = 999,999
size  = 1,000,000
```

it's much better to start from `last`.

So conceptually:

```text
if index < size / 2
    traverse from first
else
    traverse from last
```

Still asymptotically:

```text
O(n)
```

but potentially much less traversal in practice.

---

# 8. Let's implement our own LinkedList

This is worth knowing for an interview.

### Node

```java
class Node<E> {

    E data;
    Node<E> next;

    Node(E data) {
        this.data = data;
    }
}
```

### Basic singly linked list

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

    public int size() {
        return size;
    }
}
```

The structure becomes:

```text
head
 ↓
[A] → [B] → [C] → null
```

---

# 9. But this implementation has a problem

Look at:

```java
public void add(E value)
```

Every time we add:

```text
A
```

we traverse 0 nodes.

Add:

```text
B
```

traverse 1 node.

Add:

```text
C
```

traverse 2 nodes.

Add:

```text
D
```

traverse 3 nodes.

So repeatedly appending becomes expensive.

We can solve this by maintaining a `tail`.

```text
head                         tail
 ↓                             ↓
[A] → [B] → [C] → [D] → null
```

Now:

```java
tail.next = newNode;
tail = newNode;
```

No traversal required.

---

# 10. Better implementation

```java
class MyLinkedList<E> {

    private Node<E> head;
    private Node<E> tail;
    private int size;

    public void add(E value) {

        Node<E> newNode = new Node<>(value);

        if (head == null) {
            head = tail = newNode;
        } else {
            tail.next = newNode;
            tail = newNode;
        }

        size++;
    }

    public int size() {
        return size;
    }
}
```

Now:

```java
list.add("A");
list.add("B");
list.add("C");
```

works as:

```text
head
 ↓
[A] → [B] → [C]
               ↑
              tail
```

Appending is:

```text
O(1)
```

---

# 11. Doubly Linked List

Now let's model what Java's LinkedList fundamentally uses.

```java
class Node<E> {

    E data;
    Node<E> prev;
    Node<E> next;

    Node(Node<E> prev, E data, Node<E> next) {
        this.prev = prev;
        this.data = data;
        this.next = next;
    }
}
```

Suppose:

```text
A ⇄ B ⇄ C
```

Insert X between B and C.

We need:

```text
B.next = X
X.prev = B

X.next = C
C.prev = X
```

Result:

```text
A ⇄ B ⇄ X ⇄ C
```

Notice something important:

**We don't move B or C.**

We're only changing references.

---

# 12. Complexity

| Operation | ArrayList | LinkedList |
|---|---:|---:|
| `get(index)` | O(1) | O(n) |
| `set(index)` | O(1) | O(n) |
| add at end | Amortized O(1) | O(1) |
| add at beginning | O(n) | O(1) |
| remove at end | O(1) | O(1) |
| remove at beginning | O(n) | O(1) |
| insert middle by index | O(n) | O(n)* |
| remove middle by index | O(n) | O(n)* |
| search | O(n) | O(n) |

`*` For LinkedList, the actual pointer modification can be O(1), but locating the index/node costs O(n).

This distinction is crucial.

---

# 13. Production Use Cases

LinkedList can make sense when the workload is dominated by **sequential insertion/removal at the ends** or when using it through the `Deque` interface.

For example:

```java
Deque<Task> queue = new LinkedList<>();
```

You can do:

```java
queue.addFirst(task);
queue.addLast(task);

queue.removeFirst();
queue.removeLast();
```

This gives you deque semantics.

However, don't conclude:

> "Whenever there are lots of insertions, use LinkedList."

That's too simplistic.

If you're frequently doing:

```java
get(index)
```

LinkedList is generally a poor fit because traversal is required.

And in real Java applications, alternatives such as `ArrayDeque` are often preferable when you need a deque rather than a general-purpose List.

We'll cover `ArrayDeque` later in the Queue/Deque section.

---

# 14. Memory trade-off

This is another production-level point.

ArrayList essentially stores:

```text
[element][element][element][element]
```

LinkedList stores nodes containing references:

```text
[prev | data | next]
[prev | data | next]
[prev | data | next]
```

Therefore LinkedList has significantly more per-element object/reference overhead.

It also has poorer memory locality.

ArrayList elements are stored in a contiguous backing array, which tends to work well with CPU caches.

LinkedList nodes can be scattered around the heap.

So even if both have an operation that is theoretically O(n), their real-world performance can differ substantially.

**Big interview lesson:**

> Big-O does not tell you everything about production performance.

---

# 15. ArrayList vs LinkedList — Mental Model

Don't memorize:

> ArrayList = fast  
> LinkedList = slow

Instead remember:

### ArrayList

```text
Underlying structure:
Dynamic array

Strength:
Random/index access

Weakness:
Shifting elements during insertion/removal
```

### LinkedList

```text
Underlying structure:
Doubly linked nodes

Strength:
Insertion/removal once position/node is known

Weakness:
Finding a position requires traversal
Memory overhead
Poor locality
```

---

# 16. Interview Questions

Now that we've actually built the concept, these become answerable.

### Q1. Why is LinkedList `get()` O(n)?

Because nodes are not stored in contiguous indexed memory. The list must traverse from either the head or tail until it reaches the requested index.

---

### Q2. Why can LinkedList insertion be O(1)?

Once the target node/position is already known, insertion only requires updating a constant number of `next` and `prev` references.

---

### Q3. Then why is `list.add(index, value)` O(n)?

Because finding the node corresponding to `index` requires traversal.

So:

```text
locate position → O(n)
modify links    → O(1)

total → O(n)
```

---

### Q4. Why does Java LinkedList use a doubly linked list?

It supports traversal in both directions and makes unlinking/relinking nodes efficient.

---

### Q5. Why can LinkedList be slower than ArrayList despite O(1) insertion?

Because of:

- node allocation
- extra references
- pointer chasing
- poor cache locality
- traversal required to reach positions

---

### Q6. When would you choose LinkedList?

A reasonable answer:

> "When I specifically need linked-list/deque semantics, particularly frequent insertion/removal at the ends or operations where I already have the relevant node/position. I wouldn't choose it merely because I expect many indexed insertions, because locating the index is still O(n), and ArrayList often has better locality and lower memory overhead."

That's a **senior-level answer**.

---

## One very important correction to remember

If an interviewer asks:

> **"Is insertion in LinkedList O(1)?"**

Don't simply say **yes**.

Say:

> **"The link manipulation is O(1) once the node/position is known. But insertion by index is O(n) overall because finding that position requires traversal."**

That's exactly the kind of distinction an interviewer can use to separate memorization from understanding.

---

### Next in our flow

We've now covered:

```text
Collection ✅
   ↓
List
   ├── ArrayList ✅
   ├── LinkedList ✅
   ├── Vector      ← NEXT
   └── Stack
```

Next we'll do **Vector** in the same format: **problem → why it exists → internal implementation → synchronization → resizing → performance → production relevance → Vector vs ArrayList vs synchronizedList → interviewer follow-ups.**