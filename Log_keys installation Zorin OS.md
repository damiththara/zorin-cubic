# 🔐 LogKeys Installation & Usage Guide (Ubuntu)

![License](https://img.shields.io/badge/license-Open%20Source-green)
![Platform](https://img.shields.io/badge/platform-Ubuntu-orange)
![Shell](https://img.shields.io/badge/shell-bash-blue)
![Status](https://img.shields.io/badge/status-Active-success)

A complete guide to installing, configuring, and securely using **LogKeys** on Ubuntu systems.

---

## 📑 Table of Contents

- About
- Features
- Installation
- Usage
- Autostart Setup
- Keymap Fix
- Security & Legal Notice
- Alternatives
- Troubleshooting

---

## 📌 About

LogKeys is an open-source Linux keylogger used for capturing keystrokes and monitoring activity.

⚠️ Use only with proper authorization.

---

## 🚀 Features

- Background keystroke logging  
- Lightweight  
- Autostart support  
- Custom keymap support  

---

## ⚙️ Installation

### Install Dependencies

```bash
sudo apt update
sudo apt install build-essential autotools-dev autoconf kbd
```

### Download & Build

```bash
wget https://github.com/kernc/logkeys/archive/refs/heads/master.zip
unzip master.zip
cd logkeys
./autogen.sh
cd build
../configure
make
sudo make install
```

---

## ▶️ Usage

### Start

```bash
sudo logkeys -s -o /var/log/logkeys/logkey.log -m ./keymaps/en_US_ubuntu_1204.map
```

### View Logs

```bash
sudo tail -f /var/log/logkeys/logkey.log
```

### Stop

```bash
sudo logkeys --kill
```

---

## 🔁 Autostart

Create service:

```bash
sudo nano /etc/systemd/system/logkeys.service
```

Paste:

```ini
[Unit]
Description=LogKeys
After=multi-user.target

[Service]
Type=forking
ExecStart=/usr/local/bin/logkeys -s -o /var/log/logkeys/logkey.log -m /path/to/keymap
ExecStop=/usr/local/bin/logkeys -k

[Install]
WantedBy=multi-user.target
```

Enable:

```bash
sudo systemctl enable logkeys
sudo systemctl start logkeys
```

---

## ⌨️ Keymap Fix

```bash
localectl set-keymap us
```

---

## 🔐 Legal Notice

- Inform users
- Get consent
- Protect logs

---

## 🛠️ Troubleshooting

- Wrong keys → fix keymap  
- No logs → check permissions  
- Service fail → check paths  

---

## 📁 Log Location

/var/log/logkeys/logkey.log
