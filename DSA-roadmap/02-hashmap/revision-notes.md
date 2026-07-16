# HashMap Keys in Java

## Fundamental Rule

A `HashMap` can use **any Object** as its key.

```java
HashMap<K, V>
```

The real question is **not**:

> Can this object be used as a key?

The real question is:

> Will this object behave correctly as a key?

---

# How HashMap Works Internally

When inserting a key:

```java
map.put(key, value);
```

HashMap performs two steps.

## Step 1

Compute

```java
key.hashCode()
```

This determines the bucket.

```
Key
 ↓
hashCode()
 ↓
Bucket
```

---

## Step 2

Inside that bucket,

HashMap compares

```java
key.equals(existingKey)
```

If

```java
equals() == true
```

then the key already exists.

Otherwise,

a new entry is inserted.

---

# Requirements for a Good Key

A good HashMap key should satisfy:

- Correct `hashCode()`
- Correct `equals()`
- Should not change after insertion (immutable)

---

# Best HashMap Keys

## String

```java
HashMap<String, Integer>
```

✔ Immutable

✔ Overrides equals()

✔ Overrides hashCode()

Recommended.

---

## Primitive Wrapper Classes

```java
Integer
Long
Character
Double
Boolean
```

All are immutable.

Recommended.

---

## Enum

Excellent choice.

```java
HashMap<Day, Integer>
```

---

## List

```java
HashMap<List<Integer>, String>
```

Works because List overrides

```java
equals()

hashCode()
```

Example

```java
List<Integer> a = List.of(1,2,3);
List<Integer> b = List.of(1,2,3);

System.out.println(a.equals(b));
```

Output

```
true
```

---

## Set

```java
HashMap<Set<Integer>, String>
```

Also works.

Set compares based on contents.

---

## Custom Class

```java
class Employee{
    int id;
    String name;
}
```

Can be used as key.

BUT

Must override

```java
equals()

hashCode()
```

Otherwise,

Java compares memory addresses.

---

# Arrays

Arrays compile as keys.

```java
HashMap<int[], String>
```

BUT

They should generally NOT be used.

Example

```java
int[] a = {1,2};
int[] b = {1,2};

System.out.println(a.equals(b));
```

Output

```
false
```

Reason

Arrays inherit

```java
Object.equals()
Object.hashCode()
```

which compare object identity.

Therefore

```
[1,2]

!=

[1,2]
```

if they are different objects.

---

# Mutable Keys

Example

```java
List<Integer> key = new ArrayList<>();

key.add(1);

map.put(key, "Hello");
```

Later

```java
key.add(2);
```

Now

```java
map.get(key)
```

may fail.

Reason

The hashCode has changed after insertion.

HashMap stored the key using the old hash.

Never modify a key after inserting it.

---

# Group Anagrams Insight

Frequency Array

```
[1,0,0,2,...]
```

Cannot be used directly as key because

```java
int[]
```

uses reference equality.

Possible solutions:

- Convert to String

```
#1#0#0#2...
```

- Convert to immutable List<Integer>

- Create a custom class and override equals() & hashCode()

---

# HashMap Key Checklist

Before using any object as a key, ask:

□ Does it override equals()?

□ Does it override hashCode()?

□ Is it immutable?

If all answers are YES,

it is a good HashMap key.

---

# Commonly Used Keys

| Key Type | Recommended |
|----------|-------------|
| String | ✅ |
| Integer | ✅ |
| Long | ✅ |
| Enum | ✅ |
| List<Integer> | ✅ (do not modify) |
| Set<Integer> | ✅ (do not modify) |
| Custom Class | ✅ (override equals() & hashCode()) |
| int[] | ❌ |
| Integer[] | ❌ |
| Mutable ArrayList | ⚠ Only if never modified |

---

# Interview Takeaway

A HashMap accepts any object as a key.

A good HashMap key should:

- be immutable
- implement equals()
- implement hashCode()

This guarantees that equal objects map to the same bucket and can be retrieved correctly.

---

# Mental Model

```
Object
   │
   ▼
hashCode()
   │
   ▼
Bucket
   │
   ▼
equals()
   │
   ▼
Same Key?
```

Remember:

> HashMap never compares hash codes alone.
>
> It first uses `hashCode()` to locate the bucket and then uses `equals()` to verify whether two keys are actually equal.

-------------------------------------------------------------------------------------------------------
# Canonical Representation

## Mental Model

When a problem asks me to:

- Group equivalent objects
- Identify similar objects
- Compare many objects
- Detect duplicates based on some property

I should stop and ask:

> **Can I transform every equivalent object into the SAME representation?**

If YES,

I no longer need to compare every pair.

I only need to compare the representation.

---

# Problem

LeetCode 49 - Group Anagrams

---

## Step 1 - Brute Force

Compare every string with every other string.

```
eat
tea
tan
ate
nat
bat
```

```
eat <-> tea

eat <-> tan

eat <-> ate

...
```

Time Complexity

```
O(n² × k)
```

where

- n = number of strings
- k = average length of a string

Observation:

The bottleneck is comparing every pair.

---

## Step 2 - Better Observation

Do I really need to compare strings?

No.

I only care whether they contain exactly the same characters with the same frequency.

Example

```
eat

↓

a = 1
e = 1
t = 1
```

```
tea

↓

a = 1
e = 1
t = 1
```

The frequency information is identical.

So instead of comparing strings,

compare their identity.

---

## Step 3 - Canonical Representation

Every anagram should generate the same identity.

Example

```
eat

↓

1a1e1t
```

```
tea

↓

1a1e1t
```

```
ate

↓

1a1e1t
```

Now every anagram has exactly the same representation.

---

## Step 4 - HashMap

Now the problem becomes simple.

```
Key

↓

Canonical Representation
```

```
Value

↓

List<String>
```

Example

```
1a1e1t

↓

["eat","tea","ate"]
```

```
1a1n1t

↓

["tan","nat"]
```

No comparisons are required.

Each string computes its identity exactly once.

---

# Time Complexity

Building frequency

```
O(k)
```

Building key

```
O(26)
```

HashMap insertion

```
O(1)
```

Overall

```
O(n × k)
```

Space

```
O(n × k)
```

---

# Thinking Process

My first instinct was

```
Compare strings
```

↓

Realized comparison is expensive.

↓

Asked

> Can I compare frequencies instead?

↓

Realized comparison still requires O(n²).

↓

Asked

> Why compare at all?

↓

Can every equivalent string become the same object?

↓

Created a canonical representation.

↓

HashMap grouped automatically.

---

# Mental Checklist

Whenever I see a problem involving grouping or comparison, I ask:

□ Am I comparing every pair?

□ Can I avoid comparison completely?

□ Can I transform every equivalent object into one common identity?

□ Can that identity become the HashMap key?

---

# Problems Using This Mental Model

- Group Anagrams
- Valid Anagram
- Group Shifted Strings
- Isomorphic Strings (partially)
- Word Pattern (partially)

---

# One-Line Summary

> Don't compare objects. Transform equivalent objects into the same representation, then let the HashMap do the grouping.