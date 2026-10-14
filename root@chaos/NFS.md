NFS (Network File System) — RHCSA Notes
RHCSA Practical + Revision Notes
1. What is NFS?
NFS (Network File System) is a network protocol that allows a Linux server to share directories with other computers over a network.
- NFS Server: Shares directories.
- NFS Client: Accesses shared directories.
- Default port: 2049/TCP (NFSv4).
- Package: nfs-utils
- Service: nfs-server.service
Example: A server shares /mnt, and a client accesses its files through /tmp/mount without copying the data locally.
Server directory → Network → Client mount point
2. Configure NFS Server
Run these commands on the server (10.219.122.167).
Step 1 — Install package
dnf install -y nfs-utils


Step 2 — Start and enable service
systemctl enable --now nfs-server


--now starts the service immediately and enables it for future boots.
Step 3 — Configure firewall
firewall-cmd --permanent --add-service=nfs
firewall-cmd --reload


Allows NFS traffic through the firewall. For an NFSv4-only setup, the NFS service is sufficient; NFSv3 requires additional RPC-related services and ports. 

Red Hat Documentation
+1



Step 4 — Configure shared directory
mkdir -p /mnt
vim /etc/exports


Add:
/mnt 10.219.122.0/24(rw,sync,root_squash)


This shares /mnt with clients in the specified subnet.
Option	Meaning
rw	Allow reading and writing
ro	Read-only access
sync	Commit writes to stable storage before replying
root_squash	Map remote root to an unprivileged identity
no_root_squash	Disable root squashing — security risk
*	Allow any client — avoid on untrusted networks
Important: /etc/exports controls NFS sharing, not ordinary Linux filesystem permissions. For a writable share, configure suitable ownership and permissions as well. Avoid chmod 777 /mnt except in an isolated throwaway lab.
Step 5 — Apply exports
exportfs -rav


- -r — Re-export directories.
- -a — All exports.
- -v — Verbose output.
Step 6 — Verify
exportfs -v
systemctl status nfs-server


These check the exported directories and server status.
3. Configure NFS Client (Manual Mounting)
Run on the client.
Step 1 — Install package
dnf install -y nfs-utils


Step 2 — Create mount point
mkdir -p /tmp/mount


Step 3 — Mount NFS share
mount -t nfs 10.219.122.167:/mnt /tmp/mount


Explanation:
Part	Meaning
mount	Attach a filesystem
-t nfs	Filesystem type
10.219.122.167	NFS server IP
:/mnt	Shared server directory
/tmp/mount	Local client mount point
Step 4 — Verify
df -hT
findmnt /tmp/mount
ls /tmp/mount


Step 5 — Unmount
umount /tmp/mount


Note: It is umount, not unmount. Ordinary manual mounts are not automatically persistent after reboot.
4. Permanent NFS Mount Using /etc/fstab
For a mount that should persist across reboots:
vim /etc/fstab


Add:
10.219.122.167:/mnt /tmp/mount nfs defaults,_netdev 0 0


Then:
mount -a
findmnt /tmp/mount


_netdev identifies the mount as network-dependent, allowing systemd to handle it appropriately during startup.
5. AutoFS — Automatic On-Demand Mounting
AutoFS automatically mounts NFS shares when accessed and can unmount them after inactivity.
Unlike a regular /etc/fstab NFS mount, AutoFS does not need to keep the remote share continuously mounted.
Configure AutoFS on Client
Make sure /tmp/mount is not already manually mounted.
Step 1 — Install AutoFS
dnf install -y autofs nfs-utils


Step 2 — Configure master map
vim /etc/auto.master.d/nfs.autofs


Add:
/- /etc/auto.nfs


- /- → Direct mapping.
- /etc/auto.nfs → File containing mount configurations.
Step 3 — Configure NFS map
vim /etc/auto.nfs


Add:
/tmp/mount -fstype=nfs,rw 10.219.122.167:/mnt


Part	Meaning
/tmp/mount	Local mount point
-fstype=nfs	NFS filesystem
rw	Request read/write access
10.219.122.167:/mnt	NFS server and exported directory
Step 4 — Start AutoFS
systemctl enable --now autofs
systemctl restart autofs


Step 5 — Test automounting
ls /tmp/mount
findmnt -t nfs,nfs4


Accessing /tmp/mount triggers the NFS mount. 

Red Hat Documentation
+1



Note: Your teacher used /etc/auto.misc, which can work, but a separate /etc/auto.nfs is cleaner. Files inside /etc/auto.master.d/ must have the .autofs extension to be included automatically.
6. Direct vs Indirect AutoFS Mapping
Direct Mapping	Indirect Mapping
Uses /-	Uses a parent directory such as /shares
Requires an absolute mount path	Uses relative keys
Example: /tmp/mount	Example: /shares/data
Indirect map example:
/etc/auto.master.d/nfs.autofs
/shares /etc/auto.nfs


/etc/auto.nfs
data -fstype=nfs,rw 10.219.122.167:/mnt


Accessing /shares/data triggers the mount.
7. Important NFS Commands — Quick Revision
Command	Purpose
exportfs -v	Display exported directories
exportfs -rav	Reload export configuration
mount -t nfs	Manually mount NFS
umount	Unmount filesystem
findmnt -t nfs,nfs4	Show NFS mounts
df -hT	Show mounted filesystem types and sizes
systemctl status nfs-server	Check NFS server
systemctl status autofs	Check AutoFS
automount -m	Display AutoFS map information
journalctl -u autofs -xe	Troubleshoot AutoFS
8. Common RHCSA Mistakes
- Permission denied: Check /etc/exports, directory permissions, UID/GID mapping, root squashing and SELinux.
- Connection refused/timed out: Check network, NFS service and firewall.
- AutoFS not working: Check map paths, .autofs file extension, syntax and service status.
- Mount not persistent: Use /etc/fstab or AutoFS according to the requirement.
- Root cannot create files: root_squash may be responsible. Do not disable it without a justified requirement.
- Mount conflict: Avoid configuring the same mount point in both /etc/fstab and AutoFS.
- soft,intr from the whiteboard: Prefer the default hard behavior for normal NFS storage; soft can cause I/O errors and potential data loss, while intr is obsolete on modern Linux NFS clients.

9. RHCSA Practice Challenge
Try this without looking at the commands:
Practice checklist
0/6



[ ] Configure an NFS server to export /mnt.
[ ] Allow only the 10.219.122.0/24 subnet.
[ ] Mount the share manually on a client at /tmp/mount.
[ ] Unmount it and configure AutoFS direct mapping.
[ ] Verify that accessing /tmp/mount triggers the NFS mount.
[ ] Ensure AutoFS starts automatically after reboot.

Final memory trick:
NFS Server → /etc/exports → exportfs → NFS Client → mount → AutoFS
Focus especially on /etc/exports, mount, /etc/auto.master.d/, and AutoFS maps. These are the commands and configurations you should be able to reproduce from memory during an RHCSA practical exercise.
