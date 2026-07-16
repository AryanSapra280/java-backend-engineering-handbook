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