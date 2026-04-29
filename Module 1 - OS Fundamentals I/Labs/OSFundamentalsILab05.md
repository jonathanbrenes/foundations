# Lab 5 — File Permissions and Ownership

## Title and Scenario

You are a Linux system administrator responsible for securing file access across a multi-user development environment. Users must access shared files and directories with appropriate permission levels, while sensitive configuration files remain protected from unauthorized access. Your task is to master permission management, ownership changes, and special permission bits to implement proper security controls.

- **Estimated time:** 45-60 minutes.
- **VM count:** 1 (RHEL 10).

---

## Deployment

[![Click to deploy](https://aka.ms/deploytoazurebutton)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fjonathanbrenes%2Ffoundations%2Fmain%2FModule%25201%2520-%2520OS%2520Fundamentals%2520I%2FLabs%2FFoundationsLab05.json)

## Skills Required

- Basic Linux file system navigation (Labs 01-03)
- User and group administration fundamentals (Lab 02)
- Command-line text editors (vim/nano)
- File manipulation and searching (Lab 03)

## Recommended Prerequisites

- Completion of Labs 01-04
- SSH access to the lab05rhel VM
- Understanding of file ownership concepts (Lab 02)
- Familiarity with basic Linux file operations

## Objectives

By completing this lab, you will:

1. **Understand permission modes** – Interpret symbolic and numeric (octal) permission notation
2. **Modify file permissions** – Use `chmod` to change access permissions using both symbolic and numeric syntax
3. **Change file ownership** – Use `chown` and `chgrp` to reassign ownership and group membership
4. **Apply special permissions** – Understand and implement SGID (Set Group ID) and sticky bit behaviors
5. **Control default permissions** – Use `umask` to set default permission masks for newly created files
6. **Verify and audit permissions** – Use `stat` and `ls -l` to identify permission issues
7. **Apply ACL basics** – Understand getfacl/setfacl for extended permissions (optional advanced)

## Environment Overview

| Component | Value |
|-----------|-------|
| **VM Name** | lab05rhel |
| **Operating System** | Red Hat Enterprise Linux 10 |
| **VM Size** | Standard_B2s (2 vCPU, 4 GiB RAM) |
| **Admin Username** | azureuser |
| **Primary Network Interface** | lab05rhel-nic1 (10.1.0.10) |
| **Public FQDN** | Assigned via Azure Portal |
| **Default Shell** | /bin/bash |

## Your Mission

Your mission is to configure file permissions and ownership across a simulated application environment. You will:

- Create a shared project directory with collaborative permissions
- Implement security restrictions on sensitive configuration files
- Configure SGID for automatic group ownership inheritance
- Apply sticky bit to a shared temporary directory
- Verify permissions using multiple methods

---

### Scenario 1 — Foundational Exercises: Understanding Permission Notation and chmod

**Objective:** Master the interpretation and application of file permissions using both symbolic and numeric modes through hands-on practice.

**Deployment:** See the [Deployment](#deployment) section (RHEL 10 VM).

#### Instructions

#### Step 1: Create a data directory and test file

```bash
mkdir ~/data # Create a directory called data under your home directory
cd ~/data # Move to the new data directory
touch filename.txt # Create a file called filename.txt
ls -l filename.txt # Review the default permissions of the new file
```

**Explanation:** The `ls -l` command displays file permissions in symbolic notation (e.g., `-rw-r--r--`). The first character indicates file type (`-` for regular file, `d` for directory). The next nine characters break down into three groups: owner (rwx), group (rwx), and others (rwx). Permission calculation:
- `r` (read) = 4, `w` (write) = 2, `x` (execute) = 1
- Each digit (0-7) represents one permission group: owner, group, others

**Expected Output:** The filename.txt file shows default permissions `-rw-r--r--` (644). This means: owner has read+write (6), group has read (4), others have read (4).

#### Step 2: Set restrictive permissions – Owner only access

```bash
chmod -v u=rwx,g=,o= ~/data/filename.txt # Set owner-only permissions using symbolic notation
chmod 700 ~/data/filename.txt # Set owner-only permissions using numeric (octal) notation
ls -l ~/data/filename.txt # Verify permissions show -rwx------
```

**Explanation:** 
- Symbolic notation: `u=rwx,g=,o=` sets user to rwx, group to nothing, others to nothing
- Numeric notation: `700` = (7)(0)(0) = owner gets 7 (rwx), group gets 0 (---), others get 0 (---)
- The `-v` flag provides verbose output showing the permission change
- This is the most restrictive permission set for a file – only the owner can access it

**Expected Output:** The file shows `-rwx------` (700 permissions). Only the owner has full access; group and others have no permissions.

#### Step 3: Modify permissions – Add group and other access

```bash
chmod -v g+rw,o+r ~/data/filename.txt # Modify the filename.txt permissions by adding read and write permissions; to the group and read only access to other users; Using symbolic notation (add permissions incrementally):
chmod 764 ~/data/filename.txt # Or using numeric (octal) notation (set exact permissions):
ls -l ~/data/filename.txt # Verify the permission changes
```

**Explanation:** 
- Symbolic notation: `g+rw,o+r` adds read and write to group, adds read to others
- Numeric notation: `764` = (7)(6)(4)
  - Owner (7): 4+2+1 = read + write + execute
  - Group (6): 4+2 = read + write
  - Others (4): 4 = read only
- `-v` shows the permission change being applied

**Expected Output:** The file shows `-rwxrw-r--` (764 permissions). Owner has rwx, group has rw, others have r only.

#### Step 4: Practice additional permission patterns

```bash
touch ~/data/test-script.sh # Create additional test files to practice different permission scenarios
touch ~/data/config.ini # Create config.ini
touch ~/data/secrets.env # Create secrets.env
chmod 750 ~/data/test-script.sh # Make script executable for owner and group (750 = rwxr-x---)
chmod 640 ~/data/config.ini # Create read-only config file for owner and group (640 = rw-r-----)
chmod 600 ~/data/secrets.env # Highly restricted secrets file – owner access only (600 = rw-------)
ls -l ~/data/ # Review all permissions to understand the patterns
```

**Explanation:** These are real-world permission patterns:
- `750` (rwxr-x---) – Executable script; owner can execute, group can read/execute, others blocked
- `640` (rw-r-----) – Configuration file; owner reads/writes, group reads only, others blocked
- `600` (rw-------) – Secrets file; only owner can access (most sensitive)

**Expected Output:** Directory listing shows:
```
-rwxr-x---  test-script.sh
-rw-r-----  config.ini
-rw-------  secrets.env
-rwxrw-r--  filename.txt
```

---

### Scenario 2 — Changing File Ownership with chown and chgrp

**Objective:** Reassign file and directory ownership to different users and groups.

**Deployment:** Uses the same RHEL 10 VM from Scenario 1.

#### Instructions

Step 1: Create test users and groups for ownership exercises

```bash
sudo groupadd -f devteam # Create test groups
sudo groupadd -f admins # Create group admins
sudo useradd -m -s /bin/bash -G devteam dev01 2>/dev/null || echo "dev01 may already exist" # Create test users
sudo useradd -m -s /bin/bash -G admins admin01 2>/dev/null || echo "admin01 may already exist" # Create user with group membership
echo "dev01:temppass123" | sudo chpasswd 2>/dev/null || true # Set passwords for testing (optional)
echo "admin01:temppass123" | sudo chpasswd 2>/dev/null || true # Set password for user
id dev01 # Verify user and group creation
id admin01 # Verify user creation and group memberships
```

**Explanation:** `useradd` creates new users with home directories and shell access. The `-G` flag adds the user to supplementary groups. Using `2>/dev/null || true` suppresses error messages if users already exist.

**Expected Output:** Commands complete without errors. `id` output shows user IDs, group IDs, and group memberships for each user.

#### Step 2: Change file ownership with chown

```bash
sudo chown dev01 ~/data/filename.txt # Change owner of filename.txt to dev01
sudo chown dev01:devteam ~/data/test-script.sh # Change owner and group together (user:group format)
sudo chgrp admins ~/data/config.ini # Change only the group (using :group syntax)
sudo chown -R dev01:devteam ~/data/ # Recursively change ownership of entire data directory
ls -la ~/data/ # Verify changes
```

**Explanation:** 
- `chown user file` – Changes owner only
- `chown user:group file` – Changes both owner and group
- `chown :group file` – Changes group only (same as chgrp)
- `-R` flag applies changes recursively to directories and all contents

**Expected Output:** File ownership now shows dev01, admin01, or appropriate users/groups in the listing.

#### Step 3: Practical ownership scenario – Project directory setup

```bash
sudo mkdir -p /opt/myapp/{src,build,config,logs} # Create a shared project structure with proper ownership
sudo chown -R dev01:devteam /opt/myapp/src/ # Assign source code to development team
sudo chown -R admin01:admins /opt/myapp/config/ # Assign configuration to admins
sudo chmod 755 /opt/myapp/ # Set appropriate permissions for each directory
sudo chmod 755 /opt/myapp/src/ # Set permissions to 755 on 
sudo chmod 700 /opt/myapp/config/ # Set permissions to 700 on 
sudo chmod 775 /opt/myapp/logs/ # Set permissions to 775 on 
ls -laR /opt/myapp/ # Verify the complete structure
```

**Explanation:** This represents a real-world scenario where different directories have different ownership and permission levels. The `logs/` directory (775) allows team members to write logs while restricting general read access.

**Expected Output:** Directory structure shows appropriate ownership and permissions. Config directory (700) is accessible only by admin01.

---

### Scenario 3 — Special Permissions: SGID and Sticky Bit

**Objective:** Understand and implement Set Group ID (SGID) and sticky bit behaviors for collaborative environments.

**Deployment:** Uses the same RHEL 10 VM from Scenario 1.

#### Instructions

Step 1: Understand and apply SGID (Set Group ID)

```bash
sudo mkdir -p /opt/shared-project # Create a shared directory for the development team
sudo chown root:devteam /opt/shared-project # Change ownership to root:devteam
sudo chmod 755 /opt/shared-project/ # Set permissions to 755 on 
sudo chmod 2775 /opt/shared-project/ # Apply SGID bit using numeric notation (2000 = SGID); 2775 = SGID + rwxrwxr-x
ls -la /opt/shared-project/ # Or using symbolic notation:; sudo chmod g+s /opt/shared-project/; Verify SGID is set (ls should show 's' in group execute position)
stat /opt/shared-project/ # Display detailed file attributes
```

**Explanation:** SGID (Set Group ID, value 2) on a directory means:
- Any file created within inherits the directory's group (not the creator's primary group)
- Useful for collaborative projects where all team members need group access
- Appears as 's' in the group execute position: `drwxrwsr-x`

**Expected Output:** SGID bit is set. In symbolic notation: either lowercase 's' (if group has execute) or uppercase 'S' (if group has no execute).

#### Step 2: Understand and apply Sticky Bit

```bash
sudo mkdir -p /opt/temp-shared # Create a temporary shared directory
sudo chown root:devteam /opt/temp-shared # Change ownership to root:devteam
sudo chmod 777 /opt/temp-shared/ # Set permissions to 777 on 
sudo chmod 1777 /opt/temp-shared/ # Apply sticky bit using numeric notation (1000 = sticky bit); 1777 = sticky bit + rwxrwxrwx
ls -la /opt/ | grep temp-shared # Or using symbolic notation:; sudo chmod o+t /opt/temp-shared/; Verify sticky bit (ls should show 't' in others execute position)
stat /opt/temp-shared/ # Display detailed file attributes
```

**Explanation:** Sticky bit (value 1) on a directory means:
- Anyone can create files (if 777 permissions)
- Only the file owner, directory owner, or root can delete files
- Prevents users from accidentally deleting others' files
- Appears as 't' in others execute position: `drwxrwxrwt`
- Common on `/tmp` and `/var/tmp` directories

**Expected Output:** Sticky bit is set: `drwxrwxrwt` with 't' in the others execute position.

#### Step 3: Combine special bits with regular permissions

```bash
sudo mkdir -p /opt/collab-space # Create a directory with SGID and sticky bit together
sudo chown root:devteam /opt/collab-space # Change ownership to root:devteam
sudo chmod 3770 /opt/collab-space/ # 3770 = SGID + sticky bit + rwxrwx---
stat /opt/collab-space/ # Verify both bits are set
ls -la /opt/ | grep collab-space # List files and details
```

**Explanation:** You can combine special bits:
- 4xxx = SUID (Set User ID, rarely used on directories)
- 2xxx = SGID (Set Group ID)
- 1xxx = Sticky bit
- Add these values to the standard 3-digit octal mode

**Expected Output:** Directory shows both special bits active in `stat` output.

---

### Scenario 4 — Controlling Default Permissions with umask

**Objective:** Understand how umask affects default permissions for newly created files and directories.

**Deployment:** Uses the same RHEL 10 VM from Scenario 1.

#### Instructions

Step 1: Check current umask

```bash
umask # Display current umask value
```

**Explanation:** `umask` is a mask that removes permissions from the default:
- Default file permissions: 666 (rw-rw-rw-)
- Default directory permissions: 777 (rwxrwxrwx)
- umask subtracts from these defaults
- Common values: 0022 (standard), 0077 (restrictive), 0002 (collaborative)

**Expected Output:** Typically shows `0022`.

#### Step 2: Create files with current umask and observe defaults

```bash
mkdir -p /tmp/umask-test # Create test files in a test directory
cd /tmp/umask-test # Change to /tmp/umask-test directory
touch file-default.txt # Create files with default permissions
mkdir dir-default/ # Create directory 
ls -la /tmp/umask-test/ # Check resulting permissions
```

**Explanation:** The files and directories created should match the umask calculation (666 or 777 minus the umask value).

**Expected Output:** `file-default.txt` has 644 permissions, `dir-default` has 755 permissions (with umask 0022).

#### Step 3: Change umask and create new files

```bash
umask 0077 # Change umask to 0077 (most restrictive – only owner has access)
touch file-restricted.txt # Create files with restricted umask
mkdir dir-restricted/ # Create directory 
ls -la /tmp/umask-test/ # Check resulting permissions
umask 0022 # Change umask back to default for comparison
touch file-default2.txt # Create file-default2.txt
ls -la /tmp/umask-test/ # Compare the results
```

**Explanation:** 
- With umask 0077: files are 600 (rw-------), dirs are 700 (rwx------)
- With umask 0022: files are 644 (rw-r--r--), dirs are 755 (rwxr-xr-x)
- umask changes only affect the current shell session

**Expected Output:** 
- `file-restricted.txt` shows 600 permissions
- `file-default2.txt` shows 644 permissions
- Demonstrates how umask affects file creation

---

### Scenario 5 — Extended Permissions with ACLs (Optional Advanced)

**Objective:** Understand basic Access Control Lists (ACLs) for granular permission control beyond standard Unix permissions.

**Deployment:** Uses the same RHEL 10 VM from Scenario 1.

#### Instructions

Step 1: Check if ACL support is enabled

```bash
getfacl ~/data/filename.txt # Display current ACLs on a test file
mount | grep acl # Check if the filesystem supports ACLs
```

**Explanation:** `getfacl` displays extended ACLs if enabled. Most modern filesystems support ACLs, but they must be mounted with the `acl` option.

**Expected Output:** Shows file owner, owning group, and other entries. ACL support is typically enabled by default on modern systems.

#### Step 2: Set ACLs for specific users/groups

```bash
sudo setfacl -m u:dev01:r ~/data/config.ini # Grant read permission to a specific user (dev01) on a file
sudo setfacl -m g:devteam:rw ~/data/test-script.sh # Grant write permission to the development team
getfacl ~/data/config.ini # Verify ACLs
getfacl ~/data/test-script.sh # Display ACL entries
```

**Explanation:**
- `setfacl -m` (modify) adds or updates ACL entries
- `u:username:permissions` – Set permissions for a specific user
- `g:groupname:permissions` – Set permissions for a specific group
- ACLs override traditional permissions for matching users/groups

**Expected Output:** `getfacl` shows additional ACL entries for the specified users and groups.

#### Step 3: Remove ACL entries

```bash
sudo setfacl -x u:dev01 ~/data/config.ini # Remove the ACL entry for dev01
sudo setfacl -b ~/data/test-script.sh # Clear all ACLs from a file
getfacl ~/data/config.ini # Verify removal
```

**Explanation:** 
- `setfacl -x` (remove) deletes specific ACL entries
- `setfacl -b` (clear) removes all extended ACLs

**Expected Output:** ACL entries are removed.

---

## Analytical Guidance

### Permission Analysis Best Practices

1. **Always verify before and after:** Use `ls -la` and `stat` to confirm permission changes. The `stat` command shows symbolic and numeric representations together for clarity.

2. **Understand the execute bit on directories:** On directories, execute (`x`) means "permission to enter/traverse" the directory. Without it, users cannot `cd` into the directory even if they have read permission. This is critical for securing sensitive directories like `/opt/configs/`.

3. **Special bits interaction:** When SGID and sticky bit are combined, both values stack:
   - 2775 = SGID + rwxrwxr-x
   - 3770 = SGID + sticky bit + rwxrwx---
   - 4755 = SUID + rwxr-xr-x (file only, rarely used)

4. **Recursive changes caution:** `chmod -R` applies the same permissions to all files and directories recursively. This can be problematic if you want directories to have execute permissions but files not to. Example: `chmod -R 644 /opt/project/` removes execute from all directories, making them inaccessible.

5. **Permission inheritance limitations:** Linux does not inherit permissions from parent directories to child files by default (only SGID affects group inheritance). Each file/directory has independent permissions.

6. **umask and scripts:** When scripts create files (e.g., application logs), ensure the umask allows the appropriate read/write permissions. A restrictive umask (0077) might cause scripts to create unreadable files.

### Common Permission Scenarios

- **Web directory:** `755` (owner: rwx, group: rx, others: rx) – Allows web server to serve files
- **Log files:** `644` (owner: rw, group: r, others: r) – Owner/admins write, others read
- **Sensitive configs:** `600` (owner: rw only) – Only owner can read sensitive data
- **Shared project dirs:** `2775` (SGID + rwxrwxr-x) – Team collaborates, others browse
- **Temp directories:** `1777` (sticky bit + rwxrwxrwx) – Anyone writes, only owner deletes

### umask Decision Tree

- **System/production user:** umask 0077 (most restrictive, only owner)
- **Developer/interactive user:** umask 0022 (group readable, very common)
- **Highly collaborative environment:** umask 0002 (group writable, rare, requires trust)

---

## Validation Criteria

| Scenario | Step | Expected Result | Command to Verify |
|----------|------|-----------------|-------------------|
| 1 | 1 | Default files created with 644, directories with 755 | `ls -la /tmp/lab05-permissions/project/` |
| 1 | 2 | app.py shows `-rwxr--r--`, config.ini shows `-rw-------` | `ls -la /tmp/lab05-permissions/project/` |
| 1 | 3 | secrets.env has 600, configs dir has 700 permissions | `stat /tmp/lab05-permissions/configs/secrets.env` |
| 2 | 1 | dev01 and admin01 users created with group memberships | `id dev01 && id admin01` |
| 2 | 2 | app.py owner is dev01, config.ini shows dev01:devteam | `ls -la /tmp/lab05-permissions/project/` |
| 2 | 3 | /opt/myapp structure shows correct ownership per directory | `ls -laR /opt/myapp/` |
| 3 | 1 | SGID bit set on shared-project (shows 's' in group execute) | `ls -la /opt/shared-project/` |
| 3 | 3 | Sticky bit set on temp-shared (shows 't' in others execute) | `ls -la /opt/temp-shared/` |
| 3 | 4 | collab-space shows both SGID and sticky bits | `stat /opt/collab-space/` |
| 4 | 1 | umask command returns 0022 or similar value | `umask` |
| 4 | 3 | Files created with umask 077 have 600 permissions | `ls -la /tmp/umask-test/file-restricted.txt` |
| 5 | 1 | getfacl command returns output without errors | `getfacl /tmp/lab05-permissions/project/app.py` |
| 5 | 2 | ACL entries show for dev01 and devteam | `getfacl /tmp/lab05-permissions/shared/notes.txt` |

---

## Documentation Expectations

As you work through this lab, document your findings in a report covering:

1. **Permission Notation Understanding**
   - Explain the difference between symbolic (rwx) and numeric (755) notation
   - Document three real-world scenarios where specific permissions are critical

2. **Ownership Changes**
   - Describe why `chown dev01:devteam` is preferred over `chown dev01` followed by `chgrp devteam`
   - Document the impact of recursively changing ownership on a directory tree

3. **Special Permissions**
   - Explain SGID behavior and why it matters for team collaboration
   - Describe sticky bit use cases and why `/tmp` typically has it

4. **Default Permissions and umask**
   - Calculate what permissions result from `umask 0037` applied to default 666 and 777
   - Document when a developer might change umask in a shell script

5. **Access Control**
   - Discuss limitations of standard Unix permissions
   - Describe one scenario where ACLs provide better security than standard permissions

---

## What Not To Do

- ❌ **Don't use `chmod 777` on sensitive directories** – This grants write access to all users, defeating security controls
- ❌ **Don't recursively chmod without understanding the impact** – `chmod -R 644` removes execute from directories, making them inaccessible
- ❌ **Don't assume umask is sticky** – umask changes only apply to files created in the current shell session
- ❌ **Don't mix symbolic and numeric notation in the same command** – Use one style consistently
- ❌ **Don't ignore the execute bit on directories** – Files inside inaccessible directories are unreachable regardless of file permissions
- ❌ **Don't forget that root bypasses permissions** – Root can read/write/execute any file; permissions primarily protect from regular users
- ❌ **Don't set SGID on files you don't understand** – SGID on executable files changes the effective user ID (security risk if misapplied)

---

## Real-World Context

**Multi-user Development Environment:**
A software development team uses a shared Linux server. The /opt/projects directory needs to support:
- Developers creating/modifying source files
- Automated build system reading code and writing artifacts
- QA team reviewing test results and logs
- Senior engineers auditing sensitive configuration

Without proper permissions:
- Developers' new files might be unreadable by the build system
- QA might inadvertently delete another developer's work
- Configuration files might be accidentally modified

**Solution Implementation:**
```bash
sudo mkdir -p /opt/projects/{src,build,qa,config} # Create directories with appropriate ownership and SGID
sudo chown -R :developers /opt/projects/src/ # Source code: developer team with read/write
sudo chmod 2775 /opt/projects/src/ # Set permissions to 2775 on 
sudo chown -R :developers /opt/projects/build/ # Build output: SGID for automatic team access
sudo chmod 2775 /opt/projects/build/ # Set permissions to 2775 on 
sudo chown -R :qa /opt/projects/qa/ # QA/test results: QA team owns, others read-only
sudo chmod 2750 /opt/projects/qa/ # Set permissions to 2750 on 
sudo chown root:admins /opt/projects/config/ # Configuration: only senior engineers, heavily restricted
sudo chmod 750 /opt/projects/config/ # Set permissions to 750 on 
```

This structure ensures:
- All team members' files automatically belong to the appropriate group (SGID)
- Permissions enforce read/write access based on role
- Configuration changes require admin privileges
- Team members can't accidentally delete others' work (proper permissions, not sticky bit here since you want team control)

**Azure Scenario:**
When deploying applications on Azure Linux VMs:
- Application service accounts need specific file ownership (not root)
- Logs should be writable by the service account, readable by monitoring tools
- Configuration files should be restricted to the service account and admins
- Temporary directories need sticky bits to prevent race conditions in multi-instance deployments

---

## Optional Advanced Exploration

1. **Fine-grained ACLs:** Implement ACL entries for multiple users/groups on a single file with different permissions

2. **Default ACLs:** Apply default ACLs to a directory so newly created files inherit permissions automatically
   ```bash
   sudo setfacl -d -m u:dev01:rw /opt/projects/src/ # Modify ACL entry
   ```

3. **Attribute-based access:** Explore `semanage` and SELinux contexts for context-based access control (RHEL-specific)

4. **File capabilities:** Investigate Linux capabilities as an alternative to SUID for privilege separation
   ```bash
   getcap /usr/sbin/ping
   ```

5. **Audit trail:** Use `auditd` to track permission changes and access attempts:
   ```bash
   sudo auditctl -w /etc/shadow -p wa -k shadow-changes
   ```

6. **Performance impact:** Test the performance difference between standard permissions and ACLs on large file trees
