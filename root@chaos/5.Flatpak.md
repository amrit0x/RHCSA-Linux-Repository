# Flatpak — RHCSA (Complete Notes)

### **1. What is Flatpak?**

Flatpak is a package management system used to install, run, update, and remove applications in a sandboxed environment.

- Sandbox: Isolates applications and limits their access to the host system.
- Remote: A repository containing Flatpak applications.
- Application ID: Unique application identifier, e.g. `org.gimp.GIMP`.
- Runtime: Shared libraries and components required by Flatpak applications.

**Flatpak vs DNF :**

| DNF | Flatpak |
| --- | --- |
| Manages RPM packages | Manages Flatpak applications |
| Uses RPM repositories | Uses Flatpak remotes |
| Commonly manages system software | Primarily manages applications |
| Installs traditional RPM dependencies | Uses Flatpak runtimes |

### **2. Install Flatpak :**

```bash
dnf install -y flatpak
```

Verify:

```bash
flatpak --version
```

### **3. Configure Flatpak Repositories :**

Add a repository (remote):

```bash
flatpak remote-add --if-not-exists flathub https://dl.flathub.org/repo/flathub.flatpakrepo
```

- `remote-add` — Adds a repository.
- `-if-not-exists` — Avoids errors if the remote already exists.
- `flathub` — Repository name.
- URL — Location of the repository configuration.

Important repository commands:

| Command | Purpose |
| --- | --- |
| `flatpak remotes` | List configured repositories |
| `flatpak remotes --show-details` | Show repository details |
| `flatpak remote-ls flathub` | List available packages |
| `flatpak remote-ls --app flathub` | List available applications only |
| `flatpak remote-delete flathub` | Remove repository configuration |

flatpak remote-ls flathub | grep -i gimp  -> For searching a specific Flatpak application (alternative of "flatpak seach gimp" command)

### **Local repository (RHCSA practice)**

If the exam provides an existing Flatpak repository at `/srv/flatpak-repo`:

```bash
flatpak remote-add labrepo file:///srv/flatpak-repo
```

If the repository is unsigned and explicitly trusted:

```bash
flatpak remote-add --no-gpg-verify labrepo file:///srv/flatpak-repo
```

- `-no-gpg-verify` disables signature verification. Use it only when appropriate for a trusted lab repository.

### **4. Manage Flatpak Applications :**

| Command | Purpose |
| --- | --- |
| `flatpak search gimp` | Search applications |
| `flatpak install flathub org.gimp.GIMP` | Install application |
| `flatpak list` | List installed applications and runtimes |
| `flatpak list --app` | List installed applications only |
| `flatpak info org.gimp.GIMP` | Show application information |
| `flatpak run org.gimp.GIMP` | Launch application |
| `flatpak update` | Update all applications and runtimes |
| `flatpak update org.gimp.GIMP` | Update specific application |
| `flatpak uninstall org.gimp.GIMP` | Remove application |
| `flatpak uninstall --unused` | Remove unused runtimes/extensions |

### **5. System-wide vs User Installation :**

User-specific installation:

```bash
flatpak install --user flathub org.gimp.GIMP
```

For a user-specific repository:

```bash
flatpak remote-add --user --if-not-exists flathub https://dl.flathub.org/repo/flathub.flatpakrepo
```

Remember: A `--user` application requires access to an appropriate user-scoped remote. System and user installations are separate.

### **6. RHCSA Practical Lab :**

TASK: Configure a Flatpak repository, install an application, verify its installation, and remove it.

```bash
# 1. Install Flatpak
dnf install -y flatpak

# 2. Add repository
flatpak remote-add --if-not-exists flathub https://dl.flathub.org/repo/flathub.flatpakrepo

# 3. Verify repository
flatpak remotes

# 4. Search application
flatpak search gimp

# 5. Install application
flatpak install flathub org.gimp.GIMP

# 6. Verify installation
flatpak list --app

# 7. Remove application
flatpak uninstall org.gimp.GIMP

# 8. Verify removal
flatpak list --app
```

For the actual RHCSA exam, use the repository and application specified in the task. Red Hat states that internet access is not provided during the exam, so practice with supplied local or lab-network repositories as well.

### **7. Common Errors :**

| Error / Problem | Check or Solution |
| --- | --- |
| `flatpak: command not found` | `dnf install flatpak` |
| `No remote refs found` | Verify remote and application ID |
| Remote not configured | `flatpak remotes` |
| Application not found | `flatpak search <name>` |
| Permission denied | Check `--user` vs `--system` and privileges |
| Application not installed | `flatpak list --app` |

## 8. Final RHCSA Revision Checklist

Practical readiness

0/9 completed

- [ ]  Install Flatpak using DNF
- [ ]  Add a Flatpak remote
- [ ]  List configured remotes
- [ ]  List applications in a remote
- [ ]  Search and install an application
- [ ]  Verify installed applications
- [ ]  Remove an installed application
- [ ]  Understand --user vs --system
- [ ]  Configure a provided local repository

Exam priority: Focus most on `remote-add`, `remotes`, `remote-ls`, `install`, `list`, and `uninstall`. These directly support the repository configuration and package management objectives. Update, run, and troubleshooting commands are useful supporting knowledge.
