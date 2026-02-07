---
tags:
  - distributed-systems
  - consistency
  - database
  - latency
reference: obsidian://open?vault=systems&file=articles%2Fpacelc.pdf
title: Consistency Tradeoffs in Modern Distributed Database System Design by Daniel J. Abadi
draft: false
description: The tradeoff between latency and consistency in distributed database system during normal operation
date: 2026-02-03
---
## Takeaways 

* Why does a tradeoff between latency and consistency during normal operation arise?
* Low-latency data replication implementation does not imply network partition tolerance.

---

## Summary

The research article argues that when designing replicated data systems (especially over WANs), we must consider not only behavior during network partitions but also during normal operation. The fundamental implementation of data replication imposes inherent trade-offs between latency and consistency _at all times_ throughout system operation.

---

## Details

### Data Replication Implementations

*Assumptions:*

* Multiple clients are submitting updates to the system concurrently


![[static/pacelc_data_replication_implementations.png]]
Noted that (2)(b)(ii) **does not tolerate network partition**. If the master node becomes inside the minority partition, the system by default makes the data item unavailable for updates.
	* Real system example: 
		* PNUTS: a **PC/EL** system
			* PC does not indicate the system is fully consistent. It means the system does not **reduce** consistency beyond its **baseline** consistency during network partition.

---
## Q & A

***Q. What are the consistency tradeoffs in DDBS design? / What are the system constraints during normal operation?***

**PACELC:** combine CAP with latency/consistency tradeoff.
* Network **P**artition: **A**vailability vs **C**onsistency
	* Network partition causes the system to **reduce** availability or consistency.
* **E**lse(No Network Partition): **L**atency vs **C**onsistency

Consistency in here: Strong consistency
Latency in here: Latencies are **smaller than a request timeout**, but still approaching hundreds of milliseconds.

The system can reduce consistency during network partition, but the system can **maintain the consistency during normal operation**.

Why do we care about the consistency tradeoff during normal operation? 
* latency increase => customers decrease

***Q. Why does the system have a tradeoff between latency and consistency during normal operation?***
* The failure possibility => HA is needed => data replication (over WAN) =>
* Possible implementations:
	* (1) an system sends data updates to all the replicas
	* (2) sends to an agreed-upon master node first
	* (3) sends to a single (arbitrary) node first

***Q. Any reason to replicate data using (3) sends to a single (arbitrary) node first?***

1. client can always select to send to the nearest node.
2. There is no “master” bottleneck for any key.

***Q. Why Distributed Database?***

* increase data transactional **throughput** => elastic horizontal scalable DB
* increase **globalisation** => replicate data near the clients who are spread across the world

***Q. Any PA/EC system example?***

MongoDB. The author argues the partition causes more consistency issues than availability issues. (See PACELC section for the details)