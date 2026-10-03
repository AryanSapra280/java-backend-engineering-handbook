Yes. **Now I understand exactly what you need.** And honestly, the two question sets together give us a very good picture of the level you need to reach.

This is **not going to be “Java interview preparation 101.”** We are preparing you to sit in front of someone who says:

> “You have worked with Spring Boot and microservices. Fine. Design one. Explain what happens internally. Show me the code. Now tell me what happens under load. Now tell me how it fails.”

That is the level I want you at.

And we have a very specific constraint:

- **Friday 5:15 PM — Sitcom Round 1**
- **Tuesday — TCS Round 1**
- **Wednesday 11 AM — EPAM Senior Software Engineer Round 2**
- Tonight is already ~9:30 PM.
- You sleep around midnight.
- Tomorrow you have roughly **7:30 AM → 2:30 PM** for preparation.

So we're going to be **extremely strategic**.

---

# 1. First: What I think the interviews are actually testing

From both sets of questions, I see five layers.

### Layer 1 — Can you code?

They may literally make you write:

```java
reverseArray(...)
```

or:

```java
findSecondLastNonRepeatingCharacter(...)
```

or:

```java
findPattern(...)
```

And then:

> Why this approach?  
> Complexity?  
> Edge cases?  
> Can you optimize it?  
> Can you implement it now?

So coding cannot be neglected.

---

### Layer 2 — Do you actually understand Java?

Not:

> What is HashMap?

But:

> How does HashMap work internally?

Then:

> What happens during collision?

Then:

> What happens during resize?

Then:

> Why is it not thread-safe?

Then:

> How does ConcurrentHashMap solve it?

Then:

> What happens if two threads update the same key?

That's **senior interview questioning**.

---

### Layer 3 — Can you build production Spring applications?

Not:

> What is dependency injection?

Instead:

> Suppose I give you a Spring Boot service. Walk me through how you would build it from scratch.

You should be able to say:

```text
Requirement
 ↓
API contract
 ↓
Controller
 ↓
Validation
 ↓
Authentication
 ↓
Authorization
 ↓
Service
 ↓
Transaction boundary
 ↓
Repository
 ↓
Database
 ↓
Event publication
 ↓
Kafka
 ↓
Consumer
 ↓
Retry/DLQ
 ↓
Observability
 ↓
Deployment
 ↓
Kubernetes/Ingress
 ↓
Autoscaling
```

And explain **why each exists**.

---

### Layer 4 — Can you reason about distributed systems?

This is where:

- Kafka
- idempotency
- retries
- circuit breakers
- timeouts
- concurrency
- consistency
- transactions
- locks
- caching
- load balancing
- service discovery
- ingress
- autoscaling

come together.

---

### Layer 5 — Can you design?

They can give you:

> “Design a high-TPS wallet.”

or:

> “Design a ledger.”

or:

> “Design a payment system.”

or:

> “Design a rate limiter.”

And then start drilling:

> Why this database?

> What is your schema?

> How do you prevent duplicate payment?

> What happens if Kafka is down?

> What happens if DB commit succeeds but Kafka publish fails?

> What happens if two requests modify the same account simultaneously?

> How do you scale?

**This is exactly why we're going to connect the topics instead of studying them independently.**

---

# 2. Our preparation strategy

I'm going to divide everything into **8 major blocks**.

### BLOCK A — Core Java

We need:

- OOP
- encapsulation
- inheritance
- polymorphism
- abstraction
- composition
- interfaces
- abstract classes
- `equals()` / `hashCode()`
- String
- String pool
- immutable objects
- final
- static
- access modifiers
- Object class
- exception handling
- checked vs unchecked
- generics
- collections
- ArrayList
- LinkedList
- HashMap
- HashSet
- TreeMap
- ConcurrentHashMap
- Queue
- Deque
- PriorityQueue
- Comparable vs Comparator
- fail-fast / fail-safe
- Java memory model
- heap / stack
- references
- GC basics

And then Java 8+:

- lambda
- functional interfaces
- method references
- Optional
- Stream API
- collectors
- parallel streams

---

# 3. Streams get SPECIAL treatment

Because EPAM explicitly tests this.

You need to be able to solve things like:

```java
Arrays.stream(numbers)
      .filter(n -> n % 2 == 0)
      .sum();
```

but that's only the beginning.

We need:

### Stream fundamentals

```text
Source
 ↓
Intermediate operations
 ↓
Terminal operation
```

Example:

```java
numbers.stream()
       .filter(...)
       .map(...)
       .sorted(...)
       .collect(...);
```

Understand:

- lazy evaluation
- pipeline
- intermediate operations
- terminal operations
- short-circuit operations
- stateful vs stateless operations
- sequential vs parallel streams
- `map`
- `flatMap`
- `filter`
- `peek`
- `sorted`
- `distinct`
- `limit`
- `skip`
- `findFirst`
- `findAny`
- `anyMatch`
- `allMatch`
- `noneMatch`
- `reduce`
- `collect`
- `groupingBy`
- `partitioningBy`
- `toMap`
- merge function
- `Function.identity()`
- primitive streams

And especially questions like:

> Why `findAny()` instead of `findFirst()`?

> Why can `findAny()` return different results in parallel?

> Why is `Stream` lazy?

> What happens if I call two terminal operations?

> Why can't a stream be reused?

---

# 4. Spring / Spring Boot

This is one of our **highest priorities**.

You need to understand:

```text
Spring Core
 ↓
IoC
 ↓
DI
 ↓
Bean creation
 ↓
Bean lifecycle
 ↓
BeanPostProcessor
 ↓
Proxy
 ↓
AOP
 ↓
@Transactional
```

Then Spring Boot:

- auto-configuration
- starters
- component scanning
- configuration properties
- profiles
- external configuration
- Actuator
- validation
- exception handling
- REST
- Spring Data JPA
- transactions
- security

And we will go deep into:

### Bean lifecycle

```text
Instantiate
 ↓
Populate dependencies
 ↓
Aware interfaces
 ↓
BeanPostProcessor before initialization
 ↓
@PostConstruct
 ↓
InitializingBean / init-method
 ↓
BeanPostProcessor after initialization
 ↓
Bean ready
 ↓
@PreDestroy
 ↓
destroy
```

Then:

> Why does Spring need BeanPostProcessor?

> How does Spring create proxies?

> Why does `@Transactional` work?

> Why doesn't `@Transactional` work with self-invocation?

That last one is a **very realistic senior follow-up**.

---

# 5. Spring Security

For Sitcom, I'm putting this **very high**.

You need to be able to explain this entire flow:

```text
Client
 ↓
Request
 ↓
Ingress / Load Balancer
 ↓
Spring Security Filter Chain
 ↓
Authentication
 ↓
JWT validation
 ↓
SecurityContext
 ↓
Authorization
 ↓
Controller
```

And understand:

### Authentication

> Who are you?

### Authorization

> What are you allowed to do?

### JWT

```text
Header
Payload
Signature
```

And importantly:

> Does JWT encryption happen?

Usually no. A normal signed JWT is **encoded and signed, not encrypted**.

Then:

```text
Client
 ↓
Authorization: Bearer <JWT>
 ↓
Security filter
 ↓
Token validation
 ↓
Signature
 ↓
Expiry
 ↓
Claims
 ↓
Authorities/Roles
 ↓
Authorization
```

Then:

- OAuth2
- OpenID Connect
- access token
- refresh token
- authorization server
- resource server
- Keycloak
- RBAC
- roles vs authorities
- scopes
- CORS
- CSRF
- stateless authentication
- token expiration
- token revocation considerations

---

# 6. Microservices

This becomes the **central topic connecting everything**.

If they ask:

> "How would you develop a microservice?"

You should be able to answer something like:

### Step 1 — Define responsibility

Example:

```text
Payment Service
```

owns payment lifecycle.

It shouldn't own unrelated customer or ledger data.

---

### Step 2 — Define API

For example:

```http
POST /payments
GET /payments/{paymentId}
```

Define:

- request
- response
- status codes
- validation
- idempotency

---

### Step 3 — Security

JWT/OAuth2:

```text
Client → Gateway/Ingress → Service → Security filter → Controller
```

---

### Step 4 — Business layer

```text
Controller
   ↓
Service
   ↓
Domain logic
   ↓
Repository
```

---

### Step 5 — Database

Choose:

- PostgreSQL
- MongoDB

depending on requirements.

---

### Step 6 — Transaction

Define exactly what must be atomic.

---

### Step 7 — Eventing

If another service needs notification:

```text
Payment Service
      ↓
     Kafka
      ↓
Ledger Service
```

---

### Step 8 — Failure handling

You need:

- timeout
- retry
- circuit breaker
- fallback where appropriate
- DLQ
- idempotency
- observability

---

### Step 9 — Deployment

```text
Docker
 ↓
Kubernetes
 ↓
Service
 ↓
Ingress
 ↓
Load balancing
 ↓
Autoscaling
```

---

# 7. Kafka

This is one area where I want you to become **much stronger** because it can distinguish your answers.

We will cover:

### Kafka fundamentals

- broker
- topic
- partition
- producer
- consumer
- consumer group
- offset
- replication
- leader/follower
- ISR
- retention

But senior-level questions:

> Why partition?

> How does Kafka achieve parallelism?

> How does ordering work?

> What happens if consumer crashes?

> What happens after processing but before offset commit?

> How do you achieve effectively-once processing?

> What is consumer lag?

> What happens when one consumer in a group dies?

> How do you choose partition key?

> Why is Kafka not simply a database?

---

# 8. Distributed reliability

You should know this diagram almost instinctively:

```text
                 ┌──────────────┐
Client ─────────►│ API / Ingress│
                 └──────┬───────┘
                        ↓
                 ┌──────────────┐
                 │   Service    │
                 └──────┬───────┘
                        ↓
                 ┌──────────────┐
                 │  PostgreSQL  │
                 └──────────────┘

                        │
                        ↓
                     Kafka
                        │
              ┌─────────┴─────────┐
              ↓                   ↓
        Ledger Service       Notification
```

Now imagine failures.

### DB slow

→ timeout

### Downstream service unavailable

→ timeout + circuit breaker

### Request retried

→ idempotency

### Kafka consumer crashes

→ offset handling

### Message processing fails

→ retry/DLQ

### Two requests modify same wallet

→ transaction + locking/concurrency strategy

### Service receives 100K requests/sec

→ horizontal scaling + load balancing + rate limiting + partitioning/caching as applicable.

That is the thinking level we're targeting.

---

# 9. Concurrency

Your Sitcom list specifically mentions:

> multithreading + async programming + high TPS

So this gets serious attention.

We need:

```text
Thread
Process
Race condition
Critical section
Thread safety
Visibility
Atomicity
Ordering
```

Then:

- `synchronized`
- `volatile`
- `Lock`
- `ReentrantLock`
- `ReadWriteLock`
- `AtomicInteger`
- CAS
- ExecutorService
- ThreadPoolExecutor
- Future
- CompletableFuture
- parallelism
- deadlock
- livelock
- starvation
- concurrent collections

And the very important practical question:

> How do you choose thread-pool size?

For CPU-bound:

```text
threads ≈ number of available CPU cores
```

as a starting point.

For I/O-bound workloads, more threads may be appropriate, but you don't just say:

> "CPU × 2"

You explain that sizing depends on:

- CPU
- blocking time
- task arrival rate
- task duration
- downstream capacity
- memory
- queue size
- latency requirements

That's a much stronger answer.

---

# 10. PostgreSQL / DB internals

This is another area where your preparation needs to go beyond SQL syntax.

You need:

### SQL

- joins
- subqueries
- CTE
- aggregation
- `GROUP BY`
- `HAVING`
- window functions
- indexes
- pagination
- query optimization

Then PostgreSQL:

```text
Table
 ↓
Pages
 ↓
Rows / tuples
 ↓
Indexes
```

And:

- B-tree
- composite indexes
- index selectivity
- sequential scan
- index scan
- `EXPLAIN`
- `EXPLAIN ANALYZE`
- MVCC
- VACUUM
- WAL
- transactions
- isolation levels
- locks
- deadlocks

This becomes **very important for your ledger/payment design**.

---

# 11. Ledger / Payment design

This is where your actual experience can become your strongest weapon.

Suppose they say:

> Design a wallet with 50,000 TPS.

Don't immediately jump to Kafka.

Start with requirements.

### Functional

```text
Credit
Debit
Transfer
Balance
Transaction history
Idempotency
```

### Non-functional

```text
High throughput
Low latency
Strong correctness
Durability
Auditability
No double spending
```

Then schema.

Something like:

```text
account
-------
account_id
customer_id
balance
version
```

and:

```text
ledger_entry
------------
entry_id
transaction_id
account_id
entry_type
amount
created_at
```

Then the interviewer asks:

> Two requests debit the same account simultaneously. What happens?

Now we're discussing:

- transaction isolation
- row-level locks
- optimistic locking
- pessimistic locking
- atomic update
- version column

For example:

```sql
UPDATE account
SET balance = balance - :amount,
    version = version + 1
WHERE account_id = :id
  AND balance >= :amount
  AND version = :version;
```

Then:

> What if Kafka fails after DB commit?

Now we discuss:

### Transactional outbox

```text
DB Transaction
 ├── update ledger
 ├── update account
 └── insert outbox event

             ↓
        Outbox Publisher
             ↓
           Kafka
```

**This is exactly the kind of production-level reasoning I want you practicing.**

---

# 12. System Design

We will prepare at least these:

### Must know

1. Rate limiter
2. Payment service
3. Wallet
4. Ledger
5. Order/payment workflow
6. Notification service
7. URL shortener
8. High-TPS API
9. Kafka-based event architecture

And for every design, follow the same framework:

```text
1. Requirements
2. APIs
3. Data model
4. Architecture
5. Data flow
6. Concurrency
7. Consistency
8. Scaling
9. Failure handling
10. Observability
11. Security
12. Trade-offs
```

That framework will save you when you're under pressure.

---

# 13. Design Patterns

EPAM explicitly asks:

- Singleton
- thread-safe Singleton
- double-checked locking
- Facade
- Proxy
- structural patterns
- composition vs inheritance

So we will know:

### Creational

- Singleton
- Factory
- Abstract Factory
- Builder
- Prototype

### Structural

- Adapter
- Decorator
- Facade
- Proxy
- Composite

### Behavioral

- Strategy
- Observer
- Template Method
- Chain of Responsibility
- Command
- State

But more importantly:

> **Where have you actually seen these in Spring?**

Because Spring is full of patterns.

For example:

```text
@Transactional → proxy
Spring AOP → proxy
JdbcTemplate → template method
BeanFactory → factory-like abstraction
Facade → service/API abstractions
Strategy → interchangeable implementations
```

That's the level we want.

---

# 14. Coding

Your immediate coding list is actually very manageable.

We need to become extremely comfortable with:

### Arrays

- reverse
- rotate
- two pointers
- duplicates
- frequency
- sliding window

### Strings

- palindrome
- character frequency
- first/second unique
- pattern search
- anagrams

### HashMap

- frequency
- two sum
- grouping
- lookup

### Stack

- reverse stack
- balanced parentheses
- monotonic stack basics

### Queue

- BFS
- deque
- sliding window

### Streams

- grouping
- max/min
- sorting
- frequency
- second highest
- duplicates
- partitioning
- joining
- nested collections

You don't need to become a competitive programmer in five days.

You need to be **fluent at interview coding**.

---

# 15. AI — last

You're correct.

Don't burn tomorrow morning on AI.

For EPAM, however, we'll eventually cover:

```text
LLM
 ↓
Prompt
 ↓
Context
 ↓
Embedding
 ↓
Vector DB
 ↓
RAG
 ↓
Tool calling
 ↓
Agent
 ↓
Agentic workflow
```

And because you have actual hackathon experience involving **RAG / knowledge graph / Ollama**, we can turn that into a credible discussion rather than giving you generic AI definitions.

---

# NOW — THE MOST IMPORTANT PART

## We are NOT going to study everything linearly.

That would waste these five days.

We're going to use **interview dependency chains**.

For example:

### Chain 1

```text
Java
 ↓
Collections
 ↓
HashMap
 ↓
Concurrency
 ↓
ConcurrentHashMap
 ↓
ExecutorService
 ↓
CompletableFuture
 ↓
Spring async
 ↓
Microservices
 ↓
Kafka
```

### Chain 2

```text
Spring
 ↓
IoC
 ↓
Bean lifecycle
 ↓
BeanPostProcessor
 ↓
Proxy
 ↓
AOP
 ↓
@Transactional
 ↓
Transaction
 ↓
DB
```

### Chain 3

```text
REST
 ↓
Authentication
 ↓
JWT
 ↓
OAuth2
 ↓
Keycloak
 ↓
RBAC
 ↓
Ingress
 ↓
Load Balancer
 ↓
Kubernetes
```

### Chain 4

```text
PostgreSQL
 ↓
Indexes
 ↓
Query planner
 ↓
EXPLAIN
 ↓
Transactions
 ↓
MVCC
 ↓
Locks
 ↓
Deadlocks
 ↓
High-TPS ledger
```

### Chain 5

```text
Kafka
 ↓
Partition
 ↓
Ordering
 ↓
Consumer Group
 ↓
Offset
 ↓
Delivery semantics
 ↓
Retry
 ↓
DLQ
 ↓
Idempotency
 ↓
Exactly-once / effectively-once
```

This is how we're going to get **senior-level depth without wasting time**.

---

# 🔥 YOUR PLAN FROM TONIGHT

It's already late.

### 9:30 – 10:15

**Core Java + Collections**

Focus:

- ArrayList
- HashMap
- HashSet
- Queue
- Stack
- equals/hashCode
- Comparable/Comparator
- ConcurrentHashMap basics

### 10:15 – 11:00

**Java 8 + Streams**

Focus heavily on:

- `map`
- `filter`
- `flatMap`
- `collect`
- `groupingBy`
- `toMap`
- `Function.identity()`
- `reduce`
- `findFirst`
- `findAny`
- lazy evaluation
- primitive streams

### 11:00 – 11:40

**Spring Core + Spring Boot**

Especially:

- IoC
- DI
- bean lifecycle
- BeanPostProcessor
- proxy
- AOP
- `@Transactional`

### 11:40 – 12:00

**Rapid recall**

No new material.

Then sleep.

---

# 🔥 TOMORROW MORNING — SITCOM

## 7:30 – 8:30
### Spring Security

This is a **priority**.

JWT → OAuth2 → Keycloak → Authentication → Authorization → RBAC → Security Filter Chain.

---

## 8:30 – 9:30
### Microservices

Build one service mentally from scratch.

You should be able to explain:

```text
REST
Security
Validation
Service
DB
Transaction
Kafka
Idempotency
Retry
Circuit breaker
Observability
Deployment
Ingress
Autoscaling
```

---

## 9:30 – 10:15
### Concurrency + Async

- ExecutorService
- thread pool
- CompletableFuture
- race condition
- synchronized
- locks
- atomic variables
- async programming

---

## 10:15 – 11:00
### Payment / Wallet / Ledger design

We will practice:

> **Design a high-TPS wallet.**

This will combine:

- Java
- Spring
- DB
- concurrency
- transactions
- Kafka
- idempotency
- locking
- schema design

This is **extremely valuable for both companies**.

---

## 11:00 – 11:45
### DSA coding

Actually write:

1. reverse array
2. reverse ArrayList
3. string pattern search
4. second-last non-repeating character
5. sliding window

Don't just read solutions.

**Type them.**

---

## 11:45 – 12:30
### REST + pagination + ingress + autoscaling

Cover:

```text
Offset pagination
Cursor pagination
JWT
REST validation
Idempotency
Ingress
Load balancing
Autoscaling
```

---

## 12:30 – 1:00

Eat / get ready.

No heavy studying.

---

# AFTER SITCOM

Then we shift into **EPAM mode**.

Friday evening + Saturday + Sunday + Monday + Tuesday becomes our deeper preparation window.

And this is where I want to take you through the topics **one at a time**, interview-style.

---

# HOW I'LL TEACH YOU

This is important.

I'm not going to dump 100 definitions on you.

For every important topic, we'll use this format:

### Example: `@Transactional`

**Level 1 — What is it?**

Then:

**Level 2 — How does it work?**

Then:

**Level 3 — What happens internally?**

Then:

**Level 4 — Show implementation**

Then:

**Level 5 — Production problem**

> Why doesn't `@Transactional` work on self-invocation?

Then:

**Level 6 — Follow-up**

> What happens when an exception occurs?

Then:

**Level 7 — Database connection**

> How does transaction propagation relate to the DB connection?

Then:

**Level 8 — Senior scenario**

> Your service calls another service inside a transaction. Is the remote call part of the same transaction?

Now we're talking at the level these interviews are likely to probe.

---

# One more critical rule for you

When answering tomorrow, **don't bluff implementation experience**.

If they ask:

> "Have you implemented X?"

and you haven't, say:

> "I haven't implemented that directly in production, but I understand how I would implement it. The approach would be..."

Then explain it.

That's much stronger than claiming experience and getting trapped by:

> "Okay, show me the configuration."

You already experienced this kind of probing in your previous interview. **This time we're preparing specifically for the second and third follow-up.**

---

# Our first actual topic

I recommend we start **right now with Core Java → Collections → HashMap**, because it is the foundation for:

**Java + concurrency + streams + coding + production reasoning.**

And then we'll move:

**HashMap → ConcurrentHashMap → ExecutorService → CompletableFuture → Spring async → Spring → Security → Microservices → Kafka → DB → System Design.**

That gives us one continuous technical story instead of disconnected memorization.

**Let's start with HashMap at senior-interview depth.**