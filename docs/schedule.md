# Schedule

Week-by-week plan for **ECEG-5202**, 18-week semester.

---

## At a glance

| Week | Chapter | Topic | Tutorial | Lab | Assessment |
|:---:|:---:|---|---|---|---|
| 1 | 1 | Introduction — what is an OS, history | — | — | — |
| 2 | 1 | OS structures, computer system overview | T1 | L1 | A1 out |
| 3 | 2 | Processes — concepts, operations | — | — | — |
| 4 | 2 | Threads, thread models | T2 | L1 | A1 due / A2 out |
| 5 | 3 | CPU scheduling — concepts, criteria | — | — | — |
| 6 | 3 | Scheduling algorithms, multiprocessor, real-time | T3 | L2 | A2 due / A3 out |
| 7 | 4 | IPC — race conditions, critical regions | — | — | A3 due |
| 8 | — | **Mid-semester test** | — | — | **15%** |
| 9 | 4 | Synchronisation — semaphores, monitors | T4 | L2 | A4 out |
| 10 | 5 | Deadlocks — prevention, avoidance | — | — | A4 due / A5 out |
| 11 | 5 | Deadlock detection and recovery | T5 | L3 | — |
| 12 | 6 | Memory management — partitioning, paging | — | — | A5 due / A6 out |
| 13 | 6 | Segmentation, virtual memory | T6 | L3 | — |
| 14 | 7 | File systems — structure, implementation | — | — | A6 due / A7 out |
| 15 | 8 | I/O management, buffering, disk cache | T7 | — | — |
| 16 | 9 | Security — threats and defences | T8 | — | A7 due / A8 out |
| 17 | — | Review | — | — | A8 due |
| 18 | — | **Final examination** | — | — | **50%** |

**Legend.** T = tutorial · L = laboratory · A = assignment · — = no activity

---

## Detail by week

### Week 1 — Chapter 1, Part 1

- **Lecture.** What is an OS? History of OS.
- **Reading.** Tanenbaum §1.1–§1.2; Silberschatz §1.1–§1.4.

### Week 2 — Chapter 1, Part 2

- **Lecture.** OS structures (monolithic, microkernel, layered, modular); computer system overview (CPU, memory, I/O, buses, interrupts).
- **Tutorial 1.** OS structure comparison, interrupt handling.
- **Laboratory 1.** Shell basics — commands, pipes, redirection, scripts.
- **Assignment 1** released.

### Week 3 — Chapter 2, Part 1

- **Lecture.** Process concepts — process states, PCB, context switching.
- **Reading.** Tanenbaum §2.1; Silberschatz §3.1–§3.2.

### Week 4 — Chapter 2, Part 2

- **Lecture.** Operations on processes (creation, termination); threads and thread models (many-to-one, one-to-one, many-to-many).
- **Tutorial 2.** Process state diagrams, thread model comparison.
- **Assignment 1 due; Assignment 2** released.

### Week 5 — Chapter 3, Part 1

- **Lecture.** CPU scheduling — basic concepts, scheduling criteria (CPU utilisation, throughput, turnaround, waiting, response).
- **Reading.** Tanenbaum §2.4; Silberschatz §5.1–§5.2.

### Week 6 — Chapter 3, Part 2

- **Lecture.** Scheduling algorithms (FCFS, SJF, SRTN, Round Robin, priority, multilevel feedback queue); multiprocessor scheduling; real-time scheduling (RM, EDF).
- **Tutorial 3.** Trace the algorithms on given workloads, compute average waiting time.
- **Laboratory 2.** Inter-process communication — pipes, shared memory, message queues.
- **Assignment 2 due; Assignment 3** released.

### Week 7 — Chapter 4, Part 1

- **Lecture.** Inter-process communication — race conditions, critical regions, the critical-section problem.
- **Reading.** Tanenbaum §2.3; Silberschatz §6.1–§6.2.
- **Assignment 3 due.**

### Week 8 — Mid-semester test

- **Test.** 2 hours, closed book, **15%** of final grade. Covers Chapters 1–3.

### Week 9 — Chapter 4, Part 2

- **Lecture.** Synchronisation — Peterson's solution, hardware support (test-and-set, compare-and-swap), semaphores, monitors; classical problems (producer–consumer, readers–writers, dining philosophers).
- **Tutorial 4.** Trace Peterson's algorithm, semaphore exercises.
- **Assignment 4** released.

### Week 10 — Chapter 5, Part 1

- **Lecture.** Deadlocks — characterisation (four conditions), prevention, avoidance (Banker's algorithm).
- **Reading.** Tanenbaum §6.1–§6.3; Silberschatz §7.1–§7.4.
- **Assignment 4 due; Assignment 5** released.

### Week 11 — Chapter 5, Part 2

- **Lecture.** Deadlock detection (wait-for graph, detection algorithm) and recovery.
- **Tutorial 5.** Banker's algorithm trace, wait-for graph construction.
- **Laboratory 3.** Threads — creation, synchronisation, race conditions.
- **Assignment 5 due; Assignment 6** released.

### Week 12 — Chapter 6, Part 1

- **Lecture.** Memory management — introduction, memory partitioning (fixed, variable, buddy system), paging.
- **Reading.** Tanenbaum §3.1–§3.3; Silberschatz §8.1–§8.5.

### Week 13 — Chapter 6, Part 2

- **Lecture.** Segmentation, virtual memory, page-replacement algorithms (FIFO, LRU, Optimal, Clock), thrashing.
- **Tutorial 6.** Page-replacement traces.
- **Assignment 6 due; Assignment 7** released.

### Week 14 — Chapter 7

- **Lecture.** File systems — structure, implementation (contiguous, linked, indexed allocation), directory implementation, efficiency and performance.
- **Reading.** Tanenbaum §4.1–§4.3; Silberschatz §10.1–§10.5.

### Week 15 — Chapter 8

- **Lecture.** I/O management — I/O devices, principles of I/O software, buffering, disk cache.
- **Tutorial 7.** Disk scheduling (FCFS, SSTF, SCAN, C-SCAN).
- **Assignment 7 due; Assignment 8** released.

### Week 16 — Chapter 9

- **Lecture.** Security — threats, attacks (malware, buffer overflow, DoS), defences (authentication, access control, sandboxing).
- **Tutorial 8.** Threat modelling exercise.
- **Assignment 8 due.**

### Week 17 — Review

- **Lecture.** Review of all nine chapters.
- **Tutorial.** Q&A; past-paper discussion.

### Week 18 — Final examination

- **Exam.** 3 hours, closed book, **50%** of final grade. Covers the whole syllabus.