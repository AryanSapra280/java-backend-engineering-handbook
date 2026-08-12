# Factory Design Pattern

> **Core idea:** Factory Pattern centralizes object creation so that the code using an object does not need to know which concrete class to instantiate.

---

# 1. Intent

The Factory Pattern is used when object creation involves a decision.

Instead of writing:

```java
if (type == UPI) {
    return new UPIPayment();
} else if (type == CARD) {
    return new CardPayment();
}
```

everywhere in the application, we move the creation responsibility into a dedicated place:

```text
Client
   |
   ↓
Factory
   |
   +---- creates UPI
   +---- creates Card
   +---- creates PayPal
```

The client asks:

> "Give me the appropriate object."

The client does not need to know:

- which concrete class to instantiate
- which constructor to call
- what dependencies are required
- how creation should be performed

---

# 2. Problem

Imagine a notification system.

We support:

```text
EMAIL
SMS
PUSH
```

We define:

```java
public interface Notification {

    void send(String message);
}
```

Implementations:

```java
public class EmailNotification
        implements Notification {

    @Override
    public void send(String message) {
        System.out.println(
                "Sending Email: " + message);
    }
}
```

```java
public class SmsNotification
        implements Notification {

    @Override
    public void send(String message) {
        System.out.println(
                "Sending SMS: " + message);
    }
}
```

```java
public class PushNotification
        implements Notification {

    @Override
    public void send(String message) {
        System.out.println(
                "Sending Push: " + message);
    }
}
```

Now somewhere in our application:

```java
public class NotificationService {

    public void send(
            NotificationType type,
            String message) {

        Notification notification;

        if (type == NotificationType.EMAIL) {

            notification =
                    new EmailNotification();

        } else if (type == NotificationType.SMS) {

            notification =
                    new SmsNotification();

        } else if (type == NotificationType.PUSH) {

            notification =
                    new PushNotification();

        } else {

            throw new IllegalArgumentException(
                    "Unsupported notification type");
        }

        notification.send(message);
    }
}
```

This works.

But now imagine this creation logic appears in:

```text
NotificationService
OrderService
CampaignService
ReminderService
```

Now we have the same creation logic duplicated everywhere.

---

# 3. Naive Solution

The simplest implementation is:

```java
if (type == NotificationType.EMAIL) {
    return new EmailNotification();

} else if (type == NotificationType.SMS) {
    return new SmsNotification();

} else if (type == NotificationType.PUSH) {
    return new PushNotification();
}
```

This is not automatically bad.

For three tiny classes and one usage point, this may be perfectly fine.

The problem appears when object creation becomes:

- repeated
- complicated
- dependent on many conditions
- dependent on configuration
- dependent on multiple constructor arguments
- spread across many places

---

# 4. What Smell / Problem Appears?

## 4.1 Creation logic leaks into business logic

The service now knows:

```text
EmailNotification
SmsNotification
PushNotification
```

and knows how to create them.

That means the service is doing two jobs:

```text
Business operation
+
Object creation
```

---

## 4.2 Repeated creation logic

Suppose three services need Notifications.

Without a Factory:

```text
OrderService
    ↓
if EMAIL → new EmailNotification()
if SMS   → new SmsNotification()

CampaignService
    ↓
if EMAIL → new EmailNotification()
if SMS   → new SmsNotification()

ReminderService
    ↓
if EMAIL → new EmailNotification()
if SMS   → new SmsNotification()
```

Now changing creation logic requires changes in multiple places.

---

## 4.3 Concrete classes leak everywhere

Instead of depending on:

```java
Notification
```

we start seeing:

```java
new EmailNotification()
new SmsNotification()
new PushNotification()
```

throughout the codebase.

The application becomes tightly coupled to concrete implementations.

---

## 4.4 Creation becomes complicated

Imagine:

```java
new EmailNotification(
        smtpClient,
        templateService,
        retryPolicy,
        encryptionService,
        configuration
);
```

Now callers need to understand all of that.

A Factory can hide the construction process.

---

# 5. Core Idea

Move object creation into a Factory.

```java
public class NotificationFactory {

    public static Notification create(
            NotificationType type) {

        if (type == NotificationType.EMAIL) {

            return new EmailNotification();

        } else if (type == NotificationType.SMS) {

            return new SmsNotification();

        } else if (type == NotificationType.PUSH) {

            return new PushNotification();
        }

        throw new IllegalArgumentException(
                "Unsupported notification type");
    }
}
```

Now the Service becomes:

```java
public class NotificationService {

    public void send(
            NotificationType type,
            String message) {

        Notification notification =
                NotificationFactory.create(type);

        notification.send(message);
    }
}
```

The `NotificationService` no longer knows:

```text
new EmailNotification()
new SmsNotification()
new PushNotification()
```

It only knows:

```text
NotificationFactory
```

---

# 6. Structure

Basic structure:

```text
                 Client
                   |
                   ↓
                Factory
                   |
        +----------+----------+
        |          |          |
        ↓          ↓          ↓
     Product A  Product B  Product C
```

For our example:

```text
NotificationService
        |
        ↓
NotificationFactory
        |
   +----+----+----+
   |    |    |
   ↓    ↓    ↓
 Email SMS  Push
```

---

# 7. Java Implementation

## 7.1 Product Interface

```java
public interface Notification {

    void send(String message);
}
```

---

## 7.2 Concrete Products

```java
public class EmailNotification
        implements Notification {

    @Override
    public void send(String message) {

        System.out.println(
                "Sending Email: " + message);
    }
}
```

```java
public class SmsNotification
        implements Notification {

    @Override
    public void send(String message) {

        System.out.println(
                "Sending SMS: " + message);
    }
}
```

```java
public class PushNotification
        implements Notification {

    @Override
    public void send(String message) {

        System.out.println(
                "Sending Push: " + message);
    }
}
```

---

## 7.3 Type

```java
public enum NotificationType {

    EMAIL,
    SMS,
    PUSH
}
```

---

## 7.4 Factory

```java
public class NotificationFactory {

    public static Notification create(
            NotificationType type) {

        if (type == NotificationType.EMAIL) {

            return new EmailNotification();

        } else if (type == NotificationType.SMS) {

            return new SmsNotification();

        } else if (type == NotificationType.PUSH) {

            return new PushNotification();
        }

        throw new IllegalArgumentException(
                "Unsupported notification type: "
                + type);
    }
}
```

---

## 7.5 Client

```java
public class NotificationService {

    public void send(
            NotificationType type,
            String message) {

        Notification notification =
                NotificationFactory.create(type);

        notification.send(message);
    }
}
```

Usage:

```java
NotificationService service =
        new NotificationService();

service.send(
        NotificationType.EMAIL,
        "Your order has shipped");
```

The service does not know:

```java
new EmailNotification();
```

The Factory does.

---

# 8. Important: Factory Does NOT Necessarily Remove `if-else`

This is one of the most important things to understand.

A beginner might think:

> "Factory means no if-else."

Not true.

Our Factory itself contains:

```java
if (type == EMAIL) {
    return new EmailNotification();
}
else if (type == SMS) {
    return new SmsNotification();
}
```

We have **centralized** the conditional creation logic.

We have not magically removed it.

The improvement is:

```text
Before:

Service A → if/switch → creation
Service B → if/switch → creation
Service C → if/switch → creation


After:

Service A ──┐
Service B ──┼──→ Factory → creation
Service C ──┘
```

The creation decision now has one owner.

---

# 9. Why Is That Better?

Suppose the Email construction changes from:

```java
new EmailNotification();
```

to:

```java
new EmailNotification(
        smtpClient,
        templateService,
        retryPolicy);
```

Only the Factory needs to know.

The callers still do:

```java
NotificationFactory.create(
        NotificationType.EMAIL);
```

This is the main benefit:

> **Encapsulation of object creation.**

---

# 10. Factory vs Strategy

This distinction is extremely important because you just learned Strategy.

## Strategy

Answers:

> **How should I perform this operation?**

```text
Payment
   |
   +-- UPI Strategy
   +-- Card Strategy
   +-- PayPal Strategy
```

The Strategy contains the behavior.

```java
strategy.pay(amount);
```

---

## Factory

Answers:

> **Which object should I create?**

```text
PaymentType
    ↓
Factory
    ↓
UPIPayment object
```

The Factory contains object creation/selection logic.

---

## Together

Factory and Strategy can work together:

```text
Request
   |
   ↓
Factory / Registry
   |
   ↓
Strategy object
   |
   ↓
execute()
```

Example:

```java
PaymentStrategy strategy =
        PaymentStrategyFactory.create(type);

strategy.pay(amount);
```

Here:

```text
Factory → creates/selects Strategy
Strategy → performs behavior
```

---

# 11. Factory vs Registry

This is particularly important in Spring.

## Factory

The fundamental question is:

> "Which object should I create?"

Example:

```java
PaymentStrategy create(PaymentType type);
```

The Factory may do:

```java
return new UPIPaymentStrategy();
```

---

## Registry

The question is:

> "Which already-existing object should I use?"

Example:

```java
PaymentStrategy getStrategy(PaymentType type);
```

The Registry does:

```java
return strategyMap.get(type);
```

There is no `new`.

---

## Mental model

```text
Factory:

type
 ↓
create
 ↓
new object


Registry:

type
 ↓
lookup
 ↓
existing object
```

---

# 12. Simple Factory

The code we wrote above is commonly called a **Simple Factory**.

Example:

```java
public class NotificationFactory {

    public static Notification create(
            NotificationType type) {

        switch (type) {

            case EMAIL:
                return new EmailNotification();

            case SMS:
                return new SmsNotification();

            case PUSH:
                return new PushNotification();

            default:
                throw new IllegalArgumentException(
                        "Unsupported type");
        }
    }
}
```

Important:

> "Simple Factory" is a common design technique, but it is not the same thing as the formal GoF **Factory Method** pattern.

This distinction matters in interviews.

---

# 13. Factory Method

Factory Method takes the creation responsibility into subclasses.

The key idea is:

```text
Creator
   |
   +-- Factory Method
   |
   +-- Concrete Creator A
   +-- Concrete Creator B
```

Example:

```java
public interface Notification {

    void send(String message);
}
```

Creator:

```java
public abstract class NotificationCreator {

    public abstract Notification createNotification();

    public void send(String message) {

        Notification notification =
                createNotification();

        notification.send(message);
    }
}
```

Concrete Creator:

```java
public class EmailNotificationCreator
        extends NotificationCreator {

    @Override
    public Notification createNotification() {

        return new EmailNotification();
    }
}
```

Another:

```java
public class SmsNotificationCreator
        extends NotificationCreator {

    @Override
    public Notification createNotification() {

        return new SmsNotification();
    }
}
```

Now:

```text
NotificationCreator
        |
        +-- EmailNotificationCreator
        |
        +-- SmsNotificationCreator
```

Each concrete creator decides what product to create.

---

# 14. Simple Factory vs Factory Method

## Simple Factory

```text
Factory
   |
   +-- if EMAIL → new Email
   +-- if SMS → new SMS
```

One Factory decides.

---

## Factory Method

```text
Creator
   |
   +-- EmailCreator → creates Email
   |
   +-- SmsCreator → creates SMS
```

The subclasses decide.

---

## Interview distinction

If asked:

> "Does Factory Pattern always mean Factory Method?"

Answer:

> No. "Factory" is often used as a general term for encapsulating object creation. Simple Factory centralizes creation in one class, while Factory Method is a GoF pattern where subclasses override a factory method to determine which product gets created.

---

# 15. When to Use Factory

Use a Factory when:

### 1. Object creation has a decision

```text
type → object
```

### 2. Creation logic is repeated

Multiple classes need the same creation logic.

### 3. Construction is complicated

For example:

```java
new PaymentProcessor(
        gateway,
        retryPolicy,
        configuration,
        logger,
        metrics);
```

### 4. You want to hide concrete classes

Consumers should depend on:

```java
PaymentProcessor
```

instead of:

```java
UPIPaymentProcessor
```

### 5. You expect more implementations

```text
PDF
Excel
CSV
JSON
```

and creation differs.

---

# 16. When NOT to Use Factory

Do not create a Factory for every `new`.

This:

```java
User user = new User();
```

does not need:

```java
UserFactory
```

unless there is a meaningful creation decision or complex construction logic.

Bad overengineering:

```text
UserFactory
UserBuilderFactory
AddressFactory
StringFactory
IntegerFactory
```

Patterns are tools, not requirements.

---

# 17. SOLID Connection

## Single Responsibility Principle

The Factory owns:

```text
object creation
```

The Service owns:

```text
business operation
```

The Product owns:

```text
product behavior
```

---

## Open/Closed Principle

This depends on the Factory design.

A simple `switch` Factory still requires modification when adding a new type:

```java
case NEW_TYPE:
    return new NewProduct();
```

So a Simple Factory is **not automatically perfectly Open/Closed**.

This is a subtle but important point.

Factory can improve separation of concerns without completely eliminating modification.

---

## Dependency Inversion Principle

The caller can depend on:

```java
Notification
```

rather than:

```java
EmailNotification
```

The Factory becomes the place where the concrete implementation is selected.

---

# 18. Factory and Dependency Inversion

Without Factory:

```java
public class OrderService {

    private EmailNotification email;

    public OrderService() {

        email =
            new EmailNotification();
    }
}
```

The high-level class knows the concrete implementation.

With Factory:

```java
public class OrderService {

    public void notifyUser(
            NotificationType type) {

        Notification notification =
                NotificationFactory.create(type);

        notification.send("Order shipped");
    }
}
```

The service depends on:

```text
Notification
```

and delegates creation.

---

# 19. Spring Boot and Factory

Now we reach the interesting part.

In a Spring application, we often don't need to manually create objects because Spring already manages object creation.

For example:

```java
@Component
public class EmailNotification
        implements Notification {
}
```

Spring creates:

```text
EmailNotification bean
```

So writing:

```java
new EmailNotification()
```

inside a Factory can actually bypass Spring dependency injection.

Suppose:

```java
@Component
public class EmailNotification
        implements Notification {

    private final EmailClient emailClient;

    public EmailNotification(
            EmailClient emailClient) {

        this.emailClient = emailClient;
    }
}
```

If you do:

```java
new EmailNotification(...)
```

you now have to manually supply:

```text
EmailClient
```

and its dependencies.

This is usually undesirable in Spring.

---

# 20. Spring Strategy vs Traditional Factory

This is where your previous Strategy learning becomes extremely useful.

### Traditional Factory

```text
Factory
   |
   +-- new UPI()
   +-- new Card()
   +-- new PayPal()
```

### Spring Strategy Registry

```text
Spring
   |
   +-- creates UPI bean
   +-- creates Card bean
   +-- creates PayPal bean
            |
            ↓
      Registry receives them
            |
            ↓
      Map<Type, Strategy>
```

Then:

```java
strategyMap.get(type);
```

returns an existing object.

There is no:

```java
new
```

during runtime selection.

---

# 21. Factory vs Spring Dependency Injection

This is a critical interview concept.

A Factory solves:

> **How should I decide/create a concrete object?**

Spring DI solves:

> **How should objects be instantiated, configured, wired, and managed?**

They overlap in some areas, but they are not the same abstraction.

---

## Factory

```text
Factory
   ↓
create object
```

---

## Spring

```text
Spring Container
   ↓
create beans
   ↓
resolve dependencies
   ↓
manage lifecycle
   ↓
inject beans
```

If Spring already manages the possible implementations, a Registry/Map of injected beans may be more natural than a Factory that manually calls `new`.

---

# 22. Spring Factory Example

Suppose we still want a Factory abstraction.

We could inject the implementations into it:

```java
@Component
public class NotificationFactory {

    private final Map<
            NotificationType,
            Notification> notifications;

    public NotificationFactory(
            List<Notification> notifications) {

        this.notifications =
                notifications.stream()
                        .collect(Collectors.toMap(
                                Notification::getType,
                                Function.identity()
                        ));
    }

    public Notification get(
            NotificationType type) {

        Notification notification =
                notifications.get(type);

        if (notification == null) {

            throw new IllegalArgumentException(
                    "Unsupported type: " + type);
        }

        return notification;
    }
}
```

Notice something important:

```text
Factory
```

is no longer doing:

```java
new EmailNotification();
```

It is doing:

```java
map.get(type);
```

At this point, conceptually, this is very close to what we called a **Registry**.

This is why terminology matters less than understanding the responsibility.

---

# 23. Factory + Strategy Together

This is a very common design.

Suppose:

```java
public interface PaymentStrategy {

    void pay(double amount);
}
```

Implementations:

```text
UPIPaymentStrategy
CardPaymentStrategy
PayPalPaymentStrategy
```

A Factory can create the Strategy:

```java
public class PaymentStrategyFactory {

    public static PaymentStrategy create(
            PaymentType type) {

        switch (type) {

            case UPI:
                return new UPIPaymentStrategy();

            case CARD:
                return new CardPaymentStrategy();

            case PAYPAL:
                return new PayPalPaymentStrategy();

            default:
                throw new IllegalArgumentException(
                        "Unsupported type");
        }
    }
}
```

Then:

```java
PaymentStrategy strategy =
        PaymentStrategyFactory.create(type);

strategy.pay(amount);
```

The responsibilities are:

```text
Factory
    ↓
Which Strategy object?

Strategy
    ↓
How should payment happen?
```

This is a very useful combination.

---

# 24. But Don't Force Factory + Strategy Together

They solve different problems.

You don't automatically need:

```text
Factory + Strategy
```

just because you have Strategy.

If Spring can provide:

```java
List<PaymentStrategy>
```

and a Registry can select one:

```java
map.get(type)
```

then adding a separate Factory may just add another layer.

Use:

```text
Factory
```

when creation is the problem.

Use:

```text
Strategy
```

when behavior variation is the problem.

Use:

```text
Registry
```

when lookup of already-existing implementations is the problem.

---

# 25. Real-World Examples

## Notification

```text
EMAIL
SMS
PUSH
```

Factory:

```text
NotificationType
      ↓
NotificationFactory
      ↓
Notification object
```

---

## Database Connection

```text
MySQL
PostgreSQL
Oracle
MongoDB
```

Creation may vary based on configuration.

---

## File Parser

```text
CSV
JSON
XML
```

```text
FileType
   ↓
ParserFactory
   ↓
CSVParser / JSONParser / XMLParser
```

---

## Payment Processor

```text
UPI
CARD
WALLET
PAYPAL
```

---

## Vehicle

```text
CAR
TRUCK
BIKE
```

---

# 26. LLD Examples

Factory commonly appears in:

### Parking Lot

Vehicle creation:

```text
CAR
BIKE
TRUCK
```

could be handled through a factory.

```text
VehicleFactory
     |
     +-- Car
     +-- Bike
     +-- Truck
```

---

### Chess

Creating pieces:

```text
KING
QUEEN
ROOK
BISHOP
KNIGHT
PAWN
```

---

### Notification System

```text
EMAIL
SMS
PUSH
```

---

### Payment System

```text
UPI
CARD
WALLET
```

---

### Document Processing

```text
PDF
DOCX
XLSX
```

---

# 27. How to Recognize Factory

When solving an LLD problem, ask:

### Question 1

> Do I have multiple concrete implementations of an abstraction?

```text
Vehicle
  |
  +-- Car
  +-- Bike
  +-- Truck
```

### Question 2

> Is the caller deciding which concrete implementation to instantiate?

```java
if (type == CAR)
    new Car();
```

### Question 3

> Is creation logic duplicated or becoming complicated?

If yes, consider Factory.

### Question 4

> Should the caller know about the concrete classes?

If no, Factory is a candidate.

### Question 5

> Is the problem creation or behavior?

If:

```text
Which object?
```

think:

```text
Factory
```

If:

```text
Which algorithm?
```

think:

```text
Strategy
```

---

# 28. Common Mistakes

## Mistake 1: Thinking Factory removes all `if-else`

It doesn't.

A Simple Factory may contain the `if/switch`.

The improvement is:

```text
centralized creation
```

not necessarily:

```text
zero conditionals
```

---

## Mistake 2: Creating a Factory for every class

Don't do:

```java
UserFactory.create();
```

for a simple:

```java
new User();
```

---

## Mistake 3: Factory becoming a God class

Bad:

```text
ApplicationFactory
    ↓
creates 40 unrelated object types
```

Factories should generally have a coherent creation responsibility.

---

## Mistake 4: Manually using `new` in Spring for managed beans

If the class is:

```java
@Component
```

and has dependencies, manually doing:

```java
new SomeComponent(...)
```

can bypass Spring's dependency injection/lifecycle management.

---

## Mistake 5: Calling a Registry a Factory without understanding the difference

If it does:

```java
map.get(type)
```

and returns an existing object, it is conceptually a Registry/lookup mechanism.

If it does:

```java
new ConcreteProduct()
```

it is doing creation.

---

# 29. Factory vs Builder

These are also frequently confused.

## Factory

Answers:

> Which object should I create?

Example:

```text
type = CAR
      ↓
CarFactory
      ↓
Car
```

---

## Builder

Answers:

> How should I construct this complex object step by step?

Example:

```java
Car car =
        new Car.Builder()
                .engine("V8")
                .color("Black")
                .sunroof(true)
                .build();
```

Mental model:

```text
Factory
    ↓
CHOICE


Builder
    ↓
CONSTRUCTION
```

---

# 30. Factory vs Abstract Factory

Simple Factory:

```text
one product hierarchy
```

Example:

```text
Notification
   |
   +-- Email
   +-- SMS
```

Abstract Factory is useful when you need to create **families of related objects**.

Example:

```text
UI Theme

Windows UI Factory
    |
    +-- WindowsButton
    +-- WindowsCheckbox

Mac UI Factory
    |
    +-- MacButton
    +-- MacCheckbox
```

The factory creates related products that are designed to work together.

---

# 31. Factory Method vs Abstract Factory

### Factory Method

Usually focuses on creating **one product type** through an overridable creation method.

```text
Creator
   ↓
Product
```

### Abstract Factory

Creates a **family of related products**.

```text
AbstractFactory
   |
   +-- createButton()
   +-- createCheckbox()
   +-- createMenu()
```

---

# 32. Interview Questions

## Q1. What is Factory Pattern?

Strong answer:

> Factory Pattern encapsulates object creation and prevents client code from being tightly coupled to concrete implementations. The client requests an abstraction or product from the Factory without needing to know the details of construction.

---

## Q2. Does Factory remove if-else?

No.

A Simple Factory may still use:

```java
if
switch
```

The main benefit is centralizing creation logic.

---

## Q3. Factory vs Strategy?

> Factory is about object creation/selection. Strategy is about encapsulating interchangeable behavior.

---

## Q4. Factory vs Builder?

> Factory decides which object/product to create. Builder handles step-by-step construction of a potentially complex object.

---

## Q5. Factory vs Registry?

> Factory generally creates or constructs the requested object. Registry looks up an already-created object.

---

## Q6. Why not simply use `new`?

Because direct construction can:

- spread creation logic
- couple clients to concrete classes
- make complex construction visible everywhere
- make future changes harder

But simple `new` is perfectly fine when creation is trivial and stable.

---

## Q7. Is Simple Factory a GoF pattern?

Not formally.

The **Factory Method** is a GoF Creational Design Pattern.

"Simple Factory" is a commonly used design technique.

---

## Q8. How does Factory work with Spring?

A strong answer:

> In Spring, I first consider whether I need a Factory at all because Spring already manages object creation. If all implementations are Spring beans and I need runtime selection, injecting them into a Map or List and using a Registry is often cleaner. If I have custom creation logic beyond simple lookup, a Factory can still be useful, but it should generally work with Spring-managed dependencies rather than manually creating managed beans.

---

# 33. Factory and Spring: Senior-Level Thinking

This is the most important practical section.

When you see:

```java
@Component
class A implements Product {}

@Component
class B implements Product {}

@Component
class C implements Product {}
```

you should immediately think:

```text
Spring already owns object creation.
```

So don't blindly write:

```java
new A()
new B()
new C()
```

inside a Factory.

Instead consider:

```text
Can Spring inject them?

       ↓

List<Product>

       ↓

Map<Key, Product>

       ↓

Registry lookup
```

If creation itself is complex and requires decisions, a Factory can still be appropriate.

The question is not:

> "Can I use Factory?"

The question is:

> "Where is the creation complexity, and who should own it?"

---

# 34. Factory Decision Framework

When you encounter a creation problem:

```text
START
  |
  ↓
Is object creation trivial?
  |
  +-- YES → use new
  |
  +-- NO
       |
       ↓
Does creation involve choosing
between multiple concrete classes?
       |
       +-- YES → Factory candidate
       |
       ↓
Are the objects already Spring-managed?
       |
       +-- YES → consider Registry / DI
       |
       ↓
Is construction complex with many optional fields?
       |
       +-- YES → Builder candidate
```

This prevents blindly applying patterns.

---

# 35. Factory + Builder

Factory and Builder can also work together.

Suppose:

```text
VehicleFactory
```

decides:

```text
CAR
```

and then a Builder handles the complex construction:

```java
Vehicle car =
        VehicleFactory.create(
                VehicleType.CAR);
```

Internally:

```text
Factory
   ↓
chooses CarBuilder
   ↓
Builder
   ↓
constructs complex Car
```

So:

```text
Factory
    → WHICH object?

Builder
    → HOW to construct it?
```

---

# 36. Final Mental Model

Remember these four questions:

```text
1. WHICH OBJECT?
       ↓
     Factory

2. WHICH BEHAVIOR?
       ↓
     Strategy

3. HOW TO BUILD A COMPLEX OBJECT?
       ↓
     Builder

4. WHICH EXISTING OBJECT SHOULD I USE?
       ↓
     Registry
```

This is an extremely useful LLD mental model.

---

# 37. Factory in One Diagram

```text
                    CLIENT
                      |
                      | "I need a Notification"
                      ↓
                  FACTORY
                      |
               creation decision
                      |
             +--------+--------+
             |        |        |
             ↓        ↓        ↓
           Email      SMS      Push
             |        |        |
             +--------+--------+
                      |
                      ↓
               Notification
```

The Client doesn't need to know how the concrete object is constructed.

---

# 38. Factory + Strategy Diagram

```text
                         REQUEST
                            |
                            ↓
                         FACTORY
                            |
                    selects/creates
                            |
                            ↓
                    PaymentStrategy
                            |
              +-------------+-------------+
              |             |             |
              ↓             ↓             ↓
            UPI           CARD          PAYPAL
          Strategy       Strategy       Strategy
              |             |             |
              +-------------+-------------+
                            |
                            ↓
                         execute()
```

Factory answers:

```text
"Which Strategy object?"
```

Strategy answers:

```text
"How should payment be performed?"
```

---

# 39. Factory + Spring Diagram

```text
                      SPRING
                        |
          +-------------+-------------+
          |             |             |
          ↓             ↓             ↓
       Product A     Product B     Product C
          |             |             |
          +-------------+-------------+
                        |
                        ↓
                     Registry
                        |
                     map.get()
                        |
                        ↓
                 Existing Object
```

Versus a traditional Factory:

```text
                       CLIENT
                         |
                         ↓
                      FACTORY
                         |
                   if / switch
                         |
              +----------+----------+
              |          |          |
              ↓          ↓          ↓
             new A      new B      new C
```

The difference is:

```text
Traditional Factory
    → creates objects

Spring Registry
    → selects existing managed objects
```

---

# 40. Final Mental Model

If you remember only one thing:

> **Factory encapsulates the decision and mechanics of object creation so callers don't need to know which concrete class to instantiate.**

And remember the important nuance:

> **A Factory does not necessarily eliminate `if-else`; it primarily gives object creation a single, well-defined owner.**

For Spring:

> **Spring already manages object creation, so if implementations are Spring beans and the problem is runtime selection, a Registry/Map of injected beans is often more appropriate than manually constructing objects inside a Factory.**

The most useful distinction is:

```text
Factory
"Create the right object."

Strategy
"Perform the operation using the selected behavior."

Registry
"Give me the already-created object for this key."

Builder
"Construct this complex object step by step."
```

---

# 41. Factory Checklist

Before using Factory, ask:

```text
[ ] Is object creation actually a problem?

[ ] Are there multiple concrete implementations?

[ ] Is there a decision about which implementation to create?

[ ] Is creation logic duplicated?

[ ] Is construction complicated?

[ ] Should callers depend on an abstraction instead of concrete classes?

[ ] Is a Simple Factory sufficient?

[ ] Would Factory Method make more sense?

[ ] Am I confusing Factory with Strategy?

[ ] Am I confusing Factory with Builder?

[ ] Are the objects already Spring-managed?

[ ] If Spring manages them, would a Registry be better?

[ ] Am I creating a Factory unnecessarily?
```

---

# 42. Final Interview Summary

If asked:

> "Explain Factory Pattern."

A strong answer:

> "Factory Pattern encapsulates object creation so that client code doesn't need to know which concrete implementation to instantiate or how it should be constructed. Instead of spreading `new ConcreteClass()` and creation conditionals throughout the application, I centralize that responsibility behind a Factory. A Simple Factory may still contain an if/switch; the key benefit is that creation logic has a single owner. In Spring applications, I also consider whether a Factory is necessary because Spring already manages object creation. If the implementations are Spring beans and I only need runtime selection, injecting them into a Map and using a Registry can be cleaner. Factory is primarily about creation, Strategy is about interchangeable behavior, and Builder is about constructing complex objects."
