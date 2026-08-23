# OOP — Classes, Objects & Constructors

## Status
🟢 COMPLETED / PARKED

> This is a learning/reference document containing the discussion, reasoning, Java behavior, interview questions, and deferred topics.

---

# 1. Classes and Objects

## Definition

**Class:** A blueprint/type that defines the structure and behavior that its objects can have.

**Object:** An instance of a class with its own instance state.

Example:

```java
class Loan {
    int amount = 100;
}

Loan l1 = new Loan();
Loan l2 = new Loan();
```

Each object has its own instance state:

```text
              Loan class
                  │
          ┌───────┴───────┐
          ↓               ↓
       Object 1         Object 2
      amount=100       amount=100
          ↑               ↑
         l1              l2
```

Changing:

```java
l1.amount = 500;
```

does not affect `l2.amount`.

---

# 2. Instance vs Static State

```java
class Loan {
    int amount = 100;
    static int totalLoans = 0;
}
```

`amount` is instance state: each object has its own copy.

`totalLoans` is class-level/static state: it is shared.

Prefer:

```java
Loan.totalLoans
```

rather than:

```java
l1.totalLoans
```

because the static member belongs to the class.

---

# 3. Static Methods

## Definition

A static method belongs to the class rather than a particular object.

Example:

```java
class Loan {
    int amount;
    static int totalLoans;

    void update() {
        amount = 500;
        totalLoans++;
    }

    static void incrementLoans() {
        totalLoans++;
    }
}
```

A static method has no implicit `this` reference.

Therefore:

```java
static void update() {
    amount = 500; // ❌
}
```

fails because Java does not know which object's `amount` to access.

A static method can access instance state through an explicit object:

```java
static void update(Loan loan) {
    loan.amount = 500; // ✅
}
```

### Core rule

> An instance member requires an object context. An instance method has that context through `this`; a static method does not.

---

# 4. `this`

## Definition

`this` is a reference to the current object in an instance context.

```java
class Loan {
    int amount;

    void update() {
        this.amount = 500;
    }
}
```

If:

```java
Loan l1 = new Loan();
l1.update();
```

then conceptually:

```text
this → l1
```

A static method has no implicit `this`.

---

# 5. Constructors

## Definition

A constructor is a special class member used during object construction/initialization.

```java
class Loan {
    int amount;

    Loan(int amount) {
        this.amount = amount;
    }
}
```

Conceptually:

```text
new Loan(500)
      ↓
Object creation
      ↓
Constructor executes
      ↓
Object state initialized
```

---

# 6. Default Constructor vs No-Argument Constructor

If you declare **no constructor**:

```java
class Loan {
    int amount;
}
```

Java provides a compiler-generated no-argument constructor.

This is commonly called the **default constructor**.

If you explicitly declare:

```java
Loan() { }
```

it is a **no-argument constructor**, but not the compiler-provided default constructor.

### Important rule

> The compiler provides the default constructor only when you do not declare any constructor yourself.

---

# 7. Parameterized Constructor

```java
class Loan {
    int amount;

    Loan(int amount) {
        this.amount = amount;
    }
}
```

Then:

```java
new Loan(500); // ✅
new Loan();    // ❌
```

Once you declare a constructor yourself, Java does not additionally provide the implicit no-argument constructor.

If both are needed:

```java
class Loan {
    int amount;

    Loan() {
        this.amount = 0;
    }

    Loan(int amount) {
        this.amount = amount;
    }
}
```

---

# 8. Parameter Shadowing

This compiles:

```java
class Loan {
    int amount;

    Loan(int amount) {
        amount = amount;
    }
}
```

but it does not initialize the instance field.

The parameter `amount` shadows the instance field.

So:

```java
amount = amount;
```

means effectively:

```text
parameter amount = parameter amount
```

Correct:

```java
Loan(int amount) {
    this.amount = amount;
}
```

Here:

```text
this.amount → current object's field
amount      → constructor parameter
```

### Key terminology

> A local variable or parameter can shadow an instance field. `this` explicitly refers to the current object's field.

---

# 9. Constructor Overloading

## Definition

Constructor overloading means defining multiple constructors with different parameter lists.

```java
class Loan {
    int amount;
    String type;

    Loan() {
        this.amount = 0;
    }

    Loan(int amount) {
        this.amount = amount;
    }

    Loan(int amount, String type) {
        this.amount = amount;
        this.type = type;
    }
}
```

All are valid:

```java
new Loan();
new Loan(500);
new Loan(500, "HOME");
```

---

# 10. Constructor Chaining

## Definition

Constructor chaining means one constructor invokes another constructor.

`this(...)` invokes another constructor in the **same class**.

```java
class Loan {
    int amount;
    String type;

    Loan() {
        this(0, "UNKNOWN");
    }

    Loan(int amount) {
        this(amount, "UNKNOWN");
    }

    Loan(int amount, String type) {
        this.amount = amount;
        this.type = type;
    }
}
```

Conceptually:

```text
Loan()
  ↓
Loan(int, String)

Loan(int)
  ↓
Loan(int, String)
```

This avoids duplicated initialization logic.

### Important distinction

```text
this(...)
   ↓
another constructor in SAME class

super(...)
   ↓
constructor in PARENT class
```

`this(...)` does not call the parent constructor.

---

# 11. Constructor vs Method

| Constructor | Method |
|---|---|
| Same name as class | Has its own name |
| No return type | Has a return type |
| Used during object construction | Used for behavior |
| Cannot be inherited | Methods can be inherited depending on access |
| Cannot be overridden | Can be overridden |
| Can be overloaded | Can be overloaded |

---

# 12. Constructor Restrictions

Constructors cannot be:

```text
static
final
abstract
```

because constructors participate in object construction and are not inherited/overridden like normal methods.

---

# 13. Instance Initializer Block

Java also supports instance initializer blocks:

```java
class Loan {
    int amount;

    {
        amount = 100;
    }

    Loan() {
        System.out.println(amount);
    }
}
```

The block participates in object initialization.

Detailed initialization order is intentionally deferred until inheritance.

---

# 14. `this(...)` and Java Version Nuance

For Java 21:

```java
Loan() {
    this(100);
    System.out.println("Creating loan");
}
```

is valid.

But:

```java
Loan() {
    System.out.println("Creating loan");
    this(100);
}
```

is not valid under Java 21 because an explicit constructor invocation must appear first.

Flexible constructor bodies were developed after Java 21 and became permanent later. This advanced language-version detail is parked for now.

---

# 15. Interview Questions Covered

### Q1. Do two objects share an instance variable?

No. Each object has its own instance state.

### Q2. What is the difference between instance and static variables?

Instance → per-object state.

Static → class-level shared state.

### Q3. Why can't a static method directly access an instance field?

Because a static method has no implicit `this` and therefore no specific object context.

### Q4. Can a static method access instance state?

Yes, through an explicit object reference.

### Q5. What happens if you define only a parameterized constructor?

The compiler does not provide an implicit no-argument constructor.

### Q6. Why use `this.amount = amount`?

The parameter shadows the field; `this.amount` explicitly refers to the object's field.

### Q7. What does `this(...)` do?

It invokes another constructor in the same class.

### Q8. What does `super(...)` do?

It invokes a parent-class constructor. Detailed behavior is covered under inheritance.

---

# 16. Deferred Topics

These will be covered later:

- Exact object initialization order
- Parent/child constructor order
- `super()`
- Static initialization order
- Instance initialization order
- Constructor invocation during inheritance
- Object lifecycle internals
- JVM-level object layout

---

# 17. Current Status

🟢 **Classes & Objects + Constructors — COMPLETED / PARKED**

Covered:

- Classes
- Objects
- Instance variables
- Static variables
- Instance methods
- Static methods
- `this`
- Instance vs class state
- Constructors
- Default vs no-argument constructor
- Parameterized constructors
- Constructor overloading
- Constructor chaining
- `this(...)`
- Parameter shadowing
- Constructor vs method
- Constructor restrictions
- Instance initializer blocks

Next:

> **Encapsulation — Java implementation and real-world design**
