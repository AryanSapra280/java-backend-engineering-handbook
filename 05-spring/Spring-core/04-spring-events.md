# Spring Events — Interview Revision Notes

> **Goal:** Understand Spring Application Events clearly enough for an SDE-2 interview. Focus on what they are, how they work, synchronous vs asynchronous behavior, transaction-aware events, and the difference between Spring Events and Kafka/NATS.

---

# 1. What are Spring Application Events?

Spring Events provide an **in-process mechanism** for one part of a Spring application to publish an event without directly depending on the components that handle it.

Basic model:

```text
Publisher
    ↓
Event
    ↓
Listener(s)
```

Example:

```text
Contribution processed
        ↓
ContributionProcessedEvent
        ↓
 ┌───────────────┬─────────────────┐
 ↓               ↓                 ↓
Audit listener   Notification      Analytics
```

The publisher does not need to know about the individual listeners.

---

# 2. Why Do We Need Spring Events?

Without events, a service might directly call several other services:

```java
@Service
public class ContributionService {

    public void process() {

        saveContribution();

        sendNotification();

        createAuditRecord();

        updateAnalytics();
    }
}
```

This creates direct dependencies:

```text
ContributionService
 ├── NotificationService
 ├── AuditService
 └── AnalyticsService
```

With Spring Events:

```java
@Service
public class ContributionService {

    private final ApplicationEventPublisher publisher;

    public ContributionService(ApplicationEventPublisher publisher) {
        this.publisher = publisher;
    }

    public void process() {

        saveContribution();

        publisher.publishEvent(
            new ContributionProcessedEvent(1001L)
        );
    }
}
```

Listeners can independently react:

```java
@Component
public class AuditListener {

    @EventListener
    public void handle(ContributionProcessedEvent event) {
        createAuditRecord();
    }
}
```

Another listener:

```java
@Component
public class NotificationListener {

    @EventListener
    public void handle(ContributionProcessedEvent event) {
        sendNotification();
    }
}
```

Now:

```text
ContributionService
       ↓
publish event
       ↓
ContributionProcessedEvent
       ↓
 ┌──────────────┬────────────────┐
 ↓              ↓                ↓
AuditListener   Notification     Analytics
```

The publisher only knows that it is publishing an event.

---

# 3. The Three Important Components

## 3.1 Event

The event represents something that happened.

For example:

```java
public record ContributionProcessedEvent(
        Long contributionId
) {}
```

It could contain information required by the listeners.

---

## 3.2 Publisher

Spring provides:

```java
ApplicationEventPublisher
```

Inject it:

```java
@Service
public class ContributionService {

    private final ApplicationEventPublisher publisher;

    public ContributionService(ApplicationEventPublisher publisher) {
        this.publisher = publisher;
    }
}
```

Publish:

```java
publisher.publishEvent(
    new ContributionProcessedEvent(1001L)
);
```

The important method is:

```java
publishEvent(event)
```

---

## 3.3 Listener

The easiest listener implementation uses:

```java
@EventListener
```

Example:

```java
@Component
public class ContributionListener {

    @EventListener
    public void handle(ContributionProcessedEvent event) {

        System.out.println(
            "Contribution processed: "
            + event.contributionId()
        );
    }
}
```

Spring detects the listener method and invokes it when the matching event is published.

---

# 4. Multiple Listeners

A single event can have multiple listeners.

```text
                 ContributionProcessedEvent
                            ↓
              ┌─────────────┼─────────────┐
              ↓             ↓             ↓
            Audit       Notification    Analytics
```

The publisher doesn't need to call:

```java
auditService.createAudit();
notificationService.send();
analyticsService.update();
```

It only does:

```java
publisher.publishEvent(event);
```

This reduces direct coupling between the publisher and consumers.

---

# 5. Is Spring Event Asynchronous by Default?

**No.**

This is one of the most important interview points.

By default, Spring application events are handled **synchronously**.

Example:

```java
publisher.publishEvent(event);

System.out.println("After event");
```

Conceptually:

```text
publishEvent()
     ↓
Listener executes
     ↓
Listener finishes
     ↓
"After event"
```

The publisher waits for the listener execution to complete.

Therefore:

```java
publisher.publishEvent(event);
```

does NOT automatically mean:

```text
fire-and-forget
```

---

# 6. Making a Listener Asynchronous

Spring supports asynchronous execution with `@Async`.

Example:

```java
@Async
@EventListener
public void handle(ContributionProcessedEvent event) {

    sendNotification();
}
```

And enable async processing:

```java
@EnableAsync
@SpringBootApplication
public class Application {
}
```

Conceptually:

```text
Publisher
    ↓
publishEvent()
    ↓
Submit listener task
    ↓
Publisher can continue

             ↓
         Executor
             ↓
        Thread Pool
             ↓
          Listener
```

---

# 7. `@Async` Uses an Executor / Thread Pool

Do not think:

> "`@Async` simply creates a new thread every time."

Spring's async execution uses an executor.

Conceptually:

```text
Task
 ↓
Executor
 ↓
Thread Pool
 ↓
Worker Thread
```

You can configure a `ThreadPoolTaskExecutor`.

Example:

```java
@Configuration
@EnableAsync
public class AsyncConfig {

    @Bean
    public Executor taskExecutor() {

        ThreadPoolTaskExecutor executor =
                new ThreadPoolTaskExecutor();

        executor.setCorePoolSize(5);
        executor.setMaxPoolSize(10);
        executor.setQueueCapacity(100);

        executor.initialize();

        return executor;
    }
}
```

For interview purposes, understand the concept more than the exact configuration syntax.

---

# 8. Synchronous vs Asynchronous

## Synchronous

```text
Publisher
   ↓
publishEvent()
   ↓
Listener
   ↓
Listener finishes
   ↓
Publisher continues
```

## Asynchronous

```text
Publisher
   ↓
publishEvent()
   ↓
Submit to Executor
   ↓
Publisher continues

             ↓
         Thread Pool
             ↓
          Listener
```

---

# 9. Async Listener Failure

Consider:

```java
@Async
@EventListener
public void handle(Event event) {

    throw new RuntimeException();
}
```

The exception occurs on the asynchronous execution path.

It does not simply behave like a normal synchronous exception propagating back through the original caller's stack.

For production systems, think about:

```text
Logging
Error handling
Async exception handling
Retries if required
Monitoring
```

If you require durable and reliable cross-service event delivery, Spring application events should not be treated as a replacement for Kafka or NATS.

---

# 10. Spring Events vs Kafka ⭐⭐⭐

Very common interview question:

> What is the difference between Spring Application Events and Kafka?

## Spring Application Event

Usually:

```text
Application
     ↓
Spring Event
     ↓
Listener
```

It is primarily an **in-process application mechanism**.

The publisher and listener normally live inside the same application/JVM.

## Kafka

```text
Service A
   ↓
Kafka
   ↓
Service B
```

Kafka is a distributed event streaming / messaging platform.

It provides capabilities such as:

```text
Durability
Persistence
Consumer groups
Partitioning
Replay
Scalability
Cross-service communication
```

So:

```text
Spring Event
→ in-process communication

Kafka
→ distributed event streaming / messaging
```

---

# 11. Spring Event Is NOT a Message Broker

Do not say:

> "Spring Event is like Kafka."

Better interview answer:

> Spring Application Events are an in-process event mechanism used to decouple components within an application. Kafka is a distributed event streaming platform designed for durable event processing and communication across applications and services.

---

# 12. Spring Events vs NATS

For your current learning:

```text
Spring Event
    ↓
Inside one Spring application
```

Whereas:

```text
Service A
    ↓
NATS
    ↓
Service B
```

NATS is intended for distributed messaging.

With JetStream, NATS can also provide persistence and replay-oriented capabilities.

So:

```text
Spring Event → local/in-process decoupling

NATS → distributed messaging

Kafka → distributed event streaming
```

---

# 13. PF Application Example

Suppose your PF service finishes processing:

```text
PF Contribution
      ↓
Process
      ↓
Success
```

You could publish:

```java
publisher.publishEvent(
    new ContributionProcessedEvent(contributionId)
);
```

Then:

```text
ContributionProcessedEvent
          ↓
 ┌────────┼─────────┐
 ↓        ↓         ↓
Audit   Metrics   Notification
```

This can be useful when these are local application concerns.

But if the event needs to travel to another microservice:

```text
PF Service
     ↓
Transaction Service
```

then you would normally consider distributed communication such as:

```text
Kafka
NATS
HTTP
```

depending on the requirement.

---

# 14. Transaction + Spring Event ⭐⭐⭐

This is particularly important.

Suppose:

```java
@Transactional
public void process() {

    saveContribution();

    publisher.publishEvent(
        new ContributionProcessedEvent(...)
    );
}
```

By default, a normal `@EventListener` can execute while the transaction is still active, before it commits.

Conceptually:

```text
BEGIN T1
   ↓
saveContribution()
   ↓
publishEvent()
   ↓
Listener executes
   ↓
COMMIT T1
```

This can create an important consistency issue.

Imagine the listener performs an external action:

```text
DB transaction
   ↓
publish event
   ↓
send notification
   ↓
DB transaction fails
   ↓
ROLLBACK
```

You could end up with:

```text
Notification sent ✅
Database transaction ❌ rolled back
```

---

# 15. `@TransactionalEventListener` ⭐⭐⭐

Spring provides:

```java
@TransactionalEventListener
```

This allows an event listener to execute according to a transaction phase.

Common phases:

```text
BEFORE_COMMIT
AFTER_COMMIT
AFTER_ROLLBACK
AFTER_COMPLETION
```

Example:

```java
@TransactionalEventListener(
    phase = TransactionPhase.AFTER_COMMIT
)
public void handle(ContributionProcessedEvent event) {

    sendNotification();
}
```

Conceptually:

```text
BEGIN
 ↓
DB changes
 ↓
publish event
 ↓
COMMIT
 ↓
AFTER_COMMIT listener
 ↓
send notification
```

This is useful when the action should happen only after the transaction has successfully committed.

---

# 16. `AFTER_COMMIT` Does NOT Mean Guaranteed Delivery

This is a very important distinction.

Suppose:

```text
DB COMMIT
   ↓
Application crashes
   ↓
AFTER_COMMIT listener does not complete
```

The database transaction committed successfully, but the external action may never happen.

Therefore:

```text
@TransactionalEventListener(AFTER_COMMIT)
```

does not provide Kafka-level durable delivery guarantees.

For stronger reliability, consider:

```text
Transactional Outbox
```

---

# 17. Outbox Pattern and Spring Events ⭐⭐⭐

Spring Events can be used as part of an Outbox implementation, but:

> **The Outbox pattern does not depend on Spring Events.**

The essential Outbox idea is:

```text
Business DB update
       +
Outbox event insert
       ↓
SAME DATABASE TRANSACTION
       ↓
COMMIT
```

Example:

```text
                Transaction T1
                     │
          ┌──────────┴──────────┐
          ↓                     ↓
    Business tables        Outbox table
          │                     │
          └────── SAME TX ──────┘
                     ↓
                  COMMIT
                     ↓
              Outbox Publisher
                     ↓
                 Kafka/NATS
```

Spring Events can be used to make the implementation cleaner:

```text
Service
  ↓
publish Spring event
  ↓
Spring event listener
  ↓
insert Outbox record
```

But the critical property is:

> The business data and outbox record must be persisted atomically in the same transaction.

---

# 18. Spring Events vs Outbox

Do not confuse them.

### Spring Event

```text
Application
   ↓
Event
   ↓
Listener
```

Used mainly for in-process decoupling.

### Outbox

```text
Business operation
       ↓
Business DB + Outbox record
       ↓
Atomic commit
       ↓
Publisher
       ↓
Kafka/NATS
```

Used to improve reliability of event publication.

---

# 19. `@EventListener` vs `@TransactionalEventListener`

### `@EventListener`

```java
@EventListener
public void handle(Event event) {
}
```

Normal Spring event listener.

It can run while a surrounding transaction is still active.

### `@TransactionalEventListener`

```java
@TransactionalEventListener(
    phase = TransactionPhase.AFTER_COMMIT
)
public void handle(Event event) {
}
```

Transaction-aware listener.

It can be tied to a particular transaction phase.

---

# 20. When Should You Use Spring Events?

Good use case:

```text
One Spring application
        ↓
Multiple loosely coupled internal reactions
```

Examples:

```text
Order created
   ↓
Audit
Metrics
Local notification
Cache update
```

Avoid using Spring Application Events as a replacement for:

```text
Kafka
NATS
RabbitMQ
```

when you need durable distributed communication.

---

# 21. Advantages

### Loose coupling

Publisher doesn't need direct references to every listener.

### Extensibility

New listeners can be added without changing the publisher.

### Separation of concerns

The business operation can publish:

```text
Something happened
```

while other components decide what to do about it.

### Multiple consumers

One event can be consumed by multiple listeners.

---

# 22. Limitations

Remember these for interviews:

### In-process

Normal Spring Events are local to the application.

### Not a durable message broker

They don't inherently provide Kafka-like persistence and replay.

### Async execution needs configuration

`@Async` involves executor/thread-pool considerations.

### Failure handling

Async listeners require appropriate error handling.

### Transaction consistency

Publishing/listening around database transactions needs careful thought.

### No automatic distributed transaction

A Spring event does not magically coordinate transactions across microservices.

---

# 23. High-Probability Interview Questions

## Q1. What are Spring Application Events?

> They provide an in-process mechanism for publishing events and allowing other Spring-managed components to react without the publisher directly depending on those listeners.

## Q2. How do you publish an event?

Inject:

```java
ApplicationEventPublisher
```

and call:

```java
publisher.publishEvent(event);
```

## Q3. How do you listen?

```java
@EventListener
public void handle(MyEvent event) {
}
```

## Q4. Are Spring Events asynchronous by default?

> No. They are synchronous by default.

## Q5. How do you make them asynchronous?

> Enable Spring async processing with `@EnableAsync` and use `@Async` on the listener.

## Q6. Does `@Async` create a new thread every time?

> Not conceptually. It uses an executor, typically backed by a thread pool.

## Q7. Does Spring Event work between microservices?

> Normal Spring Application Events are an in-process mechanism, so they are not a replacement for distributed messaging between microservices.

## Q8. Spring Events vs Kafka?

> Spring Events are primarily for in-process decoupling. Kafka is a distributed event streaming platform with persistence, partitioning, consumer groups, replay, and cross-service communication capabilities.

## Q9. What is `@TransactionalEventListener`?

> It allows an event listener to execute according to a transaction phase such as `BEFORE_COMMIT`, `AFTER_COMMIT`, `AFTER_ROLLBACK`, or `AFTER_COMPLETION`.

## Q10. Why might `AFTER_COMMIT` still lose an event?

> The database commit and subsequent listener execution are not one durable atomic operation. The application could fail after the commit and before the listener completes.

## Q11. How can we reliably publish an event after a DB transaction?

> A common solution is the Transactional Outbox pattern: write the business change and an outbox event record in the same database transaction, then have a separate publisher send the outbox event to Kafka/NATS.

---

# 24. Monday Interview Cheat Sheet ⭐⭐⭐

If you have very little time, remember these:

```text
1. ApplicationEventPublisher
        ↓
   publishEvent()

2. @EventListener
        ↓
   receives event

3. Spring Events are synchronous by default

4. @Async
        ↓
   asynchronous listener

5. @Async uses an executor/thread pool

6. Spring Event
        ↓
   in-process

7. Kafka / NATS
        ↓
   distributed messaging

8. @TransactionalEventListener
        ↓
   transaction-aware listener

9. AFTER_COMMIT
        ↓
   execute after successful commit

10. AFTER_COMMIT ≠ guaranteed delivery

11. Outbox
        ↓
   business data + event record
   in same DB transaction
```

---

# 25. Final Mental Model

## Normal Spring Event

```text
                    Spring Application
                           |
                    publishEvent()
                           |
                           ↓
                    Spring Event
                    /      |      \
                   ↓       ↓       ↓
                Audit   Metrics   Notification
```

## Async Spring Event

```text
Publisher
    ↓
Event
    ↓
Executor / Thread Pool
    ↓
Listener
```

## Transaction-aware Event

```text
BEGIN
  ↓
DB changes
  ↓
publish event
  ↓
COMMIT
  ↓
AFTER_COMMIT listener
```

## Distributed Event

```text
Service A
    ↓
 Kafka / NATS
    ↓
Service B
```

## Outbox

```text
Service
   ↓
DB Transaction
   ├── Business data
   └── Outbox event
          ↓
       COMMIT
          ↓
  Outbox Publisher
          ↓
      Kafka/NATS
```

---

# 26. The Most Important Interview Statement

If the interviewer asks:

> "Why not just use Spring Events instead of Kafka?"

Answer:

> **Spring Application Events are primarily an in-process mechanism for decoupling components within one Spring application. They are not a durable distributed messaging system. If I need communication between microservices, persistence, replay, consumer groups, partitioning, or reliable event delivery, I would use a messaging system such as Kafka or NATS. For reliable DB-to-event publishing, I would consider the Transactional Outbox pattern.**

That answer connects **Spring Events → Transactions → Outbox → Kafka/NATS**, which is exactly the progression you should be comfortable explaining in a backend interview.
