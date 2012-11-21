## What is Hystrix?

In a distributed environment, failure of any given service is inevitable.  Hystrix is a library designed to control the interactions between these distributed services providing greater latency and fault tolerance.  Hystrix does this by isolating points of access between the services, stopping cascading failures across them, and providing fallback options, all of which improve the system's overall resiliency.

Hystrix evolved out of resilience engineering work that the Netflix API team began in 2011.  Over the course of 2012, Hystrix continued to evolve and mature, eventually leading to adoption across many teams within Netflix.  Today tens of billions of thread-isolated and hundreds of billions of semaphore-isolated calls are executed via Hystrix every day at Netflix and a dramatic improvement in uptime and resilience has been achieved through its use.

The following links provide more context around Hystrix and the challenges that it attempts to address:

* [Making Netflix API More Resilient](http://techblog.netflix.com/2011/12/making-netflix-api-more-resilient.html)
* [Fault Tolerance in a High Volume, Distributed System](http://techblog.netflix.com/2012/02/fault-tolerance-in-high-volume.html)
* [Performance and Fault Tolerance for the Netflix API](https://speakerdeck.com/benjchristensen/performance-and-fault-tolerance-for-the-netflix-api-august-2012)


## Purpose of Hystrix

* Give protection from and control over latency and failure from dependencies (typically accessed over network) accessed via 3rd party client libraries.
* Stop cascading failures in a complex distributed system. 
* Fail fast and rapidly recover. 
* Fallback and gracefully degrade when possible.
* Enable near real-time monitoring, alerting and operational control.

## Problem Definition

Applications in complex distributed architectures have dozens of dependencies that can and will fail and if not isolated from these points of failure the entire application will be taken down with them.

For example, running an application that depends on 30 services that each have 99.99% uptime we get:

>99.99<sup>30</sup>  =  99.7% uptime  
>0.3% of 1 billion requests = 3,000,000 failures  
>2+ hours downtime/month even if all dependencies have excellent uptime.  

Reality is generally worse.

Even when all dependencies are performing well the aggregate impact of even 0.01% downtime on each of dozens of services equates to potentially hours a month of downtime __if not engineered for resilience__. 

***

When everything is healthy the request flow can look like this:

[[images/soa-1-640.png]]

When one of many backend systems becomes latent it can block the entire user request:

[[images/soa-2-640.png]]

With high volume traffic a single backend dependency becoming latent can cause all resources to become saturated in seconds on all servers.

Latency is far worse for system resilience than failure. Failures naturally “fail fast” and shed load so systems can typically process the failures quickly, continue serving other requests and recover quickly.

Latency on the other hand backs up queues, threads and system resources and if isolation techniques are not used it can cause an entire system to fail. 

A system failing due to latency is often expressed in ways such as:

* rejecting incoming requests with 503s because the HTTP acceptor thread pool is saturated because all threads are blocked on slow backend calls
* user responses become very slow because they end up in network connection queues or HTTP accept queues
* no other backend systems can be connected to even if they are healthy because the connection pools are saturated and blocked on a single latent system

[[images/soa-3-640.png]]

Every point in an application that reaches out over the network or into a client library that can potentially result in network requests is a source of failure or worse, latency.

These issues are exacerbated when network access is performed through 3rd party clients which act as a "black box" where implementation details are hidden, can change at any time, and network or resource configurations are different for each client library and often difficult to monitor and change. 

Even worse are transitive dependencies that get picked up and perform potentially expensive or fault-prone network calls without explicit invocation from the application.

Network connections fail or degrade. Services and servers fail or become slow. New libraries or service deployments change behavior or performance characteristics. Client libraries have bugs. 

All of these represent failure and latency that needs to be isolated and managed so a single dependency failing can't take down an entire application or system.

## Design Principles

* Restrict any single dependency from using up all container (such as Tomcat) user threads.
* Shed load and fail fast instead of queueing.
* Provide fallbacks wherever feasible to protect users from failure
* Use isolation techniques (such as bulkhead, swimlane and circuit breaker patterns) to limit impact of any one dependency.
* Optimize for time-to-discovery through near real-time metrics, monitoring and alerting
* Optimize for time-to-recovery with low latency propagation of configuration changes and support for dynamic property changes in virtually all aspects of Hystrix to allow real-time operational modifications with low latency feedback loops.
* Protect against entire dependency client execution, not just network traffic.

## How does Hystrix accomplish this?

* Wrap all calls to external systems (dependencies) in a HystrixCommand object (command pattern) which typically executes within a separate thread.
* Time-out calls that take longer than defined thresholds. A default exists but for most dependencies is custom-set via properties to be just slightly higher than the measured 99.5th percentile performance for each dependency.
* Maintain a small thread-pool (or semaphore) for each dependency and if it becomes full commands will be immediately rejected instead of queued up.
* Measure success, failures (exceptions thrown by client), timeouts, and thread rejections.
* Trip a circuit-breaker automatically or manually to stop all requests to that service for a period of time if error percentage passes a threshold.
* Perform fallback logic when a request fails, is rejected, timed-out or short-circuited.
* Monitor metrics and configuration change in near real-time.

When Hystrix is used to wrap each underlying dependency the architecture as shown in diagrams above changes to the following where each dependency is isolated from each other, restricted in the resources it can saturate when latency occurs and covered in fallback logic to decide what happens to a user response when any type of failure occurs:

[[images/soa-4-isolation-640.png]]

Learn more about [[How It Works]] and [[How To Use]].