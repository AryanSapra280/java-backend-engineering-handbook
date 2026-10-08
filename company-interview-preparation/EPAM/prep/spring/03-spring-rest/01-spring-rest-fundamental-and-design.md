# Spring Web / REST — HTTP Fundamentals & REST API Design ⭐⭐⭐⭐⭐

We are now starting **#3 Spring Web / REST**.

For a Senior Software Engineer interview, don't learn REST as just:

> `@GetMapping`, `@PostMapping`, `@RequestBody`

You need to understand what happens **from the HTTP request arriving at the server until the response goes back**, and how to design APIs that behave correctly under retries, failures, concurrent requests, and version changes.

---

# PART 1 — HTTP FUNDAMENTALS

## 1. What is HTTP?

### Interview Question

**What is HTTP?**

### Interview-ready answer

HTTP is an **application-layer, request-response protocol** used for communication between clients and servers.

A client sends:

```text
HTTP Request
     ↓
Server
     ↓
HTTP Response
```

For example:

```http
GET /payments/123 HTTP/1.1
Host: payment-service
Authorization: Bearer <token>
Accept: application/json
```

Response:

```http
HTTP/1.1 200 OK
Content-Type: application/json

{
  "id": "123",
  "status": "SUCCESS"
}
```

HTTP itself is **stateless**: each request should contain the information required to process it, rather than relying on the server remembering the previous request.

---

# 2. What is inside an HTTP request?

An HTTP request consists conceptually of:

```text
Request
 ├── Method
 ├── URI
 ├── Headers
 ├── Query parameters
 └── Body
```

Example:

```http
POST /payments?source=mobile HTTP/1.1
Host: example.com
Authorization: Bearer xyz
Content-Type: application/json
X-Correlation-ID: abc123

{
    "amount": 500,
    "currency": "INR"
}
```

Here:

```text
POST
        → method

/payments
        → path

source=mobile
        → query parameter

Authorization
Content-Type
X-Correlation-ID
        → headers

{"amount":500,"currency":"INR"}
        → body
```

---

# 3. URI, URL and endpoint

These are often used interchangeably, but interviewers may ask the distinction.

Example:

```text
https://api.example.com/payments/123?expand=details
```

A URI identifies a resource.

A URL is a URI that also specifies how/where to locate that resource.

In practical REST discussions, you'll commonly hear:

> “API endpoint”

meaning a combination of:

```text
HTTP method + path
```

For example:

```text
GET /payments/{paymentId}
```

---

# 4. HTTP methods

The major methods you must know:

```text
GET
POST
PUT
PATCH
DELETE
```

And conceptually:

```text
HEAD
OPTIONS
```

For REST interviews, the most important are the first five.

---

# 5. GET

### Purpose

Retrieve a resource.

```http
GET /payments/123
```

Expected behavior:

```text
Server
  ↓
Retrieve payment
  ↓
Return representation
```

GET should generally **not modify server state**.

Example:

```java
@GetMapping("/payments/{id}")
public PaymentResponse getPayment(
        @PathVariable String id) {

    return paymentService.getPayment(id);
}
```

---

# 6. POST

### Purpose

Usually used to create a new resource or perform an operation that doesn't fit naturally into another method.

Example:

```http
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

```http
201 Created
```

Potentially:

```http
Location: /payments/123
```

---

# 7. PUT

### Purpose

Usually represents **replacement of a resource** at a known URI.

Example:

```http
PUT /users/123
```

Request:

```json
{
  "name": "Aryan",
  "email": "aryan@example.com"
}
```

Conceptually:

```text
Existing User
      ↓
Replace representation
      ↓
Updated User
```

PUT is generally expected to be **idempotent**.

We'll discuss idempotency in depth shortly.

---

# 8. PATCH

### Purpose

Partial modification of a resource.

Example:

```http
PATCH /users/123
```

Body:

```json
{
  "email": "new@example.com"
}
```

Only the email is changed.

Compared with:

```http
PUT /users/123
```

which conventionally represents replacement of the resource representation.

---

# 9. DELETE

Deletes a resource.

```http
DELETE /payments/123
```

Potential response:

```http
204 No Content
```

Again, DELETE is generally expected to be idempotent.

---

# 10. PUT vs PATCH — common interview question

### Question

**What is the difference between PUT and PATCH?**

### Strong answer

> PUT generally represents replacement of a resource at a specific URI, while PATCH represents a partial modification of an existing resource.

Example:

Current resource:

```json
{
  "name": "Aryan",
  "email": "a@example.com",
  "phone": "12345"
}
```

PUT:

```http
PUT /users/1
```

```json
{
  "name": "Aryan",
  "email": "new@example.com",
  "phone": "99999"
}
```

PATCH:

```http
PATCH /users/1
```

```json
{
  "phone": "99999"
}
```

---

# 11. What is idempotency?

⭐⭐⭐⭐⭐ VERY IMPORTANT

### Interview Question

**What does idempotent mean in REST?**

An operation is idempotent if making the same request multiple times has the **same intended final server state** as making it once.

For example:

```text
PUT /users/123
```

with:

```json
{
  "name": "Aryan"
}
```

Sending it:

```text
1 time
5 times
100 times
```

should leave the resource in the same final state.

---

# 12. Idempotency does NOT mean same response every time

This is a common trap.

Suppose:

```http
DELETE /users/123
```

First request:

```text
204 No Content
```

Second request:

```text
404 Not Found
```

The responses aren't identical.

But the operation can still be considered idempotent because the intended final state is:

```text
User 123 does not exist
```

So:

> Idempotent ≠ identical response.

It means repeated execution does not keep changing the intended resource state.

---

# 13. Which HTTP methods are idempotent?

Common classification:

| Method | Idempotent? | Typical purpose |
|---|---|---|
| GET | Yes | Read |
| PUT | Yes | Replace |
| DELETE | Yes | Delete |
| POST | Generally no | Create/action |
| PATCH | Not inherently guaranteed | Partial update |

Important:

**PATCH is not automatically idempotent or non-idempotent.**

It depends on what the particular PATCH operation does.

---

# 14. Why is idempotency important in distributed systems?

Imagine:

```text
Client
  |
  | POST /payments
  |
  v
Payment Service
```

The server processes the payment:

```text
Payment SUCCESS
```

But the response is lost:

```text
Payment Service
       X
      Response lost
       X
      Client
```

The client thinks:

> “Maybe payment failed.”

So it retries.

Now:

```text
POST /payments
```

arrives again.

Without idempotency:

```text
₹1000
   +
₹1000
   =
₹2000 charged
```

That's disastrous.

---

# 15. Idempotency key

For payment APIs, we commonly use an idempotency key.

```http
POST /payments
Idempotency-Key: 8f72abc123
```

Server stores:

```text
idempotencyKey
      ↓
payment result
```

First request:

```text
Key = abc123
Payment created
Result = SUCCESS
```

Retry:

```text
Key = abc123
      ↓
Already processed
      ↓
Return previous result
```

Therefore:

```text
Retry
  ↓
No duplicate payment
```

This is particularly important for the payment microservice you've been working with.

---

# 16. HTTP status codes

You should know the major categories:

```text
1xx → informational
2xx → success
3xx → redirection
4xx → client-side error
5xx → server-side error
```

---

# 17. Important 2xx status codes

### 200 OK

Request succeeded and response contains a representation/result.

```http
GET /users/123
200 OK
```

---

### 201 Created

A resource was successfully created.

```http
POST /payments
201 Created
```

Often accompanied by:

```http
Location: /payments/123
```

---

### 202 Accepted

Request has been accepted for processing but processing hasn't necessarily completed.

Very useful for asynchronous operations.

Example:

```http
POST /reports
202 Accepted
```

Response:

```json
{
  "jobId": "abc123",
  "status": "PROCESSING"
}
```

---

### 204 No Content

Request succeeded but there is no response body.

Common example:

```http
DELETE /users/123
204 No Content
```

---

# 18. Important 4xx status codes

### 400 Bad Request

Request is malformed or invalid.

Example:

```json
{
  "amount": "abc"
}
```

when amount must be numeric.

---

### 401 Unauthorized

Authentication is missing or invalid.

Think:

> “Who are you?”

Example:

```text
No valid JWT
```

---

### 403 Forbidden

The client is authenticated but doesn't have permission.

Think:

> “I know who you are, but you aren't allowed to do this.”

Example:

```text
User authenticated
User lacks ADMIN role
```

---

# 19. 401 vs 403 — MUST KNOW

```text
401 → Authentication problem
403 → Authorization problem
```

Example:

```text
No/invalid token
      ↓
401

Valid token + insufficient permission
      ↓
403
```

This will also connect directly to our upcoming **Spring Security** section.

---

# 20. 404 Not Found

The requested resource doesn't exist.

```http
GET /users/999999
```

if user doesn't exist:

```http
404 Not Found
```

---

# 21. 409 Conflict

Used when the request conflicts with the current state of the resource.

Examples:

```text
Duplicate username
Concurrent update conflict
Resource state prevents operation
```

Example:

```http
POST /users
```

with an already existing unique username could result in:

```http
409 Conflict
```

---

# 22. 422 Unprocessable Content

Often used when:

> The request is syntactically valid, but the supplied data cannot be semantically processed.

For example:

```json
{
  "startDate": "2026-12-01",
  "endDate": "2026-01-01"
}
```

The JSON is valid.

But the business data is invalid.

Some APIs use:

```text
422 Unprocessable Content
```

for such semantic validation failures.

The exact status-code convention should be consistent across the API.

---

# 23. Important 5xx codes

### 500 Internal Server Error

Unexpected server-side failure.

### 502 Bad Gateway

Gateway/proxy received an invalid response from an upstream service.

### 503 Service Unavailable

Service temporarily unavailable.

Could happen because:

```text
Overload
Maintenance
Dependency unavailable
Instance not ready
```

### 504 Gateway Timeout

Gateway didn't receive a timely response from an upstream service.

---

# 24. 500 vs 502 vs 503 vs 504

Very useful microservices interview question.

```text
500
 ↓
Application itself failed unexpectedly

502
 ↓
Gateway received invalid response from upstream

503
 ↓
Service currently unavailable

504
 ↓
Gateway waited too long for upstream
```

Example:

```text
Client
  |
  v
API Gateway
  |
  v
Payment Service
```

If Payment Service crashes internally:

```text
500
```

If gateway gets a malformed upstream response:

```text
502
```

If Payment Service is unavailable:

```text
503
```

If Payment Service takes too long:

```text
504
```

---

# 25. HTTP headers

Headers carry metadata about the request/response.

Examples:

```text
Authorization
Content-Type
Accept
Cache-Control
ETag
If-Match
Location
X-Correlation-ID
```

Example:

```http
Content-Type: application/json
Accept: application/json
Authorization: Bearer xyz
X-Correlation-ID: abc123
```

---

# 26. Content-Type vs Accept

Another common interview question.

### Content-Type

Tells the server:

> “What format is the request body in?”

Example:

```http
Content-Type: application/json
```

Meaning:

```text
Request body = JSON
```

### Accept

Tells the server:

> “What response format can I accept?”

Example:

```http
Accept: application/json
```

Therefore:

```text
Content-Type
    ↓
Request body format

Accept
    ↓
Expected response format
```

---

# 27. Path variable vs query parameter

Example:

```http
GET /users/123
```

`123` is a **path variable**.

In Spring:

```java
@GetMapping("/users/{id}")
public User getUser(
        @PathVariable Long id) {
    ...
}
```

---

Query parameter:

```http
GET /users?page=2&size=20
```

Spring:

```java
@GetMapping("/users")
public List<User> getUsers(
        @RequestParam int page,
        @RequestParam int size) {
    ...
}
```

---

# 28. When should you use path variables?

Use path variables when identifying a specific resource.

```text
/users/123
/orders/456
/payments/789
```

Think:

> “Which resource?”

---

# 29. When should you use query parameters?

Use query parameters for things such as:

```text
Filtering
Sorting
Pagination
Searching
Optional parameters
```

Examples:

```text
/users?page=2&size=20

/users?status=ACTIVE

/users?sort=name,asc

/users?search=aryan
```

Think:

> “How do I want to retrieve/filter the resource?”

---

# 30. REST resource design

Bad:

```text
GET /getUser/123
POST /createUser
POST /deleteUser/123
```

Better:

```text
GET    /users/123
POST   /users
PUT    /users/123
PATCH  /users/123
DELETE /users/123
```

The HTTP method describes the operation.

The URI describes the resource.

---

# 31. Resource-oriented thinking

Suppose we have:

```text
Customer
Orders
Payments
```

A reasonable API:

```text
GET /customers/123
GET /customers/123/orders
GET /orders/456
GET /payments/789
```

Avoid unnecessary verbs in URLs.

Bad:

```text
/getCustomer
/createOrder
/updatePayment
```

Better:

```text
GET /customers/123
POST /orders
PATCH /payments/789
```

---

# 32. Nested resources — don't overdo them

You may have:

```text
/customers/123/orders
```

This is reasonable when the relationship is important.

But avoid deeply nested URLs:

```text
/customers/123/orders/456/items/789/payments/999
```

This becomes difficult to maintain.

Often:

```text
/orders/456
/items/789
/payments/999
```

is cleaner.

---

# 33. REST API pagination

This is a major interview topic and we'll cover it deeply later.

Typical offset pagination:

```http
GET /payments?page=0&size=20
```

Response:

```json
{
  "content": [...],
  "page": 0,
  "size": 20,
  "totalElements": 1000,
  "totalPages": 50
}
```

But for very large datasets, offset pagination can become expensive.

We'll later cover:

```text
Offset pagination
Cursor pagination
Keyset pagination
```

including actual Spring implementation.

---

# 34. Sorting

Example:

```http
GET /payments?sort=createdAt,desc
```

Multiple fields:

```http
GET /payments?sort=status,asc&sort=createdAt,desc
```

Always validate allowed sort fields rather than blindly passing arbitrary user input into database query construction.

---

# 35. Filtering

Example:

```http
GET /payments?status=SUCCESS
```

Multiple filters:

```http
GET /payments?status=SUCCESS&currency=INR
```

For complex search APIs, you may use:

```text
GET /payments?status=SUCCESS&from=2026-01-01&to=2026-10-01
```

---

# 36. Searching

Example:

```http
GET /users?search=aryan
```

The exact semantics should be documented:

```text
searches name?
email?
both?
case-sensitive?
prefix?
contains?
```

A senior engineer shouldn't just create an API without defining these semantics.

---

# 37. API versioning

Suppose your API initially has:

```text
GET /api/v1/payments
```

Later you introduce breaking changes.

You might introduce:

```text
GET /api/v2/payments
```

Common versioning approaches include:

```text
URI versioning
/api/v1/payments

Header versioning
Accept: application/vnd.company.payment.v2+json

Query parameter
/payments?version=2
```

For interviews, know the tradeoffs.

URI versioning is straightforward and highly visible.

---

# 38. What is backward compatibility?

Suppose existing clients expect:

```json
{
  "id": "123",
  "amount": 100
}
```

If you suddenly remove:

```text
amount
```

old clients may break.

Therefore APIs should evolve carefully.

Generally safer changes include:

```text
Adding optional fields
Adding new endpoints
Adding optional request parameters
```

Riskier/breaking changes:

```text
Removing fields
Renaming fields
Changing field types
Changing semantics
Changing required fields
```

---

# 39. REST API error response

Don't return:

```text
500
NullPointerException at PaymentService.java:123
```

to the client.

Instead return a stable error structure.

Example:

```json
{
  "timestamp": "2026-10-05T10:30:00Z",
  "status": 400,
  "code": "INVALID_PAYMENT",
  "message": "Payment amount must be greater than zero",
  "path": "/payments",
  "correlationId": "abc123"
}
```

The internal exception should remain in logs.

---

# 40. Why include correlation ID in error responses?

Suppose client receives:

```json
{
  "code": "PAYMENT_FAILED",
  "correlationId": "abc123"
}
```

The support engineer can search:

```text
correlationId=abc123
```

across logs.

This creates a bridge:

```text
Client error
     |
     | correlationId
     v
Gateway
     |
     v
Payment Service
     |
     v
Database / Kafka / External API
```

Very useful in production.

---

# 41. REST API validation

We already covered validation in Spring Boot, but from an API design perspective:

Separate:

### Structural validation

```text
amount cannot be null
email must be valid
name cannot be blank
```

from:

### Business validation

```text
Payment amount cannot exceed daily limit
Account must be active
Payment currency must be supported
```

Structural validation can happen near the API boundary.

Business validation belongs in the business/service layer.

---

# 42. What happens when a request enters Spring MVC?

This is the bridge to the **next major section**.

Suppose:

```http
POST /payments
Content-Type: application/json
```

The request enters the application.

Conceptually:

```text
Client
  |
  v
Servlet Container
  |
  v
DispatcherServlet
  |
  v
HandlerMapping
  |
  v
Controller
  |
  v
Service
  |
  v
Repository
  |
  v
Response
```

But there are several important components in between.

The complete Spring MVC pipeline is:

```text
HTTP Request
     |
     v
Servlet Container
(Tomcat)
     |
     v
Filters
     |
     v
DispatcherServlet
     |
     v
HandlerMapping
     |
     v
HandlerAdapter
     |
     v
Controller
     |
     v
Service
     |
     v
Return Object
     |
     v
HttpMessageConverter
     |
     v
JSON Response
```

This is one of the **most important Spring interview flows**.

We'll go deeply into it next.

---

# 43. Senior interview scenario

### Question

**A request comes to `POST /payments`. Explain what happens inside Spring Boot.**

### Strong answer

> The request first reaches the embedded servlet container such as Tomcat. Servlet filters can execute before Spring MVC processing. The request then reaches the `DispatcherServlet`, which acts as the front controller. `DispatcherServlet` uses `HandlerMapping` to determine which controller method should handle the request and `HandlerAdapter` to invoke it. Spring resolves controller arguments such as `@PathVariable`, `@RequestParam`, and `@RequestBody`. For a JSON request, an `HttpMessageConverter`, commonly Jackson-based, deserializes the JSON into the Java DTO. The controller calls the service layer, and when the controller returns a response object, an `HttpMessageConverter` serializes it back to JSON. Exception handling through `@ExceptionHandler`/`@ControllerAdvice` can intercept exceptions and convert them into appropriate HTTP responses.

That answer demonstrates that you understand **Spring MVC internals**, not just annotations.

---

# 44. Important mental model for this section

Remember:

```text
                    HTTP
                     |
          +----------+----------+
          |                     |
        Request               Response
          |                     |
          v                     ^
       Filters                  |
          |                     |
          v                     |
  DispatcherServlet             |
          |                     |
          v                     |
    HandlerMapping              |
          |                     |
          v                     |
    HandlerAdapter              |
          |                     |
          v                     |
      Controller                |
          |                     |
          v                     |
       Service                 |
          |                     |
          v                     |
      Repository               |
          |                     |
          +---------------------+
                     |
                     v
              MessageConverter
                     |
                     v
                  JSON
```

---

# 45. What you should be able to answer after this section

You should now be comfortable with:

```text
HTTP
 ├── Methods
 ├── Status codes
 ├── Headers
 ├── Content-Type
 ├── Accept
 ├── Path variables
 ├── Query parameters
 ├── Idempotency
 └── Resource design

REST
 ├── URI design
 ├── CRUD semantics
 ├── Pagination
 ├── Sorting
 ├── Filtering
 ├── Searching
 ├── Versioning
 ├── Error responses
 └── Backward compatibility

Spring MVC
 ├── DispatcherServlet
 ├── HandlerMapping
 ├── HandlerAdapter
 ├── Controllers
 ├── MessageConverters
 └── Jackson
```

**Next:** **Spring MVC internals — DispatcherServlet, HandlerMapping, HandlerAdapter, argument resolvers, Jackson, message converters, Filters vs Interceptors, and the complete request lifecycle.**