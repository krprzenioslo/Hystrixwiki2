The Hystrix Dashboard enables realtime monitoring of Hystrix metrics.

Use of this dashboard improved Netflix operations by reducing discovery and recovery times during operational events. The duration of most production incidents (already less frequent due to Hystrix) became far shorter, with diminished impact, due to the realtime insights into system behavior.

<a href="images/hystrix-dashboard-single-row.png">[[images/hystrix-dashboard-single-row-640.png]]</a>
_(Click for larger view.)_

When a circuit is failing then it changes colors (gradient from green through yellow, orange and red) such as this: 

[[images/dashboard-example-open-circuit-640.png]]

The diagram below shows one "circuit" from the dashboard along with explanations of what all of the data represents.

We've purposefully tried to pack a lot of information into the dashboard so that engineers can quickly consume and correlate data.

[[images/dashboard-annoted-circuit-640.png]]

It allows monitoring a single server or a cluster of servers aggregated using <a href="https://github.com/Netflix/Turbine">Turbine</a> with low latency (typically around 1 or 2 seconds when aggregating a cluster, subsecond with a single server).

[[images/dashboard-direct-vs-turbine-640.png]]

Here is another example from the Netflix API dashboard monitoring 476 servers aggregated using <a href="https://github.com/Netflix/Turbine">Turbine</a>:

<a href="images/dashboard-example-1280.png">[[images/dashboard-example-640.png]]</a>
_(Click for larger view.)_

# Installation of Dashboard

### Download

1) Download <a href="https://github.com/downloads/Netflix/Hystrix/hystrix-dashboard-1.1.1.war">hystrix-dashboard-1.1.1.war</a>  
2) Install in servlet container such as <a href="http://tomcat.apache.org/download-70.cgi">Apache Tomcat 7</a>

Usage examples below will assume installation to /webapps/hystrix.war

### Build

```
./gradlew build
cp hystrix-dashboard/build/libs/hystrix-dashboard-*.war ./apache-tomcat-7.*/webapps/hystrix.war  (or other servlet container)
```

# Installation of Metrics Stream

The [hystrix-metrics-event-stream](Hystrix/tree/master/hystrix-contrib/hystrix-metrics-event-stream) module exposes metrics in a [text/event-stream](https://developer.mozilla.org/en-US/docs/Server-sent_events/Using_server-sent_events) formatted stream that continues as long as a client holds the connection.

Each HystrixCommand instance will emit data such as this (without line breaks):

```json
data: {
  "type": "HystrixCommand",
  "name": "PlaylistGet",
  "group": "PlaylistGet",
  "currentTime": 1355239617628,
  "isCircuitBreakerOpen": false,
  "errorPercentage": 0,
  "errorCount": 0,
  "requestCount": 121,
  "rollingCountCollapsedRequests": 0,
  "rollingCountExceptionsThrown": 0,
  "rollingCountFailure": 0,
  "rollingCountFallbackFailure": 0,
  "rollingCountFallbackRejection": 0,
  "rollingCountFallbackSuccess": 0,
  "rollingCountResponsesFromCache": 69,
  "rollingCountSemaphoreRejected": 0,
  "rollingCountShortCircuited": 0,
  "rollingCountSuccess": 121,
  "rollingCountThreadPoolRejected": 0,
  "rollingCountTimeout": 0,
  "currentConcurrentExecutionCount": 0,
  "latencyExecute_mean": 13,
  "latencyExecute": {
    "0": 3,
    "25": 6,
    "50": 8,
    "75": 14,
    "90": 26,
    "95": 37,
    "99": 75,
    "99.5": 92,
    "100": 252
  },
  "latencyTotal_mean": 15,
  "latencyTotal": {
    "0": 3,
    "25": 7,
    "50": 10,
    "75": 18,
    "90": 32,
    "95": 43,
    "99": 88,
    "99.5": 160,
    "100": 253
  },
  "propertyValue_circuitBreakerRequestVolumeThreshold": 20,
  "propertyValue_circuitBreakerSleepWindowInMilliseconds": 5000,
  "propertyValue_circuitBreakerErrorThresholdPercentage": 50,
  "propertyValue_circuitBreakerForceOpen": false,
  "propertyValue_circuitBreakerForceClosed": false,
  "propertyValue_circuitBreakerEnabled": true,
  "propertyValue_executionIsolationStrategy": "THREAD",
  "propertyValue_executionIsolationThreadTimeoutInMilliseconds": 800,
  "propertyValue_executionIsolationThreadInterruptOnTimeout": true,
  "propertyValue_executionIsolationThreadPoolKeyOverride": null,
  "propertyValue_executionIsolationSemaphoreMaxConcurrentRequests": 20,
  "propertyValue_fallbackIsolationSemaphoreMaxConcurrentRequests": 10,
  "propertyValue_metricsRollingStatisticalWindowInMilliseconds": 10000,
  "propertyValue_requestCacheEnabled": true,
  "propertyValue_requestLogEnabled": true,
  "reportingHosts": 1
}
```

The Hystrix Dashboard expects data in the format that this module emits.

See its [README](Hystrix/tree/master/hystrix-contrib/hystrix-metrics-event-stream) for installation instructions.

# Installation of Turbine (optional)

how to

# Using

how