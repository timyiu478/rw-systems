---
tags:
  - kafka
  - distributed-systems
  - log
  - message-broker
  - big-data
reference: obsidian://open?vault=systems&file=papers%2FKafka.pdf
title: Kafka - A distributed messaging system for log processing
draft: false
description: A log-based scalable messaging system
---
## Takeaways

* The design choices of Kafka

---

## Details

### Requirements

* log aggregation
* support real-time log processing with delays no more than few seconds
* support offline consumers such as data warehousing application

### Design Choices

*For efficient data transfer:*

* **No Double buffering. Use Page Cache Only.** Why?
	* Warm cache still exists after the broker process is restarted
	* Little overhead on garbage collection in the garbage-collected language
	* Normal operating system caching heuristics are very effective (write-through cache and read-ahead)
		* read-ahead: OS reads the data from disk into memory before the user request
	* Evidence: production and the consumption have **consistent** performance linear to the data size, up to many terabytes of data.
* **Use *sendfile* OS API** / "Zero Copy"(**0 user <-> kernel copy**).
	* source: https://dzone.com/articles/understanding-zero-copy

![[static/zero_copy.png]]


*"Stateless" Broker:*

* The broker does not store information about the consumer,  e.g., how much the consumer has consumed
* The messages/events are automatically deleted based on the retention policy
	* Allow **re-playing** messages
		* When? E.g. 
			* After the fix of the application logic.
			* Consumer crashes before persisting some necessary state

*Serialization*:

* Use [Avro](https://avro.apache.org/) as our serialization protocol
	* It supports **schema evolution** and is efficient.
		* can enforce a contract to ensure schema compatibility between data producer and consumer.

*Distributed Coordination:*

* A partition within a topic is the smallest unit of parallelism.
	* => one partition is only consumed by one consumer within the consumer group => no coordination is needed for consuming a partition except re-balancing.
	* Note that each consumer can consume more than one partition
* How to coordinate? Use Zookeeper
	* Usages:
		* maintain the membership of brokers and consumers
		* trigger re-balancing
		* maintaining the consumption relationship and keeping track of the consumed offset of each partition
	* How to use zookeeper: see paper section 3.2, page 4


### Figures

![[static/kafka_figures.png]]


---
## Q & A

***Q. What is a distributed messaging system?***

Example: http://activemq.apache.org/

It is a message broker. It decouples the relationships between message producer and message consumer (time, space, target - producer does not need to know who will consume the messages).

It is distributed. The producer, consumer, and broker are distributed.

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

Yes. For example,
* [consumer group](https://developer.confluent.io/learn-more/kafka-on-the-go/consumer-groups/)  - Processes do not share the same consumer, but they share the same stream of events. The stream of events is divided into partitions. Each consumer process handles one of the partitions. No coordination across consumer groups.

***Q. What are the assumptions about the consumer?***

The consumer consumes events **sequentially** instead of randomly.

***Q. In Algorithm 1, why assign partitions from j x N to (j+1) x N - 1 in $P_T$ to consumer $C_i$?***


---

## Further Study

* The re-balancing algorithm
* Broker replication 
	* [[Building a Replicated Logging System with Apache Kafka]]
* Schema Registry