# Module 4 - Linux on Azure

## Module Overview

- **Training day:** Day 2
- **Planned duration:** 4 hours
- **Time breakdown:** ~2h reading and concept primer | ~1.5h lab execution including deployment (Lab 09) | ~30m knowledge check and completion criteria
- **Required VM(s):** RHEL 9, Ubuntu 24.04, SLES 15 SP7
- **Prior module required:** Module 3 - Package Management

## Learning Objectives

- Identify Linux VM integration components used in Azure.
- Differentiate responsibilities of waagent, cloud-init, and extensions.
- Perform basic VM extension operations for access recovery.
- Understand resource disk usage and practical constraints.

## Key Topics

- Linux on Azure architecture and deployment patterns.
- Key platform integration components.
- WALinuxAgent behavior and capabilities.
- Linux Integration Services overview.
- cloud-init provisioning model.
- VM extensions and VMAccessForLinux scenarios.
- Resource disk characteristics and operational practices.

## Command Highlights

- `waagent --version`, `waagent --show-configuration`
- `systemctl status waagent`, `systemctl status walinuxagent`
- `grep -i provisioning /etc/waagent.conf`
- `cat /var/log/waagent.log`, `cat /var/log/cloud-init.log`
- `ls /var/log/azure/`, `cat /var/log/azure/<extension>/extension.log`
- `swapon -s`, `free -m`, `df -h`

## Labs

### Lab 09 — Linux on Azure
- **Duration:** ~60 minutes
- **VMs:** RHEL 9, Ubuntu 24.04, SLES 15 SP7 (FoundationsLab09.json)
- **Objectives:**
  - Identify the active provisioning agent (waagent or cloud-init)
  - Locate and validate waagent and cloud-init logs
  - Check Azure Linux Agent version and service status
  - Perform a password reset via the Azure Portal and verify the VMAccess extension
  - Configure and manage swap space using the modern per-boot script method
- **Key Commands:** `waagent --version`, `systemctl status waagent`, `grep provisioning /etc/waagent.conf`, `swapon -s`, `free -m`
