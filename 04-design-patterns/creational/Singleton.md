# Singleton Design Pattern

## 1. What is Singleton?

**Singleton** is a creational design pattern that ensures:

> **A class has only one instance throughout the application and provides a global access point to that instance.**

In simple words:

```text
Class → Only ONE object → Everyone uses the same object
```

Example:

```text
Logger
Configuration Manager
Cache Manager
Database Connection Manager
```

If 100 classes need the Logger, we don't want:

```text
Logger object 1
Logger object 2
Logger object 3
...
Logger object 100
```

We want:

```text
             ┌──────────────┐
Class A ────►│              │
Class B ────►│    Logger    │ ← ONLY ONE INSTANCE
Class C ────►│              │
Class D ────►│              │
             └──────────────┘
```

---

# 2. What Problem Does Singleton Solve?

Suppose we have:

```java
class Logger {
    public void log(String message) {
        System.out.println(message);
    }
}
```

We can create:

```java
Logger l1 = new Logger();
Logger l2 = new Logger();
Logger l3 = new Logger();
```

Now we have multiple Logger objects.

But sometimes we want exactly one shared object.

For example:

```text
Application
    |
    ├── OrderService ──┐
    ├── PaymentService ─┼──► Logger
    ├── UserService ───┤
    └── EmailService ──┘
```

All services should use the same Logger instance.

That's where Singleton comes in.

---

# 3. Three Core Rules of Singleton

A Singleton implementation generally has three important characteristics.

## Rule 1 — Constructor must be private

```java
private Logger() {
}
```

Why?

Because otherwise anyone can do:

```java
new Logger();
```

and create another instance.

---

## Rule 2 — Class owns its instance

The class itself maintains the single object.

```java
private static Logger instance;
```

---

## Rule 3 — Provide a global access method

Usually:

```java
public static Logger getInstance()
```

This becomes the access point.

---

# 4. Basic Singleton

```java
public class Logger {

    private static Logger instance;

    private Logger() {
    }

    public static Logger getInstance() {

        if (instance == null) {
            instance = new Logger();
        }

        return instance;
    }

    public void log(String message) {
        System.out.println(message);
    }
}
```

Usage:

```java
Logger logger1 = Logger.getInstance();
Logger logger2 = Logger.getInstance();
```

Now:

```java
logger1 == logger2
```

is:

```text
true
```

Both variables point to the same object.

---

# 5. Why is the Constructor Private?

This is one of the most important Singleton interview questions.

Consider:

```java
public class Logger {

    public Logger() {
    }
}
```

Anyone can do:

```java
Logger l1 = new Logger();
Logger l2 = new Logger();
```

Therefore:

```text
Singleton requirement ❌
```

Now:

```java
private Logger() {
}
```

Outside the class:

```java
new Logger();
```

is impossible.

Therefore the class controls its own instantiation.

```text
External code
     |
     X
     |
new Logger()  ← Not allowed
```

Instead:

```java
Logger.getInstance()
```

---

# 6. Lazy Initialization

The basic implementation above uses **lazy initialization**.

That means:

> Create the object only when someone actually asks for it.

```java
private static Logger instance;

public static Logger getInstance() {

    if (instance == null) {
        instance = new Logger();
    }

    return instance;
}
```

Initially:

```text
instance = null
```

First call:

```java
Logger.getInstance();
```

creates the object:

```text
instance
   ↓
Logger Object
```

Second call:

```java
Logger.getInstance();
```

returns the existing object.

---

# 7. Eager Initialization

We can also create the Singleton immediately when the class is loaded.

```java
public class Logger {

    private static final Logger INSTANCE = new Logger();

    private Logger() {
    }

    public static Logger getInstance() {
        return INSTANCE;
    }
}
```

This is called:

> **Eager initialization**

The object is created even if nobody uses it.

### Advantages

* Very simple
* Thread-safe because class initialization is handled by JVM
* No synchronization required

### Disadvantage

The object is created even if it is never used.

---

# 8. Problem with Basic Singleton: Multithreading

This is extremely important for interviews.

Consider:

```java
public static Logger getInstance() {

    if (instance == null) {
        instance = new Logger();
    }

    return instance;
}
```

Suppose two threads execute simultaneously.

```text
Thread 1                    Thread 2

if(instance == null)        if(instance == null)
        ↓                            ↓
      true                         true
        ↓                            ↓
new Logger()                 new Logger()
```

Now we have:

```text
Thread 1 → Logger Object A
Thread 2 → Logger Object B
```

Therefore:

```text
Singleton ❌
```

The simple lazy implementation is **not thread-safe**.

---

# 9. Solution 1 — Synchronized Method

```java
public class Logger {

    private static Logger instance;

    private Logger() {
    }

    public static synchronized Logger getInstance() {

        if (instance == null) {
            instance = new Logger();
        }

        return instance;
    }
}
```

Now only one thread can execute `getInstance()` at a time.

```text
Thread 1 → getInstance()
             ↓
          creates object
             ↓
          releases lock

Thread 2 → getInstance()
             ↓
          gets existing object
```

### Advantage

Simple and correct.

### Disadvantage

Every call acquires the lock.

Even after the object has already been created:

```java
Logger.getInstance();
```

still requires synchronization.

For a frequently accessed Singleton, this can be unnecessary overhead.

---

# 10. Solution 2 — Double-Checked Locking

A common optimized approach is:

```java
public class Logger {

    private static volatile Logger instance;

    private Logger() {
    }

    public static Logger getInstance() {

        if (instance == null) {

            synchronized (Logger.class) {

                if (instance == null) {
                    instance = new Logger();
                }
            }
        }

        return instance;
    }
}
```

Notice there are **two checks**.

First:

```java
if (instance == null)
```

Second:

```java
if (instance == null)
```

Why?

### First check

Avoid synchronization once the object already exists.

### Second check

Protect against multiple threads entering the synchronized block at the same time.

---

# 11. Why `volatile`?

This is an important advanced interview point.

```java
private static volatile Logger instance;
```

`volatile` ensures visibility and prevents problematic instruction reordering around publication of the object.

Without `volatile`, double-checked locking can be unsafe because another thread could observe a reference before the object's construction is safely visible.

For interview purposes:

> **Double-checked locking requires `volatile` for correct publication in Java.**

---

# 12. Solution 3 — Bill Pugh Singleton

This is one of the cleanest Java Singleton implementations.

```java
public class Logger {

    private Logger() {
    }

    private static class SingletonHelper {

        private static final Logger INSTANCE = new Logger();
    }

    public static Logger getInstance() {
        return SingletonHelper.INSTANCE;
    }
}
```

Why does this work?

Java initializes static fields when the class is initialized.

The inner class:

```java
SingletonHelper
```

is not initialized until it is actually used.

Therefore:

```java
Logger
   |
   └── SingletonHelper
           |
           └── INSTANCE
```

`INSTANCE` is created when:

```java
getInstance()
```

accesses `SingletonHelper`.

### Benefits

* Lazy initialization
* Thread-safe
* No explicit synchronization
* Simple implementation

This is commonly called:

> **Bill Pugh Singleton Pattern**

---

# 13. Solution 4 — Enum Singleton

Java provides another very strong Singleton mechanism:

```java
public enum Logger {

    INSTANCE;

    public void log(String message) {
        System.out.println(message);
    }
}
```

Usage:

```java
Logger.INSTANCE.log("Hello");
```

There is only one enum constant:

```java
INSTANCE
```

Therefore it naturally represents a single instance.

### Why is Enum Singleton powerful?

It handles many tricky Java concerns automatically:

* Thread safety
* Serialization
* Reflection resistance
* Instance creation

For pure Java Singleton implementation, Enum Singleton is often considered the safest approach.

---

# 14. Singleton and Serialization

There is another subtle problem.

Suppose:

```java
class Logger implements Serializable
```

We have:

```java
Logger logger1 = Logger.getInstance();
```

If we serialize and deserialize it, Java may create another object.

Conceptually:

```text
Original

Logger Object A


Deserialize

Logger Object B
```

Now:

```java
logger1 != deserializedLogger
```

Singleton property can be broken.

---

# 15. Fixing Serialization

One traditional solution is:

```java
private Object readResolve() {
    return getInstance();
}
```

Example:

```java
public class Logger implements Serializable {

    private static final long serialVersionUID = 1L;

    private Logger() {
    }

    private static class SingletonHelper {
        private static final Logger INSTANCE = new Logger();
    }

    public static Logger getInstance() {
        return SingletonHelper.INSTANCE;
    }

    private Object readResolve() {
        return getInstance();
    }
}
```

`readResolve()` tells Java:

> Don't return the newly deserialized object. Return the existing Singleton instance.

---

# 16. Singleton and Reflection

Reflection can potentially access a private constructor.

For example, conceptually:

```java
Constructor<Logger> constructor =
        Logger.class.getDeclaredConstructor();

constructor.setAccessible(true);

Logger logger = constructor.newInstance();
```

Now someone can potentially create another instance.

Therefore Singleton is not an absolute language-level guarantee in every possible scenario.

Enum Singleton is much stronger against this kind of attack.

---

# 17. Singleton and Cloning

Another way to break Singleton is cloning.

If:

```java
Logger implements Cloneable
```

someone could potentially do:

```java
Logger logger2 = logger1.clone();
```

creating another instance.

Possible protection:

```java
@Override
protected Object clone() throws CloneNotSupportedException {
    throw new CloneNotSupportedException();
}
```

---

# 18. Best Java Implementations

For interviews, remember these four approaches:

| Approach               | Lazy    | Thread Safe | Complexity  |
| ---------------------- | ------- | ----------- | ----------- |
| Basic Lazy             | Yes     | ❌           | Simple      |
| Synchronized           | Yes     | ✅           | Simple      |
| Double-Checked Locking | Yes     | ✅           | Medium      |
| Bill Pugh              | Yes     | ✅           | Simple      |
| Enum                   | Yes-ish | ✅           | Very Simple |

For normal Java code:

### Prefer

```text
Enum
```

or

```text
Bill Pugh
```

depending on the requirements.

---

# 19. When Should I Think of Singleton?

When you see:

> "There should only be one instance of X in the entire application."

Think:

```text
Singleton
```

Examples:

### Logger

```text
One shared logging facility
```

### Configuration

```text
One application configuration
```

### Cache Manager

```text
One shared cache manager
```

### Metrics Registry

```text
One registry for application metrics
```

### Connection Pool Manager

```text
One shared pool manager
```

---

# 20. Important: Singleton Does NOT Mean "Global Variable"

People often describe Singleton as:

> "A global variable."

That's not the real definition.

The important property is:

```text
ONE INSTANCE
+
CONTROLLED CREATION
+
GLOBAL ACCESS POINT
```

The global access is a consequence of the pattern, not the primary purpose.

---

# 21. Singleton vs Static Class

These are NOT the same.

### Static utility class

```java
class MathUtils {

    public static int add(int a, int b) {
        return a + b;
    }
}
```

You don't create an object.

```text
MathUtils → no instance
```

### Singleton

```java
Logger logger = Logger.getInstance();
```

There is an actual object.

```text
Logger
   ↓
ONE OBJECT
```

Therefore Singleton can:

* implement interfaces
* have instance state
* be passed as an object
* participate in polymorphism

---

# 22. Singleton vs Dependency Injection

This is a very important practical point.

In Spring applications, you often don't need to manually implement Singleton.

For example:

```java
@Service
public class PaymentService {
}
```

By default, Spring creates one bean instance per application context.

Therefore:

```text
Spring Bean
    ↓
Singleton scope by default
```

You generally shouldn't write:

```java
public class PaymentService {

    private static PaymentService instance;

    private PaymentService() {
    }

    public static PaymentService getInstance() {
        ...
    }
}
```

Instead, let Spring manage the lifecycle.

This gives you:

```text
Dependency Injection
+
Container-managed lifecycle
```

which is usually cleaner.

---

# 23. Singleton is a Creational Pattern

Singleton belongs to:

```text
Creational Design Patterns
```

Because it deals with:

> **How objects are created and controlled.**

Your current learning:

```text
CREATIONAL
│
├── Factory        ✅
├── Builder        ✅
└── Singleton      ← CURRENT
```

And then:

```text
BEHAVIORAL
│
└── State          ← NEXT
```

---

# 24. How to Recognize Singleton in an LLD Interview

Ask yourself:

### Question 1

> Does the requirement say there should be exactly one object?

If yes → Singleton candidate.

### Question 2

> Does multiple instances cause inconsistency or unnecessary resource usage?

Examples:

```text
Configuration
Logger
Cache
Resource Manager
```

If yes → Singleton candidate.

### Question 3

> Does the object need to maintain shared state?

If yes → Singleton may make sense.

---

# 25. Example — Parking Lot

Suppose you're designing a Parking Lot.

Could we have:

```java
ParkingLot parkingLot = ParkingLot.getInstance();
```

Possibly.

Because perhaps the application represents one physical parking lot.

But don't automatically use Singleton just because there is one object in the real world.

The real question is:

> Does the application need to enforce a single instance?

For an LLD interview, explain the reasoning rather than blindly applying Singleton.

---

# 26. Example — Logger

```java
public class Logger {

    private Logger() {
    }

    private static class SingletonHelper {
        private static final Logger INSTANCE = new Logger();
    }

    public static Logger getInstance() {
        return SingletonHelper.INSTANCE;
    }

    public void log(String message) {
        System.out.println(message);
    }
}
```

Usage:

```java
Logger logger = Logger.getInstance();

logger.log("Payment successful");
```

Anywhere else:

```java
Logger anotherLogger = Logger.getInstance();
```

And:

```java
logger == anotherLogger
```

returns:

```text
true
```

---

# 27. Interview Questions You Should Know

### Q1. What is Singleton?

> Singleton ensures that a class has only one instance and provides a global access point to it.

### Q2. Why private constructor?

> To prevent external code from directly creating objects.

### Q3. How do you make lazy Singleton thread-safe?

Possible approaches:

```text
synchronized
Double-checked locking
Bill Pugh
```

### Q4. Why volatile in double-checked locking?

> To guarantee safe publication and prevent problematic instruction reordering.

### Q5. How can Singleton be broken?

```text
Reflection
Serialization
Cloning
Multiple class loaders
```

### Q6. How do you protect against serialization?

```java
readResolve()
```

### Q7. Why is Enum Singleton recommended?

> JVM guarantees enum instance creation and provides strong protection against serialization and reflection-related issues.

### Q8. Is Spring Bean a Singleton?

By default:

> Yes, Spring beans have singleton scope within a Spring ApplicationContext.

But:

> Spring Singleton ≠ JVM-wide Singleton.

The singleton guarantee is within the Spring container/context.

---

# 28. The Mental Model

Don't memorize implementations first.

Remember this:

```text
                 SINGLETON
                     |
          ┌──────────┴──────────┐
          ↓                     ↓
    ONE INSTANCE          GLOBAL ACCESS
          |
    PRIVATE CONSTRUCTOR
          |
    CLASS CONTROLS CREATION
```

Then ask:

```text
Is lazy initialization needed?
        |
       YES
        |
Is it multithreaded?
        |
       YES
        |
Choose a thread-safe implementation
```

---

# 29. What You Actually Need to Know for LLD

You do NOT need to memorize every obscure Singleton implementation.

For interviews, know these extremely well:

* What problem Singleton solves
* Private constructor
* Static instance
* `getInstance()`
* Lazy vs eager initialization
* Why naive lazy Singleton fails in multithreading
* Synchronized Singleton
* Double-checked locking
* `volatile`
* Bill Pugh Singleton
* Enum Singleton
* Singleton vs static class
* Singleton vs Spring singleton bean
* When Singleton is appropriate
* Problems with global/shared state

Most importantly:

> **Don't force Singleton into an LLD problem.**

Use it only when the domain/application genuinely requires a single shared instance.

---

# 30. Pattern Recognition Cheat Sheet

```text
Need exactly ONE instance?
        ↓
     Singleton


Need different algorithms/behaviors?
        ↓
      Strategy


Need to create different implementations
without exposing construction logic?
        ↓
      Factory


Need to construct a complex object step-by-step?
        ↓
      Builder


Object behavior changes based on its current state?
        ↓
       State
```

---

# 31. Current LLD Pattern Progress

Your current sequence:

```text
1. Strategy   ✅
2. Factory    ✅
3. Builder    ✅
4. Singleton  ← NOW
5. State      ← NEXT
```

After Singleton, **State** is the last one we need to cover in this current pattern set.

The important thing is not to memorize patterns individually.

The goal is to develop this thought process:

```text
Read requirements
      ↓
Identify changing behavior
      ↓
Identify object creation complexity
      ↓
Identify state transitions
      ↓
Identify whether a single shared instance is required
      ↓
Choose pattern only if it solves the actual problem
```
