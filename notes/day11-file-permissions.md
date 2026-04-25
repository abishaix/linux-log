# Day 11 — File Permissions

## Reading Permissions

When you run `ls -l`, you see something like:
```
drwxr-xr-x. 2 root root 4096 Apr 23 pepsi
```

Breaking it down:

| `d` | `rwx` | `r-x` | `r-x.` | `2` | `root` | `root` | date | name |
|-----|-------|-------|--------|-----|--------|--------|------|------|
| type | user perms | group perms | other perms | links | owner | group owner | | filename |

- `d` = directory, `-` = file, `l` = symlink

---

## Permission Values

| Symbol | Name | Octal Value | What it means |
|--------|------|-------------|---------------|
| `r` | read | 4 | View file content / list directory |
| `w` | write | 2 | Modify file / create or delete files in directory |
| `x` | execute | 1 | Run file as program / enter directory |
| `-` | none | 0 | No permission |

Combinations:
- `rwx` = 4+2+1 = **7** (full permission)
- `r-x` = 4+0+1 = **5** (read + execute)
- `r--` = 4+0+0 = **4** (read only)
- `---` = 0 (no permissions)

---

## Who the Permission Applies To

```
rwx   r-x   r-x
 |     |     |
user  group other
```

- **user** = the file's owner
- **group** = members of the file's group
- **other** = everyone else

---

## `chmod` — Change Permissions

### Symbolic method:
```bash
chmod u+x file1         # add execute for user
chmod g-w file1         # remove write from group
chmod o+r file1         # add read for others
chmod a+x file1         # add execute for all (user+group+other)
```

### Numeric/octal method:
```bash
chmod 755 file1         # rwxr-xr-x
chmod 644 file1         # rw-r--r--
chmod 700 file1         # rwx------
```

---

## `umask` — Default Permission Mask

When you create a file or directory, Linux applies a default permission minus the umask value.

```bash
umask       # check current umask (usually 022)
```

- Default max for directories = 777
- Default max for files = 666
- With umask 022: directories get 755, files get 644

---

## Key Takeaway

Permissions in Linux are per-entity (user, group, other) and per-operation (read, write, execute). The octal values (4, 2, 1) are what you'll use most in real work — `chmod 755`, `chmod 644` etc. are standard patterns you'll memorize quickly.
