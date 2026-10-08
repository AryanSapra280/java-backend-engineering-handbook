Absolutely — moving to the **Spring Data JPA Repository Layer** now. This is very practical and directly relevant for EPAM.

# 4. Spring Data JPA / Hibernate ⭐⭐⭐⭐⭐
## Part 6 — Repository Layer, Query Methods, JPQL, Native Queries & Specifications

---

# 1. What is Spring Data JPA?

### Interview Question
**What is Spring Data JPA?**

Spring Data JPA is a Spring abstraction that simplifies data-access development using JPA.

Instead of manually implementing common repository operations, you define an interface:

```java
public interface UserRepository
        extends JpaRepository<User, Long> {
}
```

Spring Data creates the repository implementation for you.

Conceptually:

```text
Service
   |
   v
UserRepository
   |
   v
Spring Data JPA
   |
   v
EntityManager
   |
   v
Hibernate
   |
   v
JDBC
   |
   v
Database
```

---

# 2. `CrudRepository` vs `JpaRepository`

### Interview Question
**What's the difference between `CrudRepository` and `JpaRepository`?**

`CrudRepository` provides basic CRUD operations.

```java
public interface UserRepository
        extends CrudRepository<User, Long> {
}
```

It provides operations such as:

```java
save()
findById()
findAll()
delete()
existsById()
count()
```

`JpaRepository` provides the CRUD capabilities plus JPA-oriented functionality and extends the Spring Data repository hierarchy.

```java
public interface UserRepository
        extends JpaRepository<User, Long> {
}
```

For a typical Spring Boot JPA application, `JpaRepository` is commonly used.

### Interview answer

> "`CrudRepository` provides basic CRUD abstraction, while `JpaRepository` builds on the repository hierarchy and provides additional JPA-specific repository functionality."

---

# 3. What happens when you call `repository.findById()`?

### Interview Question

What happens underneath this?

```java
User user = userRepository.findById(10L)
                          .orElseThrow();
```

Conceptually:

```text
Service
   |
   v
UserRepository
   |
   v
Spring Data JPA implementation
   |
   v
EntityManager
   |
   v
Hibernate
   |
   v
Database
```

Hibernate may execute:

```sql
SELECT *
FROM users
WHERE id = ?;
```

The resulting entity becomes managed in the current persistence context when obtained through the JPA persistence context.

---

# 4. What is a Derived Query?

### Interview Question

**What is a derived query method in Spring Data JPA?**

Spring Data can derive a query from the repository method name.

Example:

```java
List<User> findByName(String name);
```

Spring Data interprets the method name and creates the corresponding query.

Another example:

```java
List<User> findByStatusAndDepartment(
    Status status,
    Department department
);
```

Conceptually:

```sql
SELECT *
FROM users
WHERE status = ?
  AND department_id = ?;
```

You don't manually write the query.

---

# 5. Common Derived Query Keywords

Important keywords include:

```text
findBy
getBy
readBy
countBy
existsBy
deleteBy
removeBy
```

Conditions:

```text
And
Or
Between
LessThan
GreaterThan
LessThanEqual
GreaterThanEqual
Like
Containing
StartingWith
EndingWith
In
NotIn
IsNull
IsNotNull
True
False
```

Example:

```java
List<User> findByAgeGreaterThan(int age);
```

Conceptually:

```sql
WHERE age > ?
```

---

# 6. Multiple Conditions

Example:

```java
List<User> findByStatusAndAgeGreaterThan(
    Status status,
    int age
);
```

Conceptually:

```sql
WHERE status = ?
AND age > ?
```

Another:

```java
List<User> findByNameOrEmail(
    String name,
    String email
);
```

Conceptually:

```sql
WHERE name = ?
OR email = ?
```

---

# 7. Ordering

You can express ordering through the method name.

Example:

```java
List<User> findByStatusOrderByCreatedAtDesc(
    Status status
);
```

Conceptually:

```sql
WHERE status = ?
ORDER BY created_at DESC
```

For more complex queries, `Sort` and `Pageable` are usually cleaner.

---

# 8. Pagination with Repository Methods

Example:

```java
Page<User> findByStatus(
    Status status,
    Pageable pageable
);
```

Call:

```java
Pageable pageable =
    PageRequest.of(
        0,
        20,
        Sort.by("createdAt").descending()
    );

Page<User> page =
    repository.findByStatus(status, pageable);
```

This allows Spring Data to incorporate pagination and sorting into the query.

---

# 9. `Page` vs `Slice`

### Interview Question

**What's the difference between `Page` and `Slice`?**

### `Page`

Provides pagination metadata, including total-element/total-page information.

Conceptually:

```text
Page
 ├── content
 ├── page number
 ├── page size
 ├── total elements
 └── total pages
```

Getting total counts can require an additional count query.

### `Slice`

Primarily tells you whether another slice exists.

```text
Slice
 ├── content
 ├── page number
 ├── page size
 └── hasNext
```

It can avoid the need for a total-count query.

### Production consideration

For an API like:

```text
GET /users?page=0&size=20
```

if the UI doesn't need:

```text
"there are 2,347,821 users"
```

a `Slice` can sometimes be more efficient than `Page`.

---

# 10. `@Query`

### Interview Question

**When would you use `@Query`?**

Use `@Query` when the query is too complex or expressive for a derived method name, or when you want explicit query control.

Example:

```java
@Query("""
    select u
    from User u
    where u.status = :status
      and u.age > :age
""")
List<User> findActiveUsers(
    @Param("status") Status status,
    @Param("age") int age
);
```

This is JPQL.

---

# 11. What is JPQL?

### Interview Question

**What is JPQL?**

JPQL stands for **Jakarta Persistence Query Language**.

The important distinction:

> JPQL queries **entities and their attributes**, not database tables and columns.

Suppose:

```java
@Entity
@Table(name = "users")
public class User {

    private String name;
}
```

JPQL:

```java
select u
from User u
where u.name = :name
```

Not:

```sql
select *
from users
where user_name = ?
```

The second is SQL.

---

# 12. JPQL vs SQL

### JPQL

```java
select u
from User u
where u.status = :status
```

Uses:

```text
Entity name
Java property
```

### SQL

```sql
SELECT *
FROM users
WHERE status = ?
```

Uses:

```text
Table
Database column
```

Hibernate translates JPQL into SQL appropriate for the configured database.

---

# 13. Why use JPQL?

Advantages:

- Object/entity-oriented
- Database-independent at the query language level
- Works with entity relationships
- Integrates with JPA

Example:

```java
@Query("""
    select o
    from Order o
    join o.customer c
    where c.email = :email
""")
List<Order> findOrdersByCustomerEmail(
    @Param("email") String email
);
```

Notice:

```text
Order
Customer
email
```

are entity/property concepts.

---

# 14. JPQL Relationship Navigation

Suppose:

```java
Order
 |
 +-- Customer
```

You can write:

```java
select o
from Order o
where o.customer.email = :email
```

You don't need to manually write:

```sql
JOIN customers ...
```

Hibernate generates appropriate SQL.

---

# 15. `JOIN FETCH`

We've already discussed fetch joins.

Example:

```java
@Query("""
    select o
    from Order o
    join fetch o.customer
    where o.id = :id
""")
Optional<Order> findOrderWithCustomer(
    @Param("id") Long id
);
```

The important distinction is:

```text
JOIN
=
relationship participates in query

JOIN FETCH
=
relationship participates in query
+
associated entity is fetched as part of query
```

---

# 16. Native Query

### Interview Question

**What is a native query?**

A native query is actual SQL executed against the database.

Example:

```java
@Query(
    value = """
        SELECT *
        FROM users
        WHERE status = :status
    """,
    nativeQuery = true
)
List<User> findUsers(
    @Param("status") String status
);
```

Now you're writing database SQL rather than JPQL.

---

# 17. JPQL vs Native Query

| JPQL | Native SQL |
|---|---|
| Entity-oriented | Database-oriented |
| Uses entity/property names | Uses tables/columns |
| More database-independent | Database-specific |
| Portable | Less portable |
| Good for most ORM queries | Useful for DB-specific/complex queries |

### Interview answer

> "I prefer JPQL or repository-derived queries for normal entity-oriented access. I use native SQL when I genuinely need database-specific functionality, complex SQL, vendor-specific features, or a query that is difficult or inappropriate to express through JPQL."

---

# 18. Should we always use Native SQL for performance?

### No.

This is a common misconception.

Hibernate can generate efficient SQL for many use cases.

The choice should depend on:

- query complexity
- database-specific features
- execution plan
- maintainability
- portability
- actual performance measurements

Don't say:

> "Native SQL is always faster."

That's not generally true.

---

# 19. `@Modifying`

### Interview Question

**How do you execute an UPDATE or DELETE query using `@Query`?**

For modifying queries, Spring Data JPA provides:

```java
@Modifying
@Query("""
    update User u
    set u.status = :status
    where u.id = :id
""")
int updateStatus(
    @Param("id") Long id,
    @Param("status") Status status
);
```

The method can return the number of affected rows.

---

# 20. Why is `@Modifying` needed?

Without it, Spring Data treats the query as a regular query expecting results.

`@Modifying` tells Spring Data:

> "This query modifies data."

It is commonly used with:

```text
UPDATE
DELETE
```

queries.

---

# 21. Important Trap — Bulk UPDATE and Persistence Context

This is **very important**.

Suppose:

```java
User user =
    entityManager.find(User.class, 10L);
```

Now the persistence context contains:

```text
User#10
status = ACTIVE
```

Then you execute a JPQL bulk update:

```java
@Modifying
@Query("""
    update User u
    set u.status = 'INACTIVE'
""")
int deactivateAll();
```

The database rows are updated directly.

But the already-managed `User#10` object in the persistence context may still contain:

```text
status = ACTIVE
```

because bulk operations bypass normal entity dirty checking.

---

# 22. `clearAutomatically`

Spring Data's `@Modifying` supports options such as:

```java
@Modifying(clearAutomatically = true)
```

This can clear the persistence context after the modifying query.

Example:

```java
@Modifying(clearAutomatically = true)
@Query("""
    update User u
    set u.status = :status
    where u.id = :id
""")
int updateStatus(...);
```

Now you avoid continuing with stale managed entity instances from that persistence context.

### Important

Don't blindly add it everywhere. Clearing the persistence context also detaches other managed entities.

---

# 23. `flushAutomatically`

`@Modifying` also supports:

```java
flushAutomatically = true
```

This can flush pending changes before executing the modifying query.

Why might that matter?

Suppose the persistence context contains pending changes:

```text
Managed entity changes
       |
       v
not flushed yet
       |
bulk UPDATE
```

You need to reason carefully about ordering and consistency.

---

# 24. Bulk Update vs Entity Update

### Normal entity update

```java
User user = repository.findById(id).orElseThrow();

user.setStatus(INACTIVE);
```

Hibernate:

```text
Managed entity
      |
dirty checking
      |
flush
      |
UPDATE
```

### Bulk update

```java
@Modifying
@Query("""
    update User u
    set u.status = :status
""")
```

Conceptually:

```text
JPQL bulk operation
      |
      v
Database UPDATE
```

It doesn't load every matching entity into the persistence context and perform normal dirty checking on each one.

This makes bulk operations useful for large updates, but you must understand persistence-context synchronization.

---

# 25. What is a Projection?

### Interview Question

**Why would you use a projection instead of loading the complete entity?**

Suppose your API only needs:

```text
id
name
email
```

but `User` has:

```text
id
name
email
address
profile
preferences
auditData
...
```

Loading the complete entity can be unnecessary.

A projection can retrieve only the required data.

For example, an interface projection:

```java
public interface UserSummary {

    Long getId();

    String getName();
}
```

Repository:

```java
List<UserSummary> findByStatus(Status status);
```

The exact generated SQL depends on the projection/query/provider, but conceptually you're asking for only the fields needed by the use case.

---

# 26. DTO Projection with JPQL

You can also use constructor expressions.

```java
@Query("""
    select new com.example.dto.UserSummary(
        u.id,
        u.name
    )
    from User u
    where u.status = :status
""")
List<UserSummary> findSummaries(
    @Param("status") Status status
);
```

This can be useful when you want:

```text
Database
   |
   v
Only required columns
   |
   v
DTO
   |
   v
API
```

rather than:

```text
Database
   |
   v
Full Entity
   |
   v
DTO
```

---

# 27. Why are DTO Projections Useful?

They can reduce:

- selected columns
- transferred data
- entity creation
- persistence-context management
- accidental lazy loading

They are particularly useful for read-heavy API endpoints.

---

# 28. Specifications

### Interview Question

**What is Spring Data JPA Specification?**

A `Specification` provides a programmatic way to build dynamic predicates for queries.

It is based on the JPA Criteria API.

Suppose a search API supports:

```text
status
minimumAmount
maximumAmount
createdAfter
customerId
```

Instead of creating many repository methods:

```text
findByStatus()
findByStatusAndAmount()
findByStatusAndAmountAndDate()
findByStatusAndCustomer()
...
```

you can dynamically construct predicates.

Conceptually:

```text
Request filters
      |
      v
Specification
      |
      v
Criteria predicates
      |
      v
Database query
```

---

# 29. Example Specification

Suppose:

```java
public static Specification<User> hasStatus(Status status) {

    return (root, query, cb) ->
        cb.equal(root.get("status"), status);
}
```

Another:

```java
public static Specification<User> nameContains(String name) {

    return (root, query, cb) ->
        cb.like(
            cb.lower(root.get("name")),
            "%" + name.toLowerCase() + "%"
        );
}
```

Then combine:

```java
Specification<User> spec =
    Specification.where(hasStatus(ACTIVE))
                 .and(nameContains("aryan"));
```

Repository:

```java
public interface UserRepository
        extends JpaRepository<User, Long>,
                JpaSpecificationExecutor<User> {
}
```

Then:

```java
repository.findAll(spec);
```

---

# 30. When should you use Specifications?

Good use case:

```text
Dynamic search/filter API
```

For example:

```text
GET /users?
    status=ACTIVE
    &department=IT
    &minAge=25
    &name=aryan
```

The combination of filters can vary.

Specifications prevent an explosion of repository methods.

---

# 31. Specification vs JPQL

### JPQL

Good when:

```text
Query structure is known
```

Example:

```java
@Query("""
    select u
    from User u
    where u.status = :status
""")
```

### Specification

Good when:

```text
Filters are dynamic
```

Example:

```text
status?
department?
name?
date range?
minimum amount?
```

The exact combination isn't known at compile time.

---

# 32. Criteria API

### Interview Question

**What is Criteria API?**

Criteria API is the JPA API for programmatically constructing queries.

Instead of:

```java
select u from User u where u.status = :status
```

you construct the query through Java objects such as:

```text
CriteriaBuilder
CriteriaQuery
Root
Predicate
```

Spring Data Specifications are built on this style of criteria-based querying.

### Interview-level takeaway

You don't need to memorize every Criteria API method.

Know:

> "Specifications provide a convenient Spring Data abstraction for dynamically constructing JPA criteria predicates."

---

# 33. Derived Query vs JPQL vs Specification vs Native SQL

This is an excellent interview comparison.

| Approach | Best Use |
|---|---|
| Derived query | Simple fixed query |
| JPQL `@Query` | Explicit entity-oriented query |
| Specification | Dynamic filters |
| Native SQL | DB-specific/complex SQL |
| DTO projection | Read only required fields |

Think:

```text
Simple
  ↓
Derived Query

Explicit
  ↓
JPQL

Dynamic
  ↓
Specification

Database-specific
  ↓
Native SQL

Read optimized
  ↓
Projection
```

---

# 34. What happens when you call a derived query?

Example:

```java
List<User> findByStatusAndAgeGreaterThan(
    Status status,
    int age
);
```

Conceptually:

```text
Method name
    |
    v
Spring Data parses method
    |
    v
Creates query representation
    |
    v
JPA EntityManager
    |
    v
Hibernate
    |
    v
SQL
    |
    v
Database
```

You don't manually implement the repository method.

---

# 35. Repository Layer Architecture

A clean architecture often looks like:

```text
Controller
    |
    v
Service
    |
    v
Repository
    |
    v
Spring Data JPA
    |
    v
EntityManager
    |
    v
Hibernate
    |
    v
JDBC
    |
    v
Database
```

The service layer should generally contain business logic rather than putting business workflows inside repository interfaces.

---

# 36. Important Production Question

### **Should we put complex business logic into repository queries?**

Avoid turning repositories into business-logic containers.

Repository responsibility:

```text
Data access
Querying
Persistence
```

Service responsibility:

```text
Business rules
Workflow
Transaction boundary
Coordination
```

For example:

```java
@Transactional
public void processPayment(...) {

    Payment payment =
        paymentRepository.findById(...);

    validatePayment(payment);

    reserveFunds(...);

    payment.setStatus(SUCCESS);
}
```

The repository shouldn't own the entire business workflow.

---

# 37. `Optional` in Repository Methods

Common:

```java
Optional<User> findById(Long id);
```

This communicates that the entity may not exist.

Then:

```java
User user = repository.findById(id)
                      .orElseThrow(
                          () -> new UserNotFoundException(id)
                      );
```

For collection queries, Spring Data generally returns an empty collection rather than `null`.

So prefer:

```java
List<User>
```

and handle:

```text
empty list
```

rather than expecting:

```text
null
```

---

# 38. Delete Query vs `deleteBy...`

Spring Data can derive delete operations too.

Example:

```java
long deleteByStatus(Status status);
```

But understand what behavior you're asking Spring Data/Hibernate to perform.

For large bulk operations, an explicit bulk `DELETE` query may be more appropriate than loading and deleting a large number of entities individually.

The important production question is:

> **Am I deleting entities one-by-one, or can the database perform this as a set-based operation?**

---

# 39. Large Data — Don't Accidentally Load Everything

This is particularly relevant to your large PF/ledger workloads.

Avoid:

```java
List<LedgerEntry> entries =
    repository.findAll();
```

if the table contains millions of rows.

Instead consider:

```text
Pagination
Keyset pagination
Batch processing
Streaming where appropriate
Bulk operations
Database-side filtering
Projections
```

The correct choice depends on the workload.

---

# 40. Interview Scenario — 800 Million Rows

Suppose:

```text
ledger = 800 million rows
```

and interviewer asks:

> "Would you use `findAll()` and process them in Java?"

Strong answer:

> "No. I would avoid loading the complete table into one persistence context or JVM. I'd push filtering to the database, partition the workload, use appropriate pagination or keyset/range-based processing, process bounded batches, and control persistence-context size. For very large ETL workloads, I would also evaluate whether JPA is the right abstraction at all; JDBC/bulk processing or a specialized data-processing approach may be more appropriate."

That is the kind of practical answer expected from a senior engineer.

---

# 41. EPAM Rapid-Fire

### Q: What is `JpaRepository`?

A Spring Data repository abstraction providing CRUD and additional JPA-oriented repository functionality.

### Q: What is a derived query?

A query inferred from the repository method name.

### Q: What is JPQL?

An entity-oriented query language defined by JPA.

### Q: Does JPQL use table names?

No. It normally uses entity names and entity attributes.

### Q: What is a native query?

Actual database SQL.

### Q: When would you use native SQL?

When database-specific features, complex SQL, or other concrete requirements make JPQL inappropriate.

### Q: What does `@Modifying` do?

Marks a repository `@Query` as a modifying operation such as UPDATE or DELETE.

### Q: What is the danger with bulk updates?

They operate directly on database rows and can leave already-managed entities in the persistence context stale.

### Q: What is `clearAutomatically`?

A Spring Data `@Modifying` option that can clear the persistence context after the modifying operation.

### Q: What is a projection?

A way to retrieve only the data needed by a query/use case rather than necessarily materializing the complete entity.

### Q: What is Specification?

A Spring Data mechanism for dynamically building query predicates using JPA criteria.

### Q: When use Specification?

Dynamic filtering/search requirements.

### Q: `Page` vs `Slice`?

`Page` provides total-count/page metadata; `Slice` focuses on the current chunk and whether another slice exists.

### Q: What should repositories contain?

Data-access concerns, not complex business workflows.

---

# 42. The Decision Tree to Remember

When you need a query:

```text
Is it simple?
     |
    YES
     |
Derived Query
     |
    NO
     |
Is the query fixed?
     |
    YES
     |
JPQL @Query
     |
    NO
     |
Are filters dynamic?
     |
    YES
     |
Specification
     |
    NO
     |
Does it require DB-specific SQL?
     |
    YES
     |
Native Query
```

And for response optimization:

```text
Need complete entity?
       |
      YES
       |
Entity query


Need only a few fields?
       |
      YES
       |
Projection / DTO
```

---

# 43. One Strong Senior-Level Answer

### Interviewer:

> "How do you decide between derived queries, JPQL, native queries and Specifications?"

### Answer:

> "For simple fixed conditions, I prefer derived query methods because they're concise and type-safe at the repository API level. When the query is more complex but still naturally expressed in terms of entities and relationships, I use JPQL with `@Query`. For dynamic search APIs where the combination of filters varies, I use Specifications or another dynamic query mechanism. I use native SQL when I need database-specific functionality or when the query is significantly better expressed using database features. For read-heavy endpoints, I also consider projections so that I don't load an entire entity graph unnecessarily."

That's a **Senior Software Engineer-level answer** because you're choosing based on the use case rather than saying one approach is universally better.

---

# Next: Transactions ⭐⭐⭐⭐⭐

The next section is one of the **highest-priority JPA topics** for your EPAM interview:

```text
@Transactional
     ↓
Transaction Boundary
     ↓
Propagation
     ↓
Isolation
     ↓
Rollback
     ↓
readOnly
     ↓
Self-invocation
     ↓
Async + Transactions
     ↓
External API + Transaction
```

We'll connect it directly to the transaction concepts we already covered in Spring Core, but now from the **JPA/database perspective**, including the practical scenarios senior interviewers like to ask.