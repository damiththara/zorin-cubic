# 🔐 Logkeys + Logrotate + GPG Encryption on Ubuntu

> Automatically rotate and GPG-encrypt logkeys keystroke logs on Ubuntu — keeping logs secure at rest and unreadable without your private key.

---

## Table of Contents

- [Overview](#overview)
- [Prerequisites](#prerequisites)
- [Step 1 — Install logkeys](#step-1--install-logkeys)
- [Step 2 — Start logkeys](#step-2--start-logkeys)
- [Step 3 — Generate a GPG Key](#step-3--generate-a-gpg-key)
- [Step 4 — Create the Encryption Script](#step-4--create-the-encryption-script)
- [Step 5 — Configure logrotate](#step-5--configure-logrotate)
- [Step 6 — Test the Setup](#step-6--test-the-setup)
- [Step 7 — Decrypt Logs](#step-7--decrypt-logs)
- [Directory Structure](#directory-structure)
- [Troubleshooting](#troubleshooting)
- [Security Notes](#security-notes)

---

## Overview

```
logkeys daemon
     │
     ▼  writes
/var/log/logkeys.log
     │
     ▼  logrotate (daily)
logkeys.log.1  ──► postrotate script ──► gpg --encrypt ──► logkeys.log.1.gpg
                                                        └──► shred logkeys.log.1
```

| Component | Role |
|---|---|
| `logkeys` | Captures keystrokes to a log file |
| `logrotate` | Rotates the log daily, triggers post-rotate hook |
| `gpg` | Encrypts the rotated log with your public key |
| `shred` | Securely deletes the plaintext rotated file |

---

## Prerequisites

| Requirement | Notes |
|---|---|
| Ubuntu 20.04 / 22.04 / 24.04 | Other Debian-based distros should work |
| Root / sudo access | Required for logkeys and logrotate |
| GPG key pair | Generated in Step 3 |
| `shred` | Pre-installed on Ubuntu (`coreutils`) |

---

## Step 1 — Install logkeys

```bash
sudo apt update
sudo apt install logkeys -y
```

Verify the installation:

```bash
which logkeys
logkeys --version
```

---

## Step 2 — Start logkeys

### Option A — Run manually (testing)

```bash
sudo logkeys --start --output /var/log/logkeys.log
```

### Option B — Run as a systemd service (recommended)

Create the service file:

```bash
sudo nano /etc/systemd/system/logkeys.service
```

Paste the following:

```ini
[Unit]
Description=logkeys keystroke logger
After=network.target

[Service]
ExecStart=/usr/bin/logkeys --start --output /var/log/logkeys.log
Restart=always
RestartSec=5

[Install]
WantedBy=multi-user.target
```

Enable and start:

```bash
sudo systemctl daemon-reload
sudo systemctl enable --now logkeys
sudo systemctl status logkeys
```

Confirm the log file is being written:

```bash
sudo tail -f /var/log/logkeys.log
```

---

## Step 3 — Generate a GPG Key

> Skip this step if you already have a GPG key pair you want to use.

### Generate interactively

```bash
gpg --full-generate-key
```

Recommended settings:

```
Key type:    RSA and RSA
Key size:    4096
Expiry:      1y  (or 0 for no expiry)
Name/Email:  use a real identifier (e.g. you@example.com)
```

### Verify the key exists

```bash
gpg --list-keys
```

Expected output:

```
pub   rsa4096 2024-01-01 [SC] [expires: 2025-01-01]
      ABCDEF1234567890ABCDEF1234567890ABCDEF12
uid           [ultimate] Your Name <you@example.com>
sub   rsa4096 2024-01-01 [E] [expires: 2025-01-01]
```

### Import the public key into root's keyring

The postrotate script runs as **root**, so root needs access to the public key:

```bash
# Export your public key
gpg --export -a "you@example.com" | sudo tee /root/logkeys-pubkey.asc

# Import into root's GPG keyring
sudo gpg --homedir /root/.gnupg --import /root/logkeys-pubkey.asc

# Mark it as trusted (required for non-interactive encryption)
sudo gpg --homedir /root/.gnupg --edit-key "you@example.com"
```

Inside the GPG prompt:

```
gpg> trust
Your decision? 5  (ultimate trust)
gpg> quit
```

---

## Step 4 — Create the Encryption Script

```bash
sudo nano /usr/local/bin/encrypt-logkeys.sh
```

Paste the following:

```bash
#!/bin/bash
# encrypt-logkeys.sh
# Encrypts a rotated logkeys log file with GPG and securely deletes the plaintext.

FILE="$1"
RECIPIENT="you@example.com"   # ← change to your GPG key email or fingerprint
GPG_HOME="/root/.gnupg"

if [[ -z "$FILE" ]]; then
    echo "Usage: $0 <file-to-encrypt>" >&2
    exit 1
fi

if [[ ! -f "$FILE" ]]; then
    echo "File not found: $FILE" >&2
    exit 1
fi

# Encrypt
gpg --homedir "$GPG_HOME" \
    --batch \
    --yes \
    --trust-model always \
    --encrypt \
    --recipient "$RECIPIENT" \
    --output "${FILE}.gpg" \
    "$FILE"

if [[ $? -eq 0 ]]; then
    echo "Encrypted: ${FILE}.gpg"
    shred -v -u "$FILE"
    echo "Plaintext deleted: $FILE"
else
    echo "GPG encryption FAILED for $FILE" >&2
    exit 2
fi
```

Make it executable:

```bash
sudo chmod 700 /usr/local/bin/encrypt-logkeys.sh
sudo chown root:root /usr/local/bin/encrypt-logkeys.sh
```

Test the script in isolation:

```bash
echo "test log entry" | sudo tee /tmp/test-logkeys.log
sudo /usr/local/bin/encrypt-logkeys.sh /tmp/test-logkeys.log
ls /tmp/test-logkeys.log*
# Should show: test-logkeys.log.gpg (plaintext gone)
```

---

## Step 5 — Configure logrotate

```bash
sudo nano /etc/logrotate.d/logkeys
```

Paste the following:

```
/var/log/logkeys.log {
    daily
    rotate 30
    missingok
    notifempty
    create 0600 root root
    dateext
    dateformat -%Y%m%d
    sharedscripts
    postrotate
        /usr/local/bin/encrypt-logkeys.sh /var/log/logkeys.log.1 >> /var/log/logkeys-rotate.log 2>&1
    endscript
}
```

### Config options explained

| Option | Meaning |
|---|---|
| `daily` | Rotate once per day |
| `rotate 30` | Keep 30 encrypted archives |
| `missingok` | No error if log file is missing |
| `notifempty` | Skip rotation if log is empty |
| `create 0600 root root` | Create fresh log with strict permissions |
| `dateext` + `dateformat` | Append date to rotated file name |
| `sharedscripts` | Run `postrotate` once even if multiple files match |
| `postrotate` | Script to run after rotation |

> **Why `.log.1`?** logrotate renames `logkeys.log` → `logkeys.log.1` before running `postrotate`. The script always targets `.log.1`.

---

## Step 6 — Test the Setup

### Syntax check (dry run)

```bash
sudo logrotate --debug /etc/logrotate.d/logkeys
```

### Force a real rotation

```bash
sudo logrotate --force /etc/logrotate.d/logkeys
```

### Verify results

```bash
ls -lh /var/log/logkeys*
```

Expected output:

```
-rw------- 1 root root    0 Jan 15 00:00 /var/log/logkeys.log         ← fresh empty log
-rw------- 1 root root  24K Jan 14 00:00 /var/log/logkeys.log-20240114.gpg  ← encrypted archive
```

Check the rotation log:

```bash
sudo cat /var/log/logkeys-rotate.log
```

---

## Step 7 — Decrypt Logs

Decryption requires your **private** GPG key on the machine you're decrypting on.

### Decrypt to a file

```bash
gpg --decrypt /var/log/logkeys.log-20240114.gpg > /tmp/logkeys-decrypted.log
```

### Decrypt and pipe directly to a pager

```bash
gpg --decrypt /var/log/logkeys.log-20240114.gpg | less
```

### Decrypt and search

```bash
gpg --decrypt /var/log/logkeys.log-20240114.gpg | grep "password"
```

> 🔑 **Keep your private key safe.** If it is lost, encrypted logs are permanently unreadable.

---

## Directory Structure

After setup, your relevant files will be:

```
/
├── etc/
│   ├── logrotate.d/
│   │   └── logkeys                    ← logrotate config
│   └── systemd/system/
│       └── logkeys.service            ← systemd unit
├── usr/local/bin/
│   └── encrypt-logkeys.sh             ← postrotate encryption script
├── var/log/
│   ├── logkeys.log                    ← active log (plaintext, root-only)
│   ├── logkeys.log-20240114.gpg       ← encrypted archive
│   └── logkeys-rotate.log             ← rotation audit log
└── root/
    ├── .gnupg/                        ← root's GPG keyring
    └── logkeys-pubkey.asc             ← exported public key
```

---

## Troubleshooting

### GPG encryption fails: `No public key`

```bash
# Confirm the key is in root's keyring
sudo gpg --homedir /root/.gnupg --list-keys

# Re-import if missing
sudo gpg --homedir /root/.gnupg --import /root/logkeys-pubkey.asc
```

### logrotate does not run postrotate

```bash
# Check logrotate state file — it tracks last run date
sudo cat /var/lib/logrotate/status | grep logkeys

# Force rotation regardless of date
sudo logrotate --force /etc/logrotate.d/logkeys
```

### Script exits with code 2 (GPG failed)

```bash
# Test GPG manually as root
sudo gpg --homedir /root/.gnupg \
    --batch --yes --trust-model always \
    --encrypt --recipient "you@example.com" \
    --output /tmp/test.gpg /etc/hostname

# Decrypt to verify
sudo gpg --homedir /root/.gnupg --decrypt /tmp/test.gpg
```

### `shred` is ineffective on SSD

`shred` overwrites disk blocks and is less reliable on SSDs and filesystems with journaling (ext4, btrfs). Mitigate by using full-disk encryption (LUKS/`cryptsetup`) as the primary layer of protection, and treat `shred` as a best-effort supplement.

---

## Security Notes

| Risk | Mitigation |
|---|---|
| Private key stored on same machine | Store offline (USB / paper backup / hardware key) |
| Log readable between write and rotation | Set `create 0600 root root`; access controls on `/var/log` |
| SSD does not fully erase on `shred` | Use LUKS full-disk encryption as primary protection |
| GPG key expiry | Monitor with `gpg --list-keys`; renew before expiry |
| Rotation audit trail | Check `/var/log/logkeys-rotate.log` regularly |

---

## License

MIT — use freely, modify as needed.

---

*Generated for Ubuntu 20.04 / 22.04 / 24.04. Tested with logkeys 0.1.2, GnuPG 2.2.x.*
