# MUFG Interview Preparation — Transaction Manager, Contribution Processing & Distributed Transactions

## 1. Contribution → Transaction Manager

When a contribution request is received, the flow involves the Transaction Manager (TM).

The contribution is not simply a database insert.

The Transaction Manager performs business validations and determines how the contribution should be processed.

One of the important business decisions is related to the annual PF contribution limit and the allocation between different contribution buckets.

---

# 2. PF Contribution Buckets

The PF contribution processing has multiple buckets, including:

- Tax-free member contribution
- Taxable member contribution
- EPF tax-free
- EPF taxable

The Transaction Manager determines how the contribution should be allocated.

For example, when the applicable tax-free contribution limit is reached, the additional contribution may need to be allocated to the taxable bucket according to the applicable business rules.

The Transaction Manager performs this calculation/decision and then instructs the relevant downstream processing regarding the amount to be credited to the appropriate bucket.

---

# 3. Redis in Contribution Processing

The Transaction Manager uses Redis for information such as the yearly contribution limit.

Conceptually:

Contribution
    ↓
Transaction Manager
    ↓
Check relevant limit/state in Redis
    ↓
Determine tax-free / taxable allocation
    ↓
Continue transaction processing

Redis is therefore involved in the business decision around contribution limits.

---

# 4. Failure While Processing Redis-related State

A potential failure scenario is:

Transaction Manager
    ↓
Update/cache-related state in Redis
    ↓
Later processing fails

If Redis state has already been modified, the system has a compensation mechanism to undo/revert the relevant change where required.

This is important because Redis may not participate in the same database transaction as the relational database.

Therefore, simply relying on a normal database rollback is not sufficient for every operation.

---

# 5. Transaction Manager's Local Database Transaction

The Transaction Manager persists information into multiple database tables.

Examples mentioned:

- Transaction table
- Ledger-related tables

These database operations can participate in a local transactional boundary.

Conceptually:

BEGIN TRANSACTION

    Update transaction table
    Update ledger-related data
    Perform required persistence

    ↓

If all succeed:
    COMMIT

If a failure occurs:
    ROLLBACK

Therefore, if multiple database operations are inside the same local database transaction, a failure can roll them back together.

---

# 6. Important Distinction — Local Transaction vs Distributed Transaction

A local database transaction is relatively straightforward:

Service
    ↓
Database
    ↓
BEGIN
    ↓
Multiple DB operations
    ↓
COMMIT / ROLLBACK

The difficulty starts when the business operation crosses service boundaries.

Example:

Transaction Manager
    ↓
Account Service
    ↓
Ledger Service
    ↓
Another downstream service

Each service can own its own database.

A single traditional database transaction cannot simply span all these independently owned service databases.

Therefore, a different consistency/recovery mechanism is required.

---

# 7. Distributed Failure Scenario

Consider:

Step 1 → Service A succeeds
Step 2 → Service B succeeds
Step 3 → Service C fails

At this point:

- Step 1 has already committed
- Step 2 has already committed
- Step 3 has failed

A normal database rollback cannot simply undo Steps 1 and 2 because they may belong to different services and different databases.

This is where the Saga pattern becomes relevant.

---

# 8. Saga Pattern

A Saga is a distributed transaction pattern used when a business workflow spans multiple independently managed services.

Instead of trying to create one large distributed database transaction, the workflow is divided into a sequence of local transactions.

Example:

Step 1
    ↓
Local transaction commits

Step 2
    ↓
Local transaction commits

Step 3
    ↓
Local transaction fails

The system then executes compensating actions for previously completed steps where required.

Conceptually:

Step 1 → SUCCESS
Step 2 → SUCCESS
Step 3 → FAILURE
              ↓
       Compensation
              ↓
        Undo Step 2
              ↓
        Undo Step 1

---

# 9. Important Concept — Compensation Is Not Database Rollback

This is one of the most important points to remember for interviews.

A Saga does NOT magically roll back previously committed database transactions.

Instead, it executes new business operations intended to compensate for the previous operations.

Example:

Original operation:

    Credit Account

Compensating operation:

    Reverse Credit

Another example:

Original:

    Create/activate something

Compensation:

    Deactivate/reverse it

Therefore:

> "A Saga doesn't roll back a distributed transaction. It performs compensating actions to bring the business state back toward the required consistent state."

---

# 10. LRA — Long Running Action

The platform uses LRA/Saga-related behavior around the Transaction Manager.

LRA stands for:

**Long Running Action**

LRA is a concrete specification/implementation approach for long-running distributed business transactions.

The general idea is similar to Saga:

- A long-running business operation is divided into steps.
- Each participating service performs its local work.
- If the overall operation fails, compensation can be invoked for previously completed work.

IMPORTANT:

Do not overstate the exact implementation details unless verified from the actual code/configuration.

The exact role of the LRA coordinator, participant registration, completion, cancellation, and compensation behavior should be verified before describing those implementation details in an interview.

---

# 11. Transaction Manager as an Orchestrator

The Transaction Manager coordinates important parts of the contribution business flow.

Conceptually:

Contribution
    ↓
Transaction Manager
    ↓
Business validation / calculation
    ↓
Account / Ledger / other services
    ↓
Distributed workflow
    ↓
Success OR compensation/recovery

The Transaction Manager therefore has visibility/control over the business sequence rather than simply acting as a CRUD service.

---

# 12. Why Compensation Is Required

Suppose:

1. TM performs contribution processing.
2. Local transaction succeeds.
3. Event/request goes to another service.
4. Downstream service fails.

The earlier successful work may already have been committed.

A simple DB rollback inside TM cannot undo the downstream service's committed work.

Therefore, the distributed workflow needs a business-level recovery/compensation mechanism.

---

# 13. Important Failure Scenarios

## Scenario A — Local DB failure

Example:

TM updates transaction table
    ↓
Ledger persistence attempted
    ↓
DB failure
    ↓
Local transaction rolls back

If all required DB operations are inside the same local transaction boundary, the database can rollback the changes.

---

## Scenario B — Redis state changed, later processing fails

Redis state has changed
    ↓
Later operation fails
    ↓
Compensation may be required
    ↓
Redis state is restored/reverted where applicable

---

## Scenario C — Local transaction succeeds, downstream service fails

TM local transaction
    ↓
COMMIT
    ↓
Downstream call/event
    ↓
Failure

The local DB transaction cannot simply rollback after the downstream operation has already crossed a service boundary.

This is where distributed workflow recovery/compensation becomes relevant.

---

## Scenario D — Multiple downstream services succeed, later service fails

Service A → SUCCESS
Service B → SUCCESS
Service C → FAILURE

Previously completed work cannot simply be rolled back using one database transaction.

Compensation needs to be performed for the previously completed operations where the business process requires it.

---

# 14. Retry vs Compensation

These are NOT the same.

### Retry

Used when the same operation may succeed if attempted again.

Example:

Temporary network failure
    ↓
Retry
    ↓
Operation succeeds

### Compensation

Used when an earlier business operation has already succeeded/committed but the overall workflow cannot continue and the previous business effect needs to be reversed.

Example:

Credit completed
    ↓
Later business step fails
    ↓
Compensating debit/reversal

Therefore:

> Retry attempts to complete the original operation.

> Compensation attempts to reverse/offset a previously completed operation.

---

# 15. Important Question — Should a DB Timeout Always Be Retried?

NO blanket answer should be given.

A database timeout can potentially be transient.

But before retrying, we need to consider:

- What operation was being performed?
- Did the operation actually reach the database?
- Did it commit before the timeout was returned?
- Is the operation idempotent?
- Could retry create a duplicate side effect?
- Can the current state be checked before retrying?

For example:

Request
    ↓
DB operation
    ↓
DB commits
    ↓
Network/response timeout
    ↓
Application thinks operation failed

If the application blindly retries, the same business operation could potentially be performed twice.

Therefore:

> "Retryability is determined not only by the exception type but also by whether the operation is safe to repeat and whether the system can determine its current state."

This is a key senior-level concept.

---

# 16. Financial-System Perspective

In financial systems, the most important question is not simply:

> "Can I retry?"

It is:

> **"Can I safely retry without creating an incorrect financial side effect?"**

Examples of dangerous duplicate operations:

- Duplicate contribution
- Duplicate credit
- Duplicate debit
- Duplicate settlement
- Duplicate transfer

Therefore, retry mechanisms should be considered together with:

- Idempotency
- Transaction state
- Correlation/reference IDs
- Business keys
- State checks
- Reconciliation
- Compensation

---

# 17. Contribution Processing — Mental Model

A simplified mental model:

Contribution Request
        ↓
Transaction Manager
        ↓
Validate request
        ↓
Check contribution/limit state
        ↓
Determine applicable bucket
        ↓
Perform local transactional persistence
        ↓
Commit local work
        ↓
Trigger downstream processing
        ↓
Monitor downstream outcome
        ↓
Success
   OR
Failure → Retry / DLQ / Rejection / Compensation depending on failure type

---

# 18. Senior-Level Interview Questions From This Area

These are questions to prepare:

### Business

1. What exactly is a contribution?
2. Who sends the contribution?
3. How frequently does contribution arrive?
4. What information does the contribution contain?
5. Why does the system need different contribution buckets?
6. How is the annual limit determined?
7. What happens when the limit is exceeded?

### Transaction Manager

8. What exactly is the responsibility of Transaction Manager?
9. Why does TM need Redis?
10. Why is the limit stored/read from Redis?
11. What happens if Redis is unavailable?
12. What happens if Redis is updated but later processing fails?
13. Which DB operations are part of the local transaction?
14. What happens if one DB operation fails?

### Distributed transaction

15. Why can't one database transaction cover all microservices?
16. What happens when downstream processing fails after TM commits?
17. What is Saga?
18. What is compensation?
19. How is compensation different from rollback?
20. What is LRA?
21. Why would you use LRA?
22. What happens if compensation itself fails?
23. How do you know whether compensation has already happened?
24. How do you prevent duplicate compensation?

### Retry

25. Should every exception be retried?
26. Is a DB timeout always retryable?
27. What if the DB committed but the application received a timeout?
28. How do you prevent duplicate contribution?
29. How do you determine whether an operation is safe to retry?
30. How does idempotency work in your contribution flow?

---

# 19. OPEN / VERIFY

These details must be confirmed from the actual implementation before presenting them as facts:

- Exact LRA implementation used by Transaction Manager.
- Exact LRA coordinator behavior.
- Exact participant/compensation implementation.
- Which TM operations participate in LRA.
- Exact Redis compensation mechanism.
- Exact Redis failure behavior.
- Exact list of retryable exceptions.
- Exact Spring Cloud Stream retry configuration.
- Exact relationship between TM retry and Saga/LRA compensation.
- Whether contribution operations use explicit idempotency keys/business keys.
- Exact mechanism preventing duplicate contribution.
- Exact behavior when DB timeout occurs after a DB commit.
- Exact compensation behavior if compensation itself fails.
- Exact persistence state transitions during compensation.
- Exact event acknowledgement semantics.
- Exact relationship between Transaction Manager and Rejection Management during LRA failure.
- Exact relationship between LRA compensation and Business Reconciliation.

---

# 20. Key Mental Model for MUFG

Remember these four levels:

LOCAL DATABASE CONSISTENCY

DB operations
    ↓
Transaction
    ↓
COMMIT / ROLLBACK


MESSAGE PROCESSING FAILURE

Consumer
    ↓
Retry
    ↓
DLQ


BUSINESS FAILURE MANAGEMENT

Qualified failure
    ↓
Rejection Management
    ↓
Investigation / controlled reprocessing


DISTRIBUTED BUSINESS CONSISTENCY

Multiple services
    ↓
Local transactions
    ↓
One step fails
    ↓
Saga/LRA
    ↓
Compensation

These mechanisms solve different problems and should not be presented as interchangeable.