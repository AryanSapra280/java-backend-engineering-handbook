# Two Pointers

## Objective

The Two Pointers pattern is used to solve problems by maintaining two indices that move through a data structure in a controlled manner. Instead of repeatedly scanning the same elements, both pointers work together to reduce unnecessary computations, often improving a brute-force solution from **O(n²)** to **O(n)** while using **O(1)** extra space.

---

# Pattern Recognition

Consider the Two Pointers pattern when you notice one or more of the following clues:

* ✅ The input array is **sorted**.
* ✅ The problem asks for a **pair** of elements.
* ✅ You need to compare elements from **both ends**.
* ✅ The problem requires **in-place modification**.
* ✅ The solution should use **constant extra space**.
* ✅ You need to **remove duplicates**.
* ✅ You need to **merge two sorted arrays**.
* ✅ You need to **reverse** an array or string.
* ✅ You need to **partition** elements.
* ✅ You need to maximize or minimize a value based on two positions.

> **Important:** The presence of a sorted array does not automatically mean Two Pointers is the answer, but it is one of the strongest hints.

---

# Mental Decision Tree

```text
Is the data sorted?
        │
        ├── Yes
        │      │
        │      ├── Need a pair?
        │      │         → Two Pointers
        │      │
        │      ├── Remove duplicates?
        │      │         → Slow/Fast Pointers
        │      │
        │      ├── Merge?
        │      │         → Two Pointers
        │      │
        │      └── Compare left and right?
        │                → Opposite-End Pointers
        │
        └── No
               │
               ├── Can sorting help?
               │
               └── Continue evaluating other patterns
```

---

# Types of Two Pointer Problems

## 1. Opposite-End Pointers

### Idea

Two pointers start from opposite ends of the array and move toward each other.

```text
L ---------------------------- R
```

### Recognition Clues

* Sorted array
* Pair finding
* Compare left and right values
* Reverse operations
* Maximize or minimize distance

### Generic Template

```java
int left = 0;
int right = n - 1;

while (left < right) {

    if (condition) {
        left++;
    } else {
        right--;
    }
}
```

### Common Problems

* Two Sum II
* Container With Most Water
* Boats to Save People
* Reverse String
* Valid Palindrome
* Trapping Rain Water

---

## 2. Same-Direction (Slow & Fast Pointers)

### Idea

Both pointers move in the same direction.

The **fast pointer** explores the input.

The **slow pointer** builds the answer.

```text
Slow →

Fast →
```

### Recognition Clues

* Remove duplicates
* Remove elements
* Move valid elements forward
* In-place modification
* Stable ordering

### Generic Template

```java
int slow = 0;

for (int fast = 0; fast < nums.length; fast++) {

    if (isValid(nums[fast])) {

        nums[slow] = nums[fast];

        slow++;
    }
}
```

### Common Problems

* Remove Duplicates from Sorted Array
* Remove Element
* Move Zeroes

---

## 3. Sliding Window (Extension of Two Pointers)

### Idea

Both pointers move forward while maintaining a valid window.

```text
L →

R →
```

The window grows and shrinks depending on the problem constraints.

### Recognition Clues

* Longest substring
* Shortest subarray
* Continuous sequence
* At most K
* At least K
* Exactly K

### Common Problems

* Longest Substring Without Repeating Characters
* Minimum Window Substring
* Character Replacement
* Fruit Into Baskets

> Sliding Window is generally considered a specialized form of the Two Pointers technique and is often studied separately because of its large number of variations.

---

## 4. Partitioning

### Idea

Pointers divide elements into different regions of the array.

```text
Processed | Unprocessed | Unknown
```

### Recognition Clues

* Partition array
* Rearrange elements
* Move positives, negatives or zeros
* Pivot-based partitioning

### Common Problems

* Move Zeroes
* Dutch National Flag
* Quick Sort Partition

---

## 5. Multiple Pointers

### Idea

One pointer remains fixed while two additional pointers search for the answer.

```text
i

L --------------------- R
```

### Recognition Clues

* Three Sum
* Four Sum
* Multiple element combinations

### Generic Template

```java
for (int i = 0; i < n; i++) {

    int left = i + 1;
    int right = n - 1;

    while (left < right) {

        // Logic
    }
}
```

### Common Problems

* 3Sum
* 4Sum

---

# Why Two Pointers Works

Instead of checking every possible pair (**O(n²)**), Two Pointers uses information about the input (such as sorted order or pointer movement) to eliminate impossible candidates after each comparison.

This allows the algorithm to explore the search space efficiently without revisiting elements unnecessarily.

---

# Time Complexity

| Type              | Time          | Space                                                |
| ----------------- | ------------- | ---------------------------------------------------- |
| Opposite-End      | O(n)          | O(1)                                                 |
| Slow & Fast       | O(n)          | O(1)                                                 |
| Sliding Window    | O(n)          | O(1) *(excluding auxiliary data structures if used)* |
| Partitioning      | O(n)          | O(1)                                                 |
| Multiple Pointers | Usually O(n²) | O(1)                                                 |

---

# Common Interview Mistakes

### Mistake 1

Using a HashMap immediately without noticing the array is sorted.

---

### Mistake 2

Moving the wrong pointer after each comparison.

---

### Mistake 3

Ignoring duplicate values in problems like 3Sum and 4Sum.

---

### Mistake 4

Off-by-one errors (`left < right` vs `left <= right`).

---

### Mistake 5

Choosing Two Pointers when the problem actually requires Sliding Window or Prefix Sum.

---

# Pattern Recognition Checklist

Before writing code, ask yourself:

* Is the array sorted?
* Do I need a pair of elements?
* Can I solve this in-place?
* Can two moving indices eliminate unnecessary work?
* Am I comparing opposite ends?
* Am I building the result while scanning?
* Am I maintaining a valid window?
* Am I partitioning elements?

If the answer to one or more of these questions is **Yes**, Two Pointers is a strong candidate.

---

# Problem Categories

## Opposite-End

* Merge Sorted Array
* Squares of a Sorted Array
* Two Sum II
* Container With Most Water
* Boats to Save People
* Reverse String
* Valid Palindrome
* Trapping Rain Water

---

## Same-Direction

* Remove Duplicates from Sorted Array
* Remove Element
* Move Zeroes

---

## Sliding Window

* Longest Substring Without Repeating Characters
* Character Replacement
* Minimum Window Substring
* Fruit Into Baskets

---

## Multiple Pointers

* 3Sum
* 4Sum

---

# Mastery Checklist

A pattern is considered mastered only when you can:

* Recognize it within **30 seconds**.
* Explain why Two Pointers is appropriate.
* Reject incorrect patterns confidently.
* Write the template from memory.
* Solve unseen variations without hints.
* Explain the algorithm and trade-offs clearly in an interview.
