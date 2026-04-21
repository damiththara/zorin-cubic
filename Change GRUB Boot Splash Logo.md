# 🧊 Change GRUB Boot Splash Logo in Zorin OS

This guide explains how to replace the boot splash logo (Plymouth animation) in Zorin OS.

---

## 📌 Requirements

- Zorin OS (tested on Zorin OS 18)
- Custom image in **PNG format**
- Recommended:
  - Size: **556 × 140 px**
  - File name: `animation-00.png`
  - White logo on a **transparent background**

---

## 📁 Locate Logo Files

The boot splash logo is part of the Plymouth theme:

```bash
/usr/share/plymouth/themes/zorin-logo/animation-00.png
```

👉 Note:
- The animation consists of multiple numbered images (e.g., `animation-00.png`, `animation-01.png`, etc.)
- There may be **two sets of PNG sequences** used for animation

---

## ⚠️ Backup Original File

Before making changes, back up the original:

```bash
sudo cp /usr/share/plymouth/themes/zorin-logo/animation-00.png \
/usr/share/plymouth/themes/zorin-logo/animation-00.png.bak
```

Replace the Logo
Step 1: Prepare Your Custom Image
Ensure file name:

animation-00.png

Match recommended dimensions (556×140 px)
Place it on your Desktop (or adjust path)

Step 2: Copy the File

Open Terminal (Ctrl + Alt + T) and run:

```bash
sudo cp ~/Desktop/animation-00.png /usr/share/plymouth/themes/zorin-logo/
```
Enter your password when prompted
This will overwrite the default frame

Apply Changes

Reboot your system:
```bash
sudo reboot
```
