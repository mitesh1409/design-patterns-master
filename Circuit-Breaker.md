# Design Patterns - Circuit Breaker

## Topics

1. What?
2. Why?
3. Why try–catch is NOT enough
4. How Circuit Breaker works
5. Key configurations & tuning
6. Implementation approach
7. Node.js + Opossum example
8. Circuit Breaker granularity (per API vs per service)
9. Pros
10. Cons
11. Common interview follow-up questions (with answers)
12. Summary

---

## 1. What?

### What is Circuit Breaker?

The **Circuit Breaker pattern** is a **resilience design pattern** used to  
**stop repeated calls to a failing remote dependency** and allow the system to recover gracefully.

### Short description

It behaves like an electrical circuit breaker:

* **Closed** → requests flow normally
* **Open** → requests are blocked immediately
* **Half-Open** → limited test requests check recovery

---

## 2. Why?

### Why use Circuit Breaker?

In distributed systems:

* Network calls are unreliable
* Latency and partial failures are common
* One failing service can impact many others

### What problem does it solve?

* Cascading failures
* Thread / event-loop exhaustion
* Slow response times due to retries and timeouts
* Unnecessary load on failing services

### Purpose

* Fail fast
* Isolate failures
* Protect system resources
* Enable graceful degradation

---

## 3. Why try–catch is NOT enough

### Example

```js
try {
  const data = await productService.getProducts();
} catch (err) {
  // handle error
}
```

### What try–catch does well

* Handles a **single request failure**
* Prevents application crash
* Provides local error handling

### What try–catch cannot do

* Detect repeated failures over time
* Stop future calls to a failing service
* Prevent long timeouts
* Protect shared resources
* Track failure rate or health

**Key interview line:**

> try–catch handles correctness; Circuit Breaker handles resilience.

---

## 4. How Circuit Breaker Works

### Core States

#### 1. Closed

* Requests flow normally
* Failures are counted
* If failure threshold is crossed → Open

#### 2. Open

* Requests fail immediately
* No network call is made
* Fallback logic is executed
* After cool-down → Half-Open

#### 3. Half-Open

* Limited trial requests allowed
* Success → Closed
* Failure → Open again

---

## 5. Key Configuration Parameters (Important for Interviews)

| Parameter                | Meaning                            |
| ------------------------ | ---------------------------------- |
| Timeout                  | Max time allowed for a call        |
| Error threshold %        | Failure rate to trip breaker       |
| Request volume threshold | Minimum requests before evaluation |
| Reset timeout            | Cool-down period before retry      |
| Half-open calls          | Number of test requests            |

**Interview Tip:**
Wrong configuration can be worse than no circuit breaker.

---

## 6. Implementation Approach

### Recommended Practice

* Do **not** implement Circuit Breaker manually
* Use proven, battle-tested libraries

### Popular Libraries

* Java: **Resilience4j**
* Node.js: **opossum**
* Spring Boot: **Spring Cloud Circuit Breaker**

---

## 7. Node.js + Opossum Example

### Remote calls

```js
async function getProduct(productId) {
  return axios.get(`http://product-service/products/${productId}`);
}

async function getInventory(productId) {
  return axios.get(`http://product-service/inventory/${productId}`);
}
```

---

### Separate Circuit Breakers

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

### Fallbacks

```js
productCB.fallback(() => ({
  data: { name: "Unknown Product", price: 0 }
}));

inventoryCB.fallback(() => ({
  data: { available: false }
}));
```

---

### Usage

```js
async function placeOrder(productId) {
  const product = await productCB.fire(productId);
  const inventory = await inventoryCB.fire(productId);

  if (!inventory.data.available) {
    throw new Error("Out of stock");
  }

  return "Order placed";
}
```

---

## 8. Circuit Breaker Granularity (Very Important)

### Question

One Circuit Breaker per service or per API?

### Correct Answer

**One Circuit Breaker per remote dependency or per operation.**

### Why not per service?

* APIs have different SLAs
* One slow API can block healthy ones
* Larger blast radius

### Recommendation Table

| Scope                    | Recommendation |
| ------------------------ | -------------- |
| Per service              | ❌ Too coarse   |
| Per API                  | ✅ Good         |
| Per operation / use case | ✅ Best         |
| Per external system      | ✅ Yes          |

**Golden Rule:**

> One Circuit Breaker per thing that can fail independently.

---

## 9. Pros

* Prevents cascading failures
* Enables fail-fast behavior
* Protects threads and event loop
* Improves system stability
* Enables graceful degradation
* Production-ready resilience

---

## 10. Cons

* Additional complexity
* Requires tuning
* Misconfiguration can block healthy traffic
* Does not fix root cause
* Monitoring overhead

---

## 11. Common Interview Follow-Up Questions (Answered)

### 1. Circuit Breaker vs Retry

**Retry** assumes failure is temporary.
**Circuit Breaker** assumes failure is systemic.

👉 Use retry **inside** a circuit breaker, not instead of it.

---

### 2. Circuit Breaker vs Timeout

* Timeout limits how long you wait
* Circuit Breaker decides **whether to call at all**

Timeout is a **building block**, Circuit Breaker is a **control mechanism**.

---

### 3. Where should Circuit Breaker be implemented?

* Service-to-service calls
* API Gateway (for external APIs)
* Not inside database access layer

**Rule:** Place it at network boundaries.

---

### 4. How do you decide thresholds?

* Based on SLA
* Traffic volume
* Error patterns
* Load testing and production metrics

No universal values exist.

---

### 5. What happens in Half-Open state?

* Limited requests are allowed
* Success → Closed
* Failure → Open again

Purpose: Safe recovery validation.

---

### 6. How many circuit breakers are too many?

* If every function has one → too many
* If each remote dependency has one → correct

---

### 7. Does Circuit Breaker guarantee availability?

No.
It **fails fast** but does not make a failed dependency available.

---

## 12. Summary (Interview-Ready)

* Circuit Breaker is a **resilience pattern**
* Prevents repeated calls to failing services
* Protects system resources
* Core states: **Closed → Open → Half-Open**
* try–catch is not enough for distributed systems
* Should be applied **per API / per operation**
* Common Node.js library: **opossum**
