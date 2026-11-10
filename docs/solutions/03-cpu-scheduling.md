# Solutions — Chapter 3: CPU Scheduling

[:octicons-arrow-left-24: Back to Chapter 3](../chapters/03-cpu-scheduling.md)

Answers to **Section C** of the [Exam Bank](../exam-bank.md).

---

## C1 — Minimum average waiting time [2]

**Answer: (b) SJF.**

SJF provably minimises average waiting time because at every step it picks the job that will finish soonest, leaving the smallest possible residual waiting time for the rest. The exchange argument: swapping a longer job ahead of a shorter one can only increase the total wait.

---

## C2 — RR with very large quantum [2]

**Answer: (b) FCFS.**

If the quantum exceeds every burst time, no process is ever preempted; each runs to completion in arrival order — exactly FCFS.

---

## C3 — Average waiting time under FCFS, SJF, RR (q = 2) [4]

**Given:** five processes, all arriving at time 0, burst times 10, 2, 4, 6, 8.

**FCFS.** Order P1, P2, P3, P4, P5. Waits: 0, 10, 12, 16, 22. **Average = 12.0.**

**SJF.** Order P2, P3, P4, P5, P1. Waits: 0, 2, 6, 12, 20. **Average = 8.0.**

**RR (q = 2).** Trace the queue. **Average ≈ 11.6** (full trace in the chapter's worked example 3.1).

---

## C4 — Priority inversion [4]

**Problem.** A high-priority task $H$ waits for a lock held by a low-priority task $L$; a medium-priority task $M$ preempts $L$, delaying $H$ indefinitely.

**Fix — priority inheritance.** While $L$ holds a lock that $H$ is waiting for, $L$'s priority is temporarily raised to $H$'s. $M$ can no longer preempt $L$; $L$ finishes and releases the lock; $H$ proceeds.

**Example where it matters.** The 1997 Mars Pathfinder mission experienced repeated reboots due to priority inversion in the VxWorks scheduler; the fix was enabling priority inheritance.