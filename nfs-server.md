Configuring a Network File System (NFS) server on Red Hat Enterprise Linux (RHEL) 7 involves installing the necessary utility packages, creating a shared directory, configuring permissions, and allowing traffic through the system firewall. [1, 2] 
Follow this step-by-step guide to configure both the NFS server and the client machine. [1, 3] 
------------------------------
## Part 1: NFS Server Configuration## Step 1: Install NFS Packages
Log in as root or use a user with sudo privileges. Install the nfs-utils package, which provides the necessary daemons and management tools: [1, 4] 

sudo yum install -y nfs-utils

## Step 2: Start and Enable NFS Services
You need to enable and start both rpcbind (handling port-mapping) and the nfs-server service so they run automatically at system boot: [5, 6] 

sudo systemctl enable rpcbind nfs-server
sudo systemctl start rpcbind nfs-server

## Step 3: Create the Shared Directory
Create the directory you want to share with clients, and adjust its permissions so clients can read and write to it: [1, 2] 

sudo mkdir -p /mnt/nfs_share
sudo chmod 777 /mnt/nfs_share

## Step 4: Configure Exports (/etc/exports)
Define which directories to share and specify client access parameters. Open the configuration file using a text editor: [1, 3] 

sudo vi /etc/exports

Add a line pointing to your shared directory, the allowed client IP (or IP subnet range), and permissions: [1, 2] 

/mnt/nfs_share  192.168.1.0/24(rw,sync,no_root_squash,no_subtree_check)


* rw: Grants read and write access to the share. [2, 7] 
* sync: Forces the server to reply to requests only after changes have been committed to disk (safer against data loss). [8, 9] 
* no_root_squash: Allows the client's root user to access the shared directory with root privileges on the server. [2, 10] 
* no_subtree_check: Disables subtree checking, which improves reliability when files are being renamed.

Save and exit the file.
## Step 5: Export the Share
Run the exportfs command to cleanly export the configured directory without needing a full service restart: [1, 5] 

sudo exportfs -rav


* -r: Re-exports all directories.
* -a: Exports or unexports all directories.
* -v: Verbose output (displays what is being exported). [1, 7, 8, 10] 

## Step 6: Configure Firewalld
Ensure the local firewall allows incoming NFS traffic, as well as its ancillary services (rpc-bind and mountd): [3, 11] 

sudo firewall-cmd --permanent --add-service=nfs
sudo firewall-cmd --permanent --add-service=rpc-bind
sudo firewall-cmd --permanent --add-service=mountd
sudo firewall-cmd --reload

------------------------------
## Part 2: NFS Client Configuration
Run these steps on the client machine that will be connecting to your RHEL 7 NFS server. [12] 
## Step 1: Install NFS Utilities
Just like the server, the client requires the nfs-utils package to parse network file systems: [1] 

sudo yum install -y nfs-utils

## Step 2: Discover Server Shares
Verify that the client can communicate with the server and see its exported paths (replace 192.168.1.100 with your actual NFS Server IP): [3, 10] 

showmount -e 192.168.1.100

Expected Output:

Export list for 192.168.1.100:
/mnt/nfs_share 192.168.1.0/24

## Step 3: Create Mount Point and Mount the Share
Create a local directory where you want to attach the network share, and mount it: [1, 3] 

sudo mkdir -p /mnt/nfs_client
sudo mount -t nfs 192.168.1.100:/mnt/nfs_share /mnt/nfs_client

Verify the mount is active by tracking disk space: [7, 12] 

df -h | grep nfs

## Step 4: Configure Persistent Auto-Mounting (Optional)
Manual mounts clear away when the client system reboots. To make it permanent, append an entry to /etc/fstab: [3, 10, 13] 

sudo vi /etc/fstab

Add the following line to the bottom of the file: [13] 

192.168.1.100:/mnt/nfs_share  /mnt/nfs_client  nfs  defaults  0 0

Test the /etc/fstab syntax to ensure there are no errors before restarting: [13] 

sudo umount /mnt/nfs_client
sudo mount -a

If the mount -a command succeeds without errors, your NFS share will seamlessly reattach every time the client boots. [13] 
------------------------------
If you'd like to tailor this setup further, let me know:

* Will your clients be running RHEL 7 as well, or a different operating system?
* Do you need to restrict permissions to read-only (ro) for specific hosts? [8] 
* Are you planning to enforce stricter security via Kerberos / NFSv4 authentication? [14] 


[1] [https://www.redhat.com](https://www.redhat.com/en/blog/configure-nfs-linux)
[2] [https://www.tecmint.com](https://www.tecmint.com/how-to-setup-nfs-server-in-linux/)
[3] [https://www.redhat.com](https://www.redhat.com/en/blog/nfs-server-client)
[4] [https://bobcares.com](https://bobcares.com/blog/complete-guide-to-installing-nfs-server-and-client-on-rhel-systems/)
[5] [https://docs.redhat.com](https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/7/html/storage_administration_guide/s1-nfs-start)
[6] [https://www.redhat.com](https://www.redhat.com/it/blog/configure-nfs-linux)
[7] [https://www.youtube.com](https://www.youtube.com/watch?v=zs6fKCLIx6s&t=1)
[8] [https://docs.redhat.com](https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/7/html/storage_administration_guide/nfs-serverconfig)
[9] [https://docs.redhat.com](https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/7/html/storage_administration_guide/ch-nfs)
[10] [https://avinashdesireddy.medium.com](https://avinashdesireddy.medium.com/how-to-configure-nfs-server-and-client-on-rhel-7-52deb43a9c2a)
[11] [https://www.scribd.com](https://www.scribd.com/document/469619256/NFS)
[12] [https://www.youtube.com](https://www.youtube.com/watch?v=zmDIfJtCKCk&t=633)
[13] [https://docs.redhat.com](https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/7/html/storage_administration_guide/nfs-clientconfig)
[14] [https://docs.redhat.com](https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/8/html/deploying_different_types_of_servers/deploying-an-nfs-server_deploying-different-types-of-servers)
