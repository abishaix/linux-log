# Day 4 — Basic Commands: cal, echo, date, touch, mkdir, cat

> This session covered foundational commands in a hands-on lab format.

---

## `pwd` — Print Working Directory

```bash
pwd             # absolute path of current location
echo $HOME      # home directory of current user
```

---

## `cal` — Calendar

```bash
cal                         # current month
cal -y                      # full year
cal -3                      # previous, current, next month
cal -3 2027                 # 3-month view for specific year
cal -3 july                 # 3 months around July
cal -s                      # week starts Sunday
cal -m                      # week starts Monday
cal -j                      # Julian dates (day of year)
cal -h                      # no highlighting
```

---

## `echo` — Print String or Variable

```bash
echo hello            # literal string
echo $HOME            # home directory
echo $PATH            # executable paths
echo $SHELL           # current shell
echo $BASH_VERSION    # bash version
echo $0               # current shell name

# Set a variable:
Name=Imran
echo $Name
```

---

## `date` — Display or Set Date/Time

```bash
date              # current date and time
date +%T          # time only (HH:MM:SS)
date +%Y          # year only
date +%D          # date (MM/DD/YY)
date +%B          # month name
date +%j          # day of year

# Set date (root only):
date -s "2026-01-01 10:11:22"

# Or use timedatectl (recommended from RHEL 7+):
timedatectl set-time "2026-04-11 16:47:55"
```

---

## `touch` — Create Files / Change Timestamps

```bash
touch f1 f2 f3 f4             # create multiple empty files
touch k{1..4}                 # brace expansion: k1, k2, k3, k4

# Modify timestamp:
touch -m "2026-01-01 10:22:44" File1    # -m = modification time
touch -d "2026-01-01 10:22:44" File1    # -d = access + modification time
ls -l                                    # verify timestamp changed
```

---

## `mkdir` — Make Directories

```bash
mkdir dir1 dir2 dir3 dir4     # multiple at once
mkdir d{1..4}                  # brace expansion
mkdir -p L1/L2/L3/L4           # nested structure in one go
ls -R L1                       # verify recursively
tree L1                        # visual tree (if installed)

# Complex nested:
mkdir -p NIT/{Linux/{Rhcsa,Rhce},Windows/{Mcsa,Mcse},Oracle/{Sql,Plsql}}
tree NIT
```

---

## `cat` — Create, Append, Print Files

```bash
cat > File1           # create/overwrite (Ctrl+D to save)
cat >> File1          # append (Ctrl+D to save)
cat < File1           # print via input redirect
cat File1             # print directly
cat File1 File2 > File3   # merge File1 and File2 into File3
```

---

## Lab Practice Sequence (from class)

```bash
# Create files and verify:
cat > File1     # type content, Ctrl+D
cat >> File1    # add more, Ctrl+D
cat File1       # verify

cat > File2     # create File2
cat File1 File2 > File3
cat File3       # should show combined content

# Timestamp demo:
touch f1 f2 f3 f4
ls -l           # check timestamps
touch -m "2026-01-01 10:22:44" File1
ls -l           # see timestamp changed

# Directory creation:
mkdir -p L1/L2/L3/L4
ls -R L1

mkdir -p NIT/{Linux/{Rhcsa,Rhce},Windows/{Mcsa,Mcse},Oracle/{Sql,Plsql}}
tree NIT
```
