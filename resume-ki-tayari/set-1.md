# MUFG Interview Preparation — Failure Handling & Recovery
## Provident Fund / Retirement Platform

---

# 1. Business Context

The Provident Fund / Retirement platform was built to support more flexible and evolving PF business requirements compared with the older ledger-based approach.

The platform supports business workflows such as:

- Member Creation
- Contribution
- Advance
- Settlement
- Transfer-In
- Transfer-Out
- Account Lifecycle
- Interest Processing

The platform is implemented using multiple microservices and follows an event-driven architecture for several downstream workflows.

The system currently supports approximately 300K members.

---

# 2. Important Architecture Concept

The platform consists of multiple independently deployed domain components/services.

Examples discussed:

- Account
- Transaction Manager
- Interest
- Customer Ledger
- Balance Management
- Account Closure
- TMBM
- Risk
- Rejection Management
- Business Reconciliation

Not every service participates synchronously in every business flow.

Some interactions are synchronous when the caller immediately requires a response.

Other downstream processing happens asynchronously through messaging/events.

---

# 3. Member Creation — Business Meaning

Member creation means bringing an employee/member into the Provident Fund platform and creating the appropriate account relationships for that member.

The member may be a new employee or a lateral joiner.

The member information can include information such as:

- Employee ID
- Member code
- UN number
- Employee level
- Salary-related information
- Account eligibility
- Other employee/member attributes

Based on the member's information and applicable business rules, the system determines which accounts the member should have.

Examples of account types discussed:

- Provident Fund (PF)
- Superannuation
- Employee Pension Scheme (EPS)

The exact eligibility rules depend on the applicable business configuration/rules.

---

# 4. Member Creation — Basic Flow

Conceptually:

Request
    ↓
Member Creation Service
    ↓
Validation
    ↓
Account Service
    ↓
Account(s) created
    ↓
Account numbers returned
    ↓
Member/account mapping persisted
    ↓
Member creation response

The Account service is called synchronously during member creation.

The service needs the account number immediately because the member-creation API needs to provide the caller with the account information after successful creation.

---

# 5. Why Account Creation Is Synchronous

### Business reason

The caller expects the account number immediately after member registration.

Therefore, the service needs to know:

- Whether account creation succeeded
- Which account number(s) were created
- Whether account creation failed
- Why account creation failed

Because this information is required immediately, synchronous communication with Account was chosen.

### Interview explanation

> "The member-creation API has a synchronous business requirement because after successful registration the caller needs the account number immediately. Since our service needs the result of account creation to complete its own business operation, synchronous communication with Account was appropriate. Downstream consumers that don't need to participate in the immediate response can react asynchronously through events."

---

# 6. Why Not Make Account Creation Asynchronous?

If account creation itself were asynchronous:

Request
    ↓
Publish event
    ↓
Account creates account later
    ↓
Account publishes result
    ↓
Original service consumes result

The original service would not immediately know:

- Whether the account was created
- What account number was generated
- Whether account creation failed

The caller would therefore need to deal with eventual consistency or another mechanism for retrieving the result.

For the member-creation use case, immediate account creation confirmation was considered more appropriate.

---

# 7. Important Failure Scenario — Original Design

One problem identified during testing:

Original sequence:

Request
    ↓
Persist Member
    ↓
Call Account
    ↓
Account unavailable/fails
    ↓
Return error

This can leave the system in an inconsistent state:

Member exists in our boundary
BUT
Account does not exist

Then the user retries.

The system may respond:

"Member already exists"

even though account creation never succeeded.

This is an example of a partial failure in a distributed workflow.

---

# 8. Proposed Improvement

The proposed ordering was:

Request
    ↓
Call Account synchronously
    ↓
Account creation succeeds
    ↓
Persist member/account mapping
    ↓
Return success

This avoids persisting a member before the required account has been successfully created.

IMPORTANT:

This ordering was discussed/designed as an improvement after identifying the testing problem.

Do NOT claim it was implemented unless verified.

---

# 9. Important Senior-Level Failure Scenario

Consider:

Account Service
    ↓
Account successfully created
    ↓
Returns account number
    ↓
Our database persistence fails

Now:

Account Service:
    Account = CREATED

Our Service:
    Member/account mapping = NOT PERSISTED

This creates another form of partial failure.

The key question becomes:

> How do we recover without creating the account again?

The system should not blindly restart the entire workflow if the account was already successfully created.

If the workflow/state model records the successful state, recovery can potentially resume from the failed state rather than repeating already completed operations.

Example:

Validate Member
    ↓
Create Account       ← SUCCESS
    ↓
Persist Mapping      ← FAILED

Recovery should ideally resume around:

Persist Mapping

rather than:

Validate Member
    ↓
Create Account AGAIN

This avoids duplicate side effects.

IMPORTANT:

The exact implementation of state-based recovery for this specific Member Creation scenario must be verified from the actual code/architecture before claiming it in an interview.

---

# 10. State Design Pattern in the Platform

The business workflows use a state-driven approach.

As the workflow progresses, state is persisted.

The state indicates which business step has been completed or where the workflow currently stands.

When the workflow crosses boundaries or performs important persistence/business operations, state information is maintained.

The state can be used to understand:

- Current business position
- Last completed step
- Failed step
- What behavior should execute next

The platform uses behaviors associated with states.

This allows recovery/reprocessing to reason about the business workflow instead of blindly restarting everything from the beginning.

---

# 11. Failure Handling — Important Distinction

There are several different concepts that must NOT be treated as the same thing:

1. Automatic Retry
2. Dead Letter Queue (DLQ)
3. Rejection Management
4. Business Reconciliation

They solve different problems.

---

# 12. Automatic Retry

Retry happens FIRST.

When a Kafka consumer receives an event and processing throws an exception, the Spring Cloud Stream binder/platform retry mechanism retries the message according to configured retry attempts.

Conceptually:

Kafka Event
    ↓
Consumer
    ↓
Processing
    ↓
Exception
    ↓
Retry
    ↓
Retry
    ↓
Retry

If processing succeeds during retry:

Processing continues successfully.

If configured retries are exhausted:

Message proceeds to DLQ/error handling.

IMPORTANT:

DLQ is NOT the retry mechanism.

Retry happens before DLQ.

---

# 13. Dead Letter Queue (DLQ)

DLQ is a handoff point after normal consumer processing has exhausted its configured retries.

Flow:

Consumer
    ↓
Processing failure
    ↓
Configured retries
    ↓
Retries exhausted
    ↓
DLQ

DLQ therefore acts as a place where messages can be retained after normal processing could not complete.

It is NOT automatically a "retry queue".

After DLQ consumption, the system decides whether the failure qualifies for further business failure handling.

---

# 14. DLQ → Rejection Management

Once a message reaches DLQ, the DLQ consumer examines the failure.

Certain application/business-level exception types are considered eligible for Rejection Management.

Examples mentioned include application-level rejection classes such as:

- FBB application rejection
- Process Manager exception

If the failure qualifies:

DLQ
    ↓
Build error/rejection event DTO
    ↓
Include relevant information
    ↓
Publish to Rejection Management topic

Information can include:

- Original payload information
- Error code
- Error message
- Correlation ID
- Origin
- Category
- API/Event/Batch/Process Manager information
- Retry topic
- Kafka partition
- Other information required for replay/reprocessing

The retry topic/partition information helps retain the context required for reprocessing.

---

# 15. Infrastructure Exceptions

Not every DLQ message becomes a business rejection.

If a failure does NOT qualify for Rejection Management, it can remain in the DLQ as an infrastructure/technical error.

Therefore:

Retries exhausted
        ↓
       DLQ
        ↓
    Classification
       /       \
      /         \
Qualified       Not qualified
business        infrastructure
failure         failure
   ↓                 ↓
Rejection         Remains
Management        in DLQ

---

# 16. Another Entry Path into Rejection Management

Rejection Management can also receive failures directly through an application-level mechanism rather than the DLQ path.

For example:

Exception
    ↓
Controller Advice / Application failure handling
    ↓
Rejection Management event/API

Therefore there are two broad ways rejection information can enter:

### Path 1 — Event capture

Business component
    ↓
Failure
    ↓
Retry
    ↓
DLQ
    ↓
Qualified failure
    ↓
Rejection Management

### Path 2 — Direct application/API capture

Application-level failure handling
    ↓
Rejection Management

---

# 17. What Is Rejection Management?

Rejection Management is a centralized failure-capture and reprocessing mechanism.

Its purpose is essentially to provide a safety net so that failures in business processing are not silently lost.

It is a Java/Spring Boot service backed by PostgreSQL and integrated with messaging such as Kafka and NATS.

It exposes APIs for rejection-related operations and supports controlled reprocessing.

---

# 18. What Rejection Management Stores

When a qualified rejection is captured, information is persisted in a dedicated rejection data table/schema.

Information includes things such as:

- Original payload
- Error information
- Error code
- Error message
- Correlation ID
- Origin
- Category
- Business process/event information
- Reprocessing information

This provides:

- Auditability
- Failure visibility
- Investigation capability
- Controlled reprocessing

---

# 19. Rejection Management — Reprocessing

Rejection Management supports controlled reprocessing.

There are two discussed ways of triggering reprocessing:

### 1. Recon-driven

Recon identifies the failure/state and requests reprocessing through NATS.

### 2. Manual/direct

A human/operator can trigger reprocessing through the Rejection Management UI/API, using information such as correlation ID and business process.

Conceptually:

Failure
    ↓
Rejection Management
    ↓
Persist rejection
    ↓
Investigation
    ↓
Ready for Reprocess
    ↓
Reprocess
    ↓
Republish original event
    ↓
Original business flow resumes

---

# 20. Rejection Management State Machine

Each rejection moves through a reprocessing state machine.

States discussed:

UNDER_INVESTIGATION
        ↓
READY_FOR_REPROCESS
        ↓
REPROCESSED

A failure during reprocessing can move it back to:

UNDER_INVESTIGATION

A rejection can also be ignored/skipped depending on the available operation/state.

The exact state transitions should be verified against the current implementation if needed.

---

# 21. Business Reconciliation

Business Reconciliation (BSR) is a SEPARATE concern.

IMPORTANT:

> Business Reconciliation does NOT itself perform the retry.

Its purpose is to determine:

- What is the current business state?
- Which step failed?
- What is the expected state?
- What is the actual state?
- Which component/state is inconsistent?

It uses the transaction's correlation/reference information to reconstruct the business state across multiple processing steps/components.

---

# 22. Business Reconciliation — Conceptual Flow

Transaction
    ↓
Multiple business steps
    ↓
Failure occurs
    ↓
Rejection is registered
    ↓
Recon receives/records relevant information
    ↓
Recon evaluates current business state
    ↓
Identifies failed/pending step
    ↓
Human/operator investigates
    ↓
Data/business issue fixed if required
    ↓
Reprocessing is triggered through Rejection Management

Recon therefore answers:

> "What happened to this business transaction?"

It does not itself perform the retry.

---

# 23. Correlation ID

Correlation ID is used to associate processing belonging to a business transaction/workflow.

It helps different components identify that their processing belongs to the same overall business flow.

It is useful for:

- Tracking a transaction
- Finding failures
- Linking component processing
- Investigation
- Reconciliation
- Connecting original processing and reprocessing runs

IMPORTANT:

A correlation ID is an identifier/tracing mechanism.

It does NOT by itself guarantee:

- Exactly-once processing
- No duplicate messages
- Transactional consistency
- Successful completion

These are separate concerns.

---

# 24. Reprocessing and Correlation

The discussed reprocessing flow can generate a new correlation ID for the new processing run.

Business reconciliation can then link the new run back to the original transaction/business context.

Conceptually:

Original Transaction
    |
    | correlation ID A
    ↓
Failure
    ↓
Rejection Management
    ↓
Reprocessing
    |
    | new run
    | correlation ID B
    ↓
Business flow resumes

Recon can maintain the relationship between the original transaction and the reprocessing run.

---

# 25. Important Distinction for Interviews

Do NOT say:

"DLQ retries the message."

Better:

> "The consumer retries the message before it reaches the DLQ. Once retries are exhausted, the message is handed to the DLQ. Qualified failures can then be promoted into Rejection Management for controlled investigation and reprocessing."

Do NOT say:

"Reconciliation retries the transaction."

Better:

> "Reconciliation determines the current business state and identifies the failed/inconsistent step. Reprocessing is triggered separately through Rejection Management."

Do NOT say:

"Rejection Management automatically retries everything."

Better:

> "Rejection Management captures qualified failures, persists them for audit and investigation, and provides controlled mechanisms for reprocessing."

---

# 26. Senior-Level Failure Concepts to Remember

When discussing distributed financial workflows, always think about:

### Partial failure

One service succeeds while another fails.

Example:

Account created
+
Member mapping persistence failed

---

### Duplicate processing

A message can potentially be delivered/processed more than once.

Need to consider:

- Idempotency
- Duplicate detection
- State checks
- Safe retries

---

### Retry safety

Not every operation is safe to repeat.

Before retrying, ask:

> "Could this operation create another side effect?"

Example:

Creating an account twice may be dangerous.

Updating an idempotent status may be safe.

---

### State-aware recovery

Instead of restarting the complete workflow:

Start from the last known successful state / failed step where appropriate.

This reduces:

- Duplicate side effects
- Unnecessary processing
- Risk of repeating successful operations

---

### Business failure vs technical failure

A business rejection is different from a technical failure.

Examples of business failure:

- Invalid business condition
- Business rule violation
- Invalid transaction state

Examples of technical failure:

- Database unavailable
- Network timeout
- Downstream service unavailable
- Infrastructure issue

They should not necessarily have the same retry/recovery strategy.

---

# 27. Interview Answer Structure for Failure Questions

When asked:

> "What happens if something fails?"

Don't immediately start listing Kafka/DLQ/Rejection Management.

Use this structure:

1. Identify where failure occurred.
2. Identify whether it is transient or permanent/business-related.
3. Explain automatic retry if applicable.
4. Explain what happens after retry exhaustion.
5. Explain DLQ.
6. Explain whether the failure qualifies for Rejection Management.
7. Explain business reconciliation separately.
8. Explain how reprocessing happens.
9. Explain idempotency/state considerations.
10. Explain observability/correlation.

Example:

> "First I'd classify the failure. For a transient processing failure, our consumer-level retry mechanism attempts recovery before the message reaches the DLQ. Once retries are exhausted, the message goes to the DLQ. Qualified application/business failures are then captured by Rejection Management, where they are persisted for audit and controlled reprocessing. Business Reconciliation is a separate concern that determines the current state of the multi-step transaction and identifies the failed step. Once the underlying issue is addressed, reprocessing can be triggered through Rejection Management. For any reprocessing design, I'd also ensure that already-completed side effects aren't repeated, using state and idempotency mechanisms."

---

# 28. OPEN / VERIFY BEFORE INTERVIEW

These details should NOT be guessed:

- Exact exception hierarchy used by the platform.
- Exact classification rules for every exception type.
- Exact configured retry count.
- Exact Spring Cloud Stream retry configuration.
- Exact DLQ implementation/configuration.
- Exact Rejection Management API/event contract.
- Exact correlation-ID generation rules.
- Exact Business Reconciliation implementation.
- Exact person/component that triggers reconciliation.
- Exact mapping between rejection records and reconciliation steps.
- Exact behavior when Reprocessing fails repeatedly.
- Exact Member Creation recovery flow.
- Whether the "Account first → persist member mapping" change was implemented or only proposed.
- Whether every reprocessing run always receives a new correlation ID or only certain flows.
- Exact acknowledgement semantics between every producer/consumer.
- Exact guarantees provided by the platform regarding message delivery/processing.

---

# 29. Key Mental Model

Remember this chain:

AUTOMATIC FAILURE RECOVERY

Consumer
    ↓
Retry
    ↓
DLQ

BUSINESS FAILURE MANAGEMENT

DLQ / Application failure
    ↓
Rejection Management
    ↓
Persist + Investigate
    ↓
Controlled Reprocessing

BUSINESS STATE VISIBILITY

Transaction
    ↓
Business Reconciliation
    ↓
Expected vs Actual State
    ↓
Identify failed/inconsistent step

RECOVERY

Issue fixed
    ↓
Rejection Management
    ↓
Republish/reprocess
    ↓
Business workflow resumes

---

# 30. Questions We Still Need to Practice

These are intentionally left open for the next interview session:

1. How does the platform distinguish retryable vs non-retryable exceptions?
2. Who decides whether an exception qualifies for Rejection Management?
3. What happens if DB update succeeds but Kafka acknowledgement fails?
4. What happens if the same Kafka event is delivered twice?
5. How is idempotency implemented?
6. What happens if reprocessing itself fails?
7. What happens if the original transaction partially completed?
8. What happens if reconciliation and reprocessing happen concurrently?
9. What exactly does correlation ID guarantee?
10. How does state-based recovery prevent duplicate side effects?
11. What happens if Account is unavailable?
12. What happens if Account succeeds but local persistence fails?
13. What happens if Account times out but actually created the account?
14. How do you determine whether an account was already created?
15. How are retries monitored?
16. How do operators know a rejection requires intervention?
17. How does the system guarantee auditability?
18. How does the system behave during Kafka downtime?
19. What happens if Rejection Management itself is unavailable?
20. What happens if the rejection event itself cannot be published?