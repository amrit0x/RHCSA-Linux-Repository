# Part-2 | Storage Management

Linux provides several mechanisms to manage memory and storage efficiently. In this session, we will cover:

* Swap Space
* Swap Management
* Swappiness
* Logical Volume Manager (LVM)
* Physical Volume (PV)
* Volume Group (VG)
* Logical Volume (LV)
* Physical Extents (PE)
* LVM resizing and expansion

---

# Swap Space

## What is Swap?

Physical RAM is a hardware component installed inside the system. For example, if a server has **12 GB RAM**, we cannot simply stretch that physical RAM to make it larger.

To solve this problem, Linux provides a concept called **Swap**.

Swap is a portion of disk storage that is used as **virtual memory**.

We can understand it with the example of a mobile phone.

Some mobile phones show something like:

```text
12 GB RAM + 4 GB RAM Expansion
```

The additional 4 GB is not actually physical RAM. A portion of the phone's internal storage is used as additional memory.

Linux Swap works on a similar concept.

```text
Physical RAM
     +
   Swap
     =
Available Memory Space
```

Swap is therefore used as a **safety net when physical RAM becomes heavily utilized**.

---

# How Swap Works

When a process is running, its required memory is normally kept in physical RAM.

For example:

```text
Process
   |
   v
Physical RAM
   |
   v
CPU
```

As more processes start running, more RAM is consumed.

Eventually, the system may reach a situation where physical RAM is almost full.

Linux can then move some less-active memory pages from RAM to Swap.

This process is called **Swap Out**.

```text
RAM
 |
 | Swap Out
 v
Swap
```

When that memory is required again, Linux can bring it back into RAM.

This is called **Swap In**.

```text
Swap
 |
 | Swap In
 v
RAM
```

---

# Why Does Linux Move Memory to Swap?

If we run:

```bash
top
```

we can see that many processes are not actively using the CPU all the time.

A large number of processes may be in states such as:

```text
S = Sleeping
Z = Zombie
```

These processes may still occupy memory even though they are not currently performing active CPU work.

When memory pressure increases, Linux can move less-active memory pages to Swap.

This frees physical RAM for processes that currently need it.

The overall idea is:

```text
RAM becomes full
       |
       v
Less-active memory
       |
       v
Move to Swap
       |
       v
Free RAM
       |
       v
Active processes can use RAM
```

---

# Important Point About Swap Performance

Swap is useful, but it is **not a replacement for physical RAM**.

RAM is much faster than disk storage.

Therefore:

```text
RAM  → Faster
Swap → Slower
```

If the system continuously depends heavily on Swap, system performance can decrease.

Swap mainly provides additional memory capacity and helps prevent memory exhaustion.

---

# Checking RAM and Swap

Use:

```bash
free -h
```

This displays RAM and Swap information in a human-readable format.

Example:

```text
              total   used   free
Mem:           ...
Swap:          ...
```

---

# Creating a Swap Partition

The general workflow for creating Swap is:

```text
Identify Disk
     |
     v
Create Partition
     |
     v
Change Partition Type
     |
     v
Create Swap Signature
     |
     v
Activate Swap
     |
     v
Configure /etc/fstab
```

---

## Identify the Disk

First check the available disks and partitions:

```bash
lsblk
```

For example:

```text
/dev/nvme0n2
```

---

## Create a Partition Using fdisk

Run:

```bash
fdisk /dev/nvme0n2
```

Inside `fdisk`:

```text
n
```

is used to create a new partition.

Select the required partition type and use the default values where appropriate.

For example, to create a 1 GB partition:

```text
+1G
```

---

## Change the Partition Type

After creating the partition, change its type to Swap.

Inside `fdisk`:

```text
t
```

Then specify the Swap type.

The session uses:

```text
82
```

After that, verify the partition table:

```text
p
```

Finally, write the changes:

```text
w
```

---
# Create the Swap Signature

After creating the partition, format it as Swap using:

```bash
mkswap /dev/nvme0n2p1
```

This creates the Swap signature on the partition.

The command also provides a UUID for the Swap device.

---

# Activate Swap

Activate the Swap partition using:

```bash
swapon /dev/nvme0n2p1
```

After activation, verify it using:

```bash
lsblk
```

The partition should now appear with Swap information.

You can also check Swap usage using:

```bash
free -h
```

---

# Deactivate Swap

To deactivate a particular Swap partition:

```bash
swapoff /dev/nvme0n2p1
```

This disables the Swap area.

---

# Activate or Deactivate All Swap

Linux provides the `-a` option for working with all configured Swap areas.

Activate all Swap entries configured in `/etc/fstab`:

```bash
swapon -a
```

Deactivate all active Swap areas:

```bash
swapoff -a
```

---

# Make Swap Permanent

If we only run:

```bash
swapon /dev/nvme0n2p1
```

the Swap will be active until the system is rebooted.

After reboot, the configuration needs to be available so that Linux can activate the Swap automatically.

For this, we configure:

```text
/etc/fstab
```

---

## Find the UUID

Use:

```bash
blkid /dev/nvme0n2p1
```

This displays the UUID of the partition.

---

## Edit /etc/fstab

Open:

```bash
vim /etc/fstab
```

Add the Swap configuration:

```text
UUID="<your-unique-device-uuid>"   none   swap   defaults   0   0
```

The important fields are:

```text
UUID
none
swap
defaults
0
0
```

### Why is the mount point `none`?

A normal filesystem is mounted to a directory such as:

```text
/data
/home
/backup
```

Swap is different.

Swap is not used to store normal user files. It is used by the operating system as virtual memory.

Therefore, its mount point is:

```text
none
```

---

# Removing a Swap Partition

When removing Swap, follow the reverse order of the creation process.

The correct sequence is:

```text
Deactivate Swap
      |
      v
Remove /etc/fstab entry
      |
      v
Remove filesystem signature
      |
      v
Delete partition
```

First deactivate it:

```bash
swapoff /dev/nvme0n2p1
```

Then remove or comment out its entry from:

```text
/etc/fstab
```

Next remove the filesystem signature:

```bash
wipefs /dev/nvme0n2p1
```

Finally, open the disk using:

```bash
fdisk /dev/nvme0n2
```

Inside `fdisk`:

```text
d
```

Delete the required partition and then save the changes:

```text
w
```

---

# Swappiness

## What is Swappiness?

**Swappiness** is a Linux kernel parameter that controls how aggressively the system uses Swap.

In simple words, it controls the kernel's tendency to move inactive memory pages from RAM to Swap.

The commonly used default value is:

```text
60
```

The value can be changed depending on the system requirement.

---

# Understanding Swappiness Values

A higher value means the system is more willing to use Swap.

For example:

```text
80
90
```

A lower value means the system prefers to keep memory in RAM for longer and use Swap less aggressively.

For example:

```text
30
```

The basic idea is:

```text
Higher Swappiness
        |
        v
More aggressive Swap usage
```

and:

```text
Lower Swappiness
        |
        v
Less aggressive Swap usage
```

---

# Check Current Swappiness

Use:

```bash
cat /proc/sys/vm/swappiness
```

Example:

```text
60
```

The `/proc` filesystem provides runtime information and kernel parameters.

---

# Change Swappiness Temporarily

We can change the value at runtime using:

```bash
sysctl vm.swappiness=70
```

This changes the value immediately.

However, this runtime change does not remain after a reboot unless it is configured permanently.

---

# Configure Swappiness Permanently

Open:

```bash
vim /etc/sysctl.conf
```

Add:

```text
vm.swappiness = 80
```

Save the file.

Then apply the configuration:

```bash
sysctl -p
```

This loads the configuration into the running kernel without requiring a reboot.

---

# Logical Volume Manager — LVM

Now we move from memory management to **storage management**.

In production environments, storage requirements continuously change.

For example, suppose an application server has a filesystem with limited free space.

After some time:

```text
Filesystem = 100% full
```

The application may stop working properly.

In a traditional partitioning approach, increasing the partition size can be difficult and may require significant maintenance.

This is where **LVM** becomes useful.

---

# What is LVM?

**LVM stands for Logical Volume Manager.**

LVM provides a logical layer between physical storage and the filesystem.

It allows administrators to manage storage more flexibly.

One of the major advantages of LVM is that storage can be expanded dynamically.

For example, we can have:

```text
Disk 1 = 2 GB
Disk 2 = 1 GB
```

These can be combined into a storage pool.

The storage pool can then be divided into Logical Volumes according to our requirements.

---

# LVM Architecture

LVM has three important layers:

```text
Logical Volume (LV)
        |
        v
Volume Group (VG)
        |
        v
Physical Volume (PV)
        |
        v
Physical Disk
```

The complete structure can be visualized as:

```text
+---------------------------------------------+
|             Logical Volumes (LV)            |
|                                             |
| /dev/myvg/mylv     /dev/myvg/mylv2          |
+---------------------------------------------+
|              Volume Group (VG)              |
|                                             |
|              Storage Pool                   |
+---------------------------------------------+
|             Physical Volumes (PV)            |
|                                             |
| /dev/nvme0n2p1      /dev/nvme0n2p2          |
+---------------------------------------------+
|               Physical Disks                |
+---------------------------------------------+
```

---

# PV — Physical Volume

A **Physical Volume (PV)** is a disk or partition that has been initialized for LVM.

For example:

```text
/dev/nvme0n2p1
```

can be converted into a Physical Volume using:

```bash
pvcreate /dev/nvme0n2p1
```

---

# VG — Volume Group

A **Volume Group (VG)** is a storage pool created by combining one or more Physical Volumes.

For example:

```text
PV1 = 2 GB
PV2 = 1 GB

        |
        v

VG = 3 GB storage pool
```

Create a Volume Group using:

```bash
vgcreate myvg /dev/nvme0n2p1
```

---

# LV — Logical Volume

A **Logical Volume (LV)** is a virtual storage area created from the free space available inside a Volume Group.

For example:

```text
VG = 3 GB

       |
       +---- LV1 = 500 MB
       |
       +---- LV2 = 1 GB
       |
       +---- Free Space
```

Create an LV using:

```bash
lvcreate -L +500M -n mylv myvg
```

The resulting device can be accessed as:

```text
/dev/myvg/mylv
```

---

# LVM Water Pool Analogy

A simple way to understand LVM is to imagine water.

### Physical Volume — Bucket

Each PV is like a bucket containing water.

```text
Bucket 1 → PV1
Bucket 2 → PV2
```

### Volume Group — Large Pool

The water from the buckets is combined into one large swimming pool.

```text
PV1 + PV2
    |
    v
Volume Group
```

### Logical Volume — Container

From this large pool, we can create smaller containers according to our requirements.

```text
Volume Group
     |
     +---- LV1
     |
     +---- LV2
     |
     +---- LV3
```

This makes the relationship easy to remember:

```text
PV → VG → LV
```

---

# Physical Extents — PE

LVM divides storage into smaller chunks called **Physical Extents (PEs)**.

These are the basic allocation units used inside the Volume Group.

The session uses a default PE size of:

```text
4 MB
```

For example:

```text
VG
 |
 +-- PE
 +-- PE
 +-- PE
 +-- PE
```

When an LV is created, storage is allocated using these PE units.

---

# Custom PE Size

The PE size can be specified while creating a Volume Group.

For example:

```bash
vgcreate -s 8m myvg /dev/nvme0n2p1
```

Here:

```text
-s 8m
```

sets the PE size to:

```text
8 MB
```

Storage allocation is therefore performed in multiples of the configured PE size.

---

# LVM Practical Workflow

The complete LVM workflow is:

```text
Create Partition
      |
      v
Create PV
      |
      v
Create VG
      |
      v
Create LV
      |
      v
Create Filesystem
      |
      v
Mount LV
```

Let's go through each step.

---

# Create Physical Volume

First create the required partition using `fdisk`.

The session uses partition type:

```text
8e
```

for Linux LVM on MBR.

After creating the partition, initialize it as a Physical Volume:

```bash
pvcreate /dev/nvme0n2p1
```

---

## Verify Physical Volumes

For a short summary:

```bash
pvs
```

For detailed information:

```bash
pvdisplay
```

---

# Create Volume Group

Create the Volume Group:

```bash
vgcreate myvg /dev/nvme0n2p1
```

Here:

```text
myvg
```

is the name of the Volume Group.

---

## Verify Volume Group

Use:

```bash
vgs
```

for a short summary.

Use:

```bash
vgdisplay
```

for detailed information.

`vgdisplay` is also useful for checking information such as PE size and available free extents.

---

# Create Logical Volume

There are two common ways to specify the LV size.

## Using `-L`

To create a 500 MB Logical Volume:

```bash
lvcreate -L +500M -n mylv myvg
```

Here:

```text
-L     → Specify size
-n     → Specify LV name
mylv   → Logical Volume name
myvg   → Volume Group
```

---

## Using `-l`

We can also specify the number of Physical Extents.

For example:

```bash
lvcreate -l 30 -n mylv myvg
```

This creates an LV using 30 allocation units.

---

# Verify Logical Volumes

Use:

```bash
lvs
```

for a summary.

For detailed information:

```bash
lvdisplay
```

The Logical Volume will be available through a path such as:

```text
/dev/myvg/mylv
```

---

# Format the Logical Volume

A Logical Volume is a block device.

Before using it for normal file storage, we need to create a filesystem.

For example, create an ext4 filesystem:

```bash
mkfs.ext4 /dev/myvg/mylv
```

---

# Mount the Logical Volume

Create a directory:

```bash
mkdir /lvm
```

Mount the LV:

```bash
mount /dev/myvg/mylv /lvm
```

Now the Logical Volume is available through:

```text
/lvm
```

---

# Verify the Filesystem

Use:

```bash
df -hT
```

This displays filesystem information along with the filesystem type.

---

# Extending a Logical Volume

One of the major advantages of LVM is that an LV can be expanded dynamically.

Suppose:

```text
LV = 500 MB
```

and later we need additional space.

We can add 300 MB:

```bash
lvresize -L +300M /dev/myvg/mylv
```

Another method is to allocate additional Physical Extents:

```bash
lvresize -l +30 /dev/myvg/mylv
```

`lvextend` can also be used for extending Logical Volumes.

---

# Filesystem Resize

Increasing the LV size does not automatically increase the filesystem size.

For an ext4 filesystem, use:

```bash
resize2fs /dev/myvg/mylv
```

Then verify:

```bash
df -h
```

The overall process is:

```text
Increase LV
    |
    v
Increase Filesystem
    |
    v
Verify with df -h
```

---

# Shrinking a Logical Volume

Shrinking is more risky than extending because incorrect steps can cause data corruption.

For ext4, the filesystem must be reduced before reducing the LV.

The basic sequence is:

```text
Unmount Filesystem
       |
       v
Reduce Filesystem
       |
       v
Reduce LV
```

First unmount the filesystem.

Then reduce the filesystem using:

```bash
resize2fs
```

After reducing the filesystem, reduce the LV.

For example:

```bash
lvresize -L -300M /dev/myvg/mylv
```

The important rule is:

```text
Filesystem first
LV second
```

Do not reduce the LV first.

---

# XFS and Shrinking

The session also highlights an important difference:

```text
ext4 → Can be reduced with the correct procedure
XFS  → Does not support shrinking
```

Therefore, shrinking must be planned carefully.

---

# Extending a Volume Group

Sometimes the Volume Group itself runs out of free space.

For example:

```text
VG Free Space = 0
```

Now we cannot create or extend an LV unless additional storage is added to the VG.

We can add a new disk or partition.

The workflow is:

```text
New Disk/Partition
       |
       v
Create PV
       |
       v
Add PV to Existing VG
       |
       v
More VG Free Space
       |
       v
Extend/Create LV
```

---

# Create the New Physical Volume

For example:

```bash
pvcreate /dev/nvme0n2p2
```

---

# Extend the Volume Group

Add the new PV to the existing VG:

```bash
vgextend myvg /dev/nvme0n2p2
```

Now the existing Volume Group has additional storage.

---

# Verify VG Expansion

Use:

```bash
vgs
```

or:

```bash
vgdisplay
```

The available space in the Volume Group should now be increased.

That free space can then be used to:

* Create a new Logical Volume
* Extend an existing Logical Volume

---

# Important Commands — Quick Revision

## Swap Commands

```bash
lsblk
```

Identify disks and partitions.

```bash
fdisk /dev/nvme0n2
```

Create and manage partitions.

```bash
mkswap /dev/nvme0n2p1
```

Create Swap signature.

```bash
swapon /dev/nvme0n2p1
```

Activate Swap.

```bash
swapoff /dev/nvme0n2p1
```

Deactivate Swap.

```bash
swapon -a
```

Activate all configured Swap.

```bash
swapoff -a
```

Deactivate all Swap.

```bash
blkid /dev/nvme0n2p1
```

Find UUID.

```bash
free -h
```

Check RAM and Swap usage.

```bash
wipefs /dev/nvme0n2p1
```

Remove filesystem signatures.

---

## Swappiness Commands

```bash
cat /proc/sys/vm/swappiness
```

Check current value.

```bash
sysctl vm.swappiness=70
```

Change the value temporarily.

```bash
vim /etc/sysctl.conf
```

Configure the value permanently.

Example:

```text
vm.swappiness = 80
```

Apply the configuration:

```bash
sysctl -p
```

---

## LVM Commands

### Physical Volume

```bash
pvcreate /dev/nvme0n2p1
```

```bash
pvs
```

```bash
pvdisplay
```

### Volume Group

```bash
vgcreate myvg /dev/nvme0n2p1
```

```bash
vgs
```

```bash
vgdisplay
```

### Logical Volume

```bash
lvcreate -L +500M -n mylv myvg
```

```bash
lvcreate -l 30 -n mylv myvg
```

```bash
lvs
```

```bash
lvdisplay
```

### Filesystem and Mount

```bash
mkfs.ext4 /dev/myvg/mylv
```

```bash
mkdir /lvm
```

```bash
mount /dev/myvg/mylv /lvm
```

```bash
df -hT
```

### Extend LV

```bash
lvresize -L +300M /dev/myvg/mylv
```

```bash
lvresize -l +30 /dev/myvg/mylv
```

```bash
resize2fs /dev/myvg/mylv
```

### Extend VG

```bash
pvcreate /dev/nvme0n2p2
```

```bash
vgextend myvg /dev/nvme0n2p2
```

---

# Final Revision

The most important concepts from this session are:

```text
Swap
  ↓
Virtual Memory
  ↓
RAM Pressure Management
```

For LVM:

```text
Physical Disk
     ↓
Physical Volume (PV)
     ↓
Volume Group (VG)
     ↓
Logical Volume (LV)
     ↓
Filesystem
     ↓
Mount Point
```

The most important relationship to remember is:

```text
PV → VG → LV
```

And for dynamic storage expansion:

```text
New Disk
   ↓
PV
   ↓
VG
   ↓
LV
   ↓
Filesystem
```

LVM provides flexibility because storage can be managed through logical layers instead of being tied directly to fixed physical partitions.
