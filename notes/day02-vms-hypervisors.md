# Day 2 — Virtual Machines & Hypervisors

## Why VMs?

Instead of running everything on one physical machine, we use virtual machines (VMs) so each service gets its own isolated OS.

Common setup:
```
VM — OS — DHCP
VM — OS — DNS
VM — OS — Database
VM — OS — Web Server
```

Each service runs in its own VM. This is how real lab and production environments are structured.

---

## Hypervisors (VM Software)

A hypervisor is the software that creates and manages VMs.

| Hypervisor | Platform |
|------------|----------|
| Hyper-V | Windows (built-in) |
| KVM | Linux (built-in) |
| VirtualBox | Oracle — cross-platform, free |
| VMware Workstation 17.0 Pro | Cross-platform — what we use in class |

> We're using **VMware Workstation 17.0 Pro** for the course labs.

---

## Lab Setup

We'll be spinning up VMs for:
- DHCP
- DNS
- Database
- Web Server

Each will run its own OS instance inside VMware.

---

## Key Takeaway

Virtualization lets you run multiple isolated OS environments on one physical machine. Each VM behaves like its own server — this is how cloud infrastructure works at scale too.
