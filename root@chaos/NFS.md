

# NFS (Network File System) — RHCSA Notes

RHCSA Practical + Revision Notes

## **1. What is NFS?**

NFS (Network File System) is a network protocol that allows a Linux server to share directories with other computers over a network.

- NFS Server: Shares directories.
- NFS Client: Accesses shared directories.
- Default port: `2049/TCP` (NFSv4).
- Package: `nfs-utils`
- Service: `nfs-server.service`

**Important :  we make mount point in NFC client/(our system) of an particular directory of NFS server**

Example: A server shares `/mnt`, and a client accesses its files through `/tmp/mount` without copying the data locally.

!download.svg

                                       Server directory      →     Network       →       Client mount point

→ now if do any changes like creating files or whatever in */tmp/mount* it will automatically reflect in NFS server

## **2. Configure NFS Server (if NFS server is also rhel)**

Run these commands on the server (`10.219.122.167`).

Step 1 — Install package

```bash
dnf install -y nfs-utils
```

Step 2 — Start and enable service

```bash
systemctl enable --now nfs-server
```

- `-now` starts the service immediately and enables it for future boots.

Step 3 — Configure firewall

```bash
firewall-cmd --permanent --add-service=nfs
firewall-cmd --reload
```

Allows NFS traffic through the firewall. 

Step 4 — Configure shared directory

```bash
mkdir -p /mnt
vim /etc/exports
```

Add:

```bash
/mnt 10.219.122.0/24/(or your client IP)(rw,sync,root_squash)
```

This shares `/mnt` with clients in the specified subnet.

| Option | Meaning |
| --- | --- |
| `rw` | Allow reading and writing |
| `ro` | Read-only access |
| `sync` | Commit writes to stable storage before replying |
| `root_squash` | Map remote root to an unprivileged identity |
| `no_root_squash` | Disable root squashing — security risk |
| `*` | Allow any client — avoid on untrusted networks |

Important: `/etc/exports` controls NFS sharing, not ordinary Linux filesystem permissions. For a writable share, configure suitable ownership and permissions as well. Avoid `chmod 777 /mnt` except in an isolated throwaway lab.

Step 5 — Apply exports

```bash
exportfs -rav
```

- `r` — Re-export directories.
- `a` — All exports.
- `v` — Verbose output.

**Remember** : Whenever you modify `/etc/exports`, run `exportfs -rav` to apply the changes.

Step 6 — Verify

```bash
exportfs -v
systemctl status nfs-server
```

These check the exported directories and server status.

## 3. Configure NFS Client (Manual Mounting)

Run on the client.

Step 1 — Install package

```bash
dnf install -y nfs-utils
```

Step 2 — Create mount point

```bash
mkdir -p /tmp/mount
```

Step 3 — Mount NFS share

```bash
mount -t nfs 10.219.122.167:/mnt /tmp/mount
```

Explanation:

| Part | Meaning |
| --- | --- |
| `mount` | Attach a filesystem |
| `-t nfs` | Filesystem type |
| `10.219.122.167` | NFS server IP |
| `:/mnt` | Shared server directory |
| `/tmp/mount` | Local client mount point |

Step 4 — Verify

```bash
df -hT /tmp/mount  # Verify NFS filesystem and mount details.
mountpoint /tmp/mount # Verify the directory is a mount point.
ls /tmp/mount
```

Step 5 — Unmount

```bash
umount /tmp/mount
```

## 4. Permanent NFS Mount Using /etc/fstab

For a mount that should persist across reboots:

```bash
vim /etc/fstab
```

Add:

```
10.219.122.167:/mnt /tmp/mount nfs defaults,_netdev 0 0
```

Then:

```bash
mount -a # Mount all configured filesystems
findmnt /tmp/mount # Verify the mount
```

`_netdev` identifies the mount as network-dependent, allowing systemd to handle it appropriately during startup.

## **5. AutoFS — Automatic On-Demand Mounting**

AutoFS automatically mounts NFS shares when accessed and can unmount them after inactivity.

Unlike a regular `/etc/fstab` NFS mount, AutoFS does not need to keep the remote share continuously mounted.

**Configure AutoFS on Client :** Make sure `/tmp/mount` is not already manually mounted.

Step 1 — Install AutoFS

```bash
dnf install -y autofs nfs-utils
```

Step 2 — Configure master map

```bash
vim /etc/auto.master.d/nfs.autofs
```

Add:

```
/- /etc/auto.nfs
```

- `/-` → Direct mapping.
- `/etc/auto.nfs` → File containing mount configurations.

Step 3 — Configure NFS map

```bash
vim /etc/auto.nfs
```

Add: 

```
/tmp/mount -fstype=nfs,rw 10.219.122.167:/mnt
```

| Part | Meaning |
| --- | --- |
| `/tmp/mount` | Local mount point |
| `-fstype=nfs` | NFS filesystem |
| `rw` | Request read/write access |
| `10.219.122.167:/mnt` | NFS server and exported directory |

Step 4 — Start AutoFS

```bash
systemctl enable --now autofs
systemctl restart autofs
```

Step 5 — Test automounting

```
ls /tmp/mount
findmnt -t nfs,nfs4
```

Accessing `/tmp/mount` triggers the NFS mount.

Note: Your teacher used `/etc/auto.misc`, which can work, but a separate `/etc/auto.nfs` is cleaner. Files inside `/etc/auto.master.d/` must have the `.autofs` extension to be included automatically.

## 6. Important NFS Commands — Quick Revision

| Command | Purpose |
| --- | --- |
| `exportfs -v` | Display exported directories |
| `exportfs -rav` | Reload export configuration |
| `mount -t nfs` | Manually mount NFS |
| `umount` | Unmount filesystem |
| `findmnt -t nfs,nfs4` | Show NFS mounts |
| `df -hT` | Show mounted filesystem types and sizes |
| `systemctl status nfs-server` | Check NFS server |
| `systemctl status autofs` | Check AutoFS |

## 7. RHCSA Practice Challenge

Try this without looking at the commands:

Practice checklist

0/6

- [ ]  Configure an NFS server to export /mnt.
- [ ]  Allow only the 10.219.122.0/24 subnet.
- [ ]  Mount the share manually on a client at /tmp/mount.
- [ ]  Unmount it and configure AutoFS direct mapping.
- [ ]  Verify that accessing /tmp/mount triggers the NFS mount.
- [ ]  Ensure AutoFS starts automatically after reboot.

Final memory trick:

`NFS Server → /etc/exports → exportfs → NFS Client → mount → AutoFS`

Focus especially on `/etc/exports`, `mount`, `/etc/auto.master.d/`, and AutoFS maps. These are the commands and configurations you should be able to reproduce from memory during an RHCSA practical exercise.
