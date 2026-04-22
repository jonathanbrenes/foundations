# Lab 8 — Package Management

## Title and Scenario

This lab covers package management across the three major Linux distribution families used in Azure: RPM-based (Red Hat/RHEL), Debian-based (Ubuntu), and SUSE (SLES). You will learn to query, install, update, and remove packages using the native tools for each distribution: `rpm`/`dnf` (yum), `dpkg`/`apt`, and `zypper`.

Each scenario uses a dedicated VM deployed from the templates provided.

- **Estimated time:** 50 minutes.
- **VM count:** 3 (RHEL 9, Ubuntu 24.04, SLES 15 SP7).

---

## Deployment

> **Warning:** The VMs deployed in these labs allow inbound traffic automatically from the Azure VPN (AzureCloud service tag). Make sure you are connected to the Azure VPN, or add an NSG rule to allow your public IP address before attempting to connect.

### RHEL 9 VM (Scenarios 1 and 2)

[![Click to deploy](https://aka.ms/deploytoazurebutton)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fjonathanbrenes%2Ffoundations%2Fmain%2FModule%25203%2520-%2520Package%2520Management%2FLabs%2FFoundationsLab08RHEL.json)

### Ubuntu 24.04 VM (Scenarios 3 and 4)

[![Click to deploy](https://aka.ms/deploytoazurebutton)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fjonathanbrenes%2Ffoundations%2Fmain%2FModule%25203%2520-%2520Package%2520Management%2FLabs%2FFoundationsLab08Ubuntu.json)

### SLES 15 SP7 VM (Scenario 5)

[![Click to deploy](https://aka.ms/deploytoazurebutton)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fjonathanbrenes%2Ffoundations%2Fmain%2FModule%25203%2520-%2520Package%2520Management%2FLabs%2FFoundationsLab08SLES.json)

---

## Skills Required

- Ability to connect to a Linux VM via SSH.
- Basic familiarity with running commands as root using `sudo`.

---

## Recommended Prerequisites

- Lab 1 completed — SSH connectivity and Azure CLI familiarity.
- Lab 2 completed — user management and `sudo` usage.

---

## Objectives

After completing this lab, you will be able to:

- Use `rpm` to query packages, locate file ownership, and rebuild the RPM database.
- Use `dnf` (yum) to search, install, update, remove, and download packages on RPM-based distributions.
- Use `dpkg` to query and manage packages on Debian-based distributions.
- Use `apt` to install, update, remove, and purge packages on Debian-based distributions.
- Use `zypper` to search, install, remove, update, and clean packages on SUSE distributions.

---

## Environment Overview

| Component | Details |
|---|---|
| RHEL VM | RHEL 9 (`lab08rhel`) — Scenarios 1 and 2 |
| Ubuntu VM | Ubuntu 24.04 LTS (`lab08ubuntu`) — Scenarios 3 and 4 |
| SLES VM | SLES 15 SP7 (`lab08sles`) — Scenario 5 |
| VM size | Standard_D2s_v3 |
| Access | SSH as `azureuser` with password authentication |

---

## Your Mission

Complete each scenario on the corresponding VM. Deploy VMs as needed from the Deployment section.

### Scenario 1 — Using rpm

RPM is the low-level package manager for Red Hat-based distributions. It operates on individual `.rpm` files and the local RPM database. It does not resolve dependencies automatically.

**Deployment:** See the [Deployment](#deployment) section (RHEL 9 VM).

#### Instructions

Step 1: Connect and switch to root
- Connect to the `lab08rhel` VM and switch to root.
  ```bash
  ssh azureuser@<RHEL_VM_IP> # Connect to the RHEL VM using SSH
  sudo -i # Switch to root for full administrative privileges
  ```

Step 2: Find which package owns a file
- Identify which package the file `/etc/logrotate.conf` belongs to.
  ```bash
  rpm -qf /etc/logrotate.conf # Query which package owns the specified file (-qf)
  ```

Step 3: Show package information
- List information about the `logrotate` package, including all files it contains.
  ```bash
  rpm -qil logrotate # Query package info (-qi) and list all files (-l) in one command
  ```

Step 4: Attempt to remove a package
- Try to remove the `logrotate` package and observe the dependency warnings.
  ```bash
  rpm -e logrotate # Attempt to erase (-e) the logrotate package; may fail due to dependencies
  ```

Step 5: List all installed packages
- Get a list of all installed RPMs and save a copy for later comparison.
  ```bash
  rpm -qa > /var/tmp/package_list # Query all installed packages (-qa) and redirect the list to a file
  rpm -qa # Display all installed packages to the terminal
  ```

Step 6: Back up the RPM database
- Create a backup of the RPM database directory.
  ```bash
  cp -a /var/lib/rpm /var/lib/rpm_backup # Copy the entire RPM database preserving attributes (-a)
  ```

Step 7: Rebuild the RPM database
- Rebuild the RPM database using the `rpm` command.
  ```bash
  rpm --rebuilddb # Rebuild the RPM database from installed package headers
  ```

Step 8: Compare the rebuilt database with the backup
- Compare the new database directory with the backup.
  ```bash
  diff -bur /var/lib/rpm /var/lib/rpm_backup # Compare directories recursively; -b ignores whitespace, -u unified format, -r recursive
  ls -l /var/lib/rpm | wc -l # Count files in the rebuilt database
  ls -l /var/lib/rpm_backup | wc -l # Count files in the backup for comparison
  ```

Step 9: Verify the package list after rebuild
- List all RPMs again and compare with the list generated before the rebuild.
  ```bash
  rpm -qa | tee /var/tmp/package_list_new # List all packages and save to a new file; tee writes to both file and terminal
  diff /var/tmp/package_list /var/tmp/package_list_new # Compare old and new package lists to verify nothing changed
  ```

Step 10: Clean up the backup
- Remove the RPM database backup.
  ```bash
  rm -rf /var/lib/rpm_backup # Remove the backup directory and all its contents
  ```

### Scenario 2 — Using dnf (yum)

`dnf` is the high-level package manager for RPM-based distributions. It resolves dependencies automatically and manages repositories. Starting with RHEL 8, `dnf` replaced `yum` as the default package manager. The `yum` command is still available as an alias for backward compatibility, so both commands produce the same result.

**Deployment:** Uses the same RHEL 9 VM from Scenario 1.

> All images in this lab are examples. Outputs may vary on your VM — focus on the commands and their behavior.

#### Instructions

Step 1: Connect to the VM
- Continue using the `lab08rhel` VM. If disconnected, reconnect and switch to root.
  ```bash
  sudo -i # Switch to root for full administrative privileges
  ```

Step 2: Check for available updates
- Check if there are available updates for the system.
  ```bash
  yum update # Check and apply updates; abort with N to continue the lab without applying
  yum check-update # List packages with available updates without applying
  yum list updates # Another way to list available updates
  ```

  > Using the first command, the system will prompt to apply pending updates. Abort with `N` to continue executing the remaining commands.

Step 3: Update a specific package
- Update a single package individually.
  ```bash
  yum update bash # Update only the bash package to its latest available version
  ```

Step 4: List kernel packages
- List all installed kernel-related packages, and list both installed and available.
  ```bash
  yum list installed "kernel" # List only the installed kernel package
  yum list "kernel*" # List all installed and available packages matching the kernel wildcard
  ```

Step 5: Install a package
- Install the `httpd-devel` package.
  ```bash
  yum install httpd-devel -y # Install httpd-devel; -y automatically answers yes to all prompts
  ```

Step 6: Remove a package
- Remove the `httpd-devel` package and its dependencies.
  ```bash
  yum remove httpd-devel -y # Remove httpd-devel and unused dependencies; -y confirms automatically
  ```

Step 7: Search for a package
- Find all packages with "bash" in their name or description.
  ```bash
  yum search bash # Search package names and descriptions for the term bash
  ```

Step 8: List installed and available packages
- Find all installed and available bash packages.
  ```bash
  yum list bash # Show installed and available versions of the bash package
  ```

Step 9: Get package information
- Get detailed information for the bash package.
  ```bash
  yum info bash # Display detailed metadata for the bash package
  ```

Step 10: Display package dependencies
- Display the dependencies for the bash package.
  ```bash
  yum deplist bash # List all dependencies required by the bash package
  ```

Step 11: Test elevated vs. regular user
- Test commands as both root and a regular user. Observe any differences.
  ```bash
  sudo yum info bash # Run as root using sudo
  yum info bash # Run as the current user (root in this case)
  sudo yum list bash # Run as root using sudo
  yum list bash # Run as the current user
  ```

Step 12: Install yum-utils
- Install the `yum-utils` package, which provides additional utilities.
  ```bash
  sudo yum install yum-utils -y # Install the yum-utils package which includes yumdownloader and other tools
  ```

Step 13: Download a package without installing
- Download a package file without installing it.
  ```bash
  sudo yumdownloader xterm # Download the xterm RPM file to the current directory without installing it
  ```

Step 14: Install from a local file
- Install the downloaded xterm package using `yum`.
  ```bash
  sudo yum localinstall $(pwd)/xterm-*.rpm -y # Install the local RPM file; yum resolves any additional dependencies
  ```

### Scenario 3 — Using dpkg

`dpkg` is the low-level package manager for Debian-based distributions. Like `rpm`, it operates on individual `.deb` files and the local package database without resolving dependencies.

**Deployment:** See the [Deployment](#deployment) section (Ubuntu 24.04 VM).

#### Instructions

Step 1: Connect and switch to root
- Connect to the `lab08ubuntu` VM and switch to root.
  ```bash
  ssh azureuser@<UBUNTU_VM_IP> # Connect to the Ubuntu VM using SSH
  sudo -i # Switch to root for full administrative privileges
  ```

Step 2: Find which package owns a file
- Identify which package the file `/etc/sos/sos.conf` belongs to.
  ```bash
  dpkg -S /etc/sos/sos.conf # Search (-S) for the package that owns the specified file
  ```

Step 3: Show package information
- List information about the `sosreport` package and all files it contains.
  ```bash
  dpkg -l sosreport # Show package status and version information (-l)
  dpkg -L sosreport # List all files installed by the sosreport package (-L)
  ```

Step 4: Remove a package
- Try to remove the `sosreport` package.
  ```bash
  dpkg -r sosreport # Remove (-r) the sosreport package; may fail if other packages depend on it
  ```

Step 5: List all installed packages
- List all packages installed on the system.
  ```bash
  dpkg -l # List all installed packages with their status and version
  ```

Step 6: Verify if a specific package is installed
- Check if `xauth` is installed on the system.
  ```bash
  dpkg -l | grep xauth # Filter the package list for entries matching xauth
  ```

### Scenario 4 — Using apt

`apt` (and `apt-get`) is the high-level package manager for Debian-based distributions. It resolves dependencies automatically and manages repositories.

**Deployment:** Uses the same Ubuntu 24.04 VM from Scenario 3.

#### Instructions

Step 1: Connect to the VM
- Continue using the `lab08ubuntu` VM. If disconnected, reconnect and switch to root.
  ```bash
  sudo -i # Switch to root for full administrative privileges
  ```

Step 2: Update the package index
- Update the list of available packages from all configured repositories.
  ```bash
  apt-get update # Download the latest package lists from the repositories
  ```

Step 3: Upgrade installed packages
- Update all installed packages to their latest available versions.
  ```bash
  apt-get upgrade # Upgrade all installed packages; will prompt for confirmation
  ```

Step 4: Remove unneeded packages
- Remove packages that were installed as dependencies and are no longer required.
  ```bash
  apt-get autoremove # Remove orphaned dependency packages that are no longer needed
  ```

Step 5: Install a package
- Install the `nmap` package.
  ```bash
  apt-get install nmap -y # Install nmap; -y automatically confirms the installation
  ```

Step 6: Remove a package (keep config)
- Remove the `nmap` package without removing its configuration files.
  ```bash
  apt-get remove nmap -y # Remove the nmap binary and data but keep configuration files
  ```

Step 7: Reinstall the package
- Reinstall `nmap` non-interactively.
  ```bash
  apt-get install nmap -y # Reinstall nmap; -y answers yes to all prompts
  ```

Step 8: Purge a package (remove config)
- Completely remove the `nmap` package along with its configuration files.
  ```bash
  apt-get purge nmap -y # Remove nmap and delete all its configuration files
  ```

  Or equivalently:

  ```bash
  apt-get remove nmap --remove -y # Alternative syntax to remove the package and its config files
  ```

  > Using `autoremove` afterward may be required to clean up leftover dependency packages.

Step 9: Check the changelog
- View the changelog for the `nmap` package.
  ```bash
  apt-get changelog nmap # Display the changelog for nmap; press q to exit
  ```

  > The package does not need to be installed to view its changelog. Press `q` to exit.

Step 10: Full system upgrade
- Perform a full system upgrade, handling complex dependency changes.
  ```bash
  apt-get dist-upgrade -y # Upgrade all packages including those with dependency changes; -y confirms automatically
  ```

  > This does not perform a major distribution version upgrade.

### Scenario 5 — Using zypper

`zypper` is the package manager for SUSE and OpenSUSE distributions. It handles package installation, updates, removal, and repository management.

**Deployment:** See the [Deployment](#deployment) section (SLES 15 SP7 VM).

#### Instructions

Step 1: Connect and switch to root
- Connect to the `lab08sles` VM and switch to root.
  ```bash
  ssh azureuser@<SLES_VM_IP> # Connect to the SLES VM using SSH
  sudo -i # Switch to root for full administrative privileges
  ```

Step 2: Search for a package
- Search for the `nmap` package before installing it.
  ```bash
  zypper se nmap # Search (se) for packages matching nmap in the repository
  ```

Step 3: List repositories
- Get the list of configured repositories.
  ```bash
  zypper lr # List all defined repositories (lr)
  ```

Step 4: Install a package
- Install the `nmap` package. This will install any required dependencies.
  ```bash
  zypper in nmap # Install (in) the nmap package and resolve dependencies
  ```

Step 5: Remove a package
- Remove the `nmap` package.
  ```bash
  zypper rm nmap # Remove (rm) the nmap package
  ```

Step 6: Update installed packages
- Update all installed packages to their latest available versions.
  ```bash
  zypper up # Update (up) all installed packages
  ```

Step 7: Install non-interactively
- Install `nmap` again without user interaction.
  ```bash
  zypper --non-interactive in nmap # Install nmap without prompting for confirmation
  ```

Step 8: Remove non-interactively
- Remove `nmap` without user interaction.
  ```bash
  zypper --non-interactive rm nmap # Remove nmap without prompting for confirmation
  ```

Step 9: Clean package cache
- Clean the cache of downloaded packages.
  ```bash
  zypper clean # Remove cached package downloads to free disk space
  ```

Step 10: Clean all caches
- Clean metadata, packages, and repository data.
  ```bash
  zypper clean -a # Remove all cached data including metadata, packages, and repository info (-a for all)
  ```

Step 11: Distribution upgrade
- Upgrade all installed packages, handling dependency changes.
  ```bash
  zypper dist-upgrade # Perform a distribution upgrade; may install or remove packages as needed
  ```

  > `zypper update` refreshes installed packages to their latest versions, while `zypper dist-upgrade` also handles dependency changes and can install or remove packages as needed.

---

## Analytical Guidance

- **Scenario 1:** `rpm` is the foundation of package management on Red Hat systems. Understanding how to query the database and rebuild it is essential for troubleshooting corrupted package databases — a common support case.
- **Scenario 2:** `dnf`/`yum` adds dependency resolution on top of `rpm`. The ability to download packages without installing them (`yumdownloader`) is useful for air-gapped or restricted environments.
- **Scenario 3:** `dpkg` is the equivalent of `rpm` for Debian systems. The `-S` (search) and `-L` (list files) flags are the most commonly used in troubleshooting.
- **Scenario 4:** `apt` adds dependency resolution on top of `dpkg`. Understanding the difference between `remove` (keeps config) and `purge` (deletes config) is critical for clean package removal.
- **Scenario 5:** `zypper` is the all-in-one tool for SUSE. Its `--non-interactive` flag is essential for scripted/automated deployments.

---

## Validation Criteria

| Step | Expected result |
|---|---|
| `rpm -qf` | Returns the package name that owns the queried file |
| `rpm --rebuilddb` | Completes without errors; `diff` shows database differences are structural only |
| `rpm -qa` comparison | Old and new package lists match (no packages lost during rebuild) |
| `yum install` | Package installs successfully with dependencies resolved |
| `yumdownloader` | RPM file downloaded to the current directory |
| `dpkg -S` | Returns the package name owning the file |
| `dpkg -r` | Removes the package or shows dependency errors |
| `apt-get purge` | Package and configuration files removed completely |
| `zypper in` / `zypper rm` | Package installs and removes cleanly |
| `zypper clean -a` | Cache cleaned without errors |

---

## Documentation Expectations

As you complete this lab, take note of:

- The difference between low-level (`rpm`, `dpkg`) and high-level (`dnf`/`yum`, `apt`, `zypper`) package managers.
- Which commands require root privileges and which do not.
- The difference between `apt-get remove` (keeps config) and `apt-get purge` (deletes config).
- The difference between `zypper update` and `zypper dist-upgrade`.
- How `yumdownloader` can be used to pre-stage packages for offline installation.

---

## What Not To Do

- Do not apply full system updates (`yum update`, `apt-get upgrade`) during the lab unless instructed — this can take significant time and change expected outputs.
- Do not remove core system packages (e.g., `systemd`, `bash`, `openssh-server`).
- Do not run `rpm --rebuilddb` on production systems without a confirmed backup of `/var/lib/rpm`.
- Do not delete the VMs until all Module 3 labs are complete.

---

## Real-World Context

Package management is one of the most frequent tasks in Linux system administration. In Azure support and SRE work, engineers regularly need to identify which package owns a file (to determine if a binary was modified), install diagnostic tools on customer VMs, and troubleshoot corrupted RPM databases. The `rpm --rebuilddb` workflow in Scenario 1 directly mirrors a common recovery procedure for Red Hat-based systems where `yum` stops working due to database corruption. Understanding the tooling differences across RHEL, Ubuntu, and SLES is essential for supporting Azure's multi-distribution Linux ecosystem.

---

## Optional Advanced Exploration

- Use `rpm -V` to verify the integrity of an installed package and identify modified files:
  ```bash
  rpm -V bash # Verify the bash package; shows any files that differ from the original installation
  ```
- Use `apt-cache policy` to show the installation candidate and repository source for a package:
  ```bash
  apt-cache policy nmap # Show the installed version, candidate version, and repository source
  ```
- Use `zypper info` to display detailed package metadata on SLES:
  ```bash
  zypper info nmap # Display detailed information about the nmap package
  ```
- Explore `dnf history` to view and undo past transactions:
  ```bash
  dnf history # Show the history of dnf transactions
  dnf history undo <ID> # Undo a specific transaction by ID
  ```
