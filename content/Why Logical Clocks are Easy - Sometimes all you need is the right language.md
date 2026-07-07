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

- The **happened-before** relation represents **potential** causality.
- **Causal histories** serve as a causality-tracking mechanism, allowing us to determine whether two events have a happened-before relation using **set inclusion**.
- **Vector clocks** and **version vectors** are simply optimized representations of causal histories.
- We can selectively choose which events need to be tracked (e.g., an object update event).