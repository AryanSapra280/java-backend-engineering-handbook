# Question 1: Why are `Comparable` and `Comparator` interfaces required in Java?

## 1. Why do we need them?

Java provides methods such as:

```java
Arrays.sort()
Collections.sort()
```

to sort arrays and collections.

For **primitive types** and many standard Java classes, Java already knows how to compare the values.

For example:

```java
int[] numbers = {5, 2, 8, 1};

Arrays.sort(numbers);
```

Java knows how to determine that:

```text
1 < 2 < 5 < 8
```

Similarly, Java knows how to compare common types such as `String`, `Integer`, `Double`, etc.

However, consider a custom class:

```java
class Employee {
    private int id;
    private String name;

    Employee(int id, String name) {
        this.id = id;
        this.name = name;
    }
}
```

Suppose we have:

```java
List<Employee> employees = new ArrayList<>();

employees.add(new Employee(2, "Sapra"));
employees.add(new Employee(1, "Aryan"));
```

If we simply do:

```java
Collections.sort(employees);
```

Java does not know **how two `Employee` objects should be ordered**.

For example, should employees be sorted by:

```text
ID?
Name?
Salary?
Joining date?
Age?
```

There is no universal answer.

This is where `Comparable` and `Comparator` come in.

---

# 2. `Comparable` — Define the Natural Ordering

`Comparable` is used when a class has a **natural/default ordering**.

The class itself defines how its objects should be compared.

```java
class Employee implements Comparable<Employee> {

    private int id;
    private String name;

    Employee(int id, String name) {
        this.id = id;
        this.name = name;
    }

    public int getId() {
        return id;
    }

    @Override
    public int compareTo(Employee other) {
        return Integer.compare(this.id, other.id);
    }

    @Override
    public String toString() {
        return "id: " + id + " name: " + name;
    }
}
```

Now we can write:

```java
List<Employee> employees = new ArrayList<>();

employees.add(new Employee(2, "Sapra"));
employees.add(new Employee(1, "Aryan"));
employees.add(new Employee(10, "Acyan"));

Collections.sort(employees);
```

Output:

```text
id: 1 name: Aryan
id: 2 name: Sapra
id: 10 name: Acyan
```

Why?

Because `Employee` implements:

```java
Comparable<Employee>
```

and defines:

```java
compareTo()
```

as:

```java
Integer.compare(this.id, other.id)
```

Therefore, the **natural ordering of `Employee` is by ID**.

### Key idea

```text
Comparable
     ↓
Class defines its own default ordering
     ↓
compareTo()
     ↓
Natural ordering
```

---

# 3. What does `compareTo()` return?

The general contract is:

```java
a.compareTo(b)
```

returns:

```text
< 0   → a comes before b
  0   → a and b are considered equal for ordering
> 0   → a comes after b
```

For example:

```java
Integer.compare(1, 2)
```

returns a negative value because:

```text
1 comes before 2
```

And:

```java
Integer.compare(10, 2)
```

returns a positive value because:

```text
10 comes after 2
```

You generally should **not depend on the exact value** being `-1`, `0`, or `1`. Only the sign matters.

---

# 4. The Problem with `Comparable`

Suppose our natural ordering is by ID:

```java
@Override
public int compareTo(Employee other) {
    return Integer.compare(this.id, other.id);
}
```

This works perfectly when we want:

```text
Sort by ID
```

But now suppose we want:

```text
Sort by Name
```

We could change `compareTo()`:

```java
@Override
public int compareTo(Employee other) {
    return this.name.compareTo(other.name);
}
```

But now we have changed the **natural ordering of Employee** for every place where `compareTo()` is used.

What if one part of our application needs:

```text
Sort by ID
```

while another needs:

```text
Sort by Name
```

We need a way to define **multiple different sorting strategies**.

This is where `Comparator` is useful.

---

# 5. `Comparator` — Define Custom Ordering

A `Comparator` allows us to define a sorting rule **outside the class**.

For example:

```java
Comparator<Employee> byName =
        Comparator.comparing(Employee::getName);
```

Then:

```java
employees.sort(byName);
```

Now the list is sorted by name.

We can also create another comparator:

```java
Comparator<Employee> byId =
        Comparator.comparingInt(Employee::getId);
```

So we can choose the ordering we need:

```java
employees.sort(byId);
```

or:

```java
employees.sort(byName);
```

without modifying the `Employee` class.

---

# 6. Complete Example

```java
import java.util.*;

class Employee implements Comparable<Employee> {

    private int id;
    private String name;

    Employee(int id, String name) {
        this.id = id;
        this.name = name;
    }

    public int getId() {
        return id;
    }

    public String getName() {
        return name;
    }

    // Natural ordering: by ID
    @Override
    public int compareTo(Employee other) {
        return Integer.compare(this.id, other.id);
    }

    @Override
    public String toString() {
        return "id: " + id + " name: " + name;
    }
}

public class Main {

    public static void main(String[] args) {

        Employee e1 = new Employee(1, "Aryan");
        Employee e2 = new Employee(2, "Sapra");
        Employee e3 = new Employee(10, "Acyan");

        List<Employee> employees = new ArrayList<>();

        employees.add(e2);
        employees.add(e1);
        employees.add(e3);

        // Natural ordering → by ID
        Collections.sort(employees);

        System.out.println(employees);

        // Custom ordering → by Name
        employees.sort(
            Comparator.comparing(Employee::getName)
        );

        System.out.println(employees);
    }
}
```

The important difference is:

```java
Collections.sort(employees);
```

uses the **natural ordering** defined by:

```java
Employee.compareTo()
```

whereas:

```java
employees.sort(
    Comparator.comparing(Employee::getName)
);
```

uses the **custom ordering** supplied by the `Comparator`.

---

# 7. `Comparable` vs `Comparator`

|                       | `Comparable`                   | `Comparator`            |
| --------------------- | ------------------------------ | ----------------------- |
| Package               | `java.lang`                    | `java.util`             |
| Method                | `compareTo()`                  | `compare()`             |
| Who defines ordering? | The class itself               | External object         |
| Purpose               | Natural/default ordering       | Custom ordering         |
| Number of orderings   | Generally one natural ordering | Can have many           |
| Modifies class?       | Yes, class implements it       | No                      |
| Example               | Employee → sort by ID          | Employee → sort by name |

---

# 8. Easy Way to Remember

Think:

### `Comparable` → "I can compare myself."

```java
employee1.compareTo(employee2);
```

The `Employee` class knows how to compare itself with another `Employee`.

### `Comparator` → "Someone else compares them."

```java
comparator.compare(employee1, employee2);
```

The comparison logic is provided externally.

---

# 9. Real-world analogy

Suppose you have a list of students.

A student has:

```text
ID
Name
Age
Marks
```

There is no single universally correct way to sort them.

You might want:

```text
Default → ID
```

Then:

```text
Comparable<Student>
```

could define:

```java
compareTo() → compare IDs
```

But sometimes you want:

```text
Sort by Name
Sort by Marks
Sort by Age
Sort by Name descending
Sort by Marks descending
```

These can be represented using different `Comparator`s:

```java
Comparator.comparing(Student::getName)

Comparator.comparingInt(Student::getMarks)

Comparator.comparingInt(Student::getAge)

Comparator.comparing(Student::getName).reversed()

Comparator.comparingInt(Student::getMarks).reversed()
```

Therefore:

```text
Comparable
   ↓
One natural/default ordering

Comparator
   ↓
Multiple possible/custom orderings
```

---

# 10. Interview Answer

If an interviewer asks:

> **Why are Comparable and Comparator required in Java?**

A strong answer would be:

> Java's sorting APIs need a way to determine the ordering between objects. For standard types, Java already provides comparison logic, but for custom classes there may be multiple valid ways to order the objects. `Comparable` allows a class to define its natural ordering through `compareTo()`, while `Comparator` allows us to define external/custom ordering strategies without modifying the class. This allows the same objects to be sorted by different fields such as ID, name, salary, or age.

### One-line memory trick

```text
Comparable  → Natural ordering → compareTo()
Comparator  → Custom ordering  → compare()
```
