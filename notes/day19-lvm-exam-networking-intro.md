# Day 19 — LVM Exam Scenario & Networking Introduction

## LVM — Exam Scenario Walkthrough

Exam questions won't always give you a direct size. They'll give you PE size and LE count — you calculate the LVM size yourself.

**Formula:** `PE size × LE count = LVM size`

**Example question:**
> Create a VG called `redhat` with PE size 16MB. Create an LV called `fedora` with 15 LEs. Assign EXT4 file system and mount on `/test`.

16 × 15 = **240MB** — that's your LVM size. Create the partition slightly larger (e.g. 280MB).

---

### Step-by-Step Solution

**1. Create the partition**

```bash
fdisk /dev/sda
# n → new partition
# p → primary
# first sector: accept default
# last sector: +280M
# t → change type to 8e (Linux LVM)
# w → save
partprobe /dev/sda
```

**2. Create Physical Volume**

```bash
pvcreate /dev/sda3
```

**3. Create Volume Group with 16MB PE size**

Default PE size is 4MB. Use `-s` to change it. Must be a power of 2: 4, 8, 16, 32, 64...

```bash
vgcreate -s 16 redhat /dev/sda3
vgdisplay redhat        # verify PE size is now 16MB
```

**4. Create Logical Volume with 15 LEs**

Use lowercase `-l` for LE count, not `-L` (which takes a direct size).

```bash
lvcreate -l 15 -n fedora redhat
lvdisplay               # verify: /dev/redhat/fedora, size 240MB
```

**5. Format and mount**

```bash
mkfs -t ext4 /dev/redhat/fedora

mkdir /test

vim /etc/fstab
# Add: /dev/redhat/fedora  /test  ext4  defaults  0  0

mount -a
df -Th                  # verify mount
```

---

## Networking — Introduction

### What is a Network?

**Network** = connection between two or more network devices (PCs, switches, routers, load balancers, etc.)

**Networking** = communication between two or more interconnected devices — request and response between them.

**Internetwork** = connection between two or more separate networks (e.g. 3rd floor LAN and 4th floor LAN connected by a cable).

**Internetworking** = communication between two or more interconnected networks.

---

### Why Deploy a Network?

Without networking, there is no IT infrastructure — regardless of role (sysadmin, developer, cloud engineer, DBA).

If you have 100 machines and need to share a project file — without a network, you're copying to USB drives one by one. Days of work. With a network, seconds.

---

### Types of Networks

| Type | Full Name | Scope |
|------|-----------|-------|
| LAN | Local Area Network | Within a building or campus |
| MAN | Metropolitan Area Network | Spans a city (e.g. two offices in different districts) |
| WAN | Wide Area Network | Global — best example is the internet |
| PAN | Personal Area Network | Falls under LAN |

---

### Pre-requirements to Deploy a Network

To set up a network for any project, you need 5 things:

1. NIC cards
2. Media (cables or wireless)
3. Network devices
4. Protocols (TCP, UDP, etc.)
5. Logical addressing (IP addresses)

---

## 1. NIC Card — Network Interface Card

Interface between a device and the network. Without it, a device cannot join the network — physical or virtual (EC2 instances have virtual NICs too).

**Connectors:**
- **RJ45** (Registered Jack 45) — for network/internet. Built into most modern laptops.
- **RJ11** — for phone lines (narrower, fewer pins)

**MAC Address:**
- 48-bit hexadecimal address (0–9, A–F)
- Also called: hardware address, physical address, Ethernet address

```bash
# Windows
getmac                  # MAC address only
ipconfig /all           # full NIC details including MAC and IP

# Linux/macOS
ifconfig                # shows link/ether = MAC address
ip addr                 # newer equivalent
```

If no OS: check BIOS → System Info.

**NIC Types by Speed:**

| Type | Speed |
|------|-------|
| Ethernet | 10 Mbps |
| Fast Ethernet | 100 Mbps |
| Gigabit Ethernet | 1000 Mbps |

> If you're subscribed to 1Gbps but only getting 100Mbps, check whether your NIC is Fast Ethernet or Gigabit Ethernet. Laptops can't be upgraded — desktops can.

---

## 2. Media

### Guided (Wired)

**Coaxial** — used for cable TV. Carries analog signal. Needs a modem/router to convert analog → digital for internet.

**Twisted Pair — UTP and STP**
- 4 pairs, 8 wires
- **UTP** (Unshielded Twisted Pair) — no internal wrapper, cheaper, most common
- **STP** (Shielded Twisted Pair) — has internal wrapper, more durable, slightly more expensive
- Carries data as electrical signals (bits → electrical volts)

**Fiber Optical**
- Carries data as light (bits → light pulses) — fastest medium
- Modern routers have a direct fiber port; older routers need an RJ45-to-fiber converter

### Unguided (Wireless)

- **RF (Radio Frequency)** — Wi-Fi routers, walkie-talkies
- **Infrared** — TV remotes, AC remotes (predecessor to Bluetooth)

> **Wired > Wireless for speed.** Wired = full duplex. Wireless = half duplex. A direct cable connection always outperforms Wi-Fi on the same ISP plan.

---

## 3. Network Devices

### Hub

| Property | Detail |
|----------|--------|
| Intelligence | Not intelligent |
| Works with | Bits |
| Communication | Broadcast — sends to ALL connected devices including source |
| Bandwidth | Shared equally across all ports |
| Duplex | Half duplex |

If you have 40Mbps and 4 machines connected to a hub, each gets 10Mbps. Hub is obsolete — nobody uses it today, but understanding it explains why switches were invented.

### Communication Types

| Type | Description | Example |
|------|-------------|---------|
| Simplex | One-way only | TV remote |
| Half duplex | Two-way, but not simultaneously | Walkie-talkie |
| Full duplex | Two-way simultaneously | Mobile phone call |

---

## Coming Next

- Switch, Router, Firewall, Load Balancer
- Protocols
- IPv4 addressing and subnetting

---

## Key Takeaway

LVM exam questions give PE size and LE count — multiply them to get the size, then work through PV → VG → LV → format → fstab. For networking: a network is connected devices communicating. The five pre-requirements (NIC, media, devices, protocols, IP) are the building blocks of every network — cloud VPCs included.
