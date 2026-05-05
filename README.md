# Foundations - Labs

## Module 1 - OS Fundamentals I

| Lab | Title | Description | Time | VMs |
|-----|-------|-------------|------|-----|
| [Lab 01](Module%201%20-%20OS%20Fundamentals%20I/Labs/OSFundamentalsILab01.md) | WSL2, Azure CLI and RHEL 9 VM Deployment | Set up a local Linux shell using WSL2 and Windows Terminal, install Azure CLI, deploy a RHEL 9 VM in Azure, and transition from password-based to SSH key-based authentication. | 60 min | 1 (RHEL 9) |
| [Lab 02](Module%201%20-%20OS%20Fundamentals%20I/Labs/OSFundamentalsILab02.md) | User and Group Administration | Create and delete users, manage passwords, configure SSH password authentication, work with user expiration, and organize users into groups using primary and secondary group assignments. | 30 min | 1 (RHEL 8.10) |
| [Lab 03](Module%201%20-%20OS%20Fundamentals%20I/Labs/OSFundamentalsILab03.md) | File Operations, Manipulating and Finding Files | Navigate the filesystem, create and delete directories, create and find files, copy and move files, use redirection and concatenation to manipulate text files, and use advanced find options. | 30 min | 1 (RHEL 9) |
| [Lab 04](Module%201%20-%20OS%20Fundamentals%20I/Labs/OSFundamentalsILab04.md) | Process Management | List and inspect running processes, monitor system activity in real time, manage process priorities, send signals to processes, and work with background and foreground jobs across RHEL and Ubuntu. | 45 min | 2 (RHEL 8.10, Ubuntu 24.04) |
| [Lab 05](Module%201%20-%20OS%20Fundamentals%20I/Labs/OSFundamentalsILab05.md) | File Permissions and Ownership | Master permission management, ownership changes, and special permission bits to implement proper security controls in a multi-user development environment. | 45-60 min | 1 (RHEL 10) |

## Module 2 - OS Fundamentals II

| Lab | Title | Description | Time | VMs |
|-----|-------|-------------|------|-----|
| [Lab 06](Module%202%20-%20OS%20Fundamentals%20II/Labs/OSFundamentalsIILab06.md) | Hard and Soft Links | Understand differences between hard and soft links, create symbolic shortcuts to configuration files across directories, and maintain duplicate references to critical data files. | 20-30 min | 1 (SLES 16) |
| [Lab 07](Module%202%20-%20OS%20Fundamentals%20II/Labs/OSFundamentalsIILab07.md) | Managing Services in Linux | Verify service status, troubleshoot failures, and ensure critical services are correctly enabled for boot using `systemctl` and the systemd service manager. | 30 min | 1 (RHEL 10) |

## Module 3 - Package Management

| Lab | Title | Description | Time | VMs |
|-----|-------|-------------|------|-----|
| [Lab 08](Module%203%20-%20Package%20Management/Labs/PackageManagementLab08.md) | Package Management | Query, install, update, and remove packages across RPM-based (RHEL), Debian-based (Ubuntu), and SUSE (SLES) distributions using `rpm`/`dnf`, `dpkg`/`apt`, and `zypper`. | 75 min | 3 (RHEL 9, Ubuntu 24.04, SLES 15 SP7) |

## Module 4 - Linux on Azure

| Lab | Title | Description | Time | VMs |
|-----|-------|-------------|------|-----|
| [Lab 09](Module%204%20-%20Linux%20on%20Azure/Labs/LinuxOnAzureLab09.md) | Linux on Azure | Work with the Azure Linux VM Agent (waagent), cloud-init provisioning, VM extensions, and swap management on the ephemeral disk. | 60 min | 3 (RHEL 9, Ubuntu 24.04, SLES 15 SP7) |

## Module 5 - Network Components

| Lab | Title | Description | Time | VMs |
|-----|-------|-------------|------|-----|
| [Lab 10](Module%205%20-%20Network%20Components/Labs/NetworkComponentsLab10.md) | Network Components | Linux networking fundamentals in Azure covering the `ip` command, cloud-init networking, distribution-specific network managers, DNS configuration, and troubleshooting tools (`ss`, `netstat`, `tcpdump`). | 90 min | 3 (RHEL 9, Ubuntu 24.04, SLES 15 SP7) |

## Module 6 - Storage Components

| Lab | Title | Description | Time | VMs |
|-----|-------|-------------|------|-----|
| [Lab 11](Module%206%20-%20Storage%20Components/Labs/StorageComponentsLab11.md) | Storage Management | Disk management, partitioning, filesystem operations, and Logical Volume Manager (LVM) including disk identification by LUN ID, persistent storage configurations across reboots. | 90 min | 1 (SLES 15 SP7) |
