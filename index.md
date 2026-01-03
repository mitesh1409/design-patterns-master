# Design Patterns

## Index

**Creational Design Patterns**  

1. [Singleton](./Singleton.md)

Builder

Factory

**Microservices Design Patterns**  

1. [Event Sourcing](./Event-Sourcing.md)
2. [CQRS](./CQRS.md)
3. [SAGA](./SAGA.md)
4. [API Gateway](./API-Gateway.md)
5. [Database per Service](./Database-per-Service.md)


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
