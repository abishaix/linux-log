# Day 12 — Special Permissions

## The 3 Special Permissions

Beyond the standard `rwx`, Linux has 3 special permission bits:

| Permission | Applies To | Symbol | Numeric |
|------------|------------|--------|---------|
| SUID (setuid) | Files | `s` or `S` on user's `x` bit | 4xxx |
| SGID (setgid) | Directories | `s` or `S` on group's `x` bit | 2xxx |
| Sticky bit | Directories | `t` or `T` on other's `x` bit | 1xxx |

Lowercase `s`/`t` = special permission + execute. Uppercase `S`/`T` = special permission but NO execute.

---

## SUID — Set User ID (files only)

Allows a normal user to execute a file with the **file owner's permissions** (usually root) — without needing sudo.

```bash
chmod u+s filename
chmod 4744 filename     # 4 = suid, 744 = rwxr--r--
```

Example: `fdisk -l` normally requires root. With SUID set on `/usr/sbin/fdisk`, a normal user could run it.

```bash
which fdisk
ls -l /sbin/fdisk
# -rwxr-xr-x 1 root root 117168 Mar 6 16:00 /sbin/fdisk
```

> SUID is mainly used by developers. Be careful — it's a potential security risk if misused.

---

## SGID — Set Group ID (directories)

When set on a directory, any files created inside inherit the **directory's group** — not the creating user's primary group.

This enables **directory collaboration**: multiple users can edit each other's files inside that directory.

```bash
chmod g+s dirname
chmod 2775 dirname      # 2 = sgid, 775 = rwxrwxr-x
```

**Without SGID:** u1 creates a file → owned by u1. u2 can't edit it.  
**With SGID:** u1 creates a file → owned by the shared group. u2 can edit it.

```bash
mkdir pepsi
chmod g+s pepsi         # now files created here inherit pepsi's group
```

---

## Sticky Bit (directories only)

When set on a directory where everyone has write access, the sticky bit ensures **only the file's owner (or root) can delete it** — even if others have write permission to the directory.

```bash
chmod o+t dirname
chmod 1757 dirname      # 1 = sticky, 757 = rwxr-xrwx
```

Use case: `/tmp` has world-write permissions but the sticky bit is set — so you can create files there but can't delete other users' files.

```bash
ls -ld /tmp
# drwxrwxrwt — the 't' at the end = sticky bit set
```

---

## Summary

| Special Perm | On What | Purpose |
|-------------|---------|---------|
| SUID | Files | Run file as file owner (usually root) |
| SGID | Directories | Files inherit directory's group — enables collaboration |
| Sticky bit | Directories | Only owner can delete their own files |

---

## Key Takeaway

SGID on shared directories is the most common real-world use case — it's what makes team collaboration directories work cleanly. Sticky bit is what protects `/tmp`. SUID is mostly a developer/package concern.
