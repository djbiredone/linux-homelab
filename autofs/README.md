## Objective

Configure AutoFS on a Rocky Linux client using direct and indirect maps to automatically mount NFS file systems on demand

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
- Add users and create home and shared directories

### NFS Client:

- Install the AutoFS package
- Enable and start the autofs service
- Configure Master Map
- Configure Indirect Map
- Configure Direct Map
- Add users and create home directories
- Restart the AutoFS service
- Verify access to the shared directories

## Configuration files

- /etc/exports
- /etc/auto.master
- /etc/auto.master.d/extra.autofs
- /etc/auto.home
- /etc/auto.data

## Commands


### NFS Server:

#### Install packages

dnf install nfs-utils


Installs the NFS utilities required for the NFS server


#### Enable and start the service

systemctl enable --now nfs-server


Starts the service and enables it at boot


#### Edit the NFS configuration file

vi /etc/exports


Configuration:

/rhome 192.168.1.0/24(rw,no_root_squash)
/data 192.168.1.0/24(rw,sync)


Exports the /rhome directory to clients in the 192.168.1.0/24 network with read-write access and disables root user mapping (no_root_squash)

Exports the /data directory to clients in the 192.168.1.0/24 network with read-write access and synchronizes data immediately

The no_root_squash option is used for demonstration purposes in this lab.


#### Configure firewall

firewall-cmd --permanent --add-service=nfs
firewall-cmd --permanent --add-service=mountd
firewall-cmd --permanent --add-service=rpc-bind
firewall-cmd --reload


Permanently allows the services required for NFS through firewalld and reloads the firewall configuration


#### Add users and create home and shared directories

mkdir -p /rhome /shares/data


Creates the /rhome directory for exported user home directories and the /shares/data directory for shared data



chmod 0777 /shares/data


Gives read, write and execute permissions to all users on the /shares/data directory (the 0777 permissions are used for demonstration purposes in this lab to simplify access from multiple users)



for user in alice bob tutor; do useradd -m -d /rhome/$user $user && chown -R $user:$user /rhome/$user; done


Creates the users alice, bob and tutor with home directories located under /rhome and assigns the correct ownership to each home directory


### NFS Client:


#### Install packages

dnf install autofs 


Installs AutoFS package required on the NFS client


#### Verify availability of resources shared by server

showmount -e 192.168.1.117


Displays directories shared by NFS server with IP 192.168.1.117


#### Create Master Map


vi /etc/auto.master.d/extra.autofs 


Content:

/rhome  /etc/auto.home    --timeout=60
/-      /etc/auto.data    --timeout=60


Creates a custom AutoFS Master Map that defines the indirect and direct maps and sets the automount timeout to 60 seconds

/rhome defines an indirect map for user home directories
/- defines a direct map for explicitly configured mount points
The --timeout=60 option automatically unmounts inactive file systems after 60 seconds


#### Create Indirect Map

vi /etc/auto.home


Content:

* -rw 192.168.1.117:/rhome/&


Creates a wildcard map entry that automatically mounts users' home directories from the NFS server. The & symbol is replaced with the requested key (username)

* – wildcard matches any username; the wildcard entry allows a single rule to be used for all users instead of creating a separate map entry for each home directory
& – expands to the matched key (for example alice or bob)
-rw – mounts the NFS share with read-write permissions


#### Create Direct Map

vi /etc/auto.data


Content:

/data -rw 192.168.1.117:/shares/data


Creates a direct map entry that automatically mounts the NFS export /shares/data from the server to the local /data mount point


#### Add users

for user in tutor alice bob; do  useradd -m -d /rhome/$user $user; done 


Creates the users alice, bob and tutor and assigns their home directories under the AutoFS-managed /rhome mount point


#### Restart the AutoFS service

systemctl restart autofs


Restarts the AutoFS service and applies the new map configuration


## Verification


### NFS Server:

systemctl status nfs-server


Verifies the NFS service is running


### NFS Client:

systemctl status autofs


Verifies the AutoFS service is running



automount -m


Displays the active AutoFS master and map entries



ls -l /rhome/alice
ls -l /rhome/bob
ls -l /rhome/tutor 


Triggers automatic mounting of users' home directories through the indirect map


After 60 seconds:

mount | grep nfs


Verifies that inactive NFS mounts are automatically removed after the configured timeout.



cat /proc/mounts 


Displays all currently mounted file systems, including AutoFS and NFS mounts



df -h 


Displays mounted file systems and their disk usage.



df -hT | grep data 


Verifies that the NFS export is mounted on the local /data mount point




for user in tutor alice bob; do su - $user -c "pwd && df -hT ."; done 


Verifies that each user's home directory is automatically mounted through AutoFS and displays the corresponding file system



mount | grep autofs


Displays active AutoFS mount points


## Skills Practiced

- AutoFS configuration
- NFS administration
- Linux storage administration
- On-demand filesystem mounting
- Direct maps
- Indirect maps
- systemd service management
- Basic Bash scripting (for loops)


## Result

Remote home directories and shared data are mounted automatically when accessed and are unmounted after the configured timeout, reducing unnecessary permanent mounts

