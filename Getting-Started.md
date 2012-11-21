## Getting Binaries

Binaries can be downloaded directly from Maven Central or dependency information found for Maven, Ivy, Gradle and others at [http://search.maven.org](http://search.maven.org/#search%7Cga%7C1%7Cg%3A%22com.netflix.hystrix%22%20AND%20a%3A%22hystrix-core%22).


## Hello World!

The simplest use of Hystrix is as follows:

```java
public class CommandHelloWorld extends HystrixCommand<String> {

    private final String name;

    public CommandHelloWorld(String name) {
        super(HystrixCommandGroupKey.Factory.asKey("ExampleGroup"));
        this.name = name;
    }

    @Override
    protected String run() {
        return "Hello " + name + "!";
    }
}
```
[View Source](../blob/master/hystrix-examples/src/main/java/com/netflix/hystrix/examples/basic/CommandHelloWorld.java)

This command could be used like this:

```java
String s = new CommandHelloWorld("Bob").execute();
Future<String> s = new CommandHelloWorld("Bob").queue();
```

More examples and information can be found in the [[How To Use]] section.

Example source code can be found in the [hystrix-examples](../tree/master/hystrix-examples/src/main/java/com/netflix/hystrix/examples) module.

## Building

To checkout the source and build:

```
$ git clone git@github.com:Netflix/Hystrix.git
$ cd Hystrix/
$ ./gradlew build
```

To do a clean build:

```
$ ./gradlew clean build
```

A build should look like this:

```
$ ./gradlew build
:hystrix-core:compileJava
:hystrix-core:processResources UP-TO-DATE
:hystrix-core:classes
:hystrix-core:jar
:hystrix-core:sourcesJar
:hystrix-core:signArchives SKIPPED
:hystrix-core:assemble
:hystrix-core:licenseMain UP-TO-DATE
:hystrix-core:licenseTest UP-TO-DATE
:hystrix-core:compileTestJava
:hystrix-core:processTestResources UP-TO-DATE
:hystrix-core:testClasses
:hystrix-core:test
:hystrix-core:check
:hystrix-core:build
:hystrix-examples:compileJava
:hystrix-examples:processResources UP-TO-DATE
:hystrix-examples:classes
:hystrix-examples:jar
:hystrix-examples:sourcesJar
:hystrix-examples:signArchives SKIPPED
:hystrix-examples:assemble
:hystrix-examples:licenseMain UP-TO-DATE
:hystrix-examples:licenseTest UP-TO-DATE
:hystrix-examples:compileTestJava
:hystrix-examples:processTestResources UP-TO-DATE
:hystrix-examples:testClasses
:hystrix-examples:test
:hystrix-examples:check
:hystrix-examples:build

BUILD SUCCESSFUL

Total time: 30.758 secs
```

On a clean build you will see the unit tests be run and look something like this:

```
> Building > :hystrix-core:test > 147 tests completed
```