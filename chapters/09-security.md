# Chapter 9 — Introduction to Security

!!! abstract "Chapter at a glance"
    **Coverage:** threats and attacks · defences.

---

## Learning outcomes

1. Classify security threats: malware (virus, worm, trojan), buffer overflow, DoS, social engineering.
2. Describe authentication mechanisms (passwords, biometrics, multi-factor).
3. Explain access control (ACLs, capabilities, mandatory vs. discretionary).
4. Describe defences: sandboxing, ASLR, DEP, firewalls, encryption.

---

## 9.1 Threats and attacks

### 9.1.1 Malware

| Type | Description | Propagation |
|---|---|---|
| **Virus** | Attaches to a host program | Requires user action to spread |
| **Worm** | Standalone program | Self-propagates over the network |
| **Trojan** | Malicious code disguised as legitimate software | User installs it voluntarily |

!!! note "Why worms are more dangerous"
    A **worm** propagates **automatically** over the network, exploiting vulnerabilities without any user action. A **virus** requires the user to run an infected program, so its spread is slower and more limited.

### 9.1.2 Buffer overflow

A **buffer overflow** occurs when a program writes more data into a buffer than it can hold, overwriting adjacent memory (including the return address on the stack).

**Classic attack:** overwrite the return address to jump to shellcode placed earlier in the buffer.

**Defences:**

- **Bounds checking** at compile time (safe languages).
- **Stack canaries** — a random value placed between the buffer and the return address; if it changes, the program aborts.
- **ASLR (Address Space Layout Randomisation)** — randomise stack, heap, and library addresses so the attacker cannot predict where to jump.
- **DEP / NX (Data Execution Prevention / No-eXecute)** — mark stack and heap as non-executable.

### 9.1.3 Denial of Service (DoS)

An attacker floods a service with requests until it can no longer respond to legitimate users. **Distributed DoS (DDoS)** uses a botnet of compromised machines.

**Defences:** rate limiting, traffic filtering, anycast distribution, CAPTCHAs.

### 9.1.4 Social engineering

Manipulating people into revealing confidential information or performing actions that compromise security. Phishing, pretexting, baiting.

---

## 9.2 Authentication

**Authentication** verifies **who you are**. Three factors:

| Factor | Example |
|---|---|
| **Something you know** | Password, PIN |
| **Something you have** | Smart card, hardware token, phone |
| **Something you are** | Fingerprint, face recognition, iris scan |

**Multi-factor authentication (MFA)** combines two or more factors — e.g., password + one-time code from a phone.

**Password storage.** Passwords must **never** be stored in plaintext. Use:

- **Salted hash** — a random salt per user, then a slow hash (bcrypt, scrypt, Argon2).
- **Never** MD5 or SHA-1 for passwords — they are too fast and broken.

---

## 9.3 Authorisation and access control

**Authorisation** determines **what you can do**.

| Model | Description |
|---|---|
| **Discretionary access control (DAC)** | Owner decides who gets access (Unix permissions, ACLs) |
| **Mandatory access control (MAC)** | System-wide policy decides (SELinux, AppArmor) |
| **Role-based access control (RBAC)** | Permissions assigned to roles, not individuals |

**Unix permissions** are a simple DAC scheme: each file has an owner, group, and mode bits (read / write / execute for owner, group, others).

---

## 9.4 Defences

- **Principle of least privilege** — every component should have only the permissions it needs.
- **Sandboxing** — run untrusted code in an isolated environment (browser tabs, containers).
- **Encryption** — at rest (disk encryption) and in transit (TLS).
- **Firewalls** — filter network traffic by port, source, or protocol.
- **Intrusion detection / prevention systems (IDS/IPS)** — monitor for known attack patterns.
- **Security updates** — patch known vulnerabilities promptly.
- **Audit logging** — record security-relevant events for post-incident analysis.

---

## 9.5 Quiz

1. Compare viruses, worms, and trojans on propagation.
2. Explain the buffer-overflow attack and three defences.
3. What is the difference between authentication and authorisation?
4. Why must passwords be stored as salted hashes, not plaintext?
5. Describe the principle of least privilege and give one example.

---

## Answer key

??? success "Click to reveal"

    1. Virus: needs a host, requires user action. Worm: standalone, self-propagates over the network. Trojan: disguised as legitimate software, user installs it.
    2. Overflow a buffer, overwrite the return address, jump to shellcode. Defences: stack canaries, ASLR, DEP.
    3. Authentication verifies identity; authorisation determines permissions.
    4. Salted hashes prevent rainbow-table attacks and hide identical passwords; slow hashes resist brute force.
    5. Every component should have only the privileges it needs. Example: a web server runs as an unprivileged user, not root.

---

## Further reading

- Tanenbaum §9.1–§9.5.
- Silberschatz §15.1–§15.4.