# 🧊 How To Change Folder Icons on Zorin OS

This guide explains how to install and apply a custom icon theme in Zorin OS.

---

## 📌 Method: Install a Custom Icon Theme Manually

---

## 📥 Step 1: Download an Icon Theme

- Download an icon theme (`.zip` or `.tar.gz`) from:
  - https://www.gnome-look.org/

Example:
- **Win11 Icon Theme** by *yeyushengfan258*

---

## 📁 Step 2: Create Icons Directory

Open Terminal (`Ctrl + Alt + T`) and run:

```bash
cd ~/.local/share/
mkdir -p icons
```
## Step 3: Extract the Theme

Go to your Downloads folder:

```bash
cd ~/Downloads
```

Extract the theme:
```bash
unzip <zip-file-name>
```

Or for .tar.gz files:
```bash
tar -xvf <file-name>.tar.gz
```

## Step 4: Move Theme to Icons Folder

Move the extracted folder:
```bash
mv <extracted-folder> ~/.local/share/icons/
```

## Step 5: Run Install Script (If Available)
Some themes include an install script.

Navigate into the folder:
```bash
cd ~/.local/share/icons/<extracted-folder>
```

If install.sh exists, run:
```bash
sudo ./install.sh
```

If not, you can skip this step.


## 🎨 Step 6: Apply the Icon Theme


Open Zorin Appearance

Go to:
```bash
Themes → Other → Icons
```

Select your newly installed icon theme

📝 Notes

You can install themes system-wide by placing them in:
```bash
/usr/share/icons/
```

- Some themes may require logout/restart to fully apply
- If icons don’t change, ensure the folder structure is correct
- Your folder icons should now be updated with the new theme 🎉
- If you want, I can combine this with your other guides into a **single professional Zorin OS customization README (with sections, navigation, and screenshots)** so your GitHub repo looks polished.
