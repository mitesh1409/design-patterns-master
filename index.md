# Design Patterns

## Index

1. [CQRS](./CQRS.md)
2. [Event Sourcing](./Event-Sourcing.md)
3. [SAGA](./SAGA.md)
4. [API Gateway](./API-Gateway.md)
5. [Singleton](./Singleton.md)


Singleton

Factory

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



Playlist - https://www.youtube.com/playlist?list=PLJq-63ZRPdBsPWE24vdpmgeRFMRQyjvvj
