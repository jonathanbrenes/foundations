# Lab 7 — Managing Services in Linux

## Title and Scenario

You are a Linux administrator responsible for maintaining system services on a production server. A recent deployment has introduced several new services, and you need to verify their status, troubleshoot any failures, and ensure critical services are correctly enabled for boot. Understanding `systemctl` and the systemd service manager is essential for day-to-day Linux operations and incident response.

- **Estimated time:** 30 minutes.
- **VM count:** 1 (RHEL 10).

---

## Deployment

[![Click to deploy](https://aka.ms/deploytoazurebutton)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fjonathanbrenes%2Ffoundations%2Fmain%2FModule%25202%2520-%2520OS%2520Fundamentals%2520II%2FLabs%2FFoundationsLab07.json)

---

## Skills Required

- Basic file system navigation (Labs 01-03)
- Know how to create an SSH connection
- Familiarity with the Linux command line

---

## Recommended Prerequisites

- Completion of Labs 01-06
- SSH access to the lab07rhel VM
- Understanding of Linux processes (Lab 04)

---

## Objectives

After completing this lab, you will be able to:

1. **List and inspect systemd units** – Use `systemctl` to view all loaded units and their states
2. **Filter units by type and state** – Isolate services, active units, and running processes
3. **Investigate failed services** – Identify and troubleshoot units in a failed or maintenance state
4. **Manage service lifecycle** – Start, stop, restart, and check the status of services
5. **Control boot behavior** – Enable and disable services at system startup

---

## Environment Overview

| Component | Value |
|---|---|
| **VM Name** | lab07rhel |
| **Operating System** | Red Hat Enterprise Linux (RHEL) 10 |
| **VM Size** | Standard_B2s (2 vCPU, 4 GiB RAM) |
| **Admin Username** | azureuser |
| **Primary Network Interface** | lab07rhel-nic1 (10.1.0.10) |
| **Public FQDN** | Assigned via Azure Portal |
| **Default Shell** | /bin/bash |

---

## Your Mission

Your mission is to gain hands-on experience managing Linux services using systemd. You will inspect the current state of all system units, identify and investigate failed services, practice starting and stopping services, and configure boot-time service behavior. These are core skills for any Linux administrator managing Azure VMs.

---

### Scenario 1 — Inspecting Systemd Units

In this scenario, you will explore the systemd unit hierarchy and learn to filter units by type and state.

**Deployment:** Uses the RHEL 10 VM from the [Deployment](#deployment) section.

#### Instructions

Step 1: List all systemd units
- List the state of all systemd units currently loaded on the system.
  ```bash
  systemctl
  ```

  > **What to observe:** The output shows all loaded units organized by type (service, socket, target, timer, etc.) with their LOAD, ACTIVE, and SUB states.

Step 2: List only service units
- Filter the output to show only service-type units.
  ```bash
  systemctl list-units --type=service --all
  ```

  > **What to observe:** Notice the different states — some services are `active (running)`, others are `inactive (dead)`, and some may be `failed`. The `--all` flag includes inactive units that would otherwise be hidden.

Step 3: List active units
- Display only units that are currently in an active state.
  ```bash
  systemctl list-units --state=active
  ```

Step 4: List running services
- Filter further to show only services that are currently running.
  ```bash
  systemctl list-units --type=service --state=running
  ```

  > **What to observe:** These are the services actively executing on the system right now. Compare this shorter list to the full service list from Step 2.

Step 5: View unit file enable/disable status
- View the enabled and disabled settings for all unit files.
  ```bash
  systemctl list-unit-files
  ```

  > **What to observe:** The STATE column shows whether each unit is `enabled`, `disabled`, `static`, or `masked`. Enabled units start automatically at boot; disabled units do not.

---

### Scenario 2 — Investigating Failed Services

In this scenario, you will identify and troubleshoot services that have failed to start or are in a maintenance state.

**Deployment:** Uses the same RHEL 10 VM from Scenario 1.

#### Instructions

Step 1: List failed services
- View only the services that are in a failed state.
  ```bash
  systemctl --failed --type=service
  ```

  > **What to observe:** Note any services listed here. Failed services can indicate configuration issues, missing dependencies, or startup errors.

Step 2: Investigate a failed unit
- Pick a failed service from the previous output (or use the example below) and check its detailed status.
  ```bash
  systemctl status <service-name>.service
  ```

  > Replace `<service-name>` with an actual failed service from Step 1. The status output shows the service description, load state, active state, recent log entries, and the process exit code — all critical information for troubleshooting.

Step 3: Check service logs with journalctl
- For deeper investigation, review the journal logs for the failed service.
  ```bash
  journalctl -u <service-name>.service --no-pager -n 30
  ```

  > **What to observe:** The journal provides timestamped log entries that show exactly what happened during the service startup attempt. Look for error messages indicating the root cause.

---

### Scenario 3 — Managing Service Lifecycle

In this scenario, you will practice starting, stopping, and restarting services — the most common service management operations.

**Deployment:** Uses the same RHEL 10 VM from Scenario 1.

#### Instructions

Step 1: Check the status of the waagent service
- The Azure Linux Agent (`waagent`) is a critical service on Azure VMs. Check its current status.
  ```bash
  systemctl status waagent.service
  ```

  > **What to observe:** The output shows whether the service is active, its PID, memory usage, and recent log lines.

Step 2: Stop the waagent service
- Stop the service and verify it is no longer running.
  ```bash
  sudo systemctl stop waagent.service
  systemctl status waagent.service
  ```

  > **What to observe:** The service state changes to `inactive (dead)`. The service process is no longer running.

Step 3: Start the waagent service
- Start the service again and confirm it is active.
  ```bash
  sudo systemctl start waagent.service
  systemctl status waagent.service
  ```

  > **What to observe:** The service returns to `active (running)` with a new PID.

Step 4: Restart the waagent service
- Restart combines stop and start into a single operation — useful after configuration changes.
  ```bash
  sudo systemctl restart waagent.service
  systemctl status waagent.service
  ```

---

### Scenario 4 — Controlling Boot Behavior

In this scenario, you will enable and disable services to control whether they start automatically at system boot.

**Deployment:** Uses the same RHEL 10 VM from Scenario 1.

#### Instructions

Step 1: Check if waagent is enabled at boot
- Determine whether the waagent service is configured to start at boot.
  ```bash
  systemctl list-unit-files | grep waagent
  ```

  > **What to observe:** The STATE column shows `enabled` if the service starts at boot.

Step 2: Disable waagent at boot
- Disable the service from starting automatically at the next boot.
  ```bash
  sudo systemctl disable waagent.service
  systemctl list-unit-files | grep waagent
  ```

  > **What to observe:** The state changes to `disabled`. The service will not start automatically at the next boot, but it is still running in the current session.

Step 3: Re-enable waagent at boot
- Re-enable the service so it starts automatically at boot.
  ```bash
  sudo systemctl enable waagent.service
  systemctl list-unit-files | grep waagent
  ```

  > **What to observe:** The state returns to `enabled`. Always re-enable critical services like waagent to ensure your Azure VM remains manageable after reboot.

  > **Important:** Never leave the Azure Linux Agent (`waagent`) disabled on a production Azure VM. It is essential for VM provisioning, extension handling, and platform communication.

---

## Analytical Guidance

### Systemd Unit States

| State | Meaning |
|---|---|
| `loaded` | Unit file has been parsed and loaded into memory |
| `active (running)` | Service process is running |
| `active (exited)` | Service ran and exited successfully (oneshot) |
| `inactive (dead)` | Service is not running |
| `failed` | Service attempted to start but failed |
| `enabled` | Service starts automatically at boot |
| `disabled` | Service does not start at boot |
| `static` | Unit cannot be enabled/disabled; only started manually or as a dependency |
| `masked` | Unit is completely blocked from starting |

### Key systemctl Commands

| Command | Purpose |
|---|---|
| `systemctl` | List all loaded units |
| `systemctl list-units --type=service` | Filter by service type |
| `systemctl list-units --state=running` | Filter by running state |
| `systemctl list-unit-files` | Show enable/disable state |
| `systemctl --failed` | Show failed units |
| `systemctl status <unit>` | Detailed status of a unit |
| `systemctl start/stop/restart <unit>` | Control service lifecycle |
| `systemctl enable/disable <unit>` | Control boot behavior |
| `journalctl -u <unit>` | View logs for a unit |

### Troubleshooting Flow

1. **Identify** — Use `systemctl --failed` to find failed services
2. **Investigate** — Use `systemctl status <service>` to read status and recent logs
3. **Deep dive** — Use `journalctl -u <service>` for full log history
4. **Resolve** — Fix configuration or dependencies
5. **Restart** — Use `systemctl restart <service>` to apply changes
6. **Verify** — Check `systemctl status <service>` to confirm recovery

---

## Validation Criteria

| Step | Expected Result |
|---|---|
| `systemctl` | Lists all loaded units with LOAD, ACTIVE, SUB columns |
| `systemctl list-units --type=service --all` | Filters output to service units only |
| `systemctl list-units --state=active` | Shows only active units |
| `systemctl list-units --type=service --state=running` | Shows subset of actively running services |
| `systemctl list-unit-files` | Shows enabled/disabled/static state for each unit file |
| `systemctl --failed --type=service` | Lists any failed services (may be empty on clean system) |
| `systemctl status waagent.service` | Shows active (running), PID, memory, log lines |
| `systemctl stop waagent.service` | Service state changes to inactive (dead) |
| `systemctl start waagent.service` | Service returns to active (running) with new PID |
| `systemctl restart waagent.service` | Service restarted with new PID |
| `systemctl disable waagent.service` | Unit file state changes to disabled |
| `systemctl enable waagent.service` | Unit file state returns to enabled |

---

## Documentation Expectations

As you complete this lab, take note of:

- The difference between `active (running)` and `active (exited)` service states.
- How many services are running on a default RHEL 10 Azure VM.
- Which services are `enabled` vs `disabled` at boot and why.
- The information available in `systemctl status` output (PID, memory, CGroup, logs).
- The difference between `stop` (current session only) and `disable` (affects boot behavior).
- How `journalctl -u` provides deeper troubleshooting context than `systemctl status`.

---

## What Not To Do

- ❌ **Don't leave waagent disabled on production Azure VMs** – It is required for VM management, extension handling, and platform communication.
- ❌ **Don't stop sshd on a remote VM** – You will lose your SSH connection with no way to reconnect except through the Azure Serial Console.
- ❌ **Don't mask services you don't understand** – Masked services cannot be started even manually; use `disable` instead for softer control.
- ❌ **Don't restart services in production without understanding dependencies** – Some services have cascading restarts that affect dependent services.
- ❌ **Don't ignore failed services** – They may indicate configuration drift, missing packages, or startup errors that affect system health.

---

## Real-World Context

Service management is one of the most frequent tasks in Linux administration. In Azure support scenarios, engineers regularly need to:

- **Verify Azure Linux Agent status** after a VM fails to report health or extensions fail to run. The first step is always `systemctl status waagent.service`.
- **Restart services after configuration changes** — changes to `/etc/waagent.conf`, `/etc/ssh/sshd_config`, or `/etc/httpd/httpd.conf` do not take effect until the service is restarted.
- **Troubleshoot boot failures** — if a service is `enabled` but `failed`, reviewing `journalctl -u <service>` logs is the primary diagnostic path.
- **Audit running services** for security compliance — listing all `enabled` services helps identify unexpected or unauthorized services on a system.

The `systemctl enable/disable` and `start/stop` distinction is critical: `enable/disable` controls boot behavior while `start/stop` controls the current session. A service can be `enabled` (starts at next boot) but currently `stopped`, or `disabled` (won't start at boot) but currently `running`.

---

## Optional Advanced Exploration

- Use `systemctl show <service>` to display all properties of a unit (restart policy, limits, dependencies):
  ```bash
  systemctl show waagent.service
  ```
- Use `systemctl list-dependencies <service>` to see the dependency tree of a service:
  ```bash
  systemctl list-dependencies waagent.service
  ```
- Use `systemctl cat <service>` to view the unit file contents:
  ```bash
  systemctl cat waagent.service
  ```
- Explore `systemctl list-timers` to see scheduled timer-based services (systemd equivalent of cron):
  ```bash
  systemctl list-timers --all
  ```
- Use `journalctl -b` to view all logs since the last boot and correlate with service startup order:
  ```bash
  journalctl -b --no-pager | head -50
  ```
