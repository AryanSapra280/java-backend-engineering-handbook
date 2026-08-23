# 10 — Object Methods: equals(), hashCode() and toString()

> Learning note based on our discussion, including the questions, confusions, reasoning, corrections, interview traps, and production-level considerations.

---

# 1. The Big Picture

Every Java class ultimately inherits from `Object`.

Three methods are especially important:

```text
Object
 ├── equals()
 ├── hashCode()
 └── toString()
```

They have different responsibilities:

```text
equals()
   ↓
"Are these two objects logically equal?"

hashCode()
   ↓
"What hash value represents this object's equality identity?"

toString()
   ↓
"How should this object be represented as text?"
```

The critical relationship is:

```text
equals() ↔ hashCode()
```

There is a contract between them.

`toString()` is independent of that contract.

---

# 2. `==` vs `equals()`

## Definition

`==` compares references when used with objects.

`equals()` is a method used to define **logical equality**.

Example:

```java
User u1 = new User(101);
User u2 = new User(101);
```

Even though both represent the same logical user:

```java
u1 == u2
```

is:

```text
false
```

because they are different object references.

If `User.equals()` is implemented based on `id`:

```java
u1.equals(u2)
```

can be:

```text
true
```

Therefore:

```text
==       → identity/reference comparison
equals() → logical equality
```

---

# 3. Designing `equals()`

Suppose business identity is `id`.

```java
class User {

    private int id;

    User(int id) {
        this.id = id;
    }

    @Override
    public boolean equals(Object obj) {

        if (this == obj) {
            return true;
        }

        if (!(obj instanceof User other)) {
            return false;
        }

        return this.id == other.id;
    }
}
```

The important question is:

> What does equality mean for this class?

For `User`, the answer might be:

```text
same user ID
```

For `Money`, it might be:

```text
same amount AND same currency
```

Equality should therefore come from the **business/domain definition**, not from blindly comparing every field.

---

# 4. The `equals()` Contract

We derived each rule rather than memorizing it.

## 4.1 Reflexive

### Definition

For any non-null object `x`:

```java
x.equals(x)
```

must return:

```text
true
```

Reason:

An object must be equal to itself.

---

## 4.2 Symmetric

### Definition

For objects `x` and `y`:

```java
x.equals(y) == y.equals(x)
```

If:

```java
u1.equals(u2) → true
```

then:

```java
u2.equals(u1) → true
```

must also hold.

### Why?

Equality cannot logically depend on which object is asked first.

---

## 4.3 Transitive

### Definition

If:

```text
x.equals(y) → true
y.equals(z) → true
```

then:

```text
x.equals(z) → true
```

must also be true.

Think:

```text
A = B
B = C
──────
A = C
```

Otherwise the definition of equality is inconsistent.

---

## 4.4 Consistent

### Definition

Repeated calls to:

```java
x.equals(y)
```

should return the same result as long as no relevant state changes.

Example:

```text
true
true
true
true
```

not:

```text
true
false
true
false
```

If relevant object state changes, the result can legitimately change.

---

## 4.5 Non-null

### Definition

For any non-null object `x`:

```java
x.equals(null)
```

must return:

```text
false
```

A common implementation naturally handles this:

```java
if (!(obj instanceof User other)) {
    return false;
}
```

because:

```java
null instanceof User
```

is `false`.

---

# 5. `equals()` + Inheritance — Major Interview Trap

Consider:

```java
class User {

    int id;

    @Override
    public boolean equals(Object obj) {
        if (!(obj instanceof User other)) {
            return false;
        }

        return this.id == other.id;
    }
}
```

And:

```java
class PremiumUser extends User {

    int premiumLevel;

    @Override
    public boolean equals(Object obj) {
        if (!(obj instanceof PremiumUser other)) {
            return false;
        }

        return this.id == other.id
            && this.premiumLevel == other.premiumLevel;
    }
}
```

Now:

```java
User u = new User(101);
PremiumUser p = new PremiumUser(101, 5);
```

### `u.equals(p)`

`PremiumUser` is a `User`:

```java
p instanceof User // true
```

Therefore:

```text
u.equals(p) → true
```

### `p.equals(u)`

But:

```java
u instanceof PremiumUser // false
```

Therefore:

```text
p.equals(u) → false
```

Result:

```text
u.equals(p) → true
p.equals(u) → false
```

This breaks **symmetry**.

---

# 6. `instanceof` vs `getClass()`

## `instanceof`

```java
if (!(obj instanceof User other)) {
    return false;
}
```

Means:

> This object considers instances of `User` and its subclasses eligible for equality.

Advantages:

- Allows polymorphic equality.
- Can be useful when equality is intentionally defined across a hierarchy.

Risk:

- Subclasses can introduce additional equality fields.
- This can break symmetry or transitivity.

---

## `getClass()`

```java
if (obj == null || getClass() != obj.getClass()) {
    return false;
}
```

Means:

> Equality is restricted to the exact same runtime class.

Therefore:

```text
User vs User
       ↓
eligible

User vs PremiumUser
       ↓
different runtime classes
       ↓
false
```

This avoids the specific inheritance symmetry problem.

---

## Important Interview Principle

Do NOT memorize:

> "`getClass()` is always better."

or:

> "`instanceof` is always better."

The real question is:

> **What does equality mean for this type, and can that definition remain symmetric and transitive across inheritance?**

If equality is based on exact type and value, `getClass()` is often appropriate.

If equality is intentionally polymorphic, `instanceof` can be appropriate — but the hierarchy must be designed carefully.

---

# 7. Why Immutable Value Objects Are Often `final`

Suppose:

```java
final class Money {
    private final BigDecimal amount;
    private final Currency currency;
}
```

If `Money` is `final`, nobody can create a subclass that changes the equality definition.

This is useful for value objects because their equality can remain stable.

General design lesson:

> If a class represents a value and its equality semantics should not be changed by subclasses, making it immutable and often `final` can simplify equality design.

---

# 8. `hashCode()`

## Definition

`hashCode()` returns an integer hash value used by hash-based data structures to efficiently locate objects.

Important collections include:

```text
HashMap
HashSet
Hashtable
```

Conceptually:

```text
object
  ↓
hashCode()
  ↓
hash value
  ↓
bucket
  ↓
equals() if necessary
```

---

# 9. Why `hashCode()` Exists

Suppose a `HashSet` contains thousands of objects.

It should not need to compare the new object against every existing object.

Instead:

```text
new object
    ↓
hashCode()
    ↓
candidate bucket
    ↓
compare with objects in that bucket
    ↓
equals()
```

This is why hashing can make lookup approximately constant-time under good distribution.

---

# 10. Hash Collision

Different objects can have the same hash code.

For example:

```text
Object A → hash 101
Object B → hash 101
```

This is allowed.

It does NOT mean:

```text
A.equals(B)
```

must be true.

Instead:

```text
same hash
   ↓
possible same bucket
   ↓
equals() decides logical equality
```

Therefore:

> **Hash collision is allowed.**

But poor hashing can create many collisions and hurt performance.

---

# 11. `equals()` / `hashCode()` Contract

The most important rule:

> **If two objects are equal according to `equals()`, they must have the same `hashCode()`.**

Formally:

```text
a.equals(b) == true
        ↓
a.hashCode() == b.hashCode()
```

The reverse is NOT required.

It is perfectly legal for:

```text
a.hashCode() == b.hashCode()
```

while:

```text
a.equals(b) == false
```

That is simply a collision.

---

# 12. Money Example

Suppose:

```java
class Money {

    private final int amount;
    private final String currency;
}
```

Business rule:

```text
Two Money objects are equal only if:
amount AND currency are equal.
```

Therefore:

```java
Money m1 = new Money(100, "INR");
Money m2 = new Money(100, "INR");
Money m3 = new Money(100, "USD");
```

Results:

```java
m1.equals(m2) // true
m1.equals(m3) // false
```

Both fields participate in equality:

```text
equals()
 ├── amount
 └── currency
```

Therefore the hash should be based consistently on those fields:

```java
@Override
public int hashCode() {
    return Objects.hash(amount, currency);
}
```

---

# 13. Important HashCode Nuance

It is technically legal to implement:

```java
@Override
public int hashCode() {
    return 1;
}
```

because equal objects will always have the same hash.

But this produces terrible distribution:

```text
many objects
     ↓
same hash
     ↓
same bucket
     ↓
many collisions
```

So the practical requirement is:

> `hashCode()` must be consistent with `equals()` and should distribute unequal objects well for good performance.

---

# 14. Mutable Keys — Production-Level Problem

Suppose:

```java
class User {
    int id;
}
```

and `id` participates in:

```java
equals()
hashCode()
```

Now:

```java
Map<User, String> map = new HashMap<>();

User user = new User(101);

map.put(user, "Aryan");
```

Later:

```java
user.id = 202;
```

The object's hash may now point to a different bucket.

Conceptually:

```text
put:
User(101)
   ↓
hash for 101
   ↓
bucket X


after mutation:
User(202)
   ↓
hash for 202
   ↓
bucket Y
```

But the object is still physically stored in bucket X.

A lookup using the mutated key may search bucket Y and fail to find the object.

This creates the classic:

> **"The object is in the HashMap, but I can't retrieve it."**

Production-level rule:

> **Avoid mutating fields that participate in `equals()`/`hashCode()` while an object is being used as a key in a hash-based collection.**

This is one reason immutable value objects are valuable.

---

# 15. `HashSet` and `equals()` / `hashCode()`

Suppose:

```java
User u1 = new User(101);
User u2 = new User(101);
```

If:

```java
u1.equals(u2) == true
```

and both have the same hash:

```text
hash(u1) == hash(u2)
```

then a `HashSet` treats them as the same logical element.

Conceptually:

```text
add(u1)
    ↓
bucket determined by hash
    ↓
u1 stored

add(u2)
    ↓
same hash → same candidate bucket
    ↓
equals()
    ↓
true
    ↓
u2 is not added
```

---

# 16. What If `equals()` Is Overridden But `hashCode()` Is Not?

This was an important question from our discussion.

Suppose:

```java
class User {

    int id;

    @Override
    public boolean equals(Object obj) {
        // compares id
    }

    // hashCode NOT overridden
}
```

Then `User` inherits `Object.hashCode()`.

Two logically equal objects may have different hash codes.

Conceptually:

```text
u1.equals(u2) → true

but

u1.hashCode() != u2.hashCode()
```

Now they may go to different buckets.

That violates the `equals()`/`hashCode()` contract and can break hash-based collections.

Therefore:

> **Whenever you override `equals()`, override `hashCode()` consistently.**

---

# 17. HashMap/HashSet Interview Mental Model

The simplified mental model is:

```text
key/object
    ↓
hashCode()
    ↓
hash processing
    ↓
bucket index
    ↓
candidate entries
    ↓
equals()
    ↓
match / no match
```

Important:

> Hash code identifies a candidate location; it does not prove equality.

---

# 18. `toString()`

## Definition

> **`toString()` returns a string representation of an object, primarily intended for human-readable debugging, logging, and inspection.**

Example:

```java
class User {

    int id;
    String name;

    User(int id, String name) {
        this.id = id;
        this.name = name;
    }
}
```

Without overriding `toString()`:

```java
User user = new User(101, "Aryan");

System.out.println(user);
```

may produce:

```text
User@5e2de80c
```

---

# 19. Does `println(object)` Print the HashCode?

Our initial reasoning was:

> "hashcode gets printed."

The precise answer is:

> **`println(Object)` ultimately uses `toString()`. It does not directly call `hashCode()` for printing.**

Conceptually:

```text
println(object)
      ↓
String.valueOf(object)
      ↓
object.toString()
      ↓
Object.toString() if not overridden
```

The default `Object.toString()` is essentially based on:

```java
getClass().getName() + "@" + Integer.toHexString(hashCode())
```

So output such as:

```text
User@5e2de80c
```

looks hash-related because the default `toString()` uses a hexadecimal representation of the object's hash code.

Therefore the interview answer should be:

> **"`println(Object)` uses `toString()`. The default `Object.toString()` includes the class name and a hexadecimal representation related to the hash code."**

---

# 20. Overriding `toString()`

Instead of:

```text
User@5e2de80c
```

we can provide useful information:

```java
@Override
public String toString() {
    return "User{id=" + id + ", name='" + name + "'}";
}
```

Now:

```java
System.out.println(user);
```

can produce:

```text
User{id=101, name='Aryan'}
```

Useful for:

- debugging
- logs
- troubleshooting
- inspecting objects
- debugging collections

---

# 21. `toString()` and Collections

Suppose:

```java
List<User> users = List.of(user1, user2);

System.out.println(users);
```

The collection's string representation ultimately uses the elements' `toString()` methods.

Therefore meaningful `toString()` implementations make debugging collections much easier.

---

# 22. Production-Level `toString()` Security Concern

Do NOT blindly include every field.

Bad:

```java
@Override
public String toString() {
    return "User{" +
           "username='" + username + '\'' +
           ", password='" + password + '\'' +
           ", creditCardNumber='" + creditCardNumber + '\'' +
           '}';
}
```

This is dangerous because object representations can end up in:

```text
application logs
debug logs
error messages
monitoring systems
```

Sensitive information should not be exposed.

Avoid putting things such as:

```text
passwords
access tokens
API keys
secret credentials
full card numbers
confidential data
```

into `toString()`.

A safer representation might include only non-sensitive fields:

```java
@Override
public String toString() {
    return "User{" +
           "username='" + username + '\'' +
           '}';
}
```

---

# 23. `toString()` Is Not Serialization

Do not use `toString()` as an application data format.

Bad:

```java
if (user.toString().equals(expectedValue)) {
    ...
}
```

`toString()` is for human-readable representation.

For actual serialization/API/database formats, use the appropriate serialization mechanism.

Think:

```text
toString()
   ↓
debugging / logging / inspection

equals()
   ↓
logical equality

hashCode()
   ↓
hash-based collections
```

---

# 24. Do I Have to Override `toString()` If I Override `equals()`?

No.

There is no contract requiring all three to be overridden together.

You can have:

```java
@Override
public boolean equals(...) { ... }

@Override
public int hashCode() { ... }
```

without overriding:

```java
toString()
```

The important contract is:

```text
equals() ↔ hashCode()
```

`toString()` is independent.

---

# 25. Final Mental Model

```text
                    Object
                      │
       ┌──────────────┼──────────────┐
       ↓              ↓              ↓
    equals()       hashCode()     toString()
       │              │              │
       │              │              │
       ↓              ↓              ↓
logical equality   hash lookup    text representation
       │              │              │
       └───────┐      │              │
               ↓      ↓
            HashMap / HashSet
```

### The contracts

```text
equals()
  ├── reflexive
  ├── symmetric
  ├── transitive
  ├── consistent
  └── false for null

hashCode()
  └── equal objects MUST have equal hash codes

toString()
  └── human-readable representation
```

---

# 26. Interview-Level Answer

If asked:

> "Explain `equals()`, `hashCode()` and `toString()`."

A strong concise answer:

> "`equals()` defines logical equality between objects, while `hashCode()` provides a hash representation used by hash-based collections such as `HashMap` and `HashSet`. If two objects are equal according to `equals()`, they must have the same hash code, although unequal objects can still collide. `toString()` provides a human-readable representation mainly for debugging and logging and is independent of the equals/hashCode contract. When overriding equals, I should override hashCode consistently, and when using mutable objects as hash keys, I should avoid mutating fields that participate in equality."

---

# 27. Questions We Resolved

### Q: Does `==` compare object contents?

No.

For objects:

```text
== → reference identity
equals() → logical equality
```

### Q: If `equals()` says two objects are equal, can their hash codes differ?

No.

```text
equals == true
      ↓
hashCode must be same
```

### Q: Can two unequal objects have the same hash code?

Yes.

That's a hash collision.

### Q: If `equals()` is overridden but `hashCode()` isn't?

Potentially broken hash-based collection behavior because logically equal objects can have different hash codes.

### Q: Does `HashSet` use only `hashCode()`?

No.

Hashing finds a candidate bucket; `equals()` is used to establish logical equality among candidates.

### Q: Why can inheritance break `equals()`?

A parent may define equality using fewer fields than a subclass, causing:

```text
parent.equals(child) → true
child.equals(parent) → false
```

breaking symmetry.

### Q: `instanceof` or `getClass()`?

Neither is universally correct.

The choice depends on whether equality should work across a class hierarchy and whether symmetry/transitivity can be preserved.

### Q: Does `println(object)` directly call `hashCode()`?

No.

It ultimately uses `toString()`. The default `Object.toString()` includes a hexadecimal representation related to the hash code.

### Q: Must `toString()` be overridden with `equals()`?

No.

`toString()` is independent.

### Q: Should every field be included in `toString()`?

No.

Sensitive fields must not be exposed through representations that can reach logs or monitoring systems.

---

# 28. Production-Level Rules to Remember

```text
1. Define equality from business identity/value semantics.
2. If equals() is overridden, hashCode() must be consistent.
3. Hash collisions are normal; equals() resolves logical equality.
4. Don't mutate equality/hash fields while objects are hash keys.
5. Be careful with equals() across inheritance.
6. Prefer immutable value objects where appropriate.
7. Use toString() for useful debugging/logging representation.
8. Never expose secrets or sensitive data through toString().
9. Don't use toString() as a serialization/API contract.
10. Remember: equals/hashCode are a contract pair; toString is separate.
```

---

# 29. Where This Fits in the Learning Track

```text
09. Composition vs Inheritance       ✅
10. Object Methods
    ├── equals()                    ✅
    ├── hashCode()                  ✅
    └── toString()                  ✅
11. String / String Pool             ← NEXT
12. Wrapper Classes
13. Collections
14. Generics
15. Exception Handling
16. Java 8+
17. Records / Sealed Classes
18. Modern Java Features
```

**Collections deliberately comes later.** The `HashSet`/`HashMap` discussion above was used to understand the `equals()`/`hashCode()` contract, not to start the Collections module.
