Absolutely. Let’s go into **Part 2 — Collections** at interview depth, but keep it organized enough that you can actually revise it.

The key here is not memorizing the hierarchy. You need to understand **why each collection exists, what data structure backs it, its complexity, ordering guarantees, thread-safety, and which one you would choose in production.**

# PART 2 — COLLECTIONS 🔥🔥

## 1. First: The Collections hierarchy

The big picture:

```text
                    Iterable
                       |
                  Collection
              /       |       \
            List      Set      Queue
             |         |         |
         ArrayList   HashSet   PriorityQueue
         LinkedList  LinkedHashSet
         Vector      TreeSet
         Stack

                         Map
                          |
          --------------------------------
          |              |              |
       HashMap       LinkedHashMap    TreeMap
          |
   ConcurrentHashMap
   Hashtable
```

### Important interview correction

`Map` is **not** a subtype of `Collection`.

Why?

Because:

```text
Collection → individual elements

Map → key-value mappings
```

For example:

```java
List<String>
```

stores:

```text
A
B
C
```

while:

```java
Map<Integer, String>
```

stores:

```text
1 → Aryan
2 → Rahul
3 → John
```

---

# 2. List vs Set vs Map

### Q: What's the difference?

| | List | Set | Map |
|---|---|---|---|
| Stores | Elements | Unique elements | Key-value pairs |
| Duplicates | Allowed | Not allowed | Keys unique |
| Access | Index | Usually no index | Key |
| Example | ArrayList | HashSet | HashMap |

### Interview answer

> I use a List when order and duplicate elements matter, a Set when uniqueness matters, and a Map when I need key-based lookup or association between two values.

That is much better than simply saying "List allows duplicates."

---

# 3. ArrayList 🔥

`ArrayList` is one of the most important collections.

Conceptually:

```text
ArrayList
    |
    └── resizable array
```

Example:

```java
List<String> names = new ArrayList<>();

names.add("Aryan");
names.add("Rahul");
names.add("John");
```

Internally, it maintains an array-like structure.

---

## Q: Why is ArrayList fast for get()?

```java
names.get(2);
```

Because it uses index-based array access.

Complexity:

```text
get(index)       → O(1)
set(index)       → O(1)
add(end)         → O(1) amortized
add(index)       → O(n)
remove(index)    → O(n)
contains()       → O(n)
```

### Why is insertion in the middle O(n)?

Suppose:

```text
[A, B, C, D]
```

Insert `X` at index 1:

```text
[A, X, B, C, D]
```

Elements after index 1 need to shift.

```text
B → right
C → right
D → right
```

Hence O(n).

---

# 4. ArrayList resizing

This is a common follow-up.

### Q: What happens when ArrayList's internal array becomes full?

It creates a larger array and copies the existing elements.

Conceptually:

```text
Old:
[ A B C D ]

        ↓ resize

New:
[ A B C D _ _ ]
```

Then the old array becomes eligible for garbage collection if nothing else references it.

### Why is `add()` amortized O(1)?

Most additions don't require resizing.

Occasionally a resize requires O(n), but across many insertions the average cost is amortized O(1).

---

# 5. ArrayList vs LinkedList 🔥

This is a **classic interview question**.

## ArrayList

Backed by a dynamically resized array.

## LinkedList

Implemented as a doubly linked list.

Conceptually:

```text
ArrayList

[A][B][C][D]
```

versus:

```text
LinkedList

[A] ⇄ [B] ⇄ [C] ⇄ [D]
```

---

### Complexity

| Operation | ArrayList | LinkedList |
|---|---:|---:|
| get(index) | O(1) | O(n) |
| add(end) | O(1) amortized | O(1) |
| add(index) | O(n) | O(n) to locate |
| remove(index) | O(n) | O(n) to locate |
| contains | O(n) | O(n) |

### Important correction

You'll sometimes hear:

> "LinkedList insertion is O(1)."

That's incomplete.

If you **already have the node/iterator position**, insertion can be O(1).

But:

```java
list.add(500000, value);
```

requires finding the position first.

That traversal is O(n).

### Practical answer

> ArrayList is usually the default choice for List because it provides fast random access and good cache locality. LinkedList is useful when frequent insertions/removals occur through an iterator or known node position, but it is often overused.

That's a much stronger 5-YOE answer.

---

# 6. Why LinkedList is often slower than people expect

This is a nice deeper question.

A linked list has nodes like:

```text
Node
 ├── previous
 ├── data
 └── next
```

Every node is a separate object.

Therefore:

- more memory
- pointer/reference chasing
- poorer CPU cache locality
- object allocation overhead

An ArrayList stores elements more compactly in an array.

So even when theoretical complexity looks similar, **ArrayList often performs better in real applications**.

---

# 7. Vector

`Vector` is an older synchronized dynamic array implementation.

```java
Vector<String> vector = new Vector<>();
```

Historically:

```text
Vector → synchronized
ArrayList → not synchronized
```

### Q: Should I use Vector in new code?

Generally, no.

For normal single-threaded use:

```java
ArrayList
```

For concurrency, choose an appropriate concurrent collection based on the access pattern, rather than automatically using Vector.

---

# 8. Stack

Java has:

```java
Stack<Integer> stack = new Stack<>();
```

It is a legacy class extending `Vector`.

Traditional operations:

```java
push()
pop()
peek()
```

### Preferred modern choice

Use:

```java
Deque<Integer> stack = new ArrayDeque<>();
```

Then:

```java
stack.push(10);
stack.push(20);

stack.pop();
stack.peek();
```

### Interview answer

> `Stack` is a legacy synchronized class. For stack behavior, `ArrayDeque` is generally preferred.

---

# 9. Queue

Queue represents typically FIFO processing.

```text
First In → First Out
```

Example:

```text
A
B
C
```

Remove:

```text
A
```

Common methods:

```java
queue.offer(element);
queue.poll();
queue.peek();
```

Why prefer these over `add/remove/element`?

Because the `offer/poll/peek` family provides non-exception-based behavior for failure/empty cases.

For example:

```java
poll()
```

returns:

```text
null
```

when empty.

Whereas:

```java
remove()
```

throws an exception if the queue is empty.

---

# 10. Deque

Deque = **Double Ended Queue**.

You can insert/remove from both ends.

```text
front                     rear
  ↓                         ↓
[A] [B] [C] [D]
```

Methods include:

```java
addFirst()
addLast()

removeFirst()
removeLast()

peekFirst()
peekLast()
```

And this is why `ArrayDeque` can act as both:

### Queue

```java
Deque<Integer> queue = new ArrayDeque<>();

queue.offerLast(10);
queue.offerLast(20);

queue.pollFirst();
```

### Stack

```java
Deque<Integer> stack = new ArrayDeque<>();

stack.push(10);
stack.push(20);

stack.pop();
```

---

# 11. Why ArrayDeque is preferred over Stack

For stack/queue operations:

```java
Deque<Integer> deque = new ArrayDeque<>();
```

is generally preferable because:

- modern API
- efficient
- no unnecessary legacy synchronization
- can operate from both ends

One important limitation:

> `ArrayDeque` does not allow `null` elements.

---

# 12. PriorityQueue 🔥

This one frequently appears in coding interviews.

A `PriorityQueue` does **not** behave like a normal FIFO queue.

It orders elements according to priority.

By default:

```java
PriorityQueue<Integer> pq = new PriorityQueue<>();
```

is a min-heap behavior.

```java
pq.add(30);
pq.add(10);
pq.add(20);

System.out.println(pq.poll());
```

Output:

```text
10
```

Then:

```text
20
30
```

---

## Important trap

### Q: Does iterating over PriorityQueue give sorted order?

**No.**

The queue guarantees that the **head** has the highest priority according to its ordering.

If you need sorted extraction:

```java
while (!pq.isEmpty()) {
    System.out.println(pq.poll());
}
```

---

## Complexity

For a heap-based PriorityQueue:

```text
peek()       O(1)
offer()      O(log n)
poll()       O(log n)
```

This is why it's useful for:

- top K elements
- Kth largest/smallest
- scheduling
- merging sorted data
- Dijkstra-like algorithms

---

# 13. Set hierarchy 🔥

Main implementations:

```text
Set
 |
 +-- HashSet
 |
 +-- LinkedHashSet
 |
 +-- SortedSet
      |
      +-- TreeSet
```

---

# 14. HashSet

`HashSet` guarantees uniqueness.

```java
Set<Integer> set = new HashSet<>();

set.add(10);
set.add(20);
set.add(10);
```

Result contains:

```text
10
20
```

The second `10` isn't added.

### How?

Conceptually:

> HashSet is backed by a HashMap.

It uses hashing and equality to determine whether an element already exists.

This means the quality of your:

```java
hashCode()
equals()
```

implementation matters.

---

# 15. LinkedHashSet

Difference:

```text
HashSet
→ uniqueness
→ no ordering guarantee

LinkedHashSet
→ uniqueness
→ insertion-order iteration
```

Example:

```java
Set<Integer> set = new LinkedHashSet<>();

set.add(30);
set.add(10);
set.add(20);
```

Iteration:

```text
30
10
20
```

This is exactly why we used `LinkedHashSet` earlier to remove duplicates while preserving order.

---

# 16. TreeSet 🔥

`TreeSet` maintains sorted order.

```java
Set<Integer> set = new TreeSet<>();

set.add(30);
set.add(10);
set.add(20);
```

Iteration:

```text
10
20
30
```

It is based on a tree structure and uses:

- natural ordering
- or Comparator

### Complexity

Typical operations:

```text
add       O(log n)
remove    O(log n)
contains  O(log n)
```

---

# 17. HashSet vs LinkedHashSet vs TreeSet

Memorize this table:

| | HashSet | LinkedHashSet | TreeSet |
|---|---|---|---|
| Unique | Yes | Yes | Yes |
| Insertion order | No guarantee | Yes | No |
| Sorted | No | No | Yes |
| Typical lookup | O(1) | O(1) | O(log n) |
| Backing idea | Hash table | Hash table + linked order | Tree |

### Interview question

> I need unique employee IDs and don't care about order.

Use:

```java
HashSet
```

> Unique IDs and need insertion order.

```java
LinkedHashSet
```

> Unique IDs and need sorted order.

```java
TreeSet
```

---

# 18. Map hierarchy 🔥🔥

The major ones:

```text
Map
 |
 +-- HashMap
 |
 +-- LinkedHashMap
 |
 +-- SortedMap
       |
       +-- TreeMap
 |
 +-- Hashtable
 |
 +-- ConcurrentHashMap
```

This is where the interview becomes more interesting.

---

# 19. HashMap

HashMap stores:

```text
key → value
```

Example:

```java
Map<Integer, String> employees = new HashMap<>();

employees.put(101, "Aryan");
employees.put(102, "Rahul");
```

Lookup:

```java
employees.get(101);
```

Typical average complexity:

```text
put       O(1)
get       O(1)
remove    O(1)
```

Worst-case behavior depends on collisions and implementation details.

We'll do the **full HashMap internal mechanism in Part 3**.

---

# 20. LinkedHashMap

LinkedHashMap extends the HashMap idea with predictable iteration order.

Default:

```text
insertion order
```

Example:

```java
Map<Integer, String> map = new LinkedHashMap<>();

map.put(3, "C");
map.put(1, "A");
map.put(2, "B");
```

Iteration:

```text
3 → C
1 → A
2 → B
```

---

## Important: access-order mode

LinkedHashMap can also maintain **access order**.

```java
new LinkedHashMap<>(
    16,
    0.75f,
    true
);
```

The third argument:

```text
true
```

means access order.

This makes LinkedHashMap extremely useful for implementing an **LRU cache**.

We'll revisit this in Part 11.

---

# 21. TreeMap

TreeMap maintains keys in sorted order.

```java
Map<Integer, String> map = new TreeMap<>();

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

Typical operations:

```text
get      O(log n)
put      O(log n)
remove   O(log n)
```

Why?

Because TreeMap uses a balanced tree structure.

---

# 22. HashMap vs TreeMap vs LinkedHashMap

This is a must-know comparison.

| | HashMap | LinkedHashMap | TreeMap |
|---|---|---|---|
| Ordering | None guaranteed | Insertion/access | Sorted keys |
| get | O(1) avg | O(1) avg | O(log n) |
| put | O(1) avg | O(1) avg | O(log n) |
| Null key | One | One | Generally no |
| Use when | Fast lookup | Lookup + order | Sorted keys/range operations |

### Practical examples

**Fast lookup:**

```java
HashMap
```

**Maintain insertion order:**

```java
LinkedHashMap
```

**Need sorted keys / range operations:**

```java
TreeMap
```

---

# 23. Hashtable

Another legacy collection.

Characteristics:

- synchronized
- legacy
- does not allow null keys
- does not allow null values

```java
Hashtable<Integer, String> map = new Hashtable<>();
```

### HashMap vs Hashtable

| | HashMap | Hashtable |
|---|---|---|
| Thread-safe | No | Synchronized |
| Null key | One allowed | No |
| Null value | Allowed | No |
| Modern usage | Yes | Rare |

### Interview answer

> Hashtable is a legacy synchronized Map implementation. For concurrent applications, I would generally consider ConcurrentHashMap instead because it provides better concurrency characteristics.

---

# 24. ConcurrentHashMap 🔥🔥

This is one of the most likely follow-ups after HashMap.

```java
ConcurrentHashMap<String, Integer> map =
        new ConcurrentHashMap<>();
```

It supports concurrent access safely.

But here's the important part:

### Don't say:

> ConcurrentHashMap locks the whole map.

That's an outdated oversimplification.

Modern implementations use sophisticated synchronization/CAS techniques with much finer-grained coordination than locking the entire map for ordinary updates.

For interview purposes:

> ConcurrentHashMap allows multiple threads to operate on the map concurrently while providing thread-safe operations, avoiding a single global lock for normal access.

---

# 25. ConcurrentHashMap atomic operations

This is extremely useful in real code.

Suppose:

```java
map.put(key, map.getOrDefault(key, 0) + 1);
```

Looks okay?

Not necessarily under concurrency.

Two threads can do:

```text
Thread A → get 5
Thread B → get 5

Thread A → put 6
Thread B → put 6
```

Expected:

```text
7
```

Actual:

```text
6
```

Race condition.

Use:

```java
map.merge(key, 1, Integer::sum);
```

or:

```java
map.compute(key, (k, v) -> v == null ? 1 : v + 1);
```

These provide atomic map-level operations.

Other useful methods:

```java
putIfAbsent()
computeIfAbsent()
compute()
merge()
```

This topic connects directly to **Part 8 concurrency**.

---

# 26. CopyOnWriteArrayList 🔥

This is a specialized concurrent List.

```java
CopyOnWriteArrayList<String> list =
        new CopyOnWriteArrayList<>();
```

The key idea:

> Writes create a new underlying array; readers can safely iterate over a stable snapshot.

So it is particularly useful when:

```text
Reads >>> Writes
```

Examples:

- configuration listeners
- event subscribers
- relatively static reference data

### Bad use case

If you have:

```text
millions of writes
```

then CopyOnWriteArrayList can be expensive because each mutation involves copying the array.

---

# 27. Iterator vs ListIterator

## Iterator

Works with most collections.

```java
Iterator<String> iterator = list.iterator();

while (iterator.hasNext()) {
    String value = iterator.next();
}
```

Supports:

```java
hasNext()
next()
remove()
```

---

## ListIterator

Only for Lists.

It can move in both directions:

```java
hasNext()
next()

hasPrevious()
previous()
```

And supports:

```java
add()
set()
remove()
```

Example:

```java
ListIterator<String> iterator = list.listIterator();

while (iterator.hasNext()) {
    System.out.println(iterator.next());
}
```

---

# 28. The classic ConcurrentModificationException question 🔥🔥

### Q

What's wrong with this?

```java
List<Integer> numbers =
        new ArrayList<>(Arrays.asList(1, 2, 3, 4));

for (Integer number : numbers) {

    if (number % 2 == 0) {
        numbers.remove(number);
    }
}
```

It can throw:

```text
ConcurrentModificationException
```

Why?

Because the collection is structurally modified while its iterator is being used.

---

## Correct solution 1 — Iterator

```java
Iterator<Integer> iterator = numbers.iterator();

while (iterator.hasNext()) {

    Integer number = iterator.next();

    if (number % 2 == 0) {
        iterator.remove();
    }
}
```

---

## Correct solution 2 — removeIf

Modern and clean:

```java
numbers.removeIf(number -> number % 2 == 0);
```

For an interview, I'd prefer this if the interviewer isn't specifically testing Iterator mechanics.

---

# 29. Fail-fast vs fail-safe

This terminology needs careful handling.

### Fail-fast

An iterator may detect structural modification and throw:

```text
ConcurrentModificationException
```

Example:

```java
ArrayList
HashMap
HashSet
```

But important:

> Fail-fast behavior is **best-effort**, not a concurrency guarantee.

Don't say:

> "Fail-fast always throws ConcurrentModificationException."

That's too absolute.

---

### Concurrent collections

Collections such as:

```text
ConcurrentHashMap
CopyOnWriteArrayList
```

provide concurrency-oriented iteration semantics rather than the ordinary fail-fast behavior.

### CopyOnWriteArrayList

Its iterator operates over a snapshot of the array.

Therefore modifications after iterator creation don't modify that iterator's snapshot.

---

# 30. Collections utility class

Don't confuse:

```text
Collection
```

with:

```text
Collections
```

`Collections` is a utility class containing methods such as:

```java
Collections.sort(list);
Collections.reverse(list);
Collections.shuffle(list);
Collections.max(list);
Collections.min(list);
Collections.frequency(list, value);
Collections.binarySearch(list, value);
```

Example:

```java
List<Integer> numbers =
        new ArrayList<>(Arrays.asList(5, 2, 8, 1));

Collections.sort(numbers);
```

---

# 🔥 COLLECTIONS CODING QUESTIONS

Now let's turn this into actual interview coding.

---

## Coding 1 — Remove duplicates preserving order

Input:

```java
[4, 2, 4, 1, 2, 3]
```

Expected:

```text
[4, 2, 1, 3]
```

### Best simple solution

```java
List<Integer> result =
        new ArrayList<>(new LinkedHashSet<>(numbers));
```

### Why?

`LinkedHashSet` gives us:

```text
unique + insertion order
```

Complexity:

```text
Time: O(n) average
Space: O(n)
```

---

# Coding 2 — Find first duplicate

Input:

```text
[4, 2, 3, 2, 5, 4]
```

Output:

```text
2
```

```java
public static Integer firstDuplicate(List<Integer> numbers) {

    Set<Integer> seen = new HashSet<>();

    for (Integer number : numbers) {

        if (!seen.add(number)) {
            return number;
        }
    }

    return null;
}
```

### Why this is good interview code

It's:

- simple
- O(n)
- easy to explain
- no unnecessary cleverness

---

# Coding 3 — Frequency map

Input:

```text
["Java", "Spring", "Java", "Kafka", "Spring"]
```

Output:

```text
Java   → 2
Spring → 2
Kafka  → 1
```

```java
public static Map<String, Integer> frequency(
        List<String> words) {

    Map<String, Integer> frequency = new HashMap<>();

    for (String word : words) {

        frequency.put(
                word,
                frequency.getOrDefault(word, 0) + 1
        );
    }

    return frequency;
}
```

This pattern is **extremely high-value**.

---

# Coding 4 — First non-repeating number

Input:

```text
[4, 5, 1, 2, 1, 5, 4]
```

Output:

```text
2
```

### Solution

```java
public static Integer firstNonRepeating(
        List<Integer> numbers) {

    Map<Integer, Integer> frequency =
            new LinkedHashMap<>();

    for (Integer number : numbers) {
        frequency.put(
                number,
                frequency.getOrDefault(number, 0) + 1
        );
    }

    for (Map.Entry<Integer, Integer> entry
            : frequency.entrySet()) {

        if (entry.getValue() == 1) {
            return entry.getKey();
        }
    }

    return null;
}
```

Again:

> `LinkedHashMap` is important because frequency alone isn't enough—we also need original order.

---

# Coding 5 — Top K frequent elements

This is a slightly harder one.

Input:

```text
[1, 1, 1, 2, 2, 3]
```

`k = 2`

Output:

```text
[1, 2]
```

First frequency:

```java
Map<Integer, Integer> frequency = new HashMap<>();

for (int number : numbers) {
    frequency.put(
        number,
        frequency.getOrDefault(number, 0) + 1
    );
}
```

Then use PriorityQueue.

```java
PriorityQueue<Map.Entry<Integer, Integer>> pq =
        new PriorityQueue<>(
            Comparator.comparingInt(Map.Entry::getValue)
        );

for (Map.Entry<Integer, Integer> entry
        : frequency.entrySet()) {

    pq.offer(entry);

    if (pq.size() > k) {
        pq.poll();
    }
}
```

At the end:

```java
List<Integer> result = new ArrayList<>();

while (!pq.isEmpty()) {
    result.add(pq.poll().getKey());
}
```

### Why min-heap?

We keep only the top `k`.

When size becomes `k + 1`, remove the smallest frequency.

This is an important interview pattern:

```text
Frequency Map
      ↓
Min Heap of size K
      ↓
Top K
```

---

# 31. Production decision questions

These are the questions that distinguish someone who knows APIs from someone who understands engineering.

### Q: I need a List with millions of reads and occasional writes.

Usually:

```text
ArrayList
```

because random access is efficient and memory locality is good.

---

### Q: I need uniqueness and don't care about order.

```text
HashSet
```

---

### Q: I need uniqueness + insertion order.

```text
LinkedHashSet
```

---

### Q: I need uniqueness + sorted order.

```text
TreeSet
```

---

### Q: I need key-value lookup.

```text
HashMap
```

---

### Q: I need key-value lookup + insertion order.

```text
LinkedHashMap
```

---

### Q: I need sorted keys/range queries.

```text
TreeMap
```

---

### Q: Multiple threads read/write a map.

Potential choice:

```text
ConcurrentHashMap
```

rather than blindly using:

```text
HashMap + synchronized
```

---

### Q: Many readers, very few writes to a List.

Potential choice:

```text
CopyOnWriteArrayList
```

---

### Q: I need a stack.

```text
ArrayDeque
```

rather than legacy:

```text
Stack
```

---

### Q: I need top K.

Think:

```text
PriorityQueue
```

---

# 🚨 PART 2 — Interview traps you MUST know

### Trap 1

**Is LinkedList always better for insertion?**

No.

If finding the insertion position costs O(n), the complete operation isn't O(1).

---

### Trap 2

**Does HashSet preserve insertion order?**

No.

Use LinkedHashSet if you need insertion order.

---

### Trap 3

**Does PriorityQueue iteration produce sorted output?**

No.

Repeated `poll()` produces priority order.

---

### Trap 4

**Is HashMap thread-safe?**

No.

---

### Trap 5

**Does synchronized collection automatically make compound operations atomic?**

Not necessarily.

This:

```java
if (!map.containsKey(key)) {
    map.put(key, value);
}
```

is a compound operation and can still have race conditions if performed independently by multiple threads.

Concurrent collections provide atomic methods such as:

```java
putIfAbsent()
computeIfAbsent()
merge()
```

---

### Trap 6

**Is ConcurrentModificationException only caused by multiple threads?**

No.

Even a single thread can cause it:

```java
for (String value : list) {
    list.remove(value);
}
```

because the iterator detects structural modification.

---

# 🧠 PART 2 — The mental map

Before moving to HashMap internals, you should have this in your head:

```text
                    COLLECTIONS
                         |
        ┌────────────────┼────────────────┐
        ↓                ↓                ↓
       List             Set             Queue
        |                |                |
   ArrayList          HashSet        ArrayDeque
   LinkedList       LinkedHashSet    PriorityQueue
   Vector             TreeSet
   Stack

                         Map
                          |
          ┌───────────────┼──────────────┐
          ↓               ↓              ↓
       HashMap      LinkedHashMap     TreeMap
          |
   ConcurrentHashMap
```

And selection:

```text
Need duplicates? ──────────────► List

Need uniqueness?
       |
       ├── no order ───────────► HashSet
       ├── insertion order ────► LinkedHashSet
       └── sorted ─────────────► TreeSet


Need key/value?
       |
       ├── fast lookup ────────► HashMap
       ├── insertion order ────► LinkedHashMap
       └── sorted keys ────────► TreeMap


Need concurrency?
       |
       ├── Map ────────────────► ConcurrentHashMap
       └── read-heavy List ────► CopyOnWriteArrayList


Need stack? ───────────────────► ArrayDeque

Need top-K / priority? ────────► PriorityQueue
```

## ⭐ The next part is particularly important

**PART 3 — HashMap Internals 🔥🔥🔥**

We're going to take one line:

```java
map.put("Aryan", 100);
```

and walk through **exactly what happens internally**:

```text
hashCode()
    ↓
hash spreading
    ↓
bucket calculation
    ↓
array index
    ↓
Node<K,V>
    ↓
collision
    ↓
equals()
    ↓
linked list
    ↓
treeification
    ↓
load factor
    ↓
resize
    ↓
rehashing
```

Then we'll cover the interview traps around **mutable keys, capacity, `0.75` load factor, treeification threshold 8, capacity 64, resize, and why `ConcurrentHashMap` is different**.

That is probably the **highest-value Core Java internal topic** for your EPAM preparation.