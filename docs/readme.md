# Getting Started with the NI USRP-2932 in Python (Windows) 
A step-by-step tutorial for starting with NI USRP-2932 software-defined radio from Python on Windows, using the open-source UHD driver and VS Code.

## What you need 
- NI USRP-2932 
- Ethernet cable
- Windows PC with free Ethernet port (in my case i used the only ethernet port for the USRP and used WiFi)
- Switch (in case you will use multiple USRPs or you dont have WiFi)
- Adequate Antennas

## Contents

1. [Install UHD and the Python bindings](#1-install-uhd-and-the-python-bindings)
2. [Connect the USRP to your PC](#2-connect-the-usrp-to-your-pc)
3. [Check the device](#3-check-the-device)
4. [Set up VS Code](#4-set-up-vs-code)
5. [Receiving and plotting a spectrum](examples/first-receive.md)

## 1. Install UHD and the Python bindings

UHD (USRP Hardware Driver) is the open-source driver from Ettus Research that lets your PC control the USRP. It includes official Python bindings.

### Check what's already installed

Before installing anything, check whether UHD is already on your system. Open a terminal in VS Code (**Terminal → New Terminal**) and run:

```
where.exe uhd_find_devices
python -c "import uhd; print(uhd.get_version_string())"
```

If the second command prints a version number (for example `4.8.0.0-release`), UHD is ready and you can skip to the next section.

### Install UHD

If UHD is not installed, download the Windows installer from Ettus Research:

https://files.ettus.com/binaries/uhd/latest_release/

Run the installer and allow it to add UHD to your PATH. Then open a **new** terminal and run the check commands again.

> **Multiple Python versions?** The UHD Python bindings only work with the Python version they were built for. Check which versions you have with `where.exe python`. If `import uhd` fails, try another version, for example `py -3.10 -c "import uhd"`. Use the version that works for the rest of this tutorial.

### Install the supporting Python libraries

The example uses NumPy for signal processing and Matplotlib for plotting. Check whether they are installed:

```
python -c "import numpy, matplotlib; print(numpy.__version__, matplotlib.__version__)"
```

If you get a `ModuleNotFoundError`, install them:

```
python -m pip install numpy matplotlib
```

### Checkpoint

If both commands print version numbers, your software is ready. For example:

```
4.8.0.0-release
1.26.4 3.10.8
```
## 2. Connect the USRP to your PC

The USRP connects over Gigabit Ethernet. It must be connected **directly** to your PC, not through an office or building network, because it uses a fixed IP address on its own private network.

### Plug it in

1. Connect an Ethernet cable from the USRP's front-panel Ethernet port to a Gigabit Ethernet port on your PC. If your PC's only Ethernet port is already used for your network, use a USB 3.0 Gigabit Ethernet adapter for the USRP, or switch to Wi-Fi for your internet connection.
2. Power on the USRP and wait about 15 seconds for it to boot.

> **Note:** Do not plug the USRP into a wall LAN port. Those connect to your organization's network, where the USRP will not be reachable.

### Check the link

In the terminal, run:

```
Get-NetAdapter
```

Find the adapter the USRP is connected to. Its **Status** should be `Up` and its **LinkSpeed** should be `1 Gbps`. Note its **Name** (for example `Ethernet`), since you will need it in the next step.