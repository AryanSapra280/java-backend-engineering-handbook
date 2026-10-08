# Spring Boot — Embedded Server & Application Lifecycle ⭐⭐⭐⭐⭐

This is a **very common senior-level Spring Boot interview area** because interviewers use it to check whether you understand what actually happens when a Spring Boot application starts.

---

# 1. What is an Embedded Server?

### Interview Question
**What is an embedded server in Spring Boot?**

### Interview-ready answer

An embedded server is a web server that is packaged **inside the Spring Boot application itself**, instead of requiring us to deploy the application to an externally installed application server.

For a typical Spring Boot MVC application:

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
```

Spring Boot brings an embedded servlet container, commonly **Tomcat**.

Therefore, we can run:

```bash
java -jar payment-service.jar
```

and the application starts its own web server.

Without embedded servers, traditionally we might:

```text
Build WAR
   ↓
Deploy WAR
   ↓
External Tomcat
   ↓
Application starts
```

With Spring Boot:

```text
java -jar application.jar
        ↓
Spring Boot
        ↓
Embedded Tomcat
        ↓
Spring ApplicationContext
        ↓
REST APIs available
```

---

# 2. Why does Spring Boot use an Embedded Server?

### Interview Question

**Why does Spring Boot use embedded servers?**

### Answer

The main advantages are:

1. **Simple deployment**
2. Application is self-contained
3. No need to separately install/configure Tomcat
4. Easy containerization
5. Same application artifact can run across environments
6. Convenient for microservices

For example:

```bash
docker run payment-service
```

The container contains the application, and the application contains its embedded server.

This fits very well with:

```text
Microservice
   ↓
Executable JAR
   ↓
Docker Image
   ↓
Kubernetes Pod
```

---

# 3. Is Tomcat part of Spring?

### Interview Question

**Is Tomcat a part of Spring Framework?**

### Answer

No.

Tomcat is a **Servlet container/web server**.

Spring Framework provides things such as:

```text
IoC Container
Dependency Injection
AOP
Spring MVC
Transactions
etc.
```

Tomcat provides the HTTP/Servlet runtime.

The relationship is approximately:

```text
Client
  ↓
Tomcat
  ↓
Servlet
  ↓
DispatcherServlet
  ↓
Spring MVC
  ↓
Controller
  ↓
Service
  ↓
Repository
```

This distinction is important.

---

# 4. What exactly does Tomcat do?

Suppose the client sends:

```http
GET /payments/123
```

The request first reaches Tomcat.

Tomcat handles the underlying HTTP/server-side servlet processing and eventually invokes the appropriate servlet.

For Spring MVC, that servlet is:

```text
DispatcherServlet
```

So:

```text
HTTP Request
     ↓
Tomcat
     ↓
DispatcherServlet
     ↓
Spring MVC
     ↓
Controller
```

### Important interview distinction

**Tomcat does not directly call your `PaymentController`.**

It invokes the `DispatcherServlet`.

Spring MVC then determines which controller method should handle the request.

---

# 5. What is DispatcherServlet?

### Interview Question

**What is DispatcherServlet?**

### Interview-ready answer

`DispatcherServlet` is the **front controller of Spring MVC**.

It receives incoming HTTP requests and delegates them to the appropriate Spring MVC components.

For example:

```http
GET /payments/123
```

The flow is roughly:

```text
Client
  ↓
Tomcat
  ↓
DispatcherServlet
  ↓
HandlerMapping
  ↓
PaymentController.getPayment()
  ↓
PaymentService
  ↓
Repository
```

The `DispatcherServlet` itself does not contain all the business logic.

It coordinates request processing.

---

# 6. What happens when `SpringApplication.run()` executes?

This is one of the **most important questions**.

Suppose we have:

```java
@SpringBootApplication
public class PaymentApplication {

    public static void main(String[] args) {
        SpringApplication.run(PaymentApplication.class, args);
    }
}
```

### Interview Question

**What happens internally when SpringApplication.run() is called?**

### Interview-ready answer

At a high level:

```text
main()
  ↓
SpringApplication.run()
  ↓
Prepare Environment
  ↓
Determine Application Type
  ↓
Create ApplicationContext
  ↓
Load Configuration / Bean Definitions
  ↓
Refresh ApplicationContext
  ↓
Create Beans
  ↓
Start Embedded Server
  ↓
ApplicationStartedEvent
  ↓
ApplicationRunner / CommandLineRunner
  ↓
ApplicationReadyEvent
```

Let's understand each step.

---

# 7. Step 1 — `main()` starts the JVM application

Java starts execution from:

```java
public static void main(String[] args)
```

Then:

```java
SpringApplication.run(PaymentApplication.class, args);
```

is invoked.

---

# 8. Step 2 — SpringApplication is prepared

Spring Boot creates/configures a `SpringApplication`.

Conceptually:

```java
SpringApplication application =
        new SpringApplication(PaymentApplication.class);

application.run(args);
```

Spring Boot determines things such as:

- application type
- sources
- initializers
- listeners
- environment configuration

---

# 9. Step 3 — Environment is prepared

Spring Boot creates the `Environment`.

The environment contains configuration from various property sources.

For example:

```properties
server.port=8080
spring.datasource.url=...
```

Configuration can come from:

```text
application.properties
application.yml
Environment variables
System properties
Command-line arguments
External configuration
etc.
```

So the application gets its runtime configuration before the context is fully initialized.

---

# 10. Step 4 — Spring determines the application type

Spring Boot determines whether the application is:

```text
NONE
SERVLET
REACTIVE
```

For a typical:

```xml
spring-boot-starter-web
```

application, it is a **Servlet web application**.

For:

```xml
spring-boot-starter-webflux
```

it is a reactive application.

This affects which type of ApplicationContext and web infrastructure Spring Boot creates.

---

# 11. Step 5 — ApplicationContext is created

Spring creates the appropriate `ApplicationContext`.

For a normal Spring Boot MVC application, we have a web-aware application context.

You can think of it as:

```text
ApplicationContext
        +
Web application integration
        ↓
WebApplicationContext
```

---

# 12. ApplicationContext vs WebApplicationContext

### Interview Question

**What is the difference between ApplicationContext and WebApplicationContext?**

### ApplicationContext

`ApplicationContext` is Spring's IoC container.

It manages:

```text
Beans
Dependency Injection
Configuration
Lifecycle
Events
etc.
```

### WebApplicationContext

`WebApplicationContext` is the web-aware form used by Spring MVC applications.

It provides integration with the web/servlet environment.

For example:

```text
Servlet container
       ↓
Spring MVC
       ↓
WebApplicationContext
```

### Interview-ready answer

> `ApplicationContext` is the general-purpose Spring IoC container, whereas `WebApplicationContext` extends the concept for web applications and integrates Spring's application context with the servlet/web environment.

---

# 13. Step 6 — Configuration and Bean Definitions are loaded

Spring Boot identifies configuration and bean definitions.

For example:

```java
@Service
public class PaymentService {
}
```

```java
@Repository
public class PaymentRepository {
}
```

```java
@RestController
public class PaymentController {
}
```

Component scanning discovers these components.

Additionally:

```java
@Configuration
public class PaymentConfig {

    @Bean
    public PaymentClient paymentClient() {
        return new PaymentClient();
    }
}
```

registers the `PaymentClient` bean.

At this stage Spring builds the metadata needed to create/manage the beans.

---

# 14. Step 7 — ApplicationContext is refreshed

This is a **very important lifecycle step**.

Conceptually:

```java
applicationContext.refresh();
```

The refresh process initializes the Spring container.

This is where many important things happen:

```text
BeanFactory initialization
        ↓
Bean definitions processed
        ↓
BeanPostProcessors registered
        ↓
Singleton beans created
        ↓
Dependencies injected
        ↓
@PostConstruct
        ↓
Initialization callbacks
```

This connects directly to the Spring Core lifecycle we studied earlier.

---

# 15. When are singleton beans created?

For normal singleton beans, Spring generally creates them during context initialization.

For example:

```java
@Service
public class PaymentService {
    
    @PostConstruct
    public void init() {
        System.out.println("PaymentService initialized");
    }
}
```

During startup:

```text
ApplicationContext refresh
       ↓
PaymentService created
       ↓
Dependencies injected
       ↓
@PostConstruct
       ↓
Bean ready
```

---

# 16. Where does the Embedded Tomcat start?

For a Servlet-based Spring Boot application, the embedded web server is initialized during the application context refresh process.

Conceptually:

```text
ApplicationContext refresh
        ↓
WebApplicationContext
        ↓
Create embedded WebServer
        ↓
Tomcat starts
        ↓
DispatcherServlet registered
```

This is why the server becomes available as part of application startup.

---

# 17. How does Spring Boot know to start Tomcat?

This is where the earlier topic of **auto-configuration** connects.

You have:

```xml
spring-boot-starter-web
```

which brings the required web dependencies.

Spring Boot's auto-configuration detects the servlet web environment and configures the embedded web server.

So the chain is approximately:

```text
spring-boot-starter-web
        ↓
Spring MVC + Tomcat dependencies
        ↓
Auto-configuration
        ↓
Embedded Tomcat configured
        ↓
Tomcat starts
```

---

# 18. What happens inside Tomcat?

At a useful interview level:

```text
Tomcat
  │
  ├── Connector
  │      ↓
  │   Accepts HTTP connections
  │
  └── Servlet processing
          ↓
      DispatcherServlet
```

The connector listens on a port such as:

```text
8080
```

For:

```http
GET /payments/123
```

the request enters Tomcat and eventually reaches:

```text
DispatcherServlet
```

Spring MVC then processes it.

---

# 19. How does `DispatcherServlet` find the Controller?

Suppose:

```java
@GetMapping("/payments/{id}")
public Payment getPayment(@PathVariable String id) {
    ...
}
```

The request:

```http
GET /payments/123
```

goes through roughly:

```text
Tomcat
   ↓
DispatcherServlet
   ↓
HandlerMapping
   ↓
PaymentController
```

`HandlerMapping` helps identify the controller/handler that matches the request.

Then Spring MVC invokes the method.

---

# 20. Important: Tomcat ≠ DispatcherServlet

This is a common interview trap.

### Wrong:

> DispatcherServlet is the web server.

### Correct:

> Tomcat is the servlet container/web server, while DispatcherServlet is a Spring MVC servlet that acts as the front controller.

Architecture:

```text
                 Spring
                   │
            DispatcherServlet
                   │
             Spring MVC
                   │
              Controller

                 ↑
              Tomcat
          Servlet Container
                 ↑
              HTTP
```

---

# 21. Spring Boot Application Lifecycle Events

Spring Boot publishes several lifecycle events.

A useful interview-level sequence is:

```text
ApplicationStartingEvent
        ↓
ApplicationEnvironmentPreparedEvent
        ↓
ApplicationContextInitializedEvent
        ↓
ApplicationPreparedEvent
        ↓
ApplicationContext refresh
        ↓
WebServerInitializedEvent
        ↓
ApplicationStartedEvent
        ↓
ApplicationRunner / CommandLineRunner
        ↓
ApplicationReadyEvent
```

If startup fails:

```text
ApplicationFailedEvent
```

may be published.

You do not need to memorize every internal implementation class, but you **should understand the important lifecycle events and their timing**.

---

# 22. ApplicationStartingEvent

Published very early in startup.

At this point Spring Boot is beginning the application startup process.

It happens before the environment/context is fully prepared.

Think:

```text
Application starting...
```

---

# 23. ApplicationEnvironmentPreparedEvent

This occurs after the `Environment` has been prepared.

Conceptually:

```text
Configuration sources
        ↓
Environment prepared
        ↓
ApplicationEnvironmentPreparedEvent
```

This is relevant when startup logic depends on configuration/environment setup.

---

# 24. ApplicationPreparedEvent

This indicates that the application context has been prepared but has not yet completed its refresh.

Conceptually:

```text
Environment ready
      ↓
Context prepared
      ↓
ApplicationPreparedEvent
      ↓
Context refresh
```

---

# 25. WebServerInitializedEvent

For web applications, Spring Boot publishes a web-server initialization event after the embedded web server has been initialized.

For example:

```text
Embedded Tomcat
      ↓
initialized
      ↓
WebServerInitializedEvent
```

This is particularly relevant when working with the actual web server configuration/port.

---

# 26. ApplicationStartedEvent

### Very important interview question

**What is the difference between ApplicationStartedEvent and ApplicationReadyEvent?**

`ApplicationStartedEvent` occurs after the application context has been refreshed and the application has started, but **before** application runners execute.

Conceptually:

```text
Context refresh
      ↓
ApplicationStartedEvent
      ↓
Runners
```

---

# 27. CommandLineRunner

Suppose:

```java
@Component
public class StartupRunner implements CommandLineRunner {

    @Override
    public void run(String... args) {
        System.out.println("Startup logic");
    }
}
```

Spring Boot executes it during startup.

Flow:

```text
ApplicationStartedEvent
        ↓
CommandLineRunner
        ↓
ApplicationReadyEvent
```

---

# 28. ApplicationRunner

`ApplicationRunner` is similar to `CommandLineRunner`.

Difference:

### CommandLineRunner

```java
void run(String... args)
```

Arguments are raw strings.

### ApplicationRunner

```java
void run(ApplicationArguments args)
```

Arguments are parsed into a more structured representation.

Example:

```java
@Component
public class StartupRunner implements ApplicationRunner {

    @Override
    public void run(ApplicationArguments args) {
        System.out.println(args.getOptionNames());
    }
}
```

---

# 29. CommandLineRunner vs ApplicationRunner

| Feature | CommandLineRunner | ApplicationRunner |
|---|---|---|
| Method | `run(String...)` | `run(ApplicationArguments)` |
| Arguments | Raw | Parsed |
| Use | Simple startup logic | Structured command-line arguments |

### Interview answer

> Both are startup callbacks executed after the application context has started but before `ApplicationReadyEvent`. `ApplicationRunner` provides parsed `ApplicationArguments`, while `CommandLineRunner` receives raw String arguments.

---

# 30. Can we have multiple runners?

Yes.

For example:

```java
@Component
@Order(1)
class DatabaseInitializer implements CommandLineRunner {

    @Override
    public void run(String... args) {
        // ...
    }
}
```

```java
@Component
@Order(2)
class CacheInitializer implements CommandLineRunner {

    @Override
    public void run(String... args) {
        // ...
    }
}
```

Ordering can be controlled using:

```java
@Order
```

or appropriate ordering mechanisms.

---

# 31. ApplicationReadyEvent

This is one of the most important lifecycle events for production systems.

`ApplicationReadyEvent` indicates that startup has progressed through the runners and the application is considered ready.

Conceptually:

```text
Context ready
     ↓
ApplicationStartedEvent
     ↓
Startup runners
     ↓
ApplicationReadyEvent
```

For example:

```java
@Component
public class ReadyListener {

    @EventListener
    public void onReady(ApplicationReadyEvent event) {
        System.out.println("Application is ready");
    }
}
```

---

# 32. Why is ApplicationReadyEvent important?

Imagine Kubernetes.

You don't simply want:

```text
JVM started
```

to mean:

```text
Application ready
```

The application may still be:

```text
Loading beans
Initializing caches
Running startup tasks
Connecting to required dependencies
```

Therefore readiness should represent the point at which the application is actually capable of serving traffic.

This connects directly to:

```text
Kubernetes readiness probe
```

which we'll cover later in the Microservices/Kubernetes section.

---

# 33. ApplicationFailedEvent

What happens if startup fails?

For example:

```text
Database configuration invalid
Bean creation failure
Port already in use
Invalid configuration
Missing required dependency
```

Spring Boot can publish:

```text
ApplicationFailedEvent
```

The application may then terminate instead of becoming ready.

---

# 34. Complete lifecycle

For interview purposes, remember this model:

```text
main()
  │
  ▼
SpringApplication.run()
  │
  ▼
Create/configure SpringApplication
  │
  ▼
Prepare Environment
  │
  ▼
Determine application type
  │
  ▼
Create ApplicationContext
  │
  ▼
Prepare Context
  │
  ▼
Load Bean Definitions
  │
  ▼
Refresh Context
  │
  ├── Create Beans
  ├── Dependency Injection
  ├── @PostConstruct
  ├── Bean initialization
  └── Embedded Web Server
          │
          ▼
   Tomcat initialized
          │
          ▼
ApplicationStartedEvent
          │
          ▼
ApplicationRunner
CommandLineRunner
          │
          ▼
ApplicationReadyEvent
          │
          ▼
Application serving requests
```

If something fails:

```text
ApplicationFailedEvent
```

---

# 35. What if the port is already occupied?

### Interview Question

Suppose another application is already running on port 8080. What happens?

You have:

```properties
server.port=8080
```

and another process already occupies it.

Embedded Tomcat attempts to bind to:

```text
8080
```

but the operating system rejects the bind.

Startup fails.

You may see an error such as:

```text
Web server failed to start.
Port 8080 was already in use.
```

The application does not become ready.

This is a very common real-world startup failure.

---

# 36. How do you change the port?

```properties
server.port=8081
```

or:

```yaml
server:
  port: 8081
```

You can also override configuration externally, which is especially useful in containers.

For example:

```bash
java -jar payment-service.jar --server.port=9090
```

This is an example of Spring Boot's externalized configuration.

---

# 37. Production scenario — Application starts but API returns 404

Suppose:

```text
Tomcat started successfully
ApplicationReadyEvent fired
```

but:

```http
GET /payments/123
```

returns:

```text
404
```

Does that mean Tomcat is broken?

**No.**

Tomcat may be perfectly healthy.

Possible causes include:

```text
Wrong URL
Wrong context path
Controller not scanned
Wrong @RequestMapping
Wrong HTTP method
Controller bean not created
API version mismatch
Gateway routing issue
```

The flow is:

```text
Request
   ↓
Tomcat
   ↓
DispatcherServlet
   ↓
HandlerMapping
   ↓
No matching controller
   ↓
404
```

This is a good senior-level debugging distinction.

---

# 38. Production scenario — Application starts but database is unavailable

This is more interesting.

Suppose the application starts:

```text
JVM
 ↓
Spring Boot
 ↓
Beans
 ↓
Tomcat
 ↓
ApplicationReadyEvent
```

but database connectivity is not immediately required during startup.

Then the application may technically become ready while database-dependent requests fail.

This raises an important architecture question:

> Should readiness mean only "HTTP server is running", or should it mean "application can perform its critical responsibilities"?

The answer depends on the service.

For a payment service, if the database is absolutely required to process payments, readiness may need to reflect that dependency appropriately.

But you should avoid blindly making every dependency a hard readiness dependency, because one optional dependency failure could unnecessarily remove all instances from service.

This is why health/readiness design matters.

---

# 39. Graceful Shutdown

Now consider production Kubernetes.

A pod receives:

```text
SIGTERM
```

because Kubernetes wants to terminate the pod.

You don't want:

```text
Request in progress
        ↓
Process killed immediately
        ↓
Request fails
```

Instead, graceful shutdown allows the application to stop accepting new work and finish in-flight work within the configured shutdown window.

Spring Boot supports graceful shutdown for supported web servers.

Conceptually:

```text
SIGTERM
   ↓
Spring application shutdown
   ↓
Stop accepting new requests
   ↓
Allow active requests to finish
   ↓
Destroy beans/resources
   ↓
Process exits
```

Spring Boot configuration can enable graceful shutdown:

```properties
server.shutdown=graceful
```

The exact timing should also be coordinated with the Kubernetes pod's termination grace period.

---

# 40. Why is graceful shutdown important in microservices?

Imagine:

```text
Load Balancer
      ↓
Pod A
Pod B
Pod C
```

Kubernetes wants to terminate Pod B.

If Pod B immediately dies:

```text
Request → Pod B → connection reset
```

But with graceful shutdown:

```text
Pod B receives termination
       ↓
Pod B stops taking new traffic
       ↓
Existing requests finish
       ↓
Pod B terminates
```

This greatly reduces failed requests during deployments/scaling.

---

# 41. Senior Interview Question — Is ApplicationReadyEvent the same as Tomcat starting?

### Answer

No.

They are related but not identical.

The embedded web server is initialized during the application context startup process.

Then Spring Boot continues through the startup lifecycle.

Conceptually:

```text
Context refresh
      ↓
Web server initialized
      ↓
ApplicationStartedEvent
      ↓
Runners
      ↓
ApplicationReadyEvent
```

Therefore:

> **Tomcat being initialized does not by itself mean the entire Spring Boot startup lifecycle has completed.**

---

# 42. Senior Interview Question — Where should startup initialization logic go?

Suppose you need to load some reference data when the application starts.

Options include:

```text
@PostConstruct
CommandLineRunner
ApplicationRunner
ApplicationReadyEvent
```

The choice depends on what you're initializing.

### `@PostConstruct`

Good for bean-local initialization.

```java
@PostConstruct
public void init() {
    // initialize bean
}
```

### CommandLineRunner/ApplicationRunner

Good for application startup tasks.

```java
@Override
public void run(String... args) {
    // startup task
}
```

### ApplicationReadyEvent

Useful when you specifically want logic after the application has reached the ready stage.

```java
@EventListener
public void ready(ApplicationReadyEvent event) {
    // ...
}
```

### Important production consideration

Don't put a huge blocking operation into startup unless you intentionally want startup to wait for it.

For example:

```java
@Override
public void run(String... args) {
    load10MillionRecords();
}
```

could significantly delay readiness.

In a Kubernetes environment this can affect:

```text
startup time
readiness
deployment rollout
autoscaling
pod replacement
```

---

# 43. Practical EPAM Question

### Interviewer:

> Your Spring Boot application is running in Kubernetes. The pod is `Running`, but traffic should not be sent until your application is actually ready. What would you do?

### Strong answer

I would separate **liveness** and **readiness**.

```text
Liveness
→ Is the application process fundamentally alive?

Readiness
→ Should this instance receive traffic?
```

Spring Boot Actuator can expose health endpoints, and Kubernetes can use them for probes.

Conceptually:

```text
Kubernetes
    │
    ├── Liveness Probe
    │
    └── Readiness Probe
              ↓
        Spring Boot Actuator
```

Then:

```text
Pod starts
   ↓
Application initialization
   ↓
Not Ready
   ↓
Initialization complete
   ↓
Ready
   ↓
Traffic starts
```

This is much better than assuming:

```text
container started = application ready
```

---

# 44. Practical EPAM Question

### Interviewer:

> What happens if `ApplicationRunner` throws an exception?

### Answer

The startup process fails because the runner is part of the startup sequence before `ApplicationReadyEvent`.

Conceptually:

```text
ApplicationStartedEvent
       ↓
ApplicationRunner
       ↓
Exception
       ↓
Startup failure
       ↓
ApplicationReadyEvent is not reached
```

This is why critical startup initialization should be designed carefully.

---

# 45. Practical EPAM Question

### Interviewer:

> Can I make a REST API available before all Spring beans are initialized?

### Answer

Normally, no.

The Spring application context needs to complete its startup process sufficiently for the web application to become ready.

Spring Boot's lifecycle is designed around:

```text
Initialize application
      ↓
Initialize context
      ↓
Initialize web server
      ↓
Run startup callbacks
      ↓
ApplicationReadyEvent
```

There are specialized asynchronous/background initialization patterns, but you should not casually treat them as a way to bypass normal application readiness.

---

# 46. Practical EPAM Question

### Interviewer:

> Is Spring Boot's embedded Tomcat multithreaded?

### Answer

Yes.

Tomcat uses a thread pool to process concurrent requests.

Conceptually:

```text
             Tomcat
                │
        ┌───────┼────────┐
        ↓       ↓        ↓
     Thread  Thread   Thread
        │       │        │
        ↓       ↓        ↓
    Request1 Request2 Request3
```

This is one reason your Spring service must be designed with thread safety in mind.

For example, this is dangerous:

```java
@Service
public class PaymentService {

    private int counter;

    public void process() {
        counter++;
    }
}
```

Because Spring singleton beans are typically shared across request threads.

So:

```text
Singleton bean
      +
Multiple request threads
      =
Potential race conditions
```

This directly connects to the Java concurrency topics you are preparing.

---

# 47. Very Important Senior-Level Connection

You should now connect these concepts:

```text
Spring Singleton
       ↓
Shared across requests
       ↓
Tomcat worker threads
       ↓
Concurrent execution
       ↓
Thread safety required
```

Therefore, when an interviewer asks:

> "Are Spring singleton beans thread-safe?"

The answer is:

**No. Singleton scope controls bean lifecycle/scope, not thread safety.**

If the bean only contains stateless dependencies:

```java
@Service
public class PaymentService {

    public Payment process(PaymentRequest request) {
        ...
    }
}
```

it is usually safe.

But shared mutable state requires synchronization/concurrency-safe design.

---

# 48. Final Interview Cheat Flow

If the interviewer says:

> **"Explain what happens when I start a Spring Boot application."**

Give this answer:

> "The application starts from the main method and invokes `SpringApplication.run()`. Spring Boot prepares the environment, determines the application type, creates the appropriate ApplicationContext, loads configuration and bean definitions, and refreshes the context. During the refresh, Spring creates and initializes beans and, for a servlet application, initializes the embedded web server such as Tomcat. After the context has started, Spring publishes `ApplicationStartedEvent`, executes `ApplicationRunner` and `CommandLineRunner` callbacks, and then publishes `ApplicationReadyEvent` when startup is complete. If startup fails, the application can publish `ApplicationFailedEvent` and terminate."

Then, if they ask for the request flow:

```text
Client
  ↓
Load Balancer / Ingress
  ↓
Tomcat
  ↓
DispatcherServlet
  ↓
HandlerMapping
  ↓
Controller
  ↓
Service
  ↓
Repository
  ↓
Database
```

That is the **senior-level mental model** you should have.

---

# 49. What You Should Be Able to Explain in EPAM

You should now be comfortable answering:

### Basic

**What is an embedded server?**

→ Server packaged with the application, allowing executable JAR deployment.

**Why does Spring Boot use it?**

→ Self-contained deployment and easier microservice/container deployment.

**What is Tomcat?**

→ Servlet container/web server.

**What is DispatcherServlet?**

→ Spring MVC front controller.

---

### Intermediate

**What happens inside `SpringApplication.run()`?**

→ Environment → context → bean definitions → refresh → beans → web server → runners → ready.

**ApplicationContext vs WebApplicationContext?**

→ General Spring IoC container vs web-aware application context.

**CommandLineRunner vs ApplicationRunner?**

→ Raw String arguments vs parsed `ApplicationArguments`.

**ApplicationStartedEvent vs ApplicationReadyEvent?**

→ Started after context refresh; Ready after startup runners complete.

---

### Senior

**Why can a pod be Running but not Ready?**

→ Container/process health and application traffic readiness are different concepts.

**How do you gracefully shut down a Spring Boot service in Kubernetes?**

→ SIGTERM → stop new work/traffic → finish in-flight work → release resources → terminate, coordinated with Kubernetes termination grace period.

**Why can a singleton Spring bean have concurrency problems?**

→ Singleton scope means one shared instance per context; Tomcat processes concurrent requests using multiple threads.

**Why can an application start successfully but APIs still return 404?**

→ Server can be healthy while Spring MVC routing/configuration is incorrect.

**Why can an application be Ready while a dependency is unavailable?**

→ Readiness semantics depend on health configuration; not every dependency necessarily needs to block readiness.

---

# Core Mental Model

Memorize this architecture:

```text
                JVM
                 │
                 ▼
        SpringApplication.run()
                 │
                 ▼
            Environment
                 │
                 ▼
         ApplicationContext
                 │
        ┌────────┴─────────┐
        │                  │
     Spring Beans      Web Server
        │                  │
        │                Tomcat
        │                  │
        │          DispatcherServlet
        │                  │
        └──────────┬───────┘
                   ▼
             Application
                   │
                   ▼
        ApplicationStartedEvent
                   │
                   ▼
          Startup Runners
                   │
                   ▼
          ApplicationReadyEvent
                   │
                   ▼
             Serve Traffic
```

This is the lifecycle model you should be able to draw on a whiteboard in an EPAM interview.