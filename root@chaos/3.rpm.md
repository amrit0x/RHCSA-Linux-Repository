## RPM Package Management :

**Essential commands :**

| Command | Purpose |
| --- | --- |
| **`rpm -qa`** | List all installed RPM packages |
| **`rpm -q httpd`** | Check whether httpd is installed |
| **`rpm -qi httpd`** | Show installed package information |
| **`rpm -ql httpd`** | List files installed by the package |
| **`rpm -qf /usr/bin/ls`** | Find which package owns a file |
| **`rpm -ivh file.rpm`** | Install an RPM |
| **`rpm -Uvh file.rpm`** | Install or upgrade an RPM |
| **`rpm -e httpd`** | Remove an installed package |
- `h` → Display progress marks (for human understanding)
- `e` → Erase/remove
- `q` → Query
- `a` → All installed packages

### Practical example

```
rpm -q httpd
rpm -qi httpd
rpm -ql httpd
rpm -qf /usr/bin/ls
```

Exam tip: Prefer `dnf install ./package.rpm` instead of `rpm -ivh package.rpm` because DNF can resolve dependencies from configured repositories.
