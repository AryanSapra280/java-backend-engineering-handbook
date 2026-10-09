Absolutely. I now have **both the Round 3 question-bank screenshots and your actual resume**, so we can prepare this much more intelligently.

Your resume is particularly important because Round 3's questions are clearly designed to go:

> **Basic question → “Have you used it?” → “Where?” → “How did you implement it?” → “Why did you choose it?” → “What went wrong?” → “How would you improve it?”**

And your resume has several areas where they can drill very deeply — especially **Kafka, PostgreSQL, Spring Batch, large-scale processing, Kubernetes, microservices, concurrency, performance optimization, Clojure integration, Cassandra, and your AI hackathon**. latest_resume

---

# PART 1 — QUESTIONS FROM THE IMAGE

I'm preserving the question-bank structure rather than replacing it with my own syllabus.

---

# 1. Java Versions & Features

### Q1–20

1. Which version of Java are you most comfortable with?
2. What Java 8 features have you used?
3. What Java 9+ features have you used?
4. What features were introduced from Java 9 through Java 17?
5. What Java 17 features have you used?
6. What features were introduced after Java 9?
7. What are Virtual Threads?
8. How and when would you use Virtual Threads?
9. What are sealed classes?
10. How do sealed classes work?
11. Can a sealed type be a class or an interface?
12. Who can extend or implement a sealed class/interface?
13. Why would you use sealed classes/interfaces?
14. What are Java records?
15. Have you used records? If yes, where?
16. What Java features are you strongest in up to Java 8?
17. What are lambda expressions?
18. What are functional interfaces?
19. What is the Date & Time API?
20. What are method references?

---

# 2. Java Streams & Lazy Evaluation

### Q21–40

21. What is lazy evaluation?
22. How does lazy evaluation work?
23. How are Streams related to lazy evaluation?
24. What are intermediate and terminal operations in Streams?
25. When is a Stream operation actually executed?
26. What does `findAny()` do?
27. What does `findFirst()` do?
28. What is the difference between `findAny()` and `findFirst()`?
29. If Streams are lazily evaluated, why do we have both `findAny()` and `findFirst()`?
30. In a sequential Stream, will `findAny()` and `findFirst()` always return the same element?
31. Why is `findAny()` not guaranteed to return the first element?
32. How does `findFirst()` guarantee encounter order?
33. How do parallel Streams work?
34. What is the difference between sequential and parallel Streams with respect to the thread model?
35. How does a parallel Stream divide and process data?
36. Does a parallel Stream create one thread per element?
37. What thread pool/mechanism is used by parallel Streams?
38. Is a parallel Stream always faster?
39. What factors determine whether a parallel Stream performs better?
40. When would you use a sequential Stream versus a parallel Stream?

---

# 3. Exception Handling

### Q41–57

41. What concepts are you comfortable with in exception handling?
42. What are checked and unchecked exceptions?
43. What are custom checked and unchecked exceptions?
44. How do `try`, `catch`, and `finally` work?
45. What is mandatory when handling exceptions in Java?
46. Can you use a `try` block without a `catch` block?
47. Can you use a `try` block without both `catch` and `finally`?
48. Can you use only a `try` block?
49. What is the purpose of the `finally` block?
50. How can runtime issues occur even when using Generics?
51. How do you handle exceptions globally?
52. How do you handle exceptions in Spring Boot?
53. What is `@ControllerAdvice`?
54. What is `@RestControllerAdvice`?
55. What is `@ExceptionHandler`?
56. How does `@ExceptionHandler` work?
57. How would you implement centralized/global exception handling in a Spring Boot REST application?

---

# 4. Collections

### Q58–77

58. What Java Collections are you comfortable with?
59. What are the different types of collections you have worked with?
60. When would you use a List, Set, Queue, or Map?
61. What is the use case for a List?
62. What is the use case for a Set?
63. What is the use case for a Queue?
64. What is the use case for a Map?
65. What is the difference and use case between `List` and `ArrayList`?
66. What is the difference and use case between `Set` and `HashSet`?
67. What is the difference and use case between `Map` and `HashMap`?
68. When would you use `ConcurrentHashMap`?
69. If you had to implement a dictionary, which collection would you use?
70. Which Map implementation would you use to implement a dictionary?
71. Why would you use `HashMap` for a dictionary?
72. Does `HashMap` allow duplicate keys?
73. Does `HashMap` allow duplicate values?
74. If `HashMap` allows duplicate values, why is it suitable for a dictionary?
75. Does a dictionary allow duplicate keys?
76. How would you implement a dictionary using `HashMap`?
77. When would you choose `HashMap` versus `ConcurrentHashMap`?

---

# 5. Generics

### Q78–87

78. What are Generics?
79. Why do we need Generics?
80. What are the benefits of Generics?
81. How do Generics provide type safety?
82. Do Generics provide compile-time or runtime safety?
83. If Generics provide compile-time safety, can you still face issues at runtime?
84. How can runtime issues occur when using Generics?
85. Where have you used Generics?
86. What are common use cases for Generics?
87. When should you use Generics?

---

# 6. Multithreading & Concurrency

### Q88–95

88. What is your experience with Java multithreading?
89. What concurrency concepts are you familiar with?
90. How do Java threads work?
91. What is the difference between a process and a thread?
92. What are the different ways to create/manage threads in Java?
93. How does synchronization work in Java?
94. What are the common problems in concurrent programming?
95. How do you handle thread safety in Java?

---

# 7. `this()` and `super()`

### Q96–100

96. What is `this()`?
97. What is `super()`?
98. What is the difference between `this()` and `super()`?
99. Can `this()` and `super()` be used together?
100. What are the restrictions or edge cases when using `this()` and `super()`?

---

# 8. Interfaces

### Q101–105

101. What is an interface?
102. What are abstract methods in an interface?
103. What are default methods?
104. How are default methods used?
105. What is the difference between default, static, and abstract methods in an interface?

---

# 9. Spring Boot

### Q106–116

106. Why did you use Spring Boot?
107. What features does Spring Boot provide?
108. What does Spring Boot provide on top of the Spring Framework?
109. How is Spring Boot different from the Spring Framework?
110. What are the major features of Spring Boot?
111. What is auto-configuration?
112. How does Spring Boot auto-configuration work?
113. What are Spring Boot starter dependencies?
114. What are embedded servers in Spring Boot?
115. What is Spring Boot Actuator?
116. How is Actuator used for production monitoring?

---

# 10. REST API / Spring MVC

### Q117–132

117. Suppose you are creating a REST service. How would you implement it?
118. What annotations would you use while creating a REST API?
119. How would you structure a REST API using Controller, Service, and Repository layers?
120. What is the purpose of `@RestController`?
121. What is the purpose of `@RequestMapping`?
122. What is the difference/use of `@GetMapping`, `@PostMapping`, `@DeleteMapping`, and `@PatchMapping`?
123. How do you accept JSON request data in a REST API?
124. What is `@RequestBody` used for?
125. How do you fetch data from query parameters?
126. How do you handle path variables?
127. How do you implement dependency injection in Spring?
128. What is constructor-based dependency injection?
129. Why is constructor injection preferred?
130. How do you handle exceptions in a REST API?
131. Where would you implement exception handling in a Spring Boot application?
132. How would you implement centralized exception handling?

---

# 11. ORM / JPA / Hibernate

### Q133–150

133. What is ORM?
134. What does ORM stand for?
135. How does ORM map Java objects to database tables?
136. How are JPA and Hibernate related to ORM?
137. What is a Criteria Query?
138. What is the Criteria API?
139. Why would you use a Criteria Query?
140. Why do you need a Criteria Builder?
141. Why not simply write your own query instead of using Criteria Query?
142. In what situations would you use Criteria Query?
143. How does Criteria Query compare with JPQL?
144. How do you build dynamic queries using the Criteria API?
145. Have you used Criteria Queries in a real project?
146. What kind of situation did you use Criteria Query for?
147. What is the N+1 query problem?
148. How does the N+1 query problem occur?
149. How do you resolve the N+1 query problem?
150. What is `JOIN FETCH` and how can it solve the N+1 problem?

---

# 12. Microservices

### Q151–173

151. What microservice patterns are you familiar with?
152. Explain the architecture/patterns you use in microservices.
153. What is an API Gateway?
154. Why do you need an API Gateway?
155. What is Service Discovery?
156. Why is Service Discovery needed?
157. What is a Circuit Breaker?
158. What is the Retry pattern?
159. What is the Timeout pattern?
160. What is the Saga pattern?
161. What is a Config Server?
162. Why is centralized configuration needed in microservices?
163. How do API Gateway, Service Discovery, Circuit Breaker, Retry, Timeout, Saga, and Config Server fit together in a microservice architecture?
164. Have you implemented a complete microservice ecosystem from scratch?
165. Which microservice modules have you developed?
166. Have you created any microservice modules from scratch?
167. What parts of the microservice architecture did you personally implement?
168. How do you manage transactions in microservices?
169. How do you handle distributed transactions?
170. How do microservices communicate with each other?
171. How do you communicate between microservices using REST APIs?
172. How do you communicate asynchronously between microservices?
173. What is your use of Azure Event Hubs in microservice communication?

**Important:** Q173 is in their bank, but your resume specifically says **Kafka and NATS/JetStream**, not Azure Event Hubs. So don't claim Azure Event Hubs experience unless you actually have it. Your resume supports Kafka/NATS much more strongly. latest_resume

---

# 13. Project / Architecture

### Q174–184

174. Can you talk about your current project?
175. What is the business scenario behind your application?
176. What business problem are you solving?
177. What kind of architecture are you using?
178. How does the overall notification flow work?
179. Why are you using Cosmos DB?
180. Why are you using PostgreSQL for Spring Batch metadata?
181. Why are you using Spring Batch?
182. How are failures and retries handled in your application?
183. What is your role and responsibility in the project?
184. Which components/modules did you personally develop?

### ⚠️ Important

Q178–180 appear to be specific to **the source candidate/question bank**, and they don't all match your resume.

For you, we should replace those with your actual architecture:

```text
PF / Retirement platform
PostgreSQL
Kafka
Spring Batch
Redis
NATS/JetStream
DuckDB
Parquet
Clojure integration
```

Your resume specifically states that you own backend development for a retirement/PF platform serving ~300K members and work across contribution, withdrawal, advance, transfer, settlement, interest, ledger, balance and account closure workflows. latest_resume

---

# 14. Spring Batch

### Q185–190

185. What is Spring Batch?
186. Why are you using Spring Batch?
187. What are the main components of Spring Batch?
188. How does a Spring Batch job execute?
189. How do you handle failures and retries in Spring Batch?
190. Why is PostgreSQL being used for Spring Batch metadata?

This section is **extremely important for you**, because your resume explicitly claims Spring Batch and large-scale processing work. latest_resume

---

# 16. Spring Security

### Q213–219

213. What is a Security Filter Chain?
214. How do you implement a Security Filter Chain?
215. How do you protect APIs using Spring Security?
216. How do you configure publicly accessible APIs?
217. How do you configure JWT authentication in the Security Filter Chain?
218. How would you configure Basic Authentication?
219. How does Spring Security authenticate and authorize a request?

---

# 17. Deployment / Kubernetes / Scaling

### Q220–229

220. Have you deployed your microservices?
221. How did you deploy your microservices?
222. How does deployment work in your current project?
223. How are GitHub Actions and Kubernetes used for deployment?
224. How do you scale microservices?
225. What is the difference between scaling up and scaling down?
226. How do you scale up a microservice?
227. How do you scale down a microservice?
228. How do you increase/decrease Kubernetes replicas?
229. How does Kubernetes help in microservice deployment and scaling?

Again, **Q223 specifically mentions GitHub Actions**, whereas your resume lists Docker, Kubernetes and Jenkins. latest_resume

So your answer should be based on what you've actually done, not pretend you've used GitHub Actions.

---

# 18. Coding / Problem Solving

### Q230–244

230. Find the employees who are part of more than one department.
231. How would you solve the above problem using SQL?
232. How would you solve the above problem using Java?
233. Given an array of strings, how would you identify duplicate anagrams?
234. If multiple strings are anagrams, how would you remove duplicates while keeping the first occurrence?
235. If there are no duplicate anagrams, how should they be handled?
236. After removing duplicate anagrams, how would you alphabetically sort the resulting array?
237. How would you generate a common key for anagrams?
238. Why do strings such as `"code"` and another differently ordered version need to be sorted before being considered the same anagram?
239. What does `Set.add()` return?
240. Why wasn't the duplicate being removed in the candidate's implementation?
241. How would you sort a character array?
242. What does `Arrays.sort()` return?
243. How would you fix the issue caused by `Arrays.sort()` returning `void`?
244. What imports are required for using `Arrays`?

---

# 19. Managerial / Technical Manager

### Q245–258

245. Can you talk about your current project and your role in it?
246. What business problem are you solving?
247. What is the business scenario behind your application?
248. What architecture are you using and why?
249. Have you deployed your microservices?
250. How did you scale your microservices?
251. How do you scale up and scale down?
252. How does deployment work in your current project?
253. What is your location preference?
254. Are you comfortable coming to the office five days a week?
255. How is your AI journey?
256. Have you started working on AI?
257. What AI tools are you currently using?
258. Have you worked on creating any AI models?

---

# ⚠️ Notice something important

There appears to be a **Section 15 missing from the screenshots you sent** — numbering jumps from:

```text
14. Spring Batch
190
```

to

```text
16. Spring Security
213
```

So questions **191–212** aren't visible in the images you've sent.

I won't invent those questions.

If you have that missing page, send it and I'll add it exactly.

---

# NOW: YOUR RESUME CHANGES THE PREPARATION COMPLETELY

This is the more important part.

Your resume gives them **very specific ammunition**.

Your professional summary says you have 5 years building Java/Spring Boot/REST/PostgreSQL/Kafka/microservices and that you work with distributed processing, performance optimization, reconciliation and fault handling. latest_resume

That means I would prepare these **additional resume-derived questions**, even if they aren't explicitly in their question bank.

---

# 🔥 RESUME AREA 1 — PF / RETIREMENT PLATFORM

Your resume says:

> ~300K members  
> contribution  
> withdrawal  
> advance  
> transfer  
> settlement  
> interest  
> ledger  
> balance  
> account closure

latest_resume

Expect:

### Business

1. What exactly is the PF/retirement platform?
2. Who are the users?
3. What does one member's lifecycle look like?
4. What is a contribution?
5. What is withdrawal?
6. What is settlement?
7. What is an interest calculation workflow?
8. What is the ledger responsible for?
9. Why do you need a ledger?
10. How do you ensure ledger consistency?

### Architecture

11. How many microservices are involved?
12. How are they divided by business capability?
13. Which service owns which data?
14. How do services communicate?
15. Where is Kafka used?
16. Where is synchronous REST used?
17. Why not make everything synchronous?
18. Where do you use asynchronous processing?
19. What happens if one downstream service fails?
20. How do you reconcile failed transactions?

---

# 🔥 RESUME AREA 2 — KAFKA

This is a **huge one**.

Your resume explicitly says you re-architected contribution processing from synchronous to asynchronous Kafka-based processing. latest_resume

Expect:

### Basic

21. Why Kafka?
22. Kafka vs REST?
23. What is a topic?
24. Partition?
25. Offset?
26. Consumer group?
27. Producer?
28. Consumer?

### Intermediate

29. How does Kafka achieve scalability?
30. How does Kafka maintain ordering?
31. Is Kafka exactly-once?
32. What happens when a consumer crashes?
33. What happens to unprocessed messages?
34. How does offset commit work?
35. What happens if a consumer processes a message but crashes before committing?
36. How do you handle duplicate messages?
37. How do you make consumers idempotent?
38. How do you retry failed messages?
39. DLQ?
40. How do you monitor Kafka lag?

### Senior

41. Why did you move contribution processing from synchronous to asynchronous?
42. What bottleneck existed in the old architecture?
43. What changed after Kafka was introduced?
44. What consistency trade-off did you introduce?
45. How does the caller know processing succeeded?
46. How do you handle a downstream service being unavailable?
47. How do you reconcile missing/failed events?
48. What happens if Kafka is unavailable?
49. How do you guarantee an event isn't lost after DB commit?
50. Would you use the Outbox pattern here?

That entire chain is directly motivated by your resume. latest_resume

---

# 🔥 RESUME AREA 3 — PERFORMANCE OPTIMIZATION

Your resume says:

> PostgreSQL query optimization, indexing, concurrency tuning, memory/object-creation optimizations.

And:

> 15+ performance/integration issues in 3 days.

latest_resume

This is **dangerous in an interview** because if you put "performance optimization" on your resume, they can go extremely deep.

Prepare:

51. Tell me about one performance issue you solved.
52. How did you identify the bottleneck?
53. How did you know it was DB-related?
54. What query was slow?
55. How did you optimize it?
56. Did you add an index?
57. Why that index?
58. How do you determine whether an index is being used?
59. What is `EXPLAIN ANALYZE`?
60. What is a sequential scan?
61. What is an index scan?
62. What is a composite index?
63. Why does column order matter?
64. When might PostgreSQL not use an index?
65. How can an index hurt performance?
66. What is connection pool exhaustion?
67. How can too many application threads hurt DB performance?
68. What memory issue did you encounter?
69. What caused excessive object creation?
70. How did you verify your fix?

**This is one of the areas I'd prioritize heavily.**

---

# 🔥 RESUME AREA 4 — SPRING BATCH / LARGE DATA

Your resume claims:

> Spring Batch + Kafka + bulk processing + DuckDB + Apache Parquet

and a POC:

> **~10 million records in ~90 seconds.**

latest_resume

They can absolutely ask:

71. Why Spring Batch?
72. Chunk processing?
73. Tasklet vs chunk?
74. ItemReader?
75. ItemProcessor?
76. ItemWriter?
77. Job?
78. Step?
79. JobRepository?
80. JobExecution?
81. StepExecution?
82. How do retries work?
83. How does skip work?
84. How do you restart a failed job?
85. How do you partition a batch?
86. How do you decide partition size?
87. How do you avoid loading millions of records into memory?
88. How would you process 800 million records?
89. How would you parallelize it?
90. What happens if one partition fails?
91. How do you reconcile partial completion?
92. Why DuckDB?
93. Why Parquet?
94. Why not CSV?
95. Why not directly process through JPA?
96. How did you achieve ~10 million records / 90 seconds?
97. What was the bottleneck?
98. How much memory did it consume?
99. How many threads?
100. How did you benchmark it?
101. Why NATS?
102. Why not Kafka?
103. When would you generate the Parquet file?
104. How large should each Parquet file be?

This is **probably the single most important resume-specific technical area**.

---

# 🔥 RESUME AREA 5 — NATS / PARQUET / DISTRIBUTED DATA MOVEMENT

Your resume explicitly says:

> partition-based data movement using reference-key staging, Parquet generation and NATS event-driven transfer. latest_resume

Expect:

105. Why staging?
106. Why reference-key staging?
107. Why partition?
108. Why Parquet?
109. Why not send records individually?
110. Why not send JSON?
111. Why not Kafka?
112. Why NATS?
113. What is JetStream?
114. Is NATS durable?
115. How do you handle message failure?
116. What happens if downstream is unavailable?
117. How do you guarantee no data loss?
118. How do you avoid duplicate processing?
119. How do you know all partitions completed?
120. How do you reconcile missing partitions?

---

# 🔥 RESUME AREA 6 — CLOJURE INTEGRATION

This is an **easy place for an interviewer to catch you off guard**, because your resume says:

> Java integration layer for a Clojure-based distribution engine.

latest_resume

Prepare:

121. Why is Clojure being used?
122. What does the distribution engine do?
123. Why did you need a Java integration layer?
124. How does Java communicate with Clojure?
125. What is the pool/account relationship?
126. What is equal-share distribution?
127. What is percentage-based distribution?
128. What is a runtime DAG?
129. Why DAG?
130. How do you validate allocation rules?
131. How do you handle incorrect allocation?
132. How do you test this?

Don't try to become a Clojure expert overnight. But **you must be able to explain exactly what you personally did**.

---

# 🔥 RESUME AREA 7 — DATABASES

Your resume lists:

```text
PostgreSQL
Oracle
Cassandra
```

latest_resume

Expect:

133. PostgreSQL vs Oracle?
134. Why PostgreSQL?
135. PostgreSQL indexes?
136. Transactions?
137. Isolation levels?
138. MVCC?
139. Locks?
140. Deadlocks?
141. Optimistic vs pessimistic locking?
142. PostgreSQL query optimization?
143. Cassandra vs PostgreSQL?
144. Why Cassandra?
145. Cassandra partition key?
146. Cassandra clustering key?
147. Why isn't Cassandra a replacement for PostgreSQL?
148. When would you choose NoSQL?

---

# 🔥 RESUME AREA 8 — ORACLE → POSTGRESQL

This is another **very unique resume bullet**.

You worked on an Oracle-to-PostgreSQL compatibility initiative and a C++ SQL translation layer using Lex/Yacc. latest_resume

They can ask:

149. Why migrate Oracle to PostgreSQL?
150. What compatibility issues did you encounter?
151. Why can't Oracle SQL simply run on PostgreSQL?
152. What is SQL translation?
153. What is Lex?
154. What is Yacc?
155. How does parsing work?
156. What is an AST?
157. How do you transform Oracle syntax?
158. What is `SELECT FOR UPDATE`?
159. How does row locking work?
160. How did you test SQL translation?
161. What happens with unsupported SQL?
162. How did you ensure backward compatibility?

This is a **very strong differentiator on your resume**. Be prepared to explain it confidently.

---

# 🔥 RESUME AREA 9 — KUBERNETES / PRODUCTION

Your resume lists Kubernetes, Docker and Jenkins. latest_resume

Prepare:

163. What is a Pod?
164. Deployment?
165. Service?
166. ConfigMap?
167. Secret?
168. Ingress?
169. Readiness vs liveness?
170. Resource requests vs limits?
171. OOMKilled?
172. How do you scale replicas?
173. HPA?
174. What metrics does HPA use?
175. Rolling deployment?
176. What happens during deployment?
177. How do you troubleshoot a failing Pod?
178. Where do you find logs?
179. How do you inspect CPU/memory?
180. How do you investigate CrashLoopBackOff?

---

# 🔥 RESUME AREA 10 — AI

Your AI hackathon is quite detailed:

> RAG + knowledge graph + code indexing + multi-agent workflow + Ollama + JIRA + RCA.

latest_resume

So Q255–258 are **not going to be generic AI questions for you**.

Prepare:

181. What did you build in the AI hackathon?
182. What was the business problem?
183. Why use RAG?
184. What is RAG?
185. Why a knowledge graph?
186. Why both RAG and knowledge graph?
187. What is an AI agent?
188. What makes your workflow multi-agent?
189. What agents did you have?
190. What was the role of the LLM?
191. Why Ollama?
192. Why local LLM instead of OpenAI/Gemini?
193. How did you retrieve JIRA context?
194. How did you index code?
195. How did the agent identify relevant code?
196. How did you generate confidence scores?
197. How did you prevent hallucinations?
198. How did you evaluate the solution?
199. What did you personally implement?
200. What would you improve if given another month?

Your resume says you specifically contributed to the **RAG and knowledge-graph pipeline**, so be especially strong there. latest_resume

---

# 🔥 RESUME AREA 11 — CHATIFY

Your Chatify project can also generate questions.

Your resume says Java + Spring Boot + WebSockets + REST + STOMP. latest_resume

Prepare:

201. Why WebSockets?
202. REST vs WebSocket?
203. What is STOMP?
204. How does a WebSocket connection work?
205. How do you authenticate WebSocket connections?
206. How are messages routed?
207. What happens if WebSocket disconnects?
208. Why fallback?
209. How do you scale WebSocket servers?
210. How do you handle multiple instances?
211. Do you need Redis/Kafka for distributed WebSocket messaging?
212. How would you design presence/online status?

---

# 🔥 RESUME AREA 12 — LEADERSHIP

Your resume says you:

- mentored 3 developers
- conducted code/design reviews
- participated in 7–8 technical interviews
- collaborated with Product, QA, Security/Performance and implementation teams. latest_resume

So Technical Manager can ask:

213. How do you mentor junior developers?
214. How do you conduct code reviews?
215. What do you look for in a code review?
216. Tell me about a disagreement during code review.
217. How do you handle a developer who repeatedly introduces defects?
218. How do you estimate work?
219. How do you identify risks?
220. How do you handle dependencies?
221. How do you communicate technical risk to Product?
222. How do you prioritize production bugs vs feature work?
223. Tell me about a production issue you owned.
224. Tell me about a time you disagreed with an architect.
225. Tell me about a technical decision you made.
226. Tell me about a failure.
227. What did you learn from it?
228. How do you handle pressure before a release?

---

# 🚨 THE MOST IMPORTANT THING FOR YOUR ROUND 3

Don't treat all these questions equally.

Based on **their question bank + your resume**, I would prioritize:

## Tier 1 — MUST MASTER

```text
★★★★★
Current PF project
Kafka
Microservices
Spring Boot
Spring Batch
PostgreSQL
Performance optimization
Multithreading
REST APIs
JPA/Hibernate
Kubernetes/deployment
Exception handling
Java Collections
Java 8 Streams
```

## Tier 2 — Very likely

```text
★★★★
CompletableFuture
ExecutorService
Circuit Breaker
Retry
Timeout
Saga
Transactions
N+1
Indexes
Concurrency
JWT/Spring Security
Records
Virtual Threads
Docker
Observability
```

## Tier 3 — Resume-specific depth

```text
★★★★
DuckDB
Parquet
NATS/JetStream
Clojure integration
Oracle → PostgreSQL
Lex/Yacc
Cassandra
WebSockets/STOMP
AI/RAG/Knowledge Graph/Ollama
```

## Tier 4 — Quick revision

```text
★★
this()/super()
Interface details
Criteria API
Basic Spring MVC annotations
Set.add()
Arrays.sort()
```

---

# 🎯 And here's how I want us to prepare

For each major question, I will give you:

### 1. Interview answer

Exactly what **you should say**.

### 2. Follow-up questions

The questions the interviewer can derive from your answer.

### 3. Deep answer

So you aren't trapped if they say:

> "Okay, but how exactly?"

### 4. Your resume connection

For example:

> **"You mentioned Kafka in your PF project. Explain exactly why you moved from synchronous to asynchronous processing."**

### 5. Implementation

Actual Java/Spring/Kafka/SQL/Kubernetes examples where appropriate.

### 6. Senior-level trade-offs

Not just:

> "Kafka is asynchronous."

But:

> "What did we gain, what consistency did we lose, and how do we handle failure?"

### 7. Trap questions

The things interviewers use to distinguish someone who memorized the definition from someone who has actually built systems.

---

## And one more thing

Your resume has some **extremely strong interview hooks**.

For example:

> **“15+ performance and integration issues in 3 days”**

> **“10 million records in ~90 seconds”**

> **“Re-architected contribution processing from synchronous to asynchronous Kafka processing”**

> **“C++ SQL translation layer using Lex/Yacc”**

> **“Java integration layer for Clojure distribution engine”**

> **“RAG + knowledge graph + multi-agent debugging assistant”**

These aren't ordinary resume bullets. **A Technical Manager can spend 15–20 minutes drilling into just one of them.** latest_resume

So we should absolutely prepare them as **stories + architecture + implementation + failure + trade-off**, not just as resume descriptions.

### My recommended order for your preparation

```text
1. Current Project / PF Architecture       🔥🔥🔥
2. Kafka + Event Driven Architecture       🔥🔥🔥
3. Spring Batch + 10M/90-sec POC           🔥🔥🔥
4. PostgreSQL + Performance Optimization   🔥🔥🔥
5. Microservices + Resilience               🔥🔥🔥
6. Java Concurrency                         🔥🔥
7. JPA/Hibernate                            🔥🔥
8. Kubernetes/Deployment                    🔥🔥
9. Spring Security                          🔥
10. Java 8/17/21                            🔥
11. Streams                                 🔥
12. AI/RAG/Knowledge Graph                  🔥
13. Clojure/NATS/Parquet/DuckDB             🔥
14. REST/WebSockets/Cassandra                🔥
15. Remaining basic Java questions          Quick
```

**This is the preparation I would use for your Round 3.** It targets both what they explicitly gave you **and what your resume gives them permission to ask.**