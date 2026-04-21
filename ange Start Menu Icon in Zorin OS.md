# 🧊 Change Start Menu Icon in Zorin OS

This guide explains how to replace the default start menu icon in Zorin OS with a custom SVG icon.

---

## 📌 Requirements

- Zorin OS (tested on Zorin OS 18)
- Custom icon in **SVG format**
- Icon must be:
  - Named: `zorin-icon-symbolic.svg`
  - Recommended size: **16x16 px**
  - White logo on a **transparent background**

---

## 📁 Locate the Original Icon

The default icon is located at:


/usr/share/gnome-shell/extensions/zorin-menu@zorinos.com
/


---

## ⚠️ Backup the Original File

Before making changes, back up the original icon:

```bash
sudo mv /usr/share/gnome-shell/extensions/zorin-menu@zorinos.com/zorin-icon-symbolic.svg \
/usr/share/gnome-shell/extensions/zorin-menu@zorinos.com/zorin-icon-symbolic.svg.bak
🔁 Replace the Icon
Step 1: Prepare Your Custom Icon

Ensure your file is named:

zorin-icon-symbolic.svg
Place it on your Desktop (or note its location)
Step 2: Copy the Icon

Open Terminal (Ctrl + Alt + T) and run:

cd ~/Desktop   # Change this if your file is elsewhere
sudo cp zorin-icon-symbolic.svg /usr/share/gnome-shell/extensions/zorin-menu@zorinos.com/
Enter your password when prompted
This will overwrite the default icon
🔄 Apply Changes

Reboot your system:

sudo reboot
📝 Notes
Your custom icon may reset after system updates — simply repeat the steps if needed
Test your SVG file in an image viewer before applying it
