---
revealjs:
  mode: diapo
  highlight:
    light: vs
    dark:  vs2015
  autoAnimate:  true
  autoAnimateDuration: 1.0
  transition: slide
  transitionSpeed: default
  js: assets/lecture-03.js

---

# ECEG-5202

## Chapter 3 — CPU Scheduling

---

## Learning outcomes

<div class="fragment">

1. Distinguish **preemptive** from **non-preemptive** scheduling.

</div>

<div class="fragment">

2. Compute the five **scheduling criteria** for a given workload.

</div>

<div class="fragment">

3. Trace **FCFS**, **SJF**, **SRTN**, **Round Robin**, **priority**, and **MLFQ** on a workload.

</div>

<div class="fragment">

4. Analyse **priority inversion** and its fix.

</div>

<div class="fragment">

5. Compare **RM** and **EDF** for real-time scheduling.

</div>

---

## 3.1 — Why CPU Scheduling?

???+ note "The core problem"
    A single-core CPU can run **one process at a time**. Many processes are **ready** to run. The **scheduler** decides which one.

<div class="fragment">

???+ info "The scheduling loop"
    ```
    while (true):
        if (no ready process):
            idle
        else:
            pick = choose(ready_queue)
            dispatch(pick)
            wait for interrupt (I/O, timer, syscall)
    ```

</div>

<div class="fragment">

!!! tip "Multiprogramming keeps the CPU busy"
    Without scheduling, the CPU would idle whenever the running process blocked on I/O. The scheduler switches to another process — this is what makes multiprogramming valuable.

</div>

---

## 3.2 — Preemptive vs Non-Preemptive

???+ note "When is scheduling triggered?"
    The scheduler can run on any of **four events**:
    
    1. **Running → waiting** (I/O request).
    2. **Running → ready** (interrupt / preemption).
    3. **Waiting → ready** (I/O completes).
    4. **Running → terminated** (exit).

<div class="fragment">

???+ note "Two modes"
    | Mode | Scheduling happens on |
    |---|---|
    | **Non-preemptive** | Events 1 and 4 only (running process must yield) |
    | **Preemptive** | All four events (OS can interrupt) |

</div>

<div class="fragment">

???+ info "Examples"
    - **Non-preemptive** — old Windows 3.1, basic batch systems.
    - **Preemptive** — modern Linux, Windows, macOS. **All interactive OSes are preemptive.**

</div>

<div class="fragment">

!!! warning "Preemption has costs"
    Every preemption = one context switch = cache pollution + TLB flush. Preemptive schedulers pay overhead for responsiveness.

</div>

---

## 3.3 — The Five Scheduling Criteria

???+ note "Definitions (Silberschatz §5.2)"
    | Criterion | Definition | Goal |
    |---|---|---|
    | **CPU utilisation** | % time CPU is doing useful work | **maximise** |
    | **Throughput** | Processes completed per time unit | **maximise** |
    | **Turnaround time** | Completion time − arrival time | **minimise** |
    | **Waiting time** | Time spent in ready queue | **minimise** |
    | **Response time** | Time from submission to first response | **minimise** |

<div class="fragment">

???+ info "Turnaround vs waiting"
    - **Turnaround** includes I/O wait and CPU time.
    - **Waiting** counts only ready-queue time.
    
    $T_{\text{turnaround}} = T_{\text{waiting}} + T_{\text{CPU}} + T_{\text{I/O}}$.

</div>

<div class="fragment">

!!! warning "No algorithm optimises all five"
    FCFS gives good throughput but poor response. RR gives good response but frequent switches. The choice depends on whether the workload is **batch, interactive, or real-time**.

</div>

---

## 3.4 — First-Come, First-Served (FCFS)

???+ note "The algorithm"
    Processes are served in **arrival order**. Non-preemptive.

```
Ready queue:  [P1] → [P2] → [P3] → [P4]
CPU:          runs P1 to completion, then P2, ...
```

<div class="fragment">

???+ info "Pros and cons"
    - **Pros:** Simple, no starvation, low overhead.
    - **Cons:** **Convoy effect** — a long process at the head of the queue makes all shorter processes wait.

</div>

<div class="fragment">

???+ note "The convoy effect"
    | Process | Burst |
    |---|---|
    | P1 | 100 |
    | P2 | 1 |
    | P3 | 1 |
    | P4 | 1 |
    
    FCFS order: P1, P2, P3, P4. Wait times: P1=0, P2=100, P3=101, P4=102.
    
    **Average wait = 75.75.** SJF would give 0.75.

</div>

---

## 3.5 — Shortest Job First (SJF)

???+ note "The algorithm"
    Choose the process with the **smallest burst time**. Non-preemptive.

<div class="fragment">

???+ abstract "Theorem (Silberschatz §5.3)"
    **SJF is optimal** for average waiting time among non-preemptive algorithms.

</div>

<div class="fragment">

???+ info "Proof sketch (exchange argument)"
    Suppose an optimal schedule has two adjacent jobs A, B with $A.\text{burst} > B.\text{burst}$ and A before B. Swap them:
    
    - Waiting time of A increases by $B.\text{burst}$.
    - Waiting time of B decreases by $A.\text{burst}$.
    - Net change: $B - A < 0$ — the average waiting time **decreases**.
    
    So any schedule with a longer job before a shorter one can be improved → SJF is optimal.

</div>

<div class="fragment">

!!! warning "The practical problem"
    SJF requires **knowing burst times in advance** — impossible in general. Solutions:
    - **User estimation** (type the burst time when submitting).
    - **Exponential averaging** — predict the next burst from past bursts.

</div>

<div class="fragment">

???+ info "Exponential averaging"
    $\tau_{n+1} = \alpha \cdot t_n + (1-\alpha) \cdot \tau_n$
    
    where $t_n$ is the actual burst, $\tau_n$ the previous prediction, and $\alpha \in [0,1]$ (typically $\alpha = 0.5$). This predicts the next burst from a weighted average of the past.

</div>

---

## 3.6 — Shortest Remaining Time Next (SRTN)

???+ note "The algorithm"
    Preemptive version of SJF. When a new process arrives with a **shorter remaining time** than the running process, **preempt**.

<div class="fragment">

???+ info "SJF vs SRTN"
    - **SJF:** choose shortest burst, run to completion.
    - **SRTN:** choose shortest **remaining**, preempt if a shorter job arrives.

</div>

<div class="fragment">

???+ success "SRTN is optimal for average waiting time"
    Among **all** preemptive or non-preemptive algorithms, SRTN minimises average waiting time.

</div>

<div class="fragment">

!!! warning "SRTN can starve long jobs"
    If short jobs keep arriving, a long job may never run. This is **starvation** — the price of optimal average wait.

</div>

---

## Worked example 3.1 — FCFS vs SJF vs SRTN

!!! tip "Problem:"
    | Process | Arrival | Burst |
    |---|---|---|
    | P1 | 0 | 8 |
    | P2 | 1 | 4 |
    | P3 | 2 | 9 |
    | P4 | 3 | 5 |

<div class="fragment">

???+ note "**FCFS** — order by arrival."

    | P | Wait |
    |---|---|
    | P1 | 0 |
    | P2 | 7 |
    | P3 | 10 |
    | P4 | 17 |
    | **Avg** | **8.5** |

</div>

<div class="fragment">

??? info "**SJF (non-preemptive)** — order by burst."

    Order: P2 (4), P4 (5), P1 (8), P3 (9).

    | P | Arrival | Start | Wait |
    |---|---|---|---|
    | P2 | 1 | 1 | 0 |
    | P4 | 3 | 5 | 2 |
    | P1 | 0 | 10 | 10 |
    | P3 | 2 | 18 | 16 |
    | **Avg** | | | **7.0** |

</div>

<div class="fragment">

??? info "**SRTN** — preempt when shorter arrives."

    - t=0: P1 starts (remaining 8).
    - t=1: P2 arrives (4 < 7). Preempt P1, run P2.
    - t=5: P2 done. P4 (5) vs P1 (7) vs P3 (9). Run P4.
    - t=10: P4 done. P1 (7) vs P3 (9). Run P1.
    - t=17: P1 done. Run P3.

    | P | Arrival | Start | Wait |
    |---|---|---|---|
    | P2 | 1 | 1 | 0 |
    | P4 | 3 | 5 | 2 |
    | P1 | 0 | 10 | 10 |
    | P3 | 2 | 17 | 15 |
    | **Avg** | | | **6.75** |

</div>

<div class="fragment">

??? success "**Observation.**"
    SRTN (6.75) < SJF (7.0) < FCFS (8.5). Preemption buys a small improvement but risks starvation.

</div>

---

## 3.7 — Round Robin (RR)

???+ note "The algorithm"
    Each process gets a **time quantum** $q$. When the quantum expires, the process is **preempted** and placed at the **tail** of the ready queue.

```
Ready queue: [P1] → [P2] → [P3] → [P4]
CPU: runs P1 for q, then P2 for q, ...  (circular)
```

<div class="fragment">

???+ info "Behaviour depends on $q$"
    | $q$ | Behaviour |
    |---|---|
    | **Very small** ($q <$ context switch) | Context switch overhead dominates |
    | **Too large** ($q >$ burst times) | Degenerates to FCFS |
    | **Just right** | Fair, responsive, low overhead |

</div>

<div class="fragment">

???+ tip "Rule of thumb"
    $q$ should be **large enough** that 80% of CPU bursts are shorter than $q$. Typical values: **10–100 ms** for interactive systems.

</div>

<div class="fragment">

???+ info "Pros and cons"
    - **Pros:** Fair (no starvation), responsive (short jobs finish quickly), simple.
    - **Cons:** Higher average waiting time than SJF; context switch overhead.

</div>

---

## Worked example 3.2 — Round Robin trace

!!! tip "Problem:"
    Same workload as 3.1: P1(0, 8), P2(1, 4), P3(2, 9), P4(3, 5). $q = 4$.

<div class="fragment">

???+ note "**Timeline.**"

    | Time | Running | Ready queue |
    |---|---|---|
    | 0–4 | P1 | P2, P3, P4, P1 |
    | 4–8 | P2 | P3, P4, P1 |
    | 8–12 | P3 | P4, P1, P3 |
    | 12–16 | P4 | P1, P3, P4 |
    | 16–20 | P1 | P3, P4 |
    | 20–24 | P3 | P4, P3 |
    | 24–26 | P4 | P3 |
    | 26–29 | P3 | — |

</div>

<div class="fragment">

??? info "**Compute waiting times.**"
    - P1: finishes at 20. Burst 8. Waiting = $20 - 0 - 8 = 12$. 
    - P2: finishes at 8. Burst 4. Waiting = $8 - 1 - 4 = 3$. 
    - P3: finishes at 29. Burst 9. Waiting = $29 - 2 - 9 = 18$. 
    - P4: finishes at 26. Burst 5. Waiting = $26 - 3 - 5 = 18$.

    **Average wait = (12 + 3 + 18 + 18) / 4 = 12.75.**

</div>

<div class="fragment">

??? success "**Observation.**"
    RR's average wait (12.75) is **worse** than SJF (7.0) — but RR gives every process a fair share. This is the price of **responsiveness**.

</div>

---

## 3.8 — Priority Scheduling

???+ note "The algorithm"
    Each process has a **priority**. The highest-priority ready process runs. Can be preemptive or non-preemptive.

<div class="fragment">

???+ info "Priority assignment"
    - **Static** — assigned once at creation (e.g., "system process has priority 0, user priority 50").
    - **Dynamic** — changes with behaviour (e.g., I/O-bound gets higher priority).

</div>

<div class="fragment">

???+ warning "Priority inversion"
    A **high-priority** task waits for a resource held by a **low-priority** task, while a **medium-priority** task preempts the low-priority holder. The high-priority task effectively waits for the medium-priority task.

</div>

---

## Worked example 3.3 — Priority inversion in detail

!!! tip "Problem:"
    Three tasks:
    - **H** (high priority) needs a lock L.
    - **M** (medium priority) is CPU-bound, no locks.
    - **L** (low priority) holds L.

    Trace the scenario.

<div class="fragment">

???+ note "**Step 1 — L acquires L.**"
    L starts, acquires L, and begins work.

</div>

<div class="fragment">

??? info "**Step 2 — H preempts.**"
    H arrives, preempts L. H tries to acquire L — blocks because L holds it. L is ready to resume.

</div>

<div class="fragment">

??? info "**Step 3 — M arrives.**"
    M preempts L (M > L in priority). L cannot run. L cannot release L. H waits.

</div>

<div class="fragment">

??? info "**Step 4 — the inversion.**"
    H, the highest priority, is effectively waiting for M, the medium priority — through the low-priority L. This violates the priority ordering.

</div>

<div class="fragment">

??? success "**The fix — priority inheritance.**"
    When H blocks on L, temporarily raise **L's priority to H's level**. Now M cannot preempt L. L runs, releases L, drops back to low priority. H proceeds.

</div>

<div class="fragment">

!!! tip "The Mars Pathfinder (1997)"
    The Pathfinder lander experienced repeated reboots due to priority inversion in VxWorks. The fix was a one-line change: enable priority inheritance. Famous case study in real-time systems.

</div>

---

## 3.9 — Multilevel Feedback Queue (MLFQ)

???+ note "The algorithm"
    Multiple ready queues with different priorities and time quanta. Processes **move between queues** based on behaviour.

```
Q0 (quantum 8, highest)   ← new processes start here
Q1 (quantum 16)
Q2 (quantum 32, lowest)   ← CPU-bound processes end up here
```

<div class="fragment">

???+ note "The three rules"
    1. If priority(A) > priority(B), run A.
    2. If priority(A) = priority(B), round robin.
    3. **New processes start at the top** (highest priority).
    4. **If a process uses its full quantum, drop one level.**
    5. **Periodically boost all processes** to the top (aging).

</div>

<div class="fragment">

???+ info "What MLFQ optimises for"
    - **Interactive processes** (short bursts) stay at the top → fast response.
    - **CPU-bound processes** sink to the bottom → large quanta, low overhead.
    - **Periodic boost** prevents starvation.

</div>

<div class="fragment">

!!! tip "Modern examples"
    Windows uses a variant of MLFQ with **priority boosts** for I/O-completing processes (a process that just finished I/O gets a temporary priority boost — it's likely interactive).

</div>

---

## 3.10 — Multiprocessor Scheduling

???+ note "The two approaches"
    | Approach | Description |
    |---|---|
    | **Asymmetric** | One master CPU schedules; the others execute |
    | **Symmetric (SMP)** | Each CPU schedules itself; shared ready queue |

<div class="fragment">

???+ info "SMP load balancing"
    - **Push migration** — a CPU periodically pushes tasks to underloaded CPUs.
    - **Pull migration** — an idle CPU steals tasks from overloaded CPUs.

</div>

<div class="fragment">

???+ note "Processor affinity"
    - **Soft affinity** — the scheduler tries to keep a process on the same CPU.
    - **Hard affinity** — the process is bound to a specific CPU (e.g., `taskset` in Linux).

</div>

<div class="fragment">

!!! tip "Why affinity matters"
    When a process migrates between CPUs, its working set stays in the old CPU's cache. Re-populating the new CPU's cache costs thousands of cycles. Affinity keeps the cache warm.

</div>

---

## 3.11 — Real-Time Scheduling

???+ note "Definition"
    A **real-time system** must meet **deadlines** — a result that arrives late is as bad as no result.

<div class="fragment">

???+ note "Two categories"
    | Category | Consequence of missing deadline |
    |---|---|
    | **Hard real-time** | Catastrophic (airbag, pacemaker) |
    | **Soft real-time** | Degraded quality (video streaming, VoIP) |

</div>

<div class="fragment">

???+ note "Two algorithms"
    | Algorithm | Priority | Optimal? |
    |---|---|---|
    | **Rate Monotonic (RM)** | Static — shorter period = higher priority | Optimal among static-priority |
    | **Earliest Deadline First (EDF)** | Dynamic — earliest deadline first | Optimal among all preemptive |

</div>

<div class="fragment">

???+ info "Schedulability tests"
    For $n$ periodic tasks with utilisation $U = \sum C_i / T_i$:
    - **RM** is schedulable if $U \le n(2^{1/n} - 1)$. As $n \to \infty$, this approaches $\ln 2 \approx 0.693$.
    - **EDF** is schedulable iff $U \le 1$.

</div>

---

## Worked example 3.4 — RM vs EDF

!!! tip "Problem:"
    Three periodic tasks:
    - Task A: $C=2, T=5$.
    - Task B: $C=2, T=7$.
    - Task C: $C=3, T=10$.

<div class="fragment">

???+ note "**Step 1 — utilisation.**"
    $U = 2/5 + 2/7 + 3/10 = 0.4 + 0.286 + 0.3 = 0.986$.

</div>

<div class="fragment">

??? info "**Step 2 — RM schedulable?**"
    $n(2^{1/n} - 1) = 3(2^{1/3} - 1) \approx 3(1.2599 - 1) = 0.78$.
    
    $U = 0.986 > 0.78$ → **RM not guaranteed** (may or may not be schedulable — the test is sufficient, not necessary).

</div>

<div class="fragment">

??? info "**Step 3 — EDF schedulable?**"
    $U = 0.986 \le 1$ → **EDF is schedulable**. ✓

</div>

<div class="fragment">

??? success "**Conclusion.**"
    EDF succeeds where RM fails. This is expected — EDF is optimal, RM is not. But EDF is harder to implement (dynamic priorities), and under overload, EDF's behaviour is "domino effect" — once a deadline is missed, subsequent ones often miss.

</div>

<div class="fragment">

!!! tip "RM's advantage under overload"
    Under overload, RM **fails gracefully** — high-priority tasks keep meeting deadlines; low-priority ones miss. EDF **fails catastrophically** — everything misses. This is why RM (with a lower utilisation bound) is preferred in safety-critical systems.

</div>

---

## 3.12 — Linux's CFS: A Modern Case Study

???+ note "Completely Fair Scheduler (CFS)"
    Linux's default scheduler (since 2.6.23). Key idea: **track virtual runtime** — the amount of CPU time each process has received, weighted by priority.

<div class="fragment">

???+ note "The algorithm"
    - Each process has a **vruntime** (virtual runtime).
    - The scheduler always runs the process with the **smallest vruntime**.
    - Uses a **red-black tree** to find the minimum in $O(\log n)$.
    - When a process is preempted, its vruntime is updated.

</div>

<div class="fragment">

???+ info "Why it's "fair""
    Over time, all processes get approximately equal vruntime — meaning equal CPU share. Priority is implemented as a **weight**: a nice -20 process's vruntime advances 1/20th as fast as a nice 0 process, so it gets 20× more CPU.

</div>

<div class="fragment">

!!! tip "CFS replaced the O(1) scheduler"
    The previous O(1) scheduler used a fixed-priority multilevel queue. It scaled well but had poor interactive responsiveness. CFS's virtual-runtime model gives smooth fairness without explicit priority boosting.

</div>

---

## Chapter 3 — key takeaways

<div class="fragment">

**1.** **Preemptive** schedulers can interrupt the running process; **non-preemptive** cannot. All interactive OSes are preemptive.

</div>

<div class="fragment">

**2.** Five criteria: **utilisation**, **throughput**, **turnaround**, **waiting**, **response** — no algorithm optimises all.

</div>

<div class="fragment">

**3.** **FCFS** suffers from convoy effect. **SJF** is optimal for average wait but requires burst-time knowledge. **SRTN** is optimal preemptively but can starve.

</div>

<div class="fragment">

**4.** **Round Robin** gives fairness and response time; the quantum trades overhead against responsiveness.

</div>

<div class="fragment">

**5.** **MLFQ** adapts to behaviour — interactive processes stay high, CPU-bound sink. Periodic boost prevents starvation.

</div>

<div class="fragment">

**6.** **Priority inversion** (H waits for L via M) is fixed by **priority inheritance**.

</div>

<div class="fragment">

**7.** **RM** is optimal among static-priority; **EDF** is optimal overall. RM fails gracefully; EDF catastrophically.

</div>

---

## Next — Chapter 4

**Inter-Process Communication and Synchronization.**

<div class="fragment">

- **Race conditions** — the fundamental problem.
- **Critical section** — the three requirements.
- **Peterson's solution** — software mutex for two processes.
- **Hardware primitives** — test-and-set, compare-and-swap.
- **Semaphores** and **monitors**.
- **Classical problems** — producer-consumer, readers-writers, dining philosophers.

</div>

<div class="fragment">

Chapter 3 scheduled processes on the CPU. Chapter 4 coordinates them — because when processes share data, **chaos** is the default.

</div>