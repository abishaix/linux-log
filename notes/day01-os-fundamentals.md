# Day 1 — OS Fundamentals & Linux Introduction

## What is an OS?

An OS (Operating System) is an interface between the user and the computer hardware. It's a program that translates between low-level (binary/machine) language and high-level (human-readable) language.

At the lowest level, computers only understand binary — `0`s and `1`s (bits). The OS handles that translation so we don't have to.

---

## Types of OS

| Type | Description | Example |
|------|-------------|---------|
| Single User – Single Tasking | One user, one task at a time | MS-DOS |
| Single User – Multi Tasking | One user, multiple tasks | MS Windows |
| Multi User – Multi Tasking | Multiple users, multiple tasks | Unix, Linux, macOS |

---

## Unix vs Linux History

**Unix**
- Created in 1969 at Bell Labs (AT&T)
- CLI-based
- Vendor-proprietary OS (not free)
- Spawned vendor-specific versions: IBM → AIX, HP → HP-UX
- Unix-Like family

**Linux**
- Created by **Linus Torvalds** in 1991 (started ~1990)
- CLI & GUI (also TUI — Text User Interface)
- Open Source — free to use and distribute
- A clone/alternative to Unix
- 800+ distributions exist

> Unix is proprietary. Linux is the open-source answer to Unix.

---

## Interfaces

| Interface | Description |
|-----------|-------------|
| GUI | Graphical User Interface — visual, mouse-driven |
| CLI | Command Line Interface — text-based |
| TUI | Text User Interface — text-based but with some visual structure |

---

## Ports (quick reference)

- Total port range: `0 – 65535`
- Well-known/system ports: `1 – 1023`
- Registered/dynamic ports: `1024 – 65535`

---

## Packages

- Windows uses `.exe` installers
- Linux uses `.rpm` (Red Hat Package Manager) and others
- **RHEL (Red Hat Enterprise Linux)** — paid (~$40)
- **CentOS** — free RHEL clone
- Closed source (Windows) vs Open Source (Linux)

---

## Key Takeaway

Linux is free, open source, and runs on CLI (and GUI). It's a multi-user, multi-tasking OS — that's why it dominates servers, cloud, and enterprise environments.
