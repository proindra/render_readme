Project Jackfruit: Building a Multi-Container RuntimeStep-by-Step Solution & Execution Guide with Screenshot MarkersTeam InformationMEMBER 1: Prajwalindra K H (SRN: PES2UG24AM117)MEMBER 2: Praveen Rajesh Naik (SRN: PES2UG24AM123)⚠️ IMPORTANT: Troubleshooting & Clean SlateIf you ever encounter errors like execvp failed to run /bin/sh (errno: No such file or directory) in your logs, it usually means your wget download failed, leaving you with empty folders!Whenever you need to start fresh, stop the supervisor (Ctrl+C in Terminal 1), and run these exact commands in Terminal 2 to wipe the slate clean and verify it works:# 1. Clean up everything (with sudo!)
sudo rm -rf logs/* rootfs-alpha rootfs-beta rootfs-base
make clean

# 2. Set up the fresh Alpine Filesystem
mkdir rootfs-base
URL="dl-cdn.alpinelinux.org/alpine/v3.20/releases/x86_64/alpine-minirootfs-3.20.3-x86_64.tar.gz"
wget -qO- "https://$URL" | tar -xz -C rootfs-base

# 3. VERIFY the download worked!
ls rootfs-base
# NOTE: If this prints nothing, the download failed! Try the wget command again.
# If it prints "bin dev etc home...", proceed to step 4!

# 4. Copy to container folders
cp -a ./rootfs-base ./rootfs-alpha
cp -a ./rootfs-base ./rootfs-beta

# 5. Start the container again (Make sure supervisor is running in Terminal 1!)
sudo ./engine start alpha ./rootfs-alpha "ls -l /"

# 6. Check the logs
cat logs/alpha.log
Phase 1: Environment Setup1. Compile the User-Space Enginecd boilerplate
make
2. Load the Kernel Modulesudo insmod monitor.ko
ls -l /dev/container_monitor
3. Setup the Alpine Linux Root Filesystems(If you already did the "Clean Slate" troubleshooting steps above, you can skip this step!)mkdir rootfs-base
URL="dl-cdn.alpinelinux.org/alpine/v3.20/releases/x86_64/alpine-minirootfs-3.20.3-x86_64.tar.gz"
wget -qO- "https://$URL" | tar -xz -C rootfs-base

# Verify download worked before copying!
ls rootfs-base

cp -a ./rootfs-base ./rootfs-alpha
cp -a ./rootfs-base ./rootfs-beta
Phase 2: Supervisor & Multi-Container Demo1. Start the Supervisor DaemonOpen Terminal 1.sudo ./engine supervisor ./rootfs-base
2. Launch Two Containers (Multi-Container Supervision)In Terminal 2, run:sudo ./engine start alpha ./rootfs-alpha "ls -l /"
sudo ./engine start beta ./rootfs-beta "ls -l /"
📸 SCREENSHOT #1 (Multi-container supervision): Capture Terminal 2 showing both containers being started successfully.3. Test CLI and IPC (Split View)Issuing a command in one terminal and seeing the reaction in the other.# In Terminal 2
sudo ./engine stop alpha
📸 SCREENSHOT #4 (CLI and IPC): Take a split-view screenshot showing Terminal 2 sending the command and Terminal 1 (Supervisor) printing the acknowledgement.4. Check Metadata Tracking(Make sure to restart the supervisor in Terminal 1 if you used the stop command above!)sudo ./engine ps
📸 SCREENSHOT #2 (Metadata tracking): Capture the output showing the list of containers and their current states.Phase 3: Bounded-Buffer Logging1. Verify Log Capturecat logs/alpha.log
📸 SCREENSHOT #3 (Bounded-buffer logging): Capture the terminal showing the Alpine directory listing inside the log file.Phase 4: Kernel Memory Monitor (OOM Killer)1. Inject and Run Memory Hogsudo cp memory_hog ./rootfs-alpha/
sudo ./engine start alpha-hog ./rootfs-alpha "/memory_hog 10 500"
2. Observe Memory EnforcementWait 5-10 seconds, then check kernel logs.sudo dmesg | tail -n 15
📸 SCREENSHOT #5 (Soft-limit warning): Capture the dmesg output highlighting the SOFT LIMIT warning line.📸 SCREENSHOT #6 (Hard-limit enforcement): Capture the dmesg output highlighting the HARD LIMIT kill line.Phase 5: The CPU Scheduler Experiment1. Run Simultaneous Workloadssudo cp cpu_hog ./rootfs-alpha/
sudo cp cpu_hog ./rootfs-beta/
sudo ./engine start cpu-alpha ./rootfs-alpha "/cpu_hog 10" --nice 0
sudo ./engine start cpu-beta ./rootfs-beta "/cpu_hog 10" --nice 19
2. Analyze Scheduling LogsWait 15 seconds, then run:cat logs/cpu-alpha.log
cat logs/cpu-beta.log
📸 SCREENSHOT #7 (Scheduling experiment): Capture the output of both logs side-by-side or sequentially, showing the difference in progress/accumulators.Phase 6: Clean Teardown1. Cleanup and UnloadIn Terminal 1, stop the supervisor with Ctrl+C. Then in Terminal 2:sudo killall engine
sudo rm -f /tmp/mini_runtime.sock
sudo rmmod monitor
make clean
📸 SCREENSHOT #8 (Clean teardown): Capture the terminal showing these cleanup commands finishing with no errors or kernel crashes.