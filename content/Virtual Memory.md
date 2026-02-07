---
tags:
  - operating-systems
  - memory
title: Virtual Memory
date: 2026-01-30
---

![[static/virtual_memory_mmu.png]]

**Q. Why MMU(Memory Management Unit)?**

Normal memory access is not a kernel call. Without MMU, we cannot enforce memory protection. Other processes can overwrite other address spaces.

**Q. How can the MMU tell the CPU that something went wrong like page fault?**

MMU and CPU are tightly integrated hardware components. MMU uses direct hardware signalling (buses, flags) to notify the CPU. Then the CPU can abort the current instruction and trap into kernel mode. Then the kernel will handle the exception based on the trap handler.


**Q. How do multiple CPUs/Cores use the one MMU**

They don't literally share one single MMU. Each CPU core has its own independent MMU. If they do share a single MMU, the MMU will be a massive performance bottleneck.


## References

1. https://web.mit.edu/6.1800/www/lectures/s03-all.pdf