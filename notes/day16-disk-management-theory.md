# Day 16 — Disk Management (Theory)

> Networking concepts start Friday.
> This session is theory only — practical partitioning covered in the next session.

---

## Why Format a Hard Disk?

A brand new hard disk is a **raw disk** — it has no file system and you cannot store data on it directly. Before you can use it, you must **format** it, which means assigning a file system to it.

Formatting = assigning a file system to the raw disk so data can be stored in a structured way.

---

## Windows File Systems

| File System | Era | Notes |
|-------------|-----|-------|
| FAT16 | Pre-XP | File Allocation Table 16-bit — old, slow, no security |
| FAT32 | Pre-XP | File Allocation Table 32-bit — old, slow, no security |
| NTFS | XP onwards | New Technology File System — faster, has file/folder security |
| CDFS | CD/DVD | CD File System |

- **FAT** — slow read/write, no file or folder level security
- **NTFS** — faster read/write, file and folder security introduced here
- From Windows XP onwards, NTFS is the default

After formatting with NTFS, Windows automatically assigns a **drive letter** starting at `C:\` (A and B are reserved for floppy drives).

---

## Why Partition a Hard Disk?

If you store everything (OS + personal data + work data) in a single partition (C drive) and it gets a virus, you have to format the entire drive — losing everything.

**Partitioning** solves this by dividing the disk into separate sections. Each partition is independent — formatting one doesn't affect the others.

---

## Partition Schemas

### MBR — Master Boot Record

```
fdisk       # command to create MBR partitions
```

- Maximum **4 partitions** total
- Up to **3 primary** partitions
- **1 extended** partition (you cannot store data here directly)
- Inside the extended partition, you can create up to **15 logical drives**
- MBR itself is stored in the **first sector of the hard disk**, size = **512 bytes**
- Supported by **BIOS** (older systems)

**Why the limit matters:**

If you create 4 primary partitions using all 1TB, any unused space beyond those 4 is wasted — you cannot access it. To use more than 4 partitions, create 3 primaries + 1 extended, then carve logical drives out of the extended.

**Primary vs Logical:**
- Both can store data
- OS can only be installed on a **primary** partition — not on a logical drive

**Example layout on 1TB disk:**

| Partition | Type | Size | Use |
|-----------|------|------|-----|
| C | Primary | 100 GB | OS |
| D | Primary | 200 GB | Data |
| E | Primary | 200 GB | Data |
| F | Extended | 500 GB | Container only — no data stored here |
| G | Logical | 200 GB | Data |
| H | Logical | 200 GB | Data |

---

### GPT — GUID Partition Table

```
gdisk       # command to create GPT partitions
```

- Up to **128 partitions** — no primary/extended distinction
- Has a **GPT header** and **GPT footer (tailor)** — partitions live in between
- Supported by **UEFI** (modern systems)
- Simpler and more flexible than MBR

---

## MBR vs GPT Summary

| Feature | MBR | GPT |
|---------|-----|-----|
| Command | `fdisk` | `gdisk` |
| Max partitions | 4 (3 primary + 1 extended) | 128 |
| Partition types | Primary, Extended, Logical | All equal |
| Boot firmware | BIOS (older) | UEFI (modern) |
| MBR/Header size | 512 bytes | GPT header + footer |

---

## Lab VM Partition Layout (40 GB disk)

When we installed the OS on our 40GB VM:

| Partition | Size | Type |
|-----------|------|------|
| `/` (root) | 12 GB | Primary |
| `/boot` | 400 MB | Primary |
| swap | 10 GB | Primary |
| `/home` | 8 GB | Logical (inside extended) |
| Unallocated | ~10 GB | Raw — used for disk management demos |

> MBR is created automatically during OS installation — no need to create it manually.

---

## Linux File Systems

Linux needs its own file systems to format disks — equivalent to NTFS on Windows.

### Journaling

**Journal = log of filesystem changes.** If the system crashes or shuts down improperly, the journal lets the OS recover without losing data. This is like the Windows repair feature that recovers missing boot files after an improper shutdown.

### File System Comparison

| File System | Default In | Max File Size | Max FS Size | Journaling |
|-------------|-----------|---------------|-------------|------------|
| EXT2 | Very old (1993) | 2 TB | 32 TB | No |
| EXT3 | 2001 | 2 TB | 32 TB | Yes |
| EXT4 | RHEL 6.0 | 16 TB | 1 EB | Yes (can toggle on/off) |
| XFS | RHEL 7.0+ | 8 EB | 8 EB | Yes |

- **EXT** = Extended File System
- **XFS** = Extents File System (different from EXT)
- XFS is also the default on modern Unix systems
- Max file name length = **255 bytes** (both EXT4 and XFS)
- EXT4 max subdirectories: 64,000 | XFS: unlimited

> EXT2 has **no journaling**. EXT3 **introduced journaling**. If you convert EXT2 → EXT3, you lose data — avoid it.

### Storage Unit Reference

```
1024 MB  = 1 GB
1024 GB  = 1 TB
1024 TB  = 1 PB
1024 PB  = 1 EB
1024 EB  = 1 ZB
1024 ZB  = 1 YB
```

- RHEL 6.0 (EXT4) supports up to **1 EB**
- RHEL 7.0+ (XFS) supports up to **8 EB**

### Assigning a File System (Format Command)

```bash
mkfs.ext4 /dev/sda1     # format partition with EXT4
mkfs.xfs  /dev/sda1     # format partition with XFS
```

---

## Hard Disk Types

### HDD

| Technology | Type | Full Form | Notes |
|------------|------|-----------|-------|
| ATA | IDE / PATA | Integrated Device Electronics / Parallel ATA | Old ribbon cables — obsolete |
| ATA | SATA | Serial Advanced Technology Attachment | Common in older laptops |
| SCSI | — | Small Computer System Interface | Found in servers |

### SSD

| Type | Notes |
|------|-------|
| SATA SSD | Cannot connect directly to motherboard |
| NVMe SSD | Connects directly to motherboard — faster than SATA SSD |

---

## Identifying Hard Disk Type

```bash
fdisk -l        # list partitions and disk info (less detail)
parted -l       # list partitions with type, size in MB/GB (more detail)
```

The device path tells you the disk type:

| Path | Disk Type |
|------|-----------|
| `/dev/hda` | IDE / PATA (very old — rarely seen) |
| `/dev/sda` | SATA, SCSI, or SSD |
| `/dev/vda` | Virtual hard disk (VMware, KVM) |
| `/dev/xvda` | AWS EBS volume (Xen virtual disk) |

> `parted -l` gives more information than `fdisk -l` — shows partition type, flags (boot, LVM), and sizes in human-readable format. Use either depending on what you need.

---

## Key Takeaway

Raw disk → format with a file system → create partitions → install OS on primary. Use MBR for older/BIOS systems (max 4 partitions), GPT for modern/UEFI systems (max 128). XFS is the default and recommended file system for RHEL 7+. Use `/dev/sda` naming to identify disk type — `hda` = IDE, `sda` = SATA/SCSI/SSD, `vda` = virtual.
