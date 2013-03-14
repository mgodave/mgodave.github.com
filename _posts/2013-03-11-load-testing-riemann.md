---
layout: post
title: "Load Testing Riemann"
description: ""
category: 
tags: []
---
{% include JB/setup %}

Here we go, my first attempt at a blog...

I wrote a load testing tool for [Riemann](http://riemann.io). The tool opens a configurable number of connections and worker threads and continuously pummels Riemann with events. The code can be found [here](https://github.com/mgodave/riemann-client). Part of why I wrote this tool was to try to idenfity Riemann's limits. The tool uses a Riemann client, which I also wrote, to keep pressure on Riemann. The load tool places hooks into various places in the client pipeline to track it's performance. The following metrics are kept:
* The round trip time from when a Msg is sent from the client interface until it is ack'd by Riemann. 
* The rate at which acks are received at the client. 
* The number of outstanding, unacked, requests.

There are two interesting scenarios: sending Msg packets containing single events. Sending Msg packets containing a large number of events. I refer to these two scenarios as single event sends and bulk sends respectively. This post only addresses the first scenario, I will address the bulk send scenario in a later post.

#####Single Event Sends

######Test Environment

I ran this test on a mostly unoccupied iMac with the following CPU info: Intel(R) Core(TM) i7-2600 CPU @ 3.40GHz and 16 GB RAM. Riemann is 0.1.6-SNAPSHOT. It was sunny and 65 degrees, a slight breeze out of the SouthWest and some scattered whispy clouds. The client JVM ran with -XX:+UseConcMarkSweepGC -d64 and riemann was run with the aggressive flag (-a).

######Riemann Config

<script src="https://gist.github.com/mgodave/5155644.js"> </script>

######Invocation

```
riemann-load.sh localhost:5555
```

######Results

<p/>
[Raw output data] (/images/singletest.tar.gz)
<p/>

######Summary

|Metric|Value|
|:-----|------:|
|Send Rate|103,898 msg/sec| 
|Ack Rate|103,850 ack/sec|
|Un-Acked|22,990 msg|
|RTT|0.28 sec|
|Throughput|1,870,195 bytes/sec|

<p/>

The results are all pretty self explanatory. For the most part the lines are straight, which is what I would expect on a well behaving system. The only wierdness is with the large spikes in the unacked events graph. I can, for the most part, chalk these up to garbage collection events. However, since I did not collect this data I cannot be sure; I plan on collecting this and graphing it alongside this data in future runs. There is also a large, unknown event centered around 200 seconds, it is obvious in the graph entitled "Outstanding Un-Acked Events" and can also be noticed as a slight uptick in rtt time on the last graph.

######Sends/Ack Rate
![Load](/images/SendAckRate.png)
######Outstanding Un-Acked Events
![Load](/images/QueueLength.png)
######Round Trip Time
![Load](/images/Rtt.png)
######Throughput
![Load](/images/Throughput.png)



