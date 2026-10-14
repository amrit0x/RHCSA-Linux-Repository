### Linux NFS Server and Client — Complete Practical Notes

These notes are based on the whiteboard image and organized in a simple, classroom-style format. The chapter covers NFS Server configuration, NFS Client mounting, and automatic mounting using Autofs on Linux.

### 1\. What is NFS?

NFS (Network File System) is a protocol that allows a Linux system to share directories with other Linux systems over a network.

For example, suppose we have two Linux machines:

* NFS Server: `10.219.122.167`

* NFS Client: A machine that accesses the server's shared directory.

The server shares a directory, and the client can access it as if it were a local directory.

NFS Server

IP: 10.219.122.167

Shared directory: /mnt

Network communication

NFS Client

Mount point: /tmp/nfsmount

Accesses files shared by the server

### Why do we use NFS?

* To share files between Linux servers.

* To access centralized storage from multiple machines.

* To share application data across servers.

* To avoid maintaining duplicate copies of files on every system.

Important: NFS does not automatically copy all the files to the client. The client accesses the shared filesystem over the network.

### 2\. NFS Server Configuration

The NFS server is responsible for exporting a directory so that authorized clients can access it.

### Step 1: Verify network connectivity

```bash
ping <client-or-server-IP>
```

For example, from the client:

```bash
ping 10.219.122.167
```

This checks basic IP connectivity. A failed ping does not always mean NFS is unavailable because ICMP traffic may be blocked.

### Step 2: Install NFS utilities

On a RHEL-compatible system:

```bash
yum install nfs-utils -y
```

On newer systems, `dnf` can also be used:

```bash
dnf install nfs-utils -y
```

### Step 3: Enable and start the NFS server

```bash
systemctl enable --now nfs-server
```

Check the service status:

```bash
systemctl status nfs-server
```

* `start` starts the service now.

* `enable` configures it to start automatically at boot.

* `--now` performs both actions.

### Step 4: Create the shared directory

```bash
mkdir -p /mnt
```

The whiteboard uses `/mnt` as the shared directory.

Lab note: `/mnt` is commonly used as a temporary mount location. For a dedicated NFS share, a path such as `/srv/nfs/share` can make the configuration clearer.

### Step 5: Configure `/etc/exports`

Open the exports configuration file:

```bash
vim /etc/exports
```

Example configuration:

```text
/mnt *(rw,sync)
```

A more restrictive example for a trusted lab network is:

```text
/mnt 10.219.122.0/24(rw,sync)
```

Meaning:

| Option | Meaning                                               |
| ------ | ----------------------------------------------------- |
| /mnt   | Directory to share                                    |
| *      | Allows any client, subject to other access controls   |
| rw     | Allows read and write access                          |
| sync   | Replies after changes are committed to stable storage |

Security note: Avoid `*` on production systems. Specify authorized client IPs or networks, and configure firewall and filesystem permissions appropriately.

### Step 6: Apply the exports configuration

```bash
exportfs -rav
```

Useful verification commands:

```bash
exportfs -v
showmount -e localhost
```

* `exportfs -rav` reloads the configured exports.

* `exportfs -v` displays exported directories and their options.

* `showmount -e localhost` lists exports advertised by the local server.

### Step 7: Configure the firewall

If `firewalld` is running:

```bash
firewall-cmd --permanent --add-service=nfs
firewall-cmd --reload
```

Depending on the NFS version and environment, additional RPC-related services or ports may be required. NFSv4 commonly uses TCP port 2049.

### 3\. Set directory permissions

The board shows this command:

```bash
chmod 777 /mnt
```

It gives read, write, and execute permissions to everyone on the server for that directory.

However, `chmod 777` is not recommended for production. Use appropriate ownership and permissions instead. Remember that NFS access also depends on client-side user IDs, server-side permissions, and export options.

### 4\. Restart and verify NFS services

The whiteboard also mentions restarting the NFS server and related services.

```bash
systemctl restart nfs-server
systemctl status nfs-server
```

On systems using the traditional RPC services, check `rpcbind` if required:

```bash
systemctl status rpcbind
```

Enable it when needed:

```bash
systemctl enable --now rpcbind
```

Modern NFSv4-only setups may not require `rpcbind` for normal NFS client mounting.

### 5\. NFS Client Configuration

The NFS client connects to the server and mounts its exported directory.

### Step 1: Install NFS utilities

Run on the client:

```bash
yum install nfs-utils -y
```

### Step 2: Verify the server's exports

```bash
showmount -e 10.219.122.167
```

Expected output for our lab might look like:

```text
Export list for 10.219.122.167:
/mnt 10.219.122.0/24
```

The output depends on the actual server configuration and permitted clients.

### Step 3: Create the client mount point

The whiteboard uses `/tmp/nfsmount`.

```bash
mkdir -p /tmp/nfsmount
```

### Step 4: Mount the NFS share manually

```bash
mount -t nfs 10.219.122.167:/mnt /tmp/nfsmount
```

Here is what each part means:

| Component           | Purpose                                            |
| ------------------- | -------------------------------------------------- |
| mount               | Mounts a filesystem                                |
| -t nfs              | Specifies the filesystem type                      |
| 10.219.122.167:/mnt | Server IP and exported directory                   |
| /tmp/nfsmount       | Local directory where the share becomes accessible |

After mounting, files in the server's `/mnt` directory are accessible through `/tmp/nfsmount` on the client.

### Step 5: Verify the mount

```bash
df -hT
```

Or check only the relevant mount:

```bash
findmnt /tmp/nfsmount
```

Test file access:

```bash
ls -la /tmp/nfsmount
touch /tmp/nfsmount/client-test.txt
ls -l /tmp/nfsmount
```

If the client has write access, the test file should appear in the server's `/mnt` directory.

### Step 6: Unmount the NFS share

```bash
umount /tmp/nfsmount
```

If the filesystem is busy, check which processes are using it before attempting to unmount again.

```bash
fuser -vm /tmp/nfsmount
```

### 6\. What is Autofs?

Autofs is a Linux service that automatically mounts a filesystem when a user or application accesses its mount point and can unmount it after a period of inactivity.

Without Autofs, you may need to mount an NFS share manually or configure it in `/etc/fstab`.

With Autofs, accessing a configured path triggers the mount automatically.

### How Autofs works

1\. User accesses a directory

`ls /tmp/nfsmount`

2\. Autofs detects the access

Reads its master map and mount map

3\. NFS share is mounted

Connects to `10.219.122.167:/mnt`

4\. Files become accessible

The user works with the remote files

### 7\. Configure Autofs on the NFS client

The right side of your whiteboard covers this configuration.

### Step 1: Install Autofs

```bash
yum install autofs nfs-utils -y
```

### Step 2: Create a master map file

The board shows a file under `/etc/auto.master.d/`.

For this lab, create:

```bash
vim /etc/auto.master.d/file.autofs
```

Add the following line:

```text
/-  /etc/auto.misc
```

Save the file.

Meaning of `/-`: It tells Autofs that the map contains direct mount paths, such as `/tmp/nfsmount`, rather than paths nested beneath one common parent directory.

### Step 3: Configure the map file

Open:

```bash
vim /etc/auto.misc
```

Add this line:

```text
/tmp/nfsmount  -rw  10.219.122.167:/mnt
```

The structure is:

```text
<mount-point>  <mount-options>  <NFS-server>:<exported-directory>
```

For this lab:

* `/tmp/nfsmount` — local mount point.

* `-rw` — request read-write mounting.

* `10.219.122.167:/mnt` — remote NFS share.

The client must still be authorized by the server, and its filesystem permissions must allow the requested operations.

### Step 4: Start and enable Autofs

```bash
systemctl enable --now autofs
```

Verify the service:

```bash
systemctl status autofs
```

If you edit the configuration afterward, restart the service:

```bash
systemctl restart autofs
```

### Step 5: Test automatic mounting

Do not manually mount the NFS share at this path for the Autofs test.

Access the directory:

```bash
ls -la /tmp/nfsmount
```

Then verify:

```bash
findmnt /tmp/nfsmount
df -hT
```

The first access triggers the automatic mount. If the server is reachable and the configuration is correct, the remote files should be visible.

Important: Autofs may unmount an inactive share automatically after its configured timeout. That is expected behavior.

### 8\. Complete configuration summary

### NFS Server — `10.219.122.167`

```bash
yum install nfs-utils -y

mkdir -p /mnt

vim /etc/exports
```

Example `/etc/exports` entry:

```text
/mnt 10.219.122.0/24(rw,sync)
```

```bash
systemctl enable --now nfs-server
exportfs -rav
exportfs -v
```

### NFS Client — Manual Mount

```bash
yum install nfs-utils -y

mkdir -p /tmp/nfsmount

mount -t nfs \
  10.219.122.167:/mnt \
  /tmp/nfsmount

findmnt /tmp/nfsmount
umount /tmp/nfsmount
```

### NFS Client — Autofs

Install the packages:

```bash
yum install autofs nfs-utils -y
```

`/etc/auto.master.d/file.autofs`:

```text
/-  /etc/auto.misc
```

`/etc/auto.misc`:

```text
/tmp/nfsmount -rw 10.219.122.167:/mnt
```

Start the service and test:

```bash
systemctl enable --now autofs

ls -la /tmp/nfsmount
findmnt /tmp/nfsmount
```

### 9\. Common troubleshooting commands

| Problem                   | Command to investigate                              |
| ------------------------- | --------------------------------------------------- |
| Server unreachable        | ping 10.219.122.167                                 |
| NFS service stopped       | systemctl status nfs-server                         |
| Exports missing           | exportfs -v                                         |
| Client cannot see exports | showmount -e 10.219.122.167                         |
| NFS mount failed          | journalctl -u nfs-server -u autofs --no-pager -n 50 |
| Permission denied         | ls -ld /mnt and namei -l /mnt                       |
| Autofs not running        | systemctl status autofs                             |
| Verify active mount       | findmnt /tmp/nfsmount                               |

### 10\. Practice questions

Use these questions to revise the practical before an RHCSA-style lab or interview.

1. What is the primary purpose of NFS?

2. Which file defines NFS server exports?

3. Which command reloads NFS export definitions?

4. Which package provides NFS client utilities on RHEL?

5. What does mount -t nfs do?

6. What is the purpose of /etc/auto.master.d/file.autofs?

7. What does /- mean in the Autofs master map?

8. How do you trigger an Autofs mount?

9. What is the difference between manual mounting and Autofs?

10. Why should chmod 777 be avoided in production?

Final takeaway: NFS provides network-based file sharing. The NFS server exports directories, the client mounts them, and Autofs automates mounting when those directories are accessed. In practical exams, pay particular attention to `/etc/exports`, `exportfs`, `mount`, `/etc/auto.master.d/`, map-file syntax, and the `autofs` service.
