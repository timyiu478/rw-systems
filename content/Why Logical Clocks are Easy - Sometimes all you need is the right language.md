---
tags:
  - distributed-systems
  - causality
reference: https://queue.acm.org/detail.cfm?id=2917756
title: Why Logical Clocks are Easy - Sometimes all you need is the right language
draft: false
description: Vector clock is the encoded form of causal history
date: 2026-07-07
---

# Summary

* Happened-Before = **Potential** causality
* Causal Histories allow us to determine whether two events have the happened-before relation using **set inclusion**. Or it is a causality tracking mechanism.
* Vector clocks and version vectors are simply **optimized representations of causal histories**.
* We can select what events need to be tracked e.g. object update event.
