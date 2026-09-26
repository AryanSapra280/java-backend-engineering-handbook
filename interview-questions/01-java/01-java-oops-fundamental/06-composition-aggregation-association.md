# F. Association, Aggregation & Composition — Answers

### 103. What is association?

**Association is a general relationship between two independent objects where one object knows about or interacts with another.**

Example:

```java
class Teacher {
}

class Student {
    private Teacher teacher;
}
```

A `Student` is associated with a `Teacher`.

Association does **not necessarily imply ownership** or a particular lifecycle relationship.

---

### 104. What is aggregation?

**Aggregation is a weak HAS-A relationship where the contained object can exist independently of the container.**

Example:

```java
class Employee {
}

class Department {
    private List<Employee> employees;

    Department(List<Employee> employees) {
        this.employees = employees;
    }
}
```

The `Employee` objects can exist independently of the `Department`.

```text
Department
   |
   ├── Employee
   ├── Employee
   └── Employee

Employees can exist without Department
```

---

### 105. What is composition?

**Composition is a strong HAS-A relationship where the contained object's lifecycle is strongly tied to the containing object.**

Example:

```java
class Order {
    private final List<OrderItem> items = new ArrayList<>();

    public void addItem(String product, int quantity) {
        items.add(new OrderItem(product, quantity));
    }
}

class OrderItem {
    private String product;
    private int quantity;

    OrderItem(String product, int quantity) {
        this.product = product;
        this.quantity = quantity;
    }
}
```

Here, `Order` owns its `OrderItem`s.

Conceptually:

```text
Order
  |
  ├── OrderItem
  ├── OrderItem
  └── OrderItem
```

The `OrderItem`s belong to the `Order` and are normally managed through it.

---

### 106. Aggregation vs composition?

| Aggregation                       | Composition                       |
| --------------------------------- | --------------------------------- |
| Weak HAS-A                        | Strong HAS-A                      |
| Child can exist independently     | Child lifecycle is tied to parent |
| Parent doesn't strongly own child | Parent owns child                 |
| Shared objects are possible       | Usually exclusive ownership       |
| Example: Department → Employee    | Example: Order → OrderItem        |

Think:

> **Aggregation = contains/uses, but doesn't own lifecycle.**

> **Composition = owns the contained object's lifecycle.**

---

### 107. Association vs aggregation?

**Association** is the broader relationship.

**Aggregation** is a more specific form of association representing a **whole-part relationship** where the part can exist independently.

Example association:

```java
class Doctor {
    private Patient patient;
}
```

Doctor and Patient interact, but neither necessarily owns the other.

Aggregation:

```java
class Department {
    private List<Employee> employees;
}
```

Department contains employees, but employees can exist independently of that department.

So:

```text
Association
    ↓
General relationship

Aggregation
    ↓
Specialized whole-part association
```

---

### 108. Give a Java example of association.

```java
class Customer {
    private String name;
}

class SupportAgent {
    public void handle(Customer customer) {
        System.out.println("Handling customer");
    }
}
```

`SupportAgent` is associated with `Customer` because it interacts with a customer.

There is no ownership implied.

The `Customer` can exist independently of the `SupportAgent`.

---

### 109. Give a Java example of aggregation.

```java
class Employee {
    private String name;

    Employee(String name) {
        this.name = name;
    }
}

class Department {
    private List<Employee> employees;

    Department(List<Employee> employees) {
        this.employees = employees;
    }
}
```

Usage:

```java
Employee e1 = new Employee("A");
Employee e2 = new Employee("B");

List<Employee> employees = List.of(e1, e2);

Department department = new Department(employees);
```

The employees were created independently and can conceptually continue to exist even if the department is removed.

Therefore:

```text
Department
   |
   +---- Employee
   +---- Employee
```

This represents aggregation.

---

### 110. Give a Java example of composition.

```java
class Order {

    private final List<OrderItem> items = new ArrayList<>();

    public void addItem(String product, int quantity) {
        items.add(new OrderItem(product, quantity));
    }
}

class OrderItem {

    private final String product;
    private final int quantity;

    OrderItem(String product, int quantity) {
        this.product = product;
        this.quantity = quantity;
    }
}
```

The `Order` creates and controls its `OrderItem`s:

```java
Order order = new Order();

order.addItem("Laptop", 1);
order.addItem("Mouse", 2);
```

The caller doesn't need to create and attach arbitrary `OrderItem`s.

This makes the ownership relationship explicit.

---

### 111. How does object lifecycle differ between aggregation and composition?

### Aggregation

The child can survive independently.

```text
Department ──► Employee

Department destroyed
       ↓
Employee still exists
```

For example, an employee can be moved to another department.

### Composition

The child is conceptually owned by the parent.

```text
Order ──► OrderItem

Order lifecycle ends
       ↓
OrderItems have no independent domain meaning
```

For example, an `OrderItem` normally belongs to a particular order.

**Important Java point:** Java's garbage collector doesn't automatically destroy a child object simply because its parent object becomes unreachable.

Composition is primarily a **domain/design ownership concept**. If no other references exist, both objects eventually become eligible for GC.

---

### 112. Give a backend/domain-model example where composition is appropriate.

`Order` and `OrderItem` are a good example.

```java
class Order {
    private final List<OrderItem> items = new ArrayList<>();
}
```

An `OrderItem` represents a line within a particular order.

Its meaning depends strongly on the order:

```text
Order
 ├── Item: Laptop × 1
 ├── Item: Mouse × 2
 └── Item: Keyboard × 1
```

The order should control operations such as:

```java
order.addItem(...);
order.removeItem(...);
order.updateQuantity(...);
```

rather than allowing arbitrary external modification.

---

### 113. How would you model `Order` and `OrderItem`?

I would generally model them as **composition**.

```java
class Order {

    private final Long id;
    private final List<OrderItem> items = new ArrayList<>();

    public Order(Long id) {
        this.id = id;
    }

    public void addItem(Product product, int quantity) {
        if (quantity <= 0) {
            throw new IllegalArgumentException("Invalid quantity");
        }

        items.add(new OrderItem(product, quantity));
    }

    public List<OrderItem> getItems() {
        return List.copyOf(items);
    }
}
```

```java
class OrderItem {

    private final Product product;
    private int quantity;

    OrderItem(Product product, int quantity) {
        this.product = product;
        this.quantity = quantity;
    }
}
```

The important design characteristics are:

* `Order` owns its items.
* `OrderItem` belongs to a particular order.
* `Order` controls modifications.
* The internal collection isn't exposed directly.
* Business rules can be enforced by `Order`.

In JPA, this domain relationship is often represented using `@OneToMany` with appropriate cascade/orphan-removal semantics when that lifecycle behavior matches the domain.

---

### 114. How would you model `Department` and `Employee`?

Generally as **aggregation/association**, rather than composition.

```java
class Department {

    private final List<Employee> employees = new ArrayList<>();

    public void addEmployee(Employee employee) {
        employees.add(employee);
    }

    public void removeEmployee(Employee employee) {
        employees.remove(employee);
    }
}
```

```java
class Employee {
    private String name;
}
```

An employee can move between departments:

```text
Department A
     |
   Employee

Employee moves

Department B
     |
   Employee
```

The employee doesn't cease to exist when Department A no longer contains them.

Therefore, the lifecycle is independent.

---

### 115. How would you determine whether a relationship should be inheritance, aggregation or composition?

Ask these questions:

### 1. Is it genuinely an IS-A relationship?

If yes → **Inheritance**

```text
Dog IS-A Animal
Car IS-A Vehicle
```

```java
class Dog extends Animal
```

Use this when the subclass genuinely satisfies the parent type's contract.

---

### 2. Is it a whole-part relationship where the part can exist independently?

If yes → **Aggregation**

```text
Department HAS-A Employee
Team HAS-A Developer
```

The employee/developer can exist independently.

---

### 3. Does the whole strongly own the part's lifecycle?

If yes → **Composition**

```text
Order HAS-A OrderItem
House HAS-A Room
```

The contained object is conceptually part of the owner's lifecycle.

---

### Quick decision framework

```text
Does B genuinely IS-A A?
        |
       Yes
        ↓
   Inheritance

        No
        ↓
Does A HAS-A B?
        |
       Yes
        ↓
Can B exist independently?
     /           \
   Yes            No
    ↓              ↓
Aggregation    Composition
```

### Interview shortcut

> **Inheritance → IS-A**
> **Aggregation → HAS-A, independent lifecycle**
> **Composition → HAS-A, dependent/owned lifecycle**
> **Association → general relationship between objects**
