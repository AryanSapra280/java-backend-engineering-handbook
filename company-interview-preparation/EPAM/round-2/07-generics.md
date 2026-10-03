Let's go — **Generics**. 🔥

This is one of those topics where you can look perfectly fine with Java syntax but get exposed quickly if the interviewer asks **why** things work the way they do.

# 7. Generics — deeply

We'll build this in this order:

```text
Generic class
    ↓
Generic method
    ↓
Type safety
    ↓
Invariance
    ↓
Wildcards
    ↓
? extends
    ↓
? super
    ↓
PECS
    ↓
Type erasure
    ↓
Bounded types
    ↓
Interview traps
```

---

# 1. Why do we need Generics?

Start with old Java code.

Before generics, you could have:

```java
List list = new ArrayList();

list.add("Java");
list.add(100);
list.add(true);
```

Everything can go into the list because the list effectively deals with `Object`.

Then:

```java
String value = (String) list.get(1);
```

What happens?

`list.get(1)` is actually:

```text
100
```

but you're trying to cast it to:

```text
String
```

So you get:

```text
ClassCastException
```

The problem is that the compiler didn't know what type the list was supposed to contain.

---

# 2. Generics solve this

```java
List<String> list = new ArrayList<>();

list.add("Java");
list.add("Spring");
```

Now:

```java
list.add(100);
```

is rejected at **compile time**.

That's the first major benefit:

> **Generics move many type errors from runtime to compile time.**

And:

```java
String value = list.get(0);
```

doesn't require an explicit cast.

---

# 3. What does `<String>` mean?

When you write:

```java
List<String>
```

you're saying:

> This List is parameterized with `String`.

Conceptually:

```text
List<T>

T = String

therefore:

List<String>
```

`T` is called a **type parameter**.

---

# 4. Generic class

You can define your own generic class:

```java
class Box<T> {

    private T value;

    public void set(T value) {
        this.value = value;
    }

    public T get() {
        return value;
    }
}
```

Now:

```java
Box<String> box = new Box<>();

box.set("Java");

String value = box.get();
```

Or:

```java
Box<Integer> box = new Box<>();

box.set(100);

Integer value = box.get();
```

Same class, different type.

---

# 5. What is `T`?

`T` isn't a real runtime class.

It's a **type parameter**.

Common naming conventions:

```text
T → Type
E → Element
K → Key
V → Value
N → Number
R → Result
```

For example:

```java
Map<K, V>
```

means:

```text
K = key type
V = value type
```

So:

```java
Map<String, Integer>
```

means:

```text
K = String
V = Integer
```

---

# 6. Generic methods

You can make the method itself generic.

```java
public static <T> void print(T value) {
    System.out.println(value);
}
```

Notice the unusual-looking:

```java
<T>
```

before the return type.

That's declaring the method's type parameter.

Usage:

```java
print("Java");
print(100);
print(true);
```

Java infers:

```text
T = String
T = Integer
T = Boolean
```

respectively.

---

# 7. Generic method returning a value

```java
public static <T> T identity(T value) {
    return value;
}
```

Then:

```java
String s = identity("Java");
Integer i = identity(100);
```

The compiler infers the type.

---

# 8. Why can't we just use Object?

You might ask:

> Why do I need generics? Can't I just use Object?

You could:

```java
Object value = "Java";
```

But then:

```java
String s = (String) value;
```

requires casting.

And:

```java
Object value = 100;

String s = (String) value;
```

compiles but fails at runtime.

Generics give the compiler more information.

```java
Box<String>
```

means:

```text
Only String is expected.
```

---

# 9. The BIG concept: invariance

This is where interviews get interesting.

You might logically think:

```text
String extends Object
```

therefore:

```text
List<String> extends List<Object>
```

But that's **false**.

Java generics are **invariant**.

So:

```java
List<String>
```

is NOT a subtype of:

```java
List<Object>
```

---

# 10. Why?

Imagine Java allowed this:

```java
List<String> strings = new ArrayList<>();

List<Object> objects = strings; // imagine this were allowed
```

Then:

```java
objects.add(100);
```

Now what happened?

`objects` and `strings` refer to the same list.

So:

```java
String s = strings.get(0);
```

could retrieve:

```text
100
```

That would violate type safety.

Therefore Java prohibits:

```java
List<String> → List<Object>
```

---

# 11. Very important distinction

This is valid:

```java
Object obj = "Java";
```

because:

```text
String IS-A Object
```

But:

```java
List<Object> list = new ArrayList<String>();
```

is invalid.

Because:

```text
List<String> IS-NOT-A List<Object>
```

This is one of the most important generics interview concepts.

---

# 12. Wildcards

Now Java gives us:

```java
?
```

which means:

> Unknown type.

For example:

```java
List<?> list;
```

means:

> A List of some unknown type.

It could be:

```java
List<String>
List<Integer>
List<Employee>
```

etc.

---

# 13. What can you do with `List<?>`?

You can safely read values as `Object`:

```java
List<?> list = List.of("Java", "Spring");

Object value = list.get(0);
```

Because whatever the unknown type is, every Java reference type is an Object.

But you generally cannot add arbitrary values:

```java
list.add("Java"); // compile error
list.add(100);    // compile error
```

Why?

Because Java doesn't know what the actual type is.

Suppose:

```java
List<?> list = new ArrayList<Integer>();
```

If Java allowed:

```java
list.add("Java");
```

you'd corrupt the `List<Integer>`.

---

# 14. `? extends`

Now:

```java
List<? extends Number>
```

means:

> A List of some unknown type that is Number or a subclass of Number.

It could be:

```java
List<Integer>
List<Double>
List<Long>
```

because:

```text
Integer extends Number
Double extends Number
Long extends Number
```

---

# 15. Reading from `? extends`

Example:

```java
List<? extends Number> numbers = List.of(1, 2, 3);

Number n = numbers.get(0);
```

This is safe.

Why?

Because regardless of the actual type:

```text
Integer
Double
Long
```

it's guaranteed to be a `Number`.

---

# 16. Why can't you add?

Consider:

```java
List<? extends Number> numbers;
```

The actual list could be:

```java
List<Integer>
```

If Java allowed:

```java
numbers.add(3.14);
```

you'd be putting a Double into a List<Integer>.

Therefore:

```java
numbers.add(10); // generally compile error
```

The only broadly safe value you can add is `null`.

Because `null` can represent any reference type.

---

# 17. The mental model for `extends`

Think:

```text
? extends Number
```

as:

```text
"I am a producer of Number values."
```

You primarily **READ** from it.

For example:

```java
double sum(List<? extends Number> numbers) {

    double total = 0;

    for (Number n : numbers) {
        total += n.doubleValue();
    }

    return total;
}
```

Works with:

```java
sum(List.of(1, 2, 3));
sum(List.of(1.5, 2.5));
```

---

# 18. `? super`

Now the opposite:

```java
List<? super Integer>
```

means:

> A List whose element type is Integer or one of Integer's supertypes.

Potentially:

```text
List<Integer>
List<Number>
List<Object>
```

---

# 19. What can you add?

This is the important part.

```java
List<? super Integer> list = new ArrayList<Number>();

list.add(10);
list.add(20);
```

This is safe.

Why?

Whatever the actual list type is:

```text
Integer
Number
Object
```

an Integer can always be stored in it.

---

# 20. What can you read?

This is the tricky part.

```java
Integer x = list.get(0);
```

is not safe.

Why?

The actual list could be:

```java
List<Object>
```

and the object could be something that's not an Integer.

So:

```java
Object x = list.get(0);
```

is safe.

---

# 21. The famous rule: PECS

You should absolutely remember:

> **PECS = Producer Extends, Consumer Super**

If a structure **produces** values for you to read:

```java
? extends T
```

If a structure **consumes** values that you're putting into it:

```java
? super T
```

---

# 22. Producer example

```java
double calculateTotal(
        List<? extends Number> numbers) {

    double total = 0;

    for (Number n : numbers) {
        total += n.doubleValue();
    }

    return total;
}
```

The list produces Numbers.

Therefore:

```java
? extends Number
```

---

# 23. Consumer example

```java
void addPayments(
        List<? super Payment> payments) {

    payments.add(new Payment());
}
```

The list consumes Payment objects.

Therefore:

```java
? super Payment
```

---

# 24. A very important example

Look at:

```java
public static <T> void copy(
        List<? super T> destination,
        List<? extends T> source) {

    for (T item : source) {
        destination.add(item);
    }
}
```

This is essentially the idea behind the signature of Java's collection-copy operations.

Why?

```text
source
  ↓
produces T
  ↓
? extends T


destination
  ↑
consumes T
  ↑
? super T
```

This is **PECS in action**.

---

# 25. Example with Employee

Suppose:

```java
class Employee {
}

class Developer extends Employee {
}
```

Now:

```java
List<Developer> developers;
List<Employee> employees;
List<Object> objects;
```

A producer:

```java
List<? extends Employee>
```

can refer to:

```text
List<Employee>
List<Developer>
```

A consumer:

```java
List<? super Developer>
```

can refer to:

```text
List<Developer>
List<Employee>
List<Object>
```

---

# 26. Bounded type parameter

Wildcards aren't the only way to restrict types.

You can write:

```java
public static <T extends Number>
double square(T value) {

    double n = value.doubleValue();

    return n * n;
}
```

Now `T` must extend `Number`.

Valid:

```java
square(10);
square(2.5);
```

Invalid conceptually:

```java
square("Java");
```

---

# 27. `extends` doesn't necessarily mean class inheritance

For example:

```java
<T extends Comparable<T>>
```

means T must satisfy the bound.

And a type parameter can have multiple bounds:

```java
<T extends Number & Comparable<T>>
```

The first bound can be a class; additional bounds can be interfaces.

---

# 28. Why use bounded generics?

Suppose:

```java
<T>
```

You can't assume T has:

```java
doubleValue()
```

But:

```java
<T extends Number>
```

lets the compiler know:

```text
T has Number's methods
```

So:

```java
T value;

value.doubleValue();
```

is valid.

This gives you both:

- type safety
- useful constraints

---

# 29. Type erasure — VERY important

Now we're getting into Java internals.

You write:

```java
List<String> names = new ArrayList<>();
```

You might think JVM maintains:

```text
ArrayList<String>
```

as a completely separate runtime type from:

```text
ArrayList<Integer>
```

That's not how traditional Java generics work.

Java implements generics primarily through **type erasure**.

Conceptually, much generic type information is removed/erased from runtime representation.

So:

```java
List<String>
List<Integer>
```

are both fundamentally:

```text
List
```

at runtime.

---

# 30. Why type erasure?

Java generics were introduced while maintaining compatibility with existing Java code.

Old Java code had:

```java
List list;
```

Generics were designed so that older bytecode and newer generic source could coexist.

The compiler provides type checking and inserts necessary casts.

---

# 31. Example of compiler-generated cast

You write:

```java
List<String> names = new ArrayList<>();

String name = names.get(0);
```

Conceptually, because the underlying List API returns Object, the compiler can insert a cast equivalent to:

```java
String name = (String) names.get(0);
```

The important difference is that generics ensure the unsafe situation is normally caught at compile time.

---

# 32. Can you do this?

```java
if (list instanceof List<String>) {
}
```

No.

You can't generally test a parameterized type like that because of type erasure.

You can do:

```java
if (list instanceof List<?>) {
}
```

because `List<?>` doesn't require knowing the erased element type.

---

# 33. Another type-erasure trap

You cannot overload methods solely by generic parameterization:

```java
void process(List<String> list) {
}

void process(List<Integer> list) {
}
```

Compilation error.

Why?

After erasure, both look conceptually like:

```java
void process(List list)
```

Same method signature.

---

# 34. Generic arrays

This is another classic question.

You cannot normally do:

```java
T[] array = new T[10];
```

inside generic code.

Why?

Because Java doesn't know the actual runtime component type due to type erasure.

This is one reason generic collections are generally preferred over generic arrays.

---

# 35. Raw types

This:

```java
List list = new ArrayList();
```

is a **raw type**.

Avoid it in modern Java unless you're dealing with legacy APIs for a specific reason.

Prefer:

```java
List<String> list = new ArrayList<>();
```

Raw types effectively turn off much of the generic type checking.

---

# 36. Diamond operator

Instead of:

```java
List<String> list =
    new ArrayList<String>();
```

you can write:

```java
List<String> list =
    new ArrayList<>();
```

The compiler infers the type arguments.

This is the **diamond operator**.

---

# 37. Generics in your backend code

You already use patterns like:

```java
PaymentRepository
    extends MongoRepository<Payment, String>
```

This is generics doing real work.

The interface is parameterized with:

```text
T = Payment
ID = String
```

So methods such as:

```java
Optional<Payment> findById(String id);
```

can be strongly typed.

You don't have to write:

```java
Payment p = (Payment) repository.findById(...);
```

This is exactly why generics matter in Spring Data.

---

# 38. Another real-world example

Suppose you have:

```java
public interface Repository<T, ID> {

    Optional<T> findById(ID id);

    void save(T entity);
}
```

Then:

```java
Repository<Payment, String>
```

means:

```text
T  = Payment
ID = String
```

while:

```java
Repository<Customer, Long>
```

means:

```text
T  = Customer
ID = Long
```

One generic abstraction can support multiple domain types safely.

---

# 39. Generic interface + bounded ID

You could even constrain it:

```java
interface Repository<T, ID extends Serializable> {
}
```

Now the ID must satisfy the bound.

Again, this lets the compiler enforce an API contract.

---

# 🔥 The five things you MUST know

If an interviewer asks you about Generics, your mental model should be:

### 1. Why?

```text
Compile-time type safety
↓
Less casting
↓
Reusable APIs
```

### 2. Invariance

```text
String extends Object

BUT

List<String> does NOT extend List<Object>
```

### 3. Wildcards

```text
<?>             unknown type

<? extends T>   producer / read

<? super T>     consumer / write
```

### 4. PECS

```text
Producer → Extends
Consumer → Super
```

### 5. Type erasure

```text
Generic type checking
       ↓
compile time

Runtime
       ↓
largely erased parameterized type information
```

---

# 🔥 Interview challenge

Don't answer immediately; reason through this:

```java
List<Integer> integers = new ArrayList<>();

List<? extends Number> producer = integers;

Number n = producer.get(0);

// producer.add(10);     // ?

List<? super Integer> consumer = new ArrayList<Number>();

consumer.add(10);

// Integer x = consumer.get(0); // ?
Object x = consumer.get(0);
```

The correct understanding is:

```text
producer.get(0)       → YES
producer.add(10)      → NO

consumer.add(10)      → YES
consumer.get(0)
as Integer            → NO
consumer.get(0)
as Object             → YES
```

And the reason is **not** "Java has weird syntax."

It's because the compiler must preserve type safety when the actual type hidden behind `?` is unknown.

---

## Next: Collections

And this is a **major section** for your interviews:

```text
Collection hierarchy
        ↓
List / Set / Queue / Map
        ↓
ArrayList
LinkedList
Vector
        ↓
HashSet / LinkedHashSet / TreeSet
        ↓
PriorityQueue / Deque
        ↓
HashMap
        ↓
TreeMap / LinkedHashMap
        ↓
Iterator
        ↓
fail-fast
        ↓
Concurrent collections
```

Then we'll go **very deep into `HashMap`**, including:

```text
hash()
 ↓
hash spreading
 ↓
bucket calculation
 ↓
collision
 ↓
equals()
 ↓
treeification
 ↓
resize
 ↓
load factor
 ↓
Java 7 vs Java 8+
 ↓
concurrent modification
 ↓
ConcurrentHashMap
```

That HashMap → ConcurrentHashMap chain is particularly important for the senior Java interviews you're preparing for.