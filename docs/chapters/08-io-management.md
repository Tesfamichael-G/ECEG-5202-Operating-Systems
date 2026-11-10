# Chapter 8 — I/O Management

!!! abstract "Chapter at a glance"
    **Coverage:** I/O devices · principles of I/O software · I/O buffering · disk cache.

---

## Learning outcomes

1. Classify I/O devices (block, character, network).
2. Describe the layers of I/O software (interrupt handlers, device drivers, OS-independent software, user-space I/O).
3. Explain buffering and its role in I/O performance.
4. Describe disk scheduling (FCFS, SSTF, SCAN, C-SCAN) and disk caching.

---

## 8.1 I/O devices

| Class | Examples | Transfer unit | Access |
|---|---|---|---|
| **Block devices** | Disks, SSDs, USB drives | Block (e.g., 4 KB) | Random |
| **Character devices** | Keyboards, serial ports | Byte | Sequential |
| **Network devices** | Ethernet, Wi-Fi | Packet | Streaming |

**Device controllers** interface between the bus and the device; they expose registers for data, status, and control.

---

## 8.2 Principles of I/O software

Four layers:

1. **Interrupt handlers** — handle device interrupts.
2. **Device drivers** — device-specific code that talks to the controller.
3. **OS-independent I/O software** — buffering, caching, spooling, device-independent API.
4. **User-space I/O libraries** — `printf`, `fread`, etc.

**Goals of I/O software:**

- **Device independence** — programs can access any device without knowing its specifics.
- **Uniform naming** — devices are addressed as files (`/dev/...`).
- **Error handling** — errors are reported at the lowest layer that can handle them.
- **Buffering** — smooth out speed mismatches between producer and consumer.
- **Scheduling** — order I/O requests fairly and efficiently.

---

## 8.3 I/O buffering

**Buffering** stores data temporarily between a producer and a consumer. Three types:

- **Single buffering** — one buffer; the producer fills while the consumer drains.
- **Double buffering** — two buffers; the producer fills one while the consumer drains the other.
- **Circular buffering** — $n$ buffers in a ring for continuous streaming.

**Why buffering matters.**

- **Speed mismatch:** a fast CPU produces data faster than a slow disk can absorb it.
- **Transfer-size mismatch:** a network packet is small but the file system writes in blocks.
- **Copy semantics:** buffering gives a stable snapshot if the source changes during a read.

---

## 8.4 Disk scheduling

For a rotating disk, the **seek time** (moving the head) dominates. Scheduling algorithms reduce total head movement.

| Algorithm | Description | Starvation? |
|---|---|---|
| **FCFS** | Serve requests in arrival order | No |
| **SSTF** | Shortest seek time first | **Yes** |
| **SCAN** (elevator) | Move head in one direction, serving requests; reverse at the end | No |
| **C-SCAN** | Like SCAN, but always sweeps in one direction, wrapping to the start | No |

### Worked example 8.1 — disk scheduling

**Request queue:** `98, 183, 37, 122, 14, 124, 65, 67`. Head starts at **53**.

**FCFS:** $53 \to 98 \to 183 \to 37 \to 122 \to 14 \to 124 \to 65 \to 67$

Total movement: $45 + 85 + 146 + 85 + 108 + 110 + 59 + 2 = \mathbf{640}$ cylinders.

**SSTF:** $53 \to 65 \to 67 \to 37 \to 14 \to 98 \to 122 \to 124 \to 183$

Total movement: $12 + 2 + 30 + 23 + 84 + 24 + 2 + 59 = \mathbf{236}$ cylinders.

**SCAN (moving toward 0 first):** $53 \to 37 \to 14 \to 0 \to 65 \to 67 \to 98 \to 122 \to 124 \to 183$

Total movement: $16 + 23 + 14 + 65 + 2 + 31 + 24 + 2 + 59 = \mathbf{236}$ cylinders.

The exact totals depend on whether the endpoints (0 and 199) count as served positions.

---

## 8.5 Disk cache

A **disk cache** (or **buffer cache**) is a portion of main memory that holds recently read or written disk blocks.

- **Read cache:** on a hit, no disk I/O needed.
- **Write-back cache:** writes are buffered and flushed later; improves performance but risks data loss on a crash.
- **Write-through cache:** writes are immediately propagated to disk; slower but safer.

Modern systems use **unified page caches** that hold both file data and executable pages, sharing memory between the file system and virtual memory subsystems.

---

## 8.6 Quiz

1. Classify I/O devices into block, character, and network.
2. List the four layers of I/O software.
3. Explain the purpose of buffering with an example.
4. Compute the total head movement for FCFS, SSTF, and SCAN on a given queue.
5. Compare write-back and write-through caching.

---

## Answer key

??? success "Click to reveal"

    1. Block: disks; character: keyboard; network: Ethernet.
    2. Interrupt handlers, device drivers, OS-independent software, user-space libraries.
    3. Buffering decouples producer and consumer speeds. Example: a CPU writing to a slow disk fills a buffer; the disk drains it asynchronously.
    4. (See Example 8.1.)
    5. Write-back: faster but risks data loss on crash. Write-through: slower but safe. Compromise: journaling + periodic write-back.

---

## Further reading

- Tanenbaum §5.1–§5.4.
- Silberschatz §13.1–§13.5.