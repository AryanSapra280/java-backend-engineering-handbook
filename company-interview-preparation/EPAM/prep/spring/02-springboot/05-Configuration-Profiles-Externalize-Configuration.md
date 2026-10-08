# Spring Boot — Configuration, Profiles, Externalized Configuration & Secrets ⭐⭐⭐⭐⭐

This section is **very important for senior interviews**, especially because you were previously asked:

> **"If I change `application.properties`, do I need to redeploy the application?"**

We will build this from fundamentals to production-level configuration.

---

# 1. Why do we need externalized configuration?

Imagine the same application is deployed to three environments:

```text
Development
    ↓
QA
    ↓
Production
```

The code should ideally remain the same.

But configuration changes:

```text
Database URL
Database username
Kafka brokers
Redis host
API URLs
Logging level
Feature flags
Credentials
```

We don't want:

```text
Build DEV JAR
Build QA JAR
Build PROD JAR
```

Instead:

```text
Same application artifact
        +
Environment-specific configuration
```

This is called **externalized configuration**.

---

# 2. Bad approach

Suppose you hard-code:

```java
String databaseUrl =
    "jdbc:postgresql://prod-db:5432/payment";
```

Problems:

- code must change for each environment
- requires rebuilding
- credentials can accidentally enter source control
- deployment becomes tightly coupled to configuration

Instead:

```java
@Value("${payment.database.url}")
private String databaseUrl;
```

And provide the value externally.

---

# 3. `application.properties`

The simplest configuration source is:

```properties
server.port=8080
payment.timeout=5000
payment.currency=INR
```

You can access it using:

```java
@Value("${payment.timeout}")
private int timeout;
```

Spring resolves the property from its configured property sources.

---

# 4. YAML configuration

The same configuration can be represented as:

```yaml
server:
  port: 8080

payment:
  timeout: 5000
  currency: INR
```

YAML is particularly convenient for hierarchical configuration.

For example:

```yaml
payment:
  retry:
    max-attempts: 3
    backoff: 1000
```

---

# 5. `@Value`

### Interview Question

**How do you read configuration properties in Spring Boot?**

One option is:

```java
@Value("${payment.timeout}")
private int timeout;
```

Spring injects the configured value.

You can also provide a default:

```java
@Value("${payment.timeout:5000}")
private int timeout;
```

Meaning:

```text
If payment.timeout exists
       ↓
use it

Otherwise
       ↓
use 5000
```

---

# 6. Why isn't `@Value` always the best choice?

Suppose you have:

```properties
payment.timeout=5000
payment.retry.max-attempts=3
payment.retry.backoff=1000
payment.currency=INR
payment.enabled=true
```

You could write:

```java
@Value("${payment.timeout}")
private int timeout;

@Value("${payment.retry.max-attempts}")
private int maxAttempts;

@Value("${payment.retry.backoff}")
private int backoff;

@Value("${payment.currency}")
private String currency;

@Value("${payment.enabled}")
private boolean enabled;
```

This becomes difficult to maintain.

A better approach for grouped configuration is:

```java
@ConfigurationProperties(prefix = "payment")
public class PaymentProperties {

    private int timeout;
    private Retry retry;
    private String currency;
    private boolean enabled;

    // getters/setters
}
```

This maps configuration into a structured object.

---

# 7. `@ConfigurationProperties`

### Interview Question

**What is `@ConfigurationProperties` and why would you use it?**

### Interview-ready answer

`@ConfigurationProperties` binds a group of external configuration properties to a strongly typed Java object.

For example:

```yaml
payment:
  timeout: 5000
  currency: INR
  retry:
    max-attempts: 3
    backoff: 1000
```

can map to:

```java
@ConfigurationProperties(prefix = "payment")
public class PaymentProperties {

    private int timeout;
    private String currency;
    private Retry retry;

    // getters/setters
}
```

This gives us:

- type-safe configuration
- centralized configuration
- easier testing
- cleaner code
- hierarchical property binding

---

# 8. How do we register `@ConfigurationProperties`?

A common approach is:

```java
@ConfigurationProperties(prefix = "payment")
public class PaymentProperties {
    ...
}
```

and enable scanning:

```java
@SpringBootApplication
@ConfigurationPropertiesScan
public class PaymentApplication {
}
```

Then Spring Boot discovers the configuration-properties class.

Another approach is explicit registration:

```java
@EnableConfigurationProperties(PaymentProperties.class)
```

---

# 9. `@Value` vs `@ConfigurationProperties`

### Interview Question

**When would you use `@Value` vs `@ConfigurationProperties`?**

| `@Value` | `@ConfigurationProperties` |
|---|---|
| Individual property | Group of properties |
| Simple configuration | Structured configuration |
| Quick/simple usage | Production configuration objects |
| String-expression based | Strongly typed binding |
| Less organized for large config | Easier to maintain |

Example:

```java
@Value("${server.port}")
private int port;
```

is perfectly reasonable.

But:

```text
payment.timeout
payment.retry.max-attempts
payment.retry.backoff
payment.kafka.topic
payment.kafka.bootstrap-servers
payment.feature.enabled
```

is a good candidate for:

```java
@ConfigurationProperties(prefix = "payment")
```

---

# 10. Spring Profiles

Now suppose you have:

```text
Development
QA
Production
```

and each environment requires different configuration.

Spring provides **profiles**.

Typical files:

```text
application.properties
application-dev.properties
application-qa.properties
application-prod.properties
```

For YAML:

```text
application.yml
```

can contain profile-specific sections.

---

# 11. Example of Profiles

### Common configuration

```properties
spring.application.name=payment-service
```

### Development

```properties
# application-dev.properties

server.port=8081
payment.url=http://localhost:9000
```

### Production

```properties
# application-prod.properties

server.port=8080
payment.url=https://payment.internal
```

Activate:

```bash
java -jar payment-service.jar \
    --spring.profiles.active=prod
```

Now Spring Boot loads the production profile configuration.

---

# 12. `spring.profiles.active`

You can activate profiles using:

```properties
spring.profiles.active=dev
```

or:

```bash
java -jar application.jar --spring.profiles.active=prod
```

or through an environment variable:

```bash
SPRING_PROFILES_ACTIVE=prod
```

In Kubernetes this is particularly common.

---

# 13. Multiple active profiles

You can activate multiple profiles:

```properties
spring.profiles.active=prod,aws
```

This can be useful when configuration is split by concern.

For example:

```text
prod
+
aws
```

rather than creating:

```text
prod-aws
prod-gcp
prod-azure
```

for every possible combination.

Be careful with overlapping properties because precedence determines which value wins.

---

# 14. `@Profile`

Profiles aren't only for property files.

You can conditionally create beans.

Example:

```java
@Service
@Profile("dev")
public class MockPaymentGateway {
}
```

Production implementation:

```java
@Service
@Profile("prod")
public class RealPaymentGateway {
}
```

When:

```text
dev
```

is active:

```text
MockPaymentGateway
```

is created.

When:

```text
prod
```

is active:

```text
RealPaymentGateway
```

is created.

---

# 15. Practical example

Suppose your payment service calls a bank.

Development:

```java
@Profile("dev")
@Bean
PaymentClient paymentClient() {
    return new MockPaymentClient();
}
```

Production:

```java
@Profile("prod")
@Bean
PaymentClient paymentClient() {
    return new RealBankPaymentClient();
}
```

The application code can depend on:

```java
PaymentClient
```

without knowing which implementation is active.

That's a very useful use of profiles.

---

# 16. Default Profile

What happens if no profile is active?

Spring uses the:

```text
default
```

profile.

You can configure a different default:

```properties
spring.profiles.default=dev
```

Important distinction:

```text
spring.profiles.active
```

means:

> Which profiles are explicitly active?

Whereas:

```text
spring.profiles.default
```

is used when no explicit profile is active.

---

# 17. Profile-specific configuration

Suppose:

```text
application.properties
application-prod.properties
```

contains:

```properties
# application.properties

payment.timeout=5000
payment.currency=INR
```

and:

```properties
# application-prod.properties

payment.timeout=2000
```

With:

```text
prod
```

active, the effective configuration becomes conceptually:

```text
payment.timeout = 2000
payment.currency = INR
```

The profile-specific configuration overrides the common value.

---

# 18. Environment Variables

In production, configuration is frequently supplied through environment variables.

For example:

```bash
PAYMENT_TIMEOUT=5000
```

Spring Boot can map environment variables into properties.

A common naming convention is:

```text
payment.timeout
```

→

```text
PAYMENT_TIMEOUT
```

For nested properties:

```text
payment.retry.max-attempts
```

can be represented as:

```text
PAYMENT_RETRY_MAX_ATTEMPTS
```

---

# 19. Why environment variables are useful in Kubernetes

Suppose your application runs in:

```text
Pod A
Pod B
Pod C
```

You don't want to rebuild the Docker image just because:

```text
Kafka broker changed
Database host changed
Environment changed
```

Instead:

```text
Docker image
    +
Environment configuration
    ↓
Running application
```

Kubernetes can inject environment variables into the container.

This is a major part of the **12-factor application** style.

---

# 20. Secrets

Now consider:

```text
Database password
JWT secret
API key
OAuth client secret
Kafka credentials
```

These should not be committed like:

```properties
db.password=MySuperSecretPassword123
```

into Git.

Instead, use a secret-management mechanism.

Examples:

```text
Kubernetes Secrets
AWS Secrets Manager
HashiCorp Vault
Azure Key Vault
GCP Secret Manager
```

The exact mechanism depends on the deployment environment.

---

# 21. Important distinction: configuration vs secrets

Not everything is a secret.

### Normal configuration

```text
server.port=8080
payment.timeout=5000
logging.level.root=INFO
```

### Sensitive configuration

```text
DB_PASSWORD
JWT_SECRET
API_KEY
CLIENT_SECRET
```

The second category should be managed with appropriate secret-management controls.

---

# 22. Should we put secrets directly in `application.properties`?

Generally:

**No, not for production secrets.**

This is bad:

```properties
spring.datasource.password=superSecret123
```

especially if the file is committed to Git.

Better:

```properties
spring.datasource.password=${DB_PASSWORD}
```

Then provide:

```text
DB_PASSWORD
```

through the deployment environment/secret manager.

---

# 23. Property placeholders

Spring supports placeholders such as:

```properties
spring.datasource.url=${DB_URL}
spring.datasource.username=${DB_USERNAME}
spring.datasource.password=${DB_PASSWORD}
```

The actual values can be supplied externally.

This gives:

```text
Application artifact
       ↓
External configuration
       ↓
Runtime values
```

---

# 24. The TCS Interview Question You Faced

### Interviewer:

> **"If I change application.properties, do I have to redeploy the application?"**

The answer is:

### It depends on where the configuration comes from and how your application is deployed.

If the configuration is packaged **inside the application JAR**, changing the file generally requires restarting/redeploying the application for the new configuration to be loaded.

For example:

```text
payment-service.jar
   └── application.properties
```

If you modify the packaged configuration, you generally need a new artifact/restart.

But if configuration is **externalized**, you can change the external configuration without rebuilding the application artifact.

For example:

```text
payment-service.jar
       +
external configuration
```

Then the application can load configuration from outside the artifact.

However, there is an important distinction:

> **Changing external configuration does not automatically mean an already-running application immediately picks up the new value.**

The application may need a restart or a refresh mechanism.

This distinction is exactly what you should say in an interview.

---

# 25. Senior-level answer: How can configuration change without redeployment?

There are several approaches.

### Option 1 — External configuration + restart

```text
Config
 ↓
Application
```

Change config:

```text
Config changed
 ↓
Restart application
 ↓
New configuration loaded
```

No new application build is required.

---

### Option 2 — Spring Cloud Config

Architecture:

```text
                 Git
                  │
                  ▼
           Config Server
                  │
        ┌─────────┼─────────┐
        ▼         ▼         ▼
    Service A  Service B  Service C
```

Applications retrieve configuration from a centralized Config Server.

This separates:

```text
Application code
```

from:

```text
Configuration
```

---

### Option 3 — Runtime refresh

With an appropriate configuration management/refresh mechanism, configuration can be refreshed without restarting the entire application.

Conceptually:

```text
Config changed
      ↓
Refresh signal
      ↓
Spring environment/configuration refreshed
      ↓
Beans that support refresh receive new values
```

But don't claim:

> "Spring Boot automatically reloads application.properties."

It does **not** generally work that way.

---

# 26. Very Important Interview Trap

### Interviewer:

> "Can I modify `application.properties` while the application is running and expect `@Value` to change automatically?"

### Answer:

**No, not by default.**

For example:

```java
@Value("${payment.timeout}")
private int timeout;
```

Spring injects the value during bean initialization.

Changing a file later doesn't automatically recreate that bean and update the field.

You need an explicit configuration-refresh mechanism or application restart.

---

# 27. Spring Cloud Config — Why?

Suppose you have:

```text
100 microservices
```

and configuration is distributed across:

```text
100 repositories
```

Managing configuration becomes difficult.

A centralized configuration architecture can look like:

```text
                    Git
                     │
                     ▼
              Config Server
                     │
       ┌─────────────┼─────────────┐
       ▼             ▼             ▼
 Payment Service   Order Service   User Service
```

The Config Server manages/retrieves external configuration.

---

# 28. What problem does Config Server solve?

It helps with:

- centralized configuration
- environment-specific configuration
- versioned configuration
- separation of config from application artifacts
- centralized management

For example:

```text
payment-service
payment-service-dev
payment-service-prod
```

can have separate configurations.

---

# 29. Does Config Server automatically refresh every bean?

**No.**

This is another interview trap.

Having centralized configuration does not magically mean every already-created bean changes instantly.

Runtime refresh requires an appropriate refresh mechanism, and only configuration that is designed/supported for refresh should be treated as dynamically reloadable.

For critical configuration changes, you should explicitly understand:

```text
How configuration is stored
How configuration is delivered
How clients refresh
Which beans are refreshable
What happens to in-flight requests
What happens if refresh fails
```

That's the senior-level answer.

---

# 30. Configuration Precedence

This is extremely important.

Spring Boot can receive configuration from multiple sources:

```text
application.properties
application-prod.properties
Environment variables
System properties
Command-line arguments
External configuration
etc.
```

If the same property is defined in multiple places, Spring Boot applies a defined precedence order.

The important interview concept is:

> **More specific/higher-precedence external configuration can override lower-precedence configuration.**

For example:

```properties
# application.properties

server.port=8080
```

and:

```bash
java -jar app.jar --server.port=9090
```

The command-line value can override the packaged property.

So:

```text
application.properties
server.port=8080

        ↓ overridden by

--server.port=9090

        ↓

Effective value = 9090
```

---

# 31. Why does configuration precedence matter?

Imagine production has:

```properties
payment.timeout=5000
```

but Kubernetes provides:

```text
PAYMENT_TIMEOUT=2000
```

You need to know **which source wins**.

Otherwise you get the classic production debugging problem:

> "The property file says 5000, but the application is using 2000. Why?"

The answer is often configuration precedence.

---

# 32. How do you debug configuration problems?

This is a great senior-level question.

Suppose the application is unexpectedly using:

```text
payment.timeout=2000
```

but you expected:

```text
5000
```

I would investigate:

```text
1. application.properties
2. application-{profile}.properties
3. Environment variables
4. JVM system properties
5. Command-line arguments
6. External configuration
7. Config Server / secret/config management
8. Active profiles
```

Also inspect the effective runtime configuration using appropriate Spring Boot/Actuator facilities while being careful not to expose secrets.

---

# 33. Production Example

Suppose:

```properties
# application.properties

payment.timeout=5000
```

Production environment:

```text
PAYMENT_TIMEOUT=2000
```

Application:

```java
@ConfigurationProperties(prefix = "payment")
public class PaymentProperties {

    private int timeout;

}
```

The application may effectively receive:

```text
timeout = 2000
```

not:

```text
timeout = 5000
```

The important lesson:

> Never assume the value in `application.properties` is necessarily the effective runtime value.

---

# 34. Configuration and Docker

A good container design is:

```text
Docker image
    ↓
Application JAR
    ↓
No environment-specific values hard-coded
```

Then runtime configuration is supplied separately.

For example:

```text
Docker Image
    ↓
Deployment
    ├── Environment variables
    ├── ConfigMap
    └── Secrets
```

This allows:

```text
Same image
   ↓
DEV
QA
PROD
```

with different configuration.

---

# 35. Kubernetes ConfigMap vs Secret

Very important for your microservices interviews.

### ConfigMap

Used for non-sensitive configuration.

Examples:

```text
LOG_LEVEL
PAYMENT_TIMEOUT
FEATURE_FLAG
SERVICE_URL
```

### Secret

Used for sensitive information.

Examples:

```text
DB_PASSWORD
API_KEY
JWT_SECRET
CLIENT_SECRET
```

Conceptually:

```text
Kubernetes
   │
   ├── ConfigMap
   │      ↓
   │   Non-sensitive config
   │
   └── Secret
          ↓
      Sensitive config
```

---

# 36. Senior Interview Question

### Interviewer:

> "Your application is deployed as 10 Kubernetes pods. You change the database URL. What happens?"

There isn't one universal answer.

It depends on how configuration is delivered.

If configuration is:

```text
Environment variable
```

changing the source does not necessarily mutate the environment of already-running containers.

Typically you would update the deployment/configuration and perform a rollout/restart so new pods receive the new value.

If you have a dynamic configuration system:

```text
Config Server
```

with a supported refresh mechanism, you may be able to refresh without restarting every application instance.

So the senior answer is:

> "It depends on the configuration mechanism. Externalized configuration removes the need to rebuild the application image, but it does not automatically mean existing processes see the new value. With Kubernetes environment injection, a rollout is commonly used. With a dynamic config/refresh mechanism, runtime refresh may be possible."

That answer is much stronger than:

> "No redeployment is required."

---

# 37. Configuration Best Practices

For a production Spring Boot service:

### Keep code and environment configuration separate

```text
Code
 ≠
Environment configuration
```

### Don't commit production secrets

Use:

```text
Secret Manager
Kubernetes Secret
Vault
Cloud secret service
```

### Use `@ConfigurationProperties` for grouped configuration

Prefer:

```java
@ConfigurationProperties(prefix = "payment")
```

over dozens of:

```java
@Value(...)
```

### Keep configuration strongly typed

Prefer:

```java
Duration
DataSize
boolean
int
URI
```

where appropriate instead of treating everything as arbitrary strings.

### Don't blindly refresh critical configuration

Changing:

```text
database.url
Kafka brokers
security configuration
```

at runtime can have significant operational consequences.

---

# 38. EPAM Practical Question

### Interviewer:

> "Your application has different database URLs for dev, QA and prod. Would you create three different builds?"

### Strong answer:

> "No. I would build the application artifact once and externalize environment-specific configuration. I could use Spring profiles for environment-specific configuration and inject runtime values through environment variables, Kubernetes ConfigMaps/Secrets, or a centralized configuration system such as Spring Cloud Config. This gives us the same artifact across environments."

---

# 39. EPAM Practical Question

### Interviewer:

> "Where would you store database credentials in Kubernetes?"

### Strong answer:

> "I would not put credentials directly into the application image or Git repository. I would use Kubernetes Secrets or, preferably in many production environments, an external secret manager such as AWS Secrets Manager or Vault, depending on the infrastructure. The application would consume the secret at runtime."

---

# 40. EPAM Practical Question

### Interviewer:

> "Why use `@ConfigurationProperties` instead of `@Value`?"

### Strong answer:

> "`@Value` is convenient for injecting individual properties, but `@ConfigurationProperties` is better for a group of related configuration because it provides structured, strongly typed binding and keeps configuration centralized. It is easier to validate, test and maintain as the application grows."

---

# 41. EPAM Practical Question

### Interviewer:

> "Can configuration change without redeploying?"

### Strong answer:

> "The application artifact doesn't necessarily need to be rebuilt if configuration is externalized. However, an already-running process won't automatically pick up arbitrary configuration changes. We may need a restart/rollout, or we can use a dynamic configuration and refresh mechanism such as Spring Cloud Config with appropriate refresh support."

---

# 42. EPAM Practical Question

### Interviewer:

> "How would you troubleshoot a configuration value that is unexpectedly different in production?"

### Strong answer:

> "First I would identify the active profile and inspect the configuration sources and their precedence: packaged properties, profile-specific properties, external configuration, environment variables, system properties and command-line arguments. In Kubernetes I'd also inspect the Deployment, ConfigMap and Secret references. I'd verify the effective runtime configuration through safe observability mechanisms without exposing credentials."

---

# 43. The Mental Model You Need

Remember this:

```text
                APPLICATION CODE
                       │
                       │
                 Same artifact
                       │
        ┌──────────────┼──────────────┐
        ▼              ▼              ▼
       DEV             QA            PROD
        │              │              │
    Config          Config          Config
        │              │              │
        ▼              ▼              ▼
 Environment       Environment     Environment
 Variables         Variables       Variables
 ConfigMap         ConfigMap       ConfigMap
 Secrets            Secrets         Secrets
```

The goal is:

```text
BUILD ONCE
   ↓
CONFIGURE AT RUNTIME
   ↓
DEPLOY SAME ARTIFACT
```

---

# 44. What You Must Remember for the Interview

### `@Value`

```text
Individual property
```

### `@ConfigurationProperties`

```text
Structured/grouped configuration
```

### Profile

```text
Environment/feature-specific bean/configuration selection
```

### Environment variable

```text
Runtime external configuration
```

### ConfigMap

```text
Non-sensitive Kubernetes configuration
```

### Secret

```text
Sensitive Kubernetes configuration
```

### Config Server

```text
Centralized external configuration
```

### Externalized configuration

```text
Separate configuration from application artifact
```

### Most important production principle

```text
Externalized configuration
        ≠
Automatic runtime refresh
```

That distinction is **very likely to save you if EPAM asks the same kind of practical question TCS asked you.**