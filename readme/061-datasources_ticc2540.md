---
title: "TI CC2540 sources"
permalink: /docs/readme/datasources_ticc2540/
excerpt: "TI CC2540 based datasources use a Texas Instruments CC2540 dongle with sniffer firmware to monitor btle."
docgroup: "readme"
toc: true
---

## TI CC2540

The Texas Instruments CC2540 is a chip used for bluetooth communications. It can be obtained in as a usb dongle that can be flashed with a sniffer firmware provided by TI. http://www.ti.com/tool/PACKET-SNIFFER

### Datasource - TI CC2540

Kismet must be compiled with support for libusb to use TICC2540; you will need `libusb-1.0-dev` (or the equivalent for your distribution), and you will need to make sure that the `TI CC 2540` option is enabled in the output from `./configure`.

To use the TI CC2540 capture, you must have a TI CC2540 dongle flashed with the sniffer firmware. You can flash this yourself with a CC-Debugger or purchase one online from many retailers.

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

#### Channel Hopping

Btle has 3 advertising channels 37, 38, and 39.
```


