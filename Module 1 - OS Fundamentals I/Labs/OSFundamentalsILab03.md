# Lab 3 — File Operations, Manipulating and Finding Files

## Title and Scenario

This lab covers fundamental file and directory operations on a Linux system. You will navigate the filesystem, create and delete directories, create and find files, copy and move files between directories, use redirection and concatenation to manipulate text files, and use advanced find options to locate and act on files.

This is a guided, foundational lab. All steps are provided with expected commands and validation points.

- **Estimated time:** 30 minutes.
- **VM count:** 1 (RHEL 9).

---

## Deployment

Deploy the dedicated Lab 03 VM using the template below.

[![Click to deploy](https://aka.ms/deploytoazurebutton)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fjonathanbrenes%2Ffoundations%2Fmain%2FModule%25201%2520-%2520OS%2520Fundamentals%2520I%2FLabs%2FFoundationsLab03.json)

> This template deploys a RHEL 9 VM (`lab03rhel`) with password authentication.

> **Warning:** The VMs deployed in these labs allow inbound traffic automatically from the Azure VPN (AzureCloud service tag). Make sure you are connected to the Azure VPN, or add an NSG rule to allow your public IP address before attempting to connect.

---

## Skills Required

- Ability to connect to a Linux VM via SSH.
- Basic familiarity with running commands in a Linux terminal.

---

## Recommended Prerequisites

- Lab 1 completed — WSL2 or Cloud Shell configured, Azure CLI available, SSH connectivity understood.
- Lab 2 completed — familiarity with `sudo` and switching to root.
- Lab 03 VM deployed and accessible.

---

## Objectives

After completing this lab, you will be able to:

- Navigate through a filesystem, create and destroy directories.
- Create and find files based on name or permissions.
- Copy, rename, and move files between directories.
- Execute actions on found files using `find` with `-exec`.
- Manipulate text files by redirecting, concatenating, and appending outputs into files.
- Locate files quickly using `updatedb` and `locate`.

---

## Environment Overview

| Component | Details |
|---|---|
| VM | RHEL 9 (`lab03rhel`) |
| Access | SSH as `azureuser` |
| Working directory | `~/files-lab` (created during the lab) |

---

## Your Mission

Complete all steps in order. You will work as `azureuser` for most steps. Some steps require `sudo` for elevated privileges.

### Scenario 1 — Managing files

Connect to the `lab03rhel` VM before starting.

```bash
ssh azureuser@<VM_PUBLIC_IP> # Connect to the RHEL 9 VM using SSH
```

#### Instructions

Step 1: Create an empty file inside a new directory
- Create a directory called `files-lab` in the home directory and create an empty file called `test.txt` inside it.
  ```bash
  pwd # Print current directory
  cd ~ # Change directory to user's home
  mkdir files-lab # Create a directory called files-lab
  cd files-lab # Change directory into the newly created files-lab
  touch test.txt # Create an empty file called test.txt
  # Alternative method using redirection:
  # echo '' > test.txt
  ls -la # List with details all the files inside the current directory
  ```

  > The directory `~` refers to the current user's home directory.

Step 2: Create files using redirection
- Inside the same directory, create files using redirection. The operator `>` will overwrite the file, and `>>` will append the output to the file.
  ```bash
  cd ~/files-lab # Change directory to files-lab
  ls -la > test2.txt # Redirect the output of ls -la into a new file called test2.txt; overwrites if file exists
  echo "This is a test" > test3.txt # Redirect the text string into a new file called test3.txt
  cat test2.txt test3.txt > merged.txt # Concatenate the contents of test2.txt and test3.txt into merged.txt
  ls -la >> merged.txt # Append the output of ls -la to the end of merged.txt without overwriting
  cat merged.txt # Display the full contents of merged.txt
  ```

Step 3: Copy and move files
- Use `cp` and `mv` to copy, move, and rename files. Copying to `/etc` requires elevated permissions with `sudo`.
  ```bash
  cp -p test.txt test-copy.txt # Copy test.txt preserving permissions and timestamps (-p)
  mv test-copy.txt ~/ # Move the copy to the home directory
  cd ~ # Change to the home directory
  ls -la # List files to confirm test-copy.txt is present
  mv test-copy.txt TEST.txt # Rename test-copy.txt to TEST.txt
  cp TEST.txt test-backup.txt # Copy TEST.txt to test-backup.txt
  sudo mv test-backup.txt /etc # Move test-backup.txt to /etc; requires sudo for writing to a system directory
  ls -l /etc/test-backup.txt # Verify the file exists in /etc
  ```

  > `sudo` may or may not ask for your password depending on the sudoers configuration.

Step 4: Finding files in the system
- Use the `find` command to locate files by name.
  ```bash
  find / -name test-backup.txt 2> /dev/null # Search the entire filesystem for test-backup.txt; redirect permission errors to /dev/null
  sudo find / -name test-backup.txt # Same search with root privileges; no permission errors
  sudo cp /etc/test-backup.txt /etc/test-BACKUP.txt # Create a copy with different casing
  sudo find / -iname test-backup.txt # Search ignoring case (-iname); finds both test-backup.txt and test-BACKUP.txt
  ```

  > `sudo` may be required to find files if the root user is not being used, to avoid permission denied errors.

  > The option `-iname` can be used to perform a case-insensitive search.

Step 5: Find all `.txt` files on the system
- Use `find` with a wildcard pattern and pipe through `more` to paginate the results.
  ```bash
  sudo find / -name "*.txt" | more # Search for all files ending in .txt and paginate the output with more
  ```

Step 6: Execute actions on found files
- Use the `find` command with `-exec` to change permissions on a found file in a single command.
  ```bash
  sudo find / -name test-BACKUP.txt -exec chmod 777 {} \; # Find the file and execute chmod 777 on it; {} is replaced by the found filename
  sudo ls -la /etc/test-BACKUP.txt # Verify the permissions were changed to rwxrwxrwx
  ```

  > The `-exec` option runs a command on each file found. The `{}` placeholder is replaced by the current filename. The `\;` terminates the `-exec` expression.

Step 7: Find executable files
- Use the `find` command to display all executable files in the `/usr` directory.
  ```bash
  sudo find /usr -type f -executable # Find regular files (-type f) that are executable in /usr
  ```

Step 8: Delete the lab directory
- Attempt to remove the lab directory using `rmdir` while in your home directory.
  ```bash
  cd ~ # Change to home directory first
  rmdir files-lab # Attempt to remove the directory; this should fail because it is not empty
  rm -r files-lab # Use recursive removal after the expected rmdir failure
  ```

  > `rmdir` only works if the directory is empty. This failure is expected and is part of the lab outcome.

  > After observing that behavior, use `rm -r` to remove the non-empty `files-lab` directory.

Step 9: Use `updatedb` and `locate`
- Update the locate database and run quick filename searches.
  ```bash
  sudo updatedb # Refresh the filename database used by locate
  locate test-backup.txt # Search for the lowercase filename
  locate TEST.txt # Search for the uppercase filename in home
  ```

  > `locate` is fast because it reads a database. Run `updatedb` first so recent file changes are included.

---

## Analytical Guidance

- **Step 2:** Understand the critical difference between `>` (overwrite) and `>>` (append). Using `>` on an existing file destroys its previous content.
- **Step 3:** The `-p` flag in `cp` preserves the original file's permissions, ownership, and timestamps. Without it, the copy inherits the current user's default permissions.
- **Step 4:** Redirecting `stderr` with `2> /dev/null` is a common pattern to suppress permission denied errors when running `find` as a non-root user.
- **Step 6:** The `find -exec` pattern is powerful but can be dangerous with destructive commands. Always verify the `find` results without `-exec` first before adding it.
- **Step 8:** `rmdir` only removes empty directories. Its failure on `files-lab` is expected because the directory still contains files.
- **Step 9:** `locate` searches a cached database and is usually much faster than `find` for name-only lookups.

---

## Validation Criteria

| Step | Expected result |
|---|---|
| Directory created | `ls ~/files-lab` lists the directory contents |
| Empty file created | `ls -la ~/files-lab/test.txt` shows a 0-byte file |
| Redirection works | `cat ~/files-lab/merged.txt` contains output from both `test2.txt` and `test3.txt` |
| Append works | `cat ~/files-lab/merged.txt` shows the `ls -la` output at the end |
| File copied to `/etc` | `ls -l /etc/test-backup.txt` shows the file |
| Case-insensitive find | `sudo find / -iname test-backup.txt` returns both `/etc/test-backup.txt` and `/etc/test-BACKUP.txt` |
| Permissions changed | `ls -la /etc/test-BACKUP.txt` shows `rwxrwxrwx` |
| `rmdir` behavior observed | `rmdir files-lab` fails because the directory is not empty |
| Locate results available | `locate test-backup.txt` and `locate TEST.txt` return matching paths |

---

## Documentation Expectations

As you complete this lab, take note of:

- The difference between `>` (overwrite) and `>>` (append) redirection.
- The behavior of `cp -p` versus `cp` without flags.
- The syntax of `find -exec` and what `{}` and `\;` mean.
- Why `rmdir` fails on non-empty directories.
- Why `updatedb` should be run before using `locate` for recent changes.

---

## What Not To Do

- Do not use `rm -rf /` or any variation targeting the root filesystem.
- Do not delete `/etc/test-backup.txt` or `/etc/test-BACKUP.txt` — they may be referenced in validation.
- Do not skip the `rmdir` step — observing its failure on a non-empty directory is part of the learning objective.
- Do not delete the VM after this lab.

---

## Real-World Context

File navigation, redirection, and the `find` command are among the most commonly used tools in daily Linux administration. In production environments, engineers routinely use redirection to capture command output into log files and `find -exec` to perform bulk operations on files (such as changing permissions or ownership across directory trees). Understanding why `rmdir` fails on non-empty directories is a practical safeguard against accidental deletion mistakes on shared systems.

---

## Optional Advanced Exploration

- Use `find` with `-perm` to search for files with specific permissions:
  ```bash
  sudo find /etc -perm 777 # Find all files in /etc with permissions set to 777
  ```
- Use `find` with `-mtime` to locate files modified in the last 24 hours:
  ```bash
  find ~ -mtime -1 # Find files in the home directory modified within the last day
  ```
- Experiment with `tee` to write output to both a file and the terminal simultaneously:
  ```bash
  ls -la | tee output.txt # Display ls output on screen and save it to output.txt at the same time
  ```
- Use `xargs` as an alternative to `-exec` for bulk operations:
  ```bash
  find /tmp -name "*.tmp" | xargs ls -l # Pass all found .tmp files as arguments to ls -l
  ```
