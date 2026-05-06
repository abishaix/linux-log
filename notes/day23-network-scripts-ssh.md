# Day 23 — Network Scripts, Deploying a Network & SSH

## Network Scripts — Direct Config File Editing

The third way to manage IP addresses — edit the config file directly.

```bash
vim /etc/sysconfig/network-scripts/ifcfg-eno16777736
```

Key fields in the file:

| Field | Meaning |
|-------|---------|
| `TYPE=Ethernet` | Interface type |
| `BOOTPROTO=none` | `none` = static, `dhcp` = dynamic |
| `IPADDR=192.168.1.11` | Static IP address |
| `PREFIX=24` | CIDR notation |
| `GATEWAY=192.168.1.1` | Gateway (router IP) |
| `NAME=static` | Profile name |
| `DEVICE=eno16777736` | NIC device name |
| `ONBOOT=yes` | Start on boot |

After editing, apply changes:

```bash
systemctl restart network
# or
nmcli connection reload
nmcli connection up static
```

### Delete a Profile

```bash
nmcli connection delete default
```

> Always keep at least one profile active — without a profile you lose network access.

---

## Deploying a Network Between Two Virtual Machines

### VMware Network Adapter Options

| Option | Use Case |
|--------|----------|
| Bridge | Connect VM to your Wi-Fi network |
| NAT | Connect VM through Ethernet cable |
| Host Only | Private isolated network (VM ↔ host only) |
| Custom (VMNet1) | Virtual cable — connects two VMs to each other |

To connect two VMs on the same isolated network: set **both** VMs to **Custom → VMNet1**. This creates a virtual cable between them.

### Setup Steps

**On both VMs:** Right-click VM → Settings → Network Adapter → Custom → VMNet1

**On Server:**
```bash
nmtui                                           # assign static IP 192.168.1.11/24
hostnamectl set-hostname server.example.com
systemctl restart network
ip addr                                         # verify IP
```

**On Client:**
```bash
nmtui                                           # assign static IP 192.168.1.12/24
hostnamectl set-hostname client.example.com
systemctl restart network
ip addr
```

> No gateway needed for an isolated network — gateway is only required for external communication.

**Test communication:**
```bash
ping 192.168.1.12       # from server, ping client IP
ping 192.168.1.11       # from client, ping server IP
```

### Pinging by Hostname (No DNS Server)

By default, `ping client.example.com` will fail — there's no DNS server to resolve the hostname to an IP.

**Workaround — `/etc/hosts` as local DNS:**

Add entries on **both** machines:

```bash
vim /etc/hosts
# Add:
192.168.1.11    server.example.com
192.168.1.12    client.example.com
```

Now you can ping by hostname:
```bash
ping client.example.com     # works from server
ping server.example.com     # works from client
```

> This is especially useful when setting up Ansible — hosts file lets you reference machines by name without a DNS server.

---

## SSH — Secure Shell

### What is SSH?

SSH is a **protocol** to access a remote Linux machine **securely through the network**.

- Linux → Linux remote access = **SSH**
- Windows → Windows remote access = **RDP** (Remote Desktop Protocol)
- Windows → Linux remote access = third-party tools (PuTTY, MobaXterm, Kitty)

> PuTTY is a terminal emulator for Windows that supports SSH — it lets Windows machines connect to Linux servers. Not needed on Linux or Mac, which have SSH built in.

### Telnet vs SSH

| Feature | Telnet | SSH |
|---------|--------|-----|
| Security | None — plain text | Encrypted |
| Default | Not installed | Built in |
| Use today | Obsolete | Standard |

Telnet sends all data in clear text — anyone on the network can read it. SSH encrypts everything. That's why Telnet was replaced.

Install Telnet (for reference only — not recommended):
```bash
yum install telnet
```

---

## SSH — Basic Usage

### First Connection — Known Hosts Key

When connecting to a machine for the first time, SSH doesn't recognize it and asks:

```bash
ssh u1@client.example.com
# or
ssh u1@192.168.1.12
```

Output:
```
The authenticity of host 'client.example.com (192.168.1.12)' can't be established.
ECDSA key fingerprint is SHA256:...
Are you sure you want to continue connecting (yes/no)?
```

Type `yes` → SSH generates a **known_hosts key** on the server and stores the client's details there. Uses **ECDSA algorithm**.

```
Warning: Permanently added 'client.example.com,192.168.1.12' (ECDSA) to the list of known hosts.
```

From now on, SSH recognizes the client — no more prompt. The key is stored at:

```bash
~/.ssh/known_hosts
```

If you delete `known_hosts`, SSH will ask again on next connection. This also happens when you spin up a new EC2 instance — same prompt, same `yes`.

---

## SSH — Passwordless Authentication (Public/Private Key Pair)

### Why Passwordless?

In production, you may manage 100+ machines. Remembering usernames and passwords for all of them is impractical. Passwordless SSH solves this — access any user on any machine securely without a password.

This is exactly how EC2 instance access works with `.pem` keys.

### Concept — Lock and Key

| | Real World | SSH |
|--|-----------|-----|
| Lock | Put on the door | Public key — assigned to the user |
| Key (chabi) | Kept in your pocket | Private key — kept on your machine |

Anyone can see the lock (public key). Only you have the key (private key). When your private key matches the public key on the remote user — access granted, no password needed.

### Step 1 — Generate Key Pair (on Server)

```bash
ssh-keygen
# Press Enter for all prompts (default location, no passphrase)
```

Keys are saved in `~/.ssh/`:
- `id_rsa` — **private key** (your key/chabi — keep this safe)
- `id_rsa.pub` — **public key** (the lock — copy this to the user)

### Step 2 — Copy Public Key to Remote User

```bash
ssh-copy-id -i id_rsa.pub u1@client.example.com
# Enter u1's password once — this is the last time you'll need it
```

This copies the public key into:
```
/home/u1/.ssh/authorized_keys    (on client machine)
```

`authorized_keys` = the lock placed on u1's account.

### Step 3 — Access Without Password

```bash
ssh u1@client.example.com
# No password prompt — access granted
```

### What Happens If You Delete the Authorized Key?

```bash
# On client machine, logged in as u1:
rm ~/.ssh/authorized_keys
```

SSH will ask for a password again — the lock is gone. Copy the public key again to restore passwordless access.

### EC2 Equivalent

When launching an EC2 instance:
- AWS generates a key pair
- **Public key** = automatically assigned to the EC2 user (`authorized_keys`)
- **Private key** = downloaded as `.pem` file to your machine

That `.pem` file is your private key (chabi). AWS holds the lock. You connect without a password.

---

## SSH Command Reference

```bash
ssh u1@client.example.com          # connect with password
ssh-keygen                          # generate public/private key pair
ssh-copy-id -i id_rsa.pub u1@client.example.com    # copy public key to user
ssh u1@client.example.com          # connect without password (after key copy)
```

---

## Key Takeaway

Network scripts = direct config file editing at `/etc/sysconfig/network-scripts/`. Two VMs communicate via VMNet1 virtual cable in VMware. `/etc/hosts` acts as a local DNS — essential for Ansible setups. SSH = secure remote access between Linux machines. Telnet = obsolete, plain text, insecure. Passwordless auth = generate key pair with `ssh-keygen`, copy public key with `ssh-copy-id`. Public key = lock on the user. Private key = your key. This is exactly how EC2 `.pem` access works.
