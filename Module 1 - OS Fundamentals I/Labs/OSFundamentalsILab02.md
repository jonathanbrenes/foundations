# Lab 2 — User and Group Administration

## Title and Scenario

This lab covers the fundamentals of user and group management on a Linux system. You will create and delete users, manage passwords, configure SSH password authentication, work with user expiration, and organize users into groups using primary and secondary group assignments.

This is a guided, foundational (101) lab. All steps are provided with expected commands and validation points.

- **Estimated time:** 30 minutes.
- **VM count:** 1 (RHEL 8.10).

---

## Deployment

Deploy the dedicated Lab 02 VM using the template below.

[![Click to deploy](https://aka.ms/deploytoazurebutton)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fjonathanbrenes%2Ffoundations%2Fmain%2FModule%25201%2520-%2520OS%2520Fundamentals%2520I%2FLabs%2FFoundationsLab02.json)

> This template deploys a RHEL 8.10 VM with password authentication.

> **Warning:** The VMs deployed in these labs allow inbound traffic automatically from the Azure VPN (AzureCloud service tag). Make sure you are connected to the Azure VPN, or add an NSG rule to allow your public IP address before attempting to connect.

---

## Skills Required

- Ability to connect to a Linux VM via SSH.
- Basic familiarity with running commands in a Linux terminal.

---

## Recommended Prerequisites

- Lab 1 completed — WSL2 or Cloud Shell configured, Azure CLI available, and SSH connectivity understood.
- Lab 02 VM deployed and accessible.

---

## Objectives

After completing this lab, you will be able to:

- List users on the system using `/etc/passwd` and `getent`.
- Create and delete users with `useradd` and `userdel`.
- Set passwords for users individually and in batch mode.
- Validate and change the SSH daemon configuration to allow password authentication.
- Check and change expiration dates for user accounts.
- Create groups and assign users to primary and secondary groups.

---

## Environment Overview

| Component | Details |
|---|---|
| VM | RHEL 8.10 (`lab02rhel`) |
| Access | SSH as `azureuser` with password authentication initially |
| Privilege escalation | `sudo -i` to switch to root |

---

## Your Mission

Complete all steps in order. You will work as root throughout this lab.

### Scenario 1 — User and group administration

Connect to the `lab02rhel` VM and switch to root before starting.

```bash
ssh azureuser@<VM_PUBLIC_IP> # Connect to the RHEL VM using SSH
sudo -i # Switch to the root user for full administrative privileges
whoami # Confirm current user is root
id # Confirm root UID/GID context
```

#### Instructions

Step 1: List users through `/etc/passwd`
- List the users created on the system by reading the password file directly.
  ```bash
  cat /etc/passwd # Display the contents of the password file; each line represents one user account
  ```

Step 2: List users through `getent`
- List the users using the `getent` command, which queries the system's name service databases.
  ```bash
  getent passwd # Query the system name service database for all user entries; includes local and external sources
  ```

Step 3: Create users
- Create users `user01`, `user02`, `user03`, and `user04` using a loop.
  ```bash
  for user in user01 user02 user03 user04; do useradd -m "$user"; done
  for user in user01 user02 user03 user04; do getent passwd "$user"; done
  ```

Step 4: Set passwords for the created users
- Set passwords for each user.
  ```bash
  passwd user01 # Set password interactively
  passwd user02
  passwd user03
  passwd user04

  # Non-interactive loop form (RHEL):
  for user in user01 user02 user03 user04; do echo "MyP@ssW0rd" | passwd --stdin "$user"; done
  ```

  > Use a stronger password than `MyP@ssW0rd` outside lab environments.

Step 5: Enable password authentication for SSH
- If the system was set up with SSH key authentication, password authentication may be disabled. Enable it so the newly created users can log in.
  ```bash
  sed -i 's/PasswordAuthentication no/PasswordAuthentication yes/' /etc/ssh/sshd_config # Find and replace the PasswordAuthentication value in the SSH config file; -i edits the file in place
  systemctl restart sshd # Restart the SSH daemon so it reads the updated configuration
  ssh user01@localhost # Test password login locally as user01
  ```

  In the previous commands:
  - `sed` filters the `/etc/ssh/sshd_config` configuration file, changing the line `PasswordAuthentication no` to `PasswordAuthentication yes`.
  - `systemctl restart sshd` restarts the SSH service to apply the configuration change.

Step 6: Set password expiry to 60 days
- Set maximum password age to 60 days for all four users and verify.
  ```bash
  for user in user01 user02 user03 user04; do chage -M 60 "$user"; done
  for user in user01 user02 user03 user04; do chage -l "$user"; done
  ```

  > Use `chage -l <username>` to verify each account's max password age.

Step 7: Delete users
- Remove one user without `-r` and observe that the home directory remains.
  ```bash
  userdel user01 # Delete account only
  ls -ld /home/user01 # Home directory still exists
  ```

Step 8: Delete a user with `-r`
- Remove another user with `-r` and observe that the home directory is removed.
  ```bash
  userdel -r user02 # Delete account and home directory
  ls -ld /home/user02 # Should report no such file or directory
  ```

Step 9: Review existing groups
- Check group entries and user-related groups.
  ```bash
  cat /etc/group # Show all groups
  getent group # Alternate query via name service
  ```

Step 10: Create required groups
- Create the `azure-network` and `azure-identity` groups.
  ```bash
  groupadd azure-network
  groupadd azure-identity
  ```

Step 11: Recreate deleted users
- Recreate the users from Step 3 so all four accounts exist again.
  ```bash
  for user in user01 user02 user03 user04; do id "$user" >/dev/null 2>&1 || useradd -m "$user"; done

  # If a home directory existed from a prior non--r deletion, fix ownership
  for user in user01 user02 user03 user04; do [ -d "/home/$user" ] && chown -R "$user:$user" "/home/$user"; done
  ```

Step 12: Add user01 and user02 to `azure-network`
- Assign supplementary group membership.
  ```bash
  usermod -aG azure-network user01
  usermod -aG azure-network user02
  ```

Step 13: Add user03 and user04 to `azure-identity`
- Assign supplementary group membership.
  ```bash
  usermod -aG azure-identity user03
  usermod -aG azure-identity user04
  id user01
  id user02
  id user03
  id user04
  ```

Step 14: Switch to user01 and create a file
- While root, switch to user01, create `testfile1`, and inspect ownership.
  ```bash
  su - user01
  touch testfile1
  ls -l testfile1
  id
  ```

Step 15: Exit back to root
- Leave the user01 shell and return to root.
  ```bash
  exit
  whoami
  ```

---

## Analytical Guidance

- **Step 1 vs Step 2:** Both `cat /etc/passwd` and `getent passwd` display user information, but `getent` also queries external sources like LDAP or SSSD. On a local-only system the output is identical.
- **Step 4:** `passwd` is interactive, while `passwd --stdin` is useful for scripted lab execution on RHEL.
- **Step 5:** Modifying `sshd_config` and restarting the service is a common operational task. Be aware that enabling password authentication reduces security if weak passwords are used.
- **Step 6:** Password aging with `chage -M 60` enforces rotation. This is different from hard account expiry (`chage -E`).
- **Step 7/8:** The difference between `userdel` and `userdel -r` is critical. Orphaned home directories with numeric UID/GID ownership are a common finding in security audits.
- **Step 11:** Recreating a user whose old home directory was preserved may require `chown -R user:user /home/user` before that account can write files.

---

## Validation Criteria

| Step | Expected result |
|---|---|
| Privilege escalation | `whoami` returns `root` after `sudo -i` |
| Users listed | `getent passwd user01` returns user details after creation |
| Users created | `ls /home` shows `user01`, `user02`, `user03`, `user04` directories |
| Passwords set | `passwd --stdin` reports successful token updates |
| SSH password auth | `grep PasswordAuthentication /etc/ssh/sshd_config` shows `yes` |
| Local login test | `ssh user01@localhost` accepts password when password auth is enabled |
| Expiration set | `chage -l user01` (and others) shows max age of 60 days |
| Non-`-r` deletion | `getent passwd user01` returns nothing; `/home/user01` still exists |
| `-r` deletion | `getent passwd user02` returns nothing; `/home/user02` is gone |
| Groups created | `getent group azure-network` and `getent group azure-identity` return entries |
| Groups assigned | `id user01`/`id user02` include `azure-network`; `id user03`/`id user04` include `azure-identity` |
| File ownership check | `su - user01` then `ls -l testfile1` shows owner `user01` |

---

## Documentation Expectations

As you complete this lab, take note of:

- The difference between `userdel` and `userdel -r` and the impact on the home directory.
- How `passwd` differs from `passwd --stdin` for interactive vs scripted password changes.
- The location and format of `/etc/passwd`, `/etc/group`, and `/etc/shadow`.
- The 60-day password aging policy shown in `chage -l`.

---

## What Not To Do

- Do not use hardcoded passwords in scripts that will be committed to a repository.
- Do not delete all lab users at the end — keep accounts needed for validation and follow-on exercises.
- Do not disable SSH key-based authentication while enabling password authentication. Both methods can coexist.
- Do not delete the VM after this lab.

---

## Real-World Context

User and group management is a daily task for Linux system administrators. In production environments, user creation is often automated through configuration management tools like Ansible or Puppet, but understanding the underlying commands (`useradd`, `usermod`, `groupadd`, `chage`) is essential for troubleshooting and auditing. The `chage` command is frequently used to enforce compliance policies requiring account expiration for temporary employees or contractors. Understanding the relationship between users, groups, and home directory ownership is fundamental to Linux permission management.

---

## Optional Advanced Exploration

- Examine the `/etc/shadow` file and identify the fields that correspond to password aging:
  ```bash
  getent shadow user03
  ```
- Set a maximum password age of 90 days for `user04`:
  ```bash
  chage -M 90 user04
  chage -l user04
  ```
- Create a user with a specific UID and primary group:
  ```bash
  useradd -u 2000 -g azure-network user05
  id user05
  ```
- Test logging in as `user03` via SSH from your local session to verify password authentication works end to end.
