# Day 22 — Networking Commands & IP Address Management

## Interface Commands

### `ifconfig` — Interface Configuration

```bash
ifconfig            # show all network interfaces
```

Key output fields:

| Field | Meaning |
|-------|---------|
| `eno...` | Virtual NIC card name |
| `UP` | Interface is working |
| `MTU` | Maximum Transmission Unit — default 1500, minimum 64 bytes |
| `ether` | MAC address (Ethernet/hardware/physical address) |
| `RX packets` | Received (incoming) packets |
| `TX packets` | Transferred (outgoing) packets |
| `collisions` | When two systems try to communicate at the same time — data collision |

> `ifconfig` works on any RHEL version, only the UI output differs slightly.

### `ip addr` — Modern Alternative

```bash
ip addr             # same output as ifconfig, newer command
ip addr show eno16777736    # filter one specific interface
```

> `ip addr` may not work on older versions — use `ifconfig` there.

### `lo` — Loopback Interface

Loopback = the machine talking to itself. Used to test if your NIC card is working.

```bash
ping 127.0.0.1          # ping loopback — if reply, NIC card is working
ping -c4 127.0.0.1      # limit to 4 packets only
ping localhost           # same as ping 127.0.0.1
```

> If no reply from 127.0.0.1 — your NIC card may need replacing.

**ICMP** = Internet Control Messaging Protocol — the protocol used by ping.

---

## Routing

### `ip route` — Packet Travel Path

```bash
ip route            # show routing table — very important for troubleshooting
```

Output includes:
- `default` = your gateway (router IP address) — entry and exit point for all traffic
- Your network and interface

> Your Wi-Fi router acts as your gateway. All outbound and inbound traffic passes through it.

---

## Checking Internet Connectivity

```bash
ping 8.8.8.8        # primary global DNS server — maintained by Google (NOT Google's IP)
ping 8.8.4.4        # secondary global DNS server — use if primary doesn't respond
```

> `8.8.8.8` is the correct professional way to check internet. Do NOT use `ping google.com` or `ping localhost`.

If no internet, reset the network:

```bash
systemctl restart network       # resets network, auto-gets IP from DHCP server
```

---

## Traceroute — Tracing Network Path

```bash
tracepath www.google.com        # show all networks between you and destination
traceroute www.google.com       # same — output format slightly different
```

Both show how many hops (routers) exist between your machine and the destination. `*` in output means no reply from that hop.

**Difference between ping and tracepath:**

| Command | Purpose |
|---------|---------|
| `ping` | Check if two devices can communicate (ICMP) |
| `tracepath` / `traceroute` | Trace the full network path to destination |

---

## Port Numbers — `ss` Command

```bash
ss -tlupna          # show all listening ports
```

| Flag | Meaning |
|------|---------|
| `t` | TCP sockets |
| `l` | Listening sockets |
| `u` | UDP sockets |
| `p` | Protocol/process |
| `n` | Numbers (not names) |
| `a` | All |

Key output:
- `*:22` or `0.0.0.0:22` = SSH listening on IPv4
- `:::22` = SSH listening on IPv6

For older RHEL versions (pre-7):

```bash
netstat -tlupna     # same function, slightly different output
netstat -tlupna | grep sshd     # filter for SSH
```

> `ss` is the modern command from RHEL 7+. `netstat` works on older versions.

**802-3** in output = IEEE standard for Ethernet — introduced February 3rd, 1980.

---

## Managing IP Addresses — 3 Methods

### Method 1 — NMTUI (Network Manager Text User Interface)

Easiest method. Works in CLI and on EC2 instances.

```bash
nmtui
```

Navigate with arrow keys (no mouse). Options:
- **Edit a connection** — create, modify, or delete profiles
- **Activate a connection** — enable/disable a profile
- **Set system hostname** — assign a hostname

**Profile vs Device:**
- **Device** = the physical/virtual NIC card (e.g. `eno16777736`) — hardware
- **Profile** = the configuration attached to the device (IP, gateway, etc.)

Same device can have multiple profiles, but only **one profile active at a time**.

> Why multiple profiles? If a machine connects to different networks (e.g. office network and lab network), each network needs a separate static IP profile. Switch profiles instead of reconfiguring each time.

To set a static IP in NMTUI:
1. Edit a connection → select profile → Edit
2. Change IPv4 Configuration from `Automatic` to `Manual`
3. Add IP address with CIDR (e.g. `192.168.1.11/24`)
4. Add gateway IP
5. Keep `[X] Automatically connect` checked — starts on boot
6. OK → then apply: `systemctl restart network`

---

### Method 2 — `hostnamectl` and Hostname Commands

```bash
hostname                            # check current hostname
hostnamectl                         # detailed system info including hostname
hostnamectl set-hostname client.example.com     # change hostname
```

---

### Method 3 — Hostname Config File

```bash
vim /etc/hostname       # edit directly, save and quit
hostname                # verify after restart
```

---

## NMCLI — Network Manager Command Line Interface

```bash
nmcli connection show               # list all profiles
nmcli connection show eno16777736   # detailed info for one profile
nmcli device status                 # show device and which profile it's connected to
```

### Connect / Disconnect a Device

```bash
nmcli device disconnect eno16777736     # bring interface down
nmcli device connect eno16777736        # bring interface back up
```

### Create a Dynamic Profile (DHCP)

```bash
nmcli connection add con-name default type ethernet ifname eno16777736
nmcli connection up default             # activate the profile
```

### Create a Static Profile

```bash
nmcli connection add con-name "static" type ethernet ifname eno16777736 \
  ip4 192.168.1.11/24 gw4 192.168.1.1
nmcli connection up static              # activate
```

### Modify an Existing Profile

```bash
nmcli connection modify static ipv4.addresses 192.168.1.40/24
nmcli connection down static && nmcli connection up static      # apply changes
```

### Set Auto-connect Off

```bash
nmcli connection modify static connection.autoconnect no
# no = don't start on boot  |  yes = start on boot (same as NMTUI checkbox)
```

### Fix Getting Both DHCP and Static IP

```bash
nmcli connection modify static ipv4.method manual
# forces static only — prevents DHCP from also assigning an IP
```

---

## Hostname — 3 Ways Summary

| Method | Command |
|--------|---------|
| NMTUI | `nmtui` → Set System Hostname |
| Command | `hostnamectl set-hostname server.example.com` |
| Config file | `vim /etc/hostname` |

---

## Key Takeaway

`ifconfig` / `ip addr` to inspect interfaces. `ping 8.8.8.8` to check internet — not `ping google.com`. `tracepath` to trace hops. `ss -tlupna` to find listening ports. NMTUI is the easiest IP management tool and works in CLI/EC2. NMCLI is the command-line version for scripting and automation. Device = hardware, Profile = config — one device, multiple profiles, one active at a time.
