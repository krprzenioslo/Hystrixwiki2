The Hystrix Dashboard enables realtime monitoring of Hystrix metrics.

Use of this dashboard improved Netflix operations by reducing discovery and recovery times during operational events. The duration of most production incidents (already less frequent due to Hystrix) became far shorter, with diminished impact, due to the realtime insights into system behavior.

<a href="images/hystrix-dashboard-single-row.png">[[images/hystrix-dashboard-single-row-640.png]]</a>
_(Click for larger view.)_

When a circuit is failing then it changes colors (gradient from green through yellow, orange and red) such as this: 

[[images/dashboard-example-open-circuit-640.png]]

The diagram below shows one "circuit" from the dashboard along with explanations of what all of the data represents.

We've purposefully tried to pack a lot of information into the dashboard so that engineers can quickly consume and correlate data.

[[images/dashboard-annoted-circuit-640.png]]

It allows monitoring a single server or cluster of servers in aggregate with low latency (typically around 1 or 2 seconds when aggregating a cluster, subsecond with a single server).

[[images/dashboard-direct-vs-turbine-640.png]]

<a href="images/dashboard-example-1280.png">[[images/dashboard-example-640.png]]</a>
_(Click for larger view.)_

Following is an explanation of the metrics that are reported:

[[images/dashboard-annoted-circuit-640.png]]