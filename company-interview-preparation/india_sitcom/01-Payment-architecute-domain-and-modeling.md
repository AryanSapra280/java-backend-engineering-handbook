Absolutely. Let’s do this properly. 🔥

For the next few hours, I’m going to treat this like **pair-programming + interview training**. We’ll build one project, but every piece will teach you something you can reuse in the interview.

We will **not get stuck polishing one topic forever**. I’ll explicitly tell you when a concept is “good enough for Round 1” and move on.

# PART 1 — Build the Payment Service

Our first target:

```text
Client
   ↓
POST /payments
   ↓
PaymentController
   ↓
PaymentService
   ↓
PaymentRepository
   ↓
MongoDB
```

Later we'll add:

```text
JWT
RBAC
Idempotency
Async
Concurrency
Circuit Breaker
Retry
Pagination
Testing
Ledger
```

---

# 1. First question: How would YOU structure this?

Before I show you the implementation, imagine you're asked:

> "Create a payment API using Spring Boot."

A weak implementation would be:

```text
Controller
    ↓
MongoRepository
```

with everything inside the controller.

A reasonable implementation is:

```text
Controller
    ↓
Service
    ↓
Repository
```

But because this company mentions **hexagonal architecture**, let's understand a slightly better structure.

```text
payment-service/
│
├── domain/
│   ├── Payment.java
│   ├── PaymentStatus.java
│   └── ...
│
├── application/
│   ├── PaymentService.java
│   └── ports/
│
├── adapter/
│   ├── in/
│   │   └── PaymentController.java
│   │
│   └── out/
│       └── PaymentRepositoryAdapter.java
│
└── infrastructure/
    └── ...
```

Don't memorize this structure.

Understand the principle:

> **Business logic should not be tightly coupled to Spring, MongoDB, HTTP, Kafka, etc.**

That's the important part.

---

# 2. Domain first

Let's start with our payment.

What information does a payment need?

```text
paymentId
merchantId
customerId
amount
currency
status
createdAt
updatedAt
```

And potentially:

```text
idempotencyKey
```

We'll add that now because it's extremely important in payments.

---

## Payment status

Don't use random strings:

```java
String status = "SUCCESS";
```

Use an enum.

```java
public enum PaymentStatus {

    CREATED,
    PROCESSING,
    SUCCESS,
    FAILED,
    REFUNDED
}
```

### Interview question

**Why enum instead of String?**

Because this:

```java
payment.setStatus("SUCESS");
```

can compile.

But:

```java
payment.setStatus(PaymentStatus.SUCCESS);
```

is type-safe.

The compiler helps us.

---

# 3. Payment domain object

For now:

```java
public class Payment {

    private String paymentId;

    private String merchantId;

    private String customerId;

    private BigDecimal amount;

    private String currency;

    private PaymentStatus status;

    private String idempotencyKey;

    private Instant createdAt;

    private Instant updatedAt;

    // constructors
    // getters
    // setters
}
```

Now stop.

There's an important interview question here.

## Why `BigDecimal`?

Never use:

```java
double amount;
```

for monetary values.

Because floating-point arithmetic can produce precision problems.

For example:

```java
double value = 0.1 + 0.2;
```

can produce something conceptually like:

```text
0.30000000000000004
```

For money, we want decimal arithmetic.

So:

```java
BigDecimal
```

is appropriate.

---

# 4. Another important question: Should amount be mutable?

Imagine:

```java
payment.setAmount(new BigDecimal("5000"));
```

after the payment has already been processed.

That's dangerous.

For financial systems, we want to carefully control state changes.

So a more robust domain model would avoid arbitrary setters and expose meaningful operations.

For example:

```java
public class Payment {

    private final String paymentId;

    private final String merchantId;

    private final String customerId;

    private final BigDecimal amount;

    private final String currency;

    private PaymentStatus status;

    private final String idempotencyKey;

    private final Instant createdAt;

    private Instant updatedAt;

    public Payment(
            String paymentId,
            String merchantId,
            String customerId,
            BigDecimal amount,
            String currency,
            String idempotencyKey) {

        this.paymentId = paymentId;
        this.merchantId = merchantId;
        this.customerId = customerId;
        this.amount = amount;
        this.currency = currency;
        this.idempotencyKey = idempotencyKey;

        this.status = PaymentStatus.CREATED;

        this.createdAt = Instant.now();
        this.updatedAt = this.createdAt;
    }

    public void markProcessing() {
        this.status = PaymentStatus.PROCESSING;
        this.updatedAt = Instant.now();
    }

    public void markSuccess() {
        this.status = PaymentStatus.SUCCESS;
        this.updatedAt = Instant.now();
    }

    public void markFailed() {
        this.status = PaymentStatus.FAILED;
        this.updatedAt = Instant.now();
    }
}
```

This is **encapsulation**.

Instead of:

```java
payment.setStatus(...)
```

anywhere in the application, we control how the state changes.

---

# 5. This is exactly where the interviewer may dig

They could ask:

> "Why don't you expose setters for everything?"

Answer:

> "Because unrestricted setters allow invalid state transitions. I prefer encapsulating state changes inside domain methods so that business invariants can be enforced in one place."

That's a **much stronger answer** than:

> "Encapsulation means wrapping data and methods."

---

# 6. Let's introduce a business rule

Suppose:

```text
SUCCESS → PROCESSING
```

is invalid.

We don't want:

```java
payment.markProcessing();
```

after payment has already succeeded.

So our domain can enforce the transition.

```java
public void markProcessing() {

    if (status != PaymentStatus.CREATED) {
        throw new IllegalStateException(
                "Payment cannot move to PROCESSING from " + status
        );
    }

    status = PaymentStatus.PROCESSING;
    updatedAt = Instant.now();
}
```

Similarly:

```java
public void markSuccess() {

    if (status != PaymentStatus.PROCESSING) {
        throw new IllegalStateException(
                "Payment cannot move to SUCCESS from " + status
        );
    }

    status = PaymentStatus.SUCCESS;
    updatedAt = Instant.now();
}
```

Now we're already introducing a tiny **state machine**.

---

# 7. Why this matters for payments

A payment shouldn't randomly transition:

```text
SUCCESS
   ↓
PROCESSING
```

We want valid transitions.

For example:

```text
CREATED
   ↓
PROCESSING
   ↓
SUCCESS
```

or:

```text
PROCESSING
   ↓
FAILED
```

Later:

```text
SUCCESS
   ↓
REFUNDED
```

This is a great place to demonstrate domain thinking during the interview.

---

# 8. Now our DTO

Don't expose `Payment` directly from the controller.

Create:

```java
public record CreatePaymentRequest(

        @NotBlank
        String merchantId,

        @NotBlank
        String customerId,

        @NotNull
        @DecimalMin(value = "0.01")
        BigDecimal amount,

        @NotBlank
        String currency

) {}
```

This gives us validation.

---

# 9. Why DTO instead of entity?

Interviewer:

> "Why don't you simply accept Payment as the request body?"

Good answer:

> "Because the API contract and persistence/domain model have different responsibilities. A DTO allows me to control what clients can send and prevents them from directly manipulating internal state."

For example, we don't want the client sending:

```json
{
  "status": "SUCCESS"
}
```

The client shouldn't decide whether a payment succeeded.

That's our system's responsibility.

---

# 10. Response DTO

Similarly:

```java
public record PaymentResponse(
        String paymentId,
        String merchantId,
        BigDecimal amount,
        String currency,
        PaymentStatus status,
        Instant createdAt
) {}
```

Notice:

The client gets useful information.

But not necessarily:

```text
internal database fields
internal retry counters
internal provider IDs
internal failure metadata
```

---

# 11. Now the controller

```java
@RestController
@RequestMapping("/api/v1/payments")
public class PaymentController {

    private final PaymentService paymentService;

    public PaymentController(PaymentService paymentService) {
        this.paymentService = paymentService;
    }

    @PostMapping
    public ResponseEntity<PaymentResponse> createPayment(
            @Valid @RequestBody CreatePaymentRequest request) {

        PaymentResponse response =
                paymentService.createPayment(request);

        return ResponseEntity
                .status(HttpStatus.CREATED)
                .body(response);
    }
}
```

---

# 12. Interview question: Why constructor injection?

You might be asked:

> Why not `@Autowired` on the field?

Avoid:

```java
@Autowired
private PaymentService paymentService;
```

Prefer:

```java
private final PaymentService paymentService;

public PaymentController(PaymentService paymentService) {
    this.paymentService = paymentService;
}
```

Benefits:

### 1. Dependency is explicit

You can immediately see what the controller requires.

### 2. Can make it `final`

The dependency doesn't change.

### 3. Easier testing

You can simply:

```java
new PaymentController(mockPaymentService);
```

### 4. No reflection required for basic construction

And Spring can automatically use the single constructor.

---

# 13. Service

Now:

```java
@Service
public class PaymentService {

    private final PaymentRepository paymentRepository;

    public PaymentService(PaymentRepository paymentRepository) {
        this.paymentRepository = paymentRepository;
    }

    public PaymentResponse createPayment(
            CreatePaymentRequest request) {

        Payment payment = new Payment(
                UUID.randomUUID().toString(),
                request.merchantId(),
                request.customerId(),
                request.amount(),
                request.currency(),
                null
        );

        paymentRepository.save(payment);

        return toResponse(payment);
    }

    private PaymentResponse toResponse(Payment payment) {

        return new PaymentResponse(
                payment.getPaymentId(),
                payment.getMerchantId(),
                payment.getAmount(),
                payment.getCurrency(),
                payment.getStatus(),
                payment.getCreatedAt()
        );
    }
}
```

This is where business logic belongs.

Not in the controller.

---

# 14. Repository

Since we're using Mongo:

```java
public interface PaymentRepository
        extends MongoRepository<Payment, String> {
}
```

But there's a design question.

If we're trying to follow hexagonal architecture, we don't necessarily want our domain/application layer directly depending on:

```java
MongoRepository
```

Instead:

```java
public interface PaymentRepositoryPort {

    Payment save(Payment payment);

    Optional<Payment> findById(String paymentId);

    Optional<Payment> findByIdempotencyKey(
            String idempotencyKey);
}
```

Then:

```text
Application
     ↓
PaymentRepositoryPort
     ↓
Mongo Adapter
     ↓
MongoRepository
```

That's much closer to hexagonal architecture.

---

# 15. But here's an important interview judgment

Don't over-engineer every Spring Boot application.

If interviewer says:

> "Build a simple CRUD API."

You don't need:

```text
DDD
Hexagonal
CQRS
Event sourcing
15 interfaces
```

for five endpoints.

If they specifically mention:

> "We use hexagonal architecture."

then demonstrate that you understand it.

Otherwise:

```text
Controller
   ↓
Service
   ↓
Repository
```

is perfectly reasonable.

**This is an important engineering judgment.**

---

# 16. Now let's introduce the first REAL payment concern

Currently:

```java
paymentRepository.save(payment);
```

Anyone can call:

```http
POST /payments
```

twice.

Request 1:

```text
₹1000
```

Request 2:

```text
₹1000
```

We potentially create:

```text
P1 → ₹1000
P2 → ₹1000
```

But what if the second request is actually just a retry of the first?

That's why payment APIs need:

# Idempotency

The client sends:

```http
Idempotency-Key: abc123
```

Our request DTO doesn't contain it because it comes from the HTTP header.

Controller:

```java
@PostMapping
public ResponseEntity<PaymentResponse> createPayment(
        @RequestHeader("Idempotency-Key")
        String idempotencyKey,

        @Valid @RequestBody
        CreatePaymentRequest request) {

    return ResponseEntity
            .status(HttpStatus.CREATED)
            .body(paymentService.createPayment(
                    request,
                    idempotencyKey
            ));
}
```

---

# 17. But now here's an edge case

What if:

```http
Idempotency-Key: abc123
```

is missing?

Should we allow the payment?

For a payment API, I would generally make it required for operations where duplicate execution is dangerous.

So:

```java
@RequestHeader("Idempotency-Key")
```

will cause the request to fail if missing.

But then we should validate:

```text
empty
too long
invalid characters
```

and potentially associate the key with the merchant/customer.

---

# 18. The BIG edge case

Suppose:

```text
Thread A
Idempotency key = ABC
```

and:

```text
Thread B
Idempotency key = ABC
```

at exactly the same time.

Both execute:

```text
findByIdempotencyKey("ABC")
```

Both get:

```text
null
```

Both create payment.

So this is **not enough**:

```java
if (repository.findByIdempotencyKey(key).isPresent()) {
    return existing;
}

repository.save(payment);
```

This has a race condition.

---

# 19. How do we solve it?

At the database level.

Create a **unique index** on:

```text
idempotencyKey
```

Now:

```text
ABC
ABC
```

cannot both be inserted.

This is a fantastic interview concept:

> **Application-level checks alone are insufficient for concurrency-sensitive uniqueness. Enforce the invariant at the database level as well.**

This will come up again when we discuss wallets and ledgers.

---

# 20. Our first checkpoint

At this point you should understand this entire flow:

```text
HTTP request
     ↓
Controller
     ↓
DTO validation
     ↓
Service
     ↓
Domain object
     ↓
Repository port
     ↓
Mongo adapter
     ↓
MongoDB
```

And these concepts:

```text
OOP
Encapsulation
DTO
Validation
Constructor injection
Dependency inversion
Hexagonal architecture
MongoRepository
State transitions
Idempotency
Database uniqueness
Race conditions
```

That's already a substantial chunk of your interview.

---

# Your first interview drill

Don't just read the next section. **Answer these mentally or type your answers to me.**

### Q1

Why shouldn't the controller contain the payment business logic?

### Q2

Why use `BigDecimal` instead of `double` for payment amounts?

### Q3

Why shouldn't the client be allowed to send:

```json
{
    "status": "SUCCESS"
}
```

?

### Q4

Why is this insufficient for idempotency?

```java
if (!repository.existsByIdempotencyKey(key)) {
    repository.save(payment);
}
```

### Q5

Two instances of your service receive the same idempotency key simultaneously. How do you guarantee only one payment is created?

### Q6

If your application has 5 pods, will:

```java
synchronized
```

protect the payment across all 5 pods?

### Q7

What would you use instead?

---

**Don't worry if you can't answer all seven yet.** These questions are deliberately building toward the concurrency/payment discussion we'll do next.

Once you've gone through these, we'll move to **MongoDB implementation + idempotency + global exception handling**, then quickly into **async programming and `CompletableFuture`** rather than spending the whole day on Spring CRUD.
