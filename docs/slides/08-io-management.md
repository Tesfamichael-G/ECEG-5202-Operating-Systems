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
  js: assets/lecture-08.js

---

# ECEG-5202

## Chapter 8 — I/O Management

---

## Learning outcomes

<div class="fragment">

1. Classify I/O devices (**block**, **character**, **network**) and their interfaces.

</div>

<div class="fragment">

2. Describe the **four layers** of I/O software.

</div>

<div class="fragment">

3. Compare **programmed I/O**, **interrupt-driven I/O**, and **DMA**.

</div>

<div class="fragment">

4. Trace a **disk-scheduling** algorithm (FCFS, SSTF, SCAN, C-SCAN, LOOK) on a queue.

</div>

<div class="fragment">

5. Explain **buffering**, **caching**, and **spooling**.

</div>

---

## 8.1 — I/O Device Classes

???+ note "Three classes"
    | Class | Examples | Transfer unit | Access |
    |---|---|---|---|
    | **Block** | HDD, SSD, USB drive | Block (512 B – 4 KB) | Random |
    | **Character** | Keyboard, serial | Byte | Sequential |
    | **Network** | Ethernet, Wi-Fi | Packet | Streaming |

<div class="fragment">

???+ note "Device controllers"
    Each device has a **controller** — an embedded processor that:
    - Translates CPU commands to device actions.
    - Manages the protocol (SATA, NVMe, USB, PCIe).
    - Buffers data.
    - Reports status (ready, busy, error).

</div>

<div class="fragment">

???+ note "Device registers"
    | Register | Purpose |
    |---|---|
    | **Data** | Holds the transferred data |
    | **Status** | Ready, busy, error, done |
    | **Control** | Start, direction, interrupt enable |

</div>

<div class="fragment">

!!! tip "Memory-mapped vs isolated I/O"
    - **Memory-mapped** — device registers appear at memory addresses; accessed via normal `load`/`store`.
    - **Isolated** — separate address space; needs dedicated `IN`/`OUT` instructions (x86).
    
    Memory-mapped is more common in modern systems — uniform addressing, works with any memory instruction.

</div>

---

## 8.2 — The Four I/O Software Layers

```
┌────────────────────────────────┐
│  User-space I/O library        │  printf, fread, ...
├────────────────────────────────┤
│  OS-independent I/O software   │  buffering, spooling, names
├────────────────────────────────┤
│  Device drivers                │  device-specific code
├────────────────────────────────┤
│  Interrupt handlers            │  hardware → OS
└────────────────────────────────┘
```

<div class="fragment">

???+ note "What each layer does"
    | Layer | Responsibility |
    |---|---|
    | **User-space library** | Format I/O, call syscalls |
    | **OS-independent** | Buffering, caching, naming, protection |
    | **Device drivers** | Talk to the controller |
    | **Interrupt handlers** | Respond to device events |

</div>

<div class="fragment">

!!! tip "Device drivers are ~70% of the Linux kernel"
    Linux has ~30 million lines of code, of which ~20 million are device drivers. This is the practical cost of supporting thousands of hardware devices.

</div>

---

## 8.3 — Programmed I/O (Polling)

???+ note "The algorithm"
    ```
    while (!(status & READY)) ;
    data = device.data;
    ```

<div class="fragment">

???+ note "Trace"
    | Time | CPU | Device |
    |---|---|---|
    | 0 | issues read command | starts |
    | 1 | poll: not ready | working |
    | 2 | poll: not ready | working |
    | ... | ... | ... |
    | 1000 | poll: ready | done |
    | 1001 | read data | — |

</div>

<div class="fragment">

???+ note "Cost analysis"
    - For a 10 ms disk read: the CPU wastes 10⁷ cycles polling (at 1 GHz).
    - For a 1 µs device read: the CPU wastes 1,000 cycles — acceptable.

</div>

<div class="fragment">

!!! warning "When polling is a mistake"
    Any device slower than the CPU (most) makes polling a disaster. The CPU should not busy-wait. But polling is fine for **very fast devices** (network cards at 10 Gbps) or in **bare-metal embedded systems** with no interrupt controller.

</div>

---

## 8.4 — Interrupt-Driven I/O

???+ note "The algorithm"
    1. CPU issues a read command.
    2. CPU does other work.
    3. Device raises an interrupt when ready.
    4. CPU runs the interrupt handler, which reads the data.

<div class="fragment">

???+ note "Trace"
    | Time | CPU | Device |
    |---|---|---|
    | 0 | issues command, returns to work | starts |
    | 1–999 | runs other processes | working |
    | 1000 | interrupted | raises IRQ |
    | 1001 | runs handler, reads data | — |
    | 1002 | resumes | — |

</div>

<div class="fragment">

???+ note "Interrupt overhead"
    - **Context save/restore:** ~100 ns.
    - **Handler code:** depends on the device.
    - **Total:** ~1–10 µs per interrupt.

</div>

<div class="fragment">

???+ warning "Interrupt overload"
    A 10 Gbps network card receives ~15 million packets per second. If each packet causes an interrupt, the CPU is overwhelmed. Solutions:
    - **Interrupt coalescing** — batch many packets into one interrupt.
    - **NAPI** (Linux) — switch to polling under high load.
    - **DMA** — transfer batches, one interrupt per batch.

</div>

---

## 8.5 — Direct Memory Access (DMA)

???+ note "The algorithm"
    1. CPU programs the **DMA controller**: source, destination, length, direction.
    2. CPU continues execution.
    3. DMA controller **takes ownership of the bus** and transfers data.
    4. DMA controller raises an **interrupt** when done.

<div class="fragment">

???+ note "Trace"
    | Time | CPU | DMA controller | Device |
    |---|---|---|---|
    | 0 | sets up DMA | idle | idle |
    | 1 | runs other work | transfers blocks | streaming |
    | ... | ... | ... | ... |
    | 5000 | interrupted | done | done |
    | 5001 | acknowledges | idle | idle |

</div>

<div class="fragment">

???+ note "Cost analysis"
    - Setup: ~1 µs (CPU programs DMA registers).
    - Transfer: no CPU involvement.
    - Completion: ~1 µs (interrupt).
    
    Total CPU time: **~2 µs per transfer**, regardless of transfer size.

</div>

<div class="fragment">

???+ warning "Cycle stealing"
    The DMA controller **competes with the CPU** for the memory bus. During DMA transfers, the CPU's own memory accesses are slowed. Modern systems minimise this with:
    - **Direct Cache Access (DCA)** — DMA writes go through the CPU's cache.
    - **IOMMU** — separates device memory access from CPU memory access.

</div>

---

## 8.6 — Comparison of I/O Mechanisms

???+ note "The three approaches"
    | Aspect | Programmed | Interrupt | DMA |
    |---|---|---|---|
    | CPU per byte | Yes | No | No |
    | CPU during transfer | Busy-wait | Free | Free |
    | Per-transfer overhead | High | Medium | Low |
    | Throughput | Low | Medium | High |
    | Complexity | Simple | Moderate | Complex |
    | Best for | Simple embedded | Slow devices | Bulk transfers |

<div class="fragment">

!!! tip "Which to choose"
    - **Keyboard, mouse:** interrupt.
    - **Disk, network, USB:** DMA.
    - **Simple microcontroller with no interrupt controller:** polling.

</div>

---

## Worked example 8.1 — Disk I/O time

!!! tip "Problem:"
    A disk has:
    - Average seek time: 9 ms.
    - Rotation speed: 7200 RPM.
    - Transfer rate: 150 MB/s.
    - Block size: 4 KB.

    Compute the time for a random read.

<div class="fragment">

???+ note "**Step 1 — rotational latency.**"
    One revolution: $60 / 7200 = 8.333$ ms.
    Average: half a revolution = **4.17 ms**.

</div>

<div class="fragment">

??? info "**Step 2 — transfer time.**"
    $$
    T_{\text{transfer}} = \frac{4096}{150 \times 10^6} \approx 0.027\ \text{ms}.
    $$

</div>

<div class="fragment">

??? success "**Step 3 — total.**"
    $$
    T = 9 + 4.17 + 0.027 \approx \mathbf{13.2\ \text{ms}}.
    $$
    
    **IOPS** $\approx 1 / 0.0132 \approx \mathbf{76}$.

</div>

<div class="fragment">

??? tip "Sequential vs random"
    For sequential access, seek and rotation are amortised across many blocks. A sequential 1 GB read takes ~7 seconds (150 MB/s). A random 1 GB read takes 250,000 × 13.2 ms = **55 minutes**.

</div>

---

## 8.7 — Disk Scheduling: The Problem

???+ note "The situation"
    The disk receives I/O requests from many processes. The **order** in which requests are served affects total head movement.

<div class="fragment">

???+ note "The metrics"
    - **Total head movement** — cylinders traversed.
    - **Average wait time** — per request.
    - **Fairness** — no starvation.

</div>

<div class="fragment">

???+ note "The five algorithms"
    | Algorithm | Description | Starvation |
    |---|---|---|
    | **FCFS** | Serve in arrival order | No |
    | **SSTF** | Shortest seek time first | **Yes** |
    | **SCAN** | Elevator — sweep in one direction | No |
    | **C-SCAN** | Circular SCAN — always one direction | No |
    | **LOOK** | SCAN but reverse at last request | No |

</div>

---

## Worked example 8.2 — Disk scheduling comparison

!!! tip "Problem:"
    Request queue: `98, 183, 37, 122, 14, 124, 65, 67`. Head starts at **53**.

<div class="fragment">

???+ note "**FCFS** — serve in order."
    Path: 53 → 98 → 183 → 37 → 122 → 14 → 124 → 65 → 67.
    
    Movement: $45+85+146+85+108+110+59+2 = \mathbf{640}$.

</div>

<div class="fragment">

??? info "**SSTF** — shortest seek first."
    Path: 53 → 65 → 67 → 37 → 14 → 98 → 122 → 124 → 183.
    
    Movement: $12+2+30+23+84+24+2+59 = \mathbf{236}$.

</div>

<div class="fragment">

??? info "**SCAN (toward 0 first).**"
    Path: 53 → 37 → 14 → 0 → 65 → 67 → 98 → 122 → 124 → 183.
    
    Movement: $16+23+14+65+2+31+24+2+59 = \mathbf{236}$.

</div>

<div class="fragment">

??? info "**C-SCAN (toward 0, wrap to 199).**"
    Path: 53 → 37 → 14 → 0 → 199 → 183 → 124 → 122 → 98 → 67 → 65.
    
    Movement: $16+23+14+199+16+59+2+24+31+2 = \mathbf{386}$.

</div>

<div class="fragment">

??? success "**Observation.** SSTF minimises total movement (236) but is unfair — it can starve far-away requests. SCAN gives similar movement with better fairness. C-SCAN is worse on this queue but is **uniform** — every request waits about the same."

</div>

---

## 8.8 — The Elevator Analogy

???+ note "SCAN as an elevator"
    An elevator goes up, then down, then up — never reversing mid-floor unless it has to. SCAN does the same:
    - **Sweep** in one direction, serving all requests.
    - **Reverse** at the end of the disk.
    - **Sweep** back.

<div class="fragment">

???+ note "Why this works"
    By serving all requests in one direction before reversing, SCAN minimises the **total** seek distance. The elevator doesn't zigzag — it makes one efficient pass.

</div>

<div class="fragment">

???+ tip "LOOK vs SCAN"
    SCAN reverses at the **edge of the disk** (cylinder 0 or maximum). LOOK reverses at the **last request** in the current direction — saving the trip to the edge if there are no requests there. LOOK is the practical choice.

</div>

---

## 8.9 — Buffering

???+ note "Definition"
    **Buffering** stores data temporarily between a producer and a consumer. The OS uses buffers to absorb speed mismatches.

<div class="fragment">

???+ note "Three types"
    | Type | Buffers | Use case |
    |---|---|---|
    | **Single** | One buffer | Simple I/O |
    | **Double** | Two buffers | Streaming with overlap |
    | **Circular** | Ring of buffers | Continuous data (network, audio) |

</div>

<div class="fragment">

???+ note "Double buffering"
    While the producer fills buffer A, the consumer drains buffer B. When A is full and B is empty, they swap.

</div>

<div class="fragment">

???+ info "Why buffering matters"
    - **Speed mismatch** — CPU produces 10 GB/s; disk absorbs 150 MB/s.
    - **Size mismatch** — network packets are 1500 bytes; file systems write 4 KB blocks.
    - **Copy semantics** — buffering gives a consistent snapshot even if the source changes.

</div>

---

## 8.10 — Caching

???+ note "Definition"
    A **cache** stores recently used data to avoid re-fetching it.

<div class="fragment">

???+ note "Buffering vs caching"
    | Aspect | Buffering | Caching |
    |---|---|---|
    | Purpose | Absorb mismatch | Avoid re-fetching |
    | Data | Data in transit | Recently used data |
    | Lifetime | Until consumed | Until evicted |
    | Example | Network buffer | Buffer cache |

</div>

<div class="fragment">

???+ note "The buffer cache"
    The OS keeps a **page cache** of recently accessed file pages. A read that hits the cache avoids disk I/O — 100,000× faster.

</div>

<div class="fragment">

???+ info "Cache coherence"
    Multiple caches (file system cache, disk controller cache, application cache) can disagree. **`fsync()`** forces all caches to flush — essential for durability.

</div>

---

## 8.11 — Spooling

???+ note "Definition"
    **SPOOL** (Simultaneous Peripheral Operations On-Line) — queue output for a device that can only handle one job at a time.

<div class="fragment">

???+ note "The classic example"
    ```
    Process A → spooler queue → printer
    Process B →     │
    Process C →     │
                    ▼
              printed sequentially
    ```

</div>

<div class="fragment">

???+ info "Where spooling is used"
    - **Printers** — one printer, many jobs.
    - **Batch processing** — job cards submitted to a queue.
    - **Mail servers** — outgoing messages queued before sending.

</div>

<div class="fragment">

!!! tip "Spooling vs buffering"
    Buffering is a **temporary store** during a single I/O operation. Spooling is a **persistent queue** of jobs — it survives crashes and restarts.

</div>

---

## 8.12 — I/O Performance Measurement

???+ note "Key metrics"
    | Metric | Definition |
    |---|---|
    | **IOPS** | I/O operations per second |
    | **Throughput** | Bytes per second |
    | **Latency** | Time per operation |
    | **Queue depth** | Outstanding requests |

<div class="fragment">

???+ note "The trade-offs"
    - **High IOPS** — many small transfers (databases).
    - **High throughput** — large transfers (backup, video).
    - **Low latency** — few outstanding requests.

</div>

<div class="fragment">

???+ info "Typical numbers"
    | Device | IOPS (random 4K) | Throughput (seq) |
    |---|---|---|
    | HDD 7200 RPM | 75–100 | 100–200 MB/s |
    | SATA SSD | 100,000 | 500 MB/s |
    | NVMe SSD | 500,000+ | 3–7 GB/s |

</div>

<div class="fragment">

!!! tip "IOPS matters for databases"
    A relational database workload is dominated by **random 4 KB reads** (index lookups, row fetches). This is why NVMe SSDs gave 10× speedups to databases even though sequential throughput only doubled.

</div>

---

## Chapter 8 — key takeaways

<div class="fragment">

**1.** **I/O devices** are classified as **block**, **character**, or **network**. Each has its own access model.

</div>

<div class="fragment">

**2.** The four **I/O software layers** — user library, OS-independent, drivers, interrupt handlers — separate concerns cleanly.

</div>

<div class="fragment">

**3.** **Programmed I/O** busy-waits; **interrupt I/O** frees the CPU; **DMA** minimises CPU involvement for bulk transfers.

</div>

<div class="fragment">

**4.** **Disk scheduling** (FCFS, SSTF, SCAN, C-SCAN, LOOK) trades **throughput** against **fairness**. SCAN and LOOK are standard.

</div>

<div class="fragment">

**5.** **Buffering**, **caching**, and **spooling** absorb speed mismatches — each with a different use case.

</div>

<div class="fragment">

**6.** **IOPS** and **throughput** measure different workloads. Databases care about IOPS; backup cares about throughput.

</div>

---

## Next — Chapter 9

**Introduction to Security.**

<div class="fragment">

- **Threats**: malware (virus, worm, trojan), buffer overflow, DoS, social engineering.
- **Authentication**: passwords, biometrics, MFA.
- **Authorisation**: DAC, MAC, RBAC.
- **Defences**: sandboxing, ASLR, DEP, firewalls, encryption.

</div>

<div class="fragment">

Chapters 1–8 built a working OS. Chapter 9 asks: **what can go wrong?**

</div>