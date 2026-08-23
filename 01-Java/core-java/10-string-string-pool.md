# 11 — String / String Pool

> Learning note from our discussion. Captures concepts, reasoning, corrections, interview traps, production considerations, and challenge questions. Challenge answers are intentionally omitted.

## 1. String Fundamentals

`String` is an immutable sequence of characters.

```java
String s = "hello";
s.concat(" world");
System.out.println(s); // hello
```

`concat()` produces a new String; it does not modify the existing String. Reassigning the result makes `s` refer to the new String.

## 2. Why String Is Immutable

Key reasons derived in discussion:

- **Safe String Pool sharing:** pooled Strings can be shared because nobody can mutate the shared value.
- **Security:** validated String values cannot be changed behind another component's back.
- **Thread safety:** the state of a String itself cannot be mutated, so it is safe to share.
- **HashMap/HashSet keys:** equality-relevant content cannot change after insertion.
- **Stable hashCode:** immutable content means the hash value does not need to change and can be cached.

## 3. `==` vs `equals()`

For objects:

```text
==        → reference identity
equals()  → logical/content equality
```

Example:

```java
String s1 = "hello";
String s2 = new String("hello");

s1 == s2        // false
s1.equals(s2)   // true
```

Use `equals()` for String content comparison.

## 4. String Pool

String literals participate in the String Pool.

```java
String s1 = "hello";
String s2 = "hello";
```

Conceptually both references can point to the same pooled object:

```text
        "hello"
        ↑     ↑
       s1    s2
```

Therefore:

```java
s1 == s2 // true
```

## 5. `new String()`

```java
String s1 = "hello";
String s2 = new String("hello");
```

Conceptually:

```text
String Pool                 Heap

"hello" ←──── s1

"hello" ←──── s2
```

`new String("hello")` creates a separate String object.

Therefore:

```java
s1 == s2        // false
s1.equals(s2)   // true
```

## 6. Multiple `new String()` Objects

```java
String s1 = new String("hello");
String s2 = new String("hello");
```

Each `new` creates a separate String object:

```java
s1 == s2        // false
s1.equals(s2)   // true
```

## 7. Compile-Time Concatenation

```java
String s1 = "hello";
String s2 = "hel" + "lo";
```

Both operands are compile-time String literals, so the compiler can fold:

```text
"hel" + "lo"
      ↓
"hello"
```

For interview reasoning:

```java
s1 == s2        // true
s1.equals(s2)   // true
```

The key semantic point is compile-time constant folding to `"hello"`.

## 8. Runtime Concatenation

```java
String part = "lo";

String s1 = "hello";
String s2 = "hel" + part;
```

Here concatenation is performed at runtime.

Generally:

```java
s1 == s2        // false
s1.equals(s2)   // true
```

Conceptually:

```text
"hel" + part
      ↓
runtime concatenation
      ↓
new resulting String
```

Do not claim runtime concatenation literally always executes `new String(...)`. Modern Java may use `invokedynamic` / `StringConcatFactory`.

Important:

> Runtime concatenation does not automatically intern the result.

## 9. `final` Compile-Time Constants

```java
final String part = "lo";

String s1 = "hello";
String s2 = "hel" + part;
```

Because `part` is a final compile-time constant initialized with a constant expression, the compiler can fold the concatenation.

For interview reasoning:

```java
s1 == s2 // true
```

## 10. `intern()`

`intern()` returns the canonical pooled representation.

```java
String s1 = "hello";
String s2 = new String("hello");

s2 = s2.intern();

s1 == s2 // true
```

If an equivalent pooled String already exists, `intern()` returns that canonical reference. Otherwise the value can be canonicalized into the pool.

Interning can help deliberate canonicalization and reuse, but do not blindly intern every String; excessive/inappropriate interning can create memory pressure and performance concerns.

## 11. StringBuilder

Repeated String concatenation can be expensive because String is immutable:

```java
String result = "";

for (int i = 0; i < 10000; i++) {
    result = result + i;
}
```

Conceptually:

```text
"" → "0" → "01" → "012" → ...
```

This can involve repeated allocations and copying.

`StringBuilder` provides mutable construction:

```java
StringBuilder sb = new StringBuilder();

for (int i = 0; i < 10000; i++) {
    sb.append(i);
}

String result = sb.toString();
```

Conceptually:

```text
StringBuilder
      ↓
mutable internal buffer
      ↓
append repeatedly
      ↓
toString()
      ↓
immutable String
```

Do not say StringBuilder uses no memory management. Its internal buffer can grow and be reallocated/copied.

Strong interview answer:

> "`String` is immutable, so repeated concatenation creates new String results and can repeatedly copy existing content. `StringBuilder` maintains a mutable internal buffer and appends into it, reducing intermediate String creation. We convert to the final immutable String only when needed."

## 12. StringBuilder Capacity

```java
StringBuilder sb = new StringBuilder(100);
```

means:

```text
capacity = 100
length   = 0
```

Capacity is not current content length.

```java
sb.length();
sb.capacity();
```

When capacity is insufficient, StringBuilder automatically grows its internal buffer. It does not throw an exception merely because current capacity is exceeded.

Interview-safe wording:

> "StringBuilder grows its internal storage according to its implementation's growth policy and copies existing contents as necessary."

Do not treat a specific growth factor as a Java language guarantee.

## 13. Initial Capacity

If you know the approximate output size:

```java
StringBuilder sb = new StringBuilder(10000);
```

can reduce repeated buffer growth, reallocations, and copying.

Do not massively over-allocate either; use a reasonable estimate.

## 14. `StringBuilder(int)` vs `StringBuilder(String)`

```java
new StringBuilder(100);
```

selects the `int` constructor:

```text
capacity = 100
length = 0
```

But:

```java
new StringBuilder("100");
```

takes a String and initializes the builder with the characters `"100"`.

Therefore:

```text
length = 3
```

The argument type determines overload selection.

## 15. StringBuilder vs StringBuffer

### StringBuilder

```text
mutable
not synchronized
generally preferred for thread-confined use
```

### StringBuffer

```text
mutable
synchronized methods
designed for synchronized access
```

The fact that an application is concurrent does not automatically mean StringBuffer is required.

If every request/thread has its own builder:

```text
Request 1 → StringBuilder 1
Request 2 → StringBuilder 2
Request 3 → StringBuilder 3
```

StringBuilder is appropriate.

Consider StringBuffer when the same mutable character sequence is genuinely shared across threads and synchronized operations are required.

Production principle:

> Prefer thread confinement and avoiding shared mutable state where practical.

## 16. `StringBuilder.toString()`

```java
String result = builder.toString();
```

produces an immutable String result.

It is **not automatically placed in the String Pool**.

Example:

```java
String s1 = "hello world";

StringBuilder b = new StringBuilder();
b.append("hello world");

String s2 = b.toString();
```

Do not expect:

```java
s1 == s2
```

to be true merely because contents match.

Generally:

```java
s1.equals(s2) // true
s1 == s2      // false
```

If canonical pooling is explicitly desired:

```java
s2 = s2.intern();
```

## 17. `new` and Heap — Interview Mental Model

For normal interview reasoning:

```java
new String("hello");
new StringBuilder();
new User();
```

represent object creation associated with heap allocation.

The JVM can optimize allocations in some cases, so avoid treating `new → physically allocated heap object` as an absolute implementation guarantee.

For interview mental modeling:

```text
new → object → heap
```

is appropriate.

## 18. Garbage Collection and Strings

An object becomes eligible for GC when it is no longer reachable from GC roots.

Example:

```java
String s1 = new String("hello");
String s2 = "hello";

s1 = null;
```

The explicitly created heap String can become eligible for GC if no other references point to that particular object.

The pooled literal is a separate object and remains reachable through `s2`.

Do not say that a String is immediately garbage collected merely because one local reference disappears.

## 19. Production Mental Model

```text
String literal
    ↓
String Pool

new String(...)
    ↓
separate String object

runtime concatenation
    ↓
String result
    ↓
not automatically interned

intern()
    ↓
canonical pooled reference

StringBuilder
    ↓
mutable construction buffer

builder.toString()
    ↓
immutable String result
    ↓
not automatically interned
```

## 20. Common Interview Mistakes

1. "`==` compares String values."  
   **Incorrect:** `==` compares references.

2. "Every String literal creates a new object."  
   **Incorrect:** literals participate in pooling/reuse.

3. "Runtime concatenation puts the result in the String Pool."  
   **Incorrect:** it is not automatically interned.

4. "`new String("hello")` is the same as the literal `"hello"`."  
   **Incorrect:** it creates a separate String object.

5. "StringBuilder uses no extra memory."  
   **Incorrect:** its buffer can grow/reallocate.

6. "Multithreading means StringBuffer."  
   **Incorrect:** determine whether the same mutable buffer is shared.

7. "StringBuilder always doubles capacity."  
   Avoid treating a specific growth factor as a language guarantee.

8. "`builder.toString()` returns a pooled String."  
   **Incorrect:** not automatically interned.

---

# 21. Interview Challenge Questions — Solve Later

## Q1 — Basic String Pool

```java
String a = "Java";
String b = "Java";
String c = new String("Java");

System.out.println(a == b);
System.out.println(a == c);
System.out.println(a.equals(c));
```

Give all three outputs and explain why.

## Q2 — Compile-Time Concatenation

```java
String a = "hello";
String b = "hel" + "lo";

System.out.println(a == b);
```

Why?

## Q3 — Runtime Concatenation

```java
String part = "lo";

String a = "hello";
String b = "hel" + part;

System.out.println(a == b);
System.out.println(a.equals(b));
```

Explain when the concatenation occurs.

## Q4 — `final` Constant

```java
final String part = "lo";

String a = "hello";
String b = "hel" + part;

System.out.println(a == b);
```

Why does `final` change the result?

## Q5 — `intern()`

```java
String a = "hello";
String b = new String("hello");

System.out.println(a == b);

b = b.intern();

System.out.println(a == b);
```

Explain both outputs.

## Q6 — Builder and Pool

```java
String a = "hello";

StringBuilder builder = new StringBuilder();
builder.append("hel");
builder.append("lo");

String b = builder.toString();

System.out.println(a == b);
System.out.println(a.equals(b));
```

Why?

## Q7 — Builder Capacity

```java
StringBuilder sb = new StringBuilder(100);

System.out.println(sb.length());
System.out.println(sb.capacity());
```

What are the outputs?

## Q8 — Constructor Overloading

What is the difference between:

```java
new StringBuilder(100);
```

and:

```java
new StringBuilder("100");
```

Explain which overloaded constructor is selected.

## Q9 — Performance

Why can this be expensive?

```java
String result = "";

for (int i = 0; i < 10000; i++) {
    result = result + i;
}
```

What would you use instead and why?

## Q10 — StringBuilder vs StringBuffer

A Spring Boot application receives 1,000 concurrent requests.

Would you automatically use `StringBuffer`?

Explain what information you need before choosing.

## Q11 — Production Scenario

You are processing millions of records and constructing a large JSON-like message for each record.

Would you use:

```text
String
StringBuilder
StringBuffer
```

How would you decide?

Consider:

- mutability
- repeated appends
- thread confinement
- memory
- performance
- final output

## Q12 — GC / Reachability

```java
String s1 = new String("hello");
String s2 = "hello";

s1 = null;
```

Which object can become eligible for GC?

Explain the difference between the heap-created String and the pooled literal.

---

# 22. Final Mental Model

```text
                 STRING
                   │
          ┌────────┴────────┐
          ↓                 ↓
      immutable          String Pool
          │                 │
          │          safe literal sharing
          │
          ├── == → reference
          │
          └── equals() → content

          ├── compile-time concat
          │       ↓
          │   constant folding
          │
          ├── runtime concat
          │       ↓
          │   new String result
          │
          └── intern()
                  ↓
          canonical pool reference


             STRINGBUILDER
                  │
                  ↓
              mutable
                  │
              append()
                  │
             internal buffer
                  │
             toString()
                  ↓
            immutable String


             STRINGBUFFER
                  │
                  ↓
          mutable + synchronized
```

# 23. Track Progress

```text
09. Composition vs Inheritance       ✅
10. Object Methods
    ├── equals()                    ✅
    ├── hashCode()                  ✅
    └── toString()                  ✅

11. String / String Pool
    ├── Immutability                ✅
    ├── String Pool                 ✅
    ├── == vs equals()              ✅
    ├── new String()                ✅
    ├── Concatenation               ✅
    ├── intern()                    ✅
    ├── StringBuilder               ✅
    ├── StringBuffer                ✅
    └── Interview challenges        ⏳ solve later

12. Wrapper Classes                 ← NEXT
13. Collections
14. Generics
15. Exception Handling
16. Java 8+
17. Records / Sealed Classes
18. Modern Java Features
```
