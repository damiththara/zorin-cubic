# 🧊 Change Login Screen Logo in Zorin OS

This guide explains how to replace the logo shown on the login screen in Zorin OS.

---

## 📌 Requirements

- Zorin OS (tested on Zorin OS 18)
- Custom image in **PNG format**
- Recommended:
  - Size: **206 × 28 px**
  - File name: `zorin-logo.png`
  - White logo on a **transparent background**

---

## 📁 Locate Logo File

The login screen logo is located at:

### /usr/share/plymouth/zorin-logo.png


---

## ⚠️ Backup Original File

Before making changes, back up the original:

```bash id="tq4b6y"
sudo cp /usr/share/plymouth/zorin-logo.png \
/usr/share/plymouth/zorin-logo.png.bak
```

Replace the Logo
Step 1: Prepare Your Custom Image

Ensure file name:

zorin-logo.png

Match recommended dimensions (206×28 px)
Place it on your Desktop (or adjust path)
Step 2: Copy the File

Open Terminal (Ctrl + Alt + T) and run:

```bash
sudo cp ~/Desktop/zorin-logo.png /usr/share/plymouth/
```
Enter your password when prompted
This will overwrite the default logo

Update System

Apply Changes

Log out and back in, or reboot:
```bash
sudo reboot
```
📝 Notes
-Avoid using gtk-update-icon-cache here — Plymouth does not rely on GTK icon cache
-System updates may restore the original logo — reapply if needed
-Incorrect dimensions or file format may prevent the logo from displaying correctly
