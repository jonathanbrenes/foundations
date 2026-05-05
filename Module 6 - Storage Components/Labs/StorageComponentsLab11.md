# Lab 11 — Storage Management

## Title and Scenario

This lab covers Linux storage fundamentals in Azure, including disk management, partitioning, filesystem operations, and the Logical Volume Manager (LVM). You will identify disks by their LUN ID, create and manage partitions, format filesystems, and implement persistent storage configurations across reboots.

All scenarios use a single SLES VM deployed from the template below.

- **Estimated time:** 90 minutes.
- **VM count:** 1 (SLES 15 SP7).

---

## Deployment

> **Warning:** The VMs deployed in these labs allow inbound traffic automatically from the Azure VPN (AzureCloud service tag). Make sure you are connected to the Azure VPN, or add an NSG rule to allow your public IP address before attempting to connect.

[![Click to deploy](https://aka.ms/deploytoazurebutton)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fjonathanbrenes%2Ffoundations%2Fmain%2FModule%25206%2520-%2520Storage%2520Components%2FLabs%2FFoundationsLab11.json)

---

## Skills Required

- Ability to connect to a Linux VM via SSH.
- Basic familiarity with running commands as root using `sudo`.
- Understanding of block devices and filesystem basics.
- Familiarity with disk partitioning concepts (MBR and GPT).

---

## Recommended Prerequisites

- Lab 1 completed — SSH connectivity and Azure CLI familiarity.
- Lab 2 completed — foundational Linux command-line skills.

---

## Objectives

After completing this lab, you will be able to:

- Identify a disk using its LUN ID.
- Create partitions on a disk using MBR and GPT partition tables.
- Format a block device with different filesystem types (XFS, ext4).
- Mount a filesystem and make the mount permanent using `/etc/fstab`.
- Understand and use the Logical Volume Manager (LVM) on Linux.
- Resize logical volumes and filesystems.
- Extend storage by adding new disks to existing volume groups.

---

## Environment Overview

| Component | Details |
|---|---|
| SLES VM | SLES 15 SP7 (`lab11sles`) — All scenarios |
| VM size | Standard_D2s_v3 |
| Data disks | 3 data disks (LUN 0, 1, 2) |
| Access | SSH as `azureuser` with password authentication |

---

## Your Mission

Complete each scenario on the `lab11sles` VM. Deploy the VM from the Deployment section before starting.

---

### Scenario 1 — Creating a new filesystem using an MBR partition

Create the required structure on the first data disk (LUN 0) to host a new XFS filesystem. Use `lsblk`, `blkid`, and the directory listing of `/dev/disk/azure` to identify the correct block device and its relationship to the LUN ID. Make the filesystem permanent across reboots by understanding the basic format of `/etc/fstab`.

**Deployment:** See the [Deployment](#deployment) section (SLES 15 SP7 VM).

**Reference documentation:** [Add a disk to a Linux VM](https://learn.microsoft.com/azure/virtual-machines/linux/add-disk)

#### Instructions

Step 1: Connect to the VM
- Connect to `lab11sles` and switch to the root account.
  ``` bash
  sudo -i # Switch to the root account
  ```

Step 2: Identify the first data disk
- Use `lsblk`, `blkid`, and the content of `/dev/disk/azure` to identify the disk mapped to LUN ID 0.
  ``` bash
  lsblk -f # List all block devices including the filesystem information
  blkid # Print block device attributes
  ls -lR /dev/disk/azure # Recursively lists all symbolic links to block devices
  ```
  > **Note:** Each disk has a symbolic link in `/dev/disk/azure/scsi1/`. LUN ID 0 is represented by `/dev/disk/azure/scsi1/lun0`. Use these links instead of `/dev/sdX` device names, which can change between reboots.

Step 3: Create a partition
- Identify the device corresponding to `lun0`. It could be `/dev/sdb`, `/dev/sdc`, `/dev/sdd`, or another `/dev/sdX` device.

  > Always use symbolic links in `/dev/disk/azure` to identify disks by LUN ID, not `/dev/sdX` device names, which change between reboots.

  ``` bash
  fdisk /dev/sdX # Replace sdX with the correct device for lun0
  ```
  > **Note:** `fdisk` is interactive. Use `m` for help. Common actions are listed below.

  | Fdisk command | Action |  
  |-----------|:-----------:|  
  | n | Create new partition |
  | p | Choose partition type |
  | w | Write changes to disk | 
  | m | Display fdisk help |
  
  To create the required partition, use these steps:
    - Create new partition: `n`
    - Partition type: press Enter for default (`p` for primary), or type `p`
    - Partition number: press Enter for default (`1`), or type `1`
    - First sector: press Enter for default (first available)
    - Last sector: press Enter for default (last available, uses all free space)
    - Print partition table: `p` (to verify)
    - Write changes: `w`

Step 4: Mount the filesystem
- Create a mount point at `/datadrive`, format the partition with XFS, and make it permanent in `/etc/fstab`.
  ``` bash
  mkdir /datadrive # Creates the directory /datadrive
  mkfs.xfs /dev/disk/azure/scsi1/lun0-part1 # Create a new XFS filesystem on LUN 0 partition 1
  UUID_LUN0_PART1=$(blkid -s UUID -o value /dev/disk/azure/scsi1/lun0-part1) # Get filesystem UUID for persistent mounting
  echo "UUID=${UUID_LUN0_PART1} /datadrive xfs defaults,nofail 0 0" # This will print to screen the required line to mount this filesystem across reboots
  echo "UUID=${UUID_LUN0_PART1} /datadrive xfs defaults,nofail 0 0" >> /etc/fstab # After checking previous line is correct, this line can be added to the end for the file /etc/fstab
  mount -a # Mount all filesystems
  df -h /datadrive # List filesystem utilization
  ```

---

### Scenario 2 — Create a new filesystem using LVM over a GPT partition

Create the required LVM structure on the second data disk (LUN 1) using GPT to host a new ext4 filesystem. Use `lsblk`, `blkid`, and the directory listing of `/dev/disk/azure` to identify the correct block device. Make the ext4 filesystem permanent across reboots using `/etc/fstab`.

#### Instructions

Step 1: Connect to the VM
- Connect to `lab11sles` and switch to the root account.
  ``` bash
  sudo -i # Switch to the root account
  ```

Step 2: Identify the second data disk
- Use `lsblk`, `blkid`, and the content of `/dev/disk/azure` to identify the disk mapped to LUN ID 1.
  ``` bash
  lsblk -f # List all block devices including the filesystem information
  blkid # Print block device attributes
  ls -lR /dev/disk/azure # Recursively lists all symbolic links to block devices
  ```

  > **Note:** Device assignments (`/dev/sdX`) can change between reboots. Always use symbolic links in `/dev/disk/azure/scsi1/` to identify disks by LUN ID. LUN ID 1 is represented by `/dev/disk/azure/scsi1/lun1`.

Step 3: Create a GPT partition on the second disk
- Identify the device corresponding to `lun1` from your previous output. It could be `/dev/sdb`, `/dev/sdc`, `/dev/sdd`, or another `/dev/sdX` device.

  ``` bash
  fdisk /dev/sdX # Replace sdX with the correct device for lun1
  ```
  > **Note:** `fdisk` is interactive. Use `m` for help. Create a GPT partition table in this scenario, not MBR.

  | Fdisk command | Action |  
  |-----------|:-----------:| 
  | g | Create a GPT disklabel | 
  | n | Create new partition |
  | p | Choose partition type |
  | t | Change partition type |
  | w | Write changes to disk | 
  | m | Display fdisk help |
  
  Follow these steps to create a GPT partition:
    - Create GPT disklabel: `g`
    - Create new partition: `n`
    - Partition number: press Enter for default (1), or type `1`
    - First sector: press Enter for default (first available)
    - Last sector: press Enter for default (last available, uses all free space)
    - Print partition table: `p` (to verify)
    - Change partition type: `t`, then use ID `44` for Linux LVM
    - Write changes: `w`

Step 4: Create the LVM structure
- Create a physical volume, volume group, and logical volume on the first partition of LUN 1.
  ``` bash
  pvcreate /dev/disk/azure/scsi1/lun1-part1 # Initialize physical volume on the first partition of LUN 1
  vgcreate vgdata /dev/disk/azure/scsi1/lun1-part1 # Create a volume group on the first partition of LUN 1
  lvcreate -n lvdata -L 3G vgdata # Create a logical volume of 3GB on the volume group
  lvdisplay /dev/vgdata/lvdata # Display the information of the logical volume
  ```
  > **Note:** Use symbolic links from `/dev/disk/azure/scsi1/`. The logical volume is 3GB.

Step 5: Mount the ext4 filesystem
- Create a mount point, format the logical volume, and add the mount to `/etc/fstab`.
  ``` bash
  mkdir /datadrive2 # Creates the directory /datadrive2
  mkfs.ext4 /dev/vgdata/lvdata # Create a new ext4 filesystem over the block device logical volume lvdata
  echo "/dev/vgdata/lvdata /datadrive2 ext4 defaults,nofail 0 2" # This will print to screen the required line to mount this filesystem across reboots
  echo "/dev/vgdata/lvdata /datadrive2 ext4 defaults,nofail 0 2" >> /etc/fstab # After checking previous line is correct, this line can be added to the end for the file /etc/fstab
  mount -a # Mount all filesystems
  df -h /datadrive2 # List filesystem utilization
  ```
  > **Note:** The last value in the `fstab` entry is `2` for ext4 (filesystem check enabled for non-root filesystems). XFS uses `0` (no check). The value `defaults,nofail` handles the mount safely even if the device is unavailable.

---

### Scenario 3 — Resizing a filesystem inside a logical volume

Identify free space in the volume group, extend the logical volume, and resize the filesystem to use the new space.

#### Instructions

Step 1: Check available free space
- Display information about the volume group to find free space.
  ``` bash
  vgdisplay /dev/vgdata # This will display the information related to vgdata. Find the "Free PE" section
  ```
  > **Note:** Free space is expressed in PEs (Physical Extents). For example, 255 PEs × 4 MiB per PE = 1020 MiB available.

Step 2: Extend the logical volume and filesystem
- Extend the logical volume to use all available space, then resize the filesystem.
  ``` bash
  lvs # Display information about logical volumes
  lvextend -l +100%FREE /dev/vgdata/lvdata # Extend the logical volume lvdata to use all the available free space in the volume group vgdata.
  df -h /datadrive2 # Display filesystem utilization, notice this is still the old size
  resize2fs /dev/vgdata/lvdata # Resize the filesystem on the logical volume lvdata
  df -h /datadrive2 # Display filesystem utilization; this time it shows the correct size
  ```

  > **Note:** XFS filesystems require `xfs_growfs` instead of `resize2fs`. See [Increasing the size of an XFS file system](https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/9/html/managing_file_systems/increasing-the-size-of-an-xfs-file-system_managing-file-systems) for details.

---

### Scenario 4 — Resizing a filesystem using an additional disk with LVM

Add a new data disk to the existing volume group, extend the logical volume, and expand the filesystem.

#### Instructions

Step 1: Connect to the VM
- Connect to `lab11sles` and switch to the root account.
  ``` bash
  sudo -i # Switch to the root account
  ```

Step 2: Identify the third data disk
- Identify LUN ID 2 using `lsblk`, `blkid`, and the symbolic links in `/dev/disk/azure`.
  ``` bash
  lsblk -f # List all block devices including the filesystem information
  blkid # Print block device attributes
  ls -lR /dev/disk/azure # Recursively lists all symbolic links to block devices
  ```

Step 3: Create a GPT partition on the third disk
- Identify the device corresponding to `lun2`. Create a GPT partition and set it as Linux LVM.
  ``` bash
  fdisk /dev/sdX # Replace sdX with the correct device for lun2
  # g - Create GPT partition table
  # n - Create new partition
  # t - Change partition type to Linux LVM (44)
  # w - Write changes
  ```

Step 4: Extend the volume group
- Add the new partition to the existing volume group.
  ``` bash
  pvcreate /dev/disk/azure/scsi1/lun2-part1 # Initialize physical volume on LUN 2
  vgextend vgdata /dev/disk/azure/scsi1/lun2-part1 # Add the new disk to the volume group
  vgdisplay /dev/vgdata # Check the new free space
  ```

Step 5: Extend the logical volume and filesystem
- Extend the logical volume to use all available space from both disks, then resize the filesystem.
  ``` bash
  lvextend -l +100%FREE /dev/vgdata/lvdata # Extend logical volume to all free space
  resize2fs /dev/vgdata/lvdata # Resize ext4 filesystem
  df -h /datadrive2 # Verify the new size
  ```

---

## Analytical Guidance

- **Scenario 1:** Traditional MBR partitioning with a direct XFS filesystem is the simplest storage layout. UUID-based `/etc/fstab` entries are essential in Azure because `/dev/sdX` device names change between reboots. Always use `/dev/disk/azure/scsi1/lunN` symlinks to identify disks by their LUN ID.
- **Scenario 2:** LVM adds a layer of abstraction between physical disks and filesystems. The stack is: physical disk → partition → physical volume (PV) → volume group (VG) → logical volume (LV) → filesystem. GPT is preferred over MBR for disks larger than 2 TB and for modern systems. Setting the partition type to Linux LVM (type 44 in fdisk) is a metadata hint — `pvcreate` works regardless, but it helps identification.
- **Scenario 3:** LVM's key advantage is online resizing. The logical volume can be extended while the filesystem is mounted, and `resize2fs` (ext4) or `xfs_growfs` (XFS) can grow the filesystem without unmounting. Always extend the LV first, then resize the filesystem — never the reverse.
- **Scenario 4:** Adding a new disk to an existing volume group demonstrates LVM's flexibility for expanding storage without downtime. The new disk becomes part of the same pool, and the logical volume spans both physical disks transparently. This is common in Azure when customers attach additional data disks to grow storage.

---

## Validation Criteria

| Step | Expected Result |
|---|---|
| `ls -lR /dev/disk/azure/scsi1/` | Shows lun0, lun1, lun2 symlinks to `/dev/sdX` devices |
| `fdisk` MBR partition (Scenario 1) | Creates partition 1 on LUN 0 with Linux type |
| `mkfs.xfs /dev/disk/azure/scsi1/lun0-part1` | XFS filesystem created successfully |
| `df -h /datadrive` | Shows XFS mount with ~6 GB available |
| `fdisk` GPT partition (Scenario 2) | Creates GPT partition with Linux LVM type (44) |
| `pvcreate`, `vgcreate`, `lvcreate` | PV, VG (vgdata), LV (lvdata, 3 GB) created successfully |
| `mkfs.ext4 /dev/vgdata/lvdata` | ext4 filesystem created on logical volume |
| `df -h /datadrive2` | Shows ext4 mount with ~2.9 GB available |
| `vgdisplay` free space (Scenario 3) | Shows free PEs (~1 GB remaining in VG) |
| `lvextend -l +100%FREE` + `resize2fs` | LV grows to ~4 GB; filesystem reflects new size |
| `vgextend` with LUN 2 (Scenario 4) | VG gains additional ~2 GB of free PEs |
| `lvextend` + `resize2fs` (Scenario 4) | LV grows to ~6 GB; filesystem reflects new size |
| `/etc/fstab` entries | UUID entry for XFS; device path entry for LVM ext4 |

---

## Documentation Expectations

As you complete this lab, take note of:

- The difference between `/dev/sdX` device names (unstable) and `/dev/disk/azure/scsi1/lunN` symlinks (stable by LUN ID).
- The LVM stack hierarchy: disk → partition → PV → VG → LV → filesystem.
- The difference between MBR and GPT partition tables and when to use each.
- The difference between `resize2fs` (ext4) and `xfs_growfs` (XFS) for filesystem resizing.
- Why `defaults,nofail` is used in `/etc/fstab` entries for data disks (prevents boot failure if the disk is detached).
- The difference between UUID-based and device-path-based fstab entries.

---

## What Not To Do

- ❌ **Don't use `/dev/sdX` names in `/etc/fstab`** — device names change between reboots in Azure. Use UUID or `/dev/disk/azure/` symlinks.
- ❌ **Don't resize the filesystem before extending the logical volume** — the filesystem must fit within the LV; extending the filesystem first causes errors.
- ❌ **Don't run `fdisk` on a disk with mounted partitions without understanding the risks** — changes require a reboot or `partprobe` to take effect.
- ❌ **Don't format the OS disk or ephemeral disk** — only format data disks attached as additional LUNs.
- ❌ **Don't skip the `nofail` option in fstab** — without it, a missing or detached data disk prevents the VM from booting.
- ❌ **Don't confuse `resize2fs` (ext4) with `xfs_growfs` (XFS)** — using the wrong tool causes errors.

---

## Real-World Context

Storage management is one of the most common tasks for Azure Linux engineers. Customers frequently need to add data disks, resize filesystems, and expand LVM volumes without downtime. The LUN-based disk identification pattern (`/dev/disk/azure/scsi1/lunN`) is critical because Azure VMs can reorder `/dev/sdX` assignments after reboots, host migrations, or disk attach/detach operations. A misconfigured `/etc/fstab` entry using a raw device name is one of the most common causes of Azure Linux VMs failing to boot — the disk gets reassigned to a different letter, the mount fails, and the boot process hangs. UUID-based or LUN-based entries prevent this entirely. LVM is widely used in enterprise environments because it allows online volume expansion, which is especially valuable in Azure where additional data disks can be attached to a running VM.

---

## Optional Advanced Exploration

- Use `pvs`, `vgs`, and `lvs` to display concise summaries of the LVM stack:
  ```bash
  pvs # Show physical volume summary
  vgs # Show volume group summary
  lvs # Show logical volume summary
  ```
- Use `lsblk -o NAME,SIZE,FSTYPE,MOUNTPOINT,UUID` for a comprehensive disk overview:
  ```bash
  lsblk -o NAME,SIZE,FSTYPE,MOUNTPOINT,UUID # Display disks with filesystem type, mount point, and UUID
  ```
- Create an LVM snapshot for backup purposes:
  ```bash
  lvcreate -s -n lvdata_snap -L 500M /dev/vgdata/lvdata # Create a 500 MB snapshot of lvdata
  lvs # Verify the snapshot appears
  lvremove /dev/vgdata/lvdata_snap # Remove the snapshot when done
  ```
- Test what happens when you detach a data disk and reboot — verify that `nofail` prevents boot failure:
  ```bash
  cat /etc/fstab # Review entries before detaching
  # Detach a data disk from the Azure Portal, then reboot
  # The VM should boot successfully; the mount point will be empty
  ```
- Explore `/dev/disk/by-uuid/` and `/dev/disk/by-id/` as alternative stable device references:
  ```bash
  ls -l /dev/disk/by-uuid/ # List disks by UUID
  ls -l /dev/disk/by-id/ # List disks by hardware ID
  ```
