# B. JDK, JRE & JVM — Answers

### 19. What is the JDK?

**JDK (Java Development Kit)** is the software development kit used to **develop, compile, debug, package, and run Java applications**.

It includes:

* JVM
* Java runtime libraries
* Development tools such as `javac`, `javadoc`, `jdb`, `jar`, `jlink`, etc.

```text
JDK
├── Development tools
│   ├── javac
│   ├── javadoc
│   ├── jdb
│   └── jar
│
└── Java runtime
    ├── JVM
    └── Java libraries
```

---

### 20. What is the JRE?

**JRE (Java Runtime Environment)** is the conceptual/runtime environment required to **run Java applications**.

Traditionally:

```text
JRE = JVM + Java runtime libraries
```

It provides the components required for execution, but not the complete development toolset such as `javac`.

**Modern Java note:** Starting with Java 9, Oracle/OpenJDK no longer distributes a separate general-purpose JRE download in the traditional form. Runtime images can instead be created with tools such as `jlink`.

---

### 21. What is the JVM?

**JVM (Java Virtual Machine)** is the runtime engine that **loads, verifies, links, initializes, and executes Java bytecode**.

It provides services such as:

* Class loading
* Bytecode verification
* Memory management
* Garbage collection
* Thread management
* Runtime execution
* JIT compilation

Example:

```text
Test.java
   ↓ javac
Test.class
   ↓
JVM
   ↓
Machine instructions
   ↓
CPU
```

The JVM is specified by the **JVM Specification**; implementations such as HotSpot provide the actual JVM.

---

### 22. What is the difference between JDK, JRE and JVM?

Think of them as different levels:

```text
JDK
├── Development tools
└── Runtime environment
      └── JVM
```

| JDK                                       | JRE                                  | JVM                      |
| ----------------------------------------- | ------------------------------------ | ------------------------ |
| Used to develop and run Java applications | Used to run Java applications        | Executes bytecode        |
| Contains development tools                | Runtime libraries + JVM conceptually | Execution engine/runtime |
| Includes compiler (`javac`)               | Doesn't include compiler             | Doesn't include `javac`  |
| Largest scope                             | Runtime scope                        | Core execution component |

### Interview answer

> **JDK is for development, JRE is the runtime environment, and JVM is the virtual machine that actually executes Java bytecode.**

---

### 23. What components are included in a JDK?

A JDK includes the tools and runtime components needed for Java development.

Important tools include:

* `javac` → Java compiler
* `java` → application launcher
* `javadoc` → documentation generator
* `jar` → JAR creation/manipulation
* `jdb` → debugger
* `javap` → class-file disassembler
* `jshell` → interactive Java shell
* `jlink` → creates custom runtime images
* `jpackage` → packages applications for distribution

And the runtime includes the JVM and Java platform runtime libraries/modules.

For example:

```bash id="v4d7d2"
javac Test.java
java Test
```

Both commands are supplied by a JDK installation.

---

### 24. What components are required to run a Java application?

At a conceptual level, you need:

1. **JVM**
2. **Required Java runtime libraries/modules**
3. **The application's compiled classes/JARs**
4. Any required external dependencies.

For a traditional classpath-based application:

```text
Application
   ↓
Java runtime libraries
   ↓
JVM
   ↓
OS / CPU
```

You do **not** need the Java compiler merely to execute an already-compiled application.

---

### 25. Is JRE still distributed separately in modern Java versions?

**Not in the traditional form.**

For modern Oracle/OpenJDK distributions, the old standalone JRE distribution model is no longer the standard approach.

Since **Java 9**, the Java runtime was modularized, and Oracle stopped providing the traditional separate JRE download.

Instead, developers can create a **custom runtime image** containing only the required modules using:

```bash id="j6x3gj"
jlink
```

So in modern Java, don't assume:

```text
Download JDK
Download separate JRE
```

is the normal installation model.

---

### 26. Does the JVM itself provide platform independence?

**Not by itself.**

The JVM is the **platform-specific implementation layer** that allows platform-independent Java bytecode to execute.

Think:

```text
                Same bytecode
                     │
          ┌──────────┼──────────┐
          ↓          ↓          ↓
     Windows JVM  Linux JVM  macOS JVM
          ↓          ↓          ↓
       Windows     Linux      macOS
```

The **combination of portable bytecode + JVM implementations for different platforms** provides Java's platform independence.

---

### 27. Why does Java need a different JVM implementation for different operating systems?

Because the JVM ultimately has to interact with the **underlying operating system and hardware**.

For example, a JVM running on:

```text
Windows + x86-64
```

has to interact with Windows APIs and x86-64 hardware.

A JVM running on:

```text
Linux + ARM64
```

has different OS interfaces and CPU instructions to deal with.

Therefore, the JVM implementation contains platform-specific code.

But both JVMs understand the same Java bytecode specification.

```text
Java bytecode
      ↓
┌──────────────┬──────────────┐
Windows JVM    Linux JVM
      ↓              ↓
Windows OS       Linux OS
```

---

### 28. Can the same JVM execute bytecode generated from different programming languages?

**Yes**, provided those languages compile to valid JVM bytecode and follow the JVM's requirements.

The JVM doesn't fundamentally care whether bytecode originated from Java or another JVM language.

For example:

```text
Java source ────────┐
Kotlin source ──────┤
Scala source ───────┼──→ JVM bytecode → JVM
Groovy source ──────┤
Clojure source ─────┘
```

This is one of the major strengths of the JVM ecosystem.

---

### 29. What other languages can run on the JVM?

Examples include:

* **Kotlin**
* **Scala**
* **Groovy**
* **Clojure**
* **JRuby**
* **Jython**
* **Ceylon**
* **Xtend**

Some languages compile primarily to JVM bytecode, while others use different implementation strategies but can interoperate with the JVM ecosystem.

For example:

```text
Kotlin
   ↓
JVM bytecode
   ↓
JVM
```

This also means Kotlin and Java can generally interoperate within the same JVM application.

---

### 30. What is the relationship between JVM specification and a specific JVM implementation such as HotSpot?

This is an important distinction.

### JVM Specification

The **JVM Specification** defines the rules and behavior that a conforming JVM implementation must provide, including things such as:

* Class-file format.
* Instruction set/bytecode semantics.
* Runtime data areas.
* Class loading/linking requirements.
* Execution semantics.

### JVM implementation

**HotSpot** is a concrete JVM implementation.

It implements the JVM specification and adds implementation-specific techniques and optimizations such as:

* JIT compilation.
* Garbage collectors.
* Runtime profiling.
* Method inlining.
* Escape analysis.
* Various performance optimizations.

Think:

```text
JVM Specification
       │
       │ defines contract
       ↓
┌───────────────┬────────────────┐
HotSpot         OpenJ9           Other JVMs
implementation  implementation   implementations
       │
       ↓
Actual execution
```

So:

> **JVM Specification = what a JVM must do.**
> **HotSpot = one implementation of that specification.**

### Key revision

```text
JDK
│
├── Development tools
│     ├── javac
│     ├── jar
│     ├── javadoc
│     ├── jdb
│     └── etc.
│
└── Runtime
      │
      ├── JVM
      │
      └── Java runtime libraries/modules
```

And the most important distinction:

> **JDK → develop + run**
> **JRE → conceptual runtime environment**
> **JVM → executes bytecode**
> **HotSpot → a JVM implementation**
> **JVM Specification → rules the implementation follows**
