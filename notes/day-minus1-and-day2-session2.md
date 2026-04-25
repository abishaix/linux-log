# Day -1 — Linux Filesystem Hierarchy (FHS)

> Note: "Day -1" was a pre-course or introductory session — covered before Day 1 officially started.

## The Root Directory `/`

- `/` is the top-level (root) directory — parent of everything
- Everything in Linux lives under `/`
- "Directory" = folder

---

## Key Directories

| Directory | Purpose |
|-----------|---------|
| `/root` | Home directory for the **root user** (superuser) |
| `/home` | Home directories for normal users (e.g. `/home/student`, `/home/raju`) |
| `/boot` | Contains bootable files (Grub2, kernel images) |
| `/etc` | Configuration files for services and packages — like Control Panel |
| `/bin` | Normal user executable commands (bin = binary) |
| `/sbin` | Root user executable commands (sbin = superuser bin) |
| `/usr` | User-shareable program files (like Program Files on Windows) |
| `/opt` | Third-party/optional program files (e.g. Ansible Tower GUI) |
| `/dev` | Device files — like Device Manager on Windows |
| `/var` | Variable data — log files live here (`/var/log`) |
| `/run/media` | Removable media mount point |
| `/proc` | Running processes — like Task Manager |
| `/sys` | OS-related system files |
| `/srv` | Background running services |
| `/mnt` | Empty directory — used as a manual mount point |
| `/tmp` | Temporary files — cleared on reboot |
| `/lib` | Library files (like DLLs on Windows) |
| `/lib64` | 64-bit library files |
| `/afs` | Andrew File System — distributed network file system (appears in RHEL 9.3+) |

---

## Boot Loaders by RHEL Version

| Boot Loader | RHEL Version |
|-------------|-------------|
| GRUB 2 | RHEL 7.0, 8.0, 9.0, 10.0 |
| GRUB | RHEL 6.0 |
| LILO | RHEL 5.0, 4.0, 3.0, 2.0 |

---

## Symbolic Links (from RHEL 7.0+)

From RHEL 7 onward, some directories are symlinks:

```
/bin   → /usr/bin
/sbin  → /usr/sbin
/lib   → /usr/lib
/lib64 → /usr/lib64
```

Why? Everything user-space lives under `/usr` — the symlinks just keep backward compatibility.

---

## Useful `/proc` Commands

```bash
cat /proc/cpuinfo    # CPU info
cat /proc/meminfo    # RAM and swap info
```

**Swap explained:** Virtual RAM — converts hard disk storage into temporary memory when RAM is full.
- 2GB RAM → 2GB swap
- 4GB RAM → 2–4GB swap
- 8GB RAM → max 4GB swap (not 8)

---

## Windows vs Linux — Quick Comparisons

| Windows | Linux |
|---------|-------|
| Device Manager | `/dev` |
| `.exe` installer | packages / commands |
| `C:\`, `D:\` drives | Mount points under `/mnt` |
| DLL files | `.lib` files in `/lib` |
| Control Panel | `/etc` |

> Linux has no drive letters. Everything is a mount point.

---

## Opening a Terminal by RHEL Version

| Version | Shortcut |
|---------|---------|
| RHEL 6.0 and older | `Ctrl+Alt+T` |
| RHEL 7.0, 8.0, 9.0, 10.0 | Super key (Windows key) → search "terminal" |

Terminal zoom:
```
Ctrl+Shift+"+" → zoom in
Ctrl+"-"       → zoom out
```

---

## Command Prompt Structure

```
[username@hostname ~]$ command  option  argument
```

- `$` = normal user
- `#` = root user

---

# Day 2 (Session 2) — Basic Navigation & System Commands

## `pwd` — Print Working Directory

```bash
pwd           # shows current location
echo $HOME    # prints home directory path of current user
```

---

## `cal` — Calendar

```bash
cal               # current month
cal -y            # full year
cal -3            # previous, current, next month
cal -3 2027       # 3-month view for a specific year
```

---

## `echo` — Print String or Variable Value

```bash
echo hello            # print literal string
echo $HOME            # home directory variable
echo $PATH            # PATH variable
echo $SHELL           # current shell
echo $BASH_VERSION    # bash version

# Set and use a variable:
Name=Abishai
echo $Name
```

---

## `uname` — System Info

```bash
uname -a    # all system info
uname -r    # kernel version only
```

---

## `date` — Display or Set Date/Time

```bash
date              # current date and time
date +%T          # time only (HH:MM:SS)
date +%D          # date only (MM/DD/YY)
date +%Y          # year only
date +%B          # month name
date +%j          # day of year (1–365)

# Set date/time (root only):
date -s "2026-04-11 16:47:55"
```

---

## `timedatectl` — Time & Date Control (RHEL 7+)

```bash
timedatectl                                    # show current time info
timedatectl set-time "2026-04-11 16:47:55"    # set time manually
timedatectl list-timezones                     # list all timezones
timedatectl set-timezone "Asia/Kolkata"        # set timezone
timedatectl status                             # full status
timedatectl set-ntp true                       # enable NTP auto-sync
```

Terms: UTC (Universal Time Coordinated), RTC (Real Time Clock), DST (Daylight Saving Time)

---

## `tty` — Print Terminal Name

```bash
tty     # prints e.g. /dev/pts/1
```

Each terminal session gets its own TTY. Useful when multiple users are logged in.

---

## `id` — Print User & Group IDs

```bash
id            # current user UID and GID
id student    # UID/GID for a specific user
```

---

## `whoami` vs `who am i` vs `who`

```bash
whoami        # just the username
who am i      # username + terminal + login time + source IP
who           # all logged-in users
who -a        # all sessions + boot info + run-level
```

---

## `env` — Environment Variables

```bash
env     # list all environment variables in current session
```

---

## `history` — Command History

```bash
history     # lists all previously executed commands
```

---

## `cat` — Concatenate / Print / Create Files

```bash
cat > file1                  # create/overwrite file (Ctrl+D to save)
cat >> file1                 # append to file (Ctrl+D to save)
cat file1                    # print contents
cat < file1                  # print via input redirect
cat file1 file2 > file3      # merge two files into a third
```

Redirect symbols: `>` overwrite, `>>` append, `<` input redirect
