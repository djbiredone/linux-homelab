## Objective

Practice Linux storage administration by creating and managing disk partitions using MBR (DOS) and GPT partition tables, creating ext4 and XFS file systems, and mounting them to specified mount points


---

## Environment

- OS: Rocky Linux 9
- Storage: Additional /dev/sdb disk
- Partitioning tool: fdisk
- File systems: ext4, XFS
- Firewall: firewalld
- Virtualization: VMware ESXi

## Tasks

MBR (DOS) Partition Table
- Check available block devices
- Display the existing partition table of /dev/sdb
- Create an MBR (DOS) partition table using fdisk
- Create partitions using the following layout:
  - /dev/sdb1 – 1 GiB – primary
  - /dev/sdb2 – 1 GiB – primary
  - /dev/sdb3 – 4 GiB – primary
  - /dev/sdb4 – extended
  - /dev/sdb5 – 2 GiB – logical
  - /dev/sdb6 – 5 GiB – logical
  - /dev/sdb7 – 1 GiB – logical
- Display and verify the created partition layout

GPT Partition Table
- Replace the MBR (DOS) partition table with a GPT partition table
- Create six partitions with the same sizes
- Display and verify the GPT partition layout

File System Management
- Create an ext4 file system on selected partitions
- Create an XFS file system on selected partitions
- Verify the file system types
- Display file system information using lsblk and blkid

Mounting File Systems
- Create dedicated mount points for the file systems
- Mount the ext4 and XFS file systems
- Verify the mounted file systems
- Check disk usage and file system types
- Unmount the file systems

Persistent Mounting
- Configure persistent mounts using /etc/fstab
- Use UUIDs to identify file systems
- Verify the /etc/fstab configuration
- Test automatic mounting without rebooting

## Configuration files

- /etc/fstab

## Commands

### Check block devices

lsblk


Shows all available block devices and their current partition and mount point information


### Display the existing partition table

fdisk -l /dev/sdb


Shows the existing partition table and detailed information about the /dev/sdb disk


### MBR (DOS) partition table

#### Create MBR (DOS) partition table and partitions

fdisk /dev/sdb


Creates and manages partitions on the /dev/sdb disk using the fdisk partitioning utility


Inside fdisk, create an MBR (DOS) partition table and the following partition layout:

/dev/sdb1 – 1 GiB – primary
/dev/sdb2 – 1 GiB – primary
/dev/sdb3 – 4 GiB – primary
/dev/sdb4 – extended partition
/dev/sdb5 – 2 GiB – logical
/dev/sdb6 – 5 GiB – logical
/dev/sdb7 – 1 GiB – logical

The fourth partition is created as an extended partition because an MBR partition table supports a maximum of four primary partitions. Logical partitions are created inside the extended partition

Use the following fdisk commands:

o
n
1
<Enter>
<Enter>
+1G

n
2
<Enter>
<Enter>
+1G

n
3
<Enter>
<Enter>
+4G

n
e
4
<Enter>
<Enter>

n
5
<Enter>
<Enter>
+2G

n
6
<Enter>
<Enter>
+5G

n
7
<Enter>
<Enter>
+1G

p
w

The o command creates a new MBR (DOS) partition table. The n command creates new partitions, while p creates primary partitions, e creates an extended partition. The p command displays the resulting partition table and w writes the changes to disk


#### Create an ext4 filesystem

mkfs.ext4 /dev/sdb1


Creates an ext4 filesystem on the /dev/sdb1 partition


#### Verify the filesystem

blkid /dev/sdb1


Displays the filesystem type and UUID of the /dev/sdb1 partition


#### Create an XFS filesystem

mkfs.xfs /dev/sdb2


Creates an XFS filesystem on the /dev/sdb2 partition


#### Verify the filesystem

blkid /dev/sdb2


Displays the filesystem type and UUID of the /dev/sdb2 partition


#### Create mount points

mkdir -p /mnt/ext4
mkdir -p /mnt/xfs


Creates mount points for the ext4 and XFS filesystems


#### Mount the file systems

mount /dev/sdb1 /mnt/ext4


Mounts the ext4 filesystem on the /mnt/ext4 mount point


mount /dev/sdb2 /mnt/xfs


Mounts the XFS filesystem on the /mnt/xfs mount point


#### Verify the mounted filesystems

df -hT


Displays mounted filesystems, their filesystem types, disk usage and mount points


lsblk -f


Displays block devices, filesystem types, UUIDs and mount points


#### Configure persistent mounts

##### Check /dev/sdb1 and /dev/sdb2 UUID

blkid /dev/sdb1 /dev/sdb2


Displays the UUID and filesystem type of the /dev/sdb1 and /dev/sdb2 partitions


##### Edit /etc/fstab configuration file

vi /etc/fstab


Add entries:

UUID=<UUID_OF_SDB1>  /mnt/ext4  ext4  defaults  0 0
UUID=<UUID_OF_SDB2>  /mnt/xfs   xfs   defaults  0 0


Configures the ext4 and XFS filesystems to be mounted automatically at system boot using their UUIDs


##### Test

umount /mnt/ext4
umount /mnt/xfs


Unmounts the ext4 and XFS filesystems from their respective mount points


mount -a


Mounts all filesystems defined in /etc/fstab that are not currently mounted


df -hT | grep /mnt


Verifies that the filesystems defined in /etc/fstab are mounted on the expected mount points


lsblk -f


Displays the filesystems, UUIDs and mount points of the available block devices


### GPT Partition Table

#### Unmount existing filesystems

umount /mnt/ext4
umount /mnt/xfs


Unmounts the ext4 and XFS filesystems before modifying the partition table


#### Remove persistent mount configuration

vi /etc/fstab


Remove entries:

UUID=<UUID_OF_SDB1>  /mnt/ext4  ext4  defaults  0 0
UUID=<UUID_OF_SDB2>  /mnt/xfs   xfs   defaults  0 0


Removes the existing persistent mount entries before recreating the disk partition layout


#### Create GPT partition table and partitions

fdisk /dev/sdb


Creates and manages partitions on the /dev/sdb disk using the fdisk partitioning utility


Inside fdisk, create a GPT partition table and the following partition layout:

/dev/sdb1 – 1 GiB
/dev/sdb2 – 1 GiB
/dev/sdb3 – 4 GiB
/dev/sdb4 – 2 GiB
/dev/sdb5 – 5 GiB
/dev/sdb6 – 1 GiB

Use the following fdisk commands:

g
n
1
<Enter>
<Enter>
+1G

n
2
<Enter>
<Enter>
+1G

n
3
<Enter>
<Enter>
+4G

n
4
<Enter>
<Enter>
+2G

n
5
<Enter>
<Enter>
+5G

n
6
<Enter>
<Enter>
+1G

p
w

The g command creates a new GPT partition table. The n command creates new partitions. The p command displays the resulting partition table and w writes the changes to disk


### Verify the partition layout

fdisk -l /dev/sdb


Displays the GPT partition table and detailed information about the /dev/sdb disk and its partitions


lsblk


Displays the available block devices and their current partition information


#### Create an ext4 filesystem

mkfs.ext4 /dev/sdb1


Creates an ext4 filesystem on the /dev/sdb1 partition


#### Verify the filesystem

blkid /dev/sdb1


Displays the filesystem type and UUID of the /dev/sdb1 partition


#### Create an XFS filesystem

mkfs.xfs /dev/sdb2


Creates an xfs filesystem on the /dev/sdb2 partition


#### Verify the filesystem

blkid /dev/sdb2


Displays the filesystem type and UUID of the /dev/sdb2 partition


#### Mount the file systems

mount /dev/sdb1 /mnt/ext4
mount /dev/sdb2 /mnt/xfs

Mounts the ext4 and XFS filesystems on their respective mount points


#### Verify the mounted filesystems

df -hT


Displays mounted filesystems, their filesystem types, disk usage and mount points


lsblk -f


Displays block devices, filesystem types, UUIDs and mount points


#### Configure persistent mounts

##### Check /dev/sdb1 and /dev/sdb2 UUID

blkid /dev/sdb1 /dev/sdb2


Displays the UUID and filesystem type of the /dev/sdb1 and /dev/sdb2 partitions


##### Edit /etc/fstab configuration file

vi /etc/fstab


Add entries:

UUID=<UUID_OF_SDB1>  /mnt/ext4  ext4  defaults  0 0
UUID=<UUID_OF_SDB2>  /mnt/xfs   xfs   defaults  0 0


Configures the ext4 and XFS filesystems to be mounted automatically at system boot using their UUIDs


##### Test

umount /mnt/ext4
umount /mnt/xfs


Unmounts the ext4 and XFS filesystems from their respective mount points


mount -a


Mounts all filesystems defined in /etc/fstab that are not currently mounted


df -hT | grep /mnt


Verifies that the filesystems defined in /etc/fstab are mounted on the expected mount points.


lsblk -f


Displays the filesystems, UUIDs and mount points of the available block devices


## Skills Practiced

* Disk partitioning using `fdisk`
* Creating and managing MBR (DOS) partition tables
* Creating and managing GPT partition tables
* Understanding primary, extended and logical partitions
* Creating ext4 and XFS filesystems
* Identifying filesystems using `blkid` and `lsblk`
* Mounting and unmounting filesystems
* Configuring persistent mounts using `/etc/fstab`
* Using UUIDs for persistent filesystem configuration
* Verifying filesystem configuration and disk usage


## Result

Successfully created and managed disk partitions using both MBR (DOS) and GPT partition tables. Created ext4 and XFS filesystems, mounted them to dedicated mount points, and configured persistent mounts using UUIDs in `/etc/fstab`. Verified partition layouts, filesystem types, UUIDs and mounted filesystems using `fdisk`, `lsblk`, `blkid` and `df`.
