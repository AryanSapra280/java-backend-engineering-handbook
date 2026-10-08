# Spring Boot — REST Application: Controllers, HTTP & Request Mapping ⭐⭐⭐⭐⭐

Now we move into the **REST Application** section.

For EPAM, don't stop at knowing `@GetMapping` and `@PostMapping`. You should be able to explain **how HTTP maps to Spring MVC, how parameters are bound, how responses are produced, which status code to return, and what happens internally.**

---

# 1. What is REST?

### Interview Question

**What is REST?**

REST stands for **Representational State Transfer**.

It is an architectural style for designing networked applications around **resources**.

For example, in a payment system:

```text
/payment
/payments
/payments/{id}
/payments/{id}/refunds
```

The important idea is that the URI represents a resource, while the HTTP method represents the operation.

```text
GET       → retrieve
POST      → create
PUT       → replace/update
PATCH     → partial update
DELETE    → delete
```

For example:

```http
GET /payments/123
```

means:

> Retrieve payment `123`.

---

# 2. What is a REST Controller?

### Interview Question

**What is `@RestController`?**

`@RestController` is effectively:

```java
@Controller
@ResponseBody
```

combined.

Example:

```java
@RestController
@RequestMapping("/payments")
public class PaymentController {

    @GetMapping("/{id}")
    public PaymentResponse getPayment(
            @PathVariable Long id) {

        return paymentService.getPayment(id);
    }
}
```

The returned object is written to the HTTP response body, normally as JSON through an appropriate `HttpMessageConverter`.

---

# 3. `@Controller` vs `@RestController`

### `@Controller`

Traditionally used for MVC controllers that may return a **view name**.

```java
@Controller
public class PaymentController {

    @GetMapping("/payments")
    public String payments() {
        return "payments";
    }
}
```

Spring may interpret `"payments"` as a view name.

### `@RestController`

Used primarily for REST APIs.

```java
@RestController
public class PaymentController {

    @GetMapping("/payments")
    public List<PaymentResponse> payments() {
        return service.getPayments();
    }
}
```

The returned object becomes the response body.

### Interview answer

> "`@RestController` is a convenience annotation combining `@Controller` and `@ResponseBody`, so handler return values are written directly to the HTTP response body."

---

# 4. `@RequestMapping`

`@RequestMapping` defines request mapping information.

Example:

```java
@RestController
@RequestMapping("/payments")
public class PaymentController {
}
```

Now every endpoint begins with:

```text
/payments
```

Then:

```java
@GetMapping("/{id}")
```

produces:

```text
GET /payments/{id}
```

For:

```text
GET /payments/123
```

Spring can bind:

```java
@PathVariable Long id
```

to:

```text
123
```

---

# 5. HTTP Method Mappings

Spring provides specialized annotations:

```java
@GetMapping
@PostMapping
@PutMapping
@PatchMapping
@DeleteMapping
```

These are more readable than writing:

```java
@RequestMapping(
    method = RequestMethod.GET
)
```

Example:

```java
@GetMapping("/{id}")
public PaymentResponse get(@PathVariable Long id) {
    ...
}
```

---

# 6. GET

Use GET to retrieve a resource.

```http
GET /payments/123
```

Example:

```java
@GetMapping("/{id}")
public PaymentResponse getPayment(
        @PathVariable Long id) {

    return paymentService.getPayment(id);
}
```

A GET request generally should not modify server state.

---

# 7. POST

POST is commonly used to create a resource or perform an operation that isn't naturally modeled as simple replacement.

```http
POST /payments
```

Body:

```json
{
  "amount": 1000,
  "currency": "INR"
}
```

Spring:

```java
@PostMapping
public ResponseEntity<PaymentResponse> create(
        @RequestBody PaymentRequest request) {

    PaymentResponse response =
            paymentService.createPayment(request);

    return ResponseEntity
            .status(HttpStatus.CREATED)
            .body(response);
}
```

---

# 8. PUT

PUT generally represents **replacement** of a resource.

```http
PUT /payments/123
```

For example:

```json
{
  "amount": 1500,
  "currency": "INR",
  "description": "Updated payment"
}
```

Conceptually:

```text
Existing resource
        ↓
Replace with supplied representation
```

A key property of PUT is that it is intended to be **idempotent**.

---

# 9. PATCH

PATCH is generally used for a **partial modification**.

Example:

```http
PATCH /payments/123
```

Body:

```json
{
  "status": "CANCELLED"
}
```

Unlike PUT, the client isn't necessarily sending the complete representation.

---

# 10. DELETE

DELETE removes or logically removes a resource.

```http
DELETE /payments/123
```

Example:

```java
@DeleteMapping("/{id}")
@ResponseStatus(HttpStatus.NO_CONTENT)
public void deletePayment(
        @PathVariable Long id) {

    paymentService.deletePayment(id);
}
```

A successful deletion commonly returns:

```text
204 No Content
```

---

# 11. Path Variables

Suppose:

```http
GET /payments/123
```

Controller:

```java
@GetMapping("/{id}")
public PaymentResponse getPayment(
        @PathVariable Long id) {

    return paymentService.getPayment(id);
}
```

Spring extracts:

```text
123
```

and converts it to:

```java
Long
```

---

# 12. Explicit `@PathVariable` Name

You can write:

```java
@GetMapping("/{id}")
public PaymentResponse getPayment(
        @PathVariable("id") Long paymentId) {
    ...
}
```

Here:

```text
URL variable = id
Java variable = paymentId
```

This is useful when names differ.

---

# 13. Multiple Path Variables

Example:

```http
GET /users/10/payments/123
```

Controller:

```java
@GetMapping("/users/{userId}/payments/{paymentId}")
public PaymentResponse getPayment(
        @PathVariable Long userId,
        @PathVariable Long paymentId) {

    return paymentService.getPayment(userId, paymentId);
}
```

Spring binds both variables.

---

# 14. Query Parameters

Suppose:

```http
GET /payments?status=SUCCESS&page=0&size=20
```

Use:

```java
@GetMapping
public Page<PaymentResponse> getPayments(
        @RequestParam String status,
        @RequestParam int page,
        @RequestParam int size) {

    ...
}
```

Now:

```text
status → SUCCESS
page   → 0
size   → 20
```

---

# 15. Optional Query Parameter

Suppose:

```http
GET /payments
```

may or may not contain:

```text
status
```

Use:

```java
@RequestParam(required = false)
String status
```

Example:

```java
@GetMapping
public List<PaymentResponse> getPayments(
        @RequestParam(required = false)
        String status) {

    ...
}
```

If omitted:

```text
status = null
```

---

# 16. Default Query Parameter

You can provide a default:

```java
@RequestParam(
    defaultValue = "0"
) int page
```

For:

```http
GET /payments
```

the value becomes:

```text
page = 0
```

For:

```http
GET /payments?page=3
```

it becomes:

```text
page = 3
```

---

# 17. Path Variable vs Query Parameter

This is a common interview question.

### Path variable

Use when identifying a specific resource.

```text
GET /payments/123
```

Here:

```text
123 = payment identity
```

### Query parameter

Use for filtering, searching, sorting, pagination, etc.

```text
GET /payments?status=SUCCESS
```

or:

```text
GET /payments?page=0&size=20
```

Mental model:

```text
Path
 ↓
Which resource?

Query parameters
 ↓
How do I filter/shape the collection?
```

---

# 18. Request Body

For POST/PUT/PATCH, the client often sends JSON.

Example:

```json
{
  "amount": 1000,
  "currency": "INR"
}
```

Controller:

```java
@PostMapping
public PaymentResponse create(
        @RequestBody PaymentRequest request) {

    ...
}
```

Spring uses an `HttpMessageConverter` to deserialize JSON into the Java object.

Typically Jackson handles JSON.

Conceptually:

```text
JSON
 ↓
Jackson
 ↓
PaymentRequest
```

---

# 19. What happens internally with `@RequestBody`?

This is a good senior-level question.

Suppose:

```java
@RequestBody PaymentRequest request
```

The flow is approximately:

```text
HTTP Request
      ↓
DispatcherServlet
      ↓
HandlerMapping
      ↓
Controller method
      ↓
Argument Resolver
      ↓
HttpMessageConverter
      ↓
Jackson
      ↓
PaymentRequest
```

Spring reads the HTTP body and uses the appropriate message converter based on the request's content type.

---

# 20. JSON Response

Suppose your controller returns:

```java
PaymentResponse
```

Spring needs to convert it into JSON.

Conceptually:

```text
PaymentResponse
      ↓
HttpMessageConverter
      ↓
Jackson
      ↓
JSON
      ↓
HTTP Response
```

For example:

```java
@GetMapping("/{id}")
public PaymentResponse get(
        @PathVariable Long id) {

    return service.getPayment(id);
}
```

Response:

```json
{
  "paymentId": "123",
  "amount": 1000,
  "status": "SUCCESS"
}
```

---

# 21. What is `HttpMessageConverter`?

### Interview Question

**What is an `HttpMessageConverter`?**

It converts HTTP request/response bodies between Java objects and representations such as JSON.

For example:

```text
Request:

JSON
 ↓
HttpMessageConverter
 ↓
Java Object
```

and:

```text
Response:

Java Object
 ↓
HttpMessageConverter
 ↓
JSON
```

Jackson's JSON converter is commonly used in Spring Boot REST applications.

---

# 22. Content-Type

Suppose the client sends:

```http
Content-Type: application/json
```

This tells the server:

> The request body is JSON.

Spring can then select an appropriate converter.

Example:

```http
POST /payments
Content-Type: application/json
```

Body:

```json
{
  "amount": 1000
}
```

---

# 23. Accept Header

`Accept` tells the server what response representation the client prefers.

Example:

```http
Accept: application/json
```

Conceptually:

```text
Content-Type
→ What am I sending?

Accept
→ What do I want back?
```

This is part of **content negotiation**.

---

# 24. `produces`

You can constrain the response media type:

```java
@GetMapping(
    value = "/{id}",
    produces = MediaType.APPLICATION_JSON_VALUE
)
public PaymentResponse get(...) {
    ...
}
```

---

# 25. `consumes`

You can specify what request content type an endpoint accepts:

```java
@PostMapping(
    consumes = MediaType.APPLICATION_JSON_VALUE
)
public PaymentResponse create(
        @RequestBody PaymentRequest request) {
    ...
}
```

Now the endpoint explicitly expects JSON.

---

# 26. HTTP Status Codes

You should know the major categories.

```text
1xx → Informational

2xx → Success

3xx → Redirection

4xx → Client error

5xx → Server error
```

For REST interviews, focus heavily on:

```text
200
201
202
204
400
401
403
404
409
422
429
500
502
503
504
```

---

# 27. 200 OK

Used when the request succeeds and the server returns a response.

Example:

```http
GET /payments/123
```

Response:

```http
200 OK
```

with:

```json
{
  "paymentId": "123",
  "status": "SUCCESS"
}
```

---

# 28. 201 Created

Used when a resource has been successfully created.

Example:

```http
POST /payments
```

Response:

```http
201 Created
```

Ideally the response can also communicate the location of the new resource.

For example:

```http
Location: /payments/123
```

---

# 29. 202 Accepted

This is particularly important for **asynchronous processing**.

Suppose:

```http
POST /payments
```

starts a long-running operation:

```text
Request
  ↓
Create payment job
  ↓
Kafka
  ↓
Worker
  ↓
Actual processing
```

The server may respond:

```http
202 Accepted
```

meaning:

> The request has been accepted for processing, but processing has not necessarily completed.

This is different from:

```text
201 Created
```

---

# 30. 204 No Content

Used when the operation succeeded but there is no response body.

Common example:

```http
DELETE /payments/123
```

Response:

```http
204 No Content
```

---

# 31. 400 Bad Request

Used when the request is invalid.

Examples:

```text
Malformed JSON
Invalid request structure
Invalid parameter format
```

For example:

```json
{
  "amount": "not-a-number"
}
```

could result in:

```http
400 Bad Request
```

---

# 32. 401 Unauthorized

This status generally means:

> The request lacks valid authentication credentials.

For example:

```http
GET /payments
Authorization: Bearer invalid-token
```

Potential response:

```http
401 Unauthorized
```

Remember:

```text
401 → Authentication problem
403 → Authorization problem
```

---

# 33. 403 Forbidden

The user is authenticated but doesn't have permission.

Example:

```text
User authenticated
        ↓
RBAC check
        ↓
User does not have PAYMENT_ADMIN
```

Response:

```http
403 Forbidden
```

This distinction is extremely important for your upcoming Spring Security section.

---

# 34. 404 Not Found

Resource doesn't exist.

Example:

```http
GET /payments/999999
```

If payment doesn't exist:

```http
404 Not Found
```

Your service might throw:

```java
throw new PaymentNotFoundException(...);
```

and the global exception handler maps it to `404`.

---

# 35. 409 Conflict

Very useful for payment/idempotency systems.

Suppose:

```text
Payment already exists
```

or:

```text
Duplicate resource
```

You may return:

```http
409 Conflict
```

Example:

```text
POST /payments
Idempotency-Key: abc123
```

If the request conflicts with existing state, `409` can be appropriate depending on the API contract.

---

# 36. 422 Unprocessable Content

This can be used when the request is syntactically valid but semantically unacceptable.

For example:

```json
{
  "amount": -1000
}
```

The JSON is valid.

But the business semantics are invalid.

Whether your API uses:

```text
400
```

or:

```text
422
```

should be consistent with your API contract.

Don't claim that 422 is mandatory for all validation errors.

---

# 37. 429 Too Many Requests

Used when the client exceeds a rate limit.

Example:

```text
Client
  ↓
1000 requests/sec
  ↓
Rate limiter
  ↓
Limit exceeded
```

Response:

```http
429 Too Many Requests
```

This becomes important when we cover production REST concerns.

---

# 38. 500 Internal Server Error

Unexpected server-side failure.

Example:

```text
NullPointerException
Unexpected infrastructure failure
Unhandled application exception
```

Don't expose:

```text
java.lang.NullPointerException
at com.company....
```

to the client.

Instead return a safe error response and log the detailed exception internally.

---

# 39. 502 Bad Gateway

Typically means a gateway/proxy received an invalid response from an upstream service.

Example:

```text
Client
  ↓
API Gateway
  ↓
Payment Service
```

If the upstream behaves unexpectedly, gateway infrastructure may return:

```http
502 Bad Gateway
```

---

# 40. 503 Service Unavailable

Typically means the service is temporarily unable to handle requests.

Examples:

```text
Service overloaded
Instance unavailable
Maintenance
Dependency/service availability issue
```

---

# 41. 504 Gateway Timeout

A gateway/proxy waited too long for an upstream service.

Example:

```text
Client
  ↓
Gateway
  ↓
Payment Service
       ↓
    DB/API
       ↓
    Very slow
```

Gateway timeout:

```http
504 Gateway Timeout
```

---

# 42. `ResponseEntity`

### Interview Question

**Why use `ResponseEntity`?**

`ResponseEntity` gives explicit control over:

- status code
- headers
- response body

Example:

```java
return ResponseEntity
        .status(HttpStatus.CREATED)
        .header("X-Payment-Version", "v1")
        .body(response);
```

Without it, Spring can infer a default response status based on the handler.

---

# 43. `@ResponseStatus`

You can also specify:

```java
@ResponseStatus(HttpStatus.CREATED)
@PostMapping
public PaymentResponse create(...) {
    ...
}
```

This is convenient when the status is fixed.

`ResponseEntity` is more flexible when status/headers vary.

---

# 44. `ResponseEntity` vs `@ResponseStatus`

### `@ResponseStatus`

Good for simple fixed status:

```java
@ResponseStatus(HttpStatus.NO_CONTENT)
```

### `ResponseEntity`

Better when you need dynamic:

```text
status
headers
body
```

Example:

```java
ResponseEntity
    .status(...)
    .headers(...)
    .body(...);
```

---

# 45. REST API Design — Resource Naming

Prefer nouns representing resources:

```text
GET    /payments
GET    /payments/123
POST   /payments
PUT    /payments/123
PATCH  /payments/123
DELETE /payments/123
```

Avoid RPC-style URLs where possible:

```text
/getPayment
/createPayment
/deletePayment
```

The HTTP method already communicates the operation.

---

# 46. Nested Resources

Suppose payments belong to users.

You might have:

```text
GET /users/123/payments
```

or:

```text
GET /payments?userId=123
```

Which one you choose depends on the domain and API semantics.

Don't blindly create deep nesting:

```text
/users/1/accounts/2/payments/3/refunds/4
```

because excessively deep resource paths become difficult to use.

---

# 47. Pagination

A collection endpoint should not return millions of records.

Bad:

```http
GET /payments
```

returning:

```text
10 million payments
```

Instead:

```http
GET /payments?page=0&size=20
```

or a cursor-based API.

For Spring Data:

```java
@GetMapping
public Page<PaymentResponse> getPayments(
        Pageable pageable) {

    return paymentService.getPayments(pageable);
}
```

Pagination is a major interview topic and we'll cover its implementation and **offset vs keyset/cursor pagination** in the JPA section.

---

# 48. Sorting

Example:

```http
GET /payments?sort=createdAt,desc
```

Spring can bind this to:

```java
Pageable
```

Conceptually:

```text
page=0
size=20
sort=createdAt,desc
```

---

# 49. Filtering

Example:

```http
GET /payments?status=SUCCESS
```

Multiple filters:

```http
GET /payments?status=SUCCESS&currency=INR
```

This is generally better than creating endpoints like:

```text
/getSuccessfulINRPayments
```

---

# 50. Searching vs Filtering

A useful distinction:

### Filtering

Exact/structured conditions:

```text
status=SUCCESS
currency=INR
```

### Searching

Text-based matching:

```text
?q=aryan
```

The exact API design depends on the domain and search requirements.

---

# 51. API Versioning

A common approach:

```text
/api/v1/payments
/api/v2/payments
```

This allows controlled evolution of the API.

Example:

```text
v1
 ↓
old response structure

v2
 ↓
new response structure
```

Other versioning strategies include:

```text
Header-based
Content negotiation
Query parameter
```

The important point is **backward compatibility**.

---

# 52. Backward Compatibility

Suppose clients currently consume:

```json
{
  "paymentId": "123",
  "amount": 1000
}
```

You shouldn't casually remove:

```text
amount
```

from the response.

Existing clients may break.

Prefer additive changes when possible:

```json
{
  "paymentId": "123",
  "amount": 1000,
  "currency": "INR"
}
```

API evolution is a production concern, not merely a coding concern.

---

# 53. Senior Interview Scenario

### Interviewer:

> "Your API needs to process a payment that takes 20 seconds. Would you keep the HTTP request open for 20 seconds?"

Not necessarily.

For a long-running operation, I'd consider asynchronous processing:

```text
POST /payments
       ↓
Validate request
       ↓
Create payment/job
       ↓
Publish event
       ↓
Return 202 Accepted
       ↓
Background processing
```

Then expose:

```text
GET /payments/{id}
```

to retrieve status.

For example:

```text
POST /payments
→ 202 Accepted
→ paymentId = P123

GET /payments/P123
→ PROCESSING

GET /payments/P123
→ SUCCESS
```

This avoids tying up request threads unnecessarily.

---

# 54. Senior Interview Scenario

### Interviewer:

> "Why shouldn't you return the Entity directly from your REST API?"

Strong answer:

> "Because the entity represents persistence concerns, while the API represents an external contract. Returning entities couples the API to the database model, can expose internal fields, can introduce lazy-loading problems, and makes independent API evolution harder. I would use request/response DTOs and map between DTOs and entities."

---

# 55. Senior Interview Scenario

### Interviewer:

> "How would you design a create-payment API?"

A strong initial design:

```http
POST /payments
```

Headers:

```http
Authorization: Bearer <token>
Idempotency-Key: abc123
Content-Type: application/json
```

Body:

```json
{
  "amount": 1000,
  "currency": "INR",
  "sourceAccount": "ACC123"
}
```

Response for synchronous creation:

```http
201 Created
Location: /payments/P123
```

or for asynchronous processing:

```http
202 Accepted
```

Then:

```http
GET /payments/P123
```

retrieves status.

This connects directly to your payment-microservice experience.

---

# 56. The REST Request Flow You Should Know

Memorize this:

```text
Client
  ↓
HTTP Request
  ↓
Tomcat
  ↓
DispatcherServlet
  ↓
HandlerMapping
  ↓
Controller
  ↓
Argument Resolvers
  ↓
@RequestBody / @PathVariable / @RequestParam
  ↓
Validation
  ↓
Service
  ↓
Repository
  ↓
Database
  ↓
Service
  ↓
Controller
  ↓
HttpMessageConverter
  ↓
Jackson
  ↓
JSON Response
  ↓
Client
```

This connects today's topic with the **Embedded Server + Spring Boot Lifecycle** section we just completed.

---

# 57. EPAM Must-Know Questions

You should be able to answer these without hesitation:

### Fundamentals

**What is REST?**

→ Resource-oriented architectural style using HTTP semantics.

**`@Controller` vs `@RestController`?**

→ `@RestController = @Controller + @ResponseBody`.

**What is `@RequestMapping`?**

→ Maps HTTP requests to controller handlers.

**`@PathVariable` vs `@RequestParam`?**

→ Resource identity vs filtering/pagination/query information.

**What is `@RequestBody`?**

→ Binds request body to a Java object using message converters.

---

### HTTP

**GET vs POST?**

→ GET retrieves; POST commonly creates or triggers processing.

**PUT vs PATCH?**

→ PUT represents replacement; PATCH represents partial modification.

**401 vs 403?**

→ 401 authentication; 403 authenticated but forbidden.

**201 vs 202?**

→ 201 resource created; 202 request accepted for asynchronous processing.

**400 vs 409?**

→ Invalid request vs conflict with current resource state.

**500 vs 503?**

→ Unexpected server error vs temporarily unavailable service.

---

### Spring MVC

**How does JSON become a Java object?**

```text
HTTP body
 ↓
HttpMessageConverter
 ↓
Jackson
 ↓
Java object
```

**How does a Java object become JSON?**

```text
Java object
 ↓
HttpMessageConverter
 ↓
Jackson
 ↓
JSON
```

**What does DispatcherServlet do?**

→ Front controller that coordinates request processing.

---

### Senior

**How would you design an asynchronous payment API?**

→ `POST /payments` → create/accept request → `202 Accepted` → async processing → status endpoint.

**Why use DTOs?**

→ Protect API boundaries and decouple external contracts from persistence models.

**How do you handle API evolution?**

→ Backward-compatible changes, versioning where necessary, controlled deprecation.

**How do you prevent huge collection responses?**

→ Pagination/cursor-based retrieval, filtering and limits.

**What should a production API error look like?**

→ Consistent structured error containing a safe error code/message and useful tracing information such as correlation ID, without exposing internal stack traces.