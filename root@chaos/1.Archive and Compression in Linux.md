# RHCSA — Archive & Compression

### **1. Archive vs Compression**

| Archive | Compression |
| --- | --- |
| Combines multiple files into one | Reduces file size |
| Tool: `tar` | Tools: `gzip`, `bzip2`, `xz` |
| Extension: `.tar` | Extensions: `.gz`, `.bz2`, `.xz` |

**Example:**

- Archive: `file1 + file2 → backup.tar`
- Compression: `backup.tar → backup.tar.gz`

**Remember:** `tar` combines files; compression reduces their size.

### **2. TAR — Most Important**

`tar` stands for **Tape Archive**.

**Important options:**

| Option | Meaning |
| --- | --- |
| `-c` | Create archive |
| `-x` | Extract archive |
| `-t` | List archive contents |
| `-a` | Automatically select compression based on extension |
| `-v` | Verbose (show filenames) |
| `-f` | Specify archive filename |
| `-C` | Change directory before processing files |
| `-z` | gzip compression (optional alternative) |
| `-j` | bzip2 compression (optional alternative) |

**Essential commands:**

**Create an archive (automatically selects compression):**

```
tar -caf backup.tar.gz /home/student
tar -caf backup.tar.bz2 /home/student
tar -caf backup.tar.xz /home/student
tar -caf backup.tar /home/student
```

- `a` automatically selects compression based on the archive extension.

| Extension | Compression |
| --- | --- |
| `.tar` | None |
| `.tar.gz` | gzip |
| `.tar.bz2` | bzip2 |
| `.tar.xz` | xz |

**Extract archives (auto-detect compression):**

```bash
tar -xaf backup.tar.gz  # tar -xf is also sufficient
tar -xaf backup.tar.bz2 # tar -xf is also sufficient
tar -xaf backup.tar.xz  # tar -xf is also sufficient
tar -xaf backup.tar     # tar -xf is also sufficient
```

No need to specify `-z`, `-j`, or `-a`.

**List contents without extracting:**

```
tar -tf backup.tar.gz
```

**Extract to a specific directory:**

```
mkdir -p /tmp/restore
tar -xaf backup.tar.gz -C /tmp/restore
```

**Create an archive using `-C`:**

```
tar -caf /root/etc-backup.tar.gz -C / etc
```

This archives `/etc` as `etc/`, using a relative path.

**Important:**

- `C` changes the working directory for the specified archive operation.
- The directory supplied to `C` must already exist.
- `v` is optional; use it to display filenames.
- When archiving absolute paths, tar normally removes the leading `/` for safety.

### **3. gzip and gunzip**

Used to compress and decompress individual files.

| Command | Purpose |
| --- | --- |
| `gzip file.txt` | Compress → `file.txt.gz` |
| `gunzip file.txt.gz` | Decompress |
| `gzip -k file.txt` | Compress and keep original |

**Important:** `gzip` normally removes the original file after compression. Use `-k` to keep it.

### **4. bzip2 and bunzip2**

Same concept as gzip, using `.bz2`.

| Command | Purpose |
| --- | --- |
| `bzip2 file.txt` | Compress → `file.txt.bz2` |
| `bunzip2 file.txt.bz2` | Decompress |
| `bzip2 -k file.txt` | Compress and keep original |

**Note:** `bzip2` often provides better compression than gzip but can be slower.

### **5. Quick Revision Table**

| Task | Command |
| --- | --- |
| Create `.tar` | `tar -caf a.tar dir/` |
| Create `.tar.gz` | `tar -caf a.tar.gz dir/` |
| Create `.tar.bz2` | `tar -caf a.tar.bz2 dir/` |
| Create `.tar.xz` | `tar -caf a.tar.xz dir/` |
| Extract any tar archive | `tar -xf archive.tar.gz` |
| List contents | `tar -tf archive.tar.gz` |
| Extract elsewhere | `tar -xf a.tar.gz -C /tmp/` |
| Compress one file | `gzip file.txt` |
| Decompress one file | `gunzip file.txt.gz` |

**Remember:**

- Creation → `tar -caf`
- Extraction → `tar -xf`
- Listing → `tar -tf`

### **6. Practical RHCSA Exam Tasks**

**Q1. Create a gzip-compressed archive of `/etc` named `/root/etc-backup.tar.gz`.**

```bash
tar -caf /root/etc-backup.tar.gz  / etc
```

**Q2. Create a bzip2-compressed archive of `/var/log` named `/root/logs.tar.bz2`.**

```bash
tar -caf /root/logs.tar.bz2  /var/log
```

**Q3. Extract `etc-backup.tar.gz` into `/tmp/restore`.**

```
mkdir -p /tmp/restore
tar -xf /root/etc-backup.tar.gz -C /tmp/restore
```

**Q4. Verify the files inside `etc-backup.tar.gz` without extraction.**

```
tar -tf /root/etc-backup.tar.gz
```

**Q5. Compress `report.txt` using gzip, keeping the original file.**

```
gzip -k report.txt
```

### **7. Important Exam Reminders**

- `tar -cf` creates an archive without compression.
- `tar -caf` selects compression from the archive extension.
- `tar -xf` automatically detects supported compression formats.
- `tar -tf` lists archive contents without extracting.
- `f` specifies the archive filename (place it last in a combined option group).
- `C` requires an existing directory.
- `gzip` and `bzip2` compress individual files, not directories directly.
- `z` = gzip and `j` = bzip2 (know these alternatives).
- Do not use `tar -r` to append to compressed archives.

### **Final Memorization Trick**

| Command | Remember |
| --- | --- |
| `tar -caf` | Create + Auto-compress + File |
| `tar -xf` | Extract + File |
| `tar -tf` | List + File |
| `tar -xf ... -C dir` | Extract into directory |

**RHCSA Strategy:** Master `-caf`, `-xf`, `-tf`, and `-C`. Understand `-z` and `-j`, but there's no need to use them every time.
