# What is NATS?

## 1. Introduction

NATS is an open-source messaging system that allows applications, services, and components to communicate with each other through messages.

The key idea is **decoupled communication**.

Instead of one application directly communicating with another application using its network address, the applications communicate through NATS.

```text
Direct communication

+------------------+          +----------------------+
| Interest Service | -------> | Distribution Service |
+------------------+   HTTP    +----------------------+
```

The Interest Service needs to know how to reach the Distribution Service.

With NATS:

```text
+------------------+
| Interest Service |
+------------------+
        |
        | message
        v
+------------------+
|       NATS       |
+------------------+
        |
        | message
        v
+----------------------+
| Distribution Service |
+----------------------+
```

The Interest Service does not need to know the network location of the Distribution Service.

It publishes a message to a **subject**, and NATS is responsible for routing that message to the appropriate subscriber.

This gives applications **location transparency and loose coupling**.

---

# 2. What problem does NATS solve?

Before understanding NATS, it is useful to understand the problem it addresses.

Suppose we have several microservices:

```text
                 +----------------+
                 | Interest       |
                 | Service        |
                 +----------------+
                         |
                         v
                 +----------------+
                 | Distribution   |
                 | Service        |
                 +----------------+
                         |
                         v
                 +----------------+
                 | Tax Service    |
                 +----------------+
                         |
                         v
                 +----------------+
                 | Settlement     |
                 | Service        |
                 +----------------+
```

If every service directly calls every other service, several problems appear.

### 2.1 Tight coupling

The Interest Service needs to know:

* Where Distribution Service is running
* Which protocol it uses
* Which port it uses
* How to discover it
* How to handle failures
* How to retry requests
* Which instance should receive the request

As the system grows, this becomes increasingly complicated.

---

### 2.2 Service scaling

Suppose Distribution Service has three instances:

```text
Distribution Service

Pod 1
Pod 2
Pod 3
```

Now the caller needs some mechanism to determine which instance should receive the request.

Usually this requires additional infrastructure such as:

* Service discovery
* Load balancers
* Service mesh
* DNS
* Client-side load balancing

NATS can provide messaging-level routing and load balancing patterns.

---

### 2.3 Temporal coupling

With direct synchronous communication:

```text
Interest
   |
   | request
   v
Distribution
```

Interest generally needs Distribution to be available at that moment.

If Distribution is unavailable, the request fails.

Messaging can reduce this coupling.

For example, with durable messaging through JetStream:

```text
Interest
   |
   | publish
   v
JetStream
   |
   | later
   v
Distribution
```

The producer and consumer do not necessarily have to be available at exactly the same time.

---

# 3. The basic NATS mental model

The simplest mental model is:

> **Applications connect to NATS and communicate by publishing and subscribing to subjects.**

```text
                    NATS
                     |
       +-------------+-------------+
       |             |             |
       v             v             v
  Service A      Service B      Service C
```

A service can:

* Publish messages
* Subscribe to messages
* Request another service to perform an operation
* Reply to requests
* Join a queue group for load-balanced processing

NATS becomes the communication layer between the applications.

---

# 4. Core components

There are four basic concepts to understand initially.

## 4.1 NATS Server

The NATS server is the messaging infrastructure.

Applications connect to the NATS server.

The server receives messages and routes them to the appropriate subscribers.

```text
Publisher
    |
    | publish
    v
+-----------+
| NATS      |
| Server    |
+-----------+
    |
    | route
    v
Subscriber
```

A NATS deployment can start with a single server and can scale to clusters of servers.

---

## 4.2 Publisher

A publisher is an application that sends a message.

For example:

```text
Interest Service
       |
       | publish
       v
distribution.request
```

The publisher doesn't necessarily need to know which service will process the message.

It only needs to publish to the appropriate subject.

---

## 4.3 Subscriber

A subscriber is an application that listens for messages.

For example:

```text
Distribution Service
        |
        | subscribe
        v
distribution.request
```

When a matching message arrives, the subscriber receives it.

---

## 4.4 Subject

A subject is a name used by NATS for message routing.

Examples:

```text
interest.calculated
distribution.request
distribution.completed
tax.calculated
settlement.completed
```

A publisher publishes to a subject:

```text
publish("interest.calculated", message)
```

A subscriber subscribes to a subject:

```text
subscribe("interest.calculated")
```

NATS uses the subject to determine which subscribers should receive the message.

---

# 5. Why does NATS use subjects?

The subject acts as a logical address for a message.

Compare this with direct communication.

### Direct communication

```text
Interest Service
       |
       | HTTP
       v
http://distribution-service:8080
```

The caller is tied to the network location of the service.

### NATS

```text
Interest Service
       |
       | publish
       v
"distribution.request"
       |
       v
      NATS
```

The publisher is communicating with a **logical destination**, not a physical network address.

This is an important part of NATS's location transparency.

---

# 6. Location transparency

Location transparency means that the publisher doesn't need to know where the subscriber is physically running.

Suppose Distribution Service runs as:

```text
Pod 1 → 10.0.1.10
Pod 2 → 10.0.1.11
Pod 3 → 10.0.1.12
```

The Interest Service doesn't need to know these IP addresses.

It can simply publish:

```text
distribution.request
```

NATS handles the routing.

This becomes particularly useful in distributed systems where:

* Pods are frequently created and destroyed
* Services scale horizontally
* Services move between nodes
* Services run in multiple locations
* Network topology changes

The application communicates with a logical messaging destination rather than a specific server address.

---

# 7. Core NATS and JetStream

One of the most important concepts to understand early is that **NATS is not only one messaging behavior**.

NATS provides:

```text
                         NATS
                          |
             +------------+------------+
             |                         |
             v                         v
        Core NATS                 JetStream
```

These solve somewhat different problems.

---

# 8. Core NATS

Core NATS is the fundamental messaging layer.

It provides patterns such as:

* Publish/Subscribe
* Request/Reply
* Queue Groups

The focus is:

> **Fast, lightweight communication between applications.**

A simplified flow:

```text
Publisher
    |
    | message
    v
  NATS
    |
    v
Subscriber
```

Core NATS is primarily concerned with **live message delivery**.

It does not fundamentally behave like a persistent distributed event log.

If a subscriber is not available when a message is published, Core NATS does not automatically turn that message into a durable backlog for that subscriber.

This distinction is extremely important when comparing Core NATS with Kafka.

---

# 9. JetStream

JetStream adds persistence and durable messaging capabilities to NATS.

Conceptually:

```text
                  NATS
                   |
             +-----+------+
             |            |
             v            v
        Core NATS      JetStream
             |            |
          live          durable
        messaging      messaging
```

JetStream provides capabilities such as:

* Persistent message storage
* Durable consumers
* Acknowledgements
* Redelivery
* Replay
* Retention policies
* Consumer state
* Replication

For example:

```text
Interest Service
       |
       | publish
       v
+----------------+
|   JetStream    |
|   persistent   |
|    storage     |
+----------------+
       |
       | delivery
       v
Distribution Service
```

If Distribution Service crashes, JetStream can retain the message and redeliver it according to the configured stream and consumer behavior.

We will study these semantics in detail later.

---

# 10. NATS is not simply "Kafka but faster"

This is one of the most important misconceptions to avoid.

NATS and Kafka overlap in some use cases, but their original design philosophies are different.

A useful high-level distinction is:

```text
Kafka
    |
    +--> Durable distributed event log
    +--> Partitions
    +--> Long-lived event retention
    +--> Replay
    +--> High-volume event streaming
    +--> Stream-processing ecosystem


NATS
    |
    +--> Lightweight messaging
    +--> Service-to-service communication
    +--> Pub/Sub
    +--> Request/Reply
    +--> Queue Groups
    +--> Low latency
    +--> JetStream for durable messaging
```

This is a mental model, not a claim that Kafka cannot do service communication or that NATS cannot do durable streaming.

Both systems can overlap significantly.

The important question is:

> **What is the primary problem your system is trying to solve?**

---

# 11. NATS vs Kafka: the basic conceptual difference

## Kafka

Kafka's fundamental abstraction is a **durable distributed log**.

A simplified mental model:

```text
Producer
    |
    v
+----------------------+
| Kafka Topic          |
|                      |
|  msg1                |
|  msg2                |
|  msg3                |
|  msg4                |
|  msg5                |
+----------------------+
        |
        +------> Consumer A
        |
        +------> Consumer B
```

Messages are persisted according to the topic's retention configuration.

Consumers maintain their position in the log.

This makes Kafka particularly strong when you need:

* Large-scale event streaming
* Durable event history
* Replay
* Partition-based parallelism
* Independent consumers
* Stream processing
* Long retention

---

## NATS Core

Core NATS is fundamentally a messaging fabric.

```text
Publisher
    |
    v
  NATS
    |
    v
Subscriber
```

The emphasis is on efficiently moving messages between applications.

This makes Core NATS particularly attractive for:

* Service-to-service messaging
* Request/Reply
* Real-time communication
* Low-latency messaging
* Dynamic distributed systems
* Lightweight event distribution

---

## NATS JetStream

JetStream brings durable messaging capabilities to NATS.

```text
Publisher
    |
    v
JetStream
    |
    | persisted
    v
Consumer
```

Therefore, the comparison is better thought of as:

```text
             NATS
              |
       +------+------+
       |             |
   Core NATS      JetStream
       |             |
   Messaging      Durable
                  messaging
```

rather than simply:

```text
NATS = Kafka alternative
```

---

# 12. When would NATS be attractive?

Consider a microservice architecture:

```text
API
 |
 v
Order Service
 |
 v
Payment Service
 |
 v
Inventory Service
```

If the primary requirement is lightweight service communication, NATS can be a natural choice.

For example:

```text
Order Service
      |
      | request
      v
     NATS
      |
      v
Payment Service
```

NATS supports Request/Reply as a first-class communication pattern.

This is different from the way Kafka is normally used.

---

# 13. When would Kafka be attractive?

Consider a financial transaction platform producing a large event stream:

```text
Transaction Service
        |
        v
    Kafka Topic
        |
   +----+----+----+
   |    |    |    |
   v    v    v    v
Fraud Risk Audit Analytics
```

Suppose we need:

* Months of event history
* Multiple independent consumers
* Replay
* Large-scale partitioning
* Stream processing
* Historical analysis

Kafka becomes extremely compelling.

The event log itself becomes an important part of the architecture.

---

# 14. When could both be used?

NATS and Kafka do not necessarily have to compete.

A system could use both.

For example:

```text
                  +-------------+
                  | Application |
                  +-------------+
                    /         \
                   /           \
                  v             v
               NATS           Kafka
                |                |
                v                v
        Service Communication   Event Log
        Request/Reply           Analytics
        Real-time messaging     Replay
```

For example:

### NATS

Used for:

```text
Service A
   |
   | request/reply
   v
Service B
```

### Kafka

Used for:

```text
Business Event
      |
      v
Kafka
  |
  +----> Analytics
  +----> Audit
  +----> Fraud
  +----> Data Lake
```

Whether using both is justified depends on operational complexity and actual requirements. Introducing two messaging systems unnecessarily can itself become a problem.

---

# 15. Performance

NATS is designed for:

* Low latency
* High message rates
* Small resource footprint
* Simple operational deployment

The NATS documentation describes NATS as capable of processing millions of messages per second with sub-millisecond latency under appropriate conditions.

However, performance numbers should never be interpreted independently of:

* Message size
* Hardware
* Network
* Persistence
* Number of consumers
* Replication
* Configuration
* Workload pattern

Therefore, don't memorize:

> "NATS = X million messages/sec."

Instead remember:

> **NATS is designed with low latency and lightweight messaging as major design goals.**

---

# 16. Simplicity

NATS emphasizes a small operational footprint.

The NATS server is distributed as a single `nats-server` binary.

The same server process can provide functionality including:

* Core messaging
* Clustering
* JetStream
* Leaf nodes
* MQTT
* WebSockets

This is part of NATS's philosophy of providing a relatively simple messaging infrastructure.

However:

> Simpler infrastructure does not mean simpler distributed-system semantics.

Once we introduce JetStream, replication, consumers, acknowledgements, redelivery, retention, and failure handling, there are still many distributed-system concepts to understand.

---

# 17. Multi-tenancy

NATS supports **accounts** that can isolate messaging domains.

Conceptually:

```text
                    NATS Cluster
                         |
            +------------+------------+
            |                         |
        Account A                 Account B
            |                         |
      +-----+-----+             +-----+-----+
      |           |             |           |
 interest.*   tax.*          orders.*    payment.*
```

This can be useful for:

* Multi-tenant platforms
* Team isolation
* Department isolation
* SaaS systems
* Security boundaries

The details of NATS accounts, permissions, and authentication should be studied separately rather than memorized from this introductory section.

---

# 18. Common NATS use cases

NATS can be used in several categories.

## Microservices communication

```text
Service A
    |
    v
  NATS
    |
    v
Service B
```

Useful patterns include:

* Request/Reply
* Pub/Sub
* Queue Groups

---

## Event-driven architecture

```text
Order Service
      |
      v
    NATS
      |
  +---+---+
  |   |   |
  v   v   v
Audit Tax Notification
```

Multiple services can react to events.

---

## Real-time data

Examples include:

* IoT telemetry
* Market data
* Logs
* Metrics
* Real-time application events

---

## Edge computing

NATS can be used where systems have:

* Limited resources
* Intermittent connectivity
* Edge devices
* Edge-to-cloud communication

NATS's lightweight design can be useful in such environments.

---

# 19. Important terminology to remember

| Term        | Meaning                                             |
| ----------- | --------------------------------------------------- |
| NATS Server | Messaging infrastructure that routes messages       |
| Publisher   | Application sending a message                       |
| Subscriber  | Application receiving a message                     |
| Subject     | Logical name used for routing messages              |
| Core NATS   | Lightweight messaging layer                         |
| JetStream   | Persistence and durable messaging capabilities      |
| Queue Group | Load-balanced consumption among service instances   |
| Stream      | JetStream's persistent message storage abstraction  |
| Consumer    | JetStream component that tracks message consumption |

The last two concepts—**Stream** and **Consumer**—will become much more important when we study JetStream.

---

# 20. Important misconceptions

### Misconception 1: NATS is just another Kafka

Incorrect.

NATS has a different primary design philosophy and includes messaging patterns such as Request/Reply as first-class concepts.

---

### Misconception 2: NATS always persists messages

Incorrect.

Core NATS is primarily live messaging.

Persistence is provided through JetStream.

---

### Misconception 3: JetStream and Kafka are identical

Incorrect.

JetStream provides durable messaging and streaming capabilities, but Kafka and JetStream have different architectures, scaling models, operational characteristics, and semantics.

These differences should be studied rather than reduced to a one-to-one mapping.

---

### Misconception 4: NATS eliminates service discovery

Not exactly.

NATS provides messaging-level location transparency and routing capabilities, including mechanisms that support service discovery and load balancing patterns.

It doesn't mean your entire infrastructure no longer needs concepts such as DNS, Kubernetes service discovery, or other infrastructure-level discovery mechanisms.

---

# 21. System-design mental model

When designing a system, start with the communication requirement.

### Requirement A: Direct synchronous communication

```text
Service A
   |
   | HTTP/gRPC
   v
Service B
```

HTTP/gRPC may be sufficient.

---

### Requirement B: Lightweight asynchronous communication

```text
Service A
   |
   v
Core NATS
   |
   v
Service B
```

Core NATS can be a strong candidate.

---

### Requirement C: Asynchronous communication with durability

```text
Service A
   |
   v
JetStream
   |
   v
Service B
```

JetStream becomes relevant.

---

### Requirement D: Large durable event backbone

```text
Producers
    |
    v
  Kafka
    |
    +----> Consumer A
    +----> Consumer B
    +----> Consumer C
    +----> Analytics
    +----> Data Lake
```

Kafka may be the more natural choice when the event log itself is central to the architecture.

The correct technology depends on the complete set of requirements rather than one performance number or feature.

---

# 22. Interview answer: What is NATS?

A concise answer:

> NATS is an open-source, lightweight messaging system designed for low-latency communication between distributed applications. Applications communicate through subjects rather than directly addressing each other's network locations. Core NATS provides patterns such as pub/sub, request/reply, and queue groups, while JetStream adds persistence, acknowledgements, durable consumers, replay, and redelivery.

---

# 23. Interview answer: NATS vs Kafka

A good high-level answer:

> Kafka is primarily designed as a durable distributed event-streaming platform built around partitioned logs, while NATS is primarily a lightweight messaging and communication fabric. NATS is particularly attractive for low-latency service-to-service communication, request/reply, pub/sub, and dynamic distributed systems. Kafka is particularly strong when we need a durable event backbone, long retention, replay, partition-based scaling, and stream-processing workloads. NATS JetStream provides durable messaging capabilities, so there is overlap between the two and the choice should be driven by the workload and required semantics.

---

# 24. The mental model to carry forward

The most important thing from this chapter is:

```text
                         NATS
                          |
              +-----------+-----------+
              |                       |
              v                       v
         Core NATS                JetStream
              |                       |
       Live messaging          Durable messaging
              |                       |
       +------+-------+         +-----+------+
       |      |      |         |            |
     Pub/Sub RPC Queue       Persist      Replay
                            ACK          Redelivery
                                         Retention
```

And the high-level comparison:

```text
Core NATS
    ↓
"I need fast communication between services."


JetStream
    ↓
"I need NATS messaging plus durability."


Kafka
    ↓
"I need a durable, scalable event-streaming backbone."
```

These are starting mental models, not absolute rules. As we learn NATS in depth, these models will become more precise.

---

# 25. What we need to learn next

The natural learning sequence is:

```text
NATS fundamentals
       ↓
Subjects
       ↓
Publish / Subscribe
       ↓
Wildcards
       ↓
Queue Groups
       ↓
Request / Reply
       ↓
Core NATS behavior
       ↓
Why persistence is needed
       ↓
JetStream
       ↓
Streams
       ↓
Consumers
       ↓
Acknowledgements
       ↓
Delivery semantics
       ↓
Retention
       ↓
Redelivery
       ↓
Replication
       ↓
Failure handling
       ↓
Production architecture
       ↓
NATS vs Kafka
```

Do not jump into JetStream yet.

First understand **Core NATS extremely well**, because JetStream is much easier to understand once you have a solid mental model of how normal NATS messaging works.
