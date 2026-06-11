# LogKeys — Linux Keylogger Guide (Ubuntu)

> ⚠️ **Warning:** Keylogging captures passwords, private messages, and sensitive personal data.
> In many jurisdictions it is highly regulated and can be **illegal** if done covertly or without clear consent.

---

## 1. What LogKeys Does

LogKeys is an open source Linux keylogger that:

- Records keystrokes (typewritten text) and stores them in a log file
- Runs in the background once started
- Can be configured to start automatically on boot for long-term monitoring

---

## 2. Installing LogKeys on Ubuntu

### 2.1 Install Dependencies

```bash
sudo apt update
sudo apt install build-essential autotools-dev autoconf kbd
```

### 2.2 Download and Compile LogKeys

```bash
# Download source
wget https://github.com/damiththara/zorin-cubic/blob/main/logkeys-master.zip
sudo unzip master.zip
cd logkeys

# Build and install
sudo ./autogen.sh
cd build
sudo ../configure
sudo make
sudo make install
```

> If the repository URL has changed, check the current [LogKeys GitHub page](https://github.com/kernc/logkeys) for the latest source.

---

## 3. Starting and Stopping LogKeys

### 3.1 Start Logging (with Keymap Fix)

Navigate to the logkeys master folder first:

```bash
cd /home/user/Downloads/logkeys-master/
```

Then start the keylogger:

```bash
sudo logkeys -s -o /var/log/logkeys/logkey.log -m ./keymaps/en_US_ubuntu_1204.map
```

This begins writing keystrokes to `/var/log/logkeys/logkey.log`.

### 3.2 Check the Log

```bash
sudo tail -f /var/log/logkeys/logkey.log
```

> ⚠️ The log file may contain **passwords, personal data, and sensitive information**.
> Store and protect it using encryption, access control, and retention policies.

### 3.3 Stop Logging

```bash
sudo logkeys --kill
```

---

## 4. Auto-Start LogKeys on Boot (Optional)

### Using systemd (Recommended on Ubuntu 20.04+)

**Step 1 — Create the service file:**

```bash
sudo nano /etc/systemd/system/logkeys.service
```

**Step 2 — Paste the following (adjust paths as needed):**

```ini
[Unit]
Description=LogKeys keylogger
After=multi-user.target

[Service]
Type=forking
ExecStart=/usr/local/bin/logkeys -s -o /var/log/logkeys/logkey.log -m /path/to/keymaps/en_US_ubuntu_1204.map
ExecStop=/usr/local/bin/logkeys -k
Restart=no

[Install]
WantedBy=multi-user.target
```

> Replace `/path/to/keymaps/en_US_ubuntu_1204.map` with the absolute path to your map file.

**Step 3 — Enable and start:**

```bash
sudo systemctl enable logkeys.service
sudo systemctl start logkeys.service
```

The keylogger will now start automatically at every boot.

Create directory
sudo mkdir /var/log/logkeys/

---

## 5. Keymap Mismatch Issue & Fix

LogKeys doesn't automatically detect your X11 `us` layout and often requires an explicit keymap file via the `-m` flag. The built-in `-u` (US) option alone frequently fails on modern Ubuntu releases.

### Recommended Fix

Download the keymaps from the [LogKeys GitHub repo](https://github.com/kernc/logkeys/tree/master/keymaps), then run:

```bash
sudo logkeys -s -o /var/log/logkeys/logkey.txt -m ./keymaps/en_US_ubuntu_1204.map
```

> This map works across many Ubuntu versions despite the name.
> Test by typing keys, stopping with `Ctrl+C`, and checking the log file.
> If still incorrect, try the `en_US` map or set a VC keymap first:

```bash
sudo loadkeys us
```

### Verification Steps

1. Set VC keymap explicitly (persists across reboots):
   ```bash
   localectl set-keymap us
   ```
2. Test logkeys — common symbols like `@`, `#`, and quotes should log correctly.
3. If issues remain, inspect `/usr/share/X11/xkb/symbols/us` for your exact variant and check [LogKeys GitHub Issues](https://github.com/kernc/logkeys/issues) for patches.

---

## 6. Security, Privacy & Legal Considerations

Because you are monitoring company-issued PCs used by clients or employees, you should:

- **Inform users clearly** (via written policy, contract, or onboarding document) that:
  - The PCs are company-owned
  - Activity may be monitored for security and compliance
- **Limit logging** to only what is necessary — avoid capturing passwords and extremely sensitive personal data where possible
- **Consult a lawyer or data protection officer** to ensure your setup complies with Sri Lankan law and any applicable international regulations (e.g., GDPR-style obligations)

---

## 7. Alternative: Less Invasive Monitoring Tools

Instead of raw keystroke logging, many organizations use **endpoint or employee monitoring tools** for Linux that:

- Track applications used, websites visited, and time worked
- Generate reports and alerts without saving every keystroke
- Are generally safer legally and easier to manage

> If you need help designing a monitoring setup for specific goals (e.g., fraud detection, compliance, productivity), consider exploring these tools as a more privacy-friendly alternative.

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


sudo nano /etc/logrotate.d/logkeys
and copy below script
```
/var/log/logkeys/logkey.log {
    su root root
    daily
    rotate 30
    missingok
    notifempty
    nocompress
    dateext
    dateformat -%Y-%m-%d
    extension .log
    postrotate

        export GNUPGHOME="/root/.gnupg"
        LOGFILE="/var/log/logkeys/rotate.log"
        PASS="Aquafina@2026"
        LOGDIR="/var/log/logkeys"

        echo "[$(date)] ── Rotate started ──" >> "$LOGFILE"
        ls "$LOGDIR" >> "$LOGFILE"

        FOUND=0
        for FILE in "$LOGDIR"/logkey-*.log; do

            [ -f "$FILE" ] || continue
            [ -f "${FILE}.gpg" ] && continue

            ENC="${FILE}.gpg"
            echo "[$(date)] Encrypting: $FILE" >> "$LOGFILE"

            gpg --yes \
                --batch \
                --no-tty \
                --quiet \
                --pinentry-mode loopback \
                --passphrase "$PASS" \
                --symmetric \
                --cipher-algo AES256 \
                --output "$ENC" \
                "$FILE"

            EXIT=$?

            if [ $EXIT -eq 0 ] && [ -f "$ENC" ]; then
                shred -u "$FILE"
                echo "[$(date)] SUCCESS → $(basename $ENC)" >> "$LOGFILE"
                FOUND=$((FOUND + 1))
            else
                echo "[$(date)] FAILED → $(basename $FILE) (exit: $EXIT)" >> "$LOGFILE"
                rm -f "$ENC"
            fi
        done

        echo "[$(date)] Encrypted $FOUND file(s)" >> "$LOGFILE"

        logkeys --kill 2>/dev/null
        sleep 1
        logkeys -s \
            -o /var/log/logkeys/logkey.log \
            -m /usr/local/logkeys-master/keymaps/en_US_ubuntu_1204.map
        echo "[$(date)] Logkeys restarted" >> "$LOGFILE"
        echo "[$(date)] ── Done ──" >> "$LOGFILE"

    endscript
}
```




