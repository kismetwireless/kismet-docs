---
title: "TI CC2531 sources"
permalink: /docs/readme/datasources_ticc2531/
excerpt: "TI CC2531 based datasources use a Texas Instruments CC2531 dongle with sniffer firmware to monitor 802.15.4 zigbee."
docgroup: "readme"
toc: true
---

## TI CC2531

The Texas Instruments CC2531 is a chip used for 802.15.4 zigbee communications. It can be obtained in as a usb dongle that can be flashed with a sniffer firmware provided by TI. http://www.ti.com/tool/PACKET-SNIFFER

### Datasource - TI CC2531

Kismet must be compiled with support for libusb to use TICC2531; you will need `libusb-1.0-dev` (or the equivalent for your distribution), and you will need to make sure that the `TI CC 2531` option is enabled in the output from `./configure`.

To use the TI CC2531 capture, you must have a TI CC2531 dongle flashed with the sniffer firmware. You can flash this yourself with a CC-Debugger or purchase one online from many retailers.

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

#### Channel Hopping

Zigbee has 16 advertising channels 11-26
```

