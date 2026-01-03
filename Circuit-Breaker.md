# Design Patterns - Circuit Breaker

## Topics

* What?
* Why?
* Why try–catch is NOT enough
* How Circuit Breaker Works
* Implementation Approach
* Node.js + Opossum Example
* Circuit Breaker granularity (per API vs per service)
* Pros
* Cons
* Common Interview Follow-Up Questions
* Summary

---

## 1. What?

### What is Circuit Breaker?

The **Circuit Breaker pattern** is a **resilience pattern** used in microservices to **prevent repeated calls to a failing remote dependency**.

### Short description

It works like an electrical circuit breaker:

* Allows calls when the system is healthy
* Stops calls when failures exceed a threshold
* Periodically checks if the dependency has recovered

---

## 2. Why?

### Why use Circuit Breaker?

In microservices:

* Services communicate over the network
* Network calls are slow, unreliable, and failure-prone
* One failing service can impact multiple downstream services

### What problem does it solve?

* Prevents **cascading failures**
* Avoids **resource exhaustion** (threads, event loop, connection pools)
* Prevents repeated calls to a **known failing service**

### Purpose

* Fail fast
* Isolate failures
* Maintain system stability
* Enable graceful degradation

---

## 3. Why try–catch is NOT enough

### Example without Circuit Breaker

```js
try {
    const products = productService.getProducts();
} catch (error) {
    // graceful failure
}
```

### What this handles

* Single request failure
* Prevents application crash
* Graceful handling for that request

### What this does NOT handle

* Repeated failures over time
* Long network timeouts
* Resource exhaustion
* Cascading failures
* Failure awareness or memory

**try–catch handles correctness, not resilience.**

---

## 4. How Circuit Breaker Works

### Core States

1. **Closed**

   * All requests are allowed
   * Failures are monitored
2. **Open**

   * Requests are blocked immediately
   * Fallback logic is executed
3. **Half-Open**

   * Limited test requests are allowed
   * Success → Closed
   * Failure → Open

---

### Key Configuration Parameters

* Timeout
* Failure threshold percentage
* Request volume threshold
* Reset timeout (cool-down period)
* Number of trial requests (half-open)

---

## 5. Implementation Approach

### Recommended Practice

* Do **not** implement Circuit Breaker logic manually
* Use proven libraries

### Common Libraries

* Java: Resilience4j
* Node.js: opossum
* Spring Boot: Spring Cloud Circuit Breaker

---

## 6. Node.js + Opossum Example

### Remote service calls

```js
async function getProduct(productId) {
  return axios.get(`http://product-service/products/${productId}`);
}

async function getInventory(productId) {
  return axios.get(`http://product-service/inventory/${productId}`);
}
```

---

### Create separate Circuit Breakers

```js
const CircuitBreaker = require('opossum');

const productCB = new CircuitBreaker(getProduct, {
  timeout: 3000,
  errorThresholdPercentage: 50,
  resetTimeout: 10000,
});

const inventoryCB = new CircuitBreaker(getInventory, {
  timeout: 1000,
  errorThresholdPercentage: 30,
  resetTimeout: 5000,
});
```

---

### Add fallbacks

```js
productCB.fallback(() => ({
  data: { name: "Unknown Product", price: 0 }
}));

inventoryCB.fallback(() => ({
  data: { available: false }
}));
```

---

### Use in Order Service

```js
async function placeOrder(productId) {
  const product = await productCB.fire(productId);
  const inventory = await inventoryCB.fire(productId);

  if (!inventory.data.available) {
    throw new Error("Product out of stock");
  }

  return "Order placed successfully";
}
```

---

## 7. Circuit Breaker granularity (per API vs per service)

### Question

Should we have:

* One Circuit Breaker per service?
* Or one Circuit Breaker per API?

### Correct Answer

**One Circuit Breaker per remote dependency or per operation (API).**

---

### Why NOT one Circuit Breaker per service?

* Different APIs have different SLAs
* One slow API can block healthy APIs
* Increases blast radius unnecessarily

---

### Recommended Scope

| Scope                    | Recommendation |
| ------------------------ | -------------- |
| Per service              | ❌ Too coarse   |
| Per API endpoint         | ✅ Good         |
| Per operation / use case | ✅ Best         |
| Per external system      | ✅ Yes          |

**Rule:**

> One circuit breaker per thing that can fail independently.

---

## 8. Pros

* Prevents cascading failures
* Enables fail-fast behavior
* Protects system resources
* Improves system resilience
* Supports graceful degradation
* Keeps healthy services responsive

---

## 9. Cons

* Adds design and operational complexity
* Requires careful tuning
* Misconfiguration can block healthy traffic
* Does not fix root cause of failures
* Slight overhead due to monitoring

---

## 10. Common Interview Follow-Up Questions

* Why is try–catch not enough?
* Circuit Breaker vs Retry
* Circuit Breaker vs Timeout
* Where should Circuit Breaker be implemented (service vs gateway)?
* How do you decide thresholds?
* What happens in Half-Open state?
* How many circuit breakers are too many?

---

## Summary (Key Pointers)

* Circuit Breaker is a **resilience pattern**, not just error handling
* Prevents repeated calls to failing services
* Avoids cascading failures and resource exhaustion
* Core states: **Closed → Open → Half-Open**
* Should be applied **per API / per operation**, not per service
* Common Node.js library: **opossum**
* Essential pattern in production-grade microservices systems
