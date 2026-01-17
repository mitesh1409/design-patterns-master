# Microservices Design Patterns -> API Gateway

## Analogy

It is like a receptionist for your Microservices Application.

---

## Direct Communication between Client <-> Microservices without API Gateway

Challenges:  

#1 Multiple requests, increased latency, slow page loads, bad user experience  

Client (Web/Mobile) has to make several requests to gather all the necessary data.  
Latency is increased as we are making multiple API calls.  
Page becomes less responsive resulting in a bad user experience.  

#2 Client and Microservices are tightly coupled  

Client (Web/Mobile) becomes tightly coupled to the internal microservices architecture.  
Client needs to know exactly which service to call and what each service's API looks like (request, response etc.).  
If we do any changes into microservices then we may need to change client code as well.  
Here the lack of encapsulation of microservices makes the system brittle and hard to maintain.  

#3 Communication protocols are different

Microservices might also use various interprocess communication/IPC mechanisms,  
that are not suitable for the external clients (Web/Mobile).  
For example, services might use REST, gRPC or message queues like Kafka for internal communication.  
Exposing these internal communication protocols directly to clients can introduce security risks.  
Whereas external clients expect standard communication protocols HTTP/HTTPS.  
In that case we will have to add additional translation layer which adds unnecessary complexity.  

Example,  
In an E-commerce app, on the cart page we may need to show the following data:

- Product Details
- Customer Details
- Shipping Details
- Payment Details
etc.

So client has to call multiple APIs to get all of these data.

## API Gateway

With API Gateway in place, we can do the following:  

- Routing
- Middlewares (boilerplate tasks like authentication, rate limiting/throttling etc. which are common across all the services)
- Transform data formats (requests/responses)
- Aggregate responses from multiple services
- Enforce security policies handling tasks like authentication & routing

By centralizing these tasks the API Gateway simplifies microservices architecture,  
making it more scalable and easier to maintain.

With API Gateway in place, developers can focus on business logic only while developing services.

**How to design and implement API Gateway?**  

Step #1 - Define the responsibilities of the API Gateway.  
We need to decide what tasks API Gateway will handle.  

For example,  

* Request Routing
* Response Aggregation
* Protocol Translation
* Security Enforcement

Step #2 - Deployment strategy.

* Central
* Multiple Instances
* One per region
* One per client type

This decision will impact API Gateway's performance and scalability.  

Step #3 - Implementation.  

For implementation we have several options like,  

Managed  

* Amazon API Gateway
* Azure API Management
* apigee

Open Source  

* Kong
* Tyk
* express gateway

One modern approach to building an API Gateway is using GraphQL.  
GraphQL allows clients to query API in a flexible way,  
asking for exactly the data they need and nothing more.  
This reduces over fetching and under fetching issues common with REST APIs.  

In a microservice architecture, we can implement GraphQL API that communicates  
with multiple backend services.  
The gateway translates GraphQL queries into requests to the appropriate services  
and then aggregates the results and sends them back to the client.  
This approach is particularly powerful when dealing with complex data models  
and client specific needs.  
GraphQL's strong type system also helps catch errors early in the development process  
improving the reliability of your API.  

**Benefits of using API Gateway**  

- Facilitates faster processing
- Decreases load time
- Optimizes resource utilization
- Ensures that the client needs to access only one service instead of multiple microservices
- Houses common boilerplate code like authentication, rate limiting, caching etc.

## References

* [Designing with API Gateway: Microservices Unleashed](https://www.youtube.com/watch?v=JNmiOw26PGg&list=PLJq-63ZRPdBsPWE24vdpmgeRFMRQyjvvj&index=11)
* [API Gateways in System Design Interviews w/ Ex-Meta Staff Engineer](https://www.youtube.com/watch?v=7-6F3b14baA)

---

Below are **clear, interview-focused notes** for **Pattern 3: API Gateway**, following your exact structure and keeping explanations **simple and concise**.

---

## API Gateway, Another Answer

## What?

An **API Gateway** is a **single entry point** for all client requests in a microservices system.
Clients do not call microservices directly; they call the API Gateway, which routes requests to the appropriate services.

> Client → API Gateway → Microservices

---

## Why?

### Problems Without API Gateway

* Clients must know all service URLs
* Too many network calls from client
* Security logic duplicated in every service
* Hard to change services without breaking clients

### What API Gateway Solves

* Single entry point for clients
* Centralized authentication and authorization
* Request routing and aggregation
* Hides internal service structure
* Simplifies client-side logic

---

## How?

### Responsibilities of an API Gateway

* Route requests to correct service
* Authenticate and authorize requests
* Rate limiting and throttling
* Request/response transformation
* Aggregation of multiple service calls

### What It Should NOT Do

* Business logic
* Database access

---

### Common API Gateway Implementations

* Custom Node.js gateway
* NGINX
* Kong
* AWS API Gateway
* Spring Cloud Gateway

---

## Node.js Example (TypeScript + Express + REST)

### Scenario

* Client calls API Gateway
* API Gateway routes request to:

  * User Service
  * Order Service

---

### API Gateway

```ts
// api-gateway/src/index.ts
import express from "express";
import axios from "axios";

const app = express();
app.use(express.json());

// Route to User Service
app.get("/users/:id", async (req, res) => {
  const response = await axios.get(
    `http://localhost:3001/users/${req.params.id}`
  );
  res.json(response.data);
});

// Route to Order Service
app.post("/orders", async (req, res) => {
  const response = await axios.post(
    "http://localhost:3002/orders",
    req.body
  );
  res.json(response.data);
});

app.listen(3000, () => {
  console.log("API Gateway running on port 3000");
});
```

---

### Client Perspective

Client only knows:

```http
GET /users/123
POST /orders
```

Client does **not** know:

* User Service URL
* Order Service URL

---

### Key Observations

* Single entry point (port 3000)
* Services hidden from client
* Gateway handles routing only
* Services remain independent

---

## Pros / Cons

### Pros

* Simplifies client interaction
* Centralized security
* Easy to change internal services
* Reduces client-service coupling
* Enables request aggregation

### Cons

* Single point of failure (if not designed properly)
* Additional network hop
* Can become a bottleneck
* Risk of turning into a “god service”

---

## Interview Questions

### Conceptual

1. What is an API Gateway?
2. Why do we need an API Gateway in microservices?

### Practical

3. What responsibilities belong in API Gateway?
4. What should not be implemented in API Gateway?

### Advanced

5. How do you avoid API Gateway becoming a single point of failure?
6. Difference between API Gateway and Load Balancer?
7. When would you use multiple API Gateways?

---

## Interview One-Line Summary

> “API Gateway acts as a single entry point that routes client requests to appropriate microservices while handling cross-cutting concerns like security and rate limiting.”

---

## Very Common Interview Follow-Up

**Q: Can microservices work without an API Gateway?**
**A:** Yes, but API Gateway is strongly recommended for production systems.

---

## Interview Questions

## Conceptual

### 1. What is an API Gateway?

An API Gateway is a **single entry point** for clients that routes requests to the appropriate microservices and handles cross-cutting concerns like security and rate limiting.

---

### 2. Why do we need an API Gateway in microservices?

Because it:

* Hides internal service structure from clients
* Reduces client-side complexity
* Centralizes authentication and security
* Makes microservices easier to change without breaking clients

---

## Practical

### 3. What responsibilities belong in API Gateway?

* Request routing
* Authentication and authorization
* Rate limiting and throttling
* Request/response transformation
* Aggregating responses from multiple services

---

### 4. What should not be implemented in API Gateway?

* Business logic
* Database access
* Complex workflows
* Long-running processes

The gateway should stay **thin and lightweight**.

---

## Advanced

### 5. How do you avoid API Gateway becoming a single point of failure?

* Run multiple instances of the gateway
* Place it behind a load balancer
* Use health checks and auto-scaling
* Apply circuit breakers and timeouts

---

### 6. Difference between API Gateway and Load Balancer?

| API Gateway                | Load Balancer                     |
| -------------------------- | --------------------------------- |
| Works at application level | Works at network level            |
| Knows APIs and routes      | Distributes traffic               |
| Handles auth, rate limits  | Does not handle business concerns |
| Client-facing              | Service-facing                    |

---

### 7. When would you use multiple API Gateways?

* Different clients (web, mobile, partner APIs)
* Backend for Frontend (BFF) pattern
* Different security or performance needs
* Large systems with many teams

---

## One-Line Interview Summary

> “API Gateway provides a single entry point for clients and centralizes cross-cutting concerns while routing requests to microservices.”
