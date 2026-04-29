# Module 1 - OS Fundamentals I

## Module Overview

- **Training day:** Day 1
- **Planned duration:** 5+ hours
- **Time breakdown:** ~1.5h reading and concept primer | ~3.5h lab execution (Labs 01-05) | ~30m knowledge check and completion criteria
- **Required VMs:** RHEL 9, RHEL 8.10, RHEL 10, Ubuntu 24.04
- **Prior module required:** None — this is the starting point.

## Learning Objectives

After completing Module 1, you will be able to:

- **Environment Setup:** Deploy Linux VMs in Azure and establish SSH-based remote connectivity
- **User Management:** Create and manage user accounts, groups, and authentication
- **File Operations:** Navigate the filesystem, manipulate files, and search efficiently
- **Process Management:** Inspect, monitor, and control running processes
- **File Security:** Manage file permissions, ownership, and access control

## Key Topics

**Connectivity & Environment:**
- WSL2 and Windows Terminal for Linux development workflows
- Azure CLI for VM deployment and management
- SSH key-based authentication and remote access

**User and Group Administration:**
- User account creation and deletion (`useradd`, `userdel`)
- Password management and expiration policies (`passwd`, `chage`)
- SSH password authentication configuration
- Group creation and membership management (`groupadd`, `usermod`)

**File System Operations:**
- Filesystem navigation and directory management (`cd`, `pwd`, `mkdir`, `ls`)
- File operations: creation, copying, moving, renaming
- Input/output redirection and concatenation
- File searching with `find` and `locate`

**Process Management:**
- Process listing and inspection (`ps`, `pgrep`)
- Real-time monitoring with `top`
- Process priority management (`nice`, `renice`)
- Job control (`jobs`, `bg`, `fg`, `wait`)
- Signal handling and process termination (`kill`, `pkill`)

**File Permissions and Security:**
- Permission notation: symbolic (`rwxrwxrwx`) and numeric (octal 755)
- Permission modification with `chmod`
- Ownership management with `chown` and `chgrp`
- Special permissions: SGID and sticky bit
- Default permissions with `umask`
- Access Control Lists (ACLs) with `getfacl` and `setfacl`

## Command Highlights

**Environment & Connectivity:**
- `az vm create`, `az vm list`, `ssh`, `ssh-keygen`, `ssh-copy-id`

**User & Group Management:**
- `useradd`, `userdel`, `passwd`, `chage`, `groupadd`, `usermod`, `id`, `groups`, `sudo`

**File Operations:**
- `cd`, `pwd`, `mkdir`, `ls`, `find`, `locate`, `updatedb`, `cp`, `mv`, `rm`, `cat`, `touch`

**Process Management:**
- `ps`, `ps aux`, `top`, `nice`, `renice`, `kill`, `pkill`, `jobs`, `bg`, `fg`, `wait`, `pgrep`

**Permissions & Ownership:**
- `chmod`, `chown`, `chgrp`, `stat`, `ls -l`, `umask`, `getfacl`, `setfacl`

## Labs

### Lab 01 — Environment Setup: WSL2, Azure CLI and VM Deployment
- **Duration:** ~60 minutes
- **VM:** RHEL 9 (FoundationsLab01.json)
- **Objectives:**
  - Install WSL2 and Windows Terminal for Linux development
  - Install and configure Azure CLI
  - Deploy a Linux VM in Azure
  - Establish SSH key-based authentication for secure remote access
- **Key Commands:** `az account show`, `az vm create`, `ssh-keygen`, `ssh-copy-id`

### Lab 02 — User and Group Administration
- **Duration:** ~45 minutes
- **VM:** RHEL 8.10 (FoundationsLab02.json)
- **Objectives:**
  - Create and delete users with `useradd` and `userdel`
  - Manage user passwords and password expiration policies
  - Enable and test SSH password authentication
  - Create and manage groups with `groupadd` and `usermod`
  - Understand primary and secondary group membership
- **Key Commands:** `useradd`, `userdel`, `passwd`, `chage`, `groupadd`, `usermod`, `id`, `groups`

### Lab 03 — File Operations, Manipulation and Search
- **Duration:** ~45 minutes
- **VM:** RHEL 9 (FoundationsLab03.json)
- **Objectives:**
  - Navigate the filesystem and manage directories with `cd`, `pwd`, `mkdir`
  - Create, copy, move, and rename files with `touch`, `cp`, `mv`
  - Use input/output redirection (`>`, `>>`) and concatenation with `cat`
  - Search for files using advanced `find` options and filters
  - Locate files in the system database with `updatedb` and `locate`
- **Key Commands:** `find`, `locate`, `updatedb`, `ls`, `cp`, `mv`, `rm`, `redirection operators`

### Lab 04 — Process Management
- **Duration:** ~45 minutes
- **VMs:** RHEL 8.10 and Ubuntu 24.04 (FoundationsLab04.json)
- **Objectives:**
  - List and inspect running processes with `ps aux` and `ps -ef`
  - Monitor system activity in real time using `top`
  - Manage process priority with `nice` and `renice`
  - Control foreground and background job execution with `jobs`, `bg`, `fg`
  - Send signals to processes using `kill`, `pkill`, and signal handling (SIGTERM vs SIGKILL)
  - Generate load on the system with `stress` and `stress-ng`
- **Key Commands:** `ps`, `top`, `nice`, `renice`, `kill`, `pkill`, `jobs`, `bg`, `fg`, `wait`

### Lab 05 — File Permissions and Ownership
- **Duration:** ~45-60 minutes
- **VM:** RHEL 10 (FoundationsLab05.json)
- **Objectives:**
  - Interpret and modify file permissions using symbolic (`rwx`) and numeric (octal) notation
  - Change file and directory ownership with `chown` and `chgrp`
  - Implement special permissions: Set Group ID (SGID) for group inheritance
  - Apply sticky bit for controlled file deletion in shared directories
  - Control default permissions for new files and directories using `umask`
  - Apply Access Control Lists (ACLs) using `getfacl` and `setfacl` (optional advanced)
- **Key Commands:** `chmod`, `chown`, `chgrp`, `stat`, `ls -l`, `umask`, `getfacl`, `setfacl`
