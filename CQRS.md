# Design Patterns > CQRS

## What is CQRS?

CQRS = Command Query Responsibility Segregation

Reads and Writes are handled separately.

Command = Write Operations, like Create, Update, Delete
We should write seperate service/handler for write operations.

Command -> Command Handler -> Write Model -> Write Database

Query = Read Operations, like Get, List
We should write seperate service/handler for read operations.

Query -> Query Handler -> Read Model -> Read Database

Command (Write operations) & Query (Read operations) are completely different pathways,  
they don't cross/interfere each other. So it is easy to seperate them.  

If system is becoming slow as traffic is increasing,  
it could be because you are not separating reads and writes.

## Examples

Example #1 - E-commerce Platform  
For example, in Amazon/Flipkart or any e-commerce platform,  
placing an order (write operations) is different from browsing products (read operations).  

Example #2 - Banking System  
For example, in a banking system, the command side handles account updates and transactions,  
while the query side retrieves transaction history or account balances.  

## Benefits

By separating these two operations,  
you can optimize each part of the system independently.

This pattern improves scalability, especially when reads significantly outnumber writes or  
when different parts of your system require eventual consistency.

## How to implement CQRS?

We can have seperate databases for reads and writes,
allowing for better performance and scalability.

Command -> Command Handler -> Write Model -> Write Database  

Event Sourcing

Query -> Query Handler -> Read Model -> Read Database

CQRS implementation key point is - Commands don't directly change data,  
but they trigger actions that lead to changes.

You might opt for simple and cost effective S3 Buckets for the Command/Write Database,  
and select a database with superior query capabilities such as Elastic Search for the  
Query/Read Database.  

A relational SQL database might be better fit for the Command/Write Database,  
while a NOSQL database could be more suitable for the Query/Read Database.  

Now because we are using different databases it's not possible to commit changes  
to both the Write and Read models in a single Atomic transaction.  

Typically changes made to the Write models are asynchronously propagated to the  
Read models through messaging or events resulting in "Eventual Consistency".  

**Problem with this approach?**  

However with this approach it can be challenging to keep everything in sync with using  
seperate Read and Write databases and "Eventual Consistency".  

Because the order or events published from the Write to the Read database becomes really  
important. Imagine that the same Write model instances are updated twice in close succession.  

If the first update event is delivered after the second event then our Read model might  
be updated with a stale/old data.  

Unfortunately most asynchronous message buses are designed for high availability and  
performance which means they don't guarantee that messages will be delivered in the order  
they were published.

We can solve this problem by using "Event Sourcing" pattern.

To keep data consistent between the two databases,
we can use event sourcing or messaging systems to propagate changes  
from the write database to the read database.  
For that we can use tools like Kafka, RabbitMQ, Solace etc.

## References

- [https://www.youtube.com/shorts/jclKKE8esiw](https://www.youtube.com/shorts/jclKKE8esiw)
- [Mastering CQRS in Just 5 Minutes](https://www.youtube.com/watch?v=SvjdJoNPcHs&list=PLJq-63ZRPdBsPWE24vdpmgeRFMRQyjvvj&index=13)
- [Master Event Sourcing in Just 10 Minutes](https://www.youtube.com/watch?v=ID-_ic1fLkY&list=PLJq-63ZRPdBsPWE24vdpmgeRFMRQyjvvj&index=12)
