# Design Patterns > SAGA

SAGA is a Design Pattern used to handle **Distributed Transactions** in Microservices.

## Introduction to Distributed Transactions

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

## The Saga Pattern: An Alternative

SAGA Pattern is used to handle **Distributed Transactions** in Microservices.

* Sagas are more flexible for microservices. They consist of a sequence of **local transactions**.
* If a local transaction fails, **compensating transactions** are executed to undo the changes made by previous successful steps.
* **Example (Flight Booking):** Steps include reserving a seat, charging a card, and booking a hotel. If charging the card fails, the seat reservation is canceled.

## Implementation Types: Orchestration vs. Choreography

* **Orchestrated Saga:**  
A central **Saga Orchestrator** manages the flow, sending explicit commands to services and tracking progress. It handles retries and triggers compensations if a service fails.
* **Choreographed Saga [[08:55](http://www.youtube.com/watch?v=d2z78guUR4g&t=535)]:** There is no central coordinator. Services communicate directly through events. Each service listens for specific events and reacts autonomously.

## Comparing Orchestration and Choreography

* **Orchestration:** Simpler implementation for individual services and provides a clear audit trail. However, the orchestrator can be a single point of failure.
* **Choreography:** More independent and scalable with no central bottleneck, but harder to implement and trace due to decentralized logic.

## Design Flow of an Orchestrated Saga

1. User initiates a request.
2. Orchestrator commands **Flight Service** (Reserve seat).
3. If successful, Orchestrator commands **Payment Service** (Process payment).
4. If successful, it proceeds to Hotel and Car services.
5. Finally, **Notification Service** sends an email.
6. **Failure Handling:** If payment fails at, the orchestrator triggers a compensation to release the reserved seat in the Flight Service.

## Key Differences: 2PC vs. Saga Pattern

* **Communication:** 2PC is synchronous (blocking); Sagas are primarily asynchronous (event-driven).
* **Consistency:** 2PC provides **Strong Atomicity**; Sagas provide **Eventual Consistency**.
* **Flexibility:** Sagas are more resilient to failures and better for long-running transactions.
* **Resource Locking:** 2PC locks resources (causing contention); Sagas use local transactions without global locking.

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
