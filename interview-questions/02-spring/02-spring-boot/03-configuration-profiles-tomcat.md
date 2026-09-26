Absolutely. 🔥 This section is very interview-relevant because interviewers often connect **configuration → profiles → environment variables → secrets → production troubleshooting**, and then move into **Tomcat internals and thread exhaustion**.

I'll keep the answers **easy to remember and speak**, but include the depth a ~5-year Java/Spring candidate should know.

# Configuration & Profiles

## 36. `application.properties` vs `application.yml`?

### Interview answer

Both are used for configuring a Spring Boot application. The main difference is their **format and readability**.

### `application.properties`

```properties
server.port=8080
spring.datasource.url=jdbc:postgresql://localhost:5432/pf
spring.datasource.username=postgres
```

### `application.yml`

```yaml
server:
  port: 8080

spring:
  datasource:
    url: jdbc:postgresql://localhost:5432/pf
    username: postgres
```

YAML is hierarchical, so it can be easier to read when there are many related properties.

### Important

They are not different configuration mechanisms. Spring Boot can load both.

### Interview trap

Don't say:

> "YAML is better than properties."

It's mainly a **format/readability choice**.

---

# 37. How does Spring Boot load configuration?

🔥 This is more important than simply knowing where `application.yml` is.

### Interview answer

Spring Boot loads configuration from multiple **property sources** and combines them into its `Environment`.

Conceptually:

```text
application.yml
application.properties
       +
Profile-specific configuration
       +
Environment variables
       +
System properties
       +
Command-line arguments
       +
Other configured property sources
       ↓
Spring Environment
       ↓
@Value / @ConfigurationProperties
       ↓
Application
```

For example:

```yaml
server:
  port: 8080
```

Spring Boot makes that property available through the `Environment`.

You can access it using:

```java
@Value("${server.port}")
private int port;
```

or through:

```java
@ConfigurationProperties
```

### 🔥 Key concept

The **Environment** is Spring's abstraction for accessing configuration properties from different sources.

---

# 38. What is externalized configuration?

### Interview answer

Externalized configuration means:

> **Keeping configuration outside the application's compiled code so that the same application can run in different environments with different settings.**

For example, don't hardcode:

```java
String dbUrl = "jdbc:postgresql://prod-server:5432/pf";
```

Instead:

```yaml
spring:
  datasource:
    url: ${DB_URL}
```

Then production can provide:

```text
DB_URL=jdbc:postgresql://prod-server:5432/pf
```

while development can use:

```text
DB_URL=jdbc:postgresql://localhost:5432/pf
```

### Why?

The same JAR/image can be deployed to:

```text
DEV
 ↓
TEST
 ↓
PROD
```

with different configuration.

### 🔥 Remember

**Code stays same → configuration changes by environment.**

---

# 39. What is `@Value`?

### Interview answer

`@Value` is used to inject an individual configuration property into a Spring bean.

For example:

```yaml
app:
  name: PF-Service
  timeout: 5000
```

Then:

```java
@Value("${app.name}")
private String appName;

@Value("${app.timeout}")
private int timeout;
```

Spring resolves the property and injects the value.

### Default value

You can also specify a default:

```java
@Value("${app.timeout:3000}")
private int timeout;
```

Meaning:

```text
app.timeout exists → use it
app.timeout missing → use 3000
```

### 🔥 Common interview follow-up

Can `@Value` read environment variables?

Yes.

For example:

```java
@Value("${DB_URL}")
private String databaseUrl;
```

Spring's property resolution can resolve values supplied through environment variables/property sources.

---

# 40. What is `@ConfigurationProperties`?

### Interview answer

`@ConfigurationProperties` is used to bind a **group of related configuration properties** to a Java object.

Suppose:

```yaml
pf:
  batch:
    chunk-size: 500
    thread-count: 8
    retry-count: 3
```

We can create:

```java
@ConfigurationProperties(prefix = "pf.batch")
public class BatchProperties {

    private int chunkSize;
    private int threadCount;
    private int retryCount;

    // getters/setters
}
```

Now Spring binds:

```text
pf.batch.chunk-size
        ↓
chunkSize

pf.batch.thread-count
        ↓
threadCount

pf.batch.retry-count
        ↓
retryCount
```

### Why is this useful?

Instead of having:

```java
@Value("${pf.batch.chunk-size}")
@Value("${pf.batch.thread-count}")
@Value("${pf.batch.retry-count}")
...
```

we have one configuration object:

```java
BatchProperties
```

---

# 41. `@Value` vs `@ConfigurationProperties`?

🔥 Very common interview question.

| `@Value`                              | `@ConfigurationProperties`                  |
| ------------------------------------- | ------------------------------------------- |
| Good for individual properties        | Good for groups of properties               |
| String-based expressions              | Structured binding                          |
| Less type-safe                        | Better type-safe configuration model        |
| Convenient for small configuration    | Better for larger configuration             |
| SpEL can be used                      | Designed for external configuration binding |
| Can become messy with many properties | Cleaner for many related properties         |

### Example

For one property:

```java
@Value("${server.port}")
private int port;
```

Fine.

But for:

```yaml
pf:
  batch:
    chunk-size: 500
    thread-count: 8
    retry-count: 3
    timeout: 5000
    enabled: true
```

I'd prefer:

```java
@ConfigurationProperties(prefix = "pf.batch")
```

### Interview answer

> "`@Value` is convenient for injecting individual properties, while `@ConfigurationProperties` is better for binding a group of related configuration properties into a typed object."

### 🔥 For your project

For something like your batch configuration:

```yaml
pf:
  batch:
    chunk-size: 500
    partition-count: 8
    retry-limit: 3
```

`@ConfigurationProperties` is much cleaner.

---

# 42. How do you define custom configuration properties?

Suppose we want:

```yaml
pf:
  migration:
    enabled: true
    batch-size: 1000
    timeout: 30s
```

Create:

```java
@ConfigurationProperties(prefix = "pf.migration")
public class MigrationProperties {

    private boolean enabled;
    private int batchSize;
    private Duration timeout;

    // getters/setters
}
```

Then enable/register it.

One common approach is:

```java
@Configuration
@EnableConfigurationProperties(MigrationProperties.class)
public class Configuration {
}
```

Another approach in a Boot application is to use:

```java
@ConfigurationPropertiesScan
```

and let Spring scan for `@ConfigurationProperties` classes.

Then inject it:

```java
@Service
public class MigrationService {

    private final MigrationProperties properties;

    public MigrationService(MigrationProperties properties) {
        this.properties = properties;
    }
}
```

### Flow

```text
application.yml
      ↓
pf.migration.*
      ↓
@ConfigurationProperties
      ↓
MigrationProperties
      ↓
MigrationService
```

---

# 43. What are Spring Profiles?

### Interview answer

Spring Profiles allow us to define **environment-specific beans and configuration**.

For example:

```text
dev
test
prod
```

Different environments can have different:

* Database URLs
* Kafka configuration
* Log levels
* External service URLs
* Feature settings
* Beans

### Example

```yaml
# application-dev.yml

spring:
  datasource:
    url: jdbc:postgresql://localhost:5432/devdb
```

And:

```yaml
# application-prod.yml

spring:
  datasource:
    url: jdbc:postgresql://prod-db:5432/proddb
```

Only the configuration for the active profile is applied.

### Profiles can also control beans

```java
@Profile("dev")
@Bean
public PaymentClient mockPaymentClient() {
    return new MockPaymentClient();
}
```

This bean is available when:

```text
dev
```

is active.

---

# 44. How do you configure `application-dev.yml`, `application-test.yml`, `application-prod.yml`?

The usual structure is:

```text
src/main/resources/
│
├── application.yml
├── application-dev.yml
├── application-test.yml
└── application-prod.yml
```

### Base configuration

```yaml
# application.yml

server:
  port: 8080

spring:
  application:
    name: pf-service
```

### Development

```yaml
# application-dev.yml

spring:
  datasource:
    url: jdbc:postgresql://localhost:5432/pf_dev

logging:
  level:
    root: DEBUG
```

### Production

```yaml
# application-prod.yml

spring:
  datasource:
    url: jdbc:postgresql://prod-db:5432/pf

logging:
  level:
    root: INFO
```

When `prod` is active:

```text
application.yml
       +
application-prod.yml
       ↓
Effective configuration
```

The profile-specific configuration can override applicable base properties.

---

# 45. How do you activate a profile?

There are several ways.

### 1. Configuration file

```properties
spring.profiles.active=dev
```

### 2. Command line

```bash
java -jar app.jar --spring.profiles.active=prod
```

### 3. Environment variable

```bash
SPRING_PROFILES_ACTIVE=prod
```

This is particularly useful in containers/Kubernetes.

### 4. Programmatically

Spring also provides mechanisms to set active profiles programmatically, although external configuration is generally preferable for environment selection.

### Interview answer

> "A profile can be activated using `spring.profiles.active`, commonly through configuration, an environment variable, or a command-line argument. In production, I would generally keep the environment selection outside the application artifact."

---

# 46. How do you override configuration using environment variables?

Suppose your YAML has:

```yaml
server:
  port: 8080
```

An environment variable can provide:

```text
SERVER_PORT=9090
```

Spring Boot's relaxed binding maps:

```text
SERVER_PORT
```

to:

```text
server.port
```

Similarly:

```yaml
spring:
  datasource:
    url: ...
```

can be represented as:

```text
SPRING_DATASOURCE_URL=...
```

### Why useful?

You don't need to modify the JAR or Docker image.

```text
Same application image
       ↓
DEV environment variables
       ↓
TEST environment variables
       ↓
PROD environment variables
```

### Kubernetes example

Conceptually:

```text
Kubernetes Deployment
       ↓
Environment variables
       ↓
Spring Boot Environment
       ↓
Application
```

### 🔥 Important

Environment variables are one **property source**. The exact precedence among all possible property sources depends on Spring Boot's configuration ordering, but commonly supplied environment/system/command-line values can override values from packaged configuration files.

---

# 47. How should database passwords/secrets be managed?

🔥 Very important production question.

### Don't do this

```yaml
spring:
  datasource:
    password: MySuperSecretPassword
```

and commit it to Git.

### Better approach

Use a secret management mechanism.

For example:

```text
Environment variables
        or
Kubernetes Secrets
        or
Cloud secret manager
        or
Vault
        ↓
Spring Boot
```

Your application configuration can reference an externally supplied value:

```yaml
spring:
  datasource:
    password: ${DB_PASSWORD}
```

Then:

```text
DB_PASSWORD
```

is provided securely by the deployment environment.

### Production best practice

Secrets should generally be:

* Kept outside source code
* Kept outside Git
* Access-controlled
* Rotated periodically
* Audited
* Injected at runtime

### 🔥 Interview answer

> "I wouldn't hardcode database credentials in application properties committed to Git. I'd use an environment-specific secret mechanism such as Kubernetes Secrets, a cloud secret manager, or Vault, and inject the secret into the application at runtime."

---

# 48. What happens if the same property is defined in multiple places?

🔥 Very common follow-up.

Spring Boot has multiple property sources.

For example:

```text
application.yml
        ↓
Environment variable
        ↓
Command-line argument
```

If the same property appears in multiple sources, **the value from the higher-precedence source wins**.

Example:

```yaml
server:
  port: 8080
```

and:

```text
SERVER_PORT=9090
```

If the environment variable has higher precedence in that configuration situation:

```text
Effective value = 9090
```

### Simple mental model

```text
Multiple values
      ↓
Property precedence
      ↓
Higher precedence wins
      ↓
Effective configuration
```

### 🔥 Interview trap

Don't memorize an incomplete list and claim:

> "Environment variables always have the highest precedence."

Spring Boot has several property sources, including command-line arguments, environment/system properties, config files, test-related sources, and others.

The safe interview answer is:

> **"Spring Boot has a defined property-source precedence. When the same property is supplied from multiple sources, the higher-precedence source overrides the lower-precedence one."**

---

# 49. How would you troubleshoot configuration that works locally but fails in production?

🔥🔥 This is a **real production troubleshooting question**.

I'd go systematically.

### Step 1 — Check active profile

```text
What profile is active?
```

Maybe locally:

```text
dev
```

but production is accidentally running:

```text
test
```

Check:

```text
SPRING_PROFILES_ACTIVE
```

---

### Step 2 — Check effective configuration

Don't only inspect `application-prod.yml`.

Check what Spring actually receives.

Use appropriate logging/Actuator configuration diagnostics.

---

### Step 3 — Check environment variables

For example:

```text
DB_URL
DB_USERNAME
DB_PASSWORD
SPRING_DATASOURCE_URL
```

Maybe production has:

```text
DB_URL = wrong value
```

---

### Step 4 — Check property naming

For example:

```yaml
spring:
  datasource:
    url: ...
```

should map appropriately to:

```text
SPRING_DATASOURCE_URL
```

Check spelling carefully.

---

### Step 5 — Check secrets

Maybe:

```text
DB_PASSWORD
```

is missing from the production secret.

---

### Step 6 — Check external dependencies

Maybe configuration is correct but production cannot reach:

```text
PostgreSQL
Kafka
Redis
External API
```

Check:

* DNS
* Network
* Firewall/security groups
* Credentials
* TLS certificates
* Ports

---

### Step 7 — Check logs

Look for:

```text
Could not resolve placeholder
Connection refused
Authentication failed
Timeout
SSL errors
```

### Strong interview answer

> "First I'd verify the active profile and effective configuration. Then I'd check environment variables, secrets, property names, and property precedence. After that I'd verify connectivity and credentials for external dependencies such as the database, Kafka, or Redis. Finally I'd inspect startup logs and Actuator/configuration diagnostics to determine whether the issue is property resolution or an external dependency problem."

That's a **very practical answer**.

---

# 🟡 Embedded Server

Now let's move into Tomcat internals. 🔥

---

# 50. What is an embedded Tomcat?

### Interview answer

An embedded Tomcat is a Tomcat server packaged inside the Spring Boot application.

Instead of deploying:

```text
WAR
 ↓
External Tomcat
```

we can package:

```text
Spring Boot Application
+
Embedded Tomcat
 ↓
Executable JAR
```

Then:

```bash
java -jar application.jar
```

starts the application and its web server.

### Traditional approach

```text
WAR
 ↓
Install Tomcat
 ↓
Deploy WAR
```

### Spring Boot

```text
application.jar
      ↓
Embedded Tomcat
      ↓
Application starts
```

### Why useful?

Deployment becomes much simpler because the application carries its web server runtime.

---

# 51. How does Spring Boot start Tomcat?

🔥 This is where interviewers may go deeper.

When you have:

```xml
spring-boot-starter-web
```

Spring Boot brings in the servlet web stack and, by default in the standard setup, embedded Tomcat.

Then:

```java
SpringApplication.run(Application.class, args);
```

starts the Spring application.

Conceptually:

```text
SpringApplication.run()
        ↓
Create ApplicationContext
        ↓
Determine web application type
        ↓
Create web application context
        ↓
Auto-configuration
        ↓
Embedded web server factory
        ↓
Create/start Tomcat
        ↓
Register DispatcherServlet
        ↓
Application ready
```

### Important classes/concepts

You may hear about:

```text
ServletWebServerApplicationContext
TomcatServletWebServerFactory
TomcatWebServer
```

The exact internal call sequence varies by Spring Boot version, but the important architecture is:

```text
SpringApplication
       ↓
WebApplicationContext
       ↓
ServletWebServerFactory
       ↓
Tomcat
```

### 🔥 Interview answer

> "Spring Boot detects that the application is a servlet web application, creates the appropriate web application context, auto-configures the embedded servlet container, and starts the embedded Tomcat server."

---

# 52. What happens when an HTTP request reaches embedded Tomcat?

🔥🔥 Very important.

Suppose the client sends:

```http
GET /accounts/123
```

The high-level flow is:

```text
Client
  ↓
Tomcat
  ↓
Connector
  ↓
Request-processing thread
  ↓
Servlet container
  ↓
DispatcherServlet
  ↓
HandlerMapping
  ↓
Controller
  ↓
Service
  ↓
Repository/DB
  ↓
Response
  ↓
Tomcat
  ↓
Client
```

Let's understand it.

### 1. Client sends request

```text
GET /accounts/123
```

### 2. Tomcat receives it

Tomcat's connector handles network communication.

### 3. A request-processing thread handles the request

The request is processed by Tomcat's worker/request-processing infrastructure.

### 4. Request reaches Spring's `DispatcherServlet`

For a Spring MVC application:

```text
Tomcat
  ↓
DispatcherServlet
```

### 5. DispatcherServlet finds the controller

It uses handler mappings to determine which controller method should handle the request.

For example:

```java
@GetMapping("/accounts/{id}")
public Account getAccount(@PathVariable Long id) {
    ...
}
```

### 6. Controller calls service

```text
Controller
   ↓
Service
   ↓
Repository
   ↓
Database
```

### 7. Response travels back

```text
Database
   ↓
Service
   ↓
Controller
   ↓
DispatcherServlet
   ↓
Tomcat
   ↓
Client
```

---

# 53. How do you change Tomcat's port?

### `application.properties`

```properties
server.port=9090
```

or YAML:

```yaml
server:
  port: 9090
```

Now:

```text
http://localhost:9090
```

instead of:

```text
http://localhost:8080
```

### Environment variable

You can also configure:

```text
SERVER_PORT=9090
```

---

# 54. How do you configure Tomcat thread pools?

🔥 Important because this connects directly to **thread exhaustion**.

Spring Boot exposes server configuration properties.

For example, depending on your Spring Boot/Tomcat version:

```yaml
server:
  tomcat:
    threads:
      max: 200
      min-spare: 10
```

The important concept is:

```text
Tomcat
  ↓
Request-processing thread pool
  ↓
Maximum number of concurrent request-processing threads
```

### What does `max` mean?

It limits the number of Tomcat worker/request-processing threads that can process requests concurrently.

If:

```yaml
server:
  tomcat:
    threads:
      max: 200
```

you can have up to roughly that configured number of request-processing threads handling work concurrently, subject to the server's other connection/queue settings.

### Don't confuse threads with connections

🔥

```text
Connections ≠ Threads
```

You can have many network connections while only some requests are actively using worker threads.

---

# 55. What happens if all request-processing threads are busy?

Suppose:

```text
Tomcat max threads = 200
```

and all 200 are busy.

Now another request arrives.

It cannot immediately get a free worker thread.

Depending on connector/queue configuration:

```text
Request arrives
      ↓
No worker thread available
      ↓
Request waits in the connector's queue/backlog
      ↓
If capacity is exhausted
      ↓
Requests may be rejected / connection behavior depends on configuration
```

### Example

Imagine every request is doing:

```java
service.callVerySlowDatabase();
```

and takes 30 seconds.

You can get:

```text
200 threads
   ↓
200 slow requests
   ↓
All threads busy
   ↓
New requests wait
   ↓
Latency increases
   ↓
Eventually requests fail/time out
```

### 🔥 Important

Increasing Tomcat threads isn't always the real solution.

If the database is the bottleneck:

```text
More Tomcat threads
       ↓
More concurrent DB calls
       ↓
DB becomes even more overloaded
```

You need to find the actual bottleneck.

---

# 56. How would you troubleshoot Tomcat thread exhaustion?

🔥🔥 Very good production interview question.

I would investigate several things.

### Step 1 — Check thread metrics

Look at:

```text
Current threads
Busy threads
Maximum threads
Request count
Request latency
```

Actuator/metrics and monitoring systems can help.

---

### Step 2 — Take a thread dump

For example:

```bash
jstack <pid>
```

or use JVM diagnostic tooling appropriate to the environment.

Look for many Tomcat threads blocked/waiting in the same place.

Example:

```text
http-nio-8080-exec-1
http-nio-8080-exec-2
http-nio-8080-exec-3
...
```

If they're all waiting on:

```text
Database
HTTP call
Redis
Lock
```

you have a clue.

---

### Step 3 — Check request latency

Maybe an endpoint normally takes:

```text
100 ms
```

but now:

```text
10 seconds
```

That means threads remain occupied much longer.

---

### Step 4 — Check downstream dependencies

Look at:

```text
Database connection pool
Kafka
Redis
External APIs
```

For example:

```text
Tomcat threads = 200
DB pool = 20
```

You might have many request threads waiting for DB connections.

---

### Step 5 — Check application code

Look for:

* Slow queries
* Missing indexes
* External calls
* Deadlocks
* Synchronization/locks
* Blocking operations
* Large processing inside HTTP requests

### Step 6 — Check thread pool configuration

Don't immediately increase the maximum.

Understand:

```text
Why are existing threads staying busy?
```

### Strong interview answer

> "I'd check Tomcat busy-thread metrics, request latency, thread dumps, database connection pool utilization, downstream dependency latency, and blocked threads. I'd identify why requests are holding threads for a long time before increasing the thread pool size."

🔥 That's much stronger than:

> "Increase Tomcat max threads."

---

# 57. How would you configure connection timeout?

There are actually **multiple types of timeout**, so be careful in interviews.

For Tomcat/server connections, Spring Boot exposes server timeout configuration such as:

```yaml
server:
  connection-timeout: 20s
```

This concerns how long the server waits for a connection/request-related operation under the server's connection handling.

But don't confuse this with:

```text
Database connection timeout
HTTP client connection timeout
HTTP read timeout
Kafka timeout
```

Those are configured separately.

### Example

```text
Browser
   ↓
Tomcat
   ↓
Connection timeout

Application
   ↓
PostgreSQL
   ↓
DB connection/operation timeout

Application
   ↓
External REST API
   ↓
HTTP connect/read timeout
```

### 🔥 Interview trap

If interviewer says:

> "Set timeout."

Ask yourself:

**Which timeout?**

There are several layers.

---

# 58. What is graceful shutdown in Spring Boot?

### Interview answer

Graceful shutdown means:

> **When the application is shutting down, it stops accepting new requests while allowing existing requests to complete within a configured grace period.**

Without graceful shutdown:

```text
Shutdown signal
     ↓
Application stops
     ↓
In-flight requests may be interrupted
```

With graceful shutdown:

```text
Shutdown signal
     ↓
Stop accepting new requests
     ↓
Existing requests continue
     ↓
Wait for completion
     ↓
Shutdown application
```

### Configuration

In Spring Boot, graceful shutdown can be enabled/configured using:

```yaml
server:
  shutdown: graceful
```

You can also configure a shutdown grace period using the appropriate lifecycle timeout setting, for example:

```yaml
spring:
  lifecycle:
    timeout-per-shutdown-phase: 30s
```

---

# 59. What happens to in-flight requests during graceful shutdown?

🔥 Important follow-up.

Suppose:

```text
Request A → 2 seconds remaining
Request B → 10 seconds remaining
Request C → 60 seconds remaining
```

and shutdown starts.

Conceptually:

```text
Shutdown signal
       ↓
Server stops accepting new work
       ↓
A continues
B continues
C continues
       ↓
Wait up to configured shutdown timeout
       ↓
Completed requests finish normally
       ↓
Application shuts down
```

If a request doesn't finish within the grace period, shutdown eventually continues.

### Example

```yaml
server:
  shutdown: graceful

spring:
  lifecycle:
    timeout-per-shutdown-phase: 30s
```

If a request needs 5 seconds:

```text
Shutdown
 ↓
Request continues
 ↓
5 sec
 ↓
Request completes
```

If another request needs 60 seconds:

```text
Shutdown
 ↓
Request continues
 ↓
30 sec grace period
 ↓
Shutdown proceeds
```

### Why is this important in production?

Imagine Kubernetes is deploying a new version:

```text
Old Pod
   ↓
SIGTERM
   ↓
Graceful shutdown
   ↓
Existing requests finish
   ↓
Pod terminates
```

Without graceful shutdown, users can see failed/incomplete requests during deployments.

---

# 🔥 Embedded Tomcat — Complete Mental Model

This is the part I'd make sure you can draw on a whiteboard:

```text
                    CLIENT
                       │
                       ▼
                ┌─────────────┐
                │   Tomcat    │
                │  Connector  │
                └──────┬──────┘
                       │
                       ▼
              Request Thread Pool
                       │
                       ▼
              DispatcherServlet
                       │
                 HandlerMapping
                       │
                       ▼
                  Controller
                       │
                       ▼
                   Service
                       │
                       ▼
                Repository/DB
                       │
                       ▼
                   Response
                       │
                       ▼
                    Tomcat
                       │
                       ▼
                    CLIENT
```

And the most important production problem:

```text
                    Slow DB
                       ↑
                       │
Tomcat threads → waiting → waiting → waiting
                       │
                       ↓
                 Threads exhausted
                       │
                       ↓
               New requests wait
                       │
                       ↓
                 Latency increases
                       │
                       ↓
                  Timeouts/errors
```

So when you hear **"Tomcat thread exhaustion"**, immediately think:

> **"Why are request threads staying occupied for too long?"**

—not simply:

> **"Increase the thread count."**

---

# 🧠 Rapid-Fire Revision

### Configuration

```text
properties vs YAML
→ Different formats, same configuration purpose

Environment
→ Central abstraction for configuration/property sources

Externalized configuration
→ Keep config outside code/artifact

@Value
→ Individual property

@ConfigurationProperties
→ Group of typed properties

Profile
→ Environment-specific configuration/beans

application-dev.yml
application-test.yml
application-prod.yml
→ Selected through active profile

Environment variable
→ Can override lower-precedence configuration

Secrets
→ Secret manager / Kubernetes Secret / environment injection

Same property multiple places
→ Higher-precedence property source wins

Local works, prod fails
→ Check profile → effective config → env vars → secrets →
  precedence → connectivity → logs
```

### Embedded Tomcat

```text
Embedded Tomcat
→ Tomcat packaged with application

SpringApplication.run()
→ Starts Spring context + embedded web server

HTTP request
→ Tomcat → thread → DispatcherServlet → Controller → Service → DB

server.port
→ Change port

Tomcat threads
→ Process requests concurrently

All threads busy
→ Requests wait/reject depending on capacity/configuration

Thread exhaustion
→ Check metrics + thread dump + latency + DB/downstream dependencies

Connection timeout
→ Server connection timeout; don't confuse with DB/HTTP-client timeouts

Graceful shutdown
→ Stop new work, allow existing work to finish within grace period
```

## ⭐ The 5 questions I'd expect an interviewer to chain together

```text
What is auto-configuration?
        ↓
How does Boot know what to configure?
        ↓
Where does configuration come from?
        ↓
How do profiles/environment variables override it?
        ↓
What happens in production when configuration is wrong?
```

And for Tomcat:

```text
How does Tomcat process a request?
        ↓
How many threads can process requests?
        ↓
What happens when all are busy?
        ↓
How do you diagnose thread exhaustion?
        ↓
What happens during graceful shutdown?
```

If you can explain those two chains naturally, you have the **Spring Boot fundamentals at a much deeper interview level**, rather than just memorized annotations.
