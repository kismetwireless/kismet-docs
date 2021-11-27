---
title: "802.15.4 sources"
permalink: /docs/readme/datasources_802154/
excerpt: "Zigbee datasources capture 802.15.4 data."
docgroup: "readme"
toc: true
---

## 802.15.4 (Zigbee) 

The 802.15.4 standard is a low-bandwidth low-power networking standard.  A commercial implementation is Zigbee, however other devices also implement the 802.15.4 physical layer.

Detecting and classifying 802.15.4 networks can be challenging, as they may transmit infrequently.  Often 802.15.4 networks report fragmentary network IDs, which may lead to multiple networks being identified as a single device; this is unavoidable due to how 802.15.4 addressing works.

### 802.15.4 channels

802.15.4 / Zigbee operates in up to 3 bands.  You will need a device for each band.  Most devices support the more common 2.4GHz band, with support for 800 and 900MHz being much rarer (currently only supported via the Freaklabs hardware).

| Band | Channels | Description                   |
| ---- | -------- | ----------                    |
| 800  | 0        | European band, single channel |
| 900  | 1-11     | US / International ISM band   |
| 2400 | 12-26    | US / International ISM band   |

Channels are a fixed width and are identified only by channel number (ie 1, 12, 13, 14).  There are no options for wide or fast channels in 802.15.4.

Despite sharing the frequency range with Wi-Fi on the 2.4GHz band, 802.15.4 uses a different physical encoding standard; a Wi-Fi card is not able to see 802.15.4 packets or networks, and an 802.15.4 device is not able to see Wi-Fi packets.

### Datasource - TI CC2531 

The Texas Instruments CC2531 is a chip used for 802.15.4 communications.  To use it with Kismet, it must be flashed with the sniffer firmware [provided by TI](http://www.ti.com/tool/PACKET-SNIFFER).  Often the devices are available with the sniffer firmware pre-flashed.

Kismet must be compiled with support for `libusb` to use TICC2531; you will need `libusb-1.0-dev` (or the equivalent for your distribution), and you will need to make sure that the `TI CC 2531` option is enabled in the output from `./configure`.

To use the TI CC2531 capture, you must have a TI CC2531 dongle flashed with the sniffer firmware. You can flash this yourself with a CC-Debugger or purchase one online from many retailers.

*Note*: It seems that while many CC2531 devices are *advertised* as pre-flashed with the sniffer firmware, they appear not to be!

#### TI CC2531 shortcomings

The sniffer firmware in the TI CC2531 sometimes goes into a permanent error state until the device is physically re-initialized - by disconnecting it from USB and reconnecting it.  Unfortunately, there does not seem to be any way to automate this process, as once the device enters an error state, it will remain in that state and cannot be reinitialized over USB.

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

### Datasource - Freaklabs

The [Freaklabs 2.4](https://freaklabsstore.com/index.php?main_page=product_info&products_id=215) and [Freaklabs 900](https://freaklabsstore.com/index.php?main_page=product_info&products_id=215) devices support 2.4GHz and 900MHz (depending on model) and are based on the Arduino platform.

To sniff, you will need to provision the device with the [SenSniff](https://github.com/freaklabs/sensniff-freaklabs) firmware (via the Arduino IDE).

#### Configuring the Freaklabs device

To configure a Freaklabs device for sniffing:

* Install the latest official Arduino environment for your system (either via the Arduino website, your distribution packages, or by downloading it for your operating system if on MacOS or Windows)
* Install the Freaklabs board definitions in the Arduino IDE
* Install the Freaklabs chibi-arduino library in the Arduino IDE
* Compile the chibi_ex10_sensniff example code
* Upload the compiled firmware to your device using the Arduino IDE

Further information about adding the required libraries to the Arduino IDE can be found in the [SenSniff](https://github.com/freaklabs/sensniff-freaklabs) git repository.

#### Freaklabs interfaces

The Freaklabs interface in kismet can be referred to as simply `freaklabs`.  These devices appear as serial ports, so cannot be auto-detected. 

Each `freaklabs` source must have a `device` option and a `band` option.  The `device` option tells Kismet where to find the serial device, and the `band` option chooses what radio frequency to set.  Be sure to match the frequency to the type of device you have!

An example source definition:

```bash
source=freaklabs:device=/dev/ttyUSB0,band=900
```

#### Supported options

* `device=/path/to/serial/device`

    Path to the USB serial device.  The user running Kismet must have read/write access to this device.

* `band=800|900|2400`

    Frequency band to tune to.  This must match the supported radio in your device.

