Hystrix is not only a tool for resilience engineering but also for operations. 

This pages attempts to share some of what has been learned in operating a system with 100+ HystrixCommand types, 40+ thread pools, 10+ billion thread isolated and 200+ billion semaphore isolated command executions per day.

The screenshots and incidents are from the Netflix API system and represent either real production issues or [Latency Monkey](http://techblog.netflix.com/2011/07/netflix-simian-army.html) simulations against production.

## How to Configure and Tune a Circuit

The typical approach to deploying a new circuit has been to release it into production with liberal configuration (timeouts/threads/semaphores) and then tune it down to more strict after seeing it run through a peak production cycle.

In practice what this typically looks like is:

1. Leave at default 1000ms timeout unless it's known that more time is needed.
2. Leave threadpool at default of 10 threads unless it's known that more threads are needed.
3. Deploy to canary. If all is well proceed.
4. Run in production for 24 hours on entire fleet.
5. Rely on standard alerting and monitoring to catch issues if any.
6. After 24 hours use latency percentiles and traffic volume to calculate what the lowest values are that makes sense for the circuit.
7. Change the values live in production and monitor them using the realtime dashboards until confident.
8. Only ever look at the configuration for this circuit again if behavior or performance characteristics of the circuit change and are brought to attention via alerts and/or dashboard monitoring.

The following diagram represents a typical thought process in how to choose the size of a thread-pool, queue and execution timeout (or semaphore sizing):

[[images/thread-configuration-640.png]]

Most circuits should try to have their timeout values close to the 99.5th percentile of a normal healthy system so they cut off bad requests and not let them take up system resources or affect user behavior.

Thread-pools and queues must be sized so they are a small percentage of overall application resources otherwise they will fail to restrict a dependency from saturating available resources.

The important things about configuring and tuning circuits are:
* tuning should be done in production based on real traffic patterns
* settings can be adjusted easily in realtime while monitoring to see impact of different settings

## Expect Jitter and Failure

Measuring and reporting metrics with millisecond granularity reveals "jitter" seen as bursts of timeouts, thread-pool rejections, slow downs and other such things. In a large cluster there are generally some occurring at any given point in the time for a high-volume circuit. 

This granularity at which metrics are captured by Hystrix is something many software systems don't have so it's surprising to many what is seen once looking at it. 

In this screenshot from the Netflix API dashboard monitoring HystrixCommands in production you can see the orange and purple numbers with timeouts and threadpool rejections occurring for a small number of requests in a 10-second statistical window representing 243 servers.

[[images/circuit-identity-jitter-640.png]]

Most systems are measured at a fairly high-level - even if broken into percentile latencies it's done per minute. Also, often it's done for an entire application request loop, not each individual dependency that is interacted with.

In other words, once you have the magnifying glass showing you what's going on with each dependency don't be surprised to see jitter.

Some of the causes:

* client machine garbage collection (your machine does a garbage collection in the middle of a request)
* service machine garbage collection (remote server does a garbage collection in the middle of a request to it)
* network issues
* different payload sizes for different request arguments
* cache misses
* bursty call patterns
* new machines starting up (deployments, auto-scale events) and "warming up"


## When Things Are Latent

Don't react by touching things. If a HystrixCommand is shedding load it's doing what it's supposed to _(assuming you've configured it correctly when it's healthy of course – see above)_.

In the early days of Hystrix being used at Netflix it was a common reaction by many when a circuit (what we internally call a HystrixCommand/CircuitBreaker pairing) became latent to dynamically change properties to increase thread-pools, queues, timeouts etc to "try and give it some breathing room" and get it working again.

That is the opposite of what should be done. If the command was configured correctly for a healthy system and is now rejecting, timing out, and/or short-circuiting then get the underlying root cause fixed - don't give it more resources that it can use up.

At an extreme you are causing a DDOS on yourselves by increasing the size of thread-pools, queues, timeouts, semaphores and the like.

For example, if you have a cluster of 100 servers each with 10 concurrent connections to a service allowed that is 1000 possible concurrent connections.

When healthy it normally is using 200-300 of them at any given time.

If latency occurs and backs them all up you are now using 1000 connections. 10 per box may not seem much for the client so let's try increasing it to 20, right?

Most likely if 10 were saturated, 20 will become saturated as well.

Now you have 2000 connections held open against the back end making things even worse.

This is one of the reasons why the circuit breaker exists - to "release the pressure" on underlying systems to let it recover instead of pounding it with more requests in retry loops, hung connections and the like.

For example, here is an example of a single dependency having latency resulting in timeouts high enough to cause the circuitbreaker to trip on about 1/3 of the cluster. It is the only circuit in the system having health issues and Hystrix is preventing it from taking other resources while it has latency problems.

[[images/ops-social-640.png]]

In short, let the system shed load, short-circuit, timeout and reject until the underlying system is healthy again and it will take care of itself and come back to health at the Hystrix layer. Hystrix is designed for exactly this scenario and the point is to reduce resource utilization by latent systems so that recovery can occur quickly by keeping most resources isolated and away from those that are hung up on a latent connection.

## What Dependency Failure Looks Like

The most typical type of failure in a distributed system is for a single dependency to fail or become latent while all others remain healthy.

In these cases the metrics and dashboards are very obvious in showing what is happening:

[[images/ops-cinematch-1-640.png]]

In that screenshot we have a single circuit with a 20% error rate. High enough to have impact but not enough to start tripping the circuit breakers. The other 3 circuits shown in the screenshot are unaffected.

In this particular example it is actual errors, not latency causing the problem - as shown by the red numbers instead of orange.

The following charts were captured during the same incident to show the historical trend of this circuit and how it spiked in failures and fallbacks.

[[images/ops-cinematch-2-640.png]]

## Dependency Failure with Fallback

Here is a screenshot of another incident affecting a single circuit. Note how the 99.5th percentile latency is very high. That's how long the underlying worker threads were taking to complete which would in turn saturate the thread pools and be timed out on from the calling thread.

All but one machine in the cluster has the circuit breaker tripped which accounts for most traffic being short-circuited (blue) and on the machine still trying most are being timed out (orange).

[[images/ops-getbookmarks-640.png]]

Note that the other circuits are healthy and the line graph on the left shows no increase in 500s being returned since this circuit is returning a fallback so users are receiving a degraded but still functional experience.

## Cascading Dependency Failures

This screenshot represents a failure (in this case high latency) of a single system that is heavily depended upon by many other systems and so the failure cascaded across them.

Ultimately this meant that the Netflix API system had to be resilient against latency and failure not from just the single root cause but all of the systems impacted by it as well. 

In this particular screenshot, 6 circuits representing 3 different systems are shown:

[[images/ops-ab-640.png]]

At the time of this incident Hystrix was still mostly a "Netflix-API-only" thing. As Hystrix rolls across more and more teams internally it limits the impact of cascading failure such as this next diagram illustrates:

[[images/cascading-failure-preventing-640.png]]

## When It's You, Not The Dependency

If all of the circuits are seemingly bad (the dashboard is all lit up) then there's a good chance the problem is your system, not all of your dependencies at the same time.

[[images/ops-complete-system-640.png]]

Two examples of system problems that can cause this are:

* System is overloaded (high load average, cpu usage, etc).
  * An example of when this can occur is if autoscaling policies fail or don't scale fast enough with traffic surges and machines are receiving more traffic than they can handle.
* Memory leak that eventually causes GC thrashing which steals CPU and causes pauses which in turn causes circuits to timeout, backup and reject.