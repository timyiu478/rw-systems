---
tags:
  - consistency
  - distributed-systems
reference: obsidian://open?vault=systems&file=papers%2FConsistencyAndBaseballReport.pdf
title: Replicated Data Consistency Explained Through Baseball by Doug Terry
draft: true
description:
---
## Details
### Questions that this paper attempts to answer

* Are different consistencies useful in practice?
* Can application developers cope with eventual consistency?
* Should cloud storage systems offer an even greater choice of consistency than the consistent and eventually consistent reads offered by Amazon’s SimpleDB?

### Consistency Guarantees

* *Strong consistency*
* *Eventual consistency*
	* can observe the results of **arbitrary subset of writes** =>

* *Consistent Prefix:*
	* ~ snapshot isolation
	* read can see the ordered sequence of writes
	* but the not-yet-received of the writes are **unbounded**
	* For example, the master distributes the append log that stores the write requests to the backup, the backup log cannot be in sync and only has the prefix of the master's log
* *Bounded staleness* 
	* ensures that read results are **not too out-of-date**
	* a read will return any values written after *T* time ago or more recently written
* *Monotonic Reads*
	* see increasing subset of writes **(per-client/per-session)**
	* future reads by the same client will never go backwards
* *Read My Writes*
	* the effects of all writes that were performed by the client are visible to the client’s subsequentreads


---

## Q & A

***Q. What are other systems where some of the weaker consistency guarantees are applicable?***

* A social application that allows users to post something. The *read-latest-write* guarantee is not necessary.

***Q. What type of app-specific knowledge matters when determining the “right” consistency guarantee?***

***Q. What is a consistency guarantee? What aspects of a system does it affect?***

***Q. How does a system designer choose an appropriate consistency guarantee?***

***Q. Why does the choice of consistency guarantee matter?***

Trade Off: consistency, performance, availability

***Q. Monotonic Reads vs Consistent Prefix***

Consistent Prefix guarantees that any read (by **any client**) observes some **prefix** of the **global** write order. It **does not prevent a client from going backwards** in its observed state because a client can read two different valid prefixes.

*Monotonic Reads* is a **per-client / per-session** guarantee. *Monotonic Reads* does not prevent **out-of-order** observation, especially across **multiple keys/items** or when the system doesn't enforce **global ordering**.

![[out_of_order_read.png]]