# Spring Boot — Production Application Structure & Layered Architecture ⭐⭐⭐⭐⭐

Now we move to the next Spring Boot section:

> **Controller → Service → Repository → DTO → Entity → Mapper → Exception Layer**

For EPAM, don't just memorize what each annotation means. You should be able to explain **why the layer exists, what belongs there, what should NOT belong there, and how a production request flows through the layers.**

---

# 1. Typical Spring Boot Application Structure

A production Spring Boot application commonly looks like:

```text
src/main/java
└── com.company.payment
    │
    ├── controller
    │
    ├── service
    │
    ├── repository
    │
    ├── entity
    │
    ├── dto
    │
    ├── mapper
    │
    ├── exception
    │
    ├── config
    │
    └── PaymentApplication.java
```

Conceptually:

```text
HTTP Request
     ↓
Controller
     ↓
Service
     ↓
Repository
     ↓
Database
```

with DTOs and mappers around the API/database boundaries:

```text
Request JSON
     ↓
Request DTO
     ↓
Controller
     ↓
Service
     ↓
Entity
     ↓
Repository
     ↓
Database
```

and response:

```text
Database
     ↓
Entity
     ↓
Mapper
     ↓
Response DTO
     ↓
Controller
     ↓
JSON Response
```

---

# 2. Why do we separate layers?

### Interview Question

**Why don't we put everything inside the Controller?**

Suppose we write:

```java
@PostMapping("/payments")
public Payment createPayment(@RequestBody PaymentRequest request) {

    // validate

    // calculate amount

    // check duplicate payment

    // save to database

    // send Kafka event

    // construct response

    return ...;
}
```

This quickly becomes unmaintainable.

The controller now knows:

- HTTP
- business logic
- database
- messaging
- validation
- persistence

Instead:

```text
Controller
   ↓
Service
   ↓
Repository
```

Each layer has a focused responsibility.

This is essentially **separation of concerns**.

---

# 3. Controller Layer

### Interview Question

**What is the responsibility of a Controller?**

The Controller is responsible primarily for **HTTP/API concerns**.

Typical responsibilities:

```text
Receive HTTP request
Parse request
Validate request
Call service
Return HTTP response
Map HTTP-related exceptions
```

Example:

```java
@RestController
@RequestMapping("/payments")
public class PaymentController {

    private final PaymentService paymentService;

    public PaymentController(PaymentService paymentService) {
        this.paymentService = paymentService;
    }

    @PostMapping
    public ResponseEntity<PaymentResponse> create(
            @Valid @RequestBody PaymentRequest request) {

        PaymentResponse response =
                paymentService.createPayment(request);

        return ResponseEntity
                .status(HttpStatus.CREATED)
                .body(response);
    }
}
```

The controller is thin.

---

# 4. What should NOT be inside Controller?

Avoid:

```java
@PostMapping
public ResponseEntity<?> create(...) {

    // 100 lines of business logic

    // SQL

    // transaction management

    // Kafka publishing

    // complex calculations

}
```

The controller should not become the application's business layer.

### Good:

```text
Controller
    ↓
Service
    ↓
Repository
```

### Bad:

```text
Controller
 ├── SQL
 ├── business logic
 ├── Kafka
 ├── calculations
 ├── validation
 └── database
```

---

# 5. Service Layer

### Interview Question

**What is the responsibility of the Service layer?**

The Service layer contains **business/application logic**.

For example:

```java
@Service
public class PaymentService {

    public PaymentResponse createPayment(PaymentRequest request) {

        // business validation

        // check idempotency

        // calculate amount

        // persist payment

        // publish event

        return ...;
    }
}
```

The service coordinates the application's business operation.

---

# 6. Controller vs Service

This distinction is very important.

Suppose the API receives:

```json
{
  "amount": 1000
}
```

Checking whether the HTTP request contains a valid JSON body is an API concern.

Checking whether:

```text
payment amount > user's allowed transaction limit
```

is a business rule.

So:

```text
HTTP/API validation
        ↓
Controller / validation layer

Business validation
        ↓
Service
```

---

# 7. Repository Layer

### Interview Question

**What is the responsibility of Repository?**

The Repository abstracts persistence/database access.

Example:

```java
@Repository
public interface PaymentRepository
        extends JpaRepository<Payment, Long> {

    Optional<Payment> findByIdempotencyKey(String key);
}
```

The service doesn't need to know how the database query is implemented.

It can say:

```java
paymentRepository.findByIdempotencyKey(key);
```

---

# 8. Repository Should Not Contain Business Logic

Bad:

```java
@Repository
public class PaymentRepository {

    public Payment processPayment(...) {
        // business rules
        // payment calculation
        // fraud logic
        // database
    }
}
```

Better:

```text
Service
  ↓
business logic
  ↓
Repository
  ↓
database
```

Repository should focus on persistence.

---

# 9. Entity

### Interview Question

**What is an Entity?**

An Entity represents a persistent domain object mapped to a database table.

Example:

```java
@Entity
@Table(name = "payments")
public class Payment {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private BigDecimal amount;

    private String status;

    private String idempotencyKey;
}
```

Conceptually:

```text
Payment Java object
       ↕
payments database table
```

---

# 10. Entity vs DTO

This is a **very important interview question**.

### Entity

Represents persistence/database state.

```java
@Entity
public class Payment {
    private Long id;
    private BigDecimal amount;
    private String status;
}
```

### DTO

Represents data exchanged across an API boundary.

```java
public class PaymentResponse {
    private String paymentId;
    private BigDecimal amount;
    private String status;
}
```

They serve different purposes.

---

# 11. Why shouldn't we directly expose Entity as REST response?

Suppose:

```java
@Entity
public class User {

    private Long id;
    private String name;
    private String password;
    private String internalRiskScore;
}
```

If you return:

```java
@GetMapping
public User getUser() {
    return user;
}
```

you risk exposing internal fields.

You also tightly couple your API to your database model.

If your database changes:

```text
user_name
```

to:

```text
display_name
```

you shouldn't necessarily have to change your public API.

Therefore:

```text
Database Model ≠ API Contract
```

DTOs help maintain that separation.

---

# 12. DTO

DTO means:

> **Data Transfer Object**

It carries data between boundaries.

Typical DTOs:

```text
PaymentRequest
PaymentResponse
UserRequest
UserResponse
```

Example:

```java
public class PaymentRequest {

    @NotNull
    private BigDecimal amount;

    @NotBlank
    private String currency;
}
```

Response:

```java
public class PaymentResponse {

    private String paymentId;
    private BigDecimal amount;
    private String status;
}
```

---

# 13. Request DTO vs Response DTO

Don't necessarily use one DTO for both.

For example:

```text
PaymentCreateRequest
PaymentUpdateRequest
PaymentResponse
PaymentSummaryResponse
```

Why?

Because the data needed to create something may differ from the data returned.

Example:

```json
POST /payments
```

Request:

```json
{
  "amount": 1000,
  "currency": "INR"
}
```

Response:

```json
{
  "paymentId": "P123",
  "amount": 1000,
  "currency": "INR",
  "status": "SUCCESS",
  "createdAt": "..."
}
```

The client didn't provide:

```text
paymentId
status
createdAt
```

Those are generated by the application.

---

# 14. Mapper

Now we need to convert:

```text
DTO → Entity
```

and:

```text
Entity → DTO
```

That's the Mapper's responsibility.

Example:

```java
@Component
public class PaymentMapper {

    public Payment toEntity(PaymentRequest request) {

        Payment payment = new Payment();

        payment.setAmount(request.getAmount());
        payment.setCurrency(request.getCurrency());

        return payment;
    }

    public PaymentResponse toResponse(Payment payment) {

        PaymentResponse response = new PaymentResponse();

        response.setPaymentId(payment.getId().toString());
        response.setAmount(payment.getAmount());
        response.setCurrency(payment.getCurrency());
        response.setStatus(payment.getStatus());

        return response;
    }
}
```

---

# 15. Why use a Mapper?

Without a mapper, you may end up with:

```java
payment.setAmount(request.getAmount());
payment.setCurrency(request.getCurrency());
payment.setStatus(...);
payment.setCreatedAt(...);
```

spread throughout services/controllers.

A mapper centralizes conversion logic.

Architecture:

```text
Request DTO
     ↓
   Mapper
     ↓
  Entity
     ↓
Repository
```

and:

```text
Entity
   ↓
 Mapper
   ↓
Response DTO
```

---

# 16. Should Mapping happen in Controller or Service?

There isn't a universal rule, but in a layered production application, keeping mapping logic in a dedicated mapper is often cleaner.

For example:

```java
public PaymentResponse createPayment(PaymentRequest request) {

    Payment payment = paymentMapper.toEntity(request);

    payment = repository.save(payment);

    return paymentMapper.toResponse(payment);
}
```

The controller remains focused on HTTP.

---

# 17. Exception Layer

Now suppose:

```java
paymentService.createPayment(request);
```

throws:

```java
PaymentNotFoundException
```

or:

```java
InsufficientBalanceException
```

or:

```java
DuplicatePaymentException
```

We need a consistent way to convert exceptions into HTTP responses.

That's where the exception layer comes in.

Typical structure:

```text
exception/
 ├── PaymentNotFoundException.java
 ├── DuplicatePaymentException.java
 ├── GlobalExceptionHandler.java
 └── ErrorResponse.java
```

---

# 18. Custom Exception

Example:

```java
public class PaymentNotFoundException
        extends RuntimeException {

    public PaymentNotFoundException(String message) {
        super(message);
    }
}
```

Service:

```java
public Payment getPayment(Long id) {

    return paymentRepository.findById(id)
            .orElseThrow(() ->
                new PaymentNotFoundException(
                    "Payment not found: " + id
                )
            );
}
```

---

# 19. Global Exception Handler

Instead of doing this in every controller:

```java
try {
    ...
} catch (...) {
    ...
}
```

we can use:

```java
@RestControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(PaymentNotFoundException.class)
    public ResponseEntity<ErrorResponse> handleNotFound(
            PaymentNotFoundException ex) {

        ErrorResponse error =
                new ErrorResponse(
                    "PAYMENT_NOT_FOUND",
                    ex.getMessage()
                );

        return ResponseEntity
                .status(HttpStatus.NOT_FOUND)
                .body(error);
    }
}
```

Now any controller exception can be handled centrally.

---

# 20. Why is `@RestControllerAdvice` useful?

It provides centralized exception handling across controllers.

Instead of:

```text
Controller A
 └── try/catch

Controller B
 └── try/catch

Controller C
 └── try/catch
```

we have:

```text
Controller A ─┐
Controller B ─┼──→ GlobalExceptionHandler
Controller C ─┘
```

This gives consistent error responses.

---

# 21. Standard Error Response

A production API shouldn't randomly return:

```text
Payment failed
```

or:

```text
NullPointerException
```

Instead define a standard structure.

For example:

```json
{
  "code": "PAYMENT_NOT_FOUND",
  "message": "Payment P123 was not found",
  "timestamp": "2026-10-04T19:30:00Z",
  "path": "/payments/P123",
  "correlationId": "abc-123"
}
```

This is much easier for clients and operations teams.

---

# 22. Complete Request Flow

Let's put everything together.

Client sends:

```http
POST /payments
```

with:

```json
{
  "amount": 1000,
  "currency": "INR"
}
```

Flow:

```text
                HTTP Request
                     ↓
               Controller
                     ↓
              Request DTO
                     ↓
               Validation
                     ↓
                 Service
                     ↓
                 Mapper
                     ↓
                 Entity
                     ↓
               Repository
                     ↓
                Database
                     ↓
                 Entity
                     ↓
                 Mapper
                     ↓
             Response DTO
                     ↓
               Controller
                     ↓
                HTTP 201
```

---

# 23. Where does `@Transactional` belong?

This is a very important practical question.

Suppose:

```java
public PaymentResponse createPayment(...) {

    savePayment();

    updateAccount();

    saveAudit();

}
```

These operations form one business operation.

Typically the transaction boundary belongs at the **service/application layer**:

```java
@Transactional
public PaymentResponse createPayment(PaymentRequest request) {
    ...
}
```

Why?

Because the service represents the business unit of work.

The controller should not generally decide transaction boundaries.

The repository should generally not define the entire business transaction.

---

# 24. Why not put `@Transactional` on Controller?

Technically possible, but usually poor architectural separation.

Controller:

```text
HTTP concern
```

Service:

```text
Business transaction
```

So:

```java
@RestController
class PaymentController {

    // HTTP concerns
}
```

```java
@Service
class PaymentService {

    @Transactional
    public PaymentResponse createPayment(...) {
        // business operation
    }
}
```

This creates a cleaner boundary.

---

# 25. Should Service call another Service?

Yes, but avoid unnecessary chains such as:

```text
Controller
 ↓
Service A
 ↓
Service B
 ↓
Service C
 ↓
Service D
```

without a clear reason.

A service should coordinate meaningful business operations.

For example:

```text
PaymentService
 ├── PaymentRepository
 ├── FraudService
 ├── AccountService
 └── PaymentMapper
```

This can be perfectly reasonable.

But excessive service-to-service coupling becomes difficult to maintain.

---

# 26. Should Repository call another Repository?

Usually avoid it.

A repository should focus on persistence access.

If business logic requires:

```text
PaymentRepository
+
AccountRepository
+
LedgerRepository
```

then the **Service** should coordinate them.

```text
PaymentService
   ├── PaymentRepository
   ├── AccountRepository
   └── LedgerRepository
```

not:

```text
PaymentRepository
   ↓
AccountRepository
   ↓
LedgerRepository
```

---

# 27. Senior Interview Question

### Interviewer:

> "Where should business logic live?"

Strong answer:

> "Business logic should primarily live in the service/domain layer rather than controllers or repositories. Controllers should handle HTTP concerns, repositories should handle persistence, and services should coordinate the business use case and transaction boundary."

Then add:

> "For complex domains, I would avoid putting all domain behavior into one giant service and may introduce domain objects or domain services where appropriate."

That's a much stronger senior-level answer.

---

# 28. Senior Interview Question

### Interviewer:

> "Why shouldn't an Entity be returned directly from a REST controller?"

Answer:

> "An entity represents the persistence model, while a REST API represents an external contract. Returning entities directly couples the API to the database model, can expose internal fields, can trigger lazy-loading problems, and makes API evolution harder. I prefer DTOs at the API boundary and explicit mapping between DTOs and entities."

Very strong answer.

---

# 29. Senior Interview Question

### Interviewer:

> "Why not put SQL directly inside Service?"

Answer:

> "The service should focus on business operations rather than persistence implementation. Keeping database access behind repositories improves separation of concerns, testability and maintainability. It also allows us to change persistence implementation without spreading database-specific logic throughout the business layer."

---

# 30. Senior Interview Question

### Interviewer:

> "Why not put business logic inside Repository?"

Answer:

> "Repositories should abstract persistence. If business logic is placed there, persistence and business concerns become tightly coupled. The service layer should orchestrate the repositories and implement the business use case."

---

# 31. Senior Interview Question

### Interviewer:

> "Do you always need all these layers?"

**No.**

Don't blindly create:

```text
Controller
Service
ServiceImpl
Repository
RepositoryImpl
Mapper
Factory
Helper
Util
```

for every trivial application.

For a very small application, simpler structure can be appropriate.

But for a production enterprise application such as:

```text
Payment
Ledger
PF
Banking
```

separation becomes valuable because the business complexity is significant.

The important principle is:

> **Use boundaries where they provide meaningful separation, not layers for the sake of layers.**

---

# 32. Interface + ServiceImpl — Do We Need It?

You may see:

```java
public interface PaymentService {
    PaymentResponse createPayment(PaymentRequest request);
}
```

and:

```java
@Service
public class PaymentServiceImpl
        implements PaymentService {
}
```

### Is this mandatory?

**No.**

You can simply have:

```java
@Service
public class PaymentService {
}
```

An interface is useful when you have a meaningful abstraction, multiple implementations, or a contract that benefits testing/architecture.

Creating:

```text
PaymentService
PaymentServiceImpl
```

automatically for every service isn't inherently better.

---

# 33. Production Package Structure

For a payment service, one possible structure is:

```text
com.company.payment
│
├── controller
│   └── PaymentController
│
├── service
│   └── PaymentService
│
├── repository
│   └── PaymentRepository
│
├── entity
│   └── Payment
│
├── dto
│   ├── PaymentRequest
│   └── PaymentResponse
│
├── mapper
│   └── PaymentMapper
│
├── exception
│   ├── PaymentNotFoundException
│   ├── DuplicatePaymentException
│   ├── GlobalExceptionHandler
│   └── ErrorResponse
│
├── config
│   └── PaymentConfig
│
└── PaymentApplication
```

This is a conventional layered structure.

---

# 34. But There Is Another Architecture

As applications become large, package-by-layer can become difficult.

Instead of:

```text
controller/
service/
repository/
entity/
```

for the entire application, you can organize by feature:

```text
payment/
    PaymentController
    PaymentService
    PaymentRepository
    PaymentMapper
    PaymentRequest
    PaymentResponse

refund/
    RefundController
    RefundService
    RefundRepository
    RefundMapper
```

This is often called **package-by-feature**.

---

# 35. Package-by-Layer vs Package-by-Feature

### Package by layer

```text
controller
service
repository
dto
```

Advantages:

- familiar
- simple
- easy for smaller applications

Problem:

As application grows:

```text
controller/
    50 controllers

service/
    80 services

repository/
    100 repositories
```

Everything becomes crowded.

### Package by feature

```text
payment/
refund/
settlement/
ledger/
```

Each feature contains its own components.

This can improve modularity.

---

# 36. EPAM Senior-Level Answer

If asked:

> "How would you structure a large Spring Boot application?"

A good answer:

> "For a smaller service, a conventional layered architecture with controller, service, repository and DTO packages is sufficient. For a larger domain, I prefer organizing primarily by business capability or feature, keeping the controller, application/service logic, persistence and DTOs close to the feature. This reduces coupling and makes the codebase easier to navigate and evolve. I would still maintain clear boundaries between API, business and persistence concerns."

---

# 37. Important Anti-Pattern — Fat Controller

```java
@PostMapping
public ResponseEntity<?> pay(...) {

    // validation
    // database lookup
    // fraud calculation
    // payment processing
    // Kafka publishing
    // audit
    // response construction

}
```

Problem:

```text
Hard to test
Hard to maintain
Hard to reuse
Tightly coupled to HTTP
```

---

# 38. Important Anti-Pattern — Fat Service

Moving everything from Controller to Service doesn't automatically solve the problem.

Bad:

```java
@Service
public class PaymentService {

    // 3000 lines

    // validation
    // payment
    // fraud
    // ledger
    // notification
    // mapping
    // SQL
    // Kafka
}
```

The service becomes a **God class**.

Instead, identify meaningful domain responsibilities.

For example:

```text
PaymentService
FraudService
LedgerService
NotificationService
```

and coordinate them appropriately.

---

# 39. Important Anti-Pattern — Entity as Everything

Avoid using one class simultaneously as:

```text
Database Entity
+
Request DTO
+
Response DTO
+
Business object
```

Example:

```java
@Entity
public class Payment {

    // DB fields

    // API fields

    // validation annotations

    // business logic

    // JSON configuration

}
```

This creates excessive coupling.

Prefer:

```text
PaymentRequest
       ↓
Payment
       ↓
PaymentResponse
```

with mapping between boundaries.

---

# 40. Complete Production Flow

You should be able to draw this in an interview:

```text
                 CLIENT
                    │
                    ▼
              HTTP Request
                    │
                    ▼
             ┌─────────────┐
             │ Controller  │
             └──────┬──────┘
                    │
              Request DTO
                    │
                    ▼
             ┌─────────────┐
             │   Service   │
             └──────┬──────┘
                    │
            Business Logic
                    │
             ┌──────┴──────┐
             │             │
             ▼             ▼
          Mapper       Other Services
             │
             ▼
           Entity
             │
             ▼
        ┌─────────────┐
        │ Repository  │
        └──────┬──────┘
               │
               ▼
            Database
               │
               ▼
            Entity
               │
               ▼
            Mapper
               │
               ▼
        Response DTO
               │
               ▼
          Controller
               │
               ▼
          HTTP Response
```

Exceptions can originate anywhere:

```text
Controller
    ↓
Service
    ↓
Repository
    ↓
Exception
    ↓
@RestControllerAdvice
    ↓
Standard Error Response
```

---

# 41. The Senior Mental Model

Don't memorize:

> Controller = `@RestController`

> Service = `@Service`

> Repository = `@Repository`

Instead remember:

```text
Controller
    = API / HTTP boundary

Service
    = Business use case / orchestration / transaction boundary

Repository
    = Persistence abstraction

Entity
    = Persistence model

DTO
    = External/data-transfer contract

Mapper
    = Boundary conversion

Exception Handler
    = Consistent API error mapping
```

And the most important architectural rule:

```text
HTTP concerns
      ↓
Controller

Business concerns
      ↓
Service / Domain

Persistence concerns
      ↓
Repository

Database model
      ↓
Entity

API contract
      ↓
DTO
```

This is the foundation we'll use when we move into the **REST Application** section next.