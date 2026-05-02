# Day 20 — Networking: Switch, Router & IPv4

## Switch

### Properties
| Feature | Detail |
|---------|--------|
| Intelligence | Intelligent device |
| Communication (first time) | Flooding — one-to-many excluding source |
| Communication (after) | Unicast — one-to-one |
| Bandwidth | Fixed per port — not shared |
| Duplex | Full duplex |
| Key feature | Maintains a MAC address table |

### How a Switch Works

When Machine 1 sends data to Machine 4 for the first time:
1. Switch receives the data and captures Machine 1's MAC address → stores it in the MAC table
2. Switch sends data to Machine 2, 3, and 4 (everyone except source) — this is **flooding**
3. Machine 4 responds → switch captures Machine 4's MAC address → stores it
4. From now on, switch sends directly Machine 1 ↔ Machine 4 — this is **unicast**

> First time = flooding. Every time after = unicast. This is what makes a switch intelligent — Hub broadcasts every single time.

### Bandwidth

If you connect 40Mbps to a switch with 4 hosts, each host gets **40Mbps**. Switch doesn't split bandwidth — each port has its own fixed allocation. Hub would give each host only 10Mbps.

### Types of Switch

| Type | Use Case | Notes |
|------|----------|-------|
| Manageable | Data centers, enterprise | Configurable — port security, bandwidth, VLANs. Expensive. |
| Unmanageable | Small offices | Plug and play. No configuration. Cheaper. |

---

## Communication Types (Summary)

| Type | Description | Example |
|------|-------------|---------|
| Broadcast | One-to-many including source | Hub |
| Flooding | One-to-many excluding source | Switch (first time) |
| Unicast | One-to-one | Switch (after MAC learned) |
| Multicast | One-to-group | Class D IP range |

---

## Router

### Properties
- Intelligent device
- Maintains a **route table**
- Enables communication between two **different logical networks**
- Determines the **best/shortest path** to send data

### Physical vs Logical Network

A physical network = cables + switches + machines. But physical alone is useless without logical addressing (IP addresses).

> MAC address = postal address (physical). IP address = phone number (logical). Without a phone number, you can't call someone. Without an IP address, you can't communicate across networks.

Two different networks (e.g. `192.168.1.0/24` and `10.0.0.0/8`) cannot communicate by default. A router connects them — it's the door between two classrooms.

### Path Determination

When data needs to travel from Hyderabad network → Chennai network through multiple routers, the router decides the shortest path using **routing protocols**:

| Protocol | Full Name | How it Picks Best Path |
|----------|-----------|----------------------|
| RIP | Routing Information Protocol | Least hop count — outdated, max 99 routers |
| EIGRP | Enhanced Interior Gateway Routing Protocol | Best bandwidth path |
| OSPF | Open Shortest Path First | Successor to RIP — overcomes RIP's 99-router limit |

**Routing protocols** = decide the best path.
**Routed protocols** = carry the actual data between hosts (IP, IPX for IBM, AppleTalk for Apple). Nowadays almost everyone uses IP.

---

## IPv4

### Why IP Addresses?

Every device needs a unique logical address (IP) to communicate — like a phone number. Smart TVs, CCTV cameras, mobiles, IoT devices all need IPs.

### IPv4 vs IPv6

| | IPv4 | IPv6 |
|--|------|------|
| Bits | 32 | 128 |
| Formula | 2³² | (2³²)⁴ |
| Total addresses | ~4.29 billion | 4× more than IPv4 |
| Status | Running out | Introduced to replace IPv4 |

IPv4's ~4 billion addresses ran out because of the explosion of smart devices. IPv6 (128-bit) was introduced. However, countries like India still use IPv4 via **NAT**.

**NAT (Network Address Translator)** — with 1 public IP address, you can communicate with up to 60,000 private hosts. Your home Wi-Fi router uses NAT by default.

### Who Manages IP Addresses?

```
ICANN (Internet Corporation for Assigned Names and Numbers)
  └── IANA (Internet Assigned Numbers Authority)
        └── RIR (Regional Internet Registry)
              └── ISP (Jio, Airtel, Hathway, etc.)
                    └── Customer / End User
```

---

## IPv4 Structure

IPv4 = 32 bits = 4 octets, each octet = 8 bits, separated by dots.

```
10100011 . 10001101 . 11100101 . 11000011
   ↓           ↓          ↓          ↓
 octet 1    octet 2    octet 3    octet 4
```

Each octet: 2⁸ = 256 combinations → range **0 to 255**

That's why an IP looks like `192.168.1.11` — four decimal values, each 0–255.

---

## IP Address Classification

0–255 is still a huge range, so IANA divided it into 5 classes based on **priority bits** (most significant bits of the first octet):

| Class | Range | Priority Bit | Use |
|-------|-------|-------------|-----|
| A | 0–127 | 0 | LAN/WAN |
| B | 128–191 | 10 | LAN/WAN |
| C | 192–223 | 110 | LAN/WAN |
| D | 224–239 | 1110 | Multicast |
| E | 240–255 | 1111 | Research & Development |

As system/cloud/network engineers, we only deal with **Class A, B, C**.

### How Class Ranges Are Derived (The Formula)

Each bit in an octet has a fixed value (right to left):

```
Bit:    8    7    6    5    4    3    2    1
Value: 2⁷  2⁶  2⁵  2⁴  2³  2²  2¹  2⁰
      128   64   32   16    8    4    2    1
```

Using priority bits:
- **Class A** priority bit = `0` → 1 bit → 2⁷ = 128 combinations → range **0 to 127**
- **Class B** priority bit = `10` → 2 bits → 128+64 = 192 → start from 128, end at **191** (minus 1)
- **Class C** priority bit = `110` → 3 bits → 128+64+32 = 224 → start from 192, end at **223**
- **Class D** priority bit = `1110` → 128+64+32+16 = 240 → start from 224, end at **239**
- **Class E** → 240 to 255

---

## Network vs Host Positions

| Class | Structure | Default Subnet Mask |
|-------|-----------|-------------------|
| A | N.H.H.H | 255.0.0.0 |
| B | N.N.H.H | 255.255.0.0 |
| C | N.N.N.H | 255.255.255.0 |

**Subnet mask rule:** All network bits → 1s. All host bits → 0s.

Example Class A: `11111111.00000000.00000000.00000000` = `255.0.0.0`

The subnet mask is the **boundary** between classes — like a wall between classrooms. Without it, networks bleed into each other.

---

## Private IP Ranges

These ranges are reserved for internal/private networks (intranet). Everything outside these is public (internet).

| Class | Private Range |
|-------|--------------|
| A | 10.0.0.0 – 10.255.255.255 |
| B | 172.16.0.0 – 172.31.255.255 |
| C | 192.168.0.0 – 192.168.255.255 |

> **Class C is preferred for organization networks** — smaller, more controlled. Class A has too many hosts per network for typical use.

```bash
# Check public IP
# Open browser → search "what is my IP"

# Check private IP (Linux)
ip addr
ifconfig
```

---

## Key Takeaway

Switch = intelligent, uses MAC table, fixed bandwidth per port, full duplex. Router = connects different logical networks, uses routing protocols (RIP/EIGRP/OSPF) to find the best path. IPv4 = 32-bit, 4 octets, 0–255 each, divided into 5 classes by priority bits. Classes A, B, C for networking. Class C preferred for private org networks. NAT lets one public IP serve up to 60,000 private hosts — that's how IPv4 is still in use today.
