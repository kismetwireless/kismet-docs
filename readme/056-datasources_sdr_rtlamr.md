---
title: "SDR rtlamr sources"
permalink: /docs/readme/datasources_sdr_rtlamr/
excerpt: "SDR-based rtlamr sources use the rtl-sdr radio to capture AMR based power and water meter readings."
docgroup: "readme"
toc: true
---

## SDR
SDR, or software-defined radio, uses special generic hardware to capture radio signals, and performs signal processing in software.

SDR is extremely powerful, but also often extremely brittle - configuring SDR hardware and software to work reliably can be quite difficult.  Kismet is able to use external SDR tools to interface with hardware and utilize some of the power of SDR.

### Datasource - SDR RTLAMR
The rtl-sdr radio is an extremely cheap USB SDR (software defined radio).  While very limited, it is still capable of performing some useful monitoring.

Kismet is able to process data from the rtl_amr tool; this tool can capture and process packets from the AMR metering system which can be found on power and water meters in some municipalities.

To use the rtlamr capture, you must have a rtl-sdr USB device; this cannot be done with normal Wi-Fi hardware because a Wi-Fi card is not able to tune to the needed frequencies, and cannot report raw radio samples that are not Wi-Fi packets.

Kismet requires the rtl-amr tool; you will need to build the rtl-amr tool from source, available at [https://github.com/bemasher/rtlamr](https://github.com/bemasher/rtlamr).

The Kismet rtl_amr interface uses librtlsdr, rtl_amr, and Golang; rtlamr sources will show up as normal Kismet sources using rtlamr-X naming.

For more information about the rtlamr support, see the README in the capture_sdr_rtlamr directory.

### Using other SDR sources
If you want to use multiple rtl-based sources at once (for instance, rtl433 and rtlamr), you will need multiple rtl-sdr USB devices.
