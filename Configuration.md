## Contents
1. <a href="#intro">Introduction</a>
1. <a href="#CommandProperties">Command Properties</a>
  1. <a href="#CommandExecution">Execution</a>
    1. <a href="#execution.isolation.strategy"><tt>execution.isolation.strategy</tt></a>
    1. <a href="#execution.isolation.thread.timeoutInMilliseconds"><tt>execution.isolation.thread.timeoutInMilliseconds</a>
    1. <a href="#execution.isolation.thread.interruptOnTimeout"><tt>execution.isolation.thread.interruptOnTimeout</tt></a>
    1. <a href="#execution.isolation.semaphore.maxConcurrentRequests"><tt>execution.isolation.semaphore.maxConcurrentRequests</tt></a>
  1. <a href="#CommandFallback">Fallback</a>
    1. <a href="#fallback.isolation.semaphore.maxConcurrentRequests"><tt>fallback.isolation.semaphore.maxConcurrentRequests</tt></a>
    1. <a href="#fallback.enabled"><tt>fallback.enabled</tt></a>
  1. <a href="#CommandCircuitBreaker">Circuit Breaker</a>
    1. <a href="#circuitBreaker.enabled"><tt>circuitBreaker.enabled</tt></a>
    1. <a href="#circuitBreaker.requestVolumeThreshold"><tt>circuitBreaker.requestVolumeThreshold</tt></a>
    1. <a href="#circuitBreaker.sleepWindowInMilliseconds"><tt>circuitBreaker.sleepWindowInMilliseconds</tt></a>
    1. <a href="#circuitBreaker.errorThresholdPercentage"><tt>circuitBreaker.errorThresholdPercentage</tt></a>
    1. <a href="#circuitBreaker.forceOpen"><tt>circuitBreaker.forceOpen</tt></a>
    1. <a href="#circuitBreaker.forceClosed"><tt>circuitBreaker.forceClosed</tt></a>
  1. <a href="#CommandMetrics">Metrics</a>
    1. <a href="#metrics.rollingStats.timeInMilliseconds"><tt>metrics.rollingStats.timeInMilliseconds</tt></a>
    1. <a href="#metrics.rollingStats.numBuckets"><tt>metrics.rollingStats.numBuckets</tt></a>
    1. <a href="#metrics.rollingPercentile.enabled"><tt>metrics.rollingPercentile.enabled</tt></a>
    1. <a href="#metrics.rollingPercentile.timeInMilliseconds"><tt>metrics.rollingPercentile.timeInMilliseconds</tt></a>
    1. <a href="#metrics.rollingPercentile.numBuckets"><tt>metrics.rollingPercentile.numBuckets</tt></a>
    1. <a href="#metrics.rollingPercentile.bucketSize"><tt>metrics.rollingPercentile.bucketSize</tt></a>
    1. <a href="#metrics.healthSnapshot.intervalInMilliseconds"><tt>metrics.healthSnapshot.intervalInMilliseconds</tt></a>
  1. <a href="#CommandRequestContext">Request Context</a>
    1. <a href="#requestCache.enabled"><tt>requestCache.enabled</tt></a>
    1. <a href="#requestLog.enabled"><tt>requestLog.enabled</tt></a>
1. <a href="#Collapser">Collapser Properties</a>
    1. <a href="#maxRequestsInBatch"><tt>maxRequestsInBatch</tt></a>
    1. <a href="#timerDelayInMilliseconds"><tt>timerDelayInMilliseconds</tt></a>
    1. <a href="#collapser.requestCache.enabled"><tt>requestCache.enabled</tt></a>
1. <a href="#ThreadPool">Thread Pool Properties</a>
    1. <a href="#coreSize"><tt>coreSize</tt></a>
    1. <a href="#maxQueueSize"><tt>maxQueueSize</tt></a>
    1. <a href="#queueSizeRejectionThreshold"><tt>queueSizeRejectionThreshold</tt></a>
    1. <a href="#keepAliveTimeMinutes"><tt>keepAliveTimeMinutes</tt></a>
    1. <a href="#threadpool.metrics.rollingStats.timeInMilliseconds"><tt>metrics.rollingStats.timeInMilliseconds</tt></a>
    1. <a href="#threadpool.metrics.rollingStats.numBuckets"><tt>metrics.rollingStats.numBuckets</tt></a>

<a name="intro" />
## Introduction

Hystrix uses [Archaius](https://github.com/Netflix/archaius) for the default implementation of properties for configuration.

The documentation below describes the default [`HystrixPropertiesStrategy`](http://netflix.github.com/Hystrix/javadoc/index.html?com/netflix/hystrix/strategy/properties/HystrixPropertiesStrategy.html) implementation that is used unless you override it by using a [[plugin|Plugins]].

Each property has four levels of precedence:

__1. Global default from code__ 

This is the default if none of the following 3 are set.

The global default is shown as &ldquo;**Default Value**&rdquo; in the tables below.

__2. Dynamic global default property__

You can change a global default value by using properties.

The global default property name is shown as &ldquo;**Default Property**&rdquo; in the tables below.

Example:

```java
HystrixCommandProperties.Setter()
   .withExecutionIsolationThreadTimeoutInMilliseconds(int value)
```

You would insert a command of this sort into a `HystrixCommand` constructor in a manner similar to this:

```java
public HystrixCommandInstance(int id) {
    super(Setter.withGroupKey(HystrixCommandGroupKey.Factory.asKey("ExampleGroup"))
            .andCommandPropertiesDefaults(HystrixCommandProperties.Setter()
                   .withExecutionIsolationThreadTimeoutInMilliseconds(500)));
    this.id = id;
}
```

__4. Dynamic instance property__

You can set instance-specific values dynamically which override the preceding three levels of defaults.

The dynamic instance property name is shown as &ldquo;**Instance Property**&rdquo; in the tables below.

Example:

<table><tr><th>Instance Property</th><td><tt>hystrix.command.<i>HystrixCommandKey</i>.execution.isolation.thread.timeoutInMilliseconds</tt></td></tr></table>

Replace the [_`HystrixCommandKey`_](http://netflix.github.com/Hystrix/javadoc/index.html?com/netflix/hystrix/HystrixCommandKey.html) portion of the property with the [`HystrixCommandKey.name()`](http://netflix.github.com/Hystrix/javadoc/index.html?com/netflix/hystrix/HystrixCommandKey.html#name\(\)) value of whichever `HystrixCommand` you are targeting.

For example, if the key was named &ldquo;`SubscriberGetAccount`&rdquo; then the property name would be:

> `hystrix.command.SubscriberGetAccount.execution.isolation.thread.timeoutInMilliseconds`

<a name="CommandProperties" />
## Command Properties

The following [Properties](http://netflix.github.com/Hystrix/javadoc/index.html?com/netflix/hystrix/HystrixCommandProperties.html) control [`HystrixCommand`](http://netflix.github.com/Hystrix/javadoc/index.html?com/netflix/hystrix/HystrixCommand.html) behavior:

<a name='CommandExecution'/>
### Execution

The following Properties control how [`HystrixCommand.run()`](http://netflix.github.com/Hystrix/javadoc/index.html?com/netflix/hystrix/HystrixCommand.html#run\(\)) executes.

<a name="execution.isolation.strategy" />
#### execution.isolation.strategy

This property indicates which isolation strategy `HystrixCommand.run()` executes with, one of the following two choices:

* `THREAD` &mdash; it executes on a separate thread and concurrent requests are limited by the number of threads in the thread-pool
* `SEMAPHORE` &mdash; it executes on the calling thread and concurrent requests are limited by the semaphore count

##### Thread or Semaphore

The default, and the recommended setting, is to run commands using thread isolation (`THREAD`).

Commands executed in threads have an extra layer of protection against latencies beyond what network timeouts can offer.

Generally the only time you should use semaphore isolation (`SEMAPHORE`) is when the call is so high volume (hundreds per second, per instance) that the overhead of separate threads is too high; this typically only applies to non-network calls.

>Netflix API has 100+ commands running in 40+ thread pools and only a handful of those commands are not running in a thread - those that fetch metadata from an in-memory cache or that are fa&ccedil;ades to thread-isolated commands (see [&ldquo;Primary + Secondary with Fallback&rdquo; pattern](https://github.com/Netflix/Hystrix/wiki/How-To-Use#common-patterns) for more information on this).

<a href="images/isolation-options-1280.png">[[images/isolation-options-640.png]]</a>
_(Click for larger view)_

See [[how isolation works|How-it-Works#isolation]] for more information about this decision.

<table> <tbody>
 <tr><th>Default Value</th><td><tt>THREAD</tt> (see <tt>ExecutionIsolationStrategy.THREAD</tt>)</td></tr>
 <tr><th>Possible Values</th><td><tt>THREAD</tt>, <tt>SEMAPHORE</tt></td></tr>
 <tr><th>Default Property</th><td><tt>hystrix.command.default.execution.isolation.strategy</tt></td></tr>
 <tr><th>Instance Property</th><td><tt>hystrix.command.<i>HystrixCommandKey</i>.execution.isolation.strategy</tt></td></tr>
 <tr><th>How to Set Instance Default:</th><td><pre>// to use thread isolation
HystrixCommandProperties.Setter()
   .withExecutionIsolationStrategy(ExecutionIsolationStrategy.THREAD)
// to use semaphore isolation
HystrixCommandProperties.Setter()
   .withExecutionIsolationStrategy(ExecutionIsolationStrategy.SEMAPHORE)</pre></td></tr>
</tbody></table>

<a name="execution.isolation.thread.timeoutInMilliseconds" />
#### execution.isolation.thread.timeoutInMilliseconds

This property sets the time in milliseconds after which the calling thread will timeout, walk away from the `HystrixCommand.run()` execution, mark the `HystrixCommand` as a TIMEOUT, and perform fallback logic. This property only applies when you use  `ExecutionIsolationStrategy.THREAD`.

<table><tbody>
 <tr><th>Default Value</th><td><tt>1000</tt></td></tr>
 <tr><th>Default Property</th><td><tt>hystrix.command.default.execution.isolation.thread.timeoutInMilliseconds</tt></td></tr>
 <tr><th>Instance Property</th><td><tt>hystrix.command.<i>HystrixCommandKey</i>.execution.isolation.thread.timeoutInMilliseconds</tt></td></tr>
 <tr><th>How to Set Instance Default</th><td><pre>HystrixCommandProperties.Setter()
   .withExecutionIsolationThreadTimeoutInMilliseconds(int value)</pre></td></tr>
</tbody></table>

<a name="execution.isolation.thread.interruptOnTimeout" />
#### execution.isolation.thread.interruptOnTimeout

This property indicates whether the thread executing [HystrixCommand.run()](http://netflix.github.com/Hystrix/javadoc/index.html?com/netflix/hystrix/HystrixCommand.html#run\(\)) should be interrupted when a timeout occurs.

<table><tbody>
 <tr><th>Default Value</th><td><tt>true</tt></td></tr>
 <tr><th>Default Property</th><td><tt>hystrix.command.default.execution.isolation.thread.interruptOnTimeout</tt></td></tr>
 <tr><th>Instance Property</th><td><tt>hystrix.command.<i>HystrixCommandKey</i>.execution.isolation.thread.interruptOnTimeout</tt></td></tr>
 <tr><th>How to Set Instance Default</th><td><pre>HystrixCommandProperties.Setter()
   .withExecutionIsolationThreadInterruptOnTimeout(boolean value)</pre></td></tr>
</tbody></table>

<a name="execution.isolation.semaphore.maxConcurrentRequests" />
#### execution.isolation.semaphore.maxConcurrentRequests

This property sets the maximum number of requests allowed to a `HystrixCommand.run()` method when you are using `ExecutionIsolationStrategy.SEMAPHORE`.

If this maximum concurrent limit is hit then subsequent requests will be rejected.

The logic that you use when you size a semaphore is basically the same as when you choose how many threads to add to a thread-pool, but the overhead for a semaphore is far smaller and typically the executions are far faster (sub-millisecond), otherwise you would be using threads.

> For example, 5000rps on a single instance for in-memory lookups with metrics being gathered has been seen to work with a semaphore of only 2.

The isolation principle is still the same so the semaphore should still be a small percentage of the overall container (i.e. Tomcat) thread pool, not all of or most of it, otherwise it provides no protection.

<table><tbody>
 <tr><th>Default Value</th><td><tt>10</tt></td></tr>
 <tr><th>Default Property</th><td><tt>hystrix.command.default.execution.isolation.semaphore.maxConcurrentRequests</tt></td></tr>
 <tr><th>Instance Property</th><td><tt>hystrix.command.<i>HystrixCommandKey</i>.execution.isolation.semaphore.maxConcurrentRequests</tt></td></tr>
 <tr><th>How to Set Instance Default</th><td><pre>HystrixCommandProperties.Setter()
   .withExecutionIsolationSemaphoreMaxConcurrentRequests(int value)</pre></td></tr>
</tbody></table>

<a name='CommandFallback'/>
### Fallback

The following properties control how [`HystrixCommand.getFallback()`](http://netflix.github.com/Hystrix/javadoc/index.html?com/netflix/hystrix/HystrixCommand.html#getFallback\(\)) executes. These properties apply to both `ExecutionIsolationStrategy.THREAD` and `ExecutionIsolationStrategy.SEMAPHORE`.

<a name="fallback.isolation.semaphore.maxConcurrentRequests" />
#### fallback.isolation.semaphore.maxConcurrentRequests

This property sets the maximum number of requests a `HystrixCommand.getFallback()` method is allowed to make from the calling thread.

If the maximum concurrent limit is hit then subsequent requests will be rejected and an exception thrown since no fallback could be retrieved.

<table><tbody>
 <tr><th>Default Value</th><td><tt>10</tt></td></tr>
 <tr><th>Default Property</th><td><tt>hystrix.command.default.fallback.isolation.semaphore.maxConcurrentRequests</tt></td></tr>
 <tr><th>Instance Property</th><td><tt>hystrix.command.<i>HystrixCommandKey</i>.fallback.isolation.semaphore.maxConcurrentRequests</tt></td></tr>
 <tr><th>How to Set Instance Default</th><td><pre>HystrixCommandProperties.Setter()
   .withFallbackIsolationSemaphoreMaxConcurrentRequests(int value)</pre></td></tr>
</tbody></table>

<a name="fallback.enabled" />
#### fallback.enabled

Since: 1.2

This property determines whether a call to  [HystrixCommand.getFallback()](http://netflix.github.com/Hystrix/javadoc/index.html?com/netflix/hystrix/HystrixCommand.html#getFallback\(\)) will be attempted when failure or rejection occurs.

<table><tbody>
 <tr><th>Default Value</th><td><tt>true</tt></td></tr>
 <tr><th>Default Property</th><td><tt>hystrix.command.default.fallback.enabled</tt></td></tr>
 <tr><th>Instance Property</th><td><tt>hystrix.command.<i>HystrixCommandKey</i>.fallback.enabled</tt></td></tr>
 <tr><th>How to Set Instance Default</th><td><pre>HystrixCommandProperties.Setter()
   .withFallbackEnabled(boolean value)</pre></td></tr>
</tbody></table>

<a name="CommandCircuitBreaker" />
### Circuit Breaker

The circuit breaker [properties](http://netflix.github.com/Hystrix/javadoc/index.html?com/netflix/hystrix/HystrixCommandProperties.html) control behavior of the [HystrixCircuitBreaker](http://netflix.github.com/Hystrix/javadoc/index.html?com/netflix/hystrix/HystrixCircuitBreaker.html).

<a name="circuitBreaker.enabled" />
#### circuitBreaker.enabled

This property determines whether a circuit breaker will be used to track health and to short-circuit requests if it trips.

<table><tbody>
 <tr><th>Default Value</th><td><tt>true</tt></td></tr>
 <tr><th>Default Property</th><td><tt>hystrix.command.default.circuitBreaker.enabled</tt></td></tr>
 <tr><th>Instance Property</th><td><tt>hystrix.command.<i>HystrixCommandKey</i>.circuitBreaker.enabled</tt></td></tr>
 <tr><th>How to Set Instance Default</th><td><pre>HystrixCommandProperties.Setter()
   .withCircuitBreakerEnabled(boolean value)</pre></td></tr>
</tbody></table>

<a name="circuitBreaker.requestVolumeThreshold" />
#### circuitBreaker.requestVolumeThreshold

This property sets the minimum number of requests in a rolling window that will trip the circuit.

For example, if the value is 20, then if only 19 requests are received in the rolling window (say a window of 10 seconds) the circuit will not trip open even if all 19 failed.

<table><tbody>
 <tr><th>Default Value</th><td><tt>20</tt></td></tr>
 <tr><th>Default Property</th><td><tt>hystrix.command.default.circuitBreaker.requestVolumeThreshold</tt></td></tr>
 <tr><th>Instance Property</th><td><tt>hystrix.command.<i>HystrixCommandKey</i>.circuitBreaker.requestVolumeThreshold</tt></td></tr>
 <tr><th>How to Set Instance Default</th><td><pre>HystrixCommandProperties.Setter()
   .withCircuitBreakerRequestVolumeThreshold(int value)</pre></td></tr>
</tbody></table>

<a name="circuitBreaker.sleepWindowInMilliseconds" />
#### circuitBreaker.sleepWindowInMilliseconds

This property sets the amount of time, after tripping the circuit, to reject requests before allowing attempts again to determine if the circuit should again be closed.

_Default Value:_ `5000`  
_Default Property:_ `hystrix.command.default.circuitBreaker.sleepWindowInMilliseconds`  
_Instance Property:_ `hystrix.command.`[HystrixCommandKey]`.circuitBreaker.sleepWindowInMilliseconds`  
_How to Set Instance Default:_  

```java
HystrixCommandProperties.Setter()
   .withCircuitBreakerSleepWindowInMilliseconds(int value)
```

<a name="circuitBreaker.errorThresholdPercentage" />
#### circuitBreaker.errorThresholdPercentage

This property sets the error percentage at or above which the circuit should trip open and start short-circuiting requests to fallback logic.

_Default Value:_ `50`  
_Default Property:_ `hystrix.command.default.circuitBreaker.errorThresholdPercentage`  
_Instance Property:_ `hystrix.command.`[HystrixCommandKey]`.circuitBreaker.errorThresholdPercentage`  
_How to Set Instance Default:_  

```java
HystrixCommandProperties.Setter()
   .withCircuitBreakerErrorThresholdPercentage(int value)
```

<a name="circuitBreaker.forceOpen" />
#### circuitBreaker.forceOpen

This property, if `true`, forces the circuit breaker into an open (tripped) state in which it will reject all requests.

This property takes precedence over `circuitBreaker.forceClosed`.

_Default Value:_ `false`  
_Default Property:_ `hystrix.command.default.circuitBreaker.forceOpen`  
_Instance Property:_ `hystrix.command.`[HystrixCommandKey]`.circuitBreaker.forceOpen`  
_How to Set Instance Default:_  

```java
HystrixCommandProperties.Setter()
   .withCircuitBreakerForceOpen(boolean value)
```

<a name="circuitBreaker.forceClosed" />
#### circuitBreaker.forceClosed

This property, if `true`, forces the circuit breaker into a closed state in which it will  allow requests regardless of the error percentage.

The `circuitBreaker.forceOpen` property takes precedence so if it is set to `true` this property does nothing.

_Default Value:_ `false`  
_Default Property:_ `hystrix.command.default.circuitBreaker.forceClosed`  
_Instance Property:_ `hystrix.command.`[HystrixCommandKey]`.circuitBreaker.forceClosed`  
_How to Set Instance Default:_  

```java
HystrixCommandProperties.Setter()
   .withCircuitBreakerForceClosed(boolean value)
```

<a name="CommandMetrics" />
### Metrics

The following [Properties](http://netflix.github.com/Hystrix/javadoc/index.html?com/netflix/hystrix/HystrixCommandProperties.html) are related to capturing metrics from [HystrixCommand](http://netflix.github.com/Hystrix/javadoc/index.html?com/netflix/hystrix/HystrixCommand.html) execution.

<a name="metrics.rollingStats.timeInMilliseconds" />
#### metrics.rollingStats.timeInMilliseconds

This property sets the duration of the statistical rolling window, in milliseconds. This is how long Hystrix keeps metrics for the circuit breaker to use and for publishing.

The window is divided into buckets and &ldquo;rolls&rdquo; by these increments.

For example, if this property is set to 10 seconds (`10000`) with ten 1-second buckets, the following diagram represents how it rolls new buckets on and old ones off:

[[images/rolling-stats-640.png]]

_Default Value:_ `10000`  
_Default Property:_ `hystrix.command.default.metrics.rollingStats.timeInMilliseconds`  
_Instance Property:_ `hystrix.command.`[HystrixCommandKey]`.metrics.rollingStats.timeInMilliseconds`  
_How to Set Instance Default:_  

```java
HystrixCommandProperties.Setter()
   .withMetricsRollingStatisticalWindowInMilliseconds(int value)
```

<a name="metrics.rollingStats.numBuckets" />
#### metrics.rollingStats.numBuckets

This property sets the number of buckets the rolling statistical window is divided into.

**Note:** The following must be true &mdash; &ldquo;`metrics.rollingStats.timeInMilliseconds % metrics.rollingStats.numBuckets == 0`&rdquo; &mdash; otherwise it will throw an exception.

In other words, 10000/10 is okay, so is 10000/20 but 10000/7 is not.

_Default Value:_ `10`  
_Possible Values:_ Any value that `metrics.rollingStats.timeInMilliseconds` can be evenly divided by. The result however should be buckets measuring hundreds or thousands of milliseconds. Performance at high volume has not been tested with buckets &lt;100ms.  
_Default Property:_ `hystrix.command.default.metrics.rollingStats.numBuckets`  
_Instance Property:_ `hystrix.command.`[HystrixCommandKey]`.metrics.rollingStats.numBuckets`  
_How to Set Instance Default:_  

```java
HystrixCommandProperties.Setter()
   .withMetricsRollingStatisticalWindowBuckets(int value)
```

<a name="metrics.rollingPercentile.enabled" />
#### metrics.rollingPercentile.enabled

This property indicates whether execution latencies should be tracked and calculated as percentiles.

_Default Value:_ `true`  
_Default Property:_ `hystrix.command.default.metrics.rollingPercentile.enabled`  
_Instance Property:_ `hystrix.command.`[HystrixCommandKey]`.metrics.rollingPercentile.enabled`  
_How to Set Instance Default:_  

```java
HystrixCommandProperties.Setter()
   .withMetricsRollingPercentileEnabled(boolean value)
```

<a name="metrics.rollingPercentile.timeInMilliseconds" />
#### metrics.rollingPercentile.timeInMilliseconds

This property sets the duration of the rolling window in which execution times are kept to allow for percentile calculations, in milliseconds.

The window is divided into buckets and &ldquo;rolls&rdquo; by those increments. 

_Default Value:_ `60000`  
_Default Property:_ `hystrix.command.default.metrics.rollingPercentile.timeInMilliseconds`  
_Instance Property:_ `hystrix.command.`[HystrixCommandKey]`.metrics.rollingPercentile.timeInMilliseconds`  
_How to Set Instance Default:_  

```java
HystrixCommandProperties.Setter()
   .withMetricsRollingPercentileWindowInMilliseconds(int value)
```

<a name="metrics.rollingPercentile.numBuckets" />
#### metrics.rollingPercentile.numBuckets

This property sets the number of buckets the `rollingPercentile` window will be divided into.

Note: The following must be true &mdash; &ldquo;`metrics.rollingPercentile.timeInMilliseconds % metrics.rollingPercentile.numBuckets == 0`&rdquo; &mdash; otherwise it will throw an exception.

In other words, 60000/6 is okay, so is 60000/60 but 10000/7 is not.

_Default Value:_ `6`  
_Possible Values:_ Any value that `metrics.rollingPercentile.timeInMilliseconds` can be evenly divided by. The result however should be buckets measuring thousands of milliseconds. Performance at high volume has not been tested with buckets &lt;1000ms.  
_Default Property:_ `hystrix.command.default.metrics.rollingPercentile.numBuckets`  
_Instance Property:_ `hystrix.command.[HystrixCommandKey].metrics.rollingPercentile.numBuckets`  
_How to Set Instance Default:_  

```java
HystrixCommandProperties.Setter()
   .withMetricsRollingPercentileWindowBuckets(int value)
```

<a name="metrics.rollingPercentile.bucketSize" />
#### metrics.rollingPercentile.bucketSize

This property sets the maximum number of execution times that are kept per bucket. If more executions occur during the time they will wrap around and start over-writing at the beginning of the bucket. 

For example, if bucket size is set to 100 and represents a bucket window of 10 seconds, but 500 executions occur during this time, only the last 100 executions will be kept in that 10 second bucket.

If you increase this size, this also increases the amount of memory needed to store values and increases the time needed for sorting the lists to do percentile calculations.

_Default Value:_ `100`   
_Default Property:_ `hystrix.command.default.metrics.rollingPercentile.bucketSize`  
_Instance Property:_ `hystrix.command.`[HystrixCommandKey]`.metrics.rollingPercentile.bucketSize`  
_How to Set Instance Default:_  

```java
HystrixCommandProperties.Setter()
   .withMetricsRollingPercentileBucketSize(int value)
```

<a name="metrics.healthSnapshot.intervalInMilliseconds" />
#### metrics.healthSnapshot.intervalInMilliseconds

This property sets the time to wait, in milliseconds, between allowing health snapshots to be taken that calculate success and error percentages and affect circuit breaker status.

On high-volume circuits the continual calculation of error percentages can become CPU intensive thus this property allows you to control how often it is calculated.

_Default Value:_ `500`  
_Default Property:_ `hystrix.command.default.metrics.healthSnapshot.intervalInMilliseconds`  
_Instance Property:_ `hystrix.command.`[HystrixCommandKey]`.metrics.healthSnapshot.intervalInMilliseconds`  
_How to Set Instance Default:_  

```java
HystrixCommandProperties.Setter()
   .withMetricsHealthSnapshotIntervalInMilliseconds(int value)
```

<a name="CommandRequestContext" />
### Request Context

These [Properties](http://netflix.github.com/Hystrix/javadoc/index.html?com/netflix/hystrix/HystrixCommandProperties.html) concern [HystrixRequestContext](http://netflix.github.com/Hystrix/javadoc/index.html?com/netflix/hystrix/strategy/concurrency/HystrixRequestContext.html) functionality used by [HystrixCommand](http://netflix.github.com/Hystrix/javadoc/index.html?com/netflix/hystrix/HystrixCommand.html).

<a name="requestCache.enabled" />
#### requestCache.enabled

This property indicates whether [HystrixCommand.getCacheKey()](http://netflix.github.com/Hystrix/javadoc/index.html?com/netflix/hystrix/HystrixCommand.html#getCacheKey\(\)) should be used with [HystrixRequestCache](http://netflix.github.com/Hystrix/javadoc/index.html?com/netflix/hystrix/HystrixRequestCache.html) to provide de-duplication functionality via request-scoped caching. 

_Default Value:_ `true`  
_Default Property:_ `hystrix.command.default.requestCache.enabled`  
_Instance Property:_ `hystrix.command.`[HystrixCommandKey]`.requestCache.enabled`  
_How to Set Instance Default:_  

```java
HystrixCommandProperties.Setter()
   .withRequestCacheEnabled(boolean value)
```

<a name="requestLog.enabled" />
#### requestLog.enabled

This property indicates whether [HystrixCommand](http://netflix.github.com/Hystrix/javadoc/index.html?com/netflix/hystrix/HystrixCommand.html) execution and events should be logged to [HystrixRequestLog](http://netflix.github.com/Hystrix/javadoc/index.html?com/netflix/hystrix/HystrixRequestLog.html). 

_Default Value:_ `true`  
_Default Property:_ `hystrix.command.default.requestLog.enabled`  
_Instance Property:_ `hystrix.command.`[HystrixCommandKey]`.requestLog.enabled`  
_How to Set Instance Default:_  

```java
HystrixCommandProperties.Setter()
   .withRequestLogEnabled(boolean value)
```

<a name="Collapser" />
## Collapser Properties

The following [Properties](http://netflix.github.com/Hystrix/javadoc/index.html?com/netflix/hystrix/HystrixCollapserProperties.html) control [HystrixCollapser](http://netflix.github.com/Hystrix/javadoc/index.html?com/netflix/hystrix/HystrixCollapser.html) behavior.

<a name="maxRequestsInBatch" />
#### maxRequestsInBatch

This property sets the maximum number of requests allowed in a batch before this triggers a batch execution.

_Default Value:_ `Integer.MAX_VALUE`  
_Default Property:_ `hystrix.collapser.default.maxRequestsInBatch`  
_Instance Property:_ `hystrix.collapser.`[HystrixCollapserKey]`.maxRequestsInBatch`  
_How to Set Instance Default:_  

```java
HystrixCollapserProperties.Setter()
   .withMaxRequestsInBatch(int value)
```

<a name="timerDelayInMilliseconds" />
#### timerDelayInMilliseconds

This property sets the number of milliseconds after the creation of the batch that its execution is triggered.

_Default Value:_ `10`  
_Default Property:_ `hystrix.collapser.default.timerDelayInMilliseconds`  
_Instance Property:_ `hystrix.collapser.`[HystrixCollapserKey]`.timerDelayInMilliseconds`  
_How to Set Instance Default:_  

```java
HystrixCollapserProperties.Setter()
   .withTimerDelayInMilliseconds(int value)
```

<a name="collapser.requestCache.enabled" />
#### requestCache.enabled

This property indicates whether request caching is enabled for [HystrixCollapser.execute()](http://netflix.github.com/Hystrix/javadoc/index.html?com/netflix/hystrix/HystrixCollapser.html#execute\(\)) and [HystrixCollapser.queue()](http://netflix.github.com/Hystrix/javadoc/index.html?com/netflix/hystrix/HystrixCollapser.html#queue\(\)) invocations.

_Default Value:_ `true`  
_Default Property:_ `hystrix.collapser.default.requestCache.enabled`  
_Instance Property:_ `hystrix.collapser.`[HystrixCollapserKey]`.requestCache.enabled`  
_How to Set Instance Default:_  

```java
HystrixCollapserProperties.Setter()
   .withRequestCacheEnabled(boolean value)
```

<a name="ThreadPool" />
## ThreadPool Properties

The following [Properties](http://netflix.github.com/Hystrix/javadoc/index.html?com/netflix/hystrix/HystrixThreadPoolProperties.html) control the behavior of the thread-pools that Hystrix Commands execute on.

Most of the time the default value of 10 threads will be fine (often it could be made smaller).

To determine if it needs to be larger, a basic formula for calculating the size is:

_requests per second at peak when healthy &times; 99th percentile latency in seconds + some breathing room_

See the example below to see how this formula is put into practice.

The general principle is keep the pool as small as possible, as it is the primary tool to shed load and prevent resources from becoming blocked if latency occurs.

> Netflix API has 30+ of its threadpools set at 10, two at 20, and one at 25.

<a href="images/thread-configuration-1280.png">[[images/thread-configuration-640.png]]</a>
_(Click for larger view)_

The above diagram shows an example configuration in which the dependency has no reason to hit the 99.5th percentile and therefore it cuts it short at the network timeout layer and immediately retries with the expectation that it will get median latency most of the time, and will be able to accomplish this all within the 300ms thread timeout.

If the dependency has legitimate reasons to sometimes hit the 99.5th percentile (such as cache miss with lazy generation) then the network timeout will be set higher than it, such as at 325ms with 0 or 1 retries and the thread timeout set higher (350ms+).

The thread-pool is sized at 10 to handle a burst of 99th percentile requests, but when everything is healthy this threadpool will typically only have 1 or 2 threads active at any given time to serve mostly 40ms median calls.

When you configure it correctly a timeout at the `HystrixCommand` layer should be rare, but the protection is there in case something other than network latency affects the time, or the combination of connect+read+retry+connect+read in a worst case scenario still exceeds the configured overall timeout.

The aggressiveness of configurations and tradeoffs in each direction are different for each dependency.

You can change configurations in real-time as needed as performance characteristics change or when problems are found, all without the risk of taking down the entire app if problems or misconfigurations occur.

<a name="coreSize" />
#### coreSize

This property sets the core thread-pool size. This is the maximum number of [HystrixCommands](http://netflix.github.com/Hystrix/javadoc/index.html?com/netflix/hystrix/HystrixCommand.html) that can execute concurrently.

_Default Value:_ `10`  
_Default Property:_ `hystrix.threadpool.default.coreSize`  
_Instance Property:_ `hystrix.threadpool.`[HystrixThreadPoolKey]`.coreSize`  
_How to Set Instance Default:_  

```java
HystrixThreadPoolProperties.Setter()
   .withCoreSize(int value)
```

<a name="maxQueueSize" />
#### maxQueueSize

This property sets the maximum queue size of the `BlockingQueue` implementation.

If you set this to `-1` then [SynchronousQueue](http://docs.oracle.com/javase/6/docs/api/java/util/concurrent/SynchronousQueue.html) will be used, otherwise a positive value will be used with [LinkedBlockingQueue](http://docs.oracle.com/javase/6/docs/api/java/util/concurrent/LinkedBlockingQueue.html).

**Note:** This property only applies at initialization time since queue implementations cannot be resized or changed without re-initializing the thread executor which is not supported.

If you need to overcome this limitation and to allow dynamic changes in the queue, see the `queueSizeRejectionThreshold` property. 

To change between `SynchronousQueue` and `LinkedBlockingQueue` requires a restart.

_Default Value:_ `-1`  
_Default Property:_ `hystrix.threadpool.default.maxQueueSize`  
_Instance Property:_ `hystrix.threadpool.`[HystrixThreadPoolKey]`.maxQueueSize`  
_How to Set Instance Default:_  

```java
HystrixThreadPoolProperties.Setter()
   .withMaxQueueSize(int value)
```

<a name="queueSizeRejectionThreshold" />
#### queueSizeRejectionThreshold

This property sets the queue size rejection threshold &mdash; an artificial maximum queue size at which rejections will occur even if `maxQueueSize` has not been reached. This property exists because the `maxQueueSize` of a [BlockingQueue](http://docs.oracle.com/javase/6/docs/api/java/util/concurrent/BlockingQueue.html) cannot be dynamically changed and we want to allow you to dynamically change the queue size that affects rejections.

This is used by `HystrixCommand` when queuing a thread for execution.

**Note:** This property is not applicable if `maxQueueSize == -1`.

_Default Value:_ `5`  
_Default Property:_ `hystrix.threadpool.default.queueSizeRejectionThreshold`  
_Instance Property:_ `hystrix.threadpool.`[HystrixThreadPoolKey]`.queueSizeRejectionThreshold`  
_How to Set Instance Default:_  

```java
HystrixThreadPoolProperties.Setter()
   .withQueueSizeRejectionThreshold(int value)
```

<a name="keepAliveTimeMinutes" />
#### keepAliveTimeMinutes

This property sets the keep-alive time, in minutes.

This is in practice not used since the `corePoolSize` and `maxPoolSize` are set to the same value in the default implementation, but if you were to use a custom implementation via [[plugin|Plugins]] then this would be available for you to use.

_Default Value:_ `1`  
_Default Property:_ `hystrix.threadpool.default.keepAliveTimeMinutes`  
_Instance Property:_ `hystrix.threadpool.`[HystrixThreadPoolKey]`.keepAliveTimeMinutes`  
_How to Set Instance Default:_  

```java
HystrixThreadPoolProperties.Setter()
   .withKeepAliveTimeMinutes(int value)
```

<a name="threadpool.metrics.rollingStats.timeInMilliseconds" />
#### metrics.rollingStats.timeInMilliseconds

This property sets the duration of the statistical rolling window, in milliseconds. This is how long metrics are kept for the thread pool.

The window is divided into buckets and &ldquo;rolls&rdquo; by those increments.

_Default Value:_ `10000`  
_Default Property:_ `hystrix.threadpool.default.metrics.rollingStats.timeInMilliseconds`  
_Instance Property:_ `hystrix.threadpool.`[HystrixThreadPoolKey]`.metrics.rollingStats.timeInMilliseconds`  
_How to Set Instance Default:_  

```java
HystrixThreadPoolProperties.Setter()
   .withMetricsRollingStatisticalWindowInMilliseconds(int value)
```

<a name="threadpool.metrics.rollingStats.numBuckets" />
#### metrics.rollingStats.numBuckets

This property sets the number of buckets the rolling statistical window is divided into.

**Note:** The following must be true &mdash; &ldquo;`metrics.rollingStats.timeInMilliseconds % metrics.rollingStats.numBuckets == 0`&rdquo; &mdash; otherwise it will throw an exception.

In other words, 10000/10 is okay, so is 10000/20 but 10000/7 is not.

_Default Value:_ `10`  
_Possible Values:_ Any value that `metrics.rollingStats.timeInMilliseconds` can be evenly divided by. The result however should be buckets measuring hundreds or thousands of milliseconds. Performance at high volume has not been tested with buckets &lt;100ms.  
_Default Property:_ `hystrix.threadpool.default.metrics.rollingPercentile.numBuckets`  
_Instance Property:_ `hystrix.threadpool.`[HystrixThreadPoolProperties]`.metrics.rollingPercentile.numBuckets`  
_How to Set Instance Default:_  

```java
HystrixThreadPoolProperties.Setter()
   .withMetricsRollingStatisticalWindowBuckets(int value)
```