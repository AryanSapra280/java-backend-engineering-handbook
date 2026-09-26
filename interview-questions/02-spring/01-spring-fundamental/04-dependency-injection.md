Absolutely! 🔥 This section is where Spring DI starts becoming **interview-level rather than just definitions**. The most important thing to understand is **how Spring chooses a bean when there are multiple candidates**.

---

# 🟢 D. Dependency Injection — Deeper Questions

---

## 63. How does Spring resolve a dependency by type?

Suppose we have:

```java
@Service
public class PaymentService {

    private final PaymentProcessor processor;

    public PaymentService(PaymentProcessor processor) {
        this.processor = processor;
    }
}
```

Spring sees:

```text
PaymentService needs
        ↓
PaymentProcessor
        ↓
Search container for beans of this type
```

If there is only **one** matching bean:

```java
@Component
public class RazorpayProcessor
        implements PaymentProcessor {
}
```

Spring injects it.

### Simple rule

> **First, Spring looks for a matching bean by type.**

If exactly one candidate exists:

```text
PaymentProcessor
      ↓
RazorpayProcessor
      ↓
Inject
```

If multiple candidates exist, Spring needs more information.

### Interview answer

> Spring primarily resolves autowired dependencies by type. If there is exactly one matching bean, it injects that bean. If multiple beans match, Spring uses additional rules such as `@Primary` or `@Qualifier` to select one.

---

# 64. What happens if two beans have the same interface?

Suppose:

```java
public interface PaymentProcessor {
    void process();
}
```

And:

```java
@Component
public class RazorpayProcessor
        implements PaymentProcessor {
}
```

```java
@Component
public class StripeProcessor
        implements PaymentProcessor {
}
```

Now:

```java
public PaymentService(PaymentProcessor processor) {
}
```

Spring finds:

```text
PaymentProcessor
       ↓
 ┌─────┴─────┐
 ↓           ↓
Razorpay    Stripe
```

Spring doesn't know which one you want.

So application startup fails with an ambiguity error, typically:

```text
NoUniqueBeanDefinitionException
```

### How do we solve it?

#### `@Primary`

```java
@Primary
@Component
public class RazorpayProcessor
        implements PaymentProcessor {
}
```

Now Razorpay becomes the default.

#### `@Qualifier`

```java
public PaymentService(
    @Qualifier("stripeProcessor")
    PaymentProcessor processor) {
}
```

Now Spring explicitly chooses Stripe.

### Interview answer

> If two beans implement the same interface and I inject only the interface type, Spring finds multiple candidates and cannot choose between them. I can resolve that using `@Primary` or `@Qualifier`.

---

# 65. What happens if three beans implement the same interface?

Exactly the same concept.

Suppose:

```text
PaymentProcessor
       │
 ┌─────┼──────┐
 ↓     ↓      ↓
Razor  Stripe  PayPal
```

And:

```java
public PaymentService(PaymentProcessor processor) {
}
```

Spring sees **three candidates**.

If none is marked `@Primary` and no `@Qualifier` is provided:

```text
NoUniqueBeanDefinitionException
```

### You can have:

```java
@Primary
@Component
class RazorpayProcessor
```

Then:

```text
PaymentProcessor
       ↓
Razorpay ← selected
Stripe
PayPal
```

Or explicitly:

```java
public PaymentService(
    @Qualifier("paypalProcessor")
    PaymentProcessor processor) {
}
```

### Important

Having 2 or 3 or 10 implementations doesn't fundamentally change the rule.

> **Multiple candidates → Spring needs a way to select one.**

---

# 66. How does `@Qualifier` work?

`@Qualifier` tells Spring:

> **"From all beans matching this type, I specifically want this one."**

Example:

```java
@Component("razorpay")
public class RazorpayProcessor
        implements PaymentProcessor {
}
```

```java
@Component("stripe")
public class StripeProcessor
        implements PaymentProcessor {
}
```

Then:

```java
@Service
public class PaymentService {

    private final PaymentProcessor processor;

    public PaymentService(
            @Qualifier("stripe")
            PaymentProcessor processor) {

        this.processor = processor;
    }
}
```

Spring effectively does:

```text
Need PaymentProcessor
       ↓
Find candidates
       ↓
Razorpay
Stripe
       ↓
@Qualifier("stripe")
       ↓
StripeProcessor
```

### Interview answer

> `@Qualifier` narrows down the candidates when multiple beans match the required type. It tells Spring exactly which bean should be injected.

---

# 67. Can `@Qualifier` be used with constructor injection?

**Yes — and this is very common.**

```java
@Service
public class PaymentService {

    private final PaymentProcessor processor;

    public PaymentService(
            @Qualifier("stripe")
            PaymentProcessor processor) {

        this.processor = processor;
    }
}
```

Spring sees:

```text
PaymentProcessor
      ↓
Multiple candidates
      ↓
@Qualifier("stripe")
      ↓
StripeProcessor
```

### This is actually a good combination

You get:

* Constructor injection
* Explicit dependency
* Explicit implementation selection

### Interview answer

> Yes. `@Qualifier` can be placed on a constructor parameter, which is a common way to select a specific implementation while still using constructor injection.

---

# 68. What happens if a constructor has multiple dependencies?

Suppose:

```java
@Service
public class OrderService {

    private final PaymentProcessor paymentProcessor;
    private final OrderRepository orderRepository;
    private final NotificationService notificationService;

    public OrderService(
            PaymentProcessor paymentProcessor,
            OrderRepository orderRepository,
            NotificationService notificationService) {

        this.paymentProcessor = paymentProcessor;
        this.orderRepository = orderRepository;
        this.notificationService = notificationService;
    }
}
```

Spring resolves **each parameter separately**.

Conceptually:

```text
OrderService constructor
        │
        ├── PaymentProcessor
        │       ↓
        │   resolve bean
        │
        ├── OrderRepository
        │       ↓
        │   resolve bean
        │
        └── NotificationService
                ↓
            resolve bean
```

Then Spring calls the constructor.

```text
All dependencies resolved
          ↓
OrderService created
```

### If one dependency cannot be resolved?

The bean cannot be created.

For example:

```text
PaymentProcessor → found
OrderRepository → found
NotificationService → NOT FOUND
```

Then:

```text
OrderService creation fails
```

and the application context may fail to start.

### Interview answer

> Spring resolves each constructor parameter independently by type and applies qualifiers or primary candidates where required. Once all dependencies are resolved, it invokes the constructor. If a required dependency cannot be resolved, bean creation fails.

---

# 69. Can Spring inject primitive values?

**Yes.**

For example:

```java
@Value("${payment.timeout}")
private int timeout;
```

And in:

```text
application.properties
```

```properties
payment.timeout=5000
```

Spring converts the configuration value into the required type:

```text
"5000"
   ↓
int
   ↓
5000
```

You can also use:

```java
@Value("${payment.enabled}")
private boolean enabled;
```

or:

```java
@Value("${payment.maxAmount}")
private double maxAmount;
```

### What about constructor injection?

Yes:

```java
@Service
public class PaymentService {

    private final int timeout;

    public PaymentService(
            @Value("${payment.timeout}")
            int timeout) {

        this.timeout = timeout;
    }
}
```

### Interview answer

> Yes. Spring can inject primitive or simple values using mechanisms such as `@Value`, usually from application properties, environment variables, or other property sources.

---

# 70. How do you inject configuration values into a bean?

The simple answer is:

### `@Value`

Suppose:

```properties
payment.timeout=5000
payment.retryCount=3
```

Then:

```java
@Service
public class PaymentService {

    @Value("${payment.timeout}")
    private int timeout;

    @Value("${payment.retryCount}")
    private int retryCount;
}
```

You can also use constructor injection:

```java
@Service
public class PaymentService {

    private final int timeout;

    public PaymentService(
            @Value("${payment.timeout}")
            int timeout) {
        this.timeout = timeout;
    }
}
```

### But for multiple related properties...

Use:

```java
@ConfigurationProperties
```

which leads directly to the next question.

---

# 71. `@Value` vs `@ConfigurationProperties`?

🔥 Very important for Spring Boot interviews.

Imagine you have:

```properties
payment.timeout=5000
payment.retry-count=3
payment.enabled=true
payment.base-url=https://payment.example.com
```

---

## `@Value`

You can individually inject:

```java
@Value("${payment.timeout}")
private int timeout;

@Value("${payment.retry-count}")
private int retryCount;

@Value("${payment.enabled}")
private boolean enabled;
```

### Good for

Small number of individual properties.

---

## `@ConfigurationProperties`

Instead, create a configuration object:

```java
@ConfigurationProperties(prefix = "payment")
public class PaymentProperties {

    private int timeout;
    private int retryCount;
    private boolean enabled;
    private String baseUrl;

    // getters/setters
}
```

Then Spring binds:

```text
payment.timeout
payment.retry-count
payment.enabled
payment.base-url
```

into:

```text
PaymentProperties
```

Conceptually:

```text
application.properties
        ↓
@ConfigurationProperties
        ↓
PaymentProperties
        ↓
PaymentService
```

### Comparison

| `@Value`                          | `@ConfigurationProperties`          |
| --------------------------------- | ----------------------------------- |
| Individual values                 | Group of related properties         |
| Simple                            | Better for structured configuration |
| Good for few properties           | Good for many properties            |
| Can become messy with many values | Cleaner and easier to maintain      |
| Less type-safe structure          | Strongly typed configuration object |

### Interview answer

> I use `@Value` when I need a small number of individual properties. For a group of related configuration values, I prefer `@ConfigurationProperties` because it provides a structured, strongly typed configuration object and is easier to maintain.

### Example from a real backend application

Suppose your service has:

```properties
pf.batch.chunk-size=500
pf.batch.thread-count=8
pf.batch.retry-count=3
pf.batch.timeout=30s
```

Instead of having four separate `@Value`s everywhere, you can have:

```java
@ConfigurationProperties(prefix = "pf.batch")
public class BatchProperties {
    private int chunkSize;
    private int threadCount;
    private int retryCount;
    private Duration timeout;
}
```

Much cleaner.

---

# 72. How do you inject a list of implementations of an interface?

🔥 **This is an important question.**

Suppose:

```java
public interface PaymentProcessor {
    void process();
}
```

And we have:

```java
@Component
public class RazorpayProcessor
        implements PaymentProcessor {

    public void process() {
        System.out.println("Razorpay");
    }
}
```

```java
@Component
public class StripeProcessor
        implements PaymentProcessor {

    public void process() {
        System.out.println("Stripe");
    }
}
```

Now:

```java
@Service
public class PaymentService {

    private final List<PaymentProcessor> processors;

    public PaymentService(
            List<PaymentProcessor> processors) {

        this.processors = processors;
    }
}
```

Spring sees:

```text
Need:
List<PaymentProcessor>

Container has:
    RazorpayProcessor
    StripeProcessor
```

So Spring creates:

```text
List<PaymentProcessor>
        │
        ├── RazorpayProcessor
        └── StripeProcessor
```

and injects that list.

### You don't manually create the list.

Spring does it.

---

# How exactly does Spring populate the List?

This is the important part.

Suppose the container has:

```text
Bean #1 → RazorpayProcessor
Bean #2 → StripeProcessor
Bean #3 → PayPalProcessor
```

When Spring sees:

```java
List<PaymentProcessor> processors
```

it looks for **all beans assignable to `PaymentProcessor`**.

Then:

```text
PaymentProcessor beans
        ↓
┌──────────────────────┐
│ RazorpayProcessor    │
│ StripeProcessor      │
│ PayPalProcessor      │
└──────────────────────┘
        ↓
Create List
        ↓
Inject List
```

So:

```java
processors.size()
```

would be:

```text
3
```

assuming those three beans are registered.

---

# Can we use `Set` instead?

**Yes.**

```java
public PaymentService(
        Set<PaymentProcessor> processors) {
}
```

Spring can inject all matching beans into the set.

---

# Can we use `Map`?

🔥 Yes, and this is useful.

```java
public PaymentService(
        Map<String, PaymentProcessor> processors) {
}
```

Spring uses bean names as keys.

For example:

```text
Map

"razorpayProcessor" → RazorpayProcessor
"stripeProcessor"   → StripeProcessor
"paypalProcessor"   → PayPalProcessor
```

Then:

```java
processors.get("stripeProcessor");
```

gets the Stripe implementation.

---

# Can we control the order of the List?

**Yes.**

Use `@Order`.

```java
@Component
@Order(1)
public class RazorpayProcessor
        implements PaymentProcessor {
}
```

```java
@Component
@Order(2)
public class StripeProcessor
        implements PaymentProcessor {
}
```

Then Spring can provide the list in that order.

You can also use `Ordered`.

### Interview answer

> Spring can inject all beans matching an interface into a `List`, `Set`, or `Map`. For a list, it finds all beans assignable to that interface and injects them. The order can be controlled using `@Order` or `Ordered`.

---

# 🚀 Why is List Injection Useful?

This pattern is actually **very useful in real backend applications**.

Imagine you have different validation rules:

```java
public interface Validator {
    boolean validate(Request request);
}
```

Implementations:

```text
AmountValidator
AccountValidator
MemberValidator
DateValidator
```

Instead of:

```java
if (...) {
    ...
}

if (...) {
    ...
}

if (...) {
    ...
}
```

you can do:

```java
@Service
public class ValidationService {

    private final List<Validator> validators;

    public ValidationService(List<Validator> validators) {
        this.validators = validators;
    }

    public void validate(Request request) {

        for (Validator validator : validators) {
            validator.validate(request);
        }
    }
}
```

Now adding a new validator doesn't require modifying `ValidationService`.

Just create:

```java
@Component
public class NewValidator implements Validator {
}
```

Spring automatically adds it to the list.

That's a nice example of **Dependency Injection + loose coupling + Open/Closed Principle** working together.

---

# 🧠 The DI Decision Tree

This is probably the most useful thing to memorize from this section:

```text
Spring needs dependency
        │
        ↓
Find beans by TYPE
        │
        ├── 0 beans
        │      ↓
        │   Dependency error
        │
        ├── 1 bean
        │      ↓
        │   Inject it
        │
        ├── Multiple beans
        │      ↓
        │   @Primary / @Qualifier
        │
        └── Collection requested
               ↓
        Find ALL matching beans
               ↓
          List / Set / Map
```

### Example

```java
PaymentProcessor processor
```

means:

> **Give me ONE PaymentProcessor.**

While:

```java
List<PaymentProcessor> processors
```

means:

> **Give me ALL PaymentProcessor beans.**

And:

```java
Map<String, PaymentProcessor> processors
```

means:

> **Give me ALL PaymentProcessor beans, indexed by bean name.**

---

# 🎯 10 Answers to Remember for the Interview

If you revise this section just before your interview, remember these:

**63. How does Spring resolve dependency?**

> Primarily by type.

**64. Two implementations?**

> Ambiguous → use `@Primary` or `@Qualifier`.

**65. Three implementations?**

> Same rule; multiple candidates need to be resolved.

**66. `@Qualifier`?**

> Selects a specific bean among multiple candidates.

**67. Qualifier with constructor?**

> Yes, put it on the constructor parameter.

**68. Multiple constructor dependencies?**

> Spring resolves each parameter separately and then calls the constructor.

**69. Primitive values?**

> Yes, using `@Value` or configuration properties.

**70. Configuration values?**

> `@Value` for individual values; `@ConfigurationProperties` for grouped configuration.

**71. `@Value` vs `@ConfigurationProperties`?**

> `@Value` = individual properties; `@ConfigurationProperties` = structured/grouped configuration.

**72. `List<PaymentProcessor>`?**

> Spring finds all beans implementing `PaymentProcessor` and injects them into the list.

---

## 🔥 One thing I want you to be able to explain naturally

If an interviewer writes:

```java
public PaymentService(
        List<PaymentProcessor> processors) {
}
```

and asks:

> **"How does Spring know what to put in this list?"**

Don't say merely:

> "Spring uses dependency injection."

Say:

> **"Spring sees that the dependency is a `List<PaymentProcessor>`. It searches the application context for all beans whose type is assignable to `PaymentProcessor`, collects those bean instances into a list, and injects that list into the constructor. If I use a `Map`, Spring uses the bean names as the keys."**

That's a **strong, clear SDE-2-level answer** without being unnecessarily complicated.
