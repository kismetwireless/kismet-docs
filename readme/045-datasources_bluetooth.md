---
title: "Bluetooth sources"
permalink: /docs/readme/datasources_bluetooth/
excerpt: "Bluetooth datasources capture BT and BTLE scanning and advertised data."
docgroup: "readme"
toc: true
---

## Bluetooth
Bluetooth uses a frequency-hopping system with dynamic MAC addresses and other oddities - this makes sniffing it not as straightforward as capturing Wi-Fi.

### Datasource: Linux HCI Bluetooth

Kismet can use the generic Linux HCI interface for Bluetooth discovery; this uses a generic Bluetooth adapter to perform *active scans* for discoverable Bluetooth classic and BTLE devices.  This is an active scan, not passive monitoring, and reports attributes and advertised information, not packets.

The Linux Bluetooth source will auto-detect supported interfaces by querying the bluetooth interface list.  It can be manually specified with `type=linuxbluetooth`.

The Linux Bluetooth capture uses the 'kismet_cap_linux_bluetooth' tool, and should typically be installed suid-root:  Linux requires root to manipulate the `rfkill` state and the management socket of the Bluetooth interface.

#### Example source
```
source=hci0:name=linuxbt
```

#### Supported Hardware

For simply identifying Bluetooth (and BTLE) devices, the Linux Bluetooth datasource can use any standard Bluetooth interface supported by Linux.

This includes almost any built-in Bluetooth interface, as well as external USB interfaces such as the Sena UD100.

This datasource is available *only* on Linux.

#### Service Scanning

By default, the Kismet Linux Bluetooth data source turns on the Bluetooth interface and enables scanning mode.  This allows it to see broadcasting Bluetooth (and BTLE) devices and some basic information such as the device name, but does not allow it to index services on the device.

Complex service scanning and enumeration will be coming in a future revision.

#### Bluetooth Source Parameters
Linux Bluetooth sources support all the common configuration options such as name, information elements, and UUID.

### Datasource: Ubertooth One (BTLE)

The Ubertooth One is an open-source hardware Bluetooth and BTLE sniffer by Great Scott Gadgets.

Kismet must be compiled with support for `libusb`, `libubertooth`, and `libbtbb`; you will need `libusb-1.0-dev`, `libubertooth-dev`, and `libbtbb-dev` (or the equivalents for your distribution), and you will need to make sure that the `Ubertooth` option is enabled in the output from `./configure`.

#### Ubertooth interfaces

The Ubertooth One in Kismet can be referred to as simply `ubertooth`:

```bash
$ kismet -c ubertooth
```

When using multiple Ubertooth (Uberteeth?) devices, each device is numbered, starting from 0.  The Ubertooth library indexes the devices automatically, and so is dependent on the order the devices were detected.

```bash
$ kismet -c ubertooth-1
```

Kismet will list available Ubertooth devices automatically in the datasources list.

#### Limitations

The Ubertooth One truncates all packets to a maximum of 50 bytes; packets larger than 50 bytes will be discarded and ignored because it is not possible to validate the checksum.

The Ubertooth One firmware (as of 2019-12) appears to have issues setting channels in BTLE mode, leading to frequent firmware crashes which require the USB device to be removed and re-inserted.  Kismet currently disables channel hopping on the Ubertooth One, and defaults to advertising channel 37.

Alternate channels can be set with the `channel=` source option;

```bash
$ kismet -c ubertooth:channel=39
```

To try to mitigate firmware hangs, Kismet will reset the U1 device periodically, which will reboot the U1.  This does not prevent all firmware hangs, however, and you may find it necessary to remove and re-insert the Ubertooth One periodically.

#### Supported Hardware

This datasource works with the [Ubertooth One by Great Scott Gadgets](https://greatscottgadgets.com/ubertoothone/).

This datasource should work on any platform, so long as the appropriate libraries are available.

### Datasource - TI CC2540 (BTLE)

The Texas Instruments CC2540 is a chip used for Bluetooth communications.  To use it with Kismet, it must be flashed with the sniffer firmware [provided by TI](http://www.ti.com/tool/PACKET-SNIFFER).  Often the devices are available with the sniffer firmware pre-flashed.

Kismet must be compiled with support for `libusb` to use TICC2540; you will need `libusb-1.0-dev` (or the equivalent for your distribution), and you will need to make sure that the `TI CC 2540` option is enabled in the output from `./configure`.

To use the TI CC2540 capture, you must have a TI CC2540 dongle flashed with the sniffer firmware. You can flash this yourself with a CC-Debugger or purchase one online from many retailers.

*Note*: It seems that while many CC2540 devices are *advertised* as pre-flashed with the sniffer firmware, they appear not to be!

#### TI CC2540 interfaces

TI CC2540 datasources in Kismet can be referred to as simply `ticc2540`:

```bash
$ kismet -c ticc2540
```

When using multiple TI CC2540 dongles, they can be specified by their location in the USB bus; this can be detected automatically by Kismet as a supported interface in the web UI, or specified manually.  To find the location on the USB bus, look at the output of the command `lsusb`:

```bash
$ lsusb
...
Bus 001 Device 008: ID 0451:16b3 Texas Instruments, Inc. 
Bus 005 Device 004: ID 0451:16b3 Texas Instruments, Inc.
Bus 006 Device 001: ID 1d6b:0001 Linux Foundation 1.1 root hub
...
```

In this instance the first device is on `bus 001` and `device 008` and the second device is on `bus 005` and `device 004`; we can specify this specific first device in Kismet by using:

```bash
$ kismet -c ticc2540-1-8
```

Kismet will list available Ubertooth devices automatically in the datasources list.

#### Supported Hardware

Any USB device based on the CC2540 chip, and flashed with the TI sniffer firmware, should work.  Beware!  Many online vendors sell identical-looking devices based on the CC2531 chip, which is *not* the same thing!

This datasource should work on any platform, so long as the appropriate libraries are available.

### Datasource - nRF 51822 (BTLE)

The nRF 51822 is a chip used for Bluetooth LE communications.   To use it with Kismet, it must be flashed with sniffer firmware provided by NordicRF.  A pre-flashed version is available from Adafruit and other online retailers.

The nRF 51822 utilizes serial communications so no special libraries are needed for use with Kismet, *however* not all platforms have working serial port drivers (see below).

#### nRF 51822 interfaces

nRF 51822 datasources in Kismet can be referred to as simply `nrf51822`.  These devices appear as serial ports, so cannot be auto-detected.  Each nrf51822 source must have a `device` option:

```bash
source=nrf51822:device=/dev/ttyUSB0
```

#### Channel Hopping

The firmware does it's own channel hopping so no selections are available.

#### Limitations

You must specify a `device=` configuration pointing to the serial port this device has been assigned by the kernel; it can not be automatically detected, and will not appear in the datasource list.

Multiple versions of the Adafruit BLE device are sold, and not all have the sniffer firmware.  Make sure you have a V2 or newer device, *with the sniffer firmware*.

Devices without sniffer firmware *may* be flashable, but may require an external programmer.

#### MacOS Limitations

The nRF 51822 uses a CP2104 serial chip; testing with the [latest drivers](https://www.silabs.com/products/development-tools/software/usb-to-uart-bridge-vcp-drivers) under MacOS Catalina has been unsuccessful; however a driver update may fix that in the future.

