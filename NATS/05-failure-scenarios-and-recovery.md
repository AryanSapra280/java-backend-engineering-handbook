# NATS Failure Scenarios and Recovery

> This chapter analyzes what happens when different components fail during message processing.
>
> The goal is not just to understand NATS features, but to develop the distributed-systems mindset required to answer:
>
> - What exactly failed?
> - What was the last confirmed state?
> - Was the message lost?
> - Could the operation have succeeded?
> - Should we retry?
> - Can retry create duplicates?
> - How do we recover safely?
> - Should we use Core NATS or JetStream?

---

# 1. The Basic Message Flow

A simple asynchronous flow looks like:

    Producer
       |
       | publish
       v
     NATS
       |
       | deliver
       v
    Consumer
       |
       | process
       v
    Database

A request/reply flow looks like:

    Requester
        |
        | request
        v
      NATS
        |
        v
     Responder
        |
        | response
        v
      NATS
        |
        v
     Requester

The important thing is that failure can happen at every boundary.

For example:

    Producer
       |
       X
       |
      NATS
       |
       X
       |
    Consumer
       |
       X
       |
    Database

The same high-level architecture can therefore produce many different failure scenarios.

---

# 2. The Most Important Concept: UNKNOWN

Distributed systems are not always:

    SUCCESS
    FAILURE

There is often a third state:

    UNKNOWN

For example:

    Producer
        |
        | publish()
        v
      NATS
        |
        X Producer crashes

The producer may not know whether:

1. NATS never received the message.

or:

2. NATS received the message before the producer crashed.

From the producer's point of view:

    UNKNOWN

This is extremely important because the producer may decide:

    "Let's retry."

But if the first attempt actually succeeded:

    Attempt 1 → SUCCESS
    Attempt 2 → SUCCESS

the consumer could process the same business operation twice.

Therefore:

    Retry
      +
    Idempotency

are often closely related.

---

# 3. Core NATS Failure Model

Core NATS is an ephemeral, at-most-once messaging system.

A simplified flow is:

    Producer
       |
       | publish
       v
    NATS Server
       |
       | route
       v
    Subscriber

Core NATS does not provide durable message storage.

If a subscriber is not present when a message is published, the message is not retained for that future subscriber.

Similarly, Core NATS does not provide the durable acknowledgement and redelivery model associated with JetStream.

This is intentional.

Core NATS is useful for:

- low-latency service communication
- request/reply
- transient events
- notifications
- live data
- data where missing an old message is acceptable
- situations where the next value supersedes the previous value

Examples:

    Current temperature
    Current stock price
    Cache invalidation
    Live metrics
    Service-to-service request/reply

When a message must survive failures and be processed later, JetStream becomes relevant.

---

# 4. Core NATS vs JetStream During Failure

A useful mental model is:

    Core NATS
        |
        v
    Fast + ephemeral communication

while:

    JetStream
        |
        v
    Communication
        +
    Persistence
        +
    Recovery

Therefore, whenever you encounter a failure scenario, ask:

> Is losing this message acceptable?

If yes:

    Core NATS may be enough.

If no:

    JetStream should be considered.

---

# 5. Failure Scenario Matrix

The major scenarios we will study are:

| # | Scenario | Main Problem |
|---|---|---|
| 1 | Producer crashes before publish | Message never exists |
| 2 | Producer crashes during publish | Did NATS receive it? |
| 3 | NATS unavailable before publish | Publish fails |
| 4 | NATS crashes while publishing | Message may not be delivered |
| 5 | NATS crashes after receiving message | Core NATS cannot provide durable recovery |
| 6 | Consumer is offline | Core NATS message is missed |
| 7 | NATS crashes during delivery | Delivery may be lost |
| 8 | Consumer crashes after receiving | Message may be lost with Core NATS |
| 9 | Consumer receives message but DB fails | Business operation fails |
| 10 | DB succeeds but consumer crashes | Retry can create duplicate processing |
| 11 | Consumer partially processes message | Inconsistent state |
| 12 | Requester crashes before request | Request never happens |
| 13 | Requester crashes after request | Responder may still process |
| 14 | Responder crashes before processing | Request may time out |
| 15 | Responder processes but crashes before reply | Operation succeeded but requester sees timeout |
| 16 | Response is lost | Requester cannot know result |
| 17 | Producer retries duplicate message | Duplicate event |
| 18 | Consumer retries duplicate message | Duplicate business operation |

---

# 6. Scenario 1 — Producer Crashes Before Publishing

## Problem

The producer performs some business logic but crashes before calling NATS.

    Producer
       |
       | business logic
       |
       X CRASH
       |
       v
      NATS

Example:

    calculateInterest();

    nats.publish("interest.calculated", event);

The application crashes before:

    nats.publish(...)

## Consequence

The message never reached NATS.

Therefore:

    Producer
       X
       |
       v
      NATS

    No message exists

NATS cannot recover something it never received.

## Recovery

Recovery must happen outside NATS.

Possible approaches:

- retry from persisted state
- transactional outbox
- reconciliation
- scheduled retry

## Important Lesson

NATS can only handle a message after the message actually reaches NATS.

If the producer crashes before publishing:

    NATS has nothing to recover.

---

# 7. Scenario 2 — Producer Crashes During Publishing

This is more interesting.

## Problem

The producer starts publishing and crashes.

    Producer
       |
       | publish()
       X CRASH
       |
       v
      NATS

The producer does not necessarily know whether the message reached NATS.

There are two possibilities.

### Possibility A — Message never reached NATS

    Producer
       |
       X
       |
      NATS

### Possibility B — NATS received the message

    Producer
       |
       | message
       v
      NATS
       |
       X Producer crashes

From the producer's perspective:

    UNKNOWN

## Consequence

The producer may retry.

If the first publish actually succeeded:

    Attempt 1 → SUCCESS
    Attempt 2 → SUCCESS

the consumer may receive duplicates.

## Recovery

Use some combination of:

- retry
- unique event ID
- idempotent consumer
- transactional outbox
- reconciliation

Example:

    {
      "event_id": "evt-12345",
      "account_id": "ACC-100",
      "amount": 50000
    }

The consumer can use:

    event_id

to identify duplicates.

---

# 8. Scenario 3 — NATS Is Unavailable Before Publishing

## Problem

The producer tries to publish while NATS is unavailable.

    Producer
       |
       | publish()
       X
    NATS unavailable

## Consequence

The producer knows that the publish failed.

This is different from Scenario 2.

Here:

    Publish result = FAILURE

rather than:

    Publish result = UNKNOWN

## Recovery

The producer can retry.

A production retry strategy may include:

- maximum retry attempts
- timeout
- exponential backoff
- jitter
- logging
- metrics
- alerting

Example:

    Attempt 1 → immediate
    Attempt 2 → 100 ms
    Attempt 3 → 500 ms
    Attempt 4 → 1 sec
    Attempt 5 → 5 sec

The exact values depend on the application.

---

# 9. Scenario 4 — NATS Crashes While Producer Is Publishing

## Problem

The producer is trying to publish:

    Producer
       |
       | publish
       v
      NATS
       |
       X CRASH

## Consequence

The producer may not know exactly what happened.

Possible states:

    A. Message never reached NATS
    B. Message reached NATS but was not delivered
    C. Message was delivered

Therefore the producer may again end up with:

    UNKNOWN

## Recovery

A producer may retry.

But retrying introduces the possibility of duplicate messages.

Therefore:

    Retry
      +
    Idempotency

is important.

If the message is business-critical and cannot be lost, a durable messaging mechanism such as JetStream should be considered.

---

# 10. Scenario 5 — NATS Crashes After Receiving the Message

This is one of the most important Core NATS scenarios.

## Problem

NATS receives the message:

    Producer
       |
       | message
       v
      NATS
       |
       | RECEIVED
       X
    NATS crashes

## Consequence

Core NATS is ephemeral.

The message is not treated as a durable persisted message.

Therefore, if the message was only in Core NATS and the server failed before successful delivery:

    Message may be lost.

After restart:

    NATS restarts
       |
       v
    Where is the old Core NATS message?
       |
       X
    No durable message to replay

## Recovery

Core NATS itself cannot provide durable recovery for this scenario.

If the requirement is:

> The message must survive a NATS/server failure.

consider JetStream.

    Producer
       |
       v
    JetStream
       |
       | persisted
       v
    Consumer

## Key Lesson

Core NATS:

    Communication

JetStream:

    Communication
        +
    Persistence
        +
    Recovery mechanisms

---

# 11. Scenario 6 — Consumer Is Offline

## Problem

The producer publishes while the consumer is offline.

    Producer
       |
       v
      NATS
       |
       X
    Consumer offline

## Consequence

Core NATS does not retain the message for that future subscriber.

Example:

    10:00 → Consumer offline

    10:01 → Producer publishes event

    10:02 → Consumer starts

The consumer receives future messages, but the message published at 10:01 is not waiting for it.

## Recovery

Ask:

> Is missing this message acceptable?

### If YES

Core NATS is suitable.

Examples:

- live temperature
- live metrics
- current state
- cache invalidation
- transient notifications

### If NO

Use JetStream.

    Producer
       |
       v
    JetStream
       |
       | retain
       v
    Consumer

The consumer can reconnect later and process retained messages.

---

# 12. Scenario 7 — NATS Crashes During Delivery

## Problem

NATS is delivering a message:

    Producer
       |
       v
      NATS
       |
       | message
       X CRASH
       |
       v
    Consumer

The exact delivery state may be uncertain.

The message could have:

1. Not been delivered
2. Been partially transmitted
3. Been received by the consumer

## Consequence

Core NATS does not provide durable automatic redelivery.

Therefore the message can effectively be lost.

## Recovery

If losing the message is acceptable:

    No special recovery is required.

If the message must eventually be processed:

    Use JetStream.

JetStream provides the persistence and consumer acknowledgement model needed for this type of recovery.

---

# 13. Scenario 8 — Consumer Crashes Immediately After Receiving

## Problem

The consumer receives the message:

    NATS
       |
       | message
       v
    Consumer
       |
       X CRASH

The consumer has received the message but has not completed processing.

## Consequence with Core NATS

There is no durable acknowledgement/redelivery mechanism.

The message can be lost.

## Recovery

If the message can be lost:

    Core NATS is fine.

If the message must be processed:

    Use JetStream.

Conceptually:

    JetStream
       |
       | message
       v
    Consumer
       |
       X CRASH

If the message was not successfully acknowledged according to the JetStream consumer configuration, it can become eligible for redelivery.

---

# 14. Scenario 9 — Consumer Receives Message but Database Fails

## Problem

    NATS
       |
       v
    Consumer
       |
       v
    Database
       |
       X FAILURE

The message reached the consumer successfully.

But the business operation failed because the database operation failed.

## Consequence

Core NATS does not know about your database failure.

From NATS's perspective:

    Message delivered.

It does not automatically understand:

    Database = FAILURE

## Recovery with Core NATS

The application can implement its own retry:

    Message
       |
       v
    Consumer
       |
       X DB failure
       |
       v
    Application retry

But now the application must manage:

- retry count
- retry delay
- duplicate processing
- failure storage
- dead-letter behavior if required

## Recovery with JetStream

With JetStream:

    JetStream
       |
       | message
       v
    Consumer
       |
       X DB failure
       |
       | no successful ACK
       v
    JetStream
       |
       | redelivery
       v
    Consumer

This is one reason JetStream becomes useful for reliable event processing.

---

# 15. Scenario 10 — Database Succeeds but Consumer Crashes

This is one of the classic distributed-system problems.

## Problem

    NATS
       |
       v
    Consumer
       |
       v
    Database
       |
       | COMMIT ✓
       |
       X
    Consumer crashes

The database operation succeeded.

But the consumer crashes before completing its message-processing lifecycle.

## Consequence

If the message is retried:

    First attempt
        |
        v
    DB update ✓
        |
        X Consumer crashes

    Second attempt
        |
        v
    DB update again

The business operation may happen twice.

## Recovery

Use idempotency.

For example:

    event_id = evt-12345

Maintain a record of processed event IDs:

    processed_events
    ----------------
    evt-12345

When the message arrives again:

    evt-12345
         |
         v
    Already processed?
         |
         +---- YES → Skip duplicate
         |
         +---- NO  → Process

## Important Lesson

Durable messaging does NOT automatically mean:

    No duplicates

A reliable system often requires:

    Durable messaging
      +
    Idempotent processing

---

# 16. Scenario 11 — Consumer Partially Processes a Message

Suppose one message triggers multiple operations:

    Consumer
       |
       +---- Update DB A ✓
       |
       +---- Update DB B ✓
       |
       X---- Update DB C
       |
      CRASH

## Problem

Only part of the business operation completed.

## Consequence

The system can be inconsistent:

    A → updated
    B → updated
    C → not updated

## Recovery

The solution depends on the architecture.

### Same database

Use a database transaction:

    BEGIN TRANSACTION

    Operation A
    Operation B
    Operation C

    COMMIT

If the transaction fails:

    ROLLBACK

### Multiple databases/services

A single database transaction may not be possible.

Possible patterns include:

- Saga
- compensation
- idempotency
- reconciliation
- workflow state

---

# 17. Scenario 12 — Requester Crashes Before Sending Request

For request/reply:

    Requester
        |
        X CRASH
        |
       NATS

## Problem

The request was never sent.

## Consequence

The responder never receives it.

## Recovery

If the request is still required:

    Requester restarts
          |
          v
      Retry request

No special NATS recovery is necessary because the request never entered the system.

---

# 18. Scenario 13 — Requester Sends Request and Then Crashes

    Requester
       |
       | request
       v
      NATS
       |
       v
    Responder
       |
       X Requester crashes

## Problem

The responder may continue processing even though the requester has disappeared.

## Consequence

The business operation may succeed.

But the requester may never receive the response.

From the requester's perspective:

    UNKNOWN

The operation could be:

    SUCCESS

or:

    FAILURE

## Recovery

Use:

- request ID
- idempotency
- persistent business state
- retry
- reconciliation

Example:

    {
      "request_id": "REQ-12345",
      "account_id": "ACC-100"
    }

The responder should be able to recognize the same request if it is retried.

---

# 19. Scenario 14 — Responder Crashes Before Processing

    Requester
       |
       | request
       v
      NATS
       |
       v
    Responder
       |
       X CRASH

## Consequence with Core NATS

The requester may eventually receive:

    timeout

Core NATS does not automatically guarantee durable replay of the request.

## Recovery

Depending on the business requirement:

    Requester retry

or:

    JetStream
      +
    durable workflow
      +
    idempotency

---

# 20. Scenario 15 — Responder Processes Request but Crashes Before Reply

This is extremely important for request/reply systems.

    Requester
       |
       | request
       v
    Responder
       |
       | DB operation
       v
    Database
       |
       | COMMIT ✓
       |
       X Responder crashes

The database operation succeeded.

But the responder never sent the reply.

## Consequence

The requester sees:

    TIMEOUT

It is tempting to conclude:

    "Operation failed."

But that conclusion is incorrect.

The actual possibilities are:

    A. Operation failed
    B. Operation succeeded but response was lost
    C. Responder crashed after commit but before response

Therefore:

    TIMEOUT ≠ FAILURE

The state is:

    UNKNOWN

## Recovery

The requester may retry.

But retrying may duplicate the business operation.

Therefore use:

    request_id
      +
    idempotency

---

# 21. Scenario 16 — Response Is Lost

Consider:

    Requester
       |
       | request
       v
    Responder
       |
       | DB operation ✓
       |
       | response
       X
    Response lost

## Problem

The responder completed the operation.

But the requester did not receive the response.

## Consequence

The requester sees:

    TIMEOUT

But the operation may have succeeded.

Again:

    UNKNOWN

## Recovery

Possible approaches:

### Retry using the same request ID

    REQ-12345

The responder checks whether the request has already been processed.

### Query the business state

For example:

    GET operation status

### Reconciliation

A background process can compare expected and actual business state.

---

# 22. Scenario 17 — Producer Retries and Creates a Duplicate Message

Suppose:

    Producer
       |
       | publish
       v
      NATS
       |
       v
    Consumer

The producer does not receive confirmation that the operation completed and retries.

    Attempt 1 → message
    Attempt 2 → same message

The consumer could receive:

    event_id = evt-12345
    event_id = evt-12345

## Consequence

Without protection:

    Process event 1
    Process event 2

The business operation could happen twice.

## Recovery

Use an event ID:

    {
      "event_id": "evt-12345"
    }

Then:

    Consumer
       |
       v
    Check event_id
       |
       +---- Already processed → ignore
       |
       +---- New event → process

---

# 23. Scenario 18 — Consumer Retries and Creates Duplicate Processing

Suppose:

    Consumer
       |
       | process
       v
    Database
       |
       | SUCCESS
       |
       X
    Consumer does not know

The consumer retries.

    Retry
       |
       v
    Database

Now the same business operation can execute twice.

## Recovery

Again:

    Idempotency

For example:

    event_id = evt-12345

Database constraint:

    UNIQUE(event_id)

Then duplicate processing can be prevented at the database level.

---

# 24. Request/Reply Is Not Automatically Transactional

A very important misconception is:

> "If NATS request/reply returns an error, the operation definitely failed."

Not necessarily.

Consider:

    Requester
       |
       | request
       v
    Responder
       |
       | DB commit ✓
       |
       X
    Responder crashes

Requester:

    timeout

But:

    Database = SUCCESS

Therefore:

    Network result
          ≠
    Business result

A timeout tells us:

> "I did not receive the response."

It does NOT necessarily tell us:

> "The business operation failed."

This distinction is critical when using NATS as an internal replacement for HTTP request/reply.

---

# 25. Error Propagation in NATS Request/Reply

Suppose:

    Service A
        |
        | request
        v
      NATS
        |
        v
    Service B
        |
        v
    Database

Database fails:

    Service B
        |
        v
    Database
        |
        X FAILURE

Service B can explicitly send an error response:

    Service B
        |
        | error response
        v
      NATS
        |
        v
    Service A

Conceptually:

    {
      "success": false,
      "error_code": "DB_UNAVAILABLE",
      "message": "Database operation failed"
    }

Service A receives the response and can handle the error.

Therefore NATS itself does not magically understand database failures.

Instead:

    Database failure
          |
          v
    Service B detects failure
          |
          v
    Service B creates response
          |
          v
    NATS transports response
          |
          v
    Service A

This is very similar conceptually to HTTP:

    Service B
       |
       | HTTP 500
       v
    Service A

or:

    Service B
       |
       | NATS response containing error
       v
    Service A

---

# 26. Core NATS vs HTTP Internal Communication

For internal service-to-service request/reply:

### HTTP

    Service A
       |
       | HTTP request
       v
    Service B
       |
       | HTTP response
       v
    Service A

### Core NATS

    Service A
       |
       | NATS request
       v
      NATS
       |
       v
    Service B
       |
       | NATS response
       v
      NATS
       |
       v
    Service A

The important difference is that NATS introduces a messaging layer.

Service A does not need to know Service B's network location directly.

It communicates using a subject:

    interest.calculate

instead of:

    POST http://interest-service/api/calculate

---

# 27. Why Core NATS Can Be Attractive for Internal Communication

Suppose:

    Interest Service
           |
           | calculate interest
           v
    Distribution Service

With HTTP:

    Interest Service
           |
           | HTTP
           v
    Distribution Service

With NATS:

    Interest Service
           |
           | request
           v
          NATS
           |
           v
    Distribution Service

Benefits can include:

- location transparency
- service discovery
- load balancing with queue groups
- low latency
- persistent connections
- request/reply
- easy communication between multiple service instances

---

# 28. But Core NATS Is Not a Universal HTTP Replacement

Do not conclude:

    NATS > HTTP

The technologies solve somewhat different problems.

HTTP is often better when:

- communicating with external clients
- exposing public APIs
- browser communication
- third-party integrations
- REST APIs
- standard HTTP tooling is important

Core NATS can be attractive when:

- communication is internal
- services are distributed
- low latency matters
- service discovery is useful
- request/reply is needed
- pub/sub is useful
- the system is already using NATS

---

# 29. Core NATS vs JetStream — Failure Perspective

A simple comparison:

| Requirement | Core NATS | JetStream |
|---|---|---|
| Low latency | Excellent | Excellent |
| Pub/Sub | Yes | Yes |
| Request/Reply | Yes | Possible with additional design |
| Ephemeral messages | Excellent | Possible |
| Message persistence | No | Yes |
| Replay | No | Yes |
| Durable consumers | No | Yes |
| Acknowledgements | No durable ACK model | Yes |
| Redelivery | No durable redelivery | Yes |
| Survive consumer downtime | No | Yes |
| Survive server failure with persistence | No | Yes |
| At-most-once style communication | Yes | Configurable/use-case dependent |
| Reliable event processing | Limited | Stronger fit |

---

# 30. The Most Important Recovery Patterns

When designing reliable NATS systems, the major tools are:

                     Reliability
                          |
            +-------------+-------------+
            |             |             |
          Retry       Idempotency   Persistence
            |             |             |
         Backoff       Event ID       JetStream
            |             |             |
            +-------------+-------------+
                          |
                   Reconciliation

Each mechanism solves a different problem.

---

# 31. Retry

Use retry for transient failures.

    Operation
       |
       X temporary failure
       |
       v
     Retry

But avoid:

    Retry forever

Use:

    Maximum attempts
      +
    Backoff
      +
    Jitter

---

# 32. Idempotency

Idempotency protects against duplicate execution.

Example:

    event_id = EVT-123

Consumer:

    Receive EVT-123
           |
           v
    Already processed?
           |
       +---+---+
       |       |
      YES      NO
       |       |
     Ignore   Process

Possible implementation:

    processed_events
    ----------------
    event_id

with:

    UNIQUE(event_id)

---

# 33. Persistence

If a message cannot be lost:

    Core NATS

may not be enough.

Consider:

    JetStream

Example:

    Producer
       |
       v
    JetStream
       |
       | persist
       v
    Consumer
       |
       X crash
       |
       v
    Consumer restarts
       |
       v
    Message can be redelivered

---

# 34. Reconciliation

Sometimes retries are not enough.

Suppose:

    Requester
       |
       | request
       v
    Responder
       |
       | DB commit ✓
       |
       X crash

Requester does not know the result.

Instead of blindly executing again, it may query:

    What is the status of request REQ-12345?

This is reconciliation.

Conceptually:

    Unknown state
         |
         v
    Query source of truth
         |
         +---- Completed
         |
         +---- Not completed
         |
         +---- Still unknown

---

# 35. Transactional Outbox

Consider:

    Database update
          +
    NATS publish

Without an outbox:

    DB update ✓
        |
        X application crash
        |
    NATS publish never happens

The database changed but the event was not published.

With an outbox:

                    Database
              +-----------------+
              | Business data   |
              |                 |
              | Outbox event    |
              +--------+--------+
                       |
                       | later publish
                       v
                      NATS

The business operation and event record are stored together.

A publisher can later read the outbox and publish the event.

This is especially useful when a database update and message publication must be coordinated reliably.

---

# 36. Failure Analysis Checklist

Whenever you design a NATS flow, ask these questions.

## Producer

    What if producer crashes before publish?
    What if producer crashes during publish?
    What if NATS is unavailable?
    What if publish result is unknown?

## NATS

    What if NATS crashes before receiving?
    What if NATS crashes after receiving?
    What if NATS crashes during delivery?
    What if there are no subscribers?

## Consumer

    What if consumer is offline?
    What if consumer crashes after receiving?
    What if consumer crashes during processing?
    What if consumer processes partially?

## Database

    What if DB fails?
    What if DB succeeds but consumer crashes?
    What if DB transaction partially completes?

## Request/Reply

    What if requester crashes?
    What if responder crashes?
    What if responder processes but doesn't reply?
    What if response is lost?
    What if requester times out?

## Recovery

    Should we retry?
    Can retry create duplicates?
    Do we need idempotency?
    Do we need persistence?
    Do we need reconciliation?
    Should we use Core NATS or JetStream?

---

# 37. The Key Mental Model

When a distributed operation fails, don't immediately ask:

> "Did it fail?"

Instead ask:

> "What is the last state I can prove?"

For example:

    Producer
       |
       | request
       v
      NATS
       |
       v
    Consumer
       |
       | DB commit
       v
    Database

If the consumer crashes after the DB commit:

    Database = DEFINITELY SUCCESS
    Consumer = FAILED
    Response = UNKNOWN

If the producer crashes while publishing:

    Producer = FAILED
    NATS = UNKNOWN
    Message = UNKNOWN

If NATS was down before publishing:

    Producer = ACTIVE
    NATS = UNAVAILABLE
    Message = NOT PUBLISHED

This way of thinking is much more powerful than simply saying:

    "Service crashed."

---

# 38. The Big Picture

A production-grade distributed system usually combines several mechanisms.

For example:

                      Producer
                         |
                         |
                  Transactional
                     Outbox
                         |
                         v
                     JetStream
                         |
                         v
                      Consumer
                         |
                +--------+--------+
                |                 |
                v                 v
           Idempotency        Database
                |                 |
                +--------+--------+
                         |
                         v
                    Reconciliation

Each mechanism solves a different problem:

### Transactional Outbox

Prevents a database change from becoming disconnected from event publication.

### JetStream

Provides durable messaging and consumer recovery.

### Retry

Handles transient failures.

### Idempotency

Protects against duplicate processing.

### Database Transactions

Keep related DB operations atomic within a database.

### Reconciliation

Resolves UNKNOWN states.

---

# 39. Final Takeaways

The most important lessons from these failure scenarios are:

## 1. Core NATS is ephemeral

    No durable message storage
    No durable replay
    No durable redelivery

Therefore:

    Use Core NATS when losing the message can be acceptable.

---

## 2. JetStream is for durable messaging

When the requirement becomes:

    Message must survive
    Consumer can be offline
    Message must be retried
    Message must be replayable

JetStream becomes the relevant NATS technology.

---

## 3. Timeout does not mean failure

This is one of the most important concepts:

    Timeout
       ≠
    Business failure

The operation may have succeeded but the response may have been lost.

---

## 4. Retry can create duplicates

    Retry
       |
       v
    Duplicate message
       |
       v
    Duplicate processing

Therefore:

    Retry + Idempotency

is a common combination.

---

## 5. NATS does not understand your business transaction

NATS knows about:

- messages
- subjects
- connections
- consumers
- responses

It does not automatically know:

- database succeeded
- interest calculation succeeded
- loan was created
- payment was completed

Your application must translate business outcomes into responses/events.

---

## 6. The hardest state is UNKNOWN

The most dangerous assumption is:

    Timeout = Failure

Instead:

    Timeout
       |
       v
    UNKNOWN
       |
       v
    Can we safely retry?
       |
       +---- YES → Retry with idempotency
       |
       +---- NO → Query / reconcile

---

# 40. One-Line Mental Model

Remember this:

    Core NATS = Fast ephemeral communication

    JetStream = Durable messaging and recovery

    Retry = Handle transient failures

    Idempotency = Make retries safe

    Transaction = Keep related DB operations atomic

    Reconciliation = Resolve UNKNOWN states

And the most important distributed-systems rule:

> If you don't know whether an operation happened, don't blindly assume that it didn't.

That is where most of the interesting failure-handling problems begin.