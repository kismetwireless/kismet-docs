---
title: "nRF 51822 sources"
permalink: /docs/readme/datasources_nrf_51822/
excerpt: "nRF 51822 based datasources use a nrf51822 dongle with sniffer firmware to monitor Bluetooth LE"
docgroup: "readme"
toc: true
---

## nRF 51822

The nRF 51822 is a chip used for Bluetooth LE communications.   To use it with Kismet, it must be flashed with sniffer firmware provided by NordicRF.  A pre-flashed version is available from Adafruit and other online retailers.

### Datasource - nRF 51822

The nRF 51822 utilizes serial communications so no special libraries are needed for use with Kismet.

#### nRF 51822 interfaces

nRF 51822 datasources in Kismet can be referred to as simply `nrf51822`.  These devices appear as serial ports, so cannot be auto-detected.  Each nrf51822 source must have a `device` option:

```bash
source=nrf51822:device=dev/ttyUSB0
```

#### Channel Hopping

The firmware does it's own channel hopping so no selections are available.
```

### MacOS

The nRF 51822 uses a CP2104 serial chip; testing with the [latest drivers](https://www.silabs.com/products/development-tools/software/usb-to-uart-bridge-vcp-drivers) under MacOS Catalina has been unsuccesful; however a driver update may fix that in the future.
