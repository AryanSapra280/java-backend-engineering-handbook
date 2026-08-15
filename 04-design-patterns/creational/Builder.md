````md
# Builder Design Pattern

> **Core idea:** Builder separates the construction of a complex object from the object itself, allowing us to construct the object step by step while keeping the final object clean and readable.

---

# 1. Intent

The Builder Pattern is a **Creational Design Pattern**.

Its main purpose is to solve the problem of constructing objects that have:

- many fields
- many optional fields
- different combinations of fields
- complex validation
- multiple construction steps

Instead of forcing the caller to use:

```java
new User(
    name,
    age,
    email,
    phone,
    address,
    city,
    country,
    role,
    active
);
````

we can write:

```java
User user = new User.Builder()
        .name("Aryan")
        .age(25)
        .email("aryan@example.com")
        .phone("9999999999")
        .city("Delhi")
        .role("ADMIN")
        .build();
```

The construction becomes:

```text
Readable
Explicit
Maintainable
```

---

# 2. Problem

Suppose we have a `User`.

```java
public class User {

    private String name;
    private int age;
    private String email;
    private String phone;
    private String address;
    private String city;
    private String country;
    private String role;
    private boolean active;
}
```

Some fields are mandatory:

```text
name
email
```

Some are optional:

```text
age
phone
address
city
country
role
active
```

Now we need to create a User.

---

# 3. Naive Solution — Constructor

We could create:

```java
public User(
        String name,
        int age,
        String email,
        String phone,
        String address,
        String city,
        String country,
        String role,
        boolean active) {

    this.name = name;
    this.age = age;
    this.email = email;
    this.phone = phone;
    this.address = address;
    this.city = city;
    this.country = country;
    this.role = role;
    this.active = active;
}
```

Then:

```java
User user = new User(
        "Aryan",
        25,
        "aryan@example.com",
        "9999999999",
        "ABC Street",
        "Delhi",
        "India",
        "ADMIN",
        true
);
```

At first this looks okay.

But there is a serious problem.

---

# 4. What Smell / Problem Appears?

## The Telescoping Constructor Problem

Imagine we have:

```java
User(
    name,
    age,
    email,
    phone,
    address,
    city,
    country,
    role,
    active
)
```

Now imagine 15 fields.

```java
User(
    name,
    age,
    email,
    phone,
    address,
    city,
    country,
    role,
    active,
    department,
    designation,
    manager,
    salary,
    joiningDate,
    ...
)
```

The constructor becomes extremely difficult to understand.

This is called the:

> **Telescoping Constructor Problem**

---

# 5. Why Is It Dangerous?

Look at:

```java
new User(
        "Aryan",
        25,
        "aryan@example.com",
        "9999999999",
        "Delhi",
        "India",
        "ADMIN",
        true
);
```

Can you immediately remember:

```text
parameter 1 = name
parameter 2 = age
parameter 3 = email
parameter 4 = phone
...
```

Not easily.

And imagine:

```java
new User(
        "Aryan",
        25,
        "9999999999",
        "aryan@example.com",
        ...
);
```

Now `phone` and `email` have accidentally been swapped.

Java may happily compile if both are `String`.

That creates a runtime/business bug rather than a compile-time error.

---

# 6. Another Naive Solution — Multiple Constructors

We might try:

```java
public User(String name, String email) {
    ...
}

public User(
        String name,
        String email,
        int age) {
    ...
}

public User(
        String name,
        String email,
        int age,
        String phone) {
    ...
}

public User(
        String name,
        String email,
        int age,
        String phone,
        String address) {
    ...
}
```

This is better than one giant constructor.

But now:

```text
User
 |
 +-- Constructor 1
 +-- Constructor 2
 +-- Constructor 3
 +-- Constructor 4
 +-- Constructor 5
 +-- ...
```

As the number of optional fields increases, the number of combinations becomes difficult to manage.

---

# 7. Another Naive Solution — Setters

We could use:

```java
User user = new User();

user.setName("Aryan");
user.setEmail("aryan@example.com");
user.setAge(25);
user.setPhone("9999999999");
user.setCity("Delhi");
```

This looks much better.

But now the object can exist in an incomplete or invalid state.

For example:

```java
User user = new User();

user.setAge(25);
user.setCity("Delhi");
```

What is this?

Is it a valid User?

Maybe not.

The object is mutable and can potentially be used before all required fields are configured.

---

# 8. The Core Idea of Builder

Instead of:

```text
Caller
   |
   +-- huge constructor
```

we introduce:

```text
Caller
   |
   v
Builder
   |
   +-- set name
   +-- set email
   +-- set age
   +-- set phone
   +-- set city
   |
   v
build()
   |
   v
User
```

The Builder collects all the configuration.

Then:

```java
.build()
```

creates the final object.

---

# 9. Basic Structure

```text
                Client
                  |
                  v
               Builder
                  |
       +----------+----------+
       |          |          |
     name       email       age
       |          |          |
       +----------+----------+
                  |
                  v
                build()
                  |
                  v
                 User
```

The important distinction is:

```text
Builder
    -> mutable construction object

User
    -> final object
```

---

# 10. Java Implementation

Let's create the `User` Builder.

```java
public class User {

    private final String name;
    private final String email;

    private final int age;
    private final String phone;
    private final String address;
    private final String city;
    private final String country;
    private final String role;
    private final boolean active;

    private User(Builder builder) {

        this.name = builder.name;
        this.email = builder.email;

        this.age = builder.age;
        this.phone = builder.phone;
        this.address = builder.address;
        this.city = builder.city;
        this.country = builder.country;
        this.role = builder.role;
        this.active = builder.active;
    }

    public static class Builder {

        private String name;
        private String email;

        private int age;
        private String phone;
        private String address;
        private String city;
        private String country;
        private String role;
        private boolean active;

        public Builder name(String name) {

            this.name = name;
            return this;
        }

        public Builder email(String email) {

            this.email = email;
            return this;
        }

        public Builder age(int age) {

            this.age = age;
            return this;
        }

        public Builder phone(String phone) {

            this.phone = phone;
            return this;
        }

        public Builder address(String address) {

            this.address = address;
            return this;
        }

        public Builder city(String city) {

            this.city = city;
            return this;
        }

        public Builder country(String country) {

            this.country = country;
            return this;
        }

        public Builder role(String role) {

            this.role = role;
            return this;
        }

        public Builder active(boolean active) {

            this.active = active;
            return this;
        }

        public User build() {

            return new User(this);
        }
    }
}
```

---

# 11. Using the Builder

Now the caller writes:

```java
User user =
        new User.Builder()
                .name("Aryan")
                .email("aryan@example.com")
                .age(25)
                .phone("9999999999")
                .city("Delhi")
                .country("India")
                .role("ADMIN")
                .active(true)
                .build();
```

This is much easier to understand.

We can literally read it:

```text
Create User

name = Aryan
email = aryan@example.com
age = 25
phone = 9999999999
city = Delhi
country = India
role = ADMIN
active = true

BUILD
```

---

# 12. How Does `return this` Work?

This is one of the most important things to understand.

Look at:

```java
public Builder name(String name) {

    this.name = name;

    return this;
}
```

`this` means:

> the current Builder object.

Suppose:

```java
User.Builder builder =
        new User.Builder();
```

Then:

```java
builder
    .name("Aryan");
```

returns:

```text
same builder object
```

Therefore:

```java
builder
    .name("Aryan")
    .email("aryan@example.com")
    .age(25);
```

is possible.

Internally it is effectively:

```java
builder.name("Aryan");

builder.email("aryan@example.com");

builder.age(25);
```

The fluent syntax is simply a convenient way to chain method calls.

---

# 13. How Does `build()` Work?

When we call:

```java
.build();
```

the Builder does:

```java
return new User(this);
```

The `User` constructor receives the Builder:

```java
private User(Builder builder) {

    this.name = builder.name;
    this.email = builder.email;
    this.age = builder.age;
    ...
}
```

So the flow is:

```text
User.Builder
     |
     | stores configuration
     |
     v
build()
     |
     v
new User(builder)
     |
     v
User copies values
     |
     v
Final User object
```

---

# 14. Why Is the User Constructor Private?

We write:

```java
private User(Builder builder)
```

instead of:

```java
public User(Builder builder)
```

because we want construction to happen through:

```java
new User.Builder()
        ...
        .build();
```

rather than allowing callers to bypass the Builder.

This gives us control over construction.

---

# 15. Validation in Builder

One of the biggest benefits of Builder is that we can validate before creating the final object.

For example:

```java
public User build() {

    if (name == null || name.isBlank()) {

        throw new IllegalArgumentException(
                "Name is required");
    }

    if (email == null || email.isBlank()) {

        throw new IllegalArgumentException(
                "Email is required");
    }

    if (age < 0) {

        throw new IllegalArgumentException(
                "Age cannot be negative");
    }

    return new User(this);
}
```

Now:

```java
User user =
        new User.Builder()
                .name("Aryan")
                .email("aryan@example.com")
                .age(25)
                .build();
```

is validated before the object is created.

---

# 16. Builder Can Enforce Invariants

Suppose:

```text
name = mandatory
email = mandatory
age >= 0
```

We can enforce these rules inside:

```java
build()
```

Therefore:

```text
Builder
   |
   +-- collect values
   |
   +-- validate
   |
   +-- construct
   |
   v
Valid User
```

This is a very powerful concept.

The goal isn't simply:

> "Make constructors prettier."

The goal can also be:

> **Control and validate the construction of a valid object.**

---

# 17. Immutability and Builder

Builder is commonly used together with immutable objects.

Notice:

```java
private final String name;
private final String email;
```

The fields are:

```text
final
```

and there are no setters.

After:

```java
User user = ...
```

the User cannot be modified.

So:

```text
Builder
   |
   | mutable
   v
User
   |
   | immutable
   v
safe final object
```

This is an extremely common Builder design.

---

# 18. Why Is Builder Mutable?

The Builder typically does:

```java
builder.name = ...
builder.email = ...
builder.age = ...
```

So Builder itself is mutable.

That is okay.

The Builder is temporary.

The final object can be immutable.

Think:

```text
Builder = construction workspace

User = final product
```

---

# 19. Builder vs Constructor

## Constructor

Good when:

```text
Few fields
Mostly mandatory
Simple construction
```

Example:

```java
User user =
        new User("Aryan", "email@example.com");
```

Perfectly fine.

---

## Builder

Useful when:

```text
Many fields
Many optional fields
Different combinations
Complex validation
Readable construction is important
```

Example:

```java
User user =
        new User.Builder()
                .name("Aryan")
                .email("...")
                .city("Delhi")
                .role("ADMIN")
                .build();
```

---

# 20. Builder vs Setter

### Setter approach

```java
User user = new User();

user.setName("Aryan");
user.setEmail("...");
user.setAge(25);
```

The object can potentially be incomplete.

---

### Builder approach

```java
User user =
        new User.Builder()
                .name("Aryan")
                .email("...")
                .age(25)
                .build();
```

The final `User` can be immutable.

This gives us:

```text
Construction phase
        |
        v
Builder
        |
        v
Validation
        |
        v
Immutable object
```

---

# 21. Builder vs Factory

This distinction is extremely important.

## Factory

Factory answers:

> **Which object should I create?**

Example:

```java
Payment payment =
        PaymentFactory.create(
                PaymentType.UPI);
```

Factory decides:

```text
UPI -> UPIPayment
CARD -> CardPayment
```

---

## Builder

Builder answers:

> **How should I construct this object?**

Example:

```java
User user =
        new User.Builder()
                .name("Aryan")
                .email("...")
                .age(25)
                .build();
```

Builder handles:

```text
name
email
age
phone
address
...
```

---

## Mental model

```text
Factory
    |
    +-- WHICH object?

Builder
    |
    +-- HOW to construct it?
```

---

# 22. Factory + Builder Together

Factory and Builder can work together.

Imagine:

```text
Vehicle
   |
   +-- Car
   +-- Truck
   +-- Bike
```

Factory decides:

```text
Which vehicle?
```

Builder constructs:

```text
How should that vehicle be configured?
```

Conceptually:

```text
Request
   |
   v
Factory
   |
   | decides Car
   v
Car Builder
   |
   +-- engine(...)
   +-- wheels(...)
   +-- color(...)
   +-- transmission(...)
   |
   v
Car
```

So:

```text
Factory
    -> Which product?

Builder
    -> How to construct product?
```

---

# 23. Builder vs Strategy

These solve completely different problems.

## Strategy

```text
Which algorithm/behavior?
```

Example:

```java
paymentStrategy.pay(amount);
```

---

## Builder

```text
How should I construct the object?
```

Example:

```java
userBuilder
        .name(...)
        .email(...)
        .build();
```

Therefore:

```text
Strategy -> behavior variation

Builder -> construction variation
```

---

# 24. Builder vs Singleton

Singleton controls:

```text
How many instances?
```

Builder controls:

```text
How to construct an instance?
```

They solve completely different problems.

---

# 25. Builder vs Prototype

Prototype is about:

```text
Create a new object by copying an existing object.
```

Builder is:

```text
Create a new object step by step.
```

---

# 26. Builder Recognition Pattern

When you see:

```java
something
    .a(...)
    .b(...)
    .c(...)
    .d(...)
    .build();
```

you should immediately think:

> **Builder / Fluent construction**

Typical signs:

```text
Many optional fields
        +
Complex object
        +
Readable chained configuration
        +
build()
```

---

# 27. When to Use Builder

Use Builder when:

## 1. Many optional parameters

Example:

```text
User
  name
  email
  phone?
  age?
  city?
  address?
  role?
```

---

## 2. Constructor has many parameters

Instead of:

```java
new User(
    a,
    b,
    c,
    d,
    e,
    f,
    g,
    h
);
```

use:

```java
new User.Builder()
    .a(...)
    .b(...)
    .c(...)
    .build();
```

---

## 3. You want immutable objects

Builder + final fields is a common combination.

---

## 4. Construction has validation

```java
.build()
```

can validate the complete configuration.

---

## 5. Different combinations of fields are valid

For example:

```text
User
    name
    email
    phone?
    address?
```

You don't need dozens of constructors.

---

# 28. When NOT to Use Builder

Do not use Builder automatically.

If you have:

```java
User user =
        new User("Aryan", "aryan@example.com");
```

there is probably no reason to write:

```java
new User.Builder()
        .name("Aryan")
        .email("...")
        .build();
```

That is unnecessary complexity.

---

# 29. Builder Can Become Overengineering

Bad:

```java
new BooleanBuilder()
        .value(true)
        .build();
```

You don't need a Builder for:

```java
Boolean
Integer
String
```

or simple objects with one or two straightforward constructor parameters.

Use the simplest solution that keeps the design clean.

---

# 30. SOLID Connection

## Single Responsibility Principle

Without Builder:

```text
Client
   |
   +-- knows construction details
   +-- knows validation
   +-- knows parameter ordering
```

With Builder:

```text
Client
   |
   v
Builder
   |
   +-- construction
   +-- validation
   |
   v
Object
```

The construction responsibility is separated.

---

## Open/Closed Principle

Builder can help avoid constantly adding constructor overloads.

Instead of:

```text
Constructor 1
Constructor 2
Constructor 3
Constructor 4
...
```

we can add Builder methods.

However:

> Builder itself is not automatically Open/Closed.

Don't claim that Builder magically guarantees every SOLID principle.

---

## Immutability

Builder commonly helps us create immutable objects:

```java
public final class User {

    private final String name;
    private final String email;

    ...
}
```

No setters.

Once built:

```java
User user = ...
```

the object's state cannot be changed.

---

# 31. Nested Builder vs Separate Builder Class

There are two common designs.

## Nested Builder

```java
public class User {

    public static class Builder {

        ...
    }
}
```

Usage:

```java
User user =
        new User.Builder()
                .name("Aryan")
                .build();
```

This is very common.

---

## Separate Builder

```text
User
UserBuilder
```

Then:

```java
User user =
        new UserBuilder()
                .name("Aryan")
                .build();
```

Both approaches are valid.

Nested Builder keeps the construction logic closely associated with the object.

---

# 32. Builder With Default Values

Suppose:

```text
active = true
country = India
role = USER
```

We can define:

```java
public static class Builder {

    private boolean active = true;

    private String country = "India";

    private String role = "USER";

    ...
}
```

Then:

```java
User user =
        new User.Builder()
                .name("Aryan")
                .email("...")
                .build();
```

automatically gets:

```text
active = true
country = India
role = USER
```

---

# 33. Builder With Optional Fields

Example:

```java
User user =
        new User.Builder()
                .name("Aryan")
                .email("...")
                .build();
```

We don't need:

```java
phone
address
city
country
role
```

unless required.

This is one of the main advantages over constructors.

---

# 34. Builder With Validation

A better production-style Builder:

```java
public final class User {

    private final String name;
    private final String email;
    private final int age;
    private final String phone;

    private User(Builder builder) {

        this.name = builder.name;
        this.email = builder.email;
        this.age = builder.age;
        this.phone = builder.phone;
    }

    public static class Builder {

        private String name;
        private String email;
        private int age;
        private String phone;

        public Builder name(String name) {

            this.name = name;
            return this;
        }

        public Builder email(String email) {

            this.email = email;
            return this;
        }

        public Builder age(int age) {

            this.age = age;
            return this;
        }

        public Builder phone(String phone) {

            this.phone = phone;
            return this;
        }

        public User build() {

            validate();

            return new User(this);
        }

        private void validate() {

            if (name == null ||
                    name.isBlank()) {

                throw new IllegalArgumentException(
                        "Name is required");
            }

            if (email == null ||
                    email.isBlank()) {

                throw new IllegalArgumentException(
                        "Email is required");
            }

            if (age < 0) {

                throw new IllegalArgumentException(
                        "Age cannot be negative");
            }
        }
    }
}
```

This is a much more realistic use of Builder.

---

# 35. Step-by-Step Runtime Flow

Suppose:

```java
User user =
        new User.Builder()
                .name("Aryan")
                .email("aryan@example.com")
                .age(25)
                .build();
```

What happens?

## Step 1

Create Builder:

```java
new User.Builder()
```

Memory:

```text
Builder
    name = null
    email = null
    age = 0
```

---

## Step 2

Call:

```java
.name("Aryan")
```

Builder becomes:

```text
Builder
    name = Aryan
    email = null
    age = 0
```

---

## Step 3

Call:

```java
.email("aryan@example.com")
```

Builder becomes:

```text
Builder
    name = Aryan
    email = aryan@example.com
    age = 0
```

---

## Step 4

Call:

```java
.age(25)
```

Builder becomes:

```text
Builder
    name = Aryan
    email = aryan@example.com
    age = 25
```

---

## Step 5

Call:

```java
.build()
```

Builder validates.

Then:

```java
new User(builder)
```

---

## Step 6

User copies the values:

```text
User
    name = Aryan
    email = aryan@example.com
    age = 25
```

Now we have the final object.

---

# 36. Why Builder Is Often Used With Immutable Objects

Consider:

```java
public final class User {

    private final String name;
    private final String email;

    ...
}
```

After construction:

```text
User
 |
 +-- name = Aryan
 +-- email = ...
```

No one can change these fields.

That gives us:

```text
Builder
   |
   | temporary mutable state
   v
build()
   |
   v
Immutable User
```

This is a very common design combination.

---

# 37. Fluent Interface

The chained syntax:

```java
builder
    .name("Aryan")
    .email("...")
    .age(25)
    .build();
```

is called a:

> **Fluent Interface**

Builder often uses a fluent API.

But:

> Fluent API ≠ Builder automatically.

You can create fluent APIs without implementing the Builder pattern.

The defining idea of Builder is the separation of complex construction from the final object.

---

# 38. Common Mistake — Thinking Builder Is Just Setters

This is a subtle but important distinction.

Setter:

```java
user.setName("Aryan");
```

modifies the actual User.

Builder:

```java
builder.name("Aryan");
```

modifies the temporary Builder.

Then:

```java
builder.build();
```

creates the User.

So:

```text
Setter
    |
    v
Existing object

Builder
    |
    v
Construction object
    |
    v
Final object
```

---

# 39. Common Mistake — Making Everything Final Inside Builder

You might see:

```java
private final String name;
```

inside Builder and wonder how it changes.

The Builder fields normally don't need to be final because they are being configured step by step:

```java
builder.name("Aryan");
builder.email("...");
```

The final object's fields are the ones we commonly make final:

```java
private final String name;
private final String email;
```

---

# 40. Common Mistake — Builder With No Complexity

Bad:

```java
User user =
        new User.Builder()
                .name("Aryan")
                .build();
```

when the class only has:

```java
private String name;
```

This adds unnecessary ceremony.

Use:

```java
new User("Aryan");
```

---

# 41. Common Mistake — Builder With Mutable Final Object

Builder is commonly used for immutable objects.

But you can technically build mutable objects too.

However, if the main purpose is to provide a safe construction process, immutable final objects are often the cleaner design.

---

# 42. Common Mistake — Putting Business Logic Everywhere in Builder

Builder should primarily handle:

```text
construction
validation of construction invariants
```

It shouldn't become:

```text
UserService
PaymentService
DatabaseService
```

For example, avoid:

```java
public User build() {

    database.save(...);

    sendEmail(...);

    calculateSalary(...);

    return new User(...);
}
```

That is a major design smell.

Builder should construct the object.

---

# 43. Builder in Real-World Java

Builder is extremely common in Java libraries and frameworks.

You will often see APIs like:

```java
Something.builder()
        .fieldA(...)
        .fieldB(...)
        .fieldC(...)
        .build();
```

For example, modern Java applications frequently use builders for:

```text
HTTP requests
configuration objects
DTOs
domain objects
test data
complex commands
messages
database/query configuration
```

---

# 44. LLD Example — HTTP Request

Imagine:

```java
HttpRequest request =
        new HttpRequest.Builder()
                .url("/users")
                .method("POST")
                .header("Authorization", "...")
                .header("Content-Type", "application/json")
                .body(json)
                .timeout(5000)
                .build();
```

This is much easier to understand than:

```java
new HttpRequest(
        "/users",
        "POST",
        headers,
        body,
        timeout,
        ...
);
```

---

# 45. LLD Example — Parking Lot

Builder can be useful for constructing configuration-heavy objects.

For example:

```java
ParkingLot parkingLot =
        new ParkingLot.Builder()
                .name("Central Parking")
                .floors(5)
                .carSpots(500)
                .bikeSpots(200)
                .truckSpots(50)
                .hasEVCharging(true)
                .build();
```

Notice:

Factory might decide:

```text
Which Vehicle?
```

Strategy might decide:

```text
Which Parking Fee Strategy?
```

Builder might construct:

```text
ParkingLot configuration
```

This is exactly how patterns can cooperate without being the same thing.

---

# 46. LLD Example — Pizza

Classic Builder example:

```java
Pizza pizza =
        new Pizza.Builder()
                .size(Size.LARGE)
                .cheese(true)
                .mushrooms(true)
                .olives(true)
                .pepperoni(false)
                .build();
```

This is a perfect Builder candidate because:

```text
many optional combinations
```

---

# 47. LLD Example — Computer

```java
Computer computer =
        new Computer.Builder()
                .cpu("Intel i9")
                .ram(32)
                .storage(1000)
                .gpu("RTX")
                .wifi(true)
                .bluetooth(true)
                .build();
```

Not every computer needs:

```text
GPU
WiFi
Bluetooth
extra storage
```

Builder handles these optional combinations elegantly.

---

# 48. Builder With Mandatory Fields

A common concern is:

> "What if some fields are mandatory?"

We can validate in `build()`:

```java
public User build() {

    if (name == null) {
        throw new IllegalStateException(
                "Name is mandatory");
    }

    if (email == null) {
        throw new IllegalStateException(
                "Email is mandatory");
    }

    return new User(this);
}
```

This gives:

```text
Builder
   |
   +-- mandatory fields
   +-- optional fields
   |
   v
validation
   |
   v
final object
```

---

# 49. Advanced Builder — Mandatory Fields at Compile Time

For extremely strict APIs, we can design staged builders.

For example:

```text
Builder
   |
   v
name()
   |
   v
email()
   |
   v
optional fields
   |
   v
build()
```

This can make certain mandatory fields required at compile time.

However, this introduces significantly more complexity.

For normal LLD problems, validation in:

```java
build()
```

is usually enough unless the problem specifically requires compile-time enforcement.

---

# 50. Builder and Spring

Spring does not require Builder.

Spring manages dependency injection:

```java
@Service
public class PaymentService {

    private final PaymentGateway gateway;

    public PaymentService(
            PaymentGateway gateway) {

        this.gateway = gateway;
    }
}
```

You generally don't use Builder to construct Spring services.

Builder is more appropriate for:

```text
DTOs
request objects
domain objects
configuration objects
complex immutable objects
test objects
```

---

# 51. Builder and Dependency Injection

Don't confuse:

```text
Builder
```

with:

```text
Dependency Injection
```

Builder:

```text
helps construct an object
```

Dependency Injection:

```text
provides dependencies to an object
```

For example:

```java
@Service
public class PaymentService {

    private final PaymentGateway gateway;

    public PaymentService(
            PaymentGateway gateway) {

        this.gateway = gateway;
    }
}
```

Spring creates this object.

Builder is not needed.

---

# 52. Builder + Factory + Spring

All three can theoretically coexist.

Example:

```text
Controller
    |
    v
Factory
    |
    | selects product type
    v
Builder
    |
    | constructs product
    v
Domain Object
```

But in Spring:

```text
Controller
    |
    v
Registry
    |
    | selects existing bean
    v
Strategy
```

may be better.

The key is:

> Don't combine patterns just because they can be combined.

Use each one for the problem it solves.

---

# 53. Builder Recognition Questions

When solving an LLD problem, ask:

### Question 1

Does the object have many fields?

```text
10+
```

Potential Builder.

---

### Question 2

Are many fields optional?

Potential Builder.

---

### Question 3

Are there many constructor combinations?

Potential Builder.

---

### Question 4

Would constructor parameter ordering be confusing?

Potential Builder.

---

### Question 5

Can construction be broken into readable steps?

Potential Builder.

---

### Question 6

Should the final object be immutable?

Builder becomes even more attractive.

---

### Question 7

Does construction require validation?

Builder can centralize it.

---

# 54. Builder Decision Tree

```text
Need to create an object
        |
        v
How many parameters?
        |
        +-- 1-3 simple params
        |       |
        |       v
        |     Constructor
        |
        +-- Many params
                |
                v
        Many optional combinations?
                |
                +-- NO
                |     |
                |     v
                |   Constructor may be enough
                |
                +-- YES
                      |
                      v
                   Builder
```

---

# 55. Interview Questions

## Q1. What problem does Builder solve?

Strong answer:

> Builder solves the problem of constructing complex objects with many optional parameters or combinations. It avoids telescoping constructors and improves readability by allowing step-by-step construction. It can also centralize validation and is commonly used to create immutable objects.

---

## Q2. Builder vs Constructor?

> Constructor is simple and appropriate when an object has few parameters or a straightforward invariant. Builder is useful when there are many optional parameters or construction combinations and readability becomes a concern.

---

## Q3. Builder vs Setter?

> Setters modify the actual object after construction, which can leave the object partially initialized and mutable. Builder collects configuration separately and creates the final object, often allowing the final object to remain immutable.

---

## Q4. Builder vs Factory?

> Factory answers which object to create. Builder answers how to construct a complex object.

---

## Q5. Why is `return this` used?

Because Builder methods return the same Builder instance, enabling fluent method chaining:

```java
builder
    .name(...)
    .email(...)
    .age(...);
```

---

## Q6. Why is the constructor private?

To prevent clients from bypassing the intended Builder-based construction mechanism.

---

## Q7. Why make the final object's fields final?

To make the object immutable after construction.

---

## Q8. Can Builder be used with mutable objects?

Yes. Builder doesn't require immutability. But Builder is particularly valuable when constructing immutable objects.

---

## Q9. Can Builder validate?

Yes.

The common location is:

```java
build()
```

where the complete configuration is available.

---

## Q10. Is Builder always better than constructors?

No.

For simple objects:

```java
new User("Aryan", "email");
```

is often better than introducing a Builder.

---

# 56. Advanced Interview Question

### "What happens if the Builder itself is reused?"

Example:

```java
User.Builder builder =
        new User.Builder();

builder.name("Aryan")
       .email("a@example.com");

User user1 = builder.build();

builder.name("Rahul");

User user2 = builder.build();
```

Now:

```text
user1
    name = Aryan

user2
    name = Rahul
```

Whether this is desirable depends on the implementation.

The Builder is mutable, so it should generally be treated as a construction object rather than a globally shared object.

Do not make a Builder:

```java
static
singleton
shared across threads
```

unless you have a very specific reason and understand the consequences.

---

# 57. Thread Safety

A typical Builder is:

```text
NOT thread-safe
```

because it is mutable.

This is completely fine because the normal usage is:

```text
Thread
   |
   v
create Builder
   |
   v
configure Builder
   |
   v
build
   |
   v
discard Builder
```

Each thread should generally use its own Builder.

Do not do:

```java
static User.Builder builder;
```

and share it between requests.

That can create race conditions and data corruption.

---

# 58. Builder and Immutability

The ideal mental model:

```text
              Mutable
                |
                v
             Builder
                |
                | build()
                v
             Immutable
                |
                v
               User
```

This is one of the strongest reasons Builder is popular in Java.

---

# 59. Builder and Validation

Think of:

```java
build()
```

as the boundary between:

```text
configuration
```

and:

```text
valid domain object
```

Before `build()`:

```text
Maybe incomplete
Maybe invalid
Mutable
```

After `build()`:

```text
Validated
Complete
Stable
Immutable (if designed that way)
```

---

# 60. Common Production Design

A production-style immutable class might look like:

```java
public final class PaymentRequest {

    private final String customerId;
    private final double amount;
    private final String currency;
    private final String description;
    private final String idempotencyKey;

    private PaymentRequest(Builder builder) {

        this.customerId = builder.customerId;
        this.amount = builder.amount;
        this.currency = builder.currency;
        this.description = builder.description;
        this.idempotencyKey = builder.idempotencyKey;
    }

    public static Builder builder() {
        return new Builder();
    }

    public static class Builder {

        private String customerId;
        private double amount;
        private String currency;
        private String description;
        private String idempotencyKey;

        public Builder customerId(
                String customerId) {

            this.customerId = customerId;
            return this;
        }

        public Builder amount(double amount) {

            this.amount = amount;
            return this;
        }

        public Builder currency(
                String currency) {

            this.currency = currency;
            return this;
        }

        public Builder description(
                String description) {

            this.description = description;
            return this;
        }

        public Builder idempotencyKey(
                String idempotencyKey) {

            this.idempotencyKey =
                    idempotencyKey;

            return this;
        }

        public PaymentRequest build() {

            if (customerId == null ||
                    customerId.isBlank()) {

                throw new IllegalArgumentException(
                        "customerId is required");
            }

            if (amount <= 0) {

                throw new IllegalArgumentException(
                        "amount must be positive");
            }

            if (currency == null ||
                    currency.isBlank()) {

                throw new IllegalArgumentException(
                        "currency is required");
            }

            return new PaymentRequest(this);
        }
    }
}
```

Usage:

```java
PaymentRequest request =
        PaymentRequest.builder()
                .customerId("C123")
                .amount(5000)
                .currency("INR")
                .description("Order payment")
                .idempotencyKey("ORDER-123")
                .build();
```

Notice the convenience:

```java
PaymentRequest.builder()
```

instead of:

```java
new PaymentRequest.Builder()
```

This is a common style.

---

# 61. Common Mistake — Confusing `builder()` With Builder Pattern

This:

```java
PaymentRequest.builder()
```

is just a convenient factory method for creating a Builder.

The actual Builder pattern is:

```text
Builder
    |
    +-- configuration
    |
    +-- build()
    |
    v
Product
```

---

# 62. Common Mistake — Thinking Builder Always Means Nested Class

No.

A Builder can be:

```text
Nested class
```

or:

```text
Separate class
```

The important concept is:

```text
Separate construction process
```

not the physical location of the class.

---

# 63. Common Mistake — Builder for Every DTO

Don't automatically use Builder for every class.

If you have:

```java
public record User(
        String name,
        String email) {
}
```

a Builder may add unnecessary complexity.

Use it when construction complexity justifies it.

---

# 64. Real Senior Engineer Decision

Imagine you receive this requirement:

```text
Create Customer object.

Fields:
name
email
phone
address
city
state
country
age
gender
preferences
notificationSettings
marketingConsent
```

Don't immediately say:

> "Let's use Builder."

Think:

```text
How many fields?
        |
        v
Many

How many optional?
        |
        v
Many

Many combinations?
        |
        v
Yes

Do we need immutability?
        |
        v
Probably

Does constructor become unreadable?
        |
        v
Yes

Builder is a strong candidate.
```

That's how a senior engineer approaches patterns.

---

# 65. Strategy + Factory + Builder

You now know three different questions:

```text
                    Design Problem
                         |
          +--------------+--------------+
          |              |              |
          v              v              v

       Strategy        Factory        Builder
          |              |              |
          v              v              v

    Which behavior?  Which object?  How construct?
```

Example:

```text
Payment System
```

### Strategy

```text
UPI
CARD
WALLET
```

decides:

```text
HOW payment happens
```

### Factory

```text
UPI
CARD
WALLET
```

can decide:

```text
WHICH payment implementation
```

### Builder

Constructs:

```text
PaymentRequest
```

with:

```text
customerId
amount
currency
description
idempotencyKey
metadata
```

This is how multiple patterns can coexist without overlapping responsibilities.

---

# 66. Final Mental Model

Remember:

```text
Constructor
    |
    +-- Simple construction


Builder
    |
    +-- Complex construction
    +-- Many optional fields
    +-- Fluent configuration
    +-- Validation
    +-- Immutable final object


Factory
    |
    +-- Which concrete object?


Strategy
    |
    +-- Which behavior?


Registry
    |
    +-- Which existing implementation?
```

---

# 67. One-Line Mental Model

```text
Builder -> "How do I construct this complex object?"
```

That's the line you should remember.

---

# 68. Final Builder Checklist

Before using Builder:

```text
[ ] Does the object have many fields?

[ ] Are many fields optional?

[ ] Are there many possible combinations?

[ ] Would a constructor become difficult to read?

[ ] Is there a telescoping constructor problem?

[ ] Would parameter ordering be confusing?

[ ] Do I want the final object to be immutable?

[ ] Do I need validation before construction?

[ ] Can construction be expressed naturally step by step?

[ ] Would setters allow an invalid/partial object?

[ ] Is a Builder actually simpler than the constructor?

[ ] Am I using Builder because it solves a problem,
    or simply because it is a design pattern?
```

---

# 69. Final Interview Answer

If the interviewer asks:

> "Explain Builder Design Pattern."

A strong answer:

> "Builder is a creational design pattern used when constructing an object becomes complex, especially when there are many optional parameters or different combinations of fields. Instead of using a large or telescoping constructor, we create a separate Builder that collects the object's configuration step by step and then creates the final object through `build()`. This improves readability, can centralize construction validation, and is commonly combined with immutable objects. The important distinction is that Factory answers 'which object should I create?', while Builder answers 'how should I construct this object?'"

---

# 70. Final Pattern Map So Far

```text
                    CREATIONAL / BEHAVIORAL
                              |
             +----------------+----------------+
             |                |                |
             v                v                v

          Factory          Builder          Strategy
             |                |                |
             v                v                v

       WHICH object?     HOW to build?    WHICH behavior?
```

---

# 71. The Rule I Want You to Internalize

Don't memorize:

```text
Builder = chaining methods
```

That is only the syntax.

Memorize:

```text
Many construction choices
        +
Complex object
        +
Optional parameters
        +
Readable construction
        +
Potential validation
        |
        v
      BUILDER
```

And always ask:

> **"Is object construction itself becoming a problem?"**

If yes, Builder is a candidate.

If no, don't force it.

```
```
