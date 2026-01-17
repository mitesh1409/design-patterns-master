# Microservices Design Patterns -> Inter-Service Communication

Below are **interview-ready, production-focused notes** for **Pattern 5: Inter-Service Communication**, structured exactly as you requested and kept **clear, concise, and practical**.

---

# 5. Inter-Service Communication

---

## What?

**Inter-Service Communication** defines **how microservices talk to each other** to exchange data and coordinate actions.

There are two primary styles:

1. **Synchronous communication** (request/response)
2. **Asynchronous communication** (event/message-based)

---

## Why?

### Problems It Addresses

* Services must collaborate to complete business workflows
* Direct DB sharing is not allowed (Database per Service)
* Services need to stay loosely coupled

### What It Enables

* Independent services working together
* Scalability and resilience
* Clear communication contracts
* Event-driven workflows

---

## How?

### 1. Synchronous Communication

* REST (HTTP/JSON)
* gRPC

**Characteristics**

* Immediate response
* Simple to understand
* Tighter coupling
* Caller waits for response

**Use when**

* Immediate data is required
* Simple request/response flows

---

### 2. Asynchronous Communication

* Message broker (Kafka, RabbitMQ, AWS SQS, Redis Streams)

**Characteristics**

* Non-blocking
* Loose coupling
* Eventual consistency
* Better resilience

**Use when**

* Decoupling services
* Long-running workflows
* Event-driven systems

---

### Comparison (Interview Favorite)

| Synchronous        | Asynchronous           |
| ------------------ | ---------------------- |
| Blocking           | Non-blocking           |
| Tight coupling     | Loose coupling         |
| Immediate response | Eventual consistency   |
| REST / gRPC        | Kafka / SQS / RabbitMQ |

---

## Node.js Example

(TypeScript + REST + Kafka)

### Scenario

* **Order Service** creates an order
* **Inventory Service** updates stock
* Communication:

  * REST for validation
  * Kafka event for stock update

---

### Order Service (REST + Kafka Producer)

```ts
// order-service/src/index.ts
import express from "express";
import { Kafka } from "kafkajs";

const app = express();
app.use(express.json());

const kafka = new Kafka({ brokers: ["localhost:9092"] });
const producer = kafka.producer();

async function initKafka() {
  await producer.connect();
}
initKafka();

app.post("/orders", async (req, res) => {
  const order = {
    orderId: Date.now().toString(),
    productId: req.body.productId,
    quantity: req.body.quantity,
  };

  // Publish event
  await producer.send({
    topic: "order-created",
    messages: [{ value: JSON.stringify(order) }],
  });

  res.status(201).json({ message: "Order created", order });
});

app.listen(3002, () => {
  console.log("Order Service running on port 3002");
});
```

---

### Inventory Service (Kafka Consumer)

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
      console.log(
        `Reducing stock for product ${order.productId} by ${order.quantity}`
      );
    },
  });
}

start();
```

---

### Key Observations

* Services are **not directly calling each other**
* Order Service does not know Inventory Service exists
* Kafka provides loose coupling
* Supports eventual consistency

---

## Pros / Cons

### Pros

* Enables service collaboration
* Supports scalability
* Loose coupling with async messaging
* Better fault tolerance

### Cons

* Network latency
* Async flows are harder to debug
* Data consistency becomes complex
* Requires observability tooling

---

## Interview Questions

### Conceptual

1. What is Inter-Service Communication?
2. What are the types of inter-service communication?

### Practical

3. When would you use synchronous vs asynchronous communication?
4. How do services communicate without sharing databases?

### Advanced

5. How does inter-service communication impact system reliability?
6. How do you handle failures in synchronous communication?
7. Why is async communication preferred in microservices?

---

## Interview One-Line Summary

> “Inter-Service Communication defines how microservices interact using synchronous APIs or asynchronous events while remaining loosely coupled and independently scalable.”

---

## Important Interview Tip

If asked:
**“Which is better: synchronous or asynchronous?”**

Correct answer:

> “It depends on the use case. Synchronous is simpler; asynchronous is more resilient and scalable.”

---

## Interview Questions

## Conceptual

### 1. What is Inter-Service Communication?

Inter-Service Communication is the way **microservices exchange data and coordinate actions** using APIs or messages.

---

### 2. What are the types of inter-service communication?

There are two main types:

* **Synchronous communication** (REST, gRPC)
* **Asynchronous communication** (Kafka, RabbitMQ, AWS SQS)

---

## Practical

### 3. When would you use synchronous vs asynchronous communication?

* Use **synchronous** communication when:

  * Immediate response is required
  * Simple request-response logic

* Use **asynchronous** communication when:

  * Loose coupling is needed
  * Long-running processes exist
  * High scalability and resilience are required

---

### 4. How do services communicate without sharing databases?

By:

* Calling another service’s **API**
* Publishing and consuming **events**
* Replicating required data using events (eventual consistency)

---

## Advanced

### 5. How does inter-service communication impact system reliability?

Poor communication design can cause:

* Cascading failures
* High latency
* Tight coupling

Using async messaging, retries, timeouts, and circuit breakers improves reliability.

---

### 6. How do you handle failures in synchronous communication?

By using:

* Timeouts
* Retries with backoff
* Circuit breakers
* Fallback responses

---

### 7. Why is async communication preferred in microservices?

Because it:

* Reduces tight coupling
* Improves scalability
* Prevents cascading failures
* Supports eventual consistency
* Works well in distributed systems

---

## One-Line Interview Summary

> “Inter-service communication allows microservices to collaborate using synchronous APIs or asynchronous events while maintaining loose coupling and resilience.”
