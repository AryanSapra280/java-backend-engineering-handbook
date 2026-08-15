# State Design Pattern

## 1. What is State Pattern?

The **State Design Pattern** is a behavioral design pattern that allows an object to **change its behavior when its internal state changes**.

### Core idea

> Instead of putting all state-dependent behavior inside one class using `if/else` or `switch`, move the behavior for each state into separate classes.

---

# 2. The Problem

Consider a Vending Machine.

It can be in different states:

```text
NO_COIN
HAS_COIN
DISPENSING
OUT_OF_STOCK
```

The behavior of the machine depends on its current state.

For example:

```text
NO_COIN

insertCoin()  → accept coin
selectItem()  → reject
dispense()    → reject
```

```text
HAS_COIN

insertCoin()  → reject / return coin
selectItem()  → select item
dispense()    → reject
```

```text
DISPENSING

insertCoin()  → reject
selectItem()  → reject
dispense()    → dispense item
```

A naive implementation could become:

```java
public void insertCoin() {

    if (state == NO_COIN) {
        // ...
    } else if (state == HAS_COIN) {
        // ...
    } else if (state == DISPENSING) {
        // ...
    }
}
```

And the same condition checking would be repeated in:

```text
insertCoin()
selectItem()
dispense()
cancel()
refund()
...
```

As the number of states and operations increases, the Context becomes difficult to maintain.

---

# 3. State Pattern Solution

Instead of keeping all state-specific behavior inside the Context:

```text
VendingMachine
    |
    ├── if NO_COIN
    ├── if HAS_COIN
    ├── if DISPENSING
    └── if OUT_OF_STOCK
```

we create separate classes:

```text
VendingMachine
       |
       | currentState
       ↓
VendingMachineState
       |
       ├── NoCoinState
       ├── HasCoinState
       ├── DispensingState
       └── OutOfStockState
```

The Context delegates operations to its current State.

---

# 4. Components of State Pattern

There are three primary components.

## 4.1 Context

The object whose behavior changes.

Example:

```text
VendingMachine
```

The Context maintains a reference to the current State.

```java
class VendingMachine {

    private VendingMachineState state;
}
```

---

## 4.2 State Interface

Defines the operations whose behavior depends on the current state.

```java
interface VendingMachineState {

    void insertCoin(VendingMachine machine);

    void selectItem(VendingMachine machine);

    void dispense(VendingMachine machine);
}
```

---

## 4.3 Concrete States

Each class represents a particular state.

```text
NoCoinState
HasCoinState
DispensingState
OutOfStockState
```

Each state implements the behavior appropriate for that state.

---

# 5. Basic Structure

```text
                    Context
                VendingMachine
                      |
                currentState
                      |
                      ↓
              State Interface
                      |
          ┌───────────┼───────────┐
          ↓           ↓           ↓
       State A      State B      State C
```

The key relationship is:

```text
Context HAS-A State
```

This is **composition**.

---

# 6. Context Implementation

```java
class VendingMachine {

    private VendingMachineState state;

    public VendingMachine() {
        this.state = new NoCoinState();
    }

    public void setState(VendingMachineState state) {
        this.state = state;
    }

    public VendingMachineState getState() {
        return state;
    }

    public void insertCoin() {
        state.insertCoin(this);
    }

    public void selectItem() {
        state.selectItem(this);
    }

    public void dispense() {
        state.dispense(this);
    }
}
```

Notice:

```java
public void insertCoin() {
    state.insertCoin(this);
}
```

The Context exposes the operation, but the **current State determines how that operation behaves**.

---

# 7. NoCoinState

```java
class NoCoinState implements VendingMachineState {

    @Override
    public void insertCoin(VendingMachine machine) {

        System.out.println("Coin inserted");

        machine.setState(new HasCoinState());
    }

    @Override
    public void selectItem(VendingMachine machine) {

        System.out.println("Please insert a coin first");
    }

    @Override
    public void dispense(VendingMachine machine) {

        System.out.println("Please insert a coin first");
    }
}
```

The important part:

```java
machine.setState(new HasCoinState());
```

The operation causes a state transition.

---

# 8. HasCoinState

```java
class HasCoinState implements VendingMachineState {

    @Override
    public void insertCoin(VendingMachine machine) {

        System.out.println("Coin already inserted");
    }

    @Override
    public void selectItem(VendingMachine machine) {

        System.out.println("Item selected");

        machine.setState(new DispensingState());
    }

    @Override
    public void dispense(VendingMachine machine) {

        System.out.println("Please select an item first");
    }
}
```

Transition:

```text
HAS_COIN
    |
    | selectItem()
    ↓
DISPENSING
```

---

# 9. DispensingState

```java
class DispensingState implements VendingMachineState {

    @Override
    public void insertCoin(VendingMachine machine) {

        System.out.println("Please wait, dispensing item");
    }

    @Override
    public void selectItem(VendingMachine machine) {

        System.out.println("Already dispensing");
    }

    @Override
    public void dispense(VendingMachine machine) {

        System.out.println("Item dispensed");

        machine.setState(new NoCoinState());
    }
}
```

Transition:

```text
DISPENSING
    |
    | dispense()
    ↓
NO_COIN
```

---

# 10. Complete State Flow

```text
                 insertCoin()
      ┌───────────────────────────┐
      │                           ↓
   NO_COIN ───────────────────→ HAS_COIN
      ↑                            |
      |                            |
      |                         selectItem()
      |                            |
      |                            ↓
      └──────── dispense() ─── DISPENSING
```

The important concept is:

```text
Current State
      +
Event / Operation
      ↓
New State
```

---

# 11. State Machine

State Pattern is closely related to a **finite state machine**.

A state machine consists conceptually of:

```text
States
+
Events
+
Transitions
+
State-specific behavior
```

Example:

```text
CREATED
   |
   | pay()
   ↓
PAID
   |
   | ship()
   ↓
SHIPPED
   |
   | deliver()
   ↓
DELIVERED
```

---

# 12. How to Identify State Pattern

When solving an LLD problem, ask:

### Question 1

Does the object have multiple states?

```text
CREATED
PAID
SHIPPED
DELIVERED
```

### Question 2

Does behavior change depending on the current state?

```text
if state == CREATED
    do A

if state == PAID
    do B
```

### Question 3

Are there explicit transitions?

```text
CREATED → PAID → SHIPPED → DELIVERED
```

### Question 4

Are there many state-dependent `if/else` or `switch` statements?

If most of these are true:

```text
Think → STATE PATTERN
```

---

# 13. First Step Before Implementing State Pattern

Before writing classes, **identify the state machine**.

For an Order:

```text
States:

CREATED
PAID
SHIPPED
DELIVERED
CANCELLED
```

Then identify events:

```text
pay()
ship()
deliver()
cancel()
```

Then identify transitions:

```text
CREATED
    ├── pay() ──────→ PAID
    └── cancel() ───→ CANCELLED

PAID
    ├── ship() ─────→ SHIPPED
    └── cancel() ───→ CANCELLED

SHIPPED
    └── deliver() ──→ DELIVERED
```

Only after understanding this should we create the State classes.

---

# 14. Important Concern: The State Interface May Become Too Large

A common implementation is:

```java
interface State {

    void insertCoin();

    void selectItem();

    void dispense();

    void cancel();

    void refund();

    void refill();
}
```

But not every state necessarily needs every operation.

For example:

```text
NoCoinState

insertCoin() → meaningful
selectItem() → reject
dispense()   → reject
```

while:

```text
DispensingState

insertCoin() → reject
selectItem() → reject
dispense()   → meaningful
```

If the interface keeps growing, the State classes may be forced to implement operations that aren't meaningful to them.

This can become a design smell.

---

# 15. Does This Violate Liskov Substitution Principle?

Not automatically.

LSP asks:

> Can every implementation of the State abstraction be substituted wherever the State abstraction is expected without violating its contract?

For example, if the contract is:

```text
selectItem()
→ perform selection if allowed by the current state
→ otherwise reject the operation
```

then:

```text
NoCoinState.selectItem()
    → "Please insert a coin first"
```

is valid behavior.

It doesn't necessarily violate LSP.

However, if the interface promises something stronger, such as:

```text
selectItem() MUST successfully select an item
```

then `NoCoinState` cannot satisfy that contract.

That would be a genuine LSP problem.

---

# 16. The Bigger Design Smell: Interface Segregation

If the State interface becomes:

```java
interface State {

    void insertCoin();

    void selectItem();

    void dispense();

    void cancel();

    void refund();

    void refill();

    void maintain();

    void shutdown();
}
```

and concrete states don't meaningfully support many of these methods, then the abstraction is probably too broad.

This is more closely related to the:

> **Interface Segregation Principle (ISP)**

than automatically being an LSP violation.

Possible solutions include:

* splitting interfaces
* redesigning operations
* keeping unsupported operations as valid rejected transitions
* moving some operations to the Context
* using a different state-machine design

The right choice depends on the domain.

---

# 17. Important Concern: Hardcoded State Transitions

The simple State implementation often does this:

```java
class NoCoinState implements State {

    public void insertCoin(VendingMachine machine) {

        machine.setState(new HasCoinState());
    }
}
```

This means:

```text
NoCoinState
     ↓
knows
     ↓
HasCoinState
```

The Concrete State knows its successor.

This introduces coupling.

---

# 18. Does This Violate OCP?

Not necessarily.

Suppose the requirement changes from:

```text
NO_COIN
   |
insertCoin()
   ↓
HAS_COIN
```

to:

```text
NO_COIN
   |
insertCoin()
   ↓
COIN_VALIDATION
   |
validation successful
   ↓
HAS_COIN
```

With the simple implementation, we would modify:

```java
NoCoinState
```

from:

```java
machine.setState(new HasCoinState());
```

to:

```java
machine.setState(new CoinValidationState());
```

So yes:

> The simple State implementation has concrete-state coupling, and transition changes can require modifications to existing State classes.

But this does **not automatically mean the design violates OCP**.

OCP means:

> Software entities should generally be open for extension and closed for modification.

Changing an existing business rule or transition is inherently a modification somewhere.

The important question is:

> **Where should transition knowledge live?**

---

# 19. Simple State Pattern — State Controls Transition

The simplest design is:

```text
NoCoinState
      |
      └──→ HasCoinState
```

Code:

```java
class NoCoinState {

    public void insertCoin(VendingMachine machine) {

        machine.setState(new HasCoinState());
    }
}
```

### Advantages

* Very simple
* Easy to understand
* State transition is close to the business rule
* Good for small state machines
* Less infrastructure

### Disadvantages

* States know other concrete states
* Transition graph is distributed
* Changing transitions can require modifying State classes
* Large state machines can become difficult to manage

For a small LLD problem, this is often completely acceptable.

---

# 20. More Scalable Approach — Separate Transitions

For a complex state machine, we can separate:

```text
State-specific behavior
```

from:

```text
State transitions
```

Conceptually:

```text
                 Context
                    |
                    ↓
               Current State
                    |
                    ↓
             State-specific logic


              State Machine
                    |
                    ↓
            Transition Rules
                    |
        ┌───────────┼───────────┐
        ↓           ↓           ↓
    NO_COIN      HAS_COIN    DISPENSING
```

A transition table could conceptually be:

```java
Map<State, Map<Event, State>> transitions;
```

For example:

```java
transitions.put(
    NO_COIN,
    Map.of(
        INSERT_COIN, HAS_COIN
    )
);
```

Then:

```java
State nextState =
    transitions
        .get(currentState)
        .get(event);
```

Now the transition:

```text
NO_COIN + INSERT_COIN → HAS_COIN
```

is represented separately.

---

# 21. Why Separate Transitions?

Suppose the workflow changes:

```text
NO_COIN
   |
INSERT_COIN
   ↓
HAS_COIN
```

becomes:

```text
NO_COIN
   |
INSERT_COIN
   ↓
COIN_VALIDATION
```

We can change the transition configuration rather than modifying `NoCoinState`.

This gives better separation:

```text
State
    → What behavior exists in this state?

Transition
    → Where should we go after an event?
```

---

# 22. But Don't Overengineer

This is extremely important.

For a vending machine with:

```text
3–5 states
```

building a generic:

```text
StateMachine
TransitionEngine
TransitionRepository
EventResolver
```

may be unnecessary.

A simple:

```java
machine.setState(new HasCoinState());
```

may be the better design.

For a workflow with:

```text
15+ states
many events
many transitions
permissions
retries
failure paths
dynamic workflows
```

a dedicated transition model may be justified.

### Design principle

> **Use the simplest design that handles the actual complexity of the problem.**

Don't introduce a framework because OCP sounds theoretically better.

---

# 23. Strategy vs State

This is one of the most important comparisons.

## Strategy

> **Which algorithm/behavior should I use?**

Example:

```text
Payment
   |
   ├── UPI
   ├── Credit Card
   └── Cash
```

The client/application may select:

```java
payment.setStrategy(new UpiStrategy());
```

The strategy is usually an interchangeable algorithm.

---

## State

> **What behavior should this object have because of its current state?**

Example:

```text
Order

CREATED
   ↓
PAID
   ↓
SHIPPED
   ↓
DELIVERED
```

The object's behavior evolves as its state changes.

### Easy memory trick

```text
Strategy → HOW should I do this?

State → WHAT can I do right now?
```

Or:

```text
Strategy → interchangeable behavior

State → evolving behavior
```

---

# 24. State vs Enum

You might ask:

> Why not simply use an enum?

For simple cases, that's completely valid:

```java
enum OrderStatus {
    CREATED,
    PAID,
    SHIPPED,
    DELIVERED
}
```

And:

```java
if (status == CREATED) {
    ...
}
```

may be enough.

State Pattern becomes useful when:

```text
Number of states increases
+
State-specific behavior becomes complex
+
State transitions become important
+
if/else or switch logic becomes difficult to maintain
```

Don't use State Pattern just because an enum exists.

---

# 25. State Pattern vs Simple State Field

There are two different approaches.

### Simple

```java
class Order {

    private OrderStatus status;

    public void ship() {

        if (status == PAID) {
            status = SHIPPED;
        }
    }
}
```

### State Pattern

```text
Order
  |
  ↓
OrderState
  |
  ├── CreatedState
  ├── PaidState
  ├── ShippedState
  └── DeliveredState
```

Use the second when state-specific behavior becomes sufficiently complex.

---

# 26. Who Should Control State Transitions?

There are two common approaches.

## Approach 1 — State controls transition

```java
class CreatedState {

    public void pay(Order order) {

        // business logic

        order.setState(new PaidState());
    }
}
```

Advantages:

* Simple
* Transition is close to business logic
* Easy for small state machines

---

## Approach 2 — State Machine controls transition

The State handles behavior and produces an event/result.

The State Machine determines:

```text
Current State
+
Event
↓
Next State
```

This is useful for complex workflows.

Neither approach is universally correct.

---

# 27. State Pattern Example — Order

Possible states:

```text
CREATED
PAID
SHIPPED
DELIVERED
CANCELLED
```

Possible events:

```text
pay()
ship()
deliver()
cancel()
```

Transition graph:

```text
             pay()
CREATED ─────────────→ PAID
   |                    |
   | cancel()           | ship()
   ↓                    ↓
CANCELLED            SHIPPED
                        |
                        | deliver()
                        ↓
                    DELIVERED
```

Now ask:

> What behavior is different in each state?

For example:

```text
CREATED
    pay()    → allowed
    ship()   → rejected
    cancel() → allowed

PAID
    pay()    → rejected
    ship()   → allowed
    cancel() → maybe allowed

SHIPPED
    pay()    → rejected
    ship()   → rejected
    cancel() → maybe not allowed

DELIVERED
    everything → mostly terminal
```

This is where State Pattern becomes useful.

---

# 28. Real-World Examples

State Pattern is commonly applicable to:

### Order

```text
CREATED
PAID
SHIPPED
DELIVERED
CANCELLED
```

### Vending Machine

```text
NO_COIN
HAS_COIN
DISPENSING
OUT_OF_STOCK
```

### ATM

```text
IDLE
CARD_INSERTED
PIN_VERIFIED
TRANSACTION
```

### Media Player

```text
PLAYING
PAUSED
STOPPED
```

### Document Workflow

```text
DRAFT
IN_REVIEW
APPROVED
PUBLISHED
REJECTED
```

### Traffic Light

```text
RED
GREEN
YELLOW
```

---

# 29. State Pattern and SOLID

State Pattern commonly helps with:

### Single Responsibility Principle

Each State class handles behavior associated with one state.

Instead of:

```text
Order
    → all state behavior
```

we get:

```text
CreatedState
    → Created behavior

PaidState
    → Paid behavior

ShippedState
    → Shipped behavior
```

---

### Open/Closed Principle

Adding a completely new state can often be done by:

```text
Create NewState
+
Add necessary transition
```

without modifying all existing state behavior.

However, transition management determines how strongly OCP is achieved.

---

### Liskov Substitution Principle

Concrete states should respect the contract of the State abstraction.

Different behavior is expected.

Different behavior is **not automatically an LSP violation**.

The issue occurs when a State implementation cannot satisfy the abstraction's contract.

---

### Interface Segregation Principle

Be careful with a huge State interface.

If states are forced to implement many unrelated operations, the abstraction may be too broad.

---

# 30. State Pattern — Pros

### 1. Removes large conditional logic

Instead of:

```text
if state == A
else if state == B
else if state == C
```

we use:

```text
State A
State B
State C
```

### 2. Localizes state-specific behavior

Each state owns its behavior.

### 3. Makes state transitions explicit

```text
A → B → C
```

### 4. Easier to extend with new states

A new State class can encapsulate the new behavior.

### 5. Makes complex workflows easier to reason about

Especially when the lifecycle is important.

---

# 31. State Pattern — Cons

### 1. More classes

Instead of one class:

```text
Order
```

we may have:

```text
Order
CreatedState
PaidState
ShippedState
DeliveredState
CancelledState
```

### 2. Transition coupling

If states directly create their successors:

```java
new PaidState()
```

they become coupled.

### 3. Interface can become large

If many operations are state-dependent, the State interface may become unwieldy.

### 4. Can be overengineering

For simple state-dependent behavior, an enum and a few conditionals may be better.

---

# 32. The Correct Way to Think About State Pattern

Don't start with:

> "I need to use State Pattern."

Start with:

### Step 1 — Identify the entity

```text
Order
VendingMachine
ATM
```

### Step 2 — Identify its states

```text
CREATED
PAID
SHIPPED
...
```

### Step 3 — Identify operations/events

```text
pay()
ship()
cancel()
```

### Step 4 — Identify state-dependent behavior

```text
What can this operation do in each state?
```

### Step 5 — Identify transitions

```text
State + Event → Next State
```

### Step 6 — Decide implementation

Simple:

```text
State classes own transitions
```

Complex:

```text
Separate transition management
```

### Step 7 — Check SOLID

Ask:

```text
Is the State interface too broad?
Are implementations respecting the contract?
Are transitions too tightly coupled?
Am I overengineering?
```

---

# 33. Pattern Recognition Cheat Sheet

```text
Need different algorithms?
        ↓
     STRATEGY

Need different object implementations?
        ↓
      FACTORY

Need complex object construction?
        ↓
      BUILDER

Need exactly one instance?
        ↓
     SINGLETON

Behavior changes based on object's state?
        ↓
       STATE
```

---

# 34. Strategy vs State vs Factory vs Builder vs Singleton

| Pattern   | Main Question                                            | Category   |
| --------- | -------------------------------------------------------- | ---------- |
| Factory   | Which object should I create?                            | Creational |
| Builder   | How should I construct this complex object?              | Creational |
| Singleton | Should there be only one instance?                       | Creational |
| Strategy  | Which algorithm/behavior should I use?                   | Behavioral |
| State     | What behavior should I have because of my current state? | Behavioral |

---

# 35. Interview Answer — State Pattern

If asked:

> "Explain the State Design Pattern."

A strong answer:

> "State is a behavioral design pattern used when an object's behavior changes based on its current state. Instead of maintaining large conditional logic in the Context, we encapsulate state-specific behavior into separate State classes. The Context maintains a reference to the current State and delegates operations to it. States can transition the Context to another state. For simple state machines, transitions can be handled inside the State classes; for more complex workflows, transition management can be separated into a state machine or transition table."

---

# 36. Interview Recognition Formula

When you see:

```text
Multiple states
      +
Different behavior per state
      +
State transitions
      +
Growing if/else or switch
```

think:

```text
STATE PATTERN
```

But then ask:

```text
Is the state logic actually complex enough?

        YES → State Pattern

        NO → Enum + simple logic may be enough
```

---

# 37. Final Mental Model

The entire pattern can be remembered as:

```text
                    CONTEXT
                       |
                       | has
                       ↓
                 CURRENT STATE
                       |
                       | delegates
                       ↓
               STATE INTERFACE
                       |
          ┌────────────┼────────────┐
          ↓            ↓            ↓
       STATE A       STATE B       STATE C
          |            |            |
          └────────────┼────────────┘
                       |
                  transitions
```

And the core formula is:

```text
Current State
      +
Event / Operation
      ↓
State-specific behavior
      ↓
Possibly transition
      ↓
New State
```

---

# 38. The Important Design Tradeoff

The basic implementation:

```java
machine.setState(new HasCoinState());
```

is **not wrong**.

It deliberately trades:

```text
simplicity
+
readability
```

for:

```text
some coupling between states
```

If the state machine becomes complex, we can evolve the design:

```text
Simple
    ↓
State classes own transitions

More complex
    ↓
Separate transition management

Very complex
    ↓
Dedicated state machine / transition model
```

Don't optimize for theoretical OCP/LSP purity before the problem requires it.

> **Good LLD is about choosing the right level of abstraction for the complexity you actually have.**

---

# 39. Final Takeaways

Remember these points:

1. **State Pattern encapsulates state-dependent behavior.**

2. **The Context exposes operations and delegates them to the current State.**

3. **First identify the states, then events, then transitions, then behavior.**

4. **State-specific behavior does not automatically mean LSP is violated.**

5. **A very large State interface can indicate an Interface Segregation problem.**

6. **The simple State Pattern often hardcodes transitions between concrete states.**

7. **That coupling is a tradeoff, not automatically an OCP violation.**

8. **For complex workflows, transition management can be separated from State behavior.**

9. **Don't use State Pattern just because you have an enum.**

10. **Use State when state-dependent behavior and transitions become sufficiently complex.**

11. **Strategy asks: "Which behavior/algorithm should I use?"**

12. **State asks: "What behavior should I have because of my current state?"**

---

# 40. Current Pattern Progress

```text
LLD DESIGN PATTERNS

CREATIONAL
│
├── Factory      ✅
├── Builder      ✅
└── Singleton    ✅

BEHAVIORAL
│
├── Strategy     ✅
└── State        ✅
```

The important next step is no longer simply learning more patterns.

Now the goal is:

```text
Problem
   ↓
Identify responsibilities
   ↓
Identify object creation
   ↓
Identify changing behavior
   ↓
Identify state/lifecycle
   ↓
Choose pattern only where useful
   ↓
Design interfaces/classes
   ↓
Check SOLID
   ↓
Code
```

That is the actual LLD interview skill.
