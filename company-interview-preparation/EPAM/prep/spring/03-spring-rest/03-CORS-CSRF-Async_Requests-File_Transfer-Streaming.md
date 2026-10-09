# Spring Web / REST — CORS, CSRF, Async Requests, File Transfer & Streaming ⭐⭐⭐⭐⭐

We’ll keep this **interview-focused**. The goal is to understand what these features solve, when to use them, and the practical Spring approach—not memorize framework internals.

---

# 1. CORS ⭐⭐⭐⭐

## Interview Question

**What is CORS?**

CORS stands for **Cross-Origin Resource Sharing**.

It is a browser security mechanism that controls whether a web page from one origin can make requests to a different origin.

An origin is based on:

```text
scheme + host + port
```

For example:

```text
Frontend:
https://app.example.com

Backend:
https://api.example.com
```

These are different origins because the hosts differ.

---

# 2. Why do we need CORS?

Imagine:

```text
Browser
   |
   | JavaScript request
   v
https://api.example.com
```

while the web page was loaded from:

```text
https://frontend.example.com
```

The browser applies same-origin security rules.

The backend can explicitly allow the frontend origin through CORS response headers.

For example:

```http
Access-Control-Allow-Origin: https://frontend.example.com
```

So:

```text
Browser
   |
   | Request from frontend.example.com
   v
API
   |
   | CORS policy
   v
Browser decides whether
JavaScript can access response
```

---

# 3. Is CORS a server-side security mechanism?

This is an important interview nuance.

CORS primarily controls **browser behavior**.

It does not mean:

> “The server refuses all requests from another origin.”

A non-browser client such as:

```text
curl
Postman
another backend service
```

is not subject to browser CORS enforcement.

Therefore:

> CORS should not be treated as authentication or authorization.

Authentication/authorization are separate concerns.

---

# 4. How do we configure CORS in Spring?

You can configure it globally.

For example:

```java
@Configuration
public class CorsConfig {

    @Bean
    public WebMvcConfigurer corsConfigurer() {

        return new WebMvcConfigurer() {

            @Override
            public void addCorsMappings(
                    CorsRegistry registry) {

                registry.addMapping("/api/**")
                        .allowedOrigins(
                            "https://app.example.com")
                        .allowedMethods(
                            "GET", "POST", "PUT", "DELETE")
                        .allowedHeaders("*");
            }
        };
    }
}
```

The exact configuration should reflect the application's security requirements.

Avoid blindly doing:

```java
.allowedOrigins("*")
```

especially for sensitive applications.

---

# 5. What is a CORS preflight request?

This is commonly asked.

For some cross-origin requests, the browser first sends an:

```http
OPTIONS
```

request.

This is called a **preflight request**.

Conceptually:

```text
Browser
   |
   | OPTIONS
   | "Can I make this request?"
   v
Server
   |
   | CORS response
   v
Browser
   |
   | Actual request
   v
Server
```

The server can respond with headers such as:

```http
Access-Control-Allow-Origin: https://app.example.com
Access-Control-Allow-Methods: POST, GET
Access-Control-Allow-Headers: Authorization, Content-Type
```

The browser then decides whether the actual request is allowed.

---

# 6. Why might a POST request trigger OPTIONS?

Because not every cross-origin request is a "simple" request.

For example, a request containing:

```http
Authorization: Bearer ...
Content-Type: application/json
```

may cause the browser to perform a preflight depending on the complete request characteristics.

Interview answer:

> “For non-simple cross-origin requests, the browser may send an OPTIONS preflight request to verify that the server allows the intended method and headers.”

---

# 7. CORS vs CSRF ⭐⭐⭐⭐⭐

This is a very common trap.

They solve **different problems**.

### CORS

Controls:

> Which browser origins are allowed to access resources?

### CSRF

Protects against:

> A malicious website causing a user's browser to perform an unwanted authenticated action against another site.

Think:

```text
CORS
 ↓
Cross-origin browser access

CSRF
 ↓
Unwanted authenticated action
```

---

# 8. What is CSRF?

CSRF = **Cross-Site Request Forgery**.

Suppose a user is logged into:

```text
https://bank.example.com
```

Their browser has an authentication cookie.

The user visits a malicious website:

```text
https://evil.example
```

That website attempts to cause the browser to send:

```http
POST https://bank.example.com/transfer
```

If the bank relies on cookies automatically sent by the browser, the request might carry the user's authentication context.

The attacker is trying to make the server believe:

> “This request came from the legitimate user.”

---

# 9. How does CSRF protection work?

A common approach is a **CSRF token**.

Conceptually:

```text
Server
  |
  | generates token
  v
Browser
  |
  | sends token with state-changing request
  v
Server
  |
  | validates token
  v
Accept / Reject
```

If the malicious site doesn't have the legitimate CSRF token, the request can be rejected.

---

# 10. Does every REST API need CSRF protection?

Not necessarily.

This is where interviewers expect nuance.

If an application uses:

```text
session-based authentication
+
browser cookies
```

CSRF protection is typically important.

If a stateless API uses:

```text
Authorization: Bearer <JWT>
```

and does not rely on browser cookies for authentication, the classic CSRF scenario is different.

However, you should not say:

> “JWT means CSRF is impossible.”

That's too absolute.

The actual risk depends on **where the token is stored and how authentication is transmitted**.

---

# 11. CORS vs CSRF — interview-ready answer

> “CORS and CSRF address different concerns. CORS controls whether browser JavaScript from one origin can access resources on another origin. CSRF protects authenticated users from unauthorized state-changing requests, particularly in cookie/session-based authentication. CORS isn't a replacement for CSRF protection.”

That's the answer to remember.

---

# 12. Async request processing

Now let's discuss asynchronous HTTP processing.

Suppose:

```text
Client
   |
   | GET /report
   v
Spring Boot
   |
   | expensive operation
   |
   | 10 seconds
   v
Response
```

A traditional synchronous request keeps the request-processing thread occupied while the operation executes.

For some workloads, that's undesirable.

Spring MVC provides mechanisms for asynchronous request processing.

---

# 13. Why use asynchronous request processing?

Imagine:

```text
1000 incoming requests
```

and each request waits:

```text
5 seconds
```

If worker threads remain blocked waiting for slow operations, the application can exhaust its request-processing capacity.

Async processing can allow the servlet request lifecycle to be handled differently so that the container thread doesn't have to remain occupied for the entire duration.

But don't confuse this with:

> “Async automatically makes the operation faster.”

It doesn't.

It mainly helps with **thread/resource utilization and request handling for suitable workloads**.

---

# 14. `CompletableFuture` with Spring MVC

You may see:

```java
@GetMapping("/reports/{id}")
public CompletableFuture<ReportResponse> getReport(
        @PathVariable String id) {

    return CompletableFuture.supplyAsync(
        () -> reportService.generate(id),
        executor
    );
}
```

The operation executes asynchronously.

However, for production applications:

**Don't casually use `CompletableFuture.supplyAsync()` with the common ForkJoinPool.**

Use an appropriately configured executor when you need controlled concurrency.

For example:

```java
@Bean
public ExecutorService reportExecutor() {
    return Executors.newFixedThreadPool(10);
}
```

Or use Spring's managed executor infrastructure.

---

# 15. Important async interview trap

### Question

**Does asynchronous processing make a slow database query faster?**

No.

If the DB query takes:

```text
5 seconds
```

async processing doesn't magically reduce it to:

```text
100 ms
```

It changes how application threads/resources are used while waiting.

You still need to optimize:

```text
DB query
indexes
connection pool
network
external dependency
```

if latency itself is the problem.

---

# 16. `Callable`

Spring MVC supports asynchronous return values such as:

```java
@GetMapping("/report")
public Callable<ReportResponse> report() {

    return () -> reportService.generate();
}
```

Conceptually:

```text
Request thread
      |
      v
Async processing
      |
      v
Worker execution
      |
      v
Response
```

You don't need to memorize the internal servlet mechanics for the interview.

Know:

> `Callable` allows controller processing to be performed asynchronously rather than executing the complete operation directly on the initial servlet thread.

---

# 17. `DeferredResult`

`DeferredResult` is useful when the result will be produced later.

For example:

```java
@GetMapping("/long-operation")
public DeferredResult<Response> operation() {

    DeferredResult<Response> result =
            new DeferredResult<>(30_000L);

    executor.submit(() -> {

        Response response =
                service.performOperation();

        result.setResult(response);
    });

    return result;
}
```

Conceptually:

```text
HTTP Request
     |
     v
Return DeferredResult
     |
     v
Request processing can continue asynchronously
     |
     v
Result produced later
     |
     v
HTTP Response
```

---

# 18. `Callable` vs `DeferredResult`

You don't need an exhaustive framework comparison.

Remember the conceptual difference:

```text
Callable
   ↓
"I have work to execute asynchronously."

DeferredResult
   ↓
"I'll provide the result later."
```

`DeferredResult` can be useful when the result is produced by another asynchronous event or callback.

---

# 19. Important production concern — timeout

Suppose an API waits for an external system:

```text
Payment Service
     |
     v
External Provider
     |
     | 60 seconds
     |
```

You shouldn't allow requests to wait indefinitely.

Use appropriate:

```text
Connection timeout
Read timeout
Request timeout
Async timeout
```

For distributed systems:

> Every network call should have a bounded timeout.

This becomes extremely important when we later discuss **microservices resilience**.

---

# 20. File upload

Spring supports multipart file uploads.

Example:

```java
@PostMapping("/documents")
public ResponseEntity<String> upload(
        @RequestParam("file")
        MultipartFile file) {

    ...
}
```

Client sends:

```text
multipart/form-data
```

rather than ordinary JSON.

---

# 21. What is Multipart?

A multipart request can contain multiple parts.

For example:

```text
multipart/form-data
   |
   +-- file
   |
   +-- metadata
```

Example conceptual request:

```text
file = invoice.pdf
documentType = INVOICE
```

Spring exposes uploaded content through:

```java
MultipartFile
```

---

# 22. Important production concern with file uploads

Don't simply assume:

```java
file.getBytes()
```

is always safe.

Large files can consume significant memory.

For large uploads, consider:

```text
streaming
temporary storage
object storage
size limits
content validation
virus/malware scanning
```

For example, rather than sending a 500 MB file through your application memory, a better architecture may be:

```text
Client
   |
   v
Object Storage
   |
   v
Application receives reference
```

depending on the requirements.

---

# 23. File download

A controller can return a resource/file response.

Conceptually:

```java
@GetMapping("/documents/{id}")
public ResponseEntity<Resource> download(
        @PathVariable String id) {

    Resource resource = documentService.get(id);

    return ResponseEntity.ok()
            .body(resource);
}
```

The exact implementation depends on where the file is stored.

---

# 24. Streaming responses

Sometimes the response itself is large.

Example:

```text
Large CSV
Large report
Large export
Large dataset
```

Instead of building the entire response in memory:

```text
Database
   |
   v
Load entire result
   |
   v
Memory
   |
   v
Response
```

you may stream data progressively:

```text
Database
   |
   v
Read chunk
   |
   v
Send chunk
   |
   v
Read next chunk
   |
   v
Send chunk
```

This reduces memory pressure.

---

# 25. Why is streaming useful?

Suppose:

```text
10 million records
```

If you construct:

```java
List<Record> allRecords;
```

you may create enormous memory pressure.

Streaming can instead process smaller portions.

This connects directly to the pagination/JPA streaming discussions you've had.

But there is an important distinction:

> HTTP response streaming and database streaming are different concerns.

You need to consider both sides carefully.

---

# 26. Streaming vs pagination

### Pagination

Client asks:

```text
GET /transactions?page=0&size=100
```

Then:

```text
GET /transactions?page=1&size=100
```

The client controls the pages.

### Streaming

Server continuously sends pieces of one response.

```text
Response
 ↓
chunk
 ↓
chunk
 ↓
chunk
 ↓
...
```

Use pagination when clients need controlled navigation through a dataset.

Use streaming when a single large result needs to be transferred progressively.

---

# 27. Production API — file upload architecture

For a large document system, a senior engineer might design:

```text
Client
   |
   v
API
   |
   | metadata
   v
Database

Client
   |
   | file
   v
Object Storage
```

Rather than:

```text
Client
   |
   v
Spring Boot
   |
   v
Application memory
   |
   v
Database
```

for every large file.

The right design depends on requirements, but the important principle is:

> Don't unnecessarily make your application server the data pipe for large binary objects.

---

# 28. CORS + Security scenario

### Interviewer:

> “Our frontend is getting a CORS error when calling the Spring Boot API. What would you check?”

A good answer:

```text
1. Confirm frontend and backend origins.
2. Check allowed origins.
3. Check allowed HTTP methods.
4. Check allowed headers.
5. Check whether browser is sending OPTIONS preflight.
6. Check server response to OPTIONS.
7. If Spring Security is present, verify CORS is also configured correctly
   in the security layer.
8. Check credentials/cookie configuration if applicable.
```

Don't simply say:

> “Add `@CrossOrigin`.”

That's often insufficient in a production application.

---

# 29. CORS + credentials

Suppose browser requests include cookies/credentials.

You need to configure credential handling correctly.

One important rule:

You generally cannot combine:

```text
Access-Control-Allow-Origin: *
```

with credentialed cross-origin requests.

Instead, specify permitted origins explicitly.

This is one reason wildcard CORS configuration is often inappropriate for authenticated applications.

---

# 30. Async API scenario

### Interviewer:

> “You have an API that takes 30 seconds to generate a report. What would you design?”

Don't automatically say:

> “Use CompletableFuture.”

A better production design may be:

```text
POST /reports
       |
       v
Create report job
       |
       v
202 Accepted
       |
       v
jobId = R123
```

Then:

```text
GET /reports/R123
```

returns:

```json
{
  "jobId": "R123",
  "status": "COMPLETED",
  "downloadUrl": "..."
}
```

This is often better for genuinely long-running work than keeping an HTTP connection open for 30 seconds or several minutes.

---

# 31. 202 Accepted — practical use

This is exactly where `202 Accepted` makes sense.

Request:

```http
POST /reports
```

Response:

```http
202 Accepted
```

```json
{
  "jobId": "R123",
  "status": "PROCESSING"
}
```

Then:

```text
Client
   |
   +---- POST /reports
   |
   +---- GET /reports/R123
   |
   +---- GET /reports/R123
   |
   +---- GET /reports/R123
```

Eventually:

```json
{
  "jobId": "R123",
  "status": "COMPLETED"
}
```

This pattern is common in distributed systems.

---

# 32. Important distinction — asynchronous vs non-blocking

Don't casually treat these as identical.

### Asynchronous

The caller doesn't necessarily wait for the operation to finish before continuing.

### Non-blocking

A thread isn't blocked waiting for an operation to complete.

You can have asynchronous processing that still performs blocking operations on another thread.

For example:

```text
Request thread
     |
     v
Worker thread
     |
     | blocking DB call
     v
Database
```

The operation is asynchronous from the request's perspective, but the worker thread is still blocked during the DB call.

This distinction becomes much more important when discussing WebFlux/reactive programming, which is **not a major focus of our current roadmap**, so don't spend excessive preparation time on it right now.

---

# 33. EPAM interview rapid-fire

### Q: What is CORS?

A browser security mechanism controlling cross-origin access to resources.

### Q: Is CORS authentication?

No.

### Q: What is a CORS preflight?

An `OPTIONS` request used by the browser to check whether the actual cross-origin request is permitted.

### Q: CORS vs CSRF?

```text
CORS → cross-origin browser access
CSRF → unwanted authenticated action
```

### Q: What is `MultipartFile`?

Spring's abstraction for handling uploaded multipart files.

### Q: Why stream large responses?

To avoid loading the entire response into application memory.

### Q: Does async make an operation faster?

No. It primarily changes how execution/resources are managed.

### Q: When might you return 202?

When the request has been accepted but processing will happen asynchronously.

### Q: What would you do for a 30-minute report generation?

Usually create an asynchronous job and return `202 Accepted` with a job identifier rather than keeping an HTTP request open for 30 minutes.

### Q: Filter or Interceptor for correlation ID?

Generally Filter.

### Q: Does JWT automatically eliminate CSRF?

No. The answer depends on how the authentication credential is transported and stored.

---

# 34. What we've completed

```text
Spring Web / REST

HTTP fundamentals              ✅
HTTP methods                   ✅
Status codes                   ✅
Headers                        ✅
Content-Type / Accept          ✅
Path / query parameters        ✅

REST API design                ✅
Resource design                ✅
Idempotency                    ✅
API versioning                 ✅
Backward compatibility         ✅
Error responses                ✅

Spring MVC internals           ✅
DispatcherServlet              ✅
HandlerMapping                 ✅
HandlerAdapter                 ✅
Argument resolution            ✅
Message converters             ✅
Jackson                        ✅

Filters / Interceptors         ✅
CORS                           ✅
CSRF                           ✅
Async request processing       ✅
File upload/download           ✅
Streaming                      ✅
```

# Remaining in Spring Web / REST

The major remaining **production REST** topics are:

```text
⭐⭐⭐⭐⭐
Timeouts
Rate limiting
Retry considerations
Idempotency — deeper implementation
API security
Request tracing
Observability
Backward compatibility
Production API design
```

We'll cover those next, with practical scenarios such as:

```text
Payment API
   ↓
Timeout
   ↓
Retry
   ↓
Duplicate request
   ↓
Idempotency
   ↓
Rate limiting
   ↓
Circuit breaker
```

That will bring **Spring Web / REST** to completion before we move to **JPA/Hibernate**.