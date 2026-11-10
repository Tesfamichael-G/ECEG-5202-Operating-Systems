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
  js: assets/lecture-06.js

---

# ECEG-5202

## Chapter 6 — Memory Management

---

## Learning outcomes

<div class="fragment">

1. Explain **address binding** (compile, load, execution) and the role of the **MMU**.

</div>

<div class="fragment">

2. Distinguish **fixed**, **variable**, and **buddy-system** partitioning; identify fragmentation types.

</div>

<div class="fragment">

3. Translate a **virtual address** to a **physical address** in a paging system, using the **TLB**.

</div>

<div class="fragment">

4. Compare **paging** and **segmentation**.

</div>

<div class="fragment">

5. Trace **page-replacement algorithms** (FIFO, LRU, Optimal, Clock) on a reference string.

</div>

<div class="fragment">

6. Define and detect **thrashing**; use the **working-set** model.

</div>

---

## 6.1 — The Memory Hierarchy

```
Registers         ← CPU
    │
L1 Cache          ~32 KB
    │
L2 Cache          ~256 KB
    │
L3 Cache          ~8 MB
    │
Main Memory       ~8–32 GB
    │
Disk (SSD/HDD)    ~TB
```

<div class="fragment">

!!! tip "The management problem"
    Each process wants **all the memory it needs**, at high speed, isolated from other processes. The OS and MMU together provide the illusion of **more memory** and **private memory** — using caches, paging, and virtual memory.

</div>

---

## 6.2 — Address Binding

???+ note "Three times"
    | When | Binding | Consequence |
    |---|---|---|
    | **Compile time** | Absolute addresses | Program must load at a fixed address |
    | **Load time** | Relocatable addresses | Program can load anywhere, fixed at load |
    | **Execution time** | Virtual addresses | Program can move during execution |

<div class="fragment">

???+ info "Modern systems use execution-time binding"
    Every load/store instruction computes a **virtual address**, which the **MMU** translates to a **physical address** on every memory access. This enables:
    - **Paging** — page can be anywhere in physical memory.
    - **Swapping** — page can be on disk and brought in on demand.
    - **Protection** — page permissions enforced by the MMU.

</div>

---

## 6.3 — The Memory Management Unit

???+ note "The MMU's job"
    Translate every virtual address to a physical address.

```
CPU ──► Virtual address ──► MMU ──► Physical address ──► Memory
                              │
                              └── consults page table
```

<div class="fragment">

???+ info "Two mappings"
    - **Paging** — virtual page number + offset → physical frame number + offset.
    - **Segmentation** — segment number + offset → base + offset.

</div>

<div class="fragment">

!!! tip "The MMU is hardware"
    The page-table walk is implemented in silicon. Without the MMU, every memory access would need an OS system call — 1000× slower.

</div>

---

## 6.4 — Fixed Partitioning

???+ note "The idea"
    Divide memory into **fixed-size partitions**. Each process gets one partition.

```
Memory:
┌──────────┐
│ Part 1   │  ← process A (fits)
├──────────┤
│ Part 2   │  ← process B (fits)
├──────────┤
│ Part 3   │  ← EMPTY (no process of this size)
├──────────┤
│ Part 4   │  ← process C (fits)
└──────────┘
```

<div class="fragment">

???+ warning "Internal fragmentation"
    A process **smaller** than its partition wastes the remainder. If partition = 100 MB and process = 30 MB, 70 MB is wasted.

</div>

<div class="fragment">

???+ info "How bad?"
    In the worst case, up to 50% of memory is wasted. This is why fixed partitioning is rarely used today (except in embedded systems).

</div>

---

## 6.5 — Variable Partitioning

???+ note "The idea"
    Allocate each process exactly the memory it needs. Partitions are **variable-sized**.

<div class="fragment">

???+ warning "External fragmentation"
    Free memory is scattered in small **holes** between allocations. Even if the total free memory is enough, **no single hole** may be large enough.

</div>

<div class="fragment">

???+ note "Allocation strategies"
    | Strategy | Behaviour |
    |---|---|
    | **First-fit** | First hole large enough |
    | **Best-fit** | Smallest hole large enough |
    | **Worst-fit** | Largest hole |

</div>

<div class="fragment">

???+ info "Fixes for external fragmentation"
    - **Compaction** — shuffle memory to coalesce holes. Expensive.
    - **Paging** — eliminate the requirement for contiguity (Section 6.7).
    - **Segmentation** — still has fragmentation but with logical meaning.

</div>

---

## Worked example 6.1 — First-fit vs best-fit

!!! tip "Problem:"
    Memory holes: [100, 500, 200, 300, 600]. Requests in order: [212, 417, 112, 426].

<div class="fragment">

???+ note "**First-fit.**"
    - 212 → hole 500 (splits to 288). Holes: [100, 288, 200, 300, 600].
    - 417 → hole 600 (splits to 183). Holes: [100, 288, 200, 300, 183].
    - 112 → hole 288 (splits to 176). Holes: [100, 176, 200, 300, 183].
    - 426 → no hole ≥ 426. **Fails.**

</div>

<div class="fragment">

??? info "**Best-fit.**"
    - 212 → hole 300 (splits to 88). Holes: [100, 500, 200, 88, 600].
    - 417 → hole 500 (splits to 83). Holes: [100, 83, 200, 88, 600].
    - 112 → hole 200 (splits to 88). Holes: [100, 83, 88, 88, 600].
    - 426 → hole 600 (splits to 174). Holes: [100, 83, 88, 88, 174].
    
    **All succeed.**

</div>

<div class="fragment">

??? success "**Observation.**"
    Best-fit does more work (searches all holes) but produces **smaller leftover holes** and **fewer failures**. First-fit is faster but more likely to leave large unusable holes.

</div>

---

## 6.6 — Buddy System

???+ note "The idea"
    Memory is allocated in **power-of-two** blocks. A request is rounded up to the next power of two. If unavailable, a larger block is **split in half** (buddies).

<div class="fragment">

???+ note "Example: request 50 KB"
    - Round up to 64 KB.
    - If no 64 KB block, take a 128 KB block and split into two 64 KB buddies.
    - If no 128 KB block, split 256 KB, etc.

</div>

<div class="fragment">

???+ info "Finding a buddy"
    If a block is at address $A$ and has size $2^k$, its buddy is at address $A \oplus 2^k$ (XOR).

</div>

<div class="fragment">

???+ note "Pros and cons"
    - **Fast allocation and coalescing** — buddy find is O(1).
    - **Internal fragmentation** — rounded up to power of two → up to 50% waste.

</div>

---

## 6.7 — Paging

???+ note "The idea"
    Divide **physical memory** into fixed-size **frames** (e.g., 4 KB).
    Divide **logical memory** into same-size **pages**.
    Pages can go in **any frame** — no contiguity needed.

<div class="fragment">

???+ note "Address translation"
    ```
    Virtual address:
    ┌─────────────┬────────┐
    │ page number │ offset │
    └─────────────┴────────┘
           │           │
           ▼           │
    ┌─────────────┐    │
    │ page table  │    │
    └─────────────┘    │
           │           │
           ▼           ▼
    ┌─────────────┬────────┐
    │frame number │ offset │
    └─────────────┴────────┘
    Physical address
    ```

</div>

<div class="fragment">

???+ success "Paging eliminates external fragmentation"
    Every page fits in every frame. Free frames can be reused without coalescing.

</div>

<div class="fragment">

???+ warning "Internal fragmentation remains"
    The **last page** of a process may be partially filled — waste up to (page_size − 1) bytes. With 4 KB pages, average waste is 2 KB per process.

</div>

---

## 6.8 — The Page Table

???+ note "Definition"
    The **page table** maps page numbers to frame numbers. One entry per page of the process.

<div class="fragment">

???+ note "Page table entry (PTE)"
    ```
    ┌─────┬─────┬──────────────────┐
    │ Valid│ Dirty│ Frame number     │
    └─────┴─────┴──────────────────┘
    ```
    
    - **Valid bit** — is the page in memory?
    - **Dirty bit** — has the page been written?
    - **Reference bit** — has the page been accessed? (for LRU/Clock)
    - **Protection bits** — read/write/execute.

</div>

<div class="fragment">

!!! warning "Where is the page table stored?"
    In **physical memory** (not in a register). Every memory access requires reading a PTE — **doubling** the memory traffic. This is why the TLB exists.

</div>

---

## 6.9 — The TLB

???+ note "Translation Lookaside Buffer"
    A small **associative cache** of recent page-table entries. On a TLB hit, translation is fast; on a miss, the page table is consulted.

<div class="fragment">

???+ note "Effective access time (EAT)"
    $$
    \text{EAT} = \alpha \cdot (t_{\text{TLB}} + t_{\text{mem}}) + (1-\alpha) \cdot (t_{\text{TLB}} + 2 t_{\text{mem}})
    $$
    
    where $\alpha$ is the TLB hit rate.

</div>

<div class="fragment">

???+ info "Typical numbers"
    - $t_{\text{TLB}} = 1$ ns, $t_{\text{mem}} = 100$ ns, $\alpha = 0.99$.
    - EAT = $0.99 \times 101 + 0.01 \times 201 = 99.99 + 2.01 = 102$ ns.
    
    A 1% TLB miss rate **doubles** the memory access time — TLB hit rate matters enormously.

</div>

<div class="fragment">

!!! tip "TLB flushing on context switch"
    When the OS switches to a different process, the TLB entries become invalid — they belong to the old process's address space. Modern CPUs use **ASIDs** (address space IDs) to tag TLB entries, avoiding the flush.

</div>

---

## Worked example 6.2 — Paging translation

!!! tip "Problem:"
    Page size = 4 KB (offset = 12 bits). Virtual address = `0x00002A3F`. Page table:
    
    | Page # | Frame # |
    |---|---|
    | 0 | 5 |
    | 1 | 3 |
    | 2 | 8 |

    Find the physical address.

<div class="fragment">

???+ note "**Step 1 — split the address.**"
    `0x00002A3F` = binary `0000 0000 0000 0000 0010 1010 0011 1111`.
    
    - Offset (low 12 bits) = `0xA3F` = 2623.
    - Page number = `0x2` = 2.

</div>

<div class="fragment">

??? info "**Step 2 — look up the page table.**"
    Page 2 → frame 8.

</div>

<div class="fragment">

??? success "**Step 3 — assemble the physical address.**"
    Physical = frame 8 × 4 KB + offset 2623 = 32768 + 2623 = **35391 = 0x8A3F**.

</div>

<div class="fragment">

???+ info "**Sanity check.**"
    The physical address should have the same low 12 bits as the virtual address (`A3F`), with the frame number in the high bits. ✓

</div>

---

## 6.10 — Multi-Level Page Tables

???+ note "The problem"
    For a 32-bit address space with 4 KB pages: $2^{20}$ pages = ~1M entries per page table = 4 MB per process. For 1000 processes, that's 4 GB just for page tables.

<div class="fragment">

???+ note "The solution — multi-level page tables"
    Split the page table into a tree. Only the levels that are actually used need to be allocated.

```
Virtual address:  | PGD | PUD | PMD | PTE | offset |
                    │
                    ▼
                Page global directory
                    │
                    ▼
                Page upper directory
                    │
                    ▼
                Page middle directory
                    │
                    ▼
                Page table entry
```

</div>

<div class="fragment">

???+ info "Typical x86-64 layout"
    - 4 levels of page tables.
    - Each level uses 9 bits of the 48-bit virtual address.
    - 12 bits offset.
    - Total: 4 × 9 + 12 = 48 bits.

</div>

<div class="fragment">

!!! warning "Multi-level tables are slower"
    Each level of the tree is one memory access. A 4-level walk needs **4 memory accesses** per TLB miss. This is why TLB hit rate is so critical.

</div>

---

## 6.11 — Segmentation

???+ note "The idea"
    Divide the program into **logical segments** — code, data, stack, heap — each with its own base and limit.

<div class="fragment">

???+ note "Address translation"
    Segment number + offset → segment table lookup → physical address.

</div>

<div class="fragment">

???+ note "Pros"
    - **Matches programmer's view** of memory.
    - **Natural protection** — code segment read-only, stack writable.
    - **Sharing** — two processes share the same code segment.

</div>

<div class="fragment">

???+ note "Cons"
    - **External fragmentation** — segments are variable-size.
    - **Complex hardware** — segment table plus offset check.

</div>

<div class="fragment">

!!! tip "Paging vs segmentation"
    | Aspect | Paging | Segmentation |
    |---|---|---|
    | Size | Fixed | Variable |
    | Internal frag | Yes (last page) | No |
    | External frag | No | Yes |
    | Programmer view | Invisible | Matches structure |
    | Modern use | Yes (with segmentation) | Mostly historical |

</div>

---

## 6.12 — Virtual Memory

???+ note "The idea"
    Give each process a **virtual address space** larger than physical memory. Only the **currently used** pages are in memory; the rest are on disk.

<div class="fragment">

???+ note "Demand paging"
    A page is loaded **on demand** — when the process accesses it, and only then. If the page is not in memory, a **page fault** occurs.

</div>

<div class="fragment">

???+ note "The page-fault handler"
    1. Trap to kernel (page fault).
    2. Find a **free frame** (or evict one).
    3. Read the page from disk.
    4. Update the page table (mark valid).
    5. Restart the faulting instruction.

</div>

<div class="fragment">

!!! tip "Why virtual memory works"
    Programs exhibit **locality** — at any moment, they access a small subset of their pages (the working set). The OS keeps this working set in memory and swaps the rest to disk.

</div>

---

## 6.13 — Page Replacement Algorithms

???+ note "The problem"
    Memory is full. A new page must be loaded. Which page to evict?

<div class="fragment">

???+ note "The four classic algorithms"
    | Algorithm | Rule | Optimal? |
    |---|---|---|
    | **FIFO** |