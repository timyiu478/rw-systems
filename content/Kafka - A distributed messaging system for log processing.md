---
tags:
  - kafka
  - distributed-systems
  - log
  - message-broker
  - big-data
reference: obsidian://open?vault=systems&file=papers%2FKafka.pdf
title: Kafka - A distributed messaging system for log processing
draft: true
description:
---
## Takeaways


---

## Details

### Requirements

* log aggregation
* support real-time log processing with delays no more than few seconds
* support offline consumers such as data warehousing application


### Design Principles



---
## Q & A

***Q. What is a distributed messaging system?***

Example: http://activemq.apache.org/

It is a message broker. It decouples the relationships between message producer and message consumer (time, space, target - producer does not need to know who will consume the messages).

***Q. Kafka vs traditional systems***

traditional messaging systems:
* do not focus as strongly on **throughput** as their primary design constraint.
	* E.g. JMS has no API for a producer to batch events into a single request.
* no easy way to partition or store events into different machines
* assume the published events will be quickly consumed, so the unconsumed queue length size is small
* queued-based: once a consumer acknowledges (acks) a message, it's typically removed/deleted from the queue.

traditional log aggregators:
*  use a “**push**” mode in which the broker forwards data to consumer

Kafka:
* use "**pull**" mode for log consumption
	* Why? The consumer can pull data at the maximum rate that it can handle. The "push" mode may overload the consumer because the broker floods data at a speed faster than the consumer can handle
	* How? It tells the broker about the offset of the first message that it wants to consume and the byte size it accepts to fetch.
* log-based: 
	* The events are **append-only**. 
	* Event has no ID or its ID is its log offset => easy to find the actual event location
	* Each consumer has its own read offset.
* multiple brokers

***Q. Is it possible that multiple processes share/use the same producer/consumer (config)?***

***Q. What are the assumptions about the consumer?***

The consumer consumes events **sequentially** instead of randomly.
