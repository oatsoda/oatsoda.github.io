---
title: "Microservices: A primer"
date: 2023-09-22 10:16:56
tags:
  - Architecture
  - Microservices
  - Event-Driven
---

## Microservices: A primer

**_Microservices are a hugely popular architecture pattern and there are many articles written about them. But some of these aren not clear about why and how. This article aims to set out some calm, clear, understanding._**

### What are microservices?

The microservices pattern at its core is architecting an application to be many smaller separate services - generally quite small in size - which operate as one or more processes and whose data and logic is owned and encapsulated by that service.

### Why use microservices?

There are four benefits to using microservices:

1. The **Primary** benefit is that each individual microservice can be scaled separately to meet demand. This can reduce costs and allow faster scaling because your scaling units are therefore much smaller.
2. Using microservices adds some resilience to your application. If a microservice is down or offline, then it should allow your application to continue in a degraded form as each service is hosted separately and your architecture should allow data to be buffered and picked up where it left off when the service is back up.
3. By having smaller individual services, development can more easily take place in parallel as changes are likely to be smaller and altering the logic or data of the individual service will not directly impact the rest of the application.
4. Deployment of your application can take place per individual service, so changes can be deployed much faster as the entire application does not need deploying each time.

There is a fifth reason that you _could_ give for using Microservices, but personally I don't think this can be given as an absolute benefit:

- Using microservices means the logic is much simpler in each service, so making changes in each service is much easier.

This is true, but the reason that I would not consider it a benefit is that there is an equal-and-opposite truth:

- Using a microservices architecture adds complexity to your application. It can no longer simply execute all logic in one process.

The two cancel each other out, so I do not think you should consider this benefit as a reason for implementing a microservices architecture. Otherwise you may find that you have made your application needlessly complex for no benefit.

### What are the charateristics of a microservice?

#### Data Sovreignty

Any data stored by the microservice must be wholly owned and managed by the service and not altered or read by any other service.

#### API or Message communication

All microservices will need a way to communicate with the outside world, but this depends on what behaviours they provide. Some might be called by Front Ends, some might be called by other microservices, some may do both. Depending on what's required, this communication could be via API (such as REST, RPC or GraphQL) or by a Messaging mechanism like a message broker or queue.

#### Async "Chained" microservice communication

The nature of splitting up your application in to parts means that it is very likely that microservices will need to talk to each other. It is very important that should these end up in a chain: i.e. a call to microservice 1 needs to internally call microservice 2 to satisfy the original request, that it must not cause a chain of synchronously blocked requests otherwise your services will become overloaded with requests very quickly. Instead, use asynchronous communication.

It is fine for a Front End calling an API to GET data, for example, to still be a blocking call on that first service - it is just any onward calls which that first service makes that should ensure not to block threads.

#### DDD Modelling

Bounded context analysis.

### Message Based Architectures

DDD helps.

Confusion over tooling (MassTransit etc.)
