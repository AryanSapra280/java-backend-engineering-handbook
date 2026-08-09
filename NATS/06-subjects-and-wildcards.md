# 04 - NATS Subjects and Wildcards

## 1. Why Subjects Matter

A NATS subject is the logical address used by NATS to route messages.

For example:

    orders.created

A publisher sends a message to:

    orders.created

A subscriber can express interest in:

    orders.created

NATS then determines whether the subscriber should receive the message.

The important idea is:

> The publisher does not address a specific server or service instance. It publishes to a logical subject.

This is one of the foundations of the decoupling provided by NATS.

Instead of:

    http://distribution-service:8080/api/distribute

we can have:

    distribution.execute

The publisher doesn't need to know where the distribution service is running.

NATS handles the routing.

---

# 2. A Subject Is a Sequence of Tokens

A subject is not just an arbitrary string.

It is a sequence of tokens separated by dots (`.`).

For example:

    orders.created

contains:

    orders
    created

So:

    orders.created

has two tokens.

Another example:

    orders.us.created

contains:

    orders
    us
    created

Therefore:

    orders.us.created

has three tokens.

We can visualize it as:

    orders.us.created
    |      |      |
    |      |      +---- created
    |      +----------- us
    +------------------ orders

NATS uses these individual tokens when determining whether a published subject matches a subscriber's subject.

---

# 3. Why Tokenization Is Important

At first, you might think:

> Why not just treat the entire subject as a string?

Because tokenization allows NATS to create a hierarchy.

For example:

    orders.created
    orders.shipped
    orders.cancelled

can be grouped under:

    orders

And we can introduce regions:

    orders.us.created
    orders.eu.created
    orders.us.shipped
    orders.eu.shipped

Now the subject itself carries useful structure.

For example:

    orders.us.created

can be interpreted by our application as:

    orders
      |
      +-- us
            |
            +-- created

NATS itself does NOT know that:

    us = region

or:

    created = event type

Those meanings are defined by us.

This is an important point:

> NATS understands the structure of subject tokens, but the business meaning of those tokens belongs to the application.

---

# 4. Subject Naming Is an Application Design Decision

Suppose we design:

    interest.calculated

We could alternatively design:

    interest.calculation.completed

or:

    pf.interest.calculated

or:

    pf.account.interest.calculated

NATS does not tell us which one is correct.

The subject hierarchy is part of our application architecture.

A good subject naming convention should make it easy to understand:

- What domain does this belong to?
- What entity is involved?
- What happened?
- Is there a region/tenant/environment dimension?
- Can subscribers efficiently filter what they need?

For example:

    orders.us.created

could represent:

    domain = orders
    region = us
    event = created

---

# 5. Subjects Are Case-Sensitive

NATS subjects are case-sensitive.

These are different subjects:

    orders.created

and:

    Orders.created

Likewise:

    ORDERS.CREATED

is different from:

    orders.created

Therefore:

    Publisher
        |
        | orders.created
        v
      NATS
        |
        X
    Subscriber
    subscribed to Orders.created

The subscriber does not receive the message.

Why?

Because the subjects are different.

A practical recommendation is to establish a consistent naming convention.

For example:

    lowercase
    dot-separated
    meaningful tokens

Such as:

    orders.created
    orders.shipped
    interest.calculated
    distribution.completed

---

# 6. Dot Is the Token Separator

The dot (`.`) separates tokens.

For example:

    orders.us.created

contains:

    orders
    us
    created

The dot itself is not treated as part of a token.

This becomes extremely important when we introduce wildcards.

For example:

    orders.*.created

means:

    orders
    [ANY ONE TOKEN]
    created

It does NOT mean:

    orders
    [ANY STRING]
    created

The wildcard operates at the token level.

---

# 7. Subject Hierarchies

A common pattern is to organize subjects hierarchically.

For example:

    orders.created
    orders.shipped
    orders.cancelled

Then add region:

    orders.us.created
    orders.eu.created

Then potentially add additional dimensions:

    orders.us.retail.created
    orders.eu.retail.created
    orders.us.wholesale.created

This gives us a tree-like structure:

    orders
    |
    +-- us
    |   |
    |   +-- retail
    |   |     |
    |   |     +-- created
    |   |
    |   +-- wholesale
    |
    +-- eu
        |
        +-- retail
              |
              +-- created

This hierarchy allows subscribers to express broad or narrow interest.

---

# 8. Exact Subject Subscription

The simplest subscription is an exact subject.

For example:

    orders.created

A subscriber to:

    orders.created

receives messages published to:

    orders.created

But it does not receive:

    orders.shipped
    orders.cancelled
    orders.us.created

The subject must match exactly.

For example:

    Publisher
        |
        | orders.created
        v
      NATS
        |
        v
    Subscriber
    orders.created

MATCH

But:

    Publisher
        |
        | orders.shipped
        v
      NATS
        |
        X
    Subscriber
    orders.created

NO MATCH

---

# 9. Why Wildcards Exist

Imagine we have:

    orders.us.created
    orders.eu.created
    orders.in.created
    orders.uk.created

Suppose an analytics service wants:

> Every created order, regardless of region.

Without wildcards, it would need:

    orders.us.created
    orders.eu.created
    orders.in.created
    orders.uk.created

As the number of regions grows, this becomes inconvenient.

NATS provides wildcards so the subscriber can express:

    orders.*.created

This means:

> Match any single token in this position.

Now one subscription can cover all regions.

---

# 10. NATS Has Two Wildcards

NATS provides two wildcard tokens:

    *

and:

    >

They have different meanings.

The most important distinction is:

    * = exactly ONE token

    > = ONE OR MORE tokens, including everything remaining

Remember this distinction.

---

# 11. Single-Token Wildcard: *

The `*` wildcard matches exactly one token.

Suppose we subscribe to:

    orders.*.created

This means:

    orders
    [one token]
    created

Therefore:

    orders.us.created
    orders.eu.created
    orders.in.created

all match.

---

# 12. Visualizing *

Think of:

    orders.*.created

as:

    orders
       |
       +---- ANY ONE TOKEN
                    |
                    +---- created

So:

    orders.us.created

becomes:

    orders
    us        <- matched by *
    created

And:

    orders.eu.created

becomes:

    orders
    eu        <- matched by *
    created

Both match.

---

# 13. What * Does NOT Match

The `*` wildcard represents exactly one token.

Therefore:

    orders.*.created

does NOT match:

    orders.created

Why?

Because there is no token available for `*`.

We need:

    orders
    [something]
    created

But we only have:

    orders
    created

There is no middle token.

---

# 14. * Also Does Not Match Multiple Tokens

Consider:

    orders.us.west.created

Now we have:

    orders
    us
    west
    created

But:

    orders.*.created

expects:

    orders
    [ONE TOKEN]
    created

It cannot consume:

    us.west

because that is two tokens.

Therefore:

    orders.*.created

does NOT match:

    orders.us.west.created

This is one of the most important things to understand about `*`.

It is NOT:

    "anything"

It is:

    "exactly one token"

---

# 15. Example of *

Subscription:

    orders.*.created

Messages:

    orders.us.created
    orders.eu.created
    orders.in.created
    orders.created
    orders.us.west.created

Results:

    orders.us.created          -> MATCH
    orders.eu.created          -> MATCH
    orders.in.created          -> MATCH
    orders.created             -> NO MATCH
    orders.us.west.created     -> NO MATCH

Why?

Because `*` represents exactly one token.

---

# 16. Multiple * Wildcards

You can have more than one `*`.

For example:

    orders.*.*

This means:

    orders
    [one token]
    [one token]

Therefore:

    orders.us.created
    orders.eu.created
    orders.us.shipped
    orders.eu.cancelled

can all match.

But:

    orders.created

does not match because it contains only two tokens.

And:

    orders.us.retail.created

does not match because it contains four tokens.

So:

    orders.*.*

matches subjects containing exactly three tokens with:

    orders

as the first token.

---

# 17. Multi-Token Wildcard: >

The `>` wildcard is different.

It matches:

    one or more tokens

and represents:

> Everything from this point onward in the subject hierarchy.

For example:

    orders.>

means:

    orders
    [one or more remaining tokens]

---

# 18. Understanding orders.>

Consider:

    orders.>

It matches:

    orders.created

because:

    orders
    created

Here `>` represents:

    created

It also matches:

    orders.us.created

because:

    orders
    us
    created

Here `>` represents:

    us.created

It also matches:

    orders.us.retail.created

because:

    orders
    us
    retail
    created

Here `>` represents:

    us.retail.created

---

# 19. > Requires At Least One Token

This is important.

Consider:

    orders.>

It does NOT match:

    orders

Why?

Because `>` must match one or more tokens.

There must be something after:

    orders.

So:

    orders.created

MATCH

    orders.us.created

MATCH

    orders.us.retail.created

MATCH

But:

    orders

NO MATCH

---

# 20. > Must Be the Last Token

The `>` wildcard must appear at the end of the subscription pattern.

Valid:

    orders.>

Valid:

    pf.accounts.>

Invalid:

    orders.>.created

Why?

Because `>` represents:

    everything remaining

Once NATS encounters `>` it consumes the rest of the subject.

There is therefore no meaningful way to put another token after it.

So:

    orders.>.created

is invalid.

---

# 21. * vs >

This is the most important comparison.

## *

Means:

    Exactly one token

Example:

    orders.*.created

Matches:

    orders.us.created

Does not match:

    orders.us.retail.created

---

## >

Means:

    One or more remaining tokens

Example:

    orders.>

Matches:

    orders.created

    orders.us.created

    orders.us.retail.created

The key difference is:

    * = one token

    > = everything remaining

---

# 22. Visual Comparison

Suppose the published subject is:

    orders.us.retail.created

Token structure:

    orders | us | retail | created

Now:

    orders.*.created

expects:

    orders | ONE | created

There are too many tokens.

NO MATCH.

But:

    orders.>

expects:

    orders | ONE OR MORE

Therefore:

    us.retail.created

can all be consumed by `>`.

MATCH.

---

# 23. Wildcards Are Subscriber-Only

This is a very important rule.

Wildcards are used by subscribers.

A publisher always publishes to a specific subject.

For example:

    nats pub orders.us.created ...

is a normal publish.

But this:

    nats pub orders.*.created ...

does NOT mean:

> Publish this message to every region.

The `*` is not interpreted as a publishing wildcard.

It becomes part of the literal subject.

In other words:

    orders.*.created

can actually be treated as a literal subject name when publishing.

That is almost certainly not what you intended.

Therefore:

> Wildcards are a subscription matching mechanism, not a broadcast mechanism for publishers.

---

# 24. Correct Way to Publish to Multiple Regions

Suppose we want to publish:

    orders.us.created

and:

    orders.eu.created

The publisher sends two messages:

    publish("orders.us.created", message)

    publish("orders.eu.created", message)

Then a subscriber can use:

    orders.*.created

to receive both.

The publisher is still publishing to concrete subjects.

The subscriber decides how broadly it wants to listen.

---

# 25. Wildcards Do Not Change Delivery Semantics

Suppose:

    Subscriber A
        subscription:
        orders.created

    Subscriber B
        subscription:
        orders.*.created

    Subscriber C
        subscription:
        orders.>

Publisher sends:

    orders.us.created

NATS evaluates the subscriptions.

Subscriber A:

    orders.created

does not match.

Subscriber B:

    orders.*.created

matches.

Subscriber C:

    orders.>

matches.

Therefore:

    Publisher
        |
        | orders.us.created
        v
      NATS
       / \
      /   \
     B     C

Both matching subscribers receive their own copy.

The wildcard only changes:

> Which subjects count as matching this subscription?

It does NOT change:

- pub/sub semantics
- at-most-once delivery
- persistence
- acknowledgements
- replay

Those are separate concepts.

---

# 26. Wildcards Do Not Give You Historical Messages

This is important.

Suppose:

    10:00
    orders.us.created published

At that time:

    no subscriber

Then at:

    10:05

a subscriber starts:

    orders.>

Will it receive the 10:00 message?

NO.

Why?

Because Core NATS is ephemeral.

The wildcard says:

> Match future messages that satisfy this pattern.

It does not mean:

> Search historical messages.

Historical storage/replay is a JetStream concern.

---

# 27. Wildcards and the Interest Graph

Recall the interest graph from the previous chapter.

NATS maintains information about which clients are interested in which subjects.

With exact subscriptions:

    orders.created
         |
         +---- warehouse
         +---- notification

With wildcard subscriptions:

    orders.*.created
         |
         +---- regional analytics

    orders.>
         |
         +---- audit

When a message arrives:

    orders.us.created

NATS evaluates the subject against the subscriptions.

Matching subscribers receive the message.

Therefore wildcards make the subject hierarchy extremely powerful.

---

# 28. Subjects Are Cheap to Create

NATS does not require you to pre-create every subject.

For example, you don't need to run something like:

    CREATE SUBJECT orders.us.created

before publishing.

You can simply publish:

    orders.us.created

Subjects are effectively created through usage.

The important distinction is:

> A subject is not a durable object that you have to provision beforehand.

A subject becomes relevant to the server when there is interest in it.

This makes NATS subject-based addressing lightweight.

---

# 29. Millions of Subjects

Because subjects are lightweight, an application can use a very large number of subjects.

For example:

    tenant.1.orders.created
    tenant.2.orders.created
    tenant.3.orders.created
    ...

However, this does NOT mean:

> Create arbitrary subjects without thinking about the design.

Subject naming still matters.

A poorly designed hierarchy can make subscriptions unnecessarily broad or difficult to manage.

---

# 30. Design Subjects Around How Consumers Need to Filter

This is one of the most useful architectural lessons.

Suppose consumers commonly need:

> All orders by region.

Then putting region into the subject is useful:

    orders.us.created
    orders.eu.created

because we can subscribe:

    orders.*.created

Suppose consumers commonly need:

> Everything belonging to the orders domain.

Then:

    orders.>

is useful.

Therefore:

> Subject structure should reflect the dimensions by which consumers need to subscribe.

This is a very important event-driven architecture principle.

---

# 31. Narrow Subscriptions Are Better Than Overly Broad Subscriptions

Suppose a service only needs:

    orders.*.created

It could subscribe to:

    orders.>

and then filter everything in application code.

But that is usually unnecessarily broad.

For example:

    orders.>

might receive:

    orders.us.created
    orders.us.shipped
    orders.us.cancelled
    orders.eu.created
    orders.eu.shipped
    orders.eu.cancelled
    ...

The service then has to discard most of them.

A better subscription is:

    orders.*.created

if that is exactly what the service needs.

The principle is:

> Subscribe to the narrowest subject pattern that satisfies the requirement.

This reduces unwanted traffic and unnecessary application processing.

---

# 32. Example: Regional Analytics

Suppose the company has:

    US
    EU
    India

orders.

Subjects:

    orders.us.created
    orders.eu.created
    orders.in.created

Regional analytics wants all created orders.

Instead of:

    orders.us.created
    orders.eu.created
    orders.in.created

it can subscribe to:

    orders.*.created

Now when a new region is added:

    orders.uk.created

the same subscriber automatically receives it.

No code change is necessary.

This is one of the strengths of hierarchical subjects.

---

# 33. Example: Audit Service

Suppose the audit service wants every order event.

Subjects:

    orders.us.created
    orders.us.shipped
    orders.us.cancelled
    orders.eu.created
    orders.eu.shipped
    orders.eu.cancelled

The audit service can subscribe to:

    orders.>

Now it receives everything below the `orders` hierarchy.

This is convenient.

But remember:

> Broad subscriptions should be intentional.

An audit service might legitimately need this.

A service that only calculates shipping cost probably should not.

---

# 34. Reserved `$` Prefixes

NATS reserves subjects beginning with:

    $

These are used by NATS and its subsystems.

Examples include:

    $SYS
    $JS
    $KV
    $O
    $SRV

These correspond to various system functionality such as:

- system events
- JetStream
- Key-Value
- Object Store
- Services

Application messages should generally avoid creating their own subjects beginning with `$`.

For example, don't design:

    $mycompany.orders.created

for ordinary application traffic.

Use an application namespace such as:

    mycompany.orders.created

instead.

---

# 35. The _INBOX Prefix

Another reserved subject namespace is:

    _INBOX

This is used by NATS clients for request/reply communication.

For example, conceptually:

    Requester
        |
        | request
        v
      NATS
        |
        v
    Responder
        |
        | response
        v
    _INBOX.xxxxx
        |
        v
    Requester

You normally do not manually invent `_INBOX` subjects for business events.

NATS clients generate them as part of request/reply.

We will study this in detail in the next chapter.

---

# 36. Subjects vs HTTP URLs

This is a useful comparison for your system-design understanding.

HTTP might use:

    POST /orders/123/cancel

NATS might use a subject such as:

    orders.cancel

or:

    orders.cancel.request

The important difference is that a NATS subject is not necessarily tied to one service instance.

HTTP:

    Client
       |
       | URL
       v
    Specific endpoint

NATS:

    Publisher
       |
       | subject
       v
      NATS
       |
       +---- matching service
       +---- matching service
       +---- matching service

The subject is therefore a logical messaging address.

---

# 37. Subjects and Request/Reply

Subjects become even more powerful when we combine them with request/reply.

For example:

    orders.inventory.check

could represent:

> Request the inventory service to check inventory.

A service can subscribe to:

    orders.inventory.check

and respond to the requester.

The next chapter will introduce:

    _INBOX

and explain how the response gets routed back to the correct requester.

---

# 38. Subjects and Queue Groups

Subjects also work together with queue groups.

For example:

    orders.created

might have:

    warehouse-1
    warehouse-2
    warehouse-3

all subscribing as members of the same queue group.

Instead of every instance receiving every message, NATS can distribute messages among the members.

Conceptually:

    Publisher
        |
        | orders.created
        v
      NATS
       / | \
      /  |  \
     W1  W2  W3

Queue-group behavior determines which member processes the message.

This is different from normal wildcard/pub-sub behavior.

We will study queue groups separately.

---

# 39. Common Mistake: Thinking * Means "Anything"

Wrong:

    orders.*.created
    =
    anything between orders and created

Correct:

    orders.*.created
    =
    exactly one token between orders and created

For example:

    orders.us.created

MATCH

But:

    orders.us.retail.created

NO MATCH

because:

    us.retail

contains two tokens.

---

# 40. Common Mistake: Thinking > Means Zero or More

Another common mistake is assuming:

    >

means:

    zero or more tokens

But in the pattern:

    orders.>

the `>` must match one or more tokens.

Therefore:

    orders.created

MATCH

but:

    orders

NO MATCH

---

# 41. Common Mistake: Using > in the Middle

Invalid:

    orders.>.created

`>` must be the final token.

Use:

    orders.*.created

if you need one variable token in the middle.

---

# 42. Common Mistake: Publishing to a Wildcard

Do not think:

    publish("orders.*.created")

means:

    send to all regions

It does not.

The publisher should publish to concrete subjects:

    orders.us.created
    orders.eu.created
    orders.in.created

Subscribers can use:

    orders.*.created

to receive messages across regions.

---

# 43. Common Mistake: Using orders.> Everywhere

It may be tempting to make every service subscribe to:

    orders.>

and filter messages inside application code.

This defeats much of the benefit of subject-based routing.

For example, suppose a service only needs:

    orders.*.created

Then receiving:

    orders.*.shipped
    orders.*.cancelled

is unnecessary.

Better:

    Subscribe narrowly
          |
          v
    NATS filters traffic
          |
          v
    Application processes relevant messages

rather than:

    Subscribe broadly
          |
          v
    Receive everything
          |
          v
    Application filters it

---

# 44. Core NATS Connection to Kafka Topics

Since you already know Kafka, here's the useful comparison.

Kafka has:

    Topic
       |
       +---- Partitions
       |
       +---- Consumer Groups
       |
       +---- Offsets

NATS has:

    Subject
       |
       +---- Subject hierarchy
       |
       +---- Wildcards
       |
       +---- Queue Groups
       |
       +---- Request/Reply

But don't map them one-to-one.

A NATS subject is primarily an addressing mechanism.

A Kafka topic is also a durable stream/log abstraction.

That difference becomes important when we study JetStream.

---

# 45. NATS Subject vs Kafka Topic — Mental Model

A simple mental model is:

    NATS Subject
        =
    Logical address for messaging

while:

    Kafka Topic
        =
    Named durable event stream

This is simplified, but useful.

Core NATS does not provide the same persistence model as Kafka.

For example:

    Kafka
        Producer
            |
            v
        Topic / Partition
            |
            v
        Data remains
            |
            v
        Consumer can read later

Core NATS:

    Publisher
        |
        v
       NATS
        |
        v
    Connected subscriber

If nobody is interested at publish time, there is no durable backlog.

---

# 46. Where NATS Subjects Become Powerful

The combination of:

    subject hierarchy
        +
    wildcards
        +
    pub/sub
        +
    queue groups
        +
    request/reply

allows NATS to support many communication patterns using the same addressing mechanism.

For example:

    orders.us.created

could be consumed by:

    orders.us.created
    orders.*.created
    orders.>

and potentially by queue-group subscribers.

The subject becomes the common language through which services communicate.

---

# 47. Practical Subject Design Example

For your financial workflow, imagine:

    pf.interest.calculated
    pf.distribution.requested
    pf.distribution.completed
    pf.distribution.failed

If you introduce organization:

    pf.org1.interest.calculated
    pf.org2.interest.calculated

Then a subscriber interested in all organizations could potentially use:

    pf.*.interest.calculated

If you instead need all PF events:

    pf.>

This is where subject design should be driven by actual consumer requirements.

---

# 48. Important Design Question

When designing a subject, ask:

> "What dimensions will consumers need to filter on?"

For example:

    domain
    region
    organization
    entity
    event

You might design:

    pf.<organization>.<event>

or:

    pf.<region>.<organization>.<event>

There is no universal correct format.

The important thing is to design a predictable hierarchy that makes subscriptions useful.

---

# 49. Core NATS Subject Mental Model

Keep this mental model:

    SUBJECT
       |
       v
    Hierarchical address
       |
       +---- exact match
       |
       +---- * = one token
       |
       +---- > = remaining tokens
       |
       v
    NATS routing
       |
       v
    Matching subscribers

Remember:

    Publisher
        |
        | concrete subject
        v
      NATS

while:

    Subscriber
        |
        | exact subject / wildcard
        v
      NATS

This distinction is fundamental.

---

# 50. Summary

A NATS subject is:

> A sequence of dot-separated tokens used by NATS to route messages.

Example:

    orders.us.created

contains:

    orders
    us
    created

Subjects are:

- hierarchical
- case-sensitive
- lightweight
- dynamically used
- designed for logical addressing

NATS provides two subscription wildcards:

    *

and:

    >

The rules are:

    * = exactly one token

    > = one or more tokens

Additionally:

    > must be the final token

and:

    wildcards are subscriber-only

---

# 51. Quick Comparison

| Pattern | Meaning | Example |
|---|---|---|
| `orders.created` | Exact subject | Only created orders |
| `orders.*.created` | One variable token | Created orders from any region |
| `orders.>` | Everything below orders | All order events at any depth |
| `orders.*.*` | Exactly two variable tokens | Any three-token subject beginning with orders |
| `orders.>.created` | Invalid | `>` cannot appear in the middle |

---

# 52. What You Should Be Able to Explain

After this chapter, you should be able to answer:

1. What is a NATS subject?

2. What is a subject token?

3. Why does NATS use dot-separated tokens?

4. Are NATS subjects case-sensitive?

5. What does `*` mean?

6. What does `>` mean?

7. What is the difference between `*` and `>`?

8. Why can't `>` appear in the middle of a subject pattern?

9. Can publishers publish to wildcard subjects?

10. What happens when nobody is subscribed to a subject?

11. Does a wildcard subscriber receive historical messages?

12. Why should subscriptions generally be as narrow as possible?

13. What is `orders.*.created` useful for?

14. What is `orders.>` useful for?

15. Why are `$` subjects reserved?

16. What is `_INBOX` used for?

17. How do subjects help decouple services?

18. How does a NATS subject differ conceptually from a Kafka topic?

19. How would you design subjects for a financial workflow?

20. What dimensions would you put into a subject hierarchy?

---

# 53. Practice Lab

## Exercise 1 — Exact Match

Start:

    nats sub orders.created

Then publish:

    nats pub orders.created '{"order_id":"1"}'

Then:

    nats pub orders.shipped '{"order_id":"1"}'

Question:

Which message does the subscriber receive?

---

## Exercise 2 — Single Token Wildcard

Subscribe:

    nats sub "orders.*.created"

Publish:

    nats pub orders.us.created '{"order_id":"1"}'

    nats pub orders.eu.created '{"order_id":"2"}'

    nats pub orders.created '{"order_id":"3"}'

    nats pub orders.us.retail.created '{"order_id":"4"}'

Predict the result BEFORE running it.

Then verify.

---

## Exercise 3 — Multi Token Wildcard

Subscribe:

    nats sub "orders.>"

Publish:

    orders.created
    orders.us.created
    orders.us.retail.created
    orders

Predict which messages arrive.

---

## Exercise 4 — Compare * and >

Create two subscribers:

Terminal 1:

    nats sub "orders.*.created"

Terminal 2:

    nats sub "orders.>"

Then publish:

    orders.us.created
    orders.us.retail.created
    orders.shipped
    orders.us.shipped

Observe the difference.

Write down why each subscriber received what it received.

---

## Exercise 5 — Publisher Wildcard Trap

Try:

    nats pub "orders.*.created" '{"order_id":"1"}'

Then subscribe to:

    nats sub "orders.>"

Observe that the message can be received because `*` is treated as a literal token in the published subject rather than as a publishing wildcard.

This is a useful debugging exercise.

---

## Exercise 6 — Design Subjects

Design subjects for your own financial workflow.

Start with:

    interest.calculated

Then think about:

    organization
    business date
    event type
    processing stage

Come up with a subject hierarchy.

For example:

    pf.<organization>.interest.calculated

Then ask:

> If I want interest-calculated events for every organization, what wildcard subscription would I use?

---

# 54. Key Takeaways

Remember these five things:

### 1. Subjects are addresses

    orders.us.created

is a logical messaging address.

### 2. Subjects are hierarchical

    orders
       |
       +-- us
       |
       +-- created

### 3. `*` means exactly one token

    orders.*.created

### 4. `>` means one or more remaining tokens

    orders.>

### 5. Wildcards are for subscribers

Publishers publish to concrete subjects.

The overall model is:

    Publisher
        |
        | concrete subject
        v
      NATS
        |
        | subject matching
        v
    Subscribers
        |
        +---- exact match
        +---- * wildcard
        +---- > wildcard

Subjects and wildcards form the addressing layer on top of which the other Core NATS communication patterns are built.

The next major concept is:

    Request / Reply

where we move from:

    Publisher
        |
        v
    Subscriber

to:

    Requester
        |
        v
      NATS
        |
        v
    Responder
        |
        v
      NATS
        |
        v
    Requester

That is where `_INBOX`, request timeouts, no-responders detection, and internal service-to-service RPC become important.