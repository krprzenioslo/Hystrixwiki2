This page includes many frequently-asked questions about how to operate Hystrix.

**Why am I receiving `java.lang.IllegalStateException`: Another strategy was already registered ?**

> The root cause is that all of the objects held by `HystrixPlugins` must be singletons, and must be in place upon the first HystrixCommand invocation.  As an example, the `HystrixConcurrencyStrategy` must be non-null while the first HystrixCommand is executing, so that it knows how to wrap `Callable`s and set up a `HystrixRequestContext`.

> The 2 paths for generating the `HystrixConcurrencyStrategy` are:
* No explicit initialization, and the first command invocation sets up the default `HystrixConcurrencyStrategy`.
* Explicit initialization via `HystrixPlugins.registerConcurrencyStrategy`

> Note that the latter fails with the `IllegalStateException` with cause "Another strategy was already registered" if the implicit initialization runs first.

> The above is also true for any of the other objects in `HystrixPlugins`.

> So what must be happening is that your application startup ends up creating a race between the first command running (and performing implicit initialization), and an invocation of one of the explicit init calls.  If the implicit goes first by a command executing, then you're seeing the error above.

> If possible, you should move the explicit plugin init as early as possible to avoid that race.  In practice, that has worked for us internally at Netflix.  

> The full set of calls to look for is:
* `HystrixPlugins.registerEventNotifier`
* `HystrixPlugins.registerConcurrencyStrategy`
* `HystrixPlugins.registerMetricsPublisher`
* `HystrixPlugins.registerPropertiesStrategy`
* `HystrixPlugins.registerCommandExecutionHook`