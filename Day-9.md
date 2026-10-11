
---

# Day-9 Linux Permission Management

## Permanent UMASK Configuration

`umask` defines the **default permissions** assigned to newly created files and directories.

* Default maximum permissions:

  * Files → `666`
  * Directories → `777`

The umask value is subtracted from these defaults.

Example:

```bash
umask 022
```

Result:

* New File → `644`
* New Directory → `755`

---

## Making Umask Permanent

There are multiple places where umask can be configured depending on the Linux distribution.

Common locations:

```bash
/etc/profile
```

or

```bash
/etc/bashrc
```

or

```bash
/etc/login.defs
```

For an individual user:

```bash
~/.bashrc
```

or

```bash
~/.profile
```

To configure globally using `/etc/profile`

Open the file:

```bash
vi /etc/profile
```

Add the following at the bottom:

```bash
umask 022
```

Save the file.

Reload without logging out:

```bash
source /etc/profile
```

or

```bash
. /etc/profile
```

---

## Important Notes

Adding the configuration at the end of the file is generally safer because it avoids accidentally breaking existing shell logic.

However, the following statement is **incorrect**:

> "Even if you remove the entry later, reboot is required."

The actual behavior is:

* Existing shells continue using the old umask.
* New login sessions read the updated configuration.
* A reboot is **not required**.
* Logging out and logging back in is usually sufficient.

You can also manually change the current shell using:

```bash
umask 022
```

---

# Access Control List (ACL)

## Why ACL?

Traditional Linux permissions provide permissions for only three categories:

* Owner
* Group
* Others

Suppose

```
Owner : root
Group : developers
```

Now another user

```
nikhil
```

needs access.

Without ACL, you have only two options:

* Change owner
* Change group
* Give permissions to Others

All are undesirable.

ACL solves this problem.

It allows assigning permissions to **specific users or groups** without affecting existing permissions.

---

## ACL Commands

View ACL

```bash
getfacl filename
```

Set ACL

```bash
setfacl
```

---

# Give Permission to a User

Syntax

```bash
setfacl -m u:username:rwx filename
```

Example

```bash
setfacl -m u:nikhil:rwx project.txt
```

Meaning

```
-m
Modify ACL

u
User

nikhil
Username

rwx
Permissions
```

---

# Give Permission to a Group

```bash
setfacl -m g:devteam:rwx project.txt
```

---

# Check ACL

```bash
getfacl project.txt
```

Example output

```text
# file: project.txt
# owner: root
# group: root

user::rw-
user:nikhil:rwx
group::r--
mask::rwx
other::r--
```

---

# ACL in ls Output

When ACL exists,

```bash
ls -l
```

shows

```text
-rw-rwxr--+
```

Notice the

```
+
```

The plus sign means

> Additional ACL entries exist.

---

# Modify ACL

Simply run the same command again.

Example

```bash
setfacl -m u:nikhil:rw- project.txt
```

---

# Remove Specific ACL

```bash
setfacl -x u:nikhil project.txt
```

Remove group ACL

```bash
setfacl -x g:devteam project.txt
```

---

# Remove All ACL Entries

```bash
setfacl -b project.txt
```

---

# Recursive ACL

Apply ACL to an entire directory tree.

```bash
setfacl -R -m u:nikhil:rwx project/
```

`-R`

means

```
Recursive
```

Every file and subdirectory receives the ACL.

---

# Default ACL on Directories (Very Important)

ACL also supports **default ACLs**.

Example

```bash
setfacl -d -m u:nikhil:rwx project/
```

Now every newly created file inside the directory automatically receives that ACL.

This is different from recursive ACL.

* Recursive → Changes existing files.
* Default ACL → Affects future files.

---

# Special Permissions

Linux has three special permissions.

```
SUID
SGID
Sticky Bit
```

---

# SUID (Set User ID)

## Purpose

SUID allows a program to execute with the permissions of the **file owner**, regardless of who runs it.

Most commonly:

```
Owner = root
```

Then any user executing the program temporarily gets root's privileges **for that program only**.

---

## Apply SUID

```bash
chmod u+s filename
```

Example

```bash
chmod u+s myscript
```

---

## Remove

```bash
chmod u-s filename
```

---

## Numeric Mode

```bash
chmod 4755 filename
```

```
4
SUID
```

---

## Display

Example

```text
-rwsr-xr-x
```

Owner execute position

```
s
```

---

### Small s

```
Execute permission exists
```

Example

```text
-rwsr-xr-x
```

---

### Capital S

```
Execute permission missing
```

Example

```text
-rwSr-xr-x
```

---

## Real Example

```bash
passwd
```

Every user changes their own password.

But

```
/etc/shadow
```

belongs to root.

Why does it work?

Because

```
passwd
```

has SUID.

Check

```bash
ls -l /usr/bin/passwd
```

You will usually see

```text
-rwsr-xr-x
```

> **Correction:** The earlier `useradd` example is not appropriate as a normal use case. While `useradd` is an administrative program, ordinary users still cannot create accounts simply because it has SUID. The `passwd` command is the classic and correct example of SUID.

---

# SGID (Set Group ID)

Command

```bash
chmod g+s filename
```

---

## On Files

Program executes with the permissions of the file's group.

---

## On Directories (Most Common Use)

Suppose

```
Project Directory

Owner : root

Group : developers
```

Enable SGID

```bash
chmod g+s project/
```

Now

```
Alice creates file
```

Group becomes

```
developers
```

Bob creates file

Group also becomes

```
developers
```

Everyone shares the same group automatically.

Very useful for

* Team projects
* Shared directories
* Development environments

---

## Numeric

```bash
chmod 2755 project
```

```
2
SGID
```

---

## Display

```text
drwxrwsr-x
```

Group execute position

```
s
```

---

# Sticky Bit

Purpose

Prevent users from deleting other users' files in a shared directory.

---

## Apply

```bash
chmod +t shared
```

or

```bash
chmod o+t shared
```

---

## Remove

```bash
chmod -t shared
```

---

## Numeric

```bash
chmod 1777 shared
```

```
1
Sticky Bit
```

---

## Display

```text
drwxrwxrwt
```

---

## Real Example

```bash
/tmp
```

Check

```bash
ls -ld /tmp
```

Output

```text
drwxrwxrwt
```

Everyone can create files.

But only

* Root
* File owner
* Directory owner

can delete or rename a file.

---

## Small t

Execute permission exists.

Example

```text
drwxrwxrwt
```

---

## Capital T

Execute permission missing.

Example

```text
drwxrwxrwT
```

---

## Important Clarification

The statement:

> "If you run `chmod 777` after setting Sticky Bit, the Sticky Bit is removed."

is **not universally true**.

* `chmod 777 directory` sets permissions exactly to `0777`, which **does remove** the Sticky Bit because the special bit isn't included.
* `chmod 1777 directory` preserves it.
* `chmod +t directory` adds it back without changing the normal permissions.

So, it's not that `777` has special behavior—it simply overwrites the mode and excludes all special permission bits.

---

# Numeric Values of Special Permissions

| Special Permission | Numeric Value | Example                |
| ------------------ | ------------: | ---------------------- |
| Sticky Bit         |             1 | `chmod 1777 directory` |
| SGID               |             2 | `chmod 2755 directory` |
| SUID               |             4 | `chmod 4755 file`      |

## Combined Example

```bash
chmod 6755 filename
```

Here:

* `6 = 4 + 2`
* SUID + SGID are both enabled.
