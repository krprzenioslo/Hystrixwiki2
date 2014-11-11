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

# Run via Gradle

```
$ git clone git@github.com:Netflix/Hystrix.git
$ cd Hystrix/hystrix-dashboard
$ ../gradlew jettyRun
> Building > :hystrix-dashboard:jettyRun > Running at http://localhost:7979/hystrix-dashboard
```

Once running, open <a href="http://localhost:7979/hystrix-dashboard">http://localhost:7979/hystrix-dashboard</a>.

# Installation of Dashboard

### Download

1) Download <a href="http://search.maven.org/#browse%7C1045347652">hystrix-dashboard-#.#.#.war</a>  
2) Install in servlet container such as <a href="http://tomcat.apache.org/download-70.cgi">Apache Tomcat 7</a>

Usage examples below will assume installation to /webapps/hystrix-dashboard.war

### Build

```
./gradlew build
cp hystrix-dashboard/build/libs/hystrix-dashboard-*.war ./apache-tomcat-7.*/webapps/hystrix-dashboard.war  (or other servlet container)
```

# Installation of Metrics Stream

The [hystrix-metrics-event-stream](https://github.com/Netflix/Hystrix/tree/master/hystrix-contrib/hystrix-metrics-event-stream) module exposes metrics in a [text/event-stream](https://developer.mozilla.org/en-US/docs/Server-sent_events/Using_server-sent_events) formatted stream that continues as long as a client holds the connection.

The Hystrix Dashboard expects data in the format that this module emits.

See its [README](Hystrix/tree/master/hystrix-contrib/hystrix-metrics-event-stream) for installation instructions.

# Installation of Turbine (optional)

### Download

1) Download <a href="https://github.com/downloads/Netflix/Turbine/turbine-web-1.0.0.war">turbine-web-1.0.0.war</a>  
2) Install in servlet container such as <a href="http://tomcat.apache.org/download-70.cgi">Apache Tomcat 7</a>

Usage examples below will assume installation to /webapps/turbine.war

### Build

```
git clone git://github.com/Netflix/Turbine.git
./gradlew build
cp turbine-web/build/libs/turbine-web-*.war ./apache-tomcat-7.*/webapps/turbine.war  (or other servlet container)
```

### Configure Hosts Discovery

Turbine configuration details can be found on its [Configuration Wiki](https://github.com/Netflix/Turbine/wiki/Configuration-(1.x)). It also supports custom plugins for [Instance Discovery](https://github.com/Netflix/Turbine/wiki/Plugging-in-your-own-InstanceDiscovery-(1.x)).

To get started as a "Hello World!" example a static configuration file pointing to specific instances can be used such as the following.

Create a file config.properties that lists hosts to aggregate.

This example includes 2 EC2 instances:

```
turbine.ConfigPropertyBasedDiscovery.default.instances=ec2-72-44-38-203.compute-1.amazonaws.com,ec2-23-20-84-255.compute-1.amazonaws.com
turbine.instanceUrlSuffix=:8080/hystrix.stream
```

The 'turbine.instanceUrlSuffix' property is what will be appended to each hostname to create a URL that will result in the [hystrix-metrics-event-stream](Hystrix/tree/master/hystrix-contrib/hystrix-metrics-event-stream).

The config.properties file can be:

- placed on the classpath such as in /WEB-INF/classes
- specified using a [JVM property](https://github.com/Netflix/archaius/wiki/Getting-Started) such as 

```
-Darchaius.configurationSource.additionalUrls=file:///path/to/config.properties
```

You can test that Turbine is correctly accessing instances and streaming metrics like this:

```
curl http://hostname:port/turbine/turbine.stream
```

If successful you should see something like this:

```
$ curl http://ec2-23-20-84-255.compute-1.amazonaws.com:8080/turbine/turbine.stream
: ping
data: {"rollingCountFailure":0,"propertyValue_executionIsolationThreadInterruptOnTimeout":true,"rollingCountTimeout":0,"rollingCountExceptionsThrown":0,"rollingCountFallbackSuccess":0,"errorCount":0,"type":"HystrixCommand","propertyValue_circuitBreakerEnabled":true,"reportingHosts":1,"latencyTotal":{"0":0,"95":0,"99.5":0,"90":0,"25":0,"99":0,"75":0,"100":0,"50":0},"currentConcurrentExecutionCount":0,"rollingCountSemaphoreRejected":0,"rollingCountFallbackRejection":0,"rollingCountShortCircuited":0,"rollingCountResponsesFromCache":0,"propertyValue_circuitBreakerForceClosed":false,"name":"IdentityCookieAuthSwitchProfile","propertyValue_executionIsolationThreadPoolKeyOverride":"null","rollingCountSuccess":0,"propertyValue_requestLogEnabled":true,"requestCount":0,"rollingCountCollapsedRequests":0,"errorPercentage":0,"propertyValue_circuitBreakerSleepWindowInMilliseconds":5000,"latencyTotal_mean":0,"propertyValue_circuitBreakerForceOpen":false,"propertyValue_circuitBreakerRequestVolumeThreshold":20,"propertyValue_circuitBreakerErrorThresholdPercentage":50,"propertyValue_executionIsolationStrategy":"THREAD","rollingCountFallbackFailure":0,"isCircuitBreakerOpen":false,"propertyValue_executionIsolationSemaphoreMaxConcurrentRequests":20,"propertyValue_executionIsolationThreadTimeoutInMilliseconds":1000,"propertyValue_metricsRollingStatisticalWindowInMilliseconds":10000,"propertyValue_fallbackIsolationSemaphoreMaxConcurrentRequests":10,"latencyExecute":{"0":0,"95":0,"99.5":0,"90":0,"25":0,"99":0,"75":0,"100":0,"50":0},"group":"IDENTITY","latencyExecute_mean":0,"propertyValue_requestCacheEnabled":true,"rollingCountThreadPoolRejected":0}
```

# Using

When you access the Hystrix Dashboard homepage you should see something like this:

[[images/dashboard-home-640.png]]

To monitor a single server you would use a URL such as:

```
http://hostname:port/application/hystrix.stream
```

To monitor an aggregate stream via Turbine it would be like:

```
http://hostname:port/turbine/turbine.stream
```

The landing page does nothing more than generate the /monitor/monitor.html URLs that can then be bookmarked.

The 'delay' parameter controls the latency that is injected between polling cycles on the server to slow down the stream. This can be used to reduce the network and CPU usage on the client.

The 'title' parameter is used by the monitor.html page to display a nice title instead of the raw URL.

# Customizing and Embedding

We expect that many will want to embed the dashboard functionality into their own existing dashboards.

To accommodate this we have kept the app very simple - primary just HTML, Javascript and CSS in modules that can be dropped into any app.

The only portion that is server side is a [proxy servlet](https://github.com/Netflix/Hystrix/blob/master/hystrix-dashboard/src/main/java/com/netflix/hystrix/dashboard/stream/ProxyStreamServlet.java) used to proxy streams between the browser and backend since EventSource CORS support is still [a work in progress](https://bugs.webkit.org/show_bug.cgi?id=61862).

Displaying HystrixCommand monitors on an existing page is as simple as importing the javascript module, instantiating it with a DIV to use and giving it an EventStream:

```javascript
var hystrixMonitor = new HystrixCommandMonitor('dependencies', {includeDetailIcon:false});
// start the EventSource which will open a streaming connection to the server
var source = new EventSource("http://hostname:port/hystrix.stream");
// add the listener that will process incoming events
source.addEventListener('message', hystrixMonitor.eventSourceMessageListener, false);
```

If you have UI improvements that you feel would benefit everyone please create a pull request and contribute back to the project and feel free to [ask questions and file bugs](https://github.com/Netflix/Hystrix/issues).