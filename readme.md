Below is your **PDF converted into clean, exact Markdown (`README.md`)** while preserving all sections, commands, and explanations. I only fixed formatting so it **renders properly on GitHub**.

---

# Multi-Container Runtime (Jackfruit Project)

A lightweight Linux container runtime written in **C**, featuring:

* A long-running **supervisor**
* **Bounded-buffer logging pipeline**
* **Kernel-space memory monitor**

---

# 1. Team Information

### Member 1

**Name:** Prajwalindra K H
**SRN:** PES2UG24AM117

### Member 2

**Name:** Praveen Rajesh Naik
**SRN:** PES2UG24AM123

---

# 2. Build, Load, and Run Instructions

## 2.1 Setup and Build

Compile the user-space engine and test workloads.

```bash
cd boilerplate
make
```

---

## 2.2 Load the Kernel Module

```bash
sudo insmod monitor.ko
ls -l /dev/container_monitor
```

Verify that the device `/dev/container_monitor` is created.

---

## 2.3 Setup Container Filesystems

```bash
mkdir rootfs-base

wget https://dl-cdn.alpinelinux.org/alpine/v3.20/releases/x86_64/alpine-minirootfs-3.20.3-x86_64.tar.gz

tar -xzf alpine-minirootfs-3.20.3-x86_64.tar.gz -C rootfs-base

cp -a ./rootfs-base ./rootfs-alpha
cp -a ./rootfs-base ./rootfs-beta
```

---

## 2.4 Start the Supervisor Daemon

In **Terminal 1**:

```bash
sudo ./engine supervisor ./rootfs-base
```

---

## 2.5 Launch Containers and CLI Commands

In **Terminal 2**:

### Start a basic container

```bash
sudo ./engine start alpha ./rootfs-alpha "ls -l /"
```

### View container logs

```bash
cat logs/alpha.log
```

### View running containers metadata

```bash
sudo ./engine ps
```

### Stop a container

```bash
sudo ./engine stop alpha
```

---

## 2.6 Unload and Clean Up

Stop the supervisor using **Ctrl + C** in Terminal 1.

Then run:

```bash
sudo killall engine
sudo rm -f /tmp/mini_runtime.sock
sudo rmmod monitor
sudo make clean
```

---

# 3. Demo with Screenshots

### 1. Multi-container supervision

**Caption:**
Two or more containers running simultaneously under one supervisor process.

---

### 2. Metadata tracking

**Caption:**
Output of the following command showing tracked container metadata and states:

```bash
sudo ./engine ps
```

---

### 3. Bounded-buffer logging

**Caption:**
Log file contents of `alpha.log` successfully captured through the multi-threaded logging pipeline.

---

### 4. CLI and IPC

**Caption:**
CLI `start` command being issued in Terminal 2, with the supervisor acknowledging it in Terminal 1.

---

### 5. Soft-limit warning

**Caption:**
`dmesg` output showing the kernel module emitting a **SOFT LIMIT** warning for the memory hog.

---

### 6. Hard-limit enforcement

**Caption:**
`dmesg` output showing the container being forcefully killed (`SIGKILL`) after exceeding its hard limit.

---

### 7. Scheduling experiment

**Caption:**
Terminal output of `cpu-alpha` (nice 0) and `cpu-beta` (nice 19) logs showing the **CFS scheduler prioritizing alpha**.

---

### 8. Clean teardown

**Caption:**
Evidence of clean teardown (`rmmod monitor`, `killall engine`) with **no lingering zombies or kernel panics**.

---

# 4. Engineering Analysis

## 4.1 Isolation Mechanisms

The runtime achieves process isolation using **Linux namespaces**.

The `clone()` system call is used with:

* `CLONE_NEWPID` – isolates process IDs so the container sees itself as PID 1
* `CLONE_NEWUTS` – allows a separate hostname
* `CLONE_NEWNS` – mount namespace isolation

Filesystem isolation is achieved using:

```
chroot
```

This traps the process inside a directory (e.g., `rootfs-alpha`), preventing access to the host filesystem.

However, all containers still share the **same underlying host kernel**.

---

## 4.2 Supervisor and Process Lifecycle

A long-running **supervisor daemon** manages multiple isolated processes.

Responsibilities include:

* Calling `clone()` to create container processes
* Acting as the parent process
* Tracking metadata such as container state, host PID, and limits
* Handling **IPC communication** from CLI clients
* Calling `waitpid()` to reap terminated children and prevent zombie processes

---

## 4.3 IPC, Threads, and Synchronization

Two main IPC mechanisms are used.

### Path B — CLI Communication

Uses a **UNIX Domain Socket** for CLI-to-supervisor communication.

### Path A — Container Logging

Uses **unidirectional pipes** to capture:

```
stdout
stderr
```

To avoid blocking I/O, the runtime uses a **multi-threaded bounded buffer**.

Synchronization mechanisms include:

* `pthread_mutex_t` for safe buffer access
* `pthread_cond_t` condition variables

These conditions ensure:

* Producers sleep when the buffer is full (`not_full`)
* Consumers sleep when the buffer is empty (`not_empty`)

This prevents **busy waiting and deadlocks**.

---

## 4.4 Memory Management and Enforcement

Memory usage is measured using **RSS (Resident Set Size)**.

RSS represents the physical RAM currently used by a process.

The memory policy has two tiers:

### Soft Limit

Logs a warning to alert administrators.

### Hard Limit

Immediately terminates the process using:

```
SIGKILL
```

Kernel-space enforcement is required because:

* User-space monitoring can be too slow
* A frozen process cannot terminate itself
* Kernel modules can intervene immediately

---

## 4.5 Scheduling Behavior

Two CPU-bound workloads are executed simultaneously.

| Container | Priority |
| --------- | -------- |
| cpu-alpha | nice 0   |
| cpu-beta  | nice 19  |

Linux uses the **Completely Fair Scheduler (CFS)**.

Processes with **lower nice values receive more CPU time**.

Therefore:

* `cpu-alpha` receives larger time slices
* `cpu-beta` runs slower due to reduced priority

---

# 5. Design Decisions and Tradeoffs

## Namespace Isolation vs Full Virtualization

**Choice:**
Use `clone()` namespaces instead of full virtual machines (QEMU/KVM).

**Tradeoff:**

| Advantage                  | Disadvantage                       |
| -------------------------- | ---------------------------------- |
| Containers start instantly | Less secure than VMs               |
| Minimal overhead           | Kernel vulnerabilities affect host |

---

## UNIX Domain Sockets for CLI Control

**Choice:**
Use `AF_UNIX` sockets for CLI communication.

**Tradeoff:**

| Advantage        | Disadvantage                   |
| ---------------- | ------------------------------ |
| Built-in queuing | Slight context-switch overhead |
| Simpler protocol | Slower than shared memory      |

---

## Kernel Linked List with Mutex

The kernel module tracks containers using:

```
struct list_head
```

protected by a **mutex**.

**Tradeoff:**

| Benefit                            | Cost                         |
| ---------------------------------- | ---------------------------- |
| Safe memory allocation (`kmalloc`) | Slight timer interrupt delay |
| Prevents race conditions           | Possible contention          |

---

# 6. Scheduler Experiment Results

### Configuration

Container 1:

```
cpu-alpha
/cpu_hog 10
--nice 0
```

Container 2:

```
cpu-beta
/cpu_hog 10
--nice 19
```

### Analysis

Both workloads execute a tight loop for **10 seconds of wall-clock time**.

Because `cpu-alpha` runs with **higher priority**, the **Completely Fair Scheduler (CFS)** allocates it more CPU time.

As a result:

* `cpu-alpha` performs significantly more iterations
* `cpu-beta` progresses slower due to lower priority

The log output shows **higher accumulator values** for `cpu-alpha`, confirming the scheduler’s behavior.

---

✅ This Markdown will render **cleanly on GitHub without spacing issues** and is an **exact structured conversion of your PDF**.

---

If you want, I can also give you a **🔥 “top-grade GitHub README version”** of this with:

* architecture diagram of the container runtime
* supervisor + kernel module workflow diagram
* container lifecycle diagram

which would make your project **look much more impressive for submission**.
