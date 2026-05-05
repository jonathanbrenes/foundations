# Module 6 - Storage Components

## Module Overview

- **Training day:** Day 3
- **Planned duration:** 4 hours
- **Time breakdown:** ~1.5h reading and concept primer | ~2h lab execution including deployment (Lab 11) | ~30m knowledge check and completion criteria
- **Required VM(s):** SLES 15 SP7
- **Prior module required:** Module 5 - Network Components

## Learning Objectives

- Understand Linux storage layout from block devices to mounted filesystems.
- Create and manage partitions and filesystems.
- Configure persistent mounts and maintain startup reliability.
- Use LVM for flexible storage provisioning and expansion.

## Key Topics

- Linux storage hierarchy and component relationships.
- Partitioning model and partition operations.
- Filesystem choices and filesystem-level administration.
- Mounting block devices and persistent mount strategy.
- /etc/fstab significance and reliability concerns.
- LVM architecture: PV, VG, LV workflows.
- Storage utilization and capacity analysis (df and du).
- File system repair considerations.

## Command Highlights

- `lsblk`, `blkid`, `ls -lR /dev/disk/azure/`
- `fdisk`, `parted`
- `mkfs.xfs`, `mkfs.ext4`
- `mount`, `umount`, `df -h`, `du -sh`
- `pvcreate`, `vgcreate`, `lvcreate`
- `pvs`, `vgs`, `lvs`, `vgdisplay`, `lvdisplay`
- `lvextend`, `resize2fs`, `xfs_growfs`
- `vgextend`

## Labs

### Lab 11 — Storage Management
- **Duration:** ~90 minutes
- **VM:** SLES 15 SP7 (FoundationsLab11.json)
- **Objectives:**
  - Identify disks by LUN ID using `/dev/disk/azure/` symlinks
  - Create MBR and GPT partitions using `fdisk`
  - Format filesystems with XFS and ext4
  - Persist mount configurations in `/etc/fstab` using UUID
  - Create and manage LVM structures (PV, VG, LV)
  - Resize logical volumes and filesystems online
  - Extend volume groups by adding new data disks
- **Key Commands:** `lsblk`, `blkid`, `fdisk`, `mkfs.xfs`, `mkfs.ext4`, `pvcreate`, `vgcreate`, `lvcreate`, `lvextend`, `resize2fs`
