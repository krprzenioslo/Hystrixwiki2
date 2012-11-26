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
