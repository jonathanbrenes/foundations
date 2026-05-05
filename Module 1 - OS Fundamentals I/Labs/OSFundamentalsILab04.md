# Lab 4 — Process Management

## Title and Scenario

This lab covers process management on Linux. You will list and inspect running processes, monitor system activity in real time, manage process priorities, send signals to processes, and work with background and foreground jobs. You will perform the same tasks on both a RHEL and an Ubuntu VM to observe how the two distributions handle processes and their default tooling.

This is a guided, foundational lab. All steps are provided with expected commands and validation points.

- **Estimated time:** 45 minutes.
- **VM count:** 2 (RHEL 8.10, Ubuntu 24.04).

---

## Deployment

Deploy the dedicated Lab 04 VMs using the template below.

[![Click to deploy](https://aka.ms/deploytoazurebutton)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fjonathanbrenes%2Ffoundations%2Fmain%2FModule%25201%2520-%2520OS%2520Fundamentals%2520I%2FLabs%2FFoundationsLab04.json)

> This template deploys two VMs: a RHEL 8.10 VM (`lab04rhel`) and an Ubuntu 24.04 VM (`lab04ubuntu`), both with password authentication.

> **Warning:** The VMs deployed in these labs allow inbound traffic automatically from the Azure VPN (AzureCloud service tag). Make sure you are connected to the Azure VPN, or add an NSG rule to allow your public IP address before attempting to connect.

---

## Skills Required

- Ability to connect to a Linux VM via SSH.
- Basic familiarity with running commands in a Linux terminal.

---

## Recommended Prerequisites

- Lab 1 completed — WSL2 or Cloud Shell configured, Azure CLI available, SSH connectivity understood.
- Lab 2 completed — familiarity with `sudo` and switching to root.
- Lab 04 VMs deployed and accessible.

---

## Objectives

After completing this lab, you will be able to:

- List all running processes and interpret their state, priority, and resource usage.
- Use `top` to monitor system activity in real time.
- Change process priority using `nice` and `renice`.
- Send signals to processes using `kill`, `pkill`, and `killall`.
- Manage background and foreground jobs using `jobs`, `bg`, and `fg`.
- Compare process management behavior across RHEL and Ubuntu.

---

## Environment Overview

| Component | Details |
|---|---|
| VM 1 | RHEL 8.10 (`lab04rhel`) |
| VM 2 | Ubuntu 24.04 (`lab04ubuntu`) |
| Access | SSH as `azureuser` |
| Working user | `azureuser` for most steps; `sudo` required for select operations |

---

## Your Mission

Complete all steps in order. Unless otherwise noted, run each scenario on both VMs and note any differences in output or behavior.

### Scenario 1 — Listing and inspecting processes

Connect to both VMs before starting. Open two terminal windows or tabs — one for each VM.

```bash
ssh azureuser@<LAB04RHEL_PUBLIC_IP>   # Connect to the RHEL 8.10 VM
ssh azureuser@<LAB04UBUNTU_PUBLIC_IP> # Connect to the Ubuntu 24.04 VM
```

#### Instructions

Step 1: List all running processes in BSD format
- Use `ps aux` to list all processes on the system with CPU and memory usage.
  ```bash
  ps aux # List all processes in BSD format: USER, PID, %CPU, %MEM, STAT, COMMAND
  ps aux | wc -l # Count the total number of processes currently running
  ```

  > The `STAT` column shows the process state. Common states: `R` (running), `S` (sleeping), `D` (uninterruptible sleep), `Z` (zombie), `T` (stopped).

Step 2: List all processes in UNIX format
- Use `ps -ef` to list all processes in full UNIX format.
  ```bash
  ps -ef # List all processes: UID, PID, PPID, C, STIME, TTY, TIME, CMD
  ps -ef | head -20 # Show only the first 20 lines
  ```

  > The `PPID` field shows the parent process ID. All processes on the system descend from PID 1 (`systemd`).

Step 3: Search for a specific process
- Use `pgrep` to find the PID of a process by name, and pipe `ps` output through `grep` for filtering.
  ```bash
  pgrep sshd # Find the PID of the sshd process
  ps aux | grep sshd # Filter ps output for lines containing sshd
  ps -ef | grep -v grep | grep sshd # Same filter, removing the grep process itself from results
  ```

  > `pgrep` is cleaner than grepping `ps` output and avoids the common issue of the `grep` process matching its own search term.

Step 4: Display process tree
- Use `pstree` to visualize the parent-child relationship between processes.
  ```bash
  pstree # Display all processes in a tree structure
  pstree -p # Display the tree with PIDs included
  pstree azureuser # Display only processes owned by azureuser
  ```

  > On Ubuntu, `pstree` may need to be installed: `sudo apt install -y psmisc`.

---

### Scenario 2 — Real-time monitoring with `top`

Run this scenario on both VMs.

#### Instructions

Step 1: Launch `top` and observe the output
- Run `top` and review the header and process list.
  ```bash
  top # Launch the interactive process monitor
  ```

  > The header shows uptime, number of users, load averages (1, 5, 15 min), CPU breakdown, and memory usage. The process list is sorted by CPU usage by default.

  > Key fields in the process list: `PID`, `USER`, `PR` (priority), `NI` (nice value), `VIRT` (virtual memory), `RES` (resident memory), `S` (state), `%CPU`, `%MEM`, `COMMAND`.

Step 2: Use `top` keyboard shortcuts
- While `top` is running, use the following shortcuts:
  - Press `M` to sort processes by memory usage.
  - Press `P` to sort processes by CPU usage (default).
  - Press `1` to expand per-CPU statistics.
  - Press `u`, then type `azureuser` to filter by user.
  - Press `q` to quit.

Step 3: Run `top` in batch mode for a snapshot
- Use batch mode to capture a single snapshot of process state without the interactive display.
  ```bash
  top -bn1 | head -20 # Run top in batch mode (-b), one iteration (-n1), show first 20 lines
  ```

---

### Scenario 3 — Stress-driven CPU and process inspection

This scenario restores the original Lab 04 workflow that uses `stress` (or `stress-ng` where `stress` is unavailable) to create CPU load and then inspects system behavior.

#### Instructions

Step 1: Install stress utility and start CPU stress in terminal 1
- On your first terminal session (for either VM), install one of the stress tools and start CPU workers:
  ```bash
  # Install whichever tool is available on your distro
  # Lab 04 validation results:
  # - RHEL 8.10: stress-ng available from AppStream
  # - Ubuntu 24.04: stress available from Universe
  if command -v dnf >/dev/null; then
    sudo dnf install -y stress-ng
  elif command -v apt >/dev/null; then
    sudo apt-get update && sudo DEBIAN_FRONTEND=noninteractive apt-get install -y stress
  fi

  # Run stress if present, otherwise use stress-ng
  if command -v stress >/dev/null; then
    sudo stress -c 4 # Start 4 CPU worker processes
  else
    sudo stress-ng --cpu 4 # Start 4 CPU stress workers
  fi
  ```

  > Standard_B2s has 2 vCPUs. Running 4 workers is valid for this lab (intentional oversubscription), but it can consume CPU credits quickly on burstable B-series VMs.

  > For a lower-impact run on B2s, use `--cpu 2` (or `-c 2`) instead.

  > Keep this running while you perform the next steps in terminal 2.

Step 2: Inspect CPU and memory configuration in terminal 2
- On your second terminal, run:
  ```bash
  cat /proc/cpuinfo # CPU configuration details
  cat /proc/meminfo # Memory configuration details
  free -m # RAM, buffers/cache, and swap utilization in MB
  ```

Step 3: Monitor CPU load with `top`
- Still on terminal 2:
  ```bash
  top # Observe stress workers and system utilization in real time
  ```

Step 4: Identify the stress PIDs and CPU consumers
- Note the stress process PIDs and high `%CPU` values in `top`, then confirm from the shell:
  ```bash
  ps -ef # Standard UNIX format process list
  ps aux # BSD format process list
  pgrep stress # List stress process IDs
  pgrep stress-ng # List stress-ng process IDs (if using stress-ng)
  ```

Step 5: Check files opened by a stress process
- Use one of the stress or stress-ng PIDs from `pgrep`:
  ```bash
  sudo lsof -p $(pgrep stress | head -1) # Open files for the first stress PID
  sudo lsof -p $(pgrep stress-ng | head -1) # Open files for the first stress-ng PID
  ```

Step 6: Terminate stress processes with different signals
- Kill stress processes one-by-one or by name:
  ```bash
  pgrep stress # List current stress PIDs
  pgrep stress-ng # List current stress-ng PIDs
  sudo kill <PID> # Send default SIGTERM
  sudo kill -KILL <PID> # Send SIGKILL (signal 9)
  sudo kill -15 <PID> # Send SIGTERM explicitly (signal 15)
  sudo pkill -9 stress # Force-kill all stress processes by name
  sudo pkill -9 stress-ng # Force-kill all stress-ng processes by name
  ```

  > If you run out of processes to kill, go back to terminal 1 and run `sudo stress -c 4` or `sudo stress-ng --cpu 4` again.

---

### Scenario 4 — Process priority with `nice` and `renice`

Run this scenario on both VMs.

#### Instructions

Step 1: Start a low-priority background process
- Use `nice` to launch a process with a reduced scheduling priority.
  ```bash
  nice -n 10 sleep 300 & # Start sleep with nice value +10 (lower priority); & sends it to background
  echo "PID: $!" # Print the PID of the last background process
  SLEEP_PID=$! # Save the PID for use in later steps
  ```

  > Nice values range from `-20` (highest priority) to `+19` (lowest priority). Regular users can only increase the nice value (lower the priority). Only root can set a negative nice value.

Step 2: Verify the nice value of the running process
- Confirm the process is running with the expected nice value.
  ```bash
  ps -o pid,ni,stat,comm -p $SLEEP_PID # Show PID, nice value, state, and command name for the process
  ```

Step 3: Change the priority of a running process with `renice`
- Use `renice` to adjust the nice value of the already-running process.
  ```bash
  renice -n 15 -p $SLEEP_PID # Change the nice value to +15
  ps -o pid,ni,stat,comm -p $SLEEP_PID # Verify the new nice value
  ```

Step 4: Attempt to set a higher priority as a regular user
- Try setting a negative nice value and observe the error.
  ```bash
  renice -n -5 -p $SLEEP_PID # Attempt to raise priority; will fail without root
  sudo renice -n -5 -p $SLEEP_PID # Succeed with sudo
  ps -o pid,ni,stat,comm -p $SLEEP_PID # Confirm the nice value changed to -5
  ```

---

### Scenario 5 — Background and foreground jobs

Run this scenario on both VMs.

#### Instructions

Step 1: Start multiple background jobs
- Start several background processes and list them using `jobs`.
  ```bash
  sleep 200 & # Start first background job
  sleep 300 & # Start second background job
  sleep 400 & # Start third background job
  jobs # List all current background jobs with their job numbers
  ```

  > Each job gets a job number in brackets (e.g., `[1]`, `[2]`, `[3]`). The `+` marks the most recently added job, and `-` marks the second most recent.

Step 2: Bring a background job to the foreground
- Use `fg` to resume a specific job in the foreground.
  ```bash
  jobs # Confirm current job numbers
  fg %2 # Bring job 2 to the foreground
  ```

  > The process is now running in the foreground. Press `Ctrl+Z` to suspend it and return to the shell.

Step 3: Suspend and resume a job
- Suspend the foreground job and send it back to the background.
  ```bash
  # While fg %2 is running in the foreground:
  # Press Ctrl+Z to suspend it
  jobs # Verify the job is now in Stopped state
  bg %2 # Resume the stopped job in the background
  jobs # Confirm all three jobs are running in the background again
  ```

Step 4: Wait for a background job to finish
- Use `wait` to block until a specific background job completes.
  ```bash
  sleep 5 & # Start a short sleep job
  WAIT_PID=$! # Save the PID
  wait $WAIT_PID # Wait for that specific job to finish
  echo "Job finished with exit code: $?" # Print the exit code
  ```

---

### Scenario 6 — Sending signals to processes

Run this scenario on both VMs.

#### Instructions

Step 1: List available signals
- Display all signals supported by the system.
  ```bash
  kill -l # List all signal names and numbers
  ```

  > The two most commonly used signals are `SIGTERM` (15) — a graceful termination request the process can catch and handle — and `SIGKILL` (9) — an immediate forced termination the process cannot intercept.

Step 2: Terminate a process with SIGTERM
- Start a background process and send it a graceful termination signal.
  ```bash
  sleep 500 & # Start a background sleep process
  TERM_PID=$! # Save the PID
  kill $TERM_PID # Send SIGTERM (default signal)
  jobs # Verify the job is no longer listed
  ```

Step 3: Force-kill a process with SIGKILL
- Start another background process and kill it immediately.
  ```bash
  sleep 500 & # Start another background sleep process
  KILL_PID=$! # Save the PID
  kill -9 $KILL_PID # Send SIGKILL — cannot be caught or ignored by the process
  jobs # Verify the job was killed
  ```

  > Use SIGKILL only as a last resort. It does not give the process a chance to clean up temporary files, release locks, or flush buffers.

Step 4: Kill a process by name with `pkill`
- Use `pkill` to terminate all processes matching a name pattern.
  ```bash
  sleep 100 & # Start multiple sleep processes
  sleep 200 &
  sleep 300 &
  jobs # Confirm all three are running
  pkill sleep # Kill all processes named sleep
  jobs # Verify all sleep jobs are gone
  ```

  > `pkill` matches against process names and sends SIGTERM by default. Use `pkill -9` to send SIGKILL.

Step 5: Cross-distro observation — Ubuntu-specific background processes
- On the **Ubuntu VM**, look for processes that do not exist on RHEL. Azure Ubuntu images include several daemons that are absent from the RHEL baseline.
  ```bash
  # Run on Ubuntu
  ps aux | grep -E 'multipathd|ModemManager|unattended' | grep -v grep
  ps aux | wc -l # Total process count on Ubuntu
  ```

  ```bash
  # Run on RHEL — compare the same filter
  ps aux | grep -E 'multipathd|ModemManager|unattended' | grep -v grep || echo "(none)"
  ps aux | wc -l # Total process count on RHEL
  ```

  > The Azure Ubuntu 24.04 image runs `multipathd` (multipath storage daemon), `ModemManager` (modem management), and `unattended-upgrades` (automatic security patching) by default. These are absent from the RHEL image.

  > `snapd` is installed on Azure Ubuntu images but is **inactive** by default — you will not see snap processes in `ps` output unless explicitly started. This is a deliberate hardening choice in the Azure marketplace image.

  > The total process count difference between the two VMs reflects these additional Ubuntu background services.

---

## Analytical Guidance

- **Step 1–2 (ps):** The BSD-style `ps aux` and UNIX-style `ps -ef` show the same information in different column layouts. Neither is better — the choice depends on what fields you need. Many engineers use `ps aux` for interactive work and `ps -ef` when they need the PPID.
- **Step 3 (pgrep vs grep):** Grepping `ps` output is unreliable because the `grep` process itself can appear in the results. `pgrep` is cleaner and supports extended pattern matching.
- **Scenario 2 (top):** On a nearly idle VM, CPU usage will be close to zero. The load average is a more useful signal — values consistently above the number of vCPUs indicate saturation.
- **Scenario 3 (stress workflow):** `stress` and `stress-ng` both create predictable CPU pressure so you can clearly observe scheduler behavior in `top`, `ps`, and `pgrep` output.
- **B-series VM sizing:** On Standard_B2s (2 vCPUs), 4 workers intentionally oversubscribe CPU for learning purposes. This is valid, but can burn CPU credits quickly.
- **Scenario 4 (nice/renice):** On a lightly loaded system, nice value differences have little practical effect. They matter most when multiple CPU-intensive processes compete for the same core.
- **Scenario 5 (jobs):** The `jobs` builtin is shell-scoped — it only shows jobs started in the current shell session. Background processes started in other sessions or terminals will not appear.
- **Scenario 6 (signals):** SIGKILL should be used only when SIGTERM has failed. Processes killed with SIGKILL cannot perform cleanup, which can leave temporary files, locks, or incomplete writes behind.
- **Cross-distro (Scenario 6, Step 5):** The Azure Ubuntu 24.04 image runs additional background daemons (`multipathd`, `ModemManager`, `unattended-upgrades`) not present on RHEL. `snapd` is installed but inactive on Azure Ubuntu images — this is a deliberate choice in the Azure marketplace image, not the default for all Ubuntu installs.

---

## Validation Criteria

| Step | Expected result |
|---|---|
| `ps aux` | All system processes listed with USER, PID, %CPU, %MEM, STAT, COMMAND columns |
| `pgrep sshd` | Returns one or more PIDs corresponding to the sshd process |
| `pstree -p` | Displays process hierarchy with PIDs; PID 1 is `systemd` |
| `top -bn1` | Single-iteration snapshot output captured without interactive display |
| `sudo stress -c 4` or `sudo stress-ng --cpu 4` | Four CPU worker processes start and appear as top CPU consumers |
| `pgrep stress` or `pgrep stress-ng` | Returns one or more PIDs for stress workers |
| `sudo lsof -p $(pgrep stress \| head -1)` or `sudo lsof -p $(pgrep stress-ng \| head -1)` | Open files are displayed for the selected stress process |
| `sudo pkill -9 stress` or `sudo pkill -9 stress-ng` | All stress processes are forcibly terminated |
| `nice -n 10 sleep 300 &` | Process appears in `ps -o pid,ni` with NI value of 10 |
| `renice -n 15` | `ps -o pid,ni` shows updated NI value of 15 |
| `sudo renice -n -5` | NI value changes to -5; fails without sudo |
| `jobs` after starting 3 processes | Three jobs listed with job numbers [1], [2], [3] |
| `fg %2` | Job 2 runs in foreground; `Ctrl+Z` returns it to Stopped state |
| `bg %2` | Job 2 resumes in background; `jobs` shows it as Running |
| `kill $PID` | Process removed from `jobs` output |
| `kill -9 $PID` | Process immediately terminated; `jobs` shows Killed state |
| `pkill sleep` | All sleep processes terminated; `jobs` shows no remaining sleep jobs |
| Ubuntu: `ps aux \| grep -E 'multipathd\|ModemManager\|unattended'` | `multipathd`, `ModemManager`, and `unattended-upgrades` visible on Ubuntu; absent on RHEL |

---

## Documentation Expectations

As you complete this lab, take note of:

- The difference between `ps aux` (BSD style) and `ps -ef` (UNIX style) column layouts.
- What the `STAT` column values mean and how to identify zombie or stopped processes.
- Why `pgrep` is preferred over grepping `ps` output directly.
- How to generate controlled CPU load using `stress` or `stress-ng` and verify it with `top`, `ps`, and `pgrep`.
- Why using 4 workers on a Standard_B2s (2 vCPUs) is valid for oversubscription testing, and when to use 2 workers to reduce impact.
- How to inspect process-opened files with `lsof -p`.
- The nice value range and why regular users cannot set negative values.
- The difference between SIGTERM (graceful) and SIGKILL (forced) and when to use each.
- How background and foreground job management works within a shell session.
- Why Ubuntu has additional background daemons (`multipathd`, `ModemManager`, `unattended-upgrades`) compared to RHEL, and why `snapd` is inactive on Azure Ubuntu marketplace images.

---

## What Not To Do

- Do not run `kill -9 1` or attempt to kill the `systemd` process — this will terminate the VM session.
- Do not run `pkill -9 sshd` — this will terminate your active SSH session and lock you out of the VM.
- Do not skip the `renice` steps — observing the permission boundary for regular users vs. root is part of the learning objective.
- Do not delete the VMs after this lab — they may be reused in subsequent labs.

---

## Real-World Context

Process management is a core skill in Linux system administration and troubleshooting. In production environments, engineers use `ps` and `top` to identify runaway processes consuming excessive CPU or memory, `kill` to gracefully terminate hung applications before escalating to SIGKILL, and `nice`/`renice` to prevent batch jobs from starving interactive workloads. Understanding how signals work — and the difference between SIGTERM and SIGKILL — is critical when handling application failures on live systems. The cross-distro comparison in this lab reflects real-world environments where RHEL and Ubuntu VMs coexist and administrators must be fluent on both.

---

## Optional Advanced Exploration

- Use `top` with a delay interval and output to a file for lightweight process logging:
  ```bash
  top -bd 2 -n 5 > /tmp/top-snapshot.txt # Capture 5 iterations with 2-second delay
  cat /tmp/top-snapshot.txt
  ```
- Use `ps` with custom output columns to build a minimal process report:
  ```bash
  ps -eo pid,ppid,user,ni,stat,pcpu,pmem,comm --sort=-pcpu | head -20 # Sort by CPU usage descending
  ```
- Install and use `htop` for an enhanced interactive process view:
  ```bash
  # RHEL
  sudo dnf install -y htop
  # Ubuntu
  sudo apt install -y htop
  htop # Launch htop; press F10 or q to quit
  ```
- Use `lsof` to list open files for a specific process:
  ```bash
  sudo lsof -p $(pgrep sshd | head -1) # List all files opened by the first sshd process
  ```
- Use `/proc` to inspect process details directly from the kernel:
  ```bash
  cat /proc/$(pgrep sshd | head -1)/status # View process status from the proc filesystem
  ls /proc/$(pgrep sshd | head -1)/fd | wc -l # Count open file descriptors
  ```
