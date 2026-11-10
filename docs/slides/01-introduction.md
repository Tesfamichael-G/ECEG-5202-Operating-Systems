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
  js: assets/lecture-01.js

---

# ECEG-5202

## Chapter 1 — Introduction to Operating Systems

---

## Learning outcomes

<div class="fragment">

1. Define what an **operating system** is, and why it exists.

</div>

<div class="fragment">

2. Trace the **evolution** of OS from batch to mobile, and identify what changed at each step.

</div>

<div class="fragment">

3. Compare **monolithic**, **layered**, **microkernel**, **modular**, and **hybrid** OS structures.

</div>

<div class="fragment">

4. Describe the **computer system organisation** (CPU, memory, I/O, buses) and the role of **interrupts**.

</div>

<div class="fragment">

5. Explain **dual-mode operation** and its role in OS protection.

</div>

<div class="fragment">

6. Trace a **system call** from user space to kernel and back.

</div>

---

## 1.1 — What is an Operating System?

???+ note "Definition (Tanenbaum §1.1)"
    An **operating system** is a program that acts as an **intermediary between the user and the computer hardware**. Its primary goals are **convenience** for the user and **efficient** utilisation of the hardware.

<div class="fragment">

???+ info "Two views"
    **Top-down (user's view):** the OS is the interface that makes the hardware usable — files, processes, sockets, windows.
    
    **Bottom-up (resource manager):** the OS is the allocator of CPU, memory, devices, and storage among competing processes.

</div>

<div class="fragment">

!!! tip "Where does the OS live?"
    The **kernel** is the part of the OS that runs in **privileged mode** and has direct access to hardware. User-space programs (including most of the OS "system utilities") run in **unprivileged mode** and interact with the kernel via **system calls**.

</div>

<aside class="notes">
  <div style="font-size: 16px; line-height: 1.6;">
Open by anchoring the two views. Students often think "OS = Windows/Ubuntu GUI" — but the GUI is just one user-space application. The OS is the kernel: the layer that mediates hardware access. Emphasise this distinction early; it will come up again in every chapter.
  </div>
</aside>

---

## 1.1 — What the OS Manages

```
┌──────────────────────────────────────────────────┐
│                 Applications                     │
├──────────────────────────────────────────────────┤
│              Operating System (Kernel)           │
│  ┌──────────┬──────────┬───────────┬──────────┐ │
│  │ Process  │ Memory   │ File      │ I/O      │ │
│  │ Manager  │ Manager  │ System    │ Manager  │ │
│  └──────────┴──────────┴───────────┴──────────┘ │
├──────────────────────────────────────────────────┤
│                    Hardware                      │
└──────────────────────────────────────────────────┘
```

<div class="fragment">

???+ note "Four resource categories"
    | Resource | Managed by | Chapter |
    |---|---|---|
    | **CPU** | Scheduler | 3 |
    | **Memory** | Memory manager | 6 |
    | **Storage / files** | File system | 7 |
    | **I/O devices** | I/O subsystem | 8 |

</div>

<div class="fragment">

!!! warning "The OS is not a process"
    The kernel is **not** a process that runs periodically. It is **invoked** on demand — by system calls, interrupts, and exceptions. Between invocations, it is dormant code sitting in memory.

</div>

---

## 1.2 — The OS as an Abstraction Layer

???+ note "The layering"
    ```
    User program
         │  printf()       ← libc
         ▼
    System-call wrapper
         │  write(fd, buf, n)
         ▼
    Kernel
         │  performs the actual I/O
         ▼
    Hardware
    ```

<div class="fragment">

???+ info "What abstraction buys you"
    - **Portability** — the same `write()` works on any disk.
    - **Safety** — a program cannot corrupt the disk directly.
    - **Simplicity** — a program does not manage DMA channels.
    - **Sharing** — the OS multiplexes the disk across processes.

</div>

<div class="fragment">

!!! tip "The three windows every OS exposes"
    Every OS provides at least three abstractions:
    
    1. **Process** — a running program with its own address space.
    2. **File** — a persistent, named byte stream.
    3. **Socket (or pipe)** — a channel for communication.

</div>

---

## 1.3 — Evolution of Operating Systems

???+ note "The four generations"
    | Gen | Years | Hardware | OS feature |
    |---|---|---|---|
    | **0th** | 1940s | Vacuum tubes, plugboards | No OS — direct hardware control |
    | **1st** | 1950s | Transistors, mainframes | **Batch monitors** — automated job sequencing |
    | **2nd** | 1960s | ICs | **Multiprogramming**, **spooling**, **time-sharing** |
    | **3rd** | 1970s–80s | LSI/VLSI | **UNIX**, **MS-DOS**, networked OS |
    | **4th** | 1990s–present | VLSI, mobile, cloud | **GUI**, **mobile**, **virtualisation**, **containers** |

<div class="fragment">

???+ note "What changed at each step"
    - **Batch monitor** — eliminated the human operator between jobs.
    - **Multiprogramming** — several jobs in memory; when one blocks on I/O, the CPU switches to another.
    - **Time-sharing** — many interactive users; the OS switches rapidly so each user perceives the system as dedicated.
    - **Virtualisation** — one physical machine runs many isolated VMs, each with its own kernel.

</div>

---

## 1.4 — Batch Systems

???+ note "The batch model (1950s)"
    ```
    Job 1  ──►  Memory  ──►  CPU  ──►  output
    Job 2  ──►     │
    Job 3  ──►     │  wait (job 1 finishes)
                   ▼
              "batch" queued on tape
    ```

<div class="fragment">

???+ info "The batch monitor"
    - Reads jobs from a **card reader or tape**.
    - Transfers control to the job.
    - When the job finishes (or crashes), control returns to the monitor.
    - The monitor is the first **resident** program — always in memory.

</div>

<div class="fragment">

!!! warning "The bottleneck"
    The CPU is **idle** whenever a job is doing I/O. With tape-based I/O taking seconds per byte, CPU utilisation was often < 10%.

</div>

---

## 1.5 — Multiprogramming and Time-Sharing

???+ note "Multiprogramming"
    Multiple jobs kept in memory at once. When the running job blocks on I/O, the CPU switches to another.

```
Memory:
┌──────────┐
│ Job 1    │ ← running
├──────────┤
│ Job 2    │ ← ready
├──────────┤
│ Job 3    │ ← blocked on I/O
└──────────┘
```

<div class="fragment">

???+ note "Time-sharing (multitasking)"
    A special case of multiprogramming where the CPU switches between jobs rapidly (every 10–100 ms) to give each **interactive user** the illusion of a dedicated machine.

</div>

<div class="fragment">

???+ info "The two key mechanisms"
    - **Context switch** — save the state of the running process, restore the state of the next.
    - **Timer interrupt** — the hardware periodically forces a return to the kernel, so no process can monopolise the CPU.

</div>

---

## 1.6 — OS Structures: The Five Families

???+ note "How is the kernel organised internally?"
    | Structure | Description | Example |
    |---|---|---|
    | **Monolithic** | One big kernel; all services in one address space | Traditional Linux |
    | **Layered** | Hierarchy of layers; each uses only the one below | THE OS (1968) |
    | **Microkernel** | Only essential services in kernel; the rest in user space | QNX, MINIX, seL4 |
    | **Modular** | Monolithic core + loadable modules | Linux, Solaris |
    | **Hybrid** | Mix of approaches | Windows NT, macOS |

---

## 1.7 — Monolithic Kernel

???+ note "The structure"
    ```
    ┌──────────────────────────────────────┐
    │  Scheduler  Memory  FS  Network      │
    │  Drivers    IPC     Security         │
    │         (single address space)        │
    └──────────────────────────────────────┘
    ```

<div class="fragment">

???+ info "Pros"
    - **Fast** — direct function calls between subsystems.
    - **Simple** — one linking model, one debugger.
    - **Mature** — decades of optimisation.

</div>

<div class="fragment">

???+ info "Cons"
    - **Large** — Linux kernel is ~30 million lines.
    - **Fragile** — one buggy driver can crash the whole kernel.
    - **Hard to maintain** — subsystems interlink.

</div>

<div class="fragment">

!!! tip "Why Linux is monolithic today"
    Linux started as a monolithic kernel (1991) and remains one. Efforts to add microkernel features (FUSE for user-space filesystems, eBPF for programmable kernel extensions) soften the monolithic edges, but the core design has not changed.

</div>

---

## 1.8 — Layered Structure

???+ note "The layered OS (Dijkstra 1968)"
    ```
    Layer 5 ── User programs
    Layer 4 ── I/O management
    Layer 3 ── Memory management
    Layer 2 ── Process scheduling
    Layer 1 ── Memory and I/O primitives
    Layer 0 ── Hardware
    ```

<div class="fragment">

???+ info "The rule"
    Layer $N$ may call **only** layer $N-1$. There is no upward or same-layer call.

</div>

<div class="fragment">

???+ info "Pros"
    - **Clean design** — each layer is testable in isolation.
    - **Well-documented interface** — each layer's API is clear.

</div>

<div class="fragment">

???+ warning "Cons"
    - **Performance** — every operation crosses multiple layers.
    - **Hard to define layers** — some operations need to skip layers.
    - **Rarely used in practice** — the THE OS is the classic example.

</div>

---

## 1.9 — Microkernel

???+ note "The structure"
    ```
    ┌──────────────────────────────────────────┐
    │  User space: FS, drivers, network servers │
    ├──────────────────────────────────────────┤
    │  Kernel: IPC + scheduling + basic memory  │
    └──────────────────────────────────────────┘
    ```

<div class="fragment">

???+ info "The three principles"
    1. **Minimal kernel** — only IPC, scheduling, and low-level memory.
    2. **All services user-space** — file systems and drivers run as processes.
    3. **Message passing** — services communicate via kernel IPC.

</div>

<div class="fragment">

???+ info "Pros"
    - **Reliability** — a crashed driver is a crashed process, restarted by the kernel.
    - **Security** — the attack surface in kernel mode is tiny.
    - **Formal verification** — **seL4** is a formally verified microkernel.

</div>

<div class="fragment">

???+ info "Cons"
    - **Performance** — every system call becomes a message round trip.
    - **Complexity** — IPC must be fast and correct.

</div>

<div class="fragment">

!!! tip "Where microkernels live"
    QNX (automotive, medical), MINIX 3 (embedded), seL4 (defence, aerospace), macOS's **Mach** layer (hybrid). The performance gap has narrowed — QNX's IPC is ~1 µs.

</div>

---

## 1.10 — Computer System Organisation

```
┌───────────┐        ┌──────────┐        ┌───────────┐
│    CPU    │◄──────►│  Memory  │◄──────►│   DMA     │
└─────┬─────┘  Bus   └──────────┘  Bus   └─────┬─────┘
      │                                       │
      └──────────┬────────────────────────────┘
                 │
           ┌─────▼─────┐
           │   Disk    │
           └───────────┘
```

<div class="fragment">

???+ note "The four components"
    - **CPU** — executes instructions, manages registers.
    - **Memory** — holds instructions and data.
    - **Buses** — carry addresses, data, and control signals.
    - **Devices** — disks, network, keyboard, display.

</div>

<div class="fragment">

!!! tip "The bootstrap sequence"
    1. Power on → CPU fetches instruction from address 0.
    2. **BIOS/UEFI firmware** runs from ROM.
    3. Firmware loads the **bootloader** from disk.
    4. Bootloader loads the **kernel** into memory.
    5. Kernel initialises devices, memory, scheduler.
    6. Kernel launches **init** (or **systemd**), which starts user services.

</div>

---

## 1.11 — Interrupts

???+ note "Definition"
    An **interrupt** is a signal that causes the CPU to **suspend** its current execution and transfer control to an **interrupt handler**.

<div class="fragment">

???+ note "Three types"
    | Type | Source | Example |
    |---|---|---|
    | **Hardware interrupt** | External device | Timer expires, disk completes |
    | **Trap / exception** | Current instruction | Divide by zero, page fault |
    | **Software interrupt** | Explicit instruction | System call (`int 0x80` on x86) |

</div>

<div class="fragment">

???+ note "The interrupt vector table"
    Each interrupt is assigned a **number**. The **IVT** maps the number to the handler's address.
    
    ```
    IVT[0]  → divide_error_handler
    IVT[14] → page_fault_handler
    IVT[32] → timer_handler
    IVT[33] → keyboard_handler
    ```

</div>

<div class="fragment">

???+ info "What happens on interrupt"
    1. CPU saves the current **program counter** and **flags** on the stack.
    2. CPU switches to **kernel mode** (privileged).
    3. CPU jumps to the handler address from the IVT.
    4. Handler runs, may signal the waiting process.
    5. **`iret`** (interrupt return) restores the previous state.
    6. CPU resumes the interrupted program.

</div>

---

## Worked example 1.1 — Trace a keyboard interrupt

!!! tip "Problem:"
    A program is running a calculation. The user presses 'A'. Trace the sequence.

<div class="fragment">

???+ note "**Step 1 — hardware.**"
    Keyboard controller raises IRQ 1 (interrupt request line 1). The CPU, if not masked, accepts the interrupt.

</div>

<div class="fragment">

??? info "**Step 2 — save state.**"
    CPU pushes PC, flags, and register state onto the kernel stack. Switches to ring 0.

</div>

<div class="fragment">

??? info "**Step 3 — dispatch.**"
    CPU jumps to the keyboard handler address from the IVT (typically `IVT[33]` if vector numbers are shifted).

</div>

<div class="fragment">

??? info "**Step 4 — handler.**"
    Reads the scancode from the keyboard controller port. Translates it to 'A'. Places 'A' in the input buffer. Signals any process waiting for keyboard input.

</div>

<div class="fragment">

??? info "**Step 5 — return.**"
    Handler executes `iret`. CPU restores PC and flags. Resumes the calculation. The 'A' is now available for the next `read()` system call.

</div>

<div class="fragment">

??? success "**Observation.**"
    The calculation **never noticed** the interrupt — except for a microsecond of delay. This is the fundamental abstraction of interrupts: **transparent preemption**.

</div>

---

## 1.12 — Dual-Mode Operation

???+ note "The two modes"
    | Mode | Privilege | What runs |
    |---|---|---|
    | **Kernel mode** (ring 0 on x86) | Full hardware access | The OS kernel |
    | **User mode** (rings 1–3) | Restricted | Applications |

<div class="fragment">

???+ info "The mode bit"
    The CPU status register contains a **mode bit**:
    - `0` = kernel mode (privileged instructions allowed).
    - `1` = user mode (privileged instructions trap to the kernel).

</div>

<div class="fragment">

???+ note "Privileged instructions"
    Instructions that only the kernel can execute:
    - **Halt** the CPU.
    - **Disable interrupts**.
    - **Load the page table base register**.
    - **Direct I/O port access**.
    - **Modify the mode bit itself**.

</div>

<div class="fragment">

!!! warning "Why this matters"
    Without dual-mode operation, a buggy user program could **disable interrupts** and hang the machine, or **reprogram the disk controller** and corrupt the disk. Dual-mode is the foundation of OS protection.

</div>

---

## 1.13 — System Calls

???+ note "Definition"
    A **system call** is the mechanism by which a user-space program requests a service from the kernel.

<div class="fragment">

???+ note "The three-step sequence"
    ```
    User:   write(1, "hello", 5)
              │  (library wrapper)
              ▼
    User:   mov $1, %eax        ← syscall number
            syscall             ← trap to kernel
              │
              ▼
    Kernel: dispatch to sys_write()
              │  performs the I/O
              ▼
    Kernel: iret                ← return to user
              │
              ▼
    User:   return from write()
    ```

</div>

<div class="fragment">

???+ note "Common system calls"
    | Category | Examples |
    |---|---|
    | **Process** | `fork`, `exec`, `exit`, `wait` |
    | **File** | `open`, `read`, `write`, `close`, `unlink` |
    | **Memory** | `brk`, `mmap`, `munmap` |
    | **Network** | `socket`, `bind`, `listen`, `accept`, `connect` |
    | **Signal** | `kill`, `signal`, `sigaction` |

</div>

<div class="fragment">

!!! tip "System calls are expensive"
    A system call involves a mode switch (saving/restoring registers), TLB effects, and cache effects. Typical cost: **500–2000 ns** — compared to a user-space function call at ~1 ns. This is why real programs use **buffered I/O** (e.g., `fwrite` in libc accumulates bytes and issues one `write` per 4 KB).

</div>

---

## Worked example 1.2 — Trace a `write()` system call

!!! tip "Problem:"
    A program calls `write(1, "hello\n", 6)`. Trace the full path.

<div class="fragment">

???+ note "**Step 1 — library wrapper.**"
    `write()` in libc loads the syscall number for `write` (1 on x86-64) into register `rax`, the file descriptor into `rdi`, the buffer into `rsi`, and the length into `rdx`.

</div>

<div class="fragment">

??? info "**Step 2 — trap.**"
    Executes `syscall` instruction. CPU switches to ring 0, jumps to the kernel's syscall entry point.

</div>

<div class="fragment">

??? info "**Step 3 — dispatch.**"
    Kernel reads `rax`, looks up `sys_call_table[1]` = `sys_write`. Calls it.

</div>

<div class="fragment">

??? info "**Step 4 — execute.**"
    `sys_write` validates the file descriptor (1 = stdout). Copies the 6 bytes to the terminal driver's buffer. Returns 6.

</div>

<div class="fragment">

??? info "**Step 5 — return.**"
    Kernel puts the return value (6) in `rax`. Executes `sysret`. CPU returns to ring 3 at the instruction after `syscall`.

</div>

<div class="fragment">

??? success "**Observation.**"
    The program sees a normal function returning 6. The mode switch, validation, and I/O were all **transparent** — the abstraction worked.

</div>

---

## 1.14 — Kernel Data Structures

???+ note "What the kernel needs to track"
    | Data structure | Purpose |
    |---|---|
    | **Process control block (PCB)** | Per-process state (Chapter 2) |
    | **Page tables** | Virtual-to-physical mappings (Chapter 6) |
    | **Inode table** | Open file descriptors (Chapter 7) |
    | **Buffer cache** | Cached disk blocks (Chapter 8) |
    | **Run queue** | Runnable processes (Chapter 3) |

<div class="fragment">

!!! tip "The kernel as a giant data structure"
    Modern kernels are, in large part, **data-structure management systems**. The scheduler manages the run queue; the memory manager manages the page tables; the file system manages inodes. Understanding the kernel means understanding these structures.

</div>

---

## 1.15 — Real-World OS Examples

???+ note "The three dominant families"
    | Family | Examples | Architecture | Key trait |
    |---|---|---|---|
    | **Unix-like** | Linux, macOS, BSD, Android | Monolithic / hybrid | Portable, POSIX |
    | **Windows NT** | Windows 10/11, Server | Hybrid | Win32 API, registry |
    | **Real-time** | QNX, VxWorks, FreeRTOS | Microkernel / small | Deterministic |

<div class="fragment">

???+ info "Android's structure"
    Android uses a **modified Linux kernel** with a custom **user-space stack**: Bionic libc, ART runtime, Android framework. The kernel is monolithic; the user space is a Java-like managed runtime.

</div>

<div class="fragment">

???+ info "Linux's scale"
    - ~30 million lines of code.
    - ~2,000 contributors per release.
    - Runs on mobile (Android), servers (AWS, GCP), supercomputers (100% of TOP500), embedded (routers, cars).

</div>

---

## Chapter 1 — key takeaways

<div class="fragment">

**1.** An **OS** is an intermediary between user and hardware, providing **convenience** and **efficient resource utilisation**.

</div>

<div class="fragment">

**2.** Four resources managed: **CPU, memory, storage, I/O** — each with its own subsystem.

</div>

<div class="fragment">

**3.** The OS's evolution — batch → multiprogramming → time-sharing → virtualisation — each step increased **utilisation** and **interactivity**.

</div>

<div class="fragment">

**4.** Five kernel structures: **monolithic** (fast but fragile), **layered** (clean but slow), **microkernel** (reliable but IPC-heavy), **modular** (hybrid), **hybrid** (Windows/macOS).

</div>

<div class="fragment">

**5.** **Interrupts** enable transparent preemption; the CPU saves state, dispatches to a handler, and returns.

</div>

<div class="fragment">

**6.** **Dual-mode operation** (kernel vs user) protects the OS — privileged instructions trap to the kernel.

</div>

<div class="fragment">

**7.** **System calls** are the interface — mode switch + dispatch + service + return. Cost: ~1 µs.

</div>

---

## Next — Chapter 2

**Processes.**

<div class="fragment">

- The **process concept**: program in execution, memory layout, PCB.
- The **five-state model** — new, ready, running, waiting, terminated.
- **Process operations**: `fork`, `exec`, `wait`, `exit`.
- **Threads**: models, benefits, and the context switch.
- **Inter-process communication** basics.

</div>

<div class="fragment">

Chapter 1 described the OS's role. Chapter 2 describes its first job — **managing processes**.

</div>