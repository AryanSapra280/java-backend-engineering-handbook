# Java OOP — Polymorphism

## Status

🟢 COMPLETED / PARKED

This document captures our full polymorphism discussion, including the payment-processing exercise, collection reasoning, overloading, runtime dispatch, interview traps, and the topics intentionally deferred.

---

# 1. Definition

**Polymorphism means “one interface/reference, many possible implementations.”**

For Java interviews, the most important form is **runtime polymorphism**, where an overridden instance method is selected based on the actual object type at runtime.

Example:

```java
PaymentProcessor processor = new UpiPaymentProcessor();

processor.process();
```

The reference type is:

```text
PaymentProcessor
```

The actual object type is:

```text
UpiPaymentProcessor
```

Therefore the overridden implementation in `UpiPaymentProcessor` executes.

---

# 2. Two Important Forms of Polymorphism

```text
                    POLYMORPHISM
                         │
              ┌──────────┴──────────┐
              ↓                     ↓
        Compile-time            Runtime
              │                     │
         Overloading             Overriding
              │                     │
         Same method            Parent/child
         different params       implementation
```

## Compile-time polymorphism

Usually demonstrated through **method overloading**.

```java
class PaymentService {

    void pay(int amount) {
    }

    void pay(int amount, String currency) {
    }

    void pay(PaymentRequest request) {
    }
}
```

The compiler determines which overloaded method is applicable.

## Runtime polymorphism

Demonstrated through **method overriding**.

```java
Payment payment = new UpiPayment();

payment.process();
```

The runtime object determines which overridden instance method implementation executes.

---

# 3. Payment Processing Example

This came directly from the BharatPe interview question:

> “Walk me through how you would design a simple Java class hierarchy for handling different payment types, and what collection types you'd use to store and process a batch of payment objects.”

Payment types:

```text
UPI
Card
Wallet
```

Common behavior:

```java
process()
```

We deliberately did **not** jump immediately to a design pattern.

The reasoning should start from the requirements.

---

# 4. Interface vs Abstract Class

A possible abstraction is:

```java
interface Payment {
    void process();
}
```

Concrete implementations:

```java
class UpiPayment implements Payment {

    @Override
    public void process() {
        // UPI logic
    }
}

class CardPayment implements Payment {

    @Override
    public void process() {
        // Card logic
    }
}

class WalletPayment implements Payment {

    @Override
    public void process() {
        // Wallet logic
    }
}
```

The consumer can work with:

```java
Payment payment;
```

without knowing the concrete implementation.

This demonstrates polymorphism through an interface.

---

# 5. Why Interface Can Be Appropriate

The reasoning we developed:

> If payment types primarily need to follow a common contract and different implementations may need other unrelated capabilities, an interface is useful because a class can implement multiple interfaces.

For example:

```java
class UpiPayment
        implements Payment, Auditable, Refundable {
}
```

Java does not support extending multiple classes:

```java
class UpiPayment extends Payment, Auditable { // ❌
}
```

---

# 6. Important Correction: Abstract Classes Are NOT 100% Abstract

An abstract class can contain both abstract and concrete methods.

```java
abstract class Payment {

    abstract void process();

    void validate() {
        System.out.println("Common validation");
    }
}
```

A child can implement the abstract behavior while inheriting common behavior:

```java
class UpiPayment extends Payment {

    @Override
    void process() {
        System.out.println("UPI payment");
    }
}
```

An abstract class can therefore provide:

```text
Abstract class
├── abstract methods
├── concrete methods
├── instance variables
├── constructors
└── common state / behavior
```

Therefore the correct interview statement is NOT:

> “Interface is better because abstract classes are 100% abstract.”

A stronger statement is:

> “I'd prefer an interface when the payment types primarily need a common contract and may need to implement other unrelated capabilities. If there is meaningful shared state or common implementation that should be reused, I would consider an abstract class.”

---

# 7. Don't Choose a Design Pattern Too Early

A key lesson from the BharatPe interview:

The initial answer jumped directly to:

> “Strategy Pattern.”

That can be useful, but the better engineering sequence is:

```text
Requirements
    ↓
Abstraction
    ↓
Common behavior
    ↓
Concrete implementations
    ↓
Collection / processing model
    ↓
Extensibility requirements
    ↓
Design pattern if justified
```

The pattern should solve an identified problem rather than being the starting point.

---

# 8. Collection Choice Depends on Processing Semantics

Suppose 1,000 payment objects arrive.

There is no universally correct answer such as:

> “Always use List.”

or:

> “Always use Queue.”

The requirement determines the collection.

## List

```java
List<Payment> payments = new ArrayList<>();
```

Appropriate when the application mainly needs:

- ordered storage
- iteration
- possible random access
- no producer/consumer coordination requirement

---

## Queue

If the requirement is FIFO processing:

```java
Queue<Payment> payments = new ArrayDeque<>();
```

Conceptually:

```text
Payment 1
Payment 2
Payment 3
    ↓
process in order
```

---

# 9. Concurrent Producer / Consumer Processing

If multiple threads produce and consume payments, a normal `ArrayDeque` is not enough for thread-safe producer/consumer coordination.

A `BlockingQueue` is appropriate in such a design:

```java
BlockingQueue<Payment> queue =
        new LinkedBlockingQueue<>();
```

Consumers can do:

```java
Payment payment = queue.take();
payment.process();
```

`take()` waits when the queue is empty instead of requiring busy spinning.

Conceptually:

```text
              Producers
            /     |                ↓      ↓      ↓
        ┌──────────────────┐
        │ BlockingQueue    │
        └──────────────────┘
           ↓      ↓      ↓
        Consumer Consumer Consumer
```

---

# 10. Concurrency Introduces a New Requirement

Suppose two payments belong to the same account:

```text
Account A
   │
   ├── Payment ₹500
   └── Payment ₹1000
```

If two consumers process them simultaneously, the application may have ordering or race-condition problems.

Therefore we asked:

> How can payments for the same account be processed sequentially while payments for different accounts are processed concurrently?

---

# 11. Partitioning by Business Key

The solution we derived was:

> Use a business key such as `userId`/`accountId` to determine the partition.

Conceptually:

```text
                 hash(userId)
                     ↓
             partition / queue
          ┌──────────┼──────────┐
          ↓          ↓          ↓
       Consumer 1 Consumer 2 Consumer 3
          │          │          │
       User A      User B      User C
```

Example:

```text
User A → Partition 0
User B → Partition 1
User C → Partition 2
User A → Partition 0
```

This gives:

```text
Same key
   ↓
Same partition
   ↓
Sequential ordering within that partition

Different keys
   ↓
Different partitions
   ↓
Can be processed concurrently
```

Important refinement:

Do not say:

> “The same user always goes to the same physical consumer.”

Better:

> “All payments for the same user/account are routed to the same partition, where they are processed sequentially.”

The physical consumer can change due to reassignment/rebalancing.

---

# 12. Connection to Kafka

We intentionally parked the detailed discussion for the Kafka section.

The same idea appears in Kafka:

```java
producer.send(
    new ProducerRecord<>(
        "payments",
        userId,
        payment
    )
);
```

The key determines the partition.

Conceptually:

```text
User A → Partition 0
User B → Partition 1
User C → Partition 2
User A → Partition 0
```

This is useful because it combines:

```text
Ordering per key
        +
Parallelism across keys
```

We will cover the deeper Kafka concepts later:

- partitions
- offsets
- consumer groups
- rebalancing
- retries
- idempotency
- delivery semantics
- effectively-once processing

---

# 13. Runtime Polymorphism

Example:

```java
class PaymentProcessor {

    void process() {
        System.out.println("Generic payment");
    }
}

class UpiPaymentProcessor extends PaymentProcessor {

    @Override
    void process() {
        System.out.println("UPI payment");
    }
}
```

Now:

```java
PaymentProcessor processor =
        new UpiPaymentProcessor();

processor.process();
```

The reference type is:

```text
PaymentProcessor
```

The actual object is:

```text
UpiPaymentProcessor
```

The overridden child implementation executes.

---

# 14. Compile-Time vs Runtime View

```text
PaymentProcessor processor =
        new UpiPaymentProcessor();
```

### Compile time

The compiler asks:

> Does `PaymentProcessor` expose `process()`?

Yes.

Therefore the call is valid.

### Runtime

The JVM asks:

> What is the actual object?

`UpiPaymentProcessor`.

Therefore:

```java
UpiPaymentProcessor.process()
```

executes.

Mental model:

```text
Reference type
      ↓
What can I CALL?
      ↓
Compile time

Actual object type
      ↓
Which overridden implementation RUNS?
      ↓
Runtime
```

---

# 15. Method Overloading

Overloading is compile-time polymorphism.

Example:

```java
class PaymentService {

    void pay(int amount) {
        System.out.println("int");
    }

    void pay(long amount) {
        System.out.println("long");
    }

    void pay(Integer amount) {
        System.out.println("Integer");
    }
}
```

Calls:

```java
PaymentService service = new PaymentService();

service.pay(10);
service.pay(10L);
service.pay(Integer.valueOf(10));
```

Results:

```text
10                  → int
10L                 → long
Integer.valueOf(10) → Integer
```

Because overload resolution occurs at compile time.

---

# 16. Exact Match vs Boxing

Consider:

```java
class PaymentService {

    void pay(int amount) {
        System.out.println("int");
    }

    void pay(Integer amount) {
        System.out.println("Integer");
    }
}
```

Then:

```java
service.pay(10);
```

selects:

```java
pay(int)
```

because `10` is already an `int`.

Java does not need to box it into `Integer`.

Mental model:

```text
10
 ↓
int
 ↓
exact match
 ↓
pay(int)
```

---

# 17. `null` and Primitive vs Reference Types

`null` can be assigned to a reference type but not to a primitive.

Therefore:

```java
service.pay(null);
```

with:

```java
void pay(int amount)
void pay(Integer amount)
```

selects:

```java
pay(Integer)
```

because:

```text
null → Integer ✅
null → int     ❌
```

Better interview wording:

> “`Integer` is a reference type and can hold `null`, whereas `int` is a primitive and cannot.”

---

# 18. Ambiguous `null` Overload

Consider:

```java
class PaymentService {

    void pay(Integer amount) {
        System.out.println("Integer");
    }

    void pay(Long amount) {
        System.out.println("Long");
    }
}
```

Now:

```java
service.pay(null);
```

Both are valid:

```text
null → Integer ✅
null → Long    ✅
```

But Java has no basis to prefer `Integer` over `Long`.

They are sibling reference types.

Therefore the call is:

```text
Compile-time ambiguity
```

It does not automatically choose `Integer`.

---

# 19. Overload Resolution Intuition

A useful high-level intuition is:

```text
Exact match
    ↓
Widening primitive conversion
    ↓
Boxing / unboxing
    ↓
Varargs
```

This is an interview-oriented mental model, not a replacement for the full Java Language Specification.

We intentionally stopped before diving into obscure overload-resolution rules.

---

# 20. Important Interview Traps

### Trap 1

> “Polymorphism means only method overriding.”

❌ Too narrow.

For interview purposes, know:

- compile-time polymorphism → overloading
- runtime polymorphism → overriding

### Trap 2

> “Interface is always better than abstract class.”

❌ No.

Choose based on:

- shared state
- shared implementation
- common contract
- need for multiple capabilities

### Trap 3

> “Abstract classes are 100% abstract.”

❌ No.

They can contain concrete methods, fields, constructors, and abstract methods.

### Trap 4

> “A Queue is always the right collection for batch processing.”

❌ Depends on requirements.

FIFO and producer/consumer semantics matter.

### Trap 5

> “Same user should always be handled by the same consumer.”

❌ Too specific.

The stronger distributed-system guarantee is:

> Same business key → same partition → ordered processing within that partition.

### Trap 6

> “`null` chooses the first reference-type overload.”

❌ No.

If multiple unrelated reference types match equally, the call can be ambiguous.

### Trap 7

> “`null` can be passed to `int`.”

❌ Primitive types cannot hold `null`.

---

# 21. What We Intentionally Did NOT Go Deep Into

To avoid exhausting Java Core, we parked:

- obscure overload-resolution corner cases
- complete JLS method-invocation rules
- bytecode-level dispatch instructions
- JVM method resolution internals
- advanced generic overload interactions

These can be revisited if a specific interview requires them.

---

# 22. Interview Answer Framework Derived From This Exercise

When asked an open-ended Java/design question, don't immediately name a pattern or technology.

Use:

```text
Requirement
    ↓
Data / behavior model
    ↓
Abstraction
    ↓
Constraints
    ↓
Java feature / collection
    ↓
Concurrency requirements
    ↓
Trade-offs
    ↓
Pattern / technology if justified
```

For example:

```text
Payment processing
      ↓
Common payment contract
      ↓
Payment interface
      ↓
UPI / Card / Wallet implementations
      ↓
FIFO requirement?
      ↓
Queue
      ↓
Concurrent producer/consumer?
      ↓
BlockingQueue
      ↓
Ordering per account?
      ↓
Partition by accountId/userId
      ↓
Kafka becomes a possible infrastructure choice
```

This is the **first-principles reasoning style** we want throughout the preparation.

---

# 23. Senior-Level Takeaways

1. Polymorphism allows code to work against an abstraction while supporting multiple implementations.
2. Runtime polymorphism is based on overridable instance methods and the actual runtime object.
3. Method overloading is resolved at compile time.
4. Interfaces are useful for common contracts and multiple capabilities.
5. Abstract classes are useful when meaningful shared state or implementation exists.
6. Do not select a design pattern before understanding the requirement.
7. Collection choice should follow access and processing semantics.
8. `BlockingQueue` is appropriate for thread-safe producer/consumer coordination.
9. Concurrent processing may introduce ordering requirements.
10. Partitioning by a business key can preserve per-entity ordering while allowing cross-entity parallelism.
11. The same principle appears in Kafka partitioning.
12. `null` works with reference types, not primitives.
13. Unrelated reference-type overloads can make `null` ambiguous.
14. Strong interview answers explain **why**, not just **what**.

---

# 24. Current Preparation Status

```text
Classes / Objects / Constructors    🟢
Encapsulation                       🟢
Inheritance                        🟢
Polymorphism                       🟢
Interfaces                         🔄 NEXT
Abstract Classes                   ⏳
equals / hashCode                  ⏳
Collections                        ⏳
Generics                           ⏳
Exceptions                         ⏳
Java 8+                            ⏳
Concurrency                        ⏳
Async Java                         ⏳
```

## Next Topic

**Interfaces vs Abstract Classes**

We will go deeper into Java-specific interface behavior:

- interface contracts
- multiple interfaces
- default methods
- static interface methods
- private interface methods
- functional interfaces
- interface evolution
- when to choose interface vs abstract class
- practical Spring/LLD examples
