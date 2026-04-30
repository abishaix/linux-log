# Day 14 — Disk Partitioning, Mount Points & Swap

## Disk Partitioning with `fdisk`

`fdisk` is the command for creating and managing MBR partitions.

```bash
fdisk -l                    # list all disks and their partitions
fdisk /dev/sda              # open interactive partition manager for sda
```

Inside `fdisk`, key options:
- `m` — help (list all options)
- `p` — print current partition table
- `n` — create new partition (primary or extended)
- `t` — change partition type (e.g. Linux → Linux Swap)
- `d` — delete a partition
- `w` — write changes and exit
- `L` — list hex codes for partition types

### MBR Partition Schema — Rules
- Max 4 primary partitions
- To go beyond 4: replace one primary with an **extended** partition
- Extended partition acts as a container for **logical drives** (SDA5, SDA6, etc.)
- Logical drives are what you actually format and use for storage

### Partition Types — Hex Codes
| Partition Type | Hex Code |
|----------------|----------|
| Linux | 83 |
| Linux Swap / Solaris | 82 |

---

## Understanding Sectors

A hard disk platter is divided into **tracks** (concentric circles), and each track is divided into **sectors** (triangular slices of the circle). Each sector = 512 bytes.

Partitions are defined by sector ranges — start sector to end sector. When creating a partition:
- First sector: accept the default (starts right after the previous partition ends)
- Last sector: define the size using `+2G`, `+500M`, etc.

---

## Updating the Kernel's Partition Table

After creating new partitions, the kernel doesn't automatically know about them. Two options — no reboot required:

```bash
partprobe /dev/sda      # request kernel to re-read partition table (safe)
kpartx /dev/sda         # force kernel update (use with caution — can crash file system)
```

> In production: don't reboot without approval from senior admin. Rebooting takes down services and affects customers. Use `partprobe` instead.

Verify with:
```bash
fdisk -l                # 6 partitions visible
parted -l               # detailed view including file system type
```

---

## Formatting a Partition

A partition is just raw disk space — you can't store data until you assign a file system.

```bash
mkfs -t xfs /dev/sda3       # format sda3 with XFS file system
blkid /dev/sda3             # verify: shows UUID and file system type
```

> `/dev/sda3` is a **block file**, not a directory. You can't `cd` into it.

---

## Mount Points

To access a partition, you mount it to a directory. That directory becomes the entry point for reading and writing to that partition.

```bash
# Temporary mount (lost on reboot):
mount /dev/sda3 /mnt

# Or use a custom directory:
mkdir /home/student/Desktop/dir1
mount /dev/sda3 /home/student/Desktop/dir1
```

Verify:
```bash
df -Th              # disk free space — human readable, shows file system type and mount point
```

> The mounted partition appears like a standalone hard drive on the filesystem. Data written here goes to `/dev/sda3`.

---

## Permanent Mount Points — `/etc/fstab`

A `mount` command creates a **temporary** mount — it's gone after reboot. To make it permanent, add an entry to `/etc/fstab`.

```bash
vim /etc/fstab
```

### `/etc/fstab` Entry Format

```
<device>    <mount-point>    <fs-type>    <options>    <dump>    <check>
```

Example:
```
/dev/sda3    /home/student/Desktop/dir1    xfs    defaults    0    0
```

| Field | Value | Notes |
|-------|-------|-------|
| device | `/dev/sda3` | partition name |
| mount point | `/home/student/Desktop/dir1` | target directory |
| file system | `xfs` | assigned file system type |
| options | `defaults` | system default r/w permissions (includes ACLs from RHEL 7+) |
| dump | `0` | 0 = disable backup dump |
| check sequence | `0` | 0 = disable fsck on boot (1 = enable, 2 = check later) |

Activate without rebooting:
```bash
mount -a        # mounts all entries in /etc/fstab
```

---

## Deleting a Partition

Reverse process:

```bash
vim /etc/fstab              # remove the entry
umount /dev/sda3            # unmount
mount -a                    # reload fstab
fdisk /dev/sda              # enter fdisk, use 'd' to delete the partition, 'w' to save
partprobe /dev/sda          # update kernel
```

---

## Swap Partition

Swap is virtual RAM — when physical RAM fills up, the OS uses swap to maintain performance.

### Check Current Swap and RAM

```bash
cat /proc/meminfo       # detailed RAM and swap info
free -m                 # RAM and swap in MB (use -h for human readable)
swapon -s               # swap partition status
```

### Create an Additional Swap Partition

1. Create a new partition with `fdisk`
2. Change its type to Linux Swap (hex `82`) using `t` inside fdisk
3. Format it as swap:

```bash
mkswap /dev/sda3        # assign swap file system
swapon /dev/sda3        # activate swap
swapon -s               # verify — shows both existing and new swap
```

### Make Swap Permanent — `/etc/fstab`

```
/dev/sda3    swap    swap    defaults    0    0
```

Activate:
```bash
swapon -a               # activate all swap entries in fstab
```

To remove swap:
```bash
swapoff /dev/sda3       # deactivate
vim /etc/fstab          # remove entry
fdisk /dev/sda          # delete the partition
```

---

## Key Takeaway

Standard partitions follow this workflow: create with `fdisk` → format with `mkfs` → mount to a directory → make permanent via `/etc/fstab`. Swap follows the same pattern but uses `mkswap` and `swapon` instead of `mkfs` and `mount`. Never reboot production servers without approval — use `partprobe` to update the kernel instead.
