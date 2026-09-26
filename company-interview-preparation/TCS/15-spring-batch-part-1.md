# 🚀 Spring Batch — Part 1

This is a **high-value topic for you** because you can connect almost every answer to your ETL/batch work.

---

## 1. What is Spring Batch?

**Interview answer:**

> Spring Batch is a framework for developing robust batch-processing applications that process large volumes of data with features such as chunk processing, transactions, restartability, retry, skip, job metadata, and parallel processing.

Typical use cases:

```text
ETL
Data migration
File processing
Bulk DB updates
Report generation
Large-volume processing
```

---

# 2. What is a Job?

A **Job** represents the complete batch process.

Example:

```text
Job: Process PF Contributions
```

It may contain multiple steps:

```text
Job
 ├── Step 1 → Read input
 ├── Step 2 → Transform data
 └── Step 3 → Write to database
```

---

# 3. What is a Step?

A **Step** represents an independent phase of a batch job.

Example:

```text
Job
 |
 +-- Step 1: Read files
 |
 +-- Step 2: Validate records
 |
 +-- Step 3: Write records
```

A job can contain one or multiple steps.

---

# 4. Job vs Step

Simple interview answer:

> A Job represents the overall batch process, while a Step represents one stage of that process.

```text
Job
 ↓
Step
 ↓
Reader → Processor → Writer
```

---

# 5. What are ItemReader, ItemProcessor and ItemWriter?

This is the **core Spring Batch model**.

```text
ItemReader
     ↓
ItemProcessor
     ↓
ItemWriter
```

### ItemReader

Reads data.

Examples:

```text
Database
CSV
JSON
XML
API
```

### ItemProcessor

Transforms/validates data.

```java
public Output process(Input input)
```

### ItemWriter

Writes processed data.

```text
Database
File
API
Kafka
etc.
```

---

# 6. What is chunk processing?

One of the most important questions.

Suppose we have:

```text
10,000 records
```

Chunk size:

```text
100
```

Spring Batch processes:

```text
Read 100
 ↓
Process 100
 ↓
Write 100
 ↓
Commit
```

Then:

```text
Read next 100
 ↓
Process
 ↓
Write
 ↓
Commit
```

So:

```text
10,000 records
= 100 chunks
```

---

# 7. Why use chunk processing?

Instead of:

```text
Load 10 million records
        ↓
Memory problem
```

you process manageable chunks:

```text
1000
 ↓
process
 ↓
write
 ↓
commit

1000
 ↓
process
 ↓
write
 ↓
commit
```

Benefits:

```text
Lower memory usage
Transaction boundaries
Better failure recovery
Large-volume processing
```

---

# 8. What does chunk size actually mean?

If:

```java
.chunk(100)
```

it generally means the framework processes up to **100 items per chunk/transaction boundary** for a chunk-oriented step.

Conceptually:

```text
read → process → write
        100 items
           ↓
       transaction
           ↓
         commit
```

---

# 9. What happens if record 57 fails?

Suppose chunk size = 100.

```text
Records 1–100
```

Record 57 causes an exception.

Without configured fault tolerance:

```text
Chunk fails
 ↓
Transaction rolls back
```

The exact retry/recovery behavior depends on the configured reader, processor, writer, transaction and fault-tolerance configuration.

---

# 10. What is fault tolerance in Spring Batch?

Spring Batch supports:

```text
Retry
Skip
Rollback
Recovery
```

Example:

```java
.faultTolerant()
.retry(Exception.class)
.retryLimit(3)
```

Meaning certain failures can be retried.

---

# 11. Retry vs Skip

### Retry

Use when the failure might be temporary.

Examples:

```text
Network timeout
Temporary DB issue
Transient external service failure
```

```text
Failure
 ↓
Retry
 ↓
Retry
 ↓
Success
```

### Skip

Use when a particular record is invalid and you want the batch to continue.

Example:

```text
100 records
Record 57 → invalid
```

Instead of failing the entire batch:

```text
56 → processed
57 → skipped
58 → processed
...
```

Example:

```java
.faultTolerant()
.skip(ValidationException.class)
.skipLimit(10)
```

---

# 12. Retry vs Skip — interview answer

> Retry is appropriate for transient failures where repeating the operation may succeed. Skip is appropriate for known bad records where we want to continue processing the remaining records.

This distinction is **very important**.

---

# 13. What is a JobInstance?

A `JobInstance` represents a **logical execution of a job identified by its job name and identifying job parameters**.

Example:

```text
Job:
PF_DATA_LOAD

Parameters:
businessDate=2026-09-25
```

That identifies a particular logical job instance.

---

# 14. What is JobExecution?

A `JobExecution` represents **one attempt/execution of a JobInstance**.

Example:

```text
JobInstance
   |
   +-- JobExecution #1 → FAILED
   |
   +-- JobExecution #2 → COMPLETED
```

This distinction is commonly asked.

### Easy memory

```text
JobInstance
→ WHAT logical run?

JobExecution
→ ONE attempt of that run
```

---

# 15. What is JobParameters?

Parameters supplied when launching a job.

Example:

```text
businessDate=2026-09-25
fileName=input.csv
```

They can be used to identify a JobInstance.

Example:

```java
new JobParametersBuilder()
    .addString("businessDate", "2026-09-25")
    .toJobParameters();
```

---

# 16. What is JobRepository?

This is extremely important.

`JobRepository` stores metadata about batch execution.

It stores information such as:

```text
JobInstance
JobExecution
StepExecution
JobParameters
Execution status
Start/end times
Execution context
```

Typically stored in database tables such as:

```text
BATCH_JOB_INSTANCE
BATCH_JOB_EXECUTION
BATCH_STEP_EXECUTION
BATCH_JOB_EXECUTION_PARAMS
...
```

So:

```text
Business data
      ↓
Your tables

Batch metadata
      ↓
Spring Batch metadata tables
```

---

# 17. Why does Spring Batch need JobRepository?

Because Spring Batch needs to know:

```text
What job ran?
What parameters?
Which step completed?
Where did it fail?
Can it restart?
What was the status?
```

This enables **restartability**.

---

# 18. What is StepExecution?

It represents one execution of a particular step.

It tracks information such as:

```text
readCount
writeCount
filterCount
skipCount
commitCount
rollbackCount
status
exit status
```

Example:

```text
Step: Load Accounts

readCount  = 10000
writeCount = 9980
skipCount  = 20
```

Very useful for monitoring.

---

# 19. What is ExecutionContext?

`ExecutionContext` stores state needed to support restartability.

Think:

```text
JobExecution
   ↓
ExecutionContext

StepExecution
   ↓
ExecutionContext
```

Example:

```text
lastProcessedId = 50000
```

If the job fails, this state can help the job continue from an appropriate point during restart, depending on the reader/step configuration.

---

# 20. JobExecution vs StepExecution

```text
JobExecution
    |
    +-- StepExecution 1
    |
    +-- StepExecution 2
    |
    +-- StepExecution 3
```

A JobExecution tracks the execution of the entire job.

Each StepExecution tracks execution of an individual step.

---

# 21. What happens when a Spring Batch job fails?

Example:

```text
Job
 ↓
Step 1 → COMPLETED
 ↓
Step 2 → FAILED
 ↓
Step 3 → NOT EXECUTED
```

Spring Batch persists metadata.

On restart, Spring Batch can determine which steps have already completed and which need to execute again, subject to job configuration and restartability rules.

This is one of Spring Batch's major advantages over writing a simple `for` loop.

---

# 22. What is restartability?

Suppose:

```text
1 million records
```

Job processes:

```text
700,000
```

and then crashes.

A properly designed Spring Batch job can restart from an appropriate checkpoint instead of blindly processing everything again.

Conceptually:

```text
Start
 ↓
0
 ↓
...
 ↓
700,000
 ↓
FAIL
 ↓
RESTART
 ↓
Continue
```

The exact restart position depends on the reader and state management.

---

# 23. What is a Tasklet?

A Tasklet is used when a step performs a relatively simple task rather than standard item-by-item chunk processing.

Example:

```java
@Bean
public Step cleanupStep() {
    return new StepBuilder("cleanup", jobRepository)
        .tasklet((contribution, chunkContext) -> {
            cleanupOldFiles();
            return RepeatStatus.FINISHED;
        }, transactionManager)
        .build();
}
```

Good for:

```text
Delete files
Execute a stored procedure
Simple DB operation
One-off operation
```

---

# 24. Tasklet vs Chunk-oriented processing

### Tasklet

```text
execute task
   ↓
finish
```

### Chunk

```text
Read
 ↓
Process
 ↓
Write
 ↓
Commit
 ↓
repeat
```

Use chunk processing for large item-oriented datasets.

---

# 25. What is `ItemReader` state?

Some readers are **stateful** and can save their current position in the `ExecutionContext`.

For example, a file reader may track its current position.

This supports restartability.

That's why not all readers behave the same way during restart.

---

# 🔥 Spring Batch flow to memorize

If they ask:

> "Explain Spring Batch architecture."

Say:

```text
JobLauncher
    ↓
Job
    ↓
Step
    ↓
ItemReader
    ↓
ItemProcessor
    ↓
ItemWriter
```

And behind it:

```text
              JobRepository
                   ↓
        Job/Step execution metadata
                   ↓
             ExecutionContext
```

With chunk processing:

```text
Read N
  ↓
Process N
  ↓
Write N
  ↓
Commit transaction
  ↓
Next chunk
```

---

# 🔥 Rapid Fire

| Question          | Answer                                                 |
| ----------------- | ------------------------------------------------------ |
| Job?              | Complete batch process                                 |
| Step?             | One phase of a job                                     |
| Reader?           | Reads input                                            |
| Processor?        | Transforms/validates                                   |
| Writer?           | Writes output                                          |
| Chunk?            | Group of items processed within a transaction boundary |
| Tasklet?          | Simple task-oriented step                              |
| JobRepository?    | Stores batch metadata                                  |
| JobInstance?      | Logical job identified by job + identifying parameters |
| JobExecution?     | One execution/attempt of JobInstance                   |
| StepExecution?    | Execution of one step                                  |
| JobParameters?    | Parameters used when launching/identifying job         |
| ExecutionContext? | Persistent state for execution/restart                 |
| Retry?            | Repeat transient failure                               |
| Skip?             | Ignore configured bad records                          |
| Restartability?   | Resume/re-execute appropriately after failure          |

---

## Next Spring Batch — Part 2 🔥

This is the **more important senior-level section**:

* `JdbcPagingItemReader` vs `JdbcCursorItemReader`
* JPA reader vs JDBC reader
* partitioning
* multi-threaded step
* remote partitioning
* remote chunking
* `TaskExecutor`
* parallel processing
* restartability with partitions
* skip/retry transaction behavior
* how to process **millions of records**
* Spring Batch performance tuning
* common production/interview scenarios
