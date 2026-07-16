---
tags:
  - distributed-systems
  - distributed-transaction
reference: obsidian://open?vault=systems&file=papers%2Flarge-scale-incremental-processing-using-distributed-transactions-and-notifications.pdf
title: Large-scale Incremental Processing Using Distributed Transactions and Notifications
draft: true
description:
date: 2026-07-05
---
## One-line Summary

This paper describes a new incremental system: percolator for preparing web pages for inclusion on the web search index efficiently while maintaining the invarients of the index using distributed transaction and notification.

---

## Motivation

The search Index produced MapReduce requires reprocessing the entire web discards the work done in earlier runs and makes latency proportional to the size of the repository. It is good for batch processing but it’s inefficient for liveness.

## Strengths & Weaknesses

**Strengths:**

* lazy approach to cleaning up locks left behind by transactions running on failed machines??

**Weaknesses:**

* Does not provide serializability; It provides snapshot isolation, which is subject to [write skew](https://en.wikipedia.org/wiki/Snapshot_isolation).


---
## Details

### Transaction Protocol

* Coordinator: client
* Once the primary's write is visible, the transaction must be committed
* How to deal with client failure


---
## Questions

#### Q. What is incremental processing? 

Incremental processing is about **continuous refinement** of a dataset. It is *NOT* about recomputing the result from scratch.

#### Q. What are the challenges of large-scale incremental processing?

* tens of petabytes of data => data is distributed
* processes billions of updates per day; **no batching** for liveness or responsiveness
* maintain the invariants of the index in face of **concurrent update**
	* if the same content is crawled under multiple URLs, only the URL with the highest PageRank appears in the index
	* links to a duplicate of a page should be forwarded to the highest PageRank duplicate if necessary

#### Q. Why MapReduce requires reprocessing from scratch?

* Many computations (like PageRank, duplicate detection, or clustering) depend on relationships across _all_ documents. A single new page can affect scores everywhere, so partial updates aren’t straightforward.
* It is a batch system; no native incremental operators.

#### Q. What applications are not suitable for percolator?

* applications that require strong consistency

#### Q. Why we can bypass the "master" of bigtable and gfs and talks to the tablet server and chunk server directly in figure 1?

* Bigtable tablet servers are stateless. 
* When a Chunkserver boots up, it polls its local hard drive and tells the Master, _"Here are the chunks I have."_ The Master actually builds its map _from_ the workers. If the GFS Master dies and restarts, it just asks all the workers what they have to rebuild the map.

#### Q. How to against write-write conflicts?

write-write conflicts: if transactions A and B, running concurrently, write to the same cell, at most one will commit.

How to against: if the transaction sees another write record after its start timestamp, it aborts.

#### Q. Why Get() operation requires reading locks in addition to data?


#### Q. Can the deadlock happen in the transaction protocol?

#### Q. In figure 6, why `T.Write` and `T.Erase` for primary, and `bigtable:Write` and `bigtable:Erase` for secondaries?









