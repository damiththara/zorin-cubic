# 🧊 Change "About" Window Logo in Zorin OS

This guide explains how to replace the Zorin OS logo shown in the **About** window with a custom image.

---

## 📌 Requirements

- Zorin OS (tested on Zorin OS 18)
- Custom logo in **PNG or SVG format**
- Recommended:
  - Size: **254 × 128 px**
  - File name: `zorin-logo-text.png`
  - White logo on a **transparent background**

---

## 📁 Locate Logo Files

The About window uses logo files from system directories.

Common locations:

/usr/share/pixmaps/zorin-logo-text.png
/usr/share/icons/Yaru/scalable/status/zorin-logo-text.png


To search manually:

```bash
locate zorin-logo.svg
```
# or
```bash
find /usr/share -name "*zorin*logo*"
```

## ⚠️ Backup Original File

Always back up the original before replacing:

```bash
sudo cp /usr/share/pixmaps/zorin-logo-text.png \
/usr/share/pixmaps/zorin-logo-text.png.bak
```

🔁 Replace the Logo
Step 1: Prepare Your Custom Logo

Ensure file name:

zorin-logo-text.png
Match recommended dimensions (254×128 px)
Place it on your Desktop (or adjust path)
Step 2: Copy the File

Open Terminal (Ctrl + Alt + T) and run:

```bash
sudo cp ~/Desktop/zorin-logo-text.png /usr/share/pixmaps/
```

Enter your password when prompted
This will overwrite the default logo
🔄 Update Cache

Run:
```bash
sudo gtk-update-icon-cache /usr/share/pixmaps/
```
🔁 Apply Changes

Log out and back in, or reboot:
```bash
sudo reboot
```
