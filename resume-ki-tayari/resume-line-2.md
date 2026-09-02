Yes. **Let's capture the interview-ready material first and stop drilling for a moment.** That's a much better approach.

I'll maintain this structure for every resume bullet:

* ✅ **Question**
* 🎯 **Answer you should give**
* ⚠️ **What you need to verify/clear from your project**
* 🔥 **Possible follow-up**
* 📌 **Concepts to revise**

For the **current bullet**, here's everything we've established so far.

---

# Resume Bullet 2 — Async Contribution Processing

> **"Re-architected contribution processing from synchronous to asynchronous event-driven processing; improved backend performance through query optimization, indexing, concurrency tuning, and memory/object-creation optimizations."**

---

## Q1. Explain the contribution flow and why you changed it from synchronous to asynchronous.

### 🎯 Answer

> "PF acts as an orchestrator, and each business flow is divided into multiple states. The state completion is persisted in our process context so that the workflow can progress through the different stages.
>
> Initially, during contribution processing, PF synchronously called Transaction Manager. TM was responsible for transaction processing and internally communicated asynchronously with Customer Ledger, which is the source of truth for the actual financial transaction.
>
> Because PF was synchronously waiting for TM even though TM's downstream processing was asynchronous, this created unnecessary waiting and became a throughput bottleneck. During performance validation, we were seeing around 30–40 TPS with 25 concurrent users.
>
> We therefore changed the PF-to-TM communication to an asynchronous event-based flow. PF persists the contribution as processing and publishes an event to TM. TM processes the request independently and eventually sends a business acknowledgement back to PF. PF then updates the workflow state based on that acknowledgement."

### ⚠️ Verify

* Exact TPS test conditions.
* Was **30–40 TPS** measured specifically for contribution?
* Exact Kafka/topic mechanism.
* Exact business ACK mechanism.
* Exact final state names.
* Whether Customer Ledger is officially the "source of truth" in your architecture.

---

# Q2. Why was Transaction Manager a separate microservice?

### 🎯 Answer

> "PF is an orchestrator, so its responsibility is to coordinate the business workflow rather than own transaction-processing logic. Transaction processing is also required by other platform solutions, so we wanted a reusable capability with a clearly defined boundary.
>
> Transaction Manager owns the transaction-related processing and coordinates the downstream components required to complete a credit or debit. This avoids duplicating transaction logic across different business-domain orchestrators and allows that capability to evolve independently."

### ⚠️ Don't say

❌ "TM is just updating the ledger."

Say:

> **"TM owns the transaction-processing capability and coordinates the downstream transaction operations."**

---

# Q3. How does PF know whether the asynchronous contribution succeeded?

### 🎯 Answer

> "PF doesn't block waiting for the contribution to complete. It initially persists the request with a processing status and publishes the event to Transaction Manager. Once TM completes its downstream processing, it sends a business acknowledgement. PF consumes that acknowledgement and updates the persisted workflow state to success. If the transaction fails, the failure is handled through the rejection-management flow."

### ⚠️ Important wording

Don't say:

❌ "PF waits for the ACK."

Say:

✅ **"PF continues asynchronously and reacts when the business acknowledgement arrives."**

---

# Q4. What is the difference between Kafka acknowledgement and your business acknowledgement?

### 🎯 Answer

> "They are different concepts. A Kafka producer acknowledgement is related to the broker acknowledging the successful write of a record. Our business acknowledgement is an application-level response indicating that the downstream business processing has completed successfully or failed."

### 📌 Remember

```text
Kafka ACK
    ↓
Was my message successfully written to Kafka?

Business ACK
    ↓
Did the business operation actually complete?
```

This distinction is **very important**.

---

# Q5. How did you handle duplicate contribution requests?

### 🎯 Answer

> "We identify a contribution using employee code, period and external reference number. For a given employee and period, the external reference number identifies the particular contribution request. Before processing, we check whether the corresponding request already exists in our boundary. If it is already being processed, we don't initiate the same processing again and return the processing status."

### ⚠️ 🚨 VERIFY THIS TOMORROW

The interviewer can ask:

> **"What happens if two identical requests arrive at exactly the same time?"**

Your current answer:

> "Whoever gets the database first."

is **not enough**.

You need to check your actual implementation.

Find out whether you have:

* unique constraint?
* unique index?
* transaction/locking?
* idempotency key?
* some application-level mechanism?

### 🔴 Mark this as:

**PROJECT DETAIL TO VERIFY**

---

# Q6. What happens if TM fails?

### 🎯 Current answer

> "If TM cannot complete the transaction, it sends a failure/business acknowledgement through the failure-handling flow. PF does not mark the contribution as successful. The failure is handled through our rejection-management component, which processes rejected requests through the appropriate failure/reprocessing mechanism."

### ⚠️ 🚨 VERIFY

This is **not fully clear yet**.

You need to determine:

```text
TM failure
   ↓
What exactly?
   ↓
Retry?
   ↓
DLQ?
   ↓
Rejection Management?
   ↓
Compensation?
   ↓
PF failure state?
```

We **will not invent this**.

You already recognized this confusion yourself.

---

# Q7. What is Saga/compensation doing in TM?

### 🎯 Safe answer for now

> "Transaction Manager uses a Saga-style approach for coordinating distributed transaction processing. Since the downstream operations can belong to different services, we cannot rely on one database transaction to make the entire workflow atomic. If a later operation fails after an earlier operation has succeeded, a compensating business operation can be used to counteract the earlier operation."

### ⚠️ 🚨 VERIFY

We still need to confirm your **actual TM implementation**:

* Which operations are Saga steps?
* Where is compensation triggered?
* What exactly is compensated?
* When is retry attempted?
* How many retries?
* Which failures are retryable?
* Which failures go to rejection?
* Where is the transaction state stored?

**Don't claim details until you've checked.**

---

# Q8. Is compensation the same as retry?

### 🎯 Answer

> "No. Retry attempts to execute the failed operation again, usually when the failure is considered transient. Compensation is a separate business operation used to counteract a previously successful operation when the overall distributed workflow cannot be completed."

Example:

```text
A succeeds
B succeeds
C fails

Retry:
C → try again

Compensation:
A → compensating action
B → compensating action
```

### 📌 Important

**Retry ≠ Compensation**

**Eventual consistency ≠ Compensation**

They can work together, but they're different concepts.

---

# Q9. Why didn't you simply use `@Async`?

🔥 **Prepare this.**

### 🎯 Answer

> "`@Async` would move execution to another thread within the application, but it doesn't provide the same decoupling and durability characteristics as event-driven communication. In our case, we wanted PF and Transaction Manager to be independently processing components, so an event-based approach was more appropriate."

Then interviewer may ask:

> **"So when would you use `@Async`?"**

Answer:

> "For asynchronous work that remains within the application boundary where I don't need durable messaging or independent service-level processing. For distributed workflows where reliability, decoupling and independent scaling are important, I'd consider a messaging system."

### 📌 Revise

* `@Async`
* TaskExecutor
* thread pool
* Kafka
* asynchronous messaging

---

# Q10. Why not WebClient instead of synchronous REST?

This is exactly the kind of Spring question you mentioned.

### 🎯 Answer

> "WebClient provides non-blocking HTTP communication, so it can reduce the thread-blocking problem associated with synchronous clients. But it doesn't fundamentally turn the interaction into a durable event-driven workflow. In our case, we wanted PF to decouple itself from TM and allow TM to process independently, so messaging was more suitable."

### 📌 Revise

| Approach     | Main idea                      |
| ------------ | ------------------------------ |
| RestTemplate | Blocking HTTP                  |
| WebClient    | Non-blocking HTTP              |
| `@Async`     | Async execution/thread         |
| Kafka        | Asynchronous durable messaging |

And **verify what your project actually used** before saying "RestTemplate."

---

# Q11. Why did synchronous processing become a bottleneck?

### 🎯 Answer

> "Because the PF request thread remained occupied while waiting for the Transaction Manager response. With increasing concurrency, more threads were blocked waiting for downstream processing, which limited throughput. Our performance testing showed around 30–40 TPS with 25 concurrent users, so the synchronous dependency became a bottleneck."

### 📌 Concepts

* Thread pool
* blocking I/O
* concurrency
* throughput
* latency
* TPS
* backpressure

---

# Q12. What exactly did you improve for performance?

Your resume says:

> query optimization, indexing, concurrency tuning, memory/object-creation optimizations.

So prepare **one real example for each**.

### N+1

> "We identified two places where N+1 queries were being executed. Instead of triggering additional database queries for each parent record, we changed the data-fetching approach to reduce unnecessary database round trips."

⚠️ **VERIFY THE EXACT FIX.**

---

### ObjectMapper

> "We found multiple places where ObjectMapper instances were being created repeatedly. We centralized ObjectMapper as a Spring-managed singleton bean and reused the same instance instead of repeatedly creating objects."

⚠️ Don't exaggerate this as:

> "ObjectMapper was leaking memory."

You don't have evidence of that from what you've told me.

---

### Indexing

You need to give me **one actual index you changed**.

Tomorrow check:

```text
Which table?
Which column(s)?
What query?
What was the problem?
What index did you add/change?
What improvement did you observe?
```

---

### Concurrency tuning

Again, verify:

* What thread pool?
* What component?
* Core threads?
* Max threads?
* Queue?
* Why those numbers?
* What was the bottleneck?

---

# Q13. How did you identify performance problems?

### 🎯 Answer

> "This happened during release validation. The application was passed to the performance team for code-level and performance analysis. They identified issues including N+1 queries and repeated ObjectMapper creation. I investigated those findings, implemented the fixes, and validated the changes as part of the release process."

That's completely credible.

Don't claim:

> "I used profiler X"

unless you actually did.

---

# Q14. Tell me about a performance issue you solved.

### 🎯 Best story

Use **N+1**.

Structure:

```text
Problem
 ↓
Performance validation
 ↓
N+1 identified
 ↓
Investigated query pattern
 ↓
Changed fetching/query strategy
 ↓
Reduced DB round trips
 ↓
Validated performance
```

We'll fill the **exact fix + numbers** tomorrow.

---

# 🟡 Questions you DON'T need to prepare deeply

For this bullet, don't get lost in:

* JVM GC internals
* advanced reactive programming
* Kafka internals
* distributed consensus
* advanced database internals

If interviewer goes there, we'll handle it.

---

# 🔴 Your "VERIFY TOMORROW" list for this bullet

Put this in your notes exactly:

```text
ASYNC CONTRIBUTION — PROJECT DETAILS TO VERIFY

[ ] Exact Kafka topic/event name
[ ] Exact business ACK mechanism
[ ] Exact contribution states
[ ] Exact status values
[ ] How concurrent duplicate requests are prevented
[ ] Unique constraint/index?
[ ] Exact TM retry mechanism
[ ] Retry count
[ ] Retry location
[ ] What failures go to DLQ
[ ] What failures go to rejection management
[ ] Exact compensation mechanism
[ ] What gets compensated
[ ] Exact PF → TM communication implementation
[ ] Was it RestTemplate/WebClient/other?
[ ] Exact 30–40 TPS test conditions
[ ] 25 concurrent users — confirm
[ ] Exact N+1 fix
[ ] Exact indexing fix
[ ] Exact concurrency tuning
[ ] Exact ObjectMapper change
[ ] Any measurable before/after performance numbers
```

---

# ⭐ Your cheat sheet for this bullet

If you remember nothing else, remember this:

```text
OLD:

PF
 |
 | synchronous
 v
TM
 |
 | asynchronous
 v
Customer Ledger


PROBLEM:

PF thread waits
      ↓
unnecessary blocking
      ↓
30–40 TPS @ 25 concurrent users


NEW:

PF
 |
 | event
 v
TM
 |
 | async
 v
Customer Ledger

        ...
        ↓

TM
 |
 | business ACK
 v
PF
 |
 v
Update workflow state
```

And the key sentence:

> **"The important realization was that PF was synchronously waiting for a service whose own downstream processing was already asynchronous, so we removed that unnecessary synchronous dependency and made PF-to-TM communication event-driven."**

That's the heart of the story.

---

## And yes — this is now our process.

**We capture → mark unknowns → move on.**

We are **not going to spend another hour on this bullet**.

Tomorrow, once you verify those red items, we'll come back for **10–15 minutes**, fill the gaps, and move to the next resume bullet.

### Next bullet:

> **"Personally resolved 15+ performance and integration issues in 3 days..."**

We'll turn that into **N+1 + ObjectMapper + indexing + performance debugging questions**, then move immediately to your **10M-record DuckDB/Parquet POC**.

That's how we're going to get through the entire resume without losing the day.
