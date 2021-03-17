---
title: "802.15.4 sources"
permalink: /docs/readme/datasources_802154/
excerpt: "Zigbee datasources capture 802.15.4 data."
docgroup: "readme"
toc: true
---

## 802.15.4 (Zigbee) 
802.15.4 is a low rate networking system which can be hard to detect since not as much traffic as sent compared to Wi-Fi.

### Datasource - TI CC2531 

The Texas Instruments CC2531 is a chip used for 802.15.4 communications.  To use it with Kismet, it must be flashed with the sniffer firmware [provided by TI](http://www.ti.com/tool/PACKET-SNIFFER).  Often the devices are available with the sniffer firmware pre-flashed.

Kismet must be compiled with support for `libusb` to use TICC2531; you will need `libusb-1.0-dev` (or the equivalent for your distribution), and you will need to make sure that the `TI CC 2531` option is enabled in the output from `./configure`.

To use the TI CC2531 capture, you must have a TI CC2531 dongle flashed with the sniffer firmware. You can flash this yourself with a CC-Debugger or purchase one online from many retailers.

*Note*: It seems that while many CC2531 devices are *advertised* as pre-flashed with the sniffer firmware, they appear not to be!

#### TI CC2531 interfaces

TI CC2531 datasources in Kismet can be referred to as simply `ticc2531`:

```bash
$ kismet -c ticc2531
```

When using multiple TI CC2531 dongles, they can be specified by their location in the USB bus; this can be detected automatically by Kismet as a supported interface in the web UI, or specified manually.  To find the location on the USB bus, look at the output of the command `lsusb`:

```bash
$ lsusb
...
Bus 001 Device 008: ID 0451:16ae Texas Instruments, Inc. 
Bus 005 Device 004: ID 0451:16ae Texas Instruments, Inc.
Bus 006 Device 001: ID 1d6b:0001 Linux Foundation 1.1 root hub
...
```

In this instance the first device is on `bus 001` and `device 008` and the second device is on `bus 005` and `device 004`; we can specify this specific first device in Kismet by using:

```bash
$ kismet -c ticc2531-1-8
```

Kismet will list available TI CC2531 devices automatically in the datasources list.

#### Supported Hardware

Any USB device based on the CC2531 chip, and flashed with the TI sniffer firmware, should work.  Beware!  Many online vendors sell identical-looking devices based on the CC2540 chip, which is *not* the same thing!

This datasource should work on any platform, so long as the appropriate libraries are available.

### Datasource - nRF 52840 (Zigbee)

The nRF 52840 is a chip used for 2.4Ghz communications, including 802.15.4.   To use it with Kismet, it must be flashed with sniffer firmware [provided by NordicRF](https://github.com/NordicSemiconductor/nRF-Sniffer-for-802.15.4).

The nRF 52840 utilizes serial communications so no special libraries are needed for use with Kismet, *however* not all platforms have working serial port drivers (see below).

#### nRF 52840 interfaces

nRF 52840 datasources in Kismet can be referred to as simply `nrf52840`.  These devices appear as serial ports, so cannot be auto-detected.  Each nrf52840 source must have a `device` option:

```bash
source=nrf52840:device=/dev/ttyUSB0
```

#### Limitations

You must specify a `device=` configuration pointing to the serial port this device has been assigned by the kernel; it can not be automatically detected, and will not appear in the datasource list.

Devices do not come with the sniffer firmware pre-flashed so some effort is required by the user.

### Datasource - AVR RZUSBSTICK 

The AVR RZUSBSTICK is a chip used for 802.15.4 communications.  To use it with Kismet, the stock firmware will work. The device can also be flashed with the Killerbee firmware [provided by River Loop Security](https://github.com/riverloopsec/killerbee).

Kismet must be compiled with support for `libusb` to use AVR RZUSBSTICK; you will need `libusb-1.0-dev` (or the equivalent for your distribution), and you will need to make sure that the `RZ KILLERBEE` option is enabled in the output from `./configure`.

#### AVR RZUSBSTICK interfaces

AVR RZUSBSTICK datasources in Kismet can be referred to as simply `rzkillerbee`:

```bash
$ kismet -c rzkillerbee
```

When using multiple AVR RZUSBSTICK dongles, they can be specified by their location in the USB bus; this can be detected automatically by Kismet as a supported interface in the web UI, or specified manually.  To find the location on the USB bus, look at the output of the command `lsusb`:

```bash
$ lsusb
...
Bus 001 Device 008: ID 03eb:210a Atmel Corp. AT86RF230 [RZUSBSTICK] transceiver
Bus 005 Device 004: ID 03eb:210a Atmel Corp. AT86RF230 [RZUSBSTICK] transceiver
Bus 006 Device 001: ID 1d6b:0001 Linux Foundation 1.1 root hub
...
```

In this instance the first device is on `bus 001` and `device 008` and the second device is on `bus 005` and `device 004`; we can specify this specific first device in Kismet by using:

```bash
$ kismet -c rzkillerbee-1-8
```

Kismet will list available AVR RZUSBSTICK devices automatically in the datasources list.

### Datasource - NXP KW41Z

The NXP KW41Z is a chip used for Bluetooth LE and 802.15.4 communications.   To use it with Kismet, it must be flashed with sniffer firmware provided by NXP. It comes with this firmware by default.

The NXP KW41Z utilizes serial communications so no special libraries are needed for use with Kismet, *however* not all platforms have working serial port drivers (see below).

#### NXP KW41Z interfaces

NXP KW41Z datasources in Kismet can be referred to as simply `nxp_kw41z`.  These devices appear as serial ports, so cannot be auto-detected.  Each nxp_kw41z source must have a `device` option:

```bash
source=nxp_kw41z:device=/dev/ttyUSB0
```

The NXP KW41Z can monitor both Bluetooth LE and 802.15.4. By default it will try to monitor all,

The specify only zigbee

```bash
source=nxp_kw41z:device=/dev/ttyUSB0,phy=zigbee
```
