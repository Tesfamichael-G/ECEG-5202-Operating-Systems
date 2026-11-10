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
  js: assets/lecture-09.js

---

# ECEG-5202

## Chapter 9 — Introduction to Security

---

## Learning outcomes

<div class="fragment">

1. Classify **threats**: malware, buffer overflow, DoS, social engineering.

</div>

<div class="fragment">

2. Explain the **three factors** of authentication.

</div>

<div class="fragment">

3. Distinguish **authentication** from **authorisation**.

</div>

<div class="fragment">

4. Describe **access control models**: DAC, MAC, RBAC.

</div>

<div class="fragment">

5. Explain **modern defences**: ASLR, DEP, sandboxing, encryption.

</div>

---

## 9.1 — Why Security?

???+ note "The trust problem"
    Every system has a **trust boundary**. Inside the boundary, components trust each other. Across the boundary, they must not.

<div class="fragment">

???+ note "Threat model"
    A **threat model** identifies:
    - What the attacker wants (data, execution, denial).
    - What the attacker controls (input, network, local files).
    - What the attacker cannot do (cross the trust boundary).

</div>

<div class="fragment">

!!! tip "Defence in depth"
    No single defence is unbreakable. Real systems combine many defences — a **layered** approach. Breaking in requires defeating **all** layers, not just one.

</div>

---

## 9.2 — Malware: Three Fundamental Types

???+ note "Virus"
    A **virus** is malicious code attached to a **host program**. When the host runs, the virus runs.

<div class="fragment">

???+ info "Virus characteristics"
    - **Requires user action** — running an infected program.
    - **Self-replicates** into other files.
    - **Payloads** vary — data theft, destruction, ransomware.

</div>

<div class="fragment">

???+ note "Worm"
    A **worm** is a **standalone** malicious program that **self-propagates** over the network.

<div class="fragment">

???+ info "Worm characteristics"
    - **Does not need a host** — runs on its own.
    - **Self-propagates** via network exploits.
    - **Very fast** — no user action needed.

</div>

<div class="fragment">

???+ note "Trojan"
    A **trojan** is malicious code **disguised** as legitimate software.

<div class="fragment">

???+ info "Trojan characteristics"
    - **User installs it voluntarily** — thinking it's useful.
    - **Does not self-replicate** — relies on the user to spread it.
    - **Examples:** fake antivirus, pirated software, free games.

</div>

<div class="fragment">

!!! warning "Worms are the most dangerous"
    - **Virus:** requires user action → slow spread.
    - **Worm:** no user action → exponential spread (minutes).
    - **Trojan:** requires social engineering → targeted.
    
    The **Morris worm** (1988) infected 10% of the internet in 24 hours — this is why worms are so feared.

</div>

---

## 9.3 — Buffer Overflow

???+ note "The vulnerability"
    A **buffer overflow** occurs when a program writes more data into a buffer than it can hold, overwriting adjacent memory.

<div class="fragment">

???+ note "The classic stack overflow"
    ```c
    void vulnerable(char *input) {
        char buffer[64];
        strcpy(buffer, input);   // no bounds check!
    }
    ```

</div>

<div class="fragment">

???+ note "The stack layout"
    ```
    ┌──────────────────┐  ← higher addresses
    │ return address   │  ← overwritten by overflow!
    ├──────────────────┤
    │ saved EBP        │
    ├──────────────────┤
    │ buffer[0..63]    │
    └──────────────────┘  ← lower addresses
    ```
    
    If `input` is longer than 64 bytes, it overwrites the saved EBP and return address.

</div>

<div class="fragment">

???+ note "The attack"
    1. Attacker writes a long string: 64 bytes of filler + new return address.
    2. New return address points to shellcode (also in the buffer).
    3. When the function returns, the CPU jumps to the attacker's code.

</div>

<div class="fragment">

!!! warning "Buffer overflows have dominated exploits for 30+ years"
    Morris worm (1988), Code Red (2001), Slammer (2003), Heartbleed (2014) — all buffer overflows. The pattern is universal; the defences have evolved, but the vulnerability remains common in C code.

</div>

---

## 9.4 — Defences Against Buffer Overflow

???+ note "Four defences"
    | Defence | How it works |
    |---|---|
    | **Stack canaries** | Random value between buffer and return address; checked on return |
    | **ASLR** | Randomise stack/heap/library addresses |
    | **DEP / NX** | Mark stack/heap as non-executable |
    | **Safe languages** | Java, Rust, Python check bounds automatically |

<div class="fragment">

???+ note "Stack canaries"
    ```
    ┌──────────────────┐
    │ return address   │
    ├──────────────────┤
    │ canary (random)  │  ← if it changes, crash
    ├──────────────────┤
    │ saved EBP        │
    ├──────────────────┤
    │ buffer           │
    └──────────────────┘
    ```
    
    The compiler inserts a random value before the return address and checks it on return. If it's changed, the program aborts.

</div>

<div class="fragment">

???+ note "ASLR"
    **Address Space Layout Randomisation** — the OS randomises where the stack, heap, and libraries are placed. The attacker cannot predict the return address to jump to.

</div>

<div class="fragment">

???+ note "DEP / NX"
    **Data Execution Prevention / No-eXecute** — the CPU marks memory pages as non-executable. If the attacker puts shellcode on the stack, it cannot run.

</div>

<div class="fragment">

!!! tip "Defence in depth"
    Canaries, ASLR, and DEP are **complementary**. A successful attack must bypass all three. This is why modern systems are still vulnerable — each defence is defeatable in isolation, but together they raise the bar enormously.

</div>

---

## 9.5 — Denial of Service (DoS)

???+ note "The attack"
    Flood a service with requests until it can no longer serve legitimate users.

<div class="fragment">

???+ note "Three flavours"
    | Type | Description |
    |---|---|
    | **Volumetric** | Flood bandwidth (Gbps) |
    | **Protocol** | Exploit protocol state (SYN flood) |
    | **Application** | Slow HTTP requests, expensive queries |

</div>

<div class="fragment">

???+ note "Distributed DoS (DDoS)"
    Use a **botnet** of compromised machines to attack from many sources. The target cannot simply block one IP.

</div>

<div class="fragment">

???+ info "Defences"
    - **Rate limiting** — cap requests per IP.
    - **Traffic filtering** — block known-bad IPs.
    - **Anycast** — distribute the load across many servers.
    - **CDN** — absorb the traffic at the edge.
    - **CAPTCHAs** — distinguish humans from bots.

</div>

---

## 9.6 — Social Engineering

???+ note "The attack"
    Manipulate **people**, not systems, to gain access.

<div class="fragment">

???+ note "Three common techniques"
    | Technique | Example |
    |---|---|
    | **Phishing** | Fake email from "IT" asking for password |
    | **Pretexting** | Call claiming to be a vendor needing access |
    | **Baiting** | USB drive in the parking lot |

</div>

<div class="fragment">

???+ info "Why it works"
    Humans are not trained to detect deception. Phishing emails exploit **trust**, **urgency**, and **authority** — the same levers that legitimate communication uses.

</div>

<div class="fragment">

???+ note "Defences"
    - **Training** — teach employees to recognise phishing.
    - **Verification** — call back on a known number to confirm.
    - **MFA** — even a stolen password is useless without the second factor.
    - **Restricted access** — least privilege limits the damage of a compromised account.

</div>

---

## 9.7 — Authentication: The Three Factors

???+ note "Something you know"
    **Passwords, PINs, security questions.**
    
    - **Pros:** familiar, cheap.
    - **Cons:** phishable, reusable, hard to remember securely.

<div class="fragment">

???+ note "Something you have"
    **Smart cards, hardware tokens, phones (TOTP).**
    
    - **Pros:** not phishable directly.
    - **Cons:** can be lost or stolen.

</div>

<div class="fragment">

???+ note "Something you are"
    **Fingerprints, face, iris, voice.**
    
    - **Pros:** hard to copy, always available.
    - **Cons:** cannot be changed if compromised.

</div>

<div class="fragment">

???+ note "The strength ordering"
    MFA (multi-factor) > something you have > something you are > something you know.

</div>

<div class="fragment">

!!! tip "Passwords must be hashed, never plaintext"
    Store **salted hashes** (bcrypt, scrypt, Argon2). Never store the password itself. Never use fast hashes (MD5, SHA-1) — they enable rainbow-table and brute-force attacks.

</div>

---

## 9.8 — Authorisation: Access Control Models

???+ note "Definition"
    **Authorisation** determines **what** an authenticated user can do. Authentication says "who you are"; authorisation says "what you can access".

<div class="fragment">

???+ note "Three models"
    | Model | Description | Example |
    |---|---|---|
    | **DAC** (Discretionary) | Owner decides | Unix file permissions |
    | **MAC** (Mandatory) | System-wide policy | SELinux, AppArmor |
    | **RBAC** (Role-Based) | Roles have permissions | Enterprise systems |

</div>

<div class="fragment">

???+ note "DAC — Unix permissions"
    ```
    -rwxr-xr-x  file.txt
     │││││││
     ││││││└── others: r-x
     ││││└──── group:  r-x
     │││└───── owner:  rwx
     ││└────── file type: -
     └└─────── special:  -
    ```
    
    The owner decides who gets access.

</div>

<div class="fragment">

???+ note "MAC — SELinux"
    Every process and file has a **security context** (user, role, type). The kernel enforces policies like:
    
    ```
    allow httpd_t var_log_t:file { read write };
    ```
    
    Only httpd can read/write its log files, even if the OS user has permissions.

</div>

<div class="fragment">

???+ note "RBAC — role-based"
    Users are assigned **roles**. Roles have **permissions**. Access is checked via the role.
    
    ```
    alice → doctor → read(patient_record)
    bob   → nurse  → read(patient_record)
    ```

</div>

---

## 9.9 — Encryption: At Rest and In Transit

???+ note "At rest"
    **Disk encryption** — full-disk (BitLocker, FileVault) or per-file.
    - **LUKS** (Linux) — encrypts the entire block device.
    - **eCryptfs** — per-file encryption in the filesystem.

<div class="fragment">

???+ note "In transit"
    **TLS** — encrypts network traffic.
    - **Client → server** — negotiate cipher, exchange keys.
    - **Then** — symmetric encryption (AES, ChaCha20).

</div>

<div class="fragment">

???+ note "The two encryption types"
    | Type | Speed | Use |
    |---|---|---|
    | **Symmetric** (AES) | Fast (~1 GB/s) | Bulk data |
    | **Asymmetric** (RSA, ECC) | Slow (~1 MB/s) | Key exchange, signatures |

</div>

<div class="fragment">

!!! tip "Real systems use both"
    TLS uses **asymmetric** to exchange a **symmetric** session key, then **symmetric** for the actual data. This gives the security of asymmetric with the speed of symmetric.

</div>

---

## 9.10 — Sandboxing

???+ note "Definition"
    **Sandboxing** isolates a program in a restricted environment — it can only access the resources the sandbox allows.

<div class="fragment">

???+ note "Examples"
    - **Browser tabs** — each tab runs in a separate process with restricted permissions.
    - **Containers** — Docker, LXC isolate processes.
    - **iOS/Android apps** — each app runs in its own sandbox directory.
    - **seccomp** (Linux) — restricts which syscalls a process can make.

</div>

<div class="fragment">

???+ info "What the sandbox controls"
    | Resource | How |
    |---|---|
    | **Filesystem** | Chroot, mount namespaces |
    | **Network** | Network namespaces, firewalls |
    | **CPU / memory** | Cgroups, rlimits |
    | **Syscalls** | seccomp filters |

</div>

<div class="fragment">

!!! warning "Sandboxes are not impenetrable"
    Escaping a container is possible (e.g., Dirty COW, runc CVE-2019-5736). A sandbox reduces the attack surface but does not eliminate it.

</div>

---

## 9.11 — Threat Modelling in Practice

???+ note "STRIDE framework"
    | Threat | Description |
    |---|---|
    | **Spoofing** | Pretending to be someone else |
    | **Tampering** | Modifying data |
    | **Repudiation** | Denying an action |
    | **Information disclosure** | Revealing private data |
    | **Denial of service** | Preventing access |
    | **Elevation of privilege** | Gaining higher access |

<div class="fragment">

???+ note "Applying STRIDE to a login form"
    | Threat | Mitigation |
    |---|---|
    | **Spoofing** | MFA |
    | **Tampering** | HTTPS, CSRF tokens |
    | **Repudiation** | Audit logs |
    | **Info disclosure** | Password hashing, rate limit error messages |
    | **DoS** | Rate limiting, CAPTCHA |
    | **Privilege escalation** | Least privilege |

</div>

<div class="fragment">

!!! tip "The threat model is the design"
    Security is not a feature you add at the end — it is a **design property**. A system with a clear threat model is more secure than one that adds defences reactively.

</div>

---

## 9.12 — The Security Mindset

???+ note "Three principles"
    | Principle | Meaning |
    |---|---|
    | **Least privilege** | Every component has only the access it needs |
    | **Defence in depth** | Multiple layers of defence |
    | **Fail securely** | On failure, deny access (not grant) |

<div class="fragment">

???+ info "The attacker's advantage"
    The defender must protect **every** entry point. The attacker needs to find **one** weakness. This asymmetry means security is always hard — and why **defence in depth** is essential.

</div>

<div class="fragment">

???+ note "Theoretical vs practical"
    - **Theoretical security:** unbreakable (one-time pad, formally verified).
    - **Practical security:** hard enough that the cost of attack exceeds the value.
    
    Real systems are always practical — the question is how high the bar is.

</div>

---

## Chapter 9 — key takeaways

<div class="fragment">

**1.** **Malware**: viruses need user action, **worms self-propagate**, trojans disguise as legitimate.

</div>

<div class="fragment">

**2.** **Buffer overflows** overwrite adjacent memory, including return addresses. Defences: **canaries**, **ASLR**, **DEP**, safe languages.

</div>

<div class="fragment">

**3.** **DoS** attacks flood a service; DDoS uses a botnet. Defences: rate limiting, CDN, CAPTCHA.

</div>

<div class="fragment">

**4.** **Social engineering** targets humans. **Training** and **MFA** are the best defences.

</div>

<div class="fragment">

**5.** **Authentication** verifies identity; **authorisation** controls access. Three factors: know, have, are.

</div>

<div class="fragment">

**6.** **Access control**: DAC (owner), MAC (policy), RBAC (roles).

</div>

<div class="fragment">

**7.** **Encryption**: symmetric for speed, asymmetric for key exchange. **TLS** combines both.

</div>

<div class="fragment">

**8.** **Sandboxing** isolates programs. **Least privilege**, **defence in depth**, **fail securely** are the guiding principles.

</div>

---

## Course wrap-up

<div class="fragment">

**The arc of ECEG-5202:**

1. **Chapter 1** — what an OS is, and why it exists.
2. **Chapter 2** — processes and threads.
3. **Chapter 3** — CPU scheduling.
4. **Chapter 4** — synchronization and IPC.
5. **Chapter 5** — deadlocks.
6. **Chapter 6** — memory management.
7. **Chapter 7** — file systems.
8. **Chapter 8** — I/O management.
9. **Chapter 9** — security.

</div>

<div class="fragment">

!!! tip "The unifying question"
    Every chapter asked: **how does the OS multiplex a finite resource across many processes, safely and efficiently?**
    
    - **CPU** — scheduling.
    - **Memory** — paging and virtual memory.
    - **Disk** — file systems and I/O scheduling.
    - **Shared data** — synchronisation and deadlock avoidance.
    - **Access** — security.

</div>

<div class="fragment">

**Good luck with the exam.**

</div>