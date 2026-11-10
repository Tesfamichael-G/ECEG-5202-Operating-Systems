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
  js: assets/lecture-02.js

---

# ECEG-5202

## Chapter 2 — Processes and Threads

---

## Learning outcomes

<div class="fragment">

1. Define a **process** and describe its **memory layout** and **process control block**.

</div>

<div class="fragment">

2. Draw and explain the **five-state process model** with every transition.

</div>

<div class="fragment">

3. Trace the **fork–exec–wait** sequence for process creation.

</div>

<div class="fragment">

4. Distinguish **zombie** and **orphan** processes and explain how to reap them.

</div>

<div class="fragment">

5. Compare **thread models**: many-to-one, one-to-one, many-to-many.

</div>

<div class="fragment">

6. Measure and analyse **context switch overhead**.

</div>

---

## 2.1 — What is a Process?

???+ note "Definition (Tanenbaum §2.1)"
    A **process** is a program in execution. It consists of the **program code**, the **current activity** (program counter, registers), a **stack**, a **data section**, and a **heap**.

<div class="fragment">

???+ info "Program ≠ process"
    - A **program** is a passive entity: a binary file on disk.
    - A **process** is an active entity: a running instance of that program with its own state.
    
    One program can have many processes running (e.g., multiple Chrome tabs = multiple processes).

</div>

<div class="fragment">

!!! tip "The two Chrome models"
    - **Chrome**: one process per tab. A crashed tab doesn't take down the browser, but memory usage is high.
    - **Firefox (older versions)**: many tabs in one process. Memory-efficient but one crash loses all tabs.
    
    The choice is a classic reliability-vs-resource trade-off.

</div>

---

## 2.2 — Process Memory Layout

```
High address
┌─────────────────┐
│     Stack       │  ← local variables, return addresses
│        ↓        │     (grows downward)
│                 │
│                 │
│        ↑        │
│      Heap       │  ← malloc / new (grows upward)
├─────────────────┤
│   Data (BSS)    │  ← uninitialised globals
├─────────────────┤
│   Data (init)   │  ← initialised globals
├─────────────────┤
│     Text        │  ← program code
└─────────────────┘
Low address
```

<div class="fragment">

???+ note "The five segments"
    | Segment | Contents | Size |
    |---|---|---|
    | **Text** | Machine code | Fixed |
    | **Data (initialised)** | Global/static with values | Fixed |
    | **BSS** | Global/static without values | Fixed |
    | **Heap** | `malloc`/`new` allocations | Dynamic |
    | **Stack** | Local variables, function frames | Dynamic |

</div>

<div class="fragment">

!!! warning "Stack and heap grow toward each other"
    The stack grows **down** (toward lower addresses); the heap grows **up**. They share the same region. When they meet, you have **stack overflow** or **heap overflow**. This is why deep recursion crashes with "stack overflow".

</div>

---

## 2.3 — The Process Control Block

???+ note "Definition (Silberschatz §3.1)"
    The **PCB** (or **task control block**) is the data structure the OS uses to track a process. One PCB per process.

<div class="fragment">

???+ note "PCB contents"
    | Field | Contents |
    |---|---|
    | **Process state** | new, ready, running, waiting, terminated |
    | **Program counter** | Address of the next instruction |
    | **CPU registers** | Saved on context switch (rax, rbx, ..., rip, rflags) |
    | **CPU scheduling info** | Priority, time quantum remaining |
    | **Memory-management info** | Base/limit registers, page-table base |
    | **Accounting info** | CPU time used, process ID, parent ID |
    | **I/O status** | Open files, allocated devices |

</div>

<div class="fragment">

!!! tip "Where does the PCB live?"
    In the **kernel** (kernel memory, not user memory). It is created at process creation and destroyed at process exit. The kernel's process table is the collection of all PCBs.

</div>

---

## 2.4 — The Five-State Model

```
                admitted
      new ──────────────► ready ◄───────────────┐
                            │                    │
                            │ scheduler          │ interrupt
                            │ dispatch           │ (preemption)
                            ▼                    │
                          running ───────────────┘
                            │
                            │ I/O or event wait
                            ▼
                         waiting
                            │
                            │ I/O or event completes
                            ▼
                         (back to ready)
                          
                          running ─── exit ───► terminated
```

<div class="fragment">

???+ note "The five states"
    | State | Meaning |
    |---|---|
    | **New** | Being created |
    | **Ready** | Waiting for the CPU |
    | **Running** | Executing on the CPU |
    | **Waiting** | Waiting for an I/O event |
    | **Terminated** | Finished; resources being reclaimed |

</div>

---

## 2.5 — Every Transition, Explained

???+ note "Six transitions"
    | Transition | Trigger |
    |---|---|
    | **new → ready** | OS admits the process (enough resources) |
    | **ready → running** | Scheduler selects the process |
    | **running → ready** | Interrupt (time slice expires) or preemption |
    | **running → waiting** | Process requests I/O |
    | **waiting → ready** | I/O completes |
    | **running → terminated** | Process exits (`exit()` or crash) |

<div class="fragment">

!!! warning "The forbidden transition"
    **waiting → running** is not allowed. A process that finishes I/O must go through **ready** and wait for the scheduler. This preserves fairness — the scheduler decides who runs next.

</div>

<div class="fragment">

???+ info "Why five states, not three?"
    A naive model has only ready, running, waiting. The five-state model adds **new** and **terminated** to represent resource allocation and cleanup phases. This matters in real systems — process creation can be expensive (fork copies memory), and process termination is not instantaneous.

</div>

---

## Worked example 2.1 — Trace the state of one process

!!! tip "Problem:"
    A process begins. It runs for 20 ms, then reads from disk (5 ms), then runs for 10 ms, then exits. The scheduler gives it a 15 ms time quantum.

<div class="fragment">

???+ note "**Step 1 — creation.**"
    Process enters **new**. The kernel allocates the PCB, memory, and initial state. Transitions to **ready**.

</div>

<div class="fragment">

??? info "**Step 2 — first scheduling.**"
    Scheduler picks the process. **ready → running** at t=0.

</div>

<div class="fragment">

??? info "**Step 3 — time slice expires.**"
    At t=15 ms, the timer interrupt fires. **running → ready**. Process is preempted.

</div>

<div class="fragment">

??? info "**Step 4 — rescheduled.**"
    Scheduler picks it again (no other process ready). **ready → running** at t=15.

</div>

<div class="fragment">

??? info "**Step 5 — I/O.**"
    At t=20 ms (5 ms into second slice), process issues a read. **running → waiting**. Disk seek + read takes 5 ms.

</div>

<div class="fragment">

??? info "**Step 6 — I/O completes.**"
    At t=25 ms, disk interrupt. **waiting → ready**. Process is now ready to run.

</div>

<div class="fragment">

??? info "**Step 7 — rescheduled.**"
    Scheduler picks it. **ready → running** at t=25. Runs for 10 ms.

</div>

<div class="fragment">

??? info "**Step 8 — exit.**"
    At t=35 ms, process calls `exit()`. **running → terminated**.

</div>

<div class="fragment">

??? success "**Total time.** 35 ms wall clock; 30 ms CPU time; 5 ms blocked on I/O. **CPU utilisation = 30/35 ≈ 86%.**"

</div>

---

## 2.6 — Process Creation: `fork()`

???+ note "Definition"
    `fork()` creates a **child process** that is an exact copy of the parent — same code, same data, same open files — **except** the return value.

```c
pid_t pid = fork();

if (pid == 0) {
    // child process (fork returned 0)
    printf("I am the child (pid=%d)\n", getpid());
} else if (pid > 0) {
    // parent process (fork returned child's pid)
    printf("I am the parent of %d\n", pid);
} else {
    // fork failed
    perror("fork");
}
```

<div class="fragment">

???+ note "What gets copied?"
    - **Text** (code) — usually shared (read-only).
    - **Data** — copied (copy-on-write optimisation).
    - **Heap** — copied.
    - **Stack** — copied.
    - **File descriptors** — copied (both share the same file offsets).
    - **PCB** — copied and given a new PID.

</div>

<div class="fragment">

!!! tip "Copy-on-write"
    Modern kernels do not physically copy memory at `fork()`. The parent and child share the same physical pages, marked read-only. When either writes, that page is copied (COW). This makes `fork()` very fast for the "copy-then-exec" pattern.

</div>

---

## Worked example 2.2 — Trace `fork()` with two processes

!!! tip "Problem:"
    ```c
    int main() {
        printf("start\n");
        pid_t pid = fork();
        printf("after fork\n");
        return 0;
    }
    ```
    How many times is each line printed?

<div class="fragment">

???+ note "**Step 1 — before fork.**"
    One process running. Prints "start". Call `fork()`.

</div>

<div class="fragment">

??? info "**Step 2 — fork returns.**"
    Two processes now exist:
    - Parent: `pid > 0` (the child's PID).
    - Child: `pid == 0`.

</div>

<div class="fragment">

??? info "**Step 3 — both print.**"
    Both execute `printf("after fork\n")`. Both exit.

</div>

<div class="fragment">

??? success "**Result.**"
    **"start" printed once** (before fork).
    **"after fork" printed twice** (once per process).
    
    Total output:
    ```
    start
    after fork
    after fork
    ```

</div>

<div class="fragment">

!!! warning "The order is non-deterministic"
    The parent and child run **concurrently**. The order of the two "after fork" lines depends on the scheduler. Do not rely on it.

</div>

---

## Worked example 2.3 — Trace `fork()` twice

!!! tip "Problem:"
    ```c
    fork();
    fork();
    fork();
    printf("hello\n");
    ```
    How many "hello" lines are printed?

<div class="fragment">

???+ note "**Step 1 — analyse.**"
    Each `fork()` doubles the number of processes.
    
    - Start: 1 process.
    - After 1st fork: 2 processes.
    - After 2nd fork: 4 processes.
    - After 3rd fork: 8 processes.

</div>

<div class="fragment">

??? success "**Result.**"
    **8 lines** of "hello".

</div>

<div class="fragment">

???+ info "**Generalisation.**"
    $n$ consecutive `fork()` calls produce $2^n$ processes. This is the **fork bomb** pattern — and a simple way to exhaust process-table entries.

</div>

---

## 2.7 — `exec()` and `wait()`

???+ note "`exec()` — replace the process image"
    `exec()` (actually a family: `execl`, `execv`, `execvp`, `execle`) **replaces** the calling process's image with a new program. The PID stays the same; the code, data, and stack are replaced.

<div class="fragment">

???+ note "`wait()` — collect the child's exit status"
    `wait(&status)` blocks the parent until a child terminates. It returns the child's PID and stores its exit status in `status`.

</div>

<div class="fragment">

???+ note "The typical pattern"
    ```c
    pid_t pid = fork();
    if (pid == 0) {
        execl("/bin/ls", "ls", "-l", NULL);
        perror("exec");  // only reached if exec fails
        exit(1);
    }
    wait(NULL);  // wait for the child
    ```

</div>

<div class="fragment">

!!! tip "Why fork + exec, not just spawn?"
    The fork-then-exec pattern lets the child **set up the environment** (file descriptors, working directory, signals, resource limits) between `fork()` and `exec()`. This is why Unix shells can implement redirection (`ls > out.txt`) — the child opens `out.txt` and dups it onto fd 1 before `exec`.

</div>

---

## Worked example 2.4 — A shell command

!!! tip "Problem:"
    A shell receives `ls -l > out.txt`. Trace the system calls.

<div class="fragment">

???+ note "**Step 1 — fork.**"
    Shell calls `fork()`. Child is a copy of the shell.

</div>

<div class="fragment">

??? info "**Step 2 — setup (in child).**"
    Child opens `out.txt` for writing — gets fd 3. Duplicates fd 3 onto fd 1: `dup2(3, 1)`. Now stdout goes to the file. Closes fd 3.

</div>

<div class="fragment">

??? info "**Step 3 — exec.**"
    Child calls `execl("/bin/ls", "ls", "-l", NULL)`. The shell code is replaced by `ls`.

</div>

<div class="fragment">

??? info "**Step 4 — parent waits.**"
    Parent (shell) calls `wait()`. Blocks until `ls` exits.

</div>

<div class="fragment">

??? info "**Step 5 — resume.**"
    `wait()` returns. Shell prints the prompt and waits for the next command.

</div>

<div class="fragment">

??? success "**Observation.**"
    The shell itself never runs `ls` — it creates a child that becomes `ls`. This isolation is why a crashed command does not crash the shell.

</div>

---

## 2.8 — Zombies and Orphans

???+ note "Zombie process"
    A process that has **terminated** but whose **exit status has not been read** by the parent. The PCB remains allocated; the process appears in `ps` as `<defunct>`.

<div class="fragment">

???+ note "Orphan process"
    A process whose **parent has terminated** before it. The orphan is re-parented to `init` (PID 1) or `systemd`, which eventually reaps it.

</div>

<div class="fragment">

???+ info "How zombies are created"
    ```c
    pid_t pid = fork();
    if (pid == 0) exit(0);       // child exits immediately
    sleep(3600);                  // parent sleeps; does NOT call wait()
    ```
    
    The child becomes a zombie — its PCB cannot be freed because the parent might call `wait()` later.

</div>

<div class="fragment">

???+ success "How zombies are reaped"
    - **Explicit:** parent calls `wait()` or `waitpid()`.
    - **Implicit:** parent exits → init/systemd reaps the zombie.

</div>

<div class="fragment">

!!! warning "The zombie apocalypse"
    A buggy daemon that forks many children and never waits will accumulate zombies. Eventually the process table fills, and `fork()` fails. This is a real operational risk — monitoring zombie counts is standard practice.

</div>

---

## 2.9 — Context Switching

???+ note "Definition"
    A **context switch** saves the state of the running process and loads the state of the next process, so that the scheduler can move from one to another.

<div class="fragment">

???+ note "What is saved and restored?"
    | Component | Cost |
    |---|---|
    | **CPU registers** | ~100 ns (save/restore) |
    | **Program counter** | Included in registers |
    | **Memory map** | TLB flush or tagged TLB |
    | **Cache state** | Cold cache after switch |

</div>

<div class="fragment">

???+ info "The cost"
    - **Direct cost:** register save/restore — ~100–500 ns.
    - **Indirect cost:** cache and TLB effects — often **10× the direct cost**.

</div>

<div class="fragment">

!!! tip "Why context switch is expensive"
    When you switch processes, the new process's working set is not in cache. The first thousands of memory accesses miss. This is why **thread context switches** (same address space, no TLB flush) are much cheaper than **process context switches**.

</div>

---

## Worked example 2.5 — Context switch overhead

!!! tip "Problem:"
    A context switch costs 5 µs. A system does 1000 switches per second. What fraction of CPU time is spent switching?

<div class="fragment">

???+ note "**Step 1 — total switch time per second.**"
    $1000 \times 5\ \mu\text{s} = 5000\ \mu\text{s} = 5\ \text{ms}$.

</div>

<div class="fragment">

??? info "**Step 2 — fraction of one second.**"
    $5\ \text{ms} / 1000\ \text{ms} = 0.5\%$.

</div>

<div class="fragment">

??? success "**Result.**"
    **0.5%** of CPU time. This seems small — but if the time quantum is 10 ms and a switch is 5 µs, the effective utilisation is $10/(10 + 0.005) \approx 99.95\%$.

</div>

<div class="fragment">

???+ warning "**The catch.**"
    The **indirect** cost (cache and TLB) is often **5–10×** the direct cost. Real overhead is closer to **5%** in context-switch-heavy workloads.

</div>

---

## 2.10 — Threads

???+ note "Definition"
    A **thread** is the smallest unit of execution within a process. Each thread has its own **program counter**, **register set**, and **stack**, but shares **code**, **data**, and **open files** with other threads in the same process.

<div class="fragment">

???+ note "What threads share vs. have"
    | Shared | Per-thread |
    |---|---|
    | Code segment | Program counter |
    | Data segment | Registers |
    | Heap | Stack |
    | Open files | Thread-local storage (optional) |
    | Signal handlers | Signal mask |

</div>

<div class="fragment">

???+ info "Creating threads"
    - **POSIX pthreads:** `pthread_create(&tid, NULL, func, arg)`.
    - **Java:** `new Thread(runnable).start()`.
    - **Python:** `threading.Thread(target=func).start()` (but GIL limits true parallelism).

</div>

---

## 2.11 — Benefits of Threads

???+ info "Four benefits (Silberschatz §4.2)"
    | Benefit | Description |
    |---|---|
    | **Responsiveness** | UI thread stays responsive while worker threads compute |
    | **Resource sharing** | Threads share memory — no IPC needed |
    | **Economy** | Thread creation/switch is cheaper than process |
    | **Scalability** | Threads run in parallel on multicore CPUs |

<div class="fragment">

???+ info "The economy comparison"
    | Operation | Process | Thread |
    |---|---|---|
    | Create | ~10–30 µs | ~1–2 µs |
    | Context switch | ~5–10 µs | ~0.5–1 µs |
    | Memory overhead | Full address space | Stack + minimal |

</div>

<div class="fragment">

!!! tip "The responsiveness argument"
    A web server can spawn one thread per client. While one thread blocks on a slow client, others serve requests. Without threads, the server would handle clients one at a time (or need `select()`-based multiplexing).

</div>

---

## 2.12 — Thread Models

???+ note "User threads vs kernel threads"
    - **User threads** — managed by a library (e.g., green threads). The kernel does not know they exist.
    - **Kernel threads** — managed by the kernel. Scheduled independently.

<div class="fragment">

???+ note "The three mappings"
    | Model | Description | Example |
    |---|---|---|
    | **Many-to-one** | Many user threads → one kernel thread | Old Java (green threads), some embedded |
    | **One-to-one** | Each user thread → one kernel thread | Linux, Windows, macOS |
    | **Many-to-many** | Many user threads → smaller number of kernel threads | Solaris (older), Go goroutines (partially) |

</div>

<div class="fragment">

???+ info "Trade-offs"
    | Model | Pros | Cons |
    |---|---|---|
    | Many-to-one | Fast; no kernel context switches | One blocking thread blocks all; no parallelism |
    | One-to-one | True parallelism; blocking is per-thread | More overhead; kernel thread limit |
    | Many-to-many | Best of both | Complex to implement |

</div>

<div class="fragment">

!!! tip "What modern OSes actually use"
    Linux and Windows use **one-to-one** — each user thread is a kernel thread. Modern hardware has eliminated most of the overhead that many-to-one was designed to avoid. Many-to-many exists mostly in language runtimes (Go's goroutines, Erlang's processes).

</div>

---

## Worked example 2.6 — Threads vs processes

!!! tip "Problem:"
    A program computes 100 tasks, each CPU-bound for 10 ms, on a 4-core CPU. Compare using 4 processes vs 4 threads.

<div class="fragment">

???+ note "**Step 1 — 4 processes.**"
    - Creation cost: $4 \times 25\ \mu\text{s} = 100\ \mu\text{s}$.
    - Context switch cost: $4 \times 10\ \mu\text{s} = 40\ \mu\text{s}$ per round.
    - Memory: 4 separate address spaces (4× memory).

</div>

<div class="fragment">

??? info "**Step 2 — 4 threads.**"
    - Creation cost: $4 \times 1.5\ \mu\text{s} = 6\ \mu\text{s}$.
    - Context switch cost: $4 \times 1\ \mu\text{s} = 4\ \mu\text{s}$ per round.
    - Memory: one address space, 4 stacks.

</div>

<div class="fragment">

??? info "**Step 3 — compute time.**"
    Each core processes 25 tasks in parallel: $25 \times 10\ \text{ms} = 250\ \text{ms}$.
    
    Total time (threads) ≈ $250\ \text{ms} + 6\ \mu\text{s} + 4\ \mu\text{s} \approx 250\ \text{ms}$.
    Total time (processes) ≈ $250\ \text{ms} + 100\ \mu\text{s} + 40\ \mu\text{s} \approx 250\ \text{ms}$.

</div>

<div class="fragment">

??? success "**Conclusion.**"
    For **CPU-bound work with no sharing**, threads and processes are similar in total time — but threads use **much less memory** and have lower overhead. For **I/O-bound work**, threads win **decisively** because a blocked I/O doesn't block the whole process.

</div>

---

## 2.13 — Inter-Process Communication (IPC) Preview

???+ note "Why IPC?"
    Processes are **isolated** by design — one process cannot directly access another's memory. To cooperate, they need an OS-mediated channel.

<div class="fragment">

???+ note "Three IPC mechanisms"
    | Mechanism | Direction | Data |
    |---|---|---|
    | **Pipes** | Unidirectional | Byte stream |
    | **Message queues** | Bidirectional | Structured messages |
    | **Shared memory** | Bidirectional | Raw memory (fastest) |

</div>

<div class="fragment">

!!! tip "Shared memory is the fastest"
    After the initial setup, shared memory requires no kernel involvement — processes read and write directly. But it also requires **synchronization** (Chapter 4) — the OS cannot prevent race conditions on shared memory.

</div>

---

## Chapter 2 — key takeaways

<div class="fragment">

**1.** A **process** is a running program with memory (text, data, bss, heap, stack) and a **PCB** tracking its state.

</div>

<div class="fragment">

**2.** The **five-state model** (new, ready, running, waiting, terminated) captures process lifecycle; **waiting → running** is forbidden.

</div>

<div class="fragment">

**3.** `fork()` creates a child; `exec()` replaces the image; `wait()` collects the exit status. This three-call pattern is the Unix process model.

</div>

<div class="fragment">

**4.** **Zombies** (terminated but unreaped) and **orphans** (re-parented) are process states to monitor; zombies fill the process table.

</div>

<div class="fragment">

**5.** **Context switch** saves/restores state — direct cost ~µs, indirect cost (cache/TLB) 5–10× that.

</div>

<div class="fragment">

**6.** **Threads** share address space; per-thread state is small. Benefits: responsiveness, sharing, economy, scalability.

</div>

<div class="fragment">

**7.** **Thread models:** many-to-one (fast, no parallelism), one-to-one (Linux/Windows), many-to-many (Go, Erlang). Modern OSes use one-to-one.

</div>

---

## Next — Chapter 3

**CPU Scheduling.**

<div class="fragment">

- **Scheduling criteria:** CPU utilisation, throughput, turnaround, waiting, response.
- **Algorithms:** FCFS, SJF, SRTN, Round Robin, priority, MLFQ.
- **Traces** — waiting-time computations on real workloads.
- **Multiprocessor** scheduling, affinity.
- **Real-time** scheduling — RM and EDF.

</div>

<div class="fragment">

Chapter 2 described the process. Chapter 3 describes how the OS **decides which process to run next** — the scheduler.

</div>