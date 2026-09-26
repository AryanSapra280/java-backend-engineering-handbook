# 🚀 Spring Batch — Part 2: Performance & Parallel Processing

Now we move into the **senior-level Spring Batch questions**. These are especially useful for your experience.

---

## 26. `JdbcCursorItemReader` vs `JdbcPagingItemReader`

This is a very common interview question.

### `JdbcCursorItemReader`

Uses a JDBC cursor:

```text
Database
   ↓
Cursor
   ↓
Reader
   ↓
Process
```

It keeps a database cursor open while reading.

Good when:

* You need sequential processing
* Large result set
* Database supports efficient cursor behavior

Potential concern:

> The DB connection/cursor remains open for a long-running step.

---

### `JdbcPagingItemReader`

Reads data page by page.

Conceptually:

```text
SELECT ... LIMIT 1000
        ↓
process
        ↓
SELECT ... LIMIT 1000 OFFSET/KEY
```

It doesn't keep one long-running cursor open.

Good for:

* Large datasets
* Restartability
* Partitioning
* Long-running jobs

### Interview answer

> `JdbcCursorItemReader` reads through a database cursor, while `JdbcPagingItemReader` retrieves data in pages. For very large batch jobs, paging can be attractive because it avoids keeping a long-lived cursor open and works well with partitioning.

---

# 27. What about `JpaPagingItemReader`?

Spring Batch also provides:

```java
JpaPagingItemReader<T>
```

It reads entities page by page.

Conceptually:

```text
Page 1
 ↓
Process
 ↓
Clear persistence context
 ↓
Page 2
 ↓
Process
```

This is important because you don't want millions of managed JPA entities accumulating in the persistence context.

### Interview point

For **very large ETL workloads**, JDBC readers are often preferred over JPA because they avoid some ORM overhead.

---

# 28. Why can JPA be slower for huge batch processing?

JPA introduces ORM overhead:

```text
DB row
 ↓
Entity creation
 ↓
Persistence context
 ↓
Dirty checking
 ↓
Hibernate processing
```

For a pure ETL job where you simply need to transform rows:

```text
DB
 ↓
transform
 ↓
DB/file
```

you may not need full entity management.

So:

> For very large bulk-processing workloads, JDBC-based readers/writers can often be more efficient than JPA because they reduce ORM and persistence-context overhead.

---

# 29. What is a multi-threaded Step?

Instead of:

```text
Thread 1:
read → process → write
read → process → write
...
```

you can use multiple threads.

Conceptually:

```text
Step
 |
 +-- Thread 1
 +-- Thread 2
 +-- Thread 3
 +-- Thread 4
```

Example:

```java
.stepBuilder
.taskExecutor(taskExecutor)
```

This can improve throughput.

### But!

Not every `ItemReader` is thread-safe.

You need to consider:

```text
Reader thread safety
Writer thread safety
DB connection pool
CPU
DB capacity
Transaction contention
Ordering requirements
```

---

# 30. What is `TaskExecutor` in Spring Batch?

It determines how work is executed concurrently.

Example:

```java
@Bean
TaskExecutor taskExecutor() {

    ThreadPoolTaskExecutor executor =
        new ThreadPoolTaskExecutor();

    executor.setCorePoolSize(8);
    executor.setMaxPoolSize(10);
    executor.setQueueCapacity(20);

    executor.initialize();

    return executor;
}
```

Then:

```text
Step
 ↓
TaskExecutor
 ↓
Multiple worker threads
```

---

# 31. What is partitioning?

This is **very important**.

Partitioning divides the input dataset into independent partitions.

Suppose:

```text
1,000,000 records
```

Create:

```text
Partition 1 → 1–250,000
Partition 2 → 250,001–500,000
Partition 3 → 500,001–750,000
Partition 4 → 750,001–1,000,000
```

Then:

```text
Master
 ├── P1 → Worker
 ├── P2 → Worker
 ├── P3 → Worker
 └── P4 → Worker
```

They can execute in parallel.

---

# 32. Master vs Worker in partitioning

### Master

Responsible for:

```text
Creating partitions
Assigning work
Coordinating execution
```

### Worker

Processes one partition.

```text
Master
   ↓
Partition 1 → Worker
Partition 2 → Worker
Partition 3 → Worker
```

---

# 33. How does a partition know what data to process?

Using a `Partitioner`.

Example:

```java
public Map<String, ExecutionContext>
partition(int gridSize) {

    Map<String, ExecutionContext> partitions =
        new HashMap<>();

    // partition-0
    // partition-1
    // partition-2
    // ...

    return partitions;
}
```

Each `ExecutionContext` can contain:

```text
minId
maxId
partitionId
```

For example:

```text
partition-0
minId = 1
maxId = 250000
```

Worker uses those values to query its data.

---

# 34. Range partitioning

One common strategy:

```text
minId → maxId
```

Example:

```text
1–100k
100k–200k
200k–300k
```

Worker query:

```sql
SELECT *
FROM account
WHERE id BETWEEN :minId AND :maxId;
```

### Problem

If IDs are sparse:

```text
1
100
10000
500000
900000
```

ranges may be uneven.

You could end up with:

```text
Partition 1 → 5 records
Partition 2 → 500,000 records
```

That's **data skew**.

---

# 35. Hash partitioning

Another strategy:

```sql
WHERE MOD(id, 4) = 0
```

Workers:

```text
Worker 1 → id % 4 = 0
Worker 2 → id % 4 = 1
Worker 3 → id % 4 = 2
Worker 4 → id % 4 = 3
```

Can distribute records more evenly.

But it may have implications for index usage and query performance depending on the database/query.

---

# 36. Pagination-based partitioning

Instead of ID ranges, partition by pages or key ranges.

For example:

```text
Partition 1 → first key range
Partition 2 → second key range
...
```

For very large datasets, **keyset/range-based pagination** is often preferable to huge OFFSET values.

---

# 37. Partitioning vs Multi-threaded Step

Very common question.

### Multi-threaded Step

One step:

```text
Step
 ↓
multiple threads
```

### Partitioning

Dataset is explicitly divided:

```text
Master
 ↓
Partitions
 ↓
Workers
```

Partitioning gives you a clear unit of work per partition and can scale beyond a single thread/step execution model.

---

# 38. Local partitioning vs Remote partitioning

### Local partitioning

Master and workers are inside the same application/runtime.

```text
Application
 |
 +-- Master
 +-- Worker 1
 +-- Worker 2
 +-- Worker 3
```

### Remote partitioning

Workers can run separately, potentially in different processes/pods.

```text
Master
  |
  +---- messaging ----> Worker Pod 1
  |
  +---- messaging ----> Worker Pod 2
  |
  +---- messaging ----> Worker Pod 3
```

This is useful when you need horizontal scaling.

---

# 39. What is remote chunking?

Don't confuse it with partitioning.

### Remote chunking

One master reads and processes chunks, then sends chunks to workers for writing.

```text
Master
  |
  | read/process
  ↓
Chunk
  |
  | messaging
  ↓
Workers
  ↓
Write
```

### Remote partitioning

Master divides the **input workload** into partitions.

```text
Master
 ├── Partition 1 → Worker
 ├── Partition 2 → Worker
 └── Partition 3 → Worker
```

### Easy distinction

```text
Remote chunking
→ distribute chunks

Remote partitioning
→ distribute partitions/work ranges
```

---

# 40. When would you use partitioning?

Suppose:

```text
800 million ledger records
```

One worker cannot efficiently process everything.

You can create:

```text
Partition 1 → account range A
Partition 2 → account range B
Partition 3 → account range C
...
```

and process them concurrently.

But the number of partitions should be based on:

```text
CPU
DB capacity
I/O
memory
worker count
data distribution
```

Don't simply create thousands of workers because the dataset is large.

---

# 41. How would you process 100 million records efficiently?

This is a great senior interview scenario.

I would answer:

> "I would first avoid loading everything into memory. I'd use chunk-oriented processing with a paging or cursor-based reader, choose an appropriate chunk size, use JDBC for high-throughput ETL where ORM isn't required, and add partitioning if a single worker isn't sufficient. I'd also tune database indexes, connection pools and batch writes, and measure throughput before increasing concurrency."

Then mention:

```text
1. Paging/cursor
2. Chunk processing
3. JDBC where appropriate
4. Partitioning
5. Parallel workers
6. Batch DB writes
7. Proper indexes
8. Connection pool tuning
9. Monitoring
10. Restartability
```

That's a **very strong answer**.

---

# 42. What happens if one partition fails?

Suppose:

```text
P1 → SUCCESS
P2 → SUCCESS
P3 → FAILED
P4 → SUCCESS
```

Spring Batch metadata tracks execution state.

You can restart the job and handle the failed partition according to the job's restart configuration.

The important point:

> Partitioning provides independent units of work, making failures easier to isolate and recover from.

---

# 43. How do you avoid duplicate processing after restart?

You need **idempotency**.

Suppose:

```text
Record 100
```

was written successfully but the application crashed before metadata was fully updated.

On restart, it may be processed again.

If the operation isn't idempotent:

```text
100 → inserted
100 → inserted again
```

Potential duplicate.

Solutions include:

```text
Unique constraints
Upsert
Processed-status tracking
Idempotent writes
Business keys
Transactional design
```

This is a very important **production-level answer**.

---

# 44. How do you tune Spring Batch performance?

Think in layers.

### Reader

```text
Paging size
Fetch size
SQL efficiency
Indexes
```

### Processor

```text
CPU-heavy?
External API calls?
Expensive transformations?
```

### Writer

```text
JDBC batch size
Bulk inserts
Prepared statements
```

### Concurrency

```text
Thread count
Partitions
Worker pods
```

### Database

```text
Indexes
Connection pool
Lock contention
Query plans
```

### Memory

```text
Chunk size
Persistence context
Object allocation
```

### Monitoring

```text
Throughput
Failure rate
DB CPU
Connection usage
GC
```

---

# 45. Should we always increase chunk size?

**No.**

Larger chunk:

```text
+ fewer commits
+ potentially higher throughput
```

But:

```text
- more memory
- larger transaction
- more rollback work
- longer transaction locks
```

Smaller chunk:

```text
+ smaller transactions
+ lower memory
+ easier recovery
```

but:

```text
- more commits
- potentially more overhead
```

So chunk size is a **tuning parameter**, not something to maximize blindly.

---

# 46. What if DB is the bottleneck?

Don't just increase threads.

If database CPU is already:

```text
95%
```

and you increase:

```text
4 workers → 20 workers
```

you may make things worse.

You should investigate:

```text
SQL execution plan
Indexes
DB CPU
Locks
Connection pool
I/O
Batch size
Query frequency
```

### Strong interview line:

> "Concurrency should be increased only up to the point where the downstream bottleneck can handle it."

🔥 That's a good senior-level statement.

---

# 47. Spring Batch architecture for large-scale processing

A good architecture:

```text
                 Job Launcher
                      |
                      v
                  Job/Manager
                      |
                Partitioning
          ┌───────────┼───────────┐
          ↓           ↓           ↓
       Worker 1    Worker 2    Worker 3
          |           |           |
       Reader      Reader      Reader
          ↓           ↓           ↓
      Processor    Processor    Processor
          ↓           ↓           ↓
       Writer      Writer      Writer
          |           |           |
          └────────── DB ─────────┘
```

Metadata:

```text
             JobRepository
                  |
        ┌─────────┴─────────┐
        ↓                   ↓
JobExecution          StepExecution
```

---

# 🔥 Senior Spring Batch rapid-fire

| Question                    | Answer                                                        |
| --------------------------- | ------------------------------------------------------------- |
| Cursor reader?              | Reads using DB cursor                                         |
| Paging reader?              | Reads data page by page                                       |
| JPA reader?                 | Reads entities through JPA                                    |
| JDBC vs JPA for huge ETL?   | JDBC often reduces ORM overhead                               |
| Partitioning?               | Divide workload into independent partitions                   |
| Partitioner?                | Creates partition execution contexts                          |
| Master?                     | Creates/co-ordinates partitions                               |
| Worker?                     | Processes assigned partition                                  |
| Local partitioning?         | Workers in same runtime                                       |
| Remote partitioning?        | Workers can run separately                                    |
| Remote chunking?            | Master sends chunks to workers                                |
| Multi-threaded step?        | Multiple threads execute work within a step                   |
| Retry?                      | Repeat transient failure                                      |
| Skip?                       | Continue past configured bad records                          |
| Chunk size?                 | Number of items processed per chunk/transaction boundary      |
| JobRepository?              | Batch metadata                                                |
| ExecutionContext?           | Restartable execution state                                   |
| Idempotency?                | Repeating operation doesn't cause incorrect duplicate effects |
| Bigger chunk always better? | No                                                            |
| More threads always better? | No                                                            |

---

# 🎯 One scenario to memorize

### Interviewer:

> "You have 100 million records and your Spring Batch job is too slow. What will you do?"

### Answer:

> "First I would identify the bottleneck rather than immediately adding threads. I'd check the reader SQL and execution plan, indexes, DB CPU and I/O, connection pool usage, and writer throughput. I'd avoid loading everything into memory and use paging or cursor-based reading with chunk processing. For high-throughput ETL, I'd consider JDBC rather than JPA where entity management isn't required. If one worker is still insufficient, I'd partition the dataset and process partitions concurrently. Finally I'd tune chunk size, batch writes and concurrency based on measured throughput, while preserving restartability and idempotent writes."

**That's exactly the kind of structured answer you want in a senior interview.**

---

## ✅ Spring Batch DONE

Next we move to:

# 🗄️ SQL + PostgreSQL

We'll do:

1. Joins
2. `GROUP BY` / `HAVING`
3. Subqueries
4. Window functions
5. Indexes
6. B-tree vs other indexes
7. Composite indexes
8. `EXPLAIN ANALYZE`
9. Transactions
10. Isolation levels
11. Locks/deadlocks
12. Query optimization
13. Pagination
14. Normalization
15. PostgreSQL-specific interview questions.
