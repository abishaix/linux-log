# Day 13 — ACL & Managing Services with systemd

## ACL — Access Control Lists

Standard permissions only support user/group/other. **ACL lets you assign permissions to specific users or groups** on top of that — more granular control.

---

## Default ACL

Default ACL applies to files/directories **created inside** a directory — they inherit the ACL automatically.

```bash
setfacl -m d:u:deepak:rwx oppo      # default ACL: deepak gets rwx on anything created in oppo
setfacl -m m::r-- oppo              # set mask (effective permission ceiling)
```

- `d` = default
- `u` = user
- `m` = mask

If ACL is set, `ls -l` shows `+` at the end of permissions. `.` means no ACL.

---

## Common ACL Commands

```bash
getfacl oppo                                    # view ACL on 'oppo' directory
setfacl -m u:deepak:rwx oppo                   # give deepak rwx on oppo
setfacl -x u:abhishek oppo                     # remove ACL for abhishek
setfacl -x g:NIT oppo                          # remove ACL for group NIT
setfacl -b oppo                                # remove ALL ACL (back to standard perms)
```

---

## Copy ACL from One Directory to Another

```bash
getfacl oppo | setfacl --set-file=- apple/     # copy oppo's ACL to apple directory
ls -ld apple
getfacl apple
```

---

## ACL Summary

| Task | Command |
|------|---------|
| View ACL | `getfacl dirname` |
| Add user ACL | `setfacl -m u:username:rwx dir` |
| Add default ACL | `setfacl -m d:u:username:rwx dir` |
| Set mask | `setfacl -m m::r-- dir` |
| Remove user ACL | `setfacl -x u:username dir` |
| Remove all ACL | `setfacl -b dir` |
| Copy ACL | `getfacl source \| setfacl --set-file=- dest` |

---

## Managing Services — systemd

### Process vs Service vs Daemon

- **Process** = a running instance of a launched executable
- **Daemon** = a process that runs continuously in the background (24/7), until shutdown
- **Service** = a daemon managed by systemd (e.g. httpd, sshd)

How to identify a daemon: it usually ends in `d` (e.g. `httpd`, `sshd`, `systemd`)

---

## systemd vs initd

| Feature | systemd (RHEL 7+) | initd (RHEL 6 and older) |
|---------|-------------------|--------------------------|
| PID | 1 (first process) | 1 (first process) |
| Service startup | Parallel (all at once) | Sequential (one by one) |
| Boot speed | Faster | Slower |
| On-demand starting | Yes | No |
| Dependency management | Automatic | Manual |
| Process tracking | Linux cgroups | Basic |
| Manager command | `systemctl` | `service` |

---

## systemd Objects (Units)

systemd manages everything as **units**:

| Unit Type | Examples |
|-----------|---------|
| Services | httpd, sshd, vsftpd, dhcpd |
| Devices | hard disks, USB drives |
| Targets | runlevels (multi-user.target, graphical.target) |
| Mount points | disk mounts |

**`systemctl`** is the command that manages systemd units.

---

## systemctl — Basic Service Management

```bash
systemctl start httpd           # start a service
systemctl stop httpd            # stop a service
systemctl restart httpd         # restart (stop + start)
systemctl reload httpd          # reload config without restarting
systemctl status httpd          # check if running and recent logs
systemctl enable httpd          # start automatically on boot
systemctl disable httpd         # don't start on boot
systemctl is-active httpd       # active or inactive?
systemctl is-enabled httpd      # enabled or disabled?
```

---

## Key Takeaway

systemd is the backbone of service management in RHEL 7+. `systemctl` is your main tool — you'll use it constantly for starting, stopping, enabling, and checking services. The parallel startup is why modern Linux boots so much faster than older versions.
