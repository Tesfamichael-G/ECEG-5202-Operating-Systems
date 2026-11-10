# Chapter 5 — Deadlocks

!!! abstract "Chapter at a glance"
    **Coverage:** introduction · deadlock prevention · deadlock avoidance · deadlock detection and recovery.

---

## Learning outcomes

1. State the four necessary conditions for deadlock.
2. Explain how breaking each condition corresponds to a deadlock-prevention strategy.
3. Use the Banker's algorithm for deadlock avoidance.
4. Construct a wait-for graph and detect deadlock.

---

## 5.1 Introduction

!!! note "Definition"
    A **deadlock** is a set of blocked processes, each holding a resource and waiting to acquire a resource held by another process in the set.

### 5.1.1 Four necessary conditions (Coffman conditions)

A deadlock can occur **only if all four** of the following hold:

1. **Mutual exclusion** — at least one resource is held in a non-shareable mode.
2. **Hold and wait** — a process holding a resource is waiting for additional resources held by other processes.
3. **No preemption** — resources cannot be forcibly taken from a process.
4. **Circular wait** — a circular chain of processes exists where each waits for a resource held by the next.

Breaking any one condition prevents deadlock.

---

## 5.2 Deadlock prevention

| Condition | Prevention strategy |
|---|---|
| **Mutual exclusion** | Make resources shareable (e.g., read-only files) — often impractical |
| **Hold and wait** | Require a process to request all resources at once, or release all before requesting new |
| **No preemption** | Allow preemption of resources (e.g., save/restore process state) |
| **Circular wait** | Impose a total ordering on resources; each process requests resources in increasing order |

**Trade-offs.** Prevention is safe but often hurts utilisation and throughput (e.g., requesting all resources up-front wastes them).

---

## 5.3 Deadlock avoidance

Avoidance requires **advance knowledge** of each process's maximum resource needs. The system grants a request only if the resulting state remains **safe** — i.e., there exists a **safe sequence** in which all processes can complete.

### 5.3.1 Banker's algorithm

**Data structures** (for $n$ processes, $m$ resource types):

- `Available[m]` — currently available resources.
- `Max[n][m]` — maximum demand of each process.
- `Allocation[n][m]` — current allocation.
- `Need[n][m] = Max − Allocation`.

**Safety algorithm.**

```
Work = Available
Finish = [false, ..., false]
while (there exists i with Finish[i] == false and Need[i] <= Work):
    Work = Work + Allocation[i]
    Finish[i] = true
return all Finish == true
```

**Resource-request algorithm.** When $P_i$ requests `Request[i]`:

1. If `Request[i] > Need[i]`, error.
2. If `Request[i] > Available`, $P_i$ must wait.
3. Pretend to allocate: `Available −= Request`, `Allocation[i] += Request`, `Need[i] −= Request`.
4. Run the safety algorithm. If safe, grant; if unsafe, roll back and make $P_i$ wait.

---

## Worked example 5.1 — Banker's algorithm

**System.** Three resource types A, B, C with total instances (10, 5, 7).

| Process | Allocation | Max | Need |
|---|---|---|---|
| P0 | (0,1,0) | (7,5,3) | (7,4,3) |
| P1 | (2,0,0) | (3,2,2) | (1,2,2) |
| P2 | (3,0,2) | (9,0,2) | (6,0,0) |
| P3 | (2,1,1) | (2,2,2) | (0,1,1) |
| P4 | (0,0,2) | (4,3,3) | (4,3,1) |

Currently allocated: $(0+2+3+2+0,\ 1+0+0+1+0,\ 0+0+2+1+2) = (7, 2, 5)$.
Available: $(10-7,\ 5-2,\ 7-5) = (3, 3, 2)$.

**Safety check.**

- `Work = (3, 3, 2)`.
- **P1** needs $(1,2,2) \le (3,3,2)$ → finish; `Work += (2,0,0)` → $(5,3,2)$.
- **P3** needs $(0,1,1) \le (5,3,2)$ → finish; `Work += (2,1,1)` → $(7,4,3)$.
- **P4** needs $(4,3,1) \le (7,4,3)$ → finish; `Work += (0,0,2)` → $(7,4,5)$.
- **P0** needs $(7,4,3) \le (7,4,5)$ → finish; `Work += (0,1,0)` → $(7,5,5)$.
- **P2** needs $(6,0,0) \le (7,5,5)$ → finish; `Work += (3,0,2)` → $(10,5,7)$.

**Safe sequence:** $\langle P_1, P_3, P_4, P_0, P_2 \rangle$. **System is in a safe state.**

---

## 5.4 Deadlock detection and recovery

If prevention and avoidance are too costly, the system can allow deadlock and **detect** it periodically.

### 5.4.1 Single instance per resource type — wait-for graph

Build a graph with processes as nodes and edges $P_i \to P_j$ if $P_i$ waits for a resource held by $P_j$. The system has a deadlock **iff** the graph contains a cycle.

Cycle detection: DFS or topological sort — $O(V + E)$.

### 5.4.2 Multiple instances — detection algorithm

Use a variant of the Banker's safety algorithm:

- `Work = Available`.
- `Finish[i] = false` if `Allocation[i] > 0`, else `true`.
- Repeatedly find $i$ with `Finish[i] == false` and `Request[i] ≤ Work`; set `Work += Allocation[i]`, `Finish[i] = true`.
- Deadlock exists iff some `Finish[i]` remains `false`.

### 5.4.3 Recovery

| Approach | Description |
|---|---|
| **Process termination** | Kill all deadlocked processes, or kill one at a time until the cycle breaks |
| **Resource preemption** | Take resources from some processes; roll back to a safe state; avoid starvation by tracking how many times a process has been rolled back |

---

## 5.5 Quiz

1. State the four necessary conditions for deadlock.
2. How does resource ordering prevent deadlock?
3. What is a safe state? How does the Banker's algorithm use it?
4. Construct a wait-for graph for a given scenario and detect a deadlock.
5. Compare deadlock prevention, avoidance, and detection on overhead and resource utilisation.

---

## Answer key

??? success "Click to reveal"

    1. Mutual exclusion, hold-and-wait, no preemption, circular wait.
    2. By imposing a total order on resources, a cycle in the wait-for graph becomes impossible.
    3. A safe state is one from which there exists a sequence of process completions that lets every process finish. The Banker's algorithm grants a request only if the resulting state is safe.
    4. (See §5.4.1.)
    5. Prevention: highest overhead, lowest utilisation. Avoidance: medium overhead, medium utilisation. Detection: lowest steady-state overhead, but recovery is expensive.

---

## Further reading

- Tanenbaum §6.1–§6.4.
- Silberschatz §7.1–§7.6.