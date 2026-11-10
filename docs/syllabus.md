# Syllabus

Full course outline for **ECEG-5202 — Operating Systems**.

---

## Course identification

| Field | Value |
|---|---|
| **Course code** | ECEG-5202 |
| **Course title** | Operating Systems |
| **Credit hours (ECTS)** | 5 — 3 lecture + 1 tutorial + 1 lab per week |
| **Degree program** | BSc Electrical Engineering (Computer) |
| **Module** | Programming Concepts and Software Engineering |
| **Module coordinator** | N.N. |
| **Semester** | VIII/8 |
| **Status** | Compulsory |
| **Pre-requisites** | Computer Architecture & Organization |
| **Instructor** | *(insert)* |

---

## Student workload

| Lecture | Tutorial | Practice / Laboratory | Home study |
|:---:|:---:|:---:|:---:|
| 3 h | 1 h | 1 h | ~5 h |

Total: ~10 hours per week × 18 weeks = **~180 hours** — consistent with 5 ECTS.

---

## Course description

This course examines fundamental issues in operating system design and implementation. The course covers the tradeoffs that can be made between performance and functionality during the design and implementation of an operating system. Particular emphasis is given to the major OS subsystems: **process management** (processes, threads, CPU scheduling, synchronization, and deadlock), **memory management** (segmentation and paging), **file systems**, and **I/O management**.

---

## Course objectives and competences

By the end of the course, students will be able to:

1. **Learn** basic principles and components of an operating system.
2. **Learn and be familiar** with the notion of processes and inter-process communication.
3. **Understand** CPU scheduling, deadlock detection and avoidance.
4. **Understand** how memory and I/O are managed, and the implementation of file systems.

---

## Learning outcomes

| # | Outcome | Assessed by |
|---|---|---|
| LO1 | Explain what an OS is, its history, and its structural choices | A1, Test 1 |
| LO2 | Describe process concepts, operations, and thread models | A2, Test 1 |
| LO3 | Compare CPU scheduling algorithms and evaluate them on given workloads | A3, Test 1 |
| LO4 | Analyse race conditions and apply synchronisation primitives | A4, Test 2 |
| LO5 | Detect, prevent, avoid, and recover from deadlocks | A5, Test 2 |
| LO6 | Analyse memory-management schemes (partitioning, paging, segmentation) | A6, Test 2 |
| LO7 | Describe file-system structure, implementation, and performance trade-offs | A7, Final |
| LO8 | Describe I/O software layers, buffering, and disk caching | A8, Final |
| LO9 | Identify security threats and defences | A9, Final |

---

## Course content

**Chapter 1 — Introduction**

- What is an operating system?
- History of OS
- Operating System Structures
- Computer System Overview

**Chapter 2 — Processes**

- Process concepts
- Operations on processes
- Threads

**Chapter 3 — CPU Scheduling**

- Basic concepts
- Scheduling criteria
- Scheduling algorithms
- Multiple-processor scheduling
- Real-time scheduling

**Chapter 4 — Inter-Process Communication**

- Introduction
- Race condition
- Critical region
- Process synchronisation

**Chapter 5 — Deadlocks**

- Introduction
- Deadlock prevention
- Deadlock avoidance
- Deadlock detection and recovery

**Chapter 6 — Memory Management**

- Introduction
- Memory partitioning
- Paging
- Segmentation

**Chapter 7 — File Systems**

- Introduction
- File system structure
- File system implementation
- Directory implementation
- Efficiency and performance

**Chapter 8 — I/O Management**

- I/O devices
- Principles of I/O software
- I/O buffering
- Disk cache

**Chapter 9 — Introduction to Security**

- Threats and attacks
- Defences

---

## Laboratory exercises

| Lab | Topic |
|---|---|
| L1 | Basics of the shell — commands, pipes, redirection, scripting |
| L2 | Inter-process communication — pipes, shared memory, message queues |
| L3 | Threads — creation, synchronisation, race conditions |

---

## Assessment

| Component | Weight |
|---|:---:|
| Assignments (20%) | 20% |
| Tests (30%) | 30% |
| Final examination (50%) | 50% |
| **Total** | **100%** |

---

## Textbook and references

**Textbook.**

- **Tanenbaum, A. S. & Bos, H.**, *Modern Operating Systems*, 4th ed., Pearson, 2015.

**References.**

- **Silberschatz, A., Galvin, P. B. & Gagne, G.**, *Operating System Concepts*, 9th ed., Wiley, 2013.
- **Stallings, W.**, *Operating Systems: Internals and Design Principles*, 7th ed., Prentice Hall, 2012.

---

## Policies

- **Attendance:** minimum **75%** lecture attendance.
- **Late work:** 10% per day, zero after 3 days.
- **Academic integrity:** all submitted work must be your own.