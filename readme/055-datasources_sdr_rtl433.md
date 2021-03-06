---
title: "SDR rtl433 sources"
permalink: /docs/readme/datasources_sdr_rtl433/
excerpt: "SDR-based rtl433 sources use the rtl-sdr radio to capture a wide range of sensors, thermometers, and switches."
docgroup: "readme"
toc: true
---

## Datasource - SDR RTL433

The rtl-sdr radio is an extremely cheap USB SDR (software defined radio).  While very limited, it is still capable of performing some useful monitoring.

Kismet is able to process data from the rtl_433 tool, which can read the broadcasts of a multitude of wireless thermometer, weather, electrical, tire pressure, and other sensors.

To use the rtl433 capture, you must have a rtl-sdr USB device; this cannot be done with normal Wi-Fi hardware because a Wi-Fi card is not able to tune to the needed frequencies, and cannot report raw radio samples that are not Wi-Fi packets.

More information about the rtl-sdr is available at: https://www.rtl-sdr.com

The rtl_433 tool can be downloaded from: https://github.com/merbanan/rtl_433 or as a package in your distribution.

The Kismet rtl_433 interface uses librtlsdr, rtl_433, and Python; rtl433 sources will show up as normal Kismet sources using the rtl433-X naming.

### rtl433 devices

Kismet identifies rtl433 hardware by either the serial number (if any) or by the radio position; for example:

The first rtl433 radio in the system:

`source=rtl433-0`

A specific rtl433 radio (if serial numbers are available):

`source=rtl433-324452324332`

Not all rtl433 hardware populates the serial number field.

Serial numbers can be found using the standard rtlsdr tools, like `rtl_test`:

```bash
$ rtl_test 
Found 4 device(s):
  0:  NooElec, NESDR Nano 3, SN: 2686186936
  1:  NooElec, NESDR Nano 3, SN: 1177459274
  2:  NooElec, NESDR Nano 3, SN: 2664167682
  3:  NooElec, NESDR Nano 3, SN: 0572734167
```

### Using multiple rtlsdr devices

Every datasource in Kismet must have a unique identifier, the source UUID.  Kismet calculates this using the serial number of the rtlsdr device.

Not all rtlsdr hardware supplies a valid serial number; often devices will report a serial number of "00000000".  This will not cause any problems for Kismet if it is the only rtlsdr device, however when using multiple rtlsdr radios either locally or via remote capture, each one must have a unique ID.

A unique ID can be set using the `rtl_eeprom` tool to assign a proper serial number, or by using the `uuid=...` parameter on the Kismet source definition.  A unique UUID can be generated with the `genuuid` tool on most systems.

### rtl433 parameters

RTL433 sources accept several options in the source definition, in addition to the common name, informational, and UUID elements:

* `channel=xyz`

    Manually set the frequency; by default rtl433 tunes to `433.920MHz`.

    Frequency can be in MHz:

    `source=rtl433-0:channel=400MHz`

    or KHz:

    `source=rtl433-0:channel=500000KHz`

    or raw hz:

    `source=rtl433-0:channel=512000000`

* `ppm_error=xyz`

    Set the error correction level for your radio, passed as the `-p` option to rtl433

* `gain=xyz`

    Set the gain, if supported, passed as the `g` option to rtl433

