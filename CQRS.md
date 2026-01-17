# Design Patterns - CQRS

## What is CQRS?

CQRS = Command Query Responsibility Segregation

Reads and Writes are handled separately.

Command = Write Operations, like Create, Update, Delete
We should write seperate service/handler for write operations.

Command -> Command Handler -> Write Model -> Write Database

Query = Read Operations, like Get, List
We should write seperate service/handler for read operations.

Query -> Query Handler -> Read Model -> Read Database

Command (Write operations) & Query (Read operations) are completely different pathways,  
they don't cross/interfere each other. So it is easy to seperate them.  

If system is becoming slow as traffic is increasing,  
it could be because we are not separating reads and writes.

## Examples

Example #1 - E-commerce Platform  
For example, in Amazon/Flipkart or any e-commerce platform,  
placing an order (write operations) is different from browsing products (read operations).  

Example #2 - Banking System  
For example, in a banking system, the command side handles account updates and transactions,  
while the query side retrieves transaction history or account balances.  

## Benefits

By separating these two operations,  
we can optimize each part of the system independently.

This pattern improves scalability, especially when reads significantly outnumber writes or  
when different parts of the system require eventual consistency.

## How to implement CQRS?

We can have seperate databases for reads and writes,
allowing for better performance and scalability.

Command -> Command Handler -> Write Model -> Write Database  

Event Sourcing

Query -> Query Handler -> Read Model -> Read Database

CQRS implementation key point is - Commands don't directly change data,  
but they trigger actions that lead to changes.

We might opt for simple and cost effective S3 Buckets for the Command/Write Database,  
and select a database with superior query capabilities such as Elastic Search for the  
Query/Read Database.  

A relational SQL database might be better fit for the Command/Write Database,  
while a NOSQL database could be more suitable for the Query/Read Database.  

Now because we are using different databases it's not possible to commit changes  
to both the Write and Read models in a single Atomic transaction.  

Typically changes made to the Write models are asynchronously propagated to the  
Read models through messaging or events resulting in "Eventual Consistency".  

**Problem with this approach?**  

However with this approach it can be challenging to keep everything in sync with using  
seperate Read and Write databases and "Eventual Consistency".  

Because the order or events published from the Write to the Read database becomes really  
important. Imagine that the same Write model instances are updated twice in close succession.  

If the first update event is delivered after the second event then our Read model might  
be updated with a stale/old data.  

Unfortunately most asynchronous message buses are designed for high availability and  
performance which means they don't guarantee that messages will be delivered in the order  
they were published.

We can solve this problem by using "Event Sourcing" pattern.

To keep data consistent between the two databases,
we can use event sourcing or messaging systems to propagate changes  
from the write database to the read database.  
For that we can use tools like Kafka, RabbitMQ, Solace etc.

## References

- [https://www.youtube.com/shorts/jclKKE8esiw](https://www.youtube.com/shorts/jclKKE8esiw)
- [Mastering CQRS in Just 5 Minutes](https://www.youtube.com/watch?v=SvjdJoNPcHs&list=PLJq-63ZRPdBsPWE24vdpmgeRFMRQyjvvj&index=13)
- [Master Event Sourcing in Just 10 Minutes](https://www.youtube.com/watch?v=ID-_ic1fLkY&list=PLJq-63ZRPdBsPWE24vdpmgeRFMRQyjvvj&index=12)

---

## 1. What?

### What is CQRS?

CQRS is a design pattern where **write operations (Commands)** and **read operations (Queries)** are **handled by different models**.

### Simple definition

> Commands **change state**, Queries **read state** — and they are **separated**.

---

## 2. Why?

### Why use CQRS in microservices?

In many systems:

* Reads and writes have **very different requirements**
* Read traffic is often much higher than write traffic
* Optimizing one model hurts the other

CQRS allows **independent optimization** of reads and writes.

---

### What problems does it solve?

* Complex domain logic mixed with read queries
* Poor read performance due to normalized schemas
* Scaling reads and writes independently
* Overloaded transactional databases

---

## 3. Core Concepts

### Commands

* Intention to change system state
* Imperative
* Examples:

  * `CreateOrder`
  * `CancelOrder`
  * `UpdatePaymentStatus`

### Queries

* Read-only requests
* No side effects
* Examples:

  * `GetOrderById`
  * `ListOrdersForUser`

---

## 4. CQRS Architecture

### Logical separation

* **Command side**

  * Validates business rules
  * Performs writes
  * Uses normalized schema
* **Query side**

  * Optimized for reads
  * Uses denormalized views
  * Often uses different database

> CQRS does **not** require separate databases, but it often benefits from them.

---

## 5. How CQRS Works (Typical Flow)

1. Client sends a **command**
2. Command handler:

   * Validates business rules
   * Writes to database
3. Domain event is emitted
4. Read model is updated asynchronously
5. Client queries the read model

**Result: Eventual consistency**

---

## 6. Node.js Example (TypeScript + MongoDB)

### Command side (Write model)

```ts
// command/CreateOrderCommand.ts
export interface CreateOrderCommand {
  orderId: string;
  userId: string;
  amount: number;
}
```

```ts
// command/OrderCommandHandler.ts
import { OrderWriteModel } from './orderWriteModel';

export async function createOrder(cmd: CreateOrderCommand) {
  await OrderWriteModel.create({
    _id: cmd.orderId,
    userId: cmd.userId,
    amount: cmd.amount,
    status: 'CREATED'
  });
}
```

---

### Event publishing

```ts
// events/OrderCreatedEvent.ts
export const OrderCreatedEvent = {
  type: 'OrderCreated'
};
```

---

### Query side (Read model)

```ts
// query/OrderReadModel.ts
import mongoose from 'mongoose';

const OrderReadSchema = new mongoose.Schema({
  orderId: String,
  userId: String,
  amount: Number,
  status: String
});

export const OrderReadModel =
  mongoose.model('OrderRead', OrderReadSchema);
```

```ts
// query/OrderProjection.ts
export async function onOrderCreated(event: any) {
  await OrderReadModel.create({
    orderId: event.orderId,
    userId: event.userId,
    amount: event.amount,
    status: 'CREATED'
  });
}
```

---

## 7. CQRS vs Traditional CRUD

| Aspect             | CRUD    | CQRS     |
| ------------------ | ------- | -------- |
| Read/write model   | Same    | Separate |
| Complexity         | Low     | Higher   |
| Scalability        | Limited | High     |
| Performance tuning | Hard    | Easy     |
| Consistency        | Strong  | Eventual |

---

## 8. CQRS and Event-Driven Architecture

CQRS **often uses events**:

* Commands produce events
* Events update read models
* Events integrate with Saga

However:

> **CQRS ≠ Event Sourcing**

They are often used together, but are independent patterns.

---

## 9. Pros

* Independent scaling of reads and writes
* Better performance for read-heavy systems
* Clear separation of concerns
* Simplifies complex business logic
* Works well with microservices

---

## 10. Cons

* Increased architectural complexity
* Eventual consistency
* More infrastructure (events, projections)
* Harder debugging and testing
* Not suitable for simple CRUD apps

---

## 11. When NOT to use CQRS

* Small applications
* Simple CRUD systems
* Teams new to distributed systems
* Strong consistency is mandatory

---

## 12. Common Interview Follow-Up Questions (With Answers)

---

### 1. Is CQRS mandatory in microservices?

No.
CQRS is **optional** and should be used only when read/write separation provides clear benefits.

---

### 2. Does CQRS require two databases?

No.
You can use:

* Same database with different schemas
* Different databases for better optimization

---

### 3. CQRS vs Event Sourcing?

| CQRS                   | Event Sourcing         |
| ---------------------- | ---------------------- |
| Separates reads/writes | Stores state as events |
| About models           | About persistence      |
| Can exist alone        | Often used with CQRS   |

---

### 4. How is consistency handled in CQRS?

Through **eventual consistency**.
Read model may lag behind write model.

---

### 5. How do you update the read model?

* Domain events
* Message broker (Kafka, SQS, RabbitMQ)
* Async projections

---

### 6. Can CQRS work without events?

Yes.
But events are the **most common and recommended** approach.

---

### 7. Where should CQRS be applied?

* Complex domains
* Read-heavy systems
* Systems with different read/write performance needs

---

### 8. CQRS vs Database per Service?

* Database per Service is about **data ownership**
* CQRS is about **read/write separation**
  They often complement each other.

---

## 13. One-Minute Interview Summary

> “CQRS separates command and query responsibilities, allowing independent scaling and optimization of reads and writes. It improves performance and clarity in complex, read-heavy microservices but introduces eventual consistency and additional complexity.”
