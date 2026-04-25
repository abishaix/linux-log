# Day 6 — Filters: grep, less, more, head, tail, sort, sed, find

## `grep` — Global Regular Expression Print

A filter command — finds words/patterns inside files or filters command output.

```bash
grep root /etc/passwd                      # lines containing "root"
grep -n student /etc/passwd                # with line numbers
grep -e root -e student /etc/passwd        # multiple words, one file
grep root /etc/passwd /etc/group           # one word, multiple files
grep -v root /etc/passwd                   # lines that DON'T contain "root"
grep -i hello file1                        # case-insensitive match
grep -nB4 wheel /etc/group                 # match + 4 lines BEFORE
grep -nA4 wheel /etc/group                 # match + 4 lines AFTER

# Combine with ls and pipe:
ls -l | grep ^b         # list only block files (lines starting with b)
ls -l | grep t$         # lines ending with t
ls -l | grep sda        # lines containing "sda"
ls -l | grep ^b | wc -l # count block files
```

| Flag | Meaning |
|------|---------|
| `-n` | Show line numbers |
| `-e` | Multiple patterns |
| `-v` | Invert — exclude matches |
| `-i` | Case-insensitive |
| `-B4` | Show 4 lines before match |
| `-A4` | Show 4 lines after match |
| `-r` | Recursive through directories |

---

## `less` and `more` — Page-by-Page Viewing

Both let you read large files page by page instead of dumping everything at once.

```bash
less /etc/passwd
more /etc/passwd
```

| Key | Action |
|-----|--------|
| `d` | Next page |
| `b` | Previous page (`less` only) |
| `/word` | Search for a word |
| `v` | Open in vim |
| `q` | Quit |

> `less` is more capable than `more` — it lets you go backwards.

---

## `head` and `tail` — First/Last Lines

```bash
head /etc/passwd          # first 10 lines (default)
head -n4 /etc/passwd      # first 4 lines
tail /etc/passwd          # last 10 lines (default)
tail -n4 /etc/passwd      # last 4 lines

# Multiple files:
head /etc/passwd /etc/group
```

> You'll use `tail /etc/passwd` and `tail /etc/shadow` constantly when creating/modifying users to verify changes.

---

## `sort` — Sort Output

```bash
sort file1          # alphabetical sort
sort -r file1       # reverse order
sort -u file1       # remove duplicate lines
sort -u file1 > file4    # save unique sorted output to new file
```

---

## `sed` — Stream Editor

Finds, replaces, or deletes text in a file or command output **without opening the file**.

```bash
# Basic syntax:
sed 's/old/new/' filename        # replace first occurrence per line
sed 's/old/new/g' filename       # replace ALL occurrences (global)
sed 's/Linux/Windows/1' file5    # replace 1st occurrence on each line
sed 's/Linux/Windows/2' file5    # replace 2nd occurrence on each line
sed 's/Linux/Windows/g' file5    # replace all
```

- `s` = substitute
- `g` = global (all occurrences)
- `/` = separator

> `sed` doesn't modify the original file by default — it prints the result. Use `> newfile` to save it.

---

## `find` — Search for Files

```bash
find / -name File1              # find by name
find / -inum 53558549           # find by inode number
find / -type b                  # find block files
find / -type f                  # find regular files
find / -type d                  # find directories
find / -size 10k                # files exactly 10KB
find / -size +10k               # files larger than 10KB
find / -size -10k               # files smaller than 10KB
find / -user student            # files owned by user "student"
find / -group student           # files owned by group "student"
```

| Type flag | File type |
|-----------|-----------|
| `f` | Regular file |
| `d` | Directory |
| `b` | Block device |
| `c` | Character device |
| `l` | Symbolic link |
