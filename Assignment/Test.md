# Linux Hands-On Practice Tasks

---

## Task 1: Custom YUM / DNF Repository Configuration

Configure the YUM/DNF package repositories on your local system using the specified configurations. Ensure GPG checks are set appropriately and confirm repository activation.

| Repository ID | Name | Base Source Path | GPG Check | Enabled |
|---|---|---|---|---|
| `AppStream` | AppStream | `https://mirror.stream.centos.org/9-stream/AppStream/x86_64/os/` | Disabled / Configured | `True` |
| `BaseOS` | BaseOS | `https://mirror.stream.centos.org/9-stream/BaseOS/x86_64/os/` | Disabled / Configured | `True` |

## Task 2: Web Server Service Troubleshooting & Port Binding

The Apache HTTP server on your machine is configured to run on non-standard port `8080`. Currently, requests to the `httpd` service on port `8080` are failing.

**Hint:** Change the `Listen` directive in `/etc/httpd/conf/httpd.conf`.

Requirements:

- Ensure web content remains served from `/var/www/html`. **Do not modify this document root.**
- Update SELinux policy and firewall rules to permit traffic on port `8080`.
- Enable the `httpd` service to start automatically on system boot.
- **Verification:** Executing `curl http://localhost:8080` must successfully return the custom webpage response.

## Task 3: User Account Management & Group Membership

Provision local system groups and user accounts according to the operational requirements below.

| Username | Primary / Supplementary Group | Shell Access | Initial Password |
|---|---|---|---|
| `rahul` | Supplementary: `devops` | Interactive | `SecurePass2026` |
| `priya` | Supplementary: `devops` | Interactive | `SecurePass2026` |
| `service_user` | None (must **NOT** join `devops`) | `/sbin/nologin` | `SecurePass2026` |

**Note:** Create the primary user group `devops` prior to provisioning user accounts.

## Task 4: Scheduled Tasks (Cron Job - Daily Schedule)

Configure a persistent daily scheduled task for the account `rahul`.

- **Target account:** `rahul`
- **Schedule:** Daily at `16:45` (local system time)
- **Execution command:**
  ```bash
  echo "Daily Backup Initiative"
  ```

## Task 5: Scheduled Tasks (Cron Job - Recurring Interval)

Configure a high-frequency recurring background task for the user `rahul`.

- **Target account:** `rahul`
- **Schedule:** Every 5 minutes (`*/5 * * * *`)
- **Execution command:**
  ```bash
  /usr/bin/logger "RHCSA Practice Task Active"
  ```

## Task 6: Collaborative Directory Setup & Group Permissions

Establish a collaborative workspace at `/project/devops_share` with strict access controls.

- Set group ownership of `/project/devops_share` to `devops`.
- Grant full permissions (`read`, `write`, `execute`) exclusively to members of the `devops` group.
- Restrict unprivileged/other users from accessing or reading the directory.
- Apply the **SGID** bit on `/project/devops_share` to enforce group inheritance for all newly created items.

## Task 7: Time Synchronization (NTP Client Setup)

Ensure core system time tracking remains synchronized with network reference clocks.

- **NTP server address:** `0.in.pool.ntp.org`
- Configure `chronyd` (NTP client service) to synchronize system time against external time sources.
- Ensure the `chronyd` service is enabled to start on system boot and is actively running.

## Task 8: File Location & Extraction

Locate and centralize user-owned system assets across the filesystem hierarchy.

- Search the entire filesystem for all files owned by user `simone`. **Create the user if it does not exist.**
- Copy all matching files to `/root/recovered_files/` (provision the directory if absent).

## Task 9: String Searching & Filtering

Perform string parsing against local system dictionaries.

- Search for occurrences of the exact string `"linux"` within `/usr/share/dict/words`.
- Export all matching lines to `/root/filtered_words.txt`.

## Task 10: On-Demand Automatic Mounts (autofs)

**Note:** Set up the first NFS server in your LAN VM, then solve the part below.

Implement dynamic filesystem mounting using the indirect map capabilities of `autofs`.

- **Remote NFS export:** `/home/remoteuser`
- **Target mount path:** Configure `autofs` to dynamically mount access attempts under `/home/remoteuser`.
- Enable and start the `autofs` daemon to ensure persistent operations across reboots.
- Mounting should **not** be permanent in `/etc/fstab`.

## Task 11: System Search Scripting (SGID File Audit)

Develop an automated audit script to identify specific privileged executable files.

- **Script location:** `/usr/local/bin/find_sgid.sh`
- **Criteria:** Search inside `/usr` for files that are smaller than **15 MiB** **AND** have the **SGID** bit set.
- **Output destination:** Write results directly to `/root/sgid_output.txt`.
- Ensure executable permissions are granted (`chmod +x`) and test execution.

## Task 12: System Search Scripting (SUID File Audit)

Develop an automated system script to track setuid binaries across system trees.

- **Script location:** `/usr/local/bin/find_suid.sh`
- **Criteria:** Search inside `/usr` for files larger than **3 MiB** but smaller than **12 MiB** with the **SUID** bit set.
- **Output destination:** Output results directly to `/root/suid_output.txt`.
- Grant executable permissions and verify script output generation.

## Task 13: User Account Creation with Fixed UID

Provision a local user account configured with specific system identifiers.

- **Username:** `vikram`
- **UID:** `3050`
- **Password:** `Welcome2026!`
- Confirm exact UID mapping via system identity commands.

## Task 14: Privilege Escalation (Sudoers Configuration)

Delegate system administration capabilities using standard administrative access mechanisms.

- Ensure the system group `sysadmin_team` exists.
- Grant all members of `sysadmin_team` full administrative privilege execution via `sudo` without password prompting (`NOPASSWD`).
- For checking, create some user and make one of them a group member; then check whether the permissions are applied to that user or not.

## Task 15: Default User File Creation Mask (Umask Configuration)

Restrict dynamic default permissions for created objects owned by specific accounts.

Configure user environment rules for account `amit`:

- **Default file creation permission mode:** `r--r--r--` (`mode 0444`)
- **Default directory creation permission mode:** `r-xr-xr-x` (`mode 0555`)

Validate configuration settings by logging in as `amit` and testing creation modes.

## Task 16: System Backup & Archiving

Generate a compressed system configuration archive for recovery purposes.

- **Source path:** `/etc`
- **Destination path:** `/var/backups/`
- **Archive filename:** `etc_backup.tar.xz` (using `xz` compression)
- Verify archive creation and validate payload integrity.

## Task 17: Application Binary Alias / Wrapper Configuration

Configure custom environment execution wrappers for targeted user shell environments.

- Configure system execution hooks so when user `karan` runs `custom_app`, the system outputs:
  ```text
  Linux RHCSA v10 Hands-on Practice Session
  ```
- Ensure the configuration is stored persistently in the user profile environment.

## Task 18: User-Specific Flatpak Repository & Package Management

Deploy user-isolated containerized software packages via Flatpak.

- Register a user-level Flatpak repository named `flathub_user`, scoped exclusively to user `student_user` only. **Do not configure it system-wide for all users.**
- Install application package `Tetzle` within the user environment context.
- Confirm system-wide environments remain isolated from this deployment.

## Task 19: Root Password Recovery & Rescue Boot Procedure

Gain emergency administrative access and update credentials on Node-2.

- Interrupt standard boot sequences (GRUB) to access `emergency.target` or `rescue.target`.
- Remount root read-write (`/sysroot`) and update the root administrative password to `RedHatPass2026`.
- Create `/.autorelabel` to execute full SELinux security context relabeling during initialization.
- Perform system reboot and confirm credential updates.

## Task 20: User Account Aging & Security Password Policy

Enforce baseline account security policies across local user lifecycles.

- Modify `/etc/login.defs` so maximum password validity (`MAXDAYS`) defaults to `30` days.
- Create verification account `test_user_30` and verify password lifecycle fields using password aging utilities.

## Task 21: Performance Tuning with Tuned Profile

Optimize Node-2 runtime parameters based on operational workload requirements.

- Determine the system-recommended tuning profile using `tuned-adm`.
- Apply the selected system tuning profile.
- Verify current profile state and operational metrics.

## Task 22: Disk Partitioning, Formatting, & Swap Allocation

Provision virtual memory resources on unallocated physical block devices.

- Identify an unallocated block storage disk.
- Allocate a primary partition sized at **500 MB**.
- Format the target block volume as swap storage (`mkswap`).
- Configure system auto-mount definitions in `/etc/fstab` using the device **UUID**.
- Activate swap space (`swapon -a`) and verify overall system swap usage.

## Task 23: Complete LVM Setup (PV, VG, LV, Filesystem & Persistent Mount)

Build end-to-end Logical Volume Management (LVM) structures according to the parameter layout below.

| Stage | Target / Identifier | Configuration Specifications |
|---|---|---|
| **Step 1** | Physical Volume | Initialize raw partition / unallocated storage disk |
| **Step 2** | Volume Group | `vg_data` with Physical Extent (PE) size of **8 MiB** |
| **Step 3** | Logical Volume | `lv_store` sized at **120 extents** inside `vg_data` |
| **Step 4** | Filesystem Format | `ext4` filesystem layout |
| **Step 5** | Persistent Mount | Mount on `/mnt/data_store` via `/etc/fstab` using UUID |

**Verification:** Verify mounting status and storage allocations using `df -hT`.

## Task 24: LVM Logical Volume & Filesystem Extension

Dynamically expand system capacity without disrupting operational status.

- **Target logical storage volume:** `/dev/vg_data/lv_store`
- Expand physical logical volume capacity and resize the underlying filesystem to achieve a target size of **200 MiB** (acceptable range: **200 MiB–208 MiB**).
- Perform online extension to safeguard existing system data.
- Validate increased capacity metrics using standard storage diagnostic utilities.
