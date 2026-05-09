# Day 24 — SSH Config, Archiving, Compression & Backup Types

## SSH Configuration File

```bash
vim /etc/ssh/sshd_config      # on the client/server machine
```

Key settings in the file:

| Setting | Default | Notes |
|---------|---------|-------|
| `LoginGraceTime 2m` | 2 minutes | Time allowed to complete login |
| `PermitRootLogin` | yes | Set to `no` to deny root remote access |
| `MaxAuthTries 6` | 6 | Max password attempts |
| `MaxSessions 10` | 10 | Max simultaneous sessions |
| `RSAAuthentication yes` | yes | Enable RSA key auth |
| `PubkeyAuthentication yes` | yes | Enable public key auth |
| `AuthorizedKeysFile .ssh/authorized_keys` | default | Location of public keys |
| `PasswordAuthentication` | yes | Set to `no` to force key-only access |

After editing, restart SSH service:
```bash
systemctl restart sshd
```

> To deny root SSH access: set `PermitRootLogin no`. This is why SCP gave a "permission denied" error when trying to transfer to root — root login was denied in `sshd_config`.

---

## Archiving vs Compression

| Term | What it is | Windows equivalent |
|------|-----------|-------------------|
| Compression | Reducing file size (zip) | WinZip |
| Archive | Bundling multiple compressed files into one | WinRAR |

**The flow:**
1. Compress individual folders/files (zip them)
2. Collect multiple zip files into one archive bundle

> In Linux, the `tar` command handles both compression and archiving together — no need for third-party tools.

---

## `tar` — Tape Archive Command

### Options

| Option | Meaning |
|--------|---------|
| `c` | Create archive (mandatory) |
| `x` | Extract archive |
| `t` | List contents |
| `f` | File name (mandatory) |
| `v` | Verbose — print what's happening (optional) |
| `z` | Compress with gzip (`.tar.gz`) |
| `j` | Compress with bzip2 (`.tar.bz2`) |
| `J` | Compress with xz (`.tar.xz`) |

### Create Archive Only (no compression)

```bash
tar cvf myfile.tar file1 file2 file3 file4
# c = create, v = verbose, f = filename
```

### List Archive Contents

```bash
tar tf myfile.tar
# t = list, f = filename
```

### Archive a Directory

```bash
du -h /etc                  # check directory size first (e.g. 33MB)
tar cf etc.tar /etc         # archive /etc directory
du -h etc.tar               # compare size (~29MB — not much change, no compression)
```

> Archiving alone doesn't reduce size significantly. Use compression options for real savings.

### Create Archive + Compress

```bash
# gzip compression (.tar.gz)
tar czf etc.tar.gz /etc
du -h etc.tar.gz            # ~8.2MB (from 33MB)

# bzip2 compression (.tar.bz2)
tar cjf etc.tar.bz2 /etc
du -h etc.tar.bz2           # ~6.9MB

# xz compression (.tar.xz)
tar cJf etc.tar.xz /etc
du -h etc.tar.xz            # ~5.6MB
```

> Match the extension to the option: `z` → `.tar.gz`, `j` → `.tar.bz2`, `J` → `.tar.xz`

**Compression comparison on 33MB /etc:**

| Method | Size | Option |
|--------|------|--------|
| No compression | ~29MB | `cf` |
| gzip | ~8.2MB | `czf` |
| bzip2 | ~6.9MB | `cjf` |
| xz | ~5.6MB | `cJf` |

---

## Extract Archives

```bash
tar xzf etc.tar.gz          # extract gzip archive
tar xjf etc.tar.bz2         # extract bzip2 archive
tar xJf etc.tar.xz          # extract xz archive
```

> `x` = extract. Match the decompression flag to how it was created.

---

## Transferring Files — SCP, SFTP, Rsync

### SCP — Secure Copy

```bash
scp etc.tar.gz 192.168.1.12:/root/backup
```

Transfers files from source to destination through network. Works over SSH. Upload only.

### SFTP — Secure File Transfer Protocol

```bash
sftp 192.168.1.12           # connect to remote machine
ls                          # list remote files
cd backup                   # navigate remote directory
put etc.tar.bz2             # upload file to remote
get abcd                    # download file from remote to local
```

> SCP = upload only. SFTP = both upload (`put`) and download (`get`).

### Rsync — Incremental Sync

```bash
rsync -av etc.tar.xz 192.168.1.12:/root/backup
```

Best command for backups. Only transfers **new or changed files** — skips files already transferred.

```bash
# Example: directory sync
rsync -av test/ 192.168.1.12:/root/backup
# First run: transfers F1, F2
# Second run (after adding F3, F4): transfers only F3, F4
```

> Use `rsync` over `scp` for regular backups — it's smarter and saves bandwidth.

---

## Types of Backup

### Full Backup

Every backup includes ALL files from day one onwards.

| Day | Files Created | Backup Contains | Time |
|-----|--------------|-----------------|------|
| 1 | F1 | F1 | 5 min |
| 2 | F2 | F1, F2 | 10 min |
| 3 | F3 | F1, F2, F3 | 15 min |
| 4 | F4 | F1, F2, F3, F4 | 20 min |

**Drawback:** Grows larger every day. Consumes more time and storage.

### Differential Backup

Day 1 is full. From day 2 onwards, backs up from day 2 + current day (skips day 1 after day 2).

| Day | Backup Contains | Time |
|-----|-----------------|------|
| 1 | F1 | 5 min |
| 2 | F1, F2 | 10 min |
| 3 | F2, F3 (skips F1) | 10 min |
| 4 | F3, F4 (skips F1, F2) | 10 min |

Still grows over time but less than full backup.

### Incremental Backup

Each backup only includes **that day's new files**. Most efficient.

| Day | Backup Contains | Time |
|-----|-----------------|------|
| 1 | F1 | 5 min |
| 2 | F2 only | 5 min |
| 3 | F3 only | 5 min |
| 4 | F4 only | 5 min |

**Best for production.** `rsync` implements incremental backup — only syncs new/changed files.

> EBS snapshots in AWS also use incremental backup — only the changed blocks are stored after the first snapshot.

---

## How GZIP Compression Works (Bonus)

GZIP uses **Huffman Encoding** algorithm:

1. Raw data (e.g. `XXYZYYXYZ`) = 9 bytes = 72 bits
2. Algorithm identifies patterns using position/length pairs (LZ77)
3. Encodes repeated patterns with fewer bits
4. Result: 72 bits → 14 bits = **80% storage savings**

---

## Key Takeaway

`tar` handles both archiving and compression. `c` = create, `x` = extract, `t` = list, `f` = filename, `z/j/J` = compression type. Use `rsync` for backups — it's incremental (only new files). Full backup = everything every time (slow, heavy). Differential = from day 2 onwards. Incremental = today's files only (fastest, least storage). SCP = upload only. SFTP = upload + download.
