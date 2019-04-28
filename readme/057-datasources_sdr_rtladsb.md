---
title: "SDR rtladsb sources"
permalink: /docs/readme/datasources_sdr_rtladsb/
excerpt: "SDR-based rtladsb sources use the rtl-sdr radio to capture airplane ADSB/Mode-S location and telemetry packets."
docgroup: "readme"
toc: true
---

## SDR
SDR, or software-defined radio, uses special generic hardware to capture radio signals, and performs signal processing in software.

SDR is extremely powerful, but also often extremely brittle - configuring SDR hardware and software to work reliably can be quite difficult.  Kismet is able to use external SDR tools to interface with hardware and utilize some of the power of SDR.

### Datasource - SDR RTLADSB
The rtl-sdr radio is an extremely cheap USB SDR (software defined radio).  While very limited, it is still capable of performing some useful monitoring.

Kismet is able to process ADSB transponder data transmitted from a variety of aircraft; for more information about the ADSB signal and how to decode it, check out the [mode-s introduction](https://mode-s.org/decode/adsb/introduction.html).

To use the rtladsb capture, you must have a rtl-sdr USB device; this cannot be done with normal Wi-Fi hardware because a Wi-Fi card is not able to tune to the needed frequencies, and cannot report raw radio samples that are not Wi-Fi packets.

You will also need a python2 environment, and NumPy; this will usually be available as the `python-numpy` package, or installable via `pip install numpy`

For more information about the rtladsb support, see the README in the capture_sdr_rtladsb directory.

### Using other SDR sources
If you want to use multiple rtl-based sources at once (for instance, rtl433 and rtladsb), you will need multiple rtl-sdr USB devices.
