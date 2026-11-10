# Chapter 6 — Memory Management

!!! abstract "Chapter at a glance"
    **Coverage:** introduction · memory partitioning · paging · segmentation.

---

## Learning outcomes

1. Explain the need for memory management and the role of the MMU.
2. Compare fixed, variable, and buddy-system partitioning.
3. Describe paging, page tables, and the TLB.
4. Describe segmentation and compare it with paging.

---

## 6.1 Introduction

The **memory manager** tracks which parts of memory are in use, allocates and frees memory on request, and translates **virtual addresses** to **physical addresses** via the **memory management unit (MMU)**.

**Address binding** can happen at:

- **Compile time** — absolute addresses (must know load address).
- **Load time** — relocatable addresses fixed when the program is loaded.
- **Execution time** — addresses computed at runtime by the MMU (supports swapping and paging).

---

## 6.2 Memory partitioning

### 6.2.1 Fixed partitioning

Memory is divided into fixed-size partitions.

- **Problem:** **internal fragmentation** — a process smaller than its partition wastes the remainder.

### 6.2.2 Variable partitioning

Partitions are sized to fit processes exactly.

- **Problem:** **external fragmentation** — free memory is scattered in small holes.
- **Fixes:** compaction (expensive), or allocation strategies (first-fit, best-fit, worst-fit).

### 6.2.3 Buddy system

Memory is allocated in power-of-two blocks. A request for size $s$ is rounded up to the next power of two; if unavailable, a larger block is split in half repeatedly.

- **Pros:** fast allocation and coalescing.
- **Cons:** internal fragmentation (up to 50%).

---

## 6.3 Paging

**Paging** divides physical memory into fixed-size **frames** and logical memory into same-size **pages**. A process's pages can be placed in **non-contiguous** frames.

### 6.3.1 Address translation

A virtual address is split into:

- **Page number** $p$ — index into the **page table**.
- **Offset** $d$ — position within the page.

The page table entry gives the **frame number** $f$; the physical address is $f \cdot \text{page size} + d$.

### 6.3.2 The TLB

The **translation lookaside buffer (TLB)** is a small, fast associative cache of recent page-table entries. On a TLB hit, translation is fast; on a miss, the page table is consulted.

**Effective access time:**

$$
\text{EAT} = \alpha \cdot (t_{\text{TLB}} + t_{\text{mem}}) + (1-\alpha) \cdot (t_{\text{TLB}} + 2 t_{\text{mem}})
$$

where $\alpha$ is the hit ratio.

### 6.3.3 Multi-level page tables

For large address spaces, a single page table is huge. **Multi-level page tables** split it into a tree — only the levels that are actually used need to be allocated.

### 6.3.4 Paging eliminates external fragmentation

Frames are fixed size, so any free frame can hold any page — no scattered holes. **Internal fragmentation** remains (the last page of a process may be partially used), but is bounded by the page size.

---

## 6.4 Segmentation

**Segmentation** divides a process into **logical segments** (code, data, stack, heap), each with a base and limit register.

**Address translation:** a virtual address is a pair (segment number, offset). The segment table maps segment numbers to base addresses.

**Advantages:**

- Reflects the programmer's view of memory.
- Natural protection (code segment read-only, stack writable).
- Sharing (two processes can map the same code segment).

**Disadvantages:**

- **External fragmentation** — segments are variable-sized.

**Segmentation with paging** combines both: each segment is paged, eliminating external fragmentation while preserving logical structure.

---

## Worked example 6.1 — page-replacement algorithms

**Reference string:** `7, 0, 1, 2, 0, 3, 0, 4, 2, 3, 0, 3, 2` with **3 frames**.

**FIFO.**

| Ref | Frames | Fault? |
|:---:|---|:---:|
| 7 | 7, –, – | ✓ |
| 0 | 7, 0, – | ✓ |
| 1 | 7, 0, 1 | ✓ |
| 2 | 2, 0, 1 | ✓ |
| 0 | 2, 0, 1 | – |
| 3 | 2, 0, 3 | ✓ |
| 0 | 2, 0, 3 | – |
| 4 | 2, 4, 3 | ✓ |
| 2 | 2, 4, 3 | – |
| 3 | 2, 4, 3 | – |
| 0 | 0, 4, 3 | ✓ |
| 3 | 0, 4, 3 | – |
| 2 | 0, 2, 3 | ✓ |

**FIFO faults: 9.**

**LRU.**

| Ref | Frames | Fault? |
|:---:|---|:---:|
| 7 | 7, –, – | ✓ |
| 0 | 7, 0, – | ✓ |
| 1 | 7, 0, 1 | ✓ |
| 2 | 2, 0, 1 | ✓ |
| 0 | 2, 0, 1 | – |
| 3 | 2, 0, 3 | ✓ |
| 0 | 2, 0, 3 | – |
| 4 | 2, 4, 3 | ✓ |
| 2 | 2, 4, 3 | – |
| 3 | 2, 4, 3 | – |
| 0 | 0, 4, 3 | ✓ |
| 3 | 0, 4, 3 | – |
| 2 | 0, 2, 3 | ✓ |

**LRU faults: 9.**

**Optimal.**

| Ref | Frames | Fault? |
|:---:|---|:---:|
| 7 | 7, –, – | ✓ |
| 0 | 7, 0, – | ✓ |
| 1 | 7, 0, 1 | ✓ |
| 2 | 2, 0, 1 | ✓ |
| 0 | 2, 0, 1 | – |
| 3 | 2, 0, 3 | ✓ |
| 0 | 2, 0, 3 | – |
| 4 | 2, 4, 3 | ✓ |
| 2 | 2, 4, 3 | – |
| 3 | 2, 4, 3 | – |
| 0 | 0, 4, 3 | ✓ |
| 3 | 0, 4, 3 | – |
| 2 | 0, 2, 3 | ✓ |

**Optimal faults: 7** (the theoretical minimum).

---

## 6.5 Quiz

1. Distinguish internal and external fragmentation.
2. How does paging eliminate external fragmentation?
3. What is the purpose of the TLB?
4. Compare paging and segmentation.
5. For a reference string of your choice, trace FIFO, LRU, and Optimal.

---

## Answer key

??? success "Click to reveal"

    1. Internal: wasted space inside an allocated block. External: free memory scattered in small, non-contiguous holes.
    2. Frames are fixed size, so any page fits any free frame — no scattered holes.
    3. To cache recent virtual-to-physical translations, avoiding a page-table access on most references.
    4. Paging is fixed-size (internal fragmentation, no external); segmentation is variable-size (external fragmentation, no internal); segmentation reflects logical structure, paging does not.
    5. (See Example 6.1.)

---

## Further reading

- Tanenbaum §3.1–§3.4.
- Silberschatz §8.1–§9.5.