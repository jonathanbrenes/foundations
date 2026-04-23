## Lab 11 - Storage Management

### About this lab

- This lab takes approximately 90 minutes.
- This lab provides hands-on activities.
- After this lab, you will be able to:
  - Identify a disk using its LUN ID
  - Create partitions on a disk
  - Format a block device
  - Mount a filesystem and make this a permanent change
  - Understand and use the Logical Volume Manager on Linux

### Scenario 1: Creating a new filesystem using an MBR partition

#### Overview Lab 11 Scenario 1

- In this scenario, the required structure will be created on the first data disk (LUN 0) to host a new filesystem.
- Use ```lsblk```, ```blkid```, and the directory listing of ```/dev/disk/azure``` to determine the correct block device and its relationship to the LUN ID.
- Make a new XFS filesystem permanent across reboots and understand the basic format of the file ```/etc/fstab```.
- Public documentation [Add a disk to a Linux VM](https://learn.microsoft.com/azure/virtual-machines/linux/add-disk)

#### Deployment Lab 11 Scenario 1

Step 1: Deploy a SLES VM
- Using the link below, fill in the required options identified with a `*`. Use this new VM for this lab.

  [![Click to deploy](https://aka.ms/deploytoazurebutton)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fjonathanbrenes%2Ffoundations%2Fmain%2FModule%25206%2520-%2520Storage%2520Components%2FLabs%2FFoundationsLab11SLES.json)

#### Instructions Lab 11 Scenario 1

Step 1: Connecting to the VM lab11sles
- Connect to the VM created in the previous step and switch to the root account.
  ``` bash
  sudo -i 
  ```

Step 2: Identify the right disk
- This VM has been deployed with several data disks. Use ```lsblk```, ```blkid```, and the content of ```/dev/disk/azure``` to identify the disk mapped to LUN ID 0.
  ``` bash
  lsblk -f # List all block devices including the filesystem information
  blkid # Print block device attributes
  ls -lR /dev/disk/azure # Recursively lists all symbolic links to block devices
  ```
  > **Note:** There is a symbolic link for each disk on the system, so LUN ID 0 is represented by ```/dev/disk/azure/scsi1/lun0```. You can also easily identify the OS disk (root) and the resource disk.

Step 3: Create a partition
- Based on your system's output from the previous step, identify which device corresponds to ```lun0```. Depending on your configuration, it could be ```/dev/sdb```, ```/dev/sdc```, ```/dev/sdd```, or another ```/dev/sdX``` device.

  > Remember that ```/dev/sdX``` assignments are unpredictable and can change between reboots. Always use the symbolic links in ```/dev/disk/azure``` to identify disks by LUN ID, not by sdX device name.

  ``` bash
  fdisk /dev/sdX  # Replace sdX with the correct device for lun0
  ```
  > **Note:** The command fdisk is an interactive tool, so it expects interactive input. Here are some common actions. You can check the fdisk help using ```m```.

  | Fdisk command | Action |  
  |-----------|:-----------:|  
  | n | Create new partition |
  | p | Choose partition type |
  | w | Write changes to disk | 
  | m | Display fdisk help |
  
  To create the required partition, use these steps:
    - New partition ```n```
    - Primary partition type: press Enter to accept the default (```p```), or type ```p```
    - Partition number: press Enter to accept the default (```1```), or type ```1```
    - First sector: select a sector in the printed range. The default is the first available sector
    - Last sector: select a sector in the printed range. The default is the last available sector, which uses all free space
    - Print partition table, using ```p```
    - Write changes to disk using ```w```

Step 4: Mounting a filesystem
- Create a mount point at ```/datadrive```, create a new filesystem on the block device, and make the mount permanent in ```/etc/fstab```.
  ``` bash
  mkdir /datadrive # Creates the directory /datadrive
  mkfs.xfs /dev/disk/azure/scsi1/lun0-part1 # Create a new XFS filesystem on LUN 0 partition 1
  UUID_LUN0_PART1=$(blkid -s UUID -o value /dev/disk/azure/scsi1/lun0-part1) # Get filesystem UUID for persistent mounting
  echo "UUID=${UUID_LUN0_PART1} /datadrive xfs defaults,nofail 0 0" # This will print to screen the required line to mount this filesystem across reboots
  echo "UUID=${UUID_LUN0_PART1} /datadrive xfs defaults,nofail 0 0" >> /etc/fstab # After checking previous line is correct, this line can be added to the end for the file /etc/fstab
  mount -a # Mount all filesystems
  df -h /datadrive # List filesystem utilization
  ```

### Scenario 2: Create a new filesystem using LVM over a GPT partition

#### Overview Lab 11 Scenario 2

- In this scenario, the required LVM structure will be created on the second data disk using GPT to host a new ext4 filesystem.
- Use ```lsblk```, ```blkid```, and the directory listing of ```/dev/disk/azure``` to determine the correct block device and its relationship to the LUN ID.
- Make a new ext4 filesystem permanent across reboots, and understand the basic format of the file ```/etc/fstab```.

#### Instructions Lab 11 Scenario 2

Step 1: Connecting to the VM lab11sles
- Connect to the VM created in the previous scenario and switch to the root account.
  ``` bash
  sudo -i 
  ```

Step 2: Identify the right disk
- This VM has been deployed with several data disks. Use ```lsblk```, ```blkid```, and the content of ```/dev/disk/azure``` to identify the disk mapped to LUN ID 1.
  ``` bash
  lsblk -f # List all block devices including the filesystem information
  blkid # Print block device attributes
  ls -lR /dev/disk/azure # Recursively lists all symbolic links to block devices
  ```

  > **Note:** Device assignments can change between reboots. For example, after a reboot, LUN 0 might be ```/dev/sdb``` and LUN 1 might be ```/dev/sdc```, or they could map to different devices entirely. This highlights the importance of using symbolic links in ```/dev/disk/azure``` and never relying on ```/dev/sdX``` names in ```/etc/fstab```.

  > There is a symbolic link for each disk on the system, so LUN ID 1 is represented by ```/dev/disk/azure/scsi1/lun1```. You can also easily identify the OS disk (root) and the resource disk.

  > All data disks are listed under the directory ```/dev/disk/azure/scsi1```

Step 3: Create a partition
- Based on your system's output from the previous step, identify which device corresponds to ```lun1```. Depending on your configuration, it could be ```/dev/sdb```, ```/dev/sdc```, ```/dev/sdd```, or another ```/dev/sdX``` device.

  ``` bash
  fdisk /dev/sdX  # Replace sdX with the correct device for lun1
  ```
  > **Note:** The command fdisk is an interactive tool, so it expects interactive input. Here are some common actions. You can check the fdisk help using ```m```. In this scenario, use a GPT partition table instead of MBR.

  | Fdisk command | Action |  
  |-----------|:-----------:| 
  | g | Create a GPT disklabel | 
  | n | Create new partition |
  | p | Choose partition type |
  | t | Change partition type |
  | w | Write changes to disk | 
  | m | Display fdisk help |
  
  To create the required partition, use these steps:
    - Create a GPT disklabel on the disk ```g```
    - New partition ```n```
    - Partition number: press Enter to accept the default ("1") or type ```1```. With GPT, up to 128 partitions are supported (compared to 4 primary partitions with MBR)
    - First sector: select a sector in the printed range. The default is the first available sector
    - Last sector: select a sector in the printed range. The default is the last available sector, which uses all free space
    - Print partition table, using ```p```
    - Change the type of partition ```t```, using ID 44 will set to Linux LVM
    - Write changes to disk using ```w```

Step 4: Create the LVM structure
- Using the first partition created on LUN 1, create the LVM structure.
  ``` bash
  pvcreate /dev/disk/azure/scsi1/lun1-part1 # Initialize physical volume on the first partition of LUN 1
  vgcreate vgdata /dev/disk/azure/scsi1/lun1-part1 # Create a volume group on the first partition of LUN 1
  lvcreate -n lvdata -L 3G vgdata # Create a logical volume of 3GB on the volume group
  lvdisplay /dev/vgdata/lvdata # Display the information of the logical volume
  ```
  > **Note:** Use the symbolic link available on ```/dev/disk/azure/scsi1```. The logical volume created is only 3GB

Step 5: Mounting a filesystem
- Creating a mountpoint ```/datadrive2```, create a new filesystem on the logical volume and make the mount point permanent on ```/etc/fstab```
  ``` bash
  mkdir /datadrive2 # Creates the directory /datadrive2
  mkfs.ext4 /dev/vgdata/lvdata # Create a new ext4 filesystem over the block device logical volume lvdata
  echo "/dev/vgdata/lvdata /datadrive2 ext4 defaults,nofail 0 2" # This will print to screen the required line to mount this filesystem across reboots
  echo "/dev/vgdata/lvdata /datadrive2 ext4 defaults,nofail 0 2" >> /etc/fstab # After checking previous line is correct, this line can be added to the end for the file /etc/fstab
  mount -a # Mount all filesystems
  df -h /datadrive2 # List filesystem utilization
  ```
  > **Note:** In the line added to ```/etc/fstab```, the last value changes from 0 to 2. A value of 0 means no filesystem check during boot, while a value of 2 means a filesystem check for non-root filesystems. XFS does not require a filesystem check during boot, but it is recommended for ext4 filesystems.

### Scenario 3: Resizing a filesystem inside a logical volume

#### Overview Lab 11 Scenario 3

- Identify free space in the volume group
- Resize the logical volume using the free space available on the volume group
- Resize the filesystem to match the actual size of the logical volume

#### Instructions Lab 11 Scenario 3

Step 1: Identifying the available free space in a volume group
- Find how much free space is available on the volume group ```/dev/vgdata```
  ``` bash
  vgdisplay /dev/vgdata # This will display the information related to vgdata. Find the "Free PE" section
  ```
  > **Note:** The free space is expressed in PEs (Physical Extents). In this case, there are 255 PEs, and each PE is 4 MiB, for a total of 1020 MiB.

Step 2: Extend the logical volume
- Using the free Physical Extents identified in the previous step, add the total of free space to the logical volume lvdata
  ``` bash
  lvs # Display information about logical volumes
  lvextend -l +100%FREE /dev/vgdata/lvdata # Extend the logical volume lvdata to use all the available free space in the volume group vgdata.
  lvs # Display information about logical volumes
  df -h /datadrive2 # Display filesystem utilization, notice this is still the old size
  resize2fs /dev/vgdata/lvdata # Resize the filesystem on the logical volume lvdata
  df -h /datadrive2 # Display filesystem utilization; this time it shows the correct size
  ```

  > **Note:** There are differences in the tools used to resize an XFS filesystem. [Increasing the size of an XFS file system in RHEL 9](https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/9/html/managing_file_systems/increasing-the-size-of-an-xfs-file-system_managing-file-systems)

### Scenario 4: Resizing a filesystem using an additional disk with LVM

#### Overview Lab 11 Scenario 4

- Add an additional disk to the existing volume group
- Resize the logical volume to include the new disk
- Extend the filesystem to use all available space

#### Instructions Lab 11 Scenario 4

Step 1: Connecting to the VM lab11sles
- Connect to the VM and switch to the root account.
  ``` bash
  sudo -i 
  ```

Step 2: Identify the additional disk
- Identify LUN ID 2 using ```lsblk```, ```blkid```, and the symbolic links in ```/dev/disk/azure```.
  ``` bash
  lsblk -f # List all block devices including the filesystem information
  blkid # Print block device attributes
  ls -lR /dev/disk/azure # Recursively lists all symbolic links to block devices
  ```

Step 3: Create a partition on the new disk
- Based on your system's output from Step 2, identify which device corresponds to ```lun2```. Create a GPT partition and set it as Linux LVM.
  ``` bash
  fdisk /dev/sdX  # Replace sdX with the correct device for lun2
  # g - Create GPT partition table
  # n - Create new partition
  # t - Change partition type to Linux LVM (44)
  # w - Write changes
  ```

Step 4: Extend the volume group
- Add the new partition to the existing volume group
  ``` bash
  pvcreate /dev/disk/azure/scsi1/lun2-part1 # Initialize physical volume on LUN 2
  vgextend vgdata /dev/disk/azure/scsi1/lun2-part1 # Add the new disk to the volume group
  vgdisplay /dev/vgdata # Check the new free space
  ```

Step 5: Extend the logical volume and filesystem
- Extend the logical volume to use all available space
  ``` bash
  lvextend -l +100%FREE /dev/vgdata/lvdata # Extend logical volume to all free space
  resize2fs /dev/vgdata/lvdata # Resize ext4 filesystem
  df -h /datadrive2 # Verify the new size
  ```
