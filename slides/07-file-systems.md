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
  js: assets/lecture-07.js

---

# ECEG-5202

## Chapter 7 — File Systems

---

## Learning outcomes

<div class="fragment">

1. Describe a **file** — attributes, operations, access methods.

</div>

<div class="fragment">

2. Compare **directory structures**: linear, hash, tree, DAG.

</div>

<div class="fragment">

3. Compare **file allocation methods**: contiguous, linked, indexed.

</div>

<div class="fragment">

4. Trace a lookup through a **Unix inode**, including indirect blocks.

</div>

<div class="fragment">

5. Explain **free-space management** (bitmap, linked list, grouping).

</div>

<div class="fragment">

6. Describe **journaling** and why it prevents file-system corruption.

</div>

---

## 7.1 — What is a File?

???+ note "Definition (Silberschatz §10.1)"
    A **file** is a named, persistent collection of related information stored on secondary storage.

<div class="fragment">

???+ note "File attributes"
    | Attribute | Meaning |
    |---|---|
    | **Name** | Human-readable identifier |
    | **Identifier** | Unique internal tag (usually an inode number) |
    | **Type** | Regular, directory, symbolic link, device |
    | **Location** | Pointer to disk blocks |
    | **Size** | Current length |
    | **Protection** | Read/write/execute permissions |
    | **Timestamps** | Created, modified, accessed |
    | **Owner** | User, group |

</div>

<div class="fragment">

???+ note "File operations"
    `create`, `open`, `read`, `write`, `seek`, `close`, `delete`, `truncate`, `append`.

</div>

---

## 7.2 — Access Methods

???+ note "Sequential access"
    Read/write bytes in order. The next byte is always after the previous one.
    
    **Best for:** logs, streaming media, tape.

<div class="fragment">

???+ note "Direct (random) access"
    Read/write at any position by specifying the offset.

<div class="fragment">

```
read(file, offset=1024, buffer, size=100)
```

    **Best for:** databases, executable files, anything requiring seek.

</div>

<div class="fragment">

???+ note "Indexed access"
    An **index** maps keys to byte offsets. Allows lookup by key.

</div>

<div class="fragment">

!!! tip "Modern OSes support all three"
    POSIX `read()`/`write()` operate on the file's current position. `lseek()` changes the position, enabling direct access. `open()` returns a file descriptor with a **current offset** — a per-fd cursor.

</div>

---

## 7.3 — Directory Structures

???+ note "What is a directory?"
    A directory is a **special file** that maps names to inodes. The kernel uses it to resolve pathnames.

<div class="fragment">

???+ note "Four structures"
    | Structure | Lookup | Example |
    |---|---|---|
    | **Linear list** | $O(n)$ scan | Old FAT |
    | **Hash table** | $O(1)$ average | ext4 (directory hash) |
    | **Tree (B-tree)** | $O(\log n)$ | NTFS, XFS |
    | **DAG** | $O(1)$ per link | Unix hard links |

</div>

<div class="fragment">

???+ note "Unix hard links and symbolic links"
    - **Hard link** — another name for the same inode. Two directory entries point to the same inode. Deleting one does not delete the file.
    - **Symbolic link (soft link)** — a file whose content is the path of another file. Deleting the target breaks the link.

</div>

<div class="fragment">

!!! warning "Hard links and directories"
    Modern Unix restricts hard links to files (not directories) — allowing them would create cycles in the directory tree, requiring garbage collection.

</div>

---

## 7.4 — Allocation Methods: The Core Trade-off

???+ note "The question"
    A file needs $N$ blocks. Which disk blocks does it get, and how does the file system find them?

<div class="fragment">

???+ note "Three methods"
    | Method | Sequential access | Random access | Space overhead |
    |---|---|---|---|
    | **Contiguous** | Excellent | Excellent | None |
    | **Linked** | Good | Poor | One pointer per block |
    | **Indexed** | Good | Excellent | One index block |

</div>

<div class="fragment">

!!! tip "The trade-off triangle"
    - **Performance** (contiguous wins).
    - **Fragmentation resistance** (linked wins).
    - **Direct access** (contiguous and indexed win).

</div>

---

## 7.5 — Contiguous Allocation

???+ note "The idea"
    Each file occupies a **contiguous block** of disk.

```
File A: blocks 10-19
File B: blocks 20-29
File C: blocks 30-32
```

<div class="fragment">

???+ note "Pros"
    - **Fast sequential access** — one seek to the start, then linear.
    - **Fast direct access** — one seek to `start + offset`.
    - **No per-block overhead** — just start and length.

</div>

<div class="fragment">

???+ note "Cons"
    - **External fragmentation** — as files grow and shrink, free space splits into holes.
    - **Growth is painful** — a growing file may need to be **moved** to a new contiguous region.
    - **Compaction** required to reclaim large free regions.

</div>

<div class="fragment">

!!! warning "Contiguous allocation is rare today"
    Only used in read-only file systems (CD-ROM, ISO 9660) and in specific modern uses (extent-based allocation in ext4, which is contiguous but with extents). The growth problem killed pure contiguous allocation for general-purpose FS.

</div>

---

## 7.6 — Linked Allocation

???+ note "The idea"
    Each file is a **linked list of blocks**. The first block is a pointer; each block contains a pointer to the next.

```
Block 10: [data | ptr to 20]
Block 20: [data | ptr to 35]
Block 35: [data | ptr to 42]
Block 42: [data | NULL]
```

<div class="fragment">

???+ note "Pros"
    - **No external fragmentation** — any free block can join any list.
    - **File can grow** — append a block, update the tail's pointer.

</div>

<div class="fragment">

???+ note "Cons"
    - **Direct access is slow** — to reach byte 1,000,000, walk the chain.
    - **Pointer overhead** — 4 bytes per block (or more).
    - **Fragile** — a corrupted pointer loses the rest of the file.

</div>

<div class="fragment">

???+ tip "FAT (File Allocation Table)"
    DOS's solution: store **all** pointers in a separate **File Allocation Table** at the start of the disk. Each FAT entry is one disk block. Direct access still requires walking the FAT, but the FAT is cached in memory.

</div>

---

## 7.7 — Indexed Allocation

???+ note "The idea"
    Each file has an **index block** — an array of pointers to the file's data blocks.

```
Inode:
  index_block → [data_block_1, data_block_2, ..., data_block_N]
```

<div class="fragment">

???+ note "Pros"
    - **Direct access is fast** — index gives O(1) access to any block.
    - **No external fragmentation** — any free block can be a data block.

</div>

<div class="fragment">

???+ note "Cons"
    - **Index block overhead** — for small files, an entire block (4 KB) is reserved but mostly unused.
    - **Multi-level index required for large files** — the Unix inode uses single, double, and triple indirect.

</div>

<div class="fragment">

!!! tip "Modern file systems use hybrid schemes"
    ext4 uses **extents** — a single entry describes a contiguous range of blocks. This gives the speed of contiguous allocation with the flexibility of indexing.

</div>

---

## 7.8 — The Unix Inode

???+ note "The inode structure"
    Each file has an **inode** with metadata and 15 pointers:

```
Inode:
  mode, owner, size, timestamps, ...
  direct[0..11]          12 direct pointers
  single_indirect[1]     1 pointer to a block of 1024 pointers
  double_indirect[1]     1 pointer to a block of 1024 single-indirect blocks
  triple_indirect[1]     1 pointer to a block of 1024 double-indirect blocks
```

<div class="fragment">

???+ note "File size limits (4 KB blocks, 4 B pointers)"
    | Level | Coverage |
    |---|---|
    | Direct (12) | $12 \times 4\,\text{KB} = 48\,\text{KB}$ |
    | Single indirect | $1024 \times 4\,\text{KB} = 4\,\text{MB}$ |
    | Double indirect | $1024^2 \times 4\,\text{KB} = 4\,\text{GB}$ |
    | Triple indirect | $1024^3 \times 4\,\text{KB} = 4\,\text{TB}$ |

</div>

<div class="fragment">

???+ success "The genius of the inode"
    - **Small files** — accessed via direct pointers, no extra blocks touched.
    - **Large files** — accessed via indirect pointers, scaling to 4 TB.
    - **Flexible** — the same inode structure handles files from 1 byte to 4 TB.

</div>

---

## Worked example 7.1 — Trace a file offset in the inode

!!! tip "Problem:"
    A file is 100 KB. Block size = 4 KB. Where do the following offsets live?

<div class="fragment">

???+ note "**Offset 0 (first byte).**"
    Block 0. Direct pointer 0.

</div>

<div class="fragment">

??? info "**Offset 24,000 (≈ 5.9 blocks).**"
    Block 5. Direct pointer 5.

</div>

<div class="fragment">

??? info "**Offset 100,000 (≈ 24.4 blocks).**"
    Block 24. That's > 12 (direct limit) and < 1024 (single indirect limit). Lookup:
    - Read `single_indirect[0]` → indirect block.
    - Read indirect[24 − 12] = indirect[12] → data block 24.

</div>

<div class="fragment">

??? info "**File size 10 MB.**"
    Block 2500 — within single indirect (up to 4 MB) and double indirect (up to 4 GB). Lookup:
    - Read `single_indirect[0]` → indirect block.
    - Then `double_indirect[0]` → second indirect block.
    - Then `[2500 − 1024 − 12]` → data block.

</div>

<div class="fragment">

??? success "**Trade-off.** The farther into the file, the more indirect blocks the OS must read. Small files are fast; large files are still OK, just with a few extra indirections."

</div>

---

## 7.9 — Free Space Management

???+ note "The question"
    The OS must know which blocks are free. Which data structure tracks this?

<div class="fragment">

???+ note "Three approaches"
    | Method | Structure | Trade-off |
    |---|---|---|
    | **Bitmap** | One bit per block | Fast lookup, large space |
    | **Linked list** | Linked list of free blocks | Small space, slow scan |
    | **Grouping** | Block of block addresses | Between the two |

</div>

<div class="fragment">

???+ note "Bitmap"
    ```
    1011000111000101 1101 ...
    ```
    
    - Bit = 1 → free.
    - Bit = 0 → allocated.
    
    For a 1 TB disk with 4 KB blocks ($2^{28}$ blocks): bitmap = $2^{28}$ bits = 32 MB. Small relative to disk size.

</div>

<div class="fragment">

???+ note "Linked list"
    ```
    free_list_head → block 5 → block 12 → block 19 → ... → NULL
    ```
    
    - **Space:** small (one pointer per free block).
    - **Lookup:** $O(\text{free})$ — cannot tell if a specific block is free without walking.

</div>

<div class="fragment">

!!! tip "Ext4 uses a hybrid"
    Ext4 uses a **bitmap** for the inode table and **extent-based free space management** for data blocks. Best of both worlds.

</div>

---

## 7.10 — Journaling

???+ note "The problem"
    A power failure during a write can leave the file system **inconsistent** — a pointer updated but the data not written, or a directory entry pointing to a freed block.

<div class="fragment">

???+ note "The journaling solution"
    Before making any change, **write it to a journal** first. Once the journal entry is on disk, the change is safe — it can be replayed after a crash.

</div>

<div class="fragment">

???+ note "The three-step protocol"
    1. **Journal write** — write the intended change to the journal area.
    2. **Commit** — write a commit record to the journal.
    3. **Checkpoint** — apply the change to the actual file-system data.

    If a crash occurs before step 2, the journal is ignored. If after step 2, the journal is replayed on reboot.

</div>

<div class="fragment">

???+ info "Journaling modes"
    | Mode | What is journaled | Speed | Consistency |
    |---|---|---|---|
    | **Journal** | Data + metadata | Slowest | Strongest |
    | **Ordered** (default ext4) | Metadata (data written first) | Medium | Good |
    | **Writeback** | Metadata only | Fastest | Weakest |

</div>

<div class="fragment">

!!! tip "Journaling ≠ backup"
    Journaling ensures **consistency** — the file system is always in a valid state after a crash. It does not protect against accidental deletion, corruption, or ransomware. Backup is a separate concern.

</div>

---

## Worked example 7.2 — Journal replay

!!! tip "Problem:"
    A crash occurs after the journal has a "commit" record for a new file, but the directory entry has not yet been written. What happens on reboot?

<div class="fragment">

???+ note "**Step 1 — scan the journal.**"
    The file system finds the journal entry with a valid commit record.

</div>

<div class="fragment">

??? info "**Step 2 — replay.**"
    The journal entry describes the intended operation. The file system re-applies it — creating the directory entry that wasn't written before the crash.

</div>

<div class="fragment">

??? success "**Result.**"
    The file exists. The file system is consistent. If the journal entry had **no** commit record, it would be ignored (the file would not exist — as if the crash happened before the create was requested).

</div>

<div class="fragment">

!!! tip "The order matters"
    Data must be written to disk **before** the journal is committed (in ordered mode). Otherwise, after replay, the journal points to a data block that was never written — corrupt.

</div>

---

## 7.11 — File System Performance

???+ note "The buffer cache"
    The OS caches recently accessed disk blocks in memory. A read that hits the cache avoids a disk access — 100,000× faster.

<div class="fragment">

???+ note "Performance optimizations"
    | Technique | Benefit |
    |---|---|
    | **Buffer cache** | Avoids disk reads for hot data |
    | **Read-ahead** | Prefetches sequential blocks |
    | **Journaling** | Fast recovery after crash |
    | **Extents** (ext4, XFS) | Contiguous allocation for large files |
    | **Delayed allocation** | Batches small writes into large ones |

</div>

<div class="fragment">

???+ note "The writeback problem"
    Write-back caching improves performance but delays durability. `fsync(fd)` forces a write to disk — essential for databases and journals.

</div>

<div class="fragment">

!!! tip "The small-file problem"
    Many small files (e.g., a git repo) exercise the inode allocator and directory structure heavily. Filesystems like **XFS** and **btrfs** optimize for this; ext4 is decent but not ideal.

</div>

---

## 7.12 — Modern File Systems

???+ note "A brief comparison"
    | FS | Journaled | CoW | Compression | Typical use |
    |---|---|---|---|---|
    | **ext4** | Yes | No | No | Linux default |
    | **XFS** | Yes | No | No | Large files, servers |
    | **btrfs** | CoW | Yes | Yes | Snapshots, containers |
    | **ZFS** | CoW | Yes | Yes | Storage servers, NAS |
    | **NTFS** | Yes | No | Yes | Windows |
    | **APFS** | CoW | Yes | Yes | macOS, iOS |

<div class="fragment">

!!! tip "Copy-on-write (CoW) file systems"
    Btrfs and ZFS never overwrite data — they write new blocks and update pointers. This gives:
    - **Snapshots** for free — just record the current root pointer.
    - **Cheap clones** — share blocks until modified.
    - **Atomic updates** — no half-written state.

</div>

---

## Chapter 7 — key takeaways

<div class="fragment">

**1.** A **file** is a named, persistent collection of bytes. Attributes include name, size, timestamps, and protection.

</div>

<div class="fragment">

**2.** **Access methods**: sequential, direct, indexed. Modern OSes support all three via `lseek`.

</div>

<div class="fragment">

**3.** **Directory structures**: linear (slow), hash (fast), tree (scales), DAG (Unix links).

</div>

<div class="fragment">

**4.** **Allocation methods**: contiguous (fast, fragmentation), linked (flexible, slow), indexed (good compromise, inode).

</div>

<div class="fragment">

**5.** The **Unix inode** combines direct and indirect pointers to handle files from bytes to terabytes.

</div>

<div class="fragment">

**6.** **Free-space management**: bitmap (fast, large), linked list (small, slow).

</div>

<div class="fragment">

**7.** **Journaling** ensures consistency after crash — write intent to journal, then apply.

</div>

<div class="fragment">

**8.** Modern file systems (btrfs, ZFS) use **copy-on-write** for snapshots and atomicity.

</div>

---

## Next — Chapter 8

**I/O Management.**

<div class="fragment">

- **I/O hardware**: controllers, buses, device registers.
- **Four I/O software layers**.
- **Programmed I/O**, **interrupt-driven**, **DMA**.
- **Disk scheduling**: FCFS, SSTF, SCAN, C-SCAN, LOOK.
- **Disk caching** and **buffering**.

</div>

<div class="fragment">

Chapter 7 managed **persistent** storage. Chapter 8 manages the **mechanisms** that move data between devices and memory.

</div>