Perfect. This is a **good focused scope for tomorrow**. We'll do these in order, and for each one I'll give you **concept + implementation + practical example + interview follow-ups**, not just definitions.

## 1. Service-to-Service Communication

We'll start here.

---

# 1. Why do services need to communicate?

Suppose your PF system has:

```text
Withdrawal Service
        |
        ↓
Account Service
```

Withdrawal needs account information.

There are two broad ways:

### Synchronous

```text
Withdrawal
    |
    | HTTP request
    ↓
Account
    |
    | response
    ↓
Withdrawal
```

The caller waits.

### Asynchronous

```text
Withdrawal
    |
    | Event
    ↓
Kafka
    |
    ↓
Account
```

The caller doesn't wait for Account Service to finish.

This distinction is fundamental.

---

# 2. RestClient

`RestClient` is Spring's modern synchronous HTTP client.

Example:

```java
@Service
public class WithdrawalService {

    private final RestClient restClient;

    public WithdrawalService(RestClient.Builder builder) {
        this.restClient = builder
                .baseUrl("http://account-service")
                .build();
    }

    public AccountResponse getAccount(Long id) {

        return restClient.get()
                .uri("/accounts/{id}", id)
                .retrieve()
                .body(AccountResponse.class);
    }
}
```

Flow:

```text
Withdrawal Service
       |
   RestClient
       |
       ↓
HTTP GET
       |
       ↓
Account Service
```

### Why use RestClient?

It's appropriate when:

* Communication is synchronous
* You want a simple blocking HTTP call
* You're building conventional Spring MVC applications

---

# 3. What happens internally?

When you do:

```java
restClient.get()
    .uri("/accounts/{id}", id)
    .retrieve()
    .body(AccountResponse.class);
```

conceptually:

```text
Java object
    ↓
HTTP request
    ↓
Network
    ↓
Account Controller
    ↓
Business logic
    ↓
Response
    ↓
JSON
    ↓
AccountResponse
```

Jackson typically handles JSON serialization/deserialization.

---

# 4. Handling HTTP errors with RestClient

Suppose Account Service returns:

```text
404 NOT FOUND
```

You can explicitly handle errors.

For example:

```java
return restClient.get()
        .uri("/accounts/{id}", id)
        .retrieve()
        .onStatus(
            status -> status.value() == 404,
            (request, response) -> {
                throw new AccountNotFoundException();
            }
        )
        .body(AccountResponse.class);
```

You should also think about:

```text
4xx → usually caller/request/business issue
5xx → downstream/server issue
timeout → dependency/network issue
```

Don't blindly retry every error.

---

# 5. WebClient

Now:

> "What's the difference between RestClient and WebClient?"

`WebClient` is Spring's **non-blocking/reactive HTTP client**.

Example:

```java
WebClient webClient =
    WebClient.builder()
             .baseUrl("http://account-service")
             .build();
```

Then:

```java
Mono<AccountResponse> account =
    webClient.get()
             .uri("/accounts/{id}", id)
             .retrieve()
             .bodyToMono(AccountResponse.class);
```

The important thing:

```text
RestClient
    ↓
blocking

WebClient
    ↓
non-blocking/reactive
```

### What does non-blocking mean?

With traditional blocking:

```text
Thread
 |
 |---- HTTP request ----|
 |                      |
 | waiting              |
 |                      |
 |<---- response -------|
```

The thread waits.

With reactive WebClient:

```text
Thread
 |
 | start HTTP request
 |
 | can handle other work
 |
 |<--- response/event
 |
 | continue processing
```

So WebClient can be useful for high-concurrency I/O workloads.

### Important interview trap

Don't say:

> "WebClient makes everything asynchronous."

More accurate:

> "WebClient is a non-blocking reactive HTTP client. It returns reactive types such as Mono and Flux, allowing the application to process I/O without blocking the calling thread."

---

# 6. OpenFeign

Feign makes service-to-service REST calls look like calling a Java interface.

Example:

```java
@FeignClient(name = "account-service")
public interface AccountClient {

    @GetMapping("/accounts/{id}")
    AccountResponse getAccount(
        @PathVariable Long id
    );
}
```

Then:

```java
@Service
public class WithdrawalService {

    private final AccountClient accountClient;

    public WithdrawalService(AccountClient accountClient) {
        this.accountClient = accountClient;
    }

    public void process(Long accountId) {

        AccountResponse account =
            accountClient.getAccount(accountId);

        // business logic
    }
}
```

You don't manually write:

```java
HTTP request
URL
headers
JSON conversion
```

Feign abstracts that.

---

# 7. RestClient vs WebClient vs Feign

This is worth memorizing.

|            | RestClient        | WebClient                     | OpenFeign               |
| ---------- | ----------------- | ----------------------------- | ----------------------- |
| Style      | Synchronous       | Reactive/non-blocking         | Declarative             |
| Blocking   | Yes               | No, when used reactively      | Typically blocking      |
| API        | Fluent            | Reactive                      | Interface               |
| Complexity | Simple            | Higher                        | Very simple             |
| Good for   | Normal HTTP calls | Reactive/high-concurrency I/O | Microservice REST calls |

### Interview answer

> "For a straightforward synchronous HTTP call, I can use RestClient. If I'm building a reactive non-blocking application, WebClient is appropriate. For declarative service-to-service communication, OpenFeign is convenient because I define an interface and Spring handles the HTTP invocation."

That's enough.

---

# 8. REST vs gRPC

REST:

```text
HTTP
+
JSON
```

Example:

```http
GET /accounts/123
```

Response:

```json
{
  "id": 123,
  "status": "ACTIVE"
}
```

gRPC:

```text
HTTP/2
+
Protocol Buffers
+
strongly typed contracts
```

You define a `.proto` contract.

Example:

```protobuf
service AccountService {
    rpc GetAccount(GetAccountRequest)
        returns (AccountResponse);
}
```

### REST advantages

* Easy to understand
* Browser/client friendly
* JSON
* Widely supported
* Great for public APIs

### gRPC advantages

* Efficient binary serialization
* Strong contracts
* HTTP/2
* Streaming support
* Often useful for internal service-to-service communication

### Interview answer

> "For external/public APIs, REST is often convenient because of its broad interoperability. For internal high-performance service-to-service communication, gRPC can be attractive because of its binary protocol, strong contracts and HTTP/2 support."

Don't say gRPC is **always faster**. Performance depends on the workload and implementation.

---

# 9. Synchronous vs Asynchronous

This is extremely important.

### Synchronous

```text
A → B
  ← response
```

A waits.

Example:

```text
GET account balance
```

You need the answer before continuing.

### Asynchronous

```text
A → Kafka → B
```

A doesn't need B's immediate response.

Example:

```text
ContributionCreated
        ↓
       Kafka
        ↓
Interest Service
```

The producer can continue.

---

# 10. REST vs Kafka

Don't think:

> REST is better than Kafka.

They solve different problems.

### REST

Use when:

```text
I need an answer NOW.
```

Example:

```text
GET /accounts/123
```

### Kafka

Use when:

```text
I want to notify/process something asynchronously.
```

Example:

```text
ContributionCreated
```

Then:

```text
Kafka
 ├── Ledger
 ├── Interest
 └── Notification
```

### Practical PF example

Suppose withdrawal request needs:

```text
Check account
Check balance
```

These may require synchronous calls because you need the result.

But after withdrawal succeeds:

```text
WithdrawalCompleted
```

you might publish an event:

```text
Kafka
  ├── Ledger
  ├── Notification
  └── Audit
```

That is a very natural architecture.

---

# 11. Error handling between services

Suppose:

```text
Withdrawal
    ↓
Account
```

Account can respond:

### 400

Invalid request.

```text
Bad request
```

### 401

Authentication missing/invalid.

### 403

Authenticated but not authorized.

### 404

Account doesn't exist.

### 409

Business conflict.

Example:

```text
Withdrawal already processed
```

### 500

Server-side failure.

### 503

Service temporarily unavailable.

### Timeout

No response within configured time.

---

# 12. Should we retry all errors?

**No.**

For example:

```text
404 Account Not Found
```

Retrying won't magically create the account.

Similarly:

```text
400 Invalid Request
```

Retrying the same request usually doesn't help.

Retry is more appropriate for **transient failures**, such as:

```text
connection reset
temporary network failure
temporary 503
```

Even then, retry policy should be carefully designed.

---

# 13. Timeout + Retry

Imagine:

```text
Payment
   ↓
Account
```

Configure:

```text
Timeout = 2 sec
Max attempts = 3
```

Flow:

```text
Request
   ↓
wait 2 sec
   ↓
timeout
   ↓
retry
   ↓
wait 2 sec
   ↓
retry
```

But now think:

```text
100 requests
×
3 attempts
=
up to 300 calls
```

If Account Service is already overloaded, retries can make the situation worse.

That's why retry is often combined with:

```text
Timeout
Circuit Breaker
Backoff
Bulkhead
```

---

# 14. Exponential Backoff

Instead of:

```text
Retry immediately
Retry immediately
Retry immediately
```

use increasing delays:

```text
Attempt 1 → fail
       ↓
100ms

Attempt 2 → fail
       ↓
200ms

Attempt 3 → fail
       ↓
400ms
```

Often with jitter:

```text
100ms + random variation
200ms + random variation
400ms + random variation
```

Why jitter?

If thousands of clients retry simultaneously, they can create another traffic spike — a **thundering herd**.

---

# 15. What I'd say in an interview

If they ask:

> "How do you implement service-to-service communication?"

You can answer:

> "For synchronous communication, we can use RestClient, WebClient or OpenFeign depending on the application style. In our microservices, a declarative client such as Feign can simplify REST communication between services. For asynchronous communication, we can use Kafka events. I would also configure connection/read timeouts and apply retries only for transient failures, ideally with backoff. For repeated downstream failures, I would use a circuit breaker such as Resilience4j."

That's a **strong interview answer**.

---

## One thing I want you to remember

Don't memorize:

```text
RestClient = X
WebClient = Y
Feign = Z
```

Understand the decision:

```text
Need simple synchronous HTTP?
        ↓
    RestClient

Reactive/non-blocking application?
        ↓
    WebClient

Declarative REST service client?
        ↓
    OpenFeign

Need asynchronous communication?
        ↓
      Kafka
```

---

### Next: Resilience & Fault Tolerance

We'll now go properly into:

**Circuit Breaker → Retry → Timeout → Bulkhead → Rate Limiter → Fallback → Resilience4j configuration → how to combine them without creating a mess.**
