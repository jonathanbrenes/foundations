# Lab 6 — Hard and Soft Links

## Title and Scenario

You are managing a file system where multiple access paths to the same files need to be maintained. Your application requires symbolic shortcuts to configuration files across different directories, while simultaneously needing duplicate references to critical data files that must remain accessible even if the original is accidentally moved. Understanding the differences between hard and soft links is essential for maintaining file integrity and creating flexible directory structures.

- **Estimated time:** 20-30 minutes.
- **VM count:** 1 (SLES 16).

---

## Deployment

[![Click to deploy](https://aka.ms/deploytoazurebutton)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fjonathanbrenes%2Ffoundations%2Fmain%2FModule%25202%2520-%2520OS%2520Fundamentals%2520II%2FLabs%2FFoundationsLab06.json)

---

## Skills Required

- Basic file system navigation (Lab 01-03)
- File permissions and ownership (Lab 05)
- Understanding of inode concept
- Ability to use `ls -l`, `stat`, and `find` commands

---

## Recommended Prerequisites

- Completion of Labs 01-05
- SSH access to the lab06sles VM
- Understanding of file ownership and permissions
- Familiarity with symbolic and numeric permission notation

---

## Objectives

After completing this lab, you will be able to:

1. **Understand hard links** – Create file replicas that share the same inode and content
2. **Understand soft links** – Create symbolic shortcuts to files and directories
3. **Compare link behavior** – Identify differences in inode sharing, filesystem boundaries, and deletion impacts
4. **Inspect link information** – Use `ls -l`, `stat`, and `find` to identify and analyze links
5. **Manage link dependencies** – Understand dangling links and link maintenance
6. **Identify system links** – Recognize built-in hard and soft links in the operating system

---

## Environment Overview

| Component | Value |
|---|---|
| **VM Name** | lab06sles |
| **Operating System** | SUSE Linux Enterprise Server (SLES) 16 |
| **VM Size** | Standard_B2s (2 vCPU, 4 GiB RAM) |
| **Admin Username** | azureuser |
| **Primary Network Interface** | lab06sles-nic1 (10.1.0.10) |
| **Public FQDN** | Assigned via Azure Portal |
| **Default Shell** | /bin/bash |

---

## Your Mission

Your mission is to create a comprehensive test environment demonstrating the differences between hard and soft links, and then use link concepts to build a practical file access structure. You will:

- Create hard links and observe their shared inode behavior
- Create soft links and verify their independent inode behavior
- Test link behavior when the original file is removed
- Establish a practical directory structure using links
- Identify real-world links in the operating system

---

### Scenario 1 — Creating and Inspecting Hard Links

**Objective:** Understand hard links as duplicate references to the same inode and file content.

**Deployment:** Uses the SLES 16 VM from the [Deployment](#deployment) section.

#### Instructions

Step 1: Create a test directory and source file

```bash
mkdir -p ~/lab06-links/{original,links} # Create a test directory
cd ~/lab06-links/original # Change to ~/lab06-links/original directory
echo "This is the original content" > original-file.txt # Create a source file with content
ls -li original-file.txt # Display the file and its inode information
stat original-file.txt # Display detailed file attributes
```

**Explanation:** The `ls -li` command displays the inode number (first column) along with file information. The `stat` command shows detailed inode information including the number of hard links (Links: counter).

**Expected Output:** 
- `ls -li` shows the inode number (e.g., 12345) and filename
- `stat` shows "Links: 1" (one hard link – the original file itself)

Step 2: Create hard links to the original file

```bash
ln ~/lab06-links/original/original-file.txt ~/lab06-links/original/hardlink1.txt # Create hard links to the original file
ln ~/lab06-links/original/original-file.txt ~/lab06-links/original/hardlink2.txt # Create link
ls -li ~/lab06-links/original/ # Display all files and verify they share the same inode
```

**Explanation:** 
- `ln <source> <hardlink>` creates a hard link without flags
- Hard links share the same inode number as the original file
- The link count increases for the original file (now shows "Links: 3")

**Expected Output:** All three files show the same inode number. Example:
```
12345 -rw-r--r-- 3 user user 29 Apr 27 10:15 original-file.txt
12345 -rw-r--r-- 3 user user 29 Apr 27 10:15 hardlink1.txt
12345 -rw-r--r-- 3 user user 29 Apr 27 10:15 hardlink2.txt
```

Step 3: Verify hard links share content and can be modified

```bash
echo "=== Original file ===" # Display content of each file
cat ~/lab06-links/original/original-file.txt # Display file contents
echo "=== Hardlink 1 ===" # Display output
cat ~/lab06-links/original/hardlink1.txt # Display file contents
echo "Modified content through hardlink1" > ~/lab06-links/original/hardlink1.txt # Modify content through one hard link
echo "=== After modification ===" # Verify all files show the modified content
cat ~/lab06-links/original/original-file.txt # Display file contents
cat ~/lab06-links/original/hardlink2.txt # Display file contents
stat ~/lab06-links/original/original-file.txt | grep Links # Display detailed file attributes
```

**Explanation:** Hard links share the same inode and data block. Modifying content through any hard link affects all references because they point to the same physical data.

**Expected Output:** All files display "Modified content through hardlink1". The link count remains 3 because all references point to the same inode.

Step 4: Test hard link behavior when the original is deleted

```bash
rm ~/lab06-links/original/original-file.txt # Remove the original file
echo "=== After removing original file ===" # Verify that hard links still contain the content
cat ~/lab06-links/original/hardlink1.txt # Display file contents
cat ~/lab06-links/original/hardlink2.txt # Display file contents
ls -li ~/lab06-links/original/ # Check link count
stat ~/lab06-links/original/hardlink1.txt | grep Links # Display detailed file attributes
```

**Explanation:** Deleting the "original" file merely removes one reference (link) to the inode. The data remains accessible through remaining hard links. The link count decrements but data persists.

**Expected Output:** Content is still accessible through hardlink1 and hardlink2. Link count is now 2 (both hard links remain).

Step 5: Understand hard link limitations

```bash
mkdir ~/lab06-links/testdir # Attempt to create a hard link to a directory (will fail)
ln ~/lab06-links/testdir ~/lab06-links/testdir-hardlink 2>&1 || echo "Cannot create hard link to directory" # Create link
ln ~/lab06-links/original/hardlink1.txt /tmp/cross-fs-hardlink 2>&1 || echo "May fail if /tmp is different filesystem" # Attempt to create hard link across filesystems (may fail); This depends on your mount points
stat ~/lab06-links/original/hardlink1.txt | head -n 5 # Verify with inode information
```

**Explanation:** 
- Hard links cannot be created for directories (kernel prevents this)
- Hard links are limited to the same filesystem (cannot cross filesystem boundaries)
- Soft links (covered in Scenario 2) solve both limitations

**Expected Output:** Error message attempting to create hard link to directory. Cross-filesystem attempt may fail depending on mount configuration.

---

### Scenario 2 — Creating and Inspecting Soft Links (Symbolic Links)

**Objective:** Understand soft links as symbolic shortcuts to file paths, which can reference files or directories across filesystems.

**Deployment:** Uses the same SLES 16 VM from Scenario 1.

#### Instructions

Step 1: Create soft links to the original files

```bash
cd ~/lab06-links/links # Navigate to the links directory
ln -s ~/lab06-links/original/hardlink1.txt symlink1.txt # Create soft links pointing to original files
ln -s ~/lab06-links/original/hardlink2.txt symlink2.txt # Create link
ln -s ~/lab06-links/original/hardlink1.txt symlink-to-hardlink1.txt # Also create a soft link to a file that still exists
ls -li ~/lab06-links/links/ # Display soft links and verify they have different inodes
```

**Explanation:** 
- `ln -s <source> <symlink>` creates a soft (symbolic) link
- The `-s` flag specifies symbolic link mode
- Soft links have their own inode number (different from the target)
- `ls -l` displays soft links with `->` notation showing the target path

**Expected Output:** Soft links show different inode numbers than the targets:
```
54321 lrwxrwxrwx 1 user user 35 Apr 27 10:20 symlink1.txt -> ~/lab06-links/original/hardlink1.txt
54322 lrwxrwxrwx 1 user user 35 Apr 27 10:20 symlink2.txt -> ~/lab06-links/original/hardlink2.txt
```

Step 2: Inspect soft link properties

```bash
echo "=== Hardlink (same inode as target) ===" # Compare inode information between hard and soft links
ls -li ~/lab06-links/original/hardlink1.txt # List files and details
echo "=== Soft link (different inode) ===" # Display output
ls -li ~/lab06-links/links/symlink1.txt # List files and details
echo "=== Stat on symlink (shows link properties) ===" # Use stat to examine the soft link itself
stat ~/lab06-links/links/symlink1.txt # Display detailed file attributes
echo "=== Stat on symlink target (with -L) ===" # Use stat with -L to follow the symlink to the target
stat -L ~/lab06-links/links/symlink1.txt # Display detailed file attributes
```

**Explanation:** 
- Soft links have their own inode; hard links share the target's inode
- `stat` on a soft link shows the link itself; `stat -L` follows the link to the target
- Soft link size is the length of the path string, not the target file size

**Expected Output:** 
- Hardlink and target show the same inode
- Symlink shows a different inode with size equal to path length (~35 bytes)
- `stat` vs `stat -L` shows different file sizes and modification times

Step 3: Access content through soft links

```bash
echo "=== Content via hardlink1 ===" # Read content through soft link
cat ~/lab06-links/original/hardlink1.txt # Display file contents
echo "=== Content via symlink1 ===" # Display output
cat ~/lab06-links/links/symlink1.txt # Display file contents
echo "Content modified via symlink" >> ~/lab06-links/links/symlink1.txt # Modify content through soft link
echo "=== After modification via symlink ===" # Verify modification is visible through all references
cat ~/lab06-links/original/hardlink1.txt # Display file contents
cat ~/lab06-links/links/symlink1.txt # Display file contents
```

**Explanation:** Soft links act as transparent shortcuts. Reading/writing through a soft link accesses the target file's content.

**Expected Output:** All references show the modified content. The soft link successfully redirects operations to the target file.

Step 4: Test soft link behavior when the target is deleted

```bash
rm ~/lab06-links/original/hardlink1.txt # Remove the target file (hardlink1.txt)
echo "=== Attempting to read through broken symlink ===" # Attempt to read through the soft link (will fail)
cat ~/lab06-links/links/symlink1.txt 2>&1 || echo "Error: No such file or directory (broken link)" # Display file contents
echo "=== Symlink still exists but is dangling ===" # The soft link still exists but is now dangling
ls -li ~/lab06-links/links/symlink1.txt # List files and details
ls -l ~/lab06-links/links/symlink1.txt # Display the broken link
```

**Explanation:** 
- When the target is deleted, soft links become "dangling" or "broken"
- The soft link file itself still exists (different inode)
- Attempting to access the target through a dangling link produces an error
- This contrasts with hard links, which retain data even after the original is deleted

**Expected Output:** `ls -l` shows the soft link but the arrow target is highlighted (often in red), indicating a broken link. `cat` produces "No such file or directory" error.

Step 5: Create soft links to directories

```bash
mkdir -p ~/lab06-links/original/config # Create a test directory
echo "database config" > ~/lab06-links/original/config/db.conf # Create files inside
echo "application config" > ~/lab06-links/original/config/app.conf # Display output
ln -s ~/lab06-links/original/config ~/lab06-links/links/config-shortcut # Create a soft link to the directory
echo "=== Listing through directory symlink ===" # Access files through the directory soft link
ls -la ~/lab06-links/links/config-shortcut/ # List files and details
echo "=== Reading file through directory symlink ===" # Display output
cat ~/lab06-links/links/config-shortcut/db.conf # Display file contents
echo "=== Attempting hard link to directory ===" # Try to create a hard link to the directory (should fail)
ln ~/lab06-links/original/config ~/lab06-links/original/config-hardlink 2>&1 || echo "Hard link to directory not allowed" # Create link
```

**Explanation:** 
- Soft links can reference directories; hard links cannot
- Accessing files through a soft link to a directory works transparently
- Hard links to directories are blocked by the kernel

**Expected Output:** Directory soft link allows listing and file access. Hard link attempt shows permission error.

---

### Scenario 3 — Creating Cross-Filesystem Links

**Objective:** Demonstrate the filesystem boundary limitations of hard links and capabilities of soft links.

**Deployment:** Uses the same SLES 16 VM from Scenario 1.

#### Instructions

Step 1: Identify available filesystems

```bash
mount | grep -E "ext4|xfs|btrfs|tmpfs" # Display mount points and filesystem types
df -h # Display filesystem usage
```

**Explanation:** Different filesystems (ext4, xfs, /tmp, etc.) may be mounted at different locations. Hard links cannot span filesystems; soft links can.

**Expected Output:** Lists mounted filesystems with their mount points and types.

Step 2: Create a file and attempt cross-filesystem hard link

```bash
touch ~/lab06-links/original/cross-fs-test.txt # Create a file in the original location
echo "Cross-filesystem test content" > ~/lab06-links/original/cross-fs-test.txt # Display output
echo "=== Attempting hard link to /tmp ===" # Attempt hard link to /tmp (if on different filesystem)
ln ~/lab06-links/original/cross-fs-test.txt /tmp/hardlink-tmp 2>&1 || echo "Hard link across filesystems failed (expected)" # Create link
ls -la /tmp/hardlink-tmp 2>&1 || echo "File does not exist (hard link failed)" # Verify the hard link failure
```

**Explanation:** Hard links work only within a single filesystem. Attempting to create a hard link across filesystem boundaries produces an "Invalid cross-device link" error.

**Expected Output:** Error message indicating cross-device link is not allowed. The hard link is not created.

Step 3: Create cross-filesystem soft link

```bash
echo "=== Creating soft link to /tmp ===" # Create soft link to /tmp (works across filesystems)
ln -s ~/lab06-links/original/cross-fs-test.txt /tmp/symlink-to-home # Create link
echo "=== Accessing content through cross-filesystem symlink ===" # Verify the soft link works
cat /tmp/symlink-to-home # Display file contents
ls -li /tmp/symlink-to-home # Check the soft link properties
```

**Explanation:** Soft links can reference files on different filesystems because they store the path as a string, not an inode reference.

**Expected Output:** Soft link is created successfully. Content is accessible through the symbolic link.

---

### Scenario 4 — Comparing Hard and Soft Links Side by Side

**Objective:** Build a practical directory structure using both hard and soft links and compare their behavior in a single exercise.

**Deployment:** Uses the same SLES 16 VM from Scenario 1.

#### Instructions

Step 1: Set up the test environment
- Create a shared project directory with a source file that multiple users need to access.
  ```bash
  mkdir -p ~/lab06-links/project/{shared,user-alice,user-bob} # Create directory {shared,user-alice,user-bob}
  echo "Project configuration v1.0" > ~/lab06-links/project/shared/config.txt # Display output
  ls -li ~/lab06-links/project/shared/config.txt # List files and details
  ```

  > **What to observe:** Note the inode number and link count (1) for the source file.

Step 2: Create a hard link for Alice and a soft link for Bob
- Give Alice a hard link and Bob a soft link to the same config file.
  ```bash
  ln ~/lab06-links/project/shared/config.txt ~/lab06-links/project/user-alice/config.txt # Create link
  ln -s ~/lab06-links/project/shared/config.txt ~/lab06-links/project/user-bob/config.txt # Create link
  ```

Step 3: Compare the inode and link information
- Inspect all three references side by side.
  ```bash
  echo "=== Shared (original) ===" # Display output
  ls -li ~/lab06-links/project/shared/config.txt # List files and details
  echo "=== Alice (hard link) ===" # Display output
  ls -li ~/lab06-links/project/user-alice/config.txt # List files and details
  echo "=== Bob (soft link) ===" # Display output
  ls -li ~/lab06-links/project/user-bob/config.txt # List files and details
  ```

  > **What to observe:** Alice's file shares the same inode as the original (link count is 2). Bob's file has a different inode and shows the `->` notation pointing to the original path.

Step 4: Modify content through each link type
- Write through Alice's hard link and verify the change is visible everywhere.
  ```bash
  echo "Updated by Alice" >> ~/lab06-links/project/user-alice/config.txt # Display output
  echo "=== Shared ===" # Display output
  cat ~/lab06-links/project/shared/config.txt # Display file contents
  echo "=== Bob (via soft link) ===" # Display output
  cat ~/lab06-links/project/user-bob/config.txt # Display file contents
  ```

  > **What to observe:** Both the original and Bob's soft link show Alice's change — hard links and soft links both reflect modifications to the underlying data.

Step 5: Simulate accidental deletion of the original
- Remove the shared config file and observe the impact on each link type.
  ```bash
  rm ~/lab06-links/project/shared/config.txt # Remove file or directory
  echo "=== Alice (hard link) ===" # Display output
  cat ~/lab06-links/project/user-alice/config.txt # Display file contents
  echo "=== Bob (soft link) ===" # Display output
  cat ~/lab06-links/project/user-bob/config.txt 2>&1 || echo "Broken: soft link target no longer exists" # Display file contents
  echo "=== Link details ===" # Display output
  ls -li ~/lab06-links/project/user-alice/config.txt # List files and details
  ls -li ~/lab06-links/project/user-bob/config.txt # List files and details
  ```

  > **What to observe:** Alice still has full access to the data (hard link preserves the inode). Bob's soft link is now dangling — the target path no longer exists, so `cat` fails. This is the key behavioral difference between the two link types.

Step 6: Restore the file from Alice's hard link
- Because Alice's hard link still holds the data, recreate the shared file from it.
  ```bash
  cp ~/lab06-links/project/user-alice/config.txt ~/lab06-links/project/shared/config.txt
  echo "=== Bob (soft link restored) ===" # Display output
  cat ~/lab06-links/project/user-bob/config.txt # Display file contents
  echo "=== Final state ===" # Display output
  ls -li ~/lab06-links/project/shared/config.txt # List files and details
  ls -li ~/lab06-links/project/user-alice/config.txt # List files and details
  ls -li ~/lab06-links/project/user-bob/config.txt # List files and details
  ```

  > **What to observe:** After copying the file back to the original path, Bob's soft link works again. Note that the shared file now has a new inode — it is a separate copy, not linked to Alice's file anymore. Alice's hard link still points to the old inode with link count 1.

---

## Analytical Guidance

### Hard Links vs Soft Links Comparison

| Characteristic | Hard Link | Soft Link |
|---|---|---|
| **Syntax** | `ln source hardlink` | `ln -s source symlink` |
| **Inode** | Same as target | Different from target |
| **Content** | Shares target data | Stores path string |
| **Filesystem** | Cannot cross | Can cross |
| **Directory target** | Not allowed | Allowed |
| **Broken links** | Cannot occur (data persists) | Possible (dangling) |
| **Link count** | Increments target's count | Does not increment |
| **Space usage** | Increases link count, no extra space | Small (path size) |
| **Symbolic representation** | Appears identical to original | Shows `->` notation |

### When to Use Hard Links

- **Backup strategies**: Create alternate references to important files while retaining content if any reference is deleted
- **Data consistency**: Ensure multiple access paths always reference the same content
- **Within filesystem**: When you need multiple names for the same file on the same filesystem

### When to Use Soft Links

- **Cross-filesystem references**: Link files across different mounted filesystems
- **Directory shortcuts**: Create shortcuts to frequently used directories
- **Configuration management**: Link to alternative implementations (e.g., `/etc/alternatives`)
- **Flexible paths**: Link to files that might move; update the soft link target without changing its location
- **Clear references**: The `->` notation makes it obvious that it's a shortcut

### Link Management Best Practices

1. **Use `ls -i` to identify actual inode references**: Understand what you're linking to
2. **Verify link targets before deletion**: Ensure you're not breaking dependent links
3. **Monitor broken links**: Periodically check for dangling symlinks in critical directories
4. **Document link purposes**: Maintain records of why links were created
5. **Avoid circular links**: Prevent symlinks that reference other symlinks that reference the original
6. **Secure soft link targets**: Ensure soft link targets have appropriate permissions

---

## Validation Criteria

| Scenario | Step | Expected Result | Command to Verify |
|----------|------|-----------------|-------------------|
| 1 | 1 | Source file created with inode and link count 1 | `stat ~/lab06-links/original/original-file.txt \| grep Links` |
| 1 | 2 | All three files share same inode, link count 3 | `ls -i ~/lab06-links/original/ \| awk '{print $1}' \| sort -u` |
| 1 | 3 | Modification through any hard link visible in all | `cat ~/lab06-links/original/hardlink1.txt` (same as others) |
| 1 | 4 | Content accessible after removing original | `cat ~/lab06-links/original/hardlink1.txt` succeeds |
| 1 | 5 | Hard link to directory fails | Error message from `ln ~/lab06-links/testdir ...` |
| 2 | 1 | Soft links created with different inode | `ls -i ~/lab06-links/original/hardlink1.txt` ≠ `ls -i ~/lab06-links/links/symlink1.txt` |
| 2 | 3 | Content accessible through soft link | `cat ~/lab06-links/links/symlink1.txt` succeeds |
| 2 | 4 | Broken symlink exists, target inaccessible | `ls ~/lab06-links/links/symlink1.txt` succeeds, `cat` fails |
| 2 | 5 | Directory symlink allows file access | `cat ~/lab06-links/links/config-shortcut/db.conf` succeeds |
| 3 | 2 | Hard link across filesystems fails | Error from `ln /home/... /tmp/...` |
| 3 | 3 | Soft link across filesystems succeeds | `cat /tmp/symlink-to-home` succeeds |
| 4 | 3 | Hard link shares inode, soft link has different inode | `ls -i` shows same inode for Alice, different for Bob |
| 4 | 5 | Hard link survives deletion, soft link breaks | `cat` succeeds for Alice, fails for Bob |
| 4 | 6 | Restored file gives Bob access, new inode for shared | `cat` succeeds for Bob, `ls -i` shows new inode |

---

## Documentation Expectations

As you work through this lab, document your findings in a report covering:

1. **Hard Link Behavior**
   - Explain why modifying content through one hard link affects all references
   - Describe what happens to the file data when the original is deleted but hard links remain

2. **Soft Link Behavior**
   - Compare inode numbers between a soft link and its target
   - Explain what creates a "dangling" or "broken" soft link and how to identify it

3. **Filesystem Boundaries**
   - Document which type of link (hard or soft) can cross filesystem boundaries
   - Explain the technical reason for this difference

4. **System Links**
   - Identify three examples of soft links used by the operating system
   - Describe the purpose of at least one soft link you discovered

5. **Real-World Application**
   - Design a scenario where hard links would be beneficial (e.g., backup strategy)
   - Design a scenario where soft links would be the better choice (e.g., configuration management)

---

## What Not To Do

- ❌ **Don't assume all files with the same name share content** – Without the same inode, they are separate files
- ❌ **Don't create hard links to critical system files without understanding dependencies** – It may complicate system maintenance
- ❌ **Don't ignore broken soft links** – They clutter the filesystem and can indicate missing dependencies
- ❌ **Don't expect hard links to cross filesystems** – Use soft links for cross-filesystem references
- ❌ **Don't create circular soft links** – Avoid symlinks referencing other symlinks in a loop
- ❌ **Don't modify soft link permissions expecting to affect the target** – Permissions on the link and target are independent
- ❌ **Don't rely on `rm` behavior to remove all references** – Hard link deletion only removes one reference; data persists with remaining links

---

## Real-World Context

**Configuration Management System:**
An organization maintains multiple application environments (development, staging, production). Each environment has configuration files:

```bash
/opt/config/production/app.conf # Store actual configs in a version-controlled directory
/opt/config/staging/app.conf
/opt/config/development/app.conf
/etc/app.conf -> /opt/config/production/app.conf # Create soft links for easy access and swapping
ln -sf /opt/config/staging/app.conf /etc/app.conf # Switching environments requires only link change
```

**Backup Strategy Using Hard Links:**
A backup system creates multiple snapshots while conserving space:

```bash
/data/project/files/ # Original data directory
ln /data/project/files/file1.txt /backups/2024-04-27/file1.txt # Backup creates hard links to unchanged files; New files in backup get hard links
```

**Distribution Package System:**
Package managers (apt, yum, zypper) use soft links for:

```bash
/usr/bin/editor -> /usr/bin/vim # Multiple alternatives for same functionality
/usr/bin/editor -> /usr/bin/nano  (can be switched)
/etc/alternatives/java -> /usr/lib/jvm/java-11-openjdk/bin/java # Update-alternatives system uses this pattern
```

---

## Optional Advanced Exploration

1. **Hard link behavior with inode exhaustion**: Create many hard links and monitor inode usage with `df -i`

2. **Soft link chain resolution**: Create symlinks referencing other symlinks and trace path resolution

3. **Link preserving in file operations**: Test how `cp -P` (preserve links) differs from `cp` with hard/soft links

4. **Extended attributes on links**: Use `getfattr` and `setfattr` to examine attributes on hard vs soft links

5. **Hard link distribution**: Analyze file systems using hard links extensively (e.g., `/etc/alternatives`)
