# Chapter 2 — Processes

!!! abstract "Chapter at a glance"
    **Coverage:** process concepts · operations on processes · threads.

---

## Learning outcomes

1. Define a process and describe its components (code, data, heap, stack, PCB).
2. Draw and explain the five-state process model.
3. Describe process creation and termination, including the `fork()`/`exec()` model.
4. Compare thread models (many-to-one, one-to-one, many-to-many).

---

## 2.1 Process concepts

!!! note "Definition (Tanenbaum §2.1)"
    A **process** is a program in execution. It consists of the program code, the current activity (program counter, registers), a stack, a data section, and a heap.

### 2.1.1 Process in memory

```
+------------------+  high addresses
|      Stack       |  (local variables, return addresses)
+------------------+
|        ↓         |
|                  |
|        ↑         |
+------------------+
|      Heap        |  (dynamically allocated memory)
+------------------+
|      Data        |  (global and static variables)
+------------------+
|      Text        |  (program code)
+------------------+  low addresses
```

### 2.1.2 Process control block (PCB)

The OS stores all information about a process in its **PCB**:

| Field | Contents |
|---|---|
| Process state | new, ready, running, waiting, terminated |
| Program counter | address of the next instruction |
| CPU registers | saved on context switch |
| Memory-management info | base/limit registers, page tables |
| Accounting info | CPU time used, process ID, parent ID |
| I/O status | open files, allocated devices |

### 2.1.3 Process states

```
         admitted              scheduler dispatch
  new ──────────► ready ◄────────────────────────── running
                    ▲                                  │
                    │            interrupt             │
                    └──────────────────────────────────┘
                    │                                  │
                    │         I/O or event wait        │
                    │                                  ▼
                    └──────────────────────────── waiting
                                                       │
                                                       │ exit
                                                       ▼
                                                  terminated
```

The **five-state model**: new, ready, running, waiting, terminated.

---

## 2.2 Operations on processes

### 2.2.1 Process creation

A parent process creates a child via `fork()`. The child is a **copy** of the parent's address space.

```c
pid_t pid = fork();
if (pid == 0) {
    // child process
    execl("/bin/ls", "ls", NULL);
} else {
    // parent process
    wait(NULL);
}
```

- `fork()` returns 0 in the child, the child's PID in the parent.
- `exec()` replaces the process image with a new program.
- `wait()` blocks the parent until the child terminates.

### 2.2.2 Process termination

A process terminates via `exit(status)`. The parent collects the status via `wait()`. If the parent terminates before the child, the child becomes an **orphan** (adopted by `init`/`systemd`). If the child terminates but the parent has not called `wait()`, the child becomes a **zombie** until reaped.

---

## 2.3 Threads

### 2.3.1 What is a thread?

A **thread** is the smallest unit of execution within a process. Each thread has its own:

- Program counter
- Register set
- Stack

Threads in the same process **share** the code section, data section, and open files.

### 2.3.2 Benefits of threads

- **Responsiveness** — one thread can handle UI while another computes.
- **Resource sharing** — threads share memory by default (no IPC needed).
- **Economy** — thread creation and context switching are cheaper than process creation.
- **Scalability** — threads can run in parallel on multiple cores.

### 2.3.3 Thread models

| Model | Description | Example |
|---|---|---|
| **Many-to-one** | Many user threads mapped to one kernel thread | Green threads (old Java) |
| **One-to-one** | Each user thread mapped to one kernel thread | Linux, Windows |
| **Many-to-many** | Many user threads multiplexed onto a smaller or equal number of kernel threads | Solaris, older Windows |

---

## 2.4 Quiz

1. What is a process? What is stored in its PCB?
2. Draw the five-state process model and label every transition.
3. Explain the `fork()` and `exec()` system calls.
4. What is a zombie process? How is it different from an orphan?
5. Compare the three thread models.

---

## Answer key

??? success "Click to reveal"

    1. A process is a program in execution. PCB contains state, PC, registers, memory info, accounting info, I/O status.
    2. (See diagram in §2.1.3.)
    3. `fork()` creates a child copy of the parent; `exec()` replaces the process image with a new program.
    4. Zombie: child terminated but parent has not called `wait()`. Orphan: parent terminated before child — child adopted by `init`.
    5. (See table in §2.3.3.)

---

## Further reading

- Tanenbaum §2.1–§2.2.
- Silberschatz §3.1–§3.4.