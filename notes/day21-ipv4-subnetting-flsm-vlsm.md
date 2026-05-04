# Day 21 — IPv4 Classes Deep Dive, Subnetting, FLSM & VLSM

## Class Properties — Network vs Host Bits

| Class | N (network bits) | H (host bits) | Priority Bit | Bits |
|-------|-----------------|---------------|-------------|------|
| A | 8 | 24 | 0 | 1 bit |
| B | 16 | 16 | 10 | 2 bits |
| C | 24 | 8 | 110 | 3 bits |

**Formula for networks:** `2^(n-p) - 2`
- `n` = network bits
- `p` = priority bit count
- Minus 2 = subtract unknown network (0.0.0.0) and loopback (127.0.0.0) — Class A only

**Formula for hosts:** `2^h - 2`
- Minus 2 = subtract Network ID (NID) and Broadcast ID (BID)

---

## Special Addresses

**0.0.0.0** = unknown network — reserved, not usable

**127.0.0.1** = loopback address — used to test if your NIC card is working
```bash
ping 127.0.0.1      # if no reply, NIC card may need replacing
```

**Network ID (NID)** = all host bits are 0 → first address in a network

**Broadcast ID (BID)** = all host bits are 1 → last address in a network

Example for Class A network `10.0.0.0`:
```
10.0.0.0        → NID (Network ID)
10.0.0.1        → first valid host
10.0.0.2        → second valid host
...
10.0.0.254      → last valid host (in first subnet)
10.255.255.255  → BID (Broadcast ID)
```

---

## Class A — Detailed

**N=8, H=24, Priority=0 (1 bit)**

**Networks:**
```
2^(n-p) - 2 = 2^(8-1) - 2 = 2^7 - 2 = 128 - 2 = 126 networks
```
(Minus 2 for 0.0.0.0 unknown and 127.0.0.0 loopback)

Network range: `0.0.0.0 – 127.0.0.0`

**Hosts per network:**
```
2^h - 2 = 2^24 - 2 = 16,777,216 - 2 = 16,777,214 hosts/network
```

> Class A has over 16 million hosts per network — way too large for any real org. That's why it's not preferred for internal deployment.

---

## Class B — Detailed

**N=16, H=16, Priority=10 (2 bits)**

**Networks:**
```
2^(n-p) = 2^(16-2) = 2^14 = 16,384 networks
```
(No minus 2 here — no unknown or loopback reserved in Class B)

**Hosts per network:**
```
2^h - 2 = 2^16 - 2 = 65,536 - 2 = 65,534 hosts/network
```

> Still too many hosts per network for typical use.

---

## Class C — Detailed

**N=24, H=8, Priority=110 (3 bits)**

**Networks:**
```
2^(n-p) = 2^(24-3) = 2^21 = 2,097,152 networks
```

**Hosts per network:**
```
2^h - 2 = 2^8 - 2 = 256 - 2 = 254 hosts/network
```

> **Class C is preferred for org networks** — manageable size (254 hosts/network), most networks available, and fits typical department/team sizes well.

---

## CIDR — Classless Inter-Domain Routing

CIDR (slash notation) tells you the network size. In real-time environments, you'll see CIDR, not the full subnet mask written out.

`/24` means 24 network bits.

| CIDR | Combinations | Valid Hosts |
|------|-------------|------------|
| /8 | 16,777,216 | ~16.7 million |
| /16 | 65,536 | 65,534 |
| /24 | 256 | 254 |
| /25 | 128 | 126 |
| /26 | 64 | 62 |
| /27 | 32 | 30 |
| /28 | 16 | 11 (AWS: 16-5=11) |

Default subnet mask written in binary then decimal:
```
11111111.11111111.11111111.00000000 → 255.255.255.0 = /24
```

---

## Subnetting

### What is Subnetting?

Dividing one network into multiple smaller networks.

**Technical definition:** Converting host bits into network bits.

### Why Subnet?

If IT (100 hosts) and BPO (100 hosts) are on the same network `192.168.1.0/24`, they can communicate freely — which may not be desired. Assigning them separate networks (two Class C networks) solves the isolation problem, but wastes ~150 IP addresses per network.

Subnetting fixes this — divide one network into two, each sized to fit actual needs.

### The Knife Analogy

Default subnet mask `255.255.255.0` is the boundary. To cut the network, you need a **customized subnet mask** — the knife.

Converting the first host bit to a network bit:
```
Bit value = 2^7 = 128
New subnet mask = 255.255.255.128   →   /25
```

`/25` = 128 combinations per subnet. Each subnet: 128 - 2 = **126 valid IPs**.

---

## FLSM — Fixed Length Subnet Mask

All subnets are the same size. Equal cut.

**Scenario:** IT dept = 100 hosts, BPO dept = 100 hosts. Network: `192.168.1.0/24`

Cut the apple in half → `/25` → 2 subnets of 126 valid IPs each.

| Subnet | Range | Subnet Mask | CIDR | Valid IPs |
|--------|-------|-------------|------|-----------|
| IT | 192.168.1.0 – 192.168.1.127 | 255.255.255.128 | /25 | 126 |
| BPO | 192.168.1.128 – 192.168.1.255 | 255.255.255.128 | /25 | 126 |

- `192.168.1.0` = NID for IT subnet
- `192.168.1.127` = BID for IT subnet
- `192.168.1.128` = NID for BPO subnet
- `192.168.1.255` = BID for BPO subnet

Wasted IPs: 126 - 100 = 26 per subnet (much better than 150+ wasted before).

---

## VLSM — Variable Length Subnet Mask

Different subnets get different sizes based on actual need. More efficient than FLSM.

**Scenario:** CTS org, `192.168.1.0/24`
- IT dept: 100 hosts
- Accounts dept: 50 hosts
- HR dept: 20 hosts

**Strategy:** Largest first → work down.

### Step 1 — IT (need 100)

Convert 1st host bit → network bit. Bit value = 128.
```
Subnet mask: 255.255.255.128   /25
Combinations: 128 → valid: 126
```

| | Value |
|-|-------|
| NID | 192.168.1.0 |
| BID | 192.168.1.127 |
| Valid hosts | 192.168.1.1 – 192.168.1.126 (126 IPs) |
| Assigned to | IT |

### Step 2 — Accounts (need 50)

Remaining range: `192.168.1.128 – 192.168.1.255` (128 combinations). Cut in half again → convert 2nd host bit.

Bit value = 2^6 = 64. Subnet mask = 128 + 64 = **192**.
```
Subnet mask: 255.255.255.192   /26
Combinations: 64 → valid: 62
```

| | Value |
|-|-------|
| NID | 192.168.1.128 |
| BID | 192.168.1.191 |
| Valid hosts | 192.168.1.129 – 192.168.1.190 (62 IPs) |
| Assigned to | Accounts |

### Step 3 — HR (need 20)

Remaining range: `192.168.1.192 – 192.168.1.255` (64 combinations). Cut in half → convert 3rd host bit.

Bit value = 2^5 = 32. Subnet mask = 128 + 64 + 32 = **224**.
```
Subnet mask: 255.255.255.224   /27
Combinations: 32 → valid: 30
```

| | Value |
|-|-------|
| NID | 192.168.1.192 |
| BID | 192.168.1.223 |
| Valid hosts | 192.168.1.193 – 192.168.1.222 (30 IPs) |
| Assigned to | HR |

### VLSM Summary

```
11111111.11111111.11111111.10000000 → 255.255.255.0

192.168.1.0   – 192.168.1.127   → 255.255.255.128 → IT    /25
192.168.1.128 – 192.168.1.191   → 255.255.255.192 → Acc   /26
192.168.1.192 – 192.168.1.223   → 255.255.255.224 → HR    /27
```

---

## AWS Subnetting Rules

AWS does not follow legacy CIDR rules exactly. AWS reserves **5 IPs per subnet** (not 2), so formula is `combinations - 5`.

| Legacy Class | Legacy CIDR | AWS CIDR Range |
|-------------|-------------|----------------|
| A | /8 | /16 – /28 |
| B | /16 | /16 – /28 |
| C | /24 | /16 – /28 |

AWS minimum = `/16` (65,536 - 5 = 65,531 usable)
AWS maximum = `/28` (16 - 5 = 11 usable)

**AWS subnet sizing reference:**

| CIDR | Combinations | Usable (minus 5) |
|------|-------------|-----------------|
| /16 | 65,536 | 65,531 |
| /24 | 256 | 251 |
| /25 | 128 | 123 |
| /26 | 64 | 59 |
| /27 | 32 | 27 |
| /28 | 16 | 11 |

**Recommended VPC/subnet pattern:**
- VPC: `/16` (large enough to carve subnets from)
- Subnets: `/24` or smaller depending on need

> Don't blindly assign VPC `/16` and subnet `/16` — you can't fit a 4,000 sq ft classroom inside another 4,000 sq ft space. The subnet must be smaller than the VPC.

---

## Key Takeaway

Class C (254 hosts/network) is preferred for org deployments. FLSM cuts all subnets equally — simple but wastes IPs. VLSM cuts subnets by actual need — efficient, preferred in production. CIDR tells you network size: `/25` = 128 combinations, `/26` = 64, `/27` = 32. In AWS, subtract 5 instead of 2, and CIDR range is `/16` to `/28` regardless of class.
