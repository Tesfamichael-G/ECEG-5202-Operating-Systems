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
  js: assets/lecture-05.js

---

# ECEG-5202

## Chapter 5 — Deadlocks

---

## Learning outcomes

<div class="fragment">

1. State the **four Coffman conditions** and explain why each is necessary.

</div>

<div class="fragment">

2. Draw a **resource allocation graph** and detect a deadlock in it.

</div>

<div class="fragment">

3. Apply **deadlock prevention** by breaking one condition.

</div>

<div class="fragment">

4. Execute the **Banker's algorithm** for deadlock avoidance.

</div>

<div class="fragment">

5. Use **wait-for graphs** and detection algorithms to find deadlocks.

</div>

<div class="fragment">

6. Compare **prevention**, **avoidance**, and **detection** on overhead and utilisation.

</div>

---

## 5.1 — What is a Deadlock?

???+ note "Definition (Silberschatz §7.1)"
    A **deadlock** is a set of blocked processes, each holding a resource and waiting to acquire a resource held by another process in the set.

<div class="fragment">

???+ note "The simplest example"
    Two processes, two resources:
    
    | Time | P1 | P2 |
    |---|---|---|
    | 1 | acquires R1 | |
    | 2 | | acquires R2 |
    | 3 | waits for R2 | |
    | 4 | | waits for R1 |
    
    Both processes wait forever. **Deadlock.**

</div>

<div class="fragment">

!!! warning "Deadlock requires real resources"
    Resources must be **unshareable** (mutex-held). Shareable resources (read-only files, CPU time slices) cannot cause deadlock. The problem is exclusive access to finite resources.

</div>

---

## 5.2 — System Resource Model

???+ note "Resource types and instances"
    A system has $m$ **resource types**, each with $W_i$ **instances**.
    
    - **Resource type:** printer, disk, mutex, semaphore.
    - **Instance:** each individual printer, each individual disk block.

<div class="fragment">

???+ note "Process's use of resources"
    A process uses a resource in three phases:
    
    1. **Request** — ask for the resource; wait if not available.
    2. **Use** — hold the resource; do work.
    3. **Release** — give the resource back.

</div>

<div class="fragment">

???+ info "Formal model"
    - $n$ processes.
    - $m$ resource types, with $W_i$ instances of type $i$.
    - **Allocation matrix** $A$ — $A_{ij}$ = number of type-$j$ resources held by process $i$.
    - **Request matrix** $R$ — $R_{ij}$ = number of type-$j$ resources process $i$ is requesting.
    - **Availability vector** $V$ — $V_j$ = free resources of type $j$.

</div>

---

## 5.3 — The Four Coffman Conditions

???+ abstract "Coffman conditions (1971)"
    A deadlock can occur **only if all four** of the following hold **simultaneously**:

<div class="fragment">

???+ note "1. Mutual exclusion"
    At least one resource is held in a **non-shareable** mode — only one process can use it at a time.

</div>

<div class="fragment">

???+ note "2. Hold and wait"
    A process holding at least one resource is **waiting for additional resources** held by other processes.

</div>

<div class="fragment">

???+ note "3. No preemption"
    Resources **cannot be forcibly taken** from a process. They must be released voluntarily.

</div>

<div class="fragment">

???+ note "4. Circular wait"
    A **circular chain** of processes exists where each waits for a resource held by the next in the chain.

</div>

<div class="fragment">

???+ success "The implication"
    **Breaking any one** of these conditions prevents deadlock. **All four are necessary** — the absence of even one makes deadlock impossible.

</div>

---

## 5.4 — Resource Allocation Graph

???+ note "Definition"
    A **resource allocation graph** (RAG) is a directed graph where:
    - **Circles** are processes.
    - **Rectangles** are resource types (with dots for instances).
    - **Edges:**
      - **Request** — $P \to R$ (dashed, if using explicit request edges).
      - **Assignment** — $R \to P$.

<div class="fragment">

???+ note "Example RAG"
    ```
    P1 ──► R1 ──► P2 ──► R2 ──► P1
    ```
    
    P1 holds R2 (implicitly) and requests R1. P2 holds R1 and requests R2. **Cycle → deadlock** (if one instance per resource).

</div>

<div class="fragment">

???+ abstract "The RAG rule (single instance)"
    In a RAG with **one instance per resource**:
    
    **A cycle is necessary and sufficient for deadlock.**

</div>

<div class="fragment">

???+ warning "The rule breaks for multiple instances"
    With multiple instances per resource, a cycle is **necessary but not sufficient** — the resources in the cycle might be available elsewhere. This is why we need the detection algorithm (Section 5.8).

</div>

---

## Worked example 5.1 — Detect deadlock from a RAG

!!! tip "Problem:"
    ```
    P1 ──► R1 ──► P2
                  │
                  ▼
                  R2 ──► P1
    ```
    Plus: R2 has 2 instances, one held by P1 and one free.

<div class="fragment">

???+ note "**Step 1 — check for cycles.**"
    P1 → R1 → P2 → R2 → P1. **Cycle exists.**

</div>

<div class="fragment">

??? info "**Step 2 — check instances.**"
    R2 has 2 instances; 1 held by P1, 1 free.
    
    P2 requests R2 (1 instance). R2 has a free instance — so P2 can be satisfied. The cycle is **broken** when P2 gets R2's free instance and finishes.

</div>

<div class="fragment">

??? success "**Conclusion.**"
    **No deadlock.** The cycle exists but is **not a deadlock** — a free instance breaks it. This illustrates why the RAG rule requires single instances.

</div>

---

## 5.5 — Deadlock Prevention

???+ note "Prevent by breaking one condition"
    | Condition | How to break | Cost |
    |---|---|---|
    | **Mutual exclusion** | Make resources shareable | Often impossible |
    | **Hold and wait** | Request all resources at once, or release all before requesting new | Low utilisation |
    | **No preemption** | Allow preemption | Requires save/restore state |
    | **Circular wait** | Impose a total order on resources | Restricts programming |

<div class="fragment">

???+ note "Hold-and-wait prevention"
    Two variants:
    1. **Request all at once** — the process requests every resource it will ever need before starting. **Low utilisation** — a process holding resources waits for others.
    2. **Release all before requesting new** — the process releases everything before requesting more. **Restart required** — cannot easily resume.

</div>

<div class="fragment">

???+ note "No-preemption prevention"
    - When a process requests a resource held by another, **preempt** the holder.
    - The preempted process is **rolled back** to a safe state.
    - Works only if the state is checkpointable (e.g., CPU registers, but not a partially written file).

</div>

<div class="fragment">

???+ note "Circular-wait prevention (the practical choice)"
    Assign a **global order** to all resources (e.g., by resource number). Each process must request resources in **increasing order**. This prevents a cycle — if P1 holds R1 (lower) and requests R2 (higher), and P2 holds R2 and requests R1, P2's request violates the ordering.

</div>

<div class="fragment">

!!! tip "The UNIX lock order"
    Linux's kernel uses a **lock order** documented in the source tree — every mutex has a level, and acquiring locks out of order triggers a warning (`might_sleep` / lockdep). This is circular-wait prevention in action.

</div>

---

## 5.6 — Deadlock Avoidance

???+ note "The idea"
    Prevention is static and conservative. **Avoidance** is dynamic: it grants a request **only if the resulting state is safe**.

<div class="fragment">

???+ abstract "Safe state (Silberschatz §7.5)"
    A state is **safe** if there exists a **safe sequence** of processes $\langle P_1, P_2, \ldots, P_n \rangle$ such that:
    
    For each $i$, the resources $P_i$ still needs can be satisfied by the **currently available** resources plus those held by $P_1, \ldots, P_{i-1}$.

</div>

<div class="fragment">

???+ info "Key facts"
    - **Safe state** ≠ **no deadlock** — every safe state is deadlock-free, but some unsafe states are also deadlock-free.
    - A safe state **guarantees** that all processes can finish — by following the safe sequence.
    - An unsafe state **may or may not** deadlock — depends on future requests.

</div>

<div class="fragment">

!!! warning "The conservative nature of avoidance"
    Avoidance may refuse a request even though granting it would not deadlock — because it cannot **guarantee** no future deadlock. This is the price of safety.

</div>

---

## 5.7 — Banker's Algorithm

???+ note "Data structures (Silberschatz §7.5.3)"
    For $n$ processes and $m$ resource types:

    | Structure | Dimensions | Meaning |
    |---|---|---|
    | **Available** | $m$ | Free resources |
    | **Max** | $n \times m$ | Max demand per process |
    | **Allocation** | $n \times m$ | Current allocation |
    | **Need** | $n \times m$ | $Max - Allocation$ |

<div class="fragment">

???+ note "The safety algorithm"
    ```
    Work = Available
    Finish = [false, false, ..., false]

    while (exists i with Finish[i] == false and Need[i] <= Work):
        Work = Work + Allocation[i]
        Finish[i] = true

    return all(Finish)
    ```

</div>

<div class="fragment">

???+ note "The resource-request algorithm"
    When $P_i$ requests `Request[i]`:

    1. If `Request[i] > Need[i]`, error.
    2. If `Request[i] > Available`, $P_i$ must wait.
    3. **Pretend** to allocate: `Available -= Request`, `Allocation[i] += Request`, `Need[i] -= Request`.
    4. Run the safety algorithm.
    5. If **safe**, grant; else **roll back** and make $P_i$ wait.

</div>

---

## Worked example 5.2 — Banker's algorithm safety check

!!! tip "Problem:"
    System with resource types A, B, C, totals (10, 5, 7).
    
    | Process | Allocation | Max | Need |
    |---|---|---|---|
    | P0 | (0,1,0) | (7,5,3) | (7,4,3) |
    | P1 | (2,0,0) | (3,2,2) | (1,2,2) |
    | P2 | (3,0,2) | (9,0,2) | (6,0,0) |
    | P3 | (2,1,1) | (2,2,2) | (0,1,1) |
    | P4 | (0,0,2) | (4,3,3) | (4,3,1) |

<div class="fragment">

???+ note "**Step 1 — available.**"
    Total allocated: $(0+2+3+2+0, 1+0+0+1+0, 0+0+2+1+2) = (7, 2, 5)$.
    
    Available = $(10,5,7) - (7,2,5) = (3, 3, 2)$.

</div>

<div class="fragment">

??? info "**Step 2 — find a satisfying process.**"
    `Work = (3, 3, 2)`. Which process has `Need ≤ Work`?
    
    - P1 needs (1, 2, 2) ≤ (3, 3, 2) ✓. **Finish P1.**
    - `Work = (3, 3, 2) + (2, 0, 0) = (5, 3, 2)`.

</div>

<div class="fragment">

??? info "**Step 3 — continue.**"
    - P3 needs (0, 1, 1) ≤ (5, 3, 2) ✓. **Finish P3.**
    - `Work = (5, 3, 2) + (2, 1, 1) = (7, 4, 3)`.
    - P4 needs (4, 3, 1) ≤ (7, 4, 3) ✓. **Finish P4.**
    - `Work = (7, 4, 3) + (0, 0, 2) = (7, 4, 5)`.
    - P0 needs (7, 4, 3) ≤ (7, 4, 5) ✓. **Finish P0.**
    - `Work = (7, 4, 5) + (0, 1, 0) = (7, 5, 5)`.
    - P2 needs (6, 0, 0) ≤ (7, 5, 5) ✓. **Finish P2.**

</div>

<div class="fragment">

??? success "**Conclusion.**"
    **Safe sequence: ⟨P1, P3, P4, P0, P2⟩.** The system is in a **safe state**.
    
    Any process can now request up to its Need, as long as the resulting state remains safe.

</div>

---

## Worked example 5.3 — Banker's algorithm request

!!! tip "Problem:"
    In the safe state above, P1 requests (1, 0, 2).

<div class="fragment">

???+ note "**Step 1 — validate.**"
    `Request[P1] = (1, 0, 2) ≤ Need[P1] = (1, 2, 2)` ✓.
    `Request[P1] = (1, 0, 2) ≤ Available = (3, 3, 2)` ✓.

</div>

<div class="fragment">

??? info "**Step 2 — pretend allocation.**"
    - Available = (3, 3, 2) − (1, 0, 2) = (2, 3, 0).
    - Allocation[P1] = (2, 0, 0) + (1, 0, 2) = (3, 0, 2).
    - Need[P1] = (1, 2, 2) − (1, 0, 2) = (0, 2, 0).

</div>

<div class="fragment">

??? info "**Step 3 — run safety algorithm.**"
    `Work = (2, 3, 0)`.
    
    - P1 needs (0, 2, 0) ≤ (2, 3, 0) ✓. Finish. `Work = (5, 3, 2)`.
    - P3 needs (0, 1, 1) ≤ (5, 3, 2) ✓. Finish. `Work = (7, 4, 3)`.
    - P4 needs (4, 3, 1) ≤ (7, 4, 3) ✓. Finish. `Work = (7, 4, 5)`.
    - P0 needs (7, 4, 3) ≤ (7, 4, 5) ✓. Finish. `Work = (7, 5, 5)`.
    - P2 needs (6, 0, 0) ≤ (7, 5, 5) ✓. Finish.
    
    All done.

</div>

<div class="fragment">

??? success "**Conclusion.**"
    Request is **safe** → **granted**. P1 now has (3, 0, 2), still needs (0, 2, 0), and the system remains in a safe state.

</div>

---

## 5.8 — Deadlock Detection

???+ note "The idea"
    If prevention and avoidance are too costly, allow deadlock and **detect** it periodically.

<div class="fragment">

???+ note "Single instance per resource — wait-for graph"
    Build a graph with processes as nodes. Edge $P_i \to P_j$ if $P_i$ waits for a resource held by $P_j$.
    
    **Deadlock exists iff the wait-for graph has a cycle.**

</div>

<div class="fragment">

???+ note "Multiple instances — detection algorithm"
    Variant of the safety algorithm:
    
    ```
    Work = Available
    Finish[i] = false if Allocation[i] > 0 else true

    while (exists i with Finish[i] == false and Request[i] <= Work):
        Work = Work + Allocation[i]
        Finish[i] = true

    Deadlock exists iff some Finish[i] == false.
    ```

</div>

<div class="fragment">

!!! warning "Detection cost"
    Running the detection algorithm on every request is **$O(n^2 m)$** — expensive. Real systems run it periodically (e.g., every minute) or when the CPU is idle.

</div>

---

## Worked example 5.4 — Wait-for graph

!!! tip "Problem:"
    ```
    P1 holds R1, requests R2.
    P2 holds R2, requests R3.
    P3 holds R3, requests R1.
    ```

<div class="fragment">

???+ note "**Step 1 — draw the wait-for graph.**"
    ```
    P1 ──► P2 ──► P3 ──► P1
    ```

</div>

<div class="fragment">

??? info "**Step 2 — check for a cycle.**"
    P1 → P2 → P3 → P1. **Cycle exists.**

</div>

<div class="fragment">

??? success "**Conclusion.**"
    **Deadlock** in processes P1, P2, P3.

</div>

<div class="fragment">

!!! tip "Cycle detection is fast"
    Depth-first search on the wait-for graph is $O(V + E)$ — cheap enough to run on every request in single-instance systems.

</div>

---

## 5.9 — Deadlock Recovery

???+ note "Three approaches"
    | Approach | Cost | Risk |
    |---|---|---|
    | **Process termination** | Kill processes | Loss of work |
    | **Resource preemption** | Take resources | Rollback needed |
    | **Process rollback** | Restore checkpoint | Checkpoint overhead |

<div class="fragment">

???+ note "Which process to terminate?"
    - **Kill all** deadlocked processes — simple but drastic.
    - **Kill one at a time** until cycle broken — better but slow.
    - **Priorities:** low priority, least work, fewest resources.

</div>

<div class="fragment">

???+ note "Resource preemption issues"
    1. **Victim selection** — which process to preempt?
    2. **Rollback** — restore the victim to a safe state.
    3. **Starvation** — a process may be preempted repeatedly. Fix: count preemptions, refuse to preempt a process more than $k$ times.

</div>

---

## 5.10 — Comparison: Prevention vs Avoidance vs Detection

???+ note "The trade-off"
    | Aspect | Prevention | Avoidance | Detection |
    |---|---|---|---|
    | **Overhead** | Low (static rules) | Medium (run safety on every request) | Low (periodic detection) |
    | **Resource utilisation** | Low (over-conservative) | Medium | High |
    | **Deadlock possible?** | No | No | Yes, until detected |
    | **Best for** | Small, static systems | Batch systems with known max demands | Large dynamic systems |

<div class="fragment">

!!! tip "What real systems use"
    - **Linux/Windows:** **Ostrich algorithm** — ignore deadlock, rely on the user to kill stuck processes.
    - **Databases:** **Timeout-based** — abort long transactions.
    - **Safety-critical:** **Prevention** — statically prove no deadlock possible.
    - **Distributed systems:** **Detection + recovery** — no global knowledge of resources.

</div>

<div class="fragment">

???+ note "The ostrich algorithm"
    "Put your head in the sand and pretend the problem doesn't exist." Modern general-purpose OSes do this — deadlocks are rare, and the cost of prevention (low utilisation) exceeds the cost of occasional user intervention.

</div>

---

## 5.11 — Deadlock in Practice

???+ note "Real-world deadlock scenarios"
    | Domain | Example |
    |---|---|
    | **Databases** | Two transactions lock rows in opposite order |
    | **OS kernels** | Two drivers acquire two mutexes in opposite order |
    | **Distributed systems** | Two nodes wait for network messages from each other |
    | **Web servers** | Two requests wait for each other's database locks |

<div class="fragment">

!!! tip "Prevention in practice"
    - **Lock ordering** — all code acquires locks in a global order.
    - **Try-lock with timeout** — never wait indefinitely.
    - **Deadlock detection services** — some databases monitor and abort cycles automatically.

</div>

---

## Chapter 5 — key takeaways

<div class="fragment">

**1.** A **deadlock** requires all four **Coffman conditions**: mutual exclusion, hold-and-wait, no-preemption, circular wait.

</div>

<div class="fragment">

**2.** The **resource allocation graph** shows deadlock visually — a cycle is **sufficient** for single-instance resources but only **necessary** for multi-instance.

</div>

<div class="fragment">

**3.** **Prevention** breaks one condition — cheapest (no deadlock) but lowest utilisation. **Circular-wait prevention** (lock ordering) is the practical choice.

</div>

<div class="fragment">

**4.** **Avoidance** (Banker's algorithm) requires knowing max demands and checking every request for **safety**.

</div>

<div class="fragment">

**5.** **Detection** uses wait-for graphs (single instance) or a detection algorithm (multi-instance). Cheaper than prevention but allows temporary deadlock.

</div>

<div class="fragment">

**6.** **Recovery** by process termination or resource preemption. Preemption must avoid starvation.

</div>

<div class="fragment">

**7.** Real systems often use the **ostrich algorithm** — deadlock is rare; user intervention is acceptable.

</div>

---

## Next — Chapter 6

**Memory Management.**

<div class="fragment">

- **Contiguous allocation** and fragmentation.
- **Paging** — page tables, TLB, multi-level tables.
- **Segmentation** — logical segments.
- **Virtual memory** — demand paging, page faults.
- **Page replacement** — FIFO, LRU, Optimal, Clock.
- **Thrashing** and working set.

</div>

<div class="fragment">

Chapter 5 showed what happens when processes compete for **finite** resources. Chapter 6 shows the biggest finite resource of all — **memory**.

</div>