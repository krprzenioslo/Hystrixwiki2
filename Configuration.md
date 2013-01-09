Hystrix uses [Archaius](https://github.com/Netflix/archaius) for the default implementation of properties for configuration.

The documentation below is for the default [HystrixPropertiesStrategy](http://netflix.github.com/Hystrix/javadoc/index.html?com/netflix/hystrix/strategy/properties/HystrixPropertiesStrategy.html) implementation that is used unless overridden using a [[plugin|Plugins]].

Each property has 4 levels of precedence:

__1) Global default from code__ 

This is the default if none of the following 3 are set.

It is shown as "_Default Value_" below.

__2) Dynamic global default property__

Default values can be changed globally using properties.

The property name is shown as "_Default Property_" below.

Example:

> _Default Property:_ hystrix.command.default.execution.isolation.thread.timeoutInMilliseconds  

__3) Instance default from code__

This allows defining an instance specific default in the code via the [HystrixCommand](http://netflix.github.com/Hystrix/javadoc/index.html?com/netflix/hystrix/HystrixCommand.html) constructor.

Sample code is shown as "_How to Set Instance Default_" below.

Example:

```java
HystrixCommandProperties.Setter()
   .withExecutionIsolationThreadTimeoutInMilliseconds(int value)
```

This would be injected into a constructor of HystrixCommand similar to this:

```java
    public HystrixCommandInstance(int id) {
        super(Setter.withGroupKey(HystrixCommandGroupKey.Factory.asKey("ExampleGroup"))
                .andCommandPropertiesDefaults(HystrixCommandProperties.Setter()
                        .withExecutionIsolationThreadTimeoutInMilliseconds(500)));
        this.id = id;
    }
```


__4) Dynamic instance property__

Instance specific values can be set dynamically and override the preceding 3 levels of defaults.

The property name is shown as "_Instance Property_" below.

Example:

> _Instance Property:_ hystrix.command.[HystrixCommandKey].execution.isolation.thread.timeoutInMilliseconds  

The [HystrixCommandKey](http://netflix.github.com/Hystrix/javadoc/index.html?com/netflix/hystrix/HystrixCommandKey.html) portion of the property would be replaced with the [HystrixCommandKey.name()](http://netflix.github.com/Hystrix/javadoc/index.html?com/netflix/hystrix/HystrixCommandKey.html#name(\)) value of whatever HystrixCommand is being targed.

If the key was "SubscriberGetAccount" then the property name would be:

> hystrix.command.SubscriberGetAccount.execution.isolation.thread.timeoutInMilliseconds



## Command Properties

[Properties](http://netflix.github.com/Hystrix/javadoc/index.html?com/netflix/hystrix/HystrixCommandProperties.html) for controlling [HystrixCommand](http://netflix.github.com/Hystrix/javadoc/index.html?com/netflix/hystrix/HystrixCommand.html) behavior:

<a name='CommandExecution'/>
### Execution

Properties that control how [HystrixCommand.run()](http://netflix.github.com/Hystrix/javadoc/index.html?com/netflix/hystrix/HystrixCommandKey.html#run(\)) is executed.

#### execution.isolation.strategy

What isolation strategy [HystrixCommand.run()](http://netflix.github.com/Hystrix/javadoc/index.html?com/netflix/hystrix/HystrixCommandKey.html#run(\)) will be executed with.

If THREAD then it will be executed on a separate thread and concurrent requests limited by the number of threads in the thread-pool.

If SEMAPHORE then it will be executed on the calling thread and concurrent requests limited by the semaphore count.

##### Thread or Semaphore

In most cases it is recommended to stick with the default which is to run commands using thread isolation.

Commands executed in threads have an extra layer of protection against latencies beyond what network timeouts can offer.

Generally the only time semaphore isolation should be used instead of thread is when the call is so high volume (hundreds per second per instance) that the overhead of separate threads is too high, and this typically only applies to non-network calls.

>Netflix API has 100+ commands running in 40+ thread pools and only a handful of commands are not running in a thread - those that fetch metadata from an in-memory cache or that are facades to thread-isolated commands (see [["Primary + Secondary with Fallback" pattern|How-To-Use#wiki-Common-Patterns]] for more information on this).

<a href="images/isolation-options-1280.png">[[images/isolation-options-640.png]]</a>
_(Click for larger view)_

See how [[isolation works|How It Works#wiki-Isolation]] for more information about this decision.

_Default Value:_ THREAD (see ExecutionIsolationStrategy.THREAD)  
_Possible Values:_ THREAD, SEMAPHORE  
_Default Property:_ hystrix.command.default.execution.isolation.strategy  
_Instance Property:_ hystrix.command.[HystrixCommandKey].execution.isolation.strategy  
_How to Set Instance Default:_  

```java
// to use thread isolation
HystrixCommandProperties.Setter()
   .withExecutionIsolationStrategy(ExecutionIsolationStrategy.THREAD)
// to use semaphore isolation
HystrixCommandProperties.Setter()
   .withExecutionIsolationStrategy(ExecutionIsolationStrategy.SEMAPHORE)
```

#### execution.isolation.thread.timeoutInMilliseconds

Time in milliseconds after which the calling thread will timeout and walk away from the [HystrixCommand.run()](http://netflix.github.com/Hystrix/javadoc/index.html?com/netflix/hystrix/HystrixCommandKey.html#run(\)) execution and mark the [HystrixCommand](http://netflix.github.com/Hystrix/javadoc/index.html?com/netflix/hystrix/HystrixCommandKey.html) as a TIMEOUT and perform fallback logic.  This applies when ExecutionIsolationStrategy.THREAD is used.

_Default Value:_ 1000  
_Default Property:_ hystrix.command.default.execution.isolation.thread.timeoutInMilliseconds  
_Instance Property:_ hystrix.command.[HystrixCommandKey].execution.isolation.thread.timeoutInMilliseconds  
_How to Set Instance Default:_  

```java
HystrixCommandProperties.Setter()
   .withExecutionIsolationThreadTimeoutInMilliseconds(int value)
```

#### execution.isolation.thread.interruptOnTimeout

Whether the thread executing [HystrixCommand.run()](http://netflix.github.com/Hystrix/javadoc/index.html?com/netflix/hystrix/HystrixCommandKey.html#run(\)) should be interrupted when timeout occurs.

_Default Value:_ true  
_Default Property:_ hystrix.command.default.execution.isolation.thread.interruptOnTimeout  
_Instance Property:_ hystrix.command.[HystrixCommandKey].execution.isolation.thread.interruptOnTimeout  
_How to Set Instance Default:_  

```java
HystrixCommandProperties.Setter()
   .withExecutionIsolationThreadInterruptOnTimeout(boolean value)
```

#### execution.isolation.semaphore.maxConcurrentRequests

Max number of requests allowed to a [HystrixCommand.run()](http://netflix.github.com/Hystrix/javadoc/index.html?com/netflix/hystrix/HystrixCommandKey.html#run(\)) method when ExecutionIsolationStrategy.SEMAPHORE is used.

If the max concurrent limit is hit then subsequent requests will be rejected.

The logic for sizing a semaphore is basically the same as choosing how many threads in a thread-pool but the overhead for a semaphore is far smaller and typically the executions are far faster (sub-millisecond) otherwise threads would be used.

> For example, 5000rps on a single instance for in-memory lookups with metrics being gathered has been seen to work with a semaphore of only 2.

The isolation principle is still the same so the semaphore should still be a small percentage of the overall container (ie Tomcat) threadpool, not all of or most of it, otherwise it provides no protection.

_Default Value:_ 10  
_Default Property:_ hystrix.command.default.execution.isolation.semaphore.maxConcurrentRequests  
_Instance Property:_ hystrix.command.[HystrixCommandKey].execution.isolation.semaphore.maxConcurrentRequests  
_How to Set Instance Default:_  

```java
HystrixCommandProperties.Setter()
   .withExecutionIsolationSemaphoreMaxConcurrentRequests(int value)
```

#### fallback.isolation.semaphore.maxConcurrentRequests

Max number of requests allows to a [HystrixCommand.getFallback()](http://netflix.github.com/Hystrix/javadoc/index.html?com/netflix/hystrix/HystrixCommandKey.html#getFallback(\)) method from the calling thread.

If the max concurrent limit is hit then subsequent requests will be rejected and an exception thrown since no fallback could be retrieved.

_Default Value:_ 10  
_Default Property:_ hystrix.command.default.fallback.isolation.semaphore.maxConcurrentRequests  
_Instance Property:_ hystrix.command.[HystrixCommandKey].fallback.isolation.semaphore.maxConcurrentRequests  
_How to Set Instance Default:_  

```java
HystrixCommandProperties.Setter()
   .withFallbackIsolationSemaphoreMaxConcurrentRequests(int value)
```

<a name='CommandCircuitBreaker'/>
### Circuit Breaker

The circuit breaker [properties](http://netflix.github.com/Hystrix/javadoc/index.html?com/netflix/hystrix/HystrixCommandProperties.html) control behavior of the [HystrixCircuitBreaker](http://netflix.github.com/Hystrix/javadoc/index.html?com/netflix/hystrix/HystrixCircuitBreaker.html).

#### circuitBreaker.enabled

Whether a circuit breaker will be used to track health and short-circuit requests if it trips.

_Default Value:_ true   
_Default Property:_ hystrix.command.default.circuitBreaker.enabled  
_Instance Property:_ hystrix.command.[HystrixCommandKey].circuitBreaker.enabled  
_How to Set Instance Default:_  

```java
HystrixCommandProperties.Setter()
   .withCircuitBreakerEnabled(boolean value)
```

#### circuitBreaker.requestVolumeThreshold

Minimum number of requests in rolling window needed before tripping the circuit will occur.

For example, if the value is 20, then if only 19 requests are received in the rolling window (say 10 seconds) the circuit will not trip open even if all 19 failed.

_Default Value:_ 20  
_Default Property:_ hystrix.command.default.circuitBreaker.requestVolumeThreshold  
_Instance Property:_ hystrix.command.[HystrixCommandKey].circuitBreaker.requestVolumeThreshold  
_How to Set Instance Default:_  

```java
HystrixCommandProperties.Setter()
   .withCircuitBreakerRequestVolumeThreshold(int value)
```

#### circuitBreaker.sleepWindowInMilliseconds

After tripping the circuit how long to reject requests before allowing attempts again to determine if the circuit should be closed.

_Default Value:_ 5000  
_Default Property:_ hystrix.command.default.circuitBreaker.sleepWindowInMilliseconds  
_Instance Property:_ hystrix.command.[HystrixCommandKey].circuitBreaker.sleepWindowInMilliseconds  
_How to Set Instance Default:_  

```java
HystrixCommandProperties.Setter()
   .withCircuitBreakerSleepWindowInMilliseconds(int value)
```

#### circuitBreaker.errorThresholdPercentage

Error percentage at which the circuit should trip open and start short-circuiting requests to fallback logic.

_Default Value:_ 50  
_Default Property:_ hystrix.command.default.circuitBreaker.errorThresholdPercentage  
_Instance Property:_ hystrix.command.[HystrixCommandKey].circuitBreaker.errorThresholdPercentage  
_How to Set Instance Default:_  

```java
HystrixCommandProperties.Setter()
   .withCircuitBreakerErrorThresholdPercentage(int value)
```

#### circuitBreaker.forceOpen

If true the circuit breaker will be forced open (tripped) and reject all requests.

This property takes precedence over _circuitBreaker.forceClosed_.

_Default Value:_ false  
_Default Property:_ hystrix.command.default.circuitBreaker.forceOpen  
_Instance Property:_ hystrix.command.[HystrixCommandKey].circuitBreaker.forceOpen  
_How to Set Instance Default:_  

```java
HystrixCommandProperties.Setter()
   .withCircuitBreakerForceOpen(boolean value)
```

#### circuitBreaker.forceClosed

If true the circuit breaker will remain closed and allow requests regardless of the error percentage.

The _circuitBreaker.forceOpen_ property takes precedence so if it set to true this property does nothing.

_Default Value:_ false  
_Default Property:_ hystrix.command.default.circuitBreaker.forceClosed  
_Instance Property:_ hystrix.command.[HystrixCommandKey].circuitBreaker.forceClosed  
_How to Set Instance Default:_  

```java
HystrixCommandProperties.Setter()
   .withCircuitBreakerForceClosed(boolean value)
```

<a name='CommandMetrics'/>
### Metrics

[Properties](http://netflix.github.com/Hystrix/javadoc/index.html?com/netflix/hystrix/HystrixCommandProperties.html) related to capturing metrics from [HystrixCommand](http://netflix.github.com/Hystrix/javadoc/index.html?com/netflix/hystrix/HystrixCommand.html) execution.

#### metrics.rollingStats.timeInMilliseconds

Duration of statistical rolling window in milliseconds. This is how long metrics are kept for the circuit breaker to use and for publishing.

The window is broken into buckets and "roll" by those increments.

For example, if set at 10 seconds (10000) with 10 1-second buckets, this following diagram represents how it rolls new buckets on and old ones off:

[[images/rolling-stats-640.png]]

_Default Value:_ 10000  
_Default Property:_ hystrix.command.default.  
_Instance Property:_ hystrix.command.[HystrixCommandKey].  
_How to Set Instance Default:_  

```java
HystrixCommandProperties.Setter()
   .withMetricsRollingStatisticalWindowInMilliseconds(int value)
```

#### metrics.rollingStats.numBuckets

Number of buckets the rolling statistical window is broken into.

Note: The following must be true "metrics.rollingStats.timeInMilliseconds % metrics.rollingStats.numBuckets == 0" otherwise it will throw an exception.

In other words, 10000/10 is okay, so is 10000/20 but 10000/7 is not.

_Default Value:_ 10  
_Possible Values:_ Any value that metrics.rollingStats.timeInMilliseconds can be evenly divided by. The result however should be buckets measuring 100s or 1000s of milliseconds. Performance at  high volume has not been tested with buckets <100ms.  
_Default Property:_ hystrix.command.default.metrics.rollingStats.numBuckets  
_Instance Property:_ hystrix.command.[HystrixCommandKey].metrics.rollingStats.numBuckets  
_How to Set Instance Default:_  

```java
HystrixCommandProperties.Setter()
   .withMetricsRollingStatisticalWindowBuckets(int value)
```

#### metrics.rollingPercentile.enabled

Whether execution latencies should be tracked and calculated as percentiles.

_Default Value:_ true  
_Default Property:_ hystrix.command.default.metrics.rollingPercentile.enabled  
_Instance Property:_ hystrix.command.[HystrixCommandKey].metrics.rollingPercentile.enabled  
_How to Set Instance Default:_  

```java
HystrixCommandProperties.Setter()
   .withMetricsRollingPercentileEnabled(boolean value)
```

#### metrics.rollingPercentile.timeInMilliseconds

Duration of rolling window in milliseconds that execution times are kept for to allow percentile calculations.

The window is broken into buckets and "roll" by those increments. 

_Default Value:_ 60000  
_Default Property:_ hystrix.command.default.metrics.rollingPercentile.timeInMilliseconds  
_Instance Property:_ hystrix.command.[HystrixCommandKey].metrics.rollingPercentile.timeInMilliseconds  
_How to Set Instance Default:_  

```java
HystrixCommandProperties.Setter()
   .withMetricsRollingPercentileWindowInMilliseconds(int value)
```

#### metrics.rollingPercentile.numBuckets

Number of buckets the rollingPercentile window will be broken into.

Note: The following must be true "metrics.rollingPercentile.timeInMilliseconds % metrics.rollingPercentile.numBuckets == 0" otherwise it will throw an exception.

In other words, 60000/6 is okay, so is 60000/60 but 10000/7 is not.

_Default Value:_ 6  
_Possible Values:_ Any value that metrics.rollingPercentile.timeInMilliseconds can be evenly divided by. The result however should be buckets measuring 1000s of milliseconds. Performance at high volume has not been tested with buckets <1000ms.  
_Default Property:_ hystrix.command.default.metrics.rollingPercentile.numBuckets  
_Instance Property:_ hystrix.command.[HystrixCommandKey].metrics.rollingPercentile.numBuckets  
_How to Set Instance Default:_  

```java
HystrixCommandProperties.Setter()
   .withMetricsRollingPercentileWindowBuckets(int value)
```

#### metrics.rollingPercentile.bucketSize

Max number of execution times are kept per bucket. If more executions occur during the time they will loop and start over-writing at the beginning of the bucket. 

For example, if bucket size is set to 100 and represents 10 seconds and 500 executions occur during this time only the last 100 executions will be kept for that 10 second bucket.

Increasing this size increases the amount of memory used to store values and increases time for sorting the lists to do percentile calculations.

_Default Value:_ 100   
_Default Property:_ hystrix.command.default.  
_Instance Property:_ hystrix.command.[HystrixCommandKey].  
_How to Set Instance Default:_  

```java
HystrixCommandProperties.Setter()
   .withMetricsRollingPercentileBucketSize(int value)
```

#### metrics.healthSnapshot.intervalInMilliseconds

Time in milliseconds to wait between allowing health snapshots to be taken that calculate success and error percentages and affect circuit breaker status.

On high-volume circuits the continual calculation of error percentage can become CPU intensive thus this controls how often it is calculated.

_Default Value:_ 500  
_Default Property:_ hystrix.command.default.metrics.healthSnapshot.intervalInMilliseconds  
_Instance Property:_ hystrix.command.[HystrixCommandKey].metrics.healthSnapshot.intervalInMilliseconds  
_How to Set Instance Default:_  

```java
HystrixCommandProperties.Setter()
   .withMetricsHealthSnapshotIntervalInMilliseconds(int value)
```

<a name='CommandRequestContext'/>
### Request Context

[Properties](http://netflix.github.com/Hystrix/javadoc/index.html?com/netflix/hystrix/HystrixCommandProperties.html) related to [HystrixRequestContext](http://netflix.github.com/Hystrix/javadoc/index.html?com/netflix/hystrix/strategy/concurrency/HystrixRequestContext.html) functionality used by [HystrixCommand](http://netflix.github.com/Hystrix/javadoc/index.html?com/netflix/hystrix/HystrixCommand.html).

#### requestCache.enabled

Whether [HystrixCommand.getCacheKey()](http://netflix.github.com/Hystrix/javadoc/index.html?com/netflix/hystrix/HystrixCommand.html#getCacheKey(\)) should be used with [HystrixRequestCache](http://netflix.github.com/Hystrix/javadoc/index.html?com/netflix/hystrix/HystrixRequestCAche.html) to provide de-duplication functionality via request-scoped caching. 

_Default Value:_ true  
_Default Property:_ hystrix.command.default.requestCache.enabled  
_Instance Property:_ hystrix.command.[HystrixCommandKey].requestCache.enabled  
_How to Set Instance Default:_  

```java
HystrixCommandProperties.Setter()
   .withRequestCacheEnabled(boolean value)
```

#### requestLog.enabled

Whether [HystrixCommand](http://netflix.github.com/Hystrix/javadoc/index.html?com/netflix/hystrix/HystrixCommand.html) execution and events should be logged to [HystrixRequestLog](http://netflix.github.com/Hystrix/javadoc/index.html?com/netflix/hystrix/HystrixRequestLog.html). 

_Default Value:_ true  
_Default Property:_ hystrix.command.default.requestLog.enabled  
_Instance Property:_ hystrix.command.[HystrixCommandKey].requestLog.enabled  
_How to Set Instance Default:_  

```java
HystrixCommandProperties.Setter()
   .withRequestLogEnabled(boolean value)
```

<a name='Collapser'/>
## Collapser Properties

[Properties](http://netflix.github.com/Hystrix/javadoc/index.html?com/netflix/hystrix/HystrixCollapserProperties.html) for controlling [HystrixCollapser](http://netflix.github.com/Hystrix/javadoc/index.html?com/netflix/hystrix/HystrixCollapser.html) behavior.

#### maxRequestsInBatch

The maximum number of requests allowed in a batch before triggering a batch execution.

_Default Value:_ Integer.MAX_VALUE  
_Default Property:_ hystrix.collapser.default.maxRequestsInBatch  
_Instance Property:_ hystrix.collapser.[HystrixCollapserKey].maxRequestsInBatch  
_How to Set Instance Default:_  

```java
HystrixCollapserProperties.Setter()
   .withMaxRequestsInBatch(int value)
```

#### timerDelayInMilliseconds

Number of items in batch that will trigger a batch execution before _timerDelayInMilliseconds_ triggers the batch.

_Default Value:_ 10  
_Default Property:_ hystrix.collapser.default.timerDelayInMilliseconds  
_Instance Property:_ hystrix.collapser.[HystrixCollapserKey].timerDelayInMilliseconds  
_How to Set Instance Default:_  

```java
HystrixCollapserProperties.Setter()
   .withTimerDelayInMilliseconds(int value)
```

#### requestCache.enabled

Whether request caching is enabled for [HystrixCollapser.execute()](http://netflix.github.com/Hystrix/javadoc/index.html?com/netflix/hystrix/HystrixCollapser.html#execute(\)) and [HystrixCollapser.queue()](http://netflix.github.com/Hystrix/javadoc/index.html?com/netflix/hystrix/HystrixCollapser.html#queue(\)) invocations.

_Default Value:_ true  
_Default Property:_ hystrix.collapser.default.requestCache.enabled  
_Instance Property:_ hystrix.collapser.[HystrixCollapserKey].requestCache.enabled  
_How to Set Instance Default:_  

```java
HystrixCollapserProperties.Setter()
   .withRequestCacheEnabled(boolean value)
```

<a name='ThreadPool'/>
## ThreadPool Properties

[Properties](http://netflix.github.com/Hystrix/javadoc/index.html?com/netflix/hystrix/HystrixThreadPoolProperties.html) for controlling thread-pool behavior that [HystrixCommands](http://netflix.github.com/Hystrix/javadoc/index.html?com/netflix/hystrix/HystrixCommand.html) execute on.

Most of the time the default value of 10 threads will be fine (often it could be made smaller).

To determine if it needs to be larger, a basic formula for calculating the size is:

_requests per second at peak when healthy * 99th percentile latency in seconds + some breathing room_

See the example below to see this formula put into practice.

The general principle is keep the pool as small as possible as it is the primary tool to shed load and prevent resources from becoming blocked if latency occurs.

> Netflix API has 30+ of its threadpools set at 10, 2 at 20 and 1 at 25.

<a href="images/thread-configuration-1280.png">[[images/thread-configuration-640.png]]</a>
_(Click for larger view)_

The above diagram shows an example configuration where the dependency has no reason to hit the 99.5th percentile and thus cuts it short at the network timeout layer and immediately retries with the expectation to get median latency most of the time, and accomplish this all within the 300ms thread timeout.

If the dependency has legitimate reasons to sometimes hit the 99.5th percentile (i.e. cache miss with lazy generation) then the network timeout will be set higher than it, such as at 325ms with 0 or 1 retries and the thread timeout set higher (350ms+).

The threadpool is sized at 10 to handle a burst of 99th percentile requests, but when everything is healthy this threadpool will typically only have 1 or 2 threads active at any given time to serve mostly 40ms median calls.

When configured correctly a timeout at the HystrixCommand layer should be rare, but the protection is there in case something other than network latency affects the time, or the combination of connect+read+retry+connect+read in a worst case scenario still exceeds the configured overall timeout.

The aggressiveness of configurations and tradeoffs in each direction are different for each dependency.

Configurations can be changed in realtime as needed as performance characteristics change or when problems are found all without risking the taking down of the entire app if problems or misconfigurations occur.

#### coreSize

Core thread-pool size. This is the maximum number of concurrent [HystrixCommands](http://netflix.github.com/Hystrix/javadoc/index.html?com/netflix/hystrix/HystrixCommand.html) that can be executed.

_Default Value:_ 10  
_Default Property:_ hystrix.threadpool.default.coreSize  
_Instance Property:_ hystrix.threadpool.[HystrixThreadPoolKey].coreSize  
_How to Set Instance Default:_  

```java
HystrixThreadPoolProperties.Setter()
   .withCoreSize(int value)
```

#### maxQueueSize

Max queue size of BlockingQueue implementation.

If set to -1 then [SynchronousQueue](http://docs.oracle.com/javase/6/docs/api/java/util/concurrent/SynchronousQueue.html) will be used, otherwise a positive value will be used with [LinkedBlockingQueue](http://docs.oracle.com/javase/6/docs/api/java/util/concurrent/LinkedBlockingQueue.html).

NOTE: This property only applies at initialization time since queue implementations can't be resized or changed without re-initializing the thread executor which is not supported.

To overcome this and allow dynamic changes in queue see the _queueSizeRejectionThreshold_ property. 

Changing between SynchronousQueue and LinkedBlockingQueue requires a restart.

_Default Value:_ -1  
_Default Property:_ hystrix.threadpool.default.maxQueueSize  
_Instance Property:_ hystrix.threadpool.[HystrixThreadPoolKey].maxQueueSize  
_How to Set Instance Default:_  

```java
HystrixThreadPoolProperties.Setter()
   .withMaxQueueSize(int value)
```

#### queueSizeRejectionThreshold

Queue size rejection threshold is an artificial "max" size at which rejections will occur even if _maxQueueSize_ has not been reached. This is done because the _maxQueueSize_ of a [BlockingQueue](http://docs.oracle.com/javase/6/docs/api/java/util/concurrent/BlockingQueue.html) can not be dynamically changed and we want to support dynamically changing the queue size that affects rejections.

This is used by HystrixCommand when queuing a thread for execution.

NOTE: This property is not applicable if maxQueueSize == -1.

_Default Value:_ 5  
_Default Property:_ hystrix.threadpool.default.queueSizeRejectionThreshold  
_Instance Property:_ hystrix.threadpool.[HystrixThreadPoolKey].queueSizeRejectionThreshold  
_How to Set Instance Default:_  

```java
HystrixThreadPoolProperties.Setter()
   .withQueueSizeRejectionThreshold(int value)
```

#### keepAliveTimeMinutes

Keep-alive time in minutes.

This is in practice not used since the corePoolSize and maxPoolSize are set to the same value in the default implementation, but if a custom implementation were used via [[plugin|Plugins]] then this would be available to use.

_Default Value:_ 1  
_Default Property:_ hystrix.threadpool.default.keepAliveTimeMinutes  
_Instance Property:_ hystrix.threadpool.[HystrixThreadPoolKey].keepAliveTimeMinutes  
_How to Set Instance Default:_  

```java
HystrixThreadPoolProperties.Setter()
   .withKeepAliveTimeMinutes(int value)
```

#### metrics.rollingStats.timeInMilliseconds

Duration of statistical rolling window in milliseconds. This is how long metrics are kept for the thread pool.

The window is broken into buckets and "roll" by those increments.

_Default Value:_ 10000  
_Default Property:_ hystrix.threadpool.default.metrics.rollingStats.timeInMilliseconds  
_Instance Property:_ hystrix.threadpool.[HystrixThreadPoolKey].metrics.rollingStats.timeInMilliseconds  
_How to Set Instance Default:_  

```java
HystrixThreadPoolProperties.Setter()
   .withMetricsRollingStatisticalWindowInMilliseconds(int value)
```

#### metrics.rollingStats.numBuckets

Number of buckets the rolling statistical window is broken into.

Note: The following must be true "metrics.rollingStats.timeInMilliseconds % metrics.rollingStats.numBuckets == 0" otherwise it will throw an exception.

In other words, 10000/10 is okay, so is 10000/20 but 10000/7 is not.

_Default Value:_ 10  
_Possible Values:_ Any value that metrics.rollingStats.timeInMilliseconds can be evenly divided by. The result however should be buckets measuring 100s or 1000s of milliseconds. Performance at  high volume has not been tested with buckets <100ms.  
_Default Property:_ hystrix.threadpool.default.metrics.rollingPercentile.numBuckets  
_Instance Property:_ hystrix.threadpool.[HystrixThreadPoolProperties].metrics.rollingPercentile.numBuckets  
_How to Set Instance Default:_  

```java
HystrixThreadPoolProperties.Setter()
   .withMetricsRollingStatisticalWindowBuckets(int value)
```