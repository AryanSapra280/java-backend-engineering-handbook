# Spring Profiles — Interview Revision Notes

## 1. What is a Spring Profile?

A Spring Profile allows us to configure or activate different beans/configurations for different environments.

Typical environments:

- development
- testing
- staging
- production

Example:

```text
DEV
 → local PostgreSQL
 → local Kafka
 → debug logging

PROD
 → production PostgreSQL
 → production Kafka
 → production logging
```

The goal is to keep environment-specific configuration outside business logic.

---

## 2. Profile-Specific Properties

A common Spring Boot structure is:

```text
application.properties
application-dev.properties
application-test.properties
application-prod.properties
```

Example:

### application.properties

```properties
spring.application.name=pf-service
```

### application-dev.properties

```properties
spring.datasource.url=jdbc:postgresql://localhost:5432/pf_dev
logging.level.root=DEBUG
```

### application-prod.properties

```properties
spring.datasource.url=jdbc:postgresql://prod-db:5432/pf
logging.level.root=INFO
```

Common configuration stays in `application.properties`; environment-specific configuration goes into `application-{profile}.properties`.

---

## 3. Activating a Profile

One common approach:

```properties
spring.profiles.active=dev
```

Then Spring Boot activates the `dev` profile and loads the relevant profile-specific configuration.

You can also activate a profile externally.

### Command line

```bash
java -jar app.jar --spring.profiles.active=prod
```

### Environment variable

```bash
SPRING_PROFILES_ACTIVE=prod
```

This is especially common in containers and Kubernetes.

The same application artifact can be deployed to different environments:

```text
DEV  → profile=dev
TEST → profile=test
PROD → profile=prod
```

---

## 4. `@Profile`

Profiles can also conditionally create Spring beans.

```java
@Component
@Profile("dev")
public class DevPaymentService {
}
```

This bean is registered only when the `dev` profile is active.

---

## 5. `@Profile` with `@Bean`

Example:

```java
@Configuration
public class PaymentConfig {

    @Bean
    @Profile("dev")
    public PaymentClient devPaymentClient() {
        return new MockPaymentClient();
    }

    @Bean
    @Profile("prod")
    public PaymentClient prodPaymentClient() {
        return new RealPaymentClient();
    }
}
```

Conceptually:

```text
dev
 ↓
MockPaymentClient

prod
 ↓
RealPaymentClient
```

The application can depend on the `PaymentClient` abstraction without knowing which implementation was selected.

---

## 6. `@Profile` with `@Configuration`

```java
@Configuration
@Profile("dev")
public class DevConfig {

    @Bean
    public PaymentClient paymentClient() {
        return new MockPaymentClient();
    }
}
```

Production can have its own configuration:

```java
@Configuration
@Profile("prod")
public class ProdConfig {

    @Bean
    public PaymentClient paymentClient() {
        return new RealPaymentClient();
    }
}
```

---

## 7. No Active Profile

Spring has a `default` profile.

If no explicit profile is active, the default profile is used as the fallback for beans/configuration associated with it.

Important distinction:

```text
Active profile
    ≠
Default profile
```

---

## 8. Multiple Active Profiles

Multiple profiles can be active:

```properties
spring.profiles.active=dev,debug
```

Then both are active:

```text
dev
+
debug
```

You can also specify multiple profiles on a bean:

```java
@Profile({"dev", "test"})
```

For Monday, you only need to understand the concept; complex profile expressions are lower priority.

---

## 9. Profile-Specific Configuration Files

The conventional Spring Boot naming is:

```text
application-{profile}.properties
```

or:

```text
application-{profile}.yml
```

Examples:

```text
application-dev.yml
application-test.yml
application-prod.yml
```

If `prod` is active, Spring Boot loads the base configuration together with the profile-specific configuration.

When the same property is defined in both, the profile-specific configuration takes precedence in the normal Spring Boot profile setup.

Example:

### application.properties

```properties
server.port=8080
```

### application-prod.properties

```properties
server.port=9090
```

With `prod` active:

```text
server.port=9090
```

---

## 10. Real-World PF Application Example

Imagine the PF application needs different external service configurations.

Development:

```text
PF Service
   ↓
Local Transaction Manager
   ↓
Local DB
```

Production:

```text
PF Service
   ↓
Production Transaction Manager
   ↓
Production DB
```

You can have:

```text
application-dev.properties
application-prod.properties
```

and profile-specific beans:

```java
@Configuration
@Profile("dev")
public class DevConfig {

    @Bean
    public TransactionClient transactionClient() {
        return new MockTransactionClient();
    }
}
```

Production:

```java
@Configuration
@Profile("prod")
public class ProdConfig {

    @Bean
    public TransactionClient transactionClient() {
        return new RealTransactionClient();
    }
}
```

---

## 11. Profiles vs `@ConfigurationProperties`

Do not confuse these concepts.

### Profile

Answers:

> Which environment/configuration set is active?

Examples:

```text
dev
test
prod
```

### `@ConfigurationProperties`

Answers:

> How do I bind configuration values into a Java object?

Example:

```properties
pf.batch.chunk-size=500
pf.batch.thread-count=8
```

```java
@ConfigurationProperties(prefix = "pf.batch")
public class BatchProperties {

    private int chunkSize;
    private int threadCount;
}
```

Mental model:

```text
Profile
   ↓
Which environment/configuration?

@ConfigurationProperties
   ↓
How do I map configuration into Java?
```

---

## 12. Profiles vs Conditional Bean Creation

Spring has several conditional mechanisms.

Examples:

```text
@Profile
@ConditionalOnProperty
@ConditionalOnClass
@ConditionalOnMissingBean
```

### `@Profile`

Environment-based:

```java
@Profile("prod")
```

### `@ConditionalOnProperty`

Property-based:

```java
@ConditionalOnProperty(
    name = "feature.payment.enabled",
    havingValue = "true"
)
```

`@ConditionalOnProperty` becomes particularly important in Spring Boot auto-configuration.

---

## 13. External Configuration

Do not put secrets such as:

```text
DB passwords
Kafka credentials
API keys
```

directly into application code.

Production systems commonly externalize sensitive configuration through environment variables, Kubernetes Secrets, or dedicated configuration/secrets systems.

The key principle:

> Environment-specific and sensitive configuration should generally be externalized rather than hardcoded into application code.

---

## 14. Profiles in Kubernetes

This is particularly relevant for backend systems.

The same Docker image can be deployed to multiple environments:

```text
Same Docker Image
       ↓
 ┌────────┼─────────┐
 ↓        ↓         ↓
DEV      QA        PROD
 ↓        ↓         ↓
Profile  Profile   Profile
 ↓        ↓         ↓
Config   Config    Config
```

For example:

```text
DEV Pod
  ↓
SPRING_PROFILES_ACTIVE=dev

QA Pod
  ↓
SPRING_PROFILES_ACTIVE=qa

PROD Pod
  ↓
SPRING_PROFILES_ACTIVE=prod
```

The application image remains the same; environment-specific configuration changes.

---

# Interview Questions

## Q1. What is a Spring Profile?

A mechanism for activating environment-specific configurations and beans, such as development, testing, or production.

## Q2. How do you activate a profile?

Common approaches:

```properties
spring.profiles.active=prod
```

or:

```bash
java -jar app.jar --spring.profiles.active=prod
```

or:

```bash
SPRING_PROFILES_ACTIVE=prod
```

## Q3. What is `@Profile`?

It conditionally registers a bean or configuration based on whether a specified profile is active.

## Q4. Can multiple profiles be active?

Yes.

Example:

```text
dev,debug
```

## Q5. What happens if no profile is active?

Spring uses the default profile as the fallback.

## Q6. What is `application-prod.properties`?

A profile-specific Spring Boot configuration file that is loaded when the `prod` profile is active.

## Q7. Why use profiles?

To run the same application artifact with different environment-specific configurations or beans without changing business code.

## Q8. Profiles vs `@ConfigurationProperties`?

Profiles determine which environment/configuration is active. `@ConfigurationProperties` binds configuration values into a structured Java object.

---

# Monday Interview Cheat Sheet

Remember:

```text
Profile
   ↓
Environment-specific configuration

application.properties
   ↓
Common configuration

application-dev.properties
application-test.properties
application-prod.properties
   ↓
Profile-specific configuration
```

Activation:

```text
spring.profiles.active=prod
```

or:

```bash
--spring.profiles.active=prod
```

or:

```bash
SPRING_PROFILES_ACTIVE=prod
```

Conditional beans:

```java
@Profile("prod")
```

Multiple profiles:

```text
dev,debug
```

Key distinction:

```text
Profile
   ↓
Which environment/configuration?

@ConfigurationProperties
   ↓
How do I bind configuration into Java?
```

---

# Final Mental Model

```text
                 Spring Boot
                     ↓
             Active Profiles
                     ↓
          ┌──────────┼──────────┐
          ↓          ↓          ↓
         dev        test       prod
          ↓          ↓          ↓
      dev config  test config  prod config
          ↓          ↓          ↓
        Beans      Beans       Beans
```

Same JAR/Docker image:

```text
Same Application Artifact
          ↓
 ┌────────┼─────────┐
 ↓        ↓         ↓
DEV      QA        PROD
 ↓        ↓         ↓
Profile  Profile   Profile
 ↓        ↓         ↓
Config   Config    Config
```

---

# What NOT to Study Deeply Yet

For the Monday interview, do not spend too much time on:

- Advanced profile expressions
- Complex property source precedence
- Custom `Environment` implementations
- Config data internals
- Spring Environment API internals

Focus on:

```text
@Profile
      ↓
application-{profile}.properties
      ↓
spring.profiles.active
      ↓
Multiple profiles
      ↓
Profile-specific beans
      ↓
External configuration
```
