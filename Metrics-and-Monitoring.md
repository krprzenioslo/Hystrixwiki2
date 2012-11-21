Metrics are captured using the _HystrixRollingNumber_ and _HystrixRollingPercentile_ classes in rolling windows. The rolling windows allow low-latency moving windows of metrics to be used for circuit breaker health checks and [[operations|Operations]].

Metrics are published by an implementation of [HystrixMetricsPublisher](http://netflix.github.com/Hystrix/javadoc/index.html?com/netflix/hystrix/strategy/metrics/HystrixMetricsPublisher.html).

The default implementation uses [Servo](https://github.com/Netflix/servo) or a custom implementation can be registered using [HystrixPlugins.registerMetricsPublisher(HystrixMetricsPublisher impl)](http://netflix.github.com/Hystrix/javadoc/index.html?com/netflix/hystrix/strategy/HystrixPlugins.html#registerMetricsPublisher(com.netflix.hystrix.strategy.metrics.HystrixMetricsPublisher\)).

Following are details of metrics published with the default implementation using [Servo](https://github.com/Netflix/servo):

## Command Metrics

Each [HystrixCommand](http://netflix.github.com/Hystrix/javadoc/index.html?com/netflix/HystrixCommand.html) publishes metrics with the following tags:

* Servo Tag: "instance"  Value: HystrixCommandKey.name()
* Servo Tag: "type"  Value: "HystrixCommand"

### Informational and Status

* _Boolean_ isCircuitBreakerOpen  
* _Number_ errorPercentage
* _Number_ executionSemaphorePermitsInUse
* _String_ commandGroup
* _Number_ currentTime

### Cumulative Counts ([Counter](https://github.com/Netflix/servo/blob/master/servo-core/src/main/java/com/netflix/servo/monitor/Counter.java))

The following are cumulative counts since the start of the application.

* _Long_ countCollapsedRequests
* _Long_ countExceptionsThrown
* _Long_ countFailure
* _Long_ countFallbackFailure
* _Long_ countFallbackRejection
* _Long_ countFallbackSuccess
* _Long_ countResponsesFromCache
* _Long_ countSemaphoreRejected
* _Long_ countShortCircuited
* _Long_ countSuccess
* _Long_ countThreadPoolRejected
* _Long_ countTimeout

### Rolling Counts ([Gauge](https://github.com/Netflix/servo/blob/master/servo-core/src/main/java/com/netflix/servo/monitor/Gauge.java))

The following are rolling counts as configured by [[_metrics.rollingStats.*_ properties|Configuration]].

These are "point in time" counts representing the last X seconds (for example 10 seconds).

* _Number_ rollingCountCollapsedRequests
* _Number_ rollingCountExceptionsThrown
* _Number_ rollingCountFailure
* _Number_ rollingCountFallbackFailure
* _Number_ rollingCountFallbackRejection
* _Number_ rollingCountFallbackSuccess
* _Number_ rollingCountResponsesFromCache
* _Number_ rollingCountSemaphoreRejected
* _Number_ rollingCountShortCircuited
* _Number_ rollingCountSuccess
* _Number_ rollingCountThreadPoolRejected
* _Number_ rollingCountTimeout

### Latency Percentiles: HystrixCommand.run() Execution ([Gauge](https://github.com/Netflix/servo/blob/master/servo-core/src/main/java/com/netflix/servo/monitor/Gauge.java))

Percentiles of execution times for the [HystrixCommand.run()](http://netflix.github.com/Hystrix/javadoc/index.html?com/netflix/hystrix/HystrixCommand.html#run(\)) method (on the child thread if using thread isolation).

These are rolling percentiles as configured by [[_metrics.rollingPercentile.*_ properties|Configuration]].

* _Number_ latencyExecute_mean
* _Number_ latencyExecute_percentile_5
* _Number_ latencyExecute_percentile_25
* _Number_ latencyExecute_percentile_50
* _Number_ latencyExecute_percentile_75
* _Number_ latencyExecute_percentile_90
* _Number_ latencyExecute_percentile_99
* _Number_ latencyExecute_percentile_995

### Latency Percentiles: End-to-End Execution ([Gauge](https://github.com/Netflix/servo/blob/master/servo-core/src/main/java/com/netflix/servo/monitor/Gauge.java))

Percentiles of execution times for the end-to-end execution of [HystrixCommand.execute()](http://netflix.github.com/Hystrix/javadoc/index.html?com/netflix/hystrix/HystrixCommand.html#execute(\)) or [HystrixCommand.queue()](http://netflix.github.com/Hystrix/javadoc/index.html?com/netflix/hystrix/HystrixCommand.html#queue(\)) until a response is returned (or ready to return in case of [queue()](http://netflix.github.com/Hystrix/javadoc/index.html?com/netflix/hystrix/HystrixCommand.html#queue(\))).

The purpose of this compared with the _latencyExecute*_ percentiles is to measure the cost of thread queuing/scheduling/execution, semaphores, circuit breaker logic and other aspects of overhead (including metrics capture itself).

These are rolling percentiles as configured by [[_metrics.rollingPercentile.*_ properties|Configuration]].

* _Number_ latencyTotal_mean
* _Number_ latencyTotal_percentile_5
* _Number_ latencyTotal_percentile_25
* _Number_ latencyTotal_percentile_50
* _Number_ latencyTotal_percentile_75
* _Number_ latencyTotal_percentile_90
* _Number_ latencyTotal_percentile_99
* _Number_ latencyTotal_percentile_995

### Property Values ([Informational](https://github.com/Netflix/servo/blob/master/servo-core/src/main/java/com/netflix/servo/monitor/Informational.java))

These informational metrics report the actual property values being used by the [HystrixCommand](http://netflix.github.com/Hystrix/javadoc/index.html?com/netflix/hystrix/HystrixCommand.html). This is useful to see when a dynamic property takes effect and confirm a property is set as expected.

* _Number_ propertyValue_rollingStatisticalWindowInMilliseconds
* _Number_ propertyValue_circuitBreakerRequestVolumeThreshold
* _Number_ propertyValue_circuitBreakerSleepWindowInMilliseconds
* _Number_ propertyValue_circuitBreakerErrorThresholdPercentage
* _Boolean_ propertyValue_circuitBreakerForceOpen
* _Boolean_ propertyValue_circuitBreakerForceClosed
* _Number_ propertyValue_executionIsolationThreadTimeoutInMilliseconds
* _String_ propertyValue_executionIsolationStrategy
* _Boolean_ propertyValue_metricsRollingPercentileEnabled
* _Boolean_ propertyValue_requestCacheEnabled
* _Boolean_ propertyValue_requestLogEnabled
* _Number_ propertyValue_executionIsolationSemaphoreMaxConcurrentRequests
* _Number_ propertyValue_fallbackIsolationSemaphoreMaxConcurrentRequests


## ThreadPool Metrics

Each [HystrixThreadPool](http://netflix.github.com/Hystrix/javadoc/index.html?com/netflix/hystrix/HystrixThreadPool.html) publishes metrics with the following tags:

* Servo Tag: "instance"  Value: HystrixThreadPoolKey.name()
* Servo Tag: "type"  Value: "HystrixThreadPool"

### Informational and Status

* _String_ name
* _Number_ currentTime

### Rolling Counts ([Gauge](https://github.com/Netflix/servo/blob/master/servo-core/src/main/java/com/netflix/servo/monitor/Gauge.java))

* _Number_ rollingMaxActiveThreads
* _Number_ rollingCountThreadsExecuted

### Cumulative Counts ([Counter](https://github.com/Netflix/servo/blob/master/servo-core/src/main/java/com/netflix/servo/monitor/Counter.java))

* _Long_ countThreadsExecuted

### ThreadPool State ([Gauge](https://github.com/Netflix/servo/blob/master/servo-core/src/main/java/com/netflix/servo/monitor/Gauge.java))

* _Number_ threadActiveCount
* _Number_ completedTaskCount
* _Number_ largestPoolSize
* _Number_ totalTaskCount
* _Number_ queueSize

### Property Values ([Informational](https://github.com/Netflix/servo/blob/master/servo-core/src/main/java/com/netflix/servo/monitor/Informational.java))

* _Number_ propertyValue_corePoolSize
* _Number_ propertyValue_keepAliveTimeInMinutes
* _Number_ propertyValue_queueSizeRejectionThreshold
* _Number_ propertyValue_maxQueueSize

## Dashboard

A dashboard for near real-time monitoring Hystrix circuits __will be open-sourced soon__.

It allows monitoring a single server or cluster of servers in aggregate with low latency (typically around 1 or 2 seconds when aggregating a cluster, subsecond with a single server).

<a href="images/dashboard-example-1280.png">[[images/dashboard-example-640.png]]</a>
_(Click for larger view.)_

Following is an explanation of the metrics that are reported:

[[images/dashboard-annoted-circuit-640.png]]