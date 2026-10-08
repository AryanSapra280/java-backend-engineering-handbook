Absolutely. We stay **exactly on the path**. No jumping to Spring Boot, REST, JPA, or anything else.

We've finished the earlier Spring Core sections. Now we continue from the **next missing topic: Configuration**.

# Spring Core — Configuration

## 45. `@Value`

### What is `@Value`?

`@Value` is used to inject a value from Spring's **Environment/property sources** into a field, constructor parameter, or method parameter.

For example, `application.properties`:

```properties
app.name=Payment Service
app.timeout=5000
```

Java:

```java
@Component
public class PaymentConfig {

    @Value("${app.name}")
    private String appName;

    @Value("${app.timeout}")
    private int timeout;
}
```

Spring resolves:

```text
${app.name}
      ↓
Environment
      ↓
app.name=Payment Service
      ↓
"Payment Service"
```

So:

> **`@Value` tells Spring to resolve a property expression and inject the resulting value into the target.**

---

### Where can `@Value` be used?

#### Field

```java
@Value("${app.name}")
private String appName;
```

#### Constructor

```java
@Component
public class PaymentService {

    private final int timeout;

    public PaymentService(
            @Value("${app.timeout}") int timeout) {
        this.timeout = timeout;
    }
}
```

#### Method parameter

```java
@Bean
public SomeClient client(
        @Value("${client.url}") String url) {

    return new SomeClient(url);
}
```

---

## 🔥 What does `${...}` mean?

This:

```java
@Value("${app.timeout}")
```

means:

> Find the property named `app.timeout` from Spring's available property sources.

It is **property placeholder resolution**.

---

## What if the property doesn't exist?

Suppose:

```java
@Value("${app.timeout}")
private int timeout;
```

but there is no:

```properties
app.timeout=5000
```

Spring normally fails to resolve the required placeholder and application startup can fail.

You can provide a default:

```java
@Value("${app.timeout:3000}")
private int timeout;
```

Meaning:

```text
If app.timeout exists
        ↓
use it

otherwise
        ↓
use 3000
```

This is extremely common in interviews.

---

### 🔥 Interview Question

**Q: How do you provide a default value with `@Value`?**

```java
@Value("${app.timeout:3000}")
private int timeout;
```

`3000` is the default if `app.timeout` isn't available.

---

# 46. `Environment`

Now the interviewer may ask:

> "Instead of `@Value`, can you directly access properties?"

Yes.

Spring provides the:

```java
Environment
```

abstraction.

Example:

```java
@Component
public class PaymentConfig {

    private final Environment environment;

    public PaymentConfig(Environment environment) {
        this.environment = environment;
    }

    public void printConfig() {

        String url =
            environment.getProperty("payment.url");

        String timeout =
            environment.getProperty("payment.timeout");

        System.out.println(url);
        System.out.println(timeout);
    }
}
```

Properties:

```properties
payment.url=https://payment-service
payment.timeout=5000
```

Then:

```java
environment.getProperty("payment.url");
```

returns:

```text
https://payment-service
```

---

## `Environment` vs `@Value`

This is a very common follow-up.

### `@Value`

Best when you want to inject a specific property:

```java
@Value("${payment.timeout}")
private int timeout;
```

### `Environment`

Useful when you need to **programmatically read properties**:

```java
String timeout =
    environment.getProperty("payment.timeout");
```

Think:

```text
@Value
  ↓
Declarative injection

Environment
  ↓
Programmatic property access
```

---

### 🔥 Can `Environment` retrieve different types?

Yes.

You can use:

```java
String value =
    environment.getProperty("payment.timeout");
```

Or:

```java
Integer timeout =
    environment.getProperty(
        "payment.timeout",
        Integer.class
    );
```

You can also provide a default:

```java
Integer timeout =
    environment.getProperty(
        "payment.timeout",
        Integer.class,
        5000
    );
```

So the application doesn't have to manually parse:

```java
Integer.parseInt(...)
```

---

# 47. Profiles

Now we move to **Profiles**.

This is very important in real Spring applications.

Imagine you have:

```text
Development
Testing
Production
```

You don't want the application to use the same database configuration everywhere.

For example:

### Development

```properties
database.url=jdbc:postgresql://localhost:5432/payment
```

### Production

```properties
database.url=jdbc:postgresql://prod-db:5432/payment
```

Spring Profiles allow us to activate environment-specific configuration.

---

## Profile-specific property files

You can have:

```text
application.properties
application-dev.properties
application-test.properties
application-prod.properties
```

For example:

### `application.properties`

```properties
app.name=Payment Service
```

### `application-dev.properties`

```properties
database.url=jdbc:postgresql://localhost:5432/payment
database.username=dev
```

### `application-prod.properties`

```properties
database.url=jdbc:postgresql://prod-db:5432/payment
database.username=payment_user
```

Then activate:

```properties
spring.profiles.active=dev
```

Now Spring loads the appropriate profile-specific configuration.

Conceptually:

```text
application.properties
        +
application-dev.properties
        ↓
Environment
        ↓
Application
```

---

# 48. `@Profile`

Profiles aren't only for properties.

You can conditionally create beans based on the active profile.

Example:

```java
@Service
@Profile("dev")
public class DevPaymentService
        implements PaymentService {

}
```

And:

```java
@Service
@Profile("prod")
public class ProductionPaymentService
        implements PaymentService {

}
```

If:

```properties
spring.profiles.active=dev
```

then:

```text
DevPaymentService
      ↓
created

ProductionPaymentService
      ↓
not created
```

If:

```properties
spring.profiles.active=prod
```

then the opposite happens.

---

## 🔥 Practical example

Suppose you don't want to send real emails from your development environment.

You could have:

```java
public interface EmailService {
    void send(String message);
}
```

Development implementation:

```java
@Service
@Profile("dev")
public class MockEmailService
        implements EmailService {

    public void send(String message) {
        System.out.println("Mock email: " + message);
    }
}
```

Production implementation:

```java
@Service
@Profile("prod")
public class RealEmailService
        implements EmailService {

    public void send(String message) {
        // Send actual email
    }
}
```

Now:

```text
dev
 ↓
MockEmailService

prod
 ↓
RealEmailService
```

This is a very practical use of profiles.

---

# 🔥 Interview Question

### What is the difference between a Profile and `@Profile`?

**Profile** is the concept of defining an environment/configuration group such as:

```text
dev
test
prod
```

`@Profile` is the annotation used to conditionally register a Spring bean based on the active profile.

Example:

```java
@Profile("prod")
@Bean
public PaymentClient paymentClient() {
    return new PaymentClient();
}
```

---

# Can multiple profiles be active?

Yes.

For example:

```properties
spring.profiles.active=prod,monitoring
```

Then both profiles are active.

A bean can also belong to multiple profiles:

```java
@Profile({"prod", "staging"})
@Component
public class ProductionLikeService {
}
```

The exact profile expression can also be more sophisticated, for example:

```java
@Profile("prod & cloud")
```

meaning both profiles must be active.

---

# 49. Property Sources

Now we reach the part that ties everything together.

When you write:

```java
@Value("${payment.timeout}")
```

where does Spring actually search for:

```text
payment.timeout
```

?

It searches through Spring's **property sources**.

A property source is essentially a source from which configuration properties can be obtained.

Examples include:

```text
application.properties
application.yml
Environment variables
System properties
Command-line arguments
```

Conceptually:

```text
                Environment
                    │
        ┌───────────┼───────────┐
        ↓           ↓           ↓
 application   Environment   System
 properties     variables    properties
        │           │           │
        └───────────┼───────────┘
                    ↓
             Property lookup
                    ↓
          payment.timeout
```

---

# 🔥 Why does property-source order matter?

Imagine:

```properties
payment.timeout=5000
```

in:

```text
application.properties
```

But your environment variable says:

```text
PAYMENT_TIMEOUT=10000
```

Which one should Spring use?

This is where **property precedence** matters.

Higher-precedence configuration can override lower-precedence configuration.

This is extremely useful in production because you can keep a default value in the application package while overriding it externally.

For example:

```properties
# application.properties

payment.timeout=5000
```

Production environment:

```text
PAYMENT_TIMEOUT=10000
```

The external configuration can override the packaged default.

---

# 🔥 Why external configuration?

Suppose you hard-code:

```properties
database.password=myPassword
```

inside your application.

That's a bad production practice.

Instead:

```text
Application
     ↓
reads configuration
     ↓
Environment variable / secret / external config
```

So the same application artifact can run in different environments:

```text
             SAME JAR
                │
       ┌────────┼────────┐
       ↓        ↓        ↓
      DEV      TEST     PROD
       │        │        │
    different configuration
```

You don't need to rebuild the application for every environment.

---

# 🔥 Interview Scenario

### Interviewer:

> "You have a Spring application. The database URL is different in DEV, QA and PROD. Would you create three different builds?"

**Answer:**

> No. I would keep the application artifact the same and externalize environment-specific configuration. Spring supports profiles, environment variables, configuration files and other property sources. The appropriate configuration can be supplied for each environment without changing the application code or rebuilding the artifact.

That's a **senior-level practical answer**.

---

# `application.properties` vs `application.yml`

Since this is also in your Spring Core plan, don't skip it.

### Properties

```properties
server.port=8080
database.url=jdbc:postgresql://localhost:5432/payment
database.username=payment
```

### YAML

```yaml
server:
  port: 8080

database:
  url: jdbc:postgresql://localhost:5432/payment
  username: payment
```

YAML is often easier to read for hierarchical configuration.

Both ultimately provide configuration to Spring's environment/configuration system.

---

# 🔥 Important Interview Trap

### Q: Is `application.properties` itself the Environment?

**No.**

`application.properties` is **one source of configuration**.

`Environment` is Spring's abstraction through which the application can access the resolved environment and its properties.

Think:

```text
application.properties ──┐
application.yml ──────────┤
Environment variables ────┤
System properties ────────┤
Command-line args ────────┤
                           ↓
                      Environment
                           ↓
                  getProperty(...)
```

---

# 🔥 Final Mental Model

You should now be able to explain the entire chain:

```text
Configuration Sources
        │
        ├── application.properties
        ├── application.yml
        ├── Environment variables
        ├── System properties
        └── Command-line arguments
                │
                ↓
        Spring Environment
                │
        ┌───────┴────────┐
        ↓                ↓
     @Value        getProperty()
        │                │
        └───────┬────────┘
                ↓
          Application
```

And Profiles sit alongside this:

```text
              Profiles
                 │
       ┌─────────┼─────────┐
       ↓         ↓         ↓
      dev       test      prod
       │         │         │
       ↓         ↓         ↓
application-dev.properties
application-test.properties
application-prod.properties
```

### 🎯 What you should be able to answer now

**Q: `@Value` vs `Environment`?**

> `@Value` provides declarative property injection, while `Environment` allows programmatic property access.

**Q: Why use profiles?**

> To activate environment-specific configuration and conditionally register beans.

**Q: What is `@Profile`?**

> It conditionally registers a bean when the specified profile is active.

**Q: What is a property source?**

> A source from which Spring obtains configuration properties, such as property files, environment variables and system properties.

**Q: Why externalize configuration?**

> To keep environment-specific values outside the application artifact so the same build can be deployed across environments.

**Q: Why does property precedence matter?**

> Because the same property can exist in multiple sources, and the higher-precedence source determines the effective value.

---
