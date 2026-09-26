# 🚀 B. ENTITY LIFECYCLE — Questions 31–53

This is one of the **most important JPA sections** for interviews. The interviewer can easily spend 10–15 minutes here by starting with "What are entity states?" and drilling down into `persist()`, `merge()`, dirty checking, persistence context, and first-level cache. These are the questions in your uploaded bank. 

---

## 31. What are the states of a JPA entity?

A JPA entity can be in four major states:

```text
                    persist()
        Transient -------------> Managed
           ↑                       |
           |                       |
           |                   remove()
           |                       |
           |                       v
           |                    Removed
           |                       |
           |                       |
           +---- new object <-------+

Managed
   |
   | detach() / clear() / close()
   v
Detached
```

The four states are:

1. **Transient**
2. **Managed / Persistent**
3. **Detached**
4. **Removed**

Understanding these states is fundamental to understanding JPA.

---

# 32. What is a transient entity?

A **transient entity** is a newly created Java object that is **not associated with a persistence context** and has not been persisted.

Example:

```java
Employee employee = new Employee();
employee.setName("Aryan");
```

At this point:

```text
Employee object
      ↓
Transient
      ↓
Not managed by EntityManager
      ↓
No persistence tracking
```

If you change it:

```java
employee.setName("Rahul");
```

JPA doesn't care.

There is no automatic SQL generated because the object isn't managed.

---

# 33. What is a managed/persistent entity?

A **managed entity** is an entity currently associated with a persistence context.

For example:

```java
Employee employee = entityManager.find(Employee.class, 1L);
```

The returned entity is managed.

Now:

```java
employee.setName("New Name");
```

Hibernate tracks this change.

When the persistence context is flushed, Hibernate can generate:

```sql
UPDATE employee
SET name = 'New Name'
WHERE id = 1;
```

You don't necessarily need:

```java
entityManager.save(employee);
```

In fact, `EntityManager` doesn't have a `save()` method.

This automatic change detection is called **dirty checking**.

---

# 34. What is a detached entity?

A **detached entity** was previously managed but is no longer associated with the current persistence context.

Example:

```java
Employee employee = entityManager.find(Employee.class, 1L);
```

Initially:

```text
employee → Managed
```

Then:

```java
entityManager.detach(employee);
```

Now:

```text
employee → Detached
```

If you do:

```java
employee.setName("New Name");
```

Hibernate does **not automatically track this change**.

You would generally need to reattach the state, commonly using:

```java
entityManager.merge(employee);
```

---

# 35. What is a removed entity?

A managed entity becomes **removed** when you call:

```java
entityManager.remove(employee);
```

Conceptually:

```text
Managed
   |
   | remove()
   v
Removed
```

The entity is scheduled for deletion from the database.

At flush/commit, Hibernate can execute:

```sql
DELETE FROM employee
WHERE id = 1;
```

One subtle point:

`remove()` doesn't necessarily mean the SQL `DELETE` executes immediately.

The operation is synchronized with the database during **flush**.

---

# 36. Explain the complete JPA entity lifecycle.

Let's walk through a complete example.

### Step 1 — Create object

```java
Employee employee = new Employee();
```

State:

```text
TRANSIENT
```

---

### Step 2 — Persist

```java
entityManager.persist(employee);
```

State:

```text
TRANSIENT
      ↓
MANAGED
```

The entity becomes associated with the persistence context.

---

### Step 3 — Modify

```java
employee.setName("Aryan");
```

Because it is managed, Hibernate tracks the modification.

```text
MANAGED
   ↓
dirty checking
```

---

### Step 4 — Detach

```java
entityManager.detach(employee);
```

Now:

```text
MANAGED
   ↓
DETACHED
```

Changes made now aren't automatically tracked.

---

### Step 5 — Merge

```java
Employee managedEmployee =
        entityManager.merge(employee);
```

Important:

> `merge()` returns a managed instance.

The original `employee` remains detached.

```text
Detached employee
       |
       | merge()
       v
Managed copy
```

This is a **very common interview trap**.

---

### Step 6 — Remove

```java
entityManager.remove(managedEmployee);
```

State:

```text
MANAGED
   ↓
REMOVED
```

At flush/commit:

```sql
DELETE ...
```

---

# 37. What does `persist()` do?

`persist()` makes a new entity **managed**.

Example:

```java
Employee employee = new Employee();

entityManager.persist(employee);
```

Before:

```text
employee → Transient
```

After:

```text
employee → Managed
```

Hibernate now tracks the entity.

Depending on ID generation strategy and flush behavior, SQL may execute immediately or later.

For example:

```java
entityManager.persist(employee);
```

may eventually produce:

```sql
INSERT INTO employee (...)
VALUES (...);
```

### Important

`persist()` does not simply mean:

> "Immediately execute INSERT."

It means:

> "Make this entity managed and schedule its persistence according to the persistence context/flush rules."

---

# 38. What does `merge()` do?

`merge()` is used to copy the state of a detached or otherwise non-managed entity into a **managed entity instance**.

Example:

```java
Employee detachedEmployee = ...;

Employee managedEmployee =
        entityManager.merge(detachedEmployee);
```

The critical point:

```text
detachedEmployee
       |
       | merge()
       v
managedEmployee
```

The original object does **not become managed**.

Instead, JPA returns a managed instance.

So:

```java
detachedEmployee == managedEmployee
```

is generally:

```text
false
```

This is one of the most frequently tested details.

---

# 39. `persist()` vs `merge()`?

| `persist()`                               | `merge()`                            |
| ----------------------------------------- | ------------------------------------ |
| Makes entity managed                      | Copies state into managed entity     |
| Usually used for new entities             | Commonly used with detached entities |
| Returns `void`                            | Returns managed entity               |
| Original object becomes managed           | Original object remains detached     |
| Intended for making new entity persistent | Can handle detached/new entity state |

Example:

```java
Employee e = new Employee();

entityManager.persist(e);

e.setName("A");
```

`e` is managed.

Whereas:

```java
Employee detached = ...;

Employee managed =
        entityManager.merge(detached);
```

`detached` remains detached.

### Interview trap

Don't say:

> "`merge()` reattaches the same object."

Better:

> "`merge()` copies the state of the given entity into a managed instance and returns that managed instance. The supplied entity itself remains detached."

---

# 40. What does `remove()` do?

`remove()` marks a **managed entity for deletion**.

```java
Employee employee =
    entityManager.find(Employee.class, 1L);

entityManager.remove(employee);
```

Conceptually:

```text
Managed
   |
   | remove()
   v
Removed
```

At flush:

```sql
DELETE FROM employee
WHERE id = 1;
```

Important:

```java
entityManager.remove(detachedEmployee);
```

is generally invalid because `remove()` expects a managed entity.

If you have a detached entity, you may first obtain/merge a managed instance, depending on the operation.

---

# 41. What does `detach()` do?

`detach()` removes an entity from the persistence context.

```java
entityManager.detach(employee);
```

Before:

```text
Persistence Context
        |
        v
    employee
     Managed
```

After:

```text
Persistence Context
        X
        |
     employee
     Detached
```

Changes made afterward aren't automatically detected.

Example:

```java
employee.setName("Changed");

entityManager.detach(employee);

employee.setName("Changed Again");
```

The second modification isn't tracked by that persistence context.

---

# 42. What does `clear()` do?

`clear()` detaches **all managed entities** from the persistence context.

```java
entityManager.clear();
```

Conceptually:

```text
Persistence Context

Employee 1 ──┐
Employee 2 ──┤
Employee 3 ──┤
Employee 4 ──┘

       clear()

       ↓

All detached
```

This is particularly useful in **large batch processing**.

For example:

```java
for (int i = 0; i < 1000000; i++) {

    entityManager.persist(employee);

    if (i % 1000 == 0) {
        entityManager.flush();
        entityManager.clear();
    }
}
```

Why?

Because otherwise the persistence context can keep references to a huge number of managed entities.

---

# 43. What does `refresh()` do?

`refresh()` reloads an entity's state from the database.

Example:

```java
Employee employee =
    entityManager.find(Employee.class, 1L);

employee.setName("Temporary");

entityManager.refresh(employee);
```

The in-memory state is refreshed from the database.

Conceptually:

```text
Java entity
     |
     | refresh()
     ↓
Database
     |
     ↓
Reload current DB state
```

So if the database currently contains:

```text
name = "Aryan"
```

the entity's temporary:

```text
"Temporary"
```

value can be replaced with the database value.

### When is this useful?

When you need to discard in-memory changes and synchronize the managed entity with the current database state.

---

# 44. What happens when a managed entity is modified without calling `save()`?

This is a **classic JPA interview question**.

Suppose:

```java
@Transactional
public void updateEmployee(Long id) {

    Employee employee =
        entityManager.find(Employee.class, id);

    employee.setName("New Name");
}
```

There is no:

```java
save(employee);
```

Yet Hibernate can still execute:

```sql
UPDATE employee
SET name = ?
WHERE id = ?
```

Why?

Because `employee` is **managed**.

Hibernate performs **dirty checking** during flush.

The flow is:

```text
find()
  ↓
Managed entity
  ↓
Modify entity
  ↓
Hibernate detects change
  ↓
Flush
  ↓
UPDATE SQL
  ↓
Transaction commit
```

This is one of the key advantages of JPA's persistence context.

---

# 45. What is dirty checking?

**Dirty checking** is the mechanism through which Hibernate detects changes made to managed entities and synchronizes those changes with the database.

Example:

```java
@Transactional
public void update(Long id) {

    Employee employee =
        entityManager.find(Employee.class, id);

    employee.setName("New Name");
}
```

You didn't explicitly execute:

```sql
UPDATE
```

Hibernate detects:

```text
Before:
name = "Old Name"

After:
name = "New Name"
```

and generates an update during flush.

### Conceptually

```text
Database
   ↓
Entity loaded
   ↓
Hibernate tracks state
   ↓
Application modifies entity
   ↓
Dirty checking
   ↓
Generate UPDATE
```

---

# 46. How does Hibernate detect changes to an entity?

At a high level, Hibernate maintains information about the entity's managed state and compares the current state with the previously known state when it performs dirty checking.

Conceptually:

```text
Original state
----------------
id = 1
name = "Aryan"
salary = 100000


Current state
----------------
id = 1
name = "Rahul"
salary = 100000
```

Hibernate sees:

```text
name changed
salary unchanged
```

and can generate an update for the changed state.

Modern Hibernate also uses enhanced dirty tracking in some configurations, but for interview purposes the fundamental concept is:

> Hibernate tracks managed entity state and determines whether persistent attributes have changed before synchronizing them during flush.

---

# 47. When does Hibernate generate the UPDATE statement?

The SQL `UPDATE` is generally generated during **flush**.

Flush can occur:

* Explicitly:

```java
entityManager.flush();
```

* Automatically before transaction commit
* Automatically before certain queries depending on flush mode and query/persistence-context synchronization requirements

So:

```java
employee.setName("New Name");
```

doesn't necessarily mean:

```text
UPDATE immediately
```

Instead:

```text
Modification
    ↓
Dirty state
    ↓
Flush
    ↓
SQL UPDATE
```

This distinction between **changing the Java object** and **executing SQL** is extremely important.

---

# 48. Why can Hibernate update an entity even if you never explicitly call `save()`?

Because `save()` isn't what drives dirty checking.

The entity is already managed:

```java
Employee employee =
    entityManager.find(Employee.class, id);
```

Then:

```java
employee.setName("New Name");
```

Hibernate tracks it.

At flush:

```text
Persistence Context
        ↓
Dirty checking
        ↓
SQL UPDATE
```

So in JPA:

> **Managed entity + transaction + dirty checking = automatic synchronization with DB.**

This is why code like this works:

```java
@Transactional
public void updateEmployee(Long id) {

    Employee employee = repository.findById(id)
                                  .orElseThrow();

    employee.setName("New Name");
}
```

No explicit:

```java
repository.save(employee);
```

is required for the update to be persisted when the entity is managed within the transaction.

---

# 49. What is the persistence context? 🔥

This is **one of the most important JPA concepts**.

A **persistence context** is a set of entity instances that are currently managed by an `EntityManager`.

Think of it as a managed workspace between your application and the database.

```text
Application
     |
     v
Persistence Context
     |
     v
Database
```

Suppose:

```java
Employee e =
    entityManager.find(Employee.class, 1L);
```

Hibernate puts that entity into the persistence context.

Now:

```java
e.setName("New Name");
```

Hibernate knows about the change because `e` is managed.

The persistence context provides things such as:

* Entity lifecycle management
* First-level cache
* Identity guarantee within the context
* Dirty checking
* Write-behind behavior
* Relationship management

### Think of it as:

```text
Persistence Context

Employee#1 → managed object
Employee#2 → managed object
Department#5 → managed object
...
```

It tracks these entities until they are detached, cleared, or the persistence context ends.

---

# 50. What is the first-level cache?

The **first-level cache** is the cache associated with the persistence context.

It is **enabled by default** and is effectively part of normal JPA persistence-context behavior.

Suppose:

```java
Employee e1 =
    entityManager.find(Employee.class, 1L);

Employee e2 =
    entityManager.find(Employee.class, 1L);
```

Hibernate doesn't need to issue two database queries in the same persistence context.

Conceptually:

```text
First find
    ↓
DB query
    ↓
Employee#1 stored in Persistence Context

Second find
    ↓
Persistence Context
    ↓
Return existing managed instance
```

Therefore:

```java
e1 == e2
```

will generally be:

```text
true
```

within that same persistence context.

---

# 51. Why is the first-level cache mandatory in a persistence context?

It's important because the persistence context provides an **identity guarantee** for managed entities.

Suppose you run:

```java
Employee e1 =
    entityManager.find(Employee.class, 1L);

Employee e2 =
    entityManager.find(Employee.class, 1L);
```

If Hibernate created two separate managed objects:

```text
Employee#1 → object A
Employee#1 → object B
```

you could have conflicting in-memory states.

For example:

```text
Object A:
name = Aryan

Object B:
name = Rahul
```

Both supposedly represent:

```text
employee id = 1
```

That would make persistence management extremely difficult.

Instead, within a persistence context:

```text
Employee ID 1
     ↓
ONE managed entity instance
```

This gives you **identity consistency**.

---

# 52. What happens if you load the same entity twice in the same persistence context?

Example:

```java
Employee e1 =
    entityManager.find(Employee.class, 1L);

Employee e2 =
    entityManager.find(Employee.class, 1L);
```

Within the same persistence context:

```java
e1 == e2
```

is expected to be:

```text
true
```

The second lookup can return the entity already present in the first-level cache instead of loading another instance from the database.

Conceptually:

```text
First find
   ↓
DB
   ↓
Persistence Context
   ↓
Employee#1


Second find
   ↓
Persistence Context
   ↓
Employee#1
```

This is called the **identity guarantee** of the persistence context.

---

# 53. What happens if you load the same entity from two different transactions?

This is where the previous answer changes.

Suppose:

```text
Transaction 1
    |
    ↓
Persistence Context 1
    |
    ↓
Employee#1


Transaction 2
    |
    ↓
Persistence Context 2
    |
    ↓
Employee#1
```

These are **different persistence contexts**.

Therefore, they can contain **different Java object instances** representing the same database row.

For example:

```java
// Transaction 1
Employee e1 = repository.findById(1L);

// Transaction 2
Employee e2 = repository.findById(1L);
```

Generally:

```java
e1 == e2
```

is:

```text
false
```

because they belong to different persistence contexts.

But they can represent the same database entity:

```text
e1.id = 1
e2.id = 1
```

This is why **database identity**, **persistence-context identity**, and **Java object identity** must not be confused.

---

# 🔥 The entire section in one picture

This is worth memorizing:

```text
                  new Employee()
                       |
                       v
                  TRANSIENT
                       |
                  persist()
                       |
                       v
                   MANAGED
                       |
             +---------+---------+
             |                   |
          modify()            detach()
             |                   |
             v                   v
       Dirty Checking         DETACHED
             |                   |
           flush()            merge()
             |                   |
             v                   v
          UPDATE             MANAGED COPY
                                 |
                              remove()
                                 |
                                 v
                              REMOVED
                                 |
                               flush
                                 |
                                 v
                              DELETE
```

And around the managed state:

```text
             Persistence Context
                    |
       +------------+------------+
       |            |            |
   Entity #1    Entity #2    Entity #3
       |
       ↓
 First-Level Cache
       |
       ↓
 Dirty Checking
       |
       ↓
 Flush
       |
       ↓
    Database
```

## 🎯 Interview traps from this section

Make sure these are crystal clear:

**`persist()`**

> Makes the entity managed.

**`merge()`**

> Copies state into a managed entity and returns the managed instance. It does **not** make the supplied object managed.

**`remove()`**

> Marks a managed entity for deletion.

**`detach()`**

> Detaches one entity.

**`clear()`**

> Detaches all entities in the persistence context.

**`refresh()`**

> Reloads the entity's state from the database.

**Dirty checking**

> Detects changes to managed entities and synchronizes them during flush.

**Persistence context**

> Set of entities currently managed by an `EntityManager`.

**First-level cache**

> Persistence-context-scoped cache that provides identity and avoids repeated database loads within that context.

**Most important distinction:**

```text
persist()  → entity becomes managed

merge()    → state gets copied to managed entity
             and managed entity is returned
```

Next is **Section C — EntityManager (Questions 54–68)**, where we'll go into `EntityManager` vs `EntityManagerFactory`, thread safety, `find()` vs `getReference()`, proxies, and what Hibernate actually does internally when `find()` is called.
