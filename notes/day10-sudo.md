# Day 10 — sudo

## What is `sudo`?

`sudo` = **Superuser Do** — lets a normal user run root-level commands without logging in as root. Gives controlled, restricted access instead of handing over the root password.

---

## `sudo` vs `su`

```bash
sudo su -       # switch to root WITH environment (full root session)
sudo su         # switch to root WITHOUT loading environment
su -            # same as sudo su - but requires root password directly
```

- `su -` = switch user AND bring the root environment (PATH, HOME, etc.)
- `su` without `-` = just switch user, keep current environment

---

## Root User Commands Live in `/sbin`

```bash
/sbin/useradd
/sbin/usermod
/sbin/groupadd
/sbin/fdisk
```

```bash
which useradd       # shows full path of the command
whereis useradd     # shows binary path, source, and man page locations
```

---

## `visudo` — Edit sudo Configuration

Always use `visudo` to edit `/etc/sudoers` — it validates syntax before saving and prevents breaking sudo.

```bash
visudo              # opens /etc/sudoers safely (go to line ~97)
```

### Sudoers Entry Format

```
Username    hostname=(commands)    permission
```

---

## Scenario-Based Examples

### Full sudo access (with password prompt):
```
student    ALL=(ALL)    ALL
```

### Full sudo access — no password:
```
student    ALL=(ALL)    NOPASSWD: ALL
```

### Limited sudo — specific commands only:
```
student    ALL=(ALL)    NOPASSWD: /sbin/useradd, /sbin/usermod, /sbin/groupadd, /sbin/fdisk
```

### Allow all EXCEPT specific commands:
```
student    ALL=(ALL)    NOPASSWD: ALL, !/sbin/userdel, !/sbin/groupdel
```

The `!` before a command = that command is excluded.

### Using an alias (group multiple commands under one name):
```
student    ALL=(ALL)    NOPASSWD: COMM
```
Define `COMM` as a `Cmnd_Alias` earlier in the sudoers file.

### sudo permission for a group:
```
%redhat    ALL=(ALL)    NOPASSWD: ALL
```
`%` prefix means it's a group, not a user.

---

## The `wheel` Group

In Linux, the `wheel` group is the standard way to grant sudo access to regular users.

```
%wheel    ALL=(ALL)    ALL             # with password
%wheel    ALL=(ALL)    NOPASSWD: ALL  # without password
```

Add a user to the wheel group to give them sudo:
```bash
usermod -aG wheel student
```

---

## `/etc/sudoers.d/` — Modular sudo Config

Instead of editing the main `/etc/sudoers` file directly, you can create separate files per user or team in `/etc/sudoers.d/`. Cleaner, easier to manage.

```bash
visudo -f /etc/sudoers.d/imran      # create/edit safely with syntax check
# OR
vim /etc/sudoers.d/imran            # edit directly (no syntax check — riskier)
```

Content of the file:
```
imran    ALL=(ALL)    ALL
```

---

## Key Takeaway

`sudo` is how you grant restricted root access without sharing the root password. `visudo` is the only safe way to edit sudoers. In production, grant only the specific commands each user needs — not `NOPASSWD: ALL` unless you have a good reason.
