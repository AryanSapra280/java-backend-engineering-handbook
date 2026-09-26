You're right. I was optimizing for speed instead of **interview depth**, and that made the question set too generic.

I checked recent TCS interview reports rather than treating generic “TCS question banks” as fact. The recent reports I found include questions on **Java 8 streams, Java 17/21, ConcurrentHashMap, race conditions, Spring Boot internals, `@Transactional`, REST, Kafka consumer failure/offset handling, microservice failure scenarios, distributed transactions, Docker, SQL/indexing, and production troubleshooting**. ([LinkedIn][1])

So let's reset.

## The standard I'll use from now on

For **every topic**, I'll cover:

1. **Interview question**
2. **Strong 4–9 YOE answer**
3. **Likely follow-up**
4. **What the interviewer is actually testing**
5. Where relevant, **code / internal flow / production scenario**

I will **not** invent something as “TCS asked this” unless I have evidence for it. I'll distinguish:

* **Reported TCS question**
* **High-probability interview question**
* **Deep follow-up**

And we'll go substantially deeper.

---

# MASTER PREPARATION — PART 1

# CORE JAVA

This is where we start properly. Don't memorize one-line definitions. At 4–9 YOE, the interviewer can take one answer and drill 4 levels down.

---

# 1. OOP

### Q1. What are the four pillars of OOP?

**Answer:**

The four pillars are:

* **Encapsulation** — bundling state and behavior and controlling access.
* **Abstraction** — exposing what an object does while hiding implementation details.
* **Inheritance** — deriving a new class from an existing class.
* **Polymorphism** — same interface/reference behaving differently depending on the actual object.

Example:

```java
interface Payment {
    void pay();
}

class CardPayment implements Payment {
    public void pay() {
        System.out.println("Card");
    }
}

class UpiPayment implements Payment {
    public void pay() {
        System.out.println("UPI");
    }
}

Payment p = new CardPayment();
p.pay();
```

The reference is `Payment`, but runtime behavior comes from `CardPayment`.

### Follow-up:

**What type of polymorphism does Java support?**

* Compile-time → method overloading
* Runtime → method overriding

Java doesn't support C++-style operator overloading for user-defined types.

---

# 2. Abstraction vs Encapsulation

### Q2. What's the difference?

**Abstraction** answers:

> What should the caller know?

**Encapsulation** answers:

> How do I protect and control the object's state?

Example:

```java
class BankAccount {

    private double balance;

    public void deposit(double amount) {
        if (amount > 0) {
            balance += amount;
        }
    }

    public double getBalance() {
        return balance;
    }
}
```

`balance` is encapsulated.

The user doesn't need to know how deposit validation works — that's abstraction.

### Follow-up

**Can abstraction exist without encapsulation?**

Yes, conceptually. They solve different problems, although good designs often use both.

---

# 3. `==` vs `equals()`

### Q3. Difference?

For primitives:

```java
int a = 10;
int b = 10;

a == b
```

compares values.

For objects:

```java
String a = new String("hello");
String b = new String("hello");

a == b       // false
a.equals(b)  // true
```

`==` compares object references.

`equals()` compares logical equality if the class overrides it.

### Follow-up

**What happens if a class doesn't override `equals()`?**

`Object.equals()` effectively behaves like reference equality.

---

# 4. `equals()` and `hashCode()`

### Q4. Explain the contract.

If:

```java
a.equals(b) == true
```

then:

```java
a.hashCode() == b.hashCode()
```

**must** be true.

But the reverse isn't required.

Two unequal objects can have the same hash code.

```text
equals true
    ↓
hashCode MUST be same

hashCode same
    ↓
equals may be true OR false
```

### Why is this important?

Because hash-based collections use the hash code to locate a bucket and `equals()` to determine equality among candidates.

### Follow-up

What happens if you override `equals()` but not `hashCode()`?

You can break `HashMap`, `HashSet`, etc.

Example:

```java
Set<Employee> set = new HashSet<>();

set.add(new Employee(1, "John"));

set.contains(new Employee(1, "John"));
```

If `equals()` says they're equal but hash codes differ, the lookup can go to a different bucket and fail.

---

# 5. `Comparable` vs `Comparator`

This has been explicitly reported in recent TCS Java interview experiences. ([LinkedIn][2])

### Q5. Difference?

### Comparable

Defines the object's **natural ordering**.

```java
class Employee implements Comparable<Employee> {

    private int salary;

    @Override
    public int compareTo(Employee other) {
        return Integer.compare(this.salary, other.salary);
    }
}
```

Usage:

```java
Collections.sort(employees);
```

### Comparator

Defines an **external/custom ordering**.

```java
employees.sort(
    Comparator.comparing(Employee::getName)
);
```

You can have multiple comparators:

```java
Comparator<Employee> bySalary = ...
Comparator<Employee> byName = ...
Comparator<Employee> byAge = ...
```

### Follow-up

**Can Comparable and Comparator both be used for the same class?**

Yes.

Comparable defines the default ordering; Comparator can override it for a particular use case.

---

# 6. HashMap — explain internally

🔥 **Very important**

### Q6. How does HashMap work internally?

At a high level:

```text
key
 ↓
hashCode()
 ↓
hash spreading
 ↓
bucket index
 ↓
bucket
 ↓
compare hash
 ↓
equals()
 ↓
value
```

Conceptually:

```java
index = (n - 1) & hash
```

where `n` is the table size.

HashMap uses an array of buckets.

A bucket can contain multiple entries because different keys can produce the same bucket.

### Collision

Suppose:

```text
key A → bucket 5
key B → bucket 5
key C → bucket 5
```

The entries have collided.

Modern Java can convert a heavily-collided bucket from a linked structure into a balanced tree under appropriate conditions.

### Important thresholds

Common interview values:

* Default load factor: **0.75**
* Treeification threshold: **8**
* Untreeification threshold: **6**
* Treeification also requires sufficient table capacity; commonly **64**

Don't present these as “HashMap always becomes a tree at exactly 8”; the implementation has additional conditions.

---

# 7. Why load factor 0.75?

### Q7. Why doesn't HashMap use 1.0?

Because there's a tradeoff.

Higher load factor:

```text
less memory
+
more collisions
+
potentially slower lookup
```

Lower load factor:

```text
more memory
+
fewer collisions
+
potentially faster lookup
```

`0.75` is a practical compromise used by Java's implementation.

---

# 8. What happens when HashMap resizes?

### Q8.

When:

```text
size > capacity × loadFactor
```

the table is resized.

Typically capacity doubles.

Entries are redistributed/repositioned based on the new table size.

Example:

```text
16 → 32 → 64 → 128
```

### Follow-up:

**Why is HashMap capacity generally a power of two?**

It allows efficient bucket calculation using:

```java
(hash & (n - 1))
```

instead of a more expensive modulo operation.

---

# 9. Mutable HashMap key

🔥 Excellent senior follow-up.

### Q9. What happens if a key is mutable?

Bad things.

```java
Map<Employee, String> map = new HashMap<>();

Employee e = new Employee(1);

map.put(e, "John");

e.setId(2);

map.get(e);
```

If `hashCode()` depends on `id`, the key may now map to a different bucket.

The entry is physically still in the old bucket.

So the map may not find it.

### Rule

Keys should ideally be **immutable**.

Good examples:

```text
String
Integer
Long
UUID
records with immutable components
```

---

# 10. HashMap vs Hashtable vs ConcurrentHashMap

### Q10.

|               | HashMap              | Hashtable              | ConcurrentHashMap       |
| ------------- | -------------------- | ---------------------- | ----------------------- |
| Thread-safe   | No                   | Yes                    | Yes                     |
| Null key      | Yes                  | No                     | No                      |
| Null value    | Yes                  | No                     | No                      |
| Modern choice | Yes, single-threaded | Generally legacy       | Concurrent access       |
| Concurrency   | None                 | coarse synchronization | much better concurrency |

`ConcurrentHashMap` supports atomic operations such as:

```java
putIfAbsent()
computeIfAbsent()
compute()
merge()
```

### Follow-up

Why not:

```java
if (!map.containsKey(key)) {
    map.put(key, value);
}
```

Because the operation isn't atomic.

Another thread can modify the map between the two operations.

Use:

```java
map.putIfAbsent(key, value);
```

---

# 11. ArrayList vs LinkedList

### Q11.

`ArrayList` is backed by a dynamically growing array.

Advantages:

* fast random access
* cache-friendly
* generally preferred for most list workloads

`LinkedList` is node-based.

Insertion/removal can be efficient **once you already have the node/iterator position**, but finding that position can still be O(n).

Therefore:

> “LinkedList insertion is always O(1)” is an incomplete answer.

### Follow-up:

Why is ArrayList often faster even for operations where both are theoretically O(n)?

Because contiguous memory provides better CPU cache locality and fewer object allocations/indirections.

---

# 12. HashSet vs LinkedHashSet vs TreeSet

### Q12.

**HashSet**

```text
hash-based
no ordering guarantee
average O(1) lookup
```

**LinkedHashSet**

```text
hash-based
maintains insertion order
```

**TreeSet**

```text
tree-based
sorted order
O(log n) operations
```

TreeSet uses ordering based on `Comparable` or `Comparator`.

### Follow-up

If `compareTo()` returns `0`, TreeSet considers the elements equivalent for set purposes, even if `equals()` says otherwise.

That's an important trap.

---

# 13. String immutability

🔥

### Q13. Why is String immutable?

Several reasons:

1. String pool can safely reuse instances.
2. Hash code can be cached.
3. Strings can safely be shared between threads.
4. Security-sensitive values shouldn't unexpectedly change.
5. It simplifies reasoning about String behavior.

Example:

```java
String s = "hello";
s.concat(" world");
```

The original String doesn't change.

```java
s = s.concat(" world");
```

creates/assigns a new String.

---

# 14. String vs StringBuilder vs StringBuffer

### Q14.

**String**

Immutable.

**StringBuilder**

Mutable and not synchronized.

Use for single-threaded string construction.

**StringBuffer**

Mutable and synchronized.

Usually less preferred unless synchronization semantics are actually required.

### Follow-up

Why is this bad?

```java
String result = "";

for (...) {
    result += value;
}
```

Repeated concatenation can create many intermediate String objects.

Prefer:

```java
StringBuilder sb = new StringBuilder();

for (...) {
    sb.append(value);
}
```

---

# 15. Immutable class — how do you create one?

🔥

### Q15.

Typical rules:

```java
final class Employee {

    private final int id;
    private final String name;

    public Employee(int id, String name) {
        this.id = id;
        this.name = name;
    }

    public int getId() {
        return id;
    }

    public String getName() {
        return name;
    }
}
```

Important:

* make class `final` if you want to prevent subclass-based mutability issues
* fields private/final
* initialize through constructor
* no setters
* defensive copies for mutable fields

For:

```java
private final List<String> skills;
```

don't simply expose it:

```java
return skills;
```

Use an immutable/unmodifiable representation or defensive copy depending on requirements.

---

# 16. Abstract class vs Interface

### Q16.

An interface defines a contract/capability.

An abstract class can provide shared state and implementation.

Modern Java interfaces can contain:

```java
default methods
static methods
private methods
```

but they still don't serve as a normal replacement for an abstract class when you need instance state and constructor-based initialization.

### Follow-up

**Can an interface have variables?**

Yes, but interface fields are implicitly:

```java
public static final
```

---

# 17. Default method — why introduced?

### Q17.

Default methods allow interfaces to evolve without immediately breaking every existing implementation.

Example:

```java
interface Vehicle {
    void drive();

    default void start() {
        System.out.println("Starting");
    }
}
```

Java 8 introduced this capability.

### Follow-up

What if two interfaces provide the same default method?

The implementing class must resolve the conflict.

```java
InterfaceA.super.method();
```

can explicitly invoke one implementation.

---

# 18. Overloading vs overriding

### Q18.

**Overloading**

Same method name, different parameter list.

Resolved at compile time.

```java
add(int a, int b)
add(double a, double b)
```

**Overriding**

Subclass provides implementation of inherited method.

Resolved dynamically at runtime.

```java
Animal a = new Dog();
a.sound();
```

calls `Dog.sound()`.

### Important trap

Return type alone cannot overload a method.

```java
int test()
double test()
```

is invalid.

---

# 19. Can static methods be overridden?

### Q19.

No.

Static methods belong to the class rather than being dynamically dispatched based on the runtime object.

They can be **hidden**, not overridden.

---

# 20. Can private methods be overridden?

No.

They're not inherited in the normal polymorphic sense.

A subclass method with the same signature is a separate method.

---

# 21. final / finally / finalize

### Q21.

`final`

* final variable → cannot be reassigned
* final method → cannot be overridden
* final class → cannot be extended

`finally`

Used for cleanup associated with exception handling.

```java
try {
   ...
} finally {
   ...
}
```

`finalize()`

Old GC-related cleanup mechanism.

It is deprecated/obsolete and should not be used for resource management.

Use:

```java
try-with-resources
```

instead.

---

# 22. Checked vs unchecked exceptions

### Q22.

**Checked exceptions**

Compiler requires handling/declaring them.

Examples:

```text
IOException
SQLException
```

**Unchecked exceptions**

Subclasses of `RuntimeException`.

Examples:

```text
NullPointerException
IllegalArgumentException
IllegalStateException
```

### Senior follow-up

Should every application exception be checked?

No.

Modern application code commonly uses unchecked exceptions for programming/business failures and handles them at appropriate boundaries, particularly REST/API boundaries.

The decision should reflect whether callers can reasonably recover from the condition.

---

# 23. `throw` vs `throws`

### Q23.

`throw` actually throws an exception:

```java
throw new IllegalArgumentException();
```

`throws` declares that a method may propagate an exception:

```java
void read() throws IOException {
}
```

---

# 24. Try-with-resources

### Q24.

Used for resources implementing `AutoCloseable`.

```java
try (Connection connection = dataSource.getConnection()) {
    ...
}
```

The resource is automatically closed.

### Follow-up

What happens if both the main operation and `close()` throw exceptions?

The close exception becomes a **suppressed exception** and can be retrieved using:

```java
exception.getSuppressed()
```

That's a very good senior-level detail.

---

# 25. Cloneable and `clone()`

### Q25. Difference?

`Cloneable` is a marker interface.

It doesn't define a cloning method.

`Object.clone()` performs a field-level copy when used appropriately.

Typical problems:

* shallow copy
* awkward API
* mutable nested objects
* `CloneNotSupportedException`

For application code, copy constructors/factory methods are often clearer.

Example:

```java
Employee(Employee other) {
    this.id = other.id;
    this.name = other.name;
}
```

---

# 26. Shallow copy vs deep copy

### Q26.

Suppose:

```java
class Employee {
    String name;
    Address address;
}
```

Shallow copy:

```text
Employee A
   |
 Address X

Employee B
   |
 Address X
```

Both point to the same Address.

Deep copy:

```text
Employee A → Address X
Employee B → Address Y
```

Nested mutable state is independently copied.

---

# 27. Java pass-by-value

🔥 Very common trap.

### Q27. Is Java pass-by-value or pass-by-reference?

**Java is always pass-by-value.**

For objects, the value being passed is the **reference value**.

```java
void change(Employee e) {
    e.setName("Bob");
}
```

The object's state can change.

But:

```java
void change(Employee e) {
    e = new Employee();
}
```

doesn't change what the caller's variable references.

Because the reference itself was copied.

---

# 28. Why is Java not purely object-oriented?

Primitive types:

```text
int
long
double
char
boolean
...
```

aren't objects.

Java provides wrapper classes:

```text
Integer
Long
Double
...
```

and autoboxing/unboxing.

---

# 29. Autoboxing trap

### Q29.

```java
Integer a = 100;
Integer b = 100;

System.out.println(a == b);
```

Typically true because common boxed integer values are cached.

But:

```java
Integer a = 200;
Integer b = 200;

a == b
```

typically isn't true.

**Never use `==` for wrapper value equality.**

Use:

```java
a.equals(b)
```

or appropriate value comparison.

---

# 30. Functional interface

🔥 Java 8

### Q30.

A functional interface has exactly **one abstract method**.

Examples:

```java
Runnable
Callable
Comparator
Function
Predicate
Consumer
Supplier
```

It may still contain default and static methods.

```java
@FunctionalInterface
interface Calculator {
    int calculate(int a, int b);
}
```

Then:

```java
Calculator c = (a, b) -> a + b;
```

---

# 31. Function vs Predicate vs Consumer vs Supplier

### Q31.

This is worth memorizing.

```text
Function<T,R>
T → R

Predicate<T>
T → boolean

Consumer<T>
T → void

Supplier<T>
() → T
```

Example:

```java
Function<String, Integer> length = String::length;

Predicate<Integer> positive = x -> x > 0;

Consumer<String> print = System.out::println;

Supplier<String> value = () -> "hello";
```

---

# 32. Lambda and effectively final

### Q32.

Why can't a lambda freely modify a local variable?

```java
int count = 0;

list.forEach(x -> count++); // compilation problem
```

Local variables captured by lambdas must be final or **effectively final**.

This relates to how local variables are captured rather than being shared mutable local stack variables.

---

# 33. Stream vs Collection

### Q33.

Collection stores/manages data.

Stream represents a **pipeline for processing data**.

```java
employees.stream()
    .filter(e -> e.getSalary() > 100000)
    .map(Employee::getName)
    .toList();
```

Stream doesn't normally modify the underlying collection.

---

# 34. Intermediate vs terminal operations

Intermediate:

```text
filter
map
flatMap
sorted
distinct
limit
```

Terminal:

```text
collect
toList
reduce
count
forEach
findFirst
findAny
anyMatch
```

Intermediate operations are generally lazy.

---

# 35. `map()` vs `flatMap()`

🔥

Suppose:

```java
List<Employee>
```

and each employee has:

```java
List<String> skills
```

`map()` gives:

```text
Stream<List<String>>
```

`flatMap()` gives:

```text
Stream<String>
```

Example:

```java
employees.stream()
    .flatMap(e -> e.getSkills().stream())
    .distinct()
    .toList();
```

Think:

> `map` transforms one element into one result.

> `flatMap` transforms one element into multiple results and flattens them.

---

# 36. `orElse()` vs `orElseGet()`

🔥

```java
optional.orElse(expensiveMethod());
```

The argument may be evaluated even if the Optional contains a value.

```java
optional.orElseGet(() -> expensiveMethod());
```

is lazy.

This is a very common Java 8 interview follow-up.

---

# 37. Why should Optional generally not be an entity field?

JPA entities and serialization frameworks often expect ordinary fields/getters/setters and `Optional` is primarily intended as a return-type abstraction.

Prefer:

```java
Optional<Employee> findEmployee(...)
```

rather than:

```java
Optional<Employee> employee;
```

as a persistent entity field.

---

# 38. `findFirst()` vs `findAny()`

`findFirst()` respects encounter order for ordered streams.

`findAny()` permits the implementation more freedom, especially with parallel streams.

If you don't care which matching element you get, `findAny()` can be more suitable.

---

# 39. Parallel stream — should you use it?

🔥 Senior follow-up.

Don't say:

> “Parallel stream is faster.”

That's wrong.

It can help for suitable **CPU-bound, sufficiently large, independent workloads**.

It can hurt because of:

* overhead
* synchronization
* shared resources
* ordering requirements
* blocking I/O
* common ForkJoinPool interaction

Never blindly use:

```java
.parallelStream()
```

for database/network calls.

---

# 40. Serialization

### Q40. What is Java serialization?

Converting an object's state into a byte representation that can later be reconstructed.

```java
class Employee implements Serializable {
}
```

Deserialization reconstructs the object.

### Why was it used?

Historically for:

* object persistence
* communication
* caching
* session replication

But native Java serialization has security and maintenance concerns and is generally avoided for modern external data interchange.

For APIs, formats such as JSON/Avro/Protobuf are generally preferred depending on requirements.

---

# 41. `serialVersionUID`

### Q41.

It identifies the serialization version of a class.

```java
private static final long serialVersionUID = 1L;
```

During deserialization, Java can compare the serialized object's class version with the current class version.

Mismatch can result in:

```text
InvalidClassException
```

### Why explicitly define it?

Otherwise Java may generate one based on class details, and seemingly harmless class changes can alter it.

---

# 42. `transient`

### Q42.

A `transient` field isn't serialized by default.

```java
private transient String password;
```

After deserialization, it gets the default value unless custom logic restores it.

Typical use:

```text
passwords
temporary state
derived values
non-serializable fields
```

---

# 43. Are static fields serialized?

### Q43.

No — serialization operates on **object state**, while static fields belong to the class.

Similarly, transient fields are not serialized by default.

---

# 44. Custom serialization

### Q44.

You can customize serialization using methods such as:

```java
private void writeObject(ObjectOutputStream out)
private void readObject(ObjectInputStream in)
```

For example, you may want to transform or protect certain state.

But don't confuse this with encryption. Java serialization itself does **not** provide confidentiality.

---

# 45. Why is Java native serialization considered risky?

Important senior answer:

* deserialization can instantiate object graphs
* historically it has been associated with gadget-chain vulnerabilities
* serialized forms are tightly coupled to Java class structure
* interoperability is poor
* version evolution can be difficult

Therefore don't blindly deserialize untrusted bytes.

---

# 46. JVM memory — explain the major areas

🔥

### Q46.

Simplified:

```text
JVM
├── Heap
│   ├── objects
│   └── arrays
│
├── Thread Stacks
│   └── stack frames/local variables
│
├── Metaspace
│   └── class metadata
│
└── Native memory
```

Heap is shared between threads.

Each thread has its own stack.

### Follow-up

What causes:

```text
StackOverflowError
```

Usually excessive stack depth, commonly recursion.

What causes:

```text
OutOfMemoryError
```

Various things, such as heap exhaustion, metaspace exhaustion, native-memory issues, depending on the specific error.

---

# 47. Heap vs Stack

### Q47.

```java
void test() {
    Employee e = new Employee();
}
```

The object itself is generally allocated on the heap.

The local reference `e` exists in the method's stack frame.

Don't say:

> “The object is always on heap and reference is always on stack.”

JIT optimizations such as escape analysis can affect actual allocation behavior.

For interview purposes, the conceptual model is enough unless they ask about JVM optimization.

---

# 48. Garbage Collection

### Q48.

GC identifies objects that are no longer reachable and reclaims their memory.

Modern collectors include:

```text
G1
ZGC
Shenandoah
```

depending on JVM/version and requirements.

Don't memorize:

> “GC removes objects when they become null.”

That's incorrect.

GC is based on reachability from GC roots.

---

# 49. What are GC roots?

Examples include:

* active thread references
* static references
* JNI references
* certain JVM-internal references

Objects reachable from roots are considered live.

---

# 50. How would you investigate an OutOfMemoryError?

🔥 Senior production question.

Answer systematically:

```text
1. Identify exact OOM type
2. Check heap usage
3. Check GC logs/metrics
4. Capture heap dump
5. Analyze dominator tree / retained objects
6. Identify growing collection/cache/thread-local
7. Check traffic and deployment changes
8. Fix root cause
9. Validate under load
```

Common causes:

```text
unbounded cache
large collections
memory leak through static references
ThreadLocal misuse
large object retention
excessive buffering
```

---

# 51. What is a memory leak in Java?

Java has GC, but memory leaks are still possible.

A memory leak means objects are **still reachable but no longer logically needed**.

Example:

```java
static List<Object> cache = new ArrayList<>();
```

If items continuously accumulate, GC can't collect them because the static list still references them.

---

# 52. Class loading

### Q52.

Simplified class-loading flow:

```text
Loading
   ↓
Linking
   ├── Verification
   ├── Preparation
   └── Resolution
   ↓
Initialization
```

Class loaders load classes.

Important loaders include:

* Bootstrap
* Platform
* Application/System

Modern Java uses a hierarchy rather than the old simplistic “bootstrap → extension → application” terminology.

---

# 53. What is classloader delegation?

### Q53.

Typically a class loader delegates upward first.

Conceptually:

```text
Application
    ↓
Platform
    ↓
Bootstrap
```

This prevents application code from simply replacing core Java classes such as classes under protected JDK namespaces.

---

# 54. Deadlock

### Q54.

Two threads:

```text
Thread 1:
lock A
wait for B

Thread 2:
lock B
wait for A
```

Neither can proceed.

Conditions associated with classic deadlock include:

* mutual exclusion
* hold and wait
* no preemption
* circular wait

### Prevention

Consistent lock ordering is one common strategy.

---

# 55. Race condition

### Q55.

A race occurs when the result depends on timing/interleaving of concurrent operations.

Example:

```java
count++;
```

is not a single atomic operation.

Conceptually:

```text
read
modify
write
```

Two threads can overwrite each other's updates.

Use appropriate synchronization:

```java
synchronized
AtomicInteger
Lock
concurrent collection
```

depending on the problem.

---

# 56. `volatile` vs `synchronized`

🔥

`volatile` primarily provides:

* visibility
* ordering guarantees around volatile accesses

It does **not** make compound operations atomic.

```java
volatile int count;

count++;
```

is still not atomic.

`synchronized` provides:

* mutual exclusion
* visibility/happens-before guarantees

So:

```text
volatile → visibility/order
synchronized → mutual exclusion + visibility
```

---

# 57. Happens-before

### Q57.

Happens-before is a Java Memory Model relationship guaranteeing that certain actions are visible and ordered relative to other actions.

Examples:

```text
unlock → subsequent lock
volatile write → subsequent volatile read
Thread.start() → actions in started thread
actions in thread → successful Thread.join()
```

This is one of the concepts that separates a senior Java answer from a superficial concurrency answer.

---

# 58. AtomicInteger vs synchronized

### Q58.

For a simple counter:

```java
AtomicInteger counter = new AtomicInteger();

counter.incrementAndGet();
```

can be appropriate.

It uses atomic operations rather than locking the whole critical section.

But if you need to atomically update **multiple pieces of related state**, `AtomicInteger` alone isn't enough.

You may need:

```text
synchronized
Lock
transaction
CAS-based design
```

depending on the situation.

---

# 59. ExecutorService

### Q59. Why use ExecutorService instead of creating threads manually?

Instead of:

```java
new Thread(...).start();
```

you manage a pool:

```java
ExecutorService executor =
    Executors.newFixedThreadPool(10);

executor.submit(task);
```

Advantages:

* thread reuse
* controlled concurrency
* lifecycle management
* task abstraction
* Future support

But don't blindly use `Executors.newFixedThreadPool()` in every production application. Explicit `ThreadPoolExecutor` configuration can provide better control over queue capacity, rejection policy, and resource usage.

---

# 60. Callable vs Runnable

```text
Runnable
  → no result
  → cannot directly throw checked exception

Callable<T>
  → returns T
  → can throw Exception
```

Example:

```java
Future<Integer> future =
    executor.submit(() -> calculate());
```

---

# 61. Future vs CompletableFuture

### Future

You can:

```java
future.get();
```

but composition is awkward.

### CompletableFuture

Allows asynchronous composition:

```java
CompletableFuture
    .supplyAsync(this::getCustomer)
    .thenCompose(this::getOrders)
    .thenApply(this::transform)
    .exceptionally(this::handleError);
```

This becomes extremely important when we reach **async programming**.

---

# 62. CountDownLatch vs CyclicBarrier

### CountDownLatch

One-time countdown.

```text
main thread waits
     ↑
worker 1 ─┐
worker 2 ─┼── countDown()
worker 3 ─┘
```

Once count reaches zero, waiting threads proceed.

### CyclicBarrier

A group of threads waits for each other at a barrier.

And the barrier can be reused.

---

# 63. ReentrantLock vs synchronized

`ReentrantLock` provides more explicit control:

* `tryLock()`
* timed lock attempts
* interruptible lock acquisition
* multiple `Condition`s

`synchronized` is simpler and often preferable when those features aren't required.

---

# 64. What would you choose for a thread-safe cache?

Don't answer automatically:

> ConcurrentHashMap.

Ask:

```text
Do we need TTL?
Eviction?
Maximum size?
Distributed access?
Persistence?
Cache stampede protection?
Serialization?
```

If simple in-process concurrent map:

```java
ConcurrentHashMap
```

may be enough.

For distributed caching:

```text
Redis
```

may be appropriate.

---

# 65. Singleton pattern — thread-safe implementation

A strong answer:

```java
public final class Singleton {

    private Singleton() {}

    private static class Holder {
        private static final Singleton INSTANCE =
            new Singleton();
    }

    public static Singleton getInstance() {
        return Holder.INSTANCE;
    }
}
```

The initialization-on-demand holder idiom uses class initialization guarantees.

### Follow-up

What about:

```java
enum Singleton {
    INSTANCE
}
```

Enum singletons are also robust and provide serialization protections.

---

# 66. One important interview coding question

### Q66. Find the highest-paid employee in each department using Streams.

```java
Map<String, Optional<Employee>> result =
    employees.stream()
        .collect(Collectors.groupingBy(
            Employee::getDepartment,
            Collectors.maxBy(
                Comparator.comparing(Employee::getSalary)
            )
        ));
```

If they want the employee rather than `Optional`:

```java
Map<String, Employee> result =
    employees.stream()
        .collect(Collectors.groupingBy(
            Employee::getDepartment,
            Collectors.collectingAndThen(
                Collectors.maxBy(
                    Comparator.comparing(Employee::getSalary)
                ),
                Optional::orElseThrow
            )
        ));
```

### Follow-up

“What if two employees have the same highest salary?”

Then `maxBy` returns one employee. If the requirement is **all employees tied for maximum**, the solution changes.

That's exactly the kind of clarification a senior interviewer may expect.

---

# 67. Another coding question

### Q67. First non-repeated character

```java
String s = "swiss";

Character result =
    s.chars()
     .mapToObj(c -> (char) c)
     .collect(
         Collectors.groupingBy(
             Function.identity(),
             LinkedHashMap::new,
             Collectors.counting()
         )
     )
     .entrySet()
     .stream()
     .filter(e -> e.getValue() == 1)
     .map(Map.Entry::getKey)
     .findFirst()
     .orElse(null);
```

Why `LinkedHashMap`?

Because we need to preserve encounter order.

---

# 68. Another paper-coding classic

### Q68. Remove duplicates while preserving order.

```java
List<Integer> result =
    numbers.stream()
           .distinct()
           .toList();
```

Or:

```java
new ArrayList<>(new LinkedHashSet<>(numbers));
```

### Follow-up

Time complexity?

Generally O(n) expected for hashing-based approaches.

---

# 69. Two Sum

### Q69.

Don't use nested loops unless asked for brute force.

Use a HashMap:

```java
Map<Integer, Integer> map = new HashMap<>();

for (int i = 0; i < nums.length; i++) {

    int complement = target - nums[i];

    if (map.containsKey(complement)) {
        return new int[] {
            map.get(complement), i
        };
    }

    map.put(nums[i], i);
}
```

Average:

```text
Time: O(n)
Space: O(n)
```

---

# 70. Reverse a linked list

### Q70.

Paper coding:

```java
Node prev = null;
Node curr = head;

while (curr != null) {
    Node next = curr.next;
    curr.next = prev;
    prev = curr;
    curr = next;
}

return prev;
```

You should be able to write this without thinking.

---

# 71. Detect cycle in linked list

Floyd's algorithm:

```java
Node slow = head;
Node fast = head;

while (fast != null && fast.next != null) {

    slow = slow.next;
    fast = fast.next.next;

    if (slow == fast) {
        return true;
    }
}

return false;
```

Time:

```text
O(n)
```

Space:

```text
O(1)
```

---

# 72. LRU Cache — know the design

🔥 Senior coding/design question.

Required operations:

```text
get() → O(1)
put() → O(1)
```

Classic implementation:

```text
HashMap
+
Doubly Linked List
```

HashMap provides direct lookup.

Doubly linked list maintains recency.

Or Java's:

```java
LinkedHashMap
```

can implement an LRU cache conveniently using access-order.

---

# What I want you to know from Core Java

For a **4–9 YOE interview**, don't stop at definitions. You should be able to survive this chain:

```text
HashMap
 ↓
hashCode / equals
 ↓
collision
 ↓
treeification
 ↓
resize
 ↓
load factor
 ↓
mutable key
 ↓
ConcurrentHashMap
 ↓
atomic operations
 ↓
race condition
 ↓
happens-before
```

Similarly:

```text
Stream
 ↓
lazy evaluation
 ↓
map vs flatMap
 ↓
parallel stream
 ↓
ForkJoinPool
 ↓
blocking I/O problem
```

And:

```text
Spring @Transactional
 ↓
proxy
 ↓
self invocation
 ↓
propagation
 ↓
rollback
 ↓
DB transaction
 ↓
distributed transaction
 ↓
Saga
```

That is the **depth progression** I'll use for the remaining topics.

---

## Next section: Java 8 → 11 → 17 → 21

I'll cover the important features **properly but without wasting time on obscure language changes**, including:

* Java 8 Streams in depth
* Lambda/function interfaces
* Optional
* default/static interface methods
* method references
* collectors
* parallel streams
* Java 11 important additions
* Java 17 records, sealed classes, pattern matching, text blocks, switch expressions
* Java 21 **virtual threads**, pattern matching, record patterns, sequenced collections
* **virtual threads vs CompletableFuture vs platform threads**
* the senior-level question: **“If virtual threads are cheap, why do we still need async programming?”**

Then we'll move into **Spring Core → Spring Boot internals → REST → JPA/Hibernate → Security → AOP → Transactions**, where the questions become much more scenario-heavy.

And importantly, recent TCS reports specifically support spending serious time on **Spring Boot internals, `@Transactional`, Kafka failure handling, microservice failures, SQL/indexing and concurrency**, rather than treating them as surface-level topics. ([LinkedIn][1])

[1]: https://www.linkedin.com/posts/saikumar-medishetti-604500121_java-springboot-microservices-activity-7450934872723955712-akSr?utm_source=chatgpt.com "Java Developer Interview Questions at TCS HYD | Saikumar Medishetti posted on the topic | LinkedIn"
[2]: https://www.linkedin.com/posts/akash-shet_interview-questions-asked-for-3-years-experienced-activity-7421043681983533057-nsJ3?utm_source=chatgpt.com "TCS Java Developer Interview Questions: Java 8-21 Features and Best Practices | Akash Shet posted on the topic | LinkedIn"
