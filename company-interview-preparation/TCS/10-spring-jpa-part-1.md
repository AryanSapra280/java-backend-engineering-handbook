# 🔥 Spring Data JPA + Hibernate — Part 1

This is **high priority** for tomorrow. We'll move fast but cover the questions that commonly lead to follow-ups.

---

## 1. What is JPA?

> **JPA (Jakarta Persistence API) is a specification for mapping Java objects to relational database tables.**

It defines things like:

* Entities
* Relationships
* Persistence context
* EntityManager
* JPQL
* Transactions

**JPA itself is not an implementation.**

Common implementation:

> **Hibernate**

Think:

```text
JPA = Specification
Hibernate = Implementation
Spring Data JPA = Abstraction on top of JPA
```

---

# 2. Hibernate vs JPA vs Spring Data JPA

🔥 Very common.

```text
Spring Data JPA
       ↓
      JPA
       ↓
   Hibernate
       ↓
   JDBC
       ↓
   Database
```

### JPA

Defines APIs/specification.

### Hibernate

Implements JPA and handles ORM.

### Spring Data JPA

Provides repository abstractions:

```java
public interface UserRepository
        extends JpaRepository<User, Long> {
}
```

So you don't need to write basic CRUD implementation manually.

---

# 3. What is ORM?

**Object Relational Mapping.**

Maps:

```text
Java Object       Database
-----------       --------
User       →      users
id         →      id
name       →      name
```

Instead of manually doing:

```sql
SELECT id, name FROM users;
```

and manually constructing objects, ORM frameworks handle much of this mapping.

---

# 4. What is `@Entity`?

Marks a Java class as a persistent entity.

```java
@Entity
public class User {

    @Id
    private Long id;

    private String name;
}
```

Hibernate maps it to a database table.

---

# 5. What is `@Id`?

Defines the primary key of the entity.

```java
@Id
private Long id;
```

Every JPA entity needs an identifier.

---

# 6. What is `@GeneratedValue`?

Used to specify how an ID is generated.

```java
@Id
@GeneratedValue(strategy = GenerationType.IDENTITY)
private Long id;
```

Common strategies include:

* `IDENTITY`
* `SEQUENCE`
* `TABLE`
* `AUTO`

For PostgreSQL, sequences are often a useful/high-performance choice depending on the schema and application requirements.

---

# 7. What is the Persistence Context?

🔥 **Extremely important.**

> A persistence context is a set of entity instances that are managed by an EntityManager.

Think of it as Hibernate maintaining a managed entity context:

```text
Persistence Context
       |
       +-- User #1
       +-- User #2
       +-- Order #1
```

It provides things like:

* First-level cache
* Entity identity
* Dirty checking
* Change tracking

---

# 8. What is the EntityManager?

`EntityManager` is the JPA API used to interact with the persistence context.

Examples:

```java
entityManager.persist(user);
entityManager.find(User.class, id);
entityManager.remove(user);
```

Spring Data JPA internally uses JPA infrastructure / EntityManager.

---

# 9. What are JPA Entity States?

🔥 Very important.

An entity can be:

### Transient

New Java object, not managed.

```java
User user = new User();
```

### Managed / Persistent

Associated with the persistence context.

```java
entityManager.persist(user);
```

### Detached

Previously managed but no longer associated with the persistence context.

```text
transaction/session ends
        ↓
entity becomes detached
```

### Removed

Marked for deletion.

```java
entityManager.remove(user);
```

Visual:

```text
Transient
   ↓ persist
Managed
   ↓ detach / context closes
Detached

Managed
   ↓ remove
Removed
```

---

# 10. What is First-Level Cache?

🔥 Extremely common.

The persistence context acts as the **first-level cache**.

Example:

```java
User u1 = entityManager.find(User.class, 1L);
User u2 = entityManager.find(User.class, 1L);
```

Within the same persistence context, Hibernate can return the same managed entity instance instead of issuing another database query.

Conceptually:

```text
find(User, 1)
     ↓
Database
     ↓
Persistence Context

find(User, 1)
     ↓
Persistence Context
     ↓
same managed object
```

### Important

First-level cache is associated with the persistence context/EntityManager.

---

# 11. Does `save()` immediately execute SQL?

Not necessarily.

Example:

```java
userRepository.save(user);
```

The entity can become managed, but the actual SQL may be executed later, typically during **flush/transaction commit**.

For example:

```text
save()
 ↓
Persistence Context
 ↓
flush
 ↓
SQL
 ↓
commit
```

This is why seeing `save()` doesn't necessarily mean the database has already been updated at that exact line.

---

# 12. What is Dirty Checking?

🔥🔥 Very important.

Hibernate automatically detects changes to managed entities.

Example:

```java
@Transactional
public void updateUser(Long id) {

    User user = repository.findById(id).orElseThrow();

    user.setName("New Name");
}
```

You don't necessarily need:

```java
repository.save(user);
```

Hibernate detects that the managed entity changed.

At flush:

```text
Original state:
name = "Old"

        ↓

Java code:
name = "New"

        ↓

Dirty checking

        ↓

UPDATE users
SET name = 'New'
...
```

### Interview answer

> Dirty checking is Hibernate's mechanism of detecting changes made to managed entities and synchronizing those changes with the database during flush.

---

# 13. What is `flush()`?

Flush synchronizes changes in the persistence context with the database.

```text
Persistence Context
       ↓
     flush
       ↓
SQL statements
       ↓
Database
```

**Flush is not the same as commit.**

This distinction is important.

```text
flush → synchronize changes with DB
commit → finalize transaction
```

A transaction can flush SQL and still subsequently roll back.

---

# 14. What is `save()` in Spring Data JPA?

`JpaRepository` provides:

```java
save(entity)
```

It typically delegates to JPA operations such as:

* `persist()` for new entities
* `merge()` for entities considered not new

The exact determination can depend on entity state/newness detection strategy.

---

# 15. `persist()` vs `merge()`

🔥 Important.

### `persist()`

Makes a new entity managed.

```text
new Entity
   ↓ persist
Managed Entity
```

### `merge()`

Copies state from a detached entity into a managed entity.

Important trap:

> `merge()` does not necessarily make the object you passed in become managed.

Conceptually:

```text
Detached object
      ↓ merge
Managed copy
```

The returned object is the managed instance.

---

# 16. Lazy vs Eager Loading

🔥🔥 Must know.

Suppose:

```java
@OneToMany(fetch = FetchType.LAZY)
private List<Order> orders;
```

### LAZY

Associated data is loaded when accessed.

```text
load User
   ↓
User loaded

user.getOrders()
   ↓
Orders query
```

### EAGER

Associated data is intended to be loaded immediately as part of fetching the entity association.

But don't assume it always means one SQL query; Hibernate may use different SQL/query strategies.

### Interview recommendation

For collections, **LAZY is generally safer** because EAGER relationships can cause unnecessary data loading and performance issues.

---

# 17. What is the N+1 Query Problem?

🔥🔥🔥 Extremely important.

Suppose:

```java
List<User> users = userRepository.findAll();
```

and each user has orders.

Then:

```java
for (User user : users) {
    user.getOrders();
}
```

Could produce:

```text
1 query → fetch users

N queries → fetch orders for each user
```

Total:

```text
1 + N queries
```

For 1000 users:

```text
1 + 1000 = 1001 queries
```

This can severely impact performance.

---

# 18. How do you solve N+1?

Common approaches:

### `JOIN FETCH`

```java
@Query("""
    SELECT u
    FROM User u
    JOIN FETCH u.orders
""")
List<User> findUsersWithOrders();
```

This tells Hibernate to fetch the association as part of the query.

### Entity Graph

```java
@EntityGraph(attributePaths = "orders")
List<User> findAll();
```

### Batch fetching

Hibernate can batch lazy association loading rather than executing one query per parent.

---

# 19. `JOIN` vs `JOIN FETCH`

🔥 Important.

JPQL:

```sql
JOIN
```

can join tables for filtering/query purposes.

But:

```java
JOIN FETCH u.orders
```

specifically tells JPA:

> Fetch the associated entities as part of this query.

So:

```text
JOIN
→ join for query semantics

JOIN FETCH
→ join + initialize/fetch association
```

---

# 20. What is an EntityGraph?

An EntityGraph specifies which associations should be fetched for a particular query.

Example:

```java
@EntityGraph(attributePaths = {"orders"})
List<User> findAll();
```

It can be a cleaner alternative to writing explicit fetch joins for certain use cases.

---

# 21. What is the LazyInitializationException?

🔥 Common interview scenario.

Suppose:

```java
@Transactional
public User getUser() {
    return repository.findById(id).orElseThrow();
}
```

Transaction ends.

Later, outside the persistence context:

```java
user.getOrders();
```

If `orders` is lazy and the persistence context is closed:

```text
Lazy association
      ↓
needs database
      ↓
persistence context closed
      ↓
LazyInitializationException
```

### How to prevent it?

Common solutions:

* Fetch required data inside transaction
* `JOIN FETCH`
* EntityGraph
* DTO projection
* Explicitly initialize where appropriate

Don't solve everything by making relationships EAGER.

---

# 22. What is the Open Session in View (OSIV) pattern?

Spring Boot can keep the persistence context available across the web request.

This can prevent some lazy-loading exceptions in the controller/view layer.

But it can also hide inefficient database access and cause queries to happen during response rendering.

For service-oriented APIs, many teams prefer controlling transaction boundaries explicitly rather than relying on lazy loading from the web layer.

For interview purposes:

> OSIV keeps the persistence context open for the duration of a web request, allowing lazy loading beyond the service transaction, but it can hide performance problems.

---

# 23. What is `@Transactional`?

🔥🔥 Very important.

It defines a transactional boundary.

```java
@Transactional
public void transferMoney() {
    debit();
    credit();
}
```

Conceptually:

```text
BEGIN
  ↓
debit
  ↓
credit
  ↓
COMMIT
```

If an appropriate exception causes rollback:

```text
BEGIN
  ↓
debit
  ↓
credit → failure
  ↓
ROLLBACK
```

---

# 24. Where should `@Transactional` usually be placed?

Usually on the **service layer**.

```java
@Service
public class PaymentService {

    @Transactional
    public void processPayment() {
        ...
    }
}
```

Why?

Because the service layer usually represents the business transaction boundary.

---

# 25. Is `@Transactional` magic?

No. 😄

Spring commonly implements it using **proxies/AOP**.

Conceptually:

```text
Controller
   ↓
Spring Proxy
   ↓
@Transactional method
   ↓
Transaction begins
   ↓
Actual method
   ↓
Commit / Rollback
```

This leads to an important interview trap.

---

# 26. What is the self-invocation problem with `@Transactional`?

Example:

```java
@Service
class PaymentService {

    public void process() {
        savePayment();
    }

    @Transactional
    public void savePayment() {
        ...
    }
}
```

Calling:

```java
process()
```

then:

```java
this.savePayment()
```

doesn't go through the Spring proxy.

Therefore the `@Transactional` advice on `savePayment()` may not be applied.

### Why?

```text
External call
   ↓
Proxy
   ↓
Transactional method
```

But:

```text
method A
  ↓
this.method B()
  ↓
same object
```

No proxy interception.

### Common solution

Move the transactional method to another Spring bean/service, or structure the transaction boundary differently.

---

# 27. Default rollback behavior of `@Transactional`

By default, Spring generally rolls back for:

> `RuntimeException` and `Error`

Checked exceptions do not automatically trigger rollback under the default rules.

You can configure:

```java
@Transactional(rollbackFor = Exception.class)
```

This is a **very common interview question**.

---

# 28. What is transaction propagation?

Propagation defines what happens when a transactional method calls another transactional method.

Most important:

### `REQUIRED`

Default.

```text
Existing transaction?
   YES → join it
   NO  → create one
```

### `REQUIRES_NEW`

Suspends existing transaction and starts a new one.

```text
Transaction A
    ↓
Transaction B
    ↓
B commits independently
```

### `MANDATORY`

Requires an existing transaction.

### `SUPPORTS`

Uses existing transaction if present; otherwise runs without one.

For tomorrow, remember especially:

```text
REQUIRED
REQUIRES_NEW
```

---

# 29. What is transaction isolation?

Controls how one transaction can see changes made by other concurrent transactions.

Common levels:

```text
READ_UNCOMMITTED
READ_COMMITTED
REPEATABLE_READ
SERIALIZABLE
```

Potential anomalies:

```text
Dirty Read
Non-repeatable Read
Phantom Read
```

Typical idea:

```text
Higher isolation
     ↓
More consistency
     ↓
Potentially more locking/contention
```

We'll cover locking/isolation more later if needed.

---

# 30. Optimistic vs Pessimistic Locking

🔥 Important.

### Optimistic locking

Assumes conflicts are relatively uncommon.

Typically uses:

```java
@Version
private Long version;
```

Example:

```text
Transaction A reads version 1
Transaction B reads version 1

A updates → version 2

B tries update version 1
       ↓
conflict
```

Usually results in an optimistic locking exception.

### Pessimistic locking

Assumes conflicts may happen and acquires database locks.

Example:

```java
@Lock(LockModeType.PESSIMISTIC_WRITE)
```

Conceptually:

```text
Transaction A
    ↓
Lock row
    ↓
Update
    ↓
Transaction B waits/fails depending on DB/lock configuration
```

---

# ⚡ Rapid-fire JPA/Hibernate questions

### JPA vs Hibernate?

JPA = specification.

Hibernate = implementation.

---

### Spring Data JPA?

Repository abstraction built on JPA.

---

### First-level cache?

Persistence-context-level cache.

---

### Second-level cache?

Shared across persistence contexts at the SessionFactory/EntityManagerFactory level, depending on provider/configuration.

Not enabled/configured the same way as first-level cache.

---

### Dirty checking?

Detects modifications to managed entities.

---

### `save()` always executes INSERT?

No.

It may result in INSERT or UPDATE depending on entity state/newness.

---

### `flush()` vs `commit`?

```text
flush  → synchronize persistence context with DB
commit → finalize transaction
```

---

### LAZY vs EAGER?

```text
LAZY  → association loaded when needed
EAGER → association fetched as part of entity loading according to provider/query strategy
```

---

### N+1?

1 query for parent data + N queries for associated data.

---

### Common N+1 solutions?

* Fetch join
* EntityGraph
* Batch fetching
* DTO projections

---

### `JOIN FETCH`?

Fetches association as part of the query and initializes it.

---

### Why DTOs?

API boundary, security, performance, avoiding entity exposure/coupling.

---

### `@Transactional` works through?

Typically Spring AOP proxy interception.

---

### Self-invocation?

Internal `this.method()` call bypasses the proxy.

---

### Default transaction propagation?

`REQUIRED`.

---

### Default rollback?

Runtime exceptions and Errors; checked exceptions don't trigger rollback by default.

---

# 🎯 JPA Part 1 DONE

Your **must-remember mental model**:

```text
Controller
    ↓
@Service
    ↓
@Transactional
    ↓
Spring Proxy
    ↓
Persistence Context
    ↓
Repository / EntityManager
    ↓
Hibernate
    ↓
JDBC
    ↓
Database
```

And:

```text
Persistence Context
       ↓
First-level cache
       ↓
Managed entities
       ↓
Dirty checking
       ↓
Flush
       ↓
SQL
```

### Next JPA section

We'll cover the remaining **high-frequency traps**:

* `@OneToMany`, `@ManyToOne`, `@OneToOne`, `@ManyToMany`
* owning side / `mappedBy`
* `CascadeType`
* `orphanRemoval`
* `CascadeType.REMOVE` vs `orphanRemoval`
* bidirectional relationships
* `Lazy` collection pitfalls
* pagination
* `Page` vs `Slice`
* projections
* native query vs JPQL
* Specifications
* bulk updates
* `save()` vs `saveAndFlush()`
* optimistic/pessimistic locking scenarios
* JPA performance questions

Then we can move to **Spring Security**.
