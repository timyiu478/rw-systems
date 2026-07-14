---
tags:
  - database
  - concurrency
  - mvcc
reference: https://www.cs.cmu.edu/~15712/papers/bernstein83.pdf
title: Multiversion Concurrency Control -Theory  and Algorithms
draft: false
description:
date: 2026-07-11
---

## Basic Serializability Theory

1. What is serializability theory? It tells the precise condition under which execution is correct.
2. Serializability model execution by **log**. (section 2.2)
3. Two logs are equivalent if they have the same **reads-from** relationships and they have the same final database state produced. (section 2.3)
4. A serial log is a log that for every pair of transactions $i, j$, all operations of $T_i$ either before or after all operations of $T_j$. It represents an execution which there is **no concurrency** whatsoever. (section 2.4)
5. A correct execution is a serializable log. It is equivalent to the serial log. (section 2.4)
6. If the serialization graph of a log is **acyclic**, the log is serializable. (section 2.5)


## Multiversion Serializability Theory

