---
tags:
  - distributed-systems
  - replication
reference: https://pdos.csail.mit.edu/6.824/papers/cr-osdi04.pdf
title: Chain Replication for Supporting High Throughput and Availability
draft: false
description:
date: 2026-07-11
---
## System Overview


![[static/chain_replication_overview.png]]

---
## Details

#### Chain Replication Properties

Pros:

* Read operation involves **1 server - tail server**  
* Client RPCs are split between head and tail.
* Simple recovery plan
+ **Single Object** Linearizability

Cons:

* One failure requires reconfiguration

#### Extension for read parallelism

Split objects across chains

```
         H. M.  T.
Chain 1: S1 S2 S3
Chain 2: S2 S3 S1
Chain 3: S3 S1 S2
```


---

## Questions

#### Q. Suppose Chain Replication replied to update requests from the head, as soon as the next chain server said it received the forwarded update, instead of responding from the tail. Explain how that could cause Chain Replication to produce results that are not linearizable.

This scenario outlines a classic **stale read** that directly violates the core definition of linearizability.

```
Time --------------------------------------------->
C1:  | A = 200 |
C2:                 | read A is 100 |
```


1. Invocation: Client 1 ($C1$) issues a request to update $A = 200$. 
2. This request goes to the Head ($H$).Forwarding: $H$ updates its local state to $200$ and forwards the update to the Mid ($M$) node.
3. The Premature Acknowledgment: As soon as $M$ receives the update, it sends an acknowledgment back to $H$.
4. Write Completion: Upon receiving $M$'s ack, $H$ immediately replies to $C1$ saying the update was successful.  At this exact moment, $C1$'s operation is fully complete.
5. The Propagation Delay: While $M$ is preparing to forward the update to the Tail ($T$), or while the message is in flight on the network, a slice of wall-clock time passes. $T$ still holds the old value ($A = 100$).
6. The Read Request: Client 2 ($C2$) issues a read request for $A$.  By design in Chain Replication, all read requests are served by the Tail ($T$).
7. The Stale Return:  Because the update has not yet reached $T$, $T$ looks at its local storage and replies to $C2$ with $A = 100$.

