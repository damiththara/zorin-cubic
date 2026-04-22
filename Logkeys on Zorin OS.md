# Logkeys on Zorin OS

## Important note

`logkeys` is a keyboard input logging tool for Linux. Use it only on systems you own or where you have explicit permission to monitor activity.

## What it does

`logkeys` captures keyboard input and writes it to a log file. It is mainly used for testing, debugging, and authorized monitoring.

## Install on Zorin OS

Zorin OS is Ubuntu-based, so package availability may vary by release. The Logkeys project documentation shows that it can be built from source, and the package includes helper programs and scripts. [web:2][web:6]

### Option 1: Build from source

```bash
sudo apt update
sudo apt install -y git autoconf automake build-essential libtool pkg-config
git clone https://github.com/kernc/logkeys.git
cd logkeys
./autogen.sh
mkdir -p build
cd build
../configure
make
sudo make install
```

### Option 2: Check your repositories

If your version of Zorin OS provides a package, you can search first:

```bash
apt search logkeys
```

If a package appears, install it with:

```bash
sudo apt install logkeys
```

## Basic usage

After installation, run the program with elevated privileges. The Logkeys documentation indicates it can be run directly from the command line or with helper scripts. [web:6]

Example:

```bash
sudo logkeys --help
```

A typical session may involve starting the logger, reproducing the input you need to test, then stopping the service or process afterward.

## Where logs are stored

The log file location depends on how you start the tool and which options you use. Check the help output and the project documentation for your installed version. [web:6]

## Stop and remove

To stop a running instance:

```bash
sudo pkill logkeys
```

To remove a source-built installation, go back to the source folder and run:

```bash
sudo make uninstall
```

If you installed from a package:

```bash
sudo apt remove logkeys
```

## Troubleshooting

- If key names look wrong, check keyboard layout settings.
- If the tool does not start, make sure you launched it with administrator privileges.
- If your system uses a custom keyboard layout, verify that it is configured correctly in Zorin OS Settings. Zorin provides keyboard layout management in the Settings app. [web:5]

## GitHub upload steps

1. Create a new repository on GitHub.
2. Add this file to your local project folder.
3. Run:

```bash
git init
git add logkeys-zorin-os-guide.md
git commit -m "Add Logkeys guide for Zorin OS"
git branch -M main
git remote add origin https://github.com/your-username/your-repo.git
git push -u origin main
```

## License

You may publish this guide in your own repository if all included text is original and does not copy copyrighted documentation verbatim.
