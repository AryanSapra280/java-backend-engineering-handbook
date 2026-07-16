# Java Notes: Collections, Arrays & Useful Patterns

> A collection of Java concepts, interview notes, and common pitfalls that I learned while solving DSA problems.

---

# 1. `computeIfAbsent()`

## Purpose

`computeIfAbsent()` is used when you want to create a value **only if the key does not already exist**.

Instead of writing:

```java
if (!map.containsKey(key)) {
    map.put(key, new ArrayList<>());
}
map.get(key).add(value);
```

You can simply write:

```java
map.computeIfAbsent(key, k -> new ArrayList<>()).add(value);
```

---

## Method Signature

```java
V computeIfAbsent(K key, Function<? super K, ? extends V> mappingFunction)
```

For:

```java
Map<String, List<String>> map = new HashMap<>();
```

- `K` → `String`
- `V` → `List<String>`

---

## What does `k` mean?

```java
map.computeIfAbsent("A", k -> new ArrayList<>());
```

Here,

```text
k = "A"
```

It is simply the key passed to the method.

You usually don't need it, but Java requires the lambda to accept it.

Example where `k` is actually useful:

```java
Map<String, String> greetings = new HashMap<>();

greetings.computeIfAbsent("Aryan", k -> "Hello " + k);

System.out.println(greetings);
```

Output:

```
{Aryan=Hello Aryan}
```

---

## Internal Working

```java
map.computeIfAbsent(key, k -> new ArrayList<>()).add(value);
```

Internally behaves like:

```java
if (!map.containsKey(key)) {
    map.put(key, new ArrayList<>());
}

map.get(key).add(value);
```

No explicit `if-else` is required.

---

## Common Mistake

❌ Wrong

```java
map.computeIfAbsent(key, k -> (new ArrayList<>()).add(value));
```

Reason:

```java
(new ArrayList<>()).add(value)
```

returns

```java
boolean
```

whereas `computeIfAbsent()` expects

```java
List<String>
```

---

## Correct Usage

```java
map.computeIfAbsent(key, k -> new ArrayList<>()).add(value);
```

---

# 2. Arrays.fill()

## Unnecessary Usage

```java
int[] count = new int[26];
Arrays.fill(count, 0);
```

This is unnecessary.

Reason:

Every newly created primitive array is already initialized.

| Type | Default Value |
|------|---------------|
| int | 0 |
| long | 0 |
| short | 0 |
| byte | 0 |
| double | 0.0 |
| float | 0.0 |
| char | '\u0000' |
| boolean | false |
| Object | null |

So simply write:

```java
int[] count = new int[26];
```

---

## When should `Arrays.fill()` be used?

When filling with something other than the default value.

Example:

```java
Arrays.fill(arr, -1);

Arrays.fill(arr, Integer.MAX_VALUE);

Arrays.fill(visited, true);
```

---

# 3. Iterating Over a Map

## Iterate over Keys

```java
for (String key : map.keySet()) {

}
```

---

## Iterate over Values

```java
for (List<String> value : map.values()) {

}
```

---

## Iterate over Entries (Recommended)

```java
for (Map.Entry<String, List<String>> entry : map.entrySet()) {

    String key = entry.getKey();

    List<String> value = entry.getValue();
}
```

---

# 4. keySet() vs values() vs entrySet()

## keySet()

Use when only keys are needed.

```java
for (String key : map.keySet()) {

}
```

---

## values()

Use when only values are needed.

```java
for (List<String> list : map.values()) {

}
```

---

## entrySet()

Use when both key and value are required.

```java
for (Map.Entry<String, List<String>> entry : map.entrySet()) {

}
```

It is usually more efficient than:

```java
for (String key : map.keySet()) {
    map.get(key);
}
```

because it avoids an extra lookup.

---

# 5. StringBuilder

Instead of

```java
String s = "";

for (...) {
    s += word;
}
```

Prefer

```java
StringBuilder sb = new StringBuilder();

for (...) {
    sb.append(word);
}
```

Reason:

- Faster
- Avoids creating unnecessary String objects
- Preferred in loops

---

# 6. Character Arithmetic

```java
'a' + 1
```

returns

```java
98
```

because Java promotes `char` to `int`.

If you actually need the character:

```java
(char) ('a' + 1)
```

returns

```java
'b'
```

---

# 7. Frequency Array Pattern

One of the most common DSA patterns.

```java
int[] freq = new int[26];

for (char c : word.toCharArray()) {
    freq[c - 'a']++;
}
```

Used in:

- Group Anagrams
- Valid Anagram
- Sliding Window
- Character Counting
- Hashing Strings

Time Complexity:

```
O(K)
```

where

```
K = length of string
```

---

# 8. Group Anagrams Complexity

Let

```
N = number of strings

K = average length of a string
```

Per string:

- Count frequency → O(K)
- Build frequency signature → O(26)
- HashMap insertion → O(1)

Overall:

```
Time = O(N × K)

Space = O(N)
```

Sorting approach:

```
Time = O(N × K log K)
```

Frequency-count approach is asymptotically faster.

---

# 9. Useful HashMap Methods

## put()

```java
map.put(key, value);
```

---

## get()

```java
map.get(key);
```

Returns

- value
- `null` if key is absent

---

## getOrDefault()

```java
map.getOrDefault(key, defaultValue);
```

Example:

```java
int count = map.getOrDefault(ch, 0);
```

Useful for frequency counting.

---

## containsKey()

```java
if (map.containsKey(key)) {

}
```

---

## remove()

```java
map.remove(key);
```

---

## computeIfAbsent()

```java
map.computeIfAbsent(key, k -> new ArrayList<>());
```

---

## putIfAbsent()

Adds only if key is missing.

```java
map.putIfAbsent(key, value);
```

---

# 10. Common Java Interview Tips

✅ Prefer `computeIfAbsent()` when building:

```java
Map<K, List<V>>
```

---

✅ Don't write:

```java
Arrays.fill(arr, 0);
```

after creating a new primitive array.

---

✅ Prefer `entrySet()` when both key and value are required.

---

✅ Prefer `StringBuilder` inside loops.

---

✅ Remember:

```java
'a' + i
```

returns an `int`.

Use

```java
(char) ('a' + i)
```

when a character is required.

---

✅ Learn the default values of arrays.

---

✅ Analyze both:

- Time Complexity
- Space Complexity

for every DSA solution.

---

# Quick Revision

### Create list if absent

```java
map.computeIfAbsent(key, k -> new ArrayList<>()).add(value);
```

---

### Frequency Array

```java
int[] freq = new int[26];
```

No need for:

```java
Arrays.fill(freq, 0);
```

---

### Iterate Map

```java
for (Map.Entry<K, V> entry : map.entrySet()) {

}
```

---

### StringBuilder

```java
StringBuilder sb = new StringBuilder();

sb.append(value);
```

---

### Character

```java
(char) ('a' + i)
```

---

### Complexity

Frequency Counting

```
O(NK)
```

Sorting

```
O(NK log K)
```