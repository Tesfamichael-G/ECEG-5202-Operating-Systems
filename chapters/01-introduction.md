# Chapter 1 — Introduction

!!! abstract "Chapter at a glance"
    **Contact hours:** 3 lecture + 1 tutorial · **Home study:** ~5 h

    **Coverage:** what is an operating system · history of OS · OS structures · computer system overview.

---

## Learning outcomes

1. Define an operating system and explain its role as an intermediary between user and hardware.
2. Trace the history of operating systems from batch processing to modern multiprocessor and mobile systems.
3. Compare the major OS structures: monolithic, layered, microkernel, modular, and hybrid.
4. Describe the computer system organisation: CPU, memory, I/O devices, buses, and interrupts.

---

## 1.1 What is an operating system?

!!! note "Definition (Tanenbaum §1.1)"
    An **operating system** is a program that acts as an intermediary between the user of a computer and the computer hardware. Its primary goals are **convenience** for the user and **efficient** use of the hardware.

An OS provides:

- **Resource allocation** — manages CPU, memory, and I/O devices among competing processes.
- **Abstraction** — presents a clean interface (files, processes, sockets) on top of raw hardware.
- **Protection** — prevents processes from interfering with each other.
- **Convenience** — provides a user interface (shell, GUI) and system-call API.

### 1.1.1 The OS as a resource manager

The OS manages four categories of resources:

| Resource | Managed by |
|---|---|
| **CPU** | Scheduler (Chapter 3) |
| **Memory** | Memory manager (Chapter 6) |
| **I/O devices** | I/O subsystem (Chapter 8) |
| **Files / storage** | File system (Chapter 7) |

### 1.1.2 The OS as an abstraction layer

Applications never touch hardware directly — they issue **system calls** to the kernel:

```
Application  →  System-call interface  →  Kernel  →  Hardware
```

The kernel translates these calls into hardware operations and returns results.

---

## 1.2 History of operating systems

| Generation | Era | Technology | OS characteristics |
|---|---|---|---|
| First | 1945–1955 | Vacuum tubes, plugboards | No OS — programmers controlled hardware directly |
| Second | 1955–1965 | Transistors, batch systems | Batch monitors, job sequencing |
| Third | 1965–1980 | ICs, multiprogramming | Multiprogramming, spooling, time-sharing (MULTICS, UNIX) |
| Fourth | 1980–present | VLSI, PCs | GUI (Windows, macOS), Linux, mobile (Android, iOS) |

**Key milestones.**

- **1956:** GM-NAA I/O — the first batch operating system.
- **1964:** OS/360 — IBM's first family of OSes.
- **1969:** UNIX developed at Bell Labs.
- **1981:** MS-DOS for the IBM PC.
- **1985:** Windows 1.0.
- **1991:** Linux kernel released by Linus Torvalds.
- **2008:** Android released by Google.

---

## 1.3 Operating system structures

### 1.3.1 Monolithic kernel

All OS services run in a single kernel address space.

- **Pros:** Fast — direct function calls, no message passing.
- **Cons:** Large, hard to maintain, one bug crashes the whole kernel.
- **Examples:** Linux, traditional UNIX.

### 1.3.2 Layered structure

The OS is divided into hierarchical layers, each using only the layer below.

- **Pros:** Clean design, each layer independently testable.
- **Cons:** Overhead of crossing layers; hard to define layer boundaries correctly.
- **Example:** THE operating system (Dijkstra, 1968).

### 1.3.3 Microkernel

Only essential services (IPC, scheduling, basic memory management) run in kernel space; everything else (file systems, device drivers) runs as user-space servers.

- **Pros:** Reliability (a crashed server can be restarted), security, portability.
- **Cons:** Performance (message passing overhead).
- **Examples:** QNX, L4, seL4, MINIX 3.

### 1.3.4 Modular kernel

The kernel has a core plus dynamically loadable modules (Linux's LKM model).

- **Pros:** Combines monolithic performance with modular flexibility.
- **Cons:** Modules still run in kernel space — a buggy module can crash the kernel.
- **Examples:** Linux, Solaris.

### 1.3.5 Hybrid

Combines multiple structures — e.g., Windows NT's hybrid kernel runs some services in kernel mode and others as user-mode processes.

---

## 1.4 Computer system overview

### 1.4.1 Organisation

A computer system consists of:

- **CPU(s)** — execute instructions.
- **Main memory** — volatile storage for code and data.
- **I/O devices** — disks, network cards, keyboards, displays.
- **Buses** — communication pathways between components.

### 1.4.2 Interrupts

An **interrupt** is a signal that causes the CPU to suspend its current execution and transfer control to an interrupt handler.

| Type | Cause |
|---|---|
| **Hardware interrupt** | I/O device completes, timer expires |
| **Trap / exception** | Division by zero, page fault, system call |
| **Software interrupt** | Explicit `int` instruction |

The **interrupt vector table** maps interrupt numbers to handler addresses.

### 1.4.3 Dual-mode operation

To protect the OS from user programs, the CPU supports at least two modes:

- **Kernel mode** — full hardware access (privileged instructions).
- **User mode** — restricted; privileged instructions trap to the kernel.

The **mode bit** in the CPU status register indicates the current mode. System calls transition from user to kernel mode via a controlled trap.

---

## 1.5 Quiz

1. What is an operating system? Give three of its responsibilities.
2. List three generations of OS history and one characteristic of each.
3. Compare monolithic and microkernel structures on performance and reliability.
4. What is the purpose of dual-mode operation? Give one privileged instruction.
5. Explain the difference between a hardware interrupt and a trap.

---

## Answer key

??? success "Click to reveal"

    1. An OS is an intermediary between user and hardware. Responsibilities: resource allocation, abstraction, protection.
    2. Second generation: batch systems; third: multiprogramming and time-sharing; fourth: GUI and mobile.
    3. Monolithic is faster (direct calls) but less reliable (one bug crashes the kernel); microkernel is more reliable but slower (message passing).
    4. Dual-mode operation protects the OS from user programs. Privileged instructions (e.g., `cli`, `sti`, direct I/O) are only executable in kernel mode.
    5. A hardware interrupt comes from a device (I/O completion, timer); a trap is triggered by the currently executing instruction (system call, exception).

---

## Further reading

- Tanenbaum & Bos, §1.1–§1.4.
- Silberschatz et al., §1.1–§2.3.
- OSTEP, Chapter 2.