---
tags:
  - latency
  - distributed-systems
  - tail-tolerant
  - performance
reference: obsidian://open?vault=systems&file=articles%2Fthe%20tail%20at%20scale.pdf
---
## Takeaways



---

## Q & A 

**Q. What is tail-tolerant?**

Tail-tolerant is like fault-tolerant.
Fault-tolerant/Reliable systems are built from unreliable parts.
Tail-tolerant/Predictably responsive systems are built from less-predictable parts.

**Q. What do we mean when we talk about "latency" in a system?**

The response time of the system for a client request.

**Q. Why Latency Variability Exists?**

1. processes/systems compete the shared resources
2. background cron job is invoked
3. Maintenance activities: 
	1. disk corruption triggers the distributed file system to re-construct the data on a new disk
	2. Garbage collection in the garbage collected programming languages
4. Hardware:
	1. Energy management: inactive mode -> active mode


**Q. Why does latency, and in particular the _tail_ of the latency distribution in a system, matter? Who is impacted by it?**

A user request is divided into sub-requests, and the sub-requests fan out across hundreds or thousands of servers to parallelise the request processing.
A single slow outlier can yield dramatic reductions in overall service performance.

![](tail_at_scale_slow_outlier.png)

**Q. In Reducing Component Variability section, the authors mention "Keep low-level queues short so higher-level policies take effect more quickly". Why keep low-level queues short so higher-level policies take effect more quickly?**

The low-level queues, such as the OS disk queue, are **out of the application's control**.
They are not aware of the higher-level policies/prorities and they might serve the requests in a simple FIFO manner.
If these low-level queues are long, the application has to wait for many old requests to finish first.
The shallow low-level queues **minimise the "distance"** between the smart policy decisions and actual device execution.


**Q. How can a distributed system decrease this tail latency? Give one example.**

_Managing background activities and synchronized disruption_.

1. Synchronise the background activities of all machines, so they execute at the same time, so that only user requests being handled in the synchronisation period are slow.
2. Throttle the user requests during the synchronisation period to reduce the load of the system.