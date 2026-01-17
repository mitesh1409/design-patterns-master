# Microservices Design Patterns -> Event Driven Architecture (EDA)

## What?

**Event-Driven Architecture (EDA)** is a design approach where  
**services communicate by publishing and consuming events**  
instead of calling each other directly.

An **event** represents something that already happened:

For example,  

* Order created
* Payment completed
* User registered
etc.

---

## Why?

### Problems with Direct Communication

* Tight coupling between services
* Cascading failures
* Difficult scaling
* Hard to evolve systems

### What EDA Solves

* Loose coupling
* High scalability
* Better fault tolerance
* Asynchronous processing
* Supports eventual consistency

---

## How?

### Core Components

| Component      | Role                                   |
| -------------- | -------------------------------------- |
| Event Producer | Publishes events                       |
| Event Broker   | Delivers events (Kafka, RabbitMQ, SQS) |
| Event Consumer | Reacts to events                       |

---

### Event Flow

```
Producer → Event Broker → Consumer(s)
```

Producers **do not know** who consumes their events.

---

## Node.js Example

(TypeScript + Kafka)

### Scenario

* Order Service publishes `OrderCreated` event
* Email Service and Inventory Service consume it

---

### Order Service (Producer)

```ts
// order-service/src/index.ts
import { Kafka } from "kafkajs";

const kafka = new Kafka({ brokers: ["localhost:9092"] });
const producer = kafka.producer();

async function publishOrderCreated(order: any) {
  await producer.connect();
  await producer.send({
    topic: "order-created",
    messages: [{ value: JSON.stringify(order) }],
  });
}
```

---

### Email Service (Consumer)

```ts
// email-service/src/index.ts
import { Kafka } from "kafkajs";

const kafka = new Kafka({ brokers: ["localhost:9092"] });
const consumer = kafka.consumer({ groupId: "email-group" });

async function start() {
  await consumer.connect();
  await consumer.subscribe({ topic: "order-created" });

  await consumer.run({
    eachMessage: async ({ message }) => {
      const order = JSON.parse(message.value!.toString());
      console.log(`Sending email for order ${order.orderId}`);
    },
  });
}

start();
```

---

### Inventory Service (Consumer)

```ts
// inventory-service/src/index.ts
import { Kafka } from "kafkajs";

const kafka = new Kafka({ brokers: ["localhost:9092"] });
const consumer = kafka.consumer({ groupId: "inventory-group" });

async function start() {
  await consumer.connect();
  await consumer.subscribe({ topic: "order-created" });

  await consumer.run({
    eachMessage: async ({ message }) => {
      const order = JSON.parse(message.value!.toString());
      console.log(`Updating inventory for ${order.productId}`);
    },
  });
}

start();
```

---

### Key Observations

* Producer is unaware of consumers
* Multiple consumers can react to same event
* Loose coupling
* High scalability

---

## Pros / Cons

### Pros

* Loose coupling between services
* Scales very well
* High resilience
* Easy to add new consumers
* Supports Saga and CQRS

### Cons

* Eventual consistency
* Harder debugging
* Requires message broker (additional infra)
* Requires idempotent consumers

---

## Interview Questions

### Conceptual

1. What is Event-Driven Architecture?
2. What is an event?

### Practical

3. When should you use EDA?
4. Difference between event and command?

### Advanced

5. How does EDA help in microservices?
6. What are common challenges in EDA?
7. How does EDA relate to Saga and CQRS?

---

## Interview One-Line Summary

> “Event-Driven Architecture enables loosely coupled, scalable microservices by allowing services to communicate through events instead of direct calls.”

---

## Important Interview Tip

If asked:
**“Is Event-Driven Architecture eventually consistent?”**

Correct answer:

> “Yes, EDA is eventually consistent by nature.”

---

## Please explain the cons which says - "Requires idempotent consumers"  

This is an **excellent and very important question**.
Understanding **idempotent consumers** is a strong signal of **real-world microservices experience**.

---

## What does “Requires idempotent consumers” mean?

In **Event-Driven Architecture**, a message broker (Kafka, RabbitMQ, SQS, etc.) can deliver the **same event more than once**.

An **idempotent consumer** is a consumer that can **process the same event multiple times without changing the final result**.

> Processing an event **once or many times gives the same outcome**.

---

### Why can events be delivered more than once?

This can happen due to:

* Consumer crash after processing but before acknowledging
* Network failures
* Retries by the message broker
* Rebalancing in Kafka

Because of this, **“exactly once” delivery is hard**, so systems assume **“at least once” delivery**.

---

### Problem Without Idempotency (Example)

#### Scenario: Order Created Event

Event:

```json
{
  "eventId": "evt-123",
  "orderId": "order-101",
  "productId": "p1",
  "quantity": 2
}
```

#### Inventory Service (Non-Idempotent ❌)

If the event is received **twice**:

```
First time → Reduce stock by 2
Second time → Reduce stock by 2 again ❌
```

Final result: **Wrong inventory count**

---

### Idempotent Consumer (Correct Behavior)

The consumer must ensure:

> “This event has already been processed.”

---

### Common Idempotency Strategies

#### 1. Event ID Tracking (Most Common)

Store processed event IDs.

---

#### Node.js Example (TypeScript + MongoDB)

#### Inventory Service – Idempotent Consumer

```ts
// inventory-service/src/consumer.ts
import mongoose from "mongoose";
import { Kafka } from "kafkajs";

mongoose.connect("mongodb://localhost:27017/inventory-db");

const ProcessedEventSchema = new mongoose.Schema({
  eventId: { type: String, unique: true },
});
const ProcessedEvent = mongoose.model(
  "ProcessedEvent",
  ProcessedEventSchema
);

const kafka = new Kafka({ brokers: ["localhost:9092"] });
const consumer = kafka.consumer({ groupId: "inventory-group" });

async function start() {
  await consumer.connect();
  await consumer.subscribe({ topic: "order-created" });

  await consumer.run({
    eachMessage: async ({ message }) => {
      const event = JSON.parse(message.value!.toString());

      // Check if event already processed
      const exists = await ProcessedEvent.findOne({
        eventId: event.eventId,
      });

      if (exists) {
        console.log("Duplicate event ignored");
        return;
      }

      // Business logic
      console.log(
        `Reducing stock for ${event.productId} by ${event.quantity}`
      );

      // Save processed event
      await ProcessedEvent.create({ eventId: event.eventId });
    },
  });
}

start();
```

---

#### Key Points in the Example

* `eventId` uniquely identifies an event
* Duplicate events are safely ignored
* Inventory is updated **only once**
* System remains consistent

---

### Other Idempotency Techniques (Interview-Worthy)

| Technique           | Description                              |
| ------------------- | ---------------------------------------- |
| Event ID tracking   | Store processed event IDs                |
| Upserts             | Update only if state not already updated |
| Version checks      | Process only newer versions              |
| Natural idempotency | Same input always leads to same state    |

---

### Relationship to Other Patterns

| Pattern                   | Connection                               |
| ------------------------- | ---------------------------------------- |
| Event-Driven Architecture | Messages may be delivered more than once |
| Saga                      | Compensation relies on idempotent steps  |
| Transactional Outbox      | Can still deliver duplicates             |
| CQRS                      | Read models must be idempotent           |

---

### Interview One-Line Answer

> “An idempotent consumer can safely process the same event multiple times without causing inconsistent state, which is essential because message brokers may deliver duplicate events.”

---

### Common Follow-Up Question (Be Ready)

**Q: Can we guarantee exactly-once processing?**
**A:** Rarely. Most systems use **at-least-once delivery with idempotent consumers**.

---

## Interview Questions

## Conceptual

### 1. What is Event-Driven Architecture?

Event-Driven Architecture is a design style where services **communicate by publishing and consuming events** instead of calling each other directly.

---

### 2. What is an event?

An event is a **fact that something has already happened** in the system, such as `OrderCreated` or `PaymentCompleted`.

---

## Practical

### 3. When should you use Event-Driven Architecture?

Use EDA when:

* Loose coupling is required
* Multiple services react to the same action
* High scalability is needed
* Workflows are asynchronous or long-running

---

### 4. Difference between an event and a command?

| Event                           | Command                   |
| ------------------------------- | ------------------------- |
| Something that already happened | A request to do something |
| Past tense                      | Imperative                |
| No expectation of response      | Expects action            |
| Example: OrderCreated           | Example: CreateOrder      |

---

## Advanced

### 5. How does Event-Driven Architecture help in microservices?

It:

* Reduces tight coupling
* Improves scalability
* Prevents cascading failures
* Allows independent service evolution
* Enables asynchronous workflows

---

### 6. What are common challenges in Event-Driven Architecture?

* Eventual consistency
* Duplicate message handling
* Debugging and tracing
* Message ordering
* Versioning events

---

### 7. How does Event-Driven Architecture relate to Saga and CQRS?

* **Saga** uses events to coordinate distributed transactions
* **CQRS** uses events to update read models
* EDA acts as the **communication backbone** for both patterns

---

## One-Line Interview Summary

> “Event-Driven Architecture enables loosely coupled microservices by allowing services to communicate through events rather than direct calls.”

---

## High-Impact Interview Tip

If asked:
**“Is Event-Driven Architecture synchronous or asynchronous?”**

Correct answer:

> “Event-Driven Architecture is asynchronous by nature.”
