# Day 15 — Package Management: RPM & YUM

> Note: Day 14 was missed (Saturday session). This picks up at Day 15 — Monday, April 27.

---

## Why Package Management?

In Windows, you install software by downloading `.exe` files and clicking through a wizard. In Linux:
- With GUI — same idea, download and double-click `.rpm` files
- With CLI — you use `rpm` or `yum` commands

Linux packages use the `.rpm` format (Red Hat Package Manager).

---

## Where Packages Live

In a real environment you download packages from the internet. In a lab without a subscription (like RHEL 7.0 without Red Hat subscription), you use the DVD ISO image instead — it has a `Packages/` directory with pre-bundled `.rpm` files.

```bash
cd /run/media/student/RHEL-7.0/        # navigate to mounted DVD
ls Packages/                            # list available packages
```

> `/run/media` = where removable media (DVD, USB) gets mounted automatically.

---

## RPM — Red Hat Package Manager

RPM installs **single packages** directly from `.rpm` files — **no dependency resolution**.

### Install

```bash
rpm -ivh tree-1.6.0.el7.x86_64.rpm     # install a package
                                         # i = install, v = verbose, h = hash progress
```

`-v` and `-h` are optional. `-i` is mandatory.

### Verify installation

```bash
rpm -q tree             # query — is this package installed?
rpm -qa                 # query all installed packages
rpm -qa | grep tree     # find a specific package from all installed
```

> Use `rpm -q` to check installation status — not `tree -v` or `--version`. Not every package has a version flag, but `rpm -q` works for all.

```bash
which tree              # show where the binary got installed (/usr/bin/tree)
whereis tree            # show binary, source, and man page locations
```

### Get package information

```bash
rpm -qi tree            # query info — version, size, license, description
rpm -qd tree            # query documentation — license file, README, man page paths
rpm -ql tree            # query list — where all files got installed
rpm -qc vsftpd          # query config files — config file paths for a service
```

> `rpm -qc` only works for **services**, not simple commands. `tree` is a command with no config files — use a service like `vsftpd` or `httpd` to demo this.

### Update & Uninstall

```bash
rpm -Uvh package.rpm    # Update — U = update, v = verbose, h = hash
rpm -e tree             # Erase (uninstall) the tree package
rpm -q tree             # verify it's gone
```

---

## RPM vs YUM — Key Differences

| Feature | RPM | YUM |
|---------|-----|-----|
| Installs | Single package | Package + all dependencies automatically |
| Requires repository | No | Yes |
| Dependency resolution | No — fails if deps missing | Yes — resolves automatically |
| Use case | Simple standalone packages | Services and complex software |

**Why RPM fails on something like httpd:**
```bash
rpm -ivh httpd*.rpm     # fails — "failed dependencies"
```
`httpd` has 7–8 dependencies. RPM won't install them automatically — you'd have to install each one manually. That's where YUM comes in.

---

## YUM — Yellowdog Updater Modified

YUM installs packages **along with all dependencies automatically**, pulling from a configured repository.

### How YUM works

YUM needs a **repository** — a source of packages (either a URL on the internet, or a local path). Without a repository configured, YUM can't install anything.

- EC2 instances have repositories by default (internet access + Red Hat subscription)
- Local lab machines need a manually created local repo

---

## Creating a Local YUM Repository

When there's no internet/subscription, create a local repo pointing to the DVD:

```bash
vim /etc/yum.repos.d/rhel.repo
```

> The filename can be anything, but the extension **must be `.repo`**.

Contents of the file:

```ini
[rhel]
name=Red Hat 64-bit
baseurl=file:///run/media/student/RHEL-7.0/
gpgcheck=0
enabled=1
```

| Field | Meaning |
|-------|---------|
| `[rhel]` | Repository ID (any name) |
| `name` | Human-readable label |
| `baseurl` | Where to get packages — `file://` = local path, `http://` = internet URL |
| `gpgcheck` | Verify package signature — `0` = disabled, `1` = enabled |
| `enabled` | `1` = this repo is active |

> `file:///path` — three slashes because the format is `file://` (protocol) + `/path` (absolute path starting with `/`).

After creating the repo:

```bash
yum repolist all        # verify repo is recognized and enabled
yum clean all           # clear cached repo data if needed
```

---

## YUM Commands

### Install

```bash
yum install httpd* -y           # install httpd and all related packages + dependencies
                                 # -y = answer yes to all prompts automatically
```

When prompted without `-y`:
- `y` = install
- `d` = download only (don't install yet)
- `n` = cancel

### Query / List

```bash
yum list httpd                  # check if a package is installed
yum list all                    # list all packages (installed + available in repo)
yum list all | grep tree        # filter results
```

> `@` symbol next to a package = installed in the OS. No `@` = available in repo but not installed.

### Update & Remove

```bash
yum update httpd* -y            # update package (needs internet/subscription for new version)
yum remove httpd* -y            # uninstall package and related packages
```

### Search

```bash
yum search all "web server"     # find packages related to a keyword — very useful
yum provides /var/www/html      # find which package owns a specific file or directory
```

### Group Packages

```bash
yum grouplist                   # list all package groups
                                 # Environmental groups = default OS groups
                                 # Available groups = optional installs (e.g. Development Tools)

yum groupinstall "Development Tools" -y     # install an entire group at once
                                             # installs gcc, c++, kernel headers, etc.
```

### History & Undo

```bash
yum history                     # list all install/remove actions with ID, date, user
yum history undo <ID>           # undo a specific action (reinstalls what was removed, or removes what was installed)
```

### Cache

```bash
yum clean all                   # clear YUM cache
```

---

## wget vs curl

```bash
wget <URL>      # download a file from the internet to disk
curl <URL>      # fetch and display content from a URL (doesn't save by default)
```

Neither installs packages — they're tools to retrieve files from the internet. You still use `rpm` or `yum` to install.

---

## Key Takeaway

Use **RPM** for simple standalone packages with no dependencies. Use **YUM** for anything real — services, complex software — because it handles dependencies automatically. YUM needs a repository (local or internet). EC2 gives you repos out of the box; local machines need a `.repo` file in `/etc/yum.repos.d/`.
