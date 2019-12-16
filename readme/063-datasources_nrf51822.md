---
title: "nRF 51822 sources"
permalink: /docs/readme/datasources_nrf_51822/
excerpt: "nRF 51822 based datasources use a nrf51822 dongle with sniffer firmware to monitor Bluetooth LE. IE Adafruit BLE Sniffer"
docgroup: "readme"
toc: true
---

## nRF 51822

The nRF 51822 is a chip used for Bluetooth LE communications. It can be obtained in as a usb dongle that can be flashed with a sniffer firmware provided by nRF. A pre-flashed version is available from Afafruit

### Datasource - nRF 51822

The nRF 51822 utilizes serial communications so no special libraries are needed for use with Kismet.

#### nRF 51822 interfaces

nRF 51822 datasources in Kismet can be referred to as simply `nrf51822`. Due to them being serial devices the com port used will need to be defined in the config.

```bash
source=nrf51822:device=dev/ttyUSB0
```

#### Channel Hopping

The firmware does it's own channel hopping so no selections are available.
```

