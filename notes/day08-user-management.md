# Day 8 — User Management

## Why User Accounts?

You need a user account to log in to the OS and execute tasks. Without it, you can't manage resources.

**Prompt indicators:**
- `#` = root user
- `$` = normal user

---

## 3 Types of Users in Linux

| Type | Example | UID Range (RHEL 7+) | Shell |
|------|---------|---------------------|-------|
| Root | root | 0 | `/bin/bash` |
| System user | apache, ftp | 1–999 | `/sbin/nologin` |
| Normal user | student, u1 | 1000–60000 | `/bin/bash` |

> System users are created automatically when services are installed (e.g. installing httpd creates `apache`). They use `/sbin/nologin` — no interactive shell, so they can't log in.

---

## User Properties — 7 Fields in `/etc/passwd`

```
username : password : UID : GID : comment : home-dir : shell
```

| Field | Description | Modify with |
|-------|-------------|-------------|
| username | Login name | `-l` (usermod only) |
| password | `x` → actual hash in `/etc/shadow` | `passwd` |
| UID | User ID | `-u` |
| GID | Group ID | `-g` |
| comment | Description / full name | `-c` |
| home-dir | Home directory path | `-d` (new), `-m` (existing) |
| shell | Shell environment | `-s` |

Example entries:
```
root:x:0:0:root:/root:/bin/bash
apache:x:48:48:Apache:/var/www/html:/sbin/nologin
u1:x:1001:1001::/home/u1:/bin/bash
u2:x:2002:2002:Hr:/home/dir2:/sbin/nologin
```

> `/etc/passwd` = local user database
> `/etc/shadow` = local password database (hashed passwords)

---

## `useradd` — Create a User

```bash
useradd u1                                                  # basic creation
useradd u2 -u 2002 -c "Hr" -d /home/dir2 -s /sbin/nologin  # custom options
tail /etc/passwd                                            # verify
```

---

## `usermod` — Modify a User

Same flags as `useradd`, plus:

```bash
usermod -s /bin/bash u2              # change shell
usermod -m -d /home/u2 u2            # move home directory to new path
usermod -l test-user u2              # rename user (home dir NOT renamed automatically)
usermod -m -d /home/test-user test-user  # then fix home dir separately
usermod -L u1                        # lock the user
usermod -U u1                        # unlock the user
passwd -S u1                         # check lock status
```

> `-l` (rename) is only in `usermod`, not `useradd`. Renaming does NOT move the home directory — do that with `-m -d` in a separate step.

---

## `userdel` — Delete a User

```bash
userdel u1              # delete user but KEEP home directory
userdel -r u2           # delete user AND home directory
ls /home/               # verify home dir is gone
rm -rf u1               # manually clean up leftover home if userdel without -r
```

---

## `passwd` — Set/Change Password

```bash
passwd u1               # set password for u1 (as root)
passwd -S u1            # check status (P = has password, L = locked, NP = no password)
```

---

## `su` — Switch User

```bash
su - u1         # switch to u1 WITH environment (prompts for password)
su - root       # switch to root WITH environment
su -            # shorthand for switch to root
su root         # switch to root WITHOUT loading environment
```

`su -` loads the target user's full environment (HOME, PATH, etc.). `su` without `-` just switches user but keeps current environment.

---

## When a Service is Installed — 3 Things Happen

Example: installing `httpd` (Apache web server):
1. Creates system user `apache`
2. Creates config files in `/etc/`
3. Creates home directory `/var/www/html`

---

## `/etc/login.defs` — Login Defaults

Contains default settings applied when creating users — UID ranges, password aging defaults, etc.

```bash
cat /etc/login.defs
```

---

## Key Takeaway

Know the 3 user types, the 7 fields in `/etc/passwd`, and the `useradd`/`usermod`/`userdel` flow. The lock/unlock (`-L`/`-U`) and rename + home-dir move pattern come up a lot in real admin work.
