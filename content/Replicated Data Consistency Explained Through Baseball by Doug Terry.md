---
tags:
  - consistency
  - distributed-systems
reference: obsidian://open?vault=systems&file=papers%2FConsistencyAndBaseballReport.pdf
title: Replicated Data Consistency Explained Through Baseball by Doug Terry
draft: false
description: The different consistency guarantees are useful and understand how the data is being used is useful for choosing the desired consistency guarantees
---
## Takeaways

* *Consistent Prefix* vs *Monotonic Reads*
* The types of app-specific knowledge to determine the "right" consistency guarantees
* A single client can read different items, and different items may need different consistency guarantees
	* Example: Statistician
* Different clients may want different consistencies even when accessing the same data 

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

![[consistency_guarentees_baseball_example.png]]


---

## Q & A

***Q. Monotonic Reads vs Consistent Prefix***

Consistent Prefix guarantees that any read (by **any client**) observes some **prefix** of the **global** write order. It **does not prevent a client from going backwards** in its observed state because a client can read two different valid prefixes.

*Monotonic Reads* is a **per-client / per-session** guarantee. *Monotonic Reads* does not prevent **out-of-order** observation, especially across **multiple keys/items** or when the system doesn't enforce **global ordering**.

![[out_of_order_write.png]]

***Q. What are other systems where some of the weaker consistency guarantees are applicable?***

* A social application that allows users to post something. The *read-latest-write* guarantee is not necessary.
* DNS. The DNS zone servers across the world are not always in sync.
* Online Document Share Editing App. The global write order is not needed. Per-session/Per-Document consistency guarantee is sufficient. 

***Q. What type of app-specific knowledge matters when determining the “right” consistency guarantee?***

* Number of writers
	* examples: 
		* scorekeeper - single writer
* The usage of the read value
	* examples: 
		* scorekeeper - increment the read value by one and then write it back => needs to observe the latest writes
		* umpire - determine whether can end the game earlier
		* **radio sports reporter**
			* broadcasts the scores periodically => scores are not up-to-date is ok
			* should not report scores that do not exist in the past/present
			* scores are monotonically increasing => the report scores should not go backwards
				* *consistent prefix* guarantee is not sufficient
				* we need  *consistent prefix* + *monotonic reads* guarantees or 
				* *bounded staleness + monotonic reads* guarantees
		* sport writer - only care about the final score
			* use 
* The role of the client
	* examples:
		* umpire - only read
			* read-my-write is not "right" consistency guarantee



***Q. What is a consistency guarantee? What aspects of a system does it affect?***

Consistency guarantee is a promise to the system reader about the values it can/cannot observe.

It affects the performance and the availability of the system. For example, the client of the system may only be able to read/write from the primary replica instead of any replica to ensure strong consistency.

***Q. Why does the choice of consistency guarantee matter?***

Trade Off: consistency, performance, availability, and the burden of the application developer

By relaxing the consistency requests, the clients will likely observe better performance and availability. Also, the storage system may be able to better balance the read workload across servers since it has more flexibility in selecting servers to answer weak consistency read requests.

The cost of weaker consistency is the application developer needs to think and choose the desired consistency guarantees.
