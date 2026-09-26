For revision, the important distinctions here are **immutability vs pooling vs identity** for `String`, and **mutable buffers, capacity, synchronization, and amortized growth** for `StringBuilder`/`StringBuffer`. The trickiest interview questions are usually `new String()`, `intern()`, compile-time vs runtime concatenation, and why repeated `+` in a loop is expensive.

# G. Strings — Language-Level Fundamentals

### 106. Why is `String` a special class in Java?

`String` is a normal final class (`java.lang.String`), but the Java language and JVM give strings special treatment.

For example:

**1. String literals are directly supported by the language**

```java
String s = "Java";
```

You don't normally need:

```java
String s = new String(...);
```

**2. String literals can be pooled/interned.**

```java
String a = "Java";
String b = "Java";

System.out.println(a == b); // true
```

**3. `+` operator is supported for String concatenation.**

```java
String s = "Hello " + "World";
```

**4. Strings are immutable.**

**5. String constants are represented specially in class files and participate in compile-time constant expressions.**

So `String` has special language-level support that ordinary user-defined classes don't have.

---

### 107. Why is String immutable?

Once a `String` object is created, its value cannot be changed.

```java
String s = "Java";

s.concat(" 21");

System.out.println(s); // Java
```

`concat()` didn't modify `"Java"`; it produced another String.

Immutability provides important benefits:

* String pool safety.
* Thread safety.
* Security.
* Stable `hashCode()`.
* Efficient hash caching.
* Safe use as `HashMap` keys.
* Easier sharing between different parts of an application.

If pooled Strings were mutable:

```java
String a = "Java";
String b = "Java";
```

and modifying `a` also changed the pooled object used by `b`, the result would be disastrous.

---

### 108. What is the String pool?

The **String pool** is the JVM's pool of interned Strings used to avoid unnecessary duplicate String objects.

For example:

```java
String a = "Java";
String b = "Java";
```

Typically both refer to the same interned String:

```text
a ───┐
     ├──► "Java"
b ───┘
```

Therefore:

```java
a == b       // true
a.equals(b) // true
```

The String pool is managed by the JVM. In HotSpot, interned Strings have been stored in heap-managed memory for many Java releases; don't use the outdated rule that the String pool necessarily lives in PermGen.

---

### 109. What happens when you write `String s = "Java";`?

```java
String s = "Java";
```

`"Java"` is a String literal.

The JVM uses the interned representation of that literal.

Conceptually:

```text
String Pool

┌────────────┐
│   "Java"   │ ◄──── s
└────────────┘
```

If the appropriate interned `"Java"` already exists, the existing canonical String is reused.

That's why:

```java
String s1 = "Java";
String s2 = "Java";

System.out.println(s1 == s2); // true
```

for the same interned literal.

---

### 110. What happens when you write `String s = new String("Java");`?

```java
String s = new String("Java");
```

There are two relevant things:

```text
String pool                  Heap object

"Java"                 new String("Java")
   │                           │
   │                           │
   └───────────────────────────┘
                               ▲
                               s
```

`"Java"` refers to the interned literal.

`new String(...)` explicitly creates a **new String object with a distinct identity**.

Therefore:

```java
String a = "Java";
String b = new String("Java");

System.out.println(a == b);      // false
System.out.println(a.equals(b)); // true
```

---

### 111. How many String objects can potentially be involved in `new String("Java")`?

Potentially **two relevant String objects**:

```java
String s = new String("Java");
```

1. The interned literal `"Java"`.
2. The new String object explicitly created by `new`.

If `"Java"` is already interned, then executing this expression only needs to create the explicit new String object.

So interview answer:

> Potentially two String objects are involved, but the number newly created at that point depends on whether the literal has already been loaded/interned.

---

### 112. What is String interning?

**Interning is the process of maintaining a canonical String representation so equal interned Strings can share the same object.**

For example:

```java
String a = "Java";
String b = "Java";
```

Both literals refer to the canonical interned String.

```text
a ──┐
    ├──► "Java"
b ──┘
```

Interning can reduce duplicate String instances and enables canonical identity for interned values.

---

### 113. What does `intern()` do?

`intern()` returns the **canonical representation** of a String from the String pool.

```java
String a = new String("Java");
String b = a.intern();
String c = "Java";

System.out.println(b == c); // true
```

Conceptually:

```text
a ─────► new String("Java")

b ──┐
    ├────► pooled "Java"
c ──┘
```

`intern()` does not mean "modify this String into a pooled String"; it returns the canonical reference.

---

### 114. What happens when `intern()` is called on a dynamically constructed String?

Example:

```java
String s = new StringBuilder()
        .append("Ja")
        .append("va")
        .toString();

String interned = s.intern();
```

The JVM checks the intern pool for an equal String.

If an equal interned String already exists, its canonical reference is returned.

Otherwise, that String value becomes represented by a canonical interned reference according to the JVM's interning semantics.

The important interview point is:

```java
String interned = s.intern();
```

returns the canonical pooled reference; **don't assume `interned == s` in every example** without analyzing whether that value had already been interned and the relevant runtime behavior.

---

### 115. What is the difference between compile-time and runtime String concatenation?

If all operands are compile-time constants:

```java
String s = "Ja" + "va";
```

the compiler can perform **constant folding**:

```java
String s = "Java";
```

But:

```java
String a = "Ja";
String b = "va";

String s = a + b;
```

where `a` and `b` are ordinary variables, requires concatenation at runtime.

Modern Java compilers/runtime may implement runtime concatenation using mechanisms such as `invokedynamic` rather than literally generating `StringBuilder` code in every case.

---

### 116. What happens with `String s = "Ja" + "va";`?

Because both operands are compile-time constant String literals:

```java
String s = "Ja" + "va";
```

the compiler folds them into:

```java
String s = "Java";
```

Therefore:

```java
String a = "Java";
String b = "Ja" + "va";

System.out.println(a == b); // true
```

Both refer to the same interned literal.

---

### 117. What happens with variables?

```java
String a = "Ja";
String b = "va";
String s = a + b;
```

Since `a` and `b` are ordinary variables, this is normally **runtime concatenation**.

So:

```java
String x = "Java";

System.out.println(x == s);      // false
System.out.println(x.equals(s)); // true
```

`x` and `s` have equal contents but generally different identities.

However, if they were compile-time constants:

```java
final String a = "Ja";
final String b = "va";

String s = a + b;
```

then the compiler can treat the expression as a constant expression and fold it.

---

### 118. Predict String comparisons involving literals, `new String()` and `intern()`.

Classic interview example:

```java
String s1 = "Java";
String s2 = "Java";
String s3 = new String("Java");
String s4 = s3.intern();

System.out.println(s1 == s2);
System.out.println(s1 == s3);
System.out.println(s1.equals(s3));
System.out.println(s1 == s4);
```

Output:

```text
true
false
true
true
```

Why?

```text
             String Pool
          ┌──────────────┐
s1 ──────►│    "Java"    │◄──── s2
s4 ──────►│              │
          └──────────────┘

          Heap
          ┌──────────────┐
s3 ──────►│    "Java"    │
          └──────────────┘
```

Remember:

```text
==       → identity/reference comparison
equals() → content comparison for String
```

---

### 119. Why can two Strings with the same characters have different identities?

Because two separate String objects can contain equal character sequences.

```java
String a = new String("Java");
String b = new String("Java");
```

Then:

```java
a == b       // false
a.equals(b) // true
```

Conceptually:

```text
a ───► Object #1 → "Java"
b ───► Object #2 → "Java"
```

Same **value/state**, different **identity**.

---

### 120. How does String immutability help with security?

Strings are commonly used for security-sensitive values such as:

* File paths
* Class names
* URLs
* Configuration keys
* Network endpoints

Suppose a method validates a String:

```java
validatePath(path);
openFile(path);
```

Because the String itself is immutable, another piece of code cannot mutate that same String object between validation and use.

Immutability therefore makes shared String values predictable and prevents mutation-based attacks on the String object's contents.

For highly sensitive secrets such as passwords, however, `char[]` is sometimes preferred because it can be explicitly overwritten after use, while a String cannot.

---

### 121. How does String immutability help with caching?

Because the contents never change, properties derived from those contents remain valid.

A major example is `hashCode()`.

```java
String key = "employee123";
map.put(key, employee);
```

A String's hash doesn't suddenly change because its contents cannot change.

This makes Strings excellent `HashMap` keys.

It also allows implementations to cache computed information such as hash values safely.

---

### 122. How does String immutability help with thread safety?

Since a String's contents cannot change after creation, multiple threads can safely share the same String value without synchronization for mutation of that String.

```java
String value = "Java";
```

Thread 1 and Thread 2 can both read `value`.

Neither can modify the `"Java"` object itself.

So String objects are inherently safe to share with respect to their immutable state.

---

# H. StringBuilder & StringBuffer

### 123. What is StringBuilder?

`StringBuilder` is a **mutable sequence of characters** designed for efficiently building and modifying Strings.

```java
StringBuilder sb = new StringBuilder();

sb.append("Hello");
sb.append(" ");
sb.append("World");

String result = sb.toString();
```

Unlike `String`, modifications operate on the builder rather than creating a new String for every change.

---

### 124. What is StringBuffer?

`StringBuffer` is also a **mutable character sequence**, similar to `StringBuilder`.

```java
StringBuffer sb = new StringBuffer();

sb.append("Hello");
sb.append(" World");
```

The major difference is that `StringBuffer`'s relevant operations are synchronized, making it suitable for certain shared multithreaded usage.

---

### 125. StringBuilder vs StringBuffer?

| StringBuilder                           | StringBuffer                                                  |
| --------------------------------------- | ------------------------------------------------------------- |
| Mutable                                 | Mutable                                                       |
| Not synchronized                        | Methods synchronized                                          |
| Not inherently thread-safe              | Provides synchronized operations                              |
| Generally faster                        | Usually slower due to synchronization                         |
| Preferred for normal local construction | Useful when shared mutable buffer synchronization is required |

Most application code uses `StringBuilder`.

---

### 126. Why is StringBuilder generally faster?

Because `StringBuilder` doesn't perform the synchronization that `StringBuffer` performs for many operations.

`StringBuffer` methods such as `append()` are synchronized.

That synchronization can introduce overhead.

Therefore, for code such as:

```java
void createMessage() {
    StringBuilder sb = new StringBuilder();
}
```

where the builder is local to one thread, synchronization provides no benefit.

---

### 127. Is StringBuilder thread-safe?

**No.**

`StringBuilder` is not designed for concurrent mutation by multiple threads without external synchronization.

```java
StringBuilder builder = new StringBuilder();
```

If multiple threads mutate the same builder concurrently, you need your own synchronization or a different design.

---

### 128. Is StringBuffer thread-safe?

Its individual relevant operations are **synchronized**, so it provides thread-safe operation-level mutation.

For example:

```java
buffer.append("A");
```

is synchronized.

However, this doesn't automatically make an entire sequence of multiple operations atomic.

For example:

```java
if (buffer.length() > 0) {
    buffer.deleteCharAt(0);
}
```

Another thread could intervene between operations unless the larger compound action is synchronized appropriately.

So:

> `StringBuffer` provides synchronized methods, but compound operations may still require external synchronization.

---

### 129. How does StringBuilder store its characters internally?

Conceptually, `StringBuilder` maintains a **mutable internal buffer** plus information about how much of that buffer is currently used.

Historically, implementations used structures such as `char[]`. Modern OpenJDK implementations use more optimized internal representations inherited through `AbstractStringBuilder`, and the exact representation is an implementation detail that has changed across Java versions.

For interviews, say:

> "`StringBuilder` maintains a resizable internal character/byte buffer rather than creating a new String object on every append."

Avoid insisting that modern Java must always use `char[]`.

---

### 130. What happens when StringBuilder capacity is exceeded?

The internal buffer needs to grow.

For example:

```java
StringBuilder sb = new StringBuilder(4);

sb.append("Java");
sb.append("Spring");
```

Once the existing capacity cannot hold the additional content, `StringBuilder` allocates a larger internal storage area and copies the existing contents into it.

This doesn't happen on every append — only when more capacity is required.

---

### 131. How does StringBuilder grow its internal buffer?

Conceptually:

```text
Old buffer
[ J a v a ]
capacity = 4

append more characters
        ↓
Need more space
        ↓
Allocate larger buffer
        ↓
Copy existing data
        ↓
Continue appending
```

A commonly cited historical growth rule is roughly:

```text
new capacity ≈ old capacity * 2 + 2
```

but the exact growth strategy is an **implementation detail** and shouldn't be treated as a permanent Java language guarantee.

For interviews, the important point is:

> The buffer grows geometrically rather than increasing by exactly one character each time, which makes repeated appends efficient.

---

### 132. What is the difference between StringBuilder's capacity and length?

**Length** = number of characters currently stored.

**Capacity** = amount that can currently be stored before internal expansion is needed.

Example:

```java
StringBuilder sb = new StringBuilder(100);

sb.append("Java");

System.out.println(sb.length());   // 4
System.out.println(sb.capacity()); // at least 100 here
```

So:

```text
capacity
┌─────────────────────────────────┐
│ J │ a │ v │ a │                 │
└─────────────────────────────────┘
  ← length = 4 →
```

---

### 133. What is the time complexity of repeated `append()` operations?

A normal `append()` is typically **amortized O(1)** per appended primitive/small fixed-size item, excluding the cost proportional to the amount of content being appended.

Occasionally the buffer must grow:

```text
allocate bigger buffer
+
copy existing contents
```

which costs O(n).

But because growth happens geometrically, repeated single-character appends are **amortized O(1)** each.

Therefore, constructing `n` characters using repeated append operations is generally:

```text
O(n)
```

overall.

---

### 134. Why is String concatenation inside a loop potentially expensive?

Consider:

```java
String result = "";

for (int i = 0; i < n; i++) {
    result = result + i;
}
```

Strings are immutable.

Each iteration conceptually produces a new resulting String containing the previous contents plus new content.

As `result` becomes larger, increasingly large amounts of existing content may need to be copied.

Conceptually:

```text
""
 ↓
"1"
 ↓
"12"
 ↓
"123"
 ↓
"1234"
...
```

Repeated copying can lead toward **O(n²)** character-copying behavior for growing concatenations.

Use:

```java
StringBuilder result = new StringBuilder();

for (int i = 0; i < n; i++) {
    result.append(i);
}

String finalResult = result.toString();
```

which is typically much more efficient.

---

### 135. How would you efficiently construct a million-character String?

Use `StringBuilder`, ideally with an estimated initial capacity if you know the approximate final size.

```java
StringBuilder sb = new StringBuilder(1_000_000);

for (int i = 0; i < 1_000_000; i++) {
    sb.append('a');
}

String result = sb.toString();
```

Pre-sizing reduces the number of internal buffer expansions and copies.

So if the approximate output size is known:

```java
new StringBuilder(expectedSize)
```

is a useful optimization.

---

### 136. When would StringBuffer still be appropriate?

`StringBuffer` can be appropriate when:

* The same mutable character buffer genuinely needs to be shared across multiple threads.
* Multiple threads perform operations on it.
* You want its built-in synchronized operations.
* You're maintaining legacy code/API that already uses `StringBuffer`.

Example:

```java
StringBuffer buffer = new StringBuffer();
```

However, in modern backend applications, it's often preferable to avoid sharing mutable builders between threads entirely.

If the builder is local:

```java
public String createResponse() {
    StringBuilder sb = new StringBuilder();
    // ...
    return sb.toString();
}
```

use **`StringBuilder`**.

### Key revision points

```text
String
├── Immutable
├── String pool / interning
├── == → identity
├── equals() → content
└── intern() → canonical pooled reference

"Ja" + "va"
    ↓
Compile-time constant folding
    ↓
"Java"

variableA + variableB
    ↓
Runtime concatenation
```

And:

```text
StringBuilder
├── Mutable
├── Not synchronized
├── Generally preferred
├── Resizable internal buffer
└── append → amortized efficient

StringBuffer
├── Mutable
├── Synchronized operations
└── Useful when shared-buffer synchronization is genuinely needed
```
