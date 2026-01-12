# Microservices Design Patterns -> Service Decomposition

## What?

**Service Decomposition** is the process of breaking a large monolithic application into  
**smaller, independent microservices**, where each service owns a **single, well-defined responsibility**.

Two commonly accepted decomposition strategies:

1. **Decompose by Business Capability**
2. **Decompose by Subdomain (DDD)**

> Each microservice represents a **business capability**, not a technical layer.

---

## Why?

### Problems in Monoliths

* Tight coupling between unrelated features
* Large codebase → slow development and deployments
* One failure can bring down the entire system
* Difficult to scale specific functionality

### What Service Decomposition Solves

* Independent development and deployment
* Clear ownership and responsibility
* Targeted scalability (scale only what is needed)
* Technology flexibility per service
* Improved fault isolation

---

## How?

### Step-by-Step Decomposition Approach

1. **Identify business capabilities**

   * Example: User Management, Orders, Payments, Inventory

2. **Define service boundaries**

   * Each service owns:

     * Its logic
     * Its database
     * Its APIs

3. **Avoid technical decomposition**

   * ❌ Auth Service, DB Service, Validation Service
   * ✅ User Service, Order Service, Payment Service

4. **Enforce loose coupling**

   * Communicate via APIs or events
   * No shared databases

---

### Example: E-Commerce System

| Microservice      | Responsibility                    |
| ----------------- | --------------------------------- |
| User Service      | User registration, authentication |
| Order Service     | Order creation, order lifecycle   |
| Payment Service   | Payment processing                |
| Inventory Service | Stock management                  |

---

## Node.js Example (TypeScript + MongoDB)

### Scenario

We decompose a monolithic e-commerce backend into **User Service** and **Order Service**.

Each service:

* Has its own Node.js app
* Owns its own MongoDB database
* Communicates via REST (for now)

---

### User Service

**Responsibilities**

* Create users
* Fetch user details

**Tech Stack**

* Node.js + TypeScript
* Express
* MongoDB (Mongoose)

```ts
// user-service/src/index.ts
import express from "express";
import mongoose from "mongoose";

const app = express();
app.use(express.json());

mongoose.connect("mongodb://localhost:27017/user-service");

const UserSchema = new mongoose.Schema({
  name: String,
  email: String,
});

const User = mongoose.model("User", UserSchema);

// Create users
app.post("/users", async (req, res) => {
  const user = await User.create(req.body);
  res.status(201).json(user);
});

// Fetch user details
app.get("/users/:id", async (req, res) => {
  const user = await User.findById(req.params.id);
  res.json(user);
});

app.listen(3001, () => {
  console.log("User Service running on port 3001");
});
```

---

### Order Service

**Responsibilities**

* Create orders
* Validate user existence via User Service

**Tech Stack**

* Node.js + TypeScript
* Express
* MongoDB
* REST call to User Service

```ts
// order-service/src/index.ts
import express from "express";
import mongoose from "mongoose";
import axios from "axios";

const app = express();
app.use(express.json());

mongoose.connect("mongodb://localhost:27017/order-service");

const OrderSchema = new mongoose.Schema({
  userId: String,
  product: String,
  amount: Number,
});

const Order = mongoose.model("Order", OrderSchema);

// Create orders
app.post("/orders", async (req, res) => {
  const { userId } = req.body;

  // Validate user via User Service
  const userResponse = await axios.get(
    `http://localhost:3001/users/${userId}`
  );

  if (!userResponse.data) {
    return res.status(400).json({ message: "Invalid user" });
  }

  const order = await Order.create(req.body);
  res.status(201).json(order);
});

app.listen(3002, () => {
  console.log("Order Service running on port 3002");
});
```

---

### Key Takeaways from the Example

* Each service has **its own database**
* Services communicate via **APIs**
* No shared code or shared DB
* Clear responsibility per service

---

## Pros / Cons

### Pros

* Independent deployment and scaling
* Clear ownership and responsibility
* Better fault isolation
* Faster development for large teams
* Technology flexibility

### Cons

* Increased operational complexity
* Network latency between services
* Distributed data management
* Requires DevOps maturity
* Harder debugging without proper observability

---

## Interview Questions

### Conceptual

1. What is service decomposition in microservices?
2. Difference between technical decomposition and business decomposition?
3. How do you identify service boundaries?

### Practical

4. How would you decompose a monolithic e-commerce system?
5. What happens if services are decomposed incorrectly?
6. How does service decomposition relate to Database per Service?

### Advanced

7. How does Domain-Driven Design help in service decomposition?
8. What are common anti-patterns in service decomposition?

---

## Interview One-Line Summary

> “Service Decomposition breaks a monolith into independent, business-aligned microservices, each owning its logic and data, enabling scalability, resilience, and independent deployments.”

---

Below are **clear, concise, interview-ready answers** for **Service Decomposition**.
Language is intentionally **simple and direct**, suitable for quick recall.

---

## Conceptual

### 1. What is service decomposition in microservices?

Service decomposition is the process of breaking a large application into **small, independent services**, where each service handles **one specific business responsibility** and can be developed, deployed, and scaled independently.

---

### 2. Difference between technical decomposition and business decomposition?

| Technical Decomposition        | Business Decomposition       |
| ------------------------------ | ---------------------------- |
| Split by layers (UI, DB, Auth) | Split by business capability |
| Creates tight coupling         | Creates loose coupling       |
| Not recommended                | Recommended                  |
| Example: Auth Service          | Example: Order Service       |

---

### 3. How do you identify service boundaries?

By identifying **business capabilities** and defining services around **what the business does**, not how the code is written.
Each service should have **single responsibility**, **clear ownership**, and **its own data**.

---

## Practical

### 4. How would you decompose a monolithic e-commerce system?

Break it into services like:

* User Service (users, authentication)
* Order Service (orders, order lifecycle)
* Payment Service (payments)
* Inventory Service (stock management)

Each service owns its **logic, APIs, and database**.

---

### 5. What happens if services are decomposed incorrectly?

* Too many service-to-service calls
* High latency and poor performance
* Data consistency problems
* Difficult deployments and debugging
* Microservices behave like a **distributed monolith**

---

### 6. How does service decomposition relate to Database per Service?

Each decomposed service must own **its own database**.
Sharing a database breaks service independence and tightly couples services.

---

## Advanced

### 7. How does Domain-Driven Design help in service decomposition?

DDD helps by defining **bounded contexts**, which naturally map to microservices.
Each bounded context becomes one service with clear boundaries and responsibilities.

---

### 8. What are common anti-patterns in service decomposition?

* Splitting services by technical layers
* Sharing databases across services
* Creating too many small services too early
* Strong synchronous coupling between services
* Not aligning services with business domains
