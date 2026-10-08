# Spring Boot — Starters & Dependency Management ⭐⭐⭐⭐⭐

## 1. What is a Spring Boot Starter?

A **Spring Boot Starter** is a convenient dependency descriptor that groups together dependencies commonly required for a particular capability.

Instead of manually adding many individual dependencies, you add one starter.

For example:

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
```

This brings in the dependencies needed for a typical Spring web application.

Conceptually:

```text
spring-boot-starter-web
        ↓
Spring MVC
Jackson
Embedded servlet container
Validation-related web dependencies
        ↓
Application can build REST APIs
```

The exact dependency graph depends on the Spring Boot version.

---

# 2. Why do we need Starters?

Without starters, you might manually add:

```text
Spring MVC
Jackson
Servlet API
Embedded server
Validation
etc.
```

You would also have to think about whether the versions of all those libraries are compatible.

With a starter:

```text
Developer
   ↓
Adds starter
   ↓
Required dependency set
   ↓
Spring Boot dependency management
   ↓
Compatible versions
```

This reduces configuration and dependency-management effort.

---

# 3. Common Spring Boot Starters

Some commonly used starters are:

```text
spring-boot-starter-web
spring-boot-starter-data-jpa
spring-boot-starter-security
spring-boot-starter-validation
spring-boot-starter-test
spring-boot-starter-actuator
```

For example:

### REST API

```xml
<artifactId>spring-boot-starter-web</artifactId>
```

### JPA/Hibernate

```xml
<artifactId>spring-boot-starter-data-jpa</artifactId>
```

### Security

```xml
<artifactId>spring-boot-starter-security</artifactId>
```

### Validation

```xml
<artifactId>spring-boot-starter-validation</artifactId>
```

### Testing

```xml
<artifactId>spring-boot-starter-test</artifactId>
```

---

# 4. Does a Starter contain the implementation?

No.

This is a common interview trap.

Suppose:

```xml
spring-boot-starter-web
```

is added.

The starter itself is mainly a convenient way to declare the required dependencies.

It doesn't mean:

```text
starter
    ↓
contains complete Spring MVC implementation
```

Instead:

```text
starter
    ↓
declares dependencies
    ↓
Maven resolves them
    ↓
libraries appear on classpath
    ↓
Spring Boot auto-configuration detects them
    ↓
configuration is applied
```

So remember:

> **Starter = dependency convenience.**

> **Auto-configuration = conditional configuration.**

---

# 5. Starter vs Auto-Configuration

This is worth knowing extremely well.

### Starter

Answers:

> **"What dependencies should I bring into my application?"**

```text
Starter
   ↓
Dependencies
```

### Auto-Configuration

Answers:

> **"Given the dependencies and configuration I have, what should Spring configure automatically?"**

```text
Classpath
+
Properties
+
Existing beans
+
Conditions
        ↓
Auto-configuration
```

Together:

```text
Starter
   ↓
Dependencies
   ↓
Classpath
   ↓
Auto-Configuration
   ↓
Beans
```

---

# 6. What is dependency management?

Suppose your application depends on:

```text
Spring Framework
Hibernate
Jackson
Tomcat
JUnit
```

Each library has versions.

For example, conceptually:

```text
Spring → version A
Hibernate → version B
Jackson → version C
```

These libraries often have compatibility relationships.

You don't want every application developer manually choosing every version.

Spring Boot provides a **curated dependency-management approach** so that commonly used dependency versions are compatible with the chosen Boot version.

---

# 7. Spring Boot BOM

One important mechanism is the **Bill of Materials (BOM)**.

A BOM is essentially a centralized list of dependency versions.

Conceptually:

```text
Spring Boot BOM
       |
       +── Spring Framework → version X
       +── Hibernate → version Y
       +── Jackson → version Z
       +── Tomcat → version A
       +── JUnit → version B
```

Your project can then use those managed versions instead of specifying every version manually.

---

# 8. Why don't we specify the version for every dependency?

For example:

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
```

Notice:

```text
No version
```

Why?

Because the Spring Boot parent/BOM can manage the version.

This is intentional.

You generally want:

```text
Spring Boot version
        ↓
Compatible dependency versions
```

rather than:

```text
Developer manually selects
50 dependency versions
```

---

# 9. Spring Boot parent

A typical Maven Spring Boot application historically/commonly uses:

```xml
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>...</version>
</parent>
```

The Boot parent provides useful Maven defaults and dependency management.

It can provide:

- dependency version management
- plugin management
- useful Maven defaults
- resource encoding/configuration defaults

---

# 10. Do we have to use `spring-boot-starter-parent`?

No.

This is an important distinction.

You can use Spring Boot's dependency management without inheriting from the Boot parent, for example through the Boot BOM.

Conceptually:

```text
Option 1

spring-boot-starter-parent
        ↓
dependency management + Maven defaults
```

or:

```text
Option 2

your own parent
        ↓
import Spring Boot BOM
        ↓
managed dependency versions
```

This matters in organizations that already have a corporate Maven parent.

---

# 11. What is a transitive dependency?

Suppose:

```text
Your Application
      ↓
Starter Web
      ↓
Spring MVC
      ↓
Jackson
```

You explicitly declared:

```text
spring-boot-starter-web
```

but Maven brings additional dependencies required by it.

Those are **transitive dependencies**.

Conceptually:

```text
A
 ↓
B
 ↓
C
```

You declare:

```text
A
```

but Maven also resolves:

```text
B
C
```

because they're dependencies of your dependencies.

---

# 12. Example of transitive dependencies

You add:

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
```

That starter brings other dependencies.

You don't manually declare every underlying library.

Maven constructs the dependency graph:

```text
application
     |
     ↓
starter-web
   /   |   \
  ↓    ↓    ↓
Spring MVC
Jackson
Embedded server
...
```

---

# 13. Interview question: How do you see the dependency tree?

This is a very practical question.

With Maven:

```bash
mvn dependency:tree
```

This shows the dependency graph.

For example:

```text
com.example:payment-service
   |
   +-- spring-boot-starter-web
   |     +-- spring-web
   |     +-- spring-webmvc
   |     +-- jackson
   |     +-- embedded-server
   |
   +-- spring-boot-starter-data-jpa
         +-- spring-data-jpa
         +-- hibernate
         +-- jdbc
```

This is extremely useful when debugging dependency conflicts.

---

# 14. Senior interview scenario — version conflict

Suppose your application has:

```text
Library A
    ↓
Jackson 2.x

Library B
    ↓
Jackson 3.x
```

Now Maven has to resolve a dependency graph where different components request different versions.

You need to understand:

```text
dependency resolution
+
managed versions
+
direct dependencies
+
exclusions
```

Don't simply say:

> "Maven will use the latest version."

That's an oversimplification.

---

# 15. Maven dependency mediation

Maven builds a dependency graph and applies dependency mediation rules to choose versions.

One important rule is commonly described as:

> **Nearest definition wins.**

For example:

```text
Application
   |
   +── A
   |    |
   |    └── X:1.0
   |
   └── B
        |
        └── C
             |
             └── X:2.0
```

If `X:1.0` is closer to the root than `X:2.0`, Maven may select the nearer version, subject to dependency management and the actual graph.

That's why simply looking at the library's declared version isn't enough.

Use:

```bash
mvn dependency:tree
```

to understand what Maven actually resolved.

---

# 16. Dependency Management vs Dependency

This distinction is very important.

### Dependency

Means:

> **I need this library on my classpath.**

Example:

```xml
<dependency>
    <groupId>...</groupId>
    <artifactId>...</artifactId>
</dependency>
```

### Dependency Management

Means:

> **If this dependency is used, use this version/configuration.**

Conceptually:

```xml
<dependencyManagement>
    ...
</dependencyManagement>
```

It manages versions but doesn't necessarily add the dependency to the application's classpath by itself.

This is a common Maven interview question.

---

# 17. Example

Suppose:

```xml
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>com.example</groupId>
            <artifactId>common-library</artifactId>
            <version>2.5.0</version>
        </dependency>
    </dependencies>
</dependencyManagement>
```

This says:

```text
When common-library is used,
use version 2.5.0.
```

But you still need:

```xml
<dependency>
    <groupId>com.example</groupId>
    <artifactId>common-library</artifactId>
</dependency>
```

to actually depend on it.

---

# 18. What if I want to override Boot's managed version?

You can explicitly specify a version for a dependency when appropriate.

Conceptually:

```xml
<dependency>
    <groupId>com.example</groupId>
    <artifactId>some-library</artifactId>
    <version>custom-version</version>
</dependency>
```

But this should be done carefully.

Why?

Because Spring Boot's dependency management is designed around a tested/compatible set of versions.

Overriding one library can introduce:

```text
API incompatibility
runtime errors
NoSuchMethodError
ClassNotFoundException
serialization problems
```

So the senior approach is:

> Override managed versions only when there's a clear reason and verify compatibility.

---

# 19. What is a dependency conflict?

Suppose:

```text
Service A
   ↓
Library X → Jackson 2.15

Service A
   ↓
Library Y → Jackson 2.13
```

Now two dependency paths request different Jackson versions.

This is a dependency conflict.

Potential runtime problems include:

```text
NoSuchMethodError
NoClassDefFoundError
ClassNotFoundException
Incompatible API behavior
```

This is why dependency management matters.

---

# 20. Excluding a transitive dependency

Suppose:

```text
Library A
   ↓
Library B
```

but you don't want B.

You can exclude it:

```xml
<dependency>
    <groupId>com.example</groupId>
    <artifactId>library-a</artifactId>

    <exclusions>
        <exclusion>
            <groupId>com.example</groupId>
            <artifactId>library-b</artifactId>
        </exclusion>
    </exclusions>
</dependency>
```

Conceptually:

```text
A
 |
 +── B ❌ excluded
```

Then you can explicitly add the version you want if necessary.

---

# 21. Senior scenario — security vulnerability

Imagine:

```text
Application
   ↓
Starter
   ↓
Transitive dependency
   ↓
Vulnerable library version
```

You discover a security vulnerability.

What should you do?

A good approach:

```text
1. Identify where the dependency comes from
2. Check dependency tree
3. Check whether Boot has a newer compatible version
4. Upgrade Boot if appropriate
5. Override dependency version only when necessary
6. Verify compatibility and tests
```

Don't blindly exclude a dependency without understanding which component requires it.

---

# 22. Starter naming

Spring Boot starters typically follow a convention such as:

```text
spring-boot-starter-<technology>
```

Examples:

```text
spring-boot-starter-web
spring-boot-starter-security
spring-boot-starter-data-jpa
spring-boot-starter-validation
```

Spring Boot also allows organizations to create their own starters.

For example:

```text
company-spring-boot-starter-observability
company-spring-boot-starter-security
company-spring-boot-starter-kafka
```

This is particularly useful in large organizations.

---

# 23. Custom company starter — production example

Imagine your organization has 100 microservices.

Every service needs:

```text
Correlation ID
OpenTelemetry
Standard logging
Metrics
Common security configuration
```

Without a starter:

```text
Service A → duplicate configuration
Service B → duplicate configuration
Service C → duplicate configuration
...
```

Instead:

```text
company-observability-starter
             ↓
      auto-configuration
             ↓
 ┌───────────┼───────────┐
 ↓           ↓           ↓
Logging    Metrics     Tracing
```

Each service simply adds the starter.

This gives centralized standards.

---

# 24. Starter + custom Auto-Configuration

This connects directly to the previous topic.

A custom starter commonly contains:

```text
Starter
   |
   +── Dependencies
   |
   +── Auto-Configuration
   |
   +── Configuration properties
```

For example:

```text
company-payment-starter
        |
        +── PaymentClient
        +── PaymentProperties
        +── PaymentAutoConfiguration
```

Then the application simply adds:

```xml
<dependency>
    <groupId>com.company</groupId>
    <artifactId>company-payment-starter</artifactId>
</dependency>
```

and Boot can automatically configure it.

---

# 25. Interview question

### Q: What's the benefit of Spring Boot dependency management?

### Strong answer:

> Spring Boot provides curated dependency versions that are designed to work together. This reduces the need to manually manage versions for every dependency and helps prevent incompatible dependency combinations. If necessary, individual versions can be overridden, but that should be done carefully because it can introduce compatibility issues.

---

# 26. Interview question

### Q: What's the difference between a starter and a BOM?

### Answer:

A **starter** is a convenient dependency descriptor that groups dependencies for a particular capability.

A **BOM** primarily manages dependency versions.

Conceptually:

```text
Starter
   ↓
"What libraries do I need?"

BOM
   ↓
"What versions should those libraries use?"
```

They solve different problems.

---

# 27. Interview question

### Q: If I add `spring-boot-starter-web`, why does Tomcat come into my application?

Because the starter declares the appropriate embedded servlet container dependency transitively for the standard servlet web setup.

Conceptually:

```text
starter-web
     ↓
embedded servlet container
     ↓
Tomcat available
```

Spring Boot then configures the web application around it.

---

# 28. Interview question

### Q: If I don't want Tomcat, can I use another server?

Yes.

The exact setup depends on the Boot version and servlet stack, but conceptually you can exclude the default embedded server and add another supported implementation.

For example:

```text
Tomcat
   ↓
exclude
   ↓
Jetty
```

The important idea is that the embedded server is a dependency/configuration choice rather than a hard requirement that the application must use Tomcat.

---

# 29. Dependency debugging workflow

When something unexpectedly breaks after adding a dependency:

```text
              Problem
                 ↓
        Check dependency tree
                 ↓
       Identify conflicting version
                 ↓
     Identify who introduced it
                 ↓
      Check Boot managed version
                 ↓
     Upgrade Boot if appropriate
                 ↓
    Override/exclude if justified
                 ↓
       Run tests + integration tests
```

Useful commands:

```bash
mvn dependency:tree
```

You can also inspect Maven's effective POM when you need to understand inherited dependency/plugin configuration:

```bash
mvn help:effective-pom
```

---

# 30. Common interview traps

### Trap 1

> Starter = auto-configuration.

❌ No.

```text
Starter → dependencies
Auto-config → conditional configuration
```

---

### Trap 2

> Dependency management adds the dependency.

❌ Not necessarily.

Dependency management primarily controls versions/configuration.

---

### Trap 3

> Maven always chooses the latest dependency version.

❌ Incorrect.

Dependency mediation and dependency management affect the selected version.

---

### Trap 4

> If Boot manages a dependency version, you can never override it.

❌ Incorrect.

You can override it, but you should understand the compatibility implications.

---

### Trap 5

> A transitive dependency is something you don't control.

❌ Not quite.

You can influence dependency resolution using:

- dependency management
- explicit dependencies
- exclusions
- compatible version selection

---

# 31. The complete mental model

Remember this:

```text
                  Spring Boot
                      |
                      ↓
                   Starter
                      |
                      ↓
              Dependency Graph
                      |
              ┌───────┴────────┐
              ↓                ↓
       Direct dependencies  Transitive
                              dependencies
              └───────┬────────┘
                      ↓
             Dependency Management
                      |
                      ↓
               Version Selection
                      |
                      ↓
                 Classpath
                      |
                      ↓
             Auto-Configuration
                      |
                      ↓
                    Beans
```

---

# 32. Final interview answer

### "Explain Spring Boot Starters and Dependency Management."

> Spring Boot starters are convenient dependency descriptors that group dependencies required for a particular capability, such as web, JPA, security or testing. They simplify dependency declaration and work together with Spring Boot's dependency-management system. Spring Boot provides curated versions through dependency management/BOMs so that commonly used libraries are compatible with the selected Boot version. Maven also resolves transitive dependencies, and when conflicts occur I would inspect the dependency tree, understand which dependency introduced the conflicting version, and then use managed versions, explicit dependencies or exclusions carefully.

---

## Current Spring Boot progress

### Fundamentals

- ✅ Spring vs Spring Boot
- ✅ Auto-configuration
- ✅ Starters
- ✅ Dependency management
- ⏳ Embedded server
- ⏳ Spring Boot application lifecycle

### Next

**Embedded Server + Spring Boot Application Lifecycle** — including:

- What actually starts when `SpringApplication.run()` executes
- Servlet container startup
- Tomcat architecture at a useful interview depth
- `DispatcherServlet`
- ApplicationContext vs WebApplicationContext
- startup phases
- `ApplicationStartingEvent`
- `ApplicationPreparedEvent`
- `ApplicationStartedEvent`
- `ApplicationReadyEvent`
- `CommandLineRunner`
- `ApplicationRunner`
- graceful shutdown
- what happens when startup fails

Then we'll move to **Spring Boot Configuration**, where we'll connect your previous Spring Core configuration knowledge to Boot's `application.properties`, YAML, profiles, environment variables, secrets, `@ConfigurationProperties`, precedence, and Config Server concepts.