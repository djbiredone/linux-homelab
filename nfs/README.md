## Objective

Deploy and configure an NFS Server on Rocky Linux 9 to share a directory with a remote NFS client.

---

## Environment

- NFS Server: Rocky Linux 9
- NFS Client: Rocky Linux 9
- Firewall: firewalld
- Virtualization: VMware ESXi

## Tasks

### NFS Server:

- Install the NFS utilities package
- Enable and start the NFS service
- Configure shared directories in /etc/exports
- Allow NFS-related services through firewalld

### NFS Client:

- Install the NFS utilities package
- Enable and start the NFS service
- Mount the shared directory from the NFS server
- Configure automatic mounting using /etc/fstab
- Verify access to the shared directory

## Configuration files

- /etc/exports
- /etc/fstab

## Commands


### NFS Server:

#### Install packages

dnf install nfs-utils


Installs the NFS utilities required for both the NFS server


#### Enable and start the service

systemctl enable --now nfs-server


Starts the service and enables it at boot


#### Edit the NFS configuration file

vi /etc/exports


Configuration:

/nfs_data 192.168.1.0/24(rw,no_root_squash)


Exports the /nfs_data directory to clients in the 192.168.1.0/24 network with read-write access and disables root user mapping (no_root_squash)
The no_root_squash option is used for demonstration purposes in this lab.


#### Change permissions for shared directory

chmod 0777 /nfs_data 


Gives read, write and execute permissions all users


#### Configure firewall

sudo firewall-cmd --permanent --add-service=nfs
sudo firewall-cmd --permanent --add-service=rpc-bind
sudo firewall-cmd --permanent --add-service=mountd
sudo firewall-cmd --reload


Permanently allows the services required for NFS through firewalld and reloads the firewall configuration


### NFS Client:


#### Install packages

dnf install nfs-utils


Installs the NFS utilities required for both the NFS client


#### Verify availability of resources shared by server

showmount -e 192.168.1.117


Displays directories shared by NFS server with IP 192.168.1.117


#### Create local mountpoint directory

mkdir /nfs_share


Creates /nfs_share directory which will be the mountpoint of remote resources shared by server


#### Mount remote directory on local mountpoint

mount -t nfs 192.168.1.117:/nfs_data /nfs_share


Mounts shared directory locally on /nfs_share


#### Configure automatic mounting using /etc/fstab

vi /etc/fstab

Configuration:

192.168.1.117:/nfs_data /nfs_share nfs defaults,_netdev 0 0


Configures automatic mounting of the remote directory during system boot


mount -a


Mounts all filesystems defined in /etc/fstab and verifies the configuration


## Verification

### NFS Server:

systemctl status nfs-server


Verifies that the NFS server service is running


exportfs -v


Displays all exported directories and their export options


firewall-cmd --list-services


Verifies that NFS-related services are allowed through firewalld



### NFS Client:

mount | grep nfs


Verifies that the remote NFS share is mounted



df -h


Displays mounted filesystems including the NFS share



touch /nfs_share/testfile
ls -l /nfs_share


Verifies read and write access to the shared directory


mount -a


Verifies that the /etc/fstab configuration is valid


## Skills Practiced

- NFS server configuration
- NFS client configuration
- Network file sharing
- Persistent filesystem mounting
- Linux filesystem permissions
- firewalld configuration
- systemd service management
- Filesystem verification and troubleshooting


## Result

The Rocky Linux system successfully provides an NFS share to a remote client. The shared directory is accessible over the network, persists across system reboots using /etc/fstab, and is protected by the configured firewall rules.
