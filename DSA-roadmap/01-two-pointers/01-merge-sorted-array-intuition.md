# 88. Merge Sorted Array

**LeetCode:** 88  
**Pattern:** Two Pointers (Reverse Traversal)  
**Difficulty:** Easy

---

# Problem Summary

Given two sorted arrays:

- `nums1` of size `m + n`
- `nums2` of size `n`

The first `m` elements of `nums1` are valid, and the last `n` positions are empty (filled with `0`).

Merge `nums2` into `nums1` **in-place**.

---

# My Initial Thought

My first instinct was to start comparing elements from the beginning.

```
nums1 = [1,2,3,0,0,0]
nums2 = [2,5,6]

Compare

1 vs 2
```

Whenever an element from `nums2` had to be inserted before an element in `nums1`, I had to shift elements to the right.

Example:

```
Before

[1,2,3,0,0,0]

Need to insert 2

↓

[1,2,2,3,0,0]
```

---

# Why This Approach Fails

Starting from the front causes two major problems:

1. Existing elements in `nums1` are overwritten before they are processed.
2. To avoid overwriting, elements must be shifted repeatedly.

Repeated shifting increases the complexity to **O(m × n)**.

---

# Key Observation

The extra space already exists.

```
nums1

[1,2,3,0,0,0]
       ↑ ↑ ↑
```

The empty positions are **at the end**, not at the beginning.

Instead of filling from the front,

**fill from the back.**

---

# Thinking Shift

Instead of asking:

> Which is the smallest element?

Ask:

> Which is the largest remaining element?

The largest element naturally belongs at the last available position.

Since that position is empty, nothing gets overwritten.

---

# Example Walkthrough

```
nums1 = [1,2,3,0,0,0]
nums2 = [2,5,6]
```

Compare

```
3 vs 6
```

Place 6.

```
[1,2,3,0,0,6]
```

Compare

```
3 vs 5
```

Place 5.

```
[1,2,3,0,5,6]
```

Compare

```
3 vs 2
```

Place 3.

```
[1,2,3,3,5,6]
```

Continue until one array finishes.

If `nums2` still has elements, copy them.

If `nums1` still has elements, do nothing because they are already in the correct position.

---

# Mental Trigger

Whenever a problem contains:

- Merge in-place
- Extra space already provided
- Sorted arrays
- O(1) extra space requirement

Immediately ask yourself:

> **Can I process from the opposite direction?**

---

# Pattern Recognition

Forward traversal often causes:

- Overwriting
- Shifting
- Extra work

Reverse traversal often avoids all three.

Whenever forward traversal becomes complicated,

**try reversing the direction.**

---

# Algorithm

Maintain three pointers.

```
i = last valid element of nums1
j = last element of nums2
k = last position of nums1
```

While both arrays have elements:

```
if nums1[i] > nums2[j]
    nums1[k] = nums1[i]
    i--
else
    nums1[k] = nums2[j]
    j--

k--
```

Finally,

copy any remaining elements from `nums2`.

---

# Complexity

| Operation | Complexity |
|-----------|------------|
| Time | O(m + n) |
| Space | O(1) |

---

# Mistake to Remember

❌ I focused on where the smallest element should go.

✅ I should have focused on **where free space already exists**.

The empty space determines the traversal direction.

---

# General Interview Heuristic

When solving in-place array problems, always ask:

### 1. Where is the unused space?

- Beginning?
- Middle?
- End?

Start filling from that side.

---

### 2. Am I overwriting data that I still need?

If yes,

change the traversal direction.

---

### 3. Is shifting elements making my algorithm expensive?

If yes,

look for a way to avoid shifting entirely.

---

### 4. Can processing from the opposite end simplify the problem?

Many in-place array problems become much simpler when solved backwards.

---

# Pattern Learned

**Reverse Two Pointers**

This problem teaches an important interview principle:

> **When forward traversal causes overwriting or shifting, try traversing from the opposite direction.**

The goal is not to memorize this solution.

The goal is to recognize this pattern in future problems.