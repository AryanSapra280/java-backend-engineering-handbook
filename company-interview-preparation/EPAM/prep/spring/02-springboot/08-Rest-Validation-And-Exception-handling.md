# Spring Boot — REST Validation & Global Exception Handling ⭐⭐⭐⭐⭐

We have covered the basic REST API layer. Now we go deeper into two areas that are **very frequently tested in senior interviews**:

1. **Request validation**
2. **Global exception handling**

The important part is not just knowing `@Valid` and `@ControllerAdvice`. You should understand **where validation happens, how Spring performs it, what response is generated when validation fails, and how to design a production-grade error response.**

---

# 1. Why do we need validation?

Suppose our payment API accepts:

```json
{
  "amount": -1000,
  "currency": "",
  "email": "invalid"
}
```

We don't want invalid data to reach:

```text
Controller
    ↓
Service
    ↓
Database
```

We want:

```text
HTTP Request
     ↓
Validation
     ↓
Invalid?
   ↙     ↘
 YES      NO
  ↓        ↓
400       Service
```

Validation protects the application boundary.

---

# 2. Bean Validation

Spring Boot commonly uses Jakarta Bean Validation.

Typical annotations include:

```java
@NotNull
@NotBlank
@NotEmpty
@Size
@Min
@Max
@Positive
@PositiveOrZero
@Negative
@Email
@Pattern
@Past
@Future
```

Example:

```java
public class PaymentRequest {

    @NotNull
    @Positive
    private BigDecimal amount;

    @NotBlank
    private String currency;

    @Email
    private String customerEmail;
}
```

---

# 3. `@NotNull` vs `@NotEmpty` vs `@NotBlank`

This is a common interview question.

### `@NotNull`

Checks that the value isn't `null`.

```java
@NotNull
private String name;
```

These are different:

```text
null       → invalid
""         → valid
"   "      → valid
"John"     → valid
```

---

### `@NotEmpty`

For strings/collections/arrays, checks that the value isn't null and isn't empty.

```text
null    → invalid
""      → invalid
"abc"   → valid
```

But whitespace may still pass.

---

### `@NotBlank`

For strings.

It checks that the value isn't null and contains at least one non-whitespace character.

```text
null       → invalid
""         → invalid
"   "      → invalid
"John"     → valid
```

### Interview shortcut

```text
@NotNull
→ not null

@NotEmpty
→ not null + not empty

@NotBlank
→ not null + not empty + not only whitespace
```

---

# 4. Numeric Validation

Suppose:

```java
@Positive
private BigDecimal amount;
```

Then:

```text
100     → valid
1       → valid
0       → invalid
-10     → invalid
```

For:

```java
@PositiveOrZero
private BigDecimal amount;
```

then:

```text
100 → valid
0   → valid
-1  → invalid
```

---

# 5. `@Size`

Example:

```java
@Size(min = 3, max = 20)
private String username;
```

This restricts the size.

It can also be used for collections:

```java
@Size(max = 10)
private List<String> recipients;
```

---

# 6. `@Pattern`

Used when a value must match a regular expression.

Example:

```java
@Pattern(
    regexp = "^[A-Z]{3}$"
)
private String currency;
```

Valid:

```text
INR
USD
EUR
```

Invalid:

```text
India
inr
1234
```

Be careful not to turn every validation problem into a giant regex.

Use the most appropriate built-in constraint when possible.

---

# 7. `@Email`

Example:

```java
@Email
private String email;
```

This performs basic email-format validation.

It does **not** prove that:

```text
the email exists
```

or:

```text
the user owns the email
```

That's business/application logic, not simple format validation.

---

# 8. Activating Validation

Suppose:

```java
public class PaymentRequest {

    @NotNull
    @Positive
    private BigDecimal amount;

    @NotBlank
    private String currency;
}
```

Controller:

```java
@PostMapping
public ResponseEntity<PaymentResponse> create(
        @Valid @RequestBody PaymentRequest request) {

    return ResponseEntity
            .status(HttpStatus.CREATED)
            .body(paymentService.createPayment(request));
}
```

The important annotation is:

```java
@Valid
```

It tells Spring to validate the object.

---

# 9. What happens when validation fails?

Suppose:

```json
{
  "amount": -100,
  "currency": ""
}
```

and:

```java
@Positive
private BigDecimal amount;

@NotBlank
private String currency;
```

Spring detects validation failures **before the controller method proceeds normally**.

The request can result in a validation-related exception, commonly:

```text
MethodArgumentNotValidException
```

for a validated `@RequestBody`.

The exact exception depends on how the validation is triggered.

---

# 10. Important: Validation is not Business Logic

This distinction matters a lot.

### Structural validation

```java
@NotNull
@Positive
@Size(max = 100)
```

This answers:

> Is the request structurally valid?

### Business validation

Suppose:

```text
Customer's daily limit = ₹50,000
Requested payment = ₹80,000
```

The request may pass:

```text
@NotNull
@Positive
```

but the payment is still invalid.

That belongs in the business/application layer.

```text
Request validation
       ↓
Bean Validation

Business validation
       ↓
Service / Domain
```

---

# 11. Example

```java
public class PaymentRequest {

    @NotNull
    @Positive
    private BigDecimal amount;

    @NotBlank
    private String currency;
}
```

This is structural validation.

Then:

```java
@Service
public class PaymentService {

    public PaymentResponse createPayment(
            PaymentRequest request) {

        if (request.getAmount()
                .compareTo(dailyLimit) > 0) {

            throw new PaymentLimitExceededException();
        }

        ...
    }
}
```

That's business validation.

---

# 12. `@Valid` vs `@Validated`

This is an important senior interview question.

### `@Valid`

Primarily comes from Jakarta Bean Validation.

Used for validating an object and cascading into nested objects when configured appropriately.

Example:

```java
@Valid
@RequestBody
PaymentRequest request
```

### `@Validated`

Spring's validation annotation.

It supports **validation groups** and is also commonly used for method-level validation.

Example:

```java
@Validated
@Service
public class PaymentService {
}
```

The simple mental model:

```text
@Valid
→ validate object

@Validated
→ Spring validation + groups/method-level scenarios
```

---

# 13. Nested Object Validation

Suppose:

```java
public class PaymentRequest {

    @NotNull
    @Valid
    private CustomerRequest customer;
}
```

and:

```java
public class CustomerRequest {

    @NotBlank
    private String name;

    @Email
    private String email;
}
```

The important part is:

```java
@Valid
private CustomerRequest customer;
```

Without cascading validation, nested validation may not happen as expected.

Flow:

```text
PaymentRequest
      ↓
@Valid
      ↓
CustomerRequest
      ↓
@NotBlank / @Email
```

---

# 14. Collection Validation

Suppose:

```java
public class PaymentBatchRequest {

    @Valid
    private List<PaymentRequest> payments;
}
```

Then each element can be validated.

For example:

```text
payments
 ├── PaymentRequest 1 → valid
 ├── PaymentRequest 2 → invalid
 └── PaymentRequest 3 → valid
```

The `@Valid` annotation enables cascading validation.

---

# 15. Method-Level Validation

Suppose:

```java
@Service
@Validated
public class PaymentService {

    public PaymentResponse getPayment(
            @Positive Long paymentId) {

        ...
    }
}
```

Now method parameters can be validated.

This is different from simply validating a request DTO.

It is useful for service/API method contracts.

---

# 16. Custom Validation

Sometimes built-in annotations aren't enough.

Suppose the payment request requires:

```text
startDate < endDate
```

This involves multiple fields.

A simple:

```java
@NotNull
```

cannot express the relationship.

We can create a custom constraint.

Conceptually:

```java
@ValidPaymentDates
public class PaymentRequest {
    private LocalDate startDate;
    private LocalDate endDate;
}
```

The validator implements the business-independent validation rule.

---

# 17. Cross-Field Validation

Example:

```text
paymentType = CARD
```

requires:

```text
cardNumber != null
```

while:

```text
paymentType = UPI
```

requires:

```text
upiId != null
```

This isn't a simple field-level validation.

A custom class-level validator can enforce it.

Important distinction:

> Validation can contain request consistency rules, but actual business decisions should remain in the appropriate domain/service layer.

---

# 18. Global Exception Handling

Now let's say something goes wrong.

Without global exception handling, controllers may become:

```java
try {
    ...
} catch (...) {
    ...
}
```

everywhere.

This is bad because:

```text
Controller A → custom error format
Controller B → different error format
Controller C → another format
```

Instead:

```text
All Controllers
      ↓
@RestControllerAdvice
      ↓
Centralized exception mapping
      ↓
Standard ErrorResponse
```

---

# 19. `@ExceptionHandler`

Example:

```java
@RestControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(PaymentNotFoundException.class)
    public ResponseEntity<ErrorResponse> handleNotFound(
            PaymentNotFoundException ex) {

        ErrorResponse response =
                new ErrorResponse(
                    "PAYMENT_NOT_FOUND",
                    ex.getMessage()
                );

        return ResponseEntity
                .status(HttpStatus.NOT_FOUND)
                .body(response);
    }
}
```

Now:

```java
throw new PaymentNotFoundException(...);
```

can become:

```http
404 Not Found
```

with a controlled response body.

---

# 20. `@ControllerAdvice` vs `@RestControllerAdvice`

### `@ControllerAdvice`

Global advice for controllers.

It can be used for:

- exception handling
- model binding
- shared controller behavior

### `@RestControllerAdvice`

Equivalent conceptually to:

```java
@ControllerAdvice
@ResponseBody
```

It is particularly convenient for REST APIs because handler results are written directly to the response body.

For a REST API:

```java
@RestControllerAdvice
```

is generally the convenient choice.

---

# 21. Handling Validation Errors

For:

```java
@Valid
@RequestBody PaymentRequest request
```

validation failures can produce:

```text
MethodArgumentNotValidException
```

A global handler can process it.

Example:

```java
@ExceptionHandler(MethodArgumentNotValidException.class)
public ResponseEntity<ErrorResponse> handleValidation(
        MethodArgumentNotValidException ex) {

    ...
}
```

We can extract field errors.

---

# 22. Field-Level Error Response

Suppose the request is:

```json
{
  "amount": -100,
  "currency": ""
}
```

We might return:

```json
{
  "code": "VALIDATION_ERROR",
  "message": "Request validation failed",
  "fieldErrors": {
    "amount": "must be greater than 0",
    "currency": "must not be blank"
  }
}
```

This is much more useful than:

```json
{
  "message": "Validation failed"
}
```

because the client knows exactly what needs correction.

---

# 23. Don't Expose Internal Exceptions

Bad:

```json
{
  "exception": "org.hibernate.exception.ConstraintViolationException",
  "stackTrace": "...",
  "sql": "insert into payments..."
}
```

This can leak:

- implementation details
- database information
- internal class names
- SQL
- security-sensitive information

Instead:

```json
{
  "code": "PAYMENT_ERROR",
  "message": "Unable to process payment",
  "correlationId": "abc-123"
}
```

Detailed information should go into internal logs.

---

# 24. Exception Mapping Strategy

A production API can define mappings such as:

```text
PaymentNotFoundException
        ↓
404

DuplicatePaymentException
        ↓
409

Validation exception
        ↓
400

Authentication exception
        ↓
401

Authorization exception
        ↓
403

Unexpected exception
        ↓
500
```

This gives a predictable API contract.

---

# 25. Catching `Exception.class`

You can have:

```java
@ExceptionHandler(Exception.class)
public ResponseEntity<ErrorResponse> handle(Exception ex) {
    ...
}
```

This acts as a final safety net.

But don't use it to hide every problem.

A good strategy is:

```text
Specific exceptions
        ↓
Specific HTTP mappings

Unexpected exception
        ↓
Generic safe 500 response
        ↓
Detailed internal logging
```

---

# 26. What if the service catches the exception first?

Suppose:

```java
public PaymentResponse createPayment(...) {

    try {
        ...
    } catch (PaymentNotFoundException ex) {
        ...
    }
}
```

Then the exception may never reach:

```text
@RestControllerAdvice
```

because it was already handled.

This is why exception ownership matters.

Don't catch exceptions merely to rethrow them without adding useful context.

---

# 27. Checked vs Unchecked Exceptions

For application/business exceptions, you'll commonly see:

```java
public class PaymentNotFoundException
        extends RuntimeException {
}
```

Why unchecked?

Because callers don't necessarily need to clutter every layer with:

```java
throws PaymentNotFoundException
```

Spring applications commonly use unchecked exceptions for business/application failures.

This also interacts with Spring transaction rollback behavior, which we already covered.

---

# 28. Error Response Design

A good error response might look like:

```java
public class ErrorResponse {

    private String code;
    private String message;
    private Instant timestamp;
    private String path;
    private String correlationId;
}
```

Example:

```json
{
  "code": "PAYMENT_NOT_FOUND",
  "message": "Payment P123 was not found",
  "timestamp": "2026-10-04T18:30:00Z",
  "path": "/payments/P123",
  "correlationId": "9f3d..."
}
```

The exact fields depend on the organization's API contract.

---

# 29. Why Include a Correlation ID?

Imagine a request:

```text
Client
  ↓
API Gateway
  ↓
Payment Service
  ↓
Ledger Service
  ↓
Kafka
  ↓
Notification Service
```

Something fails.

Without a correlation ID:

```text
Which logs belong to this request?
```

With:

```text
correlationId=abc123
```

we can search across services:

```text
Payment Service → abc123
Ledger Service  → abc123
Notification    → abc123
```

This is extremely useful in production debugging.

---

# 30. Validation vs Exception Handling

Don't confuse these concepts.

### Validation

Detect invalid input:

```text
amount = -100
```

### Exception handling

Determine how failures are converted into API responses:

```text
PaymentNotFoundException
        ↓
404 response
```

So:

```text
Validation
    ↓
Detect problem

Exception Handler
    ↓
Map problem → HTTP response
```

---

# 31. Practical EPAM Scenario

### Interviewer:

> "A client sends an invalid amount. Where do you validate it?"

Strong answer:

> "For basic request constraints such as non-null, positive values, length and format, I'd use Jakarta Bean Validation on the request DTO with `@Valid`. If the validation depends on business state, such as a customer's transaction limit, I'd perform that validation in the service/domain layer."

---

# 32. Practical EPAM Scenario

### Interviewer:

> "How would you return validation errors?"

Strong answer:

> "I'd use a global `@RestControllerAdvice` and handle the validation exception, extracting field-level errors into a consistent error response. I'd return a client-error status such as 400 according to the API contract, while avoiding internal implementation details."

---

# 33. Practical EPAM Scenario

### Interviewer:

> "What happens if a repository throws an unexpected database exception?"

A strong production answer:

```text
Repository
   ↓
Database exception
   ↓
Service/transaction infrastructure
   ↓
Global exception handling
   ↓
Safe 5xx response
   ↓
Detailed internal logging
```

I would not expose the raw database exception to the client.

I'd also investigate whether the error is:

```text
Transient
Permanent
Constraint violation
Timeout
Connection pool exhaustion
Deadlock
```

because the operational response can differ.

---

# 34. Practical EPAM Scenario

### Interviewer:

> "Should you catch every exception in the controller?"

**No.**

Controllers should generally not be full of:

```java
try {
    ...
} catch (Exception e) {
    ...
}
```

Instead use centralized exception handling.

```text
Controller
     ↓
Service
     ↓
Exception
     ↓
@RestControllerAdvice
```

This keeps controllers clean and gives consistent API behavior.

---

# 35. Practical EPAM Scenario

### Interviewer:

> "What if two controllers need the same exception handling?"

That's exactly what `@RestControllerAdvice` is useful for.

```text
PaymentController ─┐
OrderController   ─┼──→ GlobalExceptionHandler
UserController    ─┘
```

---

# 36. Important Trap — HTTP 500 for Every Exception

Don't do:

```java
@ExceptionHandler(Exception.class)
public ResponseEntity<?> handle(Exception e) {
    return ResponseEntity
            .status(500)
            .body(...);
}
```

for every known business failure.

For example:

```text
PaymentNotFoundException
```

shouldn't become:

```text
500
```

It is normally:

```text
404
```

Similarly:

```text
DuplicatePaymentException
```

could map to:

```text
409
```

The API should communicate the nature of the failure accurately.

---

# 37. Important Trap — Validation in Service Only

Suppose:

```java
public PaymentResponse create(PaymentRequest request) {

    if (request.getAmount() == null) {
        ...
    }

    if (request.getCurrency() == null) {
        ...
    }

    ...
}
```

This is unnecessary duplication when these are simple request constraints.

Prefer:

```java
@NotNull
@Positive
private BigDecimal amount;

@NotBlank
private String currency;
```

and:

```java
@Valid
@RequestBody PaymentRequest request
```

Then reserve service validation for actual business rules.

---

# 38. Important Trap — Validation Means Business Logic

Consider:

```text
amount > 0
```

That's a straightforward input constraint.

But:

```text
amount <= customer's remaining daily limit
```

requires business state.

So:

```text
@NotNull
@Positive
@Size
@Email
```

→ Bean Validation

while:

```text
daily transaction limit
account status
fraud rules
payment eligibility
```

→ Business/domain logic.

---

# 39. Complete REST Error Flow

You should be able to draw:

```text
                 HTTP Request
                      │
                      ▼
                 Controller
                      │
               @Valid DTO
                      │
              ┌───────┴────────┐
              │                │
          Invalid             Valid
              │                │
              ▼                ▼
       Validation Error      Service
              │                │
              │          Business Logic
              │                │
              │           Repository
              │                │
              │             Database
              │                │
              │                ▼
              │             Response
              │                │
              └───────┬────────┘
                      ▼
           Global Exception Handler
                      │
                      ▼
              Standard Error JSON
```

---

# 40. Senior-Level Mental Model

Remember these boundaries:

```text
Request format/structure
        ↓
Bean Validation

Business rules
        ↓
Service / Domain

Persistence failures
        ↓
Repository / infrastructure

HTTP error mapping
        ↓
@RestControllerAdvice

Operational details
        ↓
Logs + metrics + tracing
```

And the golden rule:

> **Don't expose internal exceptions directly to API consumers. Translate internal failures into stable, meaningful API errors.**

---

# 41. EPAM Questions You Should Now Be Ready For

### Core

**What does `@Valid` do?**

→ Triggers Bean Validation on the request object.

**`@Valid` vs `@Validated`?**

→ `@Valid` for object validation; `@Validated` adds Spring validation features such as groups and method-level validation.

**`@NotNull` vs `@NotEmpty` vs `@NotBlank`?**

→ Null check vs null/empty check vs null/empty/whitespace check.

**How do you validate nested objects?**

→ Use cascading `@Valid`.

---

### Exception handling

**What is `@ExceptionHandler`?**

→ Maps an exception to controller response handling.

**What is `@RestControllerAdvice`?**

→ Centralized REST controller exception/advice handling with response bodies.

**Why use global exception handling?**

→ Consistent API errors and less duplicated controller code.

**How do you handle unexpected exceptions?**

→ Generic safe 5xx response + detailed internal logging.

---

### Senior

**Where should validation happen?**

→ Structural validation at the API boundary; business validation in service/domain logic.

**Should all exceptions become 500?**

→ No. Map known failures to appropriate statuses.

**Why correlation IDs in errors?**

→ Allow clients/support teams to correlate an API error with distributed-service logs/traces.

**Should stack traces be returned to clients?**

→ No. Log them internally; return safe, stable error information.

**How do you design validation errors?**

→ Stable error code/message plus field-level errors, while keeping internal details out of the response.