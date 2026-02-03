---
tags:
  - distributed-systems
  - scalability
  - availablilty
reference: obsidian://open?vault=systems&file=papers%2FGiant.pdf
title: Lessons from Giant-Scale Services
draft: true
description: Lessons from Giant-Scale Services
---
## Why Giant Scale Services?

- Access anywhere, anytime
- Availability via multiple devices
- **Groupware support**. E.g. calendars
- Lower overall cost. Infrastructure resources can be multiplexed across active users

## Basic Model of Giant-Scale Sites

![[static/Lessons from Giant-Scale Services; basic mode of gaint scale site.png]]

## Why is node the basic building block for giant-scale service

- independent fault $\to$ 24/7 availability
- incremental scalability

## Load Management

- the problem of "round-robin DNS" are 
	- it **does not hide** the **inactive** servers.
	- short TLL $\to$ requires the client to make more DNS lookups.
- L4/L7 load balancer
	- monitor open TCP connections
	- server/node may serve only **some types of requests**
		- a node can be **not identical**

## Availability Metrics

1. $uptime = \frac{MTBF - MTTR}{MTBF}$
	1. so we can improve uptime by (1) reducing frequency of failure of (2) reducing the time to fix them.
2. $yield = \frac{\text{queries completed}}{\text{queries offered}}$
	1. this metrics is more useful in practice because **it directly maps to user experience**
3. $harvest = \frac{\text{data available}}{\text{complete data}}$
	1. this metrics measure **how much of the database is reflected in the answer**

The key insight is that we can **influence whether faults impact yield, harvest, or both**.
- **Replicated** systems tend to map faults to reduced capacity (and to yield at high utilizations), 
- while **partitioned** systems tend to map faults to reduced harvest, as parts of the database temporarily disappear, but the capacity in queries per second remains the same.

## DQ Principle

The DQ principle is
$$
\text{D: Data per query} \times \text{Q: queries per second } \to \text{ constant}
$$
The intuition behind this principle is that the system’s overall capacity tends to have a particular **physical bottleneck**, such as total I/O bandwidth or total seeks per second, which is tied to **data movement**.

Overall, DQ normally scales **linearly** with **the number of nodes**, which means a small test cluster is a good predictor for DQ changes on the production system.

The design goal for high availability is to control **how DQ reductions affect our three availability metrics**.

## Replication vs Partitioning from the perspective of DQ and availability metrics

Consider a two node cluster:
- The replicated version maintains 100% harvest under a fault but drop 50% yield.
	- drop yield because of **load redirection problem**, the alive nodes need to handle more loads(**overload**).
- The partition version maintains 100% yield under a fault but drop 50% harvest.
- Both versions have the same initial DQ value and lose 50 percent of it under one fault
	- Replicas maintains $D$ but drops $Q$
	- Partition maintains $Q$ but drops $D$

![[static/Lessons from Giant-Scale Services; overload due to failures.png]]

The key insight is the real cost is in the **DQ bottleneck** rather than storage space. An important exception to this is that replication requires more DQ points than partitioning for **heavy write traffic**, which is rare in giant-scale systems
 
## Avoid Saturation at a reasonable cost simply by good design is unrealistic for giant-scale systems

- The peak-to-average ratio for giant-scale systems seems to be in the range of **1.6:1 to 6:1** which can make it expensive to build out capacity well above the (normal) peak.
- **Single-event bursts**, such as online ticket sales for special events, can generate far above-average traffic. In fact, moviephone.com added 10x capacity to handle tickets for “Star Wars: The Phantom Menace” and still got overloaded.
- Some faults, such as power failures or natural disasters, **are not independent.** Overall DQ drops substantially in these cases, and the remaining nodes become saturated

## Graceful Degradation

Graceful degradation under excess load is critical for delivering high availability. The DQ principle gives us new options for graceful degradation: 
- We can either limit $Q$ (capacity) to maintain $D$
	- technique: **admission control**
- or we can reduce $D$ and increase $Q$
	- technique: **dynamic database reduction**

The insight is that graceful degradation is simply the **explicit process for managing the effect of saturation on availability**; that is, we explicitly decide how saturation should affect uptime, harvest, and quality of service.

Real Examples:
- **Cost-based AC.** Inktomi can perform AC based on the estimated query cost (measured in DQ). This reduces the average data required per query, and thus increases Q.
- **Priority AC.** Datek handles stock **trade** requests differently from other queries and guarantees they will be executed within 60 seconds, or the user pays no commission.
- **Reduced data freshness.** Under saturation, a financial site can make stock quotes expire less frequently. This reduces freshness but also reduces the work per query, and thus increases yield at the expense of harvest. (The **cached** queries **don’t reflect the current database** and thus have a lower harvest.)

## Disaster Tolerance

Disaster tolerance is a simple combination of managing replica groups and graceful degradation.

Natural disasters can affect all the replicas at one physical location, while other disasters such as fires or power failures might affect only one replica. So the basic question is **how many locations to establish** and **how many replicas to put at each**.

![[static/Lessons from Giant-Scale Services; disater tolerance analysis example.png]]

For Inktomi, the best plan would be to **dynamically reduce D by 2/3** (to get 3/2 Q) on the remaining replicas.

For load management, 
1. L4 switches do not help with the loss of whole clusters. 
2. DNS failover response time is slow.
3. With a **smart-client** approach, clients can perform higher-level redirection.

## Online evolution

We can think of maintenance and upgrades as **controlled** **failures**, and the “online evolution” process — completing upgrades without taking down the site.

By viewing online evolution as **a temporary, controlled reduction in DQ value**, we can act to minimise the impact on our availability metrics. In general, online evolution requires a fixed amount of time per node $u$, so that the total $DQ$ loss for $n$ nodes are
$$
n \times u \times (average = \frac{DQ}{n}) = DQ \times u  
$$

![[static/Lessons from Giant-Scale Services; upgrade stragety anaylsis.png]]



