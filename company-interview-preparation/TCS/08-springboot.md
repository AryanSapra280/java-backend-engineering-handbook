Let's go! 🔥 **Spring Boot — fast interview mode.**
We'll cover the important questions quickly, with enough detail for follow-ups.

# Spring Boot — Part 1

## 1. What is Spring Boot?

### Interview answer

> Spring Boot is built on top of Spring and simplifies application development by providing auto-configuration, starter dependencies, embedded servers, and production-ready features.

Traditional Spring required more manual configuration.

Spring Boot reduces that configuration.

```text
Spring
  ↓
Core framework

Spring Boot
  ↓
Spring + Auto Configuration
      + Starters
      + Embedded Server
      + Production features
```

---

# 2. Spring vs Spring Boot

| Spring                                 | Spring Boot                   |
| -------------------------------------- | ----------------------------- |
| Core framework                         | Built on Spring               |
| More configuration                     | Convention over configuration |
| Server often configured separately     | Embedded server               |
| Dependencies manually selected         | Starters                      |
| More setup                             | Faster setup                  |
| Production features need configuration | Actuator provides them        |

**Important:** Spring Boot does NOT replace Spring. It simplifies using Spring.

---

# 3. What is `@SpringBootApplication`?

🔥 Very common.

```java
@SpringBootApplication
public class Application {
    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}
```

`@SpringBootApplication` is effectively a combination of:

```java
@Configuration
@EnableAutoConfiguration
@ComponentScan
```

So:

```text
@SpringBootApplication
        |
        +-- @Configuration
        +-- @EnableAutoConfiguration
        +-- @ComponentScan
```

---

# 4. What does `@Configuration` do?

We covered this in Spring Core.

It indicates that the class contains Spring configuration / bean definitions.

```java
@Configuration
class AppConfig {

    @Bean
    PaymentService paymentService() {
        return new PaymentService();
    }
}
```

---

# 5. What does `@ComponentScan` do?

It tells Spring where to scan for components such as:

```java
@Component
@Service
@Repository
@Controller
```

Example:

```java
@ComponentScan("com.example")
```

Spring scans that package and its relevant subpackages.

### Common trap

If your main class is:

```text
com.example.Application
```

and your service is:

```text
com.other.PaymentService
```

it may not be discovered automatically.

---

# 6. What is Auto-Configuration?

🔥 **One of the most important Spring Boot questions.**

Spring Boot looks at:

* Dependencies on the classpath
* Existing beans
* Application configuration
* Conditions

and automatically configures appropriate components.

Example:

You add:

```text
spring-boot-starter-web
```

Spring Boot sees web-related dependencies and configures things such as:

* Spring MVC
* Embedded web server
* DispatcherServlet
* HTTP infrastructure

You don't have to configure all of these manually.

---

# 7. How does Auto-Configuration work?

The simplified flow:

```text
Spring Boot starts
       ↓
Reads configuration
       ↓
Checks classpath
       ↓
Checks existing beans
       ↓
Checks @Conditional conditions
       ↓
Applies matching auto-configurations
       ↓
Creates required beans
```

Auto-configuration is heavily based on conditional configuration.

For example conceptually:

```java
@ConditionalOnClass(SomeLibrary.class)
```

means:

> Configure this feature if the required class exists.

Another common condition:

```java
@ConditionalOnMissingBean
```

means:

> Create the default bean only if the application hasn't already provided one.

🔥 This last point is important.

---

# 8. What is `@ConditionalOnMissingBean`?

Suppose Spring Boot auto-configures:

```text
DefaultDataSource
```

But you define your own:

```java
@Bean
DataSource myDataSource() {
    ...
}
```

Boot can detect that a DataSource already exists and avoid creating its default one.

Conceptually:

```text
Does DataSource exist?
       |
   YES → don't create default
   NO  → create default
```

This is one of the key mechanisms that makes Spring Boot auto-configuration customizable.

---

# 9. What are Spring Boot Starters?

Starters are convenient dependency bundles.

For example:

```text
spring-boot-starter-web
```

provides dependencies needed for typical web applications.

Other examples:

```text
spring-boot-starter-data-jpa
spring-boot-starter-security
spring-boot-starter-test
```

Instead of manually adding many compatible dependencies, you use the starter.

### Interview answer

> A Spring Boot starter is a curated dependency descriptor that brings together the dependencies commonly required for a particular functionality.

---

# 10. Does a Starter contain the actual implementation?

A starter is primarily a dependency descriptor.

For example:

```text
starter-web
   ↓
brings required web dependencies
   ↓
Spring MVC
Jackson
embedded server-related dependencies
...
```

The starter itself isn't the web framework implementation.

---

# 11. What is an Embedded Server?

Spring Boot applications can package the server with the application.

For example:

```text
Spring Boot application
       +
Embedded Tomcat
       ↓
Executable JAR
```

You can run:

```bash
java -jar application.jar
```

without separately installing/configuring Tomcat.

Common embedded servers:

* Tomcat
* Jetty
* Undertow

Tomcat is commonly used by default with the standard web starter.

---

# 12. How does Spring Boot application start?

Typical:

```java
SpringApplication.run(Application.class, args);
```

High-level flow:

```text
main()
 ↓
SpringApplication.run()
 ↓
Create ApplicationContext
 ↓
Read configuration
 ↓
Component scanning
 ↓
Auto-configuration
 ↓
Create beans
 ↓
Start embedded server
 ↓
Application ready
```

Don't overcomplicate this in an interview unless asked.

---

# 13. What is `application.properties` / `application.yml`?

Used for externalized configuration.

Example:

```properties
server.port=8081
spring.datasource.url=jdbc:postgresql://localhost:5432/test
```

YAML:

```yaml
server:
  port: 8081

spring:
  datasource:
    url: jdbc:postgresql://localhost:5432/test
```

The idea is:

> Configuration should not be hardcoded into Java code.

---

# 14. What is Externalized Configuration?

Instead of:

```java
String url = "http://production-server";
```

use:

```properties
external.service.url=http://production-server
```

and inject it:

```java
@Value("${external.service.url}")
private String url;
```

This allows different configuration for:

```text
Development
Testing
Production
```

without changing the application code.

---

# 15. What are Spring Profiles?

Profiles allow environment-specific configuration.

For example:

```text
application.properties
application-dev.properties
application-prod.properties
```

Activate:

```properties
spring.profiles.active=dev
```

Or:

```bash
java -jar app.jar --spring.profiles.active=prod
```

You can also use:

```java
@Profile("prod")
@Bean
PaymentClient productionPaymentClient() {
    ...
}
```

That bean is created only when the `prod` profile is active.

---

# 16. `@Value` vs `@ConfigurationProperties`

### `@Value`

Good for individual values:

```java
@Value("${payment.timeout}")
private int timeout;
```

### `@ConfigurationProperties`

Better for grouped configuration:

```yaml
payment:
  timeout: 5000
  retry-count: 3
  base-url: https://payment
```

```java
@ConfigurationProperties(prefix = "payment")
class PaymentProperties {
    private int timeout;
    private int retryCount;
    private String baseUrl;
}
```

### Interview answer

> `@Value` is convenient for individual properties, while `@ConfigurationProperties` is better for structured and grouped configuration.

---

# 17. What is Spring Boot Actuator?

🔥 Very important.

Actuator provides production-oriented endpoints for monitoring and management.

Dependency:

```text
spring-boot-starter-actuator
```

Common endpoints include:

```text
/actuator/health
/actuator/info
/actuator/metrics
```

Depending on configuration/version, other endpoints can also be exposed.

---

# 18. What is `/actuator/health`?

Used to determine application health.

Example:

```text
GET /actuator/health
```

Response can indicate:

```json
{
  "status": "UP"
}
```

Health indicators can include things like:

* Database
* Redis
* Kafka-related dependencies
* Other configured external systems

Very useful in Kubernetes/container environments.

---

# 19. Liveness vs Readiness

🔥 Important production question.

### Liveness

> Is the application itself alive?

If liveness fails, the platform may restart the application.

### Readiness

> Is the application ready to receive traffic?

If readiness fails, traffic can be removed from the instance without necessarily restarting it.

Think:

```text
Liveness  → Should I restart you?

Readiness → Should I send traffic to you?
```

---

# 20. What is Spring Boot Actuator useful for in production?

You can use it for:

```text
Health
Metrics
Application info
Monitoring
Operational endpoints
```

For example:

```text
API slow
 ↓
Check application metrics
 ↓
CPU / memory / request metrics
 ↓
Investigate
```

But actuator endpoints should be secured appropriately.

Don't expose sensitive management endpoints publicly without considering authentication and network restrictions.

---

# 21. How do you change the server port?

```properties
server.port=8081
```

or YAML:

```yaml
server:
  port: 8081
```

---

# 22. How do you configure different ports for different environments?

```text
application.properties
application-dev.properties
application-prod.properties
```

For example:

```properties
# application-dev.properties
server.port=8081
```

```properties
# application-prod.properties
server.port=8080
```

Activate the appropriate profile.

---

# 23. What is `SpringApplication.run()`?

It's the entry point that boots the Spring application.

It:

* Creates/configures the application context
* Registers configuration
* Performs component scanning
* Applies auto-configuration
* Initializes beans
* Starts the embedded server for web applications

---

# 24. What happens if auto-configuration creates something you don't want?

You can customize configuration or exclude specific auto-configuration.

Example:

```java
@SpringBootApplication(
    exclude = DataSourceAutoConfiguration.class
)
```

This tells Spring Boot not to apply that particular auto-configuration.

---

# 25. How do you debug why Spring Boot auto-configuration happened?

🔥 Nice senior-level follow-up.

You can enable debug information, for example:

```properties
debug=true
```

Spring Boot can then provide a **condition evaluation report**, showing which auto-configurations matched or didn't match and why.

This is useful when something gets automatically configured unexpectedly.

---

# 26. What is `@EnableAutoConfiguration`?

It tells Spring Boot to attempt automatic configuration based on:

* Classpath
* Existing beans
* Properties
* Conditions

It's one of the three major pieces of:

```java
@SpringBootApplication
```

---

# 27. Common Spring Boot interview traps

### Is Spring Boot a replacement for Spring?

**No.** It's built on top of Spring and simplifies configuration/development.

### Does Spring Boot eliminate configuration?

**No.** It reduces boilerplate through conventions and auto-configuration, but configuration is still possible and often necessary.

### Does auto-configuration always create beans?

**No.** Conditional rules determine whether configuration is applied.

### Can we override auto-configuration?

**Yes.** You can provide your own beans/configuration or exclude particular auto-configurations.

### Is embedded Tomcat mandatory?

**No.** You can use other embedded servers or deploy differently depending on the application setup.

---

# 🚀 Spring Boot Part 1 COMPLETE

Your mental model should now be:

```text
@SpringBootApplication
        |
        +── @Configuration
        |
        +── @ComponentScan
        |
        +── @EnableAutoConfiguration
                  |
                  ↓
            Classpath + Conditions
                  ↓
            Auto-configured Beans
                  ↓
             ApplicationContext
                  ↓
             Embedded Server
```

### Next Spring Boot section

We'll quickly cover the **actual REST/API interview questions**:

* `@RestController`
* `@Controller`
* `@RequestMapping`
* `@GetMapping` / `@PostMapping`
* `@PathVariable` vs `@RequestParam`
* `@RequestBody`
* DTOs
* Validation
* `@Valid`
* Global exception handling
* `@ControllerAdvice`
* HTTP status codes
* Pagination
* API versioning
* Idempotency
* REST interview traps

Then we'll move into **JPA/Hibernate**, which is a much larger and very important section.
