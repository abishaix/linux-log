# Day 18 — Logical Volume Management (LVM)

## Why LVM?

With standard partitions, you can't resize. A 1TB partition stays 1TB — you can't extend it without buying new hardware, transferring all data, and losing downtime.

**LVM solves this:** you can increase or reduce a volume's size without losing data or taking downtime. In production, most storage infrastructure uses LVM for exactly this reason.

---

## LVM Architecture

```
Physical Disks / Partitions
        ↓
  Physical Volumes (PV)
        ↓
   Volume Group (VG)    ← pool of all PVs
        ↓
  Logical Volumes (LV)  ← what you actually format and mount
```

**Physical Volume (PV)** — a standard partition converted to LVM format. Acts as the raw input.

**Volume Group (VG)** — a pool that combines one or more PVs. Think of it as a total storage bucket.

**Logical Volume (LV)** — carved out of the VG. This is what you format and mount. You can resize LVs freely within the VG's total size.

---

## How the OS Already Uses LVM

When you installed RHEL with the LVM mount point option, it automatically created:

```
/dev/sda1    400MB   Primary    /boot  (standard partition)
/dev/sda2    30GB    Primary    VG: rhel (LVM)
                                  ├── /        12GB
                                  ├── swap      10GB
                                  └── /home      8GB
```

```bash
lsblk       # list block devices — shows the full tree: disk → partition → VG → LV → mount point
```

---

## LVM Formula

```
PE size × LE count = LV size
```

- **PE** = Physical Extent (default 4MB — must be a power of 2: 4, 8, 16, 32, 64...)
- **LE** = Logical Extent

Example: PE 4MB × LE 100 = **400MB** logical volume.

---

## Creating LVM — Step by Step

### 1. Create the partition

```bash
fdisk /dev/sda
# n → new partition
# p → primary
# first sector: accept default
# last sector: +2G
# t → change type to 8e (Linux LVM)
# w → save
partprobe /dev/sda
```

### 2. Create Physical Volume

```bash
pvcreate /dev/sda3

pvs                     # list all PVs
pvdisplay /dev/sda3     # detailed info: size, UUID, VG assignment
```

### 3. Create Volume Group

```bash
vgcreate my_vg /dev/sda3

vgs                     # list all VGs
vgdisplay my_vg         # detailed info: PE size, total size, free size
```

> Default PE size is 4MB. Change with `-s`: `vgcreate -s 16 my_vg /dev/sda3`

### 4. Create Logical Volume

Use lowercase `-l` for LE count, uppercase `-L` for direct size.

```bash
lvcreate -L 400M -n my_lv my_vg     # create 400MB LV named my_lv
# OR
lvcreate -l 100 -n my_lv my_vg      # 100 LEs × 4MB PE = 400MB

lvs                     # list all LVs
lvdisplay               # detailed info including path, size, current LE
```

### 5. Format and Mount

```bash
mkfs -t ext4 /dev/my_vg/my_lv       # format with EXT4 (supports both extend and reduce)

mkdir /mnt/mydata
mount /dev/my_vg/my_lv /mnt/mydata
```

> Use **EXT4** for LVM work when you need to reduce — XFS supports extend only.

### 6. Make Permanent

```bash
vim /etc/fstab
# Add:
/dev/my_vg/my_lv    /mnt/mydata    ext4    defaults    0    0

mount -a
```

---

## Extending a Logical Volume

```bash
lvextend -L +200M /dev/my_vg/my_lv      # add 200MB to the LV
resize2fs /dev/my_vg/my_lv              # resize the EXT4 file system to match
# For XFS use: xfs_growfs /dev/my_vg/my_lv

df -Th      # verify new size
```

> `lvextend` resizes the LV block. `resize2fs` / `xfs_growfs` tells the file system to use the new space. Both steps are required.

---

## Reducing a Logical Volume (EXT4 only)

XFS cannot be reduced. EXT4 can — but follow this exact order to avoid data loss:

```bash
umount /dev/my_vg/my_lv                 # 1. unmount
e2fsck -f /dev/my_vg/my_lv             # 2. check and organize the file system
resize2fs /dev/my_vg/my_lv 400M        # 3. resize file system DOWN to target size first
lvreduce -L -200M /dev/my_vg/my_lv     # 4. then reduce the LV
mount -a                                # 5. remount
```

> Think of it like marking a loaf of bread before cutting — you resize the file system first (mark the cut line), then remove the physical space.

---

## Extending a Volume Group

If your LV needs more space than the VG has, add another PV to the VG:

```bash
vgextend my_vg /dev/sda5        # add sda5 to the VG (auto-converts to PV)
pvs                             # verify: sda3 and sda5 both belong to my_vg
vgdisplay my_vg                 # verify: total size increased
```

---

## Migrating Data Between PVs (pvmove)

If you want to remove a PV from a VG but it's in use, migrate the LVM data off it first:

```bash
pvmove /dev/sda3 /dev/sda5      # move all LVM data from sda3 to sda5 (no downtime)
pvs                             # verify sda3 is now free
```

---

## Reducing a Volume Group

After migrating data off a PV, remove it from the VG:

```bash
vgreduce my_vg /dev/sda3        # remove sda3 from the VG
vgs                             # verify VG size decreased
```

---

## Deleting LVM — Reverse Process

```bash
umount /dev/my_vg/my_lv
vim /etc/fstab          # remove the entry
lvremove /dev/my_vg/my_lv
vgremove my_vg
pvremove /dev/sda3
fdisk /dev/sda          # delete the partition, then 'w' to save
partprobe /dev/sda
```

---

## LVM Command Reference

| Object | Create | Status | Detailed | Remove |
|--------|--------|--------|----------|--------|
| PV | `pvcreate` | `pvs` | `pvdisplay` | `pvremove` |
| VG | `vgcreate` | `vgs` | `vgdisplay` | `vgremove` |
| LV | `lvcreate` | `lvs` | `lvdisplay` | `lvremove` |

---

## Key Takeaway

LVM = flexible storage. PV → VG → LV is the flow. Always format and mount the **LV**, not the raw partition. Use EXT4 when you need to reduce. `pvmove` lets you migrate data between physical volumes with zero downtime — that's why production environments use LVM.
