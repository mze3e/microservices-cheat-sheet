See also: [API Design Cheat Sheet](https://github.com/RestCheatSheet/api-cheat-sheet#api-design-cheat-sheet),
[Platform Building Cheat Sheet](https://github.com/RestCheatSheet/platform-cheat-sheet#platform-building-cheat-sheet)

# Microservices Cheat Sheet
1. Do only one thing and do it well. The "one thing" is defined by a "Bounded Context" in Domain-Driven Design (DDD).
1. Own your own data. No shared data stores.
1. Embrace eventual consistency.
    * Don't read your writes.
    * Publish your own state-changes (minimally) to an event log.
1. Leverage event logging and/or streaming to replicate and denormalize data from other services.
    * The event log must be subscribable to *publish* events to subscribers.
    * The event log must be durable and must not lose messages for publishing and replay.
    * Examples: [Kafka](http://kafka.apache.org/), [Confluent.io](http://www.confluent.io), [Amazon Kinesis](https://aws.amazon.com/kinesis/).
1. When aggregating microservice calls, use asynchronous, non-blocking I/O. Perform as much as possible asynchronously.
1. Support a registry/discovery mechanism.
    * Examples: [Consul](https://www.consul.io/), [Eureka](https://github.com/Netflix/eureka)
1. Support Location Transparency.
    * Support statelessness--no sticky sessions.
1. Support the use of back-pressure (see, Reactive Streams specification) to avoid cascading failures.
    * Examples: [RxJava](https://github.com/ReactiveX/RxJava/wiki/Backpressure)
1. Support the Circuit Breaker pattern to manage faulty service dependencies.
    * Examples: [Hystrix](https://github.com/Netflix/Hystrix), [Apache Zest](https://zest.apache.org/java/2.1/library-circuitbreaker.html)
1. Consider CQRS to scale reads separately from writes.


# Controller Query Responsibility Segregation (CQRS) Architecture

![alt text](CQRS%20Architecture.jpeg?raw=true "CQRS Architecture")

* Controller:* A.K.A. UI Controller. It is responsible for intercepting both read and write operations coming from the client. So whenever a call is made to the client, the request immediately routes to the controller. All the operations, GET POST etc. reach the controller first before getting routed anywhere.

* *Requests / Commands:* A request from a client could either be a *C*ommand or a *Q*uery.  
   - *Command:* A command represents an intent to modify the state of an entity. This is normally in form of a DELETE, POST or PUT request.
   - *Query:* A query represents a read operation from an entity. i.e. usually it's a GET request. A request for data without making anyd changes. When a query comes to the controller, it is routed to the query manager. Then the query manager executes the query and returns the results to the client.

* *Command Gateway:* Provides an interface to dispatch your commands. Any command coming from the controller goes to the Command Gateway. The dispatch could be either synchronous or asynchronous.

* *Command Handler:*  Receives and responds to specific type of commands. It then executes business rule or logic based on the command. Command Handler is also responsible to fetch the domain entity and make changes to them.

* *Domain Entity:* Domain Entities are aggregate entities that are modeled to reprsent the state of the components in the domain. An aggregate is an entity or group of entities that are always kept in a consistent state. For example, an Employee entity could be an aggregate. It could have additional aggregates like Address, NextOfKin etc. If the state of an aggregate changes, then a domain event is generated.

* *Repository:* Provides functionality for managing aggregates. For example, a repository helps you find, save or delete an aggregate. A repository, could interface with a persistent storage (like a database) to store the aggregates. A repository could also use an event store to call up and aggregate and recreate it’s state at any point in history based on saved states.

* *Event:* A notification that there has been a change to the state of an entity. So when an update is made, then an event is emitted to say, what changes actually took place.

* *Event Store:* This is a storage of all the changes that have been applied to an entity. It is like a history of events. An event store can replay these historical changes and therefore rebuild the state of an entity at any point in time. Event store may also use a database for event storage.

* *Event Bus:* This is like a channel for generated events. They are normally backed up by a messaging topic in a publish/subscribe manner.

* *Event Handler:* These are methods the receive events and handle the events. For example, when an event is emited for an update to an entity, then an event handler recieves this event from the event bus. It could then update the WriteDB by mirroring this changes to materialized views.
