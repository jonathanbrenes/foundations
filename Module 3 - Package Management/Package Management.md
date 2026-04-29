# Module 3 - Package Management

## Module Overview

- **Training day:** Day 2
- **Planned duration:** 3 hours
- **Time breakdown:** ~1h reading and concept primer | ~1.5h lab execution | ~30m knowledge check and completion criteria
- **Required VM(s):** RHEL 9, Ubuntu 24.04, SLES 15 SP7
- **Prior module required:** Module 2 - OS Fundamentals II

## Learning Objectives

- Compare package concepts and repository models across distributions.
- Perform package lifecycle operations in RPM-based environments.
- Understand package update infrastructure and baseline patch workflows.
- Manage kernel package state and default kernel selection.

## Key Topics

- Package and repository fundamentals.
- RPM metadata and package information.
- DNF operations for install, update, query, and remove.
- Package update infrastructure.
- Kernel version management and default kernel changes.

## Command Highlights

**RHEL (RPM/DNF):**
- `rpm -qf`, `rpm -qil`, `rpm -qa`, `rpm --rebuilddb`
- `dnf install`, `dnf remove`, `dnf update`, `dnf search`, `dnf info`, `dnf deplist`
- `yumdownloader`, `yum localinstall`

**Ubuntu (DPKG/APT):**
- `dpkg -S`, `dpkg -l`, `dpkg -L`, `dpkg -r`
- `apt-get update`, `apt-get install`, `apt-get remove`, `apt-get purge`, `apt-get changelog`

**SLES (Zypper):**
- `zypper se`, `zypper in`, `zypper rm`, `zypper up`, `zypper lr`, `zypper clean`

## Labs

### Lab 08 — Package Management
- **Duration:** ~75 minutes
- **VMs:** RHEL 9, Ubuntu 24.04, SLES 15 SP7 (FoundationsLab08.json)
- **Objectives:**
  - Query packages using `rpm` and rebuild the RPM database
  - Search, install, update, and remove packages with `dnf`/`yum`
  - Download packages without installing using `yumdownloader`
  - Query and manage packages with `dpkg` on Ubuntu
  - Install, remove, and purge packages with `apt` on Ubuntu
  - Search, install, remove, and clean packages with `zypper` on SLES
- **Key Commands:** `rpm`, `dnf`, `yum`, `yumdownloader`, `dpkg`, `apt-get`, `zypper`
