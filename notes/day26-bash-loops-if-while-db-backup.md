# Day 26 — For Loops, Multiple Users Script, Database Backup, If/While

## For Loop — Advanced Syntax

### Iterating Over Home Directory Items

```bash
#!/bin/bash
i=1
cd ~                            # ~ = current user's home directory
for item in *                   # * = wildcard — captures every item in current dir
do
    echo "Item $((i++)) : $item"
done
```

### Filtering with AWK

`awk` is a filter command — reads file content and filters specific fields.

```bash
awk -F: '{print $1}' /etc/passwd    # print first field (username) from /etc/passwd
# -F: means field separator is colon (:)
# $1 = first field, $2 = second field, etc.
```

> `/etc/passwd` has 7 fields: username, password, UID, GID, description, home dir, shell.

Using AWK output in a for loop:

```bash
for username in `awk -F: '{print $1}' /etc/passwd`
do
    echo $username
done
```

### Filtering Config Files

```bash
for file in /etc/[abcd]*.conf     # files starting with a, b, c, or d with .conf extension
do
    echo "File $((i++)) : $file"
done
```

### Break and Continue

**Break** — stop the loop when condition is met:

```bash
for day in Mon Tue Wed Thu Fri
do
    if [ $i -eq 3 ]; then
        break                   # stops at Wednesday (i=3)
    fi
    echo "Weekday $((i++)) : $day"
done
```

**Continue** — skip specific iterations:

```bash
for day in Mon Tue Wed Thu Fri Sat Sun
do
    if [ $i -eq 7 ] || [ $i -eq 8 ]; then
        echo "Weekend : $day"
        continue
    fi
    echo "Weekday $((i++)) : $day"
done
```

### C-Style For Loop

```bash
#!/bin/bash
i=1
j=10
for (( i=1; i<=5; i++ ))
do
    j=$((j+5))
    echo "Number $i : $j"
done
# Output: 15, 20, 25, 30, 35
```

### Predefined Variable — `$RANDOM`

```bash
echo $RANDOM        # prints a random number each time
```

---

## Multiple Users Script

### Two Files Required

1. `userlist.txt` — list of usernames
2. `user.sh` — the script

**userlist.txt:**
```
nurealm
nayeem
imrahman
farid
ruby
sankar
```

**user.sh:**
```bash
#!/bin/bash
for i in `more userlist.txt`
do
    useradd $i
    echo "redhat" | passwd --stdin $i
done
```

> `more userlist.txt` reads the file line by line. Each username is stored in `i`.
> `passwd --stdin` = assign password without interactive prompt. `echo "redhat"` pipes the password in.

Run it:
```bash
chmod +x user.sh
./user.sh
tail /etc/passwd        # verify users created
```

To update the user list: just edit `userlist.txt` and run the script again. No script changes needed.

Clear the file when done:
```bash
> userlist.txt          # empties the file
```

---

## Databases — Overview

| Term | Definition |
|------|-----------|
| Data | Any information — person's details, product info, etc. |
| Database | Structured storage for data (like one Excel sheet) |
| DBMS | Database Management System — manages multiple databases (like Excel managing multiple sheets) |

**Why not just use Excel?** Excel handles up to a few thousand records well. For 1TB+ of data, dedicated database software is required.

**Common DBMS options:**

| Vendor | Product |
|--------|---------|
| Oracle | Oracle DB |
| Microsoft | SQL Server |
| Open source | MySQL, MariaDB, PostgreSQL |
| AWS | Aurora, RDS |
| SAP | SAP HANA |

> MariaDB = advanced version of MySQL. Both are interchangeable.

When you install a database service on a machine → that machine becomes a **database server**.

---

## Installing and Configuring MariaDB

### Step 1 — Create Repository (if no internet subscription)

```bash
vim /etc/yum.repos.d/rhel.repo
```

```
[rhel]
name=rhelx64
baseurl='file:///path/to/image'
gpgcheck=0
enabled=1
```

```bash
yum clean all
yum repolist
```

### Step 2 — Install MariaDB

```bash
yum install mariadb* -y
```

### Step 3 — Start and Enable Service

```bash
systemctl start mariadb
systemctl enable mariadb
```

### Step 4 — Configure (Secure Installation)

```bash
mysql_secure_installation
```

Prompts:
- Enter current root password: (blank — just press Enter)
- Set new root password: yes → enter password twice
- Remove anonymous users: yes
- Disallow root login remotely: yes
- Remove test database: yes (in production) / no (for practice)
- Reload privilege tables: yes

### Log Into Database

```bash
mysql -u root -p        # -u = user, -p = password prompt
```

```sql
show databases;         -- list all databases
```

---

## Database Backup Script

```bash
#!/bin/bash
# Purpose: Database Backup

DB_USER=root
FMT_OPTIONS="--skip-column-names -e"
BACKUP_DIR="/root/db-backup"

# Get filtered DB list (exclude system DBs)
DB_NAME=$(mysql $FMT_OPTIONS "show databases" -u $DB_USER -p | \
    grep -v information_schema | \
    grep -v performance_schema | \
    grep -v "^*")

# Backup each database
for DBNAME in $DB_NAME
do
    echo "Backing up: $DBNAME"
    mysqldump -u $DB_USER $DBNAME > $BACKUP_DIR/$DBNAME.dump
done
```

> `mysqldump` = key command to export a database to a `.dump` file.
> Filter with `grep -v` to exclude system databases (`information_schema`, `performance_schema`).

Setup:
```bash
mkdir /root/db-backup
chmod +x db.sh
./db.sh
ls /root/db-backup      # verify: mysql.dump, test.dump
```

---

## If Statement

### Simple If

```bash
#!/bin/bash
count=100
if [ $count -eq 100 ]
then
    echo "Count is 100"
fi
```

### If-Else

```bash
#!/bin/bash
count=99
if [ $count -eq 100 ]
then
    echo "Count is 100"
else
    echo "Count is not 100"
fi
```

### If-Elif-Else (Ladder)

```bash
#!/bin/bash
count=99
if [ $count -eq 100 ]
then
    echo "Count is 100"
elif [ $count -gt 100 ]
then
    echo "Count is greater than 100"
else
    echo "Count is less than 100"
fi
```

**Comparison operators:**

| Operator | Meaning |
|----------|---------|
| `-eq` | equal to |
| `-ne` | not equal to |
| `-gt` | greater than |
| `-lt` | less than |
| `-ge` | greater than or equal |
| `-le` | less than or equal |

> `elif` = checks conditions one by one. `else` = catches everything not matched above.

---

## While Loop

### Syntax

```bash
while [ condition ]
do
    commands
done
```

> While loop runs as long as the condition is true. Used most often to read file content.

### Basic Example — Counter

```bash
#!/bin/bash
n=1
while [ $n -le 5 ]
do
    echo "Welcome $n times"
    n=$((n+1))              # increment
done
# Output: Welcome 1 times ... Welcome 5 times
```

### Read File Content with While

```bash
#!/bin/bash
while IFS= read -r line
do
    echo "$line"
done < /etc/hosts           # < = input redirector
```

> `IFS` = Internal Field Separator. `read -r line` reads one line at a time.
> `<` = input redirector — feeds file content into the loop.

### Read File and Split Into Fields

```bash
#!/bin/bash
while IFS=: read -r field1 field2
do
    echo "$field1 -- $field2"
done < /etc/hosts
```

### Practical — Filter Users from `/etc/passwd`

```bash
#!/bin/bash
while IFS=: read -r username password uid gid description home shell
do
    if [ $uid -gt 500 ]
    then
        echo "$username"
    fi
done < /etc/passwd
```

> Prints only users with UID > 500 (normal users, not system users).

---

## Key Takeaway

For loop = iterate over a list, files, or C-style range. `*` wildcard captures everything. `awk` filters specific fields from files. Multiple users script = read userlist from file, loop through, create users with password. Database backup = `mysqldump` per database, filtered with `grep -v`. If statement checks conditions — `elif` goes one by one, `else` is the fallback. While loop runs until condition is false — best used to read file content line by line with `IFS`.
