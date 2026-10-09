Absolutely. I’ll keep this **EPAM-interview focused**: enough depth to explain internals confidently and handle follow-ups, but I won’t go into framework source-code details that are unlikely to matter.

# Spring Web / REST — Spring MVC Request Lifecycle & Internals ⭐⭐⭐⭐⭐

## 1. What is Spring MVC?

### Interview Question

**What is Spring MVC?**

### Interview-ready answer

Spring MVC is Spring's web framework for building HTTP-based applications, especially REST APIs.

It follows the **Model-View-Controller** architecture, although in REST APIs we typically return JSON rather than rendering server-side views.

The central component is the:

```text
DispatcherServlet
```

It acts as the **front controller** and coordinates request processing.

---

# 2. What is DispatcherServlet?

### Interview Question

**What is DispatcherServlet?**

### Answer

`DispatcherServlet` is the central entry point for Spring MVC requests.

Instead of every controller handling requests directly from the servlet container, requests first come to `DispatcherServlet`, which determines:

- which controller should handle the request
- how method arguments should be resolved
- how the controller should be invoked
- how the response should be converted
- how exceptions should be handled

Think of it as the **traffic controller of Spring MVC**.

```text
HTTP Request
      |
      v
DispatcherServlet
      |
      +---- Which controller?
      |
      +---- How to invoke it?
      |
      +---- How to convert request?
      |
      +---- How to convert response?
      |
      +---- How to handle exception?
```

---

# 3. Complete Spring MVC request flow

This is one of the most important things to understand for interviews.

Suppose we have:

```java
@PostMapping("/payments")
public PaymentResponse createPayment(
        @RequestBody CreatePaymentRequest request) {
    ...
}
```

Client sends:

```http
POST /payments
Content-Type: application/json

{
    "amount": 1000,
    "currency": "INR"
}
```

The simplified flow is:

```text
Client
   |
   v
Tomcat / Servlet Container
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
Controller Method
   |
   v
Service
   |
   v
Response Object
   |
   v
HttpMessageConverter
   |
   v
JSON Response
```

You should be able to explain this flow in an interview.

---

# 4. What happens before DispatcherServlet?

The request first reaches the **Servlet container**, such as embedded Tomcat.

Tomcat receives the HTTP request and invokes the servlet infrastructure.

Servlet filters can execute before the request reaches Spring MVC.

For example:

```text
HTTP Request
     |
     v
Correlation ID Filter
     |
     v
Authentication-related Filter
     |
     v
DispatcherServlet
```

This is why filters are useful for concerns such as:

- request logging
- correlation IDs
- security
- CORS
- request preprocessing

---

# 5. What is HandlerMapping?

### Interview Question

**How does Spring know which controller method should handle `/payments`?**

`HandlerMapping` is responsible for finding the appropriate handler for the incoming request.

For example:

```java
@PostMapping("/payments")
public PaymentResponse createPayment(...) {
}
```

When:

```http
POST /payments
```

arrives, Spring needs to determine:

> Which controller method is mapped to `POST /payments`?

`HandlerMapping` helps make that decision.

Conceptually:

```text
POST /payments
       |
       v
HandlerMapping
       |
       v
PaymentController.createPayment()
```

---

# 6. What is HandlerAdapter?

This is a common follow-up.

### Question

**Once Spring finds the controller method, how is it actually invoked?**

That's where `HandlerAdapter` comes in.

It provides a mechanism for `DispatcherServlet` to invoke the selected handler.

Simplified:

```text
Request
   |
   v
HandlerMapping
   |
   | finds handler
   v
Controller Method
   |
   v
HandlerAdapter invokes it
```

You don't normally interact with `HandlerAdapter` directly in application code.

The important interview distinction is:

```text
HandlerMapping
    ↓
Finds which handler should process request

HandlerAdapter
    ↓
Knows how to invoke that handler
```

---

# 7. HandlerMapping vs HandlerAdapter

### Interview Question

**What's the difference?**

| Component | Responsibility |
|---|---|
| HandlerMapping | Finds the handler |
| HandlerAdapter | Invokes the handler |

Memory trick:

```text
Mapping → "WHO?"
Adapter → "HOW?"
```

---

# 8. How does `@RequestBody` work?

Suppose:

```java
@PostMapping("/payments")
public PaymentResponse createPayment(
        @RequestBody CreatePaymentRequest request) {
    ...
}
```

Client sends JSON:

```json
{
  "amount": 1000,
  "currency": "INR"
}
```

Spring needs to convert:

```text
JSON
 ↓
Java Object
```

This is done through an `HttpMessageConverter`.

For JSON, Spring Boot commonly uses **Jackson**.

Conceptually:

```text
HTTP Request Body
       |
       v
HttpMessageConverter
       |
       v
Jackson
       |
       v
CreatePaymentRequest
```

---

# 9. What is HttpMessageConverter?

### Interview Question

**What is an HttpMessageConverter?**

It is a Spring MVC mechanism for converting HTTP request/response bodies between formats and Java objects.

For example:

```text
Request:

JSON
 ↓
Java object
```

and:

```text
Response:

Java object
 ↓
JSON
```

Commonly:

```text
Jackson
    ↓
JSON ↔ Java Object
```

---

# 10. Request body conversion

Suppose:

```java
public class CreatePaymentRequest {

    private BigDecimal amount;
    private String currency;

    // getters/setters
}
```

Request:

```json
{
    "amount": 1000,
    "currency": "INR"
}
```

Spring effectively performs:

```text
JSON
 ↓
Jackson
 ↓
CreatePaymentRequest
```

Then your controller receives:

```java
CreatePaymentRequest request
```

You don't manually call Jackson in normal controller code.

---

# 11. Response conversion

Now suppose your controller returns:

```java
return new PaymentResponse(
    "P123",
    "SUCCESS"
);
```

Spring needs to convert:

```text
PaymentResponse
      ↓
JSON
```

Again, an `HttpMessageConverter` handles this.

Result:

```json
{
  "paymentId": "P123",
  "status": "SUCCESS"
}
```

So:

```text
@RequestBody
     ↓
HTTP body → Java object

Response body
     ↓
Java object → HTTP body
```

---

# 12. What is Jackson?

### Interview Question

**What is Jackson and where does it fit?**

Jackson is a commonly used Java library for JSON serialization and deserialization.

### Deserialization

```text
JSON → Java object
```

### Serialization

```text
Java object → JSON
```

Example:

```text
Incoming JSON
     ↓
Jackson
     ↓
DTO
```

and:

```text
DTO
 ↓
Jackson
 ↓
JSON response
```

Spring MVC uses Jackson through the appropriate HTTP message converter.

---

# 13. What is `@RequestParam`?

Example:

```http
GET /payments?page=0&size=20
```

Controller:

```java
@GetMapping("/payments")
public List<PaymentResponse> getPayments(
        @RequestParam int page,
        @RequestParam int size) {

    ...
}
```

Here:

```text
?page=0&size=20
```

is the query string.

Spring resolves these values and supplies them to the method parameters.

---

# 14. What is `@PathVariable`?

Example:

```http
GET /payments/P123
```

Controller:

```java
@GetMapping("/payments/{id}")
public PaymentResponse getPayment(
        @PathVariable String id) {

    ...
}
```

Spring extracts:

```text
P123
```

and supplies it to:

```java
String id
```

---

# 15. What is `@RequestHeader`?

Suppose the request contains:

```http
X-Correlation-ID: abc123
```

Controller can access it:

```java
@GetMapping("/payments/{id}")
public PaymentResponse getPayment(
        @PathVariable String id,
        @RequestHeader("X-Correlation-ID")
        String correlationId) {

    ...
}
```

This is useful for headers such as:

```text
Authorization
X-Correlation-ID
If-Match
Accept
```

although security headers are often handled earlier by the security filter chain rather than manually in controllers.

---

# 16. What is argument resolution?

Spring controller methods can accept many types of arguments:

```java
@GetMapping("/payments/{id}")
public PaymentResponse getPayment(
        @PathVariable String id,
        @RequestParam String currency,
        @RequestHeader("X-Correlation-ID")
        String correlationId) {
    ...
}
```

Spring needs to determine where each value comes from.

Conceptually:

```text
@PathVariable
     ↓
Path

@RequestParam
     ↓
Query parameter

@RequestHeader
     ↓
HTTP header

@RequestBody
     ↓
HTTP body
```

Spring MVC has **argument resolvers** that perform this work.

You don't normally need to implement these yourself.

---

# 17. What is HandlerMethodArgumentResolver?

### Interview Question

**How does Spring resolve controller method parameters?**

Spring MVC uses argument resolver mechanisms, including `HandlerMethodArgumentResolver`, to resolve controller method arguments.

For example:

```java
@GetMapping("/payments/{id}")
public PaymentResponse getPayment(
        @PathVariable String id,
        @RequestParam String currency) {
}
```

Spring has logic that resolves:

```text
@PathVariable
@RequestParam
@RequestHeader
...
```

and supplies the appropriate values.

### Interview depth

You generally don't need to memorize the internal resolver class names unless specifically asked.

Know the concept:

> Spring inspects controller method parameters and uses appropriate argument resolvers to obtain values from the HTTP request.

---

# 18. What happens if JSON is invalid?

Suppose the client sends:

```json
{
    "amount": "INVALID"
}
```

but:

```java
private BigDecimal amount;
```

Jackson cannot deserialize it correctly.

The request may fail before your controller method executes.

This is important:

```text
HTTP Request
    ↓
JSON conversion
    ↓
FAILURE
    X
Controller
```

Your business logic doesn't necessarily get executed.

This is one reason global exception handling needs to account for **request parsing/deserialization errors**, not just exceptions thrown inside services.

---

# 19. Filters vs Interceptors ⭐⭐⭐⭐⭐

This is a very common Spring interview question.

### Question

**What's the difference between a Filter and an Interceptor?**

The biggest distinction is where they operate.

### Filter

A Filter belongs to the **Servlet specification/container layer**.

It can operate before Spring MVC processing.

```text
HTTP Request
     ↓
Filter
     ↓
DispatcherServlet
     ↓
Controller
```

### Interceptor

A Spring MVC `HandlerInterceptor` operates around **handler/controller execution**.

```text
HTTP Request
     ↓
DispatcherServlet
     ↓
Interceptor
     ↓
Controller
```

---

# 20. When would you use a Filter?

Good use cases:

```text
Authentication infrastructure
Correlation ID
Request/response logging
CORS
Request preprocessing
Security-related processing
```

Example:

```java
@Component
public class CorrelationIdFilter
        extends OncePerRequestFilter {
    ...
}
```

This is a natural use of a Filter because you want the correlation ID established **before controller processing**.

---

# 21. When would you use an Interceptor?

Good use cases:

```text
Controller-specific logging
Authorization checks around handlers
Request timing
Auditing
Common controller preprocessing
```

Example:

```java
@Component
public class RequestTimingInterceptor
        implements HandlerInterceptor {

    @Override
    public boolean preHandle(...) {
        // before controller
        return true;
    }

    @Override
    public void afterCompletion(...) {
        // after request completion
    }
}
```

---

# 22. Filter vs Interceptor — interview answer

Memorize this:

> “A Filter works at the Servlet/container level and can process requests before they reach Spring MVC. An Interceptor is part of Spring MVC and operates around handler/controller execution. I would use a Filter for cross-cutting concerns such as correlation IDs or security infrastructure, and an Interceptor for concerns specifically related to Spring MVC handler execution.”

---

# 23. `preHandle`, `postHandle`, `afterCompletion`

An interceptor provides lifecycle callbacks.

Simplified:

```text
Request
   |
   v
preHandle()
   |
   v
Controller
   |
   v
postHandle()
   |
   v
Response processing
   |
   v
afterCompletion()
```

### `preHandle()`

Runs before controller execution.

Can return:

```java
true
```

to continue.

Or:

```java
false
```

to stop further processing.

### `postHandle()`

Runs after controller execution, before final response completion.

### `afterCompletion()`

Runs after request processing has completed.

Useful for cleanup/timing-related work.

---

# 24. Which is better: Filter or Interceptor?

Don't answer:

> “Interceptor is better.”

There is no universally better one.

Use based on responsibility.

```text
Servlet-level concern
       ↓
Filter

Spring MVC handler concern
       ↓
Interceptor
```

---

# 25. Important production distinction

Suppose you need to add a correlation ID.

Use:

```text
Filter
```

because you want it established as early as possible.

Suppose you want to measure:

```text
Controller execution time
```

An interceptor can be appropriate.

This distinction demonstrates practical understanding.

---

# 26. Where does Spring Security fit?

This is an important connection.

Spring Security primarily integrates through the **Servlet filter chain**.

Conceptually:

```text
Request
   |
   v
Security Filters
   |
   v
Other Filters
   |
   v
DispatcherServlet
   |
   v
Controller
```

So authentication/authorization often happens **before the request reaches your controller**.

We'll cover the security filter chain in depth later under **Spring Security**.

---

# 27. Filter vs Interceptor vs Controller Advice

Don't confuse these.

```text
Filter
  ↓
Servlet level

Interceptor
  ↓
Spring MVC handler level

@ControllerAdvice
  ↓
Exception handling / controller advice
```

A useful mental model:

```text
HTTP Request
     |
     v
   Filter
     |
     v
DispatcherServlet
     |
     v
 Interceptor
     |
     v
 Controller
     |
     v
Service
```

If an exception occurs and is handled through Spring MVC exception handling:

```text
Controller
     |
     X
 Exception
     |
     v
@ControllerAdvice
     |
     v
HTTP Error Response
```

---

# 28. Common interview scenario

### Question

**You need to add correlation ID to every request. Would you use Filter or Interceptor?**

### Answer

I'd generally use a **Servlet Filter**, because the correlation ID should be established early in the request lifecycle and should be available to downstream Spring MVC processing.

For example:

```text
Request
  ↓
Correlation ID Filter
  ↓
MDC
  ↓
DispatcherServlet
  ↓
Controller
  ↓
Service
```

---

# 29. Common interview scenario

### Question

**You want to log how long each controller method takes. What would you use?**

A Spring MVC interceptor is a reasonable choice because it directly surrounds handler execution.

Conceptually:

```text
preHandle()
    ↓
Controller
    ↓
postHandle()
    ↓
afterCompletion()
```

For broader distributed request timing, however, metrics/tracing instrumentation is generally preferable to building everything manually.

---

# 30. Common interview scenario

### Question

**Can an interceptor handle requests that never reach Spring MVC?**

Generally, no.

A servlet Filter is earlier in the request processing chain.

Therefore:

```text
Filter
   ↓
can see requests before Spring MVC

Interceptor
   ↓
works around Spring MVC handler execution
```

---

# 31. Important request lifecycle — final version

For interview purposes, remember this:

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
                      | finds handler
                      v
                HandlerAdapter
                      |
                      v
             Argument Resolution
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
             Controller Response
                      |
                      v
             HttpMessageConverter
                      |
                      v
                  Jackson
                      |
                      v
                JSON Response
```

The important thing is not memorizing every internal class.

Understand the **responsibility of each layer**.

---

# 32. EPAM-style question

### Interviewer:

> “Explain what happens when a POST request containing JSON reaches your Spring Boot application.”

### Strong answer:

> “The request first reaches the servlet container, such as embedded Tomcat. Servlet filters can process it before Spring MVC. The request then reaches `DispatcherServlet`, which uses `HandlerMapping` to identify the appropriate controller handler and `HandlerAdapter` to invoke it. Spring resolves method arguments such as path variables, query parameters and headers. For `@RequestBody`, an HTTP message converter, commonly backed by Jackson, deserializes JSON into the DTO. The controller calls the service layer and returns a response object. Spring then serializes that object back into JSON through an HTTP message converter.”

That's the **right depth for an interview**.

---

# 33. What NOT to over-study

For your EPAM preparation, don't waste time memorizing:

```text
❌ Every HandlerMapping implementation
❌ Every HandlerAdapter implementation
❌ Spring MVC source code
❌ Internal DispatcherServlet private methods
❌ Every built-in argument resolver class
❌ Framework implementation details that don't affect application design
```

Know:

```text
✅ Request lifecycle
✅ Responsibility of each component
✅ Filters vs Interceptors
✅ Jackson/message conversion
✅ Argument resolution concept
✅ Where security fits
✅ Where exceptions are handled
✅ Practical use cases
```

That's the level most relevant to a Senior Software Engineer interview.

---

# 34. What we've now covered in Spring MVC

```text
Spring MVC
    |
    +-- DispatcherServlet          ✅
    +-- HandlerMapping             ✅
    +-- HandlerAdapter             ✅
    +-- Request lifecycle          ✅
    +-- @RequestBody               ✅
    +-- @RequestParam              ✅
    +-- @PathVariable              ✅
    +-- @RequestHeader             ✅
    +-- Argument resolution        ✅
    +-- HttpMessageConverter       ✅
    +-- Jackson                    ✅
    +-- Filter                     ✅
    +-- Interceptor                ✅
    +-- preHandle                  ✅
    +-- postHandle                 ✅
    +-- afterCompletion            ✅
```

**Next:** **CORS, CSRF, async request processing, file upload/download, and streaming responses** — focusing only on the interview-relevant concepts and practical Spring usage.