Hystrix captures metrics using the `HystrixRollingNumber` and `HystrixRollingPercentile` classes in rolling windows. The rolling windows allow Hystrix to use low-latency moving windows of metrics for circuit breaker health checks and [[operations|Operations]].

# Direct Access

You can access metrics programmatically with the following calls:

```java
HystrixCommandMetrics.getInstances()
HystrixThreadPoolMetrics.getInstances()
```

# Metrics Event Stream

You can use the [hystrix-metrics-event-stream](../tree/master/hystrix-contrib/hystrix-metrics-event-stream) to power the [[dashboard|Dashboard]], real-time alerting, and other such use cases.

# Metrics Publisher

You can publish metrics by using an implementation of [HystrixMetricsPublisher](http://netflix.github.com/Hystrix/javadoc/index.html?com/netflix/hystrix/strategy/metrics/HystrixMetricsPublisher.html).

Register your `HystrixMetricsPublisher` implementations by calling [HystrixPlugins.registerMetricsPublisher(HystrixMetricsPublisher impl)](http://netflix.github.com/Hystrix/javadoc/index.html?com/netflix/hystrix/strategy/HystrixPlugins.html#registerMetricsPublisher\(com.netflix.hystrix.strategy.metrics.HystrixMetricsPublisher\)).

Hystrix includes the following implementations as `hystrix-contrib` modules:
- Netflix Servo: [hystrix-servo-metrics-publisher](../tree/master/hystrix-contrib/hystrix-servo-metrics-publisher)
- Yammer Metrics: [hystrix-yammer-metrics-publisher](../tree/master/hystrix-contrib/hystrix-yammer-metrics-publisher)

The following sections explain the metrics published with those implementations:

## Command Metrics

Each [HystrixCommand](http://netflix.github.com/Hystrix/javadoc/index.html?com/netflix/HystrixCommand.html) publishes metrics with the following tags:

* Servo Tag: `"instance"`, Value: `HystrixCommandKey.name()`
* Servo Tag: `"type"`, Value: `"HystrixCommand"`

### Informational and Status

* _Boolean_ `isCircuitBreakerOpen`  
* _Number_ `errorPercentage`
* _Number_ `executionSemaphorePermitsInUse`
* _String_ `commandGroup`
* _Number_ `currentTime`

### Cumulative and Rolling Event Counts
Cumulative counts ([`Counter`](https://github.com/Netflix/servo/blob/master/servo-core/src/main/java/com/netflix/servo/monitor/Counter.java)) represent the number of events since the start of the application. 

Rolling counts ([`Gauge`](https://github.com/Netflix/servo/blob/master/servo-core/src/main/java/com/netflix/servo/monitor/Gauge.java)) are configured by [[metrics.rollingStats.* properties|Configuration]]. They are &ldquo;point in time&rdquo; counts representing the last _x_ seconds (for example 10 seconds).

<table>
 <thead>
  <tr><th>Event</th><th>Cumulative Count (Long)</th><th>Rolling Count (Number)</th></tr>
 </thead><tbody>
  <tr><td><code>BAD_REQUEST</code></td><td><code>countBadRequests</code></td><td><code>rollingCountBadRequests</code></td></tr>
  <tr><td><code>COLLAPSED</code></td><td><code>countCollapsedRequests</code></td><td><code>rollingCountCollapsedRequests</code></td></tr>
  <tr><td><code>EMIT</code></td><td><code>countEmit</code></td><td><code>rollingCountEmit</code></td></tr>
  <tr><td><code>EXCEPTION_THROWN</code></td><td><code>countExceptionsThrown</code></td><td><code>rollingCountExceptionsThrown</code></td></tr>
  <tr><td><code>FAILURE</code></td><td><code>countFailure</code></td><td><code>rollingCountFailure</code></td></tr>
  <tr><td><code>FALLBACK_EMIT</code></td><td><code>countFallbackEmit</code></td><td><code>rollingCountFallbackEmit</code></td></tr>
  <tr><td><code>FALLBACK_FAILURE</code></td><td><code>countFallbackFailure</code></td><td><code>rollingCountFallbackFailure</code></td></tr>
  <tr><td><code>FALLBACK_REJECTION</code></td><td><code>countFallbackRejection</code></td><td><code>rollingCountFallbackRejection</code></td></tr>
  <tr><td><code>FALLBACK_SUCCESS</code></td><td><code>countFallbackSuccess</code></td><td><code>rollingCountFallbackSuccess</code></td></tr>
  <tr><td><code>RESPONSE_FROM_CACHE</code></td><td><code>countResponsesFromCache</code></td><td><code>rollingCountResponsesFromCache</code></td></tr>
  <tr><td><code>SEMAPHORE_REJECTED</code></td><td><code>countSemaphoreRejected</code></td><td><code>rollingCountSemaphoreRejected</code></td></tr>
  <tr><td><code>SHORT_CIRCUITED</code></td><td><code>countShortCircuited</code></td><td><code>rollingCountShortCircuited</code></td></tr>
  <tr><td><code>SUCCESS</code></td><td><code>countSuccess</code></td><td><code>rollingCountSuccess</code></td></tr>
  <tr><td><code>THREAD_POOL_REJECTED</code></td><td><code>countThreadPoolRejected</code></td><td><code>rollingCountThreadPoolRejected</code></td></tr>
  <tr><td><code>TIMEOUR</code></td><td><code>countTimeout</code></td><td><code>rollingCountTimeout</code></td></tr>
 </tbody>
</table>

### Latency Percentiles: HystrixCommand.run() Execution ([Gauge](https://github.com/Netflix/servo/blob/master/servo-core/src/main/java/com/netflix/servo/monitor/Gauge.java))

These metrics represent percentiles of execution times for the [HystrixCommand.run()](http://netflix.github.com/Hystrix/javadoc/index.html?com/netflix/hystrix/HystrixCommand.html#run\(\)) method (on the child thread if using thread isolation).

These are rolling percentiles as configured by [[metrics.rollingPercentile.* properties|Configuration]].

* _Number_ `latencyExecute_mean`
* _Number_ `latencyExecute_percentile_5`
* _Number_ `latencyExecute_percentile_25`
* _Number_ `latencyExecute_percentile_50`
* _Number_ `latencyExecute_percentile_75`
* _Number_ `latencyExecute_percentile_90`
* _Number_ `latencyExecute_percentile_99`
* _Number_ `latencyExecute_percentile_995`

### Latency Percentiles: End-to-End Execution ([Gauge](https://github.com/Netflix/servo/blob/master/servo-core/src/main/java/com/netflix/servo/monitor/Gauge.java))

These metrics represent percentiles of execution times for the end-to-end execution of [HystrixCommand.execute()](http://netflix.github.com/Hystrix/javadoc/index.html?com/netflix/hystrix/HystrixCommand.html#execute\(\)) or [HystrixCommand.queue()](http://netflix.github.com/Hystrix/javadoc/index.html?com/netflix/hystrix/HystrixCommand.html#queue\(\)) until a response is returned (or is ready to return in case of [queue()](http://netflix.github.com/Hystrix/javadoc/index.html?com/netflix/hystrix/HystrixCommand.html#queue\(\))).

The purpose of this compared with the `latencyExecute*` percentiles is to measure the cost of thread queuing/scheduling/execution, semaphores, circuit breaker logic, and other aspects of overhead (including metrics capture itself).

These are rolling percentiles as configured by [[metrics.rollingPercentile.* properties|Configuration]].

* _Number_ `latencyTotal_mean`
* _Number_ `latencyTotal_percentile_5`
* _Number_ `latencyTotal_percentile_25`
* _Number_ `latencyTotal_percentile_50`
* _Number_ `latencyTotal_percentile_75`
* _Number_ `latencyTotal_percentile_90`
* _Number_ `latencyTotal_percentile_99`
* _Number_ `latencyTotal_percentile_995`

### Property Values ([Informational](https://github.com/Netflix/servo/blob/master/servo-core/src/main/java/com/netflix/servo/monitor/Informational.java))

These informational metrics report the actual property values being used by the [HystrixCommand](http://netflix.github.com/Hystrix/javadoc/index.html?com/netflix/hystrix/HystrixCommand.html). This enables you to see when a dynamic property takes effect and to confirm a property is set as expected.

* _Number_ `propertyValue_rollingStatisticalWindowInMilliseconds`
* _Number_ `propertyValue_circuitBreakerRequestVolumeThreshold`
* _Number_ `propertyValue_circuitBreakerSleepWindowInMilliseconds`
* _Number_ `propertyValue_circuitBreakerErrorThresholdPercentage`
* _Boolean_ `propertyValue_circuitBreakerForceOpen`
* _Boolean_ `propertyValue_circuitBreakerForceClosed`
* _Number_ `propertyValue_executionIsolationThreadTimeoutInMilliseconds`
* _String_ `propertyValue_executionIsolationStrategy`
* _Boolean_ `propertyValue_metricsRollingPercentileEnabled`
* _Boolean_ `propertyValue_requestCacheEnabled`
* _Boolean_ `propertyValue_requestLogEnabled`
* _Number_ `propertyValue_executionIsolationSemaphoreMaxConcurrentRequests`
* _Number_ `propertyValue_fallbackIsolationSemaphoreMaxConcurrentRequests`

## ThreadPool Metrics

Each [HystrixThreadPool](http://netflix.github.com/Hystrix/javadoc/index.html?com/netflix/hystrix/HystrixThreadPool.html) publishes metrics with the following tags:

* Servo Tag: `"instance"`, Value: `HystrixThreadPoolKey.name()`
* Servo Tag: `"type"`, Value: `"HystrixThreadPool"`

### Informational and Status

* _String_ `name`
* _Number_ `currentTime`

### Rolling Counts ([Gauge](https://github.com/Netflix/servo/blob/master/servo-core/src/main/java/com/netflix/servo/monitor/Gauge.java))

* _Number_ `rollingMaxActiveThreads`
* _Number_ `rollingCountThreadsExecuted`

### Cumulative Counts ([Counter](https://github.com/Netflix/servo/blob/master/servo-core/src/main/java/com/netflix/servo/monitor/Counter.java))

* _Long_ `countThreadsExecuted`

### ThreadPool State ([Gauge](https://github.com/Netflix/servo/blob/master/servo-core/src/main/java/com/netflix/servo/monitor/Gauge.java))

* _Number_ `threadActiveCount`
* _Number_ `completedTaskCount`
* _Number_ `largestPoolSize`
* _Number_ `totalTaskCount`
* _Number_ `queueSize`

### Property Values ([Informational](https://github.com/Netflix/servo/blob/master/servo-core/src/main/java/com/netflix/servo/monitor/Informational.java))

* _Number_ `propertyValue_corePoolSize`
* _Number_ `propertyValue_keepAliveTimeInMinutes`
* _Number_ `propertyValue_queueSizeRejectionThreshold`
* _Number_ `propertyValue_maxQueueSize`