# Flatpak Application Management — Linux Practical Guide

## 1. What is Flatpak?

**Flatpak** is a Linux technology used to install and run applications.

It is commonly used for **GUI/Desktop applications** such as:

* GIMP
* VSCodium
* VLC
* LibreOffice
* Firefox

Flatpak provides applications in a way that can keep the application and its required runtime/components more independent from the host operating system.

### Simple understanding

```text
Traditional Linux package

DNF
 ↓
RPM Package
 ↓
System
```

Flatpak:

```text
Flatpak
 ↓
Flatpak Remote
 ↓
Application
 ↓
Runtime
 ↓
Application runs
```

> **Important:** Flatpak does not replace DNF/YUM. Both have different purposes.

---

# 2. DNF vs Flatpak

| DNF/YUM                                  | Flatpak                                          |
| ---------------------------------------- | ------------------------------------------------ |
| Manages RPM packages                     | Manages Flatpak applications                     |
| Commonly used for system/server software | Commonly used for desktop applications           |
| Uses RPM repositories                    | Uses Flatpak remotes                             |
| Example: `dnf install httpd`             | Example: `flatpak install flathub org.gimp.GIMP` |
| Packages are integrated with the OS      | Applications can run in a sandbox                |

For example:

### Install Apache using DNF

```bash
dnf install httpd
```

### Install GIMP using Flatpak

```bash
flatpak install flathub org.gimp.GIMP
```

So don't think:

> Flatpak replaces DNF.

Think:

> **DNF and Flatpak are two different application/package management methods.**

---

# 3. What is a Flatpak Remote?

In Flatpak terminology, a repository is generally called a **remote**.

A remote is a location from which Flatpak can obtain applications and runtimes.

The most commonly used remote is:

```text
Flathub
```

For example:

```text
Flathub
   |
   +---- GIMP
   +---- VLC
   +---- VSCodium
   +---- LibreOffice
   +---- Firefox
   +---- Many more applications
```

---

# 4. Flathub

**Flathub** is a large and popular Flatpak application repository.

Before installing applications from Flathub, we normally add the Flathub remote.

```bash
flatpak remote-add --if-not-exists flathub https://dl.flathub.org/repo/flathub.flatpakrepo
```

Here:

```text
flatpak
    ↓
remote-add
    ↓
flathub
    ↓
Repository URL
```

`flathub` is the **local name** we give to the remote.

---

# 5. Check Configured Remotes

After adding Flathub:

```bash
flatpak remotes
```

Example:

```text
Name
flathub
```

For more information:

```bash
flatpak remotes --show-details
```

This can show details such as the remote URL.

---

# 6. Search for an Application

Suppose we want GIMP.

Use:

```bash
flatpak search gimp
```

Or:

```bash
flatpak search codium
```

The search result gives us the application's **Application ID**.

For example:

```text
com.vscodium.codium
```

### Important

There are two things to remember:

```text
Application Name
       ↓
VSCodium
```

and:

```text
Application ID
       ↓
com.vscodium.codium
```

When installing, we normally use the **Application ID**.

---

# 7. See All Applications Available in a Remote

If you want to see applications available in Flathub:

```bash
flatpak remote-ls --app flathub
```

This can produce a very large list.

To search inside that list:

```bash
flatpak remote-ls --app flathub | grep -i codium
```

For QR-related applications:

```bash
flatpak remote-ls --app flathub | grep -i qr
```

### Difference

```bash
flatpak search gimp
```

means:

> Search for GIMP using Flatpak's search mechanism.

Whereas:

```bash
flatpak remote-ls --app flathub
```

means:

> List applications available in the Flathub remote.

---

# 8. Install an Application

General syntax:

```bash
flatpak install <remote> <application-id>
```

Example:

```bash
flatpak install flathub org.gimp.GIMP
```

Automatically answer `yes`:

```bash
flatpak install -y flathub org.gimp.GIMP
```

### Understand the command

```text
flatpak
   ↓
install
   ↓
flathub
   ↓
org.gimp.GIMP
```

Meaning:

> Install the application `org.gimp.GIMP` from the `flathub` remote.

---

# 9. Verify Installed Applications

After installation:

```bash
flatpak list
```

If you want to see only applications:

```bash
flatpak list --app
```

For user-installed applications:

```bash
flatpak list --user
```

Only user applications:

```bash
flatpak list --user --app
```

---

# 10. Get Application Information

Use:

```bash
flatpak info <application-id>
```

Example:

```bash
flatpak info org.gimp.GIMP
```

For a user installation:

```bash
flatpak info --user com.vscodium.codium
```

This gives information such as:

* Application ID
* Version
* Branch
* Installation location
* Runtime
* Origin/remote

---

# 11. Run an Application

General syntax:

```bash
flatpak run <application-id>
```

Example:

```bash
flatpak run org.gimp.GIMP
```

For VSCodium:

```bash
flatpak run com.vscodium.codium
```

So the basic flow becomes:

```text
Search
  ↓
Get Application ID
  ↓
Install
  ↓
Run
```

Example:

```bash
flatpak search gimp

flatpak install flathub org.gimp.GIMP

flatpak run org.gimp.GIMP
```

---

# 12. Update Applications

To update installed Flatpak applications and runtimes:

```bash
flatpak update
```

Automatically answer yes:

```bash
flatpak update -y
```

For a specific user installation:

```bash
flatpak update --user
```

---

# 13. Remove an Application

General syntax:

```bash
flatpak uninstall <application-id>
```

Example:

```bash
flatpak uninstall org.gimp.GIMP
```

Automatically answer yes:

```bash
flatpak uninstall -y org.gimp.GIMP
```

For a user installation:

```bash
flatpak uninstall --user com.vscodium.codium
```

---

# 14. Remove Unused Components

Sometimes an application is removed but some runtimes are no longer required.

You can clean up unused components:

```bash
flatpak uninstall --unused
```

This is useful for cleanup.

---

# 15. Repair Flatpak

If Flatpak reports repository/object problems, use:

```bash
flatpak repair
```

For a per-user installation:

```bash
flatpak repair --user
```

This is a useful troubleshooting command.

---

# 16. AppStream Metadata

This is important when troubleshooting `flatpak search`.

Flatpak uses **AppStream metadata** to provide application information used by search and software-center-style interfaces.

You can refresh the AppStream metadata using:

```bash
flatpak update --appstream
```

---

# 17. Troubleshooting `flatpak search`

Suppose you run:

```bash
flatpak search QR-ScanGen
```

and receive:

```text
Failed to parse /var/lib/flatpak/appstream/flathub/x86_64/active/appstream.xml.gz
```

This tells us there is a problem with the **local AppStream metadata**.

It does **not necessarily mean that Flathub itself is unavailable**.

That's why this can happen:

```bash
flatpak search QR-ScanGen
```

❌ fails

while:

```bash
flatpak remote-ls --app flathub
```

✅ works.

The two commands use different information.

---

# 18. First Solution for Search Problem

First refresh AppStream metadata:

```bash
flatpak update --appstream
```

Then try:

```bash
flatpak search QR-ScanGen
```

---

# 19. Second Solution — Repair Flatpak

If the problem continues:

```bash
flatpak repair
```

Then:

```bash
flatpak update --appstream
```

Try again:

```bash
flatpak search QR-ScanGen
```

---

# 20. Third Solution — Check Flatpak Version

Check your version:

```bash
flatpak --version
```

If you are working on an older Linux system, an old Flatpak/AppStream parser can sometimes cause metadata parsing problems.

Make sure Flatpak is properly updated through your operating system's package management.

For example:

```bash
dnf update flatpak
```

Then:

```bash
flatpak --version
```

---

# 21. Alternative Search Method

If `flatpak search` is not working, you can directly list the applications available from the remote:

```bash
flatpak remote-ls --app flathub
```

Then use `grep`:

```bash
flatpak remote-ls --app flathub | grep -i qr
```

This is a very useful practical troubleshooting technique.

---

# 22. Other Flatpak Remotes

Flathub is not the only Flatpak remote.

Flatpak can work with multiple remotes.

For example:

```text
System
  |
  +---- flathub
  |
  +---- gnome
  |
  +---- another-remote
```

A project or organization can provide its own Flatpak remote.

Usually, you find the remote URL in the **official documentation of that project**.

The important command is:

```bash
flatpak remote-add <remote-name> <URL>
```

For example:

```bash
flatpak remote-add gnome https://sdk.gnome.org/gnome.flatpakrepo
```

Here:

```text
gnome
```

is the local name assigned to that remote.

---

# 23. Flatpak Repository vs YUM Repository

This is a useful comparison for students.

### YUM/DNF

You may already know:

```text
/etc/yum.repos.d/
       |
       +---- rhel.repo
       +---- appstream.repo
       +---- custom.repo
```

YUM/DNF uses `.repo` configuration files.

### Flatpak

Flatpak does **not** normally use:

```text
/etc/flatpak.repos.d/
```

Instead, Flatpak maintains its own repository and installation data.

---

# 24. Where Does Flatpak Store Its Data?

## System-wide

The normal system-wide Flatpak location is:

```text
/var/lib/flatpak/
```

You may see directories such as:

```text
/var/lib/flatpak/
├── app/
├── runtime/
└── repo/
```

---

## Per-user

For a user installation:

```text
~/.local/share/flatpak/
```

For user `bammbamm`:

```text
/home/bammbamm/.local/share/flatpak/
```

You may see:

```text
~/.local/share/flatpak/
├── app/
├── runtime/
└── repo/
```

---

# 25. Remote Configuration

For a system-wide Flatpak installation, remote configuration is maintained under the Flatpak repository data, commonly:

```text
/var/lib/flatpak/repo/config
```

For a per-user installation:

```text
~/.local/share/flatpak/repo/config
```

### Important

Don't compare this directly with:

```text
/etc/yum.repos.d/*.repo
```

They are **not the same implementation**.

The concept is similar:

```text
YUM/DNF
Repository configuration
        ↓
/etc/yum.repos.d/*.repo
```

Flatpak:

```text
Flatpak remote configuration
        ↓
Flatpak repository data
```

---

# 26. System-wide vs Per-user

This is one of the most important Flatpak concepts.

Flatpak can work at two levels.

## System-wide

```bash
flatpak install flathub org.gimp.GIMP
```

This uses the system installation.

Conceptually:

```text
System
   |
   +--- User A → Application available
   |
   +--- User B → Application available
   |
   +--- User C → Application available
```

---

## Per-user

Use:

```bash
flatpak install --user flathub org.gimp.GIMP
```

Conceptually:

```text
System
   |
   +--- User A → Application installed
   |
   +--- User B → No user installation
   |
   +--- User C → No user installation
```

The important option is:

```text
--user
```

Remember:

> `--user` means perform the operation for the current user's Flatpak installation.

---

# 27. Per-user Remote

Suppose the requirement says:

> Configure a Flatpak remote called `extra` only for user `bammbamm`.

First switch to the user:

```bash
su - bammbamm
```

Now add the remote:

```bash
flatpak remote-add --user --if-not-exists extra https://dl.flathub.org/repo/flathub.flatpakrepo
```

Check it:

```bash
flatpak remotes --user
```

You should see:

```text
extra
```

---

# 28. Install an Application for `bammbamm`

Search:

```bash
flatpak search codium
```

Suppose the Application ID is:

```text
com.vscodium.codium
```

Install:

```bash
flatpak install --user extra com.vscodium.codium -y
```

Verify:

```bash
flatpak list --user
```

Run:

```bash
flatpak run com.vscodium.codium
```

---

# 29. Verify User Isolation

Exit from `bammbamm`:

```bash
exit
```

Now check the current user's Flatpak installation:

```bash
flatpak list --user
```

The VSCodium installation belonging to `bammbamm` should not appear as the current user's per-user installation.

Likewise:

```bash
flatpak remotes --user
```

The `extra` remote configured specifically for `bammbamm` should not appear in another user's per-user remotes.

### Important clarification

This does **not** mean the application is magically invisible to the entire operating system.

It means:

> The Flatpak application and remote were configured in `bammbamm`'s **per-user Flatpak installation**.

---

# 30. Complete Practical Assignment

## Question

Configure a Flatpak remote repository named `extra` that is available only to the user `bammbamm`.

Repository:

```text
https://dl.flathub.org/repo/flathub.flatpakrepo
```

Using the `extra` repository, install **VSCodium/Codium** for `bammbamm`.

Finally, verify that the remote and application are available in `bammbamm`'s user installation and are not present in another user's per-user installation.

---

## Solution

### Step 1 — Switch to the user

```bash
su - bammbamm
```

### Step 2 — Add the remote

```bash
flatpak remote-add --user --if-not-exists extra https://dl.flathub.org/repo/flathub.flatpakrepo
```

### Step 3 — Verify remote

```bash
flatpak remotes --user
```

Expected:

```text
extra
```

### Step 4 — Search

```bash
flatpak search codium
```

### Step 5 — Install

```bash
flatpak install --user extra com.vscodium.codium -y
```

### Step 6 — Verify installation

```bash
flatpak list --user
```

### Step 7 — Run

```bash
flatpak run com.vscodium.codium
```

### Step 8 — Exit

```bash
exit
```

### Step 9 — Verify from another user

```bash
flatpak remotes --user
```

and:

```bash
flatpak list --user
```

The `extra` remote and VSCodium should not appear as **that other user's** per-user installation.

---

# 31. Complete Command Reference

## Repository / Remote

### Add remote

```bash
flatpak remote-add <name> <URL>
```

### Add remote for current user

```bash
flatpak remote-add --user <name> <URL>
```

### List remotes

```bash
flatpak remotes
```

### List user remotes

```bash
flatpak remotes --user
```

### Show remote details

```bash
flatpak remotes --show-details
```

### Remove remote

```bash
flatpak remote-delete <name>
```

### Remove user remote

```bash
flatpak remote-delete --user <name>
```

---

# 32. Search and Discovery

### Search application

```bash
flatpak search <keyword>
```

### List applications in remote

```bash
flatpak remote-ls --app flathub
```

### Search remote output

```bash
flatpak remote-ls --app flathub | grep -i <keyword>
```

### Show remote application information

```bash
flatpak remote-info flathub <application-id>
```

---

# 33. Installation

### Install

```bash
flatpak install <remote> <application-id>
```

### Install automatically

```bash
flatpak install -y <remote> <application-id>
```

### Install for current user

```bash
flatpak install --user <remote> <application-id>
```

---

# 34. Application Management

### List installed applications

```bash
flatpak list --app
```

### List user applications

```bash
flatpak list --user --app
```

### Application information

```bash
flatpak info <application-id>
```

### Run application

```bash
flatpak run <application-id>
```

### Remove application

```bash
flatpak uninstall <application-id>
```

### Remove user application

```bash
flatpak uninstall --user <application-id>
```

---

# 35. Update and Maintenance

### Update everything

```bash
flatpak update
```

### Update user installation

```bash
flatpak update --user
```

### Refresh AppStream metadata

```bash
flatpak update --appstream
```

### Remove unused components

```bash
flatpak uninstall --unused
```

### Repair Flatpak

```bash
flatpak repair
```

### Repair user installation

```bash
flatpak repair --user
```

---

# 36. The 10 Commands Students Should Remember

Don't try to memorize every Flatpak command initially.

Start with these:

```bash
flatpak --version
```

```bash
flatpak remotes
```

```bash
flatpak search <keyword>
```

```bash
flatpak remote-ls --app flathub
```

```bash
flatpak install flathub <application-id>
```

```bash
flatpak list --app
```

```bash
flatpak info <application-id>
```

```bash
flatpak run <application-id>
```

```bash
flatpak update
```

```bash
flatpak uninstall <application-id>
```

And remember one special option:

```text
--user
```

which is used when you want to work with the **current user's Flatpak installation**.

---

# 37. Easy Memory Map

```text
                 FLATPAK
                    |
                    |
          +---------+---------+
          |                   |
       REMOTE              APPLICATION
          |                   |
       flathub                |
          |                   |
          |             Application ID
          |                   |
          |          +--------+--------+
          |          |        |        |
        search     install   run     remove
          |          |        |        |
          +----------+--------+--------+
                     |
                   update
```

For per-user work:

```text
                    --user
                      |
                      ↓
             Current User Only
```

---

# 38. Practical Workflow to Remember

When you get a Flatpak task in an exam or real environment, think in this order:

```text
1. Is Flatpak installed?
        ↓
2. Is the required remote configured?
        ↓
3. Search the application
        ↓
4. Find Application ID
        ↓
5. Install application
        ↓
6. Verify installation
        ↓
7. Run application
        ↓
8. Update when required
        ↓
9. Remove when required
```

Example:

```bash
flatpak --version

flatpak remotes

flatpak search gimp

flatpak install flathub org.gimp.GIMP

flatpak list --app

flatpak run org.gimp.GIMP

flatpak update

flatpak uninstall org.gimp.GIMP
```

