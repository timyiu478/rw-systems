---
tags:
  - kafka
  - distributed-systems
  - log
reference: obsidian://open?vault=systems&file=papers%2Fp1654-wang.pdf
title: Building a Replicated Logging System with Apache Kafka
draft: false
description: How does Kafka replicate its broker log, and why
date: 2026-02-07
---
## A Log Consensus Protocol

The *primary-backup* approach:

* $f+1$ replicas can tolerate $f$ node failures.
* A leader needs to wait **all** $f$ backup acknowledgments.

The *quorum* approach:

* $2f+1$ replicas can tolerate $f$ node failures.
* A leader needs to wait **any** $f$ backup acknowledgments.

Kafka uses the ***primary-backup*** approach to replicate logs from the leader to the followers. Because the Kafka instances are deployed in the **same data centre**, the network is highly reliable and low-latency. So the benefit of using the **less replicas** outweighs the potential slight increase in latency.

Kafka uses the *zookeeper* for [**leader election**](https://zookeeper.apache.org/doc/r3.8.5/recipes.html#sc_leaderElection). The *zookeeper* is a *CP* system that requires a majority quorum for committing a write. The Kafka instance does not need to store the cluster membership information.

## Future Study

> Producer clients can specify different replication criterion for acknowledging their sent messages; for example, they can choose to wait for acknowledgement until the messages have been replicated to all in-sync replicas, or only the leader. On the other hand, consumer clients only fetch messages that have been replicated to all in-sync replicas.

Why?

