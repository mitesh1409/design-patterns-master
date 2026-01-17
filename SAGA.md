# Microservices Design Patterns -> SAGA

SAGA is a Design Pattern used to handle **Distributed Transactions** in Microservices.

## Introduction to Distributed Transactions

The core idea behind Distributed Transactions is to ensure that  
all operations within the transaction either succeed together or fail together.  
This guarantees data consistency across different part of the system.

**Monolith**  

In an application based on Monolith architecture we have a single database.

Consider a flight booking system, a user journey may look like the following:  

1. Reserve one or more seats -> flight_reservations, tickets
2. Complete payment -> payment_transactions
3. Book one or more hotel rooms -> hotel_bookings
4. Rent a car for transportation -> transportation_bookings
5. Notifications -> notifications

Now what if one of these operations fails.  
For example,  

- we don't want to deduct user's money without a confirmed flight ticket.
- we don't want to reserve a seat if payment fails.
etc.

Operations 1 to 5 can be done within a single database transaction,  
so if any one of them fails we rollback all other operations.

Single big atomic transaction, which can be rolled back if anything goes wrong.

In a Monolith application with single database we can do this easilty using database transactions,  
since everything (all data/tables) is at one place.

Database Management Systems (DBMS) use ACID (Atomicity, Consistency, Isolation, Durability) to manage transactions within a single database.

This is how we can keep application data consistent in a Monolith application.

---

**Microservices**  

In an application based on Microservices architecture we have multiple databases,  
each service has its own database.

Consider the same flight booking system, a user journey may look like the following:  

1. Reserve one or more seats -> Flight Ticket Reservation Service
2. Complete payment -> Payment Service
3. Book one or more hotel rooms -> Hotel Room Booking Service
4. Rent a car for transportation -> Transportation Service
5. Notifications -> Notification Service

Now what if one of these operations fails.  
For example,  

- we don't want to deduct user's money without a confirmed flight ticket.
- we don't want to reserve a seat if payment fails.
etc.

Here operations are happening in their own service & database as described above,  
we are having distributed transactions here.

Distributed transactions are transactions that span multiple microservices and databases,  
requiring coordination over a network to maintain consistency.  
They are called distributed because the transaction logic and data are spread across  
multiple microservices and databases.  
All the operations involved in the distributed transaction must either succeed or  
fail together to maintain data consistency.

Since a distributed transaction span multiple databases,  
we cannot simply rollback it like we do in case of Monolith.

Common approaches to handle distributed transactions are:  

1. Two-Phase Commit (2PC) Protocol
2. SAGA Pattern (Most Common)
3. Eventual Consistency

## Two-Phase Commit (2PC) Protocol

* The most common way to implement distributed transactions. It consists of two phases:  

1. **Prepare Phase:** A central coordinator asks all participating services if they can commit.
2. **Commit/Rollback Phase:** If everyone says "yes," the coordinator sends a commit message. If any say "no" (or timeout), it sends a rollback message to undo changes.

* **Implementation:** Often uses tools like **Zookeeper** for coordination.

### Challenges and Drawbacks of 2PC

* **Performance:** Introduces latency due to synchronous communication and coordination steps.
* **Deadlocks:** Services may wait on each other to release resources.
* **Blocking Nature:** If the coordinator fails, participant nodes are stuck waiting, often requiring manual intervention. It is often called an "anti-availability" protocol.
* **Mitigation:** High-availability groups (like Paxos in Google Spanner) can help, but complexity remains high.

## The Saga Pattern (Most commonly used)

SAGA Pattern is used to handle **Distributed Transactions** in Microservices.

SAGA Pattern = local transactions + compensating transactions (to reverse/rollback changes done by local transactions)

Where,  
local transactions = each updating a single service
compensating transactions = to reverse/rollback changes done by local transactions

A compensating transaction will undo changes done by a local transaction.

In SAGA Pattern, we split distributed transactions into a set of local transactions,  
each local transaction has a compensating transaction.  
If anything goes wrong (one of the local transaction fails),  
then we use compensating transactions to reverse/rollback changes done by local transaction.

For example, consider the flight booking system with the following user journey:  

1. Reserve one or more seats
    Flight Ticket Reservation Service  
    local transaction => reserve a seat  
    compensating transaction => release a seat

2. Complete payment
    Payment Service  
    local transaction => Payment Transaction  
    compensating transaction => Refund Transaction

3. Book one or more hotel rooms
    Hotel Room Booking Service  
    local transaction => Book a hotel room  
    compensating transaction => Cancel booking

4. Rent a car for transportation
    Transportation Service  
    local transaction => Book a transportation service  
    compensating transaction => Cancel booking  

5. Notifications
    Notification Service  
    local transaction => Send success notifications (flight tickets booked, hotel room booked, transportation service booked etc.)  
    compensating transaction => Send failure notifications (flight tickets canceled, hotel room booking canceled, transportation service booking canceled etc.)

### Implementation Types: Orchestration vs. Choreography

SAGA can be implemented in two ways:  

1. Orchestration
2. Choreography

#### Orchestrated Saga

One central service/**Saga Orchestrator** controls the flow.

Like a symfony orchestra, there is a central **Saga Orchestrator** who manages the flow,  
it sends explicit commands to services and tracks progress.  
It handles retries and triggers compensations if a service fails.

The **Saga Orchestrator** explicitly tells each service what action to take.  
For example,  
it can ask Flight Ticket Reservation Service to reserve a seat,  
it can ask Payment Service to do payment transaction  
etc.

The **Saga Orchestrator** waits for the response from each service before moving to the next operation.

**Saga Orchestrator** can be a different microservice or it can sit inside any of the existing microservices  
which are part of the distributed transaction.

**Pros**  

- Clear control flow, we know what is happening and when
- Great for debugging and logging

**Cons**  

- SAGA Orchestrator can become a single point of failure, if it goes down the whole flow is blocked.

#### Choreographed Saga

There is no central coordinator.  
Services communicate directly through events.  
Each service listens for specific events and reacts autonomously.

Services dance to the beats of events.

**Pros**  

- Loosely coupled
    Services are independent, they can be deployed separately,  
    they can be scaled separately, react to events in real time.

**Cons**  

- Debugging is tough
    Tracing the full transaction means stitching together events across logs  
    and you need strong guarantees from Kafka, like at least once delivery  
    and idempotency in services.

#### When to use what?

Use **Orchestration** for  

* Simpler imlementation
* Simpler debugging
* Centralized control
* Detailed audit logs/audit trails

Use **Choreography** for  

* Highly distributed scalable system with lots of independent services
* Loosely coupled systems where decentralized logic is preferred

Many companies mix both - **Orchestration** for critical flows like payments and  
**Choreography** for notifications, analytics and less sensitive tasks.

#### Orchestration Vs Choreography

**Orchestration**  
Simpler implementation for individual services and provides a clear audit trail.  
However, the orchestrator can be a single point of failure.

**Choreography**  
More independent and scalable with no central bottleneck, but harder to implement and trace due to decentralized logic.

## 2PC vs. Saga Pattern

| Aspect | Two-Phase Commit (2PC) | Saga |
| :--- | :--- | :--- |
| Coordination | Central Coordinator | Orchestrated (Central Saga) or Choreographed (events) |
| Communication | Synchronous (Blocking) | Asynchronous |
| Atomicity | Strong Atomicity | Eventual consistency, uses compensating transactions |
| Flexibility & Resilience | Less flexible, single point of failure | More flexible, resilient, no single point of failure |
| Performance | Slower, participants need to wait | Faster, independent operations |
| Use Cases | Strong consistency, limited participants | Long-running, complex transactions, multiple services, eventual consistency |

## Summary & Key Pointers

**The Saga Pattern** is a failure-management strategy for distributed systems that ensures data consistency without the heavy performance costs of traditional protocols like Two-Phase Commit.

* **Distributed Transactions:** Necessary when a single business process spans multiple microservices.
* **Two-Phase Commit (2PC):** A synchronous, central coordinator-based approach that guarantees strong consistency but suffers from performance bottlenecks and availability issues.
* **Saga Definition:** A sequence of local transactions where each step is followed by an event/message to trigger the next step or a "compensating transaction" to undo previous work in case of failure.
* **Consistency Model:** Sagas trade **Strong Consistency** for **Eventual Consistency** and higher availability.
* **Key Choice:** * Use **Orchestration** for simpler workflows where centralized control and audit trails are needed.
* Use **Choreography** for high-scale, loosely coupled systems where decentralized logic is preferred.

## References

* [Saga Pattern | Distributed Transactions | Microservices](https://www.youtube.com/watch?v=d2z78guUR4g&list=PLJq-63ZRPdBsPWE24vdpmgeRFMRQyjvvj&index=3)
* [SAGA Pattern Deep Dive | Real-World Example with Kafka + Node.js](https://www.youtube.com/watch?v=43Gez5dWH9w)
* [What Is the Saga Pattern and Why Do Microservices Need It? #microservices](https://www.youtube.com/watch?v=feV_6xk-dsg)

---

## Saga Pattern, Another Answer

## What?

The **Saga Pattern** is used to manage **distributed transactions** across multiple microservices **without using a single ACID transaction**.

A saga is a **sequence of local transactions**, where:

* Each service performs its own transaction
* If one step fails, **compensating actions** are executed to undo previous steps

---

## Why?

### Problem It Solves

In microservices:

* Each service has its **own database**
* Traditional distributed transactions (2PC) are not recommended
* Failures can happen at any step

Saga provides:

* Data consistency across services
* Failure handling without locking databases
* Better scalability and resilience

---

## How?

### Two Saga Approaches

---

### 1. Choreography-Based Saga

* Services communicate via **events**
* No central coordinator
* Each service reacts to events and emits new events

**Flow**

```
Order Created → Payment Processed → Inventory Updated
```

---

### 2. Orchestration-Based Saga

* A **Saga Orchestrator** controls the workflow
* Orchestrator tells each service what to do
* Services reply with success or failure

**Flow**

```
Orchestrator → Order Service → Payment Service → Inventory Service
```

---

### Comparison (Interview Favorite)

| Choreography       | Orchestration       |
| ------------------ | ------------------- |
| Event-driven       | Command-driven      |
| No central control | Central coordinator |
| Harder to trace    | Easier to manage    |
| Loose coupling     | Clear workflow      |

---

## Node.js Example

(TypeScript + Kafka – Choreography Saga)

### Scenario

Order creation saga:

1. Order Service creates order
2. Payment Service processes payment
3. Inventory Service updates stock
4. On failure → compensating actions

---

### Order Service (Start Saga)

```ts
// order-service/src/index.ts
import { Kafka } from "kafkajs";

const kafka = new Kafka({ brokers: ["localhost:9092"] });
const producer = kafka.producer();

async function startOrderSaga(order: any) {
  await producer.connect();
  await producer.send({
    topic: "order-created",
    messages: [{ value: JSON.stringify(order) }],
  });
}
```

---

### Payment Service (Process Payment)

```ts
// payment-service/src/index.ts
import { Kafka } from "kafkajs";

const kafka = new Kafka({ brokers: ["localhost:9092"] });
const consumer = kafka.consumer({ groupId: "payment-group" });
const producer = kafka.producer();

async function start() {
  await consumer.connect();
  await producer.connect();

  await consumer.subscribe({ topic: "order-created" });

  await consumer.run({
    eachMessage: async ({ message }) => {
      const order = JSON.parse(message.value!.toString());

      const paymentSuccess = true; // simulate

      if (paymentSuccess) {
        await producer.send({
          topic: "payment-success",
          messages: [{ value: JSON.stringify(order) }],
        });
      } else {
        await producer.send({
          topic: "payment-failed",
          messages: [{ value: JSON.stringify(order) }],
        });
      }
    },
  });
}

start();
```

---

### Inventory Service (Update Stock or Compensate)

```ts
// inventory-service/src/index.ts
import { Kafka } from "kafkajs";

const kafka = new Kafka({ brokers: ["localhost:9092"] });
const consumer = kafka.consumer({ groupId: "inventory-group" });

async function start() {
  await consumer.connect();
  await consumer.subscribe({ topic: "payment-success" });

  await consumer.run({
    eachMessage: async ({ message }) => {
      const order = JSON.parse(message.value!.toString());
      console.log(`Reducing stock for ${order.productId}`);
    },
  });
}

start();
```

---

### Compensation Example

If payment fails:

* Emit `payment-failed`
* Order Service listens and **cancels the order**

---

## Pros / Cons

### Pros

* Maintains data consistency
* No distributed locking
* Scales well
* Works with Database per Service

### Cons

* Eventual consistency
* Complex error handling
* Harder debugging (especially choreography)
* Requires idempotency and retries

---

## Interview Questions

### Conceptual

1. What is the Saga Pattern?
2. Why is Saga needed in microservices?

### Practical

3. Difference between choreography and orchestration?
4. How do compensating transactions work?

### Advanced

5. Why is 2PC not recommended in microservices?
6. How do you handle failures in Saga?
7. Where does Saga fit with Event-Driven Architecture?

---

## Interview One-Line Summary

> “Saga Pattern manages distributed transactions in microservices by using a sequence of local transactions with compensating actions instead of a global transaction.”

---

## Important Interview Tip

If asked:
**“Which saga type should we use?”**

Correct answer:

> “Choreography for simple flows, orchestration for complex workflows.”

---

Below are **clear, short, and interview-ready answers** for the **Saga Pattern**, written in **simple language** and focused on **what interviewers expect**.

---

## Conceptual

### 1. What is the Saga Pattern?

The Saga Pattern manages **distributed transactions** in microservices by breaking them into **multiple local transactions** and using **compensating actions** to handle failures.

---

### 2. Why is Saga needed in microservices?

Because each microservice has its **own database**, and traditional distributed transactions are not suitable. Saga ensures **data consistency without locking databases**.

---

## Practical

### 3. Difference between choreography and orchestration?

| Choreography              | Orchestration             |
| ------------------------- | ------------------------- |
| Event-based communication | Central coordinator       |
| No single controller      | One service controls flow |
| Loose coupling            | Clear workflow            |
| Harder to debug           | Easier to manage          |

---

### 4. How do compensating transactions work?

If a step in the saga fails, the system runs **compensating actions** to undo previously completed steps, restoring the system to a consistent state.

Example:
If payment fails → cancel order.

---

## Advanced

### 5. Why is 2PC not recommended in microservices?

Because 2PC:

* Causes tight coupling
* Locks resources for long time
* Does not scale well
* Increases failure impact

Microservices prefer **eventual consistency**, not strict ACID transactions.

---

### 6. How do you handle failures in Saga?

By:

* Executing compensating transactions
* Retrying failed steps
* Using idempotent operations
* Monitoring saga state
* Handling partial failures gracefully

---

### 7. Where does Saga fit with Event-Driven Architecture?

Saga often **uses Event-Driven Architecture**:

* Services publish events
* Other services react to events
* Especially common in **choreography-based sagas**

---

## One-Line Interview Summary

> “Saga Pattern maintains data consistency in microservices by coordinating local transactions using events and compensating actions instead of global transactions.”
