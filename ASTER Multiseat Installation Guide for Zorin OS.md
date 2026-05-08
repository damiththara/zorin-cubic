# ASTER Multiseat Installation Guide for Zorin OS

This guide explains how to install and configure ASTER Multiseat Software on Zorin OS.

ASTER allows multiple users to use one PC independently with separate monitors, keyboards, and mice.

---

# Requirements

Before installing ASTER, prepare:

- Zorin OS installed and updated
- Extra monitors
- Additional keyboards and mice
- Optional extra GPU(s)
- Internet connection for activation
- Administrator (`sudo`) access

ASTER supports multiple workplaces on one computer using connected peripherals.

---

# Step 1 — Update Zorin OS

Open Terminal and run:

```bash
sudo apt update && sudo apt upgrade -y
```

---

# Step 2 — Download ASTER

Open the official ASTER download page:

https://ibiksoft.com/download-aster-multiseat-software/

Download the Linux/Ubuntu `.deb` installer package from the **Linux x64 Ubuntu22.04** section.

Download:

```text
linux-x64-Ubuntu22.04-1.0.8.zip
```

The file will usually be saved in:

```bash
~/Downloads
```

---

# Step 3 — Install ASTER

Open Terminal:

```bash
cd ~/Downloads
```

Unzip the archive:

```bash
unzip linux-x64-Ubuntu22.04-1.0.8.zip
```

Install the package:

```bash
sudo apt install ./astermultiseat-1.0.4-Linux.deb
```

If dependency errors appear:

```bash
sudo apt --fix-broken install
```

---

# Step 4 — Start ASTER

Run ASTER using the following commands:

```bash
cd /opt/ibiksoftware/astermultiseat/
sudo ./astermultiseat
```

---

# YouTube Guide

https://www.youtube.com/watch?v=8kQWIWcsH08

---

# Notes

- ASTER works best when all monitors, keyboards, and mice are connected before starting the software.
- For NVIDIA GPUs, proprietary NVIDIA drivers are recommended.
- If graphical issues occur, try logging into an **Xorg** session instead of Wayland.

---

# Troubleshooting

## Dependency Issues

Run:

```bash
sudo apt --fix-broken install
```

Then retry the installation.

## ASTER Does Not Start

Try running it with administrator privileges:

```bash
sudo ./astermultiseat
```

## Device Detection Problems

- Reconnect USB devices
- Reboot the system
- Ensure GPU drivers are properly installed

---

# Official Links

- ASTER Official Website  
  https://ibiksoft.com/

- ASTER Download Page  
  https://ibiksoft.com/download-aster-multiseat-software/

- ASTER Documentation  
  https://www.dokwiki.ibiksoft.com/
