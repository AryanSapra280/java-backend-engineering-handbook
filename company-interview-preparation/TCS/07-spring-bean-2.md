Got it — we’ll switch to **fast interview mode**: broad coverage, high-frequency questions, concise answers, and only the depth needed for follow-ups. We’ll still make sure we don't skip important areas.

# Spring Core — Part C: DI + Bean Lifecycle

## 1. How does `@Autowired` work?

Spring looks for a bean matching the required type.

```java
public OrderService(PaymentService paymentService) {
    this.paymentService = paymentService;
}
```

Resolution is roughly:

```text
Type matching
    ↓
Multiple candidates?
    ↓
@Primary / @Qualifier
    ↓
Inject dependency
```

If no matching bean → `NoSuchBeanDefinitionException`.

If multiple matching beans → `NoUniqueBeanDefinitionException`, unless resolved with `@Primary`/`@Qualifier`.

---

## 2. What if there are multiple beans of the same interface?

```java
interface PaymentService {}

@Service
class UpiPaymentService implements PaymentService {}

@Service
class CardPaymentService implements PaymentService {}
```

Then:

```java
PaymentService paymentService
```

is ambiguous.

Solutions:

### `@Qualifier`

```java
public OrderService(
    @Qualifier("upiPaymentService")
    PaymentService paymentService) {
}
```

### `@Primary`

```java
@Primary
@Service
class UpiPaymentService implements PaymentService {}
```

`@Qualifier` has more specific selection semantics.

---

## 3. Can Spring inject multiple implementations?

Yes.

```java
public OrderService(List<PaymentService> paymentServices) {
    this.paymentServices = paymentServices;
}
```

Spring injects all matching beans.

You can also use:

```java
Map<String, PaymentService>
```

Then bean names become keys.

Useful when implementing strategies:

```text
UPI → UpiPaymentService
CARD → CardPaymentService
```

---

# 4. What is Circular Dependency?

Example:

```text
A → B
B → A
```

```java
@Service
class A {
    A(B b) {}
}

@Service
class B {
    B(A a) {}
}
```

Neither can be constructed first.

This creates a circular dependency.

### Constructor injection

Usually fails during startup because:

```text
Create A
 → needs B
    → create B
       → needs A
          → A isn't ready
```

Spring reports a circular dependency problem.

### How can it be resolved?

Best solution:

> Redesign the classes and remove the circular dependency.

Other possible approaches:

* `@Lazy`
* Setter injection in certain cases
* Refactoring common logic into another service

Example:

```java
A(@Lazy B b)
```

But don't present `@Lazy` as the preferred architectural solution.

---

# 5. What does `@Lazy` do?

Normally Spring eagerly creates singleton beans during application startup.

```java
@Service
class PaymentService {}
```

With:

```java
@Lazy
@Service
class PaymentService {}
```

Spring delays creation until the bean is actually needed/requested.

### Common uses

* Expensive bean initialization
* Breaking certain dependency cycles
* Delaying unnecessary initialization

---

# 6. What is Bean Lifecycle?

High-level lifecycle:

```text
1. Instantiate Bean
       ↓
2. Inject dependencies
       ↓
3. Aware callbacks
       ↓
4. BeanPostProcessor before initialization
       ↓
5. @PostConstruct
       ↓
6. InitializingBean / init-method
       ↓
7. BeanPostProcessor after initialization
       ↓
8. Bean ready
       ↓
9. Application runs
       ↓
10. Context shutdown
       ↓
11. @PreDestroy / destroy-method
```

You don't need to memorize every internal callback order for tomorrow unless specifically asked.

---

# 7. What is `@PostConstruct`?

Method runs after dependency injection and during bean initialization.

```java
@PostConstruct
public void init() {
    System.out.println("Bean initialized");
}
```

Typical use:

* Initialization
* Loading configuration
* Preparing resources

---

# 8. What is `@PreDestroy`?

Called when Spring destroys the bean during context shutdown.

```java
@PreDestroy
public void cleanup() {
    // cleanup
}
```

Used for cleanup of resources.

---

# 9. `@PostConstruct` vs Constructor

### Constructor

Object is being created.

```text
constructor
   ↓
dependency injection
   ↓
@PostConstruct
```

So if you need dependencies to already be injected, use `@PostConstruct`.

Example:

```java
@Service
class PaymentService {

    private final ConfigService config;

    PaymentService(ConfigService config) {
        this.config = config;
    }

    @PostConstruct
    void init() {
        // config is available here
    }
}
```

---

# 10. What is `BeanPostProcessor`?

🔥 Important interview question.

A `BeanPostProcessor` allows Spring to process beans **before and after initialization**.

Conceptually:

```java
postProcessBeforeInitialization()
        ↓
@PostConstruct
        ↓
postProcessAfterInitialization()
```

Spring uses this mechanism heavily internally.

AOP/proxy-related processing is also connected to BeanPostProcessor infrastructure.

### Interview answer

> `BeanPostProcessor` allows custom processing of Spring beans before and after their initialization.

---

# 11. `BeanPostProcessor` vs `BeanFactoryPostProcessor`

This distinction is commonly asked.

### BeanPostProcessor

Works with **bean instances**.

```text
Bean instance
    ↓
modify/process
```

### BeanFactoryPostProcessor

Works with **BeanDefinitions/metadata before beans are instantiated**.

```text
BeanDefinition
    ↓
modify/process metadata
    ↓
Bean created later
```

Easy way to remember:

```text
BeanFactoryPostProcessor → Bean definition

BeanPostProcessor        → Bean instance
```

---

# 12. What is a BeanDefinition?

Spring doesn't immediately start with the actual object.

It first has metadata describing the bean:

```text
Bean name
Class
Scope
Dependencies
Lazy?
Initialization method
Destroy method
etc.
```

This metadata is represented conceptually by a **BeanDefinition**.

Then Spring uses that information to create/manage the bean.

---

# 13. What happens if a Bean throws an exception during initialization?

Suppose:

```java
@PostConstruct
void init() {
    throw new RuntimeException();
}
```

Spring cannot successfully initialize the bean.

Application context startup can fail, particularly if the bean is required during startup.

This is why initialization logic should be handled carefully.

---

# 14. Are Spring singleton beans thread-safe?

Again:

**No.**

Singleton means:

```text
one shared object
```

not:

```text
thread-safe object
```

Avoid mutable shared instance variables in stateless services.

---

# 15. Can we inject a prototype into a singleton?

Yes.

But the important trap is:

> The prototype is resolved when the singleton is created, so simply injecting it doesn't give a new prototype on every method call.

For a fresh instance each time, use:

```java
ObjectProvider<PrototypeService>
```

or another lookup mechanism.

---

# 16. Does Spring manage prototype destruction?

Not in the same complete way as singleton beans.

Spring creates and initializes the prototype, but after handing it to the client, the container generally doesn't manage its full destruction lifecycle.

So if a prototype owns resources, the application may need to handle cleanup.

---

# 17. What is the difference between `@Bean` and `@Component`?

**Very high-frequency question.**

| `@Component`                         | `@Bean`                          |
| ------------------------------------ | -------------------------------- |
| Applied to class                     | Applied to method                |
| Component scanning discovers it      | Explicit bean definition         |
| Usually your own classes             | Often third-party/custom objects |
| Spring instantiates discovered class | Method returns object            |

Example:

```java
@Component
class PaymentService {}
```

vs.

```java
@Configuration
class Config {

    @Bean
    ObjectMapper objectMapper() {
        return new ObjectMapper();
    }
}
```

---

# 18. What happens when `@Bean` methods call each other?

Example:

```java
@Configuration
class Config {

    @Bean
    PaymentService paymentService() {
        return new PaymentService(repository());
    }

    @Bean
    PaymentRepository repository() {
        return new PaymentRepository();
    }
}
```

With a normal full `@Configuration` class, Spring manages these bean definitions so that calling `repository()` doesn't simply mean:

```java
new PaymentRepository()
```

Instead, Spring can return the managed bean.

This is one reason `@Configuration` has special semantics.

---

# 19. `@Configuration` vs `@Component`

Both can be detected as Spring components.

But:

```java
@Configuration
class AppConfig {}
```

specifically indicates that the class contains bean configuration and gets special handling for `@Bean` methods.

Interview answer:

> `@Configuration` is a specialized configuration component used to declare bean definitions, whereas `@Component` is a general-purpose Spring-managed component.

---

# 20. What happens when Spring application shuts down?

For singleton beans, Spring invokes destruction callbacks such as:

```java
@PreDestroy
```

or configured destroy methods.

Conceptually:

```text
Application shutdown
       ↓
Destroy singleton beans
       ↓
@PreDestroy
       ↓
cleanup
```

---

# ⚡ Rapid-fire Spring Core questions

These are worth knowing before moving to Spring Boot:

### Can an interface be a Spring Bean?

Not directly.

```java
@Component
interface PaymentService {}
```

Spring cannot instantiate an interface.

An implementation must exist.

---

### Can an abstract class be a Spring Bean?

Spring cannot instantiate the abstract class itself. A concrete implementation is required.

---

### Can we have two beans of the same class?

Yes.

For example:

```java
@Bean
PaymentService payment1() {
    return new PaymentService();
}

@Bean
PaymentService payment2() {
    return new PaymentService();
}
```

There are two different Spring beans.

---

### Is a Spring Bean always singleton?

No.

Singleton is the **default scope**, but Spring supports other scopes.

---

### Is singleton bean created when application starts?

Normally singleton beans are eagerly created during ApplicationContext startup, unless lazy initialization is configured.

---

### Does Spring create beans using `new` internally?

Ultimately, yes, object instantiation has to happen somehow, but Spring controls the construction process and lifecycle rather than your business code doing it directly.

---

# 🔥 Spring Core — DONE

At this point, you should be able to handle most direct Spring Core questions:

```text
IoC / DI
   ↓
@Autowired
   ↓
@Qualifier / @Primary
   ↓
@Component / @Service / @Repository
   ↓
@Bean / @Configuration
   ↓
Component scanning
   ↓
Bean scopes
   ↓
Singleton / Prototype
   ↓
Circular dependency
   ↓
@Lazy
   ↓
Bean lifecycle
   ↓
@PostConstruct / @PreDestroy
   ↓
BeanPostProcessor
   ↓
BeanFactoryPostProcessor
```

## Next: Spring Boot 🚀

We'll move quickly through:

1. `@SpringBootApplication`
2. Auto-configuration
3. How Spring Boot starts
4. Starters
5. Embedded Tomcat
6. `application.properties` / YAML
7. Profiles
8. `@ConfigurationProperties`
9. Actuator
10. Health checks
11. Logging
12. Exception handling
13. Validation
14. External configuration
15. Common Spring Boot interview traps

Then we'll move into **Spring REST → JPA/Hibernate**, where there are significantly more interview questions.
