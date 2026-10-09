# Spring Web / REST — Production APIs: Timeouts, Rate Limiting, Retries & Idempotency ⭐⭐⭐⭐⭐

This is the final **production-focused** part of Spring Web/REST. The important goal is to reason about APIs under **slow dependencies, retries, duplicate requests, and high traffic**.

---

# 1. Timeouts ⭐⭐⭐⭐⭐

## Interview Question

**Why are timeouts important in microservices?**

Suppose:

```text
Payment Service
      |
      v
Banking Service
      |
      | no response
      |
      X
```

If Payment Service waits indefinitely, its request thread can remain occupied.

With enough requests:

```text
Slow dependency
      ↓
Threads remain occupied
      ↓
Thread pool exhausted
      ↓
Requests queue
      ↓
Latency increases
      ↓
Service becomes unhealthy
```

Therefore:

> Every network call should have a bounded timeout appropriate to the operation.

---

# 2. What types of timeouts exist?

You don't need to memorize every HTTP-client-specific setting, but understand the concepts.

### Connection timeout

How long we wait to establish a connection.

```text
Client
  |
  | establish TCP connection
  |
  X
```

If the connection cannot be established within the configured period → timeout.

### Read/response timeout

How long we wait for data from an established connection.

```text
Connection established
        ↓
Waiting for response
        ↓
No response
        ↓
Read timeout
```

### Overall/request timeout

A limit on the entire operation.

For example:

```text
Request budget = 2 seconds
```

This prevents one operation from consuming resources indefinitely.

---

# 3. Senior interview scenario

### Interviewer:

> “Your Payment Service calls an external payment provider. What timeout would you configure?”

Don't say:

> “5 seconds because that's a good value.”

There is no universal magic number.

A strong answer:

> “I'd define the timeout based on the business SLA and downstream behavior. It should be bounded and short enough to prevent resource exhaustion, while allowing legitimate responses. I'd also consider connection and read timeouts separately and make the timeout observable through metrics/logging.”

The exact value should come from requirements and measurements.

---

# 4. Timeout vs retry

This distinction is critical.

Suppose:

```text
Payment Service
      |
      | request
      v
External Provider
      |
      | 3 seconds
      X
```

Payment Service times out.

Should it retry?

**Not automatically.**

You first need to determine whether the operation is safe to retry.

For example:

```text
GET /account/123
```

is usually much safer to retry than:

```text
POST /payments
```

because a POST might have been successfully processed even though the response was lost.

---

# 5. Retries ⭐⭐⭐⭐⭐

### Interview Question

**When should you retry a failed request?**

Retries are appropriate mainly for **transient failures**.

Examples:

```text
Temporary network failure
Temporary connection failure
Temporary service unavailability
Some 5xx responses
```

Not every error should be retried.

Don't retry:

```text
400 Bad Request
401 Unauthorized
403 Forbidden
Invalid business request
```

because retrying the same invalid request doesn't fix the problem.

---

# 6. Retry example

Suppose:

```text
Payment Service
      |
      v
Inventory Service
```

Inventory Service temporarily returns:

```text
503 Service Unavailable
```

A limited retry may make sense.

```text
Attempt 1 → 503
Attempt 2 → 503
Attempt 3 → success
```

But don't retry indefinitely.

---

# 7. Why unlimited retries are dangerous

Imagine:

```text
Service A
   |
   v
Service B is overloaded
```

A sends 10,000 requests.

B returns errors.

A retries every request:

```text
10,000 original
+
10,000 retries
+
10,000 retries
...
```

Now B receives even more traffic.

This can create a **retry storm**.

```text
Failure
  ↓
Retry
  ↓
More traffic
  ↓
More overload
  ↓
More failures
  ↓
More retries
  ↓
System collapse
```

Therefore:

> Retries must be bounded and carefully designed.

---

# 8. Exponential backoff

Instead of:

```text
Retry immediately
Retry immediately
Retry immediately
```

use increasing delays.

For example:

```text
Attempt 1
   ↓
100 ms

Attempt 2
   ↓
200 ms

Attempt 3
   ↓
400 ms

Attempt 4
   ↓
800 ms
```

This is **exponential backoff**.

The exact values depend on the system.

---

# 9. Jitter

If thousands of clients all retry at exactly:

```text
100 ms
200 ms
400 ms
```

they may retry simultaneously.

That's another form of synchronized load.

**Jitter** introduces some randomness into the delay.

Conceptually:

```text
Client A → retry after 183 ms
Client B → retry after 217 ms
Client C → retry after 164 ms
```

This spreads retry traffic.

Senior answer:

> “For distributed systems, I'd generally combine bounded retries with exponential backoff and jitter to avoid synchronized retry storms.”

---

# 10. Retry + timeout interaction

Suppose:

```text
Overall request budget = 3 seconds
```

and you configure:

```text
Timeout per attempt = 3 seconds
Retries = 3
```

You could potentially spend much longer than the desired overall budget.

Therefore:

```text
Overall timeout
       |
       +-- Attempt 1
       +-- Attempt 2
       +-- Attempt 3
```

Retry policy and timeout policy need to be designed together.

---

# 11. Idempotency ⭐⭐⭐⭐⭐

This is especially important for your **payment microservice**.

Suppose:

```http
POST /payments
Idempotency-Key: abc123
```

First request:

```text
Payment created
₹1000 charged
```

But the response is lost:

```text
Payment Service
      |
      X
      |
    Client
```

Client retries:

```http
POST /payments
Idempotency-Key: abc123
```

The server must recognize:

```text
abc123
```

has already been processed.

---

# 12. Basic idempotency implementation

A common approach:

```text
Request
   |
   v
Idempotency-Key
   |
   v
Check storage
   |
   +---- exists ----> return previous result
   |
   +---- doesn't exist
             |
             v
       process request
             |
             v
       store result
             |
             v
        return result
```

For example:

```text
idempotency_key = abc123
status = COMPLETED
response = payment P123
```

---

# 13. Where should the idempotency key be stored?

Potential choices include:

```text
Database
Redis
Distributed cache
```

The correct choice depends on:

- durability requirements
- TTL
- scale
- consistency
- recovery requirements

For financial operations, don't casually assume a short-lived cache is enough.

---

# 14. The race-condition problem

This is a **very important senior-level follow-up**.

Suppose two identical requests arrive at almost exactly the same time:

```text
Request A                 Request B
    |                         |
    v                         v
Check key abc123          Check key abc123
    |                         |
    | not found               | not found
    v                         v
Process payment           Process payment
```

Now you may create two payments.

So simply doing:

```text
if (!exists(key)) {
    process();
}
```

is **not enough**.

---

# 15. How do we solve the idempotency race?

We need an atomic operation.

For example, a database can have:

```text
UNIQUE(idempotency_key)
```

Then:

```text
Request A
   ↓
INSERT abc123
   ↓
SUCCESS

Request B
   ↓
INSERT abc123
   ↓
UNIQUE constraint violation
```

Request B can then retrieve the existing result.

Conceptually:

```text
               Idempotency Store
                      |
             UNIQUE key constraint
                      |
          +-----------+-----------+
          |                       |
      Request A               Request B
          |                       |
       INSERT                  INSERT
          |                       |
       success                duplicate
          |                       |
       process                 fetch result
```

This is much stronger than a non-atomic check-then-act sequence.

---

# 16. Idempotency state machine

For payment operations, you may store states such as:

```text
PROCESSING
SUCCESS
FAILED
```

Example:

```text
abc123
   |
   v
PROCESSING
   |
   +---- SUCCESS
   |
   +---- FAILED
```

If another request arrives while:

```text
abc123 = PROCESSING
```

you need a defined policy.

For example:

```text
Return "request is already being processed"
```

or safely wait/retrieve the eventual result, depending on the API design.

---

# 17. What if the application crashes?

Suppose:

```text
1. Store idempotency key
2. Start payment
3. Application crashes
4. Response never reaches client
```

Now the system needs to know:

```text
Was the payment actually completed?
```

This becomes a distributed consistency problem.

You cannot solve every failure simply with:

```text
try/catch
```

You need to reason about:

```text
Idempotency
Persistence
External provider semantics
Reconciliation
Retries
```

This is exactly the type of reasoning expected at senior level.

---

# 18. Idempotency and database transaction

Suppose:

```text
@Transactional
createPayment()
```

does:

```text
1. Insert payment
2. Store idempotency key
3. Commit
```

These local DB operations can be atomic if they're inside the same transaction.

But suppose the flow is:

```text
Database
   +
External Payment Provider
```

A single DB transaction cannot automatically make both systems atomic.

For example:

```text
DB transaction
     |
     +-- save payment
     |
     +-- call external provider
```

This is a distributed transaction problem.

We'll revisit this deeply in **Microservices**.

---

# 19. Rate limiting ⭐⭐⭐⭐⭐

### Interview Question

**What is rate limiting?**

Rate limiting controls how many requests a client can make within a certain period.

Example:

```text
100 requests / minute
```

If the client exceeds the limit:

```http
429 Too Many Requests
```

---

# 20. Why do we need rate limiting?

It protects the system from:

```text
Abuse
Accidental traffic spikes
Bots
DoS-like traffic
Noisy clients
Resource exhaustion
```

Example:

```text
Normal:
100 requests/minute

Malicious client:
100,000 requests/minute
```

Without protection:

```text
Traffic spike
   ↓
CPU increases
   ↓
Threads exhausted
   ↓
DB connections exhausted
   ↓
Latency increases
   ↓
Service failure
```

---

# 21. Where should rate limiting happen?

Often as early as possible:

```text
Internet
   |
   v
Load Balancer / API Gateway / Ingress
   |
   v
Rate Limiter
   |
   v
Application
```

Why?

Because rejecting unwanted traffic before it reaches the application saves application resources.

---

# 22. Rate limiting algorithms

You should know the major concepts, not implementation internals.

### Fixed Window

Example:

```text
10 requests per minute
```

Counter resets every minute.

Problem:

```text
59:59 → 10 requests
00:00 → 10 requests
```

Potentially 20 requests in a very short period around the boundary.

---

# 23. Sliding Window

Instead of fixed boundaries, the system considers a moving time window.

Conceptually:

```text
Current time
     |
     |---- previous 60 seconds ----|
```

This provides more accurate limiting than a simple fixed window, though implementation/storage can be more complex.

---

# 24. Token Bucket ⭐⭐⭐⭐

Very common concept.

Imagine a bucket containing tokens:

```text
Bucket
+----------------+
| ● ● ● ● ●      |
+----------------+
```

Each request consumes one token.

Tokens are replenished at a fixed rate.

If there are no tokens:

```text
Request
   |
   v
No token
   |
   v
429 Too Many Requests
```

Token bucket can allow controlled bursts while maintaining an average rate.

---

# 25. Rate limiting example

Suppose:

```text
Limit = 100 requests/minute
```

Client sends:

```text
Request 1
Request 2
...
Request 100
```

Allowed.

Request 101:

```http
429 Too Many Requests
```

The server may provide:

```http
Retry-After: 30
```

if applicable to its rate-limit policy.

---

# 26. Where do we store rate-limit state?

For a single application instance:

```text
In-memory counter
```

may work.

But in a distributed deployment:

```text
          Load Balancer
          /     |     \
        Pod A  Pod B  Pod C
```

If each pod maintains its own counter:

```text
Pod A → 100
Pod B → 100
Pod C → 100
```

the client might effectively get:

```text
300 requests
```

instead of the intended 100.

Therefore distributed rate limiting often needs shared state or an infrastructure-level solution.

Redis is a common choice.

---

# 27. Rate limiting architecture

```text
                  Client
                     |
                     v
              API Gateway
                     |
               Rate Limiter
                     |
                Redis/state
                     |
                     v
              Spring Service
```

This allows multiple service instances to share the rate-limit state.

---

# 28. Rate limiting vs throttling

These terms are sometimes used interchangeably, but a useful distinction is:

### Rate limiting

Controls how many requests are allowed.

```text
100 req/min
```

### Throttling

More broadly refers to controlling/reducing processing when demand exceeds capacity.

In interviews, don't get stuck debating terminology. Explain the actual behavior you're implementing.

---

# 29. What happens when rate limit is exceeded?

Typically:

```http
429 Too Many Requests
```

Response could contain:

```json
{
  "code": "RATE_LIMIT_EXCEEDED",
  "message": "Too many requests"
}
```

The API may also provide retry guidance through headers such as:

```http
Retry-After
```

---

# 30. Retry + rate limiting interaction

This is an important distributed-system scenario.

Suppose:

```text
Client
   |
   | request
   v
API
   |
   | 429
   v
Client
```

If the client immediately retries:

```text
retry
retry
retry
retry
```

it makes the rate-limit problem worse.

Good clients should respect server guidance such as:

```text
Retry-After
```

and use appropriate backoff.

---

# 31. Retry vs circuit breaker

This distinction will become important in Microservices.

### Retry

> “The operation failed transiently. Try again.”

### Circuit breaker

> “This dependency is failing repeatedly. Stop sending requests temporarily.”

Example:

```text
Service A
    |
    v
Service B
```

If B repeatedly fails:

```text
Retry
Retry
Retry
Retry
```

is dangerous.

Circuit breaker:

```text
CLOSED
   |
   | failures exceed threshold
   v
OPEN
   |
   | don't call B
   v
Fail fast
```

We'll cover Resilience4j/circuit breakers properly in the Microservices section.

---

# 32. Retry + idempotency + timeout

These three should be considered together.

Suppose:

```text
Payment Service
       |
       v
External Provider
```

We configure:

```text
Timeout = 2 sec
Retry = 2 attempts
Idempotency key = abc123
```

Flow:

```text
Attempt 1
   |
   | timeout
   v
Retry
   |
   v
Attempt 2
   |
   v
Provider succeeds
```

The idempotency key protects against duplicate processing if:

```text
Attempt 1 actually succeeded
but
response was lost
```

This is why these concepts cannot be designed independently.

---

# 33. Senior scenario — Payment API

### Interviewer:

> “Your payment API times out after 3 seconds. Should you retry?”

Strong answer:

> “Not blindly. A payment request is a state-changing operation, so I first need to determine whether the operation is safely retryable. I'd use an idempotency key so retries map to the same logical payment operation. I'd apply a bounded retry policy only for appropriate transient failures, with exponential backoff and jitter, and keep the total retry duration within the request/business timeout budget. For external payment providers, I'd also need reconciliation for ambiguous outcomes.”

That's a **strong senior-level answer**.

---

# 34. Senior scenario — API suddenly receives 10x traffic

### Interviewer:

> “Your service normally receives 1,000 requests/sec but suddenly gets 10,000. What would you do?”

A reasonable answer:

```text
1. Rate limit abusive/noisy clients
2. Check autoscaling
3. Check CPU/memory
4. Check thread pools
5. Check DB connection pool
6. Check downstream dependencies
7. Check queue/backlog
8. Scale horizontally if appropriate
9. Protect critical dependencies
10. Monitor error rate and latency
```

Don't say only:

> “Increase server capacity.”

The bottleneck may be:

```text
Database
Kafka
External API
Connection pool
Thread pool
```

rather than CPU.

---

# 35. API Security

For REST APIs, security typically involves:

```text
Authentication
Authorization
Input validation
TLS
Secure headers
Rate limiting
Secret management
Sensitive-data protection
```

The detailed implementation belongs in our **Spring Security** section.

For now, understand where REST fits:

```text
Client
   |
 HTTPS
   |
   v
Gateway / Ingress
   |
Authentication
   |
Authorization
   |
Rate limiting
   |
   v
Spring Boot API
```

---

# 36. Request tracing

For production debugging, a request should ideally be traceable across services.

Example:

```text
Client
  |
  | traceId = abc123
  v
Payment Service
  |
  | traceId = abc123
  v
Ledger Service
  |
  | traceId = abc123
  v
Notification Service
```

With distributed tracing:

```text
Trace abc123
 ├── Payment API       100 ms
 ├── DB query           50 ms
 ├── Ledger API        300 ms
 └── Kafka operation    20 ms
```

This lets you identify where latency occurs.

We've already covered the observability fundamentals; implementation details will connect again when we cover Microservices.

---

# 37. Production REST checklist

Before exposing an API in production, think about:

```text
API Design
   ↓
Authentication
   ↓
Authorization
   ↓
Validation
   ↓
Idempotency
   ↓
Timeout
   ↓
Retry policy
   ↓
Rate limiting
   ↓
Observability
   ↓
Error handling
   ↓
Backward compatibility
```

Not every endpoint needs every mechanism, but these are the questions you should ask.

---

# 38. EPAM rapid-fire

### Q: What happens when a dependency doesn't respond?

Use a bounded timeout.

### Q: Should every failed request be retried?

No. Retry only appropriate transient failures.

### Q: Why exponential backoff?

To avoid immediately hammering an already-failing dependency.

### Q: Why jitter?

To avoid many clients retrying simultaneously.

### Q: What status code represents rate limiting?

`429 Too Many Requests`.

### Q: Where should rate limiting ideally happen?

Often at the gateway/ingress/edge so unwanted traffic can be rejected before consuming application resources.

### Q: Why isn't in-memory rate limiting enough in Kubernetes?

Because each pod has separate state.

### Q: How do you make a POST payment retry-safe?

Use an idempotency key with atomic persistence/uniqueness and return/reuse the result of the original operation.

### Q: What if two identical payment requests arrive simultaneously?

The idempotency mechanism must be atomic; a simple `check → process` sequence has a race condition.

### Q: Does a timeout mean the operation failed?

Not necessarily.

The operation may have succeeded but the response may have been lost.

This is particularly important for payments.

---

# 39. The most important mental model

For any remote API call:

```text
              REMOTE CALL
                   |
        +----------+----------+
        |          |          |
     Timeout     Retry    Idempotency
        |          |          |
        |       Backoff       |
        |       + Jitter      |
        |          |          |
        +----------+----------+
                   |
              Rate Limiting
                   |
              Circuit Breaker
                   |
              Observability
```

And for incoming traffic:

```text
Client
  |
  v
Ingress / Gateway
  |
  +---- Authentication
  |
  +---- Authorization
  |
  +---- Rate Limiting
  |
  v
Spring Boot
  |
  +---- Validation
  |
  +---- Business Logic
  |
  +---- Idempotency
  |
  v
Dependencies
  |
  +---- Timeout
  +---- Retry
  +---- Circuit Breaker
```

---

# 40. Spring Web / REST — current status

We have now covered:

```text
HTTP fundamentals                  ✅
REST API design                    ✅
Idempotency                        ✅
Spring MVC internals               ✅
Filters / Interceptors             ✅
CORS / CSRF                        ✅
Async processing                   ✅
File upload/download               ✅
Streaming                          ✅
Timeouts                           ✅
Rate limiting                      ✅
Retry considerations               ✅
Production idempotency             ✅
Request tracing                    ✅
Production API design              ✅
```

### What remains?

Only the **deeper implementation details that belong to later sections**, rather than extending Spring Web unnecessarily:

```text
Spring Security
    ↓
JWT / OAuth2 / Keycloak / RBAC

Microservices
    ↓
Circuit Breaker / Retry / Service Discovery /
Gateway / Distributed Transactions / Resilience

JPA/Hibernate
    ↓
Persistence / Transactions / Locking / Pagination /
Performance
```

So **Spring Web / REST is now complete at the interview-relevant level.**

The next major section is:

# **#4 Spring Data JPA / Hibernate ⭐⭐⭐⭐⭐**

And this is a substantial section—we'll start from **JPA vs Hibernate → EntityManager → Persistence Context**, then build toward N+1, transactions, locking, pagination, batching, and production troubleshooting.