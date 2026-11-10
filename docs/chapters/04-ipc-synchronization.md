# Chapter 4 — Inter-Process Communication

!!! abstract "Chapter at a glance"
    **Coverage:** introduction · race condition · critical region · process synchronisation.

---

## Learning outcomes

1. Define a race condition and explain why it is dangerous.
2. State the three requirements of the critical-section problem.
3. Describe Peterson's solution and hardware synchronisation primitives.
4. Use semaphores and monitors to solve the producer–consumer, readers–writers, and dining-philosophers problems.

---

## 4.1 Race conditions

A **race condition** occurs when two or more processes access shared data concurrently and the outcome depends on the interleaving of their instructions.

**Classic example — `counter++`:**

```c
// Both processes execute:
counter++;
```

At the machine level:

```
load  R1, counter
add   R1, R1, 1
store counter, R1
```

If both processes interleave, one increment can be lost. The final value of `counter` depends on timing — a race.

---

## 4.2 The critical-section problem

The **critical section** is the code segment that accesses shared data.

!!! abstract "Three requirements (Silberschatz §6.2)"
    A correct solution to the critical-section problem must satisfy:

    1. **Mutual exclusion** — at most one process in its critical section at a time.
    2. **Progress** — if no process is in its critical section and some wish to enter, only those wishing to enter participate in the decision, and the decision cannot be postponed indefinitely.
    3. **Bounded waiting** — there is a bound on the number of times other processes can enter their critical sections after a process has requested entry and before it is granted.

### 4.2.1 Peterson's solution

For two processes $P_0$ and $P_1$, with shared variables `flag[2]` and `turn`:

```c
// Process Pi (i = 0 or 1), j = 1 - i
do {
    flag[i] = true;
    turn = j;
    while (flag[j] && turn == j) ;
    // critical section
    flag[i] = false;
    // remainder section
} while (true);
```

Peterson's solution satisfies all three requirements for two processes.

### 4.2.2 Hardware support

- **Test-and-set** — atomically sets a value and returns the old value.
- **Compare-and-swap** — atomically compares and swaps.
- **Load-link / store-conditional** — pairs of instructions for lock-free algorithms.

### 4.2.3 Semaphores

A **semaphore** $S$ is an integer variable accessed only through two atomic operations:

- **`wait(S)`** (or `P`): decrement $S$; if $S < 0$, block.
- **`signal(S)`** (or `V`): increment $S$; if $S \le 0$, wake a blocked process.

**Types:**

- **Binary semaphore** — values 0 or 1 (acts as a mutex).
- **Counting semaphore** — arbitrary non-negative values (controls access to $N$ resources).

### 4.2.4 Monitors

A **monitor** is a high-level synchronisation construct: a module with shared data and procedures, with the guarantee that only one process is active inside the monitor at a time. Condition variables (`wait` and `signal`) allow processes to suspend inside the monitor.

---

## Worked example 4.1 — producer–consumer with semaphores

**Setup.** A buffer of size $N$; producers insert items, consumers remove them.

**Semaphores:**

- `mutex = 1` — protects the buffer.
- `empty = N` — number of empty slots.
- `full = 0` — number of filled slots.

**Producer:**

```c
do {
    produce(item);
    wait(empty);
    wait(mutex);
    insert(item);
    signal(mutex);
    signal(full);
} while (true);
```

**Consumer:**

```c
do {
    wait(full);
    wait(mutex);
    item = remove();
    signal(mutex);
    signal(empty);
    consume(item);
} while (true);
```

**Why the order matters.** If the producer called `wait(mutex)` before `wait(empty)`, and the buffer were full, the producer would block inside the mutex, deadlocking the consumer (who needs the mutex to remove an item).

---

## Worked example 4.2 — dining philosophers

Five philosophers sit around a table with five forks. Each needs **two forks** (left and right) to eat.

**Naïve solution** — each picks up left fork, then right fork.

**Problem:** if all five pick up their left fork simultaneously, all wait for their right fork forever — **deadlock**.

**Solutions:**

1. **Asymmetric:** philosopher $i$ picks up left first if $i$ is even, right first if odd. Breaks the circular-wait condition.
2. **Limit concurrency:** allow at most 4 philosophers at the table at once (a counting semaphore initialised to 4).
3. **Atomic pickup:** acquire both forks atomically inside a monitor — either both or neither.
4. **Odd-even fork hierarchy:** number the forks; each philosopher picks up the lower-numbered fork first.

The **monitor solution** is the cleanest: each philosopher calls `pickup(i)` which checks that both neighbours' states allow eating, otherwise waits on a condition variable.

---

## 4.3 Quiz

1. Define a race condition. Give an example.
2. State the three requirements of the critical-section problem.
3. Explain Peterson's solution in words.
4. What is the difference between a binary and a counting semaphore?
5. Explain the dining-philosophers problem and give one deadlock-free solution.

---

## Answer key

??? success "Click to reveal"

    1. A race occurs when concurrent access to shared data produces an outcome that depends on interleaving. Example: two processes incrementing a shared counter without synchronisation.
    2. Mutual exclusion, progress, bounded waiting.
    3. Peterson's solution uses two flags (one per process) and a `turn` variable; a process sets its flag, gives the turn to the other, then waits while the other's flag is set and it is the other's turn.
    4. Binary: values 0/1, acts as a mutex. Counting: any non-negative value, controls access to $N$ identical resources.
    5. Five philosophers, five forks; each needs two forks. Solution: asymmetric fork ordering, or limit concurrency to 4, or acquire both forks atomically.

---

## Further reading

- Tanenbaum §2.3.
- Silberschatz §6.1–§6.7.