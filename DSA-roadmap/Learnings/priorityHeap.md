# Java PriorityQueue – Complete Guide

## Table of Contents

1. What is PriorityQueue?
2. Internal Data Structure
3. Heap Property
4. Time Complexities
5. Constructors
6. Min Heap
7. Max Heap
8. Custom Comparators
9. PriorityQueue with Custom Objects
10. Common Operations
11. Building from a Collection (Heapify)
12. Iteration
13. Internal Representation
14. Comparator Best Practices
15. Common Interview Patterns
16. Pitfalls
17. FAQs
18. Cheat Sheet

---

# 1. What is PriorityQueue?

`PriorityQueue<E>` is Java's implementation of a **Binary Heap**.

Unlike a normal Queue (FIFO), a PriorityQueue removes elements according to **priority**, not insertion order.

By default:

- Smallest element has the highest priority.
- Java implements a **Min Heap**.

```java
PriorityQueue<Integer> pq = new PriorityQueue<>();
```

Example:

```
Insert:
10
5
20
2

Removal Order:
2
5
10
20
```

---

# 2. Internal Data Structure

PriorityQueue is backed by a **Binary Heap**.

A Binary Heap is a:

- Complete Binary Tree
- Stored inside an array

For index `i`

```
Parent = (i - 1) / 2

Left Child = 2*i + 1

Right Child = 2*i + 2
```

Example:

```
        2
      /   \
     5     8
    / \
   10 20
```

Array representation:

```
[2,5,8,10,20]
```

Notice that it is **not sorted**.

---

# 3. Heap Property

## Min Heap

Every parent is smaller than its children.

```
Parent <= Children
```

Example

```
      2
    /   \
   5     8
```

---

## Max Heap

Every parent is larger than its children.

```
Parent >= Children
```

Example

```
      20
     /  \
    10   5
```

---

# 4. Time Complexities

| Operation | Complexity |
|-----------|------------|
| offer() | O(log n) |
| add() | O(log n) |
| poll() | O(log n) |
| peek() | O(1) |
| remove(Object) | O(n) |
| contains() | O(n) |
| clear() | O(n) |
| size() | O(1) |
| isEmpty() | O(1) |
| Heapify from Collection | O(n) |

---

# 5. Constructors

## Empty Min Heap

```java
PriorityQueue<Integer> pq = new PriorityQueue<>();
```

---

## Initial Capacity

```java
PriorityQueue<Integer> pq =
new PriorityQueue<>(100);
```

---

## Max Heap

```java
PriorityQueue<Integer> pq =
new PriorityQueue<>(Collections.reverseOrder());
```

---

## Custom Comparator

```java
PriorityQueue<Integer> pq =
new PriorityQueue<>(
(a,b)->Integer.compare(b,a)
);
```

---

## Build From Collection

```java
List<Integer> list =
Arrays.asList(5,2,8,1);

PriorityQueue<Integer> pq =
new PriorityQueue<>(list);
```

Runs in **O(n)**.

---

# 6. Min Heap

```java
PriorityQueue<Integer> pq =
new PriorityQueue<>();

pq.offer(10);
pq.offer(5);
pq.offer(20);
pq.offer(2);

System.out.println(pq.peek());

while(!pq.isEmpty())
    System.out.println(pq.poll());
```

Output

```
2

2
5
10
20
```

---

# 7. Max Heap

```java
PriorityQueue<Integer> pq =
new PriorityQueue<>(
Collections.reverseOrder()
);
```

Equivalent to

```java
PriorityQueue<Integer> pq =
new PriorityQueue<>(
(a,b)->Integer.compare(b,a)
);
```

Output

```
20
10
5
2
```

---

# 8. Why does reverseOrder() make a Max Heap?

Default comparison

```
compare(2,5)

2 is smaller

2 gets higher priority
```

Reverse comparison

```
compare(5,2)

5 gets higher priority
```

Java simply changes **how elements are compared**.

The heap implementation remains exactly the same.

Think of PriorityQueue asking:

> Which element should be closer to the root?

The Comparator answers that question.

---

# 9. Custom Comparators

## Descending Integer

```java
PriorityQueue<Integer> pq =
new PriorityQueue<>(
(a,b)->Integer.compare(b,a)
);
```

---

## Ascending Frequency

```
[value, frequency]
```

```java
PriorityQueue<int[]> pq =
new PriorityQueue<>(
(a,b)->
Integer.compare(a[1],b[1])
);
```

---

## Descending Frequency

```java
PriorityQueue<int[]> pq =
new PriorityQueue<>(
(a,b)->
Integer.compare(b[1],a[1])
);
```

---

## Multiple Conditions

Frequency descending

Value ascending

```java
PriorityQueue<int[]> pq =
new PriorityQueue<>(
(a,b)->{

    if(a[1]!=b[1])
        return Integer.compare(b[1],a[1]);

    return Integer.compare(a[0],b[0]);

});
```

---

# 10. PriorityQueue with Objects

```java
class Student{

    String name;
    int marks;

    Student(String name,int marks){

        this.name=name;
        this.marks=marks;

    }

}
```

Sort by marks

```java
PriorityQueue<Student> pq =
new PriorityQueue<>(
Comparator.comparingInt(
s->s.marks
)
);
```

Descending

```java
PriorityQueue<Student> pq =
new PriorityQueue<>(
Comparator.comparingInt(
(Student s)->s.marks
).reversed()
);
```

---

# 11. Common Operations

## Insert

```java
pq.offer(10);
```

or

```java
pq.add(10);
```

Difference

```
offer()

returns false if insertion fails
```

```
add()

throws Exception
```

---

## Peek

```java
pq.peek();
```

Returns

```
Top element

Does NOT remove
```

---

## Poll

```java
pq.poll();
```

Removes and returns top element.

---

## Remove

```java
pq.remove(10);
```

Complexity

```
O(n)
```

---

## Contains

```java
pq.contains(10);
```

Complexity

```
O(n)
```

---

## Size

```java
pq.size();
```

---

## Empty

```java
pq.isEmpty();
```

---

## Clear

```java
pq.clear();
```

---

# 12. Heapify

Instead of inserting one by one

```java
PriorityQueue<Integer> pq =
new PriorityQueue<>(list);
```

Java performs

```
Heapify
```

Complexity

```
O(n)
```

instead of

```
n insertions

O(n log n)
```

---

# 13. Internal Representation

Printing

```java
System.out.println(pq);
```

Possible output

```
[1,5,2,10]
```

This is NOT sorted.

It is the heap array.

Correct sorted traversal

```java
while(!pq.isEmpty())
    System.out.println(pq.poll());
```

---

# 14. Comparator Best Practices

Avoid

```java
(a,b)->a-b
```

Reason

```
Integer Overflow
```

Instead

```java
Integer.compare(a,b)
```

Descending

```java
Integer.compare(b,a)
```

---

# 15. Common Interview Problems

## K Largest Elements

Maintain Min Heap of size K.

---

## K Smallest Elements

Maintain Max Heap of size K.

---

## Top K Frequent Elements

Heap based on frequency.

---

## Merge K Sorted Lists

Heap based on node value.

---

## Dijkstra

Heap based on shortest distance.

---

## Prim's MST

Heap based on edge weight.

---

## Huffman Coding

Remove two smallest repeatedly.

---

## Median of Stream

Maintain

- Max Heap
- Min Heap

---

## Task Scheduler

Heap based on remaining frequency.

---

## Meeting Rooms

Heap based on end time.

---

## K Closest Points

Heap based on distance.

---

# 16. Pitfalls

PriorityQueue is NOT

- Sorted List
- FIFO Queue
- Thread Safe

Iterator is NOT sorted.

Duplicates are allowed.

Null values are NOT allowed.

contains()

```
O(n)
```

remove(Object)

```
O(n)
```

peek()

```
Does not remove.
```

---

# 17. FAQs

## Is PriorityQueue sorted?

No.

Only the root is guaranteed.

---

## Is PriorityQueue a Binary Search Tree?

No.

It is a Binary Heap.

---

## Can duplicates exist?

Yes.

---

## Can null be inserted?

No.

Throws

```
NullPointerException
```

---

## Does poll() remove?

Yes.

---

## Does peek() remove?

No.

---

## Is iterator sorted?

No.

---

## Is heap stored as a tree?

Conceptually yes.

Internally it is an array.

---

# 18. Cheat Sheet

## Min Heap

```java
PriorityQueue<Integer> pq =
new PriorityQueue<>();
```

---

## Max Heap

```java
PriorityQueue<Integer> pq =
new PriorityQueue<>(
Collections.reverseOrder()
);
```

---

## Custom Comparator

```java
PriorityQueue<int[]> pq =
new PriorityQueue<>(
(a,b)->Integer.compare(a[1],b[1])
);
```

---

## Insert

```java
pq.offer(x);
```

---

## Remove

```java
pq.poll();
```

---

## Peek

```java
pq.peek();
```

---

## Size

```java
pq.size();
```

---

## Empty

```java
pq.isEmpty();
```

---

# Interview Tips

✅ Know the difference between Min Heap and Max Heap.

✅ Understand why `Collections.reverseOrder()` creates a Max Heap.

✅ Know that the heap is **not sorted**.

✅ Remember that `peek()` is O(1), while `offer()` and `poll()` are O(log n).

✅ Prefer `Integer.compare()` over subtraction in comparators.

✅ Recognize common heap problems:
- Top K Elements
- K Closest Points
- Merge K Sorted Lists
- Dijkstra's Algorithm
- Prim's Algorithm
- Huffman Coding
- Median from Data Stream
- Task Scheduling
- Meeting Rooms