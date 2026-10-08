# Spring Web — Async Request Processing, File Upload/Download & Streaming ⭐⭐⭐⭐⭐

This is a **focused interview document**. The goal is to understand the practical design decisions and the questions EPAM is likely to ask—not memorize Spring internals.

---

# PART 1 — ASYNC REQUEST PROCESSING

## 1. Why do we need asynchronous request processing?

### Interview Question

**Why would we process an HTTP request asynchronously?**

Consider:

```text
Client
  |
  | GET /generate-report
  v
Spring Boot
  |
  | Generate report
  | 30 seconds
  v
Response
```

In normal synchronous processing, the request-processing thread remains involved while the operation is executing.

If many requests are doing this:

```text
100 requests
×
30 seconds
```

we can consume a large number of server threads.

Async processing allows the application to handle long-running work differently so that the initial request thread doesn't have to remain occupied for the entire operation.

### Important:

**Async does NOT automatically make the underlying operation faster.**

If the database query takes 10 seconds:

```text
Synchronous → 10 sec
Asynchronous → still potentially 10 sec
```

The benefit is primarily around **resource/thread utilization and request handling**.

---

# 2. Synchronous vs asynchronous

### Synchronous

```text
Request
   |
   v
Thread
   |
   | execute operation
   |
   | wait
   |
   v
Response
```

The request processing remains tied to the operation.

### Asynchronous

Conceptually:

```text
Request
   |
   v
Start async work
   |
   v
Request processing can be released
   |
   |
   v
Async work completes
   |
   v
Response
```

The exact mechanics depend on the Spring MVC async API being used.

---

# 3. `Callable` in Spring MVC

### Interview Question

**How can a Spring MVC controller perform work asynchronously?**

One option is `Callable`.

Example:

```java
@GetMapping("/reports/{id}")
public Callable<ReportResponse> getReport(
        @PathVariable String id) {

    return () -> reportService.generate(id);
}
```

Instead of directly executing:

```java
reportService.generate(id);
```

inside the controller's initial execution, Spring can perform the `Callable` asynchronously using its async request-processing infrastructure.

Conceptually:

```text
HTTP Request
     |
     v
Controller
     |
     v
Callable
     |
     v
Async execution
     |
     v
Report generated
     |
     v
HTTP Response
```

---

# 4. Is `Callable` the same as `CompletableFuture`?

No.

They solve related but different problems.

### `Callable`

A Java task that:

```text
returns a value
and
may throw an exception
```

Spring MVC can use it for asynchronous request processing.

### `CompletableFuture`

A more powerful Java concurrency abstraction supporting:

- asynchronous computation
- composition
- chaining
- combining tasks
- exception handling

Example:

```java
CompletableFuture
        .supplyAsync(() -> service.getPayment())
        .thenApply(this::mapResponse);
```

For EPAM, remember:

> `Callable` is one Spring MVC async return type. `CompletableFuture` is a general-purpose Java asynchronous computation API that Spring MVC can also adapt to asynchronous request handling.

---

# 5. `CompletableFuture` in Spring MVC

Example:

```java
@GetMapping("/payments/{id}")
public CompletableFuture<PaymentResponse> getPayment(
        @PathVariable String id) {

    return CompletableFuture.supplyAsync(
            () -> paymentService.getPayment(id),
            paymentExecutor
    );
}
```

Conceptually:

```text
HTTP request
      |
      v
Controller
      |
      v
CompletableFuture
      |
      v
Executor
      |
      v
Payment service
      |
      v
Future completed
      |
      v
HTTP response
```

---

# 6. Why should we avoid blindly using `supplyAsync()`?

A common mistake is:

```java
CompletableFuture.supplyAsync(
    () -> service.process()
);
```

without specifying an executor.

This may use the **common ForkJoinPool**.

For a production service, blindly putting blocking work there can be problematic.

For example:

```text
HTTP request
      |
      v
CompletableFuture
      |
      v
Common ForkJoinPool
      |
      v
Blocking DB call
```

If many operations block:

```text
Worker threads occupied
        ↓
Less capacity
        ↓
Latency increases
        ↓
Potential resource exhaustion
```

For controlled application workloads, use an appropriately configured executor.

---

# 7. Executor example

For example:

```java
@Bean
public Executor paymentExecutor() {

    ThreadPoolTaskExecutor executor =
            new ThreadPoolTaskExecutor();

    executor.setCorePoolSize(10);
    executor.setMaxPoolSize(20);
    executor.setQueueCapacity(100);
    executor.setThreadNamePrefix("payment-");

    executor.initialize();

    return executor;
}
```

Then:

```java
CompletableFuture.supplyAsync(
        () -> paymentService.process(),
        paymentExecutor
);
```

The exact pool size should be based on workload and tested rather than choosing arbitrary numbers.

---

# 8. Very important: thread-pool exhaustion

### Interview Question

**What happens if you have a small thread pool and thousands of requests?**

Suppose:

```text
Pool size = 10
```

and each task performs:

```text
blocking external API call
```

for:

```text
5 seconds
```

The first 10 tasks occupy the workers.

Additional tasks may:

```text
wait in queue
```

and eventually:

```text
queue fills
     ↓
tasks rejected
```

This can cause:

- increased latency
- rejected tasks
- cascading failures
- resource exhaustion

Therefore async processing still requires **capacity planning**.

---

# 9. `DeferredResult`

### Interview Question

**What is `DeferredResult`?**

`DeferredResult` is useful when the result will be produced later by some asynchronous operation.

Example:

```java
@GetMapping("/reports/{id}")
public DeferredResult<ReportResponse> getReport(
        @PathVariable String id) {

    DeferredResult<ReportResponse> result =
            new DeferredResult<>(30_000L);

    executor.execute(() -> {

        ReportResponse response =
                reportService.generate(id);

        result.setResult(response);
    });

    return result;
}
```

The important concept is:

```text
Controller
   |
   v
DeferredResult
   |
   | result produced later
   v
setResult(...)
   |
   v
HTTP response
```

---

# 10. `Callable` vs `DeferredResult`

Don't memorize framework internals.

Remember the conceptual distinction:

```text
Callable
    ↓
"I have a task that should execute asynchronously."

DeferredResult
    ↓
"I will provide the response result later."
```

`DeferredResult` is particularly useful when the result comes from an asynchronous callback/event rather than simply running one method in another thread.

---

# 11. `WebAsyncTask`

Spring MVC also provides `WebAsyncTask`.

Conceptually:

```java
@GetMapping("/reports")
public WebAsyncTask<ReportResponse> report() {

    return new WebAsyncTask<>(
            10_000,
            () -> reportService.generate()
    );
}
```

It can be useful when you want async processing together with timeout/callback handling.

For EPAM:

> Know that it exists and understand the use case. You do not need to memorize every API method.

---

# 12. Async timeout

### Interview Question

**Why do asynchronous HTTP requests need timeouts?**

Because otherwise a request could remain pending indefinitely.

For example:

```text
Payment Service
      |
      v
External Provider
      |
      | no response
      |
      X
```

Without a timeout, resources can remain occupied unnecessarily.

We should define reasonable timeouts.

Example:

```text
Request timeout
External API timeout
Async operation timeout
```

And then handle timeout explicitly.

---

# 13. Timeout does not mean cancellation is guaranteed

Important senior-level nuance.

Suppose:

```text
HTTP request timeout
        ↓
Client stops waiting
```

That doesn't automatically guarantee that every underlying operation has stopped.

For example:

```text
Client
  |
  X timeout
  |
Spring async task
  |
  v
Database operation still running
```

Therefore, cancellation/resource cleanup needs to be designed appropriately.

---

# 14. Async exception handling

Suppose:

```java
CompletableFuture.supplyAsync(
        () -> paymentService.process(),
        executor
);
```

and:

```text
paymentService.process()
       ↓
throws exception
```

The exception becomes part of the `CompletableFuture` completion state.

For example:

```java
return CompletableFuture
        .supplyAsync(
            () -> paymentService.process(),
            executor
        )
        .exceptionally(ex -> {
            log.error("Payment failed", ex);
            return fallbackResponse();
        });
```

Or use:

```java
.handle(...)
```

or:

```java
.whenComplete(...)
```

depending on the required behavior.

---

# 15. `thenApply` vs `thenCompose`

Since EPAM focuses heavily on Java 8+ concurrency, know this.

### `thenApply`

Used when the next operation returns a normal value.

```java
CompletableFuture<User> future = ...;

CompletableFuture<String> result =
        future.thenApply(User::getName);
```

Conceptually:

```text
Future<User>
     |
     v
User → String
     |
     v
Future<String>
```

### `thenCompose`

Used when the next operation itself returns a `CompletableFuture`.

```java
CompletableFuture<User> userFuture = ...;

CompletableFuture<Order> orderFuture =
        userFuture.thenCompose(
            user -> getOrderAsync(user.getId())
        );
```

It avoids nested futures:

```text
Bad conceptual result:

CompletableFuture<
    CompletableFuture<Order>
>
```

Instead:

```text
CompletableFuture<Order>
```

This is a Java concurrency topic, but it's directly relevant when discussing async Spring APIs.

---

# 16. Async + transaction — important trap

Suppose:

```java
@Transactional
public void process() {

    savePayment();

    CompletableFuture.runAsync(() -> {
        updateLedger();
    });
}
```

Do **not** assume the transaction automatically propagates to the async thread.

Transactions are generally associated with the executing thread/context.

So:

```text
Main thread
    |
    | Transaction A
    |
    +---- async task
             |
             | different thread
             |
             X
        Transaction A
        is not automatically
        propagated
```

If the async operation needs its own transaction, it should be designed explicitly.

This is a very useful Senior Engineer follow-up.

---

# 17. Async + MDC/correlation ID

Same issue applies to MDC.

If:

```java
MDC.put("correlationId", "abc123");
```

on the request thread, then:

```java
CompletableFuture.runAsync(...)
```

may execute on another thread.

The MDC context isn't automatically guaranteed to be available there.

Therefore production applications may need:

```text
TaskDecorator
context propagation
OpenTelemetry context propagation
```

This connects directly with our earlier observability topic.

---

# PART 2 — FILE UPLOAD

# 18. How does file upload work in Spring?

A typical file upload uses:

```text
multipart/form-data
```

Example controller:

```java
@PostMapping("/documents")
public ResponseEntity<String> upload(
        @RequestParam("file")
        MultipartFile file) {

    documentService.store(file);

    return ResponseEntity.ok("Uploaded");
}
```

Spring exposes the uploaded file through:

```java
MultipartFile
```

---

# 19. What is `MultipartFile`?

`MultipartFile` represents an uploaded file received in a multipart request.

Useful methods include:

```java
file.getOriginalFilename()
file.getContentType()
file.getSize()
file.getInputStream()
file.getBytes()
```

For example:

```java
if (file.isEmpty()) {
    throw new IllegalArgumentException(
            "File is empty");
}
```

---

# 20. Why shouldn't we blindly use `getBytes()`?

Consider:

```java
byte[] data = file.getBytes();
```

For a small file, this may be fine.

But imagine:

```text
500 MB file
```

Loading the entire content into memory can create significant memory pressure.

For large files, prefer streaming approaches such as:

```java
InputStream inputStream =
        file.getInputStream();
```

and process/store the data incrementally where appropriate.

---

# 21. File upload validation

Never trust uploaded files.

At minimum consider:

```text
File size
Content type
File extension
Actual file content
Filename
Authentication/authorization
Malware scanning
Storage location
```

For example, don't rely solely on:

```text
filename = something.pdf
```

to conclude that the content is actually a safe PDF.

---

# 22. File size limits

Production APIs should have upload limits.

Why?

Without limits:

```text
Attacker/client
      |
      v
Huge file
      |
      v
Memory/disk/network exhaustion
```

So configure appropriate:

```text
Maximum file size
Maximum request size
```

based on the business requirement.

Spring Boot supports multipart size configuration.

---

# 23. Should uploaded files be stored in the database?

Not always.

For large binary files, object storage is often more appropriate.

Architecture:

```text
Client
   |
   | File
   v
Object Storage
   |
   +---- file
   |
   v
Application
   |
   v
Database
   |
   +---- metadata
        fileId
        filename
        contentType
        storageKey
```

The database stores metadata/reference rather than the entire large binary object.

Examples of object storage include cloud storage systems such as S3-compatible storage.

---

# 24. When might storing files in the DB make sense?

It can make sense for:

- small files
- strong transactional requirements
- specific compliance requirements
- systems where DB-backed binary storage is deliberately chosen

But don't automatically assume:

> “Files should always go to object storage.”

Architecture depends on:

```text
File size
Access pattern
Transaction requirements
Cost
Compliance
Backup strategy
```

---

# 25. Large file upload architecture

For a large file:

```text
          Client
             |
             v
       Object Storage
             |
             | file reference
             v
       Spring Boot API
             |
             v
          Database
```

An even better design can use a pre-signed upload URL:

```text
Client
   |
   | request upload URL
   v
Backend
   |
   | pre-signed URL
   v
Client
   |
   | directly uploads file
   v
Object Storage
```

This prevents your application server from unnecessarily becoming the file-transfer bottleneck.

You don't need to memorize the exact cloud API for the interview; understand the architecture.

---

# PART 3 — FILE DOWNLOAD

# 26. How do you download a file from Spring?

A common approach is returning a `Resource`.

Example:

```java
@GetMapping("/documents/{id}")
public ResponseEntity<Resource> download(
        @PathVariable String id) {

    Resource resource =
            documentService.getResource(id);

    return ResponseEntity.ok()
            .body(resource);
}
```

Depending on requirements, the response should also specify appropriate headers.

---

# 27. Content-Type

The response should have the appropriate media type.

For example:

```text
application/pdf
text/csv
image/png
application/octet-stream
```

Example:

```java
return ResponseEntity.ok()
        .contentType(MediaType.APPLICATION_PDF)
        .body(resource);
```

---

# 28. Content-Disposition

For downloads, you may want:

```http
Content-Disposition: attachment; filename="report.pdf"
```

This tells the browser that the content should generally be treated as a downloadable attachment.

For inline viewing:

```http
Content-Disposition: inline
```

The correct choice depends on the desired client behavior.

---

# 29. Important security concern — filename/path traversal

Suppose an API accepts:

```text
filename=../../etc/passwd
```

and blindly uses it to construct a server filesystem path.

That's dangerous.

Never blindly trust user-controlled filenames or paths.

Use:

```text
server-generated IDs
safe storage keys
path validation
controlled directories
```

This is a good practical interview point.

---

# PART 4 — STREAMING

# 30. Why do we need response streaming?

Suppose your API generates:

```text
2 GB CSV
```

Bad approach:

```text
Database
   |
   v
Load entire 2 GB
   |
   v
Java memory
   |
   v
HTTP response
```

This can cause:

```text
Huge memory consumption
GC pressure
OutOfMemoryError
```

Instead:

```text
Database
   |
   v
Read chunk
   |
   v
Write response
   |
   v
Read next chunk
   |
   v
Write response
```

---

# 31. `StreamingResponseBody`

Spring MVC provides `StreamingResponseBody` for streaming response data.

Example:

```java
@GetMapping("/reports")
public ResponseEntity<StreamingResponseBody>
downloadReport() {

    StreamingResponseBody stream =
            outputStream -> {

                outputStream.write(
                        "id,name\n".getBytes()
                );

                // write more data progressively
            };

    return ResponseEntity.ok()
            .contentType(
                MediaType.TEXT_PLAIN)
            .body(stream);
}
```

The important idea:

> Instead of constructing the entire response body first, write the response progressively.

---

# 32. Streaming vs returning `List`

Suppose:

```java
@GetMapping("/transactions")
public List<Transaction> getTransactions() {
    return repository.findAll();
}
```

If there are:

```text
10 million rows
```

this can be disastrous.

Potential flow:

```text
10 million DB rows
       ↓
10 million Java objects
       ↓
Huge heap usage
       ↓
Serialization
       ↓
Huge response
```

Streaming or pagination may be much safer depending on the API requirements.

---

# 33. Streaming vs pagination

### Pagination

```text
GET /transactions?page=0&size=1000
```

Client explicitly asks for manageable chunks.

Advantages:

- controlled response size
- easier retries
- easier navigation
- better API usability

### Streaming

```text
GET /transactions/export
```

Server sends one large logical response progressively.

Advantages:

- lower application memory usage
- suitable for large exports
- client can begin receiving data before the entire result is generated

---

# 34. Streaming is not automatically better

This is important.

For a normal REST API:

```text
GET /users
```

returning 20 users:

```text
List<User>
```

is perfectly reasonable.

You don't need streaming just because streaming exists.

Use it when:

```text
Large response
Long-running generation
Large export
Memory constraints
Progressive delivery
```

make it useful.

---

# 35. Database streaming vs HTTP streaming

These are two different problems.

Suppose:

```text
Database
   |
   v
Application
   |
   v
HTTP Client
```

You might need:

```text
DB streaming/pagination
        +
HTTP response streaming
```

Otherwise you could solve one side but still buffer everything on the other.

Example:

```text
Database
   |
   | pagination
   v
Application
   |
   | streaming response
   v
Client
```

This is often a better architecture for very large exports.

---

# 36. The dangerous approach

Avoid:

```java
List<Record> records =
        repository.findAll();
```

followed by:

```java
return records;
```

for huge datasets.

Instead consider:

```text
Pagination
OR
keyset/cursor iteration
OR
controlled database streaming
+
streamed HTTP response
```

The right choice depends on the database and consistency requirements.

---

# 37. What happens if the client disconnects?

Suppose:

```text
Server
   |
   | streaming 2 GB file
   |
   v
Client disconnects
```

The server should not blindly continue doing expensive work forever.

Streaming implementations should account for:

```text
Client disconnect
IOException
Cancellation
Resource cleanup
Database cursor cleanup
```

This is especially important for expensive exports.

---

# 38. Large export — senior-level design

Suppose:

> “Export 100 million ledger records as CSV.”

Do **not** propose:

```text
DB
 ↓
findAll()
 ↓
Java List
 ↓
CSV
 ↓
HTTP
```

A much better approach is:

```text
Client
   |
   | POST /exports
   v
Create export job
   |
   v
202 Accepted
   |
   v
Background processing
   |
   +---- read DB in chunks
   |
   +---- generate file
   |
   +---- store in object storage
   |
   v
Export completed
   |
   v
Client gets download URL
```

This is generally much more robust for extremely large exports.

---

# 39. Async + file + streaming together

These concepts can work together.

Example:

> Generate a 5 GB report.

Architecture:

```text
                    Client
                       |
                       | POST /reports
                       v
                Report Service
                       |
                       | create job
                       v
                  202 Accepted
                       |
                       v
                 Background Job
                       |
              +--------+--------+
              |                 |
              v                 v
          Database         Object Storage
          in chunks          report.csv
                                |
                                v
                           Download URL
                                |
                                v
                             Client
```

Then the client downloads the completed file.

This is generally better than keeping:

```text
POST /reports
```

open for several minutes.

---

# 40. Interview scenario — API takes 30 seconds

### Interviewer:

> “Your report API takes 30 seconds. What would you do?”

Don't immediately answer:

> “Use CompletableFuture.”

First ask:

```text
Is 30 seconds acceptable for synchronous HTTP?
Is the client actually waiting for the result?
Is this a long-running business job?
How large is the result?
```

If it is a genuinely long-running operation:

```text
POST /reports
     ↓
202 Accepted
     ↓
jobId
     ↓
background processing
     ↓
GET /reports/{jobId}
     ↓
COMPLETED
     ↓
download
```

This is usually a stronger production design.

---

# 41. Interview scenario — download a 2 GB file

### Strong answer

> “I wouldn't load the entire file into application memory. I'd preferably store the file in object storage and either provide a controlled download endpoint that streams the file or, depending on the architecture, provide a pre-signed download URL. If the application serves the file itself, I'd stream it and configure appropriate response headers and timeouts.”

---

# 42. Interview scenario — upload a 1 GB file

### Strong answer

> “I wouldn't load the entire file into a byte array. I'd use multipart upload handling with size limits and stream the content to durable storage. For large files, object storage with direct or pre-signed uploads is often preferable so the application server isn't the transfer bottleneck. I'd also validate authorization, content type/file content, and consider malware scanning depending on the use case.”

---

# 43. Interview scenario — 10 million records

### Interviewer:

> “Why not simply return `List<Record>`?”

Answer:

> “Loading millions of records into a Java collection can create severe heap pressure and GC overhead. I'd use pagination, keyset/cursor iteration, or controlled database streaming depending on the use case. If the result is an export, I'd process records in chunks and stream or write them incrementally rather than building the entire result in memory.”

---

# 44. Interview scenario — async thread pool

### Interviewer:

> “Why did your async application suddenly become slow?”

Possible investigation:

```text
Check:
  ↓
Executor pool size
  ↓
Active threads
  ↓
Queue size
  ↓
Rejected tasks
  ↓
Task execution time
  ↓
Blocking DB/external calls
  ↓
Downstream latency
```

For example:

```text
Pool = 10
Queue = 100

DB calls suddenly take 10 seconds

        ↓

Workers remain occupied

        ↓

Queue grows

        ↓

Latency increases

        ↓

Eventually tasks are rejected
```

This is a classic thread-pool saturation problem.

---

# 45. Async vs non-blocking — final interview answer

### Question:

**Is asynchronous processing the same as non-blocking processing?**

No.

> Asynchronous means the caller doesn't necessarily wait for the operation to finish in the same execution flow. Non-blocking means the executing thread isn't blocked waiting for an operation to complete. An asynchronous task can still perform blocking I/O on another thread.

For example:

```text
CompletableFuture
      |
      v
Worker Thread
      |
      v
Blocking JDBC call
```

This is asynchronous from the request's perspective, but the worker thread is still blocked.

---

# 46. What you actually need to remember for EPAM

Don't memorize every API.

Remember this:

```text
ASYNC
 ├── Callable
 ├── DeferredResult
 ├── CompletableFuture
 ├── Executor
 ├── Timeout
 ├── Exception handling
 ├── Thread-pool exhaustion
 ├── MDC/context propagation
 └── Transaction context doesn't automatically cross threads

FILE
 ├── multipart/form-data
 ├── MultipartFile
 ├── size limits
 ├── validation/security
 ├── don't blindly use getBytes() for huge files
 └── object storage for large files

DOWNLOAD
 ├── Resource
 ├── ResponseEntity
 ├── Content-Type
 └── Content-Disposition

STREAMING
 ├── StreamingResponseBody
 ├── avoid huge in-memory List
 ├── DB chunking/pagination
 ├── progressive HTTP response
 ├── client disconnect
 └── large export architecture
```

---

# 47. Most likely EPAM questions

### ⭐⭐⭐⭐⭐

**1. What is asynchronous request processing in Spring MVC?**

**2. `Callable` vs `DeferredResult`?**

**3. Can I use `CompletableFuture` in a Spring controller?**

**4. Why shouldn't I blindly use the common ForkJoinPool?**

**5. What happens when an executor becomes saturated?**

**6. Does async make a slow DB query faster?**

**7. Does `@Transactional` automatically propagate to an async thread?**

**8. How do you upload a file in Spring Boot?**

**9. How would you handle a 1 GB file?**

**10. Why shouldn't you use `file.getBytes()` for huge files?**

**11. How do you download a file?**

**12. What is streaming?**

**13. Streaming vs pagination?**

**14. How would you export 10 million records?**

**15. How would you design a report API that takes several minutes?**

These are the questions worth knowing—not obscure Spring implementation details.

---

# 48. Final mental model

```text
              SPRING WEB — LARGE/ASYNC OPERATIONS
                           |
          +----------------+----------------+
          |                |                |
         ASYNC             FILE          STREAMING
          |                |                |
      Callable         MultipartFile    StreamingResponseBody
      DeferredResult       |                |
      CompletableFuture    |                |
          |                |                |
      Executor         Object Storage       |
          |                |                |
      Timeout           Large Files          |
          |                |                |
      Thread Pool       Download             |
          |                |                |
      Context           Resource             |
      Propagation       Content-Type         |
          |                |                |
          +----------------+----------------+
                           |
                           v
                  Production Design
                           |
                           v
             Don't block indefinitely
             Don't exhaust memory
             Don't exhaust threads
             Don't duplicate work
             Don't make app server
             an unnecessary file-transfer bottleneck
```

## Key Senior-level takeaway

For normal APIs, keep things simple.

For **long-running operations**, consider:

```text
202 Accepted
      +
background job
      +
job status
```

For **large files**, consider:

```text
Object Storage
      +
direct/pre-signed upload/download
```

For **large responses**, consider:

```text
DB pagination/chunking
      +
streamed output
```

For **async execution**, always think about:

```text
Executor
Timeout
Capacity
Exception handling
Context propagation
Transaction boundaries
```