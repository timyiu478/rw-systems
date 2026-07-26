---
tags:
  - database
  - isolation
  - concurrency
reference: https://www.microsoft.com/en-us/research/wp-content/uploads/2016/02/tr-95-51.pdf
title: A Critique of ANSI SQL Isolation Levels
draft: true
description: " This paper defines the isolation level as the set of histories"
date: 2026-07-24
---
## Summary

The ANSI SQL-92 defines four isolation levels by **phenomena**. However, the original definitions were ambiguous. This paper defines the isolation level as the set of **histories** that a particular lock-based concurrency control mechanism allows to resolve ambiguity. This paper also introduces **more isolation levels**, such as cursor stability and multi-version isolation type - snapshot isolation.

## Degree of Consistency and Locking Isolation Levels

* **Well-formed**: lock on (tuples/predicates) before reading/writing record
* **Long duration**: hold lock until the transaction commits/aborts

![[degree of consistency and isolation levels.png]]

## Hierarchy of Isolation Levels

Isolation level L1 is weaker than isolation level L2, denoted L1 << L2, if all non-serializable histories that obey the criteria of L2 also obey L1 and there is at least one non-serializable history that can occur in L1 but not in L2.

![[hierarchy of isolation levels.png]]

**Histories:**

* *P0 Dirty Write:* `w1[x].. w2[x].. ((c1/a1) and (c2/a2) in any order)`
* *P1 Dirty Read:*  `w1[x].. r2[x].. ((c1/a1) and (c2/a2) in any order)`
* *P2 Fuzzy Read/Non-repeatable Read:* `r1[x].. w2[x].. ((c1 or a1) and (c2 or a2) any order)`
* *P3 Phantom:* `r1[P].. w2[y in P].. ((c1 or a1) and (c2 or a2) any order)`
* *P4 Lost Update:* `r1[x].. w2[x].. w1[x].. c1`
* *P4C Cursor Lost Update:* `rc1[x].. w2[x].. w1[x].. c1`
	* A rc1[x] and a later wc1[x] preclude an intervening w2[x].
* A3: `r1[P].. w2[y in P].. c2.. r1[P].. c1`
* *A5A Read Skew:* `r1[x].. w2[x].. w2[y].. c2 r1[y].. (c1/a1)`
* *A5B Write Skew:*  `r1[x].. r2[y].. w1[y].. w2[x].. (c1 and c2 can occur)`

**Remarks:**
* A5A and A5B can't arise in histories when P2 is precluded
* **A**nomalies vs. **P**henomena: A is a **subset** of P

## Cursor Stability

* Goal: prevent **lost update**
	* P4: `r1[x] -> w2[x] -> w1[x]` (the `w1[x]` depends on `r1[x]`)
* Cursor lock: the lock is acquired when the cursor moves to a row and is **held for as long as the cursor remains positioned on that row**.

## Snapshot Isolation

```
Time:------------- t1 ------------------ t2 -------------->
               start timestamp      commit timestamp
```

* All reads see a snapshot of data as of the time the transaction started (t1).
* The transaction can be committed if the records in the write set do not overlap with other transactions between t1 and t2.
* At commit time, apply all writes with timestamp t2.
* Write skew is possible:
	* History example: `r1[x=50] r1[y=50] r2[x=50] r2[y=50] w1[y=-40] w2[x=-40] c1 c2`
* It cannot experience an A3 anomaly. Because it always sees the same set of old data/never sees the updates of concurrent transactions.
* However, it does not preclude P3.

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

### Q. Read skew vs write skew

- **Read Skew:** A single transaction **reads data from different logical points in time** (before and after another transaction commits), resulting in an inconsistent view of the database—like seeing half of a completed bank transfer.
    
- **Write Skew:** Two concurrent transactions **read the same (or overlapping) state**, but then make **disjoint writes** (they update _different_ variables) based on that initial read. Because they aren't trying to overwrite the same piece of data, the database doesn't detect a direct conflict, but their combined writes violate **a broader logical constraint**.

### Q. An example of write skew

The "**Doctors On-Call**" scenario:

- **Constraint:** A hospital requires at least one doctor to be on call at all times ($x + y \ge 1$).
- **Initial State:** Both Doctor $x$ and Doctor $y$ are on call ($x=1$, $y=1$).
    
Now, both doctors request to go off duty concurrently:

1. **$T_1$ (Doctor $y$'s transaction) reads $x$:** $T_1$ checks if $x$ is on call. It sees $x=1$ ($r_1[x]$). Since $x$ is on call, it is safe for $y$ to leave.
2. **$T_2$ (Doctor $x$'s transaction) reads $y$:** $T_2$ checks if $y$ is on call. It sees $y=1$ ($r_2[y]$). Since $y$ is on call, it is safe for $x$ to leave.
3. **$T_1$ updates $y$:** $T_1$ sets $y=0$ ($w_1[y]$).
4. **$T_2$ updates $x$:** $T_2$ sets $x=0$ ($w_2[x]$).
5. **Both Commit:** Both transactions commit successfully.
    

**The Result:** The final state is $x=0, y=0$. There are **zero** doctors on call. The overarching constraint ($x + y \ge 1$) was violated because the system allowed a **Write Skew**, which true **serializable** isolation would have prevented by forcing the transactions to execute sequentially.

### Q. An example of read skew

a banking scenario checking total balances:

- **Constraint:** The total funds between Account $x$ and Account $y$ must be accurately reported.
- **Initial State:** Account $x$ has **$50** and Account $y$ has **$50** (Total = **$100**).
    
Now, an auditor checks the balances while a transfer happens concurrently:

1. **$T_1$ (Auditor) reads $x$:** $T_1$ reads Account $x$ and sees $50$ ($r_1[x]$).
2. $T_2$ (**Transfer**) **updates** $x$ and $y$:  $T_2$ transfers **$10** from $x$ to $y$. It writes $x$ = $40$ ($w_2[x]$) and writes $y$ = $60$ ($w_2[y]$).
3. **$T_2$ Commits:** The transfer completes successfully ($c_2$).
4. **$T_1$ (Auditor) reads $y$:** $T_1$ continues its audit and reads Account $y$. Because $T_2$ already committed, $T_1$ sees the new value of **$60$ ($r_1[y]$).
5. **The Result:** The auditor calculates the total as **$50** (old $x$) + **$60** (new $y$) = **$110**.
    
The auditor has observed an inconsistent state (a read skew) where money seemingly appeared out of nowhere. 

Stronger isolation levels, like **Repeatable Read** or **Snapshot Isolation**, prevent this by ensuring a transaction sees a consistent snapshot of the database from **a single point in time**.