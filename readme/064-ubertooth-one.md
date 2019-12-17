---
title: "Ubertooth One"
permalink: /docs/readme/datasources_ubertooth_one/
excerpt: "The Ubertooth datasource supports the Ubertooth One by Great Scott Gadgets"
docgroup: "readme"
toc: true
---

## Ubertooth One

The Ubertooth One is an open-source hardware Bluetooth and BTLE sniffer by Great Scott Gadgets.

### Datasource - Ubertooth

Kismet must be compiled with support for libusb, libubertooth, and libtbb; you will need `libusb-1.0-dev`, `libubertooth-dev`, and `libtbb-dev` (or the equivalents for your distribution), and you will need to make sure that the `Ubertooth` option is enabled in the output from `./configure`.

#### Ubertooth interfaces

The Ubertooth One in Kismet can be referred to as simply `ubertooth`:

```bash
$ kismet -c ubertooth
```

When using multiple Ubertooth (Uberteeth?) devices, each device is numbered, starting from 0.  The Ubertooth library indexes the devices automatically.

```bash
$ kismet -c ubertooth-1
```

#### Channel Hopping

Btle has 3 advertising channels 37, 38, and 39.
```


