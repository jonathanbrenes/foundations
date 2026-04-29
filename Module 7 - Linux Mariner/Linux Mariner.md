# Module 7 - Azure Linux

## Module Overview

- **Training day:** Day 4
- **Planned duration:** 3 hours
- **Time breakdown:** ~1h reading and concept primer | ~1.5h lab execution | ~30m knowledge check and completion criteria
- **Required VM(s):** Azure Linux 3
- **Prior module required:** Module 6 - Storage Components

## Learning Objectives

- Explain the history and positioning of Azure Linux within the Microsoft and Azure ecosystem.
- Distinguish between Azure Linux 3 and the upcoming Azure Linux 4 and understand what changes between versions.
- Perform day-1 package, networking, and time-sync administration on Azure Linux 3.
- Understand Azure Linux extension support and the release lifecycle.
- Locate Azure Linux community and support channels.

## Key Topics

- Azure Linux overview: Microsoft-maintained open-source distribution, previously named CBL-Mariner.
- Azure Linux 3: current production version, RPM-based, used in AKS nodes, Azure Arc, and other Azure-internal workloads.
- Azure Linux 4: next major release, what is changing and what to expect.
- Azure Linux in WSL2 for local development and testing.
- RPM package model and repository hosting on packages.microsoft.com (PMC).
- DNF and TDNF tooling: when each is used and key behavioral differences.
- Netplan for network configuration.
- Chrony for time synchronization.
- Supported VM extensions on Azure Linux.
- Release lifecycle: major release cadence and support windows.
- Community channels and SLA/CVE response framework.

## Command Highlights

- az aks create --name mycluster --resource-group azurelinuxrg --os-sku AzureLinux
- dnf install <package_name>
- dnf update <package_name>
- dnf search <keyword>
- dnf remove <package_name>
- dnf info <package_name>
- dnf list installed
- dnf repolist
- tdnf install <package_name>
- tdnf list installed

## Labs

- No lab created yet. See LABS.md for planned content.
