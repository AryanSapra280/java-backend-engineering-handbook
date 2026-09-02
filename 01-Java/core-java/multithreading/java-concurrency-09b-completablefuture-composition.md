# Java Concurrency — CompletableFuture Composition

## 1. The Three Shapes

```text
thenApply:

A → B
```

```text
thenCompose:

A → Future<B>
```

```text
thenCombine:

A + B → C
```

These shapes are the easiest way to remember the APIs.

---

## 2. thenApply() — Normal Transformation

Suppose:

```java
CompletableFuture<String> future =
        CompletableFuture.supplyAsync(() -> "Aryan");
```

We want the length:

```java
CompletableFuture<Integer> result =
        future.thenApply(name -> name.length());
```

The lambda is:

```text
String → Integer
```

Therefore:

```text
CompletableFuture<String>
          ↓
      thenApply()
          ↓
CompletableFuture<Integer>
```

Use `thenApply` when you are simply changing one value into another value.

---

## 3. The Problem: What If the Next Operation Is Async?

Suppose:

```java
CompletableFuture<Customer> customerFuture =
        getCustomer();
```

and:

```java
CompletableFuture<Account> getAccount(
        Customer customer) {

    // asynchronous call
}
```

Notice:

```text
getAccount()
→ returns CompletableFuture<Account>
```

not directly:

```text
Account
```

Now:

```java
customerFuture.thenApply(
    customer -> getAccount(customer)
);
```

The lambda returns:

```text
CompletableFuture<Account>
```

So the type becomes:

```text
CompletableFuture<
    CompletableFuture<Account>
>
```

or:

```java
CompletableFuture<CompletableFuture<Account>>
```

---

## 4. Why Is That a Problem?

Visualize the result as boxes:

```text
┌──────────────────────────────┐
│ CompletableFuture            │
│                              │
│   ┌──────────────────────┐   │
│   │ CompletableFuture    │   │
│   │                      │   │
│   │      Account         │   │
│   └──────────────────────┘   │
│                              │
└──────────────────────────────┘
```

We don't want:

```text
Future
   ↓
Future
   ↓
Account
```

We want:

```text
Future
   ↓
Account
```

In Java:

```text
CompletableFuture<Account>
```

This is called flattening the nested future.

---

## 5. thenCompose()

Use:

```java
CompletableFuture<Account> accountFuture =
        customerFuture.thenCompose(
            customer -> getAccount(customer)
        );
```

Now:

```text
CompletableFuture<Customer>
             ↓
        thenCompose()
             ↓
CompletableFuture<Account>
```

The nested future is flattened.

---

## 6. The Easiest Rule

Ask:

> Does my lambda return a CompletableFuture?

### No

```java
future.thenApply(
    x -> x.getName()
);
```

The lambda returns:

```text
String
```

Use:

```text
thenApply
```

### Yes

```java
future.thenCompose(
    x -> getSomethingAsync(x)
);
```

The lambda returns:

```text
CompletableFuture<Something>
```

Use:

```text
thenCompose
```

This is the most useful rule to remember.

---

## 7. PF Application Example

Suppose:

```java
CompletableFuture<Account> getAccount(
        String accountId);
```

and:

```java
CompletableFuture<Balance> getBalance(
        Account account);
```

Start:

```java
CompletableFuture<Account> accountFuture =
        getAccount(accountId);
```

Then:

```java
CompletableFuture<Balance> balanceFuture =
        accountFuture.thenCompose(
            account -> getBalance(account)
        );
```

Why?

Because:

```text
Account
   ↓
getBalance(account)
   ↓
CompletableFuture<Balance>
```

The lambda returns a CompletableFuture.

Therefore:

```text
thenCompose
```

---

## 8. Visualize the Dependent Flow

```text
accountId
    ↓
getAccount()
    ↓
Future<Account>
    ↓
Account becomes available
    ↓
getBalance(account)
    ↓
Future<Balance>
```

The second operation depends on the first.

You cannot call:

```text
getBalance(account)
```

until you have the Account object.

---

## 9. thenCompose() Is Not Parallel

Do not say:

> "`thenCompose()` makes operations run in parallel."

It doesn't.

If:

```text
A → B → C
```

and:

```text
B depends on A
C depends on B
```

then:

```text
A completes
   ↓
B starts
   ↓
B completes
   ↓
C starts
```

`thenCompose` expresses this dependent asynchronous flow cleanly.

---

## 10. Multiple Dependent Operations

Suppose:

```java
CompletableFuture<Customer> getCustomer();

CompletableFuture<Account> getAccount(
        Customer customer);

CompletableFuture<Balance> getBalance(
        Account account);
```

We can write:

```java
getCustomer()
    .thenCompose(customer ->
        getAccount(customer)
    )
    .thenCompose(account ->
        getBalance(account)
    );
```

Flow:

```text
getCustomer()
     ↓
Customer
     ↓
getAccount(customer)
     ↓
Account
     ↓
getBalance(account)
     ↓
Balance
```

---

## 11. thenApply() in the Same Flow

Suppose after getting the balance, we only want the amount:

```java
getBalance(account)
    .thenApply(balance ->
        balance.getAmount()
    );
```

Here:

```text
Balance → Amount
```

The lambda returns a normal value.

Therefore:

```text
thenApply
```

is correct.

---

## 12. Full Dependent Example

```java
CompletableFuture<String> result =
    getCustomer()
        .thenCompose(customer ->
            getAccount(customer)
        )
        .thenCompose(account ->
            getBalance(account)
        )
        .thenApply(balance ->
            balance.getAmount()
        );
```

Mental model:

```text
Customer
   ↓
Account
   ↓
Balance
   ↓
Amount
```

Use:

```text
thenCompose
```

for:

```text
Customer → Future<Account>
Account  → Future<Balance>
```

Use:

```text
thenApply
```

for:

```text
Balance → Amount
```

---

## 13. thenCombine()

Now consider a different situation.

We have:

```java
CompletableFuture<Account> accountFuture =
        getAccount(accountId);

CompletableFuture<Interest> interestFuture =
        getInterest(accountId);
```

Neither depends on the other.

We want:

```text
Account + Interest
       ↓
AccountSummary
```

This is where `thenCombine()` is useful.

---

## 14. Independent Operations

Visualize:

```text
             Request
             /                 ↓       ↓
      Get Account  Get Interest
            ↓       ↓
         Account  Interest
            \       /
             \     /
              ↓   ↓
             Combine
                ↓
          AccountSummary
```

Both operations can be started independently.

Then their results can be combined.

---

## 15. thenCombine()

```java
CompletableFuture<AccountSummary> summaryFuture =
        accountFuture.thenCombine(
            interestFuture,
            (account, interest) ->
                createSummary(account, interest)
        );
```

Meaning:

> When both futures complete, give me both results and run the combining function.

---

## 16. Type Shape of thenCombine()

If we have:

```text
Future<A>
Future<B>
```

and:

```text
A + B → C
```

then:

```text
Future<A>
Future<B>
     ↓
thenCombine()
     ↓
Future<C>
```

Example:

```text
Future<Account>
Future<Interest>
       ↓
combine
       ↓
Future<AccountSummary>
```

---

## 17. Simple Example

```java
CompletableFuture<String> nameFuture =
        CompletableFuture.supplyAsync(() -> "Aryan");

CompletableFuture<Integer> ageFuture =
        CompletableFuture.supplyAsync(() -> 26);

CompletableFuture<String> result =
        nameFuture.thenCombine(
            ageFuture,
            (name, age) ->
                name + " is " + age
        );
```

Eventually:

```text
"Aryan is 26"
```

Inputs:

```text
String
Integer
```

Output:

```text
String
```

---

## 18. thenCompose vs thenCombine

### thenCompose

Use when:

```text
B depends on A
```

Shape:

```text
A → Future<B>
```

Example:

```java
customerFuture.thenCompose(
    customer -> getAccount(customer)
);
```

Because:

```text
Get Account requires Customer
```

---

### thenCombine

Use when:

```text
A and B are independent
```

Shape:

```text
A + B → C
```

Example:

```java
accountFuture.thenCombine(
    interestFuture,
    (account, interest) ->
        createSummary(account, interest)
);
```

---

## 19. Easy Memory Trick

```text
thenApply
→ CHANGE

thenCompose
→ CONTINUE ASYNC

thenCombine
→ COMBINE
```

Or remember the type shapes:

```text
thenApply:

A → B
```

```text
thenCompose:

A → Future<B>
```

```text
thenCombine:

A + B → C
```

---

## 20. Important Scenario

Transaction workflow:

```text
Customer
   ↓
Account
   ↓
Balance
```

Use:

```java
getCustomer()
    .thenCompose(customer ->
        getAccount(customer)
    )
    .thenCompose(account ->
        getBalance(account)
    );
```

Why?

Each operation depends on the previous result.

---

## 21. Independent Scenario

Suppose the service needs:

```text
Account
Interest Rate
```

and neither depends on the other.

```java
CompletableFuture<Account> account =
        getAccount(accountId);

CompletableFuture<Interest> interest =
        getInterest(accountId);

CompletableFuture<Result> result =
        account.thenCombine(
            interest,
            (a, i) -> calculate(a, i)
        );
```

---

## 22. Does thenCombine() Block?

The composition itself does not mean the caller waits synchronously.

It describes:

```text
When A and B complete
        ↓
run the combination function
```

But:

```java
Result result = resultFuture.get();
```

can block.

Therefore:

```text
Composition
→ can be non-blocking

get()/join()
→ blocking wait
```

---

## 23. Don't Confuse Starting Work With Combining Work

For example:

```java
CompletableFuture<Account> accountFuture =
        getAccount(accountId);

CompletableFuture<Interest> interestFuture =
        getInterest(accountId);
```

These methods create/start or return the asynchronous computations.

Then:

```java
accountFuture.thenCombine(
    interestFuture,
    (account, interest) ->
        calculate(account, interest)
);
```

describes how their results should be combined.

Mental model:

```text
Start independent operations
        ↓
Get Future handles
        ↓
Describe combination
```

---

## 24. Three Methods Together

```java
getCustomer()
    .thenCompose(customer ->
        getAccount(customer)
    )
    .thenCombine(
        interestFuture,
        (account, interest) ->
            createSummary(account, interest)
    )
    .thenApply(summary ->
        convertToDto(summary)
    );
```

Here:

```text
thenCompose
→ Customer → Future<Account>
```

```text
thenCombine
→ Account + Interest → Summary
```

```text
thenApply
→ Summary → DTO
```

---

## 25. The Key Questions

### Question 1

Does my lambda return a normal value?

```text
A → B
```

Use:

```text
thenApply
```

### Question 2

Does my lambda return another CompletableFuture?

```text
A → Future<B>
```

Use:

```text
thenCompose
```

### Question 3

Do I have two independent futures?

```text
Future<A>
Future<B>
```

and want:

```text
A + B → C
```

Use:

```text
thenCombine
```

---

## 26. Common Mistakes

### Mistake 1 — Using thenApply for another async operation

```java
customerFuture.thenApply(
    customer -> getAccount(customer)
);
```

Can produce:

```text
CompletableFuture<CompletableFuture<Account>>
```

Prefer:

```java
customerFuture.thenCompose(
    customer -> getAccount(customer)
);
```

### Mistake 2 — Thinking thenCompose means parallel execution

It doesn't.

It represents dependent asynchronous continuation.

### Mistake 3 — Using thenCombine for dependent operations

If:

```text
getBalance(account)
```

requires the Account object, it is dependent.

Use:

```text
thenCompose
```

### Mistake 4 — Calling get() after every stage

```java
Account account =
        accountFuture.get();

Balance balance =
        balanceFuture.get();
```

This can reintroduce blocking.

Use composition where appropriate.

---

## 27. MUFG Interview Questions

### What is the difference between thenApply and thenCompose?

Strong answer:

> "`thenApply` is used to transform the result of a CompletableFuture into a normal value. `thenCompose` is used when the next operation itself returns a CompletableFuture, typically because it is another asynchronous operation dependent on the previous result. `thenCompose` prevents nested CompletableFutures."

### Why would thenApply produce a nested CompletableFuture?

Strong answer:

> "Because thenApply wraps the value returned by its function as the result of the next stage. If the function itself returns a CompletableFuture, the resulting type becomes CompletableFuture<CompletableFuture<T>>."

### When would you use thenCombine?

Strong answer:

> "I would use thenCombine when I have two independent asynchronous computations and need to combine their results, such as fetching account information and interest information independently and then generating a combined response."

### Does thenCompose make operations parallel?

Answer:

> "No. It is primarily used to chain dependent asynchronous operations. If operation B needs the result of A, thenCompose lets me start B asynchronously after A completes without creating a nested CompletableFuture."

---

## 28. Final Mental Model

```text
                  CompletableFuture
                         |
             What relationship exists?
                         |
       ┌─────────────────┼─────────────────┐
       ↓                 ↓                 ↓
   Transform          Dependency       Independent
       ↓                 ↓                 ↓
  thenApply         thenCompose       thenCombine
       ↓                 ↓                 ↓
    A → B          A → Future<B>      A + B → C
```

Simplest version:

```text
thenApply
→ I already have a value.
→ Change it.

thenCompose
→ I have a value.
→ Use it to start another async operation.

thenCombine
→ I have two independent async operations.
→ Wait for both and combine their results.
```

---

## 29. Next Topics

Next:

```text
Exception Handling
      ↓
exceptionally()
handle()
whenComplete()
      ↓
Exception propagation
      ↓
Timeout handling
      ↓
allOf()
anyOf()
      ↓
Multiple parallel service calls
      ↓
Custom Executor
      ↓
Complete backend example
```

The next important topic is exception handling because production asynchronous workflows must define what happens when one stage fails.
