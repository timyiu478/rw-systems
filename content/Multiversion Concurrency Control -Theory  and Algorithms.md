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

1. Multiversion log: translate data operation into version operation (section 3.1)
	1. have the same write operations since $h(w_i[x]) = w_i(w_i)$
	2. but may not have the same reads.
2. Two MV logs are equivalent if they have the same reads-from relationships. (section 3.2)
	1. Two logs have the same read operation $r_j[x_i]$
3. The **only pattern of conflict operations**: $w_i[x_i]$ and $r_j[x_i]$. $w_i[x_i] < r_j[x_i]$
	1. $w_i[x_i] > r_j[x_i]$ is impossible because $T_j$ only read the version of $x$ until it has been produced.
	2. $w_i[x_i] < w_j[x_j]$ or  $w_i[x_i] > w_j[x_j]$ are impossible because they produce different versions.
4. In the serialization graph of a MV log $L$, the edge of $T_i$ to $T_j$ is present iff some $x$, $r_j[x_i]$ is an operation in $L$. (section 3.2)
5. A serial MV log is **one-copy** serial (or 1-serial) if $T_j$ reads-from $T_i$ then $i=j$ or $T_j$ is the last transaction preceding $T_j$ that write any version of $x$. (section 3.3)
6. A one-copy serializable log is equivalent to a 1-serial log.
7. 