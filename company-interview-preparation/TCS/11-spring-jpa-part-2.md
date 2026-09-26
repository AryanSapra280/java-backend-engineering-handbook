Absolutely — let’s continue **Spring Data JPA + Hibernate**, quickly and interview-focused. 🔥

# Spring Data JPA — Part 2

## 32. What is `@OneToMany` and `@ManyToOne`?

Suppose one **Account** can have many **Transactions**:

```java
@Entity
class Account {

    @OneToMany(mappedBy = "account")
    private List<Transaction> transactions;
}
```

```java
@Entity
class Transaction {

    @ManyToOne
    @JoinColumn(name = "account_id")
    private Account account;
}
```

Database:

```text
account
-------
id

transaction
-----------
id
account_id  --> account.id
```

### Interview point

The **foreign key exists in the Transaction table**, so `Transaction` is the owning side.

---

# 33. What is the owning side of a relationship?

The **owning side is the entity that controls the foreign-key relationship**.

Usually, it is the side containing:

```java
@JoinColumn
```

Example:

```java
@ManyToOne
@JoinColumn(name = "account_id")
private Account account;
```

`Transaction` is the owning side.

The other side uses:

```java
mappedBy = "account"
```

### Important trap

`mappedBy` means:

> "I am not the owner. The relationship is managed by the field on the other entity."

---

# 34. What does `mappedBy` mean?

Example:

```java
@OneToMany(mappedBy = "account")
private List<Transaction> transactions;
```

Here:

```text
mappedBy = "account"
```

refers to this field:

```java
@ManyToOne
private Account account;
```

It is **not the database column name**.

### Common interview trap

❌ Wrong:

```java
mappedBy = "account_id"
```

Usually it should be the **Java property name**:

```java
mappedBy = "account"
```

---

# 35. What is Cascade in JPA?

Cascade determines whether operations performed on a parent should propagate to its associated entities.

Example:

```java
@OneToMany(
    mappedBy = "account",
    cascade = CascadeType.ALL
)
private List<Transaction> transactions;
```

Now operations on `Account` can cascade to `Transaction`.

Common types:

```text
PERSIST
MERGE
REMOVE
REFRESH
DETACH
ALL
```

### Example

```java
Account account = new Account();

Transaction tx = new Transaction();

account.getTransactions().add(tx);

entityManager.persist(account);
```

With:

```java
cascade = CascadeType.PERSIST
```

the transaction can also be persisted.

---

# 36. What is `CascadeType.ALL`?

It means all cascade operations:

```text
PERSIST
MERGE
REMOVE
REFRESH
DETACH
```

are cascaded.

### Interview trap

Don't blindly use:

```java
cascade = CascadeType.ALL
```

Especially with:

```java
@ManyToMany
```

or relationships where deleting one entity should **not** delete associated entities.

---

# 37. What is `orphanRemoval = true`?

This is an important interview question.

Suppose:

```java
@OneToMany(
    mappedBy = "account",
    orphanRemoval = true
)
private List<Transaction> transactions;
```

If a transaction is removed from the parent's collection:

```java
account.getTransactions().remove(tx);
```

Hibernate can delete that orphaned child from the database.

So:

```text
Parent
  |
  +-- Child A
  +-- Child B
```

Remove Child B from collection:

```text
Parent
  |
  +-- Child A
```

With `orphanRemoval = true`:

```sql
DELETE FROM transaction WHERE id = ?
```

---

# 38. `CascadeType.REMOVE` vs `orphanRemoval`

Very common interview question.

### Cascade REMOVE

Parent is deleted:

```java
entityManager.remove(account);
```

The delete operation cascades to children.

### orphanRemoval

Child is removed from parent's relationship:

```java
account.getTransactions().remove(tx);
```

The child itself can be deleted.

### Simple distinction

```text
Cascade REMOVE
    ↓
Delete parent
    ↓
Delete children

orphanRemoval
    ↓
Remove child from relationship
    ↓
Delete child
```

---

# 39. When should you use `orphanRemoval`?

When the child has **no meaningful independent existence outside the parent**.

Example:

```text
Order
 └── OrderItems
```

An `OrderItem` may belong exclusively to an Order.

But be careful with:

```text
Customer
 └── Address
```

if the address can exist independently or be shared.

---

# 40. What happens with bidirectional relationships?

Example:

```java
Account account = new Account();
Transaction tx = new Transaction();

account.getTransactions().add(tx);
```

You also need:

```java
tx.setAccount(account);
```

Because JPA does **not automatically synchronize both Java-side references**.

A good approach is helper methods:

```java
public void addTransaction(Transaction tx) {
    transactions.add(tx);
    tx.setAccount(this);
}
```

Then:

```java
account.addTransaction(tx);
```

Keeps both sides synchronized.

### Interview point

The database relationship is controlled by the **owning side**, but your Java object graph should also be kept consistent.

---

# 41. Why can `@OneToMany` cause performance problems?

Consider:

```java
@OneToMany(mappedBy = "account")
private List<Transaction> transactions;
```

Then:

```java
List<Account> accounts = accountRepository.findAll();
```

If you subsequently do:

```java
for (Account account : accounts) {
    account.getTransactions().size();
}
```

You can get:

```text
1 query → accounts

N queries → transactions
```

That's the classic **N+1 problem**.

We already covered the solutions:

```java
JOIN FETCH
@EntityGraph
batch fetching
```

---

# 42. What is `@ManyToMany`?

Example:

```text
Student <----> Course
```

One student can have many courses.

One course can have many students.

Usually database design uses a join table:

```text
student
-------
id

course
------
id

student_course
--------------
student_id
course_id
```

JPA:

```java
@ManyToMany
@JoinTable(
    name = "student_course",
    joinColumns = @JoinColumn(name = "student_id"),
    inverseJoinColumns = @JoinColumn(name = "course_id")
)
private Set<Course> courses;
```

### Interview advice

For real business systems, if the relationship itself has attributes:

```text
Student
Course
Enrollment
  enrollmentDate
  grade
  status
```

don't blindly use `@ManyToMany`.

Model the join table as an **entity**:

```text
Student 1 --- * Enrollment * --- 1 Course
```

This is much more flexible.

---

# 43. What is `@OneToOne`?

Example:

```text
User
 |
 +--- UserProfile
```

```java
@OneToOne
@JoinColumn(name = "profile_id")
private UserProfile profile;
```

Database:

```text
user
----
id
profile_id
```

Or the foreign key can be on the profile table.

---

# 44. What is `Pageable` in Spring Data JPA?

Spring Data provides:

```java
Pageable
```

for pagination.

Example:

```java
Page<Account> findAll(Pageable pageable);
```

Usage:

```java
Pageable pageable =
    PageRequest.of(0, 20);

Page<Account> result =
    repository.findAll(pageable);
```

Meaning:

```text
page = 0
size = 20
```

SQL is conceptually:

```sql
LIMIT 20
OFFSET 0
```

---

# 45. `Page` vs `Slice`

Important interview question.

### Page

```java
Page<Account>
```

Provides:

```text
content
totalElements
totalPages
currentPage
hasNext()
...
```

To know total records, Spring may execute an additional:

```sql
SELECT COUNT(*)
```

### Slice

```java
Slice<Account>
```

Provides information about whether another slice exists:

```java
slice.hasNext()
```

but doesn't require total count.

### Simple distinction

```text
Page
→ data + total count

Slice
→ data + whether next exists
```

For huge datasets where you don't need total count, `Slice` can avoid the count query.

---

# 46. What is a projection in Spring Data JPA?

Projection means retrieving **only the fields you need**, instead of loading the complete entity.

Suppose:

```java
@Entity
class Employee {
    Long id;
    String name;
    String email;
    String address;
    String salary;
    ...
}
```

But API only needs:

```text
id
name
```

Projection:

```java
public interface EmployeeView {
    Long getId();
    String getName();
}
```

Repository:

```java
List<EmployeeView> findByDepartment(String department);
```

Instead of loading the entire entity.

### Why useful?

```text
Less data
Less memory
Less DB/network transfer
Potentially better performance
```

---

# 47. JPQL vs Native Query

### JPQL

Works with **entities and fields**:

```java
@Query("""
    SELECT e
    FROM Employee e
    WHERE e.department = :department
""")
```

It is database-independent at the query language level.

### Native SQL

Works directly with database tables:

```java
@Query(
    value = """
        SELECT *
        FROM employee
        WHERE department = :department
    """,
    nativeQuery = true
)
```

Use native queries when you need database-specific features or complex SQL that JPQL doesn't express conveniently.

---

# 48. What is `@Query`?

Spring Data lets you define custom queries:

```java
@Query("""
    SELECT a
    FROM Account a
    WHERE a.status = :status
""")
List<Account> findAccounts(
    @Param("status") String status
);
```

Without writing an implementation class.

---

# 49. What is Spring Data JPA Specification?

This is another common interview topic.

`Specification` allows **dynamic query construction**.

Suppose filters are optional:

```text
status
department
salary
joiningDate
```

Instead of creating:

```text
findByStatusAndDepartment()
findByStatusAndSalary()
findByDepartmentAndSalary()
findByStatusAndDepartmentAndSalary()
...
```

you can dynamically build predicates.

Conceptually:

```java
Specification<Employee> spec =
    (root, query, cb) ->
        cb.equal(root.get("status"), "ACTIVE");
```

Then:

```java
repository.findAll(spec);
```

Very useful for search/filter APIs.

---

# 50. What happens with bulk update queries?

Example:

```java
@Modifying
@Query("""
    UPDATE Account a
    SET a.status = 'CLOSED'
    WHERE a.id = :id
""")
int closeAccount(Long id);
```

Need:

```java
@Modifying
```

because it's not a normal SELECT query.

Often:

```java
@Transactional
@Modifying
@Query(...)
```

### VERY IMPORTANT TRAP

Bulk update operates directly on the database and **bypasses normal entity dirty checking**.

So the persistence context can become stale.

Example:

```text
Persistence Context:
Account.status = ACTIVE

Bulk SQL:
Account.status = CLOSED

Persistence Context still:
ACTIVE
```

You may need:

```java
@Modifying(clearAutomatically = true)
```

or explicitly clear/refresh as appropriate.

---

# 51. `save()` vs `saveAndFlush()`

### `save()`

Persists/merges the entity through the repository.

It doesn't necessarily immediately send SQL to the DB.

### `saveAndFlush()`

```java
repository.saveAndFlush(entity);
```

causes a flush immediately.

But:

> **Flush does NOT mean commit.**

The transaction can still roll back afterward.

Example:

```text
saveAndFlush()
     ↓
SQL sent to DB
     ↓
later exception
     ↓
transaction rollback
```

---

# 52. How do you handle 1 lakh records from JPA?

This is **very relevant to the interview question you recently got.**

Don't simply do:

```java
List<Employee> employees = repository.findAll();
```

for huge data.

Possible approaches:

### 1. Pagination

```java
Page<Employee>
```

Process chunks:

```text
0–999
1000–1999
...
```

### 2. Streaming

Spring Data JPA can return:

```java
Stream<Employee>
```

Example:

```java
@Query("SELECT e FROM Employee e")
Stream<Employee> streamEmployees();
```

Then:

```java
try (Stream<Employee> stream =
         repository.streamEmployees()) {

    stream.forEach(this::process);
}
```

But this requires careful transaction/resource handling.

### 3. Batch processing

For very large datasets:

```text
Spring Batch
+ JdbcCursorItemReader
+ JdbcPagingItemReader
```

can be more appropriate.

### Interview answer

> "I wouldn't load one lakh records into a List at once. I'd use pagination, streaming, or batch processing depending on the use case. For a large ETL-style operation, I'd generally prefer Spring Batch with paging/cursor-based readers and chunk processing."

That's a **strong answer**.

---

# 53. Optimistic locking — practical scenario

Suppose two users read the same account:

```text
User A → balance = 1000
User B → balance = 1000
```

Both modify it.

With:

```java
@Version
private Long version;
```

DB might have:

```text
id | balance | version
1  | 1000    | 5
```

User A updates:

```sql
UPDATE account
SET balance = 900,
    version = 6
WHERE id = 1
AND version = 5;
```

Success.

User B still has:

```text
version = 5
```

Its update affects:

```text
0 rows
```

Hibernate detects the conflict and throws an optimistic locking exception.

### Best for

```text
Reads >> writes
Low contention
```

---

# 54. Pessimistic locking — practical scenario

If you need to lock the row while processing:

```java
@Lock(LockModeType.PESSIMISTIC_WRITE)
Optional<Account> findById(Long id);
```

Conceptually:

```sql
SELECT ...
FROM account
WHERE id = ?
FOR UPDATE;
```

Other transactions trying to acquire conflicting locks may have to wait.

### Best when

```text
High contention
Critical concurrent updates
Strong database-level serialization required
```

But locks can increase:

```text
waiting
deadlocks
database contention
```

---

# 🔥 JPA Interview Rapid Fire

Memorize these:

| Question             | Answer                                                                            |
| -------------------- | --------------------------------------------------------------------------------- |
| JPA?                 | Specification                                                                     |
| Hibernate?           | JPA implementation                                                                |
| Spring Data JPA?     | Repository abstraction over JPA                                                   |
| Persistence Context? | Managed entity context                                                            |
| First-level cache?   | Persistence-context scoped                                                        |
| Second-level cache?  | Shared across persistence contexts/session factories depending on provider/config |
| Dirty checking?      | Detect changes to managed entities                                                |
| `flush()`?           | Synchronize persistence context with DB                                           |
| `commit()`?          | Finalize transaction                                                              |
| `save()`?            | Persist/merge depending on entity state/newness                                   |
| `merge()`?           | Copies state into managed entity                                                  |
| Lazy loading?        | Load association when accessed                                                    |
| N+1?                 | 1 query + N association queries                                                   |
| `JOIN FETCH`?        | Fetch association in same query                                                   |
| `mappedBy`?          | Indicates inverse/non-owning side                                                 |
| Cascade?             | Propagate entity operations                                                       |
| orphanRemoval?       | Remove child when orphaned from relationship                                      |
| `Page`?              | Data + total count/page metadata                                                  |
| `Slice`?             | Data + next-slice information                                                     |
| Projection?          | Fetch selected fields                                                             |
| `@Modifying`?        | Indicates UPDATE/DELETE query                                                     |
| `@Version`?          | Optimistic locking                                                                |
| Pessimistic lock?    | DB-level locking                                                                  |
| `@Transactional`?    | Defines transaction boundary                                                      |
| Default propagation? | `REQUIRED`                                                                        |
| Default rollback?    | RuntimeException/Error                                                            |

---

## ✅ JPA/Hibernate is now sufficiently covered for the interview

The next major topic is:

# 👉 Spring Security

We'll cover it in the same fast sequence:

1. Authentication vs Authorization
2. Spring Security architecture
3. Security Filter Chain
4. `UserDetails`
5. `UserDetailsService`
6. Password encoding
7. JWT
8. JWT request flow
9. Access token vs refresh token
10. Roles vs authorities
11. `hasRole()` vs `hasAuthority()`
12. CORS vs CSRF
13. Stateless security
14. OAuth2 basics
15. Method-level security
16. Common Spring Security interview traps
