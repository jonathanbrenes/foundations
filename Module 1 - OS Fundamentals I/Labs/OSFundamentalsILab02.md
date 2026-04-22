# Lab 2 — User and Group Administration

## Title and Scenario

This lab covers the fundamentals of user and group management on a Linux system. You will create and delete users, manage passwords, configure SSH password authentication, work with user expiration, and organize users into groups using primary and secondary group assignments.

This is a guided, foundational (101) lab. All steps are provided with expected commands and validation points.

- **Estimated time:** 30 minutes.
- **VM count:** 1 (Ubuntu 24.04 LTS from Lab 1).

---

## Deployment

This lab reuses the Ubuntu VM (`lab01ubuntu`) deployed in Lab 1. No additional deployment is required.

> **Warning:** The VMs deployed in these labs allow inbound traffic automatically from the Azure VPN (AzureCloud service tag). Make sure you are connected to the Azure VPN, or add an NSG rule to allow your public IP address before attempting to connect.

---

## Skills Required

- Ability to connect to a Linux VM via SSH (covered in Lab 1).
- Basic familiarity with running commands in a Linux terminal.

---

## Recommended Prerequisites

- Lab 1 completed — WSL2 or Cloud Shell configured, SSH key-based authentication working.
- The Ubuntu VM (`lab01ubuntu`) running and accessible.

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
| VM | Ubuntu 24.04 LTS (`lab01ubuntu`) from Lab 1 |
| Access | SSH as `azureuser` with key-based authentication |
| Privilege escalation | `sudo -i` to switch to root |

---

## Your Mission

Complete all steps in order. You will work as root throughout this lab.

### Scenario 1 — User and group administration

Connect to the `lab01ubuntu` VM and switch to root before starting.

```bash
ssh azureuser@<VM_PUBLIC_IP> # Connect to the Ubuntu VM using SSH key-based authentication
sudo -i # Switch to the root user for full administrative privileges
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

Step 3: Create users with a loop
- Using a `for` loop, create four users and retrieve information for each one.
  ```bash
  for user in user01 user02 user03 user04   # loop created
  do
    useradd -m $user # Create user
    getent passwd $user # Retrieve and display user information
  done
  ```

  In the previous command:
  - The `for` loop assigns a variable called `user` and iterates over a list of usernames (`user01`, `user02`, `user03`, `user04`).
  - `useradd -m $user` creates a new user for each iteration. The `-m` flag ensures the home directory is created, which is not the default behavior on all distributions.
  - `getent passwd $user` retrieves and displays details about the user in the current iteration.

Step 4: Set passwords for the created users
- Using a similar loop, set a password for all users. Replace `<your-secure-password>` with your own password.
  ```bash
  for user in user01 user02 user03 user04 # loop created
  do
  echo "$user:<your-secure-password>" | chpasswd # echo command will repeat what we have inside the quotes.   Adding the pipe will be executing the next command which is _chpasswd_ to the output.  So with each iteraction, we're changing the password for that iteraction with the contents of the quotes in echo command.
  done
  ```

  > For batch password changes on Ubuntu and Debian, use `chpasswd`. For individual password changes, `passwd` is recommended — it will prompt interactively.

  ```bash
  passwd user01 # Change the password for user01 interactively; the system will prompt for the new password twice
  ```

  > On other distributions, `passwd` can also accept input via the `--stdin` flag.

Step 5: Enable password authentication for SSH
- If the system was set up with SSH key authentication, password authentication may be disabled. Enable it so the newly created users can log in.
  ```bash
  sed -i 's/PasswordAuthentication no/PasswordAuthentication yes/' /etc/ssh/sshd_config # Find and replace the PasswordAuthentication value in the SSH config file; -i edits the file in place
  systemctl restart sshd # Restart the SSH daemon so it reads the updated configuration
  ```

  In the previous commands:
  - `sed` filters the `/etc/ssh/sshd_config` configuration file, changing the line `PasswordAuthentication no` to `PasswordAuthentication yes`.
  - `systemctl restart sshd` restarts the SSH service to apply the configuration change.

Step 6: Check and change user expiration
- Check the current expiration and password aging information for `user03`.
  ```bash
  chage -l user03 # Display password aging and expiration information for user03
  ```

- Set the account to expire on a specific date.
  ```bash
  chage -E 2026-12-31 user03 # Set the account expiration date; the user will not be able to log in after this date
  ```

- Verify the change took effect.
  ```bash
  chage -l user03 # Confirm the expiration date was updated
  ```

  > The `chage` command manages password aging and account expiration policies. The `-E` flag sets the account expiration date. This is commonly used in production to enforce temporary access or contractor account policies.

Step 7: Delete users
- Delete `user01` and `user02`. Use the `-r` flag with the second user and observe the difference.
  ```bash
  userdel user01 # Delete user01 but keep the home directory
  userdel -r user02 # Delete user02 and remove the home directory (-r)
  ls -l /home # List home directories to observe the difference
  ```

  > The home directory is kept if the `-r` flag is not used. In the `ls -l` output, the directory `/home/user01` remains but does not belong to any user in the system. The ownership shows the UID and GID of the deleted `user01`.

- Check the latest entries in the groups file.
  ```bash
  tail -10 /etc/group # Display the last 10 lines of the group file to see recent group entries
  ```

  > For each user, a new group was created using the same name. When the users are removed, those groups are removed as well.

Step 8: Create groups and assign users
- Create two new groups and assign the remaining users to those groups. Check the group membership before and after.
  ```bash
  groupadd group01 # Create a new group called group01
  groupadd group02 # Create a new group called group02
  groups user03 # Show current group membership for user03 (before change)
  groups user04 # Show current group membership for user04 (before change)
  usermod -aG group01 user03 # Append (-a) user03 to the supplementary group (-G) group01
  usermod -aG group02 user04 # Append (-a) user04 to the supplementary group (-G) group02
  groups user03 # Show updated group membership for user03 (after change)
  groups user04 # Show updated group membership for user04 (after change)
  ```

---

## Analytical Guidance

- **Step 1 vs Step 2:** Both `cat /etc/passwd` and `getent passwd` display user information, but `getent` also queries external sources like LDAP or SSSD. On a local-only system the output is identical.
- **Step 4:** Understand why `chpasswd` reads from standard input while `passwd` is interactive. In automation and scripting, `chpasswd` is the appropriate tool.
- **Step 5:** Modifying `sshd_config` and restarting the service is a common operational task. Be aware that enabling password authentication reduces security if weak passwords are used.
- **Step 6:** Account expiration is separate from password expiration. A user whose account is expired cannot log in even with a valid password.
- **Step 7:** The difference between `userdel` and `userdel -r` is critical. Orphaned home directories with numeric UID/GID ownership are a common finding in security audits.

---

## Validation Criteria

| Step | Expected result |
|---|---|
| Users listed | `getent passwd user01` returns user details |
| Users created | `ls /home` shows `user01`, `user02`, `user03`, `user04` directories |
| Passwords set | `ssh user01@localhost` prompts for password and connects |
| SSH password auth | `grep PasswordAuthentication /etc/ssh/sshd_config` shows `yes` |
| Expiration set | `chage -l user03` shows the configured expiration date |
| `user01` deleted | `getent passwd user01` returns nothing; `/home/user01` still exists |
| `user02` deleted with `-r` | `getent passwd user02` returns nothing; `/home/user02` is gone |
| Groups assigned | `groups user03` shows `user03 group01`; `groups user04` shows `user04 group02` |

---

## Documentation Expectations

As you complete this lab, take note of:

- The difference between `userdel` and `userdel -r` and the impact on the home directory.
- How `chpasswd` differs from `passwd` for batch vs. interactive password changes.
- The location and format of `/etc/passwd`, `/etc/group`, and `/etc/shadow`.
- The expiration date set for `user03` and the output of `chage -l`.

---

## What Not To Do

- Do not use hardcoded passwords in scripts that will be committed to a repository.
- Do not delete `user03` or `user04` — they will be used in subsequent labs.
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
  useradd -u 2000 -g group01 user05
  id user05
  ```
- Test logging in as `user03` via SSH from your local session to verify password authentication works end to end.
