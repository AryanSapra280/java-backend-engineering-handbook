# Core NATS Deep Dive

## 1. What is Core NATS?

Core NATS is the fundamental messaging layer of NATS.

At its simplest, Core NATS allows applications to:

1. Connect to a NATS server
2. Publish messages to subjects
3. Subscribe to subjects
4. Receive messages from matching subjects

The fundamental model is:

```text
Publisher
    |
    | message
    v
NATS Server
    |
    | route
    v
Subscriber
```

Core NATS is primarily designed for **fast, lightweight, real-time communication** between applications and services.

It is not fundamentally a persistent message store.

---

# 2. The most important property of Core NATS

The single most important concept to remember is:

> **Core NATS provides at-most-once delivery.**

A message is delivered to interested subscribers that are connected and subscribed when the message is published.

If there is no interested subscriber at that moment, Core NATS does not store the message for that subscriber.

A useful mental model is:

```text
                Core NATS
                    |
        "Who is interested right now?"
                    |
          +---------+---------+
          |         |         |
          v         v         v
       Service A Service B Service C
```

Core NATS is concerned primarily with **live message delivery**, not maintaining a durable history of messages.

---

# 3. What does "at-most-once" mean?

At-most-once means a message will be delivered **zero or one time** to a particular subscriber.

It does not mean:

> "Every message will definitely be delivered."

Instead:

```text
0 deliveries
OR
1 delivery
```

but Core NATS does not provide the durable redelivery mechanism that would make the message available again after a failure.

For example:

```text
Publisher
    |
    | orders.created
    v
  NATS
    |
    v
Warehouse
```

If Warehouse is connected and subscribed:

```text
Publisher → NATS → Warehouse
```

The message can be delivered.

But if Warehouse is offline:

```text
Publisher
    |
    v
  NATS
    |
    X
Warehouse offline
```

Core NATS does not retain the message and wait for Warehouse to return.

When Warehouse comes back:

```text
Warehouse comes back
        |
        v
      NATS
```

it does not automatically receive the old message.

---

# 4. Why is ephemeral messaging useful?

At first, losing messages may sound like a weakness.

However, not every message needs to survive.

Consider a live stock price feed:

```text
AAPL = 220
AAPL = 221
AAPL = 222
AAPL = 223
```

If a dashboard is offline for a few seconds, it may not need every historical price update.

It only needs the current price.

Similarly, Core NATS can be appropriate for data such as:

* Live market prices
* Current temperature
* Telemetry
* Presence information
* Current service state
* Cache invalidation
* Transient notifications
* Real-time status updates

The previous value may become irrelevant when a newer value arrives.

Therefore:

> **At-most-once is not inherently bad. It is appropriate when the message is transient or can be safely missed.**

---

# 5. When is at-most-once dangerous?

At-most-once becomes problematic when every message represents an important business operation.

Consider a financial system:

```text
Interest Service
       |
       | DistributionRequested
       v
     NATS
       |
       v
Distribution Service
```

Suppose the message means:

> Distribute ₹50,000 for account XYZ.

If Distribution Service is down:

```text
Interest
   |
   v
 NATS
   |
   X
Distribution unavailable
```

With Core NATS, that message can be lost.

For a financial transaction, this may be unacceptable.

In this scenario, we need durable messaging.

That is where **JetStream** comes in.

---

# 6. Core NATS vs JetStream

NATS can be viewed as having two important layers:

```text
                         NATS
                          |
             +------------+------------+
             |                         |
             v                         v
        Core NATS                  JetStream
             |                         |
      Live messaging           Durable messaging
             |                         |
       Pub/Sub                  Persistence
       Request/Reply            Acknowledgements
       Queue Groups             Redelivery
                                Replay
                                Retention
                                Consumer state
```

### Core NATS

Think:

> "I need to communicate with services that are currently available."

### JetStream

Think:

> "I need the message to survive and be processed reliably."

JetStream builds durable messaging capabilities on top of NATS.

---

# 7. Core NATS does not have a persistent broker queue

Core NATS is not fundamentally:

```text
Producer
    |
    v
+----------------+
| Persistent      |
| Broker Queue   |
+----------------+
    |
    v
Consumer
```

Instead, it is closer to:

```text
Publisher
    |
    v
   NATS
    |
    | Check current subscriptions
    |
    +------> Subscriber A
    |
    +------> Subscriber B
```

NATS routes the message according to the current subscription interest.

This is why the NATS documentation describes the concept as an **interest graph**.

---

# 8. The Interest Graph

Suppose the subject is:

```text
orders.created
```

Three services exist:

```text
Warehouse
Analytics
Notifications
```

Only two are subscribed:

```text
Warehouse      → interested
Analytics      → interested
Notifications  → not interested
```

When a message is published:

```text
                 NATS
                  |
          orders.created
             /         \
            v           v
       Warehouse     Analytics

Notifications
      X
```

The publisher does not need to know which services are interested.

NATS uses the subject and subscription information to route the message.

This provides loose coupling between publishers and subscribers.

---

# 9. Core Components

## 9.1 NATS Server

The NATS server is the messaging infrastructure.

Applications establish connections to it.

The server receives messages and routes them to matching subscribers.

```text
Application A
     |
     | connection
     v
+-------------+
| NATS Server |
+-------------+
     |
     | routing
     v
Application B
```

A NATS deployment can begin with a single server and can later be deployed as a cluster.

---

## 9.2 Publisher

A publisher is an application that sends a message.

For example:

```text
Interest Service
       |
       | publish
       v
distribution.request
```

The publisher does not need to know which specific service instance will receive the message.

It publishes to a subject.

---

## 9.3 Subscriber

A subscriber is an application that listens for messages on one or more subjects.

For example:

```text
Distribution Service
       |
       | subscribe
       v
distribution.request
```

When a matching message is published, the subscriber receives it.

---

## 9.4 Subject

A subject is a logical name used for routing messages.

Examples:

```text
orders.created
orders.shipped
orders.cancelled

interest.calculated
distribution.request
distribution.completed
```

A publisher publishes to a subject:

```text
orders.created
```

A subscriber subscribes to a subject:

```text
orders.created
```

NATS uses the subject to determine which subscriptions match the message.

Subjects become one of the most important concepts in NATS.

---

# 10. Subjects are logical addresses

A useful way to think about a subject is:

> **A subject is a logical destination for a message.**

Compare this with direct communication.

### HTTP

```text
Interest Service
       |
       | HTTP
       v
http://distribution-service:8080
```

The caller communicates with a network endpoint.

### NATS

```text
Interest Service
       |
       | publish
       v
distribution.request
       |
       v
     NATS
```

The application communicates with a logical subject.

This contributes to **location transparency**.

The publisher does not need to know the physical location of the service.

---

# 11. The four major Core NATS communication patterns

Core NATS uses the same fundamental concepts—subjects and subscriptions—to implement several communication patterns.

The major patterns are:

```text
Core NATS
    |
    +---- Publish / Subscribe
    |
    +---- Request / Reply
    |
    +---- Queue Groups
    |
    +---- Scatter / Gather
```

These are not four completely different messaging systems.

They are different ways of using the same underlying messaging primitives.

---

# 12. Publish / Subscribe

Publish/Subscribe is the simplest Core NATS pattern.

A publisher sends a message:

```text
Publisher
    |
    | orders.created
    v
  NATS
```

Multiple subscribers can be interested:

```text
                  NATS
                   |
           orders.created
                   |
        +----------+----------+
        |          |          |
        v          v          v
    Warehouse  Analytics  Notifications
```

Each matching subscriber receives its own copy of the message.

The publisher doesn't need to know:

* Who the subscribers are
* How many subscribers exist
* Where they are running
* Whether new subscribers will appear later

This creates loose coupling.

---

# 13. When should Publish / Subscribe be used?

Pub/Sub is useful when:

> **One event can be relevant to multiple independent consumers.**

For example:

```text
order.created
```

may be consumed by:

```text
Warehouse
Analytics
Audit
Notifications
```

The publisher doesn't need to call each service separately.

Instead:

```text
Order Service
      |
      | order.created
      v
     NATS
      |
      +----> Warehouse
      +----> Analytics
      +----> Audit
      +----> Notifications
```

This is an example of event-driven communication.

---

# 14. Request / Reply

Request/Reply is different from Pub/Sub.

Pub/Sub means:

> "Something happened."

Request/Reply means:

> **"Please perform an operation and give me a response."**

Example:

```text
Inventory Service
       |
       | request
       v
      NATS
       |
       v
Warehouse Service
       |
       | reply
       v
      NATS
       |
       v
Inventory Service
```

This creates a synchronous request-response interaction over NATS messaging.

---

# 15. NATS Request / Reply vs HTTP

For internal service communication, we can conceptually have:

### HTTP

```text
Service A
    |
    | HTTP request
    v
Service B
    |
    | HTTP response
    v
Service A
```

### NATS Request / Reply

```text
Service A
    |
    | NATS request
    v
   NATS
    |
    v
Service B
    |
    | NATS reply
    v
   NATS
    |
    v
Service A
```

The application-level pattern is similar:

```text
request → response
```

But the underlying communication infrastructure is different.

NATS provides messaging-level concepts such as:

* Subjects
* Queue Groups
* Request/Reply
* Location transparency
* NATS-level routing

This makes NATS Request/Reply a possible option for internal service-to-service RPC-style communication.

---

# 16. Queue Groups

Suppose we have three instances of Distribution Service:

```text
Distribution Pod 1
Distribution Pod 2
Distribution Pod 3
```

All three subscribe to:

```text
distribution.request
```

If they are ordinary subscribers:

```text
                  NATS
                   |
          distribution.request
             /       |       \
            v        v        v
          Pod 1    Pod 2    Pod 3
```

Every pod can receive the message.

That may be incorrect if the message represents work that should be performed only once.

For example, we don't want:

```text
Order 123
   |
   +----> Pod 1 packs it
   +----> Pod 2 packs it
   +----> Pod 3 packs it
```

Queue Groups solve this problem.

---

# 17. Queue Group behavior

The instances join the same queue group:

```text
                  NATS
                   |
          distribution.request
                   |
        +----------+----------+
        |          |          |
       Pod 1      Pod 2      Pod 3
       group:     group:     group:
       workers    workers    workers
```

For each message, NATS delivers it to one member of the queue group.

For example:

```text
Message 1 → Pod 1
Message 2 → Pod 3
Message 3 → Pod 2
Message 4 → Pod 1
```

This provides load balancing among the service instances.

A useful mental model is:

> **Normal subscribers = broadcast to all interested subscribers.**

> **Queue group subscribers = one member of the group receives the message.**

---

# 18. Queue Groups vs Kafka Consumer Groups

There is a useful analogy to Kafka.

### NATS Queue Group

```text
Subject
   |
   +---- Pod 1
   +---- Pod 2
   +---- Pod 3
```

One group member receives a given message.

### Kafka Consumer Group

```text
Topic
   |
   +---- Partition 1 → Consumer A
   +---- Partition 2 → Consumer B
   +---- Partition 3 → Consumer C
```

Consumers in the same group divide work.

However, these should not be considered identical concepts.

Kafka consumer groups are tightly connected to Kafka's partitioned durable log and offset model.

NATS queue groups are primarily a **messaging/load-balancing mechanism**.

---

# 19. Scatter / Gather

Scatter/Gather is useful when one request should be sent to multiple responders and the responses should be collected.

For example, imagine three shipping providers:

```text
FedEx
UPS
DHL
```

The application wants quotes.

```text
              Quote Request
                    |
                    v
                  NATS
               /    |    \
              v     v     v
            FedEx  UPS    DHL
              |     |      |
              v     v      v
             $10   $12     $8
               \     |     /
                \    |    /
                 \   |   /
                   Gather
                      |
                      v
                 Best Quote
```

The request is scattered to multiple responders, and their replies are gathered.

This pattern can be useful for:

* Multiple pricing providers
* Aggregating service responses
* Parallel queries
* Distributed searches

---

# 20. Connections

Before a client can publish or subscribe, it needs a connection to the NATS server.

Conceptually:

```text
Java Service
     |
     | NATS client
     |
     | long-lived connection
     v
NATS Server
```

Applications normally maintain a long-lived connection rather than creating a new connection for every message.

This is important for achieving low latency and efficient communication.

---

# 21. Connection lifecycle

A production application must consider what happens when the NATS connection is interrupted.

Conceptually:

```text
Application
     |
     | connection
     X
NATS Server
```

The NATS client can detect connection failures and attempt to reconnect according to its configuration.

This means the application should treat the NATS connection as a long-lived resource whose lifecycle needs to be managed.

Later topics will cover:

* Connection failure
* Reconnection
* Connection callbacks
* In-flight messages
* What happens during reconnect
* Server failure

These are important production concerns.

---

# 22. PING / PONG

NATS uses PING/PONG heartbeats as part of connection management.

Conceptually:

```text
Client → PING
Server → PONG
```

This helps determine whether the connection is alive.

This is particularly important for detecting network problems such as:

* Broken connections
* Half-open connections
* Network partitions
* Unresponsive servers

The exact protocol behavior should be studied separately when learning connection lifecycle internals.

---

# 23. Core NATS has no application-level ACK

Core NATS does not provide a persistent acknowledgement mechanism like:

```text
Consumer
    |
    | "I successfully processed this message."
    v
NATS
```

Instead, the basic flow is:

```text
Publisher
    |
    v
NATS
    |
    v
Subscriber
```

Once the message has been delivered, Core NATS does not maintain durable processing state for that subscriber.

This is another reason Core NATS is fundamentally different from a durable event log such as Kafka.

---

# 24. Delivery is not the same as successful processing

This is a very important distributed-systems distinction.

Suppose:

```text
NATS
  |
  | message delivered
  v
Distribution Service
  |
  | process
  v
Database
  |
  X
Database failure
```

The message was delivered to Distribution Service.

But the business operation did not necessarily succeed.

Core NATS does not know:

> "The database transaction failed."

It simply provides the messaging mechanism.

Durable processing semantics such as:

* ACK
* Redelivery
* Consumer state
* Retry
* Persistent messages

are provided through JetStream.

---

# 25. Core NATS and the financial workflow

Consider:

```text
Interest Service
       |
       | DistributionRequested
       v
     NATS
       |
       v
Distribution Service
```

There are three possible requirements.

### Requirement 1: I need an immediate response

Use Request/Reply:

```text
Interest
   |
   | request
   v
 NATS
   |
   v
Distribution
   |
   | reply
   v
 NATS
   |
   v
Interest
```

This is similar to internal RPC.

---

### Requirement 2: I don't need a response

Use Pub/Sub:

```text
Interest
   |
   | event
   v
 NATS
   |
   +----> Distribution
   +----> Audit
   +----> Analytics
```

---

### Requirement 3: The business operation must not be lost

Core NATS alone is not sufficient.

Use JetStream:

```text
Interest
   |
   v
JetStream
   |
   | persistent message
   v
Distribution
```

Now the message can survive consumer downtime according to the configured retention and consumer behavior.

---

# 26. Core NATS vs Kafka

Kafka and Core NATS overlap in some areas, but their fundamental models are different.

| Capability                             | Core NATS           | Kafka                     |
| -------------------------------------- | ------------------- | ------------------------- |
| Primary abstraction                    | Messaging fabric    | Distributed event log     |
| Routing                                | Subjects            | Topics + partitions       |
| Persistent message storage             | No                  | Yes                       |
| Replay                                 | No                  | Yes                       |
| Durable consumer state                 | No                  | Yes                       |
| Pub/Sub                                | Yes                 | Yes                       |
| Request/Reply                          | First-class pattern | Possible, but not primary |
| Load balancing                         | Queue Groups        | Consumer Groups           |
| Offline consumer receives old messages | No                  | Yes, while retained       |
| Long-lived event history               | No                  | Yes                       |
| Lightweight service communication      | Strong use case     | Possible                  |
| Durable asynchronous processing        | JetStream           | Kafka                     |

The important mental model is:

```text
Core NATS
    ↓
Fast live messaging


JetStream
    ↓
NATS + persistence + durable processing


Kafka
    ↓
Durable distributed event-streaming platform
```

This is a high-level comparison. The detailed comparison between Kafka and JetStream will come later.

---

# 27. Core NATS vs HTTP/gRPC for internal communication

For internal service communication, there are several options.

### HTTP

```text
Service A
    |
    | HTTP
    v
Service B
```

### gRPC

```text
Service A
    |
    | gRPC
    v
Service B
```

### NATS Request/Reply

```text
Service A
    |
    | NATS Request
    v
NATS
    |
    v
Service B
    |
    | NATS Reply
    v
NATS
    |
    v
Service A
```

NATS is particularly interesting when the system already uses NATS as its internal messaging fabric.

It can provide:

* Subject-based addressing
* Request/Reply
* Queue Groups
* Pub/Sub
* Low-latency messaging
* Location transparency
* Service-level routing

However, this does not mean:

> "NATS is always better than HTTP/gRPC."

HTTP/gRPC may still be preferable depending on:

* API contracts
* Ecosystem
* Observability
* Tooling
* External compatibility
* Protocol requirements
* Team familiarity
* Existing architecture

The choice should be driven by the communication requirements.

---

# 28. Core NATS payload size

Core NATS has a maximum message payload size of approximately **1 MB** under the standard configuration.

This reinforces an important architectural principle:

> **Messaging systems are generally not a replacement for object/file storage.**

If a message refers to a large file, a better pattern is often:

```text
Object Storage
      |
      | file
      v
large-data-store

      +

NATS
  |
  | metadata/reference
  v
Consumer
```

For example:

```json
{
  "fileId": "file-123",
  "location": "object-storage/file-123"
}
```

rather than placing a huge file directly inside the message.

---

# 29. Why Core NATS is fast

Core NATS is designed around a relatively simple messaging model:

```text
Publisher
    |
    v
NATS
    |
    v
Subscriber
```

It doesn't need to perform the same durable-log responsibilities as Kafka for every message.

For example, Core NATS does not need to:

* Persist every message to a durable log
* Maintain long-term message history
* Track Kafka-style offsets
* Support replay of historical messages

This allows NATS to focus heavily on:

* Low latency
* High message throughput
* Lightweight resource usage
* Simple messaging

This does not mean every NATS workload will always outperform every Kafka workload.

Performance depends on:

* Hardware
* Message size
* Network
* Number of subscribers
* Persistence
* Replication
* Configuration
* Workload pattern

The correct takeaway is:

> **Low-latency, lightweight messaging is one of the core design goals of Core NATS.**

---

# 30. Core NATS architectural philosophy

The Core NATS model can be summarized as:

```text
                 CORE NATS

           Subjects + Interest
                    |
        +-----------+-----------+
        |           |           |
        v           v           v
      Pub/Sub    Request/Reply Queue Groups
                                |
                                v
                         Load-balanced work

                    +
                 Connections
                    |
                    v
              NATS Server
```

The same basic messaging infrastructure enables several communication patterns.

This is one of the strengths of NATS:

> **A relatively small set of primitives can be composed into different distributed communication patterns.**

---

# 31. Common misconceptions

## Misconception 1: Core NATS is a durable queue

No.

Core NATS is ephemeral.

If a subscriber isn't present when a message is published, Core NATS does not retain that message for the subscriber.

For durable messaging, use JetStream.

---

## Misconception 2: At-most-once means NATS is unreliable

Not necessarily.

At-most-once is a deliberate semantic choice.

For transient real-time data, losing an old message may be perfectly acceptable.

For important business events, use durable messaging such as JetStream.

---

## Misconception 3: Queue Groups mean messages are stored in a queue

Not in the traditional persistent-queue sense.

Queue Groups provide **load balancing among active subscribers**.

They do not turn Core NATS into a durable message queue.

---

## Misconception 4: NATS is just Kafka but faster

Incorrect.

Kafka's core abstraction is a durable distributed event log.

Core NATS's core abstraction is live messaging using subjects and subscriptions.

JetStream introduces persistence to NATS, creating more overlap with Kafka.

---

## Misconception 5: NATS completely replaces HTTP/gRPC

No.

NATS can be used for internal service-to-service communication, particularly through Request/Reply, but HTTP and gRPC remain useful and may be better choices in some architectures.

---

## Misconception 6: Message delivery means business processing succeeded

No.

Delivery only means the message reached the subscriber.

The business operation may still fail afterward.

Reliable processing semantics require additional mechanisms.

---

# 32. Applying Core NATS to a microservice architecture

Consider:

```text
                   NATS
                    |
        +-----------+-----------+
        |           |           |
        v           v           v
     Interest   Distribution   Tax
      Service      Service    Service
```

Possible communication patterns:

### Request/Reply

```text
Interest
    |
    | calculate/distribute request
    v
NATS
    |
    v
Distribution
    |
    | response
    v
NATS
    |
    v
Interest
```

Use when the caller needs an immediate response.

### Pub/Sub

```text
Interest
    |
    | interest.calculated
    v
NATS
    |
    +----> Distribution
    +----> Audit
    +----> Analytics
```

Use when multiple services can independently react to an event.

### Queue Group

```text
                    NATS
                     |
             distribution.request
                     |
          +----------+----------+
          |          |          |
       Worker 1   Worker 2   Worker 3
       same queue group
```

Use when multiple instances collectively process work and each message should be handled by one member.

### JetStream

```text
Interest
    |
    v
JetStream
    |
    | persisted
    v
Distribution
```

Use when the message must survive consumer downtime and reliable asynchronous processing is required.

---

# 33. A practical decision framework

When deciding what NATS mechanism to use, ask:

### Question 1

**Do I need a response immediately?**

Yes:

```text
Request/Reply
```

No:

Continue.

### Question 2

**Can multiple services independently consume the event?**

Yes:

```text
Pub/Sub
```

No, and multiple service instances need to share work:

```text
Queue Group
```

### Question 3

**Can the message safely disappear if the consumer is unavailable?**

Yes:

```text
Core NATS
```

No:

```text
JetStream
```

This is a very useful mental decision tree:

```text
                   Need communication?
                          |
                          v
                 Need immediate reply?
                    /            \
                  YES             NO
                   |               |
             Request/Reply     Need durability?
                                /          \
                              YES           NO
                               |             |
                           JetStream      Core NATS
                                             |
                                    +--------+--------+
                                    |                 |
                               Broadcast          Load balance
                                    |                 |
                                  Pub/Sub         Queue Group
```

---

# 34. Interview questions to prepare

### Q1. What is Core NATS?

Core NATS is the lightweight messaging layer of NATS that provides subject-based communication through patterns such as Pub/Sub, Request/Reply, Queue Groups, and Scatter/Gather.

---

### Q2. What delivery guarantee does Core NATS provide?

Core NATS provides at-most-once delivery to currently interested subscribers. Messages are not persisted for offline subscribers.

---

### Q3. Why would anyone want at-most-once delivery?

For transient data such as live prices, telemetry, current state, or cache invalidations, older messages may no longer be useful. Avoiding persistence allows a lightweight and low-latency messaging system.

---

### Q4. What happens if a subscriber is offline?

In Core NATS, the message is not retained for that subscriber. When the subscriber reconnects, it will not automatically receive the missed message.

---

### Q5. How does NATS support load balancing?

Through Queue Groups. Multiple subscribers join the same queue group, and each message is delivered to one member of the group.

---

### Q6. How is Request/Reply implemented conceptually?

A requester publishes a request and provides a reply destination. The responder processes the request and publishes the response to that reply destination.

The detailed `_INBOX` mechanism will be covered in the Request/Reply chapter.

---

### Q7. Can Core NATS replace HTTP for internal communication?

Yes, in some architectures.

NATS Request/Reply can provide synchronous request-response communication between internal services. However, HTTP/gRPC may still be preferable depending on the system's requirements and existing architecture.

---

### Q8. Why would I use JetStream?

When messages need to survive consumer downtime, be persisted, replayed, acknowledged, or redelivered, JetStream provides the durable messaging capabilities that Core NATS intentionally does not.

---

### Q9. What is the biggest difference between Core NATS and Kafka?

Core NATS is primarily a live messaging system based on subjects and subscriptions, while Kafka is fundamentally a durable distributed event log based on topics and partitions.

---

# 35. Key mental models

Remember these five statements:

### 1.

> **Core NATS = live messaging.**

### 2.

> **Subject = logical destination used for routing.**

### 3.

> **Subscription = interest in a subject.**

### 4.

> **Queue Group = load-balanced consumption among service instances.**

### 5.

> **JetStream = persistence and durable messaging when Core NATS isn't enough.**

And the most important distinction:

```text
Core NATS
"I need to communicate now."


JetStream
"I need this message to survive."


Kafka
"I need a durable distributed event stream/log."
```

These are mental models, not absolute product boundaries. Real architecture decisions require looking at the exact workload and required semantics.

---

# 36. Current learning map

Core NATS should now be studied in the following order:

```text
Core NATS
    |
    v
Connecting
    |
    v
Publish / Subscribe
    |
    v
Subjects & Wildcards
    |
    v
Request / Reply
    |
    v
Queue Groups
    |
    v
Scatter / Gather
    |
    v
Message Headers
    |
    v
Subject Mapping
    |
    v
Connection Lifecycle
    |
    v
Debugging Delivery
    |
    v
JetStream
```

The current chapter provides the high-level mental model.

The following chapters should add the implementation details and failure semantics rather than repeating the same definitions.

---

# 37. Final summary

Core NATS is the foundation of the NATS messaging system.

Its basic model is:

```text
Publisher
    |
    | subject + message
    v
NATS
    |
    | route based on subscriptions
    v
Subscriber
```

Its defining characteristic is that it is **ephemeral and at-most-once**.

It does not persist messages for offline subscribers.

This makes Core NATS particularly useful for fast, transient communication where losing a message is acceptable.

Core NATS uses the same fundamental concepts—**subjects and subscriptions**—to implement multiple communication patterns:

```text
                  Core NATS
                      |
          +-----------+-----------+
          |           |           |
       Pub/Sub    Request/Reply Queue Groups
                                  |
                            Scatter/Gather
```

For internal microservice communication:

```text
HTTP/gRPC
    |
    | synchronous communication
    v
Service B
```

can sometimes be replaced by:

```text
NATS Request/Reply
    |
    | synchronous communication
    v
Service B
```

while asynchronous communication can use:

```text
Core NATS Pub/Sub
```

and durable asynchronous processing can use:

```text
JetStream
```

The most important progression to remember is:

```text
Core NATS
    ↓
Fast live communication

JetStream
    ↓
Durable/reliable messaging

Kafka
    ↓
Durable distributed event streaming
```

Understanding this distinction is the foundation for making good NATS architecture decisions later.
