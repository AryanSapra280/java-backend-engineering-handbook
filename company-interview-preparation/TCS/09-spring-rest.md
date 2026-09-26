Absolutely! 🔥 **Spring REST — fast interview mode.** This is high-yield, so I'll cover the questions you’re most likely to face without going too deep into internals.

# Spring REST — Part 1

## 1. What is REST?

> REST (Representational State Transfer) is an architectural style for designing web APIs around resources, using standard HTTP methods and stateless communication.

Example:

```text
GET    /users/123
POST   /users
PUT    /users/123
PATCH  /users/123
DELETE /users/123
```

Think:

```text
/users/123
    ↑
Resource
```

---

# 2. What does "stateless" mean in REST?

🔥 Very common.

Each request should contain the information necessary for the server to process it.

The server should not depend on previous request state stored in the server-side session.

For example, with JWT:

```text
Request 1 → JWT
Request 2 → JWT
Request 3 → JWT
```

Each request carries authentication information.

### Important

Stateless does **not** mean:

> "The server cannot have a database."

It means the server doesn't maintain conversational client state between requests as a requirement for processing subsequent requests.

---

# 3. `@Controller` vs `@RestController`

### `@Controller`

Typically used for MVC applications returning views.

```java
@Controller
class UserController {
}
```

### `@RestController`

Used for REST APIs.

It effectively combines:

```java
@Controller
@ResponseBody
```

```java
@RestController
class UserController {
}
```

The return value is written to the HTTP response body, commonly as JSON.

---

# 4. What is `@RequestMapping`?

Used to map HTTP requests to controller methods/classes.

```java
@RequestMapping("/users")
@RestController
class UserController {

    @RequestMapping("/123")
    public User getUser() {
        ...
    }
}
```

You can specify method:

```java
@RequestMapping(
    value = "/users",
    method = RequestMethod.GET
)
```

But usually we use:

```java
@GetMapping
@PostMapping
@PutMapping
@PatchMapping
@DeleteMapping
```

---

# 5. `@GetMapping` vs `@PostMapping`

Simple:

```java
@GetMapping("/users")
```

→ Retrieve resources.

```java
@PostMapping("/users")
```

→ Create a resource / perform a creation-oriented operation.

Example:

```java
@PostMapping("/users")
public User create(@RequestBody UserRequest request) {
    ...
}
```

---

# 6. `@PathVariable` vs `@RequestParam`

🔥 Very common.

### Path Variable

Used when the value identifies part of the resource path.

```text
GET /users/123
```

```java
@GetMapping("/users/{id}")
public User getUser(@PathVariable Long id) {
}
```

### Request Parameter

Used for filtering/options/query criteria.

```text
GET /users?page=2&size=20
```

```java
@GetMapping("/users")
public List<User> getUsers(
        @RequestParam int page,
        @RequestParam int size) {
}
```

Easy rule:

```text
/users/123
       ↑
PathVariable

/users?page=2
       ↑
RequestParam
```

---

# 7. What is `@RequestBody`?

Maps the HTTP request body to a Java object.

Request:

```json
{
  "name": "Aryan",
  "amount": 1000
}
```

Controller:

```java
@PostMapping("/payments")
public Payment create(
        @RequestBody PaymentRequest request) {
}
```

Spring uses an HTTP message converter, commonly Jackson for JSON, to deserialize JSON into the Java object.

---

# 8. What is serialization/deserialization?

### Serialization

Java object → JSON

```text
Java Object
    ↓
JSON
```

### Deserialization

JSON → Java object

```text
JSON
 ↓
Java Object
```

In Spring Boot REST APIs, Jackson commonly handles this.

---

# 9. Why should we use DTOs instead of exposing entities?

🔥 Very important.

Don't normally do:

```java
@GetMapping
public UserEntity getUser() {
    return repository.findById(...);
}
```

Prefer:

```java
@GetMapping
public UserResponse getUser() {
    ...
}
```

Reasons:

* Don't expose internal database structure
* Control API contract
* Avoid accidentally exposing sensitive fields
* Prevent tight coupling between DB model and API
* Better versioning
* Avoid some serialization problems with JPA relationships
* Allows request and response models to differ

Typical:

```text
Controller
   ↓
Request DTO
   ↓
Service
   ↓
Entity
   ↓
Repository
```

and:

```text
Repository
   ↓
Entity
   ↓
Service
   ↓
Response DTO
   ↓
Controller
```

---

# 10. What is `ResponseEntity`?

Allows you to control:

* HTTP status
* Headers
* Response body

Example:

```java
return ResponseEntity
        .status(HttpStatus.CREATED)
        .body(response);
```

Instead of simply:

```java
return response;
```

Use it when you need explicit response metadata/control.

---

# 11. Important HTTP status codes

🔥 Know these.

| Status                      | Meaning                                                                         |
| --------------------------- | ------------------------------------------------------------------------------- |
| `200 OK`                    | Successful request                                                              |
| `201 Created`               | Resource successfully created                                                   |
| `202 Accepted`              | Request accepted for asynchronous processing                                    |
| `204 No Content`            | Successful, no response body                                                    |
| `400 Bad Request`           | Invalid request                                                                 |
| `401 Unauthorized`          | Authentication missing/invalid                                                  |
| `403 Forbidden`             | Authenticated but not allowed                                                   |
| `404 Not Found`             | Resource doesn't exist                                                          |
| `409 Conflict`              | Conflict with current resource/state                                            |
| `422 Unprocessable Content` | Request understood but validation/business semantics fail; usage depends on API |
| `500 Internal Server Error` | Unexpected server error                                                         |
| `502 Bad Gateway`           | Gateway/proxy received bad upstream response                                    |
| `503 Service Unavailable`   | Service temporarily unavailable                                                 |

### Classic trap

**401 vs 403**

```text
401 → Who are you?
403 → I know who you are, but you're not allowed.
```

---

# 12. What is `@Valid`?

Used to trigger Bean Validation.

DTO:

```java
public class PaymentRequest {

    @NotNull
    private Long accountId;

    @Positive
    private BigDecimal amount;
}
```

Controller:

```java
@PostMapping("/payments")
public PaymentResponse create(
        @Valid @RequestBody PaymentRequest request) {
}
```

Spring validates the request before entering your business logic.

---

# 13. `@NotNull` vs `@NotEmpty` vs `@NotBlank`

🔥 Very common.

### `@NotNull`

Value cannot be null.

```java
@NotNull
String name;
```

Empty string is allowed.

```text
"" → valid
null → invalid
```

### `@NotEmpty`

Cannot be null or empty.

```text
null → invalid
"" → invalid
"abc" → valid
```

Works with strings/collections/maps/arrays.

### `@NotBlank`

For strings.

Rejects:

```text
null
""
"   "
```

So:

```text
NotNull  → only null check

NotEmpty → null + empty

NotBlank → null + empty + whitespace
```

---

# 14. What happens when validation fails?

For:

```java
@Valid @RequestBody PaymentRequest request
```

Spring detects the validation failure and throws an exception such as `MethodArgumentNotValidException` in the common MVC request-body case.

Instead of allowing random error responses, we usually handle it globally.

---

# 15. What is `@ControllerAdvice`?

🔥 Very important.

Used for centralized controller-level exception handling.

```java
@ControllerAdvice
public class GlobalExceptionHandler {
}
```

Then:

```java
@ExceptionHandler(PaymentNotFoundException.class)
public ResponseEntity<?> handle(...) {
    ...
}
```

This avoids repeating:

```java
try {
   ...
} catch (...) {
}
```

in every controller.

---

# 16. `@ControllerAdvice` vs `@RestControllerAdvice`

`@RestControllerAdvice` is effectively:

```java
@ControllerAdvice
@ResponseBody
```

It's commonly used for REST APIs.

Example:

```java
@RestControllerAdvice
class GlobalExceptionHandler {

    @ExceptionHandler(PaymentNotFoundException.class)
    ResponseEntity<ErrorResponse> handle(
            PaymentNotFoundException ex) {

        return ResponseEntity
                .status(HttpStatus.NOT_FOUND)
                .body(new ErrorResponse(ex.getMessage()));
    }
}
```

---

# 17. What is `@ExceptionHandler`?

It tells Spring which method handles a particular exception.

```java
@ExceptionHandler(UserNotFoundException.class)
public ResponseEntity<ErrorResponse> handle(
        UserNotFoundException ex) {
    ...
}
```

You can handle multiple exceptions too:

```java
@ExceptionHandler({
    IllegalArgumentException.class,
    IllegalStateException.class
})
```

---

# 18. How would you design a global error response?

Instead of returning random messages:

```json
{
  "message": "Something went wrong"
}
```

you can define a consistent structure:

```json
{
  "timestamp": "...",
  "status": 400,
  "error": "VALIDATION_ERROR",
  "message": "Invalid request",
  "path": "/payments",
  "traceId": "abc-123"
}
```

This becomes useful for frontend clients and debugging distributed systems.

---

# 19. How do you implement pagination?

Example:

```text
GET /payments?page=0&size=20
```

Spring Data:

```java
Page<Payment> findAll(Pageable pageable);
```

Controller:

```java
@GetMapping("/payments")
public Page<PaymentResponse> getPayments(
        Pageable pageable) {
    ...
}
```

But for large datasets, pagination strategy matters.

### Offset pagination

```text
?page=100&size=20
```

Database may need to skip many rows.

### Keyset/cursor pagination

```text
?lastId=5000&limit=20
```

Conceptually:

```sql
WHERE id > :lastId
ORDER BY id
LIMIT 20
```

For very large datasets, keyset pagination can be much more efficient.

🔥 This connects directly to the JPA streaming/pagination question you were asked recently.

---

# 20. What is API versioning?

You might have:

```text
/api/v1/payments
/api/v2/payments
```

Common strategies:

### URI versioning

```text
/api/v1/payments
```

### Header versioning

```text
X-API-Version: 2
```

### Media-type versioning

```text
Accept: application/vnd.company.v2+json
```

URI versioning is simple and commonly understood, but the appropriate approach depends on the API's compatibility/versioning strategy.

---

# 21. PUT vs PATCH

🔥 Common.

### PUT

Usually represents replacement/update of the resource representation.

```text
PUT /users/123
```

Often expected to be idempotent.

### PATCH

Partial modification.

```text
PATCH /users/123
```

Example:

```json
{
  "email": "new@example.com"
}
```

Only email changes.

---

# 22. What does idempotent mean?

An operation is idempotent if repeating the same request has the same intended effect as making it once.

Examples typically considered idempotent:

```text
GET
PUT
DELETE
```

POST is generally **not inherently idempotent**.

But you can make POST-based operations idempotent using an **idempotency key**.

Example:

```http
Idempotency-Key: abc123
```

Server:

```text
Request
 ↓
Check key
 ↓
Already processed?
 ├── Yes → return previous result
 └── No  → process + store result
```

🔥 This is especially important for payment/order APIs.

---

# 23. How would you prevent duplicate payment requests?

Good senior interview question.

Possible design:

```text
Client
  ↓
Idempotency-Key
  ↓
API
  ↓
Database
```

Store:

```text
idempotency_key
request_hash
status
response
```

Create a unique constraint on the idempotency key.

Then even if:

```text
Request 1
Request 2
```

arrive concurrently, the database uniqueness constraint helps prevent duplicate processing.

You can combine this with transactions and appropriate locking/atomic operations.

---

# 24. What is CORS?

**Cross-Origin Resource Sharing.**

Browser security prevents certain cross-origin requests unless the server allows them.

Example:

```text
Frontend:
https://app.example.com

Backend:
https://api.example.com
```

Different origins can require CORS configuration.

Spring can configure allowed:

* Origins
* Methods
* Headers
* Credentials

---

# 25. What is CSRF?

Cross-Site Request Forgery.

An attacker attempts to make a user's browser send an unwanted authenticated request to your application.

CSRF protection is particularly relevant to browser-based applications using cookies for authentication.

For stateless APIs using an Authorization header with bearer tokens, the CSRF threat model is different, and disabling CSRF can be appropriate depending on how authentication is implemented.

Don't simply say:

> "REST APIs don't need CSRF."

The authentication mechanism matters.

---

# 26. How would you make a REST API secure?

High-level answer:

```text
HTTPS
 ↓
Authentication
 ↓
Authorization
 ↓
Input validation
 ↓
Rate limiting
 ↓
Secure headers
 ↓
Avoid sensitive data exposure
 ↓
Logging/auditing
 ↓
Proper error handling
```

Spring Security will cover this in a dedicated section.

---

# 27. What happens when a request reaches a Spring REST controller?

High-level:

```text
Client
  ↓
HTTP request
  ↓
Embedded server
  ↓
Servlet Filter chain
  ↓
DispatcherServlet
  ↓
HandlerMapping
  ↓
Controller method
  ↓
Service
  ↓
Repository
  ↓
Response
```

This is a **very useful architecture to remember**.

For example:

```text
POST /payments
       ↓
Security filters
       ↓
DispatcherServlet
       ↓
PaymentController
       ↓
PaymentService
       ↓
PaymentRepository
       ↓
Database
```

---

# 28. What is DispatcherServlet?

🔥 Common Spring MVC question.

> `DispatcherServlet` is the front controller of Spring MVC. It receives incoming HTTP requests and delegates them to the appropriate controller/handler.

Conceptually:

```text
HTTP Request
     ↓
DispatcherServlet
     ↓
Find appropriate controller
     ↓
Invoke controller
     ↓
Return response
```

---

# 29. What is `HandlerMapping`?

It determines which controller/handler should process a request.

Example:

```text
GET /payments/123
        ↓
HandlerMapping
        ↓
PaymentController.getPayment()
```

You don't normally configure this manually in modern Spring Boot; Spring MVC sets it up.

---

# 30. What is `HttpMessageConverter`?

🔥 Good follow-up after `@RequestBody`.

It converts HTTP request/response bodies between formats and Java objects.

For JSON:

```text
Request:
JSON
 ↓
Jackson HttpMessageConverter
 ↓
Java DTO
```

Response:

```text
Java DTO
 ↓
Jackson
 ↓
JSON
```

---

# ⚡ REST rapid-fire

### `@RequestParam` required by default?

Yes, unless configured otherwise.

```java
@RequestParam(required = false)
```

---

### `@PathVariable` example?

```text
/users/{id}
```

```java
@PathVariable Long id
```

---

### Can GET have a request body?

HTTP doesn't universally forbid a GET body, but it's not a good interoperable API design and many clients/proxies don't handle it consistently. Prefer query/path parameters.

---

### Should passwords be returned in DTOs?

**No.** Sensitive fields should not be exposed in API responses.

---

### Should you return entities directly?

Usually avoid it; DTOs give better API boundaries.

---

### What status should creation normally return?

`201 Created`.

---

### What if an asynchronous job is accepted?

Often:

`202 Accepted`.

For example:

```text
POST /large-file-processing
        ↓
202 Accepted
        ↓
jobId returned
```

Then:

```text
GET /jobs/{jobId}
```

can provide status.

---

# 🎯 Spring REST COMPLETE

Your request flow should now be clear:

```text
HTTP Request
     ↓
Security Filters
     ↓
DispatcherServlet
     ↓
HandlerMapping
     ↓
Controller
     ↓
DTO + Validation
     ↓
Service
     ↓
Repository
     ↓
Database
     ↓
Entity
     ↓
DTO
     ↓
HttpMessageConverter
     ↓
HTTP Response
```

### Next: 🔥 Spring Data JPA + Hibernate

This is going to be one of the **largest/highest-value sections** for your interview. We'll quickly cover:

* Entity lifecycle
* Persistence Context
* `EntityManager`
* `save()` / `persist()` / `merge()`
* First-level cache
* Dirty checking
* Lazy vs Eager
* N+1
* `JOIN FETCH`
* EntityGraph
* Transactions
* `@Transactional`
* Propagation
* Isolation
* Optimistic/pessimistic locking
* Cascades
* `orphanRemoval`
* JPA pagination/streaming
* Hibernate performance
* Common interview traps
