# Day 25 — Bash Scripting Fundamentals

## What is a Script?

A script is a **file**. The system normally takes input from input devices (keyboard, mouse), but it can also take input from files. A script file contains commands that the system executes — instead of typing 40 commands one by one, you write them once and run the file.

**Scripting = semi-automation.** You write the script and execute it manually. If you schedule it with a cron job, it becomes full automation.

---

## Shell Environments

Check which shells your OS supports:

```bash
cat /etc/shells
# /bin/sh
# /bin/bash
# /sbin/nologin
# /usr/bin/sh
# /usr/bin/bash
# /usr/sbin/nologin
# /bin/tcsh
# /bin/csh
```

Check your current shell:

```bash
echo $0
echo $SHELL     # both work
```

Switch shell environments:

```bash
sh              # switch to sh
csh             # switch to csh
echo $0         # verify
exit            # return to bash
```

---

## Writing a Script — 3 Steps

1. Create the file and write the script
2. Give executable permission
3. Execute the script

### Step 1 — Create and Write

```bash
vim myfile.sh
```

> `.sh` extension is **not mandatory** — but giving it helps VIM identify the file type and catch typos through syntax highlighting.

Every bash script must start with **shebang**:

```bash
#!/bin/bash
```

`#!` = shebang (also called shabang, sharp bang, hashbang)
`/bin/bash` = the interpreter — translates your human-readable commands into machine language

### Step 2 — Give Execute Permission

```bash
chmod +x myfile.sh
```

> By default, files in Linux don't have execute permission — this is by design, for security. You must grant it manually.

### Step 3 — Execute

```bash
./myfile.sh
```

---

## Comments

`#` at the start of a line = comment. That line will not execute.

```bash
# This is a comment — won't execute
echo "Hello"    # inline comment also works
```

**Why use comments?** When a script has 100+ lines, comments help you and others understand the logic without reading every line.

### Printing a `#` in Output

```bash
echo \# not a comment \#     # backslash before # prints it literally
echo '# not a comment #'     # single quotes treat entire string as literal
```

---

## Basic Script Example

```bash
#!/bin/bash

echo "This is my first bash script" > ~/output.txt
echo " " >> ~/output.txt
echo "##################################" >> ~/output.txt

echo "LIST BLOCK DEVICES" >> ~/output.txt
echo " " >> ~/output.txt
lsblk >> ~/output.txt
echo " " >> ~/output.txt
echo "##################################" >> ~/output.txt

echo "FILESYSTEM FREE SPACE STATUS" >> ~/output.txt
echo " " >> ~/output.txt
df -h >> ~/output.txt
echo "##################################" >> ~/output.txt
```

> `>` = overwrite. `>>` = append. `~` = current user's home directory.
>
> `df -h` shows disk free space in human-readable format.
> `df -Th` adds the filesystem type column.

---

## Variables

Variables store values so you can reuse them without repeating yourself.

**Rules:**
- Cannot start with a number (`1date` = invalid)
- Cannot start with a special character (`$date`, `%date` = invalid)
- Can be uppercase, lowercase, or mixed (`DATE`, `date`, `Date` all valid)

```bash
NAME="Abishai"
echo $NAME          # prints: Abishai
```

### Storing Command Output in a Variable

Use `$()` to capture a command's output:

```bash
DATE=$(date +"%d-%b-%Y")    # stores today's date like: 08-May-2026
echo $DATE
```

> Without `$()`, the variable stores the literal word `date`, not the command's output.
> `$` = used to call/map a variable value. `$()` = used to run a command and capture output.

---

## Data Backup Script

```bash
#!/bin/bash
# Purpose = Backup of Important Data

DATE=$(date +"%d-%b-%Y")
SRCDIR="/etc"
DESDIR="/root/data-backup"
FILENAME=file-$DATE.tar.gz

# Making backup with compression
tar -cpzf $DESDIR/$FILENAME $SRCDIR
```

| Variable | Value | Purpose |
|----------|-------|---------|
| `DATE` | today's date | appended to filename so each backup is unique |
| `SRCDIR` | `/etc` | source directory to back up |
| `DESDIR` | `/root/data-backup` | where the backup file is saved |
| `FILENAME` | `file-08-May-2026.tar.gz` | archive filename with date |

`tar -cpzf`:
- `c` = create archive
- `p` = preserve permissions (extracted files keep original permissions)
- `z` = gzip compression
- `f` = filename

If you schedule this script with a cron job to run at midnight every day, you get a dated backup automatically — and can recover data from any specific date if disaster strikes.

---

## For Loop

### Syntax 1 — In List

```bash
for variable in value1 value2 value3
do
    command
done
```

### Syntax 2 — C-style

```bash
for (( i=1; i<=5; i++ ))
do
    command
done
```

### Example — Print Weekdays

```bash
#!/bin/bash

i=1
for day in Mon Tue Wed Thu Fri
do
    echo "Weekday $((i++)) : $day"
done
```

Output:
```
Weekday 1 : Mon
Weekday 2 : Tue
Weekday 3 : Wed
Weekday 4 : Thu
Weekday 5 : Fri
```

**Using a variable for the list:**

```bash
WEEKDAYS="Mon Tue Wed Thu Fri"

for day in $WEEKDAYS
do
    echo "Weekday $((i++)) : $day"
done
```

> `$((i++))` = arithmetic expression (increment). Each iteration adds 1 to `i`.
> If you wrap `$WEEKDAYS` in double quotes, it's treated as one string value — loop runs once only.

---

## Topics Remaining in Bash Scripting

- Creating multiple users (script)
- Database backup script
- Important backup
- For loop (continued)
- If statement
- While loop

---

## Key Takeaway

Scripts = files containing commands. Start every bash script with `#!/bin/bash`. Files don't have execute permission by default — use `chmod +x`. Use `#` for comments. Variables store values — use `$()` to capture command output. For loop iterates over a list or a range. The backup script is a real-world pattern: archive + date in filename + scheduled via cron = automated daily backup.
