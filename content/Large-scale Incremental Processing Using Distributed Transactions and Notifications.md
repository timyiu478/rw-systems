---
tags:
  - distributed-systems
  - distributed-transaction
reference: obsidian://open?vault=systems&file=papers%2Flarge-scale-incremental-processing-using-distributed-transactions-and-notifications.pdf
title: Large-scale Incremental Processing Using Distributed Transactions and Notifications
draft: false
description: This paper introduces Percolator, an incremental processing system that utilizes distributed transactions and notifications to efficiently update a web search index while strictly maintaining data invariants.
date: 2026-07-05
---
## One-line Summary

This paper introduces Percolator, an incremental processing system that utilizes distributed transactions and notifications to efficiently update a web search index while strictly maintaining data invariants.

---

## Motivation

The search Index produced MapReduce requires reprocessing the entire web discards the work done in earlier runs and makes latency proportional to the size of the repository. It is good for batch processing but it’s inefficient for liveness.

## Strengths & Weaknesses

**Strengths:**

* lazy approach to cleaning up locks left behind by transactions running on failed machines

**Weaknesses:**

* Does not provide serializability; It provides snapshot isolation, which is subject to [write skew](https://en.wikipedia.org/wiki/Snapshot_isolation)
* Rely on the timestamp oracle to provide the strictly increasing timestamps.
* Rely on Bigtable features: atomic row transaction, timestamp-based versioned storage


---
## Details

### Transaction Protocol

* Coordinator: client
* Once the primary's write is visible, the transaction must be committed
* How to deal with client failure
	* primary lock
		* the location of the primary is written into the locks at all other cells (figure 6's line 38)
	* crash before commit
		* **lazy** clean up by live clients
	* crash after commit (primary location lock has been replaced by write)
		* roll forward by live clients


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

#### Q. What applications are not suitable for percolator?

* applications that require strong consistency

#### Q. Why we can bypass the "master" of bigtable and gfs and talks to the tablet server and chunk server directly in figure 1?

* Bigtable tablet servers are stateless. 
* When a Chunkserver boots up, it polls its local hard drive and tells the Master, _"Here are the chunks I have."_ The Master actually builds its map _from_ the workers. If the GFS Master dies and restarts, it just asks all the workers what they have to rebuild the map.

#### Q. How to against write-write conflicts?

write-write conflicts: if transactions A and B, running concurrently, write to the same cell, at most one will commit.

How to against: if the transaction sees another write record after its start timestamp, it aborts.

#### Q. Why Get() operation requires reading locks in addition to data?

It is used to check if another transaction is actively writing to that cell by reading the locks that were created **before** the reader started. So that the Get() can guarantee to return all committed writes before the transaction's start timestamp.

#### Q. Can the deadlock happen in the transaction protocol?

No, because **only readers wait for writers**.

- **Writers** never wait for anyone (they just abort and retry).
- **Readers** only wait for uncommitted writers.

#### Q. In figure 6, why `T.Write` and `T.Erase` for primary, and `bigtable:Write` and `bigtable:Erase` for secondaries?

- **`T.Write`:** Used on the Primary row inside a strict Bigtable single-row transaction.
- **`bigtable::Write`:** A fast, raw API call used on the Secondary rows because the transaction is already officially committed, and **any failures here can be safely cleaned up later by other readers.**

---
## Educational Implementation

https://github.com/timyiu478/percolator


