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

### Set a fixed IP address on the PC

The USRP does not get an address automatically, so your PC's port needs a fixed address on the same network. This requires an administrator terminal.

1. Press the Windows key, type **PowerShell**, right-click **Windows PowerShell**, and choose **Run as administrator**.
2. Run the following, replacing `Ethernet` with your adapter's name from the previous step if it's different:

```
   netsh interface ipv4 set address name="Ethernet" static 192.168.10.1 255.255.255.0
```

   If it works, it prints nothing.
3. Back in your normal terminal, confirm the change:

```
   ipconfig
```

   Your adapter should now show `IPv4 Address: 192.168.10.1`.

> **To undo this later** (for example, to use the port for a normal network again), run this in an administrator PowerShell:
>
> ```
> netsh interface ipv4 set address name="Ethernet" dhcp
> ```
### Find the USRP

The default address of the USRP is `192.168.10.2`, but it may have been changed, especially on a shared lab device. UHD can find it without knowing the address:

```
uhd_find_devices
```

The output shows the device's address:

```
--------------------------------------------------
-- UHD Device 0
--------------------------------------------------
Device Address:
    serial: F4B623
    addr: 192.168.10.11
    name:
    type: usrp2
```

Note the `addr` value. You will use it in every command and script from here on.

> **Different network?** If the address does not start with `192.168.10.`, set your PC to an address with the same first three numbers instead. For example, if the USRP is at `192.168.20.5`, use `192.168.20.1` for the PC in the previous step.

> **Nothing found?** Check that the USRP is powered on and the link is `Up`. If Windows shows a firewall prompt, allow access and run the command again.

### Checkpoint

Test the connection with `ping`, using your device's address:

```
ping 192.168.10.11
```

If you get four replies with 0% loss, the USRP is connected.

## 3. Check the device

`uhd_usrp_probe` connects to the USRP and prints a full report of its hardware. Replace the address with yours:

```
uhd_usrp_probe --args="addr=192.168.10.11"
```

The output is long. These are the most important lines to look for:

| Line in the output | What it tells you | Example |
|--------------------|-------------------|---------|
| `Mboard` | Motherboard model | `N210r4` |
| `FW Version` / `FPGA Version` | Firmware and FPGA versions | `12.4` / `11.1` |
| `RX Dboard` → `ID` | RF daughterboard model | `WBX v3` |
| `Freq range` (under RX Frontend) | Frequencies you can receive | `68.750 to 2200.000 MHz` |
| `Gain range` | Available receive gain | `0.0 to 31.5 step 0.5 dB` |
| `Antennas` | Available antenna ports | `TX/RX, RX2, CAL` |

> **Check the frequency range.** The USRP-2932 normally ships with an SBX daughterboard (400 MHz – 4.4 GHz), but boards can be swapped. Always go by what the probe reports, and choose your antenna and test frequencies based on that range.

### Common messages

**`Malformed GPSDO string`**

```
[WARNING] [GPS] gps_ctrl_impl::update_cache(): Malformed GPSDO string: Jackson-Labs, FireFly , Firmware Rev 0.929
```

This is harmless. It comes from the older firmware of the built-in GPSDO, which UHD still detects and uses correctly. Without a GPS antenna connected, the GPSDO simply runs unlocked, which is fine for most experiments.

**Firmware or FPGA version mismatch**

If the probe stops with an error saying the firmware or FPGA image is not compatible, update the images to match your UHD version. First download them:

```
uhd_images_downloader
```

Then write them to the device, using your device's address:

```
uhd_image_loader --args="type=usrp2,addr=192.168.10.11"
```

> **Warning:** Do not power off or unplug the USRP while the images are being written. When it finishes, power-cycle the USRP and run `uhd_usrp_probe` again.

### Checkpoint

If `uhd_usrp_probe` prints the full device report without errors, your USRP is ready to use.

## 4. Set up VS Code

VS Code needs to use the same Python version that UHD works with. If you have several Python versions installed, this step makes sure scripts run with the right one.

### Install the Python extension

1. Open the **Extensions** panel (**Ctrl+Shift+X**).
2. Search for **Python** and install the one published by **Microsoft**.

### Select the Python interpreter

1. Press **Ctrl+Shift+P**, type **Python: Select Interpreter**, and press Enter.
2. Choose the Python version where `import uhd` worked in Section 1 (for example **Python 3.10.11**).
3. Check the bottom-right corner of VS Code. It should now show that version.

### Checkpoint

Create a file named `check_setup.py` with this content:

```python
import uhd
import numpy
import matplotlib

print("UHD:", uhd.get_version_string())
print("NumPy:", numpy.__version__)
print("Matplotlib:", matplotlib.__version__)
```

Run it with the **▶ Run Python File** button in the top-right corner of the editor. If all three versions print without errors, VS Code is set up correctly.

## Safety

- Never transmit without an antenna or a 50 Ω load connected to the TX/RX port.
- Keep the signal at the RX input below about 0 dBm to avoid damaging the receiver.
- Only transmit on frequencies you are licensed to use.