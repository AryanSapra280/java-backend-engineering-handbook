# Java Concurrency — 03: Race Conditions & `synchronized`

## 1. Race Condition

A race condition occurs when correctness depends on the timing/interleaving of concurrent threads accessing shared state.

```text
Multiple threads
      +
Shared mutable state
      +
At least one modification
      +
No appropriate coordination
      ↓
Race condition
```

## 2. Shared Mutable State

Examples:

```java
int balance;
List<Account> accounts;
Map<String, Integer> balances;
```

Mental trigger:

```text
shared + mutable + concurrent access
                ↓
        think about thread safety
```

## 3. Why `count++` Is Not Thread-Safe

Conceptually:

```text
READ count
   ↓
ADD 1
   ↓
WRITE count
```

Two threads can both read the same value and overwrite each other's result.

Example:

```text
Initial count = 0

Thread 1: READ 0
Thread 2: READ 0

Thread 1: WRITE 1
Thread 2: WRITE 1

Final = 1
Expected = 2
```

This is a **lost update**.

## 4. Practical Counter

```java
public class Main {

    static int count = 0;

    public static void main(String[] args)
            throws InterruptedException {

        Thread t1 = new Thread(() -> {
            for (int i = 0; i < 100000; i++) {
                count++;
            }
        });

        Thread t2 = new Thread(() -> {
            for (int i = 0; i < 100000; i++) {
                count++;
            }
        });

        t1.start();
        t2.start();

        t1.join();
        t2.join();

        System.out.println(count);
    }
}
```

Expected mathematically: `200000`.

Actual output can be lower because of a race. It can occasionally be `200000` too; that does not prove thread safety.

```text
Works in my test
        ≠
Thread-safe
```

---

## 5. Read-Modify-Write

These can involve multiple logical steps:

```java
count++;

balance -= amount;

if (balance >= amount) {
    balance -= amount;
}
```

Think:

```text
READ
 ↓
CHECK / CALCULATE
 ↓
WRITE
```

Another thread may interleave between these steps.

---

## 6. Financial Example — Concurrent Withdrawals

Initial:

```text
Balance = ₹10,000
```

Requests:

```text
A → withdraw ₹4,000
B → withdraw ₹8,000
```

Correct behavior:

```text
A succeeds → balance ₹6,000
B then fails → insufficient balance
```

Unsafe code:

```java
public void withdraw(int amount) {

    if (balance >= amount) {
        balance -= amount;
    }
}
```

Possible interleaving:

```text
Thread A                     Thread B

read 10000                   read 10000

10000 >= 4000 → true         10000 >= 8000 → true

balance = 6000               balance = 2000
```

The key business operation is:

```text
CHECK balance
      +
UPDATE balance
```

Those steps must be coordinated together.

---

## 7. Critical Section

A **critical section** is code accessing shared state that must be protected from conflicting concurrent execution.

Example:

```java
if (balance >= amount) {
    balance -= amount;
}
```

The business invariant is:

```text
balance must not become negative
```

Ask:

> What operations must be protected together so the invariant remains true?

---

## 8. Atomicity vs Thread Safety

### Atomicity

Can another thread interfere with an operation halfway through?

### Thread safety

Does the component behave correctly when accessed concurrently?

Making one operation atomic does not automatically make an entire workflow thread-safe.

---

## 9. Race Condition vs Data Race

### Data race

Conflicting unsynchronized concurrent accesses to the same memory location, with at least one write.

### Race condition

A broader condition where correctness depends on timing/order of concurrent operations.

```text
Data race
→ unsynchronized conflicting memory accesses

Race condition
→ correctness depends on timing/order
```

---

# Part 2 — `synchronized`

## 10. What Does `synchronized` Do?

Java's `synchronized` provides:

```text
Mutual exclusion
+
Memory visibility guarantees
```

Mutual exclusion means only one thread can hold a particular monitor at a time.

---

## 11. Monitor

For:

```java
synchronized (lock) {
    // critical section
}
```

the thread must acquire the monitor associated with `lock`.

Conceptually:

```text
Object: lock
       │
       └── Monitor
             │
             ├── Thread A owns it
             └── Thread B waits
```

---

## 12. Synchronized Block

```java
synchronized (lock) {
    count++;
}
```

Conceptually:

```text
acquire monitor
      ↓
execute critical section
      ↓
release monitor
```

If another thread tries to acquire the same monitor while it is owned, it can become `BLOCKED`.

---

## 13. Instance Synchronized Method

```java
class Counter {

    private int count;

    synchronized void increment() {
        count++;
    }
}
```

Conceptually equivalent to:

```java
void increment() {
    synchronized (this) {
        count++;
    }
}
```

The monitor is `this`.

---

## 14. Same Object vs Different Objects

Same object:

```java
Counter counter = new Counter();
```

```text
Thread 1 → counter.increment()
Thread 2 → counter.increment()
```

Both require the same monitor, so they cannot execute the synchronized method simultaneously.

Different objects:

```java
Counter c1 = new Counter();
Counter c2 = new Counter();
```

```text
c1 → monitor A
c2 → monitor B
```

Then:

```text
Thread 1 → c1.increment()
Thread 2 → c2.increment()
```

can execute concurrently.

Important:

> `synchronized` does not mean only one thread in the entire JVM can execute the method. It means only one thread can own the relevant monitor at a time.

---

## 15. Static Synchronized Method

```java
class Counter {

    static int count;

    static synchronized void increment() {
        count++;
    }
}
```

There is no `this` for a static method.

The monitor is associated with:

```java
Counter.class
```

Conceptually:

```java
static void increment() {
    synchronized (Counter.class) {
        count++;
    }
}
```

### Cheat sheet

| Syntax | Monitor |
|---|---|
| `synchronized` instance method | `this` |
| `static synchronized` method | `ClassName.class` |
| `synchronized(lock)` block | `lock` |

---

## 16. Multiple Synchronized Methods

```java
class Example {

    synchronized void methodA() {}

    synchronized void methodB() {}
}
```

On the same object, both use `this`.

Therefore:

```text
Thread 1 → methodA()
Thread 2 → methodB()
```

cannot execute simultaneously.

But on different objects, they can.

---

## 17. Synchronized + Non-Synchronized Method

```java
class Example {

    synchronized void methodA() {}

    void methodB() {}
}
```

A thread in `methodA()` does not automatically block another thread from entering `methodB()`.

`methodB()` does not acquire the same monitor automatically.

---

## 18. Different Locks

```java
class Example {

    private final Object lock = new Object();

    synchronized void methodA() {
        // locks this
    }

    void methodB() {
        synchronized (lock) {
            // locks lock
        }
    }
}
```

Here:

```text
methodA() → this
methodB() → lock
```

Different monitors can be acquired concurrently.

---

## 19. Private Lock

Prefer:

```java
private final Object lock = new Object();
```

and:

```java
synchronized (lock) {
    // critical section
}
```

over exposing an internal lock publicly.

A public lock lets external code interfere with your synchronization.

---

## 20. Why `synchronized` Fixes the Counter

Unsafe:

```java
void increment() {
    count++;
}
```

Safe:

```java
synchronized void increment() {
    count++;
}
```

Now:

```text
Thread 1
   ↓
acquire monitor
   ↓
READ
   ↓
ADD
   ↓
WRITE
   ↓
release monitor

Thread 2
   ↓
waits
   ↓
acquire monitor
   ↓
READ updated value
   ↓
ADD
   ↓
WRITE
```

The read-modify-write sequence is protected.

---

## 21. Why `synchronized` Fixes the Bank Example

```java
public synchronized void withdraw(int amount) {

    if (balance >= amount) {
        balance -= amount;
    }
}
```

Execution:

```text
Thread A
balance = 10000
10000 >= 4000 → true
balance = 6000
release lock
```

Then:

```text
Thread B
balance = 6000
6000 >= 8000 → false
reject
```

The check and update are protected together.

---

## 22. Mutual Exclusion

For a given monitor:

```text
Thread A owns monitor
        ↓
Thread B cannot enter
        ↓
Thread B waits / becomes BLOCKED
```

Once Thread A releases it:

```text
Thread B
   ↓
acquires monitor
   ↓
enters critical section
```

---

## 23. Memory Visibility

`synchronized` also establishes memory visibility guarantees.

Example:

```java
synchronized (lock) {
    value = 100;
}
```

Another thread later acquires the same monitor:

```java
synchronized (lock) {
    System.out.println(value);
}
```

The synchronization establishes the relevant **happens-before** relationship between the unlock and a subsequent lock of the same monitor.

Therefore, the second thread can reliably observe the earlier synchronized write under the Java Memory Model.

---

## 24. Atomicity + Visibility + Mutual Exclusion

Keep these separate:

### Atomicity

```text
Can another thread interfere with the operation halfway through?
```

### Visibility

```text
Will another thread reliably see the change?
```

### Mutual exclusion

```text
Can multiple threads enter the protected section simultaneously?
```

`synchronized` provides:

```text
Mutual exclusion
+
Visibility guarantees
```

and allows a sequence of operations inside the critical section to be protected as an atomic unit with respect to that monitor.

---

## 25. Synchronized Block vs Method

Method:

```java
public synchronized void increment() {
    count++;
}
```

locks:

```text
this
```

Block:

```java
public void increment() {
    synchronized (lock) {
        count++;
    }
}
```

locks:

```text
lock
```

A block provides finer control over the lock object and scope.

---

## 26. Keep Critical Sections Small

Potentially inefficient:

```java
public synchronized void process() {

    callExternalService(); // 5 seconds

    updateSharedState();
}
```

Potentially better:

```java
public void process() {

    callExternalService();

    synchronized (lock) {
        updateSharedState();
    }
}
```

The lock may now be held for a much shorter time.

But do not blindly move code outside the lock. All operations needed to maintain the business invariant must remain protected.

---

## 27. Reentrant Nature of `synchronized`

Java intrinsic monitors are reentrant.

```java
class Example {

    synchronized void methodA() {
        methodB();
    }

    synchronized void methodB() {
        System.out.println("B");
    }
}
```

A thread that already owns the object's monitor through `methodA()` can enter `methodB()` on the same object.

```text
Thread
 ↓
acquires monitor
 ↓
methodA()
 ↓
methodB()
 ↓
same monitor
 ↓
allowed because monitor is reentrant
```

---

## 28. Common Mistake — Different Lock Objects

```java
Object lock1 = new Object();
Object lock2 = new Object();
```

Thread A:

```java
synchronized (lock1) {
    // ...
}
```

Thread B:

```java
synchronized (lock2) {
    // ...
}
```

These do not synchronize with each other.

```text
lock1 ≠ lock2
```

---

## 29. Common Mistake — Synchronizing Only the Getter

This does not automatically make a class thread-safe:

```java
synchronized void setBalance(int balance) {
    this.balance = balance;
}

int getBalance() {
    return balance;
}
```

Thread safety depends on:

```text
shared state
+
invariants
+
all access paths
```

not on simply counting `synchronized` keywords.

---

## 30. Common Mistake — Public Lock

Avoid:

```java
public Object lock;
```

External code could do:

```java
synchronized (account.lock) {
    // hold internal lock
}
```

Prefer:

```java
private final Object lock = new Object();
```

---

# Part 3 — `synchronized` and Multiple JVMs

## 31. The Most Important Limitation

`synchronized` is **JVM-local**.

Suppose a Spring Boot service has:

```text
Payment Service
       │
       ├── Pod 1 / JVM 1
       ├── Pod 2 / JVM 2
       └── Pod 3 / JVM 3
```

Each JVM has its own heap, objects, and monitors.

Conceptually:

```text
JVM 1
  ↓
Account object
  ↓
Monitor A

JVM 2
  ↓
Account object
  ↓
Monitor B

JVM 3
  ↓
Account object
  ↓
Monitor C
```

These monitors are independent.

---

## 32. Why It Fails Across JVMs

Suppose:

```text
Account balance = ₹10,000
```

Two requests:

```text
Request A → Pod 1
Request B → Pod 2
```

Both execute:

```java
synchronized (account) {
    // check + withdraw
}
```

But:

```text
Pod 1 → JVM 1 → Monitor A
Pod 2 → JVM 2 → Monitor B
```

They are not the same Java monitor.

Therefore both can enter their critical sections simultaneously.

---

## 33. Distributed Example

```text
                 Load Balancer
                      │
             ┌────────┴────────┐
             ↓                 ↓
          Pod 1              Pod 2
          JVM 1              JVM 2
             │                 │
          Lock A             Lock B
             │                 │
             └───────┬─────────┘
                     ↓
                Same Account
                 ₹10,000
```

Request A:

```text
Pod 1
 ↓
acquires Lock A
 ↓
reads ₹10,000
```

Request B:

```text
Pod 2
 ↓
acquires Lock B
 ↓
reads ₹10,000
```

Both locks are valid, but they are independent.

Therefore `synchronized` does not solve the distributed concurrency problem.

---

## 34. Why?

Because `synchronized` coordinates:

```text
Java object
+
JVM monitor
```

It does not automatically create a shared lock in:

```text
PostgreSQL
Redis
Kafka
NATS
Kubernetes
Network
```

---

# Part 4 — What to Use Across Multiple JVMs

The correct solution depends on the architecture and consistency requirements.

Common approaches:

```text
Database transactions
Database row-level locking
Optimistic locking
Pessimistic locking
Unique constraints
Distributed locks
Message-based serialization
Partitioning by account/entity
```

---

## 35. Database Row-Level Locking

For a financial account stored in PostgreSQL, one approach is:

```sql
SELECT balance
FROM account
WHERE account_id = ?
FOR UPDATE;
```

inside an appropriate transaction.

Conceptually:

```text
Pod 1
 ↓
DB transaction
 ↓
locks account row
 ↓
checks balance
 ↓
updates balance
 ↓
commit
```

Pod 2 trying to lock the same row must wait for the relevant transaction to release it.

The coordination occurs in the shared database.

---

## 36. Optimistic Locking

Use a version column:

```text
account_id
balance
version
```

Example:

```text
balance = 10000
version = 5
```

Application reads:

```text
balance = 10000
version = 5
```

Then:

```sql
UPDATE account
SET balance = ?,
    version = version + 1
WHERE account_id = ?
  AND version = 5;
```

If another transaction changed the account first, the version is no longer 5.

The update affects zero rows.

The application detects the conflict and can retry or reject the operation.

---

## 37. Distributed Lock

A distributed lock can use an external coordination system.

Conceptually:

```text
Pod 1
   ↓
distributed lock service
   ↓
acquires "account-123"
   ↓
processes withdrawal
   ↓
releases lock
```

Pod 2:

```text
tries "account-123"
        ↓
lock unavailable
        ↓
wait/retry/fail
```

Distributed locks require careful design around:

- Expiration
- Process crashes
- Ownership
- Lock renewal
- Network failures
- Fencing
- Correct release semantics

A distributed lock is not automatically a magic solution.

---

## 38. Message-Based Serialization

For some architectures, operations for the same entity can be serialized.

For example, with Kafka:

```text
Account ID
    ↓
partition key
    ↓
same partition
    ↓
ordered processing
```

This can reduce races by ensuring events for the same entity are processed in order within the relevant consumer model.

However, it does not automatically replace database transactions or make every workflow exactly-once.

---

# Part 5 — `synchronized` vs Distributed Coordination

| `synchronized` | Distributed coordination |
|---|---|
| JVM-local | Cross-process/JVM |
| Locks Java monitor | Uses shared/external coordination |
| Very fast | More expensive |
| Simple | More complex |
| Same JVM threads | Multiple application instances |
| Monitor disappears with JVM | External mechanism has its own lifecycle |
| Not cluster-wide | Designed for cross-instance coordination |

---

# 39. Strong MUFG Interview Scenario

### Question

> You have a Spring Boot application with 5 pods. Two requests for the same account can arrive at different pods. Will `synchronized` prevent concurrent withdrawals?

### Strong answer

> No. `synchronized` only coordinates threads within the same JVM because the monitor belongs to a Java object in that JVM. If requests reach different pods, each JVM has its own object and monitor, so the locks are independent. For cross-instance consistency, I would consider database transactions with row-level locking or optimistic locking, or an architecture that serializes operations for the same account, depending on the consistency and throughput requirements.

This is much stronger than simply saying:

> "Use a distributed lock."

---

# 40. The Key Distributed-System Question

Whenever discussing concurrency, ask:

> **Where does the shared state live?**

### Same JVM

```text
Shared state
 ↓
Java objects
 ↓
synchronized / Lock / Atomic classes
```

### Multiple JVMs

```text
Shared state
 ↓
Database / distributed system
 ↓
DB locking / optimistic locking /
distributed coordination / serialization
```

This is one of the most important backend concurrency mental models.

---

# Part 6 — Coding Practice

## Q1 — Basic Counter

Create:

```java
class Counter {

    private int count = 0;

    void increment() {
        count++;
    }

    int getCount() {
        return count;
    }
}
```

Create two threads that each increment 100,000 times.

Observe the result.

Then change:

```java
void increment()
```

to:

```java
synchronized void increment()
```

Verify the expected result:

```text
200000
```

---

## Q2 — Bank Account ⭐⭐⭐⭐⭐

Implement:

```java
class BankAccount {

    private int balance;

    public BankAccount(int balance) {
        this.balance = balance;
    }

    public void withdraw(int amount) {

        // Make this thread-safe.
    }

    public int getBalance() {
        return balance;
    }
}
```

Test:

```text
Initial balance = ₹10,000

Thread A → withdraw ₹4,000
Thread B → withdraw ₹8,000
```

Requirements:

```text
Balance must never become negative.
Only valid withdrawals succeed.
Concurrent requests must be handled safely.
```

First solve using `synchronized`.

---

## Q3 — Identify the Critical Section

Given:

```java
if (balance >= amount) {
    balance -= amount;
}
```

Explain why the entire:

```text
CHECK + UPDATE
```

needs protection.

Do not simply answer:

> "Put synchronized on the method."

Explain the business invariant first.

---

## Q4 — Multi-JVM Design ⭐⭐⭐⭐⭐

Design the withdrawal operation assuming:

```text
5 Spring Boot pods
1 shared PostgreSQL database
```

Two requests for the same account may reach different pods.

Explain why:

```java
synchronized
```

is insufficient and propose a database-based solution.

---

# Part 7 — Interview Questions

## Basic

1. What is a race condition?
2. What is shared mutable state?
3. Why is `count++` not thread-safe?
4. What is a critical section?
5. What is a lost update?

## Intermediate

6. What does `synchronized` provide?
7. What is a monitor?
8. What object does an instance synchronized method lock?
9. What does a static synchronized method lock?
10. Difference between synchronized method and synchronized block?
11. Can two synchronized methods execute concurrently?
12. Can synchronized methods execute concurrently on different objects?
13. Does `synchronized` provide memory visibility?
14. Difference between atomicity, visibility, and mutual exclusion?
15. Why should a critical section generally be kept small?
16. Is `synchronized` reentrant?

## Advanced / MUFG ⭐⭐⭐⭐⭐

17. Why does `synchronized` not solve concurrency across multiple JVMs?
18. A Spring Boot service has 5 pods. Two requests for the same account hit different pods. Will `synchronized` protect the account?
19. How would you handle concurrent withdrawals across multiple application instances?
20. When would you use database row-level locking?
21. What is optimistic locking?
22. How can Kafka partitioning help serialize operations for the same account?
23. When would a distributed lock be appropriate?
24. What problems can distributed locks introduce?
25. How would you guarantee the balance never becomes negative in a distributed system?

---

# Part 8 — Strong Interview Answers

### What is a race condition?

> A race condition occurs when multiple threads access shared state concurrently and correctness depends on their timing or interleaving.

### Why isn't `count++` thread-safe?

> `count++` is a read-modify-write operation. Multiple threads can read the same value before either writes the updated value, resulting in a lost update.

### What does `synchronized` provide?

> `synchronized` provides mutual exclusion through a monitor and establishes memory visibility guarantees between threads synchronizing on the same monitor. It can therefore protect a critical section and make a sequence of operations atomic with respect to that monitor.

### What does an instance synchronized method lock?

> It locks the monitor associated with `this`.

### What does a static synchronized method lock?

> It locks the monitor associated with the class object, such as `Counter.class`.

### Why doesn't synchronized work across multiple JVMs?

> Java monitors are local to a JVM and associated with objects in that JVM's memory. Different JVMs have different object instances and monitors, so `synchronized` cannot coordinate threads across application instances.

---

# 49. Financial-Domain Mental Model

For a financial operation such as withdrawal:

```text
Request
   ↓
Identify account
   ↓
Read current balance
   ↓
Check business rule
   ↓
Update balance
```

The key question is:

> **What is the consistency boundary?**

If everything is inside one JVM:

```text
synchronized
```

may protect the critical section.

If there are multiple application instances:

```text
Pod 1 ─┐
Pod 2 ─┼──→ Shared Database
Pod 3 ─┘
```

then the database or another shared coordination mechanism must participate in enforcing the invariant.

For a real financial system, the authoritative consistency boundary generally needs to be a shared durable system such as the database/transactional data store, rather than relying solely on an in-memory Java monitor.

---

# 50. Final Mental Model

```text
Race Condition
      ↓
Shared mutable state
      ↓
Concurrent operations
      ↓
Timing/interleaving matters
      ↓
Incorrect result
```

Then:

```text
synchronized
      ↓
Acquire monitor
      ↓
Mutual exclusion
      ↓
Critical section
      ↓
Release monitor
      ↓
Visibility guarantees
```

### Single JVM

```text
Thread A ─┐
Thread B ─┼──→ same Java object
Thread C ─┘
               ↓
          synchronized
               ↓
        same monitor
               ↓
       one at a time
```

### Multiple JVMs

```text
JVM 1 ──→ Monitor A
JVM 2 ──→ Monitor B
JVM 3 ──→ Monitor C

Monitor A ≠ Monitor B ≠ Monitor C
```

Therefore:

```text
synchronized
       ↓
JVM-local coordination
```

For distributed coordination:

```text
Multiple JVMs
      ↓
Shared authoritative state
      ↓
Database locking /
Optimistic locking /
Message serialization /
Distributed coordination
```

---

# 51. Next Topics

```text
01. Thread Basics                    ✅
02. Thread Lifecycle                 ✅
03. Race Conditions                  ✅
04. synchronized                     ✅
        ↓
05. volatile                         ← NEXT
        ↓
06. AtomicInteger / AtomicLong
        ↓
07. ReentrantLock
        ↓
08. ExecutorService
        ↓
09. Thread Pools
        ↓
10. Future
        ↓
11. CompletableFuture
        ↓
12. Async Programming
        ↓
13. Concurrent Collections
        ↓
14. Deadlock / Starvation / Livelock
        ↓
15. Concurrency Coding Problems
```
