## Contents

1. <a href="#Hello-World">&ldquo;Hello World!&rdquo;</a>
1. <a href="#Synchronous-Execution">Synchronous Execution</a>
1. <a href="#Asynchronous-Execution">Asynchronous Execution</a>
1. <a href="#Reactive-Execution">Reactive Execution</a>
1. <a href="#Reactive-Commands">Reactive Commands</a>
1. <a href="#Fallback">Fallback</a>
1. <a href="#ErrorPropagation">Error Propagation</a>
1. <a href="#CommandName">Command Name</a>
1. <a href="#CommandGroup">Command Group</a>
1. <a href="#CommandThreadPool">Command Thread-Pool</a>
1. <a href="#Caching">Request Cache</a>
1. <a href="#Collapsing">Request Collapsing</a>
1. <a href="#RequestContextSetup">Request Context Setup</a>
1. <a href="#Common-Patterns">Common Patterns:</a>
  1. <a href="#Common-Patterns-FailFast">Fail Fast</a>
  1. <a href="#Common-Patterns-FailSilent">Fail Silent</a>
  1. <a href="#Common-Patterns-FallbackStatic">Fallback: Static</a>
  1. <a href="#Common-Patterns-FallbackStubbed">Fallback: Stubbed</a>
  1. <a href="#Common-Patterns-FallbackCacheViaNetwork">Fallback: Cache via Network</a>
  1. <a href="#Common-Patterns-PrimarySecondaryWithFallback">Primary + Secondary with Fallback</a>
  1. <a href="#Common-Patterns-Sempahore">Client Doesn&#8217;t Perform Network Access</a>
  1. <a href="#Common-Patterns-GetSetGet">Get-Set-Get with Request Cache Invalidation</a>
1. <a href="#MigratingLibrary">Migrating a Library to Hystrix</a>

<a name='Hello-World'/>
## Hello World!

The following is a basic &ldquo;Hello World&rdquo; implementation of a [HystrixCommand](http://netflix.github.com/Hystrix/javadoc/index.html?com/netflix/hystrix/HystrixCommand.html):

```java
public class CommandHelloWorld extends HystrixCommand<String> {

    private final String name;

    public CommandHelloWorld(String name) {
        super(HystrixCommandGroupKey.Factory.asKey("ExampleGroup"));
        this.name = name;
    }

    @Override
    protected String run() {
        // a real example would do work like a network call here
        return "Hello " + name + "!";
    }
}
```
[View Source](../blob/master/hystrix-examples/src/main/java/com/netflix/hystrix/examples/basic/CommandHelloWorld.java)

<a name='Synchronous-Execution'/>
## Synchronous Execution

You can execute Hystrix commands synchronously with the [execute()](<http://netflix.github.com/Hystrix/javadoc/index.html?com/netflix/hystrix/HystrixCommand.html#execute\(\)>) method, as in the following example:

```java
String s = new CommandHelloWorld("World").execute();
```

Execution of this form passes the following tests:

```java
        @Test
        public void testSynchronous() {
            assertEquals("Hello World!", new CommandHelloWorld("World").execute());
            assertEquals("Hello Bob!", new CommandHelloWorld("Bob").execute());
        }
```

<a name='Asynchronous-Execution'/>
## Asynchronous Execution

You can execute a `HystrixCommand` asynchronously by using the [queue()](<http://netflix.github.com/Hystrix/javadoc/com/netflix/hystrix/HystrixCommand.html#queue\(\)>) method, as in the following example:

```java
Future<String> fs = new CommandHelloWorld("World").queue();
```

You can retrieve the result of the command by using the Future:

```java
String s = fs.get();
```

The following unit tests demonstrate the behavior:

```java
        @Test
        public void testAsynchronous1() throws Exception {
            assertEquals("Hello World!", new CommandHelloWorld("World").queue().get());
            assertEquals("Hello Bob!", new CommandHelloWorld("Bob").queue().get());
        }

        @Test
        public void testAsynchronous2() throws Exception {

            Future<String> fWorld = new CommandHelloWorld("World").queue();
            Future<String> fBob = new CommandHelloWorld("Bob").queue();

            assertEquals("Hello World!", fWorld.get());
            assertEquals("Hello Bob!", fBob.get());
        }
```

The following are equivalent to each other:

```java
String s1 = new CommandHelloWorld("World").execute();
String s2 = new CommandHelloWorld("World").queue().get();
```

<a name='Reactive-Execution'/>
## Reactive Execution

You can also observe the results of a `HystrixCommand` as an `Observable` by using one of the following methods:
* [observe()](http://netflix.github.com/Hystrix/javadoc/index.html?com/netflix/hystrix/HystrixCommand.html#observe\(\)) &mdash; returns a &ldquo;hot&rdquo; Observable that executes the command immediately, though because the Observable is filtered through a `ReplaySubject` you are not in danger of losing any items that it emits before you have a chance to subscribe
* [toObservable()](http://netflix.github.com/Hystrix/javadoc/index.html?com/netflix/hystrix/HystrixCommand.html#toObservable\(\)) &mdash; returns a &ldquo;cold&rdquo; Observable that won&#8217;t execute the command and begin emitting its results until you subscribe to the Observable

```java
Observable<String> ho = new CommandHelloWorld("World").observe();
// or Observable<String> co = new CommandHelloWorld("World").toObservable();
```

You then retrieve the value of the command by subscribing to the Observable:

```java
ho.subscribe(new Action1<String>() {

    @Override
    public void call(String s) {
         // value emitted here
    }

});
```

The following unit tests demonstrate the behavior:

```java
@Test
public void testObservable() throws Exception {

    Observable<String> fWorld = new CommandHelloWorld("World").observe();
    Observable<String> fBob = new CommandHelloWorld("Bob").observe();

    // blocking
    assertEquals("Hello World!", fWorld.toBlockingObservable().single());
    assertEquals("Hello Bob!", fBob.toBlockingObservable().single());

    // non-blocking 
    // - this is a verbose anonymous inner-class approach and doesn't do assertions
    fWorld.subscribe(new Observer<String>() {

        @Override
        public void onCompleted() {
            // nothing needed here
        }

        @Override
        public void onError(Throwable e) {
            e.printStackTrace();
        }

        @Override
        public void onNext(String v) {
            System.out.println("onNext: " + v);
        }

    });

    // non-blocking
    // - also verbose anonymous inner-class
    // - ignore errors and onCompleted signal
    fBob.subscribe(new Action1<String>() {

        @Override
        public void call(String v) {
            System.out.println("onNext: " + v);
        }

    });
}
```

Using Java 8 lambdas/closures is more compact; it would look like this:

```java
    fWorld.subscribe((v) -> {
        System.out.println("onNext: " + v);
    })
    
    // - or while also including error handling
    
    fWorld.subscribe((v) -> {
        System.out.println("onNext: " + v);
    }, (exception) -> {
        exception.printStackTrace();
    })
```

More information about Observable can be found at http://reactivex.io/documentation/observable.html

<a name='Reactive-Commands' />
## Reactive Commands

Rather than converting a `HystrixCommand` into an `Observable` using the methods described above, you can also create a [`HystrixObservableCommand`](http://netflix.github.com/Hystrix/javadoc/index.html?com/netflix/hystrix/HystrixObservableCommand.java) that is a specialized version of `HystrixCommand` meant to wrap Observables. A `HystrixObservableCommand` is capable of wrapping Observables that emit multiple items, whereas ordinary `HystrixCommands`, even when converted into Observables, will never emit more than one item.

In such a case, instead of overriding the `run` method with your command logic (as you would with an ordinary `HystrixCommand`), you would override the `construct` method so that it returns the Observable you intend to wrap.

<a name='Fallback'/>
## Fallback

You can support graceful degradation by adding a [getFallback()](http://netflix.github.com/Hystrix/javadoc/index.html?com/netflix/hystrix/HystrixCommand.html#getFallback\(\)) implementation to an ordinary `HystrixCommand`. Hystrix will execute this fallback for all types of failure such as [run()](http://netflix.github.com/Hystrix/javadoc/index.html?com/netflix/hystrix/HystrixCommand.html#run\(\)) failure, timeout, thread pool or semaphore rejection, and circuit-breaker short-circuiting. The following example includes such a fallback:

```java
public class CommandHelloFailure extends HystrixCommand<String> {

    private final String name;

    public CommandHelloFailure(String name) {
        super(HystrixCommandGroupKey.Factory.asKey("ExampleGroup"));
        this.name = name;
    }

    @Override
    protected String run() {
        throw new RuntimeException("this command always fails");
    }

    @Override
    protected String getFallback() {
        return "Hello Failure " + name + "!";
    }
}
```
[View Source](../blob/master/hystrix-examples/src/main/java/com/netflix/hystrix/examples/basic/CommandHelloFailure.java)

This command&#8217;s [run()](http://netflix.github.com/Hystrix/javadoc/index.html?com/netflix/hystrix/HystrixCommand.html#run\(\)) method will fail on every execution. However, the caller will always receive the value returned by the command&#8217;s [getFallback()](http://netflix.github.com/Hystrix/javadoc/index.html?com/netflix/hystrix/HystrixCommand.html#getFallback\(\)) method instead of receiving an exception:

```java
    @Test
    public void testSynchronous() {
        assertEquals("Hello Failure World!", new CommandHelloFailure("World").execute());
        assertEquals("Hello Failure Bob!", new CommandHelloFailure("Bob").execute());
    }
```

For a `HystrixObservableCommand` you instead may override the `resumeWithFallback` method so that it returns a second Observable that will be subscribed to if the first one fails. Note that because an Observable may fail after having already emitted one or more values, your fallback should not assume that it will be emitting the only values that the resulting `HystrixObservableCommand` emits.

<a name='ErrorPropagation'/>
## Error Propagation

All exceptions thrown from the [run()](http://netflix.github.com/Hystrix/javadoc/index.html?com/netflix/hystrix/HystrixCommand.html#run\(\)) method except for [HystrixBadRequestException](http://netflix.github.io/Hystrix/javadoc/com/netflix/hystrix/exception/HystrixBadRequestException.html) count as failures and trigger [getFallback()](http://netflix.github.com/Hystrix/javadoc/index.html?com/netflix/hystrix/HystrixCommand.html#getFallback\(\)) and circuit-breaker logic.

You can wrap the exception that you would like to throw in ```HystrixBadRequestException``` and retrieve it via ```getCause()```. The [HystrixBadRequestException](http://netflix.github.io/Hystrix/javadoc/com/netflix/hystrix/exception/HystrixBadRequestException.html) is intended for use cases such as reporting illegal arguments or non-system failures that should not count against the failure metrics and should not trigger fallback logic.

In the case of a `HystrixObservableCommand`, non-recoverable errors are returned via `onError` notifications from the resulting Observable, and fallbacks are accomplished by falling back to a second Observable that Hystrix obtains through the `resumeWithFallback` method that you implement.

<a name='CommandName'/>
## Command Name

A command name is, by default, derived from the class name:

```java
getClass().getSimpleName();
```

To explicitly define the name pass it in via the [HystrixCommand](http://netflix.github.com/Hystrix/javadoc/index.html?com/netflix/hystrix/HystrixCommand.html) constructor:

```java
    public CommandHelloWorld(String name) {
        super(Setter.withGroupKey(HystrixCommandGroupKey.Factory.asKey("ExampleGroup"))
                .andCommandKey(HystrixCommandKey.Factory.asKey("HelloWorld")));
        this.name = name;
    }
```

[HystrixCommandKey](http://netflix.github.com/Hystrix/javadoc/index.html?com/netflix/hystrix/HystrixCommandKey.html) is an interface and can be implemented as an enum or regular class, but it also has the helper [Factory](http://netflix.github.com/Hystrix/javadoc/index.html?com/netflix/hystrix/HystrixCommandKey.Factory.html) class to construct and intern instances such as:

```java
HystrixCommandKey.Factory.asKey("HelloWorld")
```

<a name='CommandGroup'/>
## Command Group

Hystrix uses the command group key to group together commands such as for reporting, alerting, dashboards, or team/library ownership.

By default Hystrix uses this to define the command thread-pool unless a separate one is defined.

[HystrixCommandGroupKey](http://netflix.github.com/Hystrix/javadoc/index.html?com/netflix/hystrix/HystrixCommandGroupKey.html) is an interface and can be implemented as an enum or regular class, but it also has the helper [Factory](http://netflix.github.com/Hystrix/javadoc/index.html?com/netflix/hystrix/HystrixCommandGroupKey.Factory.html) class to construct and intern instances such as:

```java
HystrixCommandGroupKey.Factory.asKey("ExampleGroup")
```

<a name='CommandThreadPool'/>
## Command Thread-Pool

The thread-pool key represents a [HystrixThreadPool](http://netflix.github.com/Hystrix/javadoc/index.html?com/netflix/hystrix/HystrixThreadPool.html) for monitoring, metrics publishing, caching, and other such uses. A [HystrixCommand](http://netflix.github.com/Hystrix/javadoc/index.html?com/netflix/hystrix/HystrixCommand.html) is associated with a single [HystrixThreadPool](http://netflix.github.com/Hystrix/javadoc/index.html?com/netflix/hystrix/HystrixThreadPool.html) as retrieved by the [HystrixThreadPoolKey](http://netflix.github.com/Hystrix/javadoc/index.html?com/netflix/hystrix/HystrixThreadPoolKey.html) injected into it, or it defaults to one created using the [HystrixCommandGroupKey](http://netflix.github.com/Hystrix/javadoc/index.html?com/netflix/hystrix/HystrixCommandGroupKey.html) it is created with.

To explicitly define the name pass it in via the [HystrixCommand](http://netflix.github.com/Hystrix/javadoc/index.html?com/netflix/hystrix/HystrixCommand.html) constructor:

```java
    public CommandHelloWorld(String name) {
        super(Setter.withGroupKey(HystrixCommandGroupKey.Factory.asKey("ExampleGroup"))
                .andCommandKey(HystrixCommandKey.Factory.asKey("HelloWorld"))
                .andThreadPoolKey(HystrixThreadPoolKey.Factory.asKey("HelloWorldPool")));
        this.name = name;
    }
```

[HystrixThreadPoolKey](http://netflix.github.com/Hystrix/javadoc/index.html?com/netflix/hystrix/HystrixThreadPoolKey.html) is an interface and can be implemented as an enum or regular class, but it also has the helper [Factory](http://netflix.github.com/Hystrix/javadoc/index.html?com/netflix/hystrix/HystrixThreadPoolKey.Factory.html) class to construct and intern instances such as:

```java
HystrixThreadPoolKey.Factory.asKey("HelloWorldPool")
```

The reason why you might use [HystrixThreadPoolKey](http://netflix.github.com/Hystrix/javadoc/index.html?com/netflix/hystrix/HystrixThreadPoolKey.html) instead of just a different [HystrixCommandGroupKey](http://netflix.github.com/Hystrix/javadoc/index.html?com/netflix/hystrix/HystrixCommandGroupKey.html) is that multiple commands may belong to the same &ldquo;group&rdquo; of ownership or logical functionality, but certain commands may need to be isolated from each other.

Here is a simple example:

* two commands used to access Video metadata
* group name is &ldquo;VideoMetadata&rdquo;
* command A goes against resource #1
* command B goes against resource #2

If command A becomes latent and saturates its thread-pool it should not prevent command B from executing requests since they each hit different back-end resources.

Thus, we logically want these commands grouped together but want them isolated differently and would use [HystrixThreadPoolKey](http://netflix.github.com/Hystrix/javadoc/index.html?com/netflix/hystrix/HystrixThreadPoolKey.html) to give each of them a different thread-pool.

<a name='Caching'/>
## Request Cache

You enable request caching by implementing the [getCacheKey()](http://netflix.github.com/Hystrix/javadoc/index.html?com/netflix/hystrix/HystrixCommand.html#getCacheKey\(\)) method on a [HystrixCommand](http://netflix.github.com/Hystrix/javadoc/index.html?com/netflix/hystrix/HystrixCommand.html) object as follows:

```java
public class CommandUsingRequestCache extends HystrixCommand<Boolean> {

    private final int value;

    protected CommandUsingRequestCache(int value) {
        super(HystrixCommandGroupKey.Factory.asKey("ExampleGroup"));
        this.value = value;
    }

    @Override
    protected Boolean run() {
        return value == 0 || value % 2 == 0;
    }

    @Override
    protected String getCacheKey() {
        return String.valueOf(value);
    }
}
```
[View Source](../blob/master/hystrix-examples/src/main/java/com/netflix/hystrix/examples/basic/CommandUsingRequestCache.java)

Since this depends on request context we must initialize the [HystrixRequestContext](http://netflix.github.io/Hystrix/javadoc/com/netflix/hystrix/strategy/concurrency/HystrixRequestContext.html).

In a simple unit test you could do this as follows:

```java
        @Test
        public void testWithoutCacheHits() {
            HystrixRequestContext context = HystrixRequestContext.initializeContext();
            try {
                assertTrue(new CommandUsingRequestCache(2).execute());
                assertFalse(new CommandUsingRequestCache(1).execute());
                assertTrue(new CommandUsingRequestCache(0).execute());
                assertTrue(new CommandUsingRequestCache(58672).execute());
            } finally {
                context.shutdown();
            }
        }
```

Typically this context will be initialized and shut down via a `ServletFilter` that wraps a user request or some other lifecycle hook.

The following is an example that shows how commands retrieve their values from the cache (and how you can query an object to know whether its value came from the cache) within a request context:

```java
        @Test
        public void testWithCacheHits() {
            HystrixRequestContext context = HystrixRequestContext.initializeContext();
            try {
                CommandUsingRequestCache command2a = new CommandUsingRequestCache(2);
                CommandUsingRequestCache command2b = new CommandUsingRequestCache(2);

                assertTrue(command2a.execute());
                // this is the first time we've executed this command with
                // the value of "2" so it should not be from cache
                assertFalse(command2a.isResponseFromCache());

                assertTrue(command2b.execute());
                // this is the second time we've executed this command with
                // the same value so it should return from cache
                assertTrue(command2b.isResponseFromCache());
            } finally {
                context.shutdown();
            }

            // start a new request context
            context = HystrixRequestContext.initializeContext();
            try {
                CommandUsingRequestCache command3b = new CommandUsingRequestCache(2);
                assertTrue(command3b.execute());
                // this is a new request context so this 
                // should not come from cache
                assertFalse(command3b.isResponseFromCache());
            } finally {
                context.shutdown();
            }
        }
```

<a name='Collapsing'/>
## Request Collapsing

Request collapsing enables multiple requests to be batched into a single [HystrixCommand](http://netflix.github.com/Hystrix/javadoc/index.html?com/netflix/hystrix/HystrixCommand.html) instance execution.

A collapser can use the batch size and the elapsed time since the creation of the batch as triggers for executing a batch.

Following is a simple example of how to implement a [HystrixCollapser](http://netflix.github.com/Hystrix/javadoc/index.html?com/netflix/hystrix/HystrixCollapser.html):

```java
public class CommandCollapserGetValueForKey extends HystrixCollapser<List<String>, String, Integer> {

    private final Integer key;

    public CommandCollapserGetValueForKey(Integer key) {
        this.key = key;
    }

    @Override
    public Integer getRequestArgument() {
        return key;
    }

    @Override
    protected HystrixCommand<List<String>> createCommand(final Collection<CollapsedRequest<String, Integer>> requests) {
        return new BatchCommand(requests);
    }

    @Override
    protected void mapResponseToRequests(List<String> batchResponse, Collection<CollapsedRequest<String, Integer>> requests) {
        int count = 0;
        for (CollapsedRequest<String, Integer> request : requests) {
            request.setResponse(batchResponse.get(count++));
        }
    }

    private static final class BatchCommand extends HystrixCommand<List<String>> {
        private final Collection<CollapsedRequest<String, Integer>> requests;

        private BatchCommand(Collection<CollapsedRequest<String, Integer>> requests) {
                super(Setter.withGroupKey(HystrixCommandGroupKey.Factory.asKey("ExampleGroup"))
                    .andCommandKey(HystrixCommandKey.Factory.asKey("GetValueForKey")));
            this.requests = requests;
        }

        @Override
        protected List<String> run() {
            ArrayList<String> response = new ArrayList<String>();
            for (CollapsedRequest<String, Integer> request : requests) {
                // artificial response for each argument received in the batch
                response.add("ValueForKey: " + request.getArgument());
            }
            return response;
        }
    }
}
```
[View Source](../blob/master/hystrix-examples/src/main/java/com/netflix/hystrix/examples/basic/CommandCollapserGetValueForKey.java)

The following unit test shows how to use a collapser to automatically batch four executions of `CommandCollapserGetValueForKey` into a single [HystrixCommand](http://netflix.github.com/Hystrix/javadoc/index.html?com/netflix/hystrix/HystrixCommand.html) execution:

```java
@Test
public void testCollapser() throws Exception {
    HystrixRequestContext context = HystrixRequestContext.initializeContext();
    try {
        Future<String> f1 = new CommandCollapserGetValueForKey(1).queue();
        Future<String> f2 = new CommandCollapserGetValueForKey(2).queue();
        Future<String> f3 = new CommandCollapserGetValueForKey(3).queue();
        Future<String> f4 = new CommandCollapserGetValueForKey(4).queue();

        assertEquals("ValueForKey: 1", f1.get());
        assertEquals("ValueForKey: 2", f2.get());
        assertEquals("ValueForKey: 3", f3.get());
        assertEquals("ValueForKey: 4", f4.get());

        // assert that the batch command 'GetValueForKey' was in fact
        // executed and that it executed only once
        assertEquals(1, HystrixRequestLog.getCurrentRequest().getExecutedCommands().size());
        HystrixCommand<?> command = HystrixRequestLog.getCurrentRequest().getExecutedCommands().toArray(new HystrixCommand<?>[1])[0];
        // assert the command is the one we're expecting
        assertEquals("GetValueForKey", command.getCommandKey().name());
        // confirm that it was a COLLAPSED command execution
        assertTrue(command.getExecutionEvents().contains(HystrixEventType.COLLAPSED));
        // and that it was successful
        assertTrue(command.getExecutionEvents().contains(HystrixEventType.SUCCESS));
    } finally {
        context.shutdown();
    }
}
```


<a name='RequestContextSetup'/>
## Request Context Setup

To use request-scoped features (request caching, request collapsing, request log) you must manage the [HystrixRequestContext](http://netflix.github.com/Hystrix/javadoc/index.html?com/netflix/hystrix/strategy/concurrent/HystrixRequestContext.html) lifecycle (or implement an alternative [HystrixConcurrencyStrategy](http://netflix.github.com/Hystrix/javadoc/index.html?com/netflix/hystrix/strategy/concurrent/HystrixConcurrencyStrategy.html)).

This means that you must execute the following before a request:

```java
HystrixRequestContext context = HystrixRequestContext.initializeContext();
```

and then this at the end of the request:

```java
context.shutdown();
```

In a standard Java web application, you can use a Servlet Filter to initialize this lifecycle by implementing a filter similar to this:

```java
public class HystrixRequestContextServletFilter implements Filter {

    public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain) 
     throws IOException, ServletException {
        HystrixRequestContext context = HystrixRequestContext.initializeContext();
        try {
            chain.doFilter(request, response);
        } finally {
            context.shutdown();
        }
    }
}
```

You could enable the filter for all incoming traffic by adding a section to the `web.xml` as follows:

```
    <filter>
      <display-name>HystrixRequestContextServletFilter</display-name>
      <filter-name>HystrixRequestContextServletFilter</filter-name>
      <filter-class>com.netflix.hystrix.contrib.requestservlet.HystrixRequestContextServletFilter</filter-class>
    </filter>
    <filter-mapping>
      <filter-name>HystrixRequestContextServletFilter</filter-name>
      <url-pattern>/*</url-pattern>
   </filter-mapping>
```

<a name='Common-Patterns'/>
## Common Patterns

In the following sections are common uses and patterns of use for [HystrixCommand](http://netflix.github.com/Hystrix/javadoc/index.html?com/netflix/hystrix/HystrixCommand.html).

<a name='Common-Patterns-FailFast'/>
### Fail Fast

The most basic execution is one that does a single thing and has no fallback behavior. It will throw an exception if any type of failure occurs.

```java
public class CommandThatFailsFast extends HystrixCommand<String> {

    private final boolean throwException;

    public CommandThatFailsFast(boolean throwException) {
        super(HystrixCommandGroupKey.Factory.asKey("ExampleGroup"));
        this.throwException = throwException;
    }

    @Override
    protected String run() {
        if (throwException) {
            throw new RuntimeException("failure from CommandThatFailsFast");
        } else {
            return "success";
        }
    }
```
[View Source](../blob/master/hystrix-examples/src/main/java/com/netflix/hystrix/examples/basic/CommandThatFailsFast.java)

These unit tests show how it behaves:

```java
@Test
public void testSuccess() {
    assertEquals("success", new CommandThatFailsFast(false).execute());
}

@Test
public void testFailure() {
    try {
        new CommandThatFailsFast(true).execute();
        fail("we should have thrown an exception");
    } catch (HystrixRuntimeException e) {
        assertEquals("failure from CommandThatFailsFast", e.getCause().getMessage());
        e.printStackTrace();
    }
}
```
<a name='Common-Patterns-FailSilent'/>
### Fail Silent

Failing silently is the equivalent of returning an empty response or removing functionality. It can be done by returning `null`, an empty Map, empty List, or other such responses.

You do this by implementing a [getFallback()](http://netflix.github.com/Hystrix/javadoc/index.html?com/netflix/hystrix/HystrixCommand.html#getFallback\(\)) method on the [HystrixCommand](http://netflix.github.com/Hystrix/javadoc/index.html?com/netflix/hystrix/HystrixCommand.html) instance:

[[images/fallback-640.png]]

```java
public class CommandThatFailsSilently extends HystrixCommand<String> {

    private final boolean throwException;

    public CommandThatFailsSilently(boolean throwException) {
        super(HystrixCommandGroupKey.Factory.asKey("ExampleGroup"));
        this.throwException = throwException;
    }

    @Override
    protected String run() {
        if (throwException) {
            throw new RuntimeException("failure from CommandThatFailsFast");
        } else {
            return "success";
        }
    }

    @Override
    protected String getFallback() {
        return null;
    }
}
```
[View Source](../blob/master/hystrix-examples/src/main/java/com/netflix/hystrix/examples/basic/CommandThatFailsSilently.java)

```java
@Test
public void testSuccess() {
    assertEquals("success", new CommandThatFailsSilently(false).execute());
}

@Test
public void testFailure() {
    try {
        assertEquals(null, new CommandThatFailsSilently(true).execute());
    } catch (HystrixRuntimeException e) {
        fail("we should not get an exception as we fail silently with a fallback");
    }
}
```

Another implementation that returns an empty list would look like:

```java
    @Override
    protected List<String> getFallback() {
        return Collections.emptyList();
    }
```

<a name='Common-Patterns-FallbackStatic'/>
### Fallback: Static

Fallbacks can return default values statically embedded in code. This doesn&#8217;t cause the feature or service to be removed in the way that &ldquo;fail silent&rdquo; often does, but instead causes default behavior to occur.

For example, if a command returns a true/false based on user credentials but the command execution fails, it can default to true:

```java
    @Override
    protected Boolean getFallback() {
        return true;
    }
```

<a name='Common-Patterns-FallbackStubbed'/>
### Fallback: Stubbed

You typically use a stubbed fallback when your command returns a compound object containing multiple fields, some of which can be determined from other request state while other fields are set to default values.

Examples of places where you might find state appropriate to use in these stubbed values are:

* cookies
* request arguments and headers
* responses from previous service requests prior to the current one failing

Your fallback can retrieve stubbed values statically from the request scope, but typically it is recommended that they be injected at command instantiation time for use if they are needed such as this following example demonstrates in the way it treats the `countryCodeFromGeoLookup` field:

```java
public class CommandWithStubbedFallback extends HystrixCommand<UserAccount> {

    private final int customerId;
    private final String countryCodeFromGeoLookup;

    /**
     * @param customerId
     *            The customerID to retrieve UserAccount for
     * @param countryCodeFromGeoLookup
     *            The default country code from the HTTP request geo code lookup used for fallback.
     */
    protected CommandWithStubbedFallback(int customerId, String countryCodeFromGeoLookup) {
        super(HystrixCommandGroupKey.Factory.asKey("ExampleGroup"));
        this.customerId = customerId;
        this.countryCodeFromGeoLookup = countryCodeFromGeoLookup;
    }

    @Override
    protected UserAccount run() {
        // fetch UserAccount from remote service
        //        return UserAccountClient.getAccount(customerId);
        throw new RuntimeException("forcing failure for example");
    }

    @Override
    protected UserAccount getFallback() {
        /**
         * Return stubbed fallback with some static defaults, placeholders,
         * and an injected value 'countryCodeFromGeoLookup' that we'll use
         * instead of what we would have retrieved from the remote service.
         */
        return new UserAccount(customerId, "Unknown Name",
                countryCodeFromGeoLookup, true, true, false);
    }

    public static class UserAccount {
        private final int customerId;
        private final String name;
        private final String countryCode;
        private final boolean isFeatureXPermitted;
        private final boolean isFeatureYPermitted;
        private final boolean isFeatureZPermitted;

        UserAccount(int customerId, String name, String countryCode,
                boolean isFeatureXPermitted,
                boolean isFeatureYPermitted,
                boolean isFeatureZPermitted) {
            this.customerId = customerId;
            this.name = name;
            this.countryCode = countryCode;
            this.isFeatureXPermitted = isFeatureXPermitted;
            this.isFeatureYPermitted = isFeatureYPermitted;
            this.isFeatureZPermitted = isFeatureZPermitted;
        }
    }
}
```
[View Source](../blob/master/hystrix-examples/src/main/java/com/netflix/hystrix/examples/basic/CommandWithStubbedFallback.java)

The following unit test demonstrates its behavior:

```java
    @Test
    public void test() {
        CommandWithStubbedFallback command = new CommandWithStubbedFallback(1234, "ca");
        UserAccount account = command.execute();
        assertTrue(command.isFailedExecution());
        assertTrue(command.isResponseFromFallback());
        assertEquals(1234, account.customerId);
        assertEquals("ca", account.countryCode);
        assertEquals(true, account.isFeatureXPermitted);
        assertEquals(true, account.isFeatureYPermitted);
        assertEquals(false, account.isFeatureZPermitted);
    }
```

<a name='Common-Patterns-FallbackCacheViaNetwork'/>
### Fallback: Cache via Network

Sometimes if a back-end service fails, a stale version of data can be retrieved from a cache service such as memcached.

Since the fallback will go over the network it is another possible point of failure and so it also needs to be wrapped by a [HystrixCommand](http://netflix.github.com/Hystrix/javadoc/index.html?com/netflix/hystrix/HystrixCommand.html).

[[images/fallback-via-command-640.png]]

It is important to execute the fallback command on a separate thread-pool, otherwise if the main command were to become latent and fill the thread-pool, this would prevent the fallback from running if the two commands share the same pool.

The following code shows how `CommandWithFallbackViaNetwork` executes `FallbackViaNetwork` in its [getFallback()](http://netflix.github.com/Hystrix/javadoc/index.html?com/netflix/hystrix/HystrixCommand.html#getFallback\(\)) method.

Note how if the fallback fails, it _also_ has a fallback which does the &ldquo;fail silent&rdquo; approach of returning `null`.

To configure the `FallbackViaNetwork` command to run on a different threadpool than the default `RemoteServiceX` derived from the [HystrixCommandGroupKey](http://netflix.github.com/Hystrix/javadoc/index.html?com/netflix/hystrix/HystrixCommandGroupKey.html), it injects `HystrixThreadPoolKey.Factory.asKey("RemoteServiceXFallback")` into the constructor.

This means `CommandWithFallbackViaNetwork` will run on a thread-pool named `RemoteServiceX` and `FallbackViaNetwork` will run on a thread-pool named `RemoteServiceXFallback`.

```java
public class CommandWithFallbackViaNetwork extends HystrixCommand<String> {
    private final int id;

    protected CommandWithFallbackViaNetwork(int id) {
        super(Setter.withGroupKey(HystrixCommandGroupKey.Factory.asKey("RemoteServiceX"))
                .andCommandKey(HystrixCommandKey.Factory.asKey("GetValueCommand")));
        this.id = id;
    }

    @Override
    protected String run() {
        //        RemoteServiceXClient.getValue(id);
        throw new RuntimeException("force failure for example");
    }

    @Override
    protected String getFallback() {
        return new FallbackViaNetwork(id).execute();
    }

    private static class FallbackViaNetwork extends HystrixCommand<String> {
        private final int id;

        public FallbackViaNetwork(int id) {
            super(Setter.withGroupKey(HystrixCommandGroupKey.Factory.asKey("RemoteServiceX"))
                    .andCommandKey(HystrixCommandKey.Factory.asKey("GetValueFallbackCommand"))
                    // use a different threadpool for the fallback command
                    // so saturating the RemoteServiceX pool won't prevent
                    // fallbacks from executing
                    .andThreadPoolKey(HystrixThreadPoolKey.Factory.asKey("RemoteServiceXFallback")));
            this.id = id;
        }

        @Override
        protected String run() {
            MemCacheClient.getValue(id);
        }

        @Override
        protected String getFallback() {
            // the fallback also failed
            // so this fallback-of-a-fallback will 
            // fail silently and return null
            return null;
        }
    }
}
```
[View Source](../blob/master/hystrix-examples/src/main/java/com/netflix/hystrix/examples/basic/CommandWithFallbackViaNetwork.java)

<a name='Common-Patterns-PrimarySecondaryWithFallback'/>
### Primary + Secondary with Fallback

Some systems have dual-mode behavior &mdash; primary and secondary, or primary and failover.

Sometimes the secondary or failover is considered a failure state and it is intended only for fallback; in those scenarios it would fit in the same pattern as &ldquo;Cache via Network&rdquo; described above.

However, if flipping to the secondary system is common, such as a normal part of rolling out new code (sometimes this is part of how stateful systems handle code pushes) then every time the secondary system is used the primary will be in a failure state, tripping circuit breakers and firing alerts.

This is not the desired behavior, if for no other reason than to avoid the &ldquo;cry wolf&rdquo; fatigue that will cause alerts to be ignored when a real issue is occurring.

So in such a case the strategy is instead to treat the switching between primary and secondary as normal, healthy patterns and put a fa&ccedil;ade in front of them.

[[images/primary-secondary-example-640.png]]

The primary and secondary [HystrixCommand](http://netflix.github.com/Hystrix/javadoc/index.html?com/netflix/hystrix/HystrixCommand.html) implementations are thread-isolated since they are doing network traffic and business logic. They may each have very different performance characteristics (often the secondary system is a static cache) so another benefit of separate commands for each is that they can be individually tuned.

You do not expose these two commands publicly but you instead hide them behind another [HystrixCommand](http://netflix.github.com/Hystrix/javadoc/index.html?com/netflix/hystrix/HystrixCommand.html) that is semaphore-isolated and that implements the conditional logic as to whether to invoke the primary or secondary command. If both primary and secondary fail then control switches to the fallback of the fa&ccedil;ade command itself.

The fa&ccedil;ade [HystrixCommand](http://netflix.github.com/Hystrix/javadoc/index.html?com/netflix/hystrix/HystrixCommand.html) can use semaphore-isolation since all of the work it is doing is going through two other [HystrixCommands](http://netflix.github.com/Hystrix/javadoc/index.html?com/netflix/hystrix/HystrixCommand.html) that are already thread-isolated. It is unnecessary to have yet another layer of threading as long as the [run()](http://netflix.github.com/Hystrix/javadoc/index.html?com/netflix/hystrix/HystrixCommand.html#run\(\)) method of the fa&ccedil;ade is not doing any other network calls, retry logic, or other &ldquo;error prone&rdquo; things.

```java
public class CommandFacadeWithPrimarySecondary extends HystrixCommand<String> {

    private final static DynamicBooleanProperty usePrimary = DynamicPropertyFactory.getInstance().getBooleanProperty("primarySecondary.usePrimary", true);

    private final int id;

    public CommandFacadeWithPrimarySecondary(int id) {
        super(Setter
                .withGroupKey(HystrixCommandGroupKey.Factory.asKey("SystemX"))
                .andCommandKey(HystrixCommandKey.Factory.asKey("PrimarySecondaryCommand"))
                .andCommandPropertiesDefaults(
                        // we want to default to semaphore-isolation since this wraps
                        // 2 others commands that are already thread isolated
                        HystrixCommandProperties.Setter()
                                .withExecutionIsolationStrategy(ExecutionIsolationStrategy.SEMAPHORE)));
        this.id = id;
    }

    @Override
    protected String run() {
        if (usePrimary.get()) {
            return new PrimaryCommand(id).execute();
        } else {
            return new SecondaryCommand(id).execute();
        }
    }

    @Override
    protected String getFallback() {
        return "static-fallback-" + id;
    }

    @Override
    protected String getCacheKey() {
        return String.valueOf(id);
    }

    private static class PrimaryCommand extends HystrixCommand<String> {

        private final int id;

        private PrimaryCommand(int id) {
            super(Setter
                    .withGroupKey(HystrixCommandGroupKey.Factory.asKey("SystemX"))
                    .andCommandKey(HystrixCommandKey.Factory.asKey("PrimaryCommand"))
                    .andThreadPoolKey(HystrixThreadPoolKey.Factory.asKey("PrimaryCommand"))
                    .andCommandPropertiesDefaults(
                            // we default to a 600ms timeout for primary
                            HystrixCommandProperties.Setter().withExecutionTimeoutInMilliseconds(600)));
            this.id = id;
        }

        @Override
        protected String run() {
            // perform expensive 'primary' service call
            return "responseFromPrimary-" + id;
        }

    }

    private static class SecondaryCommand extends HystrixCommand<String> {

        private final int id;

        private SecondaryCommand(int id) {
            super(Setter
                    .withGroupKey(HystrixCommandGroupKey.Factory.asKey("SystemX"))
                    .andCommandKey(HystrixCommandKey.Factory.asKey("SecondaryCommand"))
                    .andThreadPoolKey(HystrixThreadPoolKey.Factory.asKey("SecondaryCommand"))
                    .andCommandPropertiesDefaults(
                            // we default to a 100ms timeout for secondary
                            HystrixCommandProperties.Setter().withExecutionTimeoutInMilliseconds(100)));
            this.id = id;
        }

        @Override
        protected String run() {
            // perform fast 'secondary' service call
            return "responseFromSecondary-" + id;
        }

    }

    public static class UnitTest {

        @Test
        public void testPrimary() {
            HystrixRequestContext context = HystrixRequestContext.initializeContext();
            try {
                ConfigurationManager.getConfigInstance().setProperty("primarySecondary.usePrimary", true);
                assertEquals("responseFromPrimary-20", new CommandFacadeWithPrimarySecondary(20).execute());
            } finally {
                context.shutdown();
                ConfigurationManager.getConfigInstance().clear();
            }
        }

        @Test
        public void testSecondary() {
            HystrixRequestContext context = HystrixRequestContext.initializeContext();
            try {
                ConfigurationManager.getConfigInstance().setProperty("primarySecondary.usePrimary", false);
                assertEquals("responseFromSecondary-20", new CommandFacadeWithPrimarySecondary(20).execute());
            } finally {
                context.shutdown();
                ConfigurationManager.getConfigInstance().clear();
            }
        }
    }
}
```
[View Source](../blob/master/hystrix-examples/src/main/java/com/netflix/hystrix/examples/basic/CommandFacadeWithPrimarySecondary.java)

<a name='Common-Patterns-Semaphore'/>
### Client Doesn't Perform Network Access

When you wrap behavior that does not perform network access, but where latency is a concern or the threading overhead is unacceptable, you can set the [executionIsolationStrategy](http://netflix.github.com/Hystrix/javadoc/index.html?com/netflix/hystrix/HystrixCommandProperties.html#executionIsolationStrategy\(\)) property to [ExecutionIsolationStrategy.SEMAPHORE](http://netflix.github.com/Hystrix/javadoc/index.html?com/netflix/hystrix/HystrixCommandProperties.ExecutionIsolationStrategy.html) and Hystrix will use semaphore isolation instead.

The following shows how to set this property as the default for a command via code (you can also override it via dynamic properties at runtime).

```java
public class CommandUsingSemaphoreIsolation extends HystrixCommand<String> {

    private final int id;

    public CommandUsingSemaphoreIsolation(int id) {
        super(Setter.withGroupKey(HystrixCommandGroupKey.Factory.asKey("ExampleGroup"))
                // since we're doing an in-memory cache lookup we choose SEMAPHORE isolation
                .andCommandPropertiesDefaults(HystrixCommandProperties.Setter()
                        .withExecutionIsolationStrategy(ExecutionIsolationStrategy.SEMAPHORE)));
        this.id = id;
    }

    @Override
    protected String run() {
        // a real implementation would retrieve data from in memory data structure
        return "ValueFromHashMap_" + id;
    }

}
```
[View Source](../blob/master/hystrix-examples/src/main/java/com/netflix/hystrix/examples/basic/CommandUsingSemaphoreIsolation.java)

<a name='Common-Patterns-GetSetGet'/>
### Get-Set-Get with Request Cache Invalidation

If you are implementing a Get-Set-Get use case where the Get receives enough traffic that request caching is desired but sometimes a Set occurs on another command that should invalidate the cache within the same request, you can invalidate the cache by calling [HystrixRequestCache.clear()](http://netflix.github.com/Hystrix/javadoc/index.html?com/netflix/hystrix/HystrixRequestCache.html).

Here is an example implementation:

```java
public class CommandUsingRequestCacheInvalidation {

    /* represents a remote data store */
    private static volatile String prefixStoredOnRemoteDataStore = "ValueBeforeSet_";

    public static class GetterCommand extends HystrixCommand<String> {

        private static final HystrixCommandKey GETTER_KEY = HystrixCommandKey.Factory.asKey("GetterCommand");
        private final int id;

        public GetterCommand(int id) {
            super(Setter.withGroupKey(HystrixCommandGroupKey.Factory.asKey("GetSetGet"))
                    .andCommandKey(GETTER_KEY));
            this.id = id;
        }

        @Override
        protected String run() {
            return prefixStoredOnRemoteDataStore + id;
        }

        @Override
        protected String getCacheKey() {
            return String.valueOf(id);
        }

        /**
         * Allow the cache to be flushed for this object.
         * 
         * @param id
         *            argument that would normally be passed to the command
         */
        public static void flushCache(int id) {
            HystrixRequestCache.getInstance(GETTER_KEY,
                    HystrixConcurrencyStrategyDefault.getInstance()).clear(String.valueOf(id));
        }

    }

    public static class SetterCommand extends HystrixCommand<Void> {

        private final int id;
        private final String prefix;

        public SetterCommand(int id, String prefix) {
            super(HystrixCommandGroupKey.Factory.asKey("GetSetGet"));
            this.id = id;
            this.prefix = prefix;
        }

        @Override
        protected Void run() {
            // persist the value against the datastore
            prefixStoredOnRemoteDataStore = prefix;
            // flush the cache
            GetterCommand.flushCache(id);
            // no return value
            return null;
        }
    }
}
```
[View Source](../blob/master/hystrix-examples/src/main/java/com/netflix/hystrix/examples/basic/CommandUsingRequestCacheInvalidation.java)

The unit test that confirms the behavior is:

```java
        @Test
        public void getGetSetGet() {
            HystrixRequestContext context = HystrixRequestContext.initializeContext();
            try {
                assertEquals("ValueBeforeSet_1", new GetterCommand(1).execute());
                GetterCommand commandAgainstCache = new GetterCommand(1);
                assertEquals("ValueBeforeSet_1", commandAgainstCache.execute());
                // confirm it executed against cache the second time
                assertTrue(commandAgainstCache.isResponseFromCache());
                // set the new value
                new SetterCommand(1, "ValueAfterSet_").execute();
                // fetch it again
                GetterCommand commandAfterSet = new GetterCommand(1);
                // the getter should return with the new prefix, not the value from cache
                assertFalse(commandAfterSet.isResponseFromCache());
                assertEquals("ValueAfterSet_1", commandAfterSet.execute());
            } finally {
                context.shutdown();
            }
        }
    }
```
<a name='MigratingLibrary'/>
## Migrating a Library to Hystrix

When you are migrating an existing client library to use Hystrix, you should replace each of the &ldquo;service methods&rdquo with a [HystrixCommand](http://netflix.github.com/Hystrix/javadoc/index.html?com/netflix/hystrix/HystrixCommand.html).

The service methods should then forward calls to the `HystrixCommand` and not have any additional business logic in them.

Thus, before migration a service library may look like this:

[[images/library-migration-to-hystrix-without-640.png]]

After migrating, users of a library will be able to access the [HystrixCommands](http://netflix.github.com/Hystrix/javadoc/index.html?com/netflix/hystrix/HystrixCommand.html) directly or indirectly via the service facade that delegates to the [HystrixCommands](http://netflix.github.com/Hystrix/javadoc/index.html?com/netflix/hystrix/HystrixCommand.html).

[[images/library-migration-to-hystrix-with-640.png]]