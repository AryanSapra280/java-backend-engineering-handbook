Absolutely. We’ll do this **part by part**, and I’ll go **deep enough for a 4–6 YOE EPAM Java interview**, especially around the places where interviewers stop asking definitions and start asking *“but how does it actually work?”*

For every part, I’ll use this structure:

1. **Core concept**
2. **Interview question**
3. **Strong interview answer**
4. **Follow-up / trap question**
5. **Implementation / code**
6. **Coding question + clean solution**
7. **Production scenario**
8. **Rapid-fire revision at the end**

And importantly, I’ll keep bringing the discussion back to **coding**, because your EPAM round is specifically **1 Java coding + 1 Stream coding**, and they care about **clear code**.

# PART 1 — CORE JAVA FUNDAMENTALS

This part is the foundation for almost everything else.

We won't spend equal time on every bullet. The **heart** is:

> OOP → equals/hashCode → String/immutability → overloading/overriding → wrappers/autoboxing → Comparable/Comparator → copying → Java pass-by-value.

---

# 1. OOP — The interview version

The four major OOP concepts are:

| Concept | Meaning | Java example |
|---|---|---|
| Encapsulation | Bundle data + behavior and control access | `private` fields + methods |
| Abstraction | Expose what, hide implementation details | interface / abstract class |
| Inheritance | Reuse/extend behavior | `extends` |
| Polymorphism | Same interface/reference, different behavior | method overriding |

### Interview question

**Q: What are the four pillars of OOP? Explain with Java examples.**

### Strong answer

> OOP is based on encapsulation, abstraction, inheritance and polymorphism.
>
> **Encapsulation** means keeping an object's state private and exposing controlled operations through methods.
>
> **Abstraction** means exposing essential behavior while hiding implementation details, typically using interfaces or abstract classes.
>
> **Inheritance** allows a class to reuse or specialize behavior from another class.
>
> **Polymorphism** allows the same reference or interface to represent different implementations. In Java, runtime polymorphism is primarily achieved through method overriding.

Example:

```java
interface Payment {
    void pay();
}

class CreditCardPayment implements Payment {

    @Override
    public void pay() {
        System.out.println("Pay using credit card");
    }
}

class UpiPayment implements Payment {

    @Override
    public void pay() {
        System.out.println("Pay using UPI");
    }
}
```

Now:

```java
Payment payment = new UpiPayment();
payment.pay();
```

The reference is `Payment`, but the actual object is `UpiPayment`.

That is **runtime polymorphism**.

---

# 2. Encapsulation vs Abstraction

This is a very common follow-up.

### Q: What's the difference?

**Encapsulation**

> Controls access to the object's internal state.

**Abstraction**

> Hides implementation complexity and exposes only required behavior.

Example:

```java
class BankAccount {

    private double balance;

    public void deposit(double amount) {
        if (amount > 0) {
            balance += amount;
        }
    }

    public double getBalance() {
        return balance;
    }
}
```

`balance` is encapsulated because callers cannot directly modify it.

Abstraction:

```java
interface PaymentService {
    void pay(double amount);
}
```

The caller knows **what** operation exists, but not **how** it is implemented.

---

# 3. `==` vs `equals()`

🔥 **Extremely important because this leads directly into HashMap.**

### Q

```java
String a = new String("Java");
String b = new String("Java");

System.out.println(a == b);
System.out.println(a.equals(b));
```

### Answer

```text
false
true
```

`==` compares references for objects.

`equals()` compares logical equality if the class has overridden it appropriately.

Conceptually:

```text
a ───────► String("Java")
           
b ───────► String("Java")
```

Different objects → `==` false.

Same content → `equals()` true.

---

## But primitive types?

```java
int a = 10;
int b = 10;

System.out.println(a == b);
```

`==` compares primitive values.

---

# 4. The REAL heart: equals + hashCode

This is one of the most important Java interview chains.

### Q

**Why must we override `hashCode()` whenever we override `equals()`?**

### Strong answer

> The `equals()` and `hashCode()` contract says that if two objects are equal according to `equals()`, they must return the same hash code.
>
> Hash-based collections such as `HashMap` and `HashSet` first use the hash code to locate a bucket and then use `equals()` to determine the exact matching key/object.
>
> Therefore, violating the contract can cause logically equal objects to be stored or looked up incorrectly.

Example:

```java
class Employee {

    private int id;
    private String name;

    public Employee(int id, String name) {
        this.id = id;
        this.name = name;
    }

    @Override
    public boolean equals(Object obj) {

        if (this == obj) {
            return true;
        }

        if (!(obj instanceof Employee)) {
            return false;
        }

        Employee other = (Employee) obj;

        return id == other.id &&
               Objects.equals(name, other.name);
    }

    @Override
    public int hashCode() {
        return Objects.hash(id, name);
    }
}
```

Then:

```java
Employee e1 = new Employee(1, "Aryan");
Employee e2 = new Employee(1, "Aryan");

System.out.println(e1.equals(e2)); // true
System.out.println(e1.hashCode() == e2.hashCode()); // true
```

---

# 5. HashMap follow-up

Interviewer:

> Okay. Suppose you override `equals()` but don't override `hashCode()`. What happens?

This is where they test whether you actually understand collections.

Imagine:

```java
Map<Employee, String> map = new HashMap<>();

map.put(e1, "Developer");

System.out.println(map.get(e2));
```

If:

```java
e1.equals(e2) == true
```

but:

```java
e1.hashCode() != e2.hashCode()
```

then the HashMap may calculate different buckets.

Conceptually:

```text
e1
 ↓
hashCode()
 ↓
Bucket 3

e2
 ↓
hashCode()
 ↓
Bucket 9
```

HashMap may never reach `e1` while looking up `e2`.

### Interview sentence

> `equals()` tells us whether two objects are logically equal, while `hashCode()` helps hash-based collections efficiently locate the candidate bucket. Equal objects must therefore have equal hash codes.

This sentence is worth remembering.

---

# 6. Can unequal objects have the same hashCode?

**YES.**

This is called a **hash collision**.

```text
Object A ──hash──► 10
Object B ──hash──► 10
```

but:

```java
A.equals(B) == false
```

HashMap handles collisions by storing multiple entries in the same bucket.

And this naturally leads to:

> linked nodes → treeification → HashMap internals.

That's Part 3.

---

# 7. Overloading vs Overriding

🔥 Very common.

## Overloading

Same method name, different parameter list.

```java
class Calculator {

    int add(int a, int b) {
        return a + b;
    }

    int add(int a, int b, int c) {
        return a + b + c;
    }
}
```

Resolved at **compile time**.

Therefore:

> **Compile-time polymorphism**

---

## Overriding

Child class provides a new implementation of an inherited method.

```java
class Animal {

    void sound() {
        System.out.println("Animal sound");
    }
}

class Dog extends Animal {

    @Override
    void sound() {
        System.out.println("Bark");
    }
}
```

```java
Animal animal = new Dog();
animal.sound();
```

Output:

```text
Bark
```

Resolved at runtime.

> **Runtime polymorphism**

---

# 8. Important overriding trap

### Q

Can a static method be overridden?

**No.**

Static methods belong to the class, not the object.

They can be **hidden**.

```java
class Parent {

    static void show() {
        System.out.println("Parent");
    }
}

class Child extends Parent {

    static void show() {
        System.out.println("Child");
    }
}
```

```java
Parent p = new Child();
p.show();
```

Output:

```text
Parent
```

Because static method resolution uses the **reference/class type**, not runtime object dispatch.

---

# 9. Can private methods be overridden?

No.

A private method is not inherited by the child class in the polymorphic sense.

```java
class Parent {

    private void test() {
        System.out.println("Parent");
    }
}
```

A method with the same name in the child is a separate method.

---

# 10. Can final methods be overridden?

No.

```java
class Parent {

    final void test() {
    }
}
```

Child cannot override it.

Why?

Because `final` prevents further overriding.

---

# 11. `final`, `finally`, `finalize`

Classic interview question.

### `final`

Keyword.

Used with:

```java
final int x = 10;
final class A {}
final void test() {}
```

Meaning depends on usage:

- final variable → cannot be reassigned
- final method → cannot be overridden
- final class → cannot be inherited

---

### `finally`

Block associated with exception handling.

```java
try {
    // risky operation
} catch (Exception e) {
    // handle
} finally {
    // cleanup
}
```

Normally executes regardless of whether exception occurs.

---

### `finalize()`

Historically a GC-related method called before object reclamation, but it has been **deprecated for removal** and should not be used for resource management.

Modern Java uses:

- try-with-resources
- `AutoCloseable`
- explicit cleanup

---

# 12. The nasty `finally` trap

```java
public static int test() {

    try {
        return 10;
    } finally {
        return 20;
    }
}
```

What happens?

```text
20
```

The `finally` return overrides the earlier return.

### Interview advice

Never write:

```java
finally {
    return something;
}
```

It can suppress:

- previous return value
- exceptions

---

# 13. Pass-by-value

🔥 Very commonly misunderstood.

### Q

**Is Java pass-by-value or pass-by-reference?**

Correct answer:

> Java is always pass-by-value.

For objects, Java passes **a copy of the reference value**.

Example:

```java
class Person {
    String name;
}
```

```java
void change(Person p) {
    p.name = "John";
}
```

```java
Person person = new Person();
person.name = "Aryan";

change(person);

System.out.println(person.name);
```

Output:

```text
John
```

Because both references point to the same object.

But:

```java
void change(Person p) {
    p = new Person();
    p.name = "John";
}
```

The caller's reference is unchanged.

### Mental model

```text
caller
person ──────────────► Object A

                 copied reference
                       ↓
method            p ───► Object A
```

If `p` is reassigned:

```text
p ─────────► Object B

person ────► Object A
```

---

# 14. String immutability

🔥 Extremely important.

### Q

Why is `String` immutable?

Several reasons:

### 1. String pool

```java
String a = "Java";
String b = "Java";
```

Both may reference the same pooled object.

If String were mutable, changing `a` could affect `b`.

---

### 2. Security

Strings are frequently used for:

- file paths
- class names
- URLs
- credentials/configuration
- database connection information

Immutability prevents unexpected modification.

---

### 3. Thread safety

Immutable objects can safely be shared between threads.

---

### 4. Hashing

Strings are heavily used as keys in HashMaps.

Their hash value can safely be cached because the contents don't change.

---

# 15. String pool

```java
String a = "Java";
String b = "Java";
```

Typically:

```java
a == b
```

is:

```text
true
```

because both point to the same pooled literal.

But:

```java
String c = new String("Java");

System.out.println(a == c);
```

is:

```text
false
```

because `new String()` creates a new object.

But:

```java
a.equals(c)
```

is:

```text
true
```

---

# 16. StringBuilder vs StringBuffer

### StringBuilder

- mutable
- not synchronized
- generally faster
- preferred for single-threaded string construction

```java
StringBuilder sb = new StringBuilder();

sb.append("Java");
sb.append(" ");
sb.append("Backend");

System.out.println(sb);
```

### StringBuffer

- mutable
- synchronized
- thread-safe
- generally slower due to synchronization

### Interview answer

> StringBuilder is preferred when thread safety is not required. StringBuffer provides synchronized operations for legacy thread-safe usage.

---

# 17. Wrapper classes

Primitive:

```text
int
long
double
boolean
```

Wrapper:

```text
Integer
Long
Double
Boolean
```

Why wrappers?

Because Java collections work with objects:

```java
List<Integer> numbers;
```

not:

```java
List<int> // invalid
```

---

# 18. Autoboxing / Unboxing

```java
Integer x = 10;
```

Autoboxing:

```text
int → Integer
```

Unboxing:

```java
Integer x = 10;
int y = x;
```

```text
Integer → int
```

---

# 19. Wrapper caching trap 🔥

```java
Integer a = 100;
Integer b = 100;

System.out.println(a == b);
```

Usually:

```text
true
```

because Java caches certain small Integer values.

But:

```java
Integer a = 1000;
Integer b = 1000;

System.out.println(a == b);
```

typically:

```text
false
```

Therefore:

> Never use `==` to compare wrapper values when logical equality is intended. Use `equals()`.

---

# 20. Comparable vs Comparator

🔥 Very important for coding.

## Comparable

Defines the object's **natural ordering**.

```java
class Employee implements Comparable<Employee> {

    private int salary;

    @Override
    public int compareTo(Employee other) {
        return Integer.compare(this.salary, other.salary);
    }
}
```

Then:

```java
Collections.sort(employees);
```

---

## Comparator

Defines an **external/custom ordering**.

```java
Comparator<Employee> bySalary =
        Comparator.comparingInt(Employee::getSalary);
```

Then:

```java
employees.sort(bySalary);
```

### Interview answer

> Comparable defines natural ordering inside the class using `compareTo()`, while Comparator defines external/custom ordering and allows multiple sorting strategies.

For example:

```java
employees.sort(Comparator.comparing(Employee::getName));

employees.sort(Comparator.comparingInt(Employee::getSalary));

employees.sort(Comparator.comparing(Employee::getName).reversed());
```

This becomes especially important in **Stream coding**.

---

# 21. Shallow copy vs Deep copy

Suppose:

```java
class Address {
    String city;
}

class Employee {
    String name;
    Address address;
}
```

A shallow copy copies the outer object but shares nested references.

```text
Employee A
   |
   └──► Address A

Employee B
   |
   └──► Address A
```

Changing:

```java
employeeB.address.city
```

also affects `employeeA`.

Deep copy:

```text
Employee A
   |
   └──► Address A

Employee B
   |
   └──► Address B
```

Nested mutable objects are independently copied.

---

# 22. Cloneable

Java provides:

```java
Cloneable
```

as a marker interface.

A common implementation:

```java
class Employee implements Cloneable {

    int id;
    String name;

    @Override
    protected Employee clone() throws CloneNotSupportedException {
        return (Employee) super.clone();
    }
}
```

Important interview point:

> `Object.clone()` performs a field-level copy, which is generally a shallow copy.

For nested mutable objects, explicit deep-copy logic is required.

---

# 🔥 PART 1 CODING — Interview-style problems

Now let's hit the coding heart.

---

## Coding 1 — Remove duplicates while preserving order

### Problem

Given:

```java
List<Integer> numbers =
        Arrays.asList(5, 2, 5, 3, 2, 4, 3);
```

Output:

```text
[5, 2, 3, 4]
```

### Clean solution

```java
List<Integer> result = new ArrayList<>();

Set<Integer> seen = new HashSet<>();

for (Integer number : numbers) {

    if (seen.add(number)) {
        result.add(number);
    }
}
```

### Why does this work?

`HashSet.add()` returns:

```text
true  → element wasn't present
false → element already existed
```

Complexity:

```text
Time:  O(n)
Space: O(n)
```

### Even cleaner if preserving insertion order:

```java
List<Integer> result =
        new ArrayList<>(new LinkedHashSet<>(numbers));
```

For an interview, explain **why `LinkedHashSet`**:

> It maintains uniqueness like HashSet while preserving insertion order.

---

# Coding 2 — Find duplicate numbers

```java
List<Integer> numbers =
        Arrays.asList(1, 2, 3, 2, 4, 5, 1, 3);
```

Expected:

```text
[1, 2, 3]
```

Clean solution:

```java
Set<Integer> seen = new HashSet<>();
Set<Integer> duplicates = new LinkedHashSet<>();

for (Integer number : numbers) {

    if (!seen.add(number)) {
        duplicates.add(number);
    }
}
```

Why `LinkedHashSet` for duplicates?

Because it preserves the order in which duplicates are discovered.

---

# Coding 3 — First non-repeating character

This is **very relevant to EPAM**.

Input:

```text
"swiss"
```

Output:

```text
"w"
```

### Clear implementation

```java
public static Character firstNonRepeating(String input) {

    Map<Character, Integer> frequency = new LinkedHashMap<>();

    for (char ch : input.toCharArray()) {
        frequency.put(ch, frequency.getOrDefault(ch, 0) + 1);
    }

    for (Map.Entry<Character, Integer> entry : frequency.entrySet()) {

        if (entry.getValue() == 1) {
            return entry.getKey();
        }
    }

    return null;
}
```

### Why LinkedHashMap?

Because we need:

1. frequency
2. original insertion order

A normal `HashMap` does not guarantee iteration order.

### Complexity

```text
Time:  O(n)
Space: O(n)
```

This exact problem has appeared in reported EPAM interviews. The Stream version is also important and we'll revisit it heavily in **Part 5**.

---

# Coding 4 — Sort employees

```java
class Employee {

    private int id;
    private String name;
    private int salary;

    // constructor/getters
}
```

Sort by ID ascending:

```java
employees.sort(
        Comparator.comparingInt(Employee::getId)
);
```

Descending:

```java
employees.sort(
        Comparator.comparingInt(Employee::getId)
                  .reversed()
);
```

Salary descending:

```java
employees.sort(
        Comparator.comparingInt(Employee::getSalary)
                  .reversed()
);
```

Name ascending:

```java
employees.sort(
        Comparator.comparing(Employee::getName)
);
```

Name descending:

```java
employees.sort(
        Comparator.comparing(Employee::getName)
                  .reversed()
);
```

### Interview trap

What if salary is `Integer`, not `int`?

Prefer:

```java
Comparator.comparingInt(Employee::getSalary)
```

when getter returns primitive `int`.

For `Integer`:

```java
Comparator.comparing(Employee::getSalary)
```

---

# Coding 5 — Frequency of elements

Input:

```java
[1, 2, 2, 3, 3, 3, 4]
```

Output:

```text
1 -> 1
2 -> 2
3 -> 3
4 -> 1
```

### Standard solution

```java
Map<Integer, Integer> frequency = new HashMap<>();

for (Integer number : numbers) {

    frequency.put(
        number,
        frequency.getOrDefault(number, 0) + 1
    );
}
```

This tiny pattern is **extremely important**.

You'll use the same concept for:

- character frequency
- word frequency
- duplicate detection
- grouping
- anagrams
- Stream `groupingBy`
- caching/counting problems

---

# 23. One important connection

Notice how today's topics connect:

```text
equals/hashCode
      ↓
HashSet
      ↓
HashMap
      ↓
frequency counting
      ↓
duplicate detection
      ↓
LinkedHashMap
      ↓
first non-repeating character
      ↓
Comparator
      ↓
sorting
      ↓
Streams
```

That is exactly how you should mentally organize Core Java rather than memorizing isolated definitions.

---

# PART 1 — Rapid-fire EPAM revision

You should be able to answer these almost instantly:

**Q: Java pass-by-reference?**

> No. Java is always pass-by-value. For objects, the copied value is the reference.

**Q: `==` for objects?**

> Reference comparison.

**Q: `equals()`?**

> Logical equality according to the class implementation.

**Q: Contract between equals and hashCode?**

> Equal objects must have equal hash codes.

**Q: Can unequal objects have same hash code?**

> Yes, that's a collision.

**Q: Overloading?**

> Compile-time polymorphism.

**Q: Overriding?**

> Runtime polymorphism.

**Q: Can static methods be overridden?**

> No, they are hidden.

**Q: Can private methods be overridden?**

> No.

**Q: Can final methods be overridden?**

> No.

**Q: String mutable?**

> No.

**Q: Why immutable?**

> String pool, security, thread safety, and stable hashing.

**Q: StringBuilder vs StringBuffer?**

> Both mutable; StringBuilder isn't synchronized, StringBuffer is synchronized.

**Q: Comparable?**

> Natural ordering via `compareTo()`.

**Q: Comparator?**

> External/custom ordering.

**Q: HashSet internally relies on?**

> Hashing and equality; conceptually backed by a HashMap.

**Q: LinkedHashSet advantage?**

> Maintains insertion order while maintaining uniqueness.

**Q: TreeSet?**

> Sorted set based on natural ordering or Comparator.

**Q: Shallow copy?**

> Copies references to nested objects.

**Q: Deep copy?**

> Independently copies nested mutable objects.

**Q: `Cloneable`?**

> Marker interface indicating cloning is supported; `Object.clone()` performs a field-level copy.

---

## 🎯 What you should actually retain from Part 1

If EPAM starts drilling you, the most important chains are:

### Chain 1

```text
equals()
   ↓
hashCode()
   ↓
HashMap
   ↓
collision
   ↓
HashSet
```

### Chain 2

```text
Overloading
   ↓
compile-time

Overriding
   ↓
runtime
   ↓
dynamic dispatch
```

### Chain 3

```text
String immutable
   ↓
String pool
   ↓
safe sharing
   ↓
stable hashCode
   ↓
good HashMap key
```

### Chain 4

```text
Comparable
   ↓
natural ordering

Comparator
   ↓
custom ordering
   ↓
Collections
   ↓
Streams
```

### Chain 5

```text
HashMap
   ↓
frequency map
   ↓
duplicates
   ↓
first non-repeating character
   ↓
Stream equivalent
```

**Part 2 will be Collections**, and that's where we'll go much deeper: `ArrayList`, `LinkedList`, `Vector`, `Stack`, `Queue`, `Deque`, `PriorityQueue`, all Set/Map variants, iterators, fail-fast behavior, `CopyOnWriteArrayList`, and the coding patterns that lead directly into the **HashMap internals** interview chain.