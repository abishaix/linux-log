# Day 3 — File & Directory Commands, File Types & Links

> **Question from class:** What's the difference between `/bin` and `/usr`? Both have executables and program files.
> `/bin` = essential commands needed at boot/recovery. `/usr/bin` = non-essential programs installed after boot. From RHEL 7+, `/bin` is a symlink to `/usr/bin` anyway.

---

## `touch` — Change Timestamps / Create Empty Files

Primary purpose: change the timestamp of a file. Secondary: create a zero-byte file.

```bash
touch file1                           # update timestamp (or create if not exists)
touch f1 f2 f3 f4                     # multiple files at once
touch k{1..4}                         # brace expansion: creates k1, k2, k3, k4
touch -d "2024-01-15 09:00:00" file1  # set a specific timestamp
touch -m "2026-01-01 10:22:44" file1  # modify timestamp only
```

> Files created with `cat` have content. Files created with `touch` are 0 bytes.
> Timestamps matter for developers and service deployments — wrong timestamp = sync issues.

---

## `mkdir` — Make Directories

```bash
mkdir dir1 dir2 dir3          # multiple at once
mkdir d{1..4}                 # creates d1, d2, d3, d4

mkdir -p L1/L2/L3/L4          # nested parent/child in one go
ls -R L1                      # verify the tree structure
tree L1                       # cleaner visual (if tree is installed)

# Complex nested structure:
mkdir -p NIT/{Linux/{Rhcsa,Rhce},Windows/{Mcsa,Mcse},Oracle/{Sql,Plsql}}
```

---

## `cd` — Change Directory

```bash
cd ..                             # one directory up
cd ../..                          # two directories up
cd                                # go to home directory
cd ~                              # also goes to home directory
cd /absolute/path/to/dir          # navigate by absolute path
```

---

## `ls` — List Files

```bash
ls                   # list filenames
ls -l                # long listing (permissions, owner, size, date)
ls -la               # include hidden files (files starting with .)
ls -ld dirname       # show directory's own properties (not its contents)
ls -ltr              # sort by time, oldest first

# Wildcards and patterns:
ls f*                # all files starting with f
ls ?at               # any single char + "at" (rat, mat, hat, cat...)
ls [ae]*             # files starting with a or e
ls [!ae]*            # files NOT starting with a or e
ls [a-m][c-z][4-9]   # files matching a specific character range
ls F?                # F + exactly one character
ls [kf]*             # files starting with k or f
```

---

## Hidden Files

Any file or directory starting with `.` is hidden — `ls` won't show it by default.

```bash
ls -la              # show all files including hidden
touch .hiddenfile   # create a hidden file
mkdir .hiddendir    # create a hidden directory

# To unhide a file — just rename it (remove the dot):
mv .hiddenfile hiddenfile
```

---

## `cp` — Copy Files

```bash
cp file1 file2          # copy into file2 (overwrites file2 content)
cp file1 dir1/          # copy into a directory
cp -rvf dir1/ dir2/     # copy directory into directory
```

Flags: `-r` (recursive), `-v` (verbose), `-f` (force)

> `cp file1 file2` when file2 already has content — file2's content gets replaced by file1's.

---

## `mv` — Move / Rename

```bash
mv file1 file2       # rename file1 to file2
mv file1 dir1/       # move file into directory
mv dir2 dir4         # rename a directory
mv dir3 demo         # rename dir3 to demo
```

Same directory = rename. Different directory = move.

---

## `rm` / `rmdir` — Remove

```bash
rmdir dir1           # remove EMPTY directory only
rm file1             # remove a file
rm f*                # remove all files matching pattern
rm -rvf dir4/        # remove non-empty directory (recursive, verbose, force)
rm -rvf *            # remove everything in current directory
```

> `rmdir` won't work on non-empty directories. Use `rm -rvf` for those.

---

## File Types in Linux

Linux identifies files by type, not extension. Use `ls -l` — the first character tells you the type.

| First char | File Type | Example |
|------------|-----------|---------|
| `-` | Normal/regular file | `file1`, `script.sh` |
| `d` | Directory | `dir1/` |
| `c` | Character file | keyboard, mouse (`/dev/tty`) |
| `b` | Block file | HDD, USB drive (`/dev/sda`) |
| `l` | Symbolic link (soft link) | `s1 -> /home/student/soft` |

```bash
file file1      # shows file type
file dir1       # "directory"
file /dev/sda   # "block special"
file /bin       # "symbolic link to usr/bin"

# Filter by type using ls + grep:
ls -l | grep ^b     # list only block files
ls -l | grep ^l     # list only symlinks
```

---

## Inodes

The kernel assigns every file a unique ID number called an **inode**. The inode stores all the file's properties (permissions, owner, size, timestamps) — everything except the filename.

```bash
ls -i filename          # show inode number of a file
ls -i /home/student/soft
```

Two files with the same inode = same underlying data on disk. Two files with different inodes = different data. This is how you tell a soft link from a hard link.

---

## `du` — Disk Usage

```bash
du -h filename          # size of a file in human-readable format (KB, MB)
du -h s1                # size of the link itself (tiny — just a pointer)
du -h /home/student/soft    # size of the actual file
```

`-h` = human readable. Without it you get raw block counts.

---

## Reading `ls -l` Output — Block Count

When you run `ls -l` you see a line like:
```
total 8
```
This means 8 blocks are allocated. 1 block = 512 bytes, so `total 8` = 4096 bytes (4 KB) of actual disk space used.

---

## Soft Links (Symbolic Links) — Shortcuts

A soft link is a pointer to another file. Like a Windows shortcut.

```bash
ln -s /home/student/soft s1     # create soft link s1 pointing to soft
ls -l                           # s1 -> /home/student/soft
cat s1                          # reads through to the original file
cat >> s1                       # appends to original file through the link
du -h s1                        # size of the link itself (tiny)
ls -i s1                        # inode of s1 (different from original)
ls -i /home/student/soft        # inode of original (different from s1)
```

**What happens if you delete the original?**
```bash
rm -rf /home/student/soft
cat s1    # broken link — "No such file or directory"
```
Soft link breaks when original is deleted. Different inodes.

---

## Hard Links — Backup

A hard link is another name for the same file. Both point to the same inode (same data on disk).

> Hard links work on **files only** — you cannot create a hard link to a directory.

```bash
ln /home/student/hard H1        # create hard link H1
ls -l                           # same size as original
cat H1                          # same content
cat >> H1                       # appending via H1 also updates original
du -h H1                        # same disk usage
ls -i H1                        # same inode as original
ls -i /home/student/hard        # same inode — they're the same file
```

**What happens if you delete the original?**
```bash
rm -rf /home/student/hard
cat H1    # still works — data isn't deleted until all links are gone
```
Hard link survives deletion of original. Same inode.

---

## Soft Link vs Hard Link — Summary

| Feature | Soft Link | Hard Link |
|---------|-----------|-----------|
| Type | Shortcut/pointer | Another name for same file |
| Inode | Different from original | Same as original |
| Breaks if original deleted? | Yes | No |
| Can link across filesystems? | Yes | No |
| Use case | Shortcuts, flexible paths | Backup, safety |
| Command | `ln -s source link` | `ln source link` |

---

## `wc` — Word Count

```bash
wc /etc/passwd          # lines, words, characters
wc -l /etc/passwd       # lines only
wc -w /etc/passwd       # words only
wc -c /etc/passwd       # characters (bytes) only

# Combine with other commands:
ls -l | grep ^b | wc -l     # count block files in /dev
```
