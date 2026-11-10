# Resources

---

## Core textbooks

| Book | Chapters used |
|---|---|
| **Tanenbaum & Bos**, *Modern Operating Systems*, 4th ed., Pearson, 2015 | 1–6 (primary reference) |
| **Silberschatz, Galvin & Gagne**, *Operating System Concepts*, 9th ed., Wiley, 2013 | 1–12 (secondary reference) |
| **Stallings**, *Operating Systems: Internals and Design Principles*, 7th ed., Prentice Hall, 2012 | 1–14 (advanced reference) |

---

## Free online resources 🆓

- **OSTEP** — *Operating Systems: Three Easy Pieces* by Remzi & Andrea Arpaci-Dusseau — [ostep.org](https://pages.cs.wisc.edu/~remzi/OSTEP/) — free, excellent textbook with rigorous explanations of processes, scheduling, memory, and file systems.
- **MIT 6.828** — *Operating System Engineering* — [pdos.csail.mit.edu/6.828/](https://pdos.csail.mit.edu/6.828/) — build a real OS kernel (xv6) — the gold-standard lab course.
- **xv6** — a simple Unix-like teaching OS — [github.com/mit-pdos/xv6-public](https://github.com/mit-pdos/xv6-public).
- **Linux kernel documentation** — [kernel.org/doc/](https://www.kernel.org/doc/).
- **OSDev Wiki** — [wiki.osdev.org](https://wiki.osdev.org/) — community reference for OS development.

---

## Software

| Tool | Use |
|---|---|
| **Linux (Ubuntu / Debian)** | Primary lab environment — shell, processes, IPC |
| **`strace`, `ltrace`** | Trace system calls |
| **`top`, `htop`, `ps`** | Process monitoring |
| **`gdb`** | Debugging |
| **`perf`** | Performance analysis |
| **`valgrind`** | Memory debugging |
| **`pthreads`** | Thread programming |
| **`sysvinit`, `systemd`** | Init systems for studying process trees |

---

## Recommended first-week reading

- **Tanenbaum §1.1** — What is an operating system?
- **Silberschatz §1.1–§1.3** — OS functions and structures.
- **OSTEP Chapter 2** — Introduction to operating systems.