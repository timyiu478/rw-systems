---
tags:
  - computer-network
  - cdn
  - distributed-systems
  - performance
reference: obsidian://open?vault=systems&file=papers%2Fthe-akamai-network-a-platform-for-high-performance-internet-applications-technical-publication.pdf
title: The Akamai Network - A Platform for High-Performance  Internet Applications
draft: false
description: How does Akamai design a delivery network to overcome the internet inherent limitations
date: 2026-02-09
---
## Takeaways

* How does Akamai design a delivery network to overcome the internet inherent limitations

---


## Q & A

***Q. What are Akamai's design goals?***

> The philosophy that failures are normal and the delivery network must operate seamlessly despite them

1. **Reliability**. No single point of failure.
2. **Scalability**. Systems that must support an ever-increasing number of distributed machines.
3. **Limit the necessity for human management.**  Automated load balancing, Self-tuned performance, Safe deployment with minimal human intervention.
4. **Performance**. More traffic served with fewer machines, High cache hit rate.

***Q. What happens when a user visits a particular URL? How does the content get to their machine?***

1. the particular URL will be resolved by the *mapping system* to the edge servers that is located close to the end user
2. the edge server will process the user request and then send the response back to the end user
3. to process the user request, the edge server may need to contact the original server via the transport system

***Q. Why wouldn't a peer-to-peer network suffice for Akamai's purposes?***

The lack of management and control features in current implementations make them unsuitable as stand-alone solutions for enterprisequality delivery.

***Q. The paper mentions "end users" and "customers". Who are Akamai's end users (are you?)? Its customers?***

End users are the people who would like to **access** the resources that are originated from the Akamai's customer. 

Customers are the people host or manage the original servers or applications and they select what resources will be **published** by the Akamai. Akamai will charge the customers for delivery their content or application.

***Q. Why is it necessary for Akamai to overcome those aspects?***

The demand for high throughput.

> In the near term (two to five years), it is reasonable to expect that throughput requirements for some single video events will reach roughly 50 to 100 **Tbps** (the equivalent of distributing a TVquality stream to a large prime time audience).

The extremely well-connected data centre approach can not support such demand.

> Because of the limited capacity at the Internet‘s various bottlenecks, even an extremely well-provisioned and wellconnected data center can **only expect to have no more than a few hundred Gbps of real throughput to end users**. This means that a CDN or other network with even 50 well-provisioned, highly connected data centers still falls well short of achieving the 100 Tbps needed to support video‘s near-term growth.

***Q. How is the platform designed to overcome internet limitations?***

* **Highly distributed edge servers**
	* close to the end users => lower latency, less middle-mile/peering points congestion
	* can push application logic and caching to the edge
* **Overlay transport system**
	* aware underlying topologies and network congestion => optimise paths across the internet
	* control both "client" and server => can optimise the transport layer protocol since it is not being constrained by client software adoption rates for internal communications
* **Publish–subscribe streaming model**
	* multiple alternate **(link-disjoint)** paths => ensure high end-to-end scalability and resilience for live events

---

## Details

### Internet inherent Limitations for delivery content, stream, and dynamic applications to global users

* Best-effort => no end-to-end performance or reliability guarantees
* WAN => latency, packet loss, inefficient protocols, inter-network data communication

Problems of inter-network data communication:

* **peering point congestion**
* BGP routes based AS hop count (not aware underlying topologies, network congestion)
	* several paths between locations **within Asia** are actually routed through peering points in the **US**
* Unreliable network
	* DDoS
* TCP is not performed well in links with high latency and packet loss
* Scalability: server capacity and the **network bandwidth at all points between end users and the applications they trying to access**

## Architecture Considerations

1. **How distributed such an architecture needs to be?**
	1. **edge-based**: deployed in thousands of networks and **local ISPs**
		1. Avoid the need for the inter-network data communication
		2. Distance is the bottleneck for video throughput (See table 1)
2. **Leverage the large network on demand and end-to-end scalability**
3. **Streaming Performance**
	1. metrics: 
		1. users want the stream to start quickly => startup time
		2. users want to watch the stream without interruptions or freezes => frequency and duration of interruptions
		3. users want to experience the media at the highest bitrate that their **last-mile connectivity**, desktop, or device would allow => effective bandwidth
4. **A Transport System for Content and Streaming Media Delivery**
	1. Caching
		1. **tired distribution**: 
			1. "Parent" clusters that are **highly connected** with the edge servers
			2. When the edge server cache miss, it can retrieve data from the parent cluster rather than the original server
	2. An Overlay Network for Live Streaming
		1. publish-subscribe model: entry point <- reflectors <- edge cluster 
			1. reflectors provide **multiple alternate (link-disjoint) paths** between each entry point and edge cluster
		2. How to (re)construct the overlay paths? https://dl.acm.org/doi/10.1145/777412.777437

### Figures

![[static/akamai_delivery_network_figures.png]]