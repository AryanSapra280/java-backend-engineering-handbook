Absolutely — moving to the **next exact topic in the roadmap**.


# Spring Boot Production — Actuator, Health Checks, Readiness, Liveness, Metrics & Logging ⭐⭐⭐⭐⭐

This is a **very important Senior Software Engineer interview area** because interviewers often move from:

> “How does Spring Boot work?”

to:

> “Your application is running in Kubernetes. How do you know whether it is healthy? How do you debug it when users report failures?”

---

# 1. What is Spring Boot Actuator?

### Interview Question

**What is Spring Boot Actuator?**

### Interview-ready answer

Spring Boot Actuator provides **production-ready monitoring and management capabilities** for a Spring Boot application.

It exposes endpoints that allow us to inspect things such as:

- application health
- application information
- metrics
- environment/configuration
- beans
- mappings
- thread information
- Prometheus metrics

It is mainly used for **observability and operational monitoring**.

Typical architecture:

```text
Spring Boot Application
        |
        v
    Actuator
        |
   +----+----+
   |         |
 Health    Metrics
   |         |
   v         v
Kubernetes Prometheus
               |
               v
             Grafana
```

---

# 2. How do we enable Actuator?

Add:

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

Then we can expose endpoints.

Example:

```properties
management.endpoints.web.exposure.include=health,info,metrics
```

For Prometheus:

```properties
management.endpoints.web.exposure.include=health,info,metrics,prometheus
```

The default management base path is:

```text
/actuator
```

So we may have:

```text
GET /actuator/health
GET /actuator/info
GET /actuator/metrics
GET /actuator/prometheus
```

### Senior-level point

**We should not blindly expose every Actuator endpoint publicly.**

For example, exposing environment/configuration information can reveal sensitive operational details.

Production systems generally:

- expose only required endpoints
- secure management endpoints
- restrict access through network/security controls
- avoid exposing sensitive details

---

# 3. What is `/actuator/health`?

### Interview Question

**What does `/actuator/health` tell us?**

It provides the health status of the application and can aggregate health information from different components.

Example:

```json
{
  "status": "UP"
}
```

With details:

```json
{
  "status": "UP",
  "components": {
    "db": {
      "status": "UP"
    },
    "redis": {
      "status": "UP"
    }
  }
}
```

The actual components depend on what health indicators are available/configured.

For example:

```text
Application
   |
   +-- Database
   |
   +-- Redis
   |
   +-- Kafka
   |
   +-- Disk
```

---

# 4. What is a HealthIndicator?

### Interview Question

**How does Spring Boot determine whether something is healthy?**

Spring Boot Actuator uses **HealthIndicator** implementations.

For example:

```java
@Component
public class PaymentServiceHealthIndicator
        implements HealthIndicator {

    @Override
    public Health health() {

        boolean healthy = checkPaymentDependency();

        if (healthy) {
            return Health.up()
                    .withDetail("payment-provider", "available")
                    .build();
        }

        return Health.down()
                .withDetail("payment-provider", "unavailable")
                .build();
    }

    private boolean checkPaymentDependency() {
        return true;
    }
}
```

Now Actuator can include this component in health information.

---

# 5. Health Check vs Readiness vs Liveness

This distinction is **extremely important for Kubernetes interviews**.

There are three different questions:

```text
Is my application alive?
        |
        v
Liveness

Can my application receive traffic?
        |
        v
Readiness
```

---

# 6. Liveness Probe

### Interview Question

**What is a liveness probe?**

Liveness answers:

> “Is this application instance still functioning, or should Kubernetes restart it?”

For example:

```text
Application deadlocked
Application stuck
Application cannot make progress
Application process is unhealthy
```

Kubernetes can restart the container when the liveness probe fails.

Conceptually:

```text
Kubernetes
    |
    | liveness probe
    v
Application

UP
 |
 +----> continue running

DOWN
 |
 +----> restart container
```

---

# 7. Readiness Probe

### Interview Question

**What is a readiness probe?**

Readiness answers:

> “Can this application instance currently receive traffic?”

This is different from liveness.

Suppose:

```text
Pod is running
Application started
But database connection initialization is incomplete
```

The application may be **alive but not ready**.

Therefore:

```text
Liveness = should Kubernetes restart me?

Readiness = should Kubernetes send traffic to me?
```

This distinction is critical.

---

# 8. Real production scenario

Imagine we have:

```text
             Load Balancer
                  |
        +---------+---------+
        |         |         |
       Pod A     Pod B     Pod C
       Ready     Ready     Not Ready
```

Kubernetes should send traffic only to:

```text
Pod A
Pod B
```

Pod C remains running but does not receive traffic.

This prevents requests from reaching an instance that is not ready.

---

# 9. Running ≠ Ready

This is a very common interview trap.

### Question

**If Kubernetes says the pod is Running, does that mean it can receive traffic?**

### Answer

No.

`Running` means the container has started and is running.

`Ready` means the application has passed its readiness check and is considered capable of serving traffic.

Therefore:

```text
Running != Ready
```

A pod can be:

```text
Running
Not Ready
```

---

# 10. Spring Boot Kubernetes probes

Spring Boot Actuator can expose dedicated health probe endpoints such as:

```text
/actuator/health/liveness
/actuator/health/readiness
```

These can be connected to Kubernetes probes.

Conceptually:

```text
Kubernetes
    |
    +---- /actuator/health/liveness
    |
    +---- /actuator/health/readiness
```

The exact configuration depends on the Spring Boot version and deployment environment.

---

# 11. Kubernetes configuration example

A simplified deployment could look like:

```yaml
livenessProbe:
  httpGet:
    path: /actuator/health/liveness
    port: 8080
  initialDelaySeconds: 30
  periodSeconds: 10

readinessProbe:
  httpGet:
    path: /actuator/health/readiness
    port: 8080
  initialDelaySeconds: 10
  periodSeconds: 5
```

Meaning:

```text
Liveness fails
      ↓
Kubernetes may restart pod

Readiness fails
      ↓
Kubernetes stops sending traffic
```

---

# 12. Very important senior-level question

### Interview Question

**Should we put database availability in the liveness probe?**

Usually, **no**.

Suppose:

```text
Application
     |
     X
Database temporarily unavailable
```

If database failure causes liveness to fail:

```text
DB down
 ↓
Liveness DOWN
 ↓
Kubernetes restarts pod
 ↓
Pod starts
 ↓
DB still down
 ↓
Liveness DOWN
 ↓
Restart again
```

This can create a restart loop.

The application process itself may still be perfectly healthy.

Therefore:

> Liveness should generally indicate whether the application itself is alive, not whether every external dependency is available.

Readiness is often more appropriate for dependency-related inability to serve traffic, but even readiness checks should be designed carefully.

---

# 13. Liveness vs Readiness — interview table

| Concept | Question | Failure action |
|---|---|---|
| Liveness | Is the application alive? | Restart |
| Readiness | Can it receive traffic? | Remove from traffic |
| Startup | Has application finished starting? | Give startup time |

A useful mental model:

```text
Startup
   ↓
Liveness
   ↓
Readiness
```

---

# 14. What are Metrics?

### Interview Question

**What are metrics?**

Metrics are numerical measurements that describe application/system behavior over time.

Examples:

```text
HTTP request count
HTTP request latency
Error count
JVM memory
CPU
GC activity
Database connection pool usage
Kafka consumer lag
Business transaction count
```

Instead of asking:

> “What happened?”

metrics help answer:

> “How is the system behaving?”

---

# 15. Micrometer

Spring Boot commonly uses **Micrometer** as its metrics abstraction.

Think of Micrometer as:

```text
Application
      |
      v
  Micrometer
      |
      +---- Prometheus
      +---- Datadog
      +---- CloudWatch
      +---- Other monitoring systems
```

This allows application instrumentation without tightly coupling application code to one monitoring vendor.

---

# 16. Types of metrics

The three important concepts are:

### Counter

Represents a value that generally increases.

Example:

```text
payments.created = 10000
```

Code:

```java
Counter counter =
        registry.counter("payments.created");

counter.increment();
```

---

### Gauge

Represents a value that can go up or down.

Examples:

```text
Queue size
Active connections
Current memory usage
```

Conceptually:

```text
Queue size:

10
 ↓
20
 ↓
5
 ↓
0
```

---

### Timer

Measures duration.

For example:

```text
Payment API latency
Database query duration
External API response time
```

Example:

```java
Timer.Sample sample = Timer.start(registry);

try {
    processPayment();
} finally {
    sample.stop(paymentTimer);
}
```

---

# 17. Prometheus + Grafana

A very common production architecture is:

```text
Spring Boot
     |
     v
 Micrometer
     |
     v
Prometheus
     |
     v
 Grafana
```

### Prometheus

Prometheus collects/stores metrics.

### Grafana

Grafana visualizes those metrics using dashboards.

Example:

```text
Grafana Dashboard

HTTP Requests
     15,234/min

Error Rate
     1.2%

P95 Latency
     240 ms

CPU
     65%

Memory
     72%
```

---

# 18. What is `/actuator/prometheus`?

When the Prometheus registry is configured, Spring Boot can expose metrics in Prometheus format.

Conceptually:

```text
GET /actuator/prometheus
```

Prometheus periodically scrapes this endpoint.

```text
Prometheus
    |
    | HTTP scrape
    v
/actuator/prometheus
```

---

# 19. VERY important metrics concept — Cardinality

### Interview Question

**What is metric cardinality and why is it dangerous?**

Suppose you create:

```text
payment_requests{userId="123"}
payment_requests{userId="124"}
payment_requests{userId="125"}
...
```

If there are millions of users, you create millions of unique time series.

That's **high cardinality**.

This can cause:

- high memory usage
- expensive monitoring
- slower queries
- excessive storage

Avoid putting highly dynamic values into metric tags.

Bad:

```text
userId
transactionId
requestId
email
```

Better:

```text
endpoint
httpMethod
status
paymentType
```

Example:

```text
http_requests{
    method="POST",
    endpoint="/payments",
    status="200"
}
```

---

# 20. Logging

### Interview Question

**How do you implement logging in a Spring Boot application?**

Use a logging abstraction such as:

```text
SLF4J
```

with an implementation such as:

```text
Logback
```

Example:

```java
private static final Logger log =
        LoggerFactory.getLogger(PaymentService.class);

log.info("Payment created successfully. paymentId={}",
        paymentId);
```

Avoid:

```java
System.out.println("Payment created");
```

because production logging requires:

- log levels
- centralized collection
- structured logging
- correlation
- filtering
- rotation
- observability integration

---

# 21. Log levels

Typical levels:

```text
TRACE
DEBUG
INFO
WARN
ERROR
```

Use them appropriately.

### INFO

Important normal application events.

```java
log.info("Payment processed. paymentId={}", paymentId);
```

### DEBUG

Detailed diagnostic information.

```java
log.debug("Calling payment provider. request={}", request);
```

Be careful with sensitive data.

### WARN

Something unexpected but not necessarily a failure.

```java
log.warn("Payment provider response is slow");
```

### ERROR

Operation failed or requires attention.

```java
log.error("Payment processing failed. paymentId={}",
        paymentId, exception);
```

---

# 22. Never log sensitive information

Senior-level production question:

### “What should you never log?”

Avoid logging:

```text
Passwords
JWT tokens
Authorization headers
Credit card numbers
CVV
Secrets
API keys
Sensitive personal information
```

Bad:

```java
log.info("Authorization={}", authorizationHeader);
```

Better:

```java
log.info("Authenticated request received. userId={}", userId);
```

Even user IDs should be evaluated according to privacy/security requirements.

---

# 23. What is structured logging?

Traditional log:

```text
Payment created successfully for payment 12345
```

Structured log:

```json
{
  "timestamp": "2026-10-05T10:30:00Z",
  "level": "INFO",
  "service": "payment-service",
  "event": "payment_created",
  "paymentId": "12345",
  "correlationId": "abc-123"
}
```

Structured logs are easier for machines to search and analyze.

For example:

```text
ELK
OpenSearch
Splunk
Cloud logging
```

can query fields directly.

---

# 24. Correlation ID

This is **very important given the practical Kubernetes questions you previously encountered.**

Imagine:

```text
Client
  |
  v
API Gateway
  |
  v
Payment Service
  |
  v
Ledger Service
  |
  v
Notification Service
```

One request passes through four services.

Without correlation:

```text
Payment Service logs
Ledger Service logs
Notification Service logs
```

How do we know which logs belong to the same request?

That's where a correlation ID helps.

---

# 25. Correlation ID flow

```text
Client
   |
   | X-Correlation-ID: abc123
   v
Gateway
   |
   v
Payment Service
   |
   | X-Correlation-ID: abc123
   v
Ledger Service
   |
   | X-Correlation-ID: abc123
   v
Notification Service
```

Every service logs:

```text
correlationId=abc123
```

Then we can search:

```bash
grep "abc123" application.log
```

and find logs related to that request.

---

# 26. Where should correlation ID be generated?

Typically:

```text
Gateway / first service
```

If the client already sends a valid correlation ID, the system may propagate it according to its trust/security policy.

Otherwise:

```java
UUID.randomUUID().toString()
```

can generate one.

---

# 27. MDC — how correlation ID gets into logs

SLF4J provides MDC.

Example:

```java
MDC.put("correlationId", correlationId);

try {
    processRequest();
} finally {
    MDC.remove("correlationId");
}
```

The logging configuration can include:

```text
correlationId
```

in every log line produced on that thread.

---

# 28. Implementing correlation ID in Spring Boot

A common approach is a Servlet filter.

```java
@Component
public class CorrelationIdFilter
        extends OncePerRequestFilter {

    private static final String HEADER =
            "X-Correlation-ID";

    @Override
    protected void doFilterInternal(
            HttpServletRequest request,
            HttpServletResponse response,
            FilterChain filterChain)
            throws ServletException, IOException {

        String correlationId =
                request.getHeader(HEADER);

        if (correlationId == null ||
            correlationId.isBlank()) {

            correlationId =
                    UUID.randomUUID().toString();
        }

        MDC.put("correlationId", correlationId);

        response.setHeader(HEADER, correlationId);

        try {
            filterChain.doFilter(request, response);
        } finally {
            MDC.remove("correlationId");
        }
    }
}
```

Flow:

```text
HTTP Request
     |
     v
Correlation Filter
     |
     +---- read/generate ID
     |
     +---- MDC.put()
     |
     v
Controller
     |
     v
Service
     |
     v
Repository
```

---

# 29. BIG trap — MDC and async execution

### Interview Question

**Will MDC automatically work when using CompletableFuture or ExecutorService?**

Not necessarily.

This is extremely important.

Suppose:

```java
MDC.put("correlationId", "abc123");

CompletableFuture.runAsync(() -> {

    log.info("Processing");

});
```

The async task may execute on another thread.

MDC is fundamentally associated with thread-local context, so the context may not automatically appear in the worker thread.

You may get:

```text
Main thread:
correlationId=abc123

Worker thread:
correlationId=null
```

---

# 30. How do we solve async context propagation?

Options include:

- explicitly propagate the context
- use a task decorator
- use context propagation mechanisms
- use tracing frameworks such as OpenTelemetry

For Spring executors, a `TaskDecorator` can copy relevant context.

Conceptually:

```text
Request Thread
      |
      | MDC = abc123
      |
      v
TaskDecorator
      |
      | copy context
      v
Worker Thread
      |
      | MDC = abc123
      v
Async task
```

This is a very good **Senior Engineer follow-up**.

---

# 31. Correlation ID vs Trace ID

These are related but not identical.

### Correlation ID

Application-level identifier used to correlate logs belonging to a request/workflow.

### Trace ID

Part of distributed tracing.

With OpenTelemetry:

```text
Trace
 |
 +-- Span: API Gateway
 |
 +-- Span: Payment Service
 |
 +-- Span: Ledger Service
 |
 +-- Span: DB call
```

A trace can show:

```text
Total request: 850 ms

Gateway:       20 ms
Payment:      300 ms
Ledger:       450 ms
DB:            80 ms
```

That's much more powerful than simply searching logs.

---

# 32. Modern production observability

A senior answer should mention the three pillars:

```text
          Observability
               |
       +-------+-------+
       |       |       |
      Logs  Metrics  Traces
       |       |       |
       v       v       v
   ELK/etc   Prometheus OpenTelemetry
                    |
                    v
                 Grafana
```

### Logs

Tell us:

> What happened?

### Metrics

Tell us:

> How is the system behaving?

### Traces

Tell us:

> Where did the request spend its time?

---

# 33. Production debugging scenario

### Interview Question

**Users say the Payment API is slow. How would you investigate?**

Don't answer:

> “I will check the logs.”

That's too shallow for a senior role.

A stronger answer:

```text
1. Check metrics
2. Check request latency
3. Check error rate
4. Check CPU/memory
5. Check DB connection pool
6. Check database query latency
7. Check external API latency
8. Check distributed traces
9. Correlate logs using correlation/trace ID
10. Identify bottleneck
```

For example:

```text
Grafana
   ↓
P95 latency increased
   ↓
OpenTelemetry trace
   ↓
DB call taking 900 ms
   ↓
DB metrics
   ↓
Connection pool exhausted
   ↓
Investigate slow query / pool configuration
```

That's a **much stronger senior-level answer**.

---

# 34. Production debugging scenario — application returns 200 but DB is down

### Interviewer:

**The application is running, but the database is unavailable. What happens?**

A good answer:

```text
Application process:
    Alive

Database:
    DOWN

Liveness:
    Should generally remain UP

Readiness:
    May become DOWN depending on whether DB is
    critical for serving requests

Traffic:
    Can be removed from instance if it cannot
    serve meaningful requests
```

Don't automatically restart the application just because the database is temporarily unavailable.

---

# 35. Production debugging scenario — application is completely stuck

Suppose:

```text
Deadlock
Infinite loop
Thread starvation
Application cannot process requests
```

The process technically exists, but it cannot function.

This is where liveness becomes useful.

```text
Liveness fails
      ↓
Kubernetes restarts pod
      ↓
Fresh application instance
```

---

# 36. Actuator security interview question

### Question

**Would you expose `/actuator/env` publicly?**

### Answer

No.

Actuator endpoints can reveal sensitive operational information.

We should:

```text
Expose only required endpoints
        +
Authenticate/authorize management endpoints
        +
Use network restrictions
        +
Avoid exposing secrets/configuration details
```

A common production configuration might expose only:

```properties
management.endpoints.web.exposure.include=health,info,metrics,prometheus
```

and then secure the management interface through application/network security.

---

# 37. Complete production observability flow

For your EPAM interview, remember this architecture:

```text
                       Kubernetes
                           |
                 +---------+---------+
                 |                   |
          Liveness Probe      Readiness Probe
                 |                   |
                 +---------+---------+
                           |
                    Spring Boot App
                           |
             +-------------+-------------+
             |             |             |
           Logs         Metrics        Traces
             |             |             |
             v             v             v
        Log Platform   Prometheus    OpenTelemetry
                           |             |
                           +------+------+
                                  |
                               Grafana
```

---

# 38. Rapid-fire interview questions

### Q1. What is Actuator?

Production monitoring and management capabilities for Spring Boot applications.

### Q2. What is `/actuator/health`?

An endpoint exposing application health status and health indicators.

### Q3. Liveness vs readiness?

```text
Liveness  → should I restart?
Readiness → should I send traffic?
```

### Q4. Can a pod be Running but not Ready?

Yes.

### Q5. Should DB failure necessarily fail liveness?

No. Usually DB failure should not cause the application to restart automatically.

### Q6. What is Micrometer?

A metrics instrumentation/facade library used by Spring Boot to expose application metrics to monitoring systems.

### Q7. Counter vs Gauge vs Timer?

```text
Counter → increasing count
Gauge   → current value
Timer   → duration/latency
```

### Q8. What is Prometheus?

A metrics collection and monitoring system that commonly scrapes application metrics.

### Q9. What is Grafana?

A visualization/dashboarding platform commonly used with Prometheus.

### Q10. What is structured logging?

Logging information in machine-readable structured fields, commonly JSON.

### Q11. What is a correlation ID?

An identifier used to correlate logs/events belonging to the same request or workflow across services.

### Q12. Does MDC automatically propagate to CompletableFuture threads?

No. Thread-local context may be lost when execution moves to another thread.

### Q13. Correlation ID vs trace ID?

Correlation ID is an application-level correlation mechanism; trace ID belongs to distributed tracing and can represent an entire distributed request trace.

### Q14. What are the three pillars of observability?

```text
Logs
Metrics
Traces
```

---

# 39. Senior scenario — Design observability for Payment Service

Suppose you have:

```text
POST /payments
```

Architecture:

```text
Client
  |
  v
Ingress
  |
  v
Payment Service
  |
  +---- MongoDB
  |
  +---- Kafka
  |
  +---- External Payment Provider
```

Your production observability should include:

### Logs

```text
paymentId
correlationId
traceId
event
status
error
```

### Metrics

```text
payments.created
payments.failed
payment_latency
external_provider_latency
Kafka consumer lag
DB connection pool
HTTP error rate
```

### Health

```text
/liveness
/readiness
```

### Tracing

```text
Ingress
  ↓
Payment API
  ↓
MongoDB
  ↓
Kafka
  ↓
External provider
```

Then if a payment takes 5 seconds:

```text
Metrics
   ↓
latency increased

Trace
   ↓
external provider = 4.5 seconds

Logs
   ↓
correlationId=abc123
```

Now we can diagnose the problem quickly.

---

# 40. The Senior Engineer answer pattern

When asked:

> “How do you monitor/debug a Spring Boot microservice in production?”

Answer in this order:

```text
1. Health
   → Actuator
   → liveness/readiness

2. Metrics
   → Micrometer
   → Prometheus
   → Grafana

3. Logs
   → SLF4J/Logback
   → structured logs
   → correlation ID

4. Tracing
   → OpenTelemetry
   → trace/span IDs

5. Kubernetes
   → probes
   → pod/container metrics
   → logs

6. Diagnosis
   → metrics identify symptom
   → traces identify bottleneck
   → logs explain exact failure
```

That answer demonstrates **production understanding**, rather than just knowing Spring annotations.

---

# 41. EPAM traps you should be ready for

### Trap 1

**“Application is UP, therefore it is ready.”**

Wrong.

```text
UP ≠ Ready
```

---

### Trap 2

**“Database is down, so restart the pod.”**

Not necessarily.

First determine whether the application itself is unhealthy or simply unable to serve because a dependency is unavailable.

---

### Trap 3

**“I use logs for everything.”**

Too weak.

Use:

```text
Logs + Metrics + Traces
```

---

### Trap 4

**“Correlation ID solves distributed tracing.”**

No.

Correlation helps correlate logs.

Distributed tracing gives request/span relationships and timing.

---

### Trap 5

**“MDC automatically works with CompletableFuture.”**

No.

Thread-local context may not automatically propagate.

---

### Trap 6

**“Expose all Actuator endpoints.”**

Bad production practice.

Expose and secure only what is needed.

---

### Trap 7

**“Put transactionId/userId/requestId as metric tags.”**

Potentially dangerous due to high cardinality.

---

# 42. One-page mental model

Memorize this:

```text
                    SPRING BOOT PRODUCTION
                              |
          +-------------------+-------------------+
          |                   |                   |
        HEALTH              METRICS             LOGS
          |                   |                   |
     Actuator             Micrometer          SLF4J
          |                   |                 Logback
     +----+----+              |                   |
     |         |              v                   v
 Liveness  Readiness      Prometheus       Structured Logs
     |         |              |                   |
 Restart   No Traffic       Grafana          Log Platform
                                                  |
                                                  v
                                           Correlation ID
                                                  |
                                                  v
                                             Debugging
                                                  |
                                                  v
                                             Tracing
                                                  |
                                                  v
                                           OpenTelemetry
```

**Core interview sentence:**

> “In production, I separate health from observability. Actuator provides health and management endpoints, liveness determines whether the instance should be restarted, readiness determines whether it should receive traffic, Micrometer provides metrics to systems such as Prometheus, structured logs with correlation IDs help investigate individual requests, and distributed tracing with OpenTelemetry helps identify where latency or failures occur across services.”

That is a **Senior Software Engineer-level answer**.


**Next exact roadmap topic:** Spring Boot **Testing — Unit Testing, Integration Testing, `@SpringBootTest`, MockMvc, Mockito, Repository Testing & Controller Testing**.