The Hystrix Dashboard allows you to monitor Hystrix metrics in real time.

When Netflix began to use this dashboard, their operations improved by reducing the time needed to discover and recover from operational events. The duration of most production incidents (already less frequent due to Hystrix) became far shorter, with diminished impact, due to the real-time insights into system behavior provided by the Hystrix Dashboard.

<a href="images/hystrix-dashboard-single-row.png">[[images/hystrix-dashboard-single-row-640.png]]</a>
_(Click for larger view.)_

When a &ldquo;circuit&rdquo; is failing it changes colors (on a gradient from green through yellow, orange, and red) like this: 

[[images/dashboard-example-open-circuit-640.png]]

The diagram below shows one circuit from the dashboard along with explanations of what all of the data represents.

Hystrix packs a lot of information into the dashboard so that engineers can quickly consume and correlate data.

[[images/dashboard-annoted-circuit-640.png]]

The Hystrix Dashboard allows you to monitor a single server or a cluster of servers aggregated using <a href="https://github.com/Netflix/Turbine">Turbine</a>, with low latency (typically around 1 or 2 seconds when aggregating a cluster, subsecond with a single server).

[[images/dashboard-direct-vs-turbine-640.png]]

Here is another example from the Netflix API dashboard monitoring 476 servers aggregated using <a href="https://github.com/Netflix/Turbine">Turbine</a>:

<a href="images/dashboard-example-1280.png">[[images/dashboard-example-640.png]]</a>
_(Click for larger view.)_

# Run via Gradle

Here is how to run the dashboard by issuing a Gradle command:

```
$ git clone git@github.com:Netflix/Hystrix.git
$ cd Hystrix/hystrix-dashboard
$ ../gradlew jettyRun
> Building > :hystrix-dashboard:jettyRun > Running at http://localhost:7979/hystrix-dashboard
```

Once you see that the dashboard has reached the &ldquo;Running&rdquo; state, open <a href="http://localhost:7979/hystrix-dashboard">http://localhost:7979/hystrix-dashboard</a>.

# Installing the Dashboard

### Download

1. Download <a href="http://search.maven.org/#browse%7C1045347652">hystrix-dashboard-#.#.#.war</a>  
1. Install it in a servlet container such as <a href="http://tomcat.apache.org/download-70.cgi">Apache Tomcat 7</a>

The usage examples below will assume that you install it into `/webapps/hystrix-dashboard.war`

### Build

To build the Hystrix Dashboard with Gradle and then install it into the servlet container, issue the following commands:

```
./gradlew build
cp hystrix-dashboard/build/libs/hystrix-dashboard-*.war ./apache-tomcat-7.*/webapps/hystrix-dashboard.war  (or other servlet container)
```

# Installation of Metrics Stream

The [`hystrix-metrics-event-stream`](https://github.com/Netflix/Hystrix/tree/master/hystrix-contrib/hystrix-metrics-event-stream) module exposes metrics in a [text/event-stream](https://developer.mozilla.org/en-US/docs/Server-sent_events/Using_server-sent_events) formatted stream that continues as long as a client holds the connection.

The Hystrix Dashboard expects data in the format that this module emits.

See its [README](https://github.com/Netflix/Hystrix/blob/master/hystrix-contrib/hystrix-metrics-event-stream/README.md) for installation instructions.

# Installation of Turbine (optional)

### Download

1. Download <a href="https://github.com/downloads/Netflix/Turbine/turbine-web-1.0.0.war">turbine-web-1.0.0.war</a>  
1. Install in servlet container such as <a href="http://tomcat.apache.org/download-70.cgi">Apache Tomcat 7</a>

The usage examples below will assume that you install it to `/webapps/turbine.war`

### Build

To build Turbine with Gradle and then install it into the servlet container, issue the following commands:

```
git clone git://github.com/Netflix/Turbine.git
./gradlew build
cp turbine-web/build/libs/turbine-web-*.war ./apache-tomcat-7.*/webapps/turbine.war  (or other servlet container)
```

### Configure Hosts Discovery

You can find Turbine configuration details on its [Configuration Wiki](https://github.com/Netflix/Turbine/wiki/Configuration-(1.x)). It also supports custom plugins for [Instance Discovery](https://github.com/Netflix/Turbine/wiki/Plugging-in-your-own-InstanceDiscovery-(1.x)).

To get started as a &ldquo;Hello World!&rdquo; example, you can use a static configuration file pointing to specific instances, such as the following.

Create a file, `config.properties`, that lists hosts to aggregate. This example includes two EC2 instances:

```
turbine.ConfigPropertyBasedDiscovery.default.instances=ec2-72-44-38-203.compute-1.amazonaws.com,ec2-23-20-84-255.compute-1.amazonaws.com
turbine.instanceUrlSuffix=:8080/hystrix.stream
```

The value of the `turbine.instanceUrlSuffix` property will be appended to each hostname to create a URL that will result in the [`hystrix-metrics-event-stream`](https://github.com/Netflix/Hystrix/tree/master/hystrix-contrib/hystrix-metrics-event-stream).

The `config.properties` file can be:

- placed on the classpath such as in `/WEB-INF/classes`
- specified using a [JVM property](https://github.com/Netflix/archaius/wiki/Getting-Started) such as 

```
-Darchaius.configurationSource.additionalUrls=file:///path/to/config.properties
```

You can test that Turbine is correctly accessing instances and streaming metrics by issuing a command like this:

```
curl http://hostname:port/turbine/turbine.stream
```

If that command is successful you should see something like this:

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

To monitor an aggregate stream via Turbine the URL would be like this:

```
http://hostname:port/turbine/turbine.stream
```

The landing page does nothing more than generate the `/monitor/monitor.html` URLs that you can then bookmark.

The `delay` parameter controls the latency that is injected between polling cycles on the server to slow down the stream. You can use this to reduce the network and CPU usage on the client.

The `title` parameter is used by the `monitor.html` page to display a nice title in the browser instead of the raw URL.

# Customizing and Embedding

Because you may want to embed the dashboard functionality into your own existing dashboard, the app is very simple &mdash; primary just HTML, Javascript, and CSS, in modules that can be dropped into any app.

The only portion that is server-side is a [proxy servlet](https://github.com/Netflix/Hystrix/blob/master/hystrix-dashboard/src/main/java/com/netflix/hystrix/dashboard/stream/ProxyStreamServlet.java) that proxies streams between the browser and back end, since EventSource CORS support [was a work in progress](https://bugs.webkit.org/show_bug.cgi?id=61862) at development-time.

To display `HystrixCommand` monitors on an existing page, simply import the javascript module, instantiate it with a `div`, and give it an `EventStream`, like this:

```javascript
var hystrixMonitor = new HystrixCommandMonitor('dependencies', {includeDetailIcon:false});
// start the EventSource which will open a streaming connection to the server
var source = new EventSource("http://hostname:port/hystrix.stream");
// add the listener that will process incoming events
source.addEventListener('message', hystrixMonitor.eventSourceMessageListener, false);
```

If you have UI improvements that you feel would benefit everyone please create a pull request and contribute back to the project and feel free to [ask questions and file bugs](https://github.com/Netflix/Hystrix/issues).