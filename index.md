# Design Patterns

## Index

1. [Design Patterns](./Design-Patterns.md)
2. [Singleton](./Singleton.md)
3. [Prototype](./Prototype.md)
4. [Builder](./Builder.md)
5. [Factory](./Factory.md)
6. [Abstract Factory](./Abstract-Factory.md)
7. [Facade](./Facade.md)
8. [Proxy](./Proxy.md)
9. [Adapter](./Adapter.md)
10. [Strategy](./Strategy.md)
11. [Observer](./Observer.md)
12. [Command](./Command.md)
13. [Microservices Design Patterns](./Microservices-Design-Patterns.md)
14. [Service Decomposition](./Service-Decomposition.md)
15. [Database per Service](./Database-per-Service.md)
16. [API Gateway](./API-Gateway.md)
17. [Service Discovery](./Service-Discovery.md)
18. [Inter-Service Communication](./Inter-Service-Communication.md)
19. [SAGA](./SAGA.md)
20. [Event-Driven Architecture](./Event-Driven-Architecture.md)
21. [Circuit Breaker](./Circuit-Breaker.md)
22. [Retry + Timeout](./Retry-+-Timeout.md)
23. [CQRS](./CQRS.md)
24. [Event Sourcing](./Event-Sourcing.md)
25. [Side Car](./Side-Car.md)


Transactional Outbox Pattern
This pattern helps to sync your DB and message queue without losing events.

---

Message Brokers
Amazon SQS
Kafka
RabbitMQ

What  
Why  
How  
Pros  
Cons  

Eventual Consistency

In CQRS, how do we keep Write DB and Read DB in sync?

Database Management Systems (DBMS) - ACID (Atomicity, Consistency, Isolation, Durability)

Exactly Once - Two Phase Commit

Polyglot Architecture

Backend-for-Frontend (BFF), Proxy and Composite Patterns

**Backend-for-Frontend (BFF)**  

BFF creates dedicated API for each client type.  
For example, BFF for Web, BFF for Mobile etc.  
That way it ensures each client gets exactly what it needs.  

The problem?  

* Managing multiple BFFs can itself become more complex.
* Code duplication.
* Maintenance overhead for similar tasks/features.


## References

* [Microservices](https://www.youtube.com/playlist?list=PLJq-63ZRPdBsPWE24vdpmgeRFMRQyjvvj)
