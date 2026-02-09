---
tags:
  - computer-network
  - cdn
  - distributed-systems
  - performance
reference: obsidian://open?vault=systems&file=papers%2Fthe-akamai-network-a-platform-for-high-performance-internet-applications-technical-publication.pdf
title: The Akamai Network - A Platform for High-Performance  Internet Applications
draft: true
description:
date: 2026-02-09
---
## Takeaways

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

***Q. The paper differentiates between a "content delivery network" and an "application delivery network". What is the difference between "content" and "applications", and why does this difference matter to Akamai?***



***Q. Why wouldn't a peer-to-peer network suffice for Akamai's purposes?***



***Q. The paper mentions "end users" and "customers". Who are Akamai's end users (are you?)? Its customers?***

End users are the people who would like to **access** the resources that are originated from the Akamai's customer. 

Customers are the people host or manage the original servers or applications and they select what resources will be **published** by the Akamai. Akamai will charge the customers for delivery their content or application.

***Q. What aspect(s) of the Internet's infrastructure is Akamai's platform designed to overcome?***



***Q. How is the platform designed to overcome those aspects?***



***Q. Why is it necessary for Akamai to overcome those aspects?***


---


## Details

### Internet inherent Limitations for delivery content, stream, and dynamic applications to global users

* Best-effort => no end-to-end performance or reliability guarantees
* WAN => latency, packet loss, inefficient protocols, inter-network data communication

Problem of inter-network data communication:

* **peering point congestion**
* BGP routes based AS hop count (not aware underlying topologies, network congestion)
	* several paths between locations **within Asia** are actually routed through peering points in the **US**
* Unreliable network
	* DDoS
* TCP is not performed well in links with high latency and packet loss
* Scalability: server capacity and the **network bandwidth at all points between end users and the applications they trying to access**

### Figures