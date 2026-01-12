# Microservices Design Patterns

Microservices Design Patterns are as follows:  

## 1. Core / Foundational Patterns (Learn First)

These are **must-know** for interviews and real-world microservices.

1. Service Decomposition

   * Decompose by Business Capability
   * Decompose by Subdomain (DDD)

2. Database per Service

3. API Gateway

4. Service Discovery

5. Inter-Service Communication

   * Synchronous (REST / gRPC)
   * Asynchronous (Message Broker)

6. Configuration Management

   * Externalized Configuration

---

## 2. Data Consistency & Transaction Patterns (Very Important)

Frequently asked in **senior / lead interviews**.

7. Saga Pattern

   * Choreography
   * Orchestration

8. Event-Driven Architecture

9. Eventual Consistency

10. CQRS (Command Query Responsibility Segregation)

11. Event Sourcing

---

## 3. Resilience & Fault Tolerance Patterns (High Priority)

Common production issues → **favorite interview topics**.

12. Circuit Breaker

13. Retry

14. Timeout

15. Bulkhead

16. Rate Limiting / Throttling

17. Fallback

---

## 4. Observability & Monitoring Patterns (Often Overlooked, but Valued)

Shows **production maturity**.

18. Centralized Logging

19. Distributed Tracing

20. Health Check

21. Metrics & Monitoring

---

## 5. Deployment & Infrastructure Patterns (Good to Know)

Useful for DevOps-aware roles.

22. Blue-Green Deployment

23. Canary Deployment

24. Rolling Deployment

25. Sidecar Pattern

26. Service Mesh

---

## 6. Security Patterns (Selective but Important)

Often asked conceptually.

27. Authentication & Authorization per Service

28. Token Propagation (JWT / OAuth2)

29. Zero Trust / Mutual TLS

---

## 7. UI / Client-Facing Patterns (Lower Priority)

Good to mention, rarely deep-dived.

30. Backend for Frontend (BFF)

31. API Composition
