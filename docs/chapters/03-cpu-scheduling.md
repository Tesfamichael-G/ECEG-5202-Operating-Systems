# Chapter 3 — CPU Scheduling

!!! abstract "Chapter at a glance"
    **Coverage:** basic concepts · scheduling criteria · scheduling algorithms · multiple-processor scheduling · real-time scheduling.

---

## Learning outcomes

1. Explain why CPU scheduling is necessary in a multiprogrammed system.
2. Define the scheduling criteria: CPU utilisation, throughput, turnaround time, waiting time, response time.
3. Trace FCFS, SJF, SRTN, Round Robin, priority, and multilevel feedback queue on a given workload.
4. Describe multiprocessor and real-time scheduling (RM, EDF).

---

## 3.1 Basic concepts

The **CPU scheduler** selects a process from the ready queue and allocates the CPU to it.

Scheduling decisions occur when a process:

1. Switches from running to waiting (I/O request).
2. Switches from running to ready (interrupt).
3. Switches from waiting to ready (I/O completes).
4. Terminates.

If scheduling is only possible at (1) and (4), the scheduler is **non-preemptive** (cooperative). If scheduling is also allowed at (2) and (3), the scheduler is **preemptive**.

---

## 3.2 Scheduling criteria

| Criterion | Definition | Goal |
|---|---|---|
| **CPU utilisation** | Fraction of time the CPU is busy | maximise |
| **Throughput** | Processes completed per time unit | maximise |
| **Turnaround time** | Completion time − arrival time | minimise |
| **Waiting time** | Time spent in the ready queue | minimise |
| **Response time** | Time from submission to first response | minimise |

No single algorithm optimises all criteria — the choice is a trade-off.

---

## 3.3 Scheduling algorithms

### 3.3.1 First-Come, First-Served (FCFS)

Processes are scheduled in arrival order.

- **Non-preemptive.**
- **Convoy effect:** a long process at the head blocks all shorter processes.
- **Average waiting time:** depends on arrival order.

### 3.3.2 Shortest Job First (SJF)

Schedule the process with the smallest burst time.

- **Non-preemptive.**
- **Optimal** for average waiting time (provably).
- **Problem:** burst time is not known in advance — must be estimated (exponential averaging).

### 3.3.3 Shortest Remaining Time Next (SRTN)

Preemptive version of SJF: if a new process arrives with a shorter remaining time than the current process, preempt.

- **Preemptive.**
- **Optimal** for average waiting time among preemptive algorithms.

### 3.3.4 Round Robin (RR)

Each process gets a **time quantum** $q$; when it expires, the process is preempted and placed at the tail of the ready queue.

- **Preemptive.**
- Fair — no starvation.
- **Problem:** if $q$ is too small, context-switch overhead dominates; if too large, degenerates to FCFS.
- **Rule of thumb:** $q$ should be large enough that 80% of CPU bursts are shorter than $q$.

### 3.3.5 Priority scheduling

Each process has a priority; the highest-priority ready process runs.

- Can be preemptive or non-preemptive.
- **Problem:** **starvation** — low-priority processes may never run.
- **Fix:** **aging** — gradually increase the priority of waiting processes.

### 3.3.6 Multilevel feedback queue (MLFQ)

Multiple queues with different priorities and time quanta. Processes move between queues based on behaviour.

- Adapts to process behaviour without prior knowledge.
- **Problem:** many parameters to tune; vulnerable to gaming.

---

## Worked example 3.1 — FCFS vs. SJF vs. RR

**Workload.** Five processes arrive at time 0 with burst times: $P_1=10, P_2=2, P_3=4, P_4=6, P_5=8$.

**FCFS.** Execution order $P_1, P_2, P_3, P_4, P_5$.

| Process | Wait |
|---|---|
| P1 | 0 |
| P2 | 10 |
| P3 | 12 |
| P4 | 16 |
| P5 | 22 |
| **Average** | **12.0** |

**SJF.** Order $P_2, P_3, P_4, P_5, P_1$.

| Process | Wait |
|---|---|
| P2 | 0 |
| P3 | 2 |
| P4 | 6 |
| P5 | 12 |
| P1 | 20 |
| **Average** | **8.0** |

**Round Robin (q = 2).**

| Time | Running | Ready queue |
|:---:|---|---|
| 0–2 | P1 | P2, P3, P4, P5, P1 |
| 2–4 | P2 | P3, P4, P5, P1 |
| 4–6 | P3 | P4, P5, P1, P3 |
| 6–8 | P4 | P5, P1, P3, P4 |
| 8–10 | P5 | P1, P3, P4, P5 |
| 10–12 | P1 | P3, P4, P5, P1 |
| ... | ... | ... |

Average waiting time ≈ **11.6**.

---

## 3.4 Multiple-processor scheduling

Two approaches:

- **Asymmetric multiprocessing** — one processor (the master) makes all scheduling decisions; the others execute.
- **Symmetric multiprocessing (SMP)** — each processor schedules itself; a shared ready queue or per-processor queues.

**Processor affinity:** keeping a process on the same CPU improves cache hit rate — **soft affinity** (preferred but not required) vs. **hard affinity** (required).

**Load balancing:** push migration (periodically move tasks from overloaded CPUs) vs. pull migration (idle CPUs steal tasks).

---

## 3.5 Real-time scheduling

Real-time systems have **deadlines**. Two algorithms:

- **Rate Monotonic (RM):** static priorities; shorter periods get higher priority. **Optimal** among static-priority algorithms.
- **Earliest Deadline First (EDF):** dynamic priorities; the process with the earliest deadline runs first. **Optimal** among all preemptive algorithms.

For a set of $n$ periodic tasks with utilisation $U = \sum C_i / T_i$:

- RM is schedulable if $U \le n(2^{1/n} - 1)$ (Liu & Layland bound).
- EDF is schedulable iff $U \le 1$.

---

## 3.6 Quiz

1. List the five scheduling criteria. Which are maximised and which are minimised?
2. What is the convoy effect? Which algorithm suffers from it?
3. Why is SJF optimal for average waiting time? What is its practical problem?
4. How does Round Robin avoid starvation?
5. Explain priority inversion and how priority inheritance fixes it.

---

## Answer key

??? success "Click to reveal"

    1. CPU utilisation (max), throughput (max), turnaround (min), waiting (min), response (min).
    2. FCFS: a long process at the head of the queue makes all shorter processes wait.
    3. SJF minimises $\sum w_i$ by always picking the smallest job; it requires knowing burst times in advance.
    4. Every process gets CPU time within one quantum cycle.
    5. A high-priority task waits for a low-priority task holding a lock; a medium-priority task preempts the low-priority one, delaying the high-priority task. Priority inheritance temporarily raises the lock holder's priority to the highest waiter's priority.

---

## Further reading

- Tanenbaum §2.4.
- Silberschatz §5.1–§5.5.