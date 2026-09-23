# Example: Receiving and plotting a spectrum

In this example, you will receive a slice of the FM radio band with the USRP and plot its spectrum in Python. FM broadcast stations are strong and always on the air, which makes them a good first signal to look for.

Before starting, complete the setup in the [README](../README.md).

## What you need

- A working setup (all checkpoints in the README passed)
- An antenna connected to the **RX2** port. A simple telescopic or wire antenna works for FM.
- Your USRP's address (for example `192.168.10.11`)

> **Frequency range:** FM radio (88–108 MHz) is within range of the WBX daughterboard. If your probe showed an SBX board (400 MHz – 4.4 GHz), choose a frequency within that range instead, for example 433 MHz or 915 MHz.