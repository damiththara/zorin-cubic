# ZORIN OS · CUBIC CUSTOM ISO
## Create "This PC" Desktop Shortcut

Full setup guide — icon, launcher, auto-restore on login, and delete protection for all users.

✔ Desktop icon: This PC · ✔ Custom icon works · ✔ Double-click opens user's Home · ✔ No "Allow Launching" issue · ✔ Works for ALL users · ✔ Auto-restores if deleted

---

## PART A — ICON SETUP

### 1. Create folder for custom icon

Inside the **Cubic terminal**, create the custom icons directory:

```bash
mkdir -p /usr/share/icons/custom
```

### 2. Copy your icon file

Copy `this-pc.png` into the custom icons folder:

```bash
# Copy from your working directory into the chroot
cp this-pc.png /usr/share/icons/custom/this-pc.png
```

### 3. Verify the icon

```bash
ls /usr/share/icons/custom/
```

✔ You should see: `this-pc.png`

---

## PART B — DESKTOP ENTRY

### 4. Create Desktop Entry in /etc/skel

This ensures the icon appears for every new user automatically on first login.

```bash
mkdir -p /etc/skel/Desktop
nano /etc/skel/Desktop/ThisPC.desktop
```

Paste the following content:

```ini
[Desktop Entry]
Name=This PC
Comment=Open Home Directory
Exec=bash -c 'xdg-open $HOME'
Icon=/usr/share/icons/custom/this-pc.png
Terminal=false
Type=Application
X-Zorin-Trusted=true
```

> **Fix applied:** `Exec=bash -c 'xdg-open $HOME'` — wrapping in `bash -c` ensures `$HOME` expands to the actual logged-in user's folder (e.g. `/home/zorin`). The previous `xdg-open /home` opened the parent `/home` folder instead.

### 5. Make it executable

```bash
chmod +x /etc/skel/Desktop/ThisPC.desktop
```

---

## PART C — AUTO-TRUST FIX

### 6. Create desktop-fix.sh ⬅ Key Fix

This script fixes the "Allow Launching" issue and restores the icon for all real users. The `chown` line ensures the file is owned by the correct user so the trust metadata sticks properly.

```bash
nano /usr/local/bin/desktop-fix.sh
```

Paste this script:

```bash
#!/bin/bash
# Restore ThisPC desktop icon for ALL real users
SOURCE="/etc/skel/Desktop/ThisPC.desktop"

for USER_HOME in /home/*/; do
    [ ! -d "$USER_HOME" ] && continue
    USER=$(basename "$USER_HOME")
    FILE="$USER_HOME/Desktop/ThisPC.desktop"

    # Create Desktop folder if missing
    mkdir -p "$USER_HOME/Desktop"

    # Restore if deleted
    if [ ! -f "$FILE" ]; then
        cp "$SOURCE" "$FILE"
    fi

    # Set correct ownership (IMPORTANT — trust metadata won't stick without this)
    chown "$USER":"$USER" "$FILE"

    # Make executable
    chmod +x "$FILE"

    # Set trusted metadata as that user
    sudo -u "$USER" gio set "$FILE" metadata::trusted true 2>/dev/null
done

# Also handle live session / zorin default user
for LIVE_USER in zorin liveuser ubuntu; do
    LIVE_HOME="/home/$LIVE_USER"
    if [ -d "$LIVE_HOME" ]; then
        FILE="$LIVE_HOME/Desktop/ThisPC.desktop"
        mkdir -p "$LIVE_HOME/Desktop"
        [ ! -f "$FILE" ] && cp "$SOURCE" "$FILE"
        chown "$LIVE_USER":"$LIVE_USER" "$FILE"
        chmod +x "$FILE"
        sudo -u "$LIVE_USER" gio set "$FILE" metadata::trusted true 2>/dev/null
    fi
done
```

> ⚠ The `chown` line is critical. Without it, the file is owned by root and `gio set metadata::trusted` won't work — causing double-click to do nothing.

Make the script executable:

```bash
chmod +x /usr/local/bin/desktop-fix.sh
```

---

## PART D — AUTO-RESTORE ON EVERY LOGIN

### 7. Add Login Autostart ⬅ New Fix

If the user deletes the icon, it comes back on next login automatically.

```bash
nano /etc/xdg/autostart/desktop-fix.desktop
```

Paste:

```ini
[Desktop Entry]
Name=Desktop Fix
Exec=/usr/local/bin/desktop-fix.sh
Type=Application
X-GNOME-Autostart-enabled=true
NoDisplay=true
```

> ℹ Files in `/etc/xdg/autostart/` run for every user on every login — no systemd needed for this step.

### 7b. Add Trust Autostart ⬅ Additional Fix

This runs `gio trust` as the actual logged-in user inside their own session — the most reliable way to ensure no "Allow Launching" prompt appears.

```bash
nano /etc/xdg/autostart/thispc-trust.desktop
```

Paste:

```ini
[Desktop Entry]
Name=ThisPC Trust Fix
Exec=bash -c 'gio set "$HOME/Desktop/ThisPC.desktop" metadata::trusted true 2>/dev/null; chmod +x "$HOME/Desktop/ThisPC.desktop"'
Type=Application
X-GNOME-Autostart-enabled=true
NoDisplay=true
```

### 8. Copy autostart entries to skel (for new users)

So that any user created after install also gets the autostart entries:

```bash
mkdir -p /etc/skel/.config/autostart

cp /etc/xdg/autostart/desktop-fix.desktop \
   /etc/skel/.config/autostart/desktop-fix.desktop

cp /etc/xdg/autostart/thispc-trust.desktop \
   /etc/skel/.config/autostart/thispc-trust.desktop
```

---

## PART E — SYSTEMD SERVICE

### 9. Attach fix to your existing service

Edit your partition-setup service to also run the fix script at boot:

```bash
nano /etc/systemd/system/partition-setup.service
```

Change the `ExecStart` line:

**Before:**
```
ExecStart=/usr/local/bin/partition-setup.sh
```

**After:**
```
ExecStart=/bin/bash -c '/usr/local/bin/partition-setup.sh && /usr/local/bin/desktop-fix.sh'
```

### 10. Enable the service

```bash
systemctl enable partition-setup.service
```

---

## REFERENCE — Final Setup Summary

| Item | Location | Purpose |
|------|----------|---------|
| Icon file | `/usr/share/icons/custom/this-pc.png` | Custom "This PC" icon |
| Desktop entry template | `/etc/skel/Desktop/ThisPC.desktop` | Source file for all new users |
| Fix script | `/usr/local/bin/desktop-fix.sh` | Restores icon + sets ownership + trusts |
| System autostart | `/etc/xdg/autostart/desktop-fix.desktop` | Runs fix script on every login |
| Trust autostart | `/etc/xdg/autostart/thispc-trust.desktop` | Trusts icon as logged-in user on every login |
| Skel autostart | `/etc/skel/.config/autostart/` | For users created after install |
| Systemd service | `/etc/systemd/system/partition-setup.service` | Runs fix at first boot |

---

## FINAL RESULT — What the user gets after install

| Feature | Status |
|---------|--------|
| "This PC" icon on desktop | ✔ |
| Custom icon visible | ✔ |
| Double-click opens **user's own Home** (e.g. `/home/zorin`) | ✔ |
| No "Allow Launching" prompt | ✔ |
| Auto-restores if deleted | ✔ |
| Works for all users | ✔ |

---

*Zorin OS · Cubic ISO Customization Guide · ThisPC Desktop Shortcut · v4 Final*
