## Objective

Practice Linux storage administration by creating and managing LVM storage, including physical volumes (PV), volume groups (VG), logical volumes (LV), ext4 filesystems, filesystem mounting, storage expansion and reduction, and LVM extent management

---

## Environment

- OS: Rocky Linux 9
- Storage: Additional /dev/sdb disk
- Partitioning tool: fdisk
- Volume management: LVM2
- File system: ext4
- Virtualization: VMware ESXi

## Tasks

### Disk and LVM Setup

- Prepare the /dev/sdb disk for LVM
- Create two partitions, including a 10 GiB partition for LVM
- Create physical volumes (PV) on both partitions
- Create a volume group (VG) named vg0
- Create a 1 GiB logical volume (LV) named lv-data

### Filesystem and Mounting

- Create an ext4 filesystem on lv-data
- Mount the filesystem under /data
- Configure persistent mounting using the filesystem UUID
- Verify the mounted filesystem

### Disk Usage and Filesystem Expansion

- Create test files on /data and monitor filesystem usage
- Analyze disk usage using df and du
- Extend lv-data and its ext4 filesystem
- Verify filesystem size and data integrity after expansion
- Check the extent size and available extents of vg0
- Extend lv-data using a specified number of extents
- Extend lv-data using all available free space in the volume group

### Volume Group Management

- Add the second physical volume to vg0
- Verify the new volume group size and available space
- Verify the physical volumes, volume group and logical volume configuration

### Logical Volume Reduction

- Safely reduce the ext4 filesystem and lv-data
- Verify the filesystem after reduction
- Confirm that existing data remains intact

### LVM Extent Management

- Create a volume group named vg1 with an extent size of 8 MiB
- Create a logical volume using a specified number of extents
- Verify the volume group and logical volume configuration

## Configuration files

- /etc/fstab

## Commands

### Disk and LVM Setup

#### Check block devices

lsblk


Displays all available block devices and their current partition and mount point information

#### Display the existing partition table

fdisk -l /dev/sdb


Displays the existing partition table and detailed information about the /dev/sdb disk

#### Create partitions for LVM

fdisk /dev/sdb


Creates and manages partitions on the /dev/sdb disk using the fdisk partitioning utility


Create the following partition layout:

/dev/sdb1 – 10 GiB – LVM
/dev/sdb2 – remaining disk space – LVM


Inside fdisk, use the following commands:

g
n
<Enter>
<Enter>
+10G
t
8e

n
<Enter>
<Enter>
<Enter>

p
w


The `g` command creates a new GPT partition table. The `n` command creates new partitions. The `t` command changes the partition type, `8e00` identifies the partition as Linux LVM, `p` displays the resulting partition table and `w` writes the changes to disk

#### Verify the partition layout

fdisk -l /dev/sdb


Displays the newly created partitions and their partition types


lsblk


Displays the available block devices and verifies the /dev/sdb1 and /dev/sdb2 partitions

#### Create physical volumes

pvcreate /dev/sdb1
pvcreate /dev/sdb2


Initializes the /dev/sdb1 and /dev/sdb2 partitions for use as LVM physical volumes

#### Verify physical volumes

pvs


Displays the physical volumes and their sizes, volume groups and available space


pvdisplay


Displays detailed information about the configured physical volumes

#### Create the volume group

vgcreate vg0 /dev/sdb1


Creates a volume group named vg0 using /dev/sdb1 as the physical volume

#### Verify the volume group

vgs


Displays the configured volume groups, their sizes and available free space


vgdisplay vg0


Displays detailed information about the vg0 volume group

#### Create the logical volume

lvcreate -L 1G -n lv_data vg0


Creates a 1 GiB logical volume named lv_data in the vg0 volume group

#### Verify the logical volume

lvs


Displays the configured logical volumes, their sizes and associated volume groups


lvdisplay /dev/vg0/lv_data


Displays detailed information about the lv_data logical volume

### Filesystem and Mounting

#### Create an ext4 filesystem

mkfs.ext4 /dev/vg0/lv_data


Creates an ext4 filesystem on the lv_data logical volume

#### Verify the filesystem

blkid /dev/mapper/vg0-lv_data


Displays the filesystem type and UUID of the lv_data logical volume

#### Create the mount point

mkdir -p /data


Creates the /data directory used as the mount point for the lv_data filesystem

#### Check the filesystem UUID

blkid /dev/vg0/lv_data


Displays the filesystem type and UUID of the lv_data logical volume

#### Mount the filesystem

mount /dev/vg0/lv_data /data


Mounts the lv_data filesystem under the /data mount point

#### Verify the mounted filesystem

df -hT


Displays mounted filesystems, their filesystem types, disk usage and mount points


lsblk -f


Displays block devices, filesystem types, UUIDs and mount points

#### Configure persistent mounting

vi /etc/fstab


Add the following entry:

UUID=<UUID_OF_LV_DATA>  /data  ext4  defaults  0 0


Configures the lv_data filesystem to be mounted automatically at system boot using its UUID

#### Test persistent mounting

umount /data


Unmounts the lv_data filesystem from the /data mount point


mount -a


Mounts all filesystems defined in /etc/fstab that are not currently mounted


df -hT | grep /data


Verifies that the lv_data filesystem is mounted on the expected /data mount point


lsblk -f


Displays the filesystem, UUID and mount point information of the available block devices

### Disk Usage and Filesystem Expansion

#### Create a test file


dd if=/dev/urandom of=/data/plik.1 bs=100M count=5


Creates a 500 MiB test file filled with random data in the /data filesystem

#### Check filesystem usage

df -hT /data


Displays the filesystem type, total size, used space, available space and mount point for /data

#### Check file and directory sizes


du -sh /data/*


Displays the sizes of files and directories located directly under /data

#### Create an additional test file

dd if=/dev/urandom of=/data/plik.2 bs=100M count=7


Attempts to create a 700 MiB test file in /data. If the filesystem runs out of free space, dd stops with an error and the partially written file remains on the filesystem

#### Verify filesystem usage

df -hT /data


Checks the available space on the /data filesystem and confirms that the filesystem is nearly full

#### Check file and directory sizes

du -sh /data/*


Displays the sizes of files and directories stored in /data and helps identify the space consumed by the test files

#### Extend the logical volume

lvextend -L +500M /dev/vg0/lv_data


Extends the lv_data logical volume by 500 MiB. The `+` sign specifies that the size should be increased by 500 MiB rather than set to an absolute size

#### Verify the logical volume size

lvs


Displays the current size of the lv_data logical volume and confirms that it was extended

#### Extend the ext4 filesystem

resize2fs /dev/vg0/lv_data


Expands the ext4 filesystem to use the additional space provided by the enlarged logical volume

#### Verify the filesystem size

df -hT /data


Displays the current size and available space of the /data filesystem after the expansion

#### Verify existing data

du -sh /data/*


Displays the sizes of the existing files in /data and confirms that the data is still present after extending the filesystem

#### Check the physical extent size

vgdisplay vg0


Displays detailed information about the vg0 volume group, including the physical extent size, total and free space available


#### Extend the logical volume by a number of extents

lvextend -l +50 /dev/vg0/lv_data


Extends the lv_data logical volume by 50 physical extents. The lowercase `-l` option specifies the number of extents, while the `+` sign increases the existing logical volume size

#### Extend the ext4 filesystem

resize2fs /dev/vg0/lv_data


Expands the ext4 filesystem to use the additional space provided by the logical volume

#### Verify the logical volume

lvs


Displays the current size of lv_data and confirms that the logical volume was extended

#### Verify the filesystem

df -hT /data


Displays the current filesystem size and available space on /data

#### Verify the volume group

vgs


Displays the remaining free space in vg0 after extending lv_data

#### Check available free space

vgs


Displays the total and free space available in the vg0 volume group before extending the logical volume


#### Extend the logical volume using all free space and extend the filesystem in one step

lvextend -l +100%FREE /dev/vg0/lv_data -r


Extends the lv_data logical volume using all available free space and automatically resizes the filesystem with the -r option

#### Verify the logical volume

lvs


Displays the final size of lv_data and confirms that the available free space in vg0 was allocated to the logical volume

#### Verify the filesystem

df -hT /data


Displays the current size and available space of the /data filesystem after the expansion

#### Check remaining free space

vgs


Verifies the remaining free space in vg0 after allocating all available extents to lv_data

### Volume Group Management

#### Extend the volume group

vgextend vg0 /dev/sdb2


Adds the /dev/sdb2 physical volume to the existing vg0 volume group and makes its space available for LVM allocation

#### Verify the volume group

vgs


Displays the updated size and available free space of vg0 after adding the second physical volume

#### Verify physical volumes

pvs


Displays all physical volumes and shows that /dev/sdb1 and /dev/sdb2 are now assigned to vg0

#### Display detailed volume group information

vgdisplay vg0


Displays detailed information about vg0, including the number of physical volumes, logical volumes, physical extents and available free extents

### Logical Volume Reduction

#### Check the filesystem

df -hT /data


Displays the current size and usage of the /data filesystem before reducing the logical volume

#### Unmount the filesystem

umount /data


Unmounts the /data filesystem before performing the filesystem reduction

#### Check and repair the filesystem

e2fsck -f /dev/vg0/lv_data


Forces a full filesystem check and verifies the integrity of the ext4 filesystem before reducing its size

#### Reduce the ext4 filesystem

resize2fs /dev/vg0/lv_data 3G


Reduces the ext4 filesystem to 3 GiB before reducing the underlying logical volume

#### Reduce the logical volume

lvreduce -L 4G /dev/vg0/lv_data


Reduces the lv_data logical volume to 4 GiB. The filesystem was reduced first to ensure that it fits safely within the new logical volume size

#### Resize the filesystem

resize2fs /dev/vg0/lv_data


Expands the filesystem to use the remaining space available within the 4 GiB logical volume

#### Mount the filesystem

mount /dev/mapper/vg0-lv_data /data


Mounts the /data filesystem using the existing /etc/fstab entry configured earlier in the lab

#### Verify the filesystem

df -hT /data


Displays the final size and usage of the reduced /data filesystem

#### Verify existing data

du -sh /data/*


Displays the sizes of files stored in /data and confirms that the existing data remains available after the reduction

### LVM Extent Management

#### Check physical volume allocation

pvs


Displays the physical volumes assigned to vg0 and verifies that /dev/sdb2 has no space allocated to logical volumes

#### Remove the physical volume from the volume group

vgreduce vg0 /dev/sdb2


Removes /dev/sdb2 from the vg0 volume group

#### Verify the volume group

vgs


Displays the updated vg0 configuration and confirms that /dev/sdb2 is no longer part of the volume group

#### Verify physical volumes

pvs

Displays the current physical volume configuration and confirms that /dev/sdb2 has been removed from vg0

#### Create a new volume group with an 8 MiB extent size

vgcreate -s 8M vg1 /dev/sdb2


Creates a new volume group named vg1 using /dev/sdb2 as its physical volume and sets the physical extent size to 8 MiB

#### Verify the new volume group

vgs


Displays the size, free space and extent information of the new vg1 volume group


#### Display detailed extent information

vgdisplay vg1


Displays detailed information about vg1, including the physical extent size and the total and available number of extents

#### Create a logical volume using 10 extents

lvcreate -l 10 vg1


Creates a logical volume in vg1 using exactly 10 physical extents
Because the physical extent size in vg1 is 8 MiB, the resulting logical volume size is 80 MiB (10 × 8 MiB)

#### Verify the logical volume

lvs


Displays the newly created logical volume and its size within vg1

#### Display detailed logical volume information

lvdisplay vg1


Displays detailed information about the logical volume created in vg1

## Skills Practiced

- Linux disk and partition management
- LVM physical volume (PV) management
- LVM volume group (VG) management
- LVM logical volume (LV) management
- Creating and managing ext4 filesystems
- Mounting filesystems using UUID
- Configuring persistent mounts with /etc/fstab
- Monitoring filesystem usage with df and du
- Extending logical volumes and filesystems
- Reducing ext4 filesystems and logical volumes safely
- Managing physical extents and logical extents
- Adding and removing physical volumes from volume groups
- Configuring custom physical extent sizes
- Verifying LVM configuration and available storage

## Result

Successfully created and managed LVM storage on Rocky Linux 9 using two partitions on /dev/sdb. Created physical volumes, configured the vg0 volume group and lv_data logical volume, created and mounted an ext4 filesystem, and configured persistent mounting using /etc/fstab

Practiced extending and reducing logical volumes and filesystems, managing LVM extents, adding and removing physical volumes from a volume group, and creating a second volume group with a custom 8 MiB physical extent size. Verified the resulting LVM and filesystem configuration throughout the lab

