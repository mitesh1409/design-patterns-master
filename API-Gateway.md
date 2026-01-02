# Design Patterns - API Gateway

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
- Middlewares (boilerplate tasks like authentication, rate limiting etc. which are common across all the services)
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
