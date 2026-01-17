# Microservices Design Patterns -> Retry + Timeout

## Is Retry + Timeout the same as Retry with Backoff + Jitter?

### Short answer

**No, but they are closely related.**

| Concept     | Purpose                            |
| ----------- | ---------------------------------- |
| **Timeout** | Limits how long a request can wait |
| **Retry**   | Re-attempts a failed request       |
| **Backoff** | Increases delay between retries    |
| **Jitter**  | Adds randomness to retry delays    |

### Correct mental model

> **Retry + Timeout is the base pattern**
> **Retry with Backoff + Jitter is the production-grade version**

In real systems, **Retry without backoff and jitter is dangerous**.

---

## 1. What?

### What is Retry + Timeout?

A **resilience pattern** where:

* A request is given a **maximum wait time (timeout)**
* If it fails, it is **retried a limited number of times**

### What is Retry with Backoff + Jitter?

An enhanced retry strategy where:

* Each retry waits **longer than the previous one (backoff)**
* A **random delay (jitter)** is added to prevent retry storms

---

## 2. Why?

### Why do we need Retry?

* Transient failures are common

  * Network glitches
  * Temporary service overload
  * DNS resolution delays
* Many failures succeed if retried after a short delay

### Why Timeout is mandatory with Retry

Without timeout:

* Requests hang indefinitely
* Retries pile up
* Threads / event loop get exhausted

**Retry without timeout is a system-killer.**

---

## 3. Why naive Retry is dangerous

### Example of BAD retry

```js
while(true) {
  callRemoteService();
}
```

### Problems

* Retry storms
* Thundering herd problem
* Cascading failures
* Makes outages worse

---

## 4. Backoff and Jitter (Core Concepts)

### Backoff

Gradually increases delay between retries.

#### Common backoff strategies

* Fixed backoff (not recommended)
* Linear backoff
* **Exponential backoff (recommended)**

```text
100ms → 200ms → 400ms → 800ms
```

---

### Jitter

Adds randomness to delay.

```text
400ms ± random(0–200ms)
```

### Why jitter is important

If 1,000 services retry at the same time:

* Without jitter → synchronized retries
* With jitter → retries spread out

**This prevents retry storms.**

---

## 5. How Retry + Timeout Works Together

### Flow

1. Call remote service with timeout
2. If timeout or retryable error occurs:

   * Wait (backoff + jitter)
   * Retry
3. Stop after max retries
4. Propagate failure or fallback

---

## 6. Implementation Guidelines

### Best Practices

* Always set:

  * Timeout
  * Max retry count
* Retry only:

  * Idempotent operations
  * Read operations
* Combine with:

  * Circuit Breaker
  * Bulkhead pattern

---

## 7. Node.js Example (Axios)

```js
const axios = require('axios');

async function callServiceWithRetry(url, retries = 3) {
  let delay = 200;

  for (let attempt = 1; attempt <= retries; attempt++) {
    try {
      return await axios.get(url, { timeout: 1000 });
    } catch (err) {
      if (attempt === retries) throw err;

      await new Promise(res =>
        setTimeout(res, delay + Math.random() * 100)
      );

      delay *= 2; // exponential backoff
    }
  }
}
```

---

## 8. Retry + Timeout vs Circuit Breaker

| Aspect         | Retry + Timeout    | Circuit Breaker     |
| -------------- | ------------------ | ------------------- |
| Handles        | Transient failures | Persistent failures |
| Behavior       | Reattempt          | Fail fast           |
| Time awareness | Per request        | Across requests     |
| Memory         | Stateless          | Stateful            |

**They complement each other, not replace.**

---

## 9. Pros

* Improves success rate
* Handles transient failures
* Simple to implement
* Works well for reads

---

## 10. Cons

* Can amplify load
* Risk of retry storms
* Increases latency
* Unsafe for non-idempotent operations

---

## 11. Common Interview Follow-Up Questions (With Answers)

---

### 1. Should you retry all failures?

**No.**

Retry only:

* Timeouts
* Network errors
* 5xx errors

Do NOT retry:

* 4xx client errors
* Validation failures
* Authentication failures

---

### 2. Why is retry dangerous for POST requests?

POST is usually **non-idempotent**.

Retry may:

* Create duplicate orders
* Charge users twice

**Use idempotency keys if retrying POST.**

---

### 3. Retry vs Circuit Breaker – which comes first?

**Retry first, Circuit Breaker outside.**

Retry handles temporary failures.
Circuit Breaker stops retries when failures persist.

---

### 4. What is the Thundering Herd problem?

When many clients retry at the same time after a failure, causing:

* Traffic spikes
* Re-failure of recovering services

**Backoff + jitter prevents this.**

---

### 5. How many retries are recommended?

No fixed number, but typically:

* 2–3 retries
* Short timeouts
* Fast backoff

More retries ≠ more reliability.

---

### 6. Can Retry cause cascading failures?

Yes, if:

* No timeout
* No backoff
* High retry counts

That’s why retry must be controlled.

---

### 7. Where should Retry be implemented?

* Service-to-service calls
* API Gateway (carefully)
* Client SDKs

Avoid retrying at multiple layers blindly.

---

### 8. Is Retry a reliability or resilience pattern?

**Resilience.**

It improves system behavior under partial failures.

---

## 12. Interview Summary (One-Minute Answer)

* Retry + Timeout handles **transient failures**
* Retry with Backoff + Jitter is the **production-grade approach**
* Timeout is mandatory
* Retry without backoff causes retry storms
* Retry complements Circuit Breaker
* Only retry idempotent operations
