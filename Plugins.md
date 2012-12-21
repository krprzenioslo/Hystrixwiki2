Hystrix supports the modification or addition of behavior using plugin implementations.

They can be registered using the [HystrixPlugins](http://netflix.github.com/Hystrix/javadoc/index.html?com/netflix/hystrix/strategy/HystrixPlugins.html) service and will then apply to all [HystrixCommand](http://netflix.github.com/Hystrix/javadoc/index.html?com/netflix/hystrix/HystrixCommand.html) and [HystrixCollapser](http://netflix.github.com/Hystrix/javadoc/index.html?com/netflix/hystrix/HystrixCollapser.html) implementations and override all others.

## Plugin Types

Following are introductions to each of the different plugins that can be implemented ([Javadocs](http://netflix.github.com/Hystrix/javadoc/index.html) contain more detail):

### Event Notifier

As events occur during HystrixCommand execution they are trigged on the [HystrixEventNotifier](http://netflix.github.com/Hystrix/javadoc/index.html?com/netflix/hystrix/strategy/eventnotifier/HystrixEventNotifier.html) to give an opportunity for alerting and statistics.


### Metrics Publisher

Each instance of metrics being captured (such as for all HystrixCommands with a given HystrixCommandKey) will ask the [HystrixMetricsPublisher](http://netflix.github.com/Hystrix/javadoc/index.html?com/netflix/hystrix/strategy/metrics/HystrixMetricsPublisher.html) for an implementation and initialize it. 

This gives the implementation the opportunity to receive the metrics data objects and start a background process for doing something with the metrics such as publishing them to a persistent store.

The default implementation publishes them to [Servo](https://github.com/Netflix/servo) which is an in-memory system that supports various mechanisms of retrieving the data such as through pollers or JMX.


### Properties Strategy

Implementing a custom [HystrixPropertiesStrategy](http://netflix.github.com/Hystrix/javadoc/index.html?com/netflix/hystrix/strategy/properties/HystrixPropertiesStrategy.html) gives full control over how properties are defined for the system.

The default implementation uses [Archaius](https://github.com/Netflix/archaius).


### Concurrency Strategy

Hystrix uses implementations of ThreadLocal, Callable, Runnable, ThreadPoolExecutor, BlockingQueue as part the thread isolation and request scoped functionality. 

By default Hystrix has implementations that work "out of the box" but many environments (including Netflix) desire the use of alternatives so this plugin allows injecting custom implementations or decorating behavior.

The [HystrixConcurrencyStrategy](http://netflix.github.com/Hystrix/javadoc/index.html?com/netflix/hystrix/strategy/concurrency/HystrixConcurrentStrategy.html) class can be implemented with the following:

* The _getThreadPool()_ and _getBlockingQueue()_ methods are straight-forward options to inject the implementation of your choice, or just a decorated version with extra logging and metrics.

* The _wrapCallable()_ method allows you to decorate every Callable executed by Hystrix. This can be essential to systems that rely upon ThreadLocal state for application functionality. The wrapping Callable can capture and copy state from parent to child thread as needed.

* The _getRequestVariable()_ method expects an implementation of [HystrixRequestVariable<T>](http://netflix.github.com/Hystrix/javadoc/index.html?com/netflix/hystrix/strategy/concurrency/HystrixRequestVariable.html) that functions like a ThreadLocal except scoped to the request - available on all threads within the request. Generally it will be easier and sufficient to just use the [HystrixRequestContext](http://netflix.github.com/Hystrix/javadoc/index.html?com/netflix/hystrix/strategy/concurrency/HystrixRequestContext.html) with its own default implementation of [HystrixRequestVariable](http://netflix.github.com/Hystrix/javadoc/index.html?com/netflix/hystrix/strategy/concurrency/HystrixRequestVariable.html).


## How to Use

Registering a plugin globally looks like this:

```java
HystrixPlugins.getInstance().registerEventNotifier(ACustomHystrixEventNotifierDefaultStrategy.getInstance());
```

## Abstract vs Interface

Each of the plugins shown above expose the base as an abstract class to be extended rather than an interface to be implemented.

This is done for 2 reasons:

#### 1) Default Behavior

Only methods needing to be customized need be overridden. Default behavior can be left on other methods by not touching them.

Each method implementation is intended to be stand-alone and not have side-effects or state so overriding one method without another should not have implications.

Extension is accomplished via abstract base classes primarily for the next reason ...

#### 2) Library Maintenance

Each of these plugins can easily end up having methods added to them over time as new functionality is added or new things found needing to be customized by users.

If an interface was used it would either require a new interface each time (basically an interface per method) or would necessitate breaking changes for anybody who has implemented the interface. This problem will go away in Java 8 when 'default methods' will exist on interfaces, but it is a long ways until that is available and this library chooses Java 8 as the minimum supported JDK version.

Thus, the use of abstracts means new optional methods can be added as point releases instead of major releases without breaking existing implementations.