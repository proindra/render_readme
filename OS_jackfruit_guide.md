Here is your **PDF content converted into clean, properly formatted Markdown**. I preserved headings, code blocks, lists, and notes so you can paste it directly into **GitHub / Notion / Markdown editors**.

---

# Project Jackfruit: Building a Multi-Container Runtime

### Step-by-Step Solution & Execution Guide with Screenshot Markers

## Team Information

* **Member 1:** Prajwalindra K H (SRN: PES2UG24AM117)
* **Member 2:** Praveen Rajesh Naik (SRN: PES2UG24AM123)

---

# ⚠️ IMPORTANT: Troubleshooting & Clean Slate

If you ever encounter errors like:

```
execvp failed to run /bin/sh (errno: No such file or directory)
```

in your logs, it usually means your `wget` download failed, leaving you with empty folders.

Whenever you need to start fresh:

1. Stop the supervisor (`Ctrl+C` in Terminal 1)
2. Run the following commands in **Terminal 2**

---

## 1. Clean everything

```bash
sudo rm -rf logs/* rootfs-alpha rootfs-beta rootfs-base
make clean
```

---

## 2. Setup fresh Alpine filesystem

```bash
mkdir rootfs-base

wget -qO- \
"https://dl-cdn.alpinelinux.org/alpine/v3.20/releases/x86_64/alpine-minirootfs-3.20.3-x86_64.tar.gz" \
| tar -xz -C rootfs-base
```

---

## 3. Verify the download worked

```bash
ls rootfs-base
```

If this prints **nothing**, the download failed.

If it prints something like:

```
bin dev etc home ...
```

then proceed.

---

## 4. Copy to container folders

```bash
cp -a ./rootfs-base ./rootfs-alpha
cp -a ./rootfs-base ./rootfs-beta
```

---

## 5. Start the container again

*(Make sure supervisor is running in Terminal 1)*

```bash
sudo ./engine start alpha ./rootfs-alpha "ls -l /"
```

---

## 6. Check logs

```bash
cat logs/alpha.log
```

---

# Phase 1: Environment Setup

## 1. Compile the User-Space Engine

```bash
cd boilerplate
make
```

---

## 2. Load the Kernel Module

```bash
sudo insmod monitor.ko
ls -l /dev/container_monitor
```

---

## 3. Setup Alpine Linux Root Filesystems

*(Skip if you already did the troubleshooting steps above)*

```bash
mkdir rootfs-base

wget -qO- \
"https://dl-cdn.alpinelinux.org/alpine/v3.20/releases/x86_64/alpine-minirootfs-3.20.3-x86_64.tar.gz" \
| tar -xz -C rootfs-base
```

Verify download:

```bash
ls rootfs-base
```

Copy filesystem:

```bash
cp -a ./rootfs-base ./rootfs-alpha
cp -a ./rootfs-base ./rootfs-beta
```

---

# Phase 2: Supervisor & Multi-Container Demo

## 1. Start the Supervisor Daemon

Open **Terminal 1**

```bash
sudo ./engine supervisor ./rootfs-base
```

---

## 2. Launch Two Containers

In **Terminal 2**:

```bash
sudo ./engine start alpha ./rootfs-alpha "ls -l /"
sudo ./engine start beta ./rootfs-beta "ls -l /"
```

📸 **Screenshot #1 — Multi-container supervision**

Capture Terminal 2 showing **both containers starting successfully**.

---

## 3. Test CLI and IPC (Split View)

Issue command in one terminal and observe supervisor reaction.

```bash
sudo ./engine stop alpha
```

📸 **Screenshot #4 — CLI and IPC**

Take a **split view screenshot** showing:

* Terminal 2 sending the command
* Terminal 1 printing acknowledgement

---

## 4. Check Metadata Tracking

*(Restart supervisor in Terminal 1 if needed)*

```bash
sudo ./engine ps
```

📸 **Screenshot #2 — Metadata Tracking**

Capture output showing **containers and their states**.

---

# Phase 3: Bounded-Buffer Logging

## Verify Log Capture

```bash
cat logs/alpha.log
```

📸 **Screenshot #3 — Bounded Buffer Logging**

Capture terminal showing **Alpine directory listing inside log file**.

---

# Phase 4: Kernel Memory Monitor (OOM Killer)

## 1. Inject Memory Hog

```bash
sudo cp memory_hog ./rootfs-alpha/
```

Run it:

```bash
sudo ./engine start alpha-hog ./rootfs-alpha "/memory_hog 10 500"
```

---

## 2. Observe Memory Enforcement

Wait **5–10 seconds**, then run:

```bash
sudo dmesg | tail -n 15
```

📸 **Screenshot #5 — Soft Limit Warning**

Capture `dmesg` output highlighting **SOFT LIMIT warning line**.

📸 **Screenshot #6 — Hard Limit Enforcement**

Capture `dmesg` output highlighting **HARD LIMIT kill line**.

---

# Phase 5: CPU Scheduler Experiment

## 1. Run Simultaneous Workloads

```bash
sudo cp cpu_hog ./rootfs-alpha/
sudo cp cpu_hog ./rootfs-beta/

sudo ./engine start cpu-alpha ./rootfs-alpha "/cpu_hog 10" --nice 0
sudo ./engine start cpu-beta ./rootfs-beta "/cpu_hog 10" --nice 19
```

---

## 2. Analyze Scheduling Logs

Wait **15 seconds**, then run:

```bash
cat logs/cpu-alpha.log
cat logs/cpu-beta.log
```

📸 **Screenshot #7 — Scheduling Experiment**

Capture logs **side-by-side** showing the difference in progress.

---

# Phase 6: Clean Teardown

## Cleanup and Unload

In **Terminal 1**

Stop supervisor with:

```
Ctrl + C
```

Then in **Terminal 2**

```bash
sudo killall engine
sudo rm -f /tmp/mini_runtime.sock

sudo rmmod monitor
make clean
```

📸 **Screenshot #8 — Clean Teardown**

Capture terminal showing cleanup commands **finishing without errors or kernel crashes**.

---

✅ If you want, I can also:

* **Convert this into a GitHub-ready README.md** 📘
* **Improve formatting with tables, warnings, and collapsible sections**
* **Generate a report version for submission (Markdown → PDF)**.
