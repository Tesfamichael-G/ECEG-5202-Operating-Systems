# Chapter 7 — File Systems

!!! abstract "Chapter at a glance"
    **Coverage:** introduction · file system structure · file system implementation · directory implementation · efficiency and performance.

---

## Learning outcomes

1. Describe the file abstraction: files, directories, and the file-system interface.
2. Compare file-allocation methods: contiguous, linked, indexed.
3. Describe Unix inodes and how they support small and large files.
4. Explain directory implementation (linear list, hash table, tree).
5. Discuss efficiency and performance trade-offs (caching, journaling, defragmentation).

---

## 7.1 Introduction

A **file system** provides persistent, named storage. It abstracts the physical disk into files and directories.

**File attributes:** name, identifier, type, location, size, protection, timestamps, owner.

**File operations:** create, open, read, write, seek, delete, truncate, close.

---

## 7.2 File system structure

A layered view:

```
Application
    ↓
Logical file system  (file names, directories, protection)
    ↓
File-organisation module  (blocks, allocation)
    ↓
Basic file system  (block I/O, buffering)
    ↓
I/O control  (device drivers, interrupts)
    ↓
Devices
```

---

## 7.3 File system implementation

### 7.3.1 Contiguous allocation

Each file occupies a contiguous block of disk.

- **Pros:** simple, fast sequential and random access.
- **Cons:** **external fragmentation**; files cannot grow easily.

### 7.3.2 Linked allocation

Each file is a linked list of blocks.

- **Pros:** no external fragmentation; files can grow.
- **Cons:** random access requires traversing the chain — $O(n)$; pointer overhead per block.

### 7.3.3 Indexed allocation

Each file has an **index block** containing pointers to its data blocks.

- **Pros:** fast random access — $O(1)$ to reach any block.
- **Cons:** index block overhead for small files.

### 7.3.4 Unix inode

The Unix inode combines all three approaches for efficiency:

| Field | Size |
|---|---|
| Mode, owner, size, timestamps | ~64 bytes |
| Direct pointers | 12 pointers |
| Single indirect | 1 pointer → block of pointers |
| Double indirect | 1 pointer → block of pointers to pointer blocks |
| Triple indirect | 1 pointer → double-indirect blocks |

With 4 KB blocks and 4-byte pointers (1024 pointers per block):

- **Direct:** $12 \times 4\text{ KB} = 48\text{ KB}$.
- **Single indirect:** $1024 \times 4\text{ KB} = 4\text{ MB}$.
- **Double indirect:** $1024^2 \times 4\text{ KB} = 4\text{ GB}$.
- **Triple indirect:** $1024^3 \times 4\text{ KB} = 4\text{ TB}$.

This design gives **fast access to small files** (direct pointers) and **support for huge files** (indirect pointers), without wasting an index block for every small file.

---

## 7.4 Directory implementation

| Method | Description | Lookup cost |
|---|---|---|
| **Linear list** | List of (name, inode) pairs | $O(n)$ |
| **Hash table** | Hash on filename | $O(1)$ average |
| **Tree (B-tree)** | Balanced tree of names | $O(\log n)$ |

Most modern file systems use a **B-tree** or **hash table** for fast lookup in large directories.

---

## 7.5 Efficiency and performance

- **Caching** — the **buffer cache** holds recently used disk blocks in memory. The **page cache** holds memory-mapped file pages.
- **Journaling** — before writing to the file system, log the intended changes. On crash, replay the journal to restore consistency. Used by ext3/ext4, NTFS, XFS.
- **Defragmentation** — periodically rearrange files to reduce fragmentation. Mostly unnecessary on SSDs.
- **Preallocation** — reserve space for a file to avoid fragmentation during growth.

---

## 7.6 Quiz

1. Compare contiguous, linked, and indexed allocation on random access and fragmentation.
2. Explain how the Unix inode supports both small and very large files.
3. Why does journaling improve reliability?
4. Compare linear, hash, and tree directory implementations.
5. What is the buffer cache and why is it important?

---

## Answer key

??? success "Click to reveal"

    1. Contiguous: O(1) random, external fragmentation. Linked: O(n) random, no external. Indexed: O(1) random, index-block overhead.
    2. Direct pointers cover files up to 48 KB; single, double, and triple indirect pointers extend coverage to 4 MB, 4 GB, and 4 TB respectively.
    3. It logs intended changes before applying them, so a crash can be recovered by replaying the log — avoiding inconsistent metadata.
    4. Linear: O(n); hash: O(1) average; tree: O(log n) — tree scales better for large directories.
    5. It holds recently used disk blocks in memory to avoid re-reading from disk.

---

## Further reading

- Tanenbaum §4.1–§4.3.
- Silberschatz §10.1–§11.5.