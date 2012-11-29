<a name='Name'/>
## Where does the name come from?

Naming things is [hard](http://martinfowler.com/bliki/TwoHardThings.html).

Many words, synonyms, animals and mashups of them all were sought that would keep the theme of resilience, defense and fault tolerance while being short, easy to say and not already taken. Not many worked.

A Hystrix is an "[Old World porcupine](http://en.wikipedia.org/wiki/Hystrix)" with an [impressive defense mechanism](http://www.arkive.org/north-african-crested-porcupine/hystrix-cristata/video-11b.html). It's also (in my opinion) a cool name, only 2 syllables (I end up saying it a lot so this was important), looks nice written out and when Googling for it only found animals so seemed free from collision with other products.

And it allows for cool artistic interpretations such as this logo:

[[images/hystrix-logo.png]]


<a name='AtNetflix'/>
## How is this used at Netflix?

Netflix uses Hystrix in many applications, particularly its edge services such as the Netflix API. Tens of billions of thread-isolated and hundreds of billions of semaphore-isolated calls are executed via Hystrix every day at Netflix.

To learn more about how Hystrix is used and where it evolved from take a look at these blogs:

* [Making Netflix API More Resilient](http://techblog.netflix.com/2011/12/making-netflix-api-more-resilient.html)
* [Fault Tolerance in a High Volume, Distributed System](http://techblog.netflix.com/2012/02/fault-tolerance-in-high-volume.html)

Also, this slidedeck is from a presentation that goes into a little more detail about its usage on the Netflix API:

* [Performance and Fault Tolerance for the Netflix API](https://speakerdeck.com/benjchristensen/performance-and-fault-tolerance-for-the-netflix-api-august-2012)


<a name='Intrusive'/>
## Why is it so intrusive?

Common first reactions to Hystrix (even internally at Netflix when first introduced to teams) include:

- Why is this so intrusive?
- Why do I need to change my client libraries?
- Why is it a command pattern that requires wrapping libraries or network calls?
- Why not intercept calls at a lower level?

By design Hystrix intends to offer a clearly defined barrier of "host app" versus "dependency". Anything that goes over the network or can possibly trigger something that goes over the network is a possible source of failure or latency. Hystrix explicitly adds a layer between these points of failure. This is not only for functional reasons but also as a standard mechanism for communicating to users of that object that it is a "protected" resource.

When developing Hystrix at Netflix we specifically sought out transparent network calls and wrapped them in HystrixCommand implementations as on multiple occasions these were the cause of production outages.

Developers interact with a library they know to execute over the network very differently than something they know to be an in-memory cache lookup.

Thus, the addition of the Hystrix layer serves these purposes:

1) Communicate resilience to anyone calling it. 
2) Ability to execute synchronously (HystrixCommand.execute()) or asynchronously (HystrixCommand.queue()).
3) Ability to query a command after execution for state (fallback, errors, metrics, etc)

The [[migration of a library|How-To-Use#wiki-MigratingLibrary]] to using Hystrix typically looks like this:

[[images/library-migration-to-hystrix-with-640.png]]

If you still feel strongly that you shouldn't have to modify libraries and add command objects then perhaps you can [[contribute an AOP module|FAQ#wiki-AOP]].


<a name='TransitiveDependencies'/>
## What about transitive dependencies?



<a name='Annotations'/>
## Can annotations be used?

Not as part of (hystrix-core)[https://github.com/Netflix/Hystrix/tree/master/hystrix-core] functionality.

It has been considered but not pursued heavily. It is definitely a candidate for someone to implement as a (sub-module)[https://github.com/Netflix/Hystrix/tree/master/hystrix-contrib].

The primary design principle this doesn't mesh very well with is that it makes the isolation barriers transparent (see "Why is it so intrusive?" for more reasoning on this). In other words, a consumer of a library would no longer see a HystrixCommand implementation with standard execute(), queue() and other functionality nor receive the communication of isolation and fault tolerance that is assumed when interacting with a HystrixCommand. They would just invoke a method and have no idea of whether it's isolated or not.

<a name='AOP'/>
## Why not use AOP?

AOP has been avoided as part of (hystrix-core)[https://github.com/Netflix/Hystrix/tree/master/hystrix-core] functionality due to the non-obviousness of using it and the desire to stay away from bytecode manipulation. 

It also goes against the principles of Hystrix which prefer explicitly exposing access points to dependencies, networks and system as points of possible failure (see "Can annotations be used?" and "Why is it so intrusive?" for more reasoning on this).

However, there may be use cases where it's applicable and thus it is a camdidate someone to implement as a (sub-module)[https://github.com/Netflix/Hystrix/tree/master/hystrix-contrib].

A related area where it may be useful is not for Hystrix command objects but for tracking drift – determining points of unwrapped network access that spring up over time.


<a name='InterceptNetwork'/>
## Why not just automatically intercept all network calls?

<a name='LoadBalancer'/>
## Why don't you just use a load-balancer?

<a name='Costs'/>
## What is the processing overhead of using Hystrix?

<a name='UnitTests'/>
## Why are unit tests inner classes?

Hystrix has all of its unit tests as inner classes rather than in a separate /test/ folder.

Why?

- Low Friction
- Context
- Encapsulation
- Refactoring
- Self-documenting

More information on the reasoning can be found in this blog post: [JUnit Tests as Inner Classes](http://benjchristensen.com/2011/10/23/junit-tests-as-inner-classes/)

<a name='Async'/>
## What about asynchronous dependency calls?







