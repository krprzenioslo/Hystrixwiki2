<a name='Flow'/>
## Flow Chart

The following diagram shows what happens when you make a request to a service dependency by means of Hystrix:

<a href="images/hystrix-flow-chart-original.png">[[images/hystrix-flow-chart-640.png]]</a>
_(Click for larger view)_

__(1) Construct HystrixCommand Command Object__

You first construct a [HystrixCommand](http://netflix.github.com/Hystrix/javadoc/index.html?com/netflix/hystrix/HystrixCommand.html) object with the necessary arguments for making the call to the dependency.

For example:
```java
HystrixCommand command = new HystrixCommand(arg1, arg2);
```

__(2) Execution Synchronously or Asynchronously__

You can then choose to execute the command either synchronously or asychronously:

```java
K value = command.execute();
```

```java
Future<K> value = command.queue();
Observable<K> oValue = command.observe(); // not shown in diagram above
```

The synchronous call [execute()](http://netflix.github.com/Hystrix/javadoc/index.html?com/netflix/hystrix/HystrixCommand.html#execute\(\)) invokes [queue().get()](http://netflix.github.com/Hystrix/javadoc/index.html?com/netflix/hystrix/HystrixCommand.html#queue\(\)) unless the command is specified to not run in a thread.

__(3) Is Circuit Open?__

When you execute the command, Hystrix first checks with the circuit-breaker to ask, &ldquo;Is the circuit open?&rdquo;

If the circuit is open (or &ldquo;tripped&rdquo;) then Hystrix will not execute the command but will route the flow to (8) [HystrixCommand.getFallback()](http://netflix.github.com/Hystrix/javadoc/index.html?com/netflix/hystrix/HystrixCommand.html#getFallback\(\)).

If the circuit is closed then the flow proceeds to (4) to check if there is capacity available to run the command.

__(4) Is Thread Pool/Queue/Semaphore Full?__

If the thread-pool and queue (or semaphore, if not running in a thread) that are associated with the command are full then Hystrix will not execute the command but will immediately route the flow to (8) [HystrixCommand.getFallback()](http://netflix.github.com/Hystrix/javadoc/index.html?com/netflix/hystrix/HystrixCommand.html#getFallback\(\)).

__(5) HystrixCommand.run()__

Here, Hystrix executes the concrete implementation of the [run()](http://netflix.github.com/Hystrix/javadoc/index.html?com/netflix/hystrix/HystrixCommand.html#run\(\)) method.

__(5a) Command Timeout__

If you have configured the command to run within a thread and the [run()](http://netflix.github.com/Hystrix/javadoc/index.html?com/netflix/hystrix/HystrixCommand.html#run\(\)) method exceeds the command&#8217;s timeout value, the thread will throw a `TimeoutException`. In that case Hystrix routes the response through _(8) [HystrixCommand.getFallback()](http://netflix.github.com/Hystrix/javadoc/index.html?com/netflix/hystrix/HystrixCommand.html#getFallback\(\))_ and it discards the eventual return value of the [run()](http://netflix.github.com/Hystrix/javadoc/index.html?com/netflix/hystrix/HystrixCommand.html#run\(\)) method (if it does not cancel/interrupt).

If the command does not run within a thread then this logic will not be applicable.

__(6) Is Command Successful?__

Hystrix routes application flow based on the response from the _(5) [HystrixCommand.run()](http://netflix.github.com/Hystrix/javadoc/index.html?com/netflix/hystrix/HystrixCommand.html#run\(\))_ method.

__(6a) Successful Response__

If the command did not throw any exceptions and it returned a response, Hystrix returns this response after it performs some some logging and metrics reporting.

__(6b) Failed Response__

If the command did throw an exception, Hystrix marks it as &ldquo;failed&rdquo &mdash; which will contribute to potentially tripping the circuit &mdash; and routes application flow to _(8) [HystrixCommand.getFallback()](http://netflix.github.com/Hystrix/javadoc/index.html?com/netflix/hystrix/HystrixCommand.html#getFallback\(\))._

__(7) Calculate Circuit Health__

Hystrix reports successes, failures, rejections, and timeouts to the circuit breaker, which maintains a rolling set of counters that calculate statistics.

It uses these stats to determine when the circuit should &ldquo;trip,&rdquo; at which point it short-circuits any subsequent requests until a recovery period elapses, upon which it closes the circuit again after first checking certain health checks.

__(8) HystrixCommand.getFallback()__

Hystrix reverts to the fallback whenever a command execution fails: when an exception is thrown by (5) [HystrixCommand.run()](http://netflix.github.com/Hystrix/javadoc/index.html?com/netflix/hystrix/HystrixCommand.html#run\(\))), when the command is _(3) short-circuited_ because the circuit is open, or when the command&#8217;s _(4) thread pool and queue or semaphore_ are at capacity.

The fallback provides a generic response, without any network dependency, from an in-memory cache or via other static logic.

_If you must use a network call in the fallback, you should do so by means of another [HystrixCommand](http://netflix.github.com/Hystrix/javadoc/index.html?com/netflix/hystrix/HystrixCommand.html)._

__(8a) Fallback Not Implemented__

If you have not implemented [HystrixCommand.getFallback()](http://netflix.github.com/Hystrix/javadoc/index.html?com/netflix/hystrix/HystrixCommand.html#getFallback\(\)) Hystrix throws an exception and the caller is left to deal with it.

__(8b) Fallback Successful__

If the fallback returns a response then Hystrix will return it to the caller.

__(8c) Fallback Failed__

If [HystrixCommand.getFallback()](http://netflix.github.com/Hystrix/javadoc/index.html?com/netflix/hystrix/HystrixCommand.html#getFallback\(\)) fails and throws an exception then the caller is left to deal with it.

It is considered a poor practice to have a fallback implementation that can fail. You shuold implement your fallback such that it is not performing any logic that could fail.

__(9) Return Successful Response__

If Hystrix receives a _(6a) successful response_ it will return this response to the caller.

<a name='CircuitBreaker'/>
## Circuit Breaker

The following diagram shows how a [HystrixCommand](http://netflix.github.com/Hystrix/javadoc/index.html?com/netflix/hystrix/HystrixCommand.html) interacts with a [HystrixCircuitBreaker](http://netflix.github.com/Hystrix/javadoc/index.html?com/netflix/hystrix/HystrixCircuitBreaker.html) and its flow of logic and decision-making, including how the counters behave in the circuit breaker.

<a href="images/circuit-breaker-1280.png">[[images/circuit-breaker-640.png]]</a>
_(Click for larger view)_

<a name='Isolation'/>
## Isolation

Hystrix employs the bulkhead pattern to isolate dependencies from each other and to limit concurrent access to any one of them.

[[images/soa-5-isolation-focused-640.png]]

### Threads & Thread Pools

Clients (libraries, network calls, etc) execute on separate threads. This isolates them from the calling thread (Tomcat thread pool) so that the caller may &ldquo;walk away&rdquo; from a dependency call that is taking too long.

Hystrix uses separate, per-dependency thread pools as a way of constraining any given dependency so latency on the underlying executions will saturate the available threads only in that pool.

<a href="images/request-example-with-latency-1280.png">[[images/request-example-with-latency-640.png]]</a>
_(Click for larger view)_

It is possible for you to protect against failure without the use of thread pools, but this requires the client being trusted to fail very quickly (network connect/read timeouts and retry configuration) and always behaving well.

Netflix chose the use of threads and thread-pools to achieve isolation for many reasons including:

* Many applications execute dozens (and sometimes well over 100) different back-end service calls against dozens of different services developed by as many different teams. 
* Each service provides its own client library.
* Client libraries are changing all the time.
* Client library logic can change to add new network calls.
* Client libraries can contain logic such as retries, data parsing, caching (in-memory or across network), and other such behavior.
* Client libraries tend to be &ldquo;black boxes&rdquo; &mdash; opaque to their users about implementation details, network access patterns, configuration defaults, etc.
* In several real-world production outages the determination was &ldquo;oh, something changed and properties should be adjusted&rdquo; or &ldquo;the client library changed its behavior.&rdquo;
* Even if a client itself doesn&#8217;t change, the service itself can change, which can then impact performance characteristics which can then cause the client configuration to be invalid.
* Transitive dependencies can pull in other client libraries that are not expected and perhaps not correctly configured.
* Most network access is performed synchronously.
* Failure and latency can occur in the client-side code as well, not just in the network call.

<a href="images/isolation-options-1280.png">[[images/isolation-options-640.png]]</a>
_(Click for larger view)_

#### Benefits of Thread Pools

The benefits of isolation via threads in their own thread pools are:

* The application is fully protected from runaway client libraries. The pool for a given dependency library can fill up without impacting the rest of the application.
* The application can accept new client libraries with far lower risk. If an issue occurs, it is isolated to the library and doesn&#8217;t affect everything else.
* When a failed client becomes healthy again, the thread pool will clear up and the application immediately resumes healthy performance, as opposed to a long recovery when the entire Tomcat container is overwhelmed.
* If a client library is misconfigured, the health of a thread pool will quickly demonstrate this (via increased errors, latency, timeouts, rejections, etc.) and you can handle it (typically in real-time via dynamic properties), without affecting application functionality.
* If a client service changes performance characteristics (which happens often enough to be an issue) which in turn cause a need to tune properties (increasing/decreasing timeouts, changing retries, etc.) this again becomes visible through thread pool metrics (errors, latency, timeouts, rejections) and can be handled without impacting other clients, requests, or users.
* Beyond the isolation benefits, having dedicated thread pools provides built-in concurrency which can be leveraged to build asynchronous facades on top of synchronous client libraries (similar to how the Netflix API built a reactive, fully-asynchronous Java API on top of Hystrix commands).

In short, the isolation provided by thread pools allows for the always-changing and dynamic combination of client libraries and subsystem performance characteristics to be handled gracefully without causing outages.

**Note:** Despite the isolation a separate thread provides, your underlying client code should also have timeouts and/or respond to Thread interrupts so it can not block indefinitely and saturate the Hystrix thread pool.

#### Drawbacks of Thread Pools

The primary drawback of thread pools is that they add computational overhead. Each command execution involves the queueing, scheduling, and context switching involved in running a command on a separate thread.

Netflix, in designing this system, decided to accept the cost of this overhead in exchange for the benefits it provides and deemed it minor enough to not have major cost or performance impact.

#### Cost of Threads

Hystrix measures the latency when it executes the `run()` method on the child thread as well as the total end-to-end time on the parent thread. This way you can see the cost of Hystrix overhead (threading, metrics, logging, circuit breaker, etc.).

The Netflix API processes 10+ billion `HystrixCommand` executions per day using thread isolation. Each API instance has 40+ thread-pools with 5&ndash;20 threads in each (most are set to 10).

The following diagram represents one [HystrixCommand](http://netflix.github.com/Hystrix/javadoc/index.html?com/netflix/hystrix/HystrixCommand.html) being executed at 60 requests-per-second on a single API instance (of about 350 total threaded executions per second per server):

<a href="images/thread-cost-60rps-original.png">[[images/thread-cost-60rps-640.png]]</a>
_(Click for larger view)_

At the median (and lower) there is no cost to having a separate thread.

At the 90<sup>th</sup> percentile there is a cost of 3ms for having a separate thread.

At the 99<sup>th</sup> percentile there is a cost of 9ms for having a separate thread. Note however that the increase in cost is far smaller than the increase in execution time of the separate thread (network request) which jumped from 2 to 28 whereas the cost jumped from 0 to 9.

This overhead at the 90<sup>th</sup> percentile and higher for circuits such as these has been deemed acceptable for most Netflix use cases for the benefits of resilience achieved.

For circuits that wrap very low-latency requests (such as those that primarily hit in-memory caches) the overhead can be too high and in those cases you can use another method such as tryable semaphores which, while they do not allow for timeouts, provide most of the resilience benefits without the overhead. The overhead in general, however, is small enough that Netflix in practice usually prefers the isolation benefits of a separate thread over such techniques.

### Semaphores

You can use semaphores (or counters) to limit the number of concurrent calls to any given dependency, instead of using thread pool/queue sizes.

This allows Hystrix to shed load without using thread pools but it does not allow for timing out and walking away.

So, if you trust the client and you only want load shedding, you could use this approach.

[HystrixCommand](http://netflix.github.com/Hystrix/javadoc/index.html?com/netflix/hystrix/HystrixCommand.html) supports semaphores in 2 places:

* **Fallback:** When Hystrix retrieves fallbacks it always does so on the calling Tomcat thread.
* **Execution:** If you set the property `execution.isolation.strategy` to `SEMAPHORE` then Hystrix will use semaphores instead of threads to limit the number of concurrent parent threads that invoke the command.

You can configure both of these uses of semaphores by means of dynamic properties that define how many concurrent threads can execute. You should size them by using similar calculations as you use when sizing a threadpool (an in-memory call that returns in sub-millisecond times can perform well over 5000rps with a semaphore of only 1 or 2 &hellip; but the default is 10).

**Note:** if a dependency is isolated with a semaphore and then becomes latent, the parent threads will remain blocked until the underlying network calls timeout.

Semaphore rejection will start once the limit is hit but the threads filling the semaphore can not walk away.

<a name='RequestCollapsing'/>
## Request Collapsing

You can front a [HystrixCommand](http://netflix.github.com/Hystrix/javadoc/index.html?com/netflix/hystrix/HystrixCommand.html) with a request collapser ([HystrixCollapser](http://netflix.github.com/Hystrix/javadoc/index.html?com/netflix/hystrix/HystrixCollapser.html) is the abstract parent) with which you can collapse multiple requests into a single back-end dependency call.

The following diagram shows the number of threads and network connections in two scenarios: first without and then with request collapsing (assuming all connections are &ldquo;concurrent&rdquo; within a short time window, in this case 10ms).

<a href="images/collapser-1280.png">[[images/collapser-640.png]]</a>
_(Click for larger view)_

### Why use with request collapsing?

Use request collapsing to reduce the number of threads and network connections needed to perform concurrent [HystrixCommand](http://netflix.github.com/Hystrix/javadoc/index.html?com/netflix/hystrix/HystrixCommand.html) executions. Request collapsing does this in an automated manner that does not force all developers of a codebase to coordinate the manual batching of requests.

#### Global Context (Across all Tomcat Threads)

The ideal type of collapsing is done at the global application level, so that requests from _any user_ on any Tomcat thread can be collapsed together.

For example, if you configure a [HystrixCommand](http://netflix.github.com/Hystrix/javadoc/index.html?com/netflix/hystrix/HystrixCommand.html) to support batching for any user on requests to a dependency that retrieves movie ratings, then when any user thread in the same JVM makes such a request, Hystrix will add its request along with any others into the same collapsed network call.

#### User Request Context (Single Tomcat Thread)

If you configure a [HystrixCommand](http://netflix.github.com/Hystrix/javadoc/index.html?com/netflix/hystrix/HystrixCommand.html) to only handle batch requests for _a single user_, then Hystrix can collapse requests from within a single Tomcat thread (request).

For example, if a user wants to load bookmarks for 300 video objects, instead of executing 300 network calls, Hystrix can combine them all into one.

#### Object Modeling and Code Complexity

Sometimes when you create an object model that makes logical sense to the consumers of the object, this does not match up well with efficient resource utilization for the producers of the object.

For example, given a list of 300 video objects, iterating over them and calling `getSomeAttribute()` on each is an obvious object model, but if implemented naively can result in 300 network calls all being made within milliseconds of each other (and very likely saturating resources).

There are manual ways with which you can handle this, such as before allowing the user to call `getSomeAttribute()`, require them to declare what video objects they want to get attributes for so that they can all be pre-fetched.

Or, you could divide the object model so a user has to get a list of videos from one place, then ask for the attributes for that list of videos from somewhere else.

These approaches can lead to awkward APIs and object models that don&#8217;t match mental models and usage patterns. They can also lead to simple mistakes and inefficiencies as multiple developers work on a codebase, since an optimization done for one use case can be broken by the implementation of another use case and a new path through the code.

By pushing the collapsing logic down to the Hystrix layer, it doesn&#8217;t matter how you create the object model, in what order the calls are made, or whether different developers know about optimizations being done or even needing to be done.

The `getSomeAttribute()` method can be put wherever it fits best and be called in whatever manner suits the usage pattern and the collapser will automatically batch calls into time windows.

#### What is the cost of request collapsing?

The cost of enabling request collapsing is an increased latency before the actual command is executed. The maximum cost is the size of the batch window.

If you have a command that takes 5ms on median to execute, and a 10ms batch window, the execution time could become 15ms in a worst case. Typically a request will not happen to be submitted to the window just as it opens, and so the median penalty is half the window time, in this case 5ms.

The determination of whether this cost is worth it depends on the command being executed. A high-latency command won&#8217;t suffer as much from a small amount of additional average latency. Also, the amount of concurrency on a given command is key: There is no point in paying the penalty if there are rarely more than 1 or 2 requests to be batched together. In fact, in a single-threaded sequential iteration collapsing would be a major performance bottleneck as each iteration will wait the 10ms batch window time.

If, however, a particular command is heavily utilized concurrently and can batch dozens or even hundreds of calls together, then the cost is typically far outweighed by the increased throughput achieved as Hystrix reduces the number of threads it requires and the number of network connections to dependencies.

#### Collapser Flow

<a href="images/collapser-flow-1280.png">[[images/collapser-flow-640.png]]</a>
_(Click for larger view)_

<a name='RequestCaching'/>
## Request Caching

[HystrixCommand](http://netflix.github.com/Hystrix/javadoc/index.html?com/netflix/hystrix/HystrixCommand.html) implementations can define a cache key which is then used to de-dupe calls within a request context in a concurrent-aware manner.

Here is an example flow involving an HTTP request lifecycle and two threads doing work within that request:

<a href="images/request-cache-1280.png">[[images/request-cache-640.png]]</a>
_(Click for larger view)_

The benefits of request caching are:

* Different code paths can execute [HystrixCommands](http://netflix.github.com/Hystrix/javadoc/index.html?com/netflix/hystrix/HystrixCommand.html) without concern of duplicate work. 

This is particularly beneficial in large codebases where many developers are implementing different pieces of functionality.

For example, multiple paths through code that all need to get a user&#8217;s `Account` object can each request it like this:

```java
Account account = new UserGetAccount(accountId).execute();
``` 

The Hystrix `RequestCache` will execute the underlying `run()` method once and only once, and both threads executing the `HystrixCommand` will receive the same data despite having instantiated different instances.

* Data retrieval is consistent throughout a request.

Instead of potentially returning a different value (or fallback) each time the command is executed, the first response is cached and returned for all subsequent calls within the same request.

* Eliminates duplicate thread executions.

Since the request cache sits in front of the [run()](http://netflix.github.com/Hystrix/javadoc/index.html?com/netflix/hystrix/HystrixCommand.html#run\(\)) method invocation, Hystrix can de-dupe calls before they result in thread execution. 

If Hystrix didn&#8217;t implement the request cache functionality then each command would need to implement it themselves inside the `run` method, which would put it after a thread is queued and executed. 