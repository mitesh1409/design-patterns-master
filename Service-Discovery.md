# Microservices Design Patterns -> Service Discovery

## What?

**Service Discovery** is a mechanism that allows microservices to  
**dynamically find the network location** (IP address and port) of other microservices.

> Services discover each other **at runtime** or **dynamically**, not via hardcoded URLs.

---

## Why?

### Problems Without Service Discovery

* Service IPs and ports change frequently
* Hardcoding service URLs breaks deployments
* Auto-scaling becomes difficult
* High operational overhead

### What Service Discovery Solves

* Dynamic service lookup
* Supports scaling and containerized deployments
* Decouples services from physical locations
* Enables resilience and automation

---

## How?

### Two Main Types

#### 1. Client-Side Service Discovery

* Client (or API Gateway) queries a **Service Registry**
* Client selects one instance and calls it directly

Examples:

* Netflix Eureka
* Consul
* etcd

---

#### 2. Server-Side Service Discovery

* Client calls a **load balancer**
* Load balancer queries Service Registry and forwards request

Examples:

* AWS ALB / ELB
* Kubernetes Service

---

### Core Components

| Component        | Role                              |
| ---------------- | --------------------------------- |
| Service Registry | Stores service locations          |
| Service Instance | Registers itself on startup       |
| Client / Gateway | Queries registry to find services |

---

## Node.js Example (TypeScript + Consul + REST)

### Scenario

* User Service registers itself in Consul
* API Gateway discovers User Service dynamically

---

### User Service (Service Registration)

```ts
// user-service/src/index.ts
import express from "express";
import Consul from "consul";

const app = express();
const consul = new Consul();

app.get("/health", (_, res) => res.send("OK"));

consul.agent.service.register({
  name: "user-service",
  address: "localhost",
  port: 3001,
  check: {
    http: "http://localhost:3001/health",
    interval: "10s",
  },
});

app.listen(3001, () => {
  console.log("User Service running on port 3001");
});
```

---

### API Gateway (Service Discovery)

```ts
// api-gateway/src/index.ts
import express from "express";
import axios from "axios";
import Consul from "consul";

const app = express();
const consul = new Consul();

app.get("/users/:id", async (req, res) => {
  const services = await consul.catalog.service.nodes("user-service");

  const service = services[0]; // simple selection
  const url = `http://${service.Address}:${service.ServicePort}`;

  const response = await axios.get(`${url}/users/${req.params.id}`);
  res.json(response.data);
});

app.listen(3000, () => {
  console.log("API Gateway running on port 3000");
});
```

---

### Key Observations

* No hardcoded service URLs
* Services register themselves
* API Gateway discovers services dynamically
* Supports scaling and failures

---

## Pros / Cons

### Pros

* Dynamic service resolution
* Works well with auto-scaling
* No hardcoded endpoints
* Improves system resilience

### Cons

* Extra infrastructure component
* Registry availability is critical
* Added operational complexity

---

## Interview Questions

### Conceptual

1. What is Service Discovery?
2. Why is Service Discovery needed in microservices?

### Practical

3. Difference between client-side and server-side discovery?
4. How do services register and deregister?

### Advanced

5. How does Service Discovery work with API Gateway?
6. What happens if Service Registry goes down?
7. How is Service Discovery handled in Kubernetes?

---

## Interview One-Line Summary

> “Service Discovery enables microservices to dynamically find each other at runtime, supporting scalability, resilience, and automated deployments.”

---

## Important Interview Tip

If asked:
**“Is Service Discovery needed in Kubernetes?”**

Correct answer:

> “Kubernetes provides built-in service discovery through Services and DNS.”

---

## Interview Questions

## Conceptual

### 1. What is Service Discovery?

Service Discovery is a mechanism that allows microservices to **find the network location of other services dynamically** without using hardcoded IPs or URLs.

---

### 2. Why is Service Discovery needed in microservices?

Because service instances:

* Scale up and down
* Change IP addresses
* Restart frequently

Service Discovery allows services to **locate each other at runtime**, enabling scalability and reliability.

---

## Practical

### 3. Difference between client-side and server-side discovery?

| Client-Side Discovery           | Server-Side Discovery          |
| ------------------------------- | ------------------------------ |
| Client queries service registry | Load balancer queries registry |
| Client selects service instance | Load balancer selects instance |
| More control at client          | Simpler client                 |
| Example: Eureka + Ribbon        | Example: AWS ALB, Kubernetes   |

---

### 4. How do services register and deregister?

* On startup, a service **registers itself** with the service registry
* It sends **health checks** periodically
* On shutdown or failure, the service is **deregistered automatically** or removed when health checks fail

---

## Advanced

### 5. How does Service Discovery work with API Gateway?

The API Gateway queries the **service registry** to find available service instances and routes client requests to one of them dynamically.

---

### 6. What happens if Service Registry goes down?

* Existing services continue working using cached service data
* New service registrations may fail
* Best practice is to run the registry in **high availability mode**

---

### 7. How is Service Discovery handled in Kubernetes?

Kubernetes provides built-in service discovery using:

* **Kubernetes Services**
* **DNS-based discovery**

Services communicate using service names instead of IP addresses.

---

## One-Line Interview Summary

> “Service Discovery allows microservices to dynamically find each other at runtime, enabling scalable and resilient systems.”
