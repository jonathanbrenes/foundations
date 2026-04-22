# Lab 1 — WSL2, Azure CLI and Linux VM Deployment

## Title and Scenario

This lab establishes the working environment required for all subsequent modules in the Azure Linux Foundations curriculum. You will set up a local Linux shell using WSL2 and Windows Terminal, install Azure CLI, deploy an Ubuntu Linux VM in Azure, and transition from password-based authentication to SSH key-based authentication.

This is a guided, foundational (101) lab. All steps are provided with expected commands and validation points.

- **Estimated time:** 60 minutes.
- **VM count:** 1 (Ubuntu 24.04 LTS).

---

## Deployment

> **Warning:** The VMs deployed in these labs allow inbound traffic automatically from the Azure VPN (AzureCloud service tag). Make sure you are connected to the Azure VPN, or add an NSG rule to allow your public IP address before attempting to connect.

### Windows 11 VM (optional)

Use this deployment only if you cannot install WSL on your work computer.

[![Click to deploy](https://aka.ms/deploytoazurebutton)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fjonathanbrenes%2Ffoundations%2Fmain%2FModule%25201%2520-%2520OS%2520Fundamentals%2520I%2FLabs%2FFoundationsLab01Windows.json)

> If you experience size issues while deploying the VM, try changing the region.

### Ubuntu VM

[![Click to deploy](https://aka.ms/deploytoazurebutton)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fjonathanbrenes%2Ffoundations%2Fmain%2FModule%25201%2520-%2520OS%2520Fundamentals%2520I%2FLabs%2FFoundationsLab01.json)

> This template deploys an Ubuntu 24.04 LTS VM with password authentication. You will generate SSH keys as part of the lab exercises.

---

## Skills Required

- Basic familiarity with Windows desktop navigation.
- Ability to open a web browser and navigate to the Azure Portal.
- No prior Linux experience required.

---

## Recommended Prerequisites

- A Windows 11 machine (or the optional Windows 11 VM from the Deployment section).
- An active Azure subscription with permissions to create resource groups and virtual machines.
- Internet connectivity.

---

## Objectives

After completing this lab, you will be able to:

- Install WSL2 and Windows Terminal on a Windows machine.
- Install and authenticate Azure CLI from a Linux shell.
- Deploy an Ubuntu Linux VM in Azure using an ARM template.
- Connect to a remote Linux system using password authentication.
- Generate an SSH key pair and understand the role of public and private keys.
- Configure key-based SSH authentication on a remote VM.
- Use Azure Cloud Shell as an alternative access method.

---

## Environment Overview

| Component | Details |
|---|---|
| Local environment | Windows 11 with WSL2 (Ubuntu 24.04) |
| Cloud VM | Ubuntu 24.04 LTS (`lab01ubuntu`) |
| VM size | Standard_D2s_v3 |
| Authentication (initial) | Password |
| Authentication (final) | SSH key pair (RSA 4096-bit) |
| Network | Single NIC, public IP, NSG allowing SSH (22/TCP) from AzureCloud |

---

## Your Mission

Complete all four scenarios in order. Each scenario builds on the previous one.

### Scenario 1 — Install WSL and Windows Terminal

Set up the local Linux environment on your Windows machine. If you cannot install software on your work computer, deploy the optional Windows 11 VM first.

**Deployment:** See the [Deployment](#deployment) section (Windows 11 VM — optional).

#### Instructions

Step 1: Check for Updates
- Make sure there are no pending updates for Windows.

Step 2: Install WSL
- Open a Command Prompt and install WSL.
  ```bash
  wsl --install # Installs WSL and the default Linux distribution (Ubuntu)
  ```

Step 3: Finish Ubuntu installation
- Reboot the system. After logging back in, the Ubuntu installation will continue and ask for a user and password.

Step 4: Get Windows Terminal
- Look for Windows Terminal on the Microsoft Store, install or update to the latest version. [Link to Microsoft Store](https://aka.ms/terminal). Click on "View in Store".

Step 5: Open Ubuntu
- Open Windows Terminal and select a new tab for Ubuntu.

Step 6: Install Linux Software Repository for Microsoft Products
- Configure the Linux Software Repository for Microsoft Products. Using the default distribution that WSL installs, which is Ubuntu 24.04, execute the following commands in the Ubuntu tab in Windows Terminal:

- Download the repo config package:
  ```bash
  curl -sSL -O https://packages.microsoft.com/config/ubuntu/24.04/packages-microsoft-prod.deb # Download the .deb package silently (-s), follow redirects (-L), and save with the original filename (-O)
  ```

- Verify the file downloaded and install the repo config package:
  ```bash
  ls # List files in the current directory to confirm the .deb file was downloaded
  sudo dpkg -i packages-microsoft-prod.deb # Install the downloaded package using dpkg; sudo elevates to root privileges
  ```

- Delete the repo config package after installing:
  ```bash
  rm packages-microsoft-prod.deb # Remove the .deb file since it is no longer needed after installation
  ```

- Update package index files:
  ```bash
  sudo apt-get update # Refresh the list of available packages from all configured repositories
  ```

  You can find more information in the following link. It is not necessary for the rest of this lab, just useful reference for the future. [Documentation](https://learn.microsoft.com/windows-server/administration/linux-package-repository-for-microsoft-software).

Step 7: Install Azure CLI
- Install Azure CLI on Ubuntu. [Documentation](https://learn.microsoft.com/cli/azure/install-azure-cli-linux).
  ```bash
  curl -sL https://aka.ms/InstallAzureCLIDeb | sudo bash # Download the install script and pipe it to bash running as root
  ```

Step 8: Log in to Azure
- By default, `az login` will try to open a web browser if available. With the option `--use-device-code`, it will generate a code to be used at [https://microsoft.com/devicelogin](https://microsoft.com/devicelogin).

  > This method is not supported if you have a MCAPS Azure Subscription. If you are using one, proceed to use Cloud Shell from the Azure Portal and jump to Scenario 4.

  ```bash
  az login --use-device-code # Authenticate to Azure using a device code instead of opening a browser
  ```

Step 9: Using Azure CLI
- Get a list of available subscriptions and change the default. Replace `<SUBSCRIPTION_ID>` with the proper value.
  ```bash
  az account list --query "[].{Name:name, Id:id, IsDefault:isDefault}" -o table # List all subscriptions in table format, showing name, ID, and which is the default
  az account set --subscription <SUBSCRIPTION_ID> # Set the active subscription for all subsequent Azure CLI commands
  ```

### Scenario 2 — Deploy and connect to the Ubuntu VM

Deploy a single Ubuntu 24.04 LTS VM using the ARM template. The VM is configured with password authentication so you can practice the transition to SSH keys in Scenario 3.

**Deployment:** See the [Deployment](#deployment) section (Ubuntu VM).

#### Instructions

Step 1: Deploy the Ubuntu VM
- Click the **Ubuntu VM** deploy button in the Deployment section. Provide a password when prompted.

  > Write down the public IP or FQDN assigned to the VM. You will need it in the next steps.

Step 2: Connect to the VM using password
- From WSL or Cloud Shell, connect using SSH with the password you set during deployment.
  ```bash
  ssh azureuser@<VM_PUBLIC_IP> # Connect to the remote VM via SSH as the user azureuser
  ```

  > Accept the host key fingerprint when prompted. Type `yes` and press Enter.

  > If password authentication is denied, verify the NSG allows SSH (22/TCP) inbound from your source.

Step 3: Verify the connection
- Run a quick command to confirm you are connected to the Ubuntu VM.
  ```bash
  hostname # Print the name of the machine you are connected to
  cat /etc/os-release | head -5 # Display the first 5 lines of the OS release file to confirm the distribution
  ```

Step 4: Exit the VM
- Disconnect from the VM to return to your local session.
  ```bash
  exit # Close the SSH session and return to your local terminal
  ```

### Scenario 3 — SSH key-based authentication

Now that you can connect using a password, you will generate an SSH key pair and configure the VM to accept key-based authentication.

#### Instructions

Step 1: Generate an SSH key pair
- From your local WSL or Cloud Shell session, generate a new SSH key pair. Accept the default file location (`~/.ssh/id_rsa`) by pressing Enter.
  ```bash
  ssh-keygen -t rsa -b 4096 -N '' # Generate an RSA key pair with 4096-bit strength and no passphrase (-N '')
  ```

- List the resulting files:
  ```bash
  ls -l ~/.ssh/id* # List all files starting with 'id' inside the .ssh directory to see the generated keys
  ```

  > `id_rsa` is the private key — never share this file. `id_rsa.pub` is the public key that will be placed on the VM.

Step 2: Copy the public key to the VM
- Use `ssh-copy-id` to install the public key on the VM. You will be prompted for the password one last time.
  ```bash
  ssh-copy-id azureuser@<VM_PUBLIC_IP> # Copy your public key to the remote VM and append it to the authorized_keys file
  ```

  > This command appends the contents of `~/.ssh/id_rsa.pub` to `~/.ssh/authorized_keys` on the remote VM.

Step 3: Test key-based authentication
- Connect to the VM again. This time you should not be prompted for a password.
  ```bash
  ssh azureuser@<VM_PUBLIC_IP> # Connect again; this time no password should be required
  ```

- Verify the authorized key is in place:
  ```bash
  cat ~/.ssh/authorized_keys # Display the contents of the authorized_keys file to confirm your public key is listed
  ```

Step 4: Run a remote command without logging in
- From your local session, execute a command on the remote VM directly.
  ```bash
  ssh azureuser@<VM_PUBLIC_IP> date # Execute the 'date' command on the remote VM without opening an interactive session
  ```

  > This pattern is useful for quick remote checks without starting an interactive session.

### Scenario 4 — Using Azure Cloud Shell (alternative)

Use this scenario if you cannot install WSL on your work computer and are using the Azure Portal directly.

#### Instructions

Step 1: Open Azure Cloud Shell
- Navigate to [https://shell.azure.com](https://shell.azure.com) or open Cloud Shell from the Azure Portal. Select **Bash** for the shell type.

  > Initial setup must be done through the web portal to define the required storage account.

Step 2: Check for existing SSH keys
- Once the Bash shell is available, check if SSH keys already exist.
  ```bash
  ls -l ~/.ssh # List the contents of the .ssh directory to check for existing keys
  ```

  > If this is the first time using Cloud Shell, the `.ssh` directory may be empty or missing.

Step 3: Generate SSH keys in Cloud Shell
- If no keys exist, generate a new pair.
  ```bash
  ssh-keygen -t rsa -b 4096 -N '' # Generate an RSA 4096-bit key pair with no passphrase
  ls -l ~/.ssh/ # Verify the key files were created
  ```

Step 4: Update the VM with the Cloud Shell public key
- Use Azure CLI to update the VM user with the new public key. This uses the VMAccess extension.
  ```bash
  az vm user update --resource-group <RESOURCE_GROUP> --name lab01ubuntu --username azureuser --ssh-key-value ~/.ssh/id_rsa.pub # Update the VM user with the new public key using the VMAccess extension
  ```

Step 5: Connect from Cloud Shell
- Test the SSH connection from Cloud Shell.
  ```bash
  ssh azureuser@<VM_PUBLIC_IP> # Connect to the VM using the key that was just registered
  ```

---

## Analytical Guidance

- **Scenario 1** focuses on tooling. If WSL installation fails, check that virtualization is enabled in your BIOS/UEFI settings and that Hyper-V features are available.
- **Scenario 2** introduces the basic SSH connection flow. Pay attention to the host key fingerprint prompt — this is the first trust decision you make when connecting to a remote system.
- **Scenario 3** is the most important learning objective. Understand the relationship between the private key (stays local), the public key (goes to the server), and the `authorized_keys` file.
- **Scenario 4** demonstrates that Cloud Shell maintains its own filesystem. Keys generated in WSL are not available in Cloud Shell and vice versa. The `az vm user update` command bridges this gap using the VMAccess extension.

---

## Validation Criteria

| Step | Expected result |
|---|---|
| WSL installed | `wsl --list --verbose` shows Ubuntu running |
| Azure CLI installed | `az version` returns version information |
| Azure login | `az account show` displays the active subscription |
| VM deployed | `az vm show --resource-group <RESOURCE_GROUP> --name lab01ubuntu` returns VM details |
| Password SSH | `ssh azureuser@<VM_PUBLIC_IP>` connects with password prompt |
| SSH key generated | `ls ~/.ssh/id_rsa ~/.ssh/id_rsa.pub` shows both files |
| Key copied to VM | `ssh azureuser@<VM_PUBLIC_IP>` connects without password prompt |
| Remote execution | `ssh azureuser@<VM_PUBLIC_IP> date` returns the remote date |

---

## Documentation Expectations

As you complete this lab, take note of:

- The public IP or FQDN of your Ubuntu VM. You will need it in all subsequent labs.
- The location of your SSH keys (`~/.ssh/id_rsa` and `~/.ssh/id_rsa.pub`).
- Which Azure subscription you selected as default.
- Whether you used WSL or Cloud Shell — this affects key availability in future labs.

---

## What Not To Do

- Do not share your SSH private key (`id_rsa`) with anyone or copy it to the VM.
- Do not skip the password authentication step — experiencing the transition from password to key-based authentication is part of the learning objective.
- Do not delete the VM after this lab. It will be reused in Lab 2, Lab 3, and Lab 4.
- Do not use `sudo` for SSH key generation — keys should belong to your user, not root.

---

## Real-World Context

In production Azure environments, Linux VMs are almost always accessed via SSH key-based authentication. Password authentication is typically disabled for security. Understanding how to generate, distribute, and manage SSH keys is a core skill for any engineer working with Linux infrastructure in the cloud. The `ssh-copy-id` workflow you practiced here is the same process used to onboard access to production systems, jump hosts, and bastion environments.

Azure Cloud Shell provides a convenient fallback when local tooling is unavailable, but its ephemeral nature means keys and configurations must be managed deliberately. The VMAccess extension used in Scenario 4 is the same mechanism Azure support engineers use to recover SSH access to VMs when keys are lost.

---

## Optional Advanced Exploration

- Generate an **Ed25519** key pair instead of RSA and compare the key sizes:
  ```bash
  ssh-keygen -t ed25519 -N ''
  ```
- Disable password authentication on the VM by editing `/etc/ssh/sshd_config` and setting `PasswordAuthentication no`, then restarting `sshd`. Verify that only key-based access works.
- Explore the `~/.ssh/known_hosts` file and understand what happens when a VM is redeployed with a different host key.
- Use `az vm user update` from WSL to add a second public key to the VM and test multi-key access.
