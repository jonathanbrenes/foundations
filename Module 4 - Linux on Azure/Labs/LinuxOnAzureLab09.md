# Lab 9 — Linux on Azure

## Title and Scenario

This lab covers the core components that enable Linux virtual machines to operate within the Azure platform. You will work with the Azure Linux VM Agent (waagent), cloud-init provisioning, VM extensions, and swap management on the ephemeral disk. Understanding these components is essential for troubleshooting Linux VMs in Azure.

Each scenario uses dedicated VMs deployed from the templates provided below.

- **Estimated time:** 40 minutes.
- **VM count:** 3 (RHEL 9, Ubuntu 24.04, SLES 15 SP7).

---

## Deployment

> **Warning:** The VMs deployed in these labs allow inbound traffic automatically from the Azure VPN (AzureCloud service tag). Make sure you are connected to the Azure VPN, or add an NSG rule to allow your public IP address before attempting to connect.

### RHEL 9 VM

[![Click to deploy](https://aka.ms/deploytoazurebutton)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fjonathanbrenes%2Ffoundations%2Fmain%2FModule%25204%2520-%2520Linux%2520on%2520Azure%2FLabs%2FFoundationsLab09RHEL.json)

### Ubuntu 24.04 VM

[![Click to deploy](https://aka.ms/deploytoazurebutton)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fjonathanbrenes%2Ffoundations%2Fmain%2FModule%25204%2520-%2520Linux%2520on%2520Azure%2FLabs%2FFoundationsLab09Ubuntu.json)

### SLES 15 SP7 VM

[![Click to deploy](https://aka.ms/deploytoazurebutton)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fjonathanbrenes%2Ffoundations%2Fmain%2FModule%25204%2520-%2520Linux%2520on%2520Azure%2FLabs%2FFoundationsLab09SLES.json)

---

## Skills Required

- Ability to connect to a Linux VM via SSH.
- Basic familiarity with running commands as root using `sudo`.
- Understanding of `systemctl` for managing services.

---

## Recommended Prerequisites

- Lab 1 completed — SSH connectivity and Azure CLI familiarity.
- Lab 8 completed — package management across distributions.

---

## Objectives

After completing this lab, you will be able to:

- Identify whether the provisioning agent is `waagent` or `cloud-init`.
- Locate and validate the waagent and provisioning logs.
- Check the version and status of the Azure Linux VM Agent.
- Identify important files related to waagent and extensions.
- Perform a password reset through the Azure Portal and verify the VMAccess extension.
- Understand how the Azure Linux VM Agent creates a swap file on the ephemeral disk.
- Validate if the system has swap space available and enable or disable it through `waagent.conf`.

---

## Environment Overview

| Component | Details |
|---|---|
| RHEL VM | RHEL 9 (`lab09rhel`) — Scenarios 1, 2, and 3 |
| Ubuntu VM | Ubuntu 24.04 LTS (`lab09ubuntu`) — Scenarios 1, 2, and 3 |
| SLES VM | SLES 15 SP7 (`lab09sles`) — Scenarios 1, 2, and 3 |
| VM size | Standard_D2s_v3 |
| Access | SSH as `azureuser` with password authentication |

---

## Your Mission

Complete each scenario on the corresponding VMs. Deploy the VMs from the Deployment section. Scenarios 1 and 2 should be performed on both the RHEL and Ubuntu VMs to compare behavior across distributions. Scenario 3 focuses on swap management and can be performed on any of the three VMs.

### Scenario 1 — Provisioning Agent in a Linux VM

Since Linux was first supported in Azure, `waagent` handled provisioning. As part of providing more flexibility, `cloud-init` was introduced and is replacing `waagent` as the provisioning agent in recent Linux distributions. `cloud-init` allows you to pass custom values to a virtual machine during provisioning apart from the parameters provided by the Azure platform. However, in some distributions the provisioning is still handled by `waagent`.

**Deployment:** See the [Deployment](#deployment) section (RHEL 9 VM and Ubuntu 24.04 VM).

#### Instructions

Step 1: Connect and switch to root
- Connect to the VM and switch to root. Repeat each step on both the RHEL and Ubuntu VMs.
  ```bash
  ssh azureuser@<VM_IP> # Connect to the VM using SSH
  sudo -i # Switch to root for full administrative privileges
  ```

Step 2: Check the provisioning agent
- Check the WALinux Agent configuration file to determine if `waagent` is the provisioning agent.
  ```bash
  grep -i provisioning /etc/waagent.conf # Filter the waagent config file for provisioning-related settings
  ```
  Inspect the output visually. If `Provisioning.Agent` is set to `auto` or `waagent`, the agent handles provisioning. If it is set to `cloud-init` or provisioning is disabled, `cloud-init` is the provisioning agent.

Step 3: Check the cloud-init log file
- If Step 2 indicates `cloud-init` is the provisioning agent, verify that the cloud-init log files exist. They should be created on first boot.
  ```bash
  ls -l /var/log/cloud-init* # List cloud-init log files; their timestamps indicate when provisioning occurred
  ```

Step 4: Check waagent provisioning logs
- In case the provisioning setting says `auto`, filter the waagent log for provisioning-related entries.
  ```bash
  grep -i provisioning /var/log/waagent.log # Search for provisioning events in the waagent log
  ```

### Scenario 2 — WAAgent and Extensions

The Azure Linux VM Agent (`waagent`) manages the lifecycle of extensions, reports health status, and handles platform-level operations. In this scenario you will identify the agent version, check its service status, review its logs, and observe how extensions are deployed.

**Deployment:** Uses the same RHEL 9 and Ubuntu 24.04 VMs from Scenario 1.

#### Instructions

Step 1: Connect to the VMs
- Continue using the RHEL and Ubuntu VMs. If disconnected, reconnect and switch to root.
  ```bash
  sudo -i # Switch to root for full administrative privileges
  ```

Step 2: Check the Linux Agent version
- Get the version of the Azure Linux Agent, including the goal state agent version.
  ```bash
  waagent --version # Display the waagent version and the goal state agent (GAAgent) version
  ```

Step 3: Check the Linux Agent package version
- Verify the installed package version using the distribution-specific package manager.

  On RHEL:
  ```bash
  rpm -qa | grep -i walinuxagent # Query RPM database for the walinuxagent package
  ```

  On Ubuntu:
  ```bash
  dpkg -l | grep walinuxagent # Query dpkg database for the walinuxagent package
  ```

Step 4: Check the status of the Azure Linux Agent systemd unit
- Check the service status using the distribution-specific service name.

  On RHEL:
  ```bash
  systemctl status waagent # Check the waagent service status on RHEL
  ```

  On Ubuntu:
  ```bash
  systemctl status walinuxagent # Check the walinuxagent service status on Ubuntu
  ```

Step 5: Check the Linux Agent logs
- Review the waagent log file.
  ```bash
  cat /var/log/waagent.log # Display the full waagent log; use less or tail for large files
  ```

  > For large log files, consider using `tail -100 /var/log/waagent.log` or `less /var/log/waagent.log` to navigate the output.

Step 6: Check the Linux Agent configuration file
- Inspect the waagent configuration file to review current settings.
  ```bash
  cat /etc/waagent.conf # Display the waagent configuration file with all current settings
  ```

Step 7: Password reset using the Azure Portal
- In the Azure Portal, navigate to the VM and perform a password reset:
  1. Go to **Virtual machines** and select the VM.
  2. Under **Help**, select **Reset password**.
  3. Enter a new username or select the existing `azureuser`, set a new password, and click **Update**.
  4. The operation deploys the `VMAccessForLinux` extension to execute the password reset.

Step 8: Check the VMAccess extension
- After the password reset operation completes in the portal, verify the extension status:
  1. In the Azure Portal, navigate to **Virtual machines** > select your VM > **Extensions + applications**.
  2. Confirm that `VMAccessForLinux` appears and shows a successful status.
  3. Log in to the VM with the new password to verify it works.

Step 9: Check the logs for the extension and the Linux Agent
- Extension logs are located under `/var/log/azure`. There is a directory for each extension deployed. Review the extension log for the VMAccess extension.
  ```bash
  cd /var/log/azure # Navigate to the Azure extensions log directory
  ls -l # List all extension directories
  cat Microsoft.OSTCExtensions.VMAccessForLinux/extension.log # Display the VMAccess extension log
  ```

  > Extension handlers are stored under `/var/lib/waagent/`. Log files for each extension are under `/var/log/azure/<ExtensionPublisher.ExtensionName>/`.

### Scenario 3 — Swap Management on Azure Linux VMs

In Azure VMs, a temporary (ephemeral) disk — usually mounted at `/mnt/resource` (or `/mnt` on some distributions) — is recommended for temporary storage including swap files. Historically, the Azure Linux Agent (`waagent`) managed swap creation through its configuration file. However, in newer distributions where `cloud-init` is the provisioning agent, swap management through `waagent.conf` is no longer supported. The recommended approach is to create the swap file using a script or `cloud-init` configuration instead.

This scenario covers both methods:
- **Legacy method (Part A):** Using `waagent.conf` — applicable to older distributions where `waagent` handles provisioning.
- **Modern method (Part B):** Using a script on the ephemeral disk — applicable to newer distributions provisioned by `cloud-init`. This is the method documented by Microsoft: [Create a SWAP partition for an Azure Linux VM](https://learn.microsoft.com/en-us/troubleshoot/azure/virtual-machines/linux/create-swap-file-linux-vm).

**Deployment:** Uses any of the VMs from Scenario 1 (RHEL, Ubuntu, or SLES).

#### Instructions

Step 1: Connect to the VM
- Connect to a VM and switch to root.
  ```bash
  ssh azureuser@<VM_IP> # Connect to the VM using SSH
  sudo -i # Switch to root for full administrative privileges
  ```

Step 2: Check the available memory and swap on the system
- Check the system status related to swap and memory.
  ```bash
  swapon -s # Display a summary of active swap devices; empty output means no swap is configured
  free -m # Display memory and swap usage in megabytes
  ```

  > If `swapon -s` returns no output, no swap file exists. The swap values in `free -m` should match.

Step 3: Determine the provisioning agent
- Before choosing which method to use, verify whether `waagent` or `cloud-init` is the provisioning agent (as done in Scenario 1).
  ```bash
  grep -i "Provisioning.Agent" /etc/waagent.conf # Check which agent handles provisioning
  ```
  - If `waagent` manages provisioning, you may use **Part A** (legacy method).
  - If `cloud-init` manages provisioning (which is the case for RHEL 9, Ubuntu 24.04, and SLES 15 SP7), use **Part B** (modern method).

#### Part A — Legacy method: Enable swap through waagent.conf

> This method applies to older distributions where `waagent` is the provisioning agent. On newer distributions using `cloud-init`, the `ResourceDisk.EnableSwap` setting in `waagent.conf` is ignored. This part is included for reference, as you may encounter it on older VMs in production.

Step 4A: Enable swap via waagent.conf
- Update the `/etc/waagent.conf` file with the following parameters.
  ```bash
  vi /etc/waagent.conf # Open the waagent configuration file for editing
  ```
  Set the following parameters:
  ```bash
  ResourceDisk.Format=y
  ResourceDisk.EnableSwap=y
  ResourceDisk.SwapSizeMB=4096
  ```

  > Make sure the swap size is not larger than the available space on the ephemeral disk. Verify with `df -h`.
  ```bash
  df -h # Display filesystem usage; check the size of /mnt/resource (or /mnt)
  ```

Step 5A: Restart the VM Agent
- Restart the waagent service to apply the configuration change. The service name varies by distribution.

  On Debian-based distributions (Ubuntu):
  ```bash
  systemctl restart walinuxagent.service # Restart the walinuxagent service on Ubuntu
  ```

  On RHEL-based distributions and SUSE:
  ```bash
  systemctl restart waagent.service # Restart the waagent service on RHEL or SLES
  ```

Step 6A: Validate that the swap file was created
- Confirm the swap space is now active and matches the configured size.
  ```bash
  swapon -s # Verify the swap file is active and shows the expected size
  free -m # Confirm total swap matches the configured SwapSizeMB value
  ```

Step 7A: Disable swap via waagent.conf
- To disable swap, update `/etc/waagent.conf` with the following parameters.
  ```bash
  vi /etc/waagent.conf # Open the waagent configuration file for editing
  ```
  Set the following parameters:
  ```bash
  ResourceDisk.Format=n
  ResourceDisk.EnableSwap=n
  ```

Step 8A: Reboot and validate
- Reboot the VM and confirm swap is no longer active.
  ```bash
  reboot # Restart the VM to apply the swap configuration change
  ```
  After reconnecting:
  ```bash
  sudo -i # Switch to root for full administrative privileges
  swapon -s # Verify no swap devices are active (should return empty)
  free -m # Confirm swap total is 0
  ```

#### Part B — Modern method: Enable swap using a script on the ephemeral disk

> This is the recommended method for newer distributions where `cloud-init` is the provisioning agent. It follows the procedure documented by Microsoft: [Create a SWAP partition for an Azure Linux VM](https://learn.microsoft.com/en-us/troubleshoot/azure/virtual-machines/linux/create-swap-file-linux-vm).

Step 4B: Disable swap in waagent.conf
- First, ensure that `waagent.conf` is not configured to manage swap, since `cloud-init` now handles provisioning.
  ```bash
  vi /etc/waagent.conf # Open the waagent configuration file for editing
  ```
  Verify or set the following parameters:
  ```bash
  ResourceDisk.Format=n
  ResourceDisk.EnableSwap=n
  ResourceDisk.MountPoint=/mnt
  ResourceDisk.SwapSizeMB=0
  ```

Step 5B: Restart the VM Agent
- Restart the waagent service after the configuration change.

  On Debian-based distributions (Ubuntu):
  ```bash
  systemctl restart walinuxagent.service # Restart the walinuxagent service on Ubuntu
  ```

  On RHEL-based distributions and SUSE:
  ```bash
  systemctl restart waagent.service # Restart the waagent service on RHEL or SLES
  ```

Step 6B: Create a swap script
- Create a script that will run on every boot to set up the swap file on the ephemeral disk.
  ```bash
  cat > /var/lib/cloud/scripts/per-boot/create_swapfile.sh << 'EOF'
  ```
  Paste the following content into the file:
  ```text
  #!/bin/sh

  # Size of the swap file in megabytes
  SWAP_SIZE_MB=2048
  SWAP_FILE="/mnt/swapfile"

  # Only create swap if it doesn't already exist
  if [ ! -f "${SWAP_FILE}" ]; then
      fallocate -l ${SWAP_SIZE_MB}M "${SWAP_FILE}"
      chmod 600 "${SWAP_FILE}"
      mkswap "${SWAP_FILE}"
  fi

  # Enable swap if not already active
  if ! swapon -s | grep -q "${SWAP_FILE}"; then
      swapon "${SWAP_FILE}"
  fi
  EOF
  ```

  > Alternatively, use `vi` or `nano` to create the file at `/var/lib/cloud/scripts/per-boot/create_swapfile.sh` with the content above.

Step 7B: Make the script executable and run it
- Set the execute permission and run the script to enable swap immediately.
  ```bash
  chmod +x /var/lib/cloud/scripts/per-boot/create_swapfile.sh # Make the script executable
  /var/lib/cloud/scripts/per-boot/create_swapfile.sh # Run the script to create and enable swap now
  ```

Step 8B: Validate that the swap file was created
- Confirm the swap space is now active and matches the configured size.
  ```bash
  swapon -s # Verify the swap file is active and shows the expected size (2048 MB)
  free -m # Confirm total swap matches the configured size
  ```

Step 9B: Verify swap persists after reboot
- Reboot the VM and confirm the swap file is recreated automatically by the per-boot script.
  ```bash
  reboot # Restart the VM to verify swap is recreated on boot
  ```
  After reconnecting:
  ```bash
  sudo -i # Switch to root for full administrative privileges
  swapon -s # Verify the swap file is active after reboot
  free -m # Confirm swap total matches the configured size
  ```

  > The script runs on every boot via `cloud-init`'s per-boot mechanism (`/var/lib/cloud/scripts/per-boot/`). This ensures swap is recreated even after the ephemeral disk is wiped during VM deallocation or host maintenance.

Step 10B: Disable swap (remove the script)
- To disable swap, remove the per-boot script and turn off swap.
  ```bash
  swapoff /mnt/swapfile # Deactivate the swap file
  rm /mnt/swapfile # Remove the swap file from the ephemeral disk
  rm /var/lib/cloud/scripts/per-boot/create_swapfile.sh # Remove the per-boot script so swap is not recreated
  ```

Step 11B: Validate that swap is disabled
- Confirm swap is no longer active.
  ```bash
  swapon -s # Verify no swap devices are active (should return empty)
  free -m # Confirm swap total is 0
  ```

---

## Analytical Guidance

- **Scenario 1:** Identifying the provisioning agent is a foundational troubleshooting step. Many provisioning failures stem from confusion about whether `waagent` or `cloud-init` handles initial setup. The `Provisioning.Agent` setting in `/etc/waagent.conf` is the authoritative source.
- **Scenario 2:** The Azure Linux Agent is the bridge between the Azure platform and the guest OS. Its health directly affects extension deployment, password resets, and VM health reporting. Knowing where logs live (`/var/log/waagent.log`, `/var/log/azure/`) is critical for diagnosing platform-level issues.
- **Scenario 3:** Swap management on Azure VMs is distribution-dependent and frequently misconfigured. Historically, `waagent.conf` managed swap creation, but newer distributions using `cloud-init` for provisioning ignore those settings. The modern approach uses a per-boot script placed in `/var/lib/cloud/scripts/per-boot/` to recreate the swap file on every boot. Using the ephemeral disk for swap is the recommended approach because it avoids consuming persistent storage. However, the ephemeral disk is not persistent — data is lost during VM deallocation or host maintenance, which is why the per-boot script pattern is necessary.

---

## Validation Criteria

| Step | Expected result |
|---|---|
| `grep -i provisioning /etc/waagent.conf` | Returns provisioning configuration lines showing the active agent |
| `ls -l /var/log/cloud-init*` | Log files exist with timestamps matching the VM creation time |
| `waagent --version` | Returns the installed waagent version and the goal state agent version |
| `rpm -qa \| grep walinuxagent` (RHEL) | Returns the installed package name and version |
| `dpkg -l \| grep walinuxagent` (Ubuntu) | Returns the installed package with status `ii` (installed) |
| `systemctl status waagent` / `walinuxagent` | Service is active and running |
| VMAccess extension in portal | Shows successful provisioning status after password reset |
| `cat /var/log/azure/.../extension.log` | Extension log shows the password reset operation completed |
| `swapon -s` after enabling swap (Part A) | Shows the swap file with the configured size |
| `free -m` after enabling swap (Part A) | Swap total matches `SwapSizeMB` value from `waagent.conf` |
| `swapon -s` after enabling swap (Part B) | Shows `/mnt/swapfile` with the configured size |
| `free -m` after enabling swap (Part B) | Swap total matches the size set in the script |
| `swapon -s` after reboot (Part B) | Swap file recreated automatically by the per-boot script |
| `swapon -s` after disabling swap | Returns empty output (no active swap) |
| `free -m` after disabling swap | Swap total is 0 |

---

## Documentation Expectations

As you complete this lab, take note of:

- Which provisioning agent is active on each distribution and how to identify it.
- The difference between the `waagent` package version and the goal state agent version.
- The different service names across distributions (`waagent` on RHEL/SLES vs. `walinuxagent` on Ubuntu).
- Where extension logs and handlers are stored on the filesystem.
- The relationship between the ephemeral disk and swap file management.
- Why the ephemeral disk is appropriate for swap (performance) but not for persistent data (data loss on deallocation).

---

## What Not To Do

- Do not delete or modify `/var/lib/waagent/` contents directly — this directory contains extension handlers and agent state.
- Do not set swap size larger than the available ephemeral disk space.
- Do not stop the waagent service and leave it stopped — the Azure platform relies on it for health reporting and extension management.
- Do not apply these swap configuration changes to production VMs without understanding the ephemeral disk lifecycle.
- Do not delete the VMs until all Module 4 labs are complete.

---

## Real-World Context

The Azure Linux VM Agent is at the center of most platform-level interactions with a Linux VM. In Azure support, engineers frequently troubleshoot issues such as: extensions failing to deploy (often caused by a stalled or crashed waagent), VMs showing as "not ready" in the portal (health reporting failure), and password resets not taking effect (VMAccess extension failures). Understanding the agent's log locations, service status, and configuration file is the first step in any platform-level Linux troubleshooting workflow. Swap management is another common support topic — customers either run out of memory because swap was never configured, or experience data loss because they placed persistent data on the ephemeral disk.

---

## Optional Advanced Exploration

- Use `cloud-init query` to inspect the metadata and userdata that was passed during provisioning:
  ```bash
  cloud-init query --all # Display all instance metadata, userdata, and vendor data
  ```
- Check the goal state agent logs separately from the main waagent log:
  ```bash
  ls /var/log/waagent* # List all waagent-related log files including goal state agent logs
  ```
- Use `waagent --show-configuration` to display the effective configuration (if supported by your agent version):
  ```bash
  waagent --show-configuration # Display the merged effective configuration from all sources
  ```
- Explore the extension handler directories under `/var/lib/waagent/` to understand the extension lifecycle:
  ```bash
  ls /var/lib/waagent/ | grep -i Microsoft # List all Microsoft extension handler directories
  ```
