# Exam Bank — ECEG-5202 Operating Systems

**40 questions spanning all nine chapters.** Use for self-assessment and exam preparation.

**Instructions (mock exam):**

- Time: **3 hours**
- Closed book
- Total marks: **100**
- Marks per question in brackets

---

## Section A — Introduction (Ch 1) — 10 marks

### A1. Multiple choice [2]

An operating system is best described as:

(a) a compiler
(b) an intermediary between the user and the hardware
(c) a database management system
(d) a programming language

### A2. Multiple choice [2]

Which OS structure places all functionality in a single kernel address space?

(a) microkernel
(b) monolithic kernel
(c) layered
(d) client-server

### A3. Short answer [3]

List three functions of an operating system. For each, give one concrete example of where the OS performs it.

### A4. Short answer [3]

Compare **monolithic** and **microkernel** OS structures on performance, reliability, and code complexity. Give one real OS that uses each.

---

## Section B — Processes (Ch 2) — 10 marks

### B1. Multiple choice [2]

A process control block (PCB) contains:

(a) only the process's program counter
(b) process state, program counter, registers, memory limits, and open-file table
(c) only the process's memory allocation
(d) only the process's priority

### B2. Multiple choice [2]

Which thread model maps each user thread to exactly one kernel thread?

(a) many-to-one
(b) one-to-one
(c) many-to-many
(d) none of the above

### B3. Short answer [3]

Draw the process state diagram for the five-state model (new, ready, running, waiting, terminated). Label every transition.

### B4. Short answer [3]

Explain the difference between a **process** and a **thread**. Give one application where threads are clearly preferable to processes.

---

## Section C — CPU Scheduling (Ch 3) — 12 marks

### C1. Multiple choice [2]

Which scheduling algorithm gives the minimum average waiting time for a given set of processes?

(a) FCFS
(b) SJF
(c) Round Robin
(d) priority

### C2. Multiple choice [2]

Round Robin scheduling with a very large time quantum degenerates to:

(a) SJF
(b) FCFS
(c) priority scheduling
(d) shortest remaining time first

### C3. Short answer [4]

Five processes arrive at time 0 with burst times 10, 2, 4, 6, 8. Compute the **average waiting time** under FCFS, SJF, and Round Robin (quantum = 2). Show your work.

### C4. Short answer [4]

Explain the **priority inversion** problem in real-time systems. Describe how priority inheritance solves it, and give one concrete scenario where it matters.

---

## Section D — IPC and Synchronisation (Ch 4) — 12 marks

### D1. Multiple choice [2]

A **race condition** occurs when:

(a) two processes share no memory
(b) the outcome depends on the relative order of execution of concurrent processes
(c) a process is starved
(d) a process holds a resource forever

### D2. Multiple choice [2]

Peterson's solution to the critical-section problem uses:

(a) a single shared flag
(b) two shared variables (`flag` and `turn`)
(c) hardware test-and-set
(d) semaphores

### D3. Short answer [4]

State the **three requirements** a correct critical-section solution must satisfy. Give the code for a **semaphore-based** solution to the producer–consumer problem with a bounded buffer of size $N$.

### D4. Short answer [4]

Explain the **dining philosophers** problem. Give one solution that avoids deadlock, and explain why it is correct.

---

## Section E — Deadlocks (Ch 5) — 12 marks

### E1. Multiple choice [2]

Which of the following is **not** a necessary condition for deadlock?

(a) mutual exclusion
(b) hold and wait
(c) preemption
(d) circular wait

### E2. Multiple choice [2]

The Banker's algorithm is used for:

(a) deadlock prevention
(b) deadlock avoidance
(c) deadlock detection
(d) deadlock recovery

### E3. Short answer [4]

State the **four necessary conditions** for deadlock. Explain how **breaking each one** corresponds to a deadlock-prevention strategy.

### E4. Short answer [4]

Given three resource types with total instances $(10, 5, 7)$ and the allocation/need tables below, use the **Banker's algorithm** to determine whether the system is in a safe state.

| Process | Allocation (A, B, C) | Max (A, B, C) |
|---|---|---|
| P0 | (0, 1, 0) | (7, 5, 3) |
| P1 | (2, 0, 0) | (3, 2, 2) |
| P2 | (3, 0, 2) | (9, 0, 2) |
| P3 | (2, 1, 1) | (2, 2, 2) |
| P4 | (0, 0, 2) | (4, 3, 3) |

---

## Section F — Memory Management (Ch 6) — 12 marks

### F1. Multiple choice [2]

**Paging** eliminates:

(a) internal fragmentation
(b) external fragmentation
(c) both internal and external fragmentation
(d) neither

### F2. Multiple choice [2]

In a paged memory system, the **TLB** is used to:

(a) store the page table on disk
(b) cache recent virtual-to-physical translations
(c) allocate physical frames
(d) swap pages

### F3. Short answer [4]

Explain the difference between **paging** and **segmentation**. Give one advantage of each.

### F4. Short answer [4]

For the reference string `7, 0, 1, 2, 0, 3, 0, 4, 2, 3, 0, 3, 2` with 3 frames, compute the number of page faults under **FIFO**, **LRU**, and **Optimal** replacement. Show the frame contents after each reference.

---

## Section G — File Systems (Ch 7) — 10 marks

### G1. Multiple choice [2]

Which file-allocation method suffers from external fragmentation?

(a) contiguous
(b) linked
(c) indexed
(d) inode-based

### G2. Short answer [4]

Describe the **inode** structure in a Unix file system. How does it support both small and very large files efficiently?

### G3. Short answer [4]

Compare **linked** and **indexed** file allocation on:

- sequential access speed,
- random access speed,
- space overhead,
- fragmentation.

---

## Section H — I/O Management (Ch 8) — 10 marks

### H1. Multiple choice [2]

Which disk-scheduling algorithm can cause **starvation**?

(a) FCFS
(b) SSTF
(c) SCAN
(d) C-SCAN

### H2. Short answer [4]

Explain **buffering** and **caching** in I/O systems. Give one situation where each is essential, and explain how they differ.

### H3. Short answer [4]

For the request queue `98, 183, 37, 122, 14, 124, 65, 67` with the disk head initially at 53, compute the total head movement under **FCFS**, **SSTF**, and **SCAN** (moving toward 0 first).

---

## Section I — Security (Ch 9) — 10 marks

### I1. Multiple choice [2]

A **buffer overflow** attack exploits:

(a) weak passwords
(b) missing bounds checking in programs
(c) unencrypted network traffic
(d) misconfigured firewalls

### I2. Short answer [3]

Define **authentication** and **authorisation**. Give one mechanism for each, and explain how they differ.

### I3. Short answer [5]

Describe three classes of **malware** (virus, worm, trojan) and one defence against each. Why is a **worm** generally more dangerous than a **virus**?

---

## Grading rubric

| Band | Marks |
|---|---|
| A | 90–100 |
| B | 75–89 |
| C | 60–74 |
| F | < 60 |

**Common error checks:**

- Wrong scheduling formula (using turnaround instead of waiting time): −2.
- Confusing deadlock prevention with avoidance: −2.
- Claiming paging eliminates internal fragmentation: −2.
- Missing a necessary condition for deadlock: −2.
- Confusing buffering with caching: −1.