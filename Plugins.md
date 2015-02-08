You can modify the behavior of Hystrix, or add additional behavior to it, by implementing plugins.

You register these plugins by means of the [HystrixPlugins](http://netflix.github.com/Hystrix/javadoc/index.html?com/netflix/hystrix/strategy/HystrixPlugins.html) service. Hystrix will then apply them to all [HystrixCommand](http://netflix.github.com/Hystrix/javadoc/index.html?com/netflix/hystrix/HystrixCommand.html) and [HystrixCollapser](http://netflix.github.com/Hystrix/javadoc/index.html?com/netflix/hystrix/HystrixCollapser.html) implementations, overriding all others.

## Plugin Types

Following are introductions to each of the different plugins that you can implement ([the Javadocs](http://netflix.github.com/Hystrix/javadoc/index.html) contain more detail):

### Event Notifier

Events that occur during `HystrixCommand` execution are trigged on the [HystrixEventNotifier](http://netflix.github.com/Hystrix/javadoc/index.html?com/netflix/hystrix/strategy/eventnotifier/HystrixEventNotifier.html) to give an opportunity for alerting and statistics-collection.

### Metrics Publisher

Each instance of metrics being captured (such as for all `HystrixCommands` with a given `HystrixCommandKey`) will ask the [HystrixMetricsPublisher](http://netflix.github.com/Hystrix/javadoc/index.html?com/netflix/hystrix/strategy/metrics/HystrixMetricsPublisher.html) for an implementation and initialize it.

This gives the implementation the opportunity to receive the metrics data objects and start a background process for doing something with the metrics such as publishing them to a persistent store.

The default implementation publishes them to [Servo](https://github.com/Netflix/servo) which is an in-memory system that supports various mechanisms of retrieving the data such as through pollers or JMX.

### Properties Strategy

If you implement a custom [HystrixPropertiesStrategy](http://netflix.github.com/Hystrix/javadoc/index.html?com/netflix/hystrix/strategy/properties/HystrixPropertiesStrategy.html), this gives you full control over how properties are defined for the system.

The default implementation uses [Archaius](https://github.com/Netflix/archaius).

### Concurrency Strategy

Hystrix uses implementations of `ThreadLocal`, `Callable`, `Runnable`, `ThreadPoolExecutor`, and `BlockingQueue` as part its thread isolation and request-scoped functionality. 

By default Hystrix has implementations that work &ldquo;out of the box&rdquo; but many environments (including Netflix) desire the use of alternatives so this plugin allows injecting custom implementations or decorating behavior.

You can implement the [HystrixConcurrencyStrategy](http://netflix.github.com/Hystrix/javadoc/index.html?com/netflix/hystrix/strategy/concurrency/HystrixConcurrentStrategy.html) class with the following:

* The `getThreadPool()` and `getBlockingQueue()` methods are straightforward options that inject the implementation of your choice, or just a decorated version with extra logging and metrics.

* The `wrapCallable()` method allows you to decorate every `Callable` executed by Hystrix. This can be essential to systems that rely upon `ThreadLocal` state for application functionality. The wrapping `Callable` can capture and copy state from parent to child thread as needed.

* The `getRequestVariable()` method expects an implementation of [HystrixRequestVariable<T>](http://netflix.github.com/Hystrix/javadoc/index.html?com/netflix/hystrix/strategy/concurrency/HystrixRequestVariable.html) that functions like a `ThreadLocal` except scoped to the request &mdash; available on all threads within the request. Generally it will be easier and sufficient to just use the [HystrixRequestContext](http://netflix.github.com/Hystrix/javadoc/index.html?com/netflix/hystrix/strategy/concurrency/HystrixRequestContext.html) with its own default implementation of [HystrixRequestVariable](http://netflix.github.com/Hystrix/javadoc/index.html?com/netflix/hystrix/strategy/concurrency/HystrixRequestVariable.html).

### Command Execution Hook

A [HystrixCommandExecutionHook](http://netflix.github.com/Hystrix/javadoc/index.html?com/netflix/hystrix/strategy/executionhook/HystrixCommandExecutionHook.html) implementation gets access to the execution lifecycle of a [HystrixCommand](http://netflix.github.com/Hystrix/javadoc/index.html?com/netflix/hystrix/HystrixCommand.html) in order to inject behavior, logging, override responses, alter thread state, etc.

## How to Use

This is how you register a plugin globally:

```java
HystrixPlugins.getInstance().registerEventNotifier(ACustomHystrixEventNotifierDefaultStrategy.getInstance());
```

## Abstract vs. Interface

Each of the plugins shown above exposes the base as an abstract class to be extended rather than as an interface to be implemented.

This is for 2 reasons:

#### 1) Default Behavior

You only need to override those methods that you need to customize. You can retain the default behavior of other methods by not altering them.

Each method implementation is intended to be stand-alone and not have side effects or state so when you override one method without changing another this should not have implications.

Extension is accomplished via abstract base classes primarily for the next reason...

#### 2) Library Maintenance

As Hystrix evolves, each of these plugins may gain new methods as new functionality is added or new things are found that need to be customized by users.

If Hystrix used an interface for this purpose, it would either require a new interface each time new functionality was added (basically an interface per method) or it would necessitate breaking changes for anybody who has implemented the interface. This problem will go away in Java 8 when &ldquo;default methods&rdquo; will exist on interfaces, but it will be a while before that is available and this library can choose Java 8 as the minimum supported JDK version.

Hystrix uses abstract base classes so that new optional methods can be added in point releases instead of major releases without breaking existing implementations.