Absolutely. For this section, let's keep it **very simple and interview-speaking style**. You should be able to remember the answer and explain it naturally rather than sounding like you've memorized documentation.

# 🟢 C. Spring Bean Scopes

---

## 49. What are Spring Bean Scopes?

### Interview answer

> Bean scope defines **how long a Spring bean lives and how many instances Spring creates**.

The commonly discussed scopes are:

```text
singleton  → one instance per Spring container
prototype  → new instance whenever requested
request    → one instance per HTTP request
session    → one instance per HTTP session
```

### Easy way to remember

> **Scope = How many objects + how long they live.**

---

# 50. What is Singleton Scope?

### Interview answer

> Singleton is the default Spring bean scope. Spring creates **one instance of that bean per Spring IoC container** and reuses that instance wherever it is injected.

Example:

```java
@Service
public class PaymentService {
}
```

By default, this is singleton.

If 100 other classes inject it:

```text
OrderService ───┐
PaymentController ─┤
RefundService ────┤──→ Same PaymentService object
KafkaConsumer ────┘
```

Not:

```text
OrderService → PaymentService #1
RefundService → PaymentService #2
```

### Remember

> **Singleton = one object, shared by everyone in that Spring container.**

---

# 51. What is Prototype Scope?

### Interview answer

> Prototype scope tells Spring to create a **new bean instance every time the bean is requested from the container**.

Example:

```java
@Component
@Scope("prototype")
public class ReportGenerator {
}
```

If you ask Spring for it twice:

```java
ReportGenerator r1 = context.getBean(ReportGenerator.class);
ReportGenerator r2 = context.getBean(ReportGenerator.class);
```

Then:

```text
r1 ≠ r2
```

They are two different objects.

### Remember

> **Prototype = new object whenever Spring is asked for one.**

---

# 52. Singleton vs Prototype?

This is a very common question.

| Singleton                   | Prototype                               |
| --------------------------- | --------------------------------------- |
| One instance per container  | New instance for each container request |
| Default scope               | Must explicitly configure               |
| Shared                      | Not shared                              |
| Good for stateless services | Useful when object has its own state    |

Example:

```text
Singleton:

Request 1 ──┐
Request 2 ──┼──→ Same object
Request 3 ──┘


Prototype:

Request 1 ──→ Object #1
Request 2 ──→ Object #2
Request 3 ──→ Object #3
```

### Interview answer

> Singleton gives one shared instance per Spring container, while prototype gives a new instance whenever the bean is requested from the container.

---

# 53. What are request and session scopes?

These are mainly relevant to **web applications**.

### Request scope

> One bean instance is created for each HTTP request.

```text
HTTP Request 1 → Bean #1
HTTP Request 2 → Bean #2
HTTP Request 3 → Bean #3
```

Example:

```java
@Component
@Scope("request")
public class RequestData {
}
```

Useful when you need data specific to one HTTP request.

---

### Session scope

> One bean instance is created for each HTTP session.

```text
User A session → Bean #1
User B session → Bean #2
```

Example:

```java
@Component
@Scope("session")
public class UserSessionData {
}
```

### Easy memory

```text
singleton → application/container
prototype → each request from container
request   → each HTTP request
session   → each HTTP session
```

---

# 54. Is Spring Singleton Thread-Safe?

🔥 Very important.

### Answer

> **No. Singleton scope does not mean thread-safe.**

Spring only guarantees that there is one instance per container.

Suppose:

```java
@Service
public class CounterService {

    private int count = 0;

    public void increment() {
        count++;
    }
}
```

Multiple HTTP requests can access the same object simultaneously:

```text
Request 1 ──┐
Request 2 ──┼──→ Same CounterService
Request 3 ──┘
```

So `count` can have concurrency problems.

### Key line to remember

> **Singleton controls the number of objects, not thread safety.**

This is an excellent interview line.

---

# 55. Why should singleton services generally be stateless?

Because the same singleton object is shared by many threads.

Example:

```java
@Service
public class PaymentService {

    private String currentUser;
}
```

Imagine:

```text
Thread 1 → currentUser = Aryan
Thread 2 → currentUser = Rahul
```

They are modifying the **same variable in the same object**.

You can get incorrect results.

Instead, keep request-specific data inside method variables:

```java
@Service
public class PaymentService {

    public void processPayment(String userId) {
        // userId is local to this method call
    }
}
```

### Interview answer

> Singleton services are shared by multiple requests and threads, so keeping mutable request-specific state in instance variables can cause race conditions. Therefore, service beans are generally designed to be stateless.

### Easy rule

> **Singleton + shared mutable state = be careful.**

---

# 56. What happens if a singleton bean contains mutable instance variables?

The variable is shared by all users/threads using that singleton.

Example:

```java
@Service
public class OrderService {

    private String currentOrderId;

    public void process(String orderId) {
        currentOrderId = orderId;
        // processing
    }
}
```

Imagine:

```text
Thread 1:
currentOrderId = A

Thread 2:
currentOrderId = B
```

Thread 1 may now see:

```text
B
```

instead of:

```text
A
```

This can create:

* Race conditions
* Data corruption
* Incorrect results
* Difficult-to-debug production issues

### Better

```java
public void process(String orderId) {
    String currentOrderId = orderId;
}
```

Local variables belong to the individual method execution/thread.

### Interview answer

> Since the singleton is shared, mutable instance variables are also shared. If multiple threads modify them, we can get race conditions and incorrect data.

---

# 57. What happens when a prototype bean is injected into a singleton?

🔥 This is a **very common tricky question**.

Suppose:

```java
@Component
@Scope("prototype")
public class ReportGenerator {
}
```

And:

```java
@Service
public class ReportService {

    private final ReportGenerator generator;

    public ReportService(ReportGenerator generator) {
        this.generator = generator;
    }
}
```

You might think:

> "Every time `ReportService` uses `generator`, I'll get a new object."

**No.**

`ReportService` is singleton.

When Spring creates `ReportService`, it injects a prototype instance **at that time**.

So:

```text
Singleton ReportService
        │
        └── Prototype ReportGenerator #1
```

That same `ReportGenerator #1` remains inside the singleton.

### Important rule

> **Prototype dependency does not automatically become a new instance every time a singleton uses it.**

This is a classic interview trap.

---

# 58. How can you get a new prototype instance every time?

There are several ways.

The most practical one to know:

### `ObjectProvider`

```java
@Service
public class ReportService {

    private final ObjectProvider<ReportGenerator> provider;

    public ReportService(ObjectProvider<ReportGenerator> provider) {
        this.provider = provider;
    }

    public void generate() {

        ReportGenerator generator =
                provider.getObject();

        // use generator
    }
}
```

Now each call to:

```java
provider.getObject();
```

can obtain a new prototype instance.

Conceptually:

```text
ReportService
      │
      ↓
ObjectProvider
      │
      ├── getObject() → Prototype #1
      ├── getObject() → Prototype #2
      └── getObject() → Prototype #3
```

Other approaches include:

* `@Lookup`
* `Provider<T>`
* Factory pattern

For interviews, know **ObjectProvider + @Lookup**.

---

# 59. What is `ObjectProvider`?

### Interview answer

> `ObjectProvider` is a Spring interface that allows us to retrieve a bean from the container when we need it, instead of having Spring inject the actual bean instance immediately.

Example:

```java
@Service
public class ReportService {

    private final ObjectProvider<ReportGenerator> provider;

    public ReportService(
            ObjectProvider<ReportGenerator> provider) {
        this.provider = provider;
    }

    public void generate() {

        ReportGenerator generator =
                provider.getObject();
    }
}
```

If `ReportGenerator` is prototype scoped:

```java
@Scope("prototype")
```

then:

```java
provider.getObject();
```

gets a new instance.

### Easy memory

> **ObjectProvider = "Give me the bean when I need it."**

---

# 60. What is `@Lookup`?

`@Lookup` is another way to get a fresh bean from the Spring container.

Example:

```java
@Service
public abstract class ReportService {

    @Lookup
    protected abstract ReportGenerator getReportGenerator();

    public void generate() {
        ReportGenerator generator =
                getReportGenerator();
    }
}
```

Spring overrides the lookup method and gets the bean from the container.

If `ReportGenerator` is prototype:

```text
getReportGenerator() → #1
getReportGenerator() → #2
getReportGenerator() → #3
```

### Interview answer

> `@Lookup` tells Spring to override a method and resolve the required bean from the container each time that method is called. It can be used when a singleton needs a new prototype instance.

### Which one should you mention first?

For practical code:

> **I would usually prefer `ObjectProvider` because the dependency is explicit and easier to understand and test.**

---

# 61. What does `@Lazy` do?

### Interview answer

> `@Lazy` tells Spring to delay bean creation until the bean is actually needed instead of creating it during application startup.

Normally:

```text
Application starts
      ↓
Create singleton beans
      ↓
Application ready
```

With:

```java
@Lazy
@Component
public class HeavyService {
}
```

Spring delays creating `HeavyService`.

```text
Application starts
      ↓
HeavyService NOT created
      ↓
Someone requests HeavyService
      ↓
Create HeavyService
```

### Example

```java
@Lazy
@Service
public class HeavyReportService {
}
```

The bean is initialized when first requested.

---

# 62. When would you use lazy initialization?

Use it when creating a bean is **expensive** and you don't need it immediately.

For example:

```text
Heavy database initialization
Large cache
Expensive client
Rarely used functionality
```

Example:

```java
@Lazy
@Service
public class LargeReportService {
    
    public LargeReportService() {
        // expensive initialization
    }
}
```

### But don't use it everywhere.

Lazy initialization can:

* Reduce startup time
* Delay resource usage

But:

* First request may become slower
* Configuration/initialization problems may appear later instead of at startup

### Interview answer

> I would use lazy initialization when a bean is expensive to create but isn't needed during startup. It can reduce startup time, but the first use may have the initialization cost.

---

# 🧠 The 4 Concepts You MUST Understand

These questions are really testing four concepts:

### 1. Singleton

```text
One object
    ↓
Shared by many threads
```

Therefore:

```text
Don't keep request-specific mutable state
```

---

### 2. Prototype

```text
New object
    ↓
Every time container is asked
```

But:

```text
Prototype injected into singleton
        ↓
Does NOT magically create a new object each time
```

---

### 3. ObjectProvider

```text
Singleton
    ↓
ObjectProvider
    ↓
getObject()
    ↓
Get bean when needed
```

---

### 4. Lazy

```text
Normal singleton:

Startup → Create bean


@Lazy:

Startup → Don't create
             ↓
First use → Create
```

---

# 🎯 Interview Rapid-Fire Revision

If you're sitting in an interview, these are the answers I want you to be able to say **without thinking**:

**Q: What is scope?**

> Scope defines how many instances of a bean Spring creates and how long those instances live.

**Q: Default scope?**

> Singleton.

**Q: Is singleton thread-safe?**

> No. Singleton only means one instance per container; it doesn't provide thread safety.

**Q: Why stateless services?**

> Because the singleton is shared across multiple threads, and mutable instance state can cause race conditions.

**Q: Prototype?**

> Spring creates a new instance whenever the bean is requested from the container.

**Q: Prototype injected into singleton?**

> The prototype is created when the singleton is created and that same instance is held by the singleton. It doesn't automatically create a new prototype on every method call.

**Q: Need a new prototype every time?**

> Use `ObjectProvider`, `@Lookup`, or a factory.

**Q: What is `@Lazy`?**

> It delays bean creation until the bean is first needed.

---

## 🔥 One scenario to remember all of this

Imagine your PF application has:

```java
@Service
public class PFCalculationService {
    
    private final CalculationContext context;

    public PFCalculationService(CalculationContext context) {
        this.context = context;
    }
}
```

If `PFCalculationService` is singleton and `CalculationContext` contains:

```java
private Member currentMember;
private BigDecimal currentInterest;
```

you should immediately think:

> ⚠️ **Is this shared mutable state?**

Because multiple PF calculations could execute concurrently:

```text
Member A ──┐
Member B ──┼──→ Same PFCalculationService
Member C ──┘
```

If each calculation needs its own state, that state should generally be **local to the operation**, passed explicitly, or otherwise isolated—not casually stored in a shared singleton field.

That's the kind of practical connection that makes your Spring knowledge useful beyond simply answering definitions.
