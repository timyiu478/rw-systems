---
tags:
  - database
  - isolation
reference: https://www.microsoft.com/en-us/research/wp-content/uploads/2016/02/tr-95-51.pdf
title: A Critique of ANSI SQL Isolation Levels
draft: true
description:
date: 2026-07-24
---
## Summary

The ANSI SQL-92 defines four isolation levels by **phenomena**. However, the original definitions were ambiguous. This paper defines the isolation level as **the set of histories that a particular lock-based concurrency control mechanism allows** to resolve ambiguity. This paper also introduces more isolation levels, such as cursor stability and multi-version isolation type - snapshot isolation.

## Degree of Consistency and Locking Isolation Levels

* **Well-formed**: lock on (tuples/predicates) before reading/writing record
* **Long duration**: hold lock until the transaction commits/aborts

![[degree of consistency and isolation levels.png]]

## Hierarchy of Isolation Levels

Isolation level L1 is weaker than isolation level L2, denoted L1 << L2, if all non-serializable histories that obey the criteria of L2 also obey L1 and there is at least one non-serializable history that can occur in L1 but not in L2.

## Cursor Stability

* Goal: prevent **lost update**
	* P4: `r1[x] -> w2[x] -> w1[x]` (the `w1[x]` depends on `r1[x]`)
* Cursor lock: the lock is acquired when the cursor moves to a row and is **held for as long as the cursor remains positioned on that row**.

## Snapshot Isolation




## Comments

**C1 - typo.** In section 3, the author said: "H2 would now be disqualified when **w2[x=20]** occurs to overwrite r1[x=50]." It should be H2 would now be disqualified when **w2[x=10]** occurs to overwrite r1[x=50] because H2 is `H2: r1[x=50]r2[x=50]w2[x=10]r2[y=50]w2[y=90]c2r1[y=90]c1`. 


## Questions

### Q. What's wrong with the ANSI definition?

It fails to characterise some popular isolation levels, including the standard locking implementation of the levels.

* Dirty Write (P0)
* Several isolation levels popular in commercial systems fall between READ COMMITTED and REPEATABLE READ.

It leads to a misconception that disallowing the three phenomena implies serialisability.

It is intended to define Repeatable Read to exclude all anomalies except phantom. However, the 
anomaly definition of Table 1 does not achieve this goal.

The definition of dirty read (P1) does not actually insist that T1 abort.

### Q. In section 3, why does the author say the broad interpretation of the three ANSI phenomena is required?

Because the strict interpretation of the three ANSI phenomena fails to disallow many non-serializable histories.

Read section 3 for examples of non-serializable histories.