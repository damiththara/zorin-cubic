# 🧊 Change System Name in Zorin OS Boot Screen

This guide explains how to change the displayed system name (title) in the Plymouth boot screen in Zorin OS.

---

## 📌 Requirements

- Zorin OS (tested on Zorin OS 18)
- Root (sudo) access

---

## 📁 Locate Configuration File

The boot screen text is controlled by:

- /usr/share/plymouth/themes/text.plymouth


---

## ⚠️ Backup Original File

Before making changes, create a backup:

```bash
sudo cp /usr/share/plymouth/themes/text.plymouth \
/usr/share/plymouth/themes/text.plymouth.bak
```


---

## ⚠️ Backup Original File

Before making changes, create a backup:

```bash
sudo cp /usr/share/plymouth/themes/text.plymouth \
/usr/share/plymouth/themes/text.plymouth.bak
```

Edit the File

Open the file using a terminal editor:
```bash
sudo nano /usr/share/plymouth/themes/text.plymouth
```

Modify System Name
Find the line:
```bash
title=...
```

Replace it with your desired name:
```bash
title=My Custom Zorin OS
```

Apply Changes
Reboot

Restart your system:
```bash
sudo reboot
```
📝 Notes
- Ensure the file syntax remains correct to avoid boot issues
- Changes may be reset after system updates
- You can restore the original file using the backup if needed
