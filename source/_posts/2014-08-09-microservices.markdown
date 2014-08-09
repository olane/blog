---
layout: post
title: "Microservices"
date: 2014-08-09 17:55:55 +0100
comments: true
categories: 
---


The "Microservice Architecture" is a term that's appeared recently to describe a way of building complex software as a set of independent services which each serve a small, defined role. I find the architecture quite interesting because it approaches the concepts of modularity and maintainability from the perspective of the deployment of those modules, not just how the modules are written.

How it works
-------------

There's no precise definition of the architecture, so instead I'll describe the general design ideas and the basics of how we use microservices at Red Gate, where I'm currently an intern.

It's useful to consider a monolithic approach for comparison. The traditional model of a monolithic application is an application written, built and deployed as a single unit. All the functionality lives inside a single executable which spins up as a single process, and there's one big "run" button that starts the whole thing up. 

By contrast, a microservice architecture is built of many small services, each of which is independently buildable, deployable and runnable. You can spin up or spin down each service individually, and they don't even all have to live on the same computer.

<Figure 1. Monolithic vs Microservice Architectures>

One important question is how these microservices communicate. Microservice architectures generally favour the principle of smart endpoints and dumb pipes. That means that the method of communication between services is intentionally as minimal as possible, leaving all of the business logic in the services. This allows the services to own their own domain logic, and to remain as decoupled from the rest of the service as possible. The services, in many ways, act like classical Unix pipeline -- each service taking input, transforming it in some way, and producing some output.

One approach is to use some sort of REST-like protocol to hand messages between services. The second approach, and the one I’ll discuss, is to use a lightweight message bus like RabbitMQ.

The bus
-------

RabbitMQ is a message broker which maintains a set of queues, and can assign different types of messages to different queues based on simple rules such as header names. It is “dumb” in that it deals only with message routing -- no filtering, no transformation and no business logic happens here.

Once messages are in a queue, they can be picked up by a service and handled. Services can send messages, subscribe to as many queues as they like, and messages can be routed to multiple queues simultaneously. For example, a service could queue one message to be picked up by multiple other services, each of which could do a different job based on the information contained). Information flows through the system asynchronously.





TODO:
- libraries vs services
- Advantages
- Disadvantages

