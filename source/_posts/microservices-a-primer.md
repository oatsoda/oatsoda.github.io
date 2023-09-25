---
title: "Microservices: A primer"
date: 2023-09-22 10:16:56
tags:
  - Architecture
  - Microservices
  - Event-Driven
---

**_Microservices are a hugely popular architecture pattern and there are many articles written about it. Some of these are unclear about *why* and *how*. This article aims to set out some calm, clear, understanding._**

## What are microservices?

The microservices pattern at its core is architecting an application to be many smaller, separate, services - generally quite small in size - which operate as one or more processes and whose data and logic is owned and encapsulated by that service.

## Why use microservices?

There are four benefits to using microservices:

1. The **primary** benefit is that each individual microservice can be scaled separately to meet demand. This can reduce costs and allow faster scaling because your scaling units are therefore much smaller.
2. Using microservices adds some resilience to your application. If a microservice is down or offline, then it should allow your application to continue in a degraded form as each service is hosted separately and your architecture should allow data to be buffered and picked up where it left off when the service is back up.
3. Having smaller individual services means that development can more easily take place in parallel as changes are likely to be smaller and altering the logic or data of the individual service will not directly impact the rest of the application. It also means that each service can be architected and implemented in its own way: even in a different programming language.
4. Deployment of your application can take place per individual service, so changes can be deployed much faster as the entire application does not need deploying each time.

There is a fifth reason that you _could_ give for using Microservices:

- Using microservices means the logic is much simpler in each service, so making changes in each service is much easier.

This is true, but the reason that I would not consider it a benefit is that there is an equal-and-opposite downside:

- Using a microservices architecture adds complexity to your application. It can no longer simply execute all logic in one process.

The two cancel each other out, so I do not think you should consider this as a reason for choosing a microservices architecture. Otherwise you may find that you have made your application needlessly complex for no benefit.

## Designing microservices using DDD Bounded Contexts

The best way of deciding how to structure your microservices is using [Domain Driven Design](https://en.wikipedia.org/wiki/Domain-driven_design) (DDD) and identifying the [Bounded Contexts](https://martinfowler.com/bliki/BoundedContext.html).

These will allow you to identify the "seams" between contexts in your application where there is very little logic which needs to cross over between those contexts.

For example, in a basic online shop, the user's details such as name, profile and avatar have very little cross-over with the ordering part of your application. Managing the user's name, profile and avatar is done very separately from creating or fulfilling an order. Equally, a stock inventory is likely to be managed very separately from both user and order contexts.

So you may decide that users, orders and stock are three bounded contexts and could be represented as separate microservices. Of course, this doesn't mean you cannot have places in your application which aggregate that data to display it - such as in an orders UI or in a label printing part of the application.

## What are the characteristics of a microservice?

### Data Sovereignty

Any data stored by the microservice must be wholly owned and managed by the service and not altered or read by any other microservice. Changes to the internal workings of a microservice must not impact any other microservice; only changes to its public interfaces can impact others.

Note that a microservice can have multiple processes and those processes can all manage the data as they are part of the internal implementation of that microservice.

### Data Duplication & Eventual Consistency

Whilst the above point on data sovereignty means that no other microservice can manage the internal data or state, it does not mean the same data cannot live in more than one microservice.

In fact, when designing a microservice architecture there will often be times when data from multiple bounded contexts is required. You may be able to request this data from each microservice and aggregate it somewhere in your application each time it is required. This can create slow responses and blockages, so instead, data can be duplicated in other microservices where required.

For example, in our basic online shop, the orders bounded context might often require the name of the ordering user. Rather than requesting this from the users microservice each time, the name can be made part of the orders microservice too.

This duplication is fine; in fact, with respect to DDD bounded contexts, whilst the data has the same value and seems equivalent, the context of it is slightly different. In the users service it is the "user's profile name" whereas in the orders service it is the "orderer's name".

When the user updates their name in the users microservice, it will also need to be changed in the other service that requires it. This is not transactional as data sovereignty means they are managed separately. Instead we use eventual consistency; the idea that _eventually_ this data will be in sync. This is fine in almost all cases because this delay is minimal and not mission critical. If you did have any mission critical situations, it may be that your microservice boundaries are not correct.

To provide this eventually consistent data synchronisation, you would use an asynchronous message-based architecture.

### API or Message communication

Every microservice will need a way to communicate with the outside world, but this depends on the behaviours which they provide. Some might be called by front-ends, some might be called by other microservices, some may do both. Depending on what's required, this communication could be via API (such as REST, RPC or GraphQL) or by a Messaging mechanism like a message broker or queue.

### Asynchronous "Chained" microservice communication

The nature of splitting up your application into parts means it is very likely that microservices will need to talk to each other. It is very important that if these calls end up in a chain: i.e. a call to microservice 1 needs to internally call microservice 2 to satisfy the original request, that it [must not cause a chain of _synchronously_ blocked requests](https://learn.microsoft.com/en-us/dotnet/architecture/microservices/architect-microservice-container-applications/communication-in-microservice-architecture#asynchronous-microservice-integration-enforces-microservices-autonomy) otherwise your services will become overloaded with requests very quickly. Instead, use asynchronous communication such as messaging or even polling.

Ideally you want to avoid any chains like this, but that could mean duplicating a lot of data between microservices. In practice, initially, you may want to allow chaining for some front-facing APIs until you can later optimise. You may also later need to re-evaluate whether you have drawn your bounded contexts correctly.

Note that there are some approaches like API [Gateway aggregation](https://learn.microsoft.com/en-us/azure/architecture/patterns/gateway-aggregation) and [Backend-for-Frontend](https://samnewman.io/patterns/architectural/bff/) (BFF) which do actively use chaining of requests.

## Using Message-Based Architectures

Above, I have mentioned both eventual consistency and also a need to avoid chained blocking calls between microservices.

A message based architecture is the best approach for implementing most intercommunication.

A simple way to think of this is that you publish a message to a broker or queue and one or more interested services pick up the message and act upon it. This makes the communication asynchronous.

### Event-Based Logical Architecture

When talking about message-based architectures, this is referring to the physical architecture. A message is used to communicate between processes.

On top of that physical architecture, it is best to model the logical architecture to use an event-based architecture.

This is the idea that those messages are in fact events, no matter how or who they get delivered to.

For example, in the eventual consistency example earlier - where the user updates their name in the users service and the orders service needs to be updated - could be modelled as the "User name updated" event. The users orders service can publish this to a broker and the orders service can handle the event to update the orderer's name.

The event could just contain the user's identifier, or it could contain all of the data that the user changed. The former requires the orders service to request the data from the users service, the latter approach requires more data on the message. This latter approach is called [Event-Carried State Transfer](https://martinfowler.com/articles/201701-event-driven.html#Event-carriedStateTransfer).

### Resilience

As mentioned in the initial section of this article, using a microservices architecture means that you don't have the benefits of resilient transactional based updates to data.

But even with simple monolithic applications, synchronising internal data with external third-parties is not resilient.

Message-based architectures aid with that resilience - while adding complexity too - because their async nature means that messages can be tracked and retried.

Discussing this resilience is beyond this article; Some key points regarding this are:

- A [Transactional Outbox](https://en.wikipedia.org/wiki/Inbox_and_outbox_pattern#The_outbox_pattern) can be used to ensure data updates and events are kept in sync by ensuring that failures that occur after a transactional update to a database, for example, but before an event can be raised, will be recorded and retryable.
- A [Transactional Inbox]() can be used to ensure subscribers handling events successfully handle them at least once.
- Ensuring [Idempotence](https://en.wikipedia.org/wiki/Idempotence) - where multiple executions with the same input parameters causes no additional effect on the outcome - can be used to be resilient against at-least-once delivery and retries.

### Internal communication

It's worth mentioning that message-based communication can also be used within the implementation of a microservice itself; it's not just a pattern for communicating between separate microservices.

For example, a microservice which has a public API to update the email address of a user might want to send an email to that address once updated. Initially this call might be made from within the API handler, but over time for performance and resilience reasons you move it into an async process within the microservice - so an event is published once the update to the user is made and the async process subscribes to that to send the email.

When reading documentation or looking at examples of various frameworks for message-based architectures, bear this in mind as it is not always clear when messages are being used for internal async processing or external inter-microservice communication.

### Frameworks

In the .NET ecosystem there are a number of frameworks that can be used. For example:

- [MassTransit](https://masstransit.io/)
- [Brighter](https://www.goparamore.io/)
- [Wolverine](https://wolverine.netlify.app/)

Some of the introductory documentation gives great overviews (especially Wolverine) of the architectural patterns; But as I state above, _there is not always clarity in the examples between inter-microservice communication and internal async processing_.

Your architecture could get very confusing and messy if you don't have this clarity.

## Further topics

There are further topics which are worth discussing, but this article is long enough already.

These additional topics include:

- Implementing a logical event-based architecture using DDD modelling with Domain Events and Integration Events.
- Using API [Gateway aggregation](https://learn.microsoft.com/en-us/azure/architecture/patterns/gateway-aggregation)/[BFF](https://samnewman.io/patterns/architectural/bff/) to simplify your front-end application and avoid exposing your microservice structure to a public front-end (plus other benefits).
- Resilience patterns (as noted above) in async communication.
- Other resilience patterns such as circuit breaker for coping with temporarily offline services.

For further reading, I recomment [.NET Microservices: Architecture for Containerized .NET Applications](https://learn.microsoft.com/en-us/dotnet/architecture/microservices/) (available as a [PDF](http://aka.ms/MicroservicesEbook) too) - the chapters on containerization can be skipped if you prefer to just read about microservices as the content is separate from that of containers.
