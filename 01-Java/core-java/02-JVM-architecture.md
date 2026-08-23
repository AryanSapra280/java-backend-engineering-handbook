# JVM Architecture — Complete Study Notes & Discussion

## Status
🟢 COMPLETED / PARKED

> This is a learning/reference document, not just a revision sheet. It includes the concepts, reasoning, interview questions, corrections, production discussions, and topics intentionally deferred until after the interview.

---

# 1. Big Picture

```text
                         JVM
                          │
        ┌─────────────────┼─────────────────┐
        │                 │                 │
        ▼                 ▼                 ▼
 Class Loader       Runtime Data       Execution Engine
  Subsystem             Areas
        │                 │                 │
        │          ┌──────┼──────┐          ├── Interpreter
        │          │      │      │          └── JIT
        │        Heap   Stack   Metaspace
        │
        ▼
   Loads Classes
```

The interview-relevant JVM areas we covered:

- Runtime Data Areas
- Heap
- Java Stack
- Stack Frames
- Metaspace
- PC Register
- Native Method Stack
- Execution Engine
- Interpreter
- JIT
- Garbage Collection
- GC Roots and reachability
- Mark/Sweep/Compact
- Generational GC
- Young / Old Generation
- Stop-the-World
- G1 GC
- Production performance investigation

---

# 2. Runtime Data Areas

```text
Runtime Data Areas
│
├── Heap
├── Method Area / Metaspace
├── Java Stack
├── PC Register
└── Native Method Stack
```

Useful interview distinction:

```text
                 JVM
                  │
       ┌──────────┴──────────┐
       │                     │
    Shared               Per Thread
       │                     │
   ┌───┴────┐          ┌─────┼─────┐
   │        │          │     │     │
 Heap   Metaspace     Stack   PC  Native
                             Register Stack
```

---

# 3. Heap

The Heap is the main memory area where Java objects are generally allocated.

```java
Employee e = new Employee();
```

Conceptually:

```text
Stack                         Heap

e ─────────────────────────→ Employee object
```

Key points:

- Heap is shared among JVM threads.
- Objects are generally allocated on the Heap.
- Heap is managed by Garbage Collection.
- Objects that become unreachable can eventually have their memory reclaimed.

---

# 4. Java Stack

Each JVM thread has its own Java Stack.

A method invocation creates a stack frame.

```text
Thread Stack
────────────

┌─────────────┐
│ process()   │ ← top
├─────────────┤
│ calculate() │
├─────────────┤
│ main()      │
└─────────────┘
```

Local execution state is associated with the current thread's stack frames.

This is why the Stack is per-thread.

---

# 5. Stack Frames

For:

```text
main()
   ↓
calculate()
   ↓
process()
```

the call stack conceptually becomes:

```text
┌─────────────┐
│ process()   │
├─────────────┤
│ calculate() │
├─────────────┤
│ main()      │
└─────────────┘
```

When `process()` returns, its frame is removed.

---

# 6. StackOverflowError

Example:

```java
void recursive() {
    recursive();
}
```

Every invocation creates another stack frame:

```text
recursive()
recursive()
recursive()
recursive()
...
```

Eventually the thread's stack cannot accommodate more frames and the JVM throws:

```text
StackOverflowError
```

---

# 7. Metaspace

The JVM needs memory for information about loaded classes, including class metadata and related structures.

Historical distinction:

```text
Java 7 and earlier
      ↓
PermGen

Java 8+
      ↓
Metaspace
```

Useful mental model:

```text
Heap
→ Objects

Metaspace
→ Class metadata
```

Do not interpret this as meaning every class-related implementation detail literally lives in one place; the model is for interview understanding.

---

# 8. PC Register

Each JVM thread has its own Program Counter Register.

At a high level, it tracks the current instruction being executed by that thread.

```text
Thread 1 → PC → current instruction
Thread 2 → PC → its current instruction
```

Important:

> PC Register is per-thread.

---

# 9. Native Method Stack

Java can interact with native code through mechanisms such as JNI.

Native methods can be implemented outside Java, commonly in C/C++.

The JVM maintains a native method stack for native method execution.

For the current interview:

> Know what it is and why it exists. Do not spend preparation time on JNI internals.

---

# 10. Execution Engine

The Execution Engine executes JVM bytecode.

```text
Execution Engine
│
├── Interpreter
└── JIT Compiler
```

Garbage Collection is closely related to JVM memory management and is treated separately.

---

# 11. Interpreter

The Interpreter executes bytecode instructions.

```text
Bytecode
   ↓
Interpreter
   ↓
Execution
```

This lets the application start executing without compiling every method into native code upfront.

---

# 12. JIT Compiler

JIT = Just-In-Time Compiler.

The JVM can initially interpret bytecode and observe runtime behavior.

Frequently executed or “hot” code can then be compiled by the JIT into native machine instructions.

```text
Bytecode
   ↓
Interpreter
   ↓
Runtime profiling
   ↓
Hot code detected
   ↓
JIT Compiler
   ↓
Optimization
   ↓
Native Machine Code
```

The compiled code can be reused for subsequent execution.

---

# 13. Why Doesn't Java Compile Everything Immediately?

Suppose Java compiled every method before execution:

```text
.class bytecode
      ↓
Compile EVERYTHING
      ↓
Native machine code
      ↓
Execute
```

A large application may contain thousands of methods while only a subset is executed frequently.

Compiling everything immediately can cause:

- Extra startup time
- Extra compilation work
- Extra memory usage
- Compilation effort spent on rarely executed code

JIT instead uses runtime behavior:

```text
Bytecode
   ↓
Interpreter
   ↓
Observe execution
   ↓
Identify hot code
   ↓
JIT compiles hot code
```

Core idea:

> JIT allows the JVM to make compilation decisions based on actual runtime behavior.

---

# 14. Why Runtime Information Helps JIT

Example:

```java
Animal a = getAnimal();
a.speak();
```

At runtime the JVM can observe actual behavior and use it for optimization.

Important:

> JIT is not simply “compile bytecode again.” It can use runtime information to identify hot code and optimize it.

Advanced optimization names to know eventually:

- Method Inlining
- Dead Code Elimination
- Loop Optimizations
- Escape Analysis
- Devirtualization

---

# 15. Garbage Collection

Java provides automatic memory management.

A beginner statement is:

> “When an object is no longer referenced, GC removes it.”

A better senior-level model is:

> **GC determines reachability from GC Roots.**

When an object is unreachable from GC Roots, it becomes eligible for garbage collection.

Important distinction:

> Eligible for GC does not mean immediately deleted.

---

# 16. GC Roots and Reachability

Example:

```text
GC Root
   │
   ▼
Object A
   │
   ▼
Object B
```

A and B are reachable.

But:

```text
GC Roots
   │
   X

Object C
Object D
```

If there is no path from a GC Root to C or D, they are unreachable and eligible for collection.

Examples of GC Roots include:

- Active thread references
- Local references in active stack frames
- Static references
- JNI references

---

# 17. Circular References

Consider:

```text
A → B
B → A
```

If nothing else references A or B from a GC Root, both can be collected.

```text
GC Roots
   │
   X

A ───→ B
↑       │
└───────┘
```

Important:

> GC uses reachability from GC Roots, not simple reference counting.

---

# 18. Example — Object Becomes Unreachable

```java
Employee e1 = new Employee();
Employee e2 = new Employee();

e1 = e2;
```

Initially:

```text
e1 ─────→ Employee A
e2 ─────→ Employee B
```

After `e1 = e2`:

```text
e1 ─────┐
        ├────→ Employee B
e2 ─────┘

Employee A
    ↑
    │
  no reachable reference
```

Employee A becomes unreachable and is eligible for GC.

---

# 19. Mark and Sweep

Simplified model:

```text
Heap
 ↓
Mark reachable objects
 ↓
Identify unreachable objects
 ↓
Sweep/reclaim their memory
```

Example:

```text
Before:
[A][B][C][D]

A,B → reachable
C,D → garbage

After:
[A][B][free][free]
```

---

# 20. Fragmentation and Compaction

After sweeping, free space may be scattered:

```text
[A][free][B][free][C][free]
```

This is fragmentation.

A simplified compacting process can produce:

```text
[A][B][C][free][free][free]
```

Conceptually:

```text
Mark
 ↓
Sweep
 ↓
Compact
```

Modern collectors use different strategies, so this is a conceptual model rather than a description of every collector.

---

# 21. Generational GC

The key observation:

> **Most objects in typical applications are short-lived.**

Example:

```text
HTTP Request
    ↓
Create temporary objects
    ↓
Process
    ↓
Response
    ↓
Objects become unreachable
```

Therefore, treating every object identically is inefficient.

Conceptually:

```text
Heap
│
├── Young Generation
│   ├── Eden
│   └── Survivor
│
└── Old Generation
```

---

# 22. Young Generation

New objects generally start in the Young Generation.

```text
New Object
    ↓
Young Generation
```

Many objects become unreachable quickly, making Young Generation collection efficient.

---

# 23. Old Generation

Objects that survive collections and remain useful longer may eventually be promoted to the Old Generation.

```text
New Object
    ↓
Young Generation
    ↓
survives collections
    ↓
Old Generation
```

Mental model:

```text
Young → short-lived
Old   → longer-lived
```

---

# 24. Why Young and Old Generations?

Suppose:

```text
900,000 → temporary objects
100,000 → long-lived objects
```

The JVM can focus collection effort where garbage is more likely to exist instead of repeatedly treating all long-lived objects the same way.

Interview answer:

> “The JVM uses generational memory management because most objects are short-lived. This allows collection effort to focus on newly created objects while longer-lived objects can be handled separately.”

---

# 25. Young / Minor GC

When GC primarily collects Young Generation, terms such as Young GC or Minor GC are commonly used.

```text
Young Generation
      ↓
Many objects die
      ↓
Young/Minor GC
      ↓
Dead objects reclaimed
      ↓
Some survivors remain/promote
```

Exact behavior varies by collector.

---

# 26. Production Scenario — Excessive Temporary Objects

Scenario:

```text
Requests
   ↓
Millions of temporary objects
   ↓
Young Generation fills quickly
   ↓
Frequent Young GC
   ↓
CPU/time spent in GC
   ↓
Application performance may degrade
```

Important:

> Frequent Young GC is a symptom, not automatically the root problem.

Investigate:

- Allocation rate
- Temporary object creation
- Large intermediate collections
- DTO creation
- Serialization/deserialization
- Request volume
- Batch processing patterns

Potential approaches:

- Reduce unnecessary allocations
- Stream instead of loading huge collections
- Process in batches
- Reduce intermediate objects
- Improve object lifetime
- Tune JVM/GC only after understanding the workload

Do not immediately solve it by simply increasing the Heap.

---

# 27. Real Resume Example — ObjectMapper Performance Bug

A real performance issue you remembered involved excessive creation of `ObjectMapper` instances for serialization/deserialization.

The reasoning is:

```text
Repeatedly creating ObjectMapper
        ↓
More object allocation
        ↓
Higher allocation rate
        ↓
More short-lived objects
        ↓
More Young GC pressure
        ↓
Potential CPU / latency impact
```

The implementation was changed so that an appropriately configured `ObjectMapper` could be reused instead of repeatedly creating new instances.

A strong interview explanation:

> “One performance issue reported by our performance team was excessive creation of ObjectMapper instances for serialization and deserialization. I investigated the allocation pattern and found that we were repeatedly creating these objects instead of reusing an appropriately configured instance. I changed the implementation to reuse a single instance, which reduced unnecessary object allocation and therefore reduced GC pressure.”

Do not claim a percentage improvement unless you have measured evidence.

---

# 28. Stop-the-World

Stop-the-World means application threads are temporarily paused for a particular JVM/GC phase.

```text
Normal:

Thread 1  ─────────────────────────
Thread 2  ─────────────────────────
Thread 3  ─────────────────────────

STW:

Thread 1  ─────── XXXXXXXXX ───────
Thread 2  ─────── XXXXXXXXX ───────
Thread 3  ─────── XXXXXXXXX ───────
                 ↑
               paused
```

Important:

> Not every GC operation means the entire application is continuously stopped.

Modern collectors perform significant work concurrently, but certain phases can still require pauses.

---

# 29. Why STW Can Matter

Suppose an API normally responds in 20 ms but a GC pause takes 200 ms.

```text
20ms  18ms  22ms  200ms  21ms
```

This can create a latency spike.

Impact depends on workload:

```text
Batch processing:
500ms pause → potentially acceptable

Low-latency API:
500ms pause → potentially serious
```

Senior framing:

> Evaluate GC pauses against application latency and availability requirements.

---

# 30. Throughput vs Latency

GC behavior can involve a trade-off.

### Throughput

How much useful work the application performs over time.

### Latency

How long an individual request takes.

A batch system may prioritize throughput.

A customer-facing API may prioritize predictable low latency.

---

# 31. Production Investigation — GC and Latency

Scenario:

```text
Normal API latency → 30 ms
Occasional latency → 500–800 ms
GC activity occurs around the same time
```

Investigation:

```text
API latency spike
      ↓
Check traces / application metrics
      ↓
Correlate latency spike with GC events
      ↓
Check:
  ├── GC frequency
  ├── GC pause duration
  ├── Young GC
  ├── Old / Full GC
  └── Heap occupancy
      ↓
Determine whether GC actually caused the latency
      ↓
If yes:
      ↓
Investigate allocation rate
      ↓
Identify excessive temporary object creation
      ↓
Reduce unnecessary allocations
      ↓
If workload is legitimate:
consider JVM / GC tuning
```

Do not only check frequency.

Compare:

```text
1000 GC pauses × 1 ms
```

with:

```text
10 GC pauses × 500 ms
```

Pause duration can matter more than raw count.

---

# 32. G1 Garbage Collector

G1 = **Garbage-First Garbage Collector**.

For the current interview we only need high-level knowledge.

G1 is designed for efficient collection of large heaps with more predictable/manageable pause behavior.

---

# 33. G1 Heap Regions

G1 conceptually divides the Heap into many regions:

```text
┌────┬────┬────┬────┬────┬────┐
│ R1 │ R2 │ R3 │ R4 │ R5 │ R6 │
├────┼────┼────┼────┼────┼────┤
│ R7 │ R8 │ R9 │ R10│ R11│ R12│
└────┴────┴────┴────┴────┴────┘
```

Different regions can contain objects associated with different generations.

Key idea:

> G1 manages the Heap as a collection of regions.

---

# 34. Why “Garbage-First”?

Suppose:

```text
R1 → 90% live
R2 → 10% live
R3 → 80% live
R4 → 5% live
R5 → 70% live
```

R4 and R2 contain a high proportion of garbage.

G1 can prioritize regions that provide more reclaimable space relative to collection work.

Hence:

> Garbage-First.

---

# 35. G1 and Pause Time

Conceptually:

```text
Large Heap
    ↓
Regions
    ↓
Identify regions with lots of garbage
    ↓
Select work within pause target
    ↓
Collect
```

G1 is not pause-free.

It still has STW phases, while significant GC work can happen concurrently.

---

# 36. G1 — What to Know Now

Know:

- G1 = Garbage-First Garbage Collector
- Heap is divided into regions
- Regions can represent different generations
- G1 prioritizes regions with more reclaimable garbage
- Designed for large heaps
- Aims for more predictable/manageable pause behavior
- Still has STW phases
- Performs significant work concurrently

Do not currently spend time on:

- Remembered Sets
- Card Tables
- SATB
- Detailed marking phases
- Write Barriers
- Region-selection internals
- Humongous-object internals
- Detailed evacuation internals
- ZGC internals
- Shenandoah internals
- GC tuning flags

---

# 37. Interview Questions We Discussed

## Q1 — Heap vs Stack

> What is the difference between Heap and Stack? Where does `Employee e = new Employee()` go?

Key answer:

- Object → generally Heap
- Reference/local execution state → current stack frame
- Heap → shared
- Stack → per thread

---

## Q2 — JIT

> Why doesn't Java compile all bytecode into native machine code immediately?

Key answer:

- Avoid unnecessary upfront compilation
- Reduce startup compilation work
- Use runtime profiling
- Identify hot code
- Optimize actual runtime behavior
- Compile hot paths into native code

---

## Q3 — GC Reachability

> How does GC know which objects can be removed?

Key answer:

- Start from GC Roots
- Follow references
- Objects with no path from GC Roots become unreachable
- They become eligible for GC

---

## Q4 — Circular References

> Can GC collect two objects that reference each other?

Yes.

```text
A → B
B → A
```

If no GC Root can reach either object, both are eligible.

---

## Q5 — Mark and Sweep

> How does GC reclaim memory?

Simplified:

```text
Mark reachable
     ↓
Identify garbage
     ↓
Sweep/reclaim
     ↓
Possibly compact
```

---

## Q6 — Young vs Old Generation

> Why does JVM distinguish Young and Old generations?

Because most objects are short-lived, allowing collection effort to focus on areas where garbage is more likely to exist.

---

## Q7 — Frequent Young GC

> Your service experiences frequent Young GC and high CPU. What do you do?

Answer direction:

- Measure allocation rate
- Identify allocation-heavy code
- Investigate temporary objects
- Reduce unnecessary allocations
- Consider streaming/batching
- Tune JVM/GC only after understanding the workload

---

## Q8 — GC and Latency

> API latency jumps from 30 ms to 800 ms and GC happens around the same time. What do you investigate?

Answer direction:

- Correlate traces and GC events
- Check GC frequency
- Check pause duration
- Check heap occupancy
- Check allocation rate
- Establish causality
- Investigate application allocation
- Tune only after diagnosis

---

## Q9 — G1

> Which GC would you consider for a large-heap, latency-sensitive Spring Boot application?

Answer direction:

- Consider G1
- Region-based Heap
- Garbage-first selection
- Designed for large heaps
- More predictable/manageable pauses
- Still has STW phases
- Choose based on workload and metrics

---

# 38. Final Interview-Level Summary

> “The JVM provides the runtime environment for executing Java bytecode. It loads required classes through the Class Loader subsystem and maintains runtime data areas such as the Heap, thread-specific Java Stacks, Metaspace and PC Registers. The Execution Engine can interpret bytecode and use JIT compilation to optimize frequently executed code into native machine instructions. The JVM also manages Heap memory through Garbage Collection, which determines object reachability from GC Roots and reclaims memory from unreachable objects. Generational approaches separate short-lived and long-lived objects, while collectors such as G1 use region-based Heap management and aim for efficient collection with manageable pause times.”

---

# 39. What We Intentionally Deferred

These are good topics for a post-interview JVM deep dive.

## Class Loading — Advanced

- Class Loader Internals
- Custom Class Loaders
- Class Loader Isolation
- Class Identity
- Class Unloading
- Class Loader Leaks
- Java Module System
- JPMS
- `Class.forName()`
- Context Class Loader

## JVM Internals

- JVM Specification
- Detailed Runtime Data Areas
- Stack Frame Internals
- Operand Stack
- Local Variable Array
- Bytecode Instructions
- Bytecode Verification
- JVM Instruction Set
- Method Area Internals
- Metaspace Internals
- Compressed OOPs
- Object Headers
- Memory Alignment
- TLAB
- JNI
- JVMTI

## JIT / HotSpot

- HotSpot JVM
- C1 Compiler
- C2 Compiler
- Tiered Compilation
- Method Inlining
- Deoptimization
- Speculative Optimization
- On-Stack Replacement
- Escape Analysis
- Devirtualization
- JIT Compilation Thresholds
- JIT Diagnostics

## GC Deep Dive

- GC Roots Internals
- Generational GC Internals
- Eden Internals
- Survivor Age Counters
- Object Promotion Internals
- Card Tables
- Remembered Sets
- Write Barriers
- SATB
- Detailed G1 Marking
- G1 Evacuation Internals
- Humongous Objects
- ZGC Internals
- Shenandoah Internals
- GC Logs
- GC Tuning Flags
- Advanced GC Profiling

---

# 40. Production Lessons

The most important production lesson from this topic:

> **Investigate application allocation behavior before blindly tuning JVM configuration.**

Example:

```text
High CPU + frequent GC
        ↓
Check allocation rate
        ↓
Find allocation-heavy code
        ↓
Reduce unnecessary allocations
        ↓
Measure again
        ↓
Tune JVM if still required
```

The ObjectMapper issue is a practical example of this principle.

---

# 41. Final Status

🟢 **JVM Architecture — COMPLETED / PARKED**

Covered to the required interview depth:

- Runtime Data Areas
- Heap
- Stack
- Stack Frames
- StackOverflowError
- Metaspace
- PC Register
- Native Method Stack
- Interpreter
- JIT
- Runtime profiling
- JIT motivation
- Garbage Collection
- GC Roots
- Reachability
- Circular references
- Mark/Sweep/Compact
- Generational GC
- Young / Old Generation
- Minor / Young GC
- Stop-the-World
- Throughput vs Latency
- GC performance investigation
- G1 high level
- Real-world ObjectMapper performance example

Next Java Core topic:

> **OOP — Classes, Objects, Encapsulation, Abstraction, Inheritance, Polymorphism, Interfaces and Abstract Classes**
