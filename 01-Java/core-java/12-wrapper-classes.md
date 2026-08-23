# 12 — Wrapper Classes

## 1. Why Wrapper Classes Exist

Java provides wrappers for primitive types:

| Primitive | Wrapper |
|---|---|
| byte | Byte |
| short | Short |
| int | Integer |
| long | Long |
| float | Float |
| double | Double |
| char | Character |
| boolean | Boolean |

Generics work with reference types, not primitives:

```java
List<int> numbers = new ArrayList<>(); // invalid
List<Integer> numbers = new ArrayList<>(); // valid
```

Fundamental idea:

> Primitives are efficient value types; wrappers provide object/reference representations needed by generics and object-based APIs.

---

## 2. Autoboxing

Java automatically converts a primitive to its wrapper when required.

```java
List<Integer> numbers = new ArrayList<>();
numbers.add(10);
```

Conceptually:

```text
int 10 → Integer
```

This is **autoboxing**.

Think of boxing as being associated with mechanisms such as:

```java
Integer.valueOf(10)
```

Do not mentally model every autoboxing operation as `new Integer(10)`, because caching can be involved.

---

## 3. Unboxing

The reverse is **unboxing**:

```java
Integer x = 10;
int y = x;
```

Conceptually:

```text
Integer → int
```

---

## 4. Primitive vs Wrapper and `null`

```java
int x = null;       // invalid
Integer x = null;  // valid
```

A wrapper is a reference type and can represent absence with `null`.

This matters in backend applications because `null` and `0` can have different business meanings.

---

## 5. Integer Cache — Classic Interview Trap

```java
Integer a = 10;
Integer b = 10;

System.out.println(a == b);
```

Common result:

```text
true
```

Autoboxing can reuse cached Integer instances.

Conceptually:

```text
Integer Cache

Integer(10)
   ↑   ↑
   a   b
```

The standard Integer implementation normally caches `-128` to `127`; implementations may cache a wider range.

---

## 6. Wrapper `==` vs `equals()`

```java
Integer a = 200;
Integer b = 200;

a == b        // generally false
a.equals(b)   // true
```

Why:

```text
==        → reference comparison
equals()  → value comparison
```

Interview rule:

> Never use `==` when the intention is wrapper value comparison. Use `equals()` or `Objects.equals()` when null safety is required.

Do not make business logic depend on wrapper caching.

---

## 7. Wrapper + Primitive Comparison

```java
Integer a = 10;
int b = 10;

System.out.println(a == b);
```

Result:

```text
true
```

Because the wrapper is unboxed:

```text
Integer
   ↓
unboxing
   ↓
int
   ↓
int == int
```

This differs from wrapper + wrapper, where `==` compares references.

---

## 8. Null Unboxing — Production Trap

```java
Integer a = null;
int b = a;
```

This **compiles**, but throws `NullPointerException` at runtime.

Conceptually:

```text
Integer a
   ↓
a.intValue()
   ↓
null.intValue()
   ↓
NullPointerException
```

Important interview answer:

> Unboxing is a valid conversion, so compilation succeeds; a null wrapper causes NPE when the runtime tries to unbox it.

---

## 9. Null in Comparisons

```java
Integer count = null;

if (count > 0) {
    ...
}
```

This compiles but throws NPE because `count` must be unboxed for the numeric comparison.

Safer handling depends on business semantics:

```java
if (count != null && count > 0) {
    ...
}
```

or, when zero is genuinely the correct default:

```java
int count = request.getCount() != null
        ? request.getCount()
        : 0;
```

Do not replace null with zero blindly.

---

## 10. Wrapper Arithmetic

```java
Integer a = 10;
Integer b = 20;

Integer c = a + b;
```

Conceptually:

```text
Integer a → unbox → int 10
Integer b → unbox → int 20
                     ↓
                  int 30
                     ↓
                autoboxing
                     ↓
                Integer 30
                     ↓
                     c
```

Mental model:

```text
Integer + Integer
      ↓
unboxing
      ↓
int + int
      ↓
int
      ↓
autoboxing
      ↓
Integer
```

We intentionally did not go deeper into obscure operator rules because they are lower priority for the target interview level.

---

## 11. `valueOf()`

Wrapper boxing is conceptually associated with `valueOf()` methods such as:

```java
Integer.valueOf(10)
```

`valueOf()` can reuse cached instances.

Mental model:

```text
autoboxing
   ↓
valueOf()
   ↓
cache lookup
   ↓
possibly reused wrapper
```

---

## 12. Wrapper Immutability

Wrappers such as `Integer` are immutable.

```java
Integer x = 10;
x = 20;
```

This does not mutate the existing `Integer(10)` object.

Conceptually:

```text
x → Integer(10)

x = 20

x → Integer(20)
```

The reference changes; the original wrapper object does not.

---

## 13. Primitive vs Wrapper — Production Decision

Prefer a primitive when:

- `null` is not meaningful
- primitive arithmetic is all you need
- the API does not require a reference type

Example:

```java
int retryCount;
long timestamp;
boolean enabled;
```

Prefer a wrapper when:

- generics/collections require it
- `null` has meaningful semantics
- an API/framework requires an object/reference type
- nullable values need to be represented

Example:

```java
List<Integer> ids;
Integer optionalCount;
```

---

## 14. Production Pitfalls

### Pitfall 1 — `==` with wrappers

Avoid:

```java
if (a == b) {
    ...
}
```

when `a` and `b` are wrappers and you mean value equality.

Prefer:

```java
Objects.equals(a, b);
```

when either side may be null.

### Pitfall 2 — Accidental unboxing

```java
Integer count = request.getCount();

if (count > 0) {
    ...
}
```

If `count == null`, this can throw NPE.

### Pitfall 3 — Assuming wrapper caching

Do not rely on:

```java
Integer a = 10;
Integer b = 10;

a == b
```

for business logic.

Caching affects reference identity; it is not value comparison.

---

# 15. Interview Challenge Questions — Solve Later

## Q1 — Why Wrappers?

Why is this invalid?

```java
List<int> numbers = new ArrayList<>();
```

Why is this valid?

```java
List<Integer> numbers = new ArrayList<>();
```

Explain from first principles.

## Q2 — Autoboxing

What happens here?

```java
Integer x = 10;
```

Explain the conversion.

## Q3 — Integer Cache

What are the outputs?

```java
Integer a = 10;
Integer b = 10;

System.out.println(a == b);
System.out.println(a.equals(b));
```

Explain.

## Q4 — Cache Boundary

What are the likely outputs?

```java
Integer a = 127;
Integer b = 127;

Integer c = 128;
Integer d = 128;

System.out.println(a == b);
System.out.println(c == d);
```

Explain the role of caching.

## Q5 — Wrapper vs Primitive

What happens here?

```java
Integer a = 10;
int b = 10;

System.out.println(a == b);
```

Does `==` compare references or values? Why?

## Q6 — Null Unboxing

What happens?

```java
Integer a = null;
int b = a;
```

Does it compile? What happens at runtime?

## Q7 — Null in Comparison

What happens?

```java
Integer count = null;

if (count > 0) {
    System.out.println("Positive");
}
```

Why?

## Q8 — Wrapper Arithmetic

Explain the conversion sequence:

```java
Integer a = 10;
Integer b = 20;

Integer c = a + b;
```

## Q9 — Production Scenario

You receive:

```java
Integer transactionCount;
```

Would you change it to:

```java
int transactionCount;
```

What questions would you ask before deciding?

## Q10 — Production Bug

This code occasionally behaves unexpectedly:

```java
Integer statusCode = getStatusCode();

if (statusCode == 200) {
    ...
}
```

What could be wrong? How would you fix it?

## Q11 — Null-Safe Comparison

Which is safer when both values may be null?

```java
a.equals(b)
```

or:

```java
Objects.equals(a, b)
```

Explain.

---

# 16. Final Mental Model

```text
                 WRAPPER CLASS
                       │
             object representation
                       │
        ┌──────────────┼──────────────┐
        ↓              ↓              ↓
    Generics         null          methods
        │              │
        ↓              ↓
 List<Integer>    Integer x=null
        │
        ↓
   autoboxing
        │
        ↓
      Integer

Integer → int
    ↓
unboxing

int → Integer
    ↓
autoboxing

Wrapper + Wrapper with ==
    ↓
reference comparison
    ↓
cache may matter

Wrapper + primitive
    ↓
unboxing
    ↓
primitive comparison

equals()
    ↓
value comparison

null wrapper
    ↓
unboxing
    ↓
NullPointerException

valueOf()
    ↓
cache lookup
    ↓
possibly reused wrapper
```

# 17. Track Progress

```text
09. Composition vs Inheritance       ✅
10. Object Methods
    ├── equals()                    ✅
    ├── hashCode()                  ✅
    └── toString()                  ✅

11. String / String Pool             ✅

12. Wrapper Classes
    ├── Primitive vs Wrapper        ✅
    ├── Why wrappers exist          ✅
    ├── Autoboxing                  ✅
    ├── Unboxing                    ✅
    ├── null / NPE                  ✅
    ├── Integer caching             ✅
    ├── == vs equals()              ✅
    ├── valueOf()                   ✅
    ├── Wrapper immutability        ✅
    ├── Production pitfalls         ✅
    └── Interview challenges        ⏳ solve later

13. Collections                     ← NEXT
14. Generics
15. Exception Handling
16. Java 8+
17. Records / Sealed Classes
18. Modern Java Features
```

# 18. Interview Goal

Be able to naturally explain:

- Why can't generics use primitives?
- What are autoboxing and unboxing?
- Why can `Integer a = 10; Integer b = 10; a == b` be true?
- Why can the same comparison be false for 200?
- Why use `equals()` instead of `==` for wrappers?
- What happens when a null Integer is unboxed?
- Why can `if (count > 0)` throw NPE?
- When would you choose `int` vs `Integer`?

The goal is reasoning, not memorizing cache trivia:

```text
primitive
   ↓
object representation
   ↓
boxing / unboxing
   ↓
caching
   ↓
reference vs value
   ↓
null safety
   ↓
production choice
```
