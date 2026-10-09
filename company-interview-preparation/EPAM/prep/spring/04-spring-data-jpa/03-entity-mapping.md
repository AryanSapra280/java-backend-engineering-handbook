Next up — **Entity Mapping & Relationships**, one of the highest-value JPA topics for EPAM.

# 4. Spring Data JPA / Hibernate ⭐⭐⭐⭐⭐
## Part 3 — Entity Mapping & Relationships

---

# 1. What does Entity Mapping mean?

### Interview Question
**How does Hibernate map a Java entity to a database table?**

Entity mapping defines how:

```text
Java class        → Database table
Java field        → Database column
Object reference  → Foreign key relationship
```

Example:

```java
@Entity
@Table(name = "users")
public class User {

    @Id
    @GeneratedValue
    private Long id;

    @Column(name = "user_name")
    private String name;

    private String email;
}
```

Conceptually:

```text
User class
    |
    +-- id      → users.id
    +-- name    → users.user_name
    +-- email   → users.email
```

---

# 2. `@Entity`

```java
@Entity
public class User {
}
```

Marks the class as a JPA entity.

The JPA provider will treat instances of this class as persistent entities.

---

# 3. `@Table`

### Interview Question
**Why do we use `@Table`?**

`@Table` allows you to specify the database table explicitly.

```java
@Entity
@Table(name = "customer_accounts")
public class Customer {
}
```

Without it, the ORM may derive the table name from the entity name according to its naming strategy.

### Production recommendation

Explicit naming is often preferable when:

- database naming doesn't match Java naming
- legacy tables exist
- multiple schemas are involved

Example:

```java
@Table(
    name = "users",
    schema = "customer"
)
```

---

# 4. `@Id`

### Interview Question
**What does `@Id` do?**

It identifies the entity's primary-key field.

```java
@Id
private Long id;
```

Every JPA entity must have an identifier.

Conceptually:

```text
Java                         DB

User                         users
 |
 +-- id  ----------------->  id (PK)
```

---

# 5. `@GeneratedValue`

### Interview Question
**Why do we use `@GeneratedValue`?**

It tells JPA how the identifier should be generated.

Example:

```java
@Id
@GeneratedValue(strategy = GenerationType.IDENTITY)
private Long id;
```

Common strategies:

```text
IDENTITY
SEQUENCE
TABLE
AUTO
```

---

# 6. `IDENTITY` vs `SEQUENCE`

This is a good interview question.

### IDENTITY

The database generates the ID, commonly using an identity/auto-increment column.

```java
@GeneratedValue(strategy = GenerationType.IDENTITY)
```

Conceptually:

```text
INSERT
   |
   v
Database generates ID
```

### SEQUENCE

The database uses a sequence.

```java
@GeneratedValue(strategy = GenerationType.SEQUENCE)
```

Conceptually:

```text
Sequence
   |
   v
Generate ID
   |
   v
INSERT
```

### Why does this matter?

Identifier generation strategy can affect:

- batching
- insert performance
- database round trips
- portability

For high-throughput applications, sequence-based strategies can be advantageous because Hibernate can often obtain IDs ahead of inserts and work more effectively with JDBC batching.

---

# 7. `@Column`

### Interview Question
**What does `@Column` do?**

It controls how a Java field maps to a database column.

```java
@Column(
    name = "user_name",
    nullable = false,
    length = 100
)
private String name;
```

Common attributes:

```text
name
nullable
length
unique
insertable
updatable
```

Example:

```java
@Column(nullable = false, unique = true)
private String email;
```

This communicates constraints to the ORM/schema tooling, but **database constraints should ultimately be enforced by the database itself**.

---

# 8. `@Transient`

Don't confuse this with the JPA lifecycle state **Transient**.

```java
@Transient
private String displayName;
```

This tells JPA:

> Do not persist this field as a database column.

Example:

```java
@Entity
public class User {

    private String firstName;
    private String lastName;

    @Transient
    private String displayName;
}
```

`displayName` exists in Java but isn't mapped to the database.

---

# 9. Relationships in JPA

The four major relationship mappings are:

```text
@OneToOne
@OneToMany
@ManyToOne
@ManyToMany
```

The important thing is to understand the **database representation**, not just memorize annotations.

---

# 10. `@ManyToOne` ⭐⭐⭐⭐⭐

This is probably the most important relationship mapping.

Suppose:

```text
Department
    |
    +---- Employee
    +---- Employee
    +---- Employee
```

Many employees belong to one department.

Database:

```text
departments
------------
id
name


employees
---------
id
name
department_id  → departments.id
```

Entity:

```java
@Entity
public class Employee {

    @Id
    private Long id;

    private String name;

    @ManyToOne
    @JoinColumn(name = "department_id")
    private Department department;
}
```

The foreign key lives in:

```text
employees.department_id
```

Therefore, the `Employee` side is the owning side.

---

# 11. Why is `@ManyToOne` usually the owning side?

### ⭐⭐⭐⭐⭐ Interview Question

**Which side owns a bidirectional relationship?**

The side containing the foreign key is generally the **owning side**.

For:

```text
Employee → Department
```

the employee table contains:

```text
department_id
```

Therefore:

```java
@ManyToOne
@JoinColumn(name = "department_id")
private Department department;
```

is the owning side.

---

# 12. `@OneToMany`

Suppose:

```text
Department
     |
     +---- Employee
     +---- Employee
     +---- Employee
```

We can represent the relationship from the Department side:

```java
@OneToMany(mappedBy = "department")
private List<Employee> employees;
```

Complete example:

```java
@Entity
public class Department {

    @Id
    private Long id;

    private String name;

    @OneToMany(mappedBy = "department")
    private List<Employee> employees;
}
```

Notice:

```java
mappedBy = "department"
```

This points to the field in `Employee`:

```java
@ManyToOne
private Department department;
```

---

# 13. What exactly does `mappedBy` mean?

### ⭐⭐⭐⭐⭐ Interview Question

`mappedBy` tells Hibernate:

> "This side is not responsible for managing the database relationship. The other entity owns it."

Example:

```java
@OneToMany(mappedBy = "department")
private List<Employee> employees;
```

`department` refers to:

```java
Employee.department
```

So:

```text
Department.employees
        |
        | mappedBy
        v
Employee.department
        |
        v
Owns FK department_id
```

### Critical point

`mappedBy` refers to the **Java field/property name**, not the database column name.

Correct:

```java
mappedBy = "department"
```

Not:

```java
mappedBy = "department_id"
```

---

# 14. Owning Side vs Inverse Side

Consider:

```java
@Entity
class Employee {

    @ManyToOne
    @JoinColumn(name = "department_id")
    private Department department;
}
```

and:

```java
@Entity
class Department {

    @OneToMany(mappedBy = "department")
    private List<Employee> employees;
}
```

Then:

```text
Employee
   |
   | OWNING SIDE
   |
   +---- department_id


Department
   |
   | INVERSE / NON-OWNING SIDE
   |
   +---- employees
```

The database relationship is controlled through:

```java
Employee.department
```

---

# 15. Classic Interview Trap

Suppose you do:

```java
department.getEmployees().add(employee);
```

but don't do:

```java
employee.setDepartment(department);
```

Will the foreign key necessarily be updated?

**No.**

Because `Employee` is the owning side.

The safer approach for a bidirectional relationship is to maintain both sides.

For example:

```java
public void addEmployee(Employee employee) {

    employees.add(employee);
    employee.setDepartment(this);
}
```

Now:

```text
Department
    |
    +---- employees
             |
             v
         Employee
             |
             +---- department
```

Both Java sides stay consistent.

---

# 16. Why use Helper Methods?

Bidirectional relationships can easily become inconsistent.

Bad:

```java
department.getEmployees().add(employee);
```

but:

```java
employee.getDepartment()
```

still returns `null`.

Better:

```java
public void addEmployee(Employee employee) {
    employees.add(employee);
    employee.setDepartment(this);
}
```

Similarly:

```java
public void removeEmployee(Employee employee) {
    employees.remove(employee);
    employee.setDepartment(null);
}
```

This is a practical production pattern.

---

# 17. `@JoinColumn`

### Interview Question
**What does `@JoinColumn` do?**

It specifies the database column used for the relationship.

Example:

```java
@ManyToOne
@JoinColumn(name = "department_id")
private Department department;
```

This means:

```text
employees.department_id
        |
        v
departments.id
```

The exact referenced column is normally the target entity's primary key unless configured otherwise.

---

# 18. `@OneToOne`

Suppose each user has one profile:

```text
User 1 ---- 1 Profile
```

Example:

```java
@Entity
class User {

    @OneToOne
    @JoinColumn(name = "profile_id")
    private Profile profile;
}
```

Database:

```text
users
-----
id
profile_id
```

The foreign key establishes the relationship.

A database-level `UNIQUE` constraint may be needed to truly enforce one-to-one semantics.

---

# 19. `@ManyToMany`

Suppose:

```text
Student
   |
   +---- Course
   +---- Course

Course
   |
   +---- Student
```

A relational database cannot normally represent this with one foreign-key column.

We use a **join table**.

```text
students
--------
id
name


courses
-------
id
name


student_courses
--------------
student_id
course_id
```

JPA:

```java
@ManyToMany
@JoinTable(
    name = "student_courses",
    joinColumns = @JoinColumn(name = "student_id"),
    inverseJoinColumns = @JoinColumn(name = "course_id")
)
private Set<Course> courses;
```

---

# 20. Why is `Set` often preferred for Many-to-Many?

Suppose:

```java
Set<Course> courses;
```

A `Set` naturally represents uniqueness.

A `List` permits duplicates conceptually.

However, don't blindly choose `Set`.

You must consider:

- `equals()`
- `hashCode()`
- entity identity
- collection behavior
- ordering requirements

Badly implemented `equals/hashCode` can cause subtle Hibernate collection bugs.

---

# 21. Why is Many-to-Many often discouraged in production?

### Senior-level question

A direct `@ManyToMany` looks convenient:

```java
@ManyToMany
private Set<Role> roles;
```

But real systems often need attributes on the relationship.

For example:

```text
User ---- Role
       |
       +-- assignedAt
       +-- assignedBy
       +-- status
```

Now the relationship itself has data.

Instead of:

```text
User <----> Role
```

create an explicit entity:

```text
User
 |
 +---- UserRole ---- Role
          |
          +-- assignedAt
          +-- assignedBy
```

This gives much better control.

For complex production systems, an explicit join entity is often preferable.

---

# 22. Cascade — ⭐⭐⭐⭐⭐

### Interview Question

**What is cascade in JPA?**

Cascade defines which operations performed on a parent entity should propagate to associated entities.

Example:

```java
@OneToMany(
    mappedBy = "order",
    cascade = CascadeType.ALL
)
private List<OrderItem> items;
```

If an `Order` is persisted:

```java
entityManager.persist(order);
```

the persist operation can cascade to its `OrderItem`s.

---

# 23. Cascade Types

Common cascade options:

```text
PERSIST
MERGE
REMOVE
REFRESH
DETACH
ALL
```

### `CascadeType.ALL`

Means all supported cascade operations.

Conceptually:

```text
Order
  |
  +-- persist  → items persist
  +-- merge    → items merge
  +-- remove   → items remove
  ...
```

---

# 24. Cascade Does NOT Mean Database Cascade

### ⭐ Important trap

These are different concepts.

JPA:

```java
cascade = CascadeType.REMOVE
```

means the ORM propagates the remove operation.

Database:

```sql
ON DELETE CASCADE
```

is a database-level foreign-key behavior.

So:

```text
JPA Cascade
    =
Hibernate/JPA operation propagation

DB Cascade
    =
Database foreign-key behavior
```

Don't treat them as the same thing.

---

# 25. `CascadeType.REMOVE` — Be Careful

Suppose:

```java
@OneToMany(
    cascade = CascadeType.ALL
)
private List<Employee> employees;
```

Deleting:

```java
entityManager.remove(department);
```

can cause employees to be deleted too.

That may be correct for:

```text
Order → OrderItems
```

because order items have no independent meaning.

But it can be dangerous for:

```text
User → Role
```

because roles may be shared.

You don't want:

```text
Delete User
   |
   v
Delete Role
   |
   v
Other users lose role
```

So cascade must reflect **ownership/lifecycle semantics**, not just convenience.

---

# 26. `orphanRemoval` — ⭐⭐⭐⭐⭐

### Interview Question

**What is `orphanRemoval = true`?**

It means that when a child entity is removed from the parent's relationship, JPA can delete that child entity from the database.

Example:

```java
@OneToMany(
    mappedBy = "order",
    orphanRemoval = true
)
private List<OrderItem> items;
```

Suppose:

```java
order.getItems().remove(item);
```

With orphan removal, that `OrderItem` can be deleted from the database.

Conceptually:

```text
Order
 |
 +-- Item A
 +-- Item B
 +-- Item C

remove Item B from collection

        ↓

Order
 |
 +-- Item A
 +-- Item C

Item B
  ↓
DELETE
```

---

# 27. `orphanRemoval` vs `CascadeType.REMOVE`

This is a **very common interview question**.

### `CascadeType.REMOVE`

Parent is removed:

```text
Delete Order
    |
    v
Delete OrderItems
```

### `orphanRemoval = true`

Child is removed from the relationship:

```text
Order
 |
 +-- Item A
 +-- Item B

remove Item B

       ↓

DELETE Item B
```

So:

```text
Cascade REMOVE
=
Parent deletion propagates to children

orphanRemoval
=
Removing child from parent's relationship can delete child
```

They can be used together.

---

# 28. When should orphan removal be used?

Use it when the child has a strong lifecycle dependency on the parent.

Good example:

```text
Order
 |
 +-- OrderItem
```

An order item generally doesn't make sense without its order.

Less appropriate:

```text
Employee
 |
 +-- Department
```

where the department is an independent entity.

---

# 29. `cascade = ALL` + `orphanRemoval = true`

A common aggregate pattern:

```java
@OneToMany(
    mappedBy = "order",
    cascade = CascadeType.ALL,
    orphanRemoval = true
)
private List<OrderItem> items;
```

Meaning:

```text
Order lifecycle
      |
      +---- create → items created
      |
      +---- update → item changes propagated
      |
      +---- remove item → orphan deleted
      |
      +---- delete order → items deleted
```

This can be appropriate when `OrderItem` is fully owned by `Order`.

---

# 30. Bidirectional Relationship Example

Let's build a realistic example.

### Department

```java
@Entity
public class Department {

    @Id
    @GeneratedValue
    private Long id;

    private String name;

    @OneToMany(
        mappedBy = "department",
        cascade = CascadeType.ALL,
        orphanRemoval = true
    )
    private List<Employee> employees = new ArrayList<>();

    public void addEmployee(Employee employee) {
        employees.add(employee);
        employee.setDepartment(this);
    }

    public void removeEmployee(Employee employee) {
        employees.remove(employee);
        employee.setDepartment(null);
    }
}
```

### Employee

```java
@Entity
public class Employee {

    @Id
    @GeneratedValue
    private Long id;

    private String name;

    @ManyToOne
    @JoinColumn(name = "department_id")
    private Department department;
}
```

Database:

```text
department
-----------
id
name


employee
--------
id
name
department_id
```

Relationship:

```text
Department
     |
     | 1
     |
     | *
     v
Employee
```

---

# 31. Why is `mappedBy` important in this example?

Without:

```java
mappedBy = "department"
```

Hibernate may interpret the relationship differently and potentially create an additional join structure depending on the mapping.

With:

```java
@OneToMany(mappedBy = "department")
```

we explicitly say:

> The relationship is already represented by `Employee.department`.

Therefore the foreign key remains:

```text
employee.department_id
```

---

# 32. Interview Scenario

### Question

You have:

```java
Department department;
Employee employee;
```

You execute:

```java
department.getEmployees().add(employee);
```

but don't set:

```java
employee.setDepartment(department);
```

Will Hibernate necessarily update `department_id`?

### Answer

**No.**

Because `Employee.department` is the owning side.

The database relationship is controlled by the owning side.

Correct:

```java
department.addEmployee(employee);
```

where:

```java
public void addEmployee(Employee employee) {
    employees.add(employee);
    employee.setDepartment(this);
}
```

---

# 33. Another Interview Question

### **Can both sides of a bidirectional relationship be owning sides?**

Typically, no.

There should be one owning side responsible for maintaining the relationship.

The other side uses:

```java
mappedBy
```

to indicate that it is the inverse/non-owning side.

---

# 34. `mappedBy` Does NOT Mean "Map By Database Column"

This mistake is extremely common.

Given:

```java
@ManyToOne
@JoinColumn(name = "department_id")
private Department department;
```

The inverse side uses:

```java
@OneToMany(mappedBy = "department")
```

Not:

```java
mappedBy = "department_id"
```

because `mappedBy` refers to:

```text
Java property:
department
```

not:

```text
Database column:
department_id
```

---

# 35. Production Consideration — Don't Automatically Use `CascadeType.ALL`

This is a senior-level point.

Avoid:

```java
cascade = CascadeType.ALL
```

just because it is convenient.

Ask:

> "Does the child have the same lifecycle as the parent?"

If yes:

```text
Order → OrderItem
```

could be appropriate.

If no:

```text
User → Role
```

it may be dangerous.

Cascade is fundamentally about **lifecycle ownership**.

---

# 36. Production Consideration — Avoid Huge Bidirectional Collections

Suppose:

```text
Customer
  |
  +---- 10 million transactions
```

Having:

```java
@OneToMany
private List<Transaction> transactions;
```

doesn't mean you should load all transactions into memory.

For large collections, prefer querying the child side with:

```text
pagination
filtering
keyset pagination
streaming where appropriate
```

rather than blindly traversing:

```java
customer.getTransactions();
```

This connects directly with the pagination and performance topics we'll cover later.

---

# 37. EPAM Rapid-Fire

### Q: What is the owning side?

The side responsible for maintaining the relationship, typically the side containing the foreign key.

### Q: What does `mappedBy` mean?

The relationship is owned by the field/property specified in `mappedBy`.

### Q: Does `mappedBy` refer to a column?

No. It refers to the Java property name.

### Q: Where is the foreign key usually stored for `ManyToOne`?

On the "many" side.

```text
employee.department_id
```

### Q: What is cascade?

Propagation of certain entity operations from one entity to associated entities.

### Q: Is JPA cascade the same as database `ON DELETE CASCADE`?

No.

### Q: What is orphan removal?

Removing a child from an association can cause the child to be deleted from the database.

### Q: Difference between cascade REMOVE and orphan removal?

`CascadeType.REMOVE` propagates parent removal. `orphanRemoval` handles removal of a child from the relationship.

### Q: Why are helper methods useful?

They keep both sides of bidirectional relationships synchronized in the Java object model.

### Q: Why can `ManyToMany` be problematic?

It becomes difficult when the relationship itself has attributes or requires independent lifecycle management; an explicit join entity is often better.

---

# 38. The Mental Model You Should Remember

For every relationship, ask:

```text
1. What is the cardinality?
       |
       v
2. Where is the foreign key?
       |
       v
3. Which side owns it?
       |
       v
4. Is the relationship bidirectional?
       |
       v
5. Do I need mappedBy?
       |
       v
6. Should operations cascade?
       |
       v
7. Should orphanRemoval apply?
       |
       v
8. What happens when the collection becomes large?
```

That reasoning is much more valuable in an interview than simply memorizing annotations.

---

# 39. Interview-Ready Final Explanation

If EPAM asks:

> **"Explain JPA relationships and owning side."**

A strong answer is:

> "JPA supports One-to-One, One-to-Many, Many-to-One and Many-to-Many relationships. In a relational database, the relationship is normally represented using foreign keys or join tables. The owning side is the side responsible for maintaining the relationship, typically the side containing the foreign key. For example, in Employee-to-Department, Employee has `department_id`, so `Employee.department` is the owning side and `Department.employees` uses `mappedBy = "department"`. `mappedBy` refers to the Java property, not the database column. Cascade controls propagation of entity operations, while orphanRemoval handles deletion of children removed from an association. I use cascade/orphan removal based on lifecycle ownership rather than adding them indiscriminately."

---

## Next: Fetching ⭐⭐⭐⭐⭐

Next we'll tackle one of the **highest-probability senior JPA interview areas**:

```text
LAZY vs EAGER
      ↓
N+1 Query Problem
      ↓
Fetch Join
      ↓
@EntityGraph
      ↓
Batch Fetching
      ↓
LazyInitializationException
```

And we'll use actual SQL examples so you can explain **exactly why an innocent-looking JPA method can suddenly execute 101 database queries instead of 1**.