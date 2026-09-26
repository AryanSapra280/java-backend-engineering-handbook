# Encapsulation — Answers

### 21. What is encapsulation?

**Encapsulation is the practice of bundling data and the methods that operate on that data inside a class, while controlling access to the object's internal state.**

In Java, this is commonly achieved using `private` fields and controlled public methods.

---

### 22. Why do we need encapsulation?

Encapsulation helps us:

* Protect object state from uncontrolled modification.
* Enforce business rules and validation.
* Reduce coupling.
* Hide implementation details.
* Make code easier to maintain and change.
* Prevent objects from entering invalid states.

---

### 23. How do you achieve encapsulation in Java?

Common techniques are:

* Make fields `private`.
* Expose only required operations through methods.
* Validate data before modifying state.
* Avoid exposing mutable internal objects directly.
* Use defensive copies or unmodifiable views where appropriate.

Example:

```java
class BankAccount {
    private double balance;

    public void deposit(double amount) {
        if (amount <= 0) {
            throw new IllegalArgumentException("Invalid amount");
        }
        balance += amount;
    }

    public double getBalance() {
        return balance;
    }
}
```

The caller cannot directly modify `balance`.

---

### 24. Is making all fields `private` enough to achieve encapsulation?

**No.**

`private` prevents direct access, but you can still break encapsulation through poorly designed public methods.

For example:

```java
class Employee {
    private List<String> skills = new ArrayList<>();

    public List<String> getSkills() {
        return skills;
    }
}
```

The field is private, but callers can modify the internal list:

```java
employee.getSkills().clear();
```

Therefore, encapsulation is about **controlling access to state**, not merely making fields private.

---

### 25. Why are getters and setters used?

Getters and setters provide **controlled access** to fields.

They allow you to add logic such as:

```java
public void setAge(int age) {
    if (age < 0) {
        throw new IllegalArgumentException();
    }
    this.age = age;
}
```

However, getters/setters aren't mandatory for encapsulation. Sometimes exposing a **business operation** is better than exposing a setter.

For example:

```java
account.withdraw(500);
```

is often better than:

```java
account.setBalance(account.getBalance() - 500);
```

---

### 26. Can getters and setters still violate encapsulation?

**Yes.**

A setter like:

```java
public void setBalance(double balance) {
    this.balance = balance;
}
```

may allow callers to bypass business rules.

Similarly, a getter can expose mutable internal state:

```java
public List<Transaction> getTransactions() {
    return transactions;
}
```

The caller can then modify the internal list.

So **getters/setters don't automatically guarantee encapsulation**.

---

### 27. How can exposing a mutable object through a getter break encapsulation?

Suppose:

```java
class Employee {
    private Address address;

    public Address getAddress() {
        return address;
    }
}
```

If `Address` is mutable:

```java
employee.getAddress().setCity("Delhi");
```

The caller has directly modified the internal state of `Employee`.

The `Employee` class has lost control over its state.

A defensive copy can be used:

```java
public Address getAddress() {
    return new Address(address);
}
```

assuming `Address` has an appropriate copy constructor.

---

### 28. How can exposing a `List` through a getter break encapsulation?

Example:

```java
class Order {
    private List<String> items = new ArrayList<>();

    public List<String> getItems() {
        return items;
    }
}
```

Caller:

```java
order.getItems().clear();
order.getItems().add("Invalid item");
```

Now the caller can modify the internal collection without going through `Order`'s business rules.

That's a violation of encapsulation.

---

### 29. How would you safely expose a collection from a class?

One common approach is returning an **unmodifiable view**:

```java
public List<String> getItems() {
    return Collections.unmodifiableList(items);
}
```

Or:

```java
public List<String> getItems() {
    return List.copyOf(items);
}
```

You can also return a defensive copy:

```java
public List<String> getItems() {
    return new ArrayList<>(items);
}
```

The appropriate choice depends on whether you want a **snapshot/copy** or a **read-only view**.

---

### 30. What is defensive copying?

**Defensive copying means creating a copy of mutable data before storing it or returning it, so external code cannot modify the object's internal state through an aliased reference.**

Example:

```java
class Employee {
    private List<String> skills;

    public Employee(List<String> skills) {
        this.skills = new ArrayList<>(skills);
    }

    public List<String> getSkills() {
        return new ArrayList<>(skills);
    }
}
```

Both input and output are copied.

---

### 31. How does defensive copying protect encapsulation?

Without copying:

```text
Caller ───────┐
              ↓
         internal List
```

The caller and object share the same mutable list.

With defensive copying:

```text
Caller List ──► Copy

Internal List ──► Separate object
```

Changes made to the caller's list don't affect the object's internal state, and changes to the returned list don't affect the object's internal state.

---

### 32. What is the difference between defensive copying and returning an unmodifiable collection?

**Defensive copy:**

```java
return new ArrayList<>(items);
```

Creates a **new collection**.

Changes to the returned collection don't affect the original.

The caller can modify its own copy.

---

**Unmodifiable view:**

```java
return Collections.unmodifiableList(items);
```

Returns a **read-only view of the same underlying collection**.

The caller cannot modify it through that reference, but changes made internally to `items` can be visible through the view.

So:

| Defensive Copy                      | Unmodifiable View                     |
| ----------------------------------- | ------------------------------------- |
| Creates a new collection            | Same underlying collection            |
| Caller can modify the returned copy | Caller cannot modify through the view |
| Snapshot of current contents        | Reflects later internal changes       |
| More memory                         | Less memory                           |

`List.copyOf(items)` is different again: it creates an **unmodifiable copy**.

---

### 33. Design a class where callers cannot put the object into an invalid state.

Instead of exposing unrestricted setters, enforce invariants inside the class.

```java
class BankAccount {

    private double balance;

    public BankAccount(double initialBalance) {
        if (initialBalance < 0) {
            throw new IllegalArgumentException("Balance cannot be negative");
        }
        this.balance = initialBalance;
    }

    public void deposit(double amount) {
        if (amount <= 0) {
            throw new IllegalArgumentException("Amount must be positive");
        }

        balance += amount;
    }

    public void withdraw(double amount) {
        if (amount <= 0) {
            throw new IllegalArgumentException("Amount must be positive");
        }

        if (amount > balance) {
            throw new IllegalArgumentException("Insufficient balance");
        }

        balance -= amount;
    }

    public double getBalance() {
        return balance;
    }
}
```

There is no:

```java
setBalance()
```

Therefore callers cannot arbitrarily set the balance.

The class itself owns the business rules.

---

### 34. How would you encapsulate business rules inside an object rather than exposing setters?

Prefer **behavior-oriented methods** over exposing raw state mutation.

Instead of:

```java
order.setStatus("SHIPPED");
```

use:

```java
order.ship();
```

Instead of:

```java
account.setBalance(account.getBalance() - amount);
```

use:

```java
account.withdraw(amount);
```

The object can then enforce rules:

```java
public void ship() {
    if (items.isEmpty()) {
        throw new IllegalStateException("Cannot ship empty order");
    }

    if (status != Status.PAID) {
        throw new IllegalStateException("Order must be paid first");
    }

    status = Status.SHIPPED;
}
```

This is stronger encapsulation because the caller says **what it wants to do**, while the object controls **how and whether it can happen**.

---

### 35. Give a real-world backend example where poor encapsulation can cause bugs.

Consider a **Provident Fund account**:

```java
class PFAccount {
    private BigDecimal balance;

    public void setBalance(BigDecimal balance) {
        this.balance = balance;
    }
}
```

If any service can call:

```java
account.setBalance(new BigDecimal("0"));
```

it can bypass rules such as:

* contribution processing
* interest calculation
* withdrawal validation
* transaction history
* audit requirements

A better design would expose domain operations:

```java
account.creditContribution(amount);
account.applyInterest(interest);
account.processWithdrawal(amount);
```

Each operation can validate the required business rules and maintain the object's invariants.

**Interview takeaway:**

> Encapsulation isn't simply `private + getter/setter`. Strong encapsulation means the object controls how its state can change and prevents callers from bypassing its invariants.
