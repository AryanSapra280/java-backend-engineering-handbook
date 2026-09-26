Absolutely bro! 🔥 Let's continue with **Spring Core — Part B: Spring Beans**.

This is a **very important interview area** because TCS can easily start with something simple like "`@Component` kya hai?" and then drill into bean lifecycle, scopes, `@Bean`, singleton behavior, etc.

---

# Spring Core — Part B: Spring Beans

## Q11. What is a Spring Bean?

### Interview answer

> A Spring Bean is an object that is created, managed, and maintained by the Spring IoC container.

For example:

```java
@Service
public class PaymentService {
}
```

Spring sees `@Service`, creates an instance of `PaymentService`, and stores/manages it inside the Spring container.

Then another class can use it:

```java
@Service
public class OrderService {

    private final PaymentService paymentService;

    public OrderService(PaymentService paymentService) {
        this.paymentService = paymentService;
    }
}
```

Both objects are managed by Spring.

### Important distinction

Not every Java object is a Spring Bean.

```java
PaymentService service = new PaymentService();
```

This is just a normal Java object.

But:

```java
@Service
class PaymentService {}
```

when discovered by Spring becomes a Spring Bean.

---

# Q12. How does Spring create a Bean?

Suppose:

```java
@Service
public class PaymentService {
}
```

At application startup, roughly:

```text
Spring Boot starts
      ↓
ApplicationContext created
      ↓
Component scanning
      ↓
PaymentService discovered
      ↓
Bean definition registered
      ↓
Spring creates PaymentService object
      ↓
Dependencies injected
      ↓
Bean lifecycle callbacks
      ↓
Bean becomes ready
```

The important concept is:

> **Spring first knows how a bean should be created through its BeanDefinition, and then the IoC container creates and manages the actual object.**

---

# Q13. What is `@Component`?

`@Component` tells Spring:

> "Detect this class during component scanning and create it as a Spring Bean."

Example:

```java
@Component
public class EmailSender {
}
```

Spring will discover it and register it as a bean.

---

# Q14. What are `@Service`, `@Repository`, and `@Controller`?

They are specialized forms of `@Component`.

```text
@Component
   ├── @Service
   ├── @Repository
   └── @Controller
```

### `@Service`

Used for business/service logic:

```java
@Service
class PaymentService {
}
```

### `@Repository`

Used for persistence/data-access components:

```java
@Repository
class PaymentRepository {
}
```

It also has an important Spring feature: **exception translation** for persistence exceptions.

### `@Controller`

Used for Spring MVC controllers:

```java
@Controller
class PaymentController {
}
```

### `@RestController`

Effectively combines:

```java
@Controller
@ResponseBody
```

So:

```java
@RestController
class PaymentController {
}
```

is commonly used for REST APIs.

### Interview trap

Are `@Service`, `@Repository`, and `@Controller` fundamentally different from `@Component`?

Conceptually they are specialized stereotypes built on top of `@Component`, but some provide additional framework semantics.

For example:

`@Repository` → persistence exception translation.

---

# Q15. What is component scanning?

Spring needs to find classes annotated with things like:

```java
@Component
@Service
@Repository
@Controller
```

Component scanning searches configured packages for these classes.

For a typical Spring Boot application:

```java
@SpringBootApplication
public class Application {
}
```

`@SpringBootApplication` includes component scanning behavior.

If your structure is:

```text
com.example
 ├── Application
 ├── service
 │    └── PaymentService
 └── controller
      └── PaymentController
```

Spring Boot can discover them because they're under the application's component-scan package hierarchy.

### Interview trap

If your service is outside the scanned package:

```text
com.example
   └── Application

com.other
   └── PaymentService
```

Spring may not discover `PaymentService`.

Then injection can fail because there is no corresponding Spring Bean.

---

# Q16. `@Component` vs `@Bean`

🔥 **Very common question.**

### `@Component`

Put it on the class:

```java
@Component
class PaymentService {
}
```

Spring discovers the class through component scanning.

### `@Bean`

Put it on a method:

```java
@Configuration
class AppConfig {

    @Bean
    PaymentService paymentService() {
        return new PaymentService();
    }
}
```

Here **you explicitly tell Spring how to create the object**.

### Simple distinction

```text
@Component
→ Spring discovers the class

@Bean
→ You explicitly provide the object creation method
```

---

# Q17. When would you use `@Bean` instead of `@Component`?

Suppose you need to configure a third-party library:

```java
ObjectMapper objectMapper = new ObjectMapper();
```

You can't modify `ObjectMapper` and put:

```java
@Component
```

on it.

So:

```java
@Configuration
class AppConfig {

    @Bean
    ObjectMapper objectMapper() {
        return new ObjectMapper();
    }
}
```

Now Spring manages that object.

Typical use cases:

* Third-party classes
* Custom object creation
* Complex initialization
* Custom configuration
* Multiple differently configured instances

---

# Q18. What is `@Configuration`?

`@Configuration` tells Spring:

> "This class contains bean definitions/configuration."

Example:

```java
@Configuration
public class AppConfig {

    @Bean
    public PaymentService paymentService() {
        return new PaymentService();
    }
}
```

Spring processes this configuration and registers the returned object as a bean.

---

# Q19. What happens if two `@Bean` methods return the same type?

Example:

```java
@Configuration
class AppConfig {

    @Bean
    PaymentService upiPaymentService() {
        return new UpiPaymentService();
    }

    @Bean
    PaymentService cardPaymentService() {
        return new CardPaymentService();
    }
}
```

Now Spring has:

```text
PaymentService
   ├── upiPaymentService
   └── cardPaymentService
```

If you do:

```java
OrderService(PaymentService paymentService)
```

Spring has multiple candidates.

You can use:

```java
@Qualifier("upiPaymentService")
```

or mark one:

```java
@Primary
```

---

# Q20. What are Spring Bean scopes?

Bean scope determines **how long/how many instances of a bean Spring creates and manages**.

The commonly important scopes are:

### 1. Singleton

Default.

```java
@Service
class PaymentService {
}
```

Normally Spring creates **one bean instance per ApplicationContext**.

### 2. Prototype

A new instance is created whenever Spring is asked for that bean.

```java
@Scope("prototype")
@Component
class PaymentService {
}
```

### Web scopes

There are also:

* Request
* Session
* Application
* WebSocket

For normal backend interviews, understand **singleton and prototype extremely well**.

---

# Q21. Is Spring Singleton the same as Singleton Design Pattern?

🔥 **Important trap.**

No.

Java Singleton Design Pattern typically means:

> Only one instance exists for the entire JVM/classloader according to the implementation.

Spring Singleton means:

> **One instance per Spring IoC container/ApplicationContext.**

For example:

```text
ApplicationContext A
    → PaymentService instance #1

ApplicationContext B
    → PaymentService instance #2
```

So there can be multiple instances if there are multiple Spring containers.

### Interview-ready answer

> Spring singleton scope guarantees one bean instance per Spring ApplicationContext, not necessarily one instance for the entire JVM.

---

# Q22. Are Spring Singleton Beans thread-safe?

🔥 Very important.

**No, not automatically.**

Spring creates one shared instance:

```text
Thread 1 ──┐
Thread 2 ──┼──> Same PaymentService object
Thread 3 ──┘
```

If your bean has mutable shared state:

```java
@Service
class PaymentService {

    private int count = 0;

    public void process() {
        count++;
    }
}
```

multiple threads can access the same `count`.

That can create race conditions.

### Better design

Prefer stateless services:

```java
@Service
class PaymentService {

    public PaymentResponse process(PaymentRequest request) {
        // local variables
    }
}
```

Local variables belong to each thread's stack.

### Strong interview statement

> **Singleton scope and thread safety are two different concepts. Spring manages the lifecycle/scope, but thread safety is the developer's responsibility.**

---

# Q23. What is Prototype scope?

```java
@Component
@Scope("prototype")
class ReportGenerator {
}
```

Unlike singleton:

```text
getBean()
   ↓
new instance

getBean()
   ↓
another instance

getBean()
   ↓
another instance
```

Example:

```java
ReportGenerator r1 =
    context.getBean(ReportGenerator.class);

ReportGenerator r2 =
    context.getBean(ReportGenerator.class);
```

Then:

```java
r1 != r2
```

---

# Q24. Singleton vs Prototype

| Singleton                           | Prototype                                                              |
| ----------------------------------- | ---------------------------------------------------------------------- |
| Default scope                       | Explicitly configured                                                  |
| One instance per ApplicationContext | New instance when requested from container                             |
| Shared between consumers            | Separate instances                                                     |
| Good for stateless services         | Useful for stateful objects                                            |
| Lifecycle fully managed differently | Spring does not manage full destruction lifecycle of prototype objects |

That last point is a good interview follow-up.

---

# Q25. What happens if a Singleton depends on a Prototype?

🔥 **Very common tricky question.**

Suppose:

```java
@Component
@Scope("prototype")
class PrototypeService {
}
```

and:

```java
@Service
class SingletonService {

    private final PrototypeService prototypeService;

    SingletonService(PrototypeService prototypeService) {
        this.prototypeService = prototypeService;
    }
}
```

What happens?

You might think:

> Every time `SingletonService` uses it, Spring will create a new PrototypeService.

**No.**

Spring creates the singleton once.

During creation of the singleton, it obtains a prototype instance:

```text
SingletonService created
       ↓
PrototypeService #1 injected
       ↓
SingletonService keeps #1
```

Later:

```text
singleton.method()
       ↓
same PrototypeService #1
```

It does **not** automatically create a new prototype each time.

---

# Q26. How can a Singleton get a new Prototype instance every time?

One approach is `ObjectProvider`.

```java
@Service
class SingletonService {

    private final ObjectProvider<PrototypeService> provider;

    SingletonService(ObjectProvider<PrototypeService> provider) {
        this.provider = provider;
    }

    public void process() {
        PrototypeService service = provider.getObject();
    }
}
```

Now each call to:

```java
provider.getObject()
```

can obtain a new prototype instance.

Another approach is `@Lookup`, but know `ObjectProvider` well for interviews.

---

# Q27. What is the lifecycle of a Spring Bean?

At a high level:

```text
Bean Definition
      ↓
Instantiate object
      ↓
Populate dependencies
      ↓
Aware callbacks
      ↓
BeanPostProcessor - before initialization
      ↓
@PostConstruct
      ↓
InitializingBean / init method
      ↓
BeanPostProcessor - after initialization
      ↓
Bean ready
      ↓
Application runs
      ↓
Context shutdown
      ↓
@PreDestroy / destroy callbacks
```

We'll cover this **properly in the next part**, because bean lifecycle is important enough to deserve its own section.

---

## 🎯 Part B complete

So far:

```text
Spring Bean
   ↓
@Component
   ↓
@Service / Repository / Controller
   ↓
Component Scanning
   ↓
@Bean
   ↓
@Configuration
   ↓
Bean Scopes
   ↓
Singleton
   ↓
Prototype
   ↓
Singleton + Prototype trap
   ↓
Thread safety
```

### Next: **Spring Core — Part C: Dependency Injection + Bean Lifecycle**

We'll hit the questions interviewers love to use as follow-ups:

* How exactly does `@Autowired` resolve a bean?
* What happens with 2, 3, or 10 beans of the same interface?
* `@Primary` vs `@Qualifier`
* Injecting `List<PaymentService>`
* Circular dependency
* Why constructor injection exposes circular dependency
* `@Lazy`
* Complete Bean Lifecycle
* `BeanPostProcessor`
* `BeanFactoryPostProcessor`
* `@PostConstruct`
* `@PreDestroy`

That'll take us deeper without jumping ahead to Spring Boot.
