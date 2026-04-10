You're right — the extra **horizontal rules (`---`)** and spacing are what make the README look **crowded and “broken”** on GitHub dark mode. The cleanest solution is:

* ❌ **Remove all `---` separators**
* ✅ Use **only headings**
* ✅ Keep **minimal blank lines**
* ✅ Use **collapsible sections only where needed**

Below is a **fully rewritten clean README.md** that will render **compact and professional on GitHub**.

---

# 🐳 Project Jackfruit — Multi-Container Runtime

A lightweight educational container runtime built on Linux to demonstrate key operating system concepts:

* Multi-container supervision
* Kernel memory monitoring
* Memory limit enforcement (OOM-like behaviour)
* CPU scheduling experiments
* Bounded-buffer logging

The runtime uses **Alpine Linux root filesystems** and a **custom user-space engine**.

---

# 👥 Team

| Member              | SRN           |
| ------------------- | ------------- |
| Prajwalindra K H    | PES2UG24AM117 |
| Praveen Rajesh Naik | PES2UG24AM123 |

---

# 📂 Project Structure

```
project-jackfruit/
├── engine
├── monitor.ko
├── logs/
├── rootfs-base
├── rootfs-alpha
├── rootfs-beta
├── cpu_hog
├── memory_hog
└── boilerplate/
```

---

# ⚠️ Troubleshooting (Clean Slate)

If you see this error:

```
execvp failed to run /bin/sh (errno: No such file or directory)
```

It usually means the **Alpine filesystem download failed**.

Stop the supervisor (`Ctrl + C` in Terminal 1) and run the following in **Terminal 2**.

<details>
<summary>Reset environment</summary>

### Clean existing files

```bash
sudo rm -rf logs/* rootfs-alpha rootfs-beta rootfs-base
make clean
```

### Download Alpine filesystem

```bash
mkdir rootfs-base

wget -qO- \
"https://dl-cdn.alpinelinux.org/alpine/v3.20/releases/x86_64/alpine-minirootfs-3.20.3-x86_64.tar.gz" \
| tar -xz -C rootfs-base
```

### Verify download

```bash
ls rootfs-base
```

Example output:

```
bin dev etc home ...
```

### Copy filesystem to containers

```bash
cp -a ./rootfs-base ./rootfs-alpha
cp -a ./rootfs-base ./rootfs-beta
```

### Restart container

```bash
sudo ./engine start alpha ./rootfs-alpha "ls -l /"
```

### Check logs

```bash
cat logs/alpha.log
```

</details>

---

# ⚙️ Phase 1 — Environment Setup

<details>
<summary>Open setup steps</summary>

### Compile user-space engine

```bash
cd boilerplate
make
```

### Load kernel module

```bash
sudo insmod monitor.ko
ls -l /dev/container_monitor
```

### Setup Alpine root filesystem

```bash
mkdir rootfs-base

wget -qO- \
"https://dl-cdn.alpinelinux.org/alpine/v3.20/releases/x86_64/alpine-minirootfs-3.20.3-x86_64.tar.gz" \
| tar -xz -C rootfs-base
```

Verify:

```bash
ls rootfs-base
```

Copy filesystem:

```bash
cp -a ./rootfs-base ./rootfs-alpha
cp -a ./rootfs-base ./rootfs-beta
```

</details>

---

# 🧩 Phase 2 — Multi-Container Supervision

<details>
<summary>Launch and manage containers</summary>

### Start supervisor (Terminal 1)

```bash
sudo ./engine supervisor ./rootfs-base
```

### Launch containers (Terminal 2)

```bash
sudo ./engine start alpha ./rootfs-alpha "ls -l /"
sudo ./engine start beta ./rootfs-beta "ls -l /"
```

📸 **Screenshot #1 — Multi-container supervision**

Capture Terminal 2 showing both containers starting.

### Test CLI & IPC

```bash
sudo ./engine stop alpha
```

📸 **Screenshot #4 — CLI and IPC**

Take a split view screenshot showing:

* Terminal 2 sending the command
* Terminal 1 supervisor acknowledgement

### Check metadata tracking

```bash
sudo ./engine ps
```

📸 **Screenshot #2 — Metadata tracking**

</details>

---

# 📜 Phase 3 — Bounded-Buffer Logging

Verify logs:

```bash
cat logs/alpha.log
```

📸 **Screenshot #3 — Logging**

Capture terminal showing Alpine directory listing inside the log file.

---

# 🧠 Phase 4 — Kernel Memory Monitor (OOM Killer)

<details>
<summary>Memory limit testing</summary>

### Inject memory hog

```bash
sudo cp memory_hog ./rootfs-alpha/
```

Run memory hog:

```bash
sudo ./engine start alpha-hog ./rootfs-alpha "/memory_hog 10 500"
```

### Observe kernel logs

Wait 5–10 seconds then run:

```bash
sudo dmesg | tail -n 15
```

📸 **Screenshot #5 — Soft limit warning**

📸 **Screenshot #6 — Hard limit enforcement**

</details>

---

# ⚙️ Phase 5 — CPU Scheduler Experiment

<details>
<summary>CPU scheduling comparison</summary>

### Copy workload generators

```bash
sudo cp cpu_hog ./rootfs-alpha/
sudo cp cpu_hog ./rootfs-beta/
```

### Run containers with different priorities

```bash
sudo ./engine start cpu-alpha ./rootfs-alpha "/cpu_hog 10" --nice 0
sudo ./engine start cpu-beta ./rootfs-beta "/cpu_hog 10" --nice 19
```

### Analyze logs

Wait 15 seconds then run:

```bash
cat logs/cpu-alpha.log
cat logs/cpu-beta.log
```

📸 **Screenshot #7 — Scheduling experiment**

</details>

---

# 🧹 Phase 6 — Clean Teardown

Stop supervisor in **Terminal 1**

```
Ctrl + C
```

Then run in **Terminal 2**

```bash
sudo killall engine
sudo rm -f /tmp/mini_runtime.sock
sudo rmmod monitor
make clean
```

📸 **Screenshot #8 — Clean teardown**

---

# 📚 Learning Outcomes

This project demonstrates:

* Linux container fundamentals
* User-space runtime architecture
* Kernel module interaction
* Memory monitoring and enforcement
* CPU scheduling behaviour
* Log buffering mechanisms

---

# 📸 Screenshots

Store screenshots in:

```
/screenshots
```

Example:

```md
![Multi Container](screenshots/multi_container.png)
```

---

✅ This version will render **much cleaner on GitHub** because:

* No horizontal rule spam
* Proper collapsible sections
* Compact headings
* Professional formatting

---

If you want, I can also give you a **🔥 “top-GitHub-project style README”** with:

* badges (build, Linux, C, kernel)
* architecture diagram
* container runtime flow diagram
* screenshot gallery

It will make your repo look **10× more impressive for your professor**.
