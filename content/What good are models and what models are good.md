---
tags:
  - distributed-systems
  - modeling
reference: obsidian://open?vault=systems&file=papers%2Fgood-models.pdf
title: What good are models and what models are good
date: 2026-01-30
---

## Takeaways

* To postulate a system is synchronous or asynchronous, we need to consider both **processing speed** and message delivery channel.

---
## Pre-reading

What good are models?

* simplify the problem by making assumptions about what will/will not happen
	* => determine whether an implemented program is correct or not
		* whether the system behaviours are consistent with the model
	* exp1. Failure mode: the node will not restart after it crashes
		* Some persistence problems may not need to be worried about
	* exp2. Message channel: no message will be altered
* understand the limitations of the solution
	* exp.1. Consensus in the Byzantine failure model: $3f + 1$ nodes at most tolerant $f$ number of failures
	* exp.2. Liveness in the asynchronous network is not possible if there is $1$ node fails


What models are good?

* Realistic => we can use the guaranteed properties in that model to solve real problems
	* exp.1. Safety in the asynchronous network

---
## Model

What is a model?

A model for an object is a collection of attributes and a set of rules that govern how these attributes interact

What good are models?

* They can develop our intuition to understand the distributed systems. 
* We can answer different questions about the system by employing different models.
	* **Feasibility**. What classes of problems can be solved?
	* **Cost**. For those classes that can be solved, how expensive must the solution be?

What models are good?

A good model is **accurate** and **tractable**.
* accurate: analyse it yields truths about the object of interest
* tractable: it is possible to analyse it

## The Coordination Problem

We established that the Coordination Problem could not be solved by building a simple, informal model. Two insights were used in this model: 

1. All **protocols** between two processes are equivalent to **a series of message exchanges**. 
2. **Actions** taken by a process depend only on **the sequence of messages it has received**.

## Synchronous versus Asynchronous Systems

Postulating that a system is asynchronous is a non-assumption. Every system is asynchronous. Even a system in which processes run in **lock step (= no bound on processing speed) and message delivery is instantaneous** satisfies the definition of an asynchronous system


