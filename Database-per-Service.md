# Microservices Design Patterns -> Database per Service

## 1. What?

**Database Per Service** is a microservices design pattern where  
**each microservice owns and manages its own database**.  
No other service is allowed to directly access that database.

**Key idea:**

> *A service’s data is private and can only be accessed via its API.*

---

## 2. Why?

### Why to use it?

To ensure **loose coupling** and **true service autonomy** in a microservices architecture.

### What problem does it solve?

* Prevents **tight coupling** caused by multiple services sharing the same database
* Avoids **cross-service schema dependencies**
* Eliminates **accidental data corruption** by other services
* Allows independent schema changes without impacting other services

### Purpose

* Enable **independent development, deployment, and scaling**
* Support **polyglot persistence** (each service can use a different database type)
* Enforce **clear ownership of data**

---

## 3. How?

### Implementation details

1. **One service → one database**

   * Each microservice has its own database (or schema, ideally separate DB)

2. **No direct database access across services**

   * Services communicate only via APIs (REST, gRPC, messaging)

3. **Data sharing via events or APIs**

   * Use async messaging (Kafka, RabbitMQ) or synchronous APIs to share data between services.

4. **Service owns its schema**

   * Only that service can modify its database schema

---

### Example

#### Scenario

An **E-commerce system** with two services:

* **Order Service**
* **User Service**

#### Architecture

```
Order Service  ----->  Order DB (PostgreSQL)
      |
      | REST / Event
      v
User Service   ----->  User DB (MongoDB)
```

#### Example flow

* Order Service needs user details
* It **does NOT query User DB**
* It calls **User Service API** or listens to a **UserCreated event**

```plaintext
Order Service → GET /users/{id} → User Service
```

Each service:

* Controls its own database
* Can choose its own database technology
* Can evolve independently

---

## 4. Pros

* **Loose coupling**
* **High service autonomy**
* **Independent scaling**
* **Independent deployment**
* **Polyglot persistence support**
* **Clear data ownership**
* **Better fault isolation**

---

## 5. Cons

* **Complex data consistency**

  * Distributed transactions are hard (no joins across DBs)

* **Eventual consistency**

  * Data may not be immediately consistent across services

* **Increased operational overhead**

  * More databases to manage

* **More complex queries**

  * Reporting across services requires data aggregation

* **Requires strong discipline**

  * Teams must not bypass service boundaries

---

## Summary (Key Interview Pointers)

* Each microservice **owns its database**
* No **shared database** across services
* Enforces **loose coupling and autonomy**
* Communication happens via **APIs or events**
* Improves scalability and flexibility
* Introduces **eventual consistency challenges**

**One-liner for interview:**

> *Database Per Service ensures that each microservice has full ownership of its data, enabling independent development, deployment, and scalability at the cost of increased complexity in data consistency.*

## References

* [Ultimate Guide to Database Per Service pattern in Microservices](https://www.youtube.com/watch?v=DKQLhy9bgdk&list=PLJq-63ZRPdBsPWE24vdpmgeRFMRQyjvvj&index=6)
* [Database per Service Pattern in Microservices](https://www.youtube.com/watch?v=la2q1vFA5q0)
