# Java 8 — Optional

## 1. What Is Optional?

`Optional<T>` is a container that represents:

```text
A value may exist
        OR
A value may be absent
```

Example:

```java
Optional<String> name =
    Optional.of("Aryan");
```

contains:

```text
Aryan
```

An empty Optional:

```java
Optional<String> name =
    Optional.empty();
```

represents:

```text
No value
```

---

# 2. Why Was Optional Introduced?

Before Java 8, a method that might not return a value would commonly return:

```java
null
```

Example:

```java
User user = findUser(userId);
```

If the user doesn't exist:

```java
user == null
```

Then every caller potentially needs:

```java
if (user != null) {
    ...
}
```

Otherwise:

```java
user.getName();
```

could result in:

```text
NullPointerException
```

Optional provides a way to explicitly represent the possibility of absence.

Instead of:

```java
User findUser(Long id);
```

we can have:

```java
Optional<User> findUser(Long id);
```

The method signature now communicates:

> A User may or may not exist.

---

# 3. IMPORTANT DOUBT

## "But if Optional still needs an if-check, what's the advantage?"

This is a very good question.

At first glance:

```java
if (user != null) {
    ...
}
```

and:

```java
if (optional.isPresent()) {
    ...
}
```

look almost identical.

So:

> Why use Optional?

The answer is:

**Optional is NOT simply a replacement for `if`.**

Its main advantages are:

1. It makes possible absence explicit in the API.
2. It allows absence to flow safely through transformations.
3. It allows operations to be composed without repeated null checks.
4. It provides standard ways to handle absence.
5. It works naturally with Stream operations such as `findFirst()` and `findById()`.

---

# 4. Optional Is Not "if Without null"

Don't think:

```text
Optional
   =
if replacement
```

Instead think:

```text
Optional
   =
explicit representation of possible absence
   +
operations for safely handling that absence
```

This distinction is extremely important.

---

# 5. Simple Case — Optional Doesn't Magically Remove the Condition

Suppose:

```java
User user = findUser(userId);

if (user != null) {
    return user.getName();
}

return "Unknown";
```

You could write:

```java
Optional<User> user =
    findUser(userId);

if (user.isPresent()) {
    return user.get().getName();
}

return "Unknown";
```

This doesn't look much better.

In fact, using:

```java
isPresent()
get()
```

like this is often not the best way to use Optional.

So the question is valid:

> "If I am just doing `isPresent()` and `get()`, what did I gain?"

Not much.

---

# 6. Where Optional Becomes Useful

Optional becomes much more useful when we **compose operations**.

Suppose we have:

```text
User
 ↓
Address
 ↓
City
 ↓
City Name
```

And every step may potentially be absent.

Without Optional:

```java
User user = findUser(userId);

if (user != null) {

    Address address = user.getAddress();

    if (address != null) {

        City city = address.getCity();

        if (city != null) {

            String name = city.getName();

            if (name != null) {
                return name;
            }
        }
    }
}

return "Unknown";
```

This becomes deeply nested.

---

# 7. Same Example With Optional

Suppose:

```java
Optional<User> user =
    findUser(userId);
```

We can write:

```java
String cityName =
    user.map(User::getAddress)
        .map(Address::getCity)
        .map(City::getName)
        .orElse("Unknown");
```

This is where Optional becomes powerful.

The pipeline says:

```text
Optional<User>
      ↓
if User exists
      ↓
get Address
      ↓
if Address exists
      ↓
get City
      ↓
if City exists
      ↓
get City Name
      ↓
if anything was absent
      ↓
"Unknown"
```

We don't manually write a separate null check for every level.

---

# 8. Optional's Biggest Concept — Composition

The important mental model is:

```text
Optional<User>
      ↓
map()
      ↓
Optional<Address>
      ↓
map()
      ↓
Optional<City>
      ↓
map()
      ↓
Optional<String>
      ↓
orElse()
      ↓
String
```

Absence automatically flows through the pipeline.

If the User is absent:

```text
Optional.empty()
      ↓
map()
      ↓
Optional.empty()
      ↓
map()
      ↓
Optional.empty()
```

No NullPointerException.

---

# 9. Optional Makes Absence Explicit in APIs

Compare:

```java
User findUser(Long id);
```

with:

```java
Optional<User> findUser(Long id);
```

The first doesn't tell the caller from the type alone whether:

```java
null
```

is possible.

The caller may have to read documentation or implementation.

The second explicitly communicates:

```text
Optional<User>
```

means:

> A User may or may not exist.

This is one of the major design benefits.

---

# 10. Optional and Spring Data

A very common real-world example:

```java
Optional<User> user =
    userRepository.findById(userId);
```

The repository is saying:

> This ID may or may not correspond to a User.

Then the caller can decide how to handle that.

For example:

```java
User user =
    userRepository.findById(userId)
                  .orElseThrow(
                      () -> new UserNotFoundException(userId)
                  );
```

Or:

```java
userRepository.findById(userId)
              .ifPresent(
                  user -> sendNotification(user)
              );
```

The absence handling becomes explicit.

---

# 11. Creating Optional

There are three important ways.

## `Optional.of()`

```java
Optional<String> name =
    Optional.of("Aryan");
```

The value must not be null.

This:

```java
Optional.of(null);
```

throws:

```text
NullPointerException
```

Use `of()` when you know the value must exist.

---

# 12. `Optional.ofNullable()`

Use when the value may be null:

```java
String name = getName();

Optional<String> optionalName =
    Optional.ofNullable(name);
```

If:

```text
name = "Aryan"
```

then:

```text
Optional[Aryan]
```

If:

```text
name = null
```

then:

```text
Optional.empty()
```

Mental model:

```text
of()
    → null is NOT allowed

ofNullable()
    → null becomes Optional.empty()
```

---

# 13. `Optional.empty()`

Explicitly creates an empty Optional:

```java
Optional<String> name =
    Optional.empty();
```

Meaning:

```text
No value exists.
```

---

# 14. Creation Cheat Sheet

```java
Optional.of(value)
```

```text
Value MUST NOT be null.
```

---

```java
Optional.ofNullable(value)
```

```text
Value MAY be null.
null → Optional.empty()
```

---

```java
Optional.empty()
```

```text
No value.
```

---

# 15. `isPresent()`

Checks whether a value exists.

```java
Optional<String> name =
    Optional.of("Aryan");

if (name.isPresent()) {
    System.out.println("Value exists");
}
```

Returns:

```text
true
```

for a non-empty Optional.

For:

```java
Optional.empty()
```

returns:

```text
false
```

---

# 16. `get()` ⚠️

Retrieves the value:

```java
Optional<String> name =
    Optional.of("Aryan");

String value =
    name.get();
```

Result:

```text
Aryan
```

But:

```java
Optional<String> name =
    Optional.empty();

name.get();
```

throws:

```text
NoSuchElementException
```

Therefore:

> Don't blindly call `get()`.

---

# 17. Why `isPresent()` + `get()` Is Not the Main Goal

This:

```java
if (optional.isPresent()) {
    return optional.get();
}
```

works, but it misses much of the benefit of Optional.

Instead, use operations such as:

```text
map()
flatMap()
orElse()
orElseGet()
orElseThrow()
ifPresent()
```

depending on what you are trying to accomplish.

---

# 18. `orElse()` ⭐⭐⭐

Provides a default value when Optional is empty.

```java
Optional<String> name =
    Optional.empty();

String result =
    name.orElse("Unknown");
```

Result:

```text
Unknown
```

If the Optional contains:

```text
Aryan
```

then:

```java
name.orElse("Unknown");
```

returns:

```text
Aryan
```

Mental model:

```text
Value exists?
    ↓ YES
return value

    ↓ NO
return default
```

---

# 19. `orElse()` Important Trap ⭐⭐⭐

Consider:

```java
optional.orElse(createDefault());
```

Many people assume:

> `createDefault()` executes only if Optional is empty.

That is incorrect.

The argument to `orElse()` is evaluated before the method is called.

Therefore:

```java
optional.orElse(createDefault());
```

can execute:

```java
createDefault()
```

even when the Optional already contains a value.

This matters if:

```java
createDefault()
```

is expensive.

---

# 20. `orElseGet()` ⭐⭐⭐

`orElseGet()` accepts a Supplier:

```java
String result =
    optional.orElseGet(
        () -> createDefault()
    );
```

The fallback computation happens only if the Optional is empty.

Mental model:

```text
orElse()
    → default expression is evaluated

orElseGet()
    → default is calculated only when needed
```

---

# 21. `orElse()` vs `orElseGet()`

### `orElse()`

```java
optional.orElse(createDefault());
```

The default expression can execute even when a value exists.

### `orElseGet()`

```java
optional.orElseGet(
    () -> createDefault()
);
```

The fallback is evaluated only when required.

Use `orElseGet()` when the fallback computation is expensive or has side effects that should only happen when needed.

---

# 22. `orElseThrow()`

Use when absence should be treated as an error.

Example:

```java
User user =
    optionalUser.orElseThrow(
        () -> new UserNotFoundException()
    );
```

If value exists:

```text
return User
```

If empty:

```text
throw exception
```

Very common in backend applications.

---

# 23. `ifPresent()`

Perform an action only when a value exists.

```java
Optional<String> name =
    Optional.of("Aryan");

name.ifPresent(
    value -> System.out.println(value)
);
```

Output:

```text
Aryan
```

If the Optional is empty:

```text
nothing happens
```

`ifPresent()` accepts a:

```text
Consumer<T>
```

which connects directly to Lambda and Functional Interfaces.

---

# 24. `map()` ⭐⭐⭐

`map()` transforms the value inside Optional.

Example:

```java
Optional<String> name =
    Optional.of("aryan");

Optional<String> upper =
    name.map(String::toUpperCase);
```

Result:

```text
Optional[ARYAN]
```

Mental model:

```text
Optional<String>
      ↓
     map()
      ↓
transform value
      ↓
Optional<String>
```

---

# 25. Optional `map()` and Empty Values

Suppose:

```java
Optional<String> name =
    Optional.empty();
```

Then:

```java
Optional<Integer> length =
    name.map(String::length);
```

Result:

```text
Optional.empty()
```

The mapping function is not executed.

This gives us:

```text
Optional
   ↓
value exists?
   ↓
YES → transform
NO  → remain empty
```

---

# 26. Why `map()` Is Important

This is where Optional becomes more useful than just:

```java
if (value != null)
```

Suppose:

```java
Optional<User> user;
```

You want the name:

```java
Optional<String> name =
    user.map(User::getName);
```

If the User doesn't exist:

```text
Optional.empty()
```

If the User exists:

```text
Optional["Aryan"]
```

The absence is automatically propagated.

---

# 27. `map()` Example — String Length

```java
Optional<String> name =
    Optional.of("Aryan");

Optional<Integer> length =
    name.map(String::length);
```

Result:

```text
Optional[5]
```

Pipeline:

```text
Optional["Aryan"]
        ↓
String::length
        ↓
Optional[5]
```

---

# 28. `flatMap()` ⭐⭐⭐

`flatMap()` is used when the function you're calling **already returns an Optional**.

Suppose:

```java
Optional<User> user;
```

and:

```java
Optional<Address> getAddress(User user)
```

Notice:

```text
getAddress()
    → Optional<Address>
```

If we use:

```java
user.map(u -> getAddress(u));
```

we would conceptually get:

```text
Optional<Optional<Address>>
```

That's usually not what we want.

Use:

```java
Optional<Address> address =
    user.flatMap(
        u -> getAddress(u)
    );
```

Now:

```text
Optional<Address>
```

---

# 29. `map()` vs `flatMap()`

The most useful rule:

```text
Function returns normal value
        ↓
      map()
```

Example:

```java
user.map(User::getName);
```

because:

```text
getName()
    → String
```

---

```text
Function returns Optional
        ↓
     flatMap()
```

Example:

```java
user.flatMap(User::getAddress);
```

because:

```text
getAddress()
    → Optional<Address>
```

---

# 30. Easy Mental Model

```text
map()

Optional<T>
    ↓
Function<T, R>
    ↓
Optional<R>
```

---

```text
flatMap()

Optional<T>
    ↓
Function<T, Optional<R>>
    ↓
Optional<R>
```

The key difference:

```text
map()
    can conceptually produce
    Optional<Optional<R>>

flatMap()
    flattens that nesting
```

---

# 31. Real Backend Example

Suppose:

```java
Optional<User> user =
    userRepository.findById(userId);
```

We want the email:

```java
Optional<String> email =
    user.map(User::getEmail);
```

Then:

```java
String emailValue =
    user.map(User::getEmail)
        .orElse("unknown@example.com");
```

Pipeline:

```text
find user
   ↓
Optional<User>
   ↓
map(getEmail)
   ↓
Optional<String>
   ↓
orElse(default)
   ↓
String
```

---

# 32. Deep Optional Chain Example

Suppose:

```text
User
 ↓
Address
 ↓
City
 ↓
City Name
```

Without Optional:

```java
User user = findUser(id);

if (user != null) {

    Address address = user.getAddress();

    if (address != null) {

        City city = address.getCity();

        if (city != null) {

            String name = city.getName();

            if (name != null) {
                return name;
            }
        }
    }
}

return "Unknown";
```

With Optional:

```java
String cityName =
    user.map(User::getAddress)
        .map(Address::getCity)
        .map(City::getName)
        .orElse("Unknown");
```

This is the key practical advantage.

---

# 33. Optional and Streams

Several Stream terminal operations naturally return Optional.

Examples:

```java
findFirst()
findAny()
min()
max()
reduce()   // overload without identity
```

Why?

Because there may be no result.

Example:

```java
Optional<Integer> first =
    numbers.stream()
           .findFirst();
```

For an empty Stream:

```text
Optional.empty()
```

Instead of:

```text
null
```

---

# 34. Example — `findFirst()`

```java
List<Integer> numbers =
    List.of(10, 20, 30);

Optional<Integer> first =
    numbers.stream()
           .findFirst();
```

Result:

```text
Optional[10]
```

If:

```java
List<Integer> numbers =
    List.of();
```

result:

```text
Optional.empty()
```

This is why `findFirst()` doesn't simply return `Integer`.

---

# 35. Optional Is NOT a Replacement for Every `null`

Don't use Optional everywhere just because it exists.

The most important use case to remember:

```text
Method result may legitimately be absent
        ↓
Optional<T>
```

For example:

```java
Optional<User> findUserById(Long id);
```

is a meaningful API.

Optional is especially useful for return values.

Don't think:

```text
"Every String should become Optional<String>"
```

That's not the purpose.

---

# 36. Optional Fields and Parameters

For interview purposes, remember:

Optional was primarily designed around expressing potentially absent **return values**.

Avoid blindly doing:

```java
class User {

    private Optional<String> name;
    private Optional<Integer> age;
}
```

or using Optional everywhere as a universal replacement for null.

The exact best practice can depend on framework/API design, but the important interview principle is:

> Use Optional where absence is meaningful and should be explicitly communicated.

---

# 37. Can Optional Itself Be `null`?

Technically:

```java
Optional<String> name = null;
```

is possible.

But this defeats the purpose.

Then:

```java
name.orElse("Unknown");
```

throws:

```text
NullPointerException
```

Instead:

```java
Optional.empty()
```

should represent absence.

Therefore:

```text
BAD:

Optional<T> = null


GOOD:

Optional.empty()
```

---

# 38. Optional vs `null`

## Traditional approach

```java
User user = findUser(id);

if (user != null) {
    return user.getName();
}

return "Unknown";
```

## Optional approach

```java
Optional<User> user =
    findUser(id);

String name =
    user.map(User::getName)
        .orElse("Unknown");
```

The advantage is not simply fewer lines.

The important benefits are:

```text
1. Absence is explicit in the type.

2. Transformations can be composed.

3. Null checks don't have to be repeated at every step.

4. Standard operations exist for absence handling.

5. APIs become clearer.
```

---

# 39. The Key Answer to the Original Doubt

Question:

> "If Optional still brings the same if-check, what is the advantage?"

Answer:

> Optional does not eliminate conditional logic. It makes the possibility of absence explicit and gives us composable operations for handling that absence.

Simple:

```java
if (user != null) {
    return user.getName();
}
```

may not be dramatically improved by Optional.

But complex:

```text
User
 ↓
Address
 ↓
City
 ↓
Name
```

can become:

```java
user.map(User::getAddress)
    .map(Address::getCity)
    .map(City::getName)
    .orElse("Unknown");
```

instead of deeply nested null checks.

So:

```text
Optional
    ≠
replacement for if

Optional
    =
explicit absence
+
safe/composable operations
```

---

# 40. Optional Cheat Sheet

```text
CREATION
========

Optional.of(value)
    → value must not be null

Optional.ofNullable(value)
    → value may be null

Optional.empty()
    → no value


CHECKING
========

isPresent()
    → value exists?

isEmpty()
    → empty? (Java 11+)


RETRIEVING
==========

get()
    → retrieve value
    → throws if empty

orElse(default)
    → value or fallback

orElseGet(supplier)
    → value or lazily calculated fallback

orElseThrow(...)
    → value or exception


ACTION
======

ifPresent(consumer)
    → execute action if value exists


TRANSFORMATION
==============

map(function)
    → transform existing value

flatMap(function)
    → transform when function already returns Optional
```

---

# 41. `orElse()` vs `orElseGet()`

```text
orElse()
    ↓
fallback expression is evaluated eagerly

orElseGet()
    ↓
fallback is calculated lazily
```

Example:

```java
optional.orElse(createDefault());
```

versus:

```java
optional.orElseGet(
    () -> createDefault()
);
```

Use `orElseGet()` when the fallback computation is expensive or should happen only when needed.

---

# 42. `map()` vs `flatMap()`

```text
map()
    ↓
function returns normal value

Example:

Optional<User>
    ↓
map(User::getName)
    ↓
Optional<String>
```

---

```text
flatMap()
    ↓
function already returns Optional

Example:

Optional<User>
    ↓
flatMap(User::getAddress)
    ↓
Optional<Address>
```

Easy rule:

```text
returns R
    → map()

returns Optional<R>
    → flatMap()
```

---

# 43. Interview Questions

## Basic

1. What is Optional?
2. Why was Optional introduced?
3. What problem does Optional solve?
4. Difference between `of()` and `ofNullable()`?
5. What does `Optional.empty()` represent?
6. What happens if `get()` is called on an empty Optional?

## Important

7. Why shouldn't you blindly use `get()`?
8. `orElse()` vs `orElseGet()`?
9. Why can `orElse()` execute an expensive fallback unnecessarily?
10. What does `orElseThrow()` do?
11. What does `ifPresent()` do?
12. What does `map()` do?
13. What happens when `map()` is called on an empty Optional?
14. Difference between `map()` and `flatMap()`?
15. Why can `map()` conceptually result in nested Optional?
16. Why does `flatMap()` avoid nested Optional?
17. Why do `findFirst()` and `findAny()` return Optional?
18. Should Optional be used everywhere instead of null?
19. Can an Optional itself be null?
20. What is the main advantage of Optional if we still need conditional logic?

---

# 44. Coding Practice

## Q1 — Basic

```java
Optional<String> name =
    Optional.of("Aryan");
```

Print the name only if it exists.

Use:

```java
ifPresent()
```

---

## Q2 — Basic

```java
Optional<String> name =
    Optional.empty();
```

Return:

```text
Unknown
```

using:

```java
orElse()
```

---

## Q3 — Medium

Given:

```java
Optional<String> name =
    Optional.of("aryan");
```

Convert it to:

```text
Optional[ARYAN]
```

using:

```java
map()
```

---

## Q4 — Medium

Given:

```java
Optional<String> name;
```

Return the String length if present, otherwise:

```text
0
```

Expected approach:

```java
name.map(String::length)
    .orElse(0);
```

---

## Q5 — Interview ⭐

Explain why:

```java
optional.orElse(createDefault());
```

can execute:

```java
createDefault()
```

even when the Optional contains a value.

Then explain why:

```java
optional.orElseGet(
    () -> createDefault()
);
```

behaves differently.

---

## Q6 — Interview ⭐⭐⭐

Given:

```java
Optional<User> user;
```

and:

```java
Address getAddress(User user)
```

return the user's city name safely without nested null checks.

Try using:

```text
map()
```

multiple times.

---

## Q7 — Interview ⭐⭐⭐

Given:

```java
Optional<User> user;
```

and:

```java
Optional<Address> getAddress(User user)
```

Explain why:

```java
user.map(u -> getAddress(u))
```

can produce:

```text
Optional<Optional<Address>>
```

and why:

```java
user.flatMap(u -> getAddress(u))
```

doesn't.

---

# 45. MUFG Priority

## 🔴 Must Know

```text
Optional
of()
ofNullable()
empty()
isPresent()
get() and why it is risky
orElse()
orElseGet()
orElseThrow()
ifPresent()
map()
flatMap()
```

## 🟡 Know Conceptually

```text
isEmpty()
ifPresentOrElse()
```

These are newer Java additions and will be covered with Modern Java.

---

# 46. Java 8 Progress

```text
Lambda                         ✅
Functional Interfaces          ✅
Method References              ✅

Streams                        ✅
    ├── Fundamentals           ✅
    ├── Sources                ✅
    ├── Intermediate            ✅
    ├── Terminal                ✅
    └── Collectors              ✅

Optional                       ✅

        ↓

Java 8 Remaining Features
        ↓
Java 8 COMPLETE
        ↓
Multithreading / Concurrency
```

## Key takeaway

Remember this sentence:

> **Optional is not about eliminating `if`; it is about making absence explicit and allowing that absence to be handled safely and compositionally.**