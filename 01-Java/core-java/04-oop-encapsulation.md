# OOP — Encapsulation

## Status

🟢 COMPLETED / PARKED

> This document captures the complete discussion, practical examples, reasoning, interview traps, and deferred topics from our Encapsulation preparation. It is intended as a future learning/reference document, not just a revision sheet.

---

# 1. Definition

**Encapsulation is the practice of keeping an object's state and the operations that work on that state together, while controlling how the object's internal state can be accessed or modified from outside.**

In Java, encapsulation is primarily supported through:

- Access modifiers
- `private` fields
- Controlled methods/APIs
- Encapsulated collections
- Immutability where appropriate
- Protecting object invariants

The senior-level idea:

> Encapsulation is not simply “make fields private.” It is about controlling access to state and protecting the object's valid state/business rules.

---

# 2. Basic Encapsulation

```java
class BankAccount {

    private double balance;

    public double getBalance() {
        return balance;
    }

    public void deposit(double amount) {
        if (amount > 0) {
            balance += amount;
        }
    }
}
```

Because `balance` is private:

```java
account.balance = -5000; // ❌
```

Instead:

```java
account.deposit(500);
```

The class controls how its state changes.

---

# 3. Encapsulation Protects Business Rules

If `balance` were public:

```java
account.balance = BigDecimal.valueOf(-1000000);
```

callers could bypass validation and business rules.

A better design:

```java
class BankAccount {

    private BigDecimal balance;

    public void withdraw(BigDecimal amount) {
        if (amount.compareTo(balance) > 0) {
            throw new IllegalArgumentException("Insufficient balance");
        }

        balance = balance.subtract(amount);
    }
}
```

Strong interview explanation:

> “Making balance private prevents callers from bypassing business rules and invariants. State changes can be forced through controlled APIs where validation and business logic can be enforced.”

---

# 4. Invariants

An **invariant** is a condition that should remain true for an object's valid state.

Example:

```text
BankAccount balance >= 0
```

Encapsulation allows the class to protect such invariants by controlling state changes.

---

# 5. Getters and Setters Are Not Automatically Good Encapsulation

This:

```java
class BankAccount {

    private BigDecimal balance;

    public BigDecimal getBalance() {
        return balance;
    }

    public void setBalance(BigDecimal balance) {
        this.balance = balance;
    }
}
```

still permits:

```java
account.setBalance(BigDecimal.valueOf(-100000));
```

So:

> `private` + getter + setter does not automatically mean good encapsulation.

Prefer meaningful behavior where appropriate:

```java
public void deposit(BigDecimal amount) {
    validateAmount(amount);
    balance = balance.add(amount);
}

public void withdraw(BigDecimal amount) {
    validateAmount(amount);

    if (amount.compareTo(balance) > 0) {
        throw new IllegalArgumentException("Insufficient balance");
    }

    balance = balance.subtract(amount);
}
```

---

# 6. Access Modifiers

## Definition

**Access modifiers control where a class, field, method, or constructor can be accessed from.**

Java has:

```text
private
package-private
protected
public
```

---

# 7. `private`

```java
class BankAccount {
    private double balance;
}
```

Direct access is allowed only inside the class.

```java
class BankAccount {

    private double balance;

    void deposit(double amount) {
        balance += amount; // ✅
    }
}
```

Outside:

```java
account.balance = 500; // ❌
```

---

# 8. Package-Private

If no modifier is specified:

```java
class BankAccount {
    double balance;
}
```

the member is package-private.

It can be accessed by classes in the same package, but not directly from unrelated classes in another package.

“Default access” is commonly used informally; “package-private” is clearer.

---

# 9. `protected`

A protected member is accessible:

1. Within the same package.
2. From subclasses, subject to Java's protected-access rules.

Example:

```java
class Account {
    protected double balance;
}

class SavingsAccount extends Account {

    void addInterest() {
        balance += 100;
    }
}
```

The exact cross-package behavior becomes important in inheritance.

---

# 10. `public`

```java
class BankAccount {

    public void deposit(double amount) {
    }
}
```

A public member can be accessed wherever the containing type is accessible.

---

# 11. Access Modifier Table

| Modifier | Same class | Same package | Subclass different package | Other package |
|---|---|---|---|---|
| `private` | ✅ | ❌ | ❌ | ❌ |
| package-private | ✅ | ✅ | ❌* | ❌ |
| `protected` | ✅ | ✅ | ✅ | ❌ |
| `public` | ✅ | ✅ | ✅ | ✅ |

`*` A subclass in the same package can access package-private members because it is also in the same package.

Important:

> Cross-package `protected` access has specific Java rules. It is not unrestricted access through arbitrary parent-class references.

---

# 12. `protected` Fields vs Encapsulated Behavior

Consider:

```java
class Account {
    protected BigDecimal balance;
}
```

A subclass can directly manipulate the field:

```java
class SavingsAccount extends Account {

    void applyInterest() {
        balance = balance.add(...);
    }
}
```

This can weaken encapsulation because subclasses depend on the internal representation.

A more encapsulated design:

```java
class Account {

    private BigDecimal balance;

    protected void increaseBalance(BigDecimal amount) {
        // validation/business rules
        balance = balance.add(amount);
    }
}
```

Then:

```java
class SavingsAccount extends Account {

    void applyInterest() {
        increaseBalance(...);
    }
}
```

Design principle:

> Prefer hiding state and exposing behavior when possible.

---

# 13. Collection Encapsulation

This is a common Java trap:

```java
class BankAccount {

    private List<Transaction> transactions;

    public List<Transaction> getTransactions() {
        return transactions;
    }
}
```

The field is private, but the internal mutable collection has been leaked.

Caller can do:

```java
account.getTransactions().clear();
```

and modify internal state.

Therefore:

> `private` alone is not enough if a mutable internal object is exposed through a reference.

---

# 14. `final` Does Not Make a Collection Immutable

```java
private final List<Transaction> transactions = new ArrayList<>();
```

This prevents:

```java
transactions = new ArrayList<>(); // ❌
```

but allows:

```java
transactions.add(transaction); // ✅
transactions.clear();          // ✅
transactions.remove(0);        // ✅
```

Core distinction:

```text
final reference
      ↓
reference cannot be reassigned

BUT

referenced object
      ↓
may still be mutable
```

---

# 15. `Collections.unmodifiableList()`

```java
public List<Transaction> getTransactions() {
    return Collections.unmodifiableList(transactions);
}
```

The caller cannot structurally modify the returned collection:

```java
list.add(tx);    // ❌
list.remove(0);  // ❌
list.clear();    // ❌
```

But:

> `unmodifiableList()` protects the collection structure, not necessarily the objects inside it.

It is a read-only view of the underlying List.

---

# 16. Mutable Element Trap

Example:

```java
class Employee {

    private List<Address> addresses;

    public List<Address> getAddresses() {
        return Collections.unmodifiableList(addresses);
    }
}
```

The list is protected, but if `Address` is mutable:

```java
Address address = employee.getAddresses().get(0);

address.setCity("Delhi");
```

the caller may still indirectly modify Employee's internal state.

Therefore:

```text
unmodifiableList()
       ↓
protects List structure
       ↓
does NOT automatically protect element state
```

---

# 17. Immutable Element

A stronger design is an immutable `Address`:

```java
public final class Address {

    private final String city;
    private final String country;

    public Address(String city, String country) {
        this.city = city;
        this.country = country;
    }

    public String getCity() {
        return city;
    }

    public String getCountry() {
        return country;
    }
}
```

There is no setter, so callers cannot change its state through the API.

Now:

```text
Employee
   │
   └── unmodifiable List
          │
          ├── immutable Address
          ├── immutable Address
          └── immutable Address
```

This creates a much stronger encapsulation boundary.

---

# 18. Defensive Copy

Another option:

```java
public List<Transaction> getTransactions() {
    return new ArrayList<>(transactions);
}
```

The caller receives a separate List object.

```text
BankAccount
    │
    └── internal list → [A, B, C]

Caller
    │
    └── copied list   → [A, B, C]
```

Caller can modify the copy without modifying the internal List.

### Trade-off

`unmodifiableList()`:

```text
→ read-only view
→ no full List copy
→ reflects changes to underlying List
```

Defensive copy:

```text
→ new List allocation/copy
→ stronger collection isolation
→ caller modifications do not affect internal List
```

The choice depends on the required API semantics.

---

# 19. Shallow vs Deep Copy

This:

```java
new ArrayList<>(addresses);
```

creates a new List but does not automatically clone each Address.

```text
Original List ──→ [A, B, C]
                    ↑  ↑  ↑
                    │  │  │
Copied List   ───→ [A, B, C]
```

The Lists are different, but the elements may be the same objects.

Therefore:

> `new ArrayList<>(...)` is a shallow copy.

If the elements are mutable, changing an element through one List can affect the same element referenced by the other List.

For stronger isolation:

- Make elements immutable, or
- Create defensive copies of the elements too.

---

# 20. Immutability

## Definition

**An immutable object is an object whose observable state cannot be changed after the object has been created.**

Example:

```java
public final class Transaction {

    private final long id;
    private final BigDecimal amount;

    public Transaction(long id, BigDecimal amount) {
        this.id = id;
        this.amount = amount;
    }

    public long getId() {
        return id;
    }

    public BigDecimal getAmount() {
        return amount;
    }
}
```

After construction, there is no operation in the API that changes its state.

---

# 21. Designing an Immutable Class

Consider:

```text
Immutable Class
│
├── Private state
├── State cannot change after construction
├── No mutating setters/methods
├── Final fields where appropriate
├── Don't expose mutable internal references
├── Defensive copies where necessary
└── Consider final class to prevent problematic subclassing
```

Important:

> `final` fields alone do not make a class immutable.

---

# 22. Mutable Collection Inside an Immutable Class

This is not enough:

```java
public final class Employee {

    private final List<Address> addresses;

    public Employee(List<Address> addresses) {
        this.addresses = addresses;
    }
}
```

The caller still owns the original List:

```java
List<Address> addresses = new ArrayList<>();

Employee employee = new Employee(addresses);

addresses.add(new Address(...));
```

The Employee can observe the modification.

Use a defensive copy when appropriate:

```java
public Employee(List<Address> addresses) {
    this.addresses = new ArrayList<>(addresses);
}
```

And don't expose the internal List directly:

```java
public List<Address> getAddresses() {
    return Collections.unmodifiableList(addresses);
}
```

If `Address` is immutable, this becomes a much stronger design.

---

# 23. Encapsulation vs Immutability

These are related but different.

### Encapsulation

> Controls access to state.

Question:

```text
Who can access/change my state?
```

### Immutability

> Prevents state from changing after construction.

Question:

```text
Can my state change at all?
```

A class can be encapsulated but mutable:

```java
class BankAccount {

    private BigDecimal balance;

    public void deposit(BigDecimal amount) {
        balance = balance.add(amount);
    }
}
```

`balance` is encapsulated, but the object is mutable.

An immutable `Transaction` does not allow its state to change after construction.

---

# 24. Why Immutability Helps With Concurrency

If multiple threads share an immutable object:

```text
Thread 1 ──┐
Thread 2 ──┼──→ same immutable Transaction
Thread 3 ──┘
```

none of them can change its state.

This makes shared state easier to reason about and removes a class of synchronization problems.

This does not mean every immutable object eliminates every concurrency concern.

---

# 25. String as a Java Example

`String` is a classic immutable Java type.

```java
String s = "Hello";

s.concat(" World");
```

The original String does not change.

Instead:

```java
String s2 = s.concat(" World");
```

gives:

```text
s  → "Hello"
s2 → "Hello World"
```

---

# 26. Encapsulation Mental Model

```text
                 Encapsulation
                      │
          ┌───────────┴───────────┐
          ↓                       ↓
   Hide internal state       Control behavior
          │                       │
       private               methods/API
          │                       │
          └───────────┬───────────┘
                      ↓
             Protect invariants
                      ↓
          Maintain valid object state
```

For collections:

```text
private
   ↓
Don't expose internal reference directly

unmodifiableList()
   ↓
Protect collection structure

defensive copy
   ↓
Protect caller from modifying internal collection

immutable elements
   ↓
Protect element state

controlled methods
   ↓
Best place for business rules
```

---

# 27. Interview Questions Covered

### Q1. Why make fields private?

To prevent uncontrolled external modification and force state changes through controlled APIs.

### Q2. Is private + getter/setter automatically good encapsulation?

No. A setter may allow callers to bypass business rules and invariants.

### Q3. Does final make an object immutable?

No. `final` prevents reference reassignment; the referenced object may still be mutable.

### Q4. What does `unmodifiableList()` protect?

The collection structure through that returned reference. It does not make the elements immutable.

### Q5. What is a defensive copy?

A new object/collection created so the caller does not directly hold the internal mutable reference.

### Q6. Is `new ArrayList<>(list)` a deep copy?

No. It is a shallow copy of the List. Element references are still shared.

### Q7. How would you safely expose a List of mutable objects?

Consider whether the elements should be immutable, whether defensive copies of elements are required, and whether the API should expose a read-only view or snapshot.

### Q8. Encapsulation vs immutability?

Encapsulation controls access to state.

Immutability prevents state from changing after construction.

---

# 28. Senior-Level Takeaways

1. Encapsulation is more than private fields.
2. Good encapsulation protects business rules and invariants.
3. Getters/setters can still expose too much behavior.
4. Avoid exposing mutable internal collections directly.
5. `final` prevents reassignment, not mutation.
6. `unmodifiableList()` protects collection structure, not element state.
7. Defensive copies provide isolation but have allocation/copying cost.
8. Immutable elements make collection encapsulation much stronger.
9. Encapsulation and immutability are related but different.
10. Prefer exposing meaningful behavior over exposing raw internal state.

---

# 29. Topics Intentionally Deferred

- Advanced access-control rules
- Exact cross-package `protected` JLS rules
- Java records and immutability
- Value objects
- Defensive copying of complex object graphs
- Serialization and immutability
- Reflection and access control
- Java Memory Model implications of immutable objects
- Thread safety and safe publication
- Deep immutability
- Persistent/functional collections

---

# 30. Current Status

🟢 **Encapsulation — COMPLETED / PARKED**

Covered:

- Definition
- Private fields
- Access modifiers
- Package-private
- Protected
- Public
- Encapsulation vs getters/setters
- Business rules and invariants
- Protected fields vs behavior
- Mutable collection leakage
- `final`
- `unmodifiableList()`
- Mutable element trap
- Defensive copies
- Shallow vs deep copy
- Immutability
- Designing immutable classes
- Encapsulation vs immutability
- Concurrency benefits of immutable objects
- String as an immutable Java example

Next:

> **Inheritance — Java implementation, constructor order, `super`, method overriding, field hiding, `protected`, and inheritance traps**
