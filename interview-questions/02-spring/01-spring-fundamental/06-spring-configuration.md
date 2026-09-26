Absolutely! 🔥 This section is straightforward, but **101–114 is very practical** because interviewers often connect Spring configuration with **Docker/Kubernetes, environment variables, secrets, and production deployment**.

I'll keep the answers simple and in a form you can actually speak in an interview.

# 🟢 G. Spring Configuration

---

## 101. What is `application.properties`?

### Interview answer

> `application.properties` is a configuration file used by Spring Boot to define application settings such as database configuration, server port, logging, Kafka configuration, and custom application properties.

Example:

```properties
server.port=8081

spring.datasource.url=jdbc:postgresql://localhost:5432/pfdb
spring.datasource.username=postgres

pf.batch.chunk-size=500
pf.batch.retry-count=3
```

Spring Boot automatically reads this file from the standard configuration locations.

### Easy memory

> **`application.properties` = key-value configuration.**

---

# 102. What is `application.yml`?

`application.yml` serves the same general purpose as `application.properties`, but uses **YAML syntax**.

Example:

```yaml
server:
  port: 8081

spring:
  datasource:
    url: jdbc:postgresql://localhost:5432/pfdb
    username: postgres

pf:
  batch:
    chunk-size: 500
    retry-count: 3
```

Notice how YAML represents hierarchy using indentation.

### Properties equivalent

```properties
pf.batch.chunk-size=500
pf.batch.retry-count=3
```

YAML:

```yaml
pf:
  batch:
    chunk-size: 500
    retry-count: 3
```

### Easy memory

> **YAML = hierarchical/structured configuration.**

---

# 103. YAML vs Properties?

Both can configure Spring Boot.

### Properties

```properties
server.port=8080
spring.datasource.url=jdbc:postgresql://localhost/db
spring.datasource.username=postgres
```

### YAML

```yaml
server:
  port: 8080

spring:
  datasource:
    url: jdbc:postgresql://localhost/db
    username: postgres
```

### Comparison

| Properties                 | YAML                                    |
| -------------------------- | --------------------------------------- |
| Simple key-value format    | Hierarchical format                     |
| Very explicit              | More compact for nested config          |
| Easy for small config      | Convenient for large structured config  |
| No indentation sensitivity | Indentation matters                     |
| Can become repetitive      | Can be easier to read for nested config |

### Interview answer

> Both are supported by Spring Boot. Properties uses flat key-value pairs, while YAML represents hierarchical configuration using indentation. For deeply nested configuration, YAML can be easier to read.

### One warning

YAML is indentation-sensitive:

```yaml
spring:
  datasource:
    url: ...
```

Incorrect indentation can cause configuration problems.

---

# 104. How does Spring Boot load configuration?

🔥 Important.

At startup, Spring Boot builds its configuration environment from multiple **property sources**.

Conceptually:

```text id="g4lq1e"
Application starts
       ↓
Spring Boot creates Environment
       ↓
Reads configuration sources
       ↓
application.properties / YAML
       ↓
Profile-specific config
       ↓
Environment variables
       ↓
Command-line arguments
       ↓
Other configured property sources
       ↓
Resolve final property values
       ↓
Inject into beans
```

For example:

```yaml
pf:
  batch:
    chunk-size: 500
```

Then:

```java
@Value("${pf.batch.chunk-size}")
private int chunkSize;
```

Spring resolves:

```text
${pf.batch.chunk-size}
        ↓
500
```

### Interview answer

> Spring Boot loads configuration into its `Environment` from multiple property sources such as application properties/YAML, profile-specific configuration, environment variables, system properties, and command-line arguments. If a property is requested, Spring resolves its value according to the property-source precedence rules.

---

# 105. What is `@Value`?

### Interview answer

> `@Value` is used to inject a specific configuration value into a Spring bean.

Example:

```yaml
pf:
  batch:
    chunk-size: 500
```

Then:

```java
@Service
public class BatchService {

    @Value("${pf.batch.chunk-size}")
    private int chunkSize;
}
```

Spring injects:

```text
500
```

### You can also provide a default

```java
@Value("${pf.batch.chunk-size:100}")
private int chunkSize;
```

Meaning:

> If the property doesn't exist, use `100`.

### Another example

```java
@Value("${server.port}")
private int port;
```

### Remember

> **`@Value` = inject one/small number of configuration values.**

For large configuration groups, prefer `@ConfigurationProperties`.

---

# 106. What is `@ConfigurationProperties`?

### Interview answer

> `@ConfigurationProperties` binds a group of related configuration properties to a Java object.

Suppose:

```yaml
pf:
  batch:
    chunk-size: 500
    retry-count: 3
    thread-count: 8
    enabled: true
```

Create:

```java
@ConfigurationProperties(prefix = "pf.batch")
public class BatchProperties {

    private int chunkSize;
    private int retryCount;
    private int threadCount;
    private boolean enabled;

    // getters/setters
}
```

Spring binds:

```text id="yut4kr"
pf.batch.chunk-size  → chunkSize
pf.batch.retry-count → retryCount
pf.batch.thread-count → threadCount
pf.batch.enabled     → enabled
```

### In Spring Boot

You need to make the configuration properties class available for binding, commonly using either:

```java
@ConfigurationPropertiesScan
```

or:

```java
@EnableConfigurationProperties(BatchProperties.class)
```

depending on how you structure the application.

---

# 107. Why might `@ConfigurationProperties` be preferred for grouped configuration?

Imagine you have 10 properties.

Using `@Value`:

```java
@Value("${pf.batch.chunk-size}")
private int chunkSize;

@Value("${pf.batch.retry-count}")
private int retryCount;

@Value("${pf.batch.thread-count}")
private int threadCount;

@Value("${pf.batch.timeout}")
private Duration timeout;

// ...more
```

This becomes messy.

With:

```java
@ConfigurationProperties(prefix = "pf.batch")
```

you get:

```java
BatchProperties
```

containing all related configuration.

### Benefits

**1. Better organization**

```text
pf.batch.*
      ↓
BatchProperties
```

**2. Type-safe configuration**

Instead of dealing with strings everywhere, Spring binds to:

```java
int
boolean
Duration
List
Map
```

etc.

**3. Easier validation**

You can use validation annotations when configured appropriately.

**4. Easier testing**

You can test the configuration object separately.

### Interview answer

> `@ConfigurationProperties` is preferred for grouped configuration because it keeps related properties together in a strongly typed object, improves readability and maintainability, and works well for validation and structured configuration.

---

# 108. What are Spring Profiles?

### Interview answer

> Spring Profiles allow us to define different configurations or beans for different environments or situations, such as development, testing, and production.

Typical profiles:

```text id="1p1phz"
dev
test
prod
```

For example:

```text id="9l6ozf"
application.yml
application-dev.yml
application-test.yml
application-prod.yml
```

You might have:

### `application-dev.yml`

```yaml
spring:
  datasource:
    url: jdbc:postgresql://localhost/pfdb
```

### `application-prod.yml`

```yaml
spring:
  datasource:
    url: jdbc:postgresql://prod-db/pfdb
```

Same application, different configuration.

### Easy memory

> **Profile = environment-specific configuration.**

---

# 109. How do you create `dev`, `test`, and `prod` configurations?

A common structure is:

```text id="82nxga"
src/main/resources/

application.yml
application-dev.yml
application-test.yml
application-prod.yml
```

For example:

### `application.yml`

Common configuration:

```yaml
spring:
  application:
    name: pf-service
```

### `application-dev.yml`

```yaml
spring:
  datasource:
    url: jdbc:postgresql://localhost:5432/pfdb
```

### `application-test.yml`

```yaml
spring:
  datasource:
    url: jdbc:postgresql://test-db:5432/pfdb
```

### `application-prod.yml`

```yaml
spring:
  datasource:
    url: jdbc:postgresql://prod-db:5432/pfdb
```

Then activate the required profile.

---

# 110. How do you activate a profile?

Several ways.

### 1. Configuration

```yaml
spring:
  profiles:
    active: dev
```

### 2. Command line

```bash
java -jar app.jar --spring.profiles.active=prod
```

### 3. Environment variable

```bash
SPRING_PROFILES_ACTIVE=prod
```

This is very common in containers/Kubernetes.

### Interview answer

> A Spring profile can be activated through configuration, command-line arguments, environment variables, or programmatically. In deployed environments, environment variables such as `SPRING_PROFILES_ACTIVE=prod` are commonly used.

### Practical deployment example

```text id="2klx74"
Kubernetes
    ↓
SPRING_PROFILES_ACTIVE=prod
    ↓
Spring Boot
    ↓
application-prod.yml
```

---

# 111. What happens if the same property exists in multiple configuration sources?

🔥 Important.

Suppose:

```yaml
# application.yml
pf:
  batch:
    chunk-size: 500
```

But environment variable/configuration provides:

```text
PF_BATCH_CHUNK_SIZE=1000
```

Spring doesn't randomly choose one.

It uses **property-source precedence**.

The higher-precedence source wins.

### Common simplified hierarchy

A useful interview-level understanding is:

```text id="h4gl1f"
Command-line arguments
        ↑
Environment variables / system properties
        ↑
Profile-specific configuration
        ↑
application.yml / application.properties
```

The exact precedence list is longer, and there are additional property sources such as external configuration files, so don't claim this is the complete ordering.

### Interview answer

> When the same property is present in multiple sources, Spring Boot resolves it using its property-source precedence rules. The value from the higher-precedence source wins.

### Why is this useful?

It allows us to keep defaults in Git:

```yaml
pf:
  batch:
    chunk-size: 500
```

and override them in production:

```text
PF_BATCH_CHUNK_SIZE=1000
```

without modifying the application code.

---

# 112. How would you externalize database credentials?

🔥 This is a **production-oriented question**.

Don't hardcode:

```yaml
spring:
  datasource:
    username: postgres
    password: myPassword123
```

Instead, use external configuration.

For example:

```yaml
spring:
  datasource:
    url: ${DB_URL}
    username: ${DB_USERNAME}
    password: ${DB_PASSWORD}
```

Then provide:

```text id="nq7svy"
DB_URL=...
DB_USERNAME=...
DB_PASSWORD=...
```

through your deployment environment.

### In Kubernetes

A common pattern is:

```text id="jj8kqt"
Kubernetes Secret
       ↓
Environment variables / mounted secret
       ↓
Spring Boot
       ↓
Datasource
```

### Interview answer

> I would externalize database credentials rather than storing them in source code. Spring Boot can read them from environment variables, Kubernetes Secrets, or a dedicated secrets-management system.

---

# 113. Should passwords be stored directly in `application.yml`?

### Interview answer

> Generally, no — especially not production credentials committed to source control.

This is bad:

```yaml
spring:
  datasource:
    password: SuperSecretPassword
```

because it can end up in:

* Git history
* source repositories
* logs or configuration dumps
* deployment artifacts

Instead:

```yaml
spring:
  datasource:
    username: ${DB_USERNAME}
    password: ${DB_PASSWORD}
```

And keep the actual secret in a secret-management system.

Examples include:

```text id="b7d4bw"
Kubernetes Secrets
AWS Secrets Manager
HashiCorp Vault
Azure Key Vault
GCP Secret Manager
```

### Strong interview answer

> Production secrets should be stored in a dedicated secret-management mechanism rather than committed to application configuration. The application should receive them at runtime through environment variables, mounted secrets, or a secrets-management integration.

---

# 114. How would you manage configuration across environments?

This is a **very good system-design-style Spring question.**

I would separate:

### 1. Common configuration

```text id="3iw3v7"
application.yml
```

Example:

```yaml
spring:
  application:
    name: pf-service

pf:
  batch:
    retry-count: 3
```

### 2. Environment-specific configuration

```text id="2q5m4a"
application-dev.yml
application-test.yml
application-prod.yml
```

For example:

```yaml
# application-prod.yml

pf:
  batch:
    thread-count: 10
```

### 3. Secrets

Don't put secrets into Git.

Use:

```text id="ly4u4j"
Secret Manager / Kubernetes Secret
```

### 4. Environment variables

Use environment variables for deployment-specific overrides:

```text
SPRING_PROFILES_ACTIVE=prod
DB_URL=...
DB_USERNAME=...
DB_PASSWORD=...
```

### Overall architecture

```text id="q4n7rj"
                  Git
                   │
          ┌────────┴────────┐
          ↓                 ↓
 application.yml     application-prod.yml
   common config       prod config
          │                 │
          └────────┬────────┘
                   ↓
              Spring Boot
                   ↑
                   │
          Environment Variables
                   ↑
                   │
           Secret Management
                   │
                   ↓
             DB credentials
```

### Strong interview answer

> I would keep common configuration in `application.yml`, environment-specific values in profile-specific files such as `application-dev.yml` and `application-prod.yml`, and sensitive values such as database passwords in a secret-management system. Deployment-specific values can be injected through environment variables. This keeps configuration externalized and prevents secrets from being committed to source control.

---

# 🧠 One Practical Example

Imagine your PF service has:

```yaml
pf:
  batch:
    chunk-size: 500
    thread-count: 8
```

### Development

```yaml
# application-dev.yml

pf:
  batch:
    chunk-size: 100
    thread-count: 2
```

### Production

```yaml
# application-prod.yml

pf:
  batch:
    chunk-size: 1000
    thread-count: 10
```

Then:

```text id="f7a1e9"
Developer
   ↓
SPRING_PROFILES_ACTIVE=dev
   ↓
chunk-size = 100


Production
   ↓
SPRING_PROFILES_ACTIVE=prod
   ↓
chunk-size = 1000
```

Same Java code.

Different configuration.

That's the main purpose of **Spring Profiles + externalized configuration**.

---

# 🎯 Rapid-Fire Interview Revision

### 101. `application.properties`?

> Spring Boot configuration file using key-value syntax.

### 102. `application.yml`?

> Spring Boot configuration file using hierarchical YAML syntax.

### 103. YAML vs properties?

> Both configure Spring Boot; YAML is hierarchical and properties is key-value based.

### 104. How does Boot load configuration?

> It builds an Environment from multiple property sources and resolves values using precedence rules.

### 105. `@Value`?

> Injects an individual configuration value.

### 106. `@ConfigurationProperties`?

> Binds a group of related configuration properties to a typed Java object.

### 107. Why prefer it?

> Cleaner, structured, type-safe, and easier to validate/manage.

### 108. Profiles?

> Different configurations/beans for different environments.

### 109. Dev/test/prod?

```text
application.yml
application-dev.yml
application-test.yml
application-prod.yml
```

### 110. Activate profile?

> `spring.profiles.active`, environment variable, command line, etc.

### 111. Same property in multiple places?

> Spring uses property-source precedence; the higher-precedence source wins.

### 112. Database credentials?

> Externalize them using environment variables/secrets management.

### 113. Password in YAML?

> Don't commit production secrets directly into source-controlled configuration.

### 114. Environment management?

> Common config + profile-specific config + external environment variables + secret manager.

---

# 🔥 One SDE-2 Follow-up You Should Be Ready For

An interviewer may now take this:

```yaml
spring:
  datasource:
    password: ${DB_PASSWORD}
```

and ask:

> **"Where does `${DB_PASSWORD}` come from?"**

A good answer is:

> **Spring resolves placeholders from its Environment. In a deployed application, `DB_PASSWORD` can be supplied as an environment variable, or through another configured property source such as a secret-management integration.**

Then they may ask:

> **"What if both `application.yml` and an environment variable define it?"**

You answer:

> **"Spring Boot's property-source precedence determines the final value; the higher-precedence source overrides the lower-precedence one."**

And then they may ask:

> **"How would you handle this in Kubernetes?"**

You should be able to say:

```text
Kubernetes Secret
       ↓
Environment variable / mounted secret
       ↓
Spring Environment
       ↓
${DB_PASSWORD}
       ↓
DataSource
```

**That chain — Configuration → Profiles → Environment → Secrets → Kubernetes — is worth understanding as one connected concept.**
