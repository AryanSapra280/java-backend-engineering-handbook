# Two Pointers - Thinking Framework

> **Goal:** Learn to derive two-pointer solutions instead of memorizing them.

---

# What I Learned

Initially, I used to recognize two-pointer problems only because I had seen them before.

Now I understand that every two-pointer solution works because I can **prove that moving one pointer does not discard the optimal answer**.

That is the essence of the pattern.

---

# The Biggest Question 

Whenever I think a two-pointer solution exists, I ask:

> **Can I safely discard one side?**

If the answer is **YES**, then there is a good chance a two-pointer solution exists.

---

# General Framework

## Step 1

Read the problem carefully.

Ask:

- Is the array sorted?
- Is there a monotonic property?
- Is there any ordering I can exploit?

If yes,

don't ignore it.

The sorted property is usually the biggest hint.

---

## Step 2

Ask:

> What is preventing me from finding the answer?

Examples:

- Sum is too large
- Sum is too small
- Width is shrinking
- Smaller height limits area
- Values are duplicated
- Invalid element exists

Find the bottleneck.

---

## Step 3

Ask:

> Can I mathematically prove that one element can never belong to the optimal answer?

If yes,

discard it forever.

This is the heart of two pointers.

---

# Common Mental Models

## 1. Discard an Impossible Candidate

Question:

> Can this element ever participate in the optimal answer?

If the answer is **No**,

move that pointer.

Examples:

- Two Sum II
- Container With Most Water

---

## 2. Read Pointer + Write Pointer

Question:

> Do I only need to keep valid elements?

If yes,

don't search for bad elements.

Instead,

scan everything and copy only good elements.

Examples:

- Remove Element
- Move Zeroes
- Remove Duplicates

---

## 3. Fill From Free Space

Question:

> Where is the empty space?

If extra space exists at one end,

process from that end.

Example:

Merge Sorted Array

---

## 4. Reverse the Direction

Question:

> Does processing forward overwrite useful data?

If yes,

try processing backwards.

Often this removes shifting and overwriting.

---

# Problem Notes

---

# 88. Merge Sorted Array

## Initial Thought

Merge from the beginning.

Whenever an element from nums2 belongs before nums1,

shift all remaining elements.

Problem:

```
[1,2,3,0,0,0]

Insert 2

↓

[1,2,2,3,0,0]
```

Too much shifting.

---

## Observation

Extra space already exists.

```
[1,2,3,0,0,0]
       ↑ ↑ ↑
```

Why not fill from the end?

---

## Thinking Shift

Instead of placing the smallest element,

place the largest remaining element.

Nothing gets overwritten.

---

## Lesson

Whenever forward traversal causes

- shifting
- overwriting

ask

> Can I process from the opposite direction?

---

# 977. Squares of a Sorted Array

## Initial Thought

Find the first positive number.

Reverse the negative half.

Merge both sorted halves.

This works in O(n).

---

## Better Observation

The largest square always comes from

- first element

or

- last element

because they have the largest absolute values.

No need to split the array.

---

## Thinking Shift

Don't ask

> Where is the first positive?

Ask

> Where can the largest square come from?

---

## Lesson

For sorted arrays,

always inspect both ends first.

Sometimes the answer naturally grows from the outside.

---

# 167. Two Sum II

## Initial Thought

Need two numbers.

Array is sorted.

Can sorting help?

---

## Observation

Suppose

```
left + right > target
```

Even the smallest possible partner with the current right is already too large.

Therefore,

the current right element can never belong to the answer.

Discard it.

---

Similarly,

If

```
left + right < target
```

Even the largest partner with the current left is too small.

Discard left.

---

## Lesson

Don't think

> Move left.

Think

> Can I prove this element is impossible?

---

# 11. Container With Most Water

## Initial Thought

For every index,

find the farthest larger height.

Multiple passes.

Complicated.

---

## Observation

Area is

```
min(height) × width
```

Suppose

```
left height = 2

right height = 8
```

The shorter height limits the area.

Keeping the shorter height while decreasing width can never improve the answer.

Therefore,

discard the shorter side.

---

## Thinking Shift

Exactly like Two Sum.

Instead of

```
Sum too large
```

here

```
Shorter height limits the answer.
```

Discard the bottleneck.

---

## Lesson

The common idea is NOT "two pointers."

The common idea is

> Prove one side can never produce a better answer.

---

# 27. Remove Element

## Initial Thought

Find the first occurrence of val.

Find the next non-val.

Swap.

Repeat.

Works,

but requires both pointers to search.

Lots of edge cases.

---

## Better Observation

I don't care where bad elements are.

I only care where good elements should go.

---

## Thinking Shift

Instead of

```
Find bad element
```

think

```
Read every element.

Write only valid ones.
```

One pointer reads.

One pointer writes.

---

## Lesson

Whenever the problem says

- Remove
- Keep
- Filter
- Delete

Immediately think

```
Read Pointer

↓

Write Pointer
```

---

# Patterns I Have Learned

## Pattern 1

Discard an impossible candidate.

Questions:

- Can this element ever belong to the answer?
- If not, discard it.

Examples:

- Two Sum II
- Container With Most Water

---

## Pattern 2

Read + Write.

Questions:

- Am I filtering elements?
- Do I only care about valid elements?

Examples:

- Remove Element
- Move Zeroes
- Remove Duplicates

---

## Pattern 3

Fill From Empty Space.

Questions:

- Where is the free space?
- Can I process from that side?

Examples:

- Merge Sorted Array

---

## Pattern 4

Reverse Traversal.

Questions:

- Does forward traversal overwrite useful data?
- Can I reverse the direction?

Examples:

- Merge Sorted Array
- Many in-place array problems

---

## Pattern 5

Maximum/Minimum Comes From the Ends.

Questions:

- Does the array remain sorted?
- Can the answer only come from the extremes?

Examples:

- Squares of a Sorted Array

---

# My Two Pointer Checklist

Whenever I see an array problem, I will ask myself:

### □ Is the array sorted?

---

### □ Can I eliminate one side with a proof?

---

### □ Is one side obviously impossible?

---

### □ Is there empty space that changes traversal direction?

---

### □ Am I overwriting useful data?

---

### □ Do I need Read + Write pointers instead?

---

### □ Is the answer determined by the ends of the array?

---

### □ Is one value acting as the bottleneck?

Examples:

- Smaller height
- Larger sum
- Smaller sum
- Invalid element

---

# Biggest Takeaway

I should stop memorizing:

> Move left.

> Move right.

Instead I should ask:

> **Why is it safe to move this pointer?**

If I can mathematically prove that moving a pointer never removes the optimal answer,

then I have understood the algorithm.

That is the real two-pointer pattern.