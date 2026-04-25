# Day 7 — man Pages & Redirectors

## `man` — Manual Pages

Built-in documentation for every command.

```bash
man passwd              # manual for passwd
man -k passwd           # search for "passwd" across all sections
man 5 passwd            # open section 5 specifically (file formats)
man -k tar              # find which sections tar appears in
man 5 tar               # open tar in section 5
man 1 tar               # open tar in section 1 (user commands)
man -k .                # list ALL man page entries
man -k . | wc -l        # count total man pages available
man man                 # man page for man itself
```

### Man Page Sections
Man pages are organized into numbered sections. `man -k` shows the section ID, then you use the number to open the right one.

| Section | Content |
|---------|---------|
| 1 | User commands |
| 5 | File formats and conventions |
| 8 | System administration commands |

---

### Printing man Pages

You can't print directly — convert to PostScript (`.ps`) first:

```bash
man -t passwd > passwd.ps       # convert to PostScript
file passwd.ps                  # verify: "PostScript document"
evince passwd.ps                # open in GUI viewer (needs desktop)
lp passwd.ps -P 1-4             # print pages 1–4 (lp = line print)
```

> `.ps` = PostScript. Not human-readable with `cat` or `vim`.

---

## File Descriptors

The kernel assigns a number to every open file — this is the **file descriptor**.

```
                    Kernel
                   ┌──────────────────────────────┐
Keyboard ──────▶   │  0 – stdin    <              │
                   │  1 – stdout   >  >>          │  ──▶  Monitor
                   │  2 – stderr   2>             │  ──▶  Monitor
                   │  3–n – open files            │
                   └──────────────────────────────┘
```

| Descriptor | Name | Default device | Symbol |
|------------|------|----------------|--------|
| `0` | stdin | Keyboard | `<` |
| `1` | stdout | Monitor | `>` or `>>` |
| `2` | stderr | Monitor | `2>` |
| `3–n` | open files | — | — |

Descriptors 0, 1, 2 are assigned at boot time. Any file you open during a session gets the next available number (3, 4...).

> stdout and stderr both go to the monitor by default — but they're separate streams. That's why you can redirect them independently.

---

## Redirectors

### Output Redirect `>`

```bash
ls -l > /home/student/Desktop/file1                 # save output to file (overwrites)
echo "This is a test" > /var/www/html/index.html    # write to web server index
ls -la >> /home/student/Desktop/file1               # append output to file
```

### Error Redirect `2>`

```bash
ls -l 2> /home/student/Desktop/file2               # redirect errors to file
find / -type b 2>> /home/student/Desktop/file2     # append errors
find / -type b 2> /dev/null                        # discard all errors silently
```

> `/dev/null` = the black hole. Anything redirected here is silently discarded.

### Redirect Output and Errors Separately

```bash
find / -type b > /home/student/Desktop/output.txt 2> /home/student/Desktop/err.txt
```

### Redirect Both to the Same File

```bash
find / -type b &> /home/student/Desktop/file4      # stdout + stderr → same file
find / -type c &>> /home/student/Desktop/file4     # append both
```

### Input Redirect `<`

```bash
cat < file1     # read from file instead of keyboard
```

---

## `journalctl` — View System Logs

```bash
journalctl                      # view all system logs (systemd journal)
journalctl > log.txt            # dump logs to file
vim log.txt                     # open and read
```

---

## Key Takeaway

Redirectors control data flow. You'll use `>`, `>>`, `2>`, and `/dev/null` constantly — especially when scripting, debugging services, or running `find` and `grep` on the whole filesystem where permission errors flood the output.
