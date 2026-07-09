---
tags:
  - distributed-systems
  - Time
  - Consensus
reference: https://queue.acm.org/detail.cfm?id=2745385
title: There is No Now - Problems with simultaneity in distributed systems
draft: true
description: Design for Failure, Not Perfection; Causality over Chronology
date: 2026-07-09
---
> Design for Failure, Not Perfection; Causality over Chronology


* Zookeeper atomic protocol vs Paxos
	* able to process many requests at once
	* built on top of TCP => can assume **prefix** property (sender sends A->B, then receiver receives A->B)
* Last Write Win policy = Most Writes Lose policy