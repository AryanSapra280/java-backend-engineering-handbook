Perfect. **Postman next**, because this was an actual question from your VP, so it's a concrete gap we should close.

## Postman — interview level you need

Think of Postman as a tool to **call APIs, validate responses, and automate API testing**.

You already know the basic part:

```text
Postman
   ↓
Send HTTP request
   ↓
API
   ↓
Receive response
   ↓
Verify response
```

The part you were missing is **automation**.

---

# 1. Collection

A **Collection** is a group of related API requests.

For example:

```text
Loan IQ APIs
│
├── Create Loan
├── Get Loan
├── Update Loan
├── Delete Loan
└── Search Loans
```

Instead of keeping requests separately, you organize them into a collection.

---

# 2. Environment variables

Suppose your API URL changes between environments.

Development:

```text
http://dev-server:8080
```

Production:

```text
http://prod-server:8080
```

Instead of hardcoding:

```http
GET http://dev-server:8080/loans/123
```

you can use:

```http
GET {{baseUrl}}/loans/123
```

And define:

```text
baseUrl = http://dev-server:8080
```

in your environment.

Then you can switch environments without modifying every request.

This is something you should be able to explain.

---

# 3. Tests — THIS is what your VP was asking about

Postman allows JavaScript-based scripts to validate the API response.

Suppose:

```http
GET /employees/123
```

returns:

```json
{
    "id": 123,
    "name": "Aryan"
}
```

You can write a test to verify:

```javascript
pm.test("Status code is 200", function () {
    pm.response.to.have.status(200);
});
```

And verify the response body:

```javascript
pm.test("Employee ID is correct", function () {
    const response = pm.response.json();
    pm.expect(response.id).to.eql(123);
});
```

So instead of manually looking at the response and saying:

> "Looks correct."

Postman automatically says whether the test passed or failed.

---

# 4. Multiple assertions

You can test several things:

```javascript
pm.test("Status code is 200", function () {
    pm.response.to.have.status(200);
});

pm.test("Response contains employee name", function () {
    const response = pm.response.json();
    pm.expect(response.name).to.exist;
});

pm.test("Employee ID is correct", function () {
    const response = pm.response.json();
    pm.expect(response.id).to.eql(123);
});
```

Result:

```text
✓ Status code is 200
✓ Response contains employee name
✓ Employee ID is correct
```

If something fails:

```text
✗ Employee ID is correct
```

---

# 5. Pre-request Script

There are also scripts that execute **before the request is sent**.

For example:

```text
Pre-request Script
       ↓
Generate token / timestamp / dynamic data
       ↓
Send API request
       ↓
Tests
       ↓
Validate response
```

A simple example:

```javascript
pm.environment.set("employeeId", "123");
```

Then:

```http
GET /employees/{{employeeId}}
```

---

# 6. Chaining requests ⭐

This is a useful interview concept.

Suppose we have:

```text
POST /employees
```

which creates:

```json
{
    "id": 123
}
```

Then we want:

```text
GET /employees/123
```

We can capture the ID from the first response and save it as a variable.

Conceptually:

```text
POST Create Employee
       ↓
Response contains ID = 123
       ↓
Save ID into variable
       ↓
GET /employees/{{employeeId}}
```

This is called **request chaining**.

---

# 7. Collection Runner

Now we get to **automation**.

Suppose you have:

```text
100 API requests
```

You don't want to manually click:

```text
Send
Send
Send
Send
...
```

The Collection Runner can execute a collection of requests automatically.

For example:

```text
Collection
   ↓
Request 1
   ↓
Request 2
   ↓
Request 3
   ↓
Request 4
   ↓
Tests
   ↓
Pass / Fail report
```

You can also run the same tests with multiple data sets.

---

# 8. Newman

This is another keyword worth knowing.

**Newman** is the command-line runner for Postman collections.

Why is that useful?

Because you can integrate API tests into CI/CD.

Conceptually:

```text
Developer commits code
        ↓
CI/CD pipeline
        ↓
Deploy application
        ↓
Run Postman collection using Newman
        ↓
API tests
        ↓
PASS → continue
FAIL → pipeline can fail
```

That's the important idea.

You don't need to memorize Newman commands right now.

---

# 9. What should YOU answer if they ask:

### "Do you know how to automate testing in Postman?"

Your previous answer:

> "We can do scripting in Postman."

was **directionally correct**, but incomplete.

A much better answer:

> **"Yes. Postman supports automated API testing using JavaScript test scripts. We can validate status codes, response bodies, headers and other assertions. We can organize requests into collections and execute them using the Collection Runner. For CI/CD automation, Postman collections can also be executed using Newman."**

🔥 That's enough.

If they ask:

> "Have you personally written the scripts?"

Be honest.

Don't claim that you did if your QA team actually handled it.

You can say:

> **"In my project, the QA team handled most of the Postman scripting, but I understand how the automation works. We can add JavaScript assertions in the Tests section, use environment variables, chain requests, and execute collections through the runner. I'm comfortable working with and debugging those tests."**

That's a **much safer answer**.

---

# 10. Postman things to remember

```text
Postman
│
├── Request
│
├── Collection
│
├── Environment variables
│
├── Pre-request Script
│
├── Tests / Assertions ⭐
│
├── Request chaining
│
├── Collection Runner ⭐
│
└── Newman → CI/CD ⭐
```

### One-line mental model

> **Postman lets us manually test APIs, while scripts, collections, runners and Newman allow us to automate those API tests.**

---

Now I want to test you.

### Interview question:

**"Aryan, suppose you have 50 REST APIs. You want to automatically verify that every API returns the expected status code and response. How would you do this using Postman?"**

Answer in your own words. Don't worry about syntax.
