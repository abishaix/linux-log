# Day 9 — Password Management & Group Management

## Password Fields — `/etc/shadow`

9 fields, colon-separated:

| Field | Description |
|-------|-------------|
| username | Login name |
| password | Hashed (`!` = no password or locked) |
| last-changed | Days since epoch (Jan 1, 1970) password last changed |
| min-age | Minimum days before user can change password again |
| max-age | Maximum days before password must be changed |
| warn | Days before expiry to warn the user |
| inactive | Days after expiry before account is disabled |
| expire | Account expiration date (blank = never expires) |
| blank | Reserved |

Example from `/etc/shadow`:
```
u1:!:20563:2:30:4:1:20595:
```
Two colons `::` in `/etc/passwd` for the password field = no password set.

---

## `chage` — Change Password Expiry Info

```bash
chage u1                        # interactive — prompts for each field
chage -l u1                     # list current password aging info
chage --help                    # all available flags
chage -E 2026-05-21 u1          # set account expiration date
chage -W 2 u1                   # set warning days to 2
chage -d 0 u1                   # force password change on next login
```

### Interactive example:
```
Minimum Password Age [0]: 2
Maximum Password Age [99999]: 30
Last Password Change (YYYY-MM-DD) [2026-04-20]:
Password Expiration Warning [7]: 4
Password Inactive [-1]: 1
Account Expiration Date (YYYY-MM-DD) [-1]: 2026-05-22
```

After setting: verify with `tail /etc/shadow` and `chage -l u1`.

---

## Group Management

### Why Groups?

Instead of assigning permissions to 20 individual users, create one group, add all 20, set permissions once. Scales to any size team — HR, Finance, Dev, IT — each with their own access.

---

## Group Properties — `/etc/group`

```
groupname : password : GID : members
```

| File | Purpose |
|------|---------|
| `/etc/group` | Local group database |
| `/etc/gshadow` | Group password database |

---

## Group Commands

```bash
groupadd redhat                     # create a group
groupadd linux -g 2002              # create with custom GID
groupmod -n rhel linux              # rename group (new name first, old name second)
groupmod -og 2002 redhat            # assign same GID to multiple groups
groupmod -p "1234" rhel             # set group password
gpasswd rhel                        # set group password interactively
gpasswd -r rhel                     # remove group password
groupdel rhel                       # delete a group
tail /etc/group                     # verify
tail /etc/gshadow                   # verify password info
```

---

## Adding Users to Groups

```bash
useradd u2 -g redhat                # set primary group at creation time
id u2                               # verify primary group

usermod -aG redhat u1               # add u1 to redhat as secondary group (safe)
usermod -aG redhat u2               # add u2 as well
tail /etc/group                     # verify: redhat:x:1001:u1,u2
id u1                               # check secondary groups
```

- `-G` = secondary (supplementary) group
- `-g` = primary group
- `-a` = append — always use with `-G` so you don't remove existing group memberships

---

## `gpasswd` — Group Password Tool (also manages members)

```bash
gpasswd -M u3,u4 redhat             # set member list (WARNING: OVERWRITES existing)
gpasswd -a u1 redhat                # add single user (does NOT overwrite)
gpasswd -d u3 redhat                # remove u3 from group
gpasswd -A u1 redhat                # make u1 a group admin
tail /etc/gshadow                   # verify: redhat:!:u1:u3,u4
```

> `gpasswd -M` replaces the entire member list — use `usermod -aG` to safely add one at a time without wiping existing members.

---

## `/etc/login.defs` — Login Defaults

Contains default UID/GID ranges, password aging defaults, and other settings applied during user creation.

```bash
cat /etc/login.defs     # view defaults
```

---

## Key Takeaway

Groups are how you scale permissions in a real environment. Know the difference between primary group (`-g`) and secondary group (`-G`). Always use `usermod -aG` over `gpasswd -M` when adding users — `gpasswd -M` is destructive.
