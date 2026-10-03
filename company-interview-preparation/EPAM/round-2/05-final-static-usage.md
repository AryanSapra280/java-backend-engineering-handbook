Let's go. 🔥 **`final`, `static`, and Java initialization order** is another deceptively simple topic. Interviewers love combining these with inheritance, constructors, class loading, and multithreading.

# 5. `final`, `static` & Initialization

---

# Part 1 — `final`

`final` means, broadly:

> **This thing cannot be changed in the particular way that matters for that declaration.**

But the meaning differs depending on where you use it.

```text
final variable
final reference
final method
final class
```

Let's separate them.

---

## 1. `final` variable

```java
final int x = 10;

x = 20;   // compilation error
```

Once assigned, `x` cannot be assigned again.

But there's an important distinction.

### Primitive

```java
final int x = 10;
```

The value cannot change.

### Reference

```java
final Payment payment = new Payment();
```

The **reference cannot point somewhere else**, but the object itself may still be mutable.

```java
final Payment payment = new Payment();

payment.setAmount(500);   // valid
payment = new Payment();  // ERROR
```

Think:

```text
final reference
      |
      v
+----------------+
| Payment object |
| amount = 500   |
+----------------+
```

The arrow cannot be redirected:

```text
payment ---> another Payment
```

But the object can potentially change internally.

### Interview answer

> `final` on a reference prevents reassignment of the reference; it does not make the referenced object immutable.

**This is extremely important.**

---

# 2. Can a final variable be assigned later?

Yes.

This is called a **blank final variable**.

```java
class Payment {

    private final String paymentId;

    Payment(String paymentId) {
        this.paymentId = paymentId;
    }
}
```

This is valid.

The field wasn't initialized at declaration:

```java
private final String paymentId;
```

But the constructor assigns it exactly once.

---

## 3. `final` local variable

```java
void process() {

    final int retryCount;

    retryCount = 3;

    System.out.println(retryCount);
}
```

Valid.

But:

```java
retryCount = 4;
```

afterward is invalid.

---

# 4. `final` method

```java
class PaymentService {

    public final void processPayment() {
    }
}
```

A subclass cannot override it.

```java
class SpecialPaymentService extends PaymentService {

    @Override
    public void processPayment() {   // ERROR
    }
}
```

Why would you use it?

When the parent class wants to guarantee that a particular behavior cannot be replaced by subclasses.

For example, imagine:

```java
public final void execute() {
    validate();
    process();
    audit();
}
```

You may want the overall workflow protected while allowing certain internal operations to be overridden.

This connects to the **Template Method design pattern**, which we'll cover later.

---

# 5. `final` class

```java
public final class PaymentProcessor {
}
```

Cannot be extended:

```java
class MyProcessor extends PaymentProcessor {
}
```

Compilation error.

Famous examples include:

```java
String
Integer
Long
```

`String` being final is particularly important because Java relies on its immutable behavior.

---

# 6. `final` does NOT mean immutable

This is one of the most common traps.

```java
final List<String> names = new ArrayList<>();

names.add("Aryan");       // valid
names.add("Java");        // valid

names = new ArrayList<>(); // invalid
```

So:

```text
final
≠
immutable
```

`final` controls reassignment/overriding/inheritance depending on context.

Immutability is a property of the object's design.

---

# Part 2 — `static`

Now the important distinction.

A normal instance field belongs conceptually to an **object**.

```java
class Employee {

    String name;
}
```

Each object gets its own `name`.

```java
Employee e1 = new Employee();
Employee e2 = new Employee();

e1.name = "A";
e2.name = "B";
```

Conceptually:

```text
e1
+----------+
| name = A |
+----------+

e2
+----------+
| name = B |
+----------+
```

---

# 7. Static field

Now:

```java
class Employee {

    static String company;
}
```

`company` belongs to the **class**, rather than being separate state per object.

Conceptually:

```text
Employee class
      |
      v
 company = "EdgeVerve"
      ^
      |
   e1 / e2
```

So:

```java
Employee.company = "EdgeVerve";
```

Both instances observe that class-level state.

---

# 8. Static variable example

```java
class Employee {

    static int employeeCount = 0;

    Employee() {
        employeeCount++;
    }
}
```

Then:

```java
new Employee();
new Employee();
new Employee();

System.out.println(Employee.employeeCount);
```

Output:

```text
3
```

There isn't one `employeeCount` per object.

There's one static field associated with the class.

---

# 9. Static method

```java
class MathUtil {

    static int add(int a, int b) {
        return a + b;
    }
}
```

Call:

```java
MathUtil.add(10, 20);
```

You don't need:

```java
MathUtil obj = new MathUtil();
obj.add(10, 20);
```

---

# 10. Why can't a static method directly access instance variables?

Example:

```java
class Payment {

    int amount;

    static void printAmount() {
        System.out.println(amount); // ERROR
    }
}
```

Why?

Because:

```text
static method
    |
    | belongs to class
    v
no particular Payment object
```

But:

```text
instance field
    |
    v
belongs to a particular Payment object
```

Which `amount` should it access?

There could be:

```java
Payment p1 = new Payment();
Payment p2 = new Payment();
```

with different amounts.

So the static method has no implicit `this`.

---

# 11. `this` inside static method

This is illegal:

```java
static void test() {
    System.out.println(this);
}
```

Because `this` represents the current instance.

A static method isn't invoked against a particular instance.

---

# 12. Can static methods be overridden?

This is a favorite interview question.

**No. Static methods are hidden, not overridden.**

Example:

```java
class Parent {

    static void print() {
        System.out.println("Parent");
    }
}

class Child extends Parent {

    static void print() {
        System.out.println("Child");
    }
}
```

Now:

```java
Parent p = new Child();
p.print();
```

Output:

```text
Parent
```

Why?

Static method dispatch is based on the **reference/class type**, not runtime polymorphism.

This differs from instance methods:

```java
class Parent {
    void print() {
        System.out.println("Parent");
    }
}

class Child extends Parent {
    @Override
    void print() {
        System.out.println("Child");
    }
}
```

Then:

```java
Parent p = new Child();
p.print();
```

prints:

```text
Child
```

That's runtime polymorphism.

---

# Part 3 — Static initialization

Now we connect this to JVM class initialization.

Consider:

```java
class PaymentService {

    static int timeout = 30;

    static {
        System.out.println("Static block");
    }
}
```

When the class is initialized, the static initialization logic executes.

Static initialization is associated with the **class**, not each object.

---

# 13. Multiple static blocks

```java
class Demo {

    static {
        System.out.println("A");
    }

    static {
        System.out.println("B");
    }
}
```

They execute in source order:

```text
A
B
```

---

# 14. Static field initialization order

Look carefully:

```java
class Demo {

    static int x = 10;

    static {
        System.out.println(x);
    }

    static int y = 20;
}
```

Output:

```text
10
```

The declarations execute in textual order during class initialization.

Think:

```text
static x = 10
      ↓
static block
      ↓
static y = 20
```

---

# 15. Now the nasty initialization example

```java
class Demo {

    static int x = 10;

    static {
        System.out.println("A");
        System.out.println(x);
    }

    static int y = 20;

    static {
        System.out.println("B");
        System.out.println(y);
    }
}
```

The order is:

```text
x = 10
A
10
y = 20
B
20
```

Why?

Because static initialization follows the class's initialization sequence in textual order.

---

# Part 4 — Instance initialization blocks

Now this:

```java
class Demo {

    {
        System.out.println("Instance block");
    }

    Demo() {
        System.out.println("Constructor");
    }
}
```

When you do:

```java
new Demo();
```

output:

```text
Instance block
Constructor
```

The instance initialization block runs before the constructor body.

---

# 16. Why do instance blocks exist?

They're relatively uncommon in modern application code.

You can think of them as code that should execute during every object construction, before the constructor body.

But in production Spring applications, you will more commonly encounter:

```java
@PostConstruct
```

or constructor injection rather than initialization blocks.

Still, interviewers love asking about them.

---

# 17. The BIG initialization-order question

Now memorize the concept, not a random sequence.

```java
class Demo {

    static {
        System.out.println("Static");
    }

    {
        System.out.println("Instance");
    }

    Demo() {
        System.out.println("Constructor");
    }
}
```

What happens?

```java
new Demo();
new Demo();
```

Output:

```text
Static
Instance
Constructor
Instance
Constructor
```

Why?

### First object

Class hasn't been initialized yet:

```text
Class initialization
       ↓
Static block
       ↓
Object creation
       ↓
Instance initialization
       ↓
Constructor
```

### Second object

Class is already initialized:

```text
Object creation
       ↓
Instance initialization
       ↓
Constructor
```

Therefore static initialization happens once, instance initialization happens once **per object**.

---

# 18. Connect this to JVM class initialization

This is where your earlier JVM knowledge becomes useful.

Earlier we discussed:

```text
Loading
   ↓
Linking
   ↓
Initialization
```

During initialization, static initialization logic runs.

So conceptually:

```text
Class first actively used
        ↓
Class initialization
        ↓
static fields / static blocks
        ↓
Class becomes initialized
        ↓
new object
        ↓
instance fields / instance blocks
        ↓
constructor
```

This is why you should not treat:

> class loading

and

> object creation

as the same thing.

---

# 19. Full initialization order with inheritance

Now we're entering **senior interview territory**.

```java
class Parent {

    static {
        System.out.println("Parent static");
    }

    {
        System.out.println("Parent instance");
    }

    Parent() {
        System.out.println("Parent constructor");
    }
}

class Child extends Parent {

    static {
        System.out.println("Child static");
    }

    {
        System.out.println("Child instance");
    }

    Child() {
        System.out.println("Child constructor");
    }
}
```

Now:

```java
new Child();
```

Output:

```text
Parent static
Child static
Parent instance
Parent constructor
Child instance
Child constructor
```

This is **very important**.

---

# 20. Why this order?

Break it down.

Before a Child object can be initialized, the superclass class must be initialized.

So:

```text
Parent static
      ↓
Child static
```

Then object construction begins.

A Child object contains the Parent portion too.

So the superclass portion is initialized first:

```text
Parent instance
Parent constructor
```

Then:

```text
Child instance
Child constructor
```

Conceptually:

```text
          Child object
       +----------------+
       | Parent state   |
       | Parent part    |
       +----------------+
       | Child state    |
       | Child part     |
       +----------------+
```

Parent construction must happen before Child construction.

---

# 21. Even more complete initialization sequence

Suppose:

```java
class Parent {

    static int a = initParentStatic();

    static {
        System.out.println("Parent static block");
    }

    int x = initParentInstance();

    {
        System.out.println("Parent instance block");
    }

    Parent() {
        System.out.println("Parent constructor");
    }

    static int initParentStatic() {
        System.out.println("Parent static field");
        return 1;
    }

    int initParentInstance() {
        System.out.println("Parent instance field");
        return 1;
    }
}
```

For the first use of the class, static initialization follows textual order.

Then object initialization follows instance initialization order.

You should understand this pattern rather than memorize one particular output.

---

# 22. `static final`

This is extremely common in production Java.

```java
public static final int MAX_RETRIES = 3;
```

Meaning:

- `static` → class-level
- `final` → cannot be reassigned

Often used for constants.

Example:

```java
public static final String DEFAULT_CURRENCY = "INR";
```

Naming convention:

```text
UPPER_SNAKE_CASE
```

---

# 23. Compile-time constants

There is a deeper distinction.

```java
static final int MAX_RETRIES = 3;
```

is a compile-time constant.

But:

```java
static final Integer MAX_RETRIES = Integer.valueOf(3);
```

has different characteristics.

Likewise:

```java
static final String SERVICE_NAME = "PAYMENT";
```

can be a compile-time constant because it's a constant expression.

This matters for things such as `switch`, annotation values, and compiler inlining.

You don't need to go too deep unless the interviewer asks.

---

# 24. Dangerous `static` state in Spring applications

This is **very relevant to your backend work**.

Suppose:

```java
@Service
public class PaymentService {

    private static int paymentCount;
}
```

You now have shared mutable state across all instances/threads in that JVM.

In a Spring application:

```text
Request 1 ──┐
Request 2 ──┼──> PaymentService
Request 3 ──┘
```

If multiple requests modify:

```java
paymentCount++;
```

you can get a race condition.

Remember:

```text
static
≠
thread-safe
```

and:

```text
final
≠
thread-safe
```

These distinctions are very important.

---

# 25. `static final` object is not necessarily immutable

Another trap:

```java
public static final List<String> SUPPORTED_CURRENCIES =
        new ArrayList<>();
```

The reference can't change:

```java
SUPPORTED_CURRENCIES = anotherList; // ERROR
```

But:

```java
SUPPORTED_CURRENCIES.add("INR");
```

may be perfectly legal.

So:

```text
static final
```

doesn't automatically mean:

```text
immutable
```

If you want an immutable collection, you need an immutable/unmodifiable design, e.g.:

```java
static final List<String> CURRENCIES =
        List.of("INR", "USD", "EUR");
```

---

# 26. Initialization trap

What happens here?

```java
class Test {

    static int x = 10;

    static {
        x = 20;
    }

    public static void main(String[] args) {
        System.out.println(x);
    }
}
```

Output:

```text
20
```

Because:

```text
x = 10
   ↓
static block
   ↓
x = 20
   ↓
main()
```

---

# 27. Forward reference trap

Now:

```java
class Test {

    static {
        System.out.println(x);
    }

    static int x = 10;
}
```

This is problematic because Java has restrictions around referring to a static field before its declaration in certain contexts.

Don't casually say:

> "Java always allows forward references."

It doesn't.

The exact rules differ between reading and writing variables and between contexts.

For interview purposes, the safe principle is:

> Static field initialization follows textual order, and illegal forward references are compile-time errors.

---

# 28. Initialization circular dependency

Senior interviewers sometimes ask about:

```java
class A {
    static int x = B.y + 1;
}

class B {
    static int y = A.x + 1;
}
```

This can produce surprising results because class initialization can trigger initialization of another class.

The key lesson:

> Avoid complicated static initialization dependencies between classes.

Prefer explicit dependency injection and normal initialization where appropriate.

This is especially relevant in Spring applications, where bean dependencies should generally be managed by the container rather than hidden inside static initialization.

---

# 29. `final` + constructor + immutability

This is a connection worth remembering.

Consider:

```java
public final class Payment {

    private final String paymentId;
    private final BigDecimal amount;

    public Payment(String paymentId, BigDecimal amount) {
        this.paymentId = paymentId;
        this.amount = amount;
    }

    public String getPaymentId() {
        return paymentId;
    }

    public BigDecimal getAmount() {
        return amount;
    }
}
```

This is moving toward an immutable object design.

But don't blindly say:

> "All fields are final, therefore the class is immutable."

You also need to consider whether the referenced objects themselves are mutable and whether you expose mutable internal state.

For example:

```java
private final List<String> items;
```

doesn't automatically make `items` immutable.

---

# Interview mental model

Keep this diagram:

```text
                 final
                   |
       +-----------+-----------+
       |           |           |
    variable     method      class
       |           |           |
   no reassign   no override  no extend


                 static
                   |
       +-----------+-----------+
       |           |           |
     field       method      block
       |           |           |
   class-level   no this    runs during
    state                    class init
```

And initialization:

```text
FIRST active class use
        ↓
Parent class initialization
        ↓
Child class initialization
        ↓
Object creation
        ↓
Parent instance fields/blocks
        ↓
Parent constructor
        ↓
Child instance fields/blocks
        ↓
Child constructor
```

---

# 🔥 Interview test

Before we move to **Exceptions**, try this one mentally:

```java
class Parent {

    static {
        System.out.println("P-S");
    }

    {
        System.out.println("P-I");
    }

    Parent() {
        System.out.println("P-C");
    }
}

class Child extends Parent {

    static {
        System.out.println("C-S");
    }

    {
        System.out.println("C-I");
    }

    Child() {
        System.out.println("C-C");
    }
}

public class Main {

    public static void main(String[] args) {

        System.out.println("1");

        new Child();

        System.out.println("2");

        new Child();
    }
}
```

The exact output is:

```text
1
P-S
C-S
P-I
P-C
C-I
C-C
2
P-I
P-C
C-I
C-C
```

The crucial observation is:

**static initialization happens once per class, while instance initialization and constructors happen once per object.**

That distinction will come back when we discuss **Spring singleton beans**, because a Spring singleton is **not the same thing as a Java `static` field**.

Next in our sequence is **Exceptions**, where we'll go beyond `try/catch` into:

```text
checked vs unchecked
      ↓
Exception hierarchy
      ↓
throw vs throws
      ↓
finally
      ↓
try-with-resources
      ↓
custom exceptions
      ↓
exception propagation
      ↓
suppressed exceptions
      ↓
Spring REST exception handling
      ↓
production error-handling strategy
```

That last part is especially important for your backend interviews.