---
layout: post
title: "Microservices"
date: 2014-08-09 17:55:55 +0100
comments: true
categories: 
---


The "Microservice Architecture" is a term that's flurried around the blogosphere recently to describe a way of building complex software as a set of independent services which each serve a small, defined role. I find the architecture quite interesting because it approaches the concepts of modularity and maintainability from the perspective of how an application's modules are deployed, not just how they are written.

How it works
-------------

There's no precise definition of the architecture, so instead I'll describe the general design ideas and the basics of how we use microservices at [Red Gate](http://www.red-gate.com), where I'm currently a software intern.

It's useful to consider a monolithic approach for comparison. The traditional model of a monolithic application is an application written, built and deployed as a single unit. All the functionality lives inside a single executable which spins up as a single process, and there's one big "run" button that starts the whole thing up. 

By contrast, a microservice architecture is built of many small services, each of which is independently buildable, deployable and runnable. You can spin up or spin down each service individually, and they don't even all have to live on the same box.

{% img /images/microservice_diagram.svg 690 %}

One important question is how these microservices communicate. Microservice architectures generally favour the principle of smart endpoints and dumb pipes. That means that the method of communication between services is intentionally as minimal as possible, leaving all of the business logic in the services. This allows the services to own their own domain logic, and to remain as decoupled from the rest of the service as possible. The services, in many ways, act like classical Unix pipeline -- each service taking input, transforming it in some way, and producing some output.

An approach you could use is to hand messages between services with some sort of REST-ish protocol. The second approach, and the one I’ll discuss, is to use a lightweight message bus like [RabbitMQ](http://www.rabbitmq.com/).

The bus
-------

RabbitMQ is a message broker which maintains a set of queues, and can assign different types of messages to different queues based on simple rules such as header names. It is “dumb” in that it deals only with message routing -- no filtering, no transformation and no business logic happens here.

{% img /images/AMQP.svg 'Source: Wikimedia commons' %}

Once messages are in a queue, they can be picked up by a service and handled. Services can send messages, subscribe to as many queues as they like, and messages can be routed to multiple queues simultaneously. For example, a service could queue one message to be picked up by multiple other services, each of which could do a different job based on the information contained). Information flows through the system asynchronously.

For example, say a customer buys a product using your website. The website could queue some sort of `product.bought` messsage to alert any interested services that this had happened. One service might pick up that message and subscribe the customer to a mailing list based on the purchase, another might log the event for metrics purposes, and another could send them a billing email.

One thing to note is that the bus *only* handles asynchronous messages. If you have a service that needs to return an answer synchronously, like the API for some database, you'll need to use the REST-ish approach.


Advantages
----------

The microservice approach has a ton of advantages over the monolith architecture:

+ Firstly, **services are simple**. Each service can concentrate on doing only one thing, and doing it well. This is essentially the basis of the [Unix Philosophy](http://en.wikipedia.org/wiki/Unix_philosophy).

+ **You can scale each module individually**. A monolithic app scales by duplicating the app onto two or more servers, and adding some sort of load balancing to route requests to each version. By contrast, if a service in your microservice app becomes a bottleneck, you can scale it by simply spinning up a new version of that service, and binding it to the same queues. The two sibling services will happily coexist, taking turns to grab messages from the queue.

+ **You can deploy modules individually**. If you make a small change to your metrics implementation, there's no need to re-deploy the whole app. Just deploy the affected service and you're good to go.

+ On a related note, a microservice approach primarily approaches componentization through services, instead of libraries. If you have a piece of shared functionality, you can often make it into its own service instead of importing it everywhere it's needed as a library. This is useful because it means that **changing a service's functionality only requires you to re-deploy that service**. If you'd imported a library into lots of places, on the other hand, you'd have to re-deploy every component that used the library.

+ **Services can easily pick back up where they left off**. This one's a consequence of the asychronous nature of the message bus. If a service goes down, or gets shut down and re-deployed, the unhandled messages will just sit waiting on the queue. Once the service comes back up, it can pick up where it left off.

+ **You can use different languages for each service**. You can freely use C++ to handle the gnarly performance-critical code and ClojureScript for your fancy hipster metrics service.

+ **Services can own their own data**. The architecture strongly encourages you to modularise everything, right down to the data. A monolithic application will often store everything in one colossal database, but a service-based approach will usually have a separate service responsible for each section of the data. This is entirely possible to do with a monolithic architecture, but the more loose coupling is encouraged the better.



Disadvantages
-------------

+ **It's hard to refactor if you get the boundaries wrong**. The boundaries between services are akin to the interfaces between modules in an object-oriented application. However, they're much harder to refactor, because all of the services interacting with the interface must usually be changed and re-deployed simultaneously. Careful thought is required during these deployments because they could cause non-obvious interactions with the live system.

+ Partly as a result of the above point, **developers need to be very aware of the production environment**. As a distributed, asynchronous system, a microservice architecture can be difficult to reason about and deployment of the system can't really be left to a separate team. Developers must either deploy and maintain the system themselves, or work closely with those that do. This can be a challenging, if rewarding, way of working.

+ **Robust automation in build and deployment systems is needed**. Going from a monolithic app to a microservice app could mean multiplying the number of things you have to deploy by 20-30! This can be managed, but it requires investment in robust, well understood tooling. Luckily, this space is improving all the time.

+ **Significant overhead**. All those messages do come at a performance cost. This isn't ideal for performance critical infrastructures.

+ **Emergent behaviour is hard to test**. Although each service is simple, once you get dozens of them interacting with each other, they start to become hard to reason about and even harder to automate tests for.


Conclusion
-----------

I've had a lot of fun working with a microservice architecture at Red Gate, and it's taught me a lot about application architectures. Microservices, like any architecture, are definitely not a silver bullet. However, they're a very powerful tool if used in the right context for the right reasons, and they're certainly an interesting topic to delve into.

Further reading:

+ [The human benefits of a microservice architecture](http://damianm.com/articles/human-benefits-of-a-microservice-architecture/)
+ [Martin Fowler's summary of microservices](http://martinfowler.com/articles/microservices.html)
+ [Microservices -- Not a free lunch!](http://highscalability.com/blog/2014/4/8/microservices-not-a-free-lunch.html)

