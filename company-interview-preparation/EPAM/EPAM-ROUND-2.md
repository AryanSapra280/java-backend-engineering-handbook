Absolutely. I went through the screenshots again. I’ll separate **the numbered EPAM question list** from the **additional detailed coding/system-design/AI screenshots**, so we have a clean master checklist.

# EPAM — Questions from the screenshots

## 1. Data Structures & Coding

1. What data structures are you experienced with in Java?
2. Do you have experience working with stacks and queues?
3. How does a stack differ from a queue?
4. Explain LIFO and FIFO.
5. Given a stack containing `1 2 3 4`, how would you reverse the stack so it becomes `4 3 2 1`?
6. Given an array `1 2 3 4`, how would you reverse the array efficiently without using extra space?
7. Can you reverse the array by iterating only up to `n/2`?
8. How can you swap two elements without using a third/temporary variable?
9. What mathematical approach can be used to swap two numbers without an extra variable?
10. What is the time complexity of your array-reversal approach?
11. Given an `ArrayList<Integer>` containing `1,2,3,4,5,6`, write code to reverse the list.
12. Debug the implementation of the list-reversal code.
13. Why are you getting an `IndexOutOfBoundsException`?
14. Can you write and execute the complete Java code for the reversal problem?

---

# 2. Java 8 / Streams

15. Generate a stream of numbers from 1 to 100 and find the sum of all even numbers.
16. If `n = 200`, generate a stream containing numbers from 1 to 200.
17. Can you use `IntStream` / `Stream.iterate()` to generate the numbers?
18. How would you filter only the even numbers from the stream?
19. How would you calculate the sum of the filtered numbers?
20. What is the difference between `IntStream.range()` and `IntStream.rangeClosed()`?
21. What is a Stream in Java?
22. How is a Stream different from a Collection?
23. Why are Java Streams called lazy?
24. What are intermediate and terminal operations in Streams?
25. What is the difference between `findFirst()` and `findAny()`?
26. Give an example where `findFirst()` and `findAny()` can produce different results.
27. How does `findAny()` behave with parallel streams?
28. How are lambda expressions different from anonymous inner classes?
29. Give an example of an anonymous inner class you have used.

---

# 3. Design Patterns / OOP

30. Explain the Singleton design pattern.
31. Why would you use a Singleton?
32. Give a real-world example where Singleton could be useful.
33. How would you implement a thread-safe Singleton?
34. Explain the double-checked locking approach for Singleton.
35. Why do we need the second null check inside the synchronized block?
36. Why can't we simply use one `if` check inside the synchronized block?
37. What is the Facade design pattern?
38. Have you used any structural design patterns in your project?
39. What is the difference between creational, structural, and behavioral design patterns?
40. What is the difference between composition and inheritance?
41. Explain "has-a" vs "is-a" relationships.
42. When would you choose composition over inheritance?
43. What are the coupling implications of composition and inheritance?

---

# 4. Spring / Spring Boot

44. Do you have experience with Spring Core / IoC containers?
45. Have you customized the Spring bean lifecycle?
46. What are the different stages of the Spring Bean lifecycle?
47. What is `@PostConstruct`?
48. What is `@PreDestroy`?
49. What are BeanPostProcessors?
50. How can you customize bean creation/initialization in Spring?
51. Explain the Proxy design pattern.
52. How does Spring use proxies internally?
53. How does `@Transactional` use a proxy?
54. How do interceptors and filters relate to proxy-based behavior?
55. What is the purpose of the `@Qualifier` annotation?
56. What problem does `@Qualifier` solve when multiple beans implement the same interface?
57. What is the difference between `@Qualifier` and `@Primary`?
58. What are Spring Boot Actuators?
59. What are some common Actuator endpoints?
60. How would you customize the Actuator `/health` endpoint to expose additional health information?

---

# 5. Multithreading / Concurrency

61. What experience do you have with multithreading?
62. Have you worked with Executors, Futures, or concurrency APIs?
63. How do you decide the number of threads in an Executor thread pool?
64. How would you determine the appropriate thread-pool size for a workload?
65. What factors should be considered when choosing the number of threads?
66. How does a fixed thread pool work?

---

# 6. Unit Testing / JUnit / AI

67. Have you written JUnit test cases yourself?
68. Are you currently using AI tools to generate JUnit test cases?
69. How would you prompt an AI to generate unit tests for a newly implemented class or method?
70. What information would you provide to an AI so it understands the business logic and expected behavior?
71. How do you use existing code/methods as context when asking AI to generate tests?

---

# 7. Database / SQL

72. How deep is your database experience?
73. Have you worked only on query optimization, or do you understand database internals and transactions?
74. Have you worked with relational and NoSQL databases?
75. Explain the different types of SQL JOINs.
76. What is an INNER JOIN?
77. What is a LEFT JOIN / LEFT OUTER JOIN?
78. What is the difference between INNER JOIN and LEFT JOIN?
79. What other joins do you know?
80. What is a RIGHT JOIN?
81. What is a SELF JOIN?

---

# 8. REST API / System Design

82. Describe a design problem you solved while developing a REST endpoint.
83. What is your thought process when designing a production-grade REST API?
84. How do you handle authentication and authorization for REST APIs?
85. How do you validate REST API inputs?
86. Suppose a duplicate request arrives at your server. How would you handle it?
87. What is an idempotency key?
88. How can an idempotency key prevent duplicate processing?
89. What are the trade-offs of checking for duplicate requests in application code?
90. How would you design a REST API to handle duplicate/retried requests safely?

---

# 9. AWS / Cloud

91. How deep is your AWS knowledge?
92. What AWS services have you worked with?
93. What is AWS Lambda?
94. How is Lambda different from traditional compute such as EC2?
95. What are the alternatives to Lambda for running compute workloads?
96. What does serverless computing mean?
97. What responsibilities are handled by AWS when using Lambda?
98. What are the advantages of Lambda compared with EC2?
99. What are the disadvantages/limitations of Lambda?
100. How much control do you have over the underlying compute resources in Lambda?
101. What AWS knowledge do you have around S3?
102. How have you used Jenkins/AWS for deployment?

---

# 10. AI / GenAI / Agentic AI

103. Which AI coding tools have you worked with?
104. Have you used GitHub Copilot?
105. Have you used Amazon Q?
106. What is your experience with GenAI?
107. Have you worked with agentic AI / AI agents / agentic workflows?
108. What do you mean by a coding agent?
109. Have you worked with AI agents beyond simply using AI for coding?

---

# Additional practical questions / exercises from the screenshots

These are **very important** because the screenshots explicitly say these were particularly important hands-on exercises.

## Coding Problem 1 — String Pattern Search

> Given a text string and a pattern string, determine whether the pattern exists in the text. If it exists, return its starting index.

Follow-ups:

1. What approach would you use?
2. What happens if `text` is `null`?
3. What happens if `pattern` is `null`?
4. What happens if the pattern is empty?
5. What if `pattern.length() > text.length()`?
6. What is the time complexity?
7. Why did you say it was `O(n²)`?
8. Can this solution be optimized?
9. Why are you using:
   ```java
   text.charAt(i + j)
   ```
   instead of:
   ```java
   text.charAt(j)
   pattern.charAt(i)
   ```

---

# Coding Problem 2 — Reverse a Stack

Given a stack containing:

```text
Apple
Banana
Carrot
```

reverse it to:

```text
Carrot
Banana
Apple
```

Follow-ups:

1. How do you reverse a stack?
2. What operations does a Stack provide?
3. Can you use a Queue to reverse the Stack?
4. Explain the difference between Stack and Queue.
5. What is FIFO?
6. What is LIFO?
7. Which Queue implementation would you use in Java?
8. Is Queue an interface or concrete class?
9. How do you instantiate a Queue?
10. What method do you use to insert an element into a Queue?
11. Difference/use of:
   - `push()`
   - `pop()`
   - `offer()`
   - `poll()`
12. Implement the solution in Java.

---

# Coding Problem 3 — Second-Last Non-Repeating Character

> Given a string, find the second-last non-repeating character using Java 8 Streams.

Follow-ups:

1. What is a non-repeating character?
2. How would you determine character frequency?
3. How would you create a frequency map?
4. How would you find non-repeating characters?
5. How would you find the second-last one?
6. Can you solve it using Java 8 Streams?
7. What syntax would you use for converting the String into a Stream?
8. How would you use a HashMap for frequency counting?
9. How would you traverse the String from the end?
10. What happens if there is another unique character added at the end?
11. How do you stop once you've found the second unique character?

And the screenshot explicitly notes:

> **Java 8 skills need to be validated and Streams should be used.**

---

# Practical Exercise — Reverse Array In-Place

Input:

```text
1 2 3 4
```

Expected:

```text
4 3 2 1
```

Requirements:

- No extra array
- Iterate roughly `n/2`
- Follow-up: swap without temporary variable
- Explain complexity

---

# Practical Exercise — Reverse ArrayList

Input:

```text
1 2 3 4 5 6
```

Expected:

```text
6 5 4 3 2 1
```

You were expected to:

- implement the code
- execute it
- debug an index-related error

---

# Practical Exercise — Java Stream

Tasks:

1. Generate numbers from `1` to `100` / `n`
2. Filter even numbers
3. Calculate their sum
4. Follow-up: `range()` vs `rangeClosed()`
5. Other Stream concepts

---

# System Design — Rate Limiter

Question:

> **Design a rate limiter that allows 100 requests per minute from a specific IP.**

Requirements:

- 100 requests/minute
- Request #101 should be rejected

This can lead into:

- fixed window
- sliding window
- token bucket
- leaky bucket
- Redis
- distributed rate limiting
- atomic operations
- race conditions
- horizontal scaling

---

# Separate AI / AI-Native Development screenshot

There is also another AI-specific question set in the screenshots:

1. How would you rate your AI nativeness skill/level?
2. How good are you with AI-native development?
3. Have you worked with agents / agentic PDLC?
4. Have you deployed a feature end-to-end using AI?
5. Explain your experience with Agentic AI.
6. How did you integrate:
   - Gemini AI
   - BigQuery
   - NLP model
   - Python API
   - GPT
   - Angular
   - Java backend
7. Do you use AI in your day-to-day development?
8. Which AI coding/development tools have you used?
   - GitHub Copilot
   - Gemini
   - Claude / other AI tools
   - Codex
9. How do you use AI during architecture and coding?

---

# What this tells us

The **109 numbered questions are not 109 independent topics**.

They collapse into roughly these **12 interview domains**:

```text
1. Core Java
2. Collections
3. Java 8 / Streams
4. OOP / Design Patterns
5. Spring Core / Spring Boot
6. Concurrency
7. JUnit / Testing
8. SQL / PostgreSQL / DB Internals
9. REST / Security / API Design
10. Microservices / Kafka
11. System Design
12. AWS / AI
```

And there are some **very clear high-priority signals**:

### 🔴 Highest priority

**Java + Collections + Streams**

**Spring Core / Spring Boot internals**

**Concurrency**

**REST + Idempotency**

**SQL + DB internals**

**Microservices**

**System design**

**Coding implementation**

### 🟠 Very important

**Spring Security / JWT / OAuth2 / Keycloak**

**Kafka**

**Design patterns**

**Testing**

**Pagination**

**Kubernetes / Ingress / Load balancing / Autoscaling**

### 🟡 Later

**AWS**

**AI / GenAI / Agentic AI**

---

And this is exactly why I would **not start randomly with HashMap anymore**. Now that we have the complete question bank, we can build the preparation around **the actual interview dependency graph** and make sure every EPAM question is covered while simultaneously preparing the Sitcom topics.

If you want, **next I can turn all of these into a single prioritized checklist — 🔴 Must Know / 🟠 Should Know / 🟡 If Time — and map exactly what we should study tonight, Friday morning, Saturday, Sunday, Monday, and Tuesday.**