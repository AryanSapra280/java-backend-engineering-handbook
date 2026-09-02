📚 Your Stream API Notes — Q1 to Q9

Let's consolidate everything you've learned so far. This is our running notebook. We'll keep adding to it as we solve more questions.

1. filter() — Keep elements matching a condition
Q1 — Even Numbers

Given:

[1, 2, 3, 4, 5, 6]

Want:

[2, 4, 6]
Derivation

Ask:

"Which elements should remain?"

Therefore → filter().

numbers.stream()
       .filter(x -> x % 2 == 0)
       .toList();
Pattern
Collection
    ↓
stream()
    ↓
filter(condition)
    ↓
result
Key point

filter() expects a condition returning:

boolean

Example:

x -> x > 10
2. max() + Comparator
Q2 — Find Maximum

Given:

[10, 20, 30, 40, 50]

Want:

50

You discovered:

numbers.stream()
       .max(Integer::compare);

Equivalent lambda:

.max((a, b) -> Integer.compare(a, b))
What does max() need?

A:

Comparator<Integer>

A Comparator compares two elements.

Its result is:

negative → first comes before second
0        → equal
positive → first comes after second
Important distinction

You initially tried:

(a, b) -> a > b

❌ Wrong because this returns boolean.

Comparator needs:

int
Another possibility
.max((a, b) -> a - b)

works for normal integers, but:

Integer.compare(a, b)

is safer because subtraction can overflow.

3. sorted() + Comparator
Q3 — Sort Descending

You solved it three ways:

numbers.stream()
       .sorted((x, y) -> y - x)
       .toList();

or:

.sorted((x, y) -> Integer.compare(y, x))

or:

.sorted(Collections.reverseOrder())
Why does y - x produce descending?

Comparator contract:

negative → x before y
positive → x after y

For:

x = 10
y = 20

x - y:

10 - 20 = -10

Therefore:

10 comes before 20

→ ascending.

But:

y - x
20 - 10 = +10

Therefore:

10 comes after 20

→ descending.

Mental shortcut
x - y → ascending
y - x → descending

But understand why, don't merely memorize it.

4. count() — Count matching elements
Q4

Given:

["Alice", "Bob", "Annie", "Alex", "Charlie"]

Find how many start with A.

You first produced:

.filter(s -> s.charAt(0) == 'A')
.toList()

which gives:

[Alice, Annie, Alex]

Then recognized:

"The question wants the number, not the elements."

So:

names.stream()
     .filter(s -> s.charAt(0) == 'A')
     .count();
Pattern
filter() + toList()

→ give me matching elements.

filter() + count()

→ give me how many matching elements.

5. chars() + IntStream + mapToObj() + findFirst()
Q5 — First Non-Repeated Character

Input:

"swiss"

Answer:

w

This was our first more interesting problem.

Step 1 — Convert String to characters

You initially thought:

Stream.of(input)

But that produces:

["swiss"]

not:

[s, w, i, s, s]

Then you discovered:

input.chars()

This produces an:

IntStream

Conceptually:

"s" → 115
"w" → 119
"i" → 105
...
Step 2 — Convert int → Character

Because chars() gives IntStream, we used:

.mapToObj(c -> (char) c)

Now:

IntStream
   ↓
Stream<Character>
Step 3 — Find characters occurring once

You derived this condition yourself:

input.indexOf(x) == input.lastIndexOf(x)

Why?

For:

s → indexOf = 0
    lastIndexOf = 4

Different → repeated.

For:

w → indexOf = 1
    lastIndexOf = 1

Same → appears once.

Step 4 — First one
.findFirst()

Complete solution:

input.chars()
     .mapToObj(c -> (char) c)
     .filter(x -> input.indexOf(x) == input.lastIndexOf(x))
     .findFirst();
New concept

findFirst() returns:

Optional<Character>

because there may be no matching character.

6. map() — Transform every element
Q6 — Convert Strings to Uppercase

Input:

["java", "stream", "api"]

Output:

[JAVA, STREAM, API]

You solved:

names.stream()
     .map(s -> s.toUpperCase())
     .toList();
Pattern

map() means:

Take every element and transform it.

Unlike filter():

filter → should this element stay?
map    → what should this element become?
Method References

You then discovered:

.map(String::toUpperCase)

is equivalent to:

.map(s -> s.toUpperCase())

Why does Java know s is a String?

Because:

names.stream()

is:

Stream<String>

Therefore map() knows the input to the function is String.

More examples
.map(String::length)

is equivalent to:

.map(s -> s.length())

For Character:

.map(Character::toUpperCase)

is equivalent to:

.map(c -> Character.toUpperCase(c))
Mental rule

If you're simply calling an existing method on the incoming object:

x -> x.someMethod()

check whether you can write:

Type::someMethod
7. reduce() — Many elements → one value
Q7 — Sum

Given:

[1, 2, 3, 4, 5]

Want:

15

You correctly recognized that we need:

Many values → repeatedly combine → one value

That's reduce().

Conceptually:

initial = 0

0 + 1 → 1
1 + 2 → 3
3 + 3 → 6
6 + 4 → 10
10 + 5 → 15

The form is:

.reduce(0, (i, j) -> i + j)
Important

The:

0

is the initial value.

The:

(i, j) -> i + j

is the combining function.

Think:

i → accumulated value
j → current element
8. distinct() — Remove duplicates
Q8

Input:

[1, 2, 2, 3, 4, 4, 5, 5]

You solved:

numbers.stream()
       .distinct()
       .toList();

Output:

[1, 2, 3, 4, 5]
Pattern
distinct()

→ remove duplicate elements.

It determines uniqueness using the elements' equality semantics (equals() / hashCode()).

9. mapToInt() + average()
Q9 — Average

Input:

[10, 20, 30, 40, 50]

You initially solved it using:

.reduce(0, (i, j) -> i + j) / numbers.size()

Your mathematical reasoning was correct, but there is an important Java issue:

int / int → int

So:

61 / 3

would produce:

20

instead of:

20.333...

Then we learned:

.mapToInt(x -> x)

which converts:

Stream<Integer>
       ↓
IntStream

Now we can use numeric operations:

.sum()
.average()
.max()
.min()

So:

numbers.stream()
       .mapToInt(x -> x)
       .average();

returns:

OptionalDouble[30.0]
Why OptionalDouble?

Because the stream could be empty.

values exist → OptionalDouble[30.0]
no values    → OptionalDouble.empty

You can safely get a default:

.average()
.orElse(0.0);
🧠 Your Stream Mental Model So Far

This is probably the most important section to remember.

When you see a Stream problem, first ask:

1. Am I deciding which elements stay?
filter()
2. Am I changing every element?
map()
3. Am I removing duplicates?
distinct()
4. Am I ordering elements?
sorted()
5. Do I want the number of elements?
count()
6. Do I want one element?
findFirst()
findAny()
max()
min()
7. Do I want to combine everything into one result?
reduce()
8. Do I need numeric operations?
mapToInt()
mapToLong()
mapToDouble()
🔥 Current Pattern Map
                   STREAM
                     │
        ┌────────────┼────────────┐
        ↓            ↓            ↓
    SELECT        TRANSFORM      COMBINE
        │            │            │
     filter         map         reduce
        │
        ↓
   REMOVE DUPES
        │
    distinct()
        │
        ↓
      SORT
        │
    sorted()
        │
        ↓
   NUMERIC STREAM
        │
    mapToInt()
        │
        ├── sum()
        ├── average()
        ├── max()
        └── min()

And we're not going to jump into groupingBy() yet. We'll continue increasing the difficulty gradually so that each new operation has a reason to exist.