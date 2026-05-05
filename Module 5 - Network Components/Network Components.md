# Module 5 - Network Components

## Module Overview

- **Training day:** Day 3
- **Planned duration:** 4 hours
- **Time breakdown:** ~1.5h reading and concept primer | ~2h lab execution including deployment (Lab 10) | ~30m knowledge check and completion criteria
- **Required VM(s):** RHEL 9, Ubuntu 24.04, SLES 15 SP7
- **Prior module required:** Module 4 - Linux on Azure

## Learning Objectives

- Inspect and validate Linux network configuration.
- Understand distro-specific network management tooling.
- Troubleshoot DNS, routes, ports, and service reachability.
- Apply host/network security controls during diagnostics.

## Key Topics

- Network baseline validation and interface/route inspection.
- Configuration files and effective settings verification.
- NetworkManager-related operational views and demonstrations.
- SELinux context in network troubleshooting.
- Port-level diagnostics and practical check sequences.

## Command Highlights

- `ip addr show`, `ip route show`, `ip link show`
- `nmcli connection show`, `nmcli connection show --active`
- `wicked show all`
- `cat /etc/netplan/50-cloud-init.yaml`, `netplan try`
- `ss -tuln`, `netstat -tuln`
- `firewall-cmd --list-all`, `firewall-cmd --add-service`
- `sestatus`, `semanage port -l`
- `tcpdump -i any port 80`

## Labs

### Lab 10 — Network Components
- **Duration:** ~90 minutes
- **VMs:** RHEL 9, Ubuntu 24.04, SLES 15 SP7 (FoundationsLab10.json)
- **Objectives:**
  - Inspect network configuration using `ip addr`, `ip route`, and `ip link`
  - Review cloud-init logs for network provisioning verification
  - Understand distribution-specific network managers: NetworkManager (RHEL), Wicked (SLES), Netplan/systemd-networkd (Ubuntu)
  - Disable and restore cloud-init network management
  - Configure custom DNS servers and search domains
  - Verify web server connectivity across service, firewall, SELinux, and NSG layers
  - Capture network traffic with `tcpdump`
- **Key Commands:** `ip`, `nmcli`, `wicked`, `netplan`, `ss`, `netstat`, `firewall-cmd`, `sestatus`, `tcpdump`
