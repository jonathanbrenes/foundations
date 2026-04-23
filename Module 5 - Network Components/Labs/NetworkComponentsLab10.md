# Lab 10 — Network Components

## Title and Scenario

This lab covers Linux networking fundamentals in Azure across three distributions: RHEL, Ubuntu, and SLES. You will work with the `ip` command, cloud-init networking, distribution-specific network managers (NetworkManager, Netplan/systemd-networkd, and Wicked), DNS configuration, and network troubleshooting tools including `ss`, `netstat`, SELinux, `firewalld`, and `tcpdump`.

- **Estimated time:** 90 minutes.
- **VM count:** 3 (RHEL 9, Ubuntu 24.04, SLES 15 SP7).

---

## Deployment

> **Warning:** The VMs deployed in these labs allow inbound traffic automatically from the Azure VPN (AzureCloud service tag). Make sure you are connected to the Azure VPN, or add an NSG rule to allow your public IP address before attempting to connect.

### Lab 10 Multi-VM Deployment (Scenarios 1-6)

Deploy a single ARM template that provisions all three VMs (`lab10rhel`, `lab10ubuntu`, and `lab10sles`) and the additional NIC resources used in the lab scenarios.

[![Click to deploy](https://aka.ms/deploytoazurebutton)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fjonathanbrenes%2Ffoundations%2Fmain%2FModule%25205%2520-%2520Network%2520Components%2FLabs%2FFoundationsLab10.json)

---

## Skills Required

- Ability to connect to a Linux VM via SSH.
- Basic familiarity with running commands as root using `sudo`.
- Understanding of `systemctl` for managing services.
- Basic understanding of IP addressing, subnets, and routing.

---

## Recommended Prerequisites

- Lab 1 completed — SSH connectivity and Azure CLI familiarity.
- Lab 9 completed — Azure Linux Agent and cloud-init concepts.

---

## Objectives

After completing this lab, you will be able to:

- Use basic network commands such as `ip`.
- Locate cloud-init log files and use them to verify network configuration and current status.
- Explain how network configuration is handled between cloud-init and each tool: NetworkManager for RHEL, Netplan for Ubuntu, and Wicked for SLES.
- Identify the steps to disable cloud-init networking and explain the impact.
- Implement custom DNS and domain settings.
- Analyze the status of ports using `ss`/`netstat`.
- Evaluate SELinux and `firewalld` configurations.
- Execute a packet capture using `tcpdump`.

---

## Environment Overview

| Component | Details |
|---|---|
| RHEL VM | RHEL 9 (`lab10rhel`) — Scenarios 1, 2, 3, 6 |
| Ubuntu VM | Ubuntu 24.04 LTS (`lab10ubuntu`) — Scenarios 3, 4 |
| SLES VM | SLES 15 SP7 (`lab10sles`) — Scenarios 3, 5 |
| VM size | Standard_D2s_v3 |
| Access | SSH as `azureuser` with password authentication |

---

## Your Mission

Complete each scenario on the corresponding VM. Deploy VMs as needed from the Deployment section.

### Scenario 1 — Basic Network Configuration

Using the `ip` command, identify the actual network configuration on the server and review the output.

**Deployment:** See the [Deployment](#deployment) section (RHEL 9 VM).

#### Instructions

Step 1: Connect and switch to root
- Connect to the `lab10rhel` VM and switch to root.
  ```bash
  ssh azureuser@<RHEL_VM_IP> # Connect to the RHEL VM using SSH
  sudo -i # Switch to root for full administrative privileges
  ```

Step 2: List IP addresses
- List IP addresses using the `ip` command and understand the output.
  ```bash
  ip addr show # Show all the network interfaces and their IP addresses
  ```

  > **Understanding the output:**

  | Interface | Details |
  |---|---|
  | **Loopback (lo)** | The loopback interface is used for internal communication within the host. It always has the IPv4 address `127.0.0.1/8` and the IPv6 address `::1/128`. The `<LOOPBACK,UP,LOWER_UP>` flags indicate it is active. MTU is 65536. |
  | **Ethernet (eth0)** | The primary Ethernet interface. `<BROADCAST,MULTICAST,UP,LOWER_UP>` indicates it supports broadcast and multicast and is active. MTU is 1500. The `inet` line shows the assigned IPv4 address and subnet mask. The `inet6 fe80::` address is the IPv6 link-local address. The `link/ether` value shows the MAC address. |

Step 3: View the routing table
- View the routing table using the `ip` command and understand the output.
  ```bash
  ip route show # Displays the routing table with all routes the system uses to determine where to send network traffic
  ```

  > **Understanding the output:**

  | Route | Explanation |
  |---|---|
  | `default via 10.1.0.1 dev eth0` | Default route — all traffic not matching a specific route goes through gateway `10.1.0.1` via `eth0`. `proto dhcp` means it was assigned via DHCP. |
  | `10.1.0.0/24 dev eth0` | Local subnet route — traffic to the `10.1.0.0/24` network goes directly through `eth0`. `proto kernel` means it was configured automatically by the kernel. `scope link` means it is valid only for directly connected networks. |
  | `168.63.129.16 via 10.1.0.1 dev eth0` | Route to the Azure wireserver IP used for DHCP, DNS, health probes, and other platform services. [Documentation](https://learn.microsoft.com/azure/virtual-network/what-is-ip-address-168-63-129-16) |
  | `169.254.169.254 via 10.1.0.1 dev eth0` | Route to the Azure Instance Metadata Service (IMDS). VMs use this to retrieve metadata about their configuration and network settings. [Documentation](https://learn.microsoft.com/azure/virtual-machines/instance-metadata-service?tabs=linux) |

Step 4: Check interface status
- Check the status of network interfaces using the `ip` command.
  ```bash
  ip link show # Displays the status and details of all network interfaces
  ```

  > **Understanding the output:**

  | Interface | Key fields |
  |---|---|
  | **lo** | `state UNKNOWN` — loopback is always active but reports as UNKNOWN. `qdisc noqueue` — no queuing discipline needed. MAC address is all zeros. |
  | **eth0** | `state UP` — interface is active. `qdisc mq` — multi-queue discipline. The `link/ether` value shows the MAC address assigned by Azure. |

### Scenario 2 — Cloud-init Logs and Network Configuration

Using the cloud-init logs and configuration files, review how the network was configured during provisioning.

**Deployment:** Uses the same RHEL 9 VM from Scenario 1.

#### Instructions

Step 1: Connect to the VM
- Continue using the `lab10rhel` VM. If disconnected, reconnect and switch to root.
  ```bash
  sudo -i # Switch to root for full administrative privileges
  ```

Step 2: Check the cloud-init.log file
- Check the cloud-init log file filtered for network-related messages. This is a detailed log with debugging output.
  ```bash
  grep -E -i 'network|eth|dhcp' /var/log/cloud-init.log # Search for lines containing network, eth, or dhcp (case-insensitive)
  ```

  > This shows cloud-init making the network configuration on the system. Any potential errors will be logged in this file.

Step 3: Check the cloud-init-output.log file
- Check the cloud-init output log, which includes the output from each stage of cloud-init.
  ```bash
  cat /var/log/cloud-init-output.log # Display the cloud-init output log containing the output of each initialization stage
  ```

  > This is a smaller file than `cloud-init.log` and shows the high-level output of each stage.

### Scenario 3 — Network Managers Across Linux Distributions

Each major Linux distribution uses a different network management tool. This scenario explores NetworkManager on RHEL, Wicked on SLES, and Netplan/systemd-networkd on Ubuntu. All three are configured initially by cloud-init during provisioning.

**Deployment:** See the [Deployment](#deployment) section (all three VMs).

#### Instructions — RHEL (NetworkManager)

Step 1: Connect to the RHEL VM
- Connect to `lab10rhel` and switch to root.
  ```bash
  sudo -i # Switch to root for full administrative privileges
  ```

Step 2: Verify the NetworkManager status
- Check the status of the NetworkManager systemd unit.
  ```bash
  systemctl -l --no-pager status NetworkManager # Display the full status of NetworkManager without truncation or paging
  ```

  > Check that the unit is `enabled`, `active`, and `running`.

Step 3: List NetworkManager connections
- List all active network connections managed by NetworkManager.
  ```bash
  nmcli connection show --active # List all currently active network connections managed by NetworkManager
  ```

Step 4: Review connection details for System eth0
- Review the connection details for `System eth0`. A filter can be used to extract the most important values during troubleshooting.
  ```bash
  nmcli connection show 'System eth0' | grep -E 'GENERAL.STATE|IP4.ADDRESS|IP4.GATEWAY|IP4.ROUTE|IP4.DNS|IP4.DOMAIN|GENERAL.DEVICES|GENERAL.UUID' # Filter key connection properties
  ```

  | Option | Explanation |
  |---|---|
  | `GENERAL.UUID` | Unique identifier for the connection profile |
  | `GENERAL.DEVICES` | Network interface device associated with the connection |
  | `GENERAL.STATE` | Current state of the connection (e.g., activated, deactivated) |
  | `IP4.ADDRESS` | IPv4 address assigned to the interface |
  | `IP4.GATEWAY` | Default gateway for the IPv4 network |
  | `IP4.ROUTE` | Routing information for the IPv4 network |
  | `IP4.DNS` | DNS servers configured for the connection |
  | `IP4.DOMAIN` | Domain name associated with the connection |

Step 5: Check the status of cloud-init
- Check the status of the cloud-init systemd unit.
  ```bash
  systemctl -l --no-pager status cloud-init # Display the full status of cloud-init without truncation or paging
  ```

  > The unit should be `enabled` and `active`. It will appear as `exited` and not `running` because cloud-init is designed to run only during the initial boot process.

Step 6: Review the main cloud-init configuration file
- Filter the cloud-init configuration file for network-related settings.
  ```bash
  grep -i 'network' /etc/cloud/cloud.cfg # Search for lines containing the word network (case-insensitive)
  ```

  > **Network Configuration Renderers in cloud-init:**

  | Renderer | Description |
  |---|---|
  | `sysconfig` | Uses traditional network configuration files in `/etc/sysconfig/network-scripts/`, commonly used in Red Hat-based distributions |
  | `eni` | Uses the `/etc/network/interfaces` file, typically used in Debian-based distributions |
  | `netplan` | Uses Netplan configuration files in `/etc/netplan/`, a newer method for configuring networking on Ubuntu |
  | `network-manager` | Uses NetworkManager, a dynamic network control and configuration system |
  | `networkd` | Uses systemd-networkd, a system service that manages network configurations for systemd-based systems |

  > These renderers allow cloud-init to be flexible and compatible with various Linux distributions and their respective network configuration methods.

Step 7: Check the traditional network configuration file
- The file `/etc/sysconfig/network-scripts/ifcfg-eth0` is used to configure network settings for the `eth0` interface on Red Hat-based distributions. This file was automatically created by cloud-init. **Do not edit it manually.**
  ```bash
  cat /etc/sysconfig/network-scripts/ifcfg-eth0 # Display the network configuration file for eth0 created by cloud-init
  ```

#### Instructions — SLES (Wicked)

Step 8: Connect to the SLES VM
- Connect to `lab10sles` and switch to root.
  ```bash
  ssh azureuser@<SLES_VM_IP> # Connect to the SLES VM using SSH
  sudo -i # Switch to root for full administrative privileges
  ```

Step 9: Check the status of Wicked
- Check the status of the Wicked systemd unit.
  ```bash
  systemctl -l --no-pager status wicked # Display the full status of the Wicked service without truncation or paging
  ```

  > The unit should be `enabled` and `active`. It will appear as `exited` because Wicked configures the network interfaces (lo, eth0) and then exits successfully.

Step 10: Inspect Wicked configuration
- List network interfaces managed by Wicked.
  ```bash
  wicked show all # Display the status and configuration details of all network interfaces managed by Wicked
  ```

Step 11: Check the status of cloud-init
- Check the status of the cloud-init systemd unit on SLES.
  ```bash
  systemctl -l --no-pager status cloud-init # Display the full status of cloud-init without truncation or paging
  ```

  > The unit should be `enabled` and `active` with status `exited`.

Step 12: Review the main cloud-init configuration file
- Filter the cloud-init configuration file for network-related settings.
  ```bash
  grep -ri 'network' /etc/cloud/cloud.cfg # Search for lines containing the word network (case-insensitive, recursive)
  ```

  > This may not return any output on SLES. Network management on SLES is hardcoded in cloud-init. Checking the cloud-init logs confirms network configuration.

Step 13: Check the traditional network configuration file
- The file `/etc/sysconfig/network/ifcfg-eth0` is used to configure network settings for `eth0` on SLES. This file was automatically created by cloud-init. **Do not edit it manually.**
  ```bash
  cat /etc/sysconfig/network/ifcfg-eth0 # Display the network configuration file for eth0 created by cloud-init
  ```

#### Instructions — Ubuntu (Netplan / systemd-networkd)

Step 14: Connect to the Ubuntu VM
- Connect to `lab10ubuntu` and switch to root.
  ```bash
  ssh azureuser@<UBUNTU_VM_IP> # Connect to the Ubuntu VM using SSH
  sudo -i # Switch to root for full administrative privileges
  ```

Step 15: Check the status of systemd-networkd
- Check the status of systemd-networkd.
  ```bash
  systemctl -l --no-pager status systemd-networkd.service # Display the full status of systemd-networkd without truncation or paging
  ```

  > The unit should be `enabled`, `active`, and `running`.

Step 16: Check the status of cloud-init
- Check the status of the cloud-init systemd unit on Ubuntu.
  ```bash
  systemctl -l --no-pager status cloud-init # Display the full status of cloud-init without truncation or paging
  ```

  > The unit should be `enabled` and `active` with status `exited`.

Step 17: Review the main cloud-init configuration file
- Filter the cloud-init configuration file for network-related settings.
  ```bash
  grep -i 'network' /etc/cloud/cloud.cfg # Search for lines containing the word network (case-insensitive)
  ```

  > **Network Configuration Renderers in cloud-init (Ubuntu):**

  | Renderer | Description |
  |---|---|
  | `netplan` | Uses Netplan configuration files in `/etc/netplan/` |
  | `eni` | Uses `/etc/network/interfaces` |
  | `network-manager` | Uses NetworkManager |
  | `networkd` | Uses systemd-networkd |

Step 18: Verify Netplan configuration
- List Netplan configuration files.
  ```bash
  ls /etc/netplan/ # List all Netplan configuration files
  ```

Step 19: Review the contents of the Netplan configuration file
- Review the Netplan file created by cloud-init.
  ```bash
  cat /etc/netplan/50-cloud-init.yaml # Display the Netplan configuration file created by cloud-init
  ```

### Scenario 4 — Disabling Networking from Cloud-init

This scenario demonstrates the impact of disabling cloud-init networking. You will disable cloud-init network management, replace the NIC on the VM, observe the resulting network failure, and restore connectivity.

**Deployment:** Uses the Ubuntu 24.04 VM from Scenario 3.

#### Instructions

Step 1: Connect to the Ubuntu VM
- Continue using the `lab10ubuntu` VM. If disconnected, reconnect and switch to root.
  ```bash
  sudo -i # Switch to root for full administrative privileges
  ```

Step 2: Disable network management in cloud-init
- Create an additional configuration file for cloud-init to disable network management.
  ```bash
  echo "network: {config: disabled}" | tee /etc/cloud/cloud.cfg.d/99-disable-network-config.cfg # Create a cloud-init drop-in that disables network configuration
  ```

Step 3: Verify the change
- Confirm the configuration file was created correctly.
  ```bash
  cat /etc/cloud/cloud.cfg.d/99-disable-network-config.cfg # Verify the content of the drop-in file
  ```

Step 4: Stop the VM
- In the Azure Portal, navigate to the VM overview page and click **Stop** to deallocate the VM.

Step 5: Attach the pre-deployed secondary NIC
- In the Azure Portal:
  1. Navigate to **Virtual machines** > `lab10ubuntu` > **Networking** > **Network settings**.
  2. Click **Attach network interface**.
  3. From the dropdown, select the pre-deployed `lab10ubuntu-nic2` and click **OK**.

Step 6: Detach the old NIC
- Click **Detach network interface** to remove the original NIC (`lab10ubuntu-nic1`) from the VM.

Step 7: Start the VM
- Return to the VM overview page and click **Start** to power on the VM with the new NIC attached.

Step 8: Connect to the VM using the Serial Console
- The VM will not have SSH connectivity because the network is misconfigured. Use the Azure Serial Console:
  1. In the Azure Portal, navigate to **Virtual machines** > `lab10ubuntu` > **Help** > **Serial console**.
  2. Log in with the `azureuser` credentials defined during deployment.

Step 9: Check current NIC hardware address
- Compare the MAC address on the system with the one in the Netplan configuration file.
  ```bash
  sudo -i # Switch to root for full administrative privileges
  ip addr show # Show the current MAC address assigned to the interface
  cat /etc/netplan/50-cloud-init.yaml # Show the MAC address in the Netplan configuration
  ```

  > Compare both MAC addresses. The MAC address assigned to the new NIC is different from the one configured in the Netplan file created by cloud-init during initial provisioning. As a result, the network has not been configured correctly — no IP address or routes were assigned.

Step 10: Restore cloud-init network management
- Remove the configuration file created in Step 2 and reboot to allow cloud-init to reconfigure the network.
  ```bash
  cat /etc/cloud/cloud.cfg.d/99-disable-network-config.cfg # Verify the file before removing
  rm /etc/cloud/cloud.cfg.d/99-disable-network-config.cfg # Remove the drop-in file to restore cloud-init networking
  reboot # Reboot the system to trigger cloud-init network reconfiguration
  ```

Step 11: Verify connectivity is restored
- After the reboot, connect to the VM via SSH and verify the network configuration.
  ```bash
  sudo -i # Switch to root for full administrative privileges
  ip addr show # Show the current IP address and MAC assigned to the interface
  cat /etc/netplan/50-cloud-init.yaml # Show the updated Netplan configuration with the new MAC address
  ```

  > Compare both MAC addresses. This time they should match, and an IP address should be assigned to the interface. Cloud-init detected the new NIC and updated the Netplan configuration automatically.

### Scenario 5 — Setting Custom DNS and Domain

Configure custom DNS servers and search domains on the SLES VM through the Azure Portal and the OS.

**Deployment:** Uses the SLES 15 SP7 VM from Scenario 3.

#### Instructions

Step 1: Connect to the SLES VM
- Connect to `lab10sles` and switch to root.
  ```bash
  sudo -i # Switch to root for full administrative privileges
  ```

Step 2: Change the VNet to custom DNS servers in the Azure Portal
- In the Azure Portal:
  1. Navigate to **Virtual machines** > `lab10sles` > **Network settings**.
  2. Click on the Virtual Network name to open the VNet resource.

Step 3: Add DNS IP addresses to the VNet
- In the VNet resource:
  1. Navigate to **DNS servers**.
  2. Select **Custom** and add the following dummy DNS values: `1.2.3.4` and `2.3.4.5`.
  3. Click **Save**.

  > These are not real DNS servers. After saving, restart the VM from the Azure Portal to pick up the new DNS configuration.

Step 4: Check the DNS server changes
- Reconnect to the VM and verify the new DNS servers are configured.
  ```bash
  sudo -i # Switch to root for full administrative privileges
  cat /etc/resolv.conf # Check the DNS configuration; the new DNS servers should be listed
  ```

Step 5: Set custom search domains
- The default search domain is `reddog.microsoft.com`. Change it to `microsoft.com`.
  ```bash
  echo NETCONFIG_DNS_STATIC_SEARCHLIST="microsoft.com" >> /etc/sysconfig/network/config # Append a static DNS search list entry for microsoft.com
  systemctl restart wicked.service # Restart Wicked to apply the network configuration change
  cat /etc/resolv.conf # Verify the search domain was updated in resolv.conf
  ```

### Scenario 6 — Check Connection for a Web Server

Verify the pre-configured web server, configure the host firewall for HTTP traffic, and capture traffic using `tcpdump`. The RHEL template deploys `httpd` automatically via customData by installing, enabling, and starting the service and by creating a default Hello World page at `/var/www/html/index.html`. An NSG rule for TCP port 80 is also included in the template.

**Deployment:** Uses the RHEL 9 VM from Scenario 1.

#### Instructions

Step 1: Connect to the RHEL VM
- Continue using the `lab10rhel` VM. If disconnected, reconnect and switch to root.
  ```bash
  sudo -i # Switch to root for full administrative privileges
  ```

Step 2: Verify the Apache web server is pre-installed and running
- The RHEL VM template pre-installs and configures `httpd` via customData during deployment. Verify the service is running and listening on port 80.
  ```bash
  systemctl -l --no-pager status httpd # Display the detailed status of the httpd service without truncation or paging
  ss -tuln | grep :80 # List all listening TCP/UDP sockets and filter for port 80
  netstat -tuln | grep :80 # Alternative command to list listening sockets on port 80
  ```

  > The httpd service should already be `enabled`, `active`, and `running`. A default "Hello World" page was created at `/var/www/html/index.html` during provisioning.

Step 3: Check SELinux
- SELinux can block traffic to port 80. Verify the SELinux status and the rules for HTTP ports.
  ```bash
  sestatus # Display the current status of SELinux on the system
  semanage port -l | grep http_port_t # List all SELinux port contexts and filter for HTTP-related port types
  ```

Step 4: Add the HTTP service to firewalld
- On RHEL, `firewalld` manages the host-level firewall. Check its current state, then add the HTTP service and reload the rules.
  ```bash
  firewall-cmd --state # Display the current state of the firewalld service
  firewall-cmd --list-all # Display all current firewalld settings and rules
  firewall-cmd --add-service=http --permanent # Add the HTTP service to the permanent firewalld configuration
  firewall-cmd --reload # Reload firewalld to apply the permanent rule changes
  firewall-cmd --list-all # Verify the http service now appears in the allowed services list
  ```

  > The ARM template also includes an NSG rule allowing TCP port 80 from AzureCloud, so no manual NSG changes are needed.

Step 5: Capture traffic for port 80 using tcpdump
- Capture traffic on port 80. Access the VM public IP from a web browser. Once the test is done, press `Ctrl+C` to stop the capture.
  ```bash
  tcpdump -i any port 80 and not host 168.63.129.16 # Capture traffic on port 80 from all interfaces, excluding Azure wireserver health probes
  ```

  > The parameter `-w` can be used to save the capture to a pcap file for later analysis:
  ```bash
  tcpdump -w mycapture.pcap -i any port 80 and not host 168.63.129.16 # Save the capture to a pcap file
  ```

---

## Analytical Guidance

- **Scenario 1:** The `ip` command is the primary tool for inspecting network configuration on modern Linux systems. Understanding the output of `ip addr show` and `ip route show` is essential for diagnosing connectivity issues. Pay attention to the Azure-specific routes (`168.63.129.16` and `169.254.169.254`) — these are critical for platform services and metadata.
- **Scenario 2:** Cloud-init logs (`/var/log/cloud-init.log` and `/var/log/cloud-init-output.log`) are the first place to check when network provisioning fails. Errors during the network stage will appear here.
- **Scenario 3:** Understanding the distribution-specific network manager is critical for troubleshooting. RHEL uses NetworkManager (`nmcli`), SLES uses Wicked, and Ubuntu uses Netplan with systemd-networkd as the backend. All three are initially configured by cloud-init.
- **Scenario 4:** Disabling cloud-init networking and then replacing the NIC demonstrates why cloud-init network management matters. Without it, the Netplan configuration retains the old NIC's MAC address, and the new NIC gets no IP configuration. This is a common support scenario when customers manually manage networking.
- **Scenario 5:** DNS and search domain changes propagated through the Azure VNet are applied via DHCP on the next lease renewal or reboot. The SLES-specific `NETCONFIG_DNS_STATIC_SEARCHLIST` setting allows customizing the search domain at the OS level.
- **Scenario 6:** Troubleshooting web server connectivity requires checking multiple layers: the service itself (`httpd`), port binding (`ss`/`netstat`), the host firewall (`firewalld`), SELinux port contexts, and the Azure NSG. `tcpdump` is the final tool for confirming whether traffic is reaching or leaving the VM.

---

## Validation Criteria

| Step | Expected result |
|---|---|
| `ip addr show` | Shows `lo` and `eth0` with assigned IP addresses |
| `ip route show` | Shows default route, subnet route, and Azure-specific routes |
| `ip link show` | Shows both interfaces with `state UP` for `eth0` |
| `grep -E -i 'network\|eth\|dhcp' /var/log/cloud-init.log` | Returns network configuration entries from cloud-init |
| `systemctl status NetworkManager` (RHEL) | Service is `enabled`, `active`, `running` |
| `nmcli connection show --active` (RHEL) | Shows `System eth0` as an active connection |
| `systemctl status wicked` (SLES) | Service is `enabled`, `active`, `exited` |
| `wicked show all` (SLES) | Shows `lo` and `eth0` with configuration details |
| `systemctl status systemd-networkd` (Ubuntu) | Service is `enabled`, `active`, `running` |
| `cat /etc/netplan/50-cloud-init.yaml` (Ubuntu) | Shows the Netplan configuration with MAC and IP details |
| Scenario 4 — after NIC swap with cloud-init disabled | No IP assigned; MAC mismatch in Netplan config |
| Scenario 4 — after restoring cloud-init and reboot | IP assigned; MAC addresses match |
| `cat /etc/resolv.conf` after DNS change | Shows custom DNS servers `1.2.3.4` and `2.3.4.5` |
| `cat /etc/resolv.conf` after search domain change | Shows `search microsoft.com` |
| `ss -tuln \| grep :80` | Shows httpd listening on port 80 |
| `sestatus` | Shows SELinux status; `http_port_t` includes port 80 |
| `firewall-cmd --list-all` | Shows `http` in allowed services |
| `tcpdump` output | Captures HTTP traffic on port 80 when accessing the VM |

---

## Documentation Expectations

As you complete this lab, take note of:

- The difference between `ip addr show`, `ip link show`, and `ip route show`.
- The Azure-specific routes (`168.63.129.16` and `169.254.169.254`) and their purpose.
- The distribution-specific network managers: NetworkManager (RHEL), Wicked (SLES), Netplan/systemd-networkd (Ubuntu).
- How cloud-init configures networking and what happens when you disable it.
- The multiple layers that must be checked for web server connectivity: service, port binding, host firewall, SELinux, and Azure NSG.
- The difference between `ss` and `netstat` (both show similar information, but `ss` is the modern replacement).

---

## What Not To Do

- Do not manually edit files created by cloud-init (e.g., `ifcfg-eth0`, Netplan YAML) on running VMs — cloud-init may overwrite your changes on the next boot.
- Do not disable cloud-init networking on production VMs unless you have a clear plan for manual network management.
- Do not use the dummy DNS servers (`1.2.3.4`, `2.3.4.5`) in production — they are non-functional and will break name resolution.
- Do not leave `firewalld` rules or NSG rules open to the internet beyond what is needed for testing.
- Do not delete the VMs until all Module 5 labs are complete.

---

## Real-World Context

Network troubleshooting is one of the most common tasks in Azure Linux support. Engineers regularly need to verify that the IP address, routes, and DNS were provisioned correctly by cloud-init, identify whether NetworkManager, Wicked, or Netplan is managing the network, and diagnose connectivity failures caused by mismatched NIC configurations. The Scenario 4 workflow (disabling cloud-init networking and swapping NICs) directly mirrors a common support case where customers disable cloud-init, replace a NIC, and lose connectivity. Scenario 6 demonstrates the multi-layer troubleshooting approach (service → port → firewall → SELinux → NSG → packet capture) that is essential for diagnosing end-to-end connectivity issues in Azure.

---

## Optional Advanced Exploration

- Use `nmcli` to add a static route on the RHEL VM:
  ```bash
  nmcli connection modify 'System eth0' +ipv4.routes "192.168.100.0/24 10.1.0.1" # Add a static route via the default gateway
  nmcli connection up 'System eth0' # Reactivate the connection to apply the change
  ip route show # Verify the new route appears in the routing table
  ```
- Use `netplan try` on the Ubuntu VM to test a network configuration change with automatic rollback:
  ```bash
  netplan try --timeout 30 # Apply the current Netplan config temporarily; reverts after 30 seconds if not confirmed
  ```
- Use `tcpdump` to capture DNS queries and identify resolution issues:
  ```bash
  tcpdump -i any port 53 # Capture all DNS traffic on port 53
  ```
- Check the Azure Instance Metadata Service from inside the VM:
  ```bash
  curl -s -H "Metadata:true" "http://169.254.169.254/metadata/instance/network?api-version=2021-02-01" | python3 -m json.tool # Query IMDS for network metadata and format the JSON output
  ```
