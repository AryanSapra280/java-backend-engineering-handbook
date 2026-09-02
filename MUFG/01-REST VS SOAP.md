Great. Let's continue with **REST**, and we'll keep it interview-oriented. We don't need to memorize textbook definitions — you need to be able to **explain why and how you would use it**.

## 1. REST fundamentals

Suppose we have an Employee service.

A REST API could look like:

```text
GET    /employees/123
POST   /employees
PUT    /employees/123
PATCH  /employees/123
DELETE /employees/123
```

The important idea is that **HTTP methods communicate the intended operation**.

### GET

Retrieve data.

```http
GET /employees/123
```

### POST

Create something or trigger an operation that isn't naturally idempotent.

```http
POST /employees
```

### PUT

Replace/update a resource.

```http
PUT /employees/123
```

### PATCH

Partially update a resource.

```http
PATCH /employees/123
```

### DELETE

Delete a resource.

```http
DELETE /employees/123
```

---

# 2. The question they may ask: PUT vs PATCH

**PUT** generally represents replacing the resource with the supplied representation.

**PATCH** represents a partial modification.

For example, employee:

```json
{
  "name": "Aryan",
  "department": "IT",
  "salary": 100000
}
```

If I want to replace the employee representation:

```http
PUT /employees/123
```

If I only want to change salary:

```http
PATCH /employees/123
```

with:

```json
{
  "salary": 110000
}
```

### Interview answer

> "`PUT` is generally used when we want to replace the resource representation, whereas `PATCH` is used for partial updates."

---

# 3. HTTP status codes

You absolutely should know these.

### 2xx — Success

```text
200 OK
201 Created
204 No Content
```

Typical usage:

```text
GET successful       → 200
POST creates resource → 201
DELETE successful     → 204
```

### 4xx — Client-side/request problem

```text
400 Bad Request
401 Unauthorized
403 Forbidden
404 Not Found
409 Conflict
```

Don't confuse:

### 401

Authentication problem.

> "Who are you?"

### 403

Authorization problem.

> "I know who you are, but you aren't allowed to do this."

### 404

Resource doesn't exist.

### 409

Request conflicts with the current state.

Example:

```text
Trying to create an employee
but employee ID already exists.
```

---

### 5xx — Server-side problem

```text
500 Internal Server Error
502 Bad Gateway
503 Service Unavailable
504 Gateway Timeout
```

For your production troubleshooting preparation, these are particularly useful.

---

# 4. Statelessness ⭐

This is something you already encountered.

When we say REST is stateless, we mean:

> **Each request should contain the information necessary for the server to process it; the server should not depend on stored conversational/session state from a previous request.**

Example:

```text
Request 1:
GET /employees/123
Authorization: token

Request 2:
GET /employees/456
Authorization: token
```

The server shouldn't need to remember:

```text
"What did Aryan ask me in request 1?"
```

to process request 2.

### Why is this useful?

It makes horizontal scaling easier.

Suppose we have:

```text
             Load Balancer
             /     |     \
            /      |      \
         Pod A   Pod B   Pod C
```

A request can go to any pod because the request contains the required context.

---

# 5. Idempotency ⭐⭐⭐

**This is worth learning very well.**

An operation is idempotent if making the same request multiple times has the **same intended effect on the resource state** as making it once.

Typical HTTP semantics:

```text
GET     → idempotent
PUT     → idempotent
DELETE  → idempotent
POST    → generally NOT idempotent
```

Example:

```http
PUT /employees/123

{
   "salary": 100000
}
```

Send it once:

```text
salary = 100000
```

Send it five times:

```text
salary = 100000
```

Same resulting state.

But:

```http
POST /payments
```

could create a new payment every time:

```text
Request 1 → Payment A
Request 2 → Payment B
Request 3 → Payment C
```

So POST is generally non-idempotent.

---

# 6. Why idempotency matters for payments

This is especially relevant to your financial-domain background.

Imagine:

```text
Client
  ↓
POST /payment
  ↓
Server processes payment
  ↓
Server sends response
```

But the network fails **after the payment succeeds**.

The client doesn't know whether it succeeded.

It retries:

```text
POST /payment
```

Without idempotency:

```text
₹10,000 deducted
+
₹10,000 deducted again
```

🔥 That's a serious problem.

So we can introduce an **idempotency key**:

```http
POST /payments
Idempotency-Key: abc123
```

The server stores the result associated with:

```text
abc123
```

If the same request comes again:

```text
Idempotency-Key: abc123
```

the server recognizes it as the same operation rather than creating another payment.

This is a **very good design concept to know for MUFG**.

---

# 7. REST API design

Suppose they ask:

> "Aryan, design an API for retrieving an employee."

Don't say:

```text
/getEmployee?id=123
```

A resource-oriented REST API would typically be:

```http
GET /employees/123
```

For creating:

```http
POST /employees
```

For deleting:

```http
DELETE /employees/123
```

For listing:

```http
GET /employees
```

---

# 8. Query parameters vs path parameters

Example:

```http
GET /employees/123
```

`123` is a **path parameter**.

It identifies a specific resource.

Example:

```http
GET /employees?department=IT&page=2&size=20
```

These are **query parameters**.

They are useful for filtering, searching, sorting and pagination.

### Mental model

```text
/employees/123
        ↑
Resource identification


/employees?department=IT
            ↑
Filtering / options
```

---

# 9. Pagination

Suppose your employee table has:

```text
10 million employees
```

You should NOT return all 10 million records from:

```http
GET /employees
```

Instead:

```http
GET /employees?page=0&size=20
```

or potentially cursor-based pagination for large/changing datasets.

Response might contain:

```json
{
  "data": [...],
  "page": 0,
  "size": 20,
  "totalElements": 10000000
}
```

For a Loan IQ system, this becomes relevant because you're potentially dealing with **large financial datasets**.

---

# 10. API error handling

Don't just return:

```text
500
Something went wrong
```

A useful API should provide structured error information.

For example:

```json
{
  "code": "EMPLOYEE_NOT_FOUND",
  "message": "Employee 123 does not exist",
  "correlationId": "abc-123"
}
```

The `correlationId` is particularly useful when troubleshooting distributed systems.

You can trace:

```text
Client
 ↓
API Gateway
 ↓
REST service
 ↓
Another service
 ↓
Database
```

using the same correlation ID.

---

# 11. One VERY likely question for you

Because they asked you about SOAP and are apparently working on SOAP → REST modernization:

> **"If we have an existing SOAP service and want to expose it as REST, what would you do?"**

Your thought process should be:

```text
Existing SOAP
     ↓
Understand WSDL / existing contract
     ↓
Understand operations + request/response
     ↓
Design REST resources
     ↓
REST Controller/API layer
     ↓
Translate REST request
     ↓
Existing SOAP / Loan IQ SDK
     ↓
Translate response
     ↓
REST response
```

And importantly:

```text
Don't blindly change business logic.
```

The first goal is to **preserve existing business behavior while modernizing the interface**.

---

## Your REST checklist

For the interview, I want you to be comfortable with:

```text
✅ REST
✅ SOAP
✅ WSDL
✅ HTTP methods
✅ PUT vs PATCH
✅ HTTP status codes
✅ Statelessness
✅ Idempotency
✅ Idempotency keys
✅ Path vs query parameters
✅ Pagination
✅ Error handling
✅ Correlation ID
✅ REST API design
✅ SOAP → REST migration
```

**Next, I'd move directly to Postman**, because the VP specifically asked you about Postman automation. That's a concrete gap from Round 1 and probably a much higher ROI than spending another hour on advanced REST theory.
