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
  js: assets/lecture-04.js

---

# ECEG-5202

## Chapter 4 — Inter-Process Communication and Synchronization

---

## Learning outcomes

<div class="fragment">

1. Define a **race condition** and explain why it is dangerous.

</div>

<div class="fragment">

2. State the **three requirements** of the critical-section problem.

</div>

<div class="fragment">

3. Explain **Peterson's solution** and hardware primitives.

</div>

<div class="fragment">

4. Use **semaphores** and **monitors** to solve the producer–consumer problem.

</div>

<div class="fragment">

5. Solve the **readers–writers** and **dining philosophers** problems without deadlock.

</div>

---

## 4.1 — Why Synchronization?

???+ note "The fundamental problem"
    Processes (and threads) **share memory**. When two of them read and write the same location concurrently, the final value can depend on the exact order of their instructions.

<div class="fragment">

???+ note "Definition — Race condition"
    A **race condition** occurs when the outcome of a computation depends on the **relative timing** of two or more concurrent operations on shared data.

</div>

<div class="fragment">

!!! warning "Race conditions are common"
    Any shared **counter**, **list**, **queue**, or **cache** is a candidate. Most real-world data corruption bugs — in databases, OS kernels, and networked applications — are race conditions.

</div>

---

## 4.2 — The Counter Example

???+ note "Two threads incrementing a counter"
    ```c
    // shared: int counter = 0
    void* worker(void* arg) {
        for (int i = 0; i < N; i++)
            counter++;
        return NULL;
    }
    ```

<div class="fragment">

???+ info "The machine code"
    ```
    load  R1, counter
    add   R1, R1, 1
    store R1, counter
    ```

</div>

<div class="fragment">

???+ note "The interleaving"
    | Time | Thread A | Thread B | Memory |
    |---|---|---|---|
    | 1 | load R1, 0 | | 0 |
    | 2 | add R1, 1 | | 0 |
    | 3 | | load R2, 0 | 0 |
    | 4 | | add R2, 1 | 0 |
    | 5 | store R1, 1 | | 1 |
    | 6 | | store R2, 1 | 1 |

    **Both threads incremented the counter. Final value = 1.** One increment is **lost**.

</div>

<div class="fragment">

???+ success "The consequence"
    If both threads each increment 1,000,000 times, the final counter may be **anywhere from 1,000,000 to 2,000,000** — depending on the timing.

</div>

---

## 4.3 — The Critical Section Problem

???+ note "Definition"
    The **critical section** is the code segment that accesses shared data. The **critical-section problem** is to design a protocol so that:

<div class="fragment">

???+ abstract "Three requirements (Silberschatz §6.2)"
    **1. Mutual exclusion.** At most one process may be in its critical section at a time.

    **2. Progress.** If no process is in the critical section, and some are waiting to enter, only those wishing to enter participate in the decision — and the decision cannot be postponed indefinitely.

    **3. Bounded waiting.** There is a bound on the number of times other processes can enter their critical sections after a process has requested entry and before it is granted.

</div>

<div class="fragment">

???+ note "The skeleton"
    ```c
    do {
        // ENTRY section — request permission
        // CRITICAL section — access shared data
        // EXIT section — release permission
        // REMAINDER section — non-critical code
    } while (true);
    ```

</div>

<div class="fragment">

!!! warning "All three are required"
    Mutual exclusion alone is not enough — a process could starve. Bounded waiting alone is not enough — two processes could enter simultaneously. All three must hold.

</div>

---

## 4.4 — Peterson's Solution

???+ note "The setup (two processes, $P_0$ and $P_1$)"
    Two shared variables:
    - `int turn` — whose turn it is.
    - `bool flag[2]` — which process wants to enter.

<div class="fragment">

???+ note "The algorithm"
    ```c
    // Process Pi (i = 0 or 1), j = 1 - i
    do {
        flag[i] = true;           // I want to enter
        turn = j;                 // but give the turn to the other
        while (flag[j] && turn == j) ; // wait while other wants in AND has the turn
        // CRITICAL SECTION
        flag[i] = false;          // I'm done
        // REMAINDER SECTION
    } while (true);
    ```

</div>

<div class="fragment">

???+ info "Why it works"
    - **Mutual exclusion.** If both are in the critical section, both `flag` are true. `turn` has one value — say 0. Then $P_1$ must have exited the loop (because $turn = 1$ was true for $P_1$, but `turn` is now 0). Contradiction.
    - **Progress.** If $P_0$ wants to enter and $P_1$ does not, $flag[1]$ = false → $P_0$ enters immediately.
    - **Bounded waiting.** After $P_1$ exits, it sets `flag[1] = false`; $P_0$ then enters. $P_0$ waits at most one entry by $P_1$.

</div>

<div class="fragment">

!!! warning "Not practical on modern CPUs"
    Peterson's solution assumes **sequential consistency** of memory. Modern CPUs and compilers **reorder** operations. Without memory barriers, it fails. It is pedagogically valuable but not used in production.

</div>

---

## 4.5 — Hardware Primitives

???+ note "The problem with software-only solutions"
    They rely on the CPU's memory model. **Atomic hardware instructions** eliminate that assumption.

<div class="fragment">

???+ note "Test-and-set"
    ```
    bool test_and_set(bool* target):
        bool rv = *target
        *target = true
        return rv
    ```
    
    **Atomically:** reads the old value and sets to true, in a single instruction.

</div>

<div class="fragment">

???+ note "Compare-and-swap (CAS)"
    ```
    bool compare_and_swap(int* ptr, int expected, int new):
        if *ptr == expected:
            *ptr = new
            return true
        return false
    ```

</div>

<div class="fragment">

???+ note "Load-linked / store-conditional (LL/SC)"
    - **LL** — loads a value and marks the memory address.
    - **SC** — stores a value only if the address has not changed since LL.
    
    More expressive than CAS — allows lock-free algorithms on arbitrary words.

</div>

---

## 4.6 — Semaphores

???+ note "Definition (Dijkstra 1965)"
    A **semaphore** $S$ is an integer variable accessed only through two atomic operations:

<div class="fragment">

???+ note "The two operations"
    ```
    wait(S):      // also called P() or down()
        S = S - 1
        if S < 0: block
    
    signal(S):    // also called V() or up()
        S = S + 1
        if S <= 0: wake up one blocked process
    ```

</div>

<div class="fragment">

???+ note "Two types"
    | Type | Values | Use |
    |---|---|---|
    | **Binary semaphore** | 0 or 1 | Acts as a mutex |
    | **Counting semaphore** | Any non-negative | Controls access to $N$ identical resources |

</div>

<div class="fragment">

!!! warning "Atomicity"
    `wait()` and `signal()` are **atomic** — the kernel (or hardware) guarantees no interleaving. Without atomicity, the semaphore itself has a race condition.

</div>

---

## Worked example 4.1 — Mutex via binary semaphore

!!! tip "Problem:"
    Two threads increment a shared counter 1,000,000 times each. Use a semaphore to make it correct.

<div class="fragment">

???+ note "**Step 1 — the setup.**"
    ```c
    sem_t mutex;
    sem_init(&mutex, 0, 1);       // initial value 1
    int counter = 0;
    ```

</div>

<div class="fragment">

??? info "**Step 2 — the critical section.**"
    ```c
    void* worker(void* arg) {
        for (int i = 0; i < N; i++) {
            sem_wait(&mutex);
            counter++;
            sem_post(&mutex);
        }
        return NULL;
    }
    ```

</div>

<div class="fragment">

??? info "**Step 3 — trace.**"
    - Thread A calls `sem_wait`: S = 1 → S = 0, proceeds.
    - Thread B calls `sem_wait`: S = 0 → S = -1, blocks.
    - Thread A increments counter, calls `sem_post`: S = -1 → S = 0, wakes B.
    - Thread B proceeds.
    
    **Only one thread in the critical section at a time.** ✓

</div>

<div class="fragment">

??? success "**Result.** Final counter = 2,000,000."

</div>

---

## 4.7 — The Producer-Consumer Problem

???+ note "The setup"
    - A **bounded buffer** of size $N$.
    - **Producers** insert items into the buffer.
    - **Consumers** remove items from the buffer.
    - Constraint: producer must wait if buffer is full; consumer must wait if buffer is empty.

<div class="fragment">

???+ note "Three semaphores (Silberschatz §6.6)"
    | Semaphore | Initial | Meaning |
    |---|---|---|
    | **mutex** | 1 | Protects the buffer |
    | **empty** | $N$ | Counts empty slots |
    | **full** | 0 | Counts filled slots |

</div>

<div class="fragment">

???+ note "The producer"
    ```c
    do {
        item = produce();
        wait(empty);          // wait for an empty slot
        wait(mutex);          // enter critical section
        buffer[in] = item;
        in = (in + 1) % N;
        signal(mutex);        // exit critical section
        signal(full);         // signal a filled slot
    } while (true);
    ```

</div>

<div class="fragment">

???+ note "The consumer"
    ```c
    do {
        wait(full);           // wait for a filled slot
        wait(mutex);          // enter critical section
        item = buffer[out];
        out = (out + 1) % N;
        signal(mutex);        // exit critical section
        signal(empty);        // signal an empty slot
        consume(item);
    } while (true);
    ```

</div>

<div class="fragment">

!!! warning "The order of wait() calls matters"
    Producer does `wait(empty)` **before** `wait(mutex)`. If reversed, a full buffer would deadlock — the producer would hold `mutex` while waiting for `empty`, and the consumer could never acquire `mutex` to free a slot.

</div>

---

## Worked example 4.2 — Trace producer-consumer

!!! tip "Problem:"
    $N = 3$. Trace the state when a producer inserts 4 items and a consumer removes 2.

<div class="fragment">

???+ note "**Initial:** `empty = 3`, `full = 0`, `mutex = 1`."

</div>

<div class="fragment">

??? info "**Producer inserts A.**"
    `wait(empty)` → empty = 2. `wait(mutex)` → mutex = 0. Insert A. `signal(mutex)` → mutex = 1. `signal(full)` → full = 1.

</div>

<div class="fragment">

??? info "**Producer inserts B, C.**"
    Same pattern. Now: empty = 0, full = 3, mutex = 1.

</div>

<div class="fragment">

??? info "**Producer tries D.**"
    `wait(empty)` → empty = -1, **blocks**. Producer is stuck until a consumer frees a slot.

</div>

<div class="fragment">

??? info "**Consumer removes A.**"
    `wait(full)` → full = 2. `wait(mutex)` → mutex = 0. Remove A. `signal(mutex)` → mutex = 1. `signal(empty)` → empty = 0, **wakes producer**.

</div>

<div class="fragment">

??? info "**Producer resumes.**"
    Producer unblocks from `wait(empty)`, proceeds with `wait(mutex)`, inserts D. `signal(full)` → full = 3. Done.

</div>

<div class="fragment">

??? success "**Final state.** empty = 0, full = 3, mutex = 1. Buffer contains B, C, D."

</div>

---

## 4.8 — Monitors

???+ note "Definition (Hoare 1974)"
    A **monitor** is a synchronization construct that bundles:
    - Shared data.
    - Procedures that operate on the data.
    - A **mutex** that ensures only one process is active in the monitor at a time.

<div class="fragment">

???+ note "The structure"
    ```java
    monitor Buffer {
        int[] data;
        int count;
        
        procedure insert(item):
            // only one thread in here at a time
            ...
        
        procedure remove():
            // only one thread in here at a time
            ...
    }
    ```

</div>

<div class="fragment">

???+ note "Condition variables"
    Monitors include **condition variables** for waiting:
    - `cond.wait()` — suspend the calling process; release the monitor.
    - `cond.signal()` — wake one waiting process.

</div>

<div class="fragment">

!!! tip "Why monitors are easier"
    Semaphores require you to **remember the order** of `wait`/`signal` calls — a wrong order causes deadlock or corruption. Monitors **enforce** mutual exclusion automatically — you just call procedures.

</div>

---

## Worked example 4.3 — Producer-consumer with monitors

!!! tip "Problem:"
    Same problem, using a monitor.

<div class="fragment">

???+ note "**The monitor.**"
    ```java
    monitor BoundedBuffer {
        int[] buffer = new int[N];
        int count = 0, in = 0, out = 0;
        condition notFull, notEmpty;
        
        procedure insert(item) {
            while (count == N) notFull.wait();
            buffer[in] = item;
            in = (in + 1) % N;
            count++;
            notEmpty.signal();
        }
        
        procedure remove() {
            while (count == 0) notEmpty.wait();
            int item = buffer[out];
            out = (out + 1) % N;
            count--;
            notFull.signal();
            return item;
        }
    }
    ```

</div>

<div class="fragment">

??? info "**Producer:** `BoundedBuffer.insert(item)`."

</div>

<div class="fragment">

??? info "**Consumer:** `BoundedBuffer.remove()`."

</div>

<div class="fragment">

??? success "**Why this is simpler.**"
    No explicit `mutex` — the monitor guarantees it. No risk of wrong `wait`/`signal` order — the compiler or runtime handles it.

</div>

---

## 4.9 — The Readers-Writers Problem

???+ note "The setup"
    - **Readers** access the shared resource without modifying it.
    - **Writers** modify it.
    - Multiple readers can access simultaneously.
    - A writer requires exclusive access.

<div class="fragment">

???+ note "Two variants"
    | Variant | Preference | Starvation risk |
    |---|---|---|
    | **First readers-writers** | Reader-priority | Writers may starve |
    | **Second readers-writers** | Writer-priority | Readers may starve |
    | **Fair** | FIFO | None |

</div>

<div class="fragment">

???+ note "Reader-priority solution (Silberschatz)"
    ```c
    sem_t rw_mutex;       // 1, shared between readers and writers
    sem_t mutex;          // 1, protects read_count
    int read_count = 0;
    
    // Reader
    wait(mutex);
    read_count++;
    if (read_count == 1) wait(rw_mutex);
    signal(mutex);
    // READ
    wait(mutex);
    read_count--;
    if (read_count == 0) signal(rw_mutex);
    signal(mutex);
    
    // Writer
    wait(rw_mutex);
    // WRITE
    signal(rw_mutex);
    ```

</div>

<div class="fragment">

!!! warning "Writers can starve"
    If readers keep arriving, `read_count` never reaches 0, and the writer never acquires `rw_mutex`. **Writer-priority** solves this by having arriving writers block new readers.

</div>

---

## 4.10 — The Dining Philosophers

???+ note "The setup"
    Five philosophers around a table. Five forks between them. Each philosopher needs **two forks** to eat.

```
        P0
       /  \
      F0   F4
     /      \
    P1      P4
    |        |
    F1      F3
     \      /
       P2-P3
```

<div class="fragment">

???+ note "The naïve solution"
    ```c
    // Philosopher i
    do {
        wait(fork[i]);              // pick up left
        wait(fork[(i+1) % 5]);      // pick up right
        eat();
        signal(fork[i]);
        signal(fork[(i+1) % 5]);
    } while (true);
    ```

</div>

<div class="fragment">

!!! warning "The deadlock"
    If all five pick up their left fork simultaneously, all five wait for their right fork forever. **Circular wait** — one of the four Coffman conditions (Chapter 5).

</div>

<div class="fragment">

???+ note "Solution 1 — asymmetric"
    Philosopher $i$ picks up the **left** fork first if $i$ is even, the **right** first if $i$ is odd. This breaks the circular-wait chain.

</div>

<div class="fragment">

???+ note "Solution 2 — limit concurrency"
    A semaphore `seats = 4`. Each philosopher does `wait(seats)` before picking forks. At most 4 philosophers compete → at least one pair of forks is free.

</div>

<div class="fragment">

???+ note "Solution 3 — atomic pickup"
    A monitor with `pickup(i)` and `putdown(i)`. `pickup` acquires both forks atomically or waits on a condition variable.

</div>

<div class="fragment">

???+ success "**Which is best?**"
    The **monitor solution** is cleanest. The **asymmetric** solution is clever but non-obvious. The **concurrency limit** is simple but reduces throughput.

</div>

---

## 4.11 — Deadlock vs Starvation

???+ note "Two failure modes"
    | Failure | Definition | Fix |
    |---|---|---|
    | **Deadlock** | Two or more processes wait for each other forever | Break one Coffman condition (Chapter 5) |
    | **Starvation** | A process waits indefinitely because others keep preempting | Aging (increase priority of waiters) |

<div class="fragment">

!!! tip "Deadlock implies starvation, but not vice versa"
    All deadlocked processes are starved — but starved processes are not deadlocked. The distinction matters: deadlock is a **global** condition; starvation is a **local** one.

</div>

---

## Chapter 4 — key takeaways

<div class="fragment">

**1.** **Race conditions** arise when concurrent access to shared data produces non-deterministic results. The counter example makes this concrete.

</div>

<div class="fragment">

**2.** The **critical-section problem** has three requirements: **mutual exclusion**, **progress**, **bounded waiting**.

</div>

<div class="fragment">

**3.** **Peterson's solution** is a two-process software mutex — elegant but relies on sequential consistency.

</div>

<div class="fragment">

**4.** **Hardware primitives** (test-and-set, CAS, LL/SC) provide atomicity at the instruction level.

</div>

<div class="fragment">

**5.** **Semaphores** are counting or binary; `wait`/`signal` must be called in the right order to avoid deadlock.

</div>

<div class="fragment">

**6.** **Monitors** bundle shared data with procedures and a mutex — easier to use than raw semaphores.

</div>

<div class="fragment">

**7.** **Producer-consumer**, **readers-writers**, **dining philosophers** are the canonical synchronization problems — each with known correct solutions.

</div>

---

## Next — Chapter 5

**Deadlocks.**

<div class="fragment">

- The **four Coffman conditions**.
- **Resource allocation graphs**.
- **Prevention** — break one condition.
- **Avoidance** — the Banker's algorithm.
- **Detection** — wait-for graph, detection algorithm.
- **Recovery** — kill or preempt.

</div>

<div class="fragment">

Chapter 4 showed how to synchronize with mutexes and semaphores — and that wrong ordering **deadlocks**. Chapter 5 formalises this.

</div>