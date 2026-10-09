Exactly. I understand the direction now. We should **optimize for EPAM Round 2 interview performance**, not try to finish the entire Java universe. 🔥

The key change is: for topics like **Exceptions, Concurrency, ExecutorService, CompletableFuture**, I’ll teach both:

1. **Java theory/internal working** — enough to answer direct questions.
2. **Production/microservice scenarios** — how it actually behaves in your Spring Boot application.
3. **Interviewer follow-ups** — especially “what happens if…?” questions.
4. **Implementation-level examples** — but not giant code exercises.

And yes, when we reach concurrency, I'll explicitly separate:

> **Java concurrency** vs **concurrency in a Spring Boot application**

because that's an important Senior Engineer discussion.

For example, you'll need to be comfortable explaining things like:

```text
HTTP Request
    ↓
Spring MVC thread
    ↓
Service
    ↓
Repository
    ↓
Database
```

and then:

```text
HTTP Request
    ↓
CompletableFuture / @Async
    ↓
Executor
    ↓
Worker thread
    ↓
DB / downstream service
```

and understand what happens to **exceptions, transactions, thread context, timeouts, cancellation, and propagation** in each case.

---

# Our revised remaining Java roadmap

### 🔥 Tier 1 — Must know

1. **Generics**
2. **Exception Handling**
   - Java exception hierarchy
   - checked vs unchecked
   - throw vs throws
   - try/catch/finally
   - custom exceptions
   - exception propagation
   - exception chaining
   - Spring REST exception handling
   - downstream exception propagation
   - DB/repository exceptions
   - transaction + exception interaction
   - global exception handling
   - what customer ultimately receives
3. **Multithreading & Concurrency**
4. **ExecutorService**
5. **Future / Callable**
6. **CompletableFuture**
7. **Spring concurrency**
   - `@Async`
   - TaskExecutor
   - thread pools
   - async exception handling
   - transaction/thread boundaries
   - MDC/correlation ID considerations
   - common production problems

### 🔥 Tier 2 — Important

8. **JVM Internals**
9. **Garbage Collection**
10. **Java Serialization / I/O**
11. **Testing**

### 🟡 Tier 3 — Selected only

12. **Design Patterns**
   - Singleton
   - Factory
   - Strategy
   - Builder
   - **Facade**
   - Observer
   - Template Method
   - Proxy
   - Adapter
   - Decorator

Not every GoF pattern. We'll focus on patterns that are realistically useful in Java/Spring interviews.

And you're right about **Facade**. It absolutely belongs in our selected list.

For example, a facade in a payment system could look conceptually like:

```text
PaymentFacade
      |
      +---- PaymentService
      |
      +---- FraudService
      |
      +---- WalletService
      |
      +---- NotificationService
```

The controller doesn't need to know all those subsystem interactions.

That's exactly the kind of practical explanation an interviewer likes.

### ❌ For now, skip

- Modern Java beyond what is needed
- Date/Time
- obscure collection classes
- deep Reflection
- obscure serialization internals
- every possible design pattern

Then after this:

**Spring concurrency → Microservices → Kafka → System/Schema Design → DSA revision.**

---

# Let's start Generics

We'll keep this **interview-focused**.

# 1. Why do Generics exist?

Imagine pre-generics Java:

```java
List list = new ArrayList();

list.add("Aryan");
list.add(100);

String name = (String) list.get(0);
String value = (String) list.get(1); // Runtime ClassCastException
```

The compiler doesn't know what the list contains.

Generics allow us to specify the type:

```java
List<String> names = new ArrayList<>();

names.add("Aryan");
names.add(100); // compilation error
```

So the first important answer:

> **Generics provide compile-time type safety and reduce the need for explicit casting.**

---

# 2. What actually happens internally?

This is a **very important EPAM question**.

Consider:

```java
List<String> names = new ArrayList<>();
```

You might think Java creates:

```text
ArrayList<String>
```

as a completely different runtime class from:

```text
ArrayList<Integer>
```

It doesn't.

Java uses **type erasure**.

Conceptually:

```java
List<String>
List<Integer>
```

become approximately:

```java
List
List
```

at runtime.

The generic type information is primarily used by the **compiler**.

For example:

```java
List<String> names = new ArrayList<>();

names.add("A");
String name = names.get(0);
```

The compiler effectively ensures type safety and inserts the necessary cast when retrieving.

Conceptually:

```java
String name = (String) names.get(0);
```

You don't normally write that cast yourself.

### Interview answer

> Java implements generics using type erasure. Generic type information is primarily enforced at compile time, and the JVM generally operates on the erased type at runtime.

---

# 3. Why can't we do this?

```java
List<String> list = new ArrayList<>();
List<Integer> list2 = new ArrayList<>();

if (list instanceof List<String>) {
}
```

Because at runtime:

```text
List<String>
List<Integer>
```

both effectively become:

```text
List
```

So Java cannot reliably distinguish them.

You can do:

```java
if (list instanceof List<?>) {
}
```

because `?` represents an unknown generic type.

---

# 4. Generic Class

You can create your own generic class:

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
box.set("Hello");

String value = box.get();
```

Or:

```java
Box<Integer> box = new Box<>();
box.set(100);

Integer value = box.get();
```

`T` is simply a **type parameter**.

It isn't restricted to `T`.

You could use:

```java
class Box<E>
class Box<K, V>
class Box<Request, Response>
```

Common conventions:

| Symbol | Meaning |
|---|---|
| T | Type |
| E | Element |
| K | Key |
| V | Value |
| R | Result/Return type |

---

# 5. Generic Method

You don't need a generic class to have a generic method.

```java
public static <T> T getFirst(List<T> list) {
    return list.get(0);
}
```

Notice:

```java
<T> T
```

The first `T` declares the type parameter.

The second `T` is the return type.

So:

```java
String s = getFirst(List.of("A", "B"));
```

and:

```java
Integer i = getFirst(List.of(10, 20));
```

Both work.

---

# 6. Bounded Generics

This is important.

Suppose we want to accept only numbers:

```java
<T extends Number>
```

Example:

```java
public static <T extends Number> double square(T value) {
    return value.doubleValue() * value.doubleValue();
}
```

Now:

```java
square(10);       // Integer
square(10.5);     // Double
square(100L);     // Long
```

But:

```java
square("hello");
```

doesn't compile.

Because:

```text
Integer ──┐
Double  ──┤
Long    ──┤──> Number
Float   ──┘
```

### Important syntax

```java
<T extends Number>
```

means:

> T must be Number or a subtype of Number.

And `extends` can also be used with interfaces:

```java
<T extends Comparable<T>>
```

---

# 7. The BIG Interview Topic — Wildcards

This is where interviewers usually start digging.

Suppose:

```java
List<Integer>
```

and:

```java
List<Number>
```

Are they related through inheritance?

**No.**

Even though:

```text
Integer extends Number
```

this does NOT mean:

```text
List<Integer> extends List<Number>
```

Otherwise this would become dangerous:

```java
List<Integer> integers = new ArrayList<>();
List<Number> numbers = integers; // not allowed
```

Because then:

```java
numbers.add(10.5); // Double
```

would put a Double into a list that is supposed to contain only Integers.

Hence Java uses **invariance**.

---

# 8. `? extends`

Suppose you want to **read** from a list of Number subclasses:

```java
List<? extends Number>
```

It could be:

```text
List<Integer>
List<Double>
List<Long>
```

Example:

```java
public static double sum(List<? extends Number> numbers) {

    double total = 0;

    for (Number n : numbers) {
        total += n.doubleValue();
    }

    return total;
}
```

You can pass:

```java
List<Integer>
List<Double>
List<Long>
```

### Why can't you add?

Because Java doesn't know the exact subtype.

If:

```java
List<? extends Number> numbers
```

is actually:

```java
List<Integer>
```

then:

```java
numbers.add(10.5);
```

would be unsafe.

Therefore:

```java
numbers.add(...); // generally not allowed
```

But reading is safe:

```java
Number n = numbers.get(0);
```

---

# 9. `? super`

Now:

```java
List<? super Integer>
```

means the list can be:

```text
List<Integer>
List<Number>
List<Object>
```

Now you can safely add Integer:

```java
List<? super Integer> list = new ArrayList<Number>();

list.add(10);
list.add(20);
```

Because Integer can safely go into:

```text
List<Integer>
List<Number>
List<Object>
```

But when retrieving:

```java
Object value = list.get(0);
```

you can safely assume only `Object`.

You **cannot** directly assume Integer.

---

# 10. PECS — memorize this

This is one of the most interview-friendly rules in Java generics:

> **PECS = Producer Extends, Consumer Super**

### Producer → `extends`

If you're primarily **reading/producing** values:

```java
List<? extends Number>
```

### Consumer → `super`

If you're primarily **putting/consuming** values:

```java
List<? super Integer>
```

Classic example:

```java
public static <T> void copy(
        List<? super T> destination,
        List<? extends T> source) {

    for (T item : source) {
        destination.add(item);
    }
}
```

That's essentially the idea behind the JDK's generic collection APIs.

---

# 11. One thing I want you to remember

Don't memorize this as:

> "extends means read and super means write."

That's a useful shortcut, but the more accurate reasoning is:

### `? extends T`

You don't know the exact subtype.

Therefore:

```java
T value = list.get(0); // safe
```

but you cannot safely insert arbitrary T.

### `? super T`

You know the collection accepts T or a parent of T.

Therefore:

```java
list.add(t); // safe
```

but retrieval can only safely be treated as Object.

That explanation is much stronger in an interview.

---

# 12. Generics + Collections — very common interview question

Why does this work?

```java
List<? extends Number> list = List.of(1, 2, 3);
```

Because `Integer` is a subtype of `Number`.

But this doesn't:

```java
List<Number> list = List.of(1, 2, 3);
```

Actually, **this one does compile** because `List.of(1,2,3)` is inferred appropriately and can produce a `List<Integer>` only if the target type permits inference of `Number`.

The important distinction is:

```java
List<Integer> integers = ...
List<Number> numbers = integers; // ❌
```

versus:

```java
List<Number> numbers = List.of(1, 2, 3); // ✅
```

The second uses **generic type inference** at the method call.

That's a nice interview trap.

---

# 13. What about primitive types?

Generics don't work directly with primitives:

```java
List<int> list; // ❌
```

Instead:

```java
List<Integer> list; // ✅
```

Java uses boxing/unboxing:

```java
list.add(10);      // int → Integer
int x = list.get(0); // Integer → int
```

This is one reason primitive streams such as `IntStream` exist: they can avoid some boxing overhead.

---

# 14. Generic arrays — another classic trap

This is not allowed:

```java
T[] array = new T[10]; // ❌
```

because due to type erasure the runtime doesn't know what `T` actually is.

You usually use:

```java
Object[] array
```

with an appropriate cast/design, or prefer collections.

This is enough for EPAM unless they specifically dive into generic arrays.

---

# 15. What EPAM can realistically ask

For Generics, I would prepare these **very strongly**:

### Must know

**Q1. Why do we need generics?**

→ Compile-time type safety + less casting.

**Q2. What is type erasure?**

→ Generic type information is primarily removed/erased for runtime representation.

**Q3. Why is `List<Integer>` not a subtype of `List<Number>`?**

→ Generics are invariant; otherwise unsafe writes would be possible.

**Q4. Difference between `? extends` and `? super`?**

→ PECS.

**Q5. Why can't you add to `List<? extends Number>`?**

→ Exact subtype is unknown.

**Q6. Why can you add Integer to `List<? super Integer>`?**

→ The underlying list is guaranteed to accept Integer.

**Q7. Difference between generic method and generic class?**

→ Generic class parameterizes the class; generic method declares its own type parameter.

**Q8. Can generics use primitive types?**

→ No; use wrapper types.

**Q9. What is bounded type parameter?**

```java
<T extends Number>
```

**Q10. Why can't we create `new T[]`?**

→ Type erasure/runtime type information issue.

---

## Where we stop Generics

For **your EPAM preparation**, I would call Generics ~90% done after this.

I don't think we need to waste time on obscure generic corner cases right now.

### Next: Exception Handling 🔥🔥🔥🔥

And this one will be much more interesting for your actual experience because we'll build a complete flow:

```text
Controller
   ↓
Service
   ↓
Repository
   ↓
Database
```

Then:

```text
Controller
   ↓
Service
   ↓
Downstream REST service
   ↓
Timeout / 4xx / 5xx
   ↓
Exception
   ↓
Exception propagation
   ↓
GlobalExceptionHandler
   ↓
HTTP response to customer
```

We'll also cover:

- checked vs unchecked
- `throw` vs `throws`
- exception hierarchy
- custom exceptions
- exception chaining
- try/catch/finally
- try-with-resources
- `@ExceptionHandler`
- `@ControllerAdvice`
- `ResponseStatusException`
- Spring's `DataAccessException` hierarchy
- DB exceptions
- transaction rollback rules
- downstream exceptions
- circuit breaker interaction
- what **not** to catch
- logging without duplicating logs
- how to design customer-safe error responses

Then we'll move directly into the **biggest remaining EPAM area: Concurrency → ExecutorService → Future → CompletableFuture → Spring `@Async` and thread pools**.

That is where I'll explicitly teach you how to answer:

> **"Have you worked on multithreaded applications?"**

with a genuine Spring Boot production-style answer rather than just textbook Java concurrency.