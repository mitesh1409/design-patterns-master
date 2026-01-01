# Design Patterns > Event Sourcing

## Introduction to Event Sourcing

* Event sourcing is an architectural pattern that treats an application's memory like a "time machine."
* Unlike traditional databases that only store the **current state**, event sourcing records every change as an immutable event in an append-only log.

## Why Choose Event Sourcing?

* **Audit Trail:** Provides a perfect history of who did what and when, which is critical for compliance and debugging.
* **State Reconstruction:** Allows you to rebuild the state of an entity at any specific point in time by replaying events up to that date.
* **CQRS Support:** Naturally separates the models that change data from the models that read data.

## Sourcing and Hydration

* **Sourcing:** The concept of deriving current state from past events.
* **Hydration:** The mechanism of rebuilding an object’s state from the event log on demand.
* *Example:* To find a product's price, the system starts with an empty object and sequentially applies every "Price Updated" or "Discount Applied" event until the current value is reached.

## Hydration vs. Replay

* **Hydration:** Reconstructing an entity's current state for immediate application use (e.g., answering a specific query).
* **Replay:** Processing the entire history of events to regenerate the system state, often used for debugging, migrations, or recovering from a corrupted database.

## Optimizing Performance: Snapshots & Materialized Views

* **Snapshots:** Periodically saving the state (e.g., every 100 events) so that hydration only needs to replay events from the last snapshot forward.
* **Materialized Views:** Maintaining the "current state" in a separate, query-optimized database that updates in real-time as new events occur.

## Integration with CQRS

* In a CQRS setup, **Commands** generate events rather than updating a database directly.
* The **Query side** listens to these event streams to update its own read models. This keeps the write and read sides synchronized while allowing them to scale independently.

## Practical Scenario: Updating Product Price

* **Command Service:** Validates the business rule and generates a `PriceUpdated` event.
* **Event Store:** Stores the event permanently.
* **Query Service:** Detects the new event, updates the read model, and serves the latest data to the user.

## Event Propagation and Resilience

* Events are typically distributed via message brokers (like Kafka or RabbitMQ).
* If a read model is corrupted or needs a new schema, it can be entirely rebuilt by replaying the history from the event store.

---

## Summary & Key Points

**Event Sourcing** is a powerful pattern where the "source of truth" is a sequence of events rather than a snapshot of the current state. It is best suited for complex distributed systems where auditing and historical context are vital.

**Key Takeaways:**

* **Immutable History:** You never lose data; every action is preserved in an append-only log.
* **Hydration is Core:** The system "hydrates" objects by replaying events to reach the current state.
* **Performance Fixes:** Use **Snapshots** to speed up the reconstruction of state and **Materialized Views** to provide fast reads.
* **Resilience:** Because the event log is the source of truth, you can recover from database failures or bugs by replaying history into a fresh database.
* **Synergy with CQRS:** Event sourcing provides the perfect bridge between the command (write) and query (read) sides of an application.

## References

* [Master Event Sourcing in Just 10 Minutes](https://www.youtube.com/watch?v=ID-_ic1fLkY&list=PLJq-63ZRPdBsPWE24vdpmgeRFMRQyjvvj&index=12)
