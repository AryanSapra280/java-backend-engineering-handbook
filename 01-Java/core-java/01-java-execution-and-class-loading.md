# Java Execution & Class Loading

## Status

🟢 COMPLETED / PARKED

---

# 1. What is Java?

Java is a statically typed, object-oriented programming language and platform.

Java source code is compiled into platform-independent JVM bytecode, which is then executed by a JVM.

## Basic Flow

```text
Java Source Code
        ↓
      javac
        ↓
   JVM Bytecode
      (.class)
        ↓
       JVM
        ↓
Native Machine Instructions
        ↓
       CPU
```

---

# 2. What happens when a Java program runs?

Suppose we have:

```java
public class Hello {
    public static void main(String[] args) {
        System.out.println("Hello");
    }
}
```

## Step 1 — Write Java source code

The source code is stored in a `.java` file.

Example:

```text
Hello.java
```

## Step 2 — Compile the source code

We use:

```bash
javac Hello.java
```

The Java compiler (`javac`) performs compilation checks such as syntax and type checking.

If compilation succeeds, it generates:

```text
Hello.class
```

## Step 3 — `.class` contains bytecode

The `.class` file contains JVM bytecode.

Bytecode is:

- Platform-independent
- Designed to be executed by the JVM
- Not native machine code

Important distinction:

> JVM bytecode is an intermediate instruction set; it is not CPU-specific machine code.

---

# 3. Starting the Java application

When we run:

```bash
java Hello
```

the JVM starts and loads the required classes.

The Class Loader is responsible for loading class definitions required by the application.

Conceptually:

```text
Hello.class
     ↓
Class Loader
     ↓
JVM
     ↓
Execution Engine
     ↓
Program execution
```

---

# 4. Class Loader

The Class Loader loads class definitions into the JVM.

The JVM needs the class definition before it can perform operations that require that class, such as creating an object or invoking methods.

Example:

```java
Employee employee = new Employee();
```

Conceptually:

```text
Employee.class
     ↓
Class Loader loads Employee
     ↓
JVM knows Employee's class definition
     ↓
new Employee()
     ↓
Employee object is created
```

## Important distinction

Class loading and object creation are different operations.

```text
Class Loading
    ↓
Loads class definition

Object Creation
    ↓
Creates an instance of that class
```

---

# 5. Classpath vs Class Loader

These should not be confused.

### Classpath

The classpath tells the JVM/Class Loader where classes and libraries can be found.

### Class Loader

The Class Loader actually loads the required class definitions into the JVM.

Think:

```text
.class files / JARs
        ↓
     Classpath
        ↓
   Class Loader
        ↓
        JVM
```

Do NOT say:

> "The class is loaded into the classpath."

Better:

> "The classpath provides locations from which the Class Loader can find classes, and the Class Loader loads the class definition into the JVM."

---

# 6. Types of Class Loaders

There are three important built-in Class Loader levels to know.

## 6.1 Bootstrap Class Loader

Responsible for loading core Java classes.

Examples include classes from:

```text
java.lang
java.util
java.io
```

## 6.2 Platform Class Loader

Loads Java platform classes that are outside the core Bootstrap-loaded set.

In older Java versions, this concept was associated with the Extension Class Loader.

Modern Java terminology:

> Platform Class Loader

## 6.3 Application Class Loader

Loads application classes from the application's classpath/module path.

Examples:

```text
MyApplication.class
Employee.class
OrderService.class
PaymentService.class
```

This is the Class Loader a typical backend developer encounters most frequently.

---

# 7. Class Loader Hierarchy

Conceptually:

```text
Bootstrap Class Loader
        ↑
Platform Class Loader
        ↑
Application Class Loader
```

The Application Class Loader is associated with the Platform Class Loader as its parent, and the Platform Class Loader delegates to the Bootstrap Class Loader.

---

# 8. Parent Delegation Model

Class loading generally follows the parent delegation model.

When a Class Loader receives a request to load a class, it gives its parent Class Loader the opportunity to load it first.

Conceptually:

```text
Application Class Loader
          ↓
     asks parent
          ↓
Platform Class Loader
          ↓
     asks parent
          ↓
Bootstrap Class Loader
```

If the parent can load the class, the parent does so.

If the parent cannot load it, the child Class Loader gets the opportunity to load it.

## Why Parent Delegation?

One important reason is security and consistency.

For example, suppose an application attempts to provide its own implementation of:

```text
java.lang.String
```

We don't want an application to replace trusted core Java classes with arbitrary implementations.

Parent delegation allows the trusted parent Class Loader to get the opportunity to load such classes first.

---

# 9. Interpreter vs JIT

The JVM's Execution Engine executes bytecode.

Conceptually:

```text
             Bytecode
                ↓
        Execution Engine
          /          \
         /            \
Interpreter          JIT
    ↓                  ↓
Executes          Compiles hot
bytecode          code into
                  native code
```

The JVM can interpret bytecode.

The JIT (Just-In-Time) compiler identifies frequently executed or "hot" code and compiles it into native machine instructions.

The compiled native code can then be reused for subsequent execution.

### Interview-ready explanation

> "The JVM can initially interpret bytecode. During execution it identifies frequently executed code paths, and the JIT compiler compiles those hot code paths into native machine code so they can execute more efficiently."

---

# 10. Application Deployment and Class Loading

For a normal Spring Boot production application:

```text
Application JAR v1
       ↓
JVM starts
       ↓
Application Class Loader
       ↓
Classes from JAR are loaded
       ↓
Application runs
```

If we build a new version:

```text
Application JAR v2
```

the already-running JVM does not automatically replace the already-loaded class definitions with the new versions.

A normal deployment therefore looks like:

```text
Build new JAR
      ↓
Deploy new JAR
      ↓
Restart application
      ↓
New JVM process
      ↓
Classes loaded from new JAR
      ↓
Application runs new version
```

Important:

> The JVM process is running with the class definitions it loaded; simply placing a new JAR on disk does not automatically replace those loaded classes.

---

# 11. Class Unloading — High-Level Understanding

Class unloading is an advanced topic and is NOT required to go deeply into for the current interview preparation.

The important idea is:

> A loaded class is not simply replaced whenever a new `.class` file appears.

Class unloading is associated with the Class Loader becoming eligible for garbage collection, rather than the JVM arbitrarily replacing an individual loaded class.

For normal Spring Boot deployments, application restart is the standard way to load the new application classes.

---

# 12. Important Interview Distinctions

## Bytecode vs Machine Code

```text
Bytecode
→ Platform-independent
→ Executed by JVM

Machine Code
→ CPU/platform-specific
→ Executed by CPU
```

## Classpath vs Class Loader

```text
Classpath
→ Location from which classes can be found

Class Loader
→ Loads class definitions into JVM
```

## Class Loading vs Object Creation

```text
Class Loading
→ Loads class definition

Object Creation
→ Creates an instance of the class
```

## Interpreter vs JIT

```text
Interpreter
→ Executes bytecode

JIT
→ Compiles frequently executed bytecode into native machine code
```

---

# 13. Interview-Ready Answer

### Question

> "What happens when you run a Java program?"

### Answer

When we write a Java program, the source code is stored in a `.java` file.

We compile it using `javac`, which performs compilation checks and generates a `.class` file containing platform-independent JVM bytecode.

When we run the application using the `java` command, the JVM starts and the Class Loader loads the required class definitions.

The JVM's Execution Engine executes the bytecode. Bytecode can initially be interpreted, while the JIT compiler identifies frequently executed code and compiles those hot code paths into native machine instructions for better performance.

Finally, the JVM invokes:

```java
public static void main(String[] args)
```

as the entry point of the application.

---

# 14. Senior-Level Points to Remember

- `.java` → source code
- `javac` → compiler
- `.class` → JVM bytecode
- Bytecode is NOT native machine code
- JVM executes bytecode
- Class Loader loads class definitions
- Classpath provides locations from which classes can be found
- Bootstrap, Platform and Application Class Loaders are important
- Parent delegation is an important Class Loader concept
- Interpreter executes bytecode
- JIT compiles frequently executed code into native code
- A running JVM does not automatically replace loaded application classes when a new JAR is deployed
- Normal Spring Boot deployments restart the application to load the new version

---

# 15. Topics to Learn Later

These are intentionally only topic names for post-interview learning.

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

## JVM — Advanced

- JVM Architecture
- Runtime Data Areas
- Heap
- Stack
- Metaspace
- Program Counter Register
- Native Method Stack
- Stack Frames
- Method Area
- Runtime Constant Pool
- JVM Execution Engine
- JIT Compilation
- HotSpot JVM
- JVM Profiling
- JVM Tuning

## Garbage Collection — Advanced

- GC Fundamentals
- Young Generation
- Old Generation
- Minor GC
- Major GC
- Full GC
- Stop-The-World
- G1 GC
- ZGC
- Shenandoah
- GC Tuning
- GC Logs
- Memory Leaks
