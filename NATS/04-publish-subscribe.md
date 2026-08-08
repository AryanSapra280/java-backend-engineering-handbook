# NATS Publish-Subscribe

## 1. What is Publish-Subscribe?

Publish-Subscribe, or **Pub/Sub**, is one of the fundamental communication patterns in Core NATS.

The basic idea is very simple:

> A publisher sends a message to a subject, and every subscriber currently interested in that subject receives a copy.

The basic flow is:

```text
Publisher
    |
    | publish(message)
    v
NATS Server
    |
    | find subscribers interested in subject
    |
    +----------+----------+
    |          |          |
    v          v          v
Subscriber A Subscriber B Subscriber C
```

The publisher does not need to know:

- Who the subscribers are
- How many subscribers exist
- Where they are running
- Whether they are running on the same machine
- Whether another subscriber will appear later

The NATS server handles the routing.

This is one of the main ways NATS provides **loose coupling** between services.

---

# 2. The Core Idea

Suppose we have an order system.

Three things can happen:

```text
orders.created
orders.shipped
orders.canceled
```

Different services may be interested in these events.

For example:

```text
orders.created
      |
      +----> Warehouse
      +----> Analytics
      +----> Audit

orders.shipped
      |
      +----> Notifications
      +----> Analytics

orders.canceled
      |
      +----> Notifications
      +----> Analytics
```

The Order Service does not directly call each service.

Instead:

```text
Order Service
      |
      | publish("orders.created")
      v
    NATS
      |
      +----> Warehouse
      +----> Analytics
      +----> Audit
```

This is the fundamental Pub/Sub model.

---

# 3. Publisher

A **publisher** is a NATS client that sends a message to a subject.

For example:

```text
Order Service
      |
      | publish
      |
      | subject = orders.created
      |
      v
    NATS
```

The publisher chooses a concrete subject:

```text
orders.created
```

and sends a payload.

For example:

```json
{
  "order_id": "ord_8w2k",
  "customer": "acme-co",
  "total_cents": 4200,
  "ts": "2026-05-22T10:14:22Z"
}
```

The important point is:

> The publisher publishes to a subject. It does not publish directly to a particular subscriber.

---

# 4. Subscriber

A **subscriber** is a NATS client that registers interest in a subject.

For example:

```text
Warehouse Service
       |
       | subscribe
       |
       | orders.created
       v
      NATS
```

The subscriber is essentially saying:

> "I am interested in messages matching this subject."

Then whenever a matching message arrives, NATS delivers it to that subscriber.

---

# 5. The Simplest Example

Imagine:

```text
Warehouse
   |
   | subscribe("orders.created")
   v
 NATS
```

Then the publisher sends:

```text
Order Service
   |
   | publish("orders.created", order)
   v
 NATS
```

NATS sees:

```text
orders.created
       |
       v
Warehouse is interested
```

So:

```text
Order Service
      |
      | orders.created
      v
    NATS
      |
      v
  Warehouse
```

---

# 6. What Happens When Multiple Subscribers Exist?

Now suppose three services subscribe to the same subject:

```text
Warehouse
    |
    | orders.created
    |
    +------------------+
                       |
Analytics              |
    |                  |
    | orders.created    |
    |                  |
    +------------------+
                       |
Audit                  |
    |                  |
    | orders.created    |
    |                  |
    +------------------+
                       |
                       v
                     NATS
```

More clearly:

```text
                    NATS
                     |
              orders.created
                     |
          +----------+----------+
          |          |          |
          v          v          v
      Warehouse  Analytics     Audit
```

Now the publisher sends one message:

```text
Order Service
      |
      | orders.created
      v
    NATS
```

NATS delivers a copy to each matching subscriber:

```text
                    NATS
                     |
              orders.created
                     |
          +----------+----------+
          |          |          |
          v          v          v
      Warehouse  Analytics     Audit
          |          |          |
        copy       copy       copy
```

This is called **fan-out**.

One published message can result in multiple deliveries.

---

# 7. Important: Subscribers Do Not Consume From a Shared Queue

This is one of the most important differences to understand.

Suppose:

```text
Subscriber A
Subscriber B
Subscriber C
```

all subscribe to:

```text
orders.created
```

When a message arrives:

```text
orders.created
```

it is not:

```text
Message
   |
   v
Queue
   |
   +----> Subscriber A
```

where A removes the message and B/C no longer see it.

Instead:

```text
                    NATS
                     |
              orders.created
                     |
          +----------+----------+
          |          |          |
          v          v          v
          A          B          C
        copy       copy       copy
```

Each subscriber gets its own copy.

Therefore:

> **A normal Core NATS subscription is broadcast-style delivery, not shared-queue consumption.**

This distinction becomes very important when we later study **Queue Groups**.

---

# 8. Pub/Sub vs Queue Groups

Consider three instances of the same service:

```text
Distribution-1
Distribution-2
Distribution-3
```

If all three are normal subscribers:

```text
                    NATS
                     |
           distribution.request
                     |
          +----------+----------+
          |          |          |
          v          v          v
       Pod 1       Pod 2      Pod 3
```

One message can reach all three.

That is normal Pub/Sub.

But sometimes we don't want that.

Suppose:

> "I have three instances of Distribution Service because I want to scale processing horizontally. Each request should be processed by only one instance."

Then we use a **Queue Group**.

```text
                    NATS
                     |
           distribution.request
                     |
               Queue Group
                     |
          +----------+----------+
          |          |          |
          v          v          v
       Pod 1       Pod 2      Pod 3
```

Now one message is delivered to one member of the queue group.

So keep this mental model:

```text
Normal Pub/Sub

One message
     |
     +----> Subscriber A
     +----> Subscriber B
     +----> Subscriber C


Queue Group

One message
     |
     +----> One member of the group
```

Queue Groups will be studied separately.

---

# 9. Fire-and-Forget Publishing

One of the most important properties of Core NATS Pub/Sub is:

> **Publishing is fire-and-forget.**

When the publisher publishes a message:

```text
Publisher
    |
    | publish()
    v
  NATS
```

the publisher does not wait for every subscriber to process the message.

It also does not receive a result such as:

```text
Delivered to Warehouse ✓
Delivered to Analytics ✓
Delivered to Audit ✓
```

The publish operation is not an application-level acknowledgement from the subscribers.

The NATS documentation describes this as:

> The publisher hands the message to the server and moves on.

Conceptually:

```text
Publisher
    |
    | publish()
    v
NATS
    |
    | return
    v
Publisher continues
```

Meanwhile NATS handles delivery to matching subscribers.

---

# 10. What Does a Successful Publish Mean?

This is an important question.

Suppose:

```text
nats pub orders.created ...
```

returns successfully.

Does that mean:

```text
Warehouse processed it?
```

No.

Does it mean:

```text
Analytics processed it?
```

No.

Does it mean:

```text
Three subscribers received it?
```

No.

It means the publish operation itself succeeded from the publisher/client perspective.

Core NATS does not make the publisher wait for subscriber processing.

Therefore:

```text
Publish success
      ≠
Business processing success
```

This distinction is extremely important in distributed systems.

---

# 11. What If Nobody Is Listening?

This is one of the most important Core NATS behaviors.

Suppose:

```text
Order Service
      |
      | publish("orders.created")
      v
    NATS
```

But nobody is subscribed:

```text
No subscribers
```

What happens?

The publish still succeeds, but there is nobody to receive the message.

The message is discarded.

Conceptually:

```text
Publisher
    |
    | orders.created
    v
  NATS
    |
    X
No interested subscriber
```

There is no persistent backlog in Core NATS.

The message is gone.

---

# 12. Silent No-Op

This behavior is worth remembering.

A publish with no interested subscriber is effectively a **silent no-op** from the publisher's perspective.

The publisher cannot distinguish:

```text
Case A:

Published
   |
   v
3 subscribers received it
```

from:

```text
Case B:

Published
   |
   v
0 subscribers
   |
   v
Message discarded
```

Both can look like a successful publish operation.

Therefore:

> **Core NATS does not tell the publisher how many subscribers received the message.**

This is a major consequence of fire-and-forget messaging.

---

# 13. Example: Subscriber Restart

Imagine:

```text
Warehouse Service
       |
       | subscribed
       v
orders.created
```

Now Warehouse restarts:

```text
Warehouse
    |
    X
Restarting
```

During this period:

```text
Order Service
      |
      | orders.created
      v
    NATS
      |
      X
Warehouse unavailable
```

Core NATS does not store that message for Warehouse.

When Warehouse comes back:

```text
Warehouse
    |
    | subscribe("orders.created")
    v
 NATS
```

it will receive future messages.

It will not automatically receive the message published while it was offline.

Therefore:

```text
Message published while subscriber offline
                  |
                  v
               LOST
```

This is a direct consequence of Core NATS being **ephemeral** and **at-most-once**.

---

# 14. At-Most-Once Delivery

Core NATS provides:

> **At-most-once delivery.**

For a subscriber that is connected and interested when the message is published:

```text
0 or 1 delivery
```

More precisely:

```text
Connected + interested
        |
        v
      Message
        |
        v
    Delivered once
```

If the subscriber is absent or disconnected:

```text
Subscriber offline
        |
        v
Message published
        |
        v
No delivery
```

There is no automatic second attempt.

---

# 15. What Does At-Most-Once NOT Guarantee?

At-most-once does not mean:

> "Every subscriber will receive every message."

It means:

> A message will not be automatically redelivered to a subscriber by Core NATS.

A subscriber can receive:

```text
0 times
```

or:

```text
1 time
```

but Core NATS does not provide durable redelivery.

This is why Core NATS is appropriate for transient information but not automatically sufficient for important business events that must survive failures.

---

# 16. Why Would Anyone Want At-Most-Once?

At first, this may sound like a limitation.

But consider a live stock price:

```text
AAPL = 220
AAPL = 221
AAPL = 222
AAPL = 223
```

Suppose a dashboard misses:

```text
AAPL = 221
```

The next update might be:

```text
AAPL = 222
```

The old value may no longer matter.

Another example:

```text
Temperature = 30.2°C
Temperature = 30.3°C
Temperature = 30.4°C
```

If one update is missed, another update is coming shortly.

Core NATS can be very appropriate for such transient information.

Examples include:

- Live telemetry
- Current status
- Live dashboards
- Cache invalidation
- Transient notifications
- Real-time updates

The important question is:

> **Can the system safely tolerate missing this particular message?**

If yes, Core NATS may be a good fit.

---

# 17. When At-Most-Once Is NOT Enough

Consider a financial operation:

```text
Interest Service
      |
      | distribution.request
      v
    NATS
      |
      v
Distribution Service
```

Suppose the message means:

```text
Distribute ₹50,000
for account XYZ
```

If Distribution is temporarily unavailable:

```text
Interest
    |
    v
  NATS
    |
    X
Distribution unavailable
```

Core NATS can lose the message.

That may be unacceptable.

For important business events, we generally need durable messaging semantics.

That is where **JetStream** becomes relevant.

---

# 18. Core NATS vs JetStream

A useful mental model is:

```text
                    NATS
                     |
          +----------+----------+
          |                     |
          v                     v
     Core NATS              JetStream
          |                     |
     Live messaging        Durable messaging
          |                     |
     At-most-once          Persistence
                           Acknowledgements
                           Redelivery
                           Replay
                           Retention
```

Core NATS asks:

> "Who is interested right now?"

JetStream adds capabilities for:

> "How do I retain this message and process it reliably?"

So:

```text
Transient communication
        |
        v
    Core NATS


Durable business event
        |
        v
     JetStream
```

---

# 19. The Interest Graph

One of the most important internal concepts behind Core NATS Pub/Sub is the **interest graph**.

The NATS server maintains an in-memory representation of:

> Which clients are currently subscribed to which subjects?

For example:

```text
orders.created
     |
     +----> Warehouse
     +----> Analytics
     +----> Audit

orders.shipped
     |
     +----> Notifications
     +----> Analytics
```

This is the interest graph.

---

# 20. How the Interest Graph Works

Suppose:

```text
Warehouse → orders.created
Analytics → orders.created
Audit     → orders.created
```

The server knows:

```text
orders.created
      |
      +---- Warehouse
      +---- Analytics
      +---- Audit
```

Now the publisher sends:

```text
publish("orders.created")
```

The NATS server looks at the interest graph.

It finds:

```text
orders.created
      |
      +---- Warehouse
      +---- Analytics
      +---- Audit
```

Then it routes the message to each matching subscriber.

```text
                    NATS
                     |
              Interest Graph
                     |
          +----------+----------+
          |          |          |
          v          v          v
      Warehouse  Analytics     Audit
```

The publisher itself does not maintain this list.

---

# 21. Why the Interest Graph Creates Loose Coupling

Without NATS, the Order Service might have to know:

```text
Warehouse URL
Analytics URL
Audit URL
Notification URL
```

and directly call each one.

That creates coupling:

```text
Order Service
   |
   +----> Warehouse
   |
   +----> Analytics
   |
   +----> Audit
```

With NATS:

```text
Order Service
      |
      | orders.created
      v
    NATS
      |
      +----> Warehouse
      +----> Analytics
      +----> Audit
```

The Order Service only needs to know:

```text
orders.created
```

It does not need to maintain the subscriber list.

This is a major benefit of Pub/Sub.

---

# 22. Subscribers Can Come and Go

Because the NATS server maintains the interest graph, subscribers can dynamically appear and disappear.

Suppose initially:

```text
orders.created
      |
      +---- Warehouse
      +---- Analytics
```

A new service starts:

```text
Audit Service
```

and subscribes:

```text
Audit → orders.created
```

The graph becomes:

```text
orders.created
      |
      +---- Warehouse
      +---- Analytics
      +---- Audit
```

The next message automatically reaches Audit.

The publisher does not change.

Now Audit shuts down.

Its subscription disappears:

```text
orders.created
      |
      +---- Warehouse
      +---- Analytics
```

Again, the publisher does not change.

This is an important property of decoupled messaging.

---

# 23. Publisher Does Not Need Subscriber Information

Consider:

```text
Order Service
```

It publishes:

```text
orders.created
```

It doesn't need to know:

```text
How many subscribers?
Who are they?
Where are they?
Which pods are running?
Which region are they in?
```

The NATS server handles that routing.

This gives us:

> **Location transparency and loose coupling.**

---

# 24. Fan-Out

The Pub/Sub pattern naturally provides fan-out.

Suppose:

```text
Order Service
      |
      | One message
      v
    NATS
```

and five subscribers exist:

```text
Warehouse
Analytics
Audit
Notifications
Fraud Detection
```

The server can fan the message out:

```text
                     NATS
                      |
             orders.created
                      |
       +------+------+------+------+------+
       |      |      |      |      |
       v      v      v      v      v
   Warehouse Analytics Audit Notification Fraud
```

The publisher publishes only once.

The server performs the fan-out.

---

# 25. Pub/Sub vs HTTP

This is a useful comparison for your internal communication use case.

## HTTP

Suppose Order Service needs to notify three services:

```text
Order Service
   |
   +---- HTTP ----> Warehouse
   |
   +---- HTTP ----> Analytics
   |
   +---- HTTP ----> Audit
```

The Order Service has to know about these destinations.

---

## NATS Pub/Sub

With NATS:

```text
Order Service
      |
      | publish("orders.created")
      v
    NATS
      |
      +----> Warehouse
      +----> Analytics
      +----> Audit
```

The publisher does not need to know the subscribers.

This is particularly useful when:

> **Multiple independent services need to react to the same event.**

---

# 26. Pub/Sub vs Request/Reply

We have already discussed Request/Reply separately.

The conceptual difference is:

### Pub/Sub

```text
Order Service
      |
      | "Order created"
      v
    NATS
      |
      +----> Warehouse
      +----> Analytics
      +----> Audit
```

The publisher is announcing an event.

The publisher does not expect a business response.

---

### Request/Reply

```text
Interest Service
      |
      | "Calculate distribution"
      v
    NATS
      |
      v
Distribution
      |
      | response
      v
Interest Service
```

The requester expects a response.

Therefore:

> **Pub/Sub is primarily event notification. Request/Reply is RPC-style communication.**

---

# 27. A Practical Internal Architecture

Suppose your financial platform has:

```text
Interest Service
Distribution Service
Tax Service
Audit Service
Notification Service
```

A possible Core NATS event flow could be:

```text
Interest Service
      |
      | interest.calculated
      v
    NATS
      |
      +----> Distribution Service
      +----> Audit Service
      +----> Analytics
```

Then:

```text
Distribution Service
      |
      | distribution.completed
      v
    NATS
      |
      +----> Audit Service
      +----> Notification Service
```

This allows independent consumers to react to events.

---

# 28. Important Limitation for Financial Workflows

For your kind of financial workflow, don't automatically conclude:

> "We can replace every Kafka event with Core NATS Pub/Sub."

The delivery semantics matter.

If the event is:

```text
interest.calculated
```

and losing one event is unacceptable, Core NATS alone may not be sufficient.

For example:

```text
Interest calculated
       |
       v
distribution event
       |
       X
Distribution unavailable
       |
       v
Event lost
```

If that event represents a required business transition, you need durable semantics.

That is where JetStream should be evaluated.

A useful design question is:

> **Can I safely lose this event if the consumer is temporarily unavailable?**

If:

```text
YES
```

Core NATS may be appropriate.

If:

```text
NO
```

investigate JetStream.

---

# 29. Kafka Comparison

Since you already know Kafka, here is the useful comparison.

## NATS Core Pub/Sub

Conceptually:

```text
Publisher
    |
    v
 NATS
    |
    +---- Subscriber A
    +---- Subscriber B
    +---- Subscriber C
```

Core NATS:

- Primarily live messaging
- No persistence
- At-most-once
- No replay
- Subscriber must be present to receive the message
- Very lightweight
- Low-latency communication

---

## Kafka

Kafka is fundamentally based around a durable distributed log:

```text
Producer
    |
    v
Kafka Topic
    |
    +---- Partition 0
    +---- Partition 1
    +---- Partition 2
          |
          v
     Consumers
```

Kafka provides:

- Persistent messages
- Partitions
- Consumer offsets
- Replay
- Durable retention
- Consumer groups
- Ordered records within partitions

Therefore, the important distinction is not simply:

> "NATS vs Kafka."

It is:

> **Core NATS live messaging vs Kafka durable log semantics.**

And remember:

> NATS also has JetStream, which changes the comparison significantly because JetStream adds persistence and durable consumer capabilities.

---

# 30. When NATS Core Pub/Sub Is Attractive

Core NATS Pub/Sub can be a strong fit when:

### 1. Low latency matters

You want lightweight communication with very low messaging overhead.

### 2. The message is transient

Missing an individual message is acceptable.

### 3. Multiple services need the same event

For example:

```text
order.created
```

can be consumed by:

```text
Warehouse
Analytics
Audit
```

### 4. You want loose coupling

The publisher should not know individual subscribers.

### 5. You need lightweight internal communication

For example:

```text
Service A
   |
   v
 NATS
   |
   +---- Service B
   +---- Service C
```

---

# 31. When Core NATS Pub/Sub Is Not Enough

Core NATS alone may not be enough when you require:

- Durable message storage
- Replay
- Redelivery
- Consumer recovery
- Processing after downtime
- Business-critical event retention

For example:

```text
Payment completed
```

If the payment event must never be lost:

```text
Core NATS alone
      |
      X
Consumer offline
      |
      v
Message lost
```

You should evaluate JetStream.

---

# 32. 1 MB Payload Limit

Core NATS has a maximum message payload size.

By default, the server caps the payload at:

```text
1 MB
```

More precisely:

```text
1,048,576 bytes
```

The server communicates this limit to clients when the connection opens.

This means the client can know the maximum payload size.

---

# 33. Why Shouldn't We Put Large Data Into NATS Messages?

Suppose you have a 500 MB file.

Don't do:

```text
NATS message
    |
    v
500 MB file
```

Instead, store the large data somewhere appropriate:

```text
Object Storage
     |
     | file
     v
large-data.bin
```

Then publish a reference:

```json
{
  "file_id": "abc123",
  "location": "object-storage-key"
}
```

The consumer can then retrieve the data separately.

Conceptually:

```text
Publisher
    |
    | NATS message
    | { fileId: "abc123" }
    v
  NATS
    |
    v
Consumer
    |
    | fetch file
    v
Object Storage
```

This is a common distributed-system pattern:

> **Put metadata/reference in the message and large payloads in dedicated storage.**

---

# 34. What Happens When Payload Is Too Large?

An official client can detect that the payload exceeds the configured maximum.

Conceptually:

```text
Application
    |
    | publish(2 MB)
    v
NATS Client
    |
    X
Payload exceeds limit
```

The publish fails before the message is sent.

The connection can remain up.

The server also enforces the maximum as a backstop.

The important takeaway is:

> **NATS messages are intended to be relatively small messaging payloads, not arbitrary large-data transport.**

---

# 35. Short-Lived Publisher and Flush

Consider:

```text
publish()
exit()
```

The client may buffer the outgoing message and send it in the background.

If the application exits immediately:

```text
publish()
   |
   | message still buffered
   v
process exits
   |
   X
message may never reach NATS
```

For a short-lived publisher, you may need:

```text
publish()
   |
   v
flush()
   |
   v
exit()
```

The idea of `flush()` is:

> **Wait until the server has acknowledged the data sent before continuing.**

This is especially relevant for:

- CLI tools
- Batch utilities
- Short-lived jobs
- Tests
- Scripts

Long-running services generally don't have this particular lifecycle problem because they remain alive.

---

# 36. Echo Behavior

Suppose one NATS connection:

```text
Client A
```

does both:

```text
subscribe("orders.created")
```

and:

```text
publish("orders.created")
```

By default, the client can receive its own message.

Conceptually:

```text
Client A
   |
   | publish
   v
 NATS
   |
   | orders.created
   |
   +----> Client A
```

This behavior is called **echo**.

NATS clients can disable it using `NoEcho`.

---

# 37. Why Can Echo Matter?

Suppose a service publishes status updates:

```text
service.status
```

and also subscribes to:

```text
service.status
```

It could receive its own event.

In some designs that is perfectly fine.

In others, it could accidentally create a loop:

```text
Service
   |
   | status event
   v
 NATS
   |
   v
Same Service
   |
   | processes event
   | publishes another status
   v
 NATS
   |
   v
Same Service
   |
   ...
```

Therefore, when a service both publishes and subscribes to related subjects, understand whether self-delivery is expected.

---

# 38. Slow Consumers

Another important Core NATS behavior is the concept of a **slow consumer**.

Suppose messages arrive very quickly:

```text
Publisher
   |
   | 1000 msg/sec
   v
 NATS
   |
   v
Subscriber
   |
   | processes only 100 msg/sec
```

The subscriber cannot keep up.

Messages can accumulate for that subscriber.

Conceptually:

```text
NATS
 |
 +--> msg1
 +--> msg2
 +--> msg3
 +--> msg4
 +--> msg5
       |
       v
   Subscriber
   processing slowly
```

Eventually, if the subscriber falls too far behind, NATS can detect the slow consumer condition and close that connection.

The other subscribers are not necessarily affected.

---

# 39. How to Think About Slow Consumers

The problem is:

```text
Producer rate
      >
Consumer processing rate
```

For example:

```text
Producer = 10,000 msg/sec
Consumer = 1,000 msg/sec
```

The consumer cannot keep up.

The solution is not simply:

> "Make NATS store unlimited messages for the consumer."

Core NATS is not designed to provide durable consumer backlogs.

Instead, the application should consider:

- Faster processing
- Worker pools
- Backpressure
- Queue Groups
- JetStream
- Appropriate architecture

The correct solution depends on whether the messages are transient or must be retained.

---

# 40. Slow Consumer vs JetStream

This gives us another useful boundary.

### Core NATS

```text
Fast transient messaging
        |
        v
Consumer must keep up
```

### JetStream

```text
Durable stream
        |
        v
Consumer can process from retained messages
```

So if your architecture requires:

> "The consumer can be temporarily down or slow, but messages must remain available."

you are moving toward JetStream territory.

---

# 41. Core NATS Pub/Sub Mental Model

Keep this picture in your head:

```text
                         Publisher
                             |
                             |
                    publish(subject, msg)
                             |
                             v
                      +-------------+
                      | NATS Server |
                      +-------------+
                             |
                     Interest Graph
                             |
                +------------+------------+
                |            |            |
                v            v            v
           Subscriber A  Subscriber B  Subscriber C
                |            |            |
              copy         copy         copy
```

The important properties are:

```text
Subject-based routing
        +
Interest graph
        +
Fan-out
        +
Fire-and-forget
        +
At-most-once
        +
No persistence
```

---

# 42. Core NATS Pub/Sub vs Kafka — The Interview Answer

If someone asks:

> **"How is Core NATS Pub/Sub different from Kafka?"**

A good answer is:

> Core NATS is primarily a lightweight, real-time messaging system where publishers send messages to subjects and currently interested subscribers receive them. Core NATS is ephemeral and provides at-most-once delivery, so if a consumer is offline when a message is published, that message is not retained for it.
>
> Kafka, on the other hand, is fundamentally a durable distributed log. Messages are persisted in topics and partitions, consumers track offsets, and messages can be replayed. Kafka is therefore particularly strong when durable event streaming, retention, replay, and high-throughput stream processing are core requirements.
>
> NATS also has JetStream, which adds persistence, retention, acknowledgements, and redelivery, so the real comparison for a durable use case is often JetStream vs Kafka rather than Core NATS vs Kafka.

That is a much stronger answer than simply saying:

> "NATS is faster and Kafka is for big data."

---

# 43. How This Applies to Your Architecture

Suppose your system looks like:

```text
Interest Service
        |
        | interest.calculated
        v
      NATS
        |
        +----> Distribution
        +----> Audit
        +----> Analytics
```

Ask:

### Question 1

Are all these consumers interested in the same event?

If yes:

```text
Pub/Sub
```

may be appropriate.

### Question 2

Can Audit safely miss an event?

If yes, Core NATS may be sufficient.

### Question 3

Can Distribution safely miss an event?

If no, Core NATS alone is probably not sufficient.

### Question 4

Do we need replay?

If yes:

```text
JetStream / Kafka
```

should be evaluated.

### Question 5

Do multiple Distribution instances need to share the workload?

Then:

```text
Queue Groups
```

may be relevant.

This is how you should approach NATS design:

> **Don't choose a NATS feature because it exists. Choose it because it solves a specific requirement.**

---

# 44. Important Questions to Ask Before Choosing Core Pub/Sub

Whenever you see an event-driven requirement, ask:

### 1. Can the message be lost?

```text
YES → Core NATS may be suitable
NO  → Evaluate JetStream/Kafka
```

### 2. Does the consumer need replay?

```text
YES → Need durable retention
NO  → Core NATS may be enough
```

### 3. Does the consumer need to recover missed messages?

```text
YES → Need persistence
NO  → Core NATS may be enough
```

### 4. Should every subscriber receive the message?

```text
YES → Normal Pub/Sub
NO, one instance should process it → Queue Group
```

### 5. Is the message transient?

```text
YES → Core NATS is attractive
NO  → Consider JetStream/Kafka
```

---

# 45. Interview Questions

## Q1. What is Publish-Subscribe in NATS?

It is a messaging pattern where a publisher sends a message to a subject and every currently interested subscriber receives a copy.

---

## Q2. Does a subscriber consume the message from a shared queue?

No.

Normal Core NATS subscriptions provide independent copies to each matching subscriber.

---

## Q3. What happens if three subscribers subscribe to the same subject?

Each subscriber receives its own copy of the published message.

---

## Q4. What happens if nobody is subscribed?

The publish can still succeed, but the message is discarded because there is no interested subscriber.

---

## Q5. Does Core NATS retain messages?

No.

Core NATS is ephemeral.

For durable retention, look at JetStream.

---

## Q6. What delivery guarantee does Core NATS provide?

At-most-once delivery.

A subscriber can receive a message zero or one time.

---

## Q7. Does a successful publish mean that the subscriber processed the message?

No.

Publishing is fire-and-forget.

Publish success does not mean business processing succeeded.

---

## Q8. What is the interest graph?

It is the NATS server's in-memory representation of which clients are currently subscribed to which subjects.

---

## Q9. Why is the interest graph important?

It allows NATS to dynamically route messages to currently interested subscribers without the publisher maintaining a list of subscribers.

---

## Q10. What happens when a subscriber disconnects?

Its subscription/interest disappears from the active interest graph.

Messages published while it is disconnected are not retained by Core NATS for later delivery.

---

## Q11. What is fan-out?

One published message is delivered to multiple matching subscribers.

```text
Publisher
    |
    v
  NATS
  / | \
 /  |  \
A   B   C
```

---

## Q12. How is Pub/Sub different from Queue Groups?

Normal Pub/Sub:

```text
Message
   |
   +----> A
   +----> B
   +----> C
```

Queue Group:

```text
Message
   |
   +----> One group member
```

Queue Groups are used when multiple instances should share work rather than all process the same message.

---

## Q13. Why is Core NATS good for live telemetry?

Because the data can be transient.

If one telemetry update is missed, a newer update can replace it.

---

## Q14. Why might Core NATS be insufficient for a financial event?

Because a consumer can be offline when the message is published and Core NATS will not retain the message for later processing.

---

## Q15. What should I use if I need replay?

Core NATS does not provide replay.

You need a durable mechanism such as JetStream or Kafka, depending on the broader requirements.

---

## Q16. What is the default maximum Core NATS payload?

The documentation describes a default maximum of:

```text
1 MB
```

or:

```text
1,048,576 bytes
```

---

## Q17. What should I do with very large payloads?

Store the large data externally and publish a reference such as:

```text
object-store-key
```

rather than putting the entire large object into the NATS message.

---

## Q18. What happens if a short-lived application publishes and immediately exits?

The message may still be buffered by the client.

A short-lived publisher should flush/drain appropriately before exiting so the message reaches the server.

---

## Q19. Can a client receive its own published messages?

Yes.

Echo is enabled by default.

It can be disabled using `NoEcho`.

---

## Q20. What is a slow consumer?

A subscriber that cannot process matching messages quickly enough and builds up a backlog.

If it becomes sufficiently slow, NATS can close the connection to protect the system.

---

# 46. The Most Important Mental Model

Remember this:

```text
                    PUBLISH
                       |
                       v
                 +-----------+
                 |    NATS   |
                 |   Server  |
                 +-----------+
                       |
                 Interest Graph
                       |
          +------------+------------+
          |            |            |
          v            v            v
       Service A    Service B    Service C
          |            |            |
        copy         copy         copy
```

And remember the behavior:

```text
Publisher
    |
    | publish()
    v
NATS
    |
    +----> interested subscriber
    +----> interested subscriber
    +----> interested subscriber
```

But:

```text
No subscriber
      |
      v
Message discarded
```

And:

```text
Subscriber offline
      |
      v
Message not retained
```

Therefore:

```text
Core NATS Pub/Sub
       |
       +-- Subject-based routing
       +-- Interest graph
       +-- Fan-out
       +-- Fire-and-forget
       +-- At-most-once
       +-- No persistence
```

---

# 47. Key Takeaways

1. **Publish-Subscribe is one of the fundamental Core NATS communication patterns.**

2. A publisher publishes a message to a **subject**.

3. Subscribers register interest in subjects.

4. Every matching normal subscriber receives its **own copy**.

5. Normal Pub/Sub is **not a shared queue**.

6. Multiple subscribers naturally create **fan-out**.

7. Publishing is **fire-and-forget**.

8. A successful publish does **not** mean that subscribers processed the message.

9. If nobody is interested in a subject, the message is **discarded**.

10. If a subscriber is offline when a message is published, Core NATS does **not** retain the message for it.

11. Core NATS provides **at-most-once delivery**.

12. The NATS server maintains an **interest graph** to determine where messages should be routed.

13. The publisher does not need to know the list or location of subscribers.

14. This provides **loose coupling and location transparency**.

15. If multiple service instances need to **share work**, use Queue Groups rather than normal Pub/Sub.

16. Core NATS is a good fit for **transient, real-time communication**.

17. Core NATS alone is not sufficient when messages must survive consumer downtime.

18. For persistence, replay, retention, acknowledgements, and redelivery, investigate **JetStream**.

19. Core NATS messages have a default maximum payload of **1 MB**.

20. Large data should generally be stored externally, with the NATS message carrying a reference.

21. Short-lived publishers should flush/drain before exiting.

22. Echo is enabled by default, so a connection can receive messages it publishes itself.

23. Slow consumers can be disconnected when they cannot keep up.

24. **Core NATS vs Kafka is primarily a comparison of messaging models and durability semantics, not simply performance.**

25. The most important design question is:

> **"Can I safely lose this message?"**

If the answer is **yes**, Core NATS may be an excellent fit.

If the answer is **no**, start evaluating **JetStream or Kafka**.