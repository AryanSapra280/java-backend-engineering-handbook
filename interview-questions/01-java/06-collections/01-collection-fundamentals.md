# A. Collections Framework Fundamentals

### 253. 🟢 What is the Java Collections Framework?

The **Java Collections Framework (JCF)** is a unified set of **interfaces, implementations, and utility algorithms** for storing and manipulating groups of objects.

It provides:

**Interfaces**

* `Collection`
* `List`
* `Set`
* `Queue`
* `Deque`
* `Map`

**Implementations**

* `ArrayList`
* `LinkedList`
* `HashSet`
* `LinkedHashSet`
* `TreeSet`
* `PriorityQueue`
* `ArrayDeque`
* `HashMap`
* `LinkedHashMap`
* `TreeMap`

**Utilities**

* `Collections`
* `Arrays`

Example:

```java
List<String> names = new ArrayList<>();

names.add("Aryan");
names.add("Rahul");
```

The major benefit is that you program against standardized interfaces instead of implementing common data structures yourself.

---

### 254. 🟢 Why was the Collections Framework introduced?

Before JCF, Java had older collection-like classes such as:

* `Vector`
* `Stack`
* `Hashtable`
* arrays

They had inconsistent APIs and different design approaches.

The Collections Framework introduced a **common architecture and consistent APIs**.

For example:

```java
List<String> list = new ArrayList<>();
```

and:

```java
List<String> list = new LinkedList<>();
```

Both follow the same `List` contract.

This provides:

* Consistent APIs
* Reusable algorithms
* Multiple implementations
* Better type safety with generics
* Easier replacement of implementations
* Standard iteration mechanisms

---

### 255. 🟢 What problems does the Collections Framework solve?

It solves several common problems involved in managing groups of objects.

#### 1. Data structure implementation

Instead of implementing a dynamic array yourself:

```java
ArrayList<String> list = new ArrayList<>();
```

#### 2. Standard APIs

Common operations such as:

```java
add()
remove()
contains()
size()
iterator()
```

have standardized contracts.

#### 3. Different performance characteristics

You can choose based on requirements:

```text
ArrayList       → fast indexed access
LinkedList      → linked-list semantics
HashSet         → hash-based uniqueness
TreeSet         → sorted uniqueness
HashMap         → key-value lookup
PriorityQueue   → priority-based retrieval
```

#### 4. Reusable algorithms

`Collections` provides algorithms/utilities such as:

```java
Collections.sort(list);
Collections.reverse(list);
Collections.shuffle(list);
```

#### 5. Type safety

Generics allow:

```java
List<String> names;
```

instead of legacy raw collections.

---

### 256. 🟢 What is the difference between a Collection and a Collections?

They are completely different.

### `Collection`

`Collection` is an **interface** representing a group of objects.

```java
Collection<String> data;
```

It is part of the collection hierarchy.

### `Collections`

`Collections` is a **utility class** containing static methods for working with collections.

```java
Collections.sort(list);
Collections.reverse(list);
Collections.max(list);
```

So:

```text
Collection  → interface
Collections → utility class
```

**Interview trap:** `Collections` is not the parent of `Collection`.

---

### 257. 🟢 What is the difference between Collection and Map?

`Collection` represents a **group of individual elements**.

```java
List<String> names =
    List.of("A", "B", "C");
```

`Map` represents **key-value associations**.

```java
Map<Integer, String> employees =
    new HashMap<>();

employees.put(101, "Aryan");
employees.put(102, "Rahul");
```

Conceptually:

```text
Collection:
Element
Element
Element

Map:
Key → Value
Key → Value
Key → Value
```

Important:

```text
Collection
 ├── List
 ├── Set
 └── Queue

Map
 ├── HashMap
 ├── LinkedHashMap
 ├── TreeMap
 └── ...
```

`Map` is part of the Collections Framework but **not a subtype of `Collection`**.

---

### 258. 🟢 Explain the overall hierarchy of the Java Collections Framework.

A simplified hierarchy is:

```text
Iterable
   │
   └── Collection
         │
         ├── List
         │     ├── ArrayList
         │     └── LinkedList
         │
         ├── Set
         │     ├── HashSet
         │     ├── LinkedHashSet
         │     └── SortedSet
         │           └── NavigableSet
         │                 └── TreeSet
         │
         └── Queue
               ├── PriorityQueue
               └── Deque
                     ├── ArrayDeque
                     └── LinkedList


Map
 │
 ├── HashMap
 ├── LinkedHashMap
 ├── SortedMap
 │     └── NavigableMap
 │           └── TreeMap
 └── ConcurrentMap
       └── ConcurrentHashMap
```

There are also specialized/concurrent implementations.

For example:

```text
BlockingQueue
ConcurrentMap
CopyOnWriteArrayList
CopyOnWriteArraySet
```

are important in concurrent programming.

---

### 259. 🟢 What is the relationship between `Iterable`, `Collection`, `List`, `Set`, and `Queue`?

The inheritance relationship is:

```text
Iterable
   ↓
Collection
   ├── List
   ├── Set
   └── Queue
```

### `Iterable`

Provides the ability to iterate over elements.

```java
for (String s : collection) {
    System.out.println(s);
}
```

### `Collection`

Represents a general group of elements and provides common operations.

### `List`

Ordered collection that generally allows duplicates.

```java
List<String>
```

### `Set`

Collection that does not permit duplicate elements according to its equality semantics.

```java
Set<String>
```

### `Queue`

Collection designed primarily for holding elements before processing.

```java
Queue<String>
```

---

### 260. 🟢 Why does `Map` not extend `Collection`?

Because a `Map` does not represent a collection of individual elements in the same semantic sense.

A `Map` stores **mappings between keys and values**.

For example:

```java
Map<Integer, Employee> employees;
```

Its fundamental operation is:

```java
put(key, value)
```

whereas `Collection` fundamentally deals with:

```java
add(element)
```

A map can expose:

```java
map.keySet();
map.values();
map.entrySet();
```

These are collections/views, but the `Map` itself is not one.

### Design reason

If `Map` extended `Collection`, it would create awkward semantics:

```java
add(element)
```

What would an "element" of a map be?

A key?

A value?

A key-value pair?

Instead, Java gives `Map` its own abstraction.

---

### 261. 🟢 What is the purpose of the `Iterable` interface?

`Iterable<T>` represents something whose elements can be traversed.

Its key method is:

```java
Iterator<T> iterator();
```

This enables the enhanced `for` loop:

```java
for (String name : names) {
    System.out.println(name);
}
```

The compiler essentially uses an iterator-based traversal.

`Iterable` is broader than `Collection`.

For example, a custom class can implement `Iterable` without being a `Collection`.

```java
class Numbers implements Iterable<Integer> {
    // implement iterator()
}
```

### Important distinction

```text
Iterable → can be iterated
Collection → represents a collection of elements
```

---

### 262. 🟢 What is the purpose of the `Collection` interface?

`Collection` provides the **common contract for groups of objects**.

It defines operations such as:

```java
add()
remove()
contains()
size()
isEmpty()
clear()
iterator()
```

and bulk operations such as:

```java
addAll()
removeAll()
retainAll()
containsAll()
```

It allows algorithms to work with many different collection implementations.

For example:

```java
void process(Collection<String> data) {
    for (String value : data) {
        // process
    }
}
```

This method can accept:

```java
ArrayList
HashSet
LinkedList
...
```

without depending on a specific implementation.

---

### 263. 🟢 What common operations are defined by `Collection`?

Some important methods are:

#### Query operations

```java
size()
isEmpty()
contains(Object)
containsAll(Collection)
```

#### Modification

```java
add(E)
remove(Object)
clear()
```

#### Bulk operations

```java
addAll(Collection)
removeAll(Collection)
retainAll(Collection)
```

#### Traversal

```java
iterator()
```

and modern Java also provides:

```java
stream()
parallelStream()
forEach()
```

#### Array conversion

```java
toArray()
toArray(T[])
```

Example:

```java
Collection<String> names = new ArrayList<>();

names.add("Aryan");

System.out.println(names.size());
System.out.println(names.contains("Aryan"));
```

---

### 264. 🟡 Why does Java provide interfaces such as `List`, `Set`, and `Queue` instead of exposing concrete implementations directly?

Because the interface describes **what behavior is required**, while the implementation determines **how that behavior is achieved**.

For example:

```java
List<String> list = new ArrayList<>();
```

The application depends on the `List` contract rather than `ArrayList` implementation details.

This provides:

* Loose coupling
* Implementation flexibility
* Better testability
* Easier maintenance
* Ability to select implementation based on performance requirements

For example, you can later change:

```java
List<String> list = new ArrayList<>();
```

to:

```java
List<String> list = new LinkedList<>();
```

without changing code that only depends on `List` operations.

---

### 265. 🟡 What are the advantages of programming against the `List` interface instead of `ArrayList`?

Prefer:

```java
List<Employee> employees = new ArrayList<>();
```

over unnecessarily exposing:

```java
ArrayList<Employee> employees = new ArrayList<>();
```

because the first expresses the **required abstraction**.

Advantages:

1. Implementation can be changed.
2. Lower coupling.
3. Easier testing/mocking.
4. Communicates that the caller needs `List` behavior, not `ArrayList` specifically.
5. Allows future implementations such as `LinkedList` or another `List` implementation.

However, if code genuinely requires an `ArrayList`-specific property, using `ArrayList` can be appropriate.

---

### 266. 🟡 What happens when you declare a variable as `List` but instantiate it as `ArrayList`?

```java
List<String> list = new ArrayList<>();
```

There are two types involved:

```text
Reference/compile-time type → List
Actual/runtime object type  → ArrayList
```

The compiler allows you to invoke methods available through `List`.

```java
list.add("A");
list.get(0);
```

At runtime, the actual `ArrayList` implementation executes those operations.

This is a combination of:

* Programming to an interface
* Upcasting
* Runtime polymorphism

You cannot directly call methods that exist only on `ArrayList` through a `List` reference.

---

### 267. 🟡 Can you change the implementation from `ArrayList` to `LinkedList` without changing the variable type?

**Yes**, provided your code only relies on the `List` contract.

```java
List<String> list = new ArrayList<>();
```

can become:

```java
List<String> list = new LinkedList<>();
```

The variable type remains:

```java
List<String>
```

This is one of the main benefits of programming against interfaces.

But behavior/performance characteristics can change significantly.

For example:

```java
list.get(index);
```

is typically:

```text
ArrayList  → O(1)
LinkedList → O(n)
```

So changing implementation may not require source-code changes, but it can still affect performance.

---

### 268. 🟡 What is the difference between `Collection`, `Collections`, and `Arrays` utility classes?

They are three different concepts.

| Name          | What is it?   | Purpose                              |
| ------------- | ------------- | ------------------------------------ |
| `Collection`  | Interface     | Represents a group of elements       |
| `Collections` | Utility class | Algorithms/utilities for collections |
| `Arrays`      | Utility class | Algorithms/utilities for arrays      |

Examples:

```java
Collection<String> c;
```

```java
Collections.sort(list);
Collections.reverse(list);
Collections.max(list);
```

```java
Arrays.sort(array);
Arrays.binarySearch(array, value);
Arrays.toString(array);
```

Important:

```text
Collection  → abstraction
Collections → collection utility methods
Arrays      → array utility methods
```

---

### 269. 🟡 What are the major characteristics of the main Java collection types?

A useful interview table:

| Type            | Duplicates  | Ordering               | Sorted     | Typical lookup/access                         |
| --------------- | ----------- | ---------------------- | ---------- | --------------------------------------------- |
| `ArrayList`     | Yes         | Insertion order        | No         | Fast index access                             |
| `LinkedList`    | Yes         | Insertion order        | No         | Sequential access                             |
| `HashSet`       | No          | No guaranteed order    | No         | Fast average lookup                           |
| `LinkedHashSet` | No          | Insertion order        | No         | Hash-based                                    |
| `TreeSet`       | No          | Sorted                 | Yes        | O(log n) typical                              |
| `PriorityQueue` | Yes         | Priority-based head    | Heap order | Head O(1), insertion/removal O(log n) typical |
| `ArrayDeque`    | Yes         | Deque order            | No         | Fast ends                                     |
| `HashMap`       | Keys unique | No guaranteed order    | No         | Fast average lookup                           |
| `LinkedHashMap` | Keys unique | Insertion/access order | No         | Hash-based                                    |
| `TreeMap`       | Keys unique | Sorted by key          | Yes        | O(log n) typical                              |

The exact performance can depend on implementation details and workload.

---

### 270. 🟡 Which collection types maintain insertion order?

Common ones:

```text
ArrayList
LinkedList
LinkedHashSet
LinkedHashMap
```

For maps, `LinkedHashMap` maintains a defined encounter/order behavior; by default this is insertion order, and it can alternatively be configured for access order.

Example:

```java
Set<String> set = new LinkedHashSet<>();
```

If inserted:

```text
A
C
B
```

iteration gives:

```text
A
C
B
```

### Important trap

`HashSet` and `HashMap` **do not guarantee insertion order**.

Never write code that assumes their current iteration order is insertion order.

---

### 271. 🟡 Which collection types maintain sorted order?

Common examples:

```text
TreeSet
TreeMap
```

They are based on tree structures and maintain ordering according to:

* Natural ordering (`Comparable`)
* Or a supplied `Comparator`

Example:

```java
Set<Integer> numbers = new TreeSet<>();

numbers.add(30);
numbers.add(10);
numbers.add(20);
```

Iteration:

```text
10
20
30
```

For a map:

```java
Map<Integer, String> map = new TreeMap<>();
```

keys are maintained in sorted order.

---

### 272. 🟡 Which collections allow duplicates?

### Allow duplicates

```text
ArrayList
LinkedList
Vector
Stack
Queue implementations generally allow duplicates
Deque implementations generally allow duplicates
```

For example:

```java
List<String> list =
    new ArrayList<>();

list.add("A");
list.add("A");
```

Both remain.

### Do not allow duplicate elements

```text
HashSet
LinkedHashSet
TreeSet
```

For maps, **keys cannot be duplicated**, but values can.

```java
Map<Integer, String> map = new HashMap<>();

map.put(1, "A");
map.put(2, "A"); // duplicate value is fine
```

---

### 273. 🟢 Which collections allow `null`?

This is an important interview question because the answer depends on the implementation.

Common behavior:

| Collection      | `null`                                                |
| --------------- | ----------------------------------------------------- |
| `ArrayList`     | Yes                                                   |
| `LinkedList`    | Yes                                                   |
| `HashSet`       | Yes, typically one                                    |
| `LinkedHashSet` | Yes                                                   |
| `TreeSet`       | Generally no with natural ordering                    |
| `PriorityQueue` | No                                                    |
| `ArrayDeque`    | No                                                    |
| `HashMap`       | One null key + multiple null values                   |
| `LinkedHashMap` | Yes                                                   |
| `TreeMap`       | Generally null keys not allowed with natural ordering |

For example:

```java
List<String> list = new ArrayList<>();
list.add(null);
```

is valid.

For `HashMap`:

```java
Map<String, Integer> map = new HashMap<>();

map.put(null, 10);
map.put("A", null);
```

both are allowed.

### Important nuance

Don't memorize this as an absolute rule for all implementations. `Map`, `Set`, etc. are interfaces, and their implementations can have different null policies.

---

### 274. 🟡 Which collections are thread-safe by default?

Most modern general-purpose collections such as:

```text
ArrayList
HashMap
HashSet
LinkedList
TreeMap
TreeSet
ArrayDeque
```

are **not thread-safe by default**.

Legacy synchronized collections include:

```text
Vector
Hashtable
```

There are also dedicated concurrent collections:

```text
ConcurrentHashMap
CopyOnWriteArrayList
CopyOnWriteArraySet
BlockingQueue implementations
ConcurrentLinkedQueue
ConcurrentLinkedDeque
```

You can also create synchronized wrappers:

```java
List<String> list =
    Collections.synchronizedList(new ArrayList<>());
```

But this does not make every compound operation automatically atomic.

For example, a sequence like:

```java
if (!list.contains(x)) {
    list.add(x);
}
```

still requires appropriate external synchronization if atomicity of the entire sequence is required.

---

### 275. 🟡 Which collection would you choose if you need fast random access?

Usually:

```java
ArrayList
```

because it is backed by a dynamically resizable array and provides:

```java
list.get(index)
```

with typically **O(1)** random access.

Example:

```java
List<Integer> numbers = new ArrayList<>();

numbers.get(500000);
```

is efficient.

Compared with:

```java
LinkedList
```

which generally requires traversal to reach an arbitrary index, giving **O(n)** indexed access.

---

### 276. 🟡 Which collection would you choose if duplicate elements are not allowed?

Use a:

```text
Set
```

Common choices:

```text
HashSet       → uniqueness + fast average lookup
LinkedHashSet → uniqueness + insertion order
TreeSet       → uniqueness + sorted order
```

So the requirement determines the implementation.

For example:

```java
Set<String> emails = new HashSet<>();
```

---

### 277. 🟡 Which collection would you choose if elements must remain sorted?

Use:

```java
TreeSet
```

if you need unique sorted elements.

```java
Set<Integer> numbers = new TreeSet<>();
```

If you need sorted key-value pairs:

```java
Map<Integer, Employee> employees =
    new TreeMap<>();
```

The ordering can use natural ordering or a `Comparator`.

---

### 278. 🟡 Which collection would you choose for FIFO processing?

Use a **`Queue`**, commonly:

```java
Queue<Task> queue = new ArrayDeque<>();
```

FIFO means:

```text
First In → First Out
```

Example:

```text
A
B
C

poll() → A
poll() → B
poll() → C
```

Typical methods:

```java
offer()  // insert
poll()   // remove head
peek()   // inspect head
```

For concurrent producer-consumer systems, you may instead use a suitable `BlockingQueue`, such as:

```java
BlockingQueue<Task> queue =
    new LinkedBlockingQueue<>();
```

---

### 279. 🟡 Which collection would you choose for LIFO processing?

Use a **`Deque`**, commonly:

```java
Deque<Integer> stack = new ArrayDeque<>();
```

LIFO means:

```text
Last In → First Out
```

Example:

```java
stack.push(10);
stack.push(20);
stack.push(30);

stack.pop(); // 30
```

Prefer `ArrayDeque` for stack behavior rather than the legacy `Stack` class in most new code.

```text
push() → add to front
pop()  → remove from front
peek() → inspect front
```

---

### 280. 🔴 What design principles were considered while designing the Java Collections Framework?

This is a very good experienced-level interview question.

The Collections Framework was designed around several important principles.

### 1. Program to interfaces

The framework separates **contracts from implementations**.

```java
List<String> list = new ArrayList<>();
```

`List` defines the abstraction while `ArrayList` provides the implementation.

---

### 2. Separation of interface and implementation

The same abstraction can have multiple implementations with different characteristics.

```text
List
 ├── ArrayList
 └── LinkedList

Set
 ├── HashSet
 ├── LinkedHashSet
 └── TreeSet
```

This allows implementation selection based on requirements.

---

### 3. Reusable algorithms

Algorithms are separated from data structures.

For example:

```java
Collections.sort(list);
```

The algorithm can operate on many compatible collection types.

This avoids putting every algorithm into every implementation.

---

### 4. Minimize implementation effort

If a class implements a collection interface, it can inherit/use common behavior rather than implementing every operation from scratch.

The framework provides reusable abstract base classes such as:

```text
AbstractCollection
AbstractList
AbstractSet
AbstractMap
```

---

### 5. Interoperability

Different collections should be able to work with common APIs.

For example:

```java
Collection<String>
```

can be passed to methods that operate on the `Collection` abstraction.

---

### 6. Performance choices

The framework provides multiple implementations because there is no single data structure that is optimal for every workload.

For example:

```text
ArrayList   → indexed access
HashSet     → hash-based lookup
TreeSet     → sorted data
ArrayDeque  → operations at both ends
HashMap     → key-value lookup
```

---

### 7. Type safety

Generics allow compile-time checking:

```java
List<String> names = new ArrayList<>();
```

This prevents many accidental type errors.

Without generics:

```java
List list = new ArrayList();
list.add("Aryan");
list.add(100);
```

would allow unrelated types and require casts later.

---

### 8. Extensibility

Developers can create their own collection implementations by implementing/extending the appropriate framework abstractions.

For example:

```java
class MyCollection<E> extends AbstractCollection<E> {
    // implementation
}
```

---

### 9. Avoid unnecessary coupling

Code can depend on:

```java
List
Set
Map
Queue
```

rather than concrete implementations.

This supports maintainability and easier substitution.

---

### 10. Common vocabulary

The framework gives developers standardized concepts:

```text
List
Set
Queue
Deque
Map
Iterator
Comparator
```

So when an interviewer says:

> "I need unique elements while preserving insertion order."

you immediately think:

```java
LinkedHashSet
```

rather than designing a custom data structure.

### 🔥 Interview summary

The core design philosophy can be remembered as:

```text
Interfaces
    ↓
Multiple implementations
    ↓
Reusable algorithms
    ↓
Generic/type-safe APIs
    ↓
Interoperability + flexibility
    ↓
Performance choices without changing the abstraction
```

The most important principle is **separation of abstraction from implementation**: your code should generally depend on `List`, `Set`, `Map`, etc., while choosing `ArrayList`, `HashSet`, `HashMap`, `TreeMap`, etc. based on the required semantics and performance characteristics.
